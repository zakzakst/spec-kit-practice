---
name: paperclip-dev-workspace-run-verify-fix
description: >
  Paperclip の isolated dev workspace service を起動・検証・再シード・修復します。
  managed な project / worktree service を起動または修復し、health、login readiness、
  clone 済みデータ、runtime visibility、正しい port ownership を証明したいときに使います。
---

# Paperclip 開発ワークスペースの起動・検証・修復

このスキルは、プロジェクト実行ワークスペースのランタイムサービスを通じて起動される、Paperclip 固有の開発ワークスペース向けです。典型例は `paperclip-dev` のような worktree サービスです。

成功条件は次のすべてです:

- サービスが detached workaround ではなく、通常の managed runtime path で起動されていること
- worktree database が primary instance database の完全な bootstrap 済み isolated clone であること
- `/api/health` が `status: ok` と `bootstrapStatus: ready` を返すこと
- root page が `200` を返し、first-admin setup gate を表示しないこと
- board user が通常の開発用 credential でログインできること
- app に、手動で auth user をコピーしただけではない、十分にデータが入った clone data が表示されること
- main control plane でサービスが期待 URL 付きの `running` / `healthy` と表示されること
- 提供中の workspace app もそのサービスを認識し、`running` / `healthy` と表示すること
- service port を所有している process が実際に target workspace のものであること。`readlink /proc/<pid>/cwd` が target worktree 内を指し、兄弟 workspace ではないこと

どれか1つでも失敗したら、修復を続けます。1つの probe が通っただけで issue を完了にしないでください。

## 厳守ルール

- 実 instance の `.env` を持つ primary checkout `/srv/paperclip/home/paperclipai/paperclip` は workspace repair の対象にしません。そこを worktree runtime service の対象にしたり、`<master>/.paperclip/worktrees/` の下に `git worktree add` したり、workspace を修復している間に `.env` を編集したりしないでください。managed workspace は、たとえば `~/.paperclip-worktrees/instances/<slug>/…` のような専用フォルダで動きます。
- `running` / `healthy` の runtime row や `/api/health` の成功だけでは信じないでください。`service:paperclip-dev` の port は service config で固定されているため、兄弟 workspace の process が port を握っていても health check に答えられます。修復の前後で必ず port owner の実際の cwd（`readlink /proc/<pid>/cwd`）を確認してください。runtime row に記録された `cwd` や `pid` は port adoption により実態と異なることがあります。
- Paperclip CLI、dev server、worktree、database、build、test コマンドを実行する前に `doc/DEVELOPING.md` を読んでください。
- worktree と database の操作では、Paperclip CLI と API を source of truth にしてください。通常の修復 path では `psql`、raw な embedded-Postgres コマンド、手作業の row copying を使いません。
- 最終修復として auth row だけを手でコピーしないでください。ログインの証明にはなっても、isolated workspace に完全な bootstrapped database がある証明にはなりません。
- 再利用する workspace service の task では、`pnpm dev` や detached shell process より managed runtime の start / stop route を優先します。
- destructive な git / database 操作は避けます。worktree 内のユーザー変更は保持してください。
- 修復を証明する最小限の検証を実行してください。コードを変更したときだけ、対象を絞ったテストを追加します。
- root cause、正確な fix、verification、必要なら commit link を issue comment に残してください。最終 status も明確に設定します。

## 収集する入力

環境変数が使えるならそれを使います。API key や password は出力しないでください。

- `PAPERCLIP_API_URL`: main control-plane API URL
- `PAPERCLIP_API_KEY`: agent API key
- `PAPERCLIP_RUN_ID`: runtime-service mutation に使う現在の run id
- `PAPERCLIP_TASK_ID`: 現在の issue id
- `PAPERCLIP_COMPANY_ID`: company id
- `PAPERCLIP_AGENT_ID`: 現在の agent id
- worktree service の execution workspace id
- runtime workspace command id（通常は `service:paperclip-dev`）
- 期待する service URL（例: `http://paperclip-dev:40631`）
- ユーザーから提供された場合の dev credential の所有者。password は決して投稿しないでください

execution workspace id か service command id がない場合は、まず issue、project、execution-workspace API record を読みます。推測して random port で unmanaged server を起動しないでください。

## 通常の実行順

ユーザーが workspace を起動したい、再起動したい、あるいは fresh に ready であるべき workspace を修復したいと言ったときに使います。

1. 最新の issue comment を確認し、成功条件を自分の言葉で言い直します。
2. main control plane から現在の runtime state を確認します。
3. 対象 port の競合を確認します。現在の port owner とその実際の cwd を特定し、兄弟 workspace の runtime row が同じ port を主張していないか確認します（「port の競合と workspace の識別」を参照）。
4. ほかの live workspace や agent run が port を所有している場合は、その owner を managed control path から停止し、port が空いたままであることを確認します。対象を再起動する前にこれを行ってください。そうしないと owner が再起動して port を取り戻すことがあります。
5. 対象 runtime service の managed instance を停止します。
6. database が完全でない疑いがある場合は、primary instance から full clone で worktree を reseed します。
7. managed runtime API を通じて runtime service を開始します。
8. main control plane と提供中の workspace app の両方から、health、bootstrap state、login readiness、populated data、runtime visibility、port-owner identity を検証します。
9. どれかの検証項目が失敗したら、その失敗箇所を正確に診断し、最も限定的な修復手順に戻ります。
10. issue にコメントし、最終的な処置を設定します。

## managed start / stop

main control plane の runtime-service endpoint を使います。mutation が現在の heartbeat に
紐づくように、`X-Paperclip-Run-Id` を付けます。

```sh
curl -sS -X POST \
  "$PAPERCLIP_API_URL/api/execution-workspaces/$EXECUTION_WORKSPACE_ID/runtime-services/stop" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
  -H "Content-Type: application/json" \
  --data-binary '{"workspaceCommandId":"service:paperclip-dev"}'
```

```sh
curl -sS -X POST \
  "$PAPERCLIP_API_URL/api/execution-workspaces/$EXECUTION_WORKSPACE_ID/runtime-services/start" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
  -H "Content-Type: application/json" \
  --data-binary '{"workspaceCommandId":"service:paperclip-dev"}'
```

API が既存サービスを返しても、それは候補としてのみ扱います。信じる前に
本物の `/api/health` と port-owner identity を確認してください。

## port の競合と workspace の識別

`service:paperclip-dev` は service config で明示的な port を固定しているため、
その service を実現するすべての workspace が同じ port を要求します。start 時には、
command line が似ていると、service adoption がすでに port を握っている process を
採用します。そして、owner の実 cwd ではなく *要求した* cwd を記録します。
結果は2つあります:

- 複数の workspace が、実際には1つの process しか存在しないのに、同じ port に対して `running` row を持てます。どの workspace URL を開いても、その1つの process が返されます。
- runtime row の `cwd`、`pid`、health はすべて正しく見えても、URL が実際には兄弟 worktree の app を返していることがあります。

識別確認。既存サービスを信じる前と、start / restart のたびにこれを実行します:

```sh
pid=$(lsof -nP -iTCP:"$SERVICE_PORT" -sTCP:LISTEN -t | head -1)
readlink "/proc/$pid/cwd"
```

解決された cwd は target worktree 内にある必要があります（dev server は通常 `<worktree>/server` で動きます）。権威があるのは `/proc/<pid>/cwd` だけです。runtime row の `cwd` field、root の `200`、healthy な `/api/health` を identity の証拠として受け取らないでください。横取りした sibling ならその3つ全部に通ってしまいます。

port owner が兄弟 workspace の process だった場合（port の占有）:

1. 今この pinned port をどの workspace が持つべきかを決めます。通常は issue に
   書かれた workspace です。
2. 現在の owner の workspace と、それを生かし続けているのが live agent run なのか
   managed runtime service なのかを特定します。実際の `/proc/<pid>/cwd`、兄弟
   execution-workspace record、owner の issue/run state を使い、target runtime row から
   ownership を推測しないでください。
3. まず owner 側をその managed control path で止めます。兄弟 workspace の runtime
   service を止めるか、監督して再起動している live run を stop / cancel します。
   supervisor がまだ生きている間に sibling process を直接 kill しないでください。
   すぐに戻ってきて port を取り返します。
4. port が安定して unbound のままかを確認します。すぐに再び bind されるなら、
   target をループ restart するのではなく、まだ生きている supervisor を見つけて止めます。
5. 衝突している owner を止めたあとでのみ、target service を start または restart し、
   identity check を再実行します:

```sh
curl -sS -X POST \
  "$PAPERCLIP_API_URL/api/execution-workspaces/$EXECUTION_WORKSPACE_ID/runtime-services/restart" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" \
  -H "X-Paperclip-Run-Id: $PAPERCLIP_RUN_ID" \
  -H "Content-Type: application/json" \
  --data-binary '{"workspaceCommandId":"service:paperclip-dev"}'
```

6. restart が維持されるのは、衝突していた owner が止まり続けている場合だけです。
   port を取り返されたら、owner / supervisor を止めて再診断し、target restart を
   ループしないでください。
7. 2つの workspace が本当に同時に動く必要があるなら、pinned port は共有できません。
   不要なほうを止め（その workspace の managed stop を使うか、再起動し続ける run を終了し）、
   issue comment で今どの workspace が port を持っているかを伝え、pinned-port collision は
   product issue としてエスカレーションします。restart をループしてはいけません。

成功と宣言する前に、疑わしい sibling workspace も取得し、それらの
`runtimeServices[].port` を比較してください。workspace をまたいで target port を
claim する `running` row が複数あるのは、必ず解決すべき conflict であり、
回避すべき state ではありません。

## database の完全な reseed

アプリが setup incomplete を示す、ログインはできるが data がない、clone された app に
期待した companies / issues / agents がない、またはユーザーが通常の
isolated-workspace database を明示的に求めているときは、full reseed を使います。

```sh
npx paperclipai worktree reseed --from-instance default --seed-mode full --yes
```

reseed 後は、managed runtime path で restart します。reseed によって、
id が local process registry と一致しなくなった runtime-service row がコピーされることが
あるため、start 後に runtime adoption と reconciliation を確認しなければなりません。

served app に auth と populated product data の両方が揃うまで reseed 完了とは
見なしません。1回だけ挿入された user/account row は診断の手がかりであって、
最終状態ではありません。

## 基本的な検証 probe

runtime API が返したサービス URL を `SERVICE_URL` に設定します。

```sh
curl -sS "$SERVICE_URL/api/health" | jq
curl -sS -I "$SERVICE_URL/" | head
```

期待する health:

- `status: "ok"`
- `bootstrapStatus: "ready"`
- `bootstrapInviteActive: false`

次の状態は失敗として扱います:

- `bootstrap_pending`: instance に first-admin setup gate が表示されます
- `database_unreachable`: web process は listen しているものの、database が停止しています
- `/api/health` が unhealthy なのに root が `200`: stale process adoption の不具合、または
  稼働中の Node process の背後で embedded database が停止しています

process がすでに listen しているときは port owner を確認します:

```sh
lsof -nP -iTCP:"$SERVICE_PORT" -sTCP:LISTEN || true
```

managed stop が失敗したあとに stale な一致する Paperclip dev-runner process を
特定して除去するためだけに使います。無関係な process は kill しないでください。

## main control plane の runtime state を確認する

main API から execution workspace を読み取り、runtime service record を確認します。

```sh
curl -sS \
  "$PAPERCLIP_API_URL/api/execution-workspaces/$EXECUTION_WORKSPACE_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" | jq
```

対象サービスには次が表示されるはずです:

- 一致する workspace command id。通常は `service:paperclip-dev`
- `status: "running"`
- 存在する場合は healthy な health field
- 今 probe しているのと同じ URL
- running port owner に対応する local provider reference

main app が running と言っているのに `/api/health` が不良なら、managed runtime を通して
stale process を止めて置き換えます。

## served workspace の runtime state を確認する

clone された Paperclip app も、その service を把握していなければなりません。
agent auth が使えるなら、served app 経由で同じ execution workspace を問い合わせます:

```sh
curl -sS \
  "$SERVICE_URL/api/execution-workspaces/$EXECUTION_WORKSPACE_ID" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" | jq
```

served app も、同じ URL のサービスが `running` / `healthy` であることを
示しているべきです。reseed 後に main control plane と served app が食い違うなら、
clone された database に local process registry と一致しない runtime-service id が
含まれている可能性があります。通常の start / adoption path を再度使い、両方を確認してください。
この領域でコードが変わったなら、対象を絞った回帰テストを追加します。

## auth と populated data を確認する

auth と data の確認には、raw DB query ではなく product API と browser / QA review を使います。

最低限の API 確認:

```sh
curl -sS "$SERVICE_URL/api/health" | jq '.status, .bootstrapStatus'
curl -sS "$SERVICE_URL/api/companies" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" | jq
curl -sS "$SERVICE_URL/api/agents/me" \
  -H "Authorization: Bearer $PAPERCLIP_API_KEY" | jq
```

続いて、primary instance にあるはずの known project、issue key、company、
execution workspace など、少なくとも1つの期待される cloned product record を API で確認します。
random な table count ではなく、現在の issue に関連する record を選んでください。

browser / QA での確認:

- service URL を開く
- first-admin setup gate が消えていることを確認する
- 通常の開発用 credential でサインインする
- board にデータが入った companies / projects / issues / runs が読み込まれることを確認する
- workspace service が稼働 URL とともに表示されることを確認する

この環境で browser を起動できないなら、QA に画面表示とログインの確認を依頼し、
自分で実行できる API check はすべて済ませてください。browser verification を
委任したことと、その理由を報告します。

## よくある失敗と修正

### Service URL が別 workspace を返す（port の占有）

症状: workspace URL は動く Paperclip app を返すが、それは兄弟 worktree の app で、
branch や data が違う、あるいは UI が別 workspace のページへ飛ばし続ける。

原因候補: pinned service port を別 workspace の process が握っていて、start 時の adoption が
その process をこの workspace の runtime row に結びつけた。health check は兄弟 app が
本当に healthy なので通ってしまう。

修正: 「port の競合と workspace の識別」の owner-first 手順に従います。
兄弟サービスまたは監督している run を特定して停止し、port が空いたままであることを確認して、
対象サービスを start または restart します。新しい port owner の `/proc/<pid>/cwd` が
対象 worktree 内を指すことを再確認してください。

確認: 識別確認が通り、URL 経由で提供される workspace 固有 record（branch、
issue key、workspace id のいずれか）が対象 workspace と一致します。

### ad hoc start のあとで記録された port がずれる

症状: runtime row と実際の listener が port について食い違っている、または誰かが
手で app を起動したあとに、以前 pinned だった port が変わってしまった。

原因候補: unmanaged な `pnpm dev`（たとえば `--bind lan` 付き）が、managed な
pinned port ではなく自分自身の port を使っている。

修正: unmanaged process を停止し、そのあと managed start します。記録された URL port が
bind された port と一致し、識別確認が通ることを確認します。

### setup gate が表示される

症状: ページに、まだ admin が instance を claim していないと表示される。

修正: primary instance から worktree の完全な reseed を実行し、そのあと managed service を
再起動します。first admin を claim するだけでも gate は消えることがありますが、
ユーザーが通常の isolated workspace database を求めているなら、full reseed が
正しい修正です。

確認: `/api/health` に `bootstrapStatus: ready` があり、ログインが通り、
データが存在します。

### ログインが失敗する

症状: bootstrap は ready だが、ユーザーの通常の開発用 credential が通らない。

原因候補: isolated DB には roles や bootstrap state はあるが、primary instance の
auth users / accounts がない。

修正: 完全な reseed です。Better Auth の user/account row だけを手でコピーして
最終修正にしないでください。

確認: browser / QA でユーザーログインを行い、agent key 付きで `/api/agents/me`
を確認します。

### ログインは通るが data がない

症状: user はサインインできるが、companies / issues / projects / runs が空か、
明らかに不完全である。

原因候補: full cloned database ではなく、partial auth repair だけが行われた。

修正: 完全な reseed を行い、managed restart し、代表的な cloned record を確認します。

### port は listen しているが health が database unreachable になる

症状: `curl -I /` は応答するのに、`/api/health` が `database_unreachable`
を報告する。

原因候補: embedded Postgres が停止したあとも stale な Node / web process が生き残っている。

修正: まず managed stop します。process が残るなら、対象 port に対応する
Paperclip dev-runner process group を特定し、その group だけを終了します。
そのあと managed start します。

確認: 安定するまで待ったあとで `/api/health` が ok になり、runtime record も healthy です。

### main control plane が running service を見失う

症状: service URL は動くのに、main app は service が作られていないか、
止まっていると言う。

原因候補: detached workaround process、stale provider ref、または service adoption が
health ではなく root URL を信じている。

修正: unmanaged process を停止し、managed runtime 経由で restart します。
コード修復が必要なら、adoption が `/api/health` を確認し、unhealthy な adopted process を
置き換え、現在の provider ref を記録するようにします。

確認: main runtime row と `/api/health` が一致します。

### full reseed 後に served app が見失う

症状: main app では `paperclip-dev` が running なのに、clone された app は
local registry と一致しない id の runtime-service row をコピーしている。

原因候補: 通常の DB clone が、primary instance から persisted runtime row を
isolated environment にコピーしたが、local process metadata は違っている。

修正: managed start / adoption を再度使います。code 修復が必要なら、adoption は
コピーされた row id だけでなく service identity と port で reconcile すべきです。

確認: main app と served app の両方が同じ service を `running` / `healthy`
として示します。

### cloned app の runtime start が permission または run-FK error を返す

症状: served app から start すると `403` が返るか、activity logging が失敗する前に
mutation が部分的に適用される。

原因候補: clone された issue / run / agent state が現在の heartbeat と一致しない、
または reseed 後の cloned database に run id がない。

修正: main control plane の managed runtime path と full reseed を優先します。
cloned app state 自体を修復する必要があるなら、まず通常の Paperclip issue / run
transition を使ってください。raw DB edit で隠さず、通常 path を妨げている
正確な guard や不足 row を報告します。

## 過去の修復で何度も出た落とし穴

これらはどれも以前の修復で問題になったものです。深掘りの前に確認してください。

- Worktree instance の log は `~/.paperclip-worktrees/instances/<slug>/logs` にあります。
  ログインや起動失敗を推測する前に読んでください。
- LAN や tailscale IP 経由で service を probe すると `403` が返るのに、
  `127.0.0.1`（または `paperclip-dev` hostname）では動くことがあります。
  auth が壊れたと結論づける前に loopback を先に probe してください。
- blank な company path（たとえば `/FOR/...`）に着地した browser QA は、
  reseed が失敗したと結論づける前に、clone された data に存在する company key
  （たとえば `/PAP/...`）へ切り替えるべきです。
- runtime や issue mutation で `409` が出るのは、たいてい別の live run が ownership lock を
  持っているという意味です。強行せず、issue を通して待つか調整してください。
- full reseed のあと、clone された DB に local process と一致しない runtime-service row が
  含まれることがあります。コピーされた row を信じるのではなく、managed start と identity check を
  再実行してください。

## コード変更が必要になる場合

通常の運用修復でプロダクトのバグが明らかになったときだけコードを変更します。
この失敗分類の例:

- local port owner の検出が誤った `lsof` 引数を使っていた
- service adoption が `/api/health` ではなく root の `200` を信頼していた
- port-owner adoption が owner の実際の `/proc/<pid>/cwd` を検証せず、要求された cwd を
  記録していたため、兄弟 workspace の process を採用してしまった
- pinned service port が兄弟 workspace 間で衝突していた
- unhealthy な adopted service を停止または置き換えしていなかった
- reseed 済みの runtime-service id が local process registry と照合されていなかった

影響を受けた service test file に対象を絞ったテストを追加します。workspace runtime repair の
場合、限定的な検証は通常次の通りです:

```sh
pnpm exec vitest run server/src/__tests__/workspace-runtime.test.ts
git diff --check
```

実質的なコード変更を commit し、その commit を issue comment にリンクします。
tracked code が変わっていないなら、そのことを明示します。

## 最終 issue コメントのテンプレート

あいまいな「動作します」ではなく、具体的な証拠を使ってください。

```md
workspace service を修復し、検証しました。

原因:
- <なぜ壊れていたか>

修正:
- <通常の reseed/start/repair 手順>
- <コード変更がある場合は commit>

確認済み:
- main control plane で <service> が <url> で running / healthy になっている
- served workspace app でも同じ service が running / healthy になっている
- port owner の識別: `/proc/<pid>/cwd` が対象 worktree 内を指している
- <url>/api/health が `bootstrapStatus: ready` 付きで ok になっている
- root page が 200 を返し、setup gate が表示されない
- <agent browser / QA / user> により、credential を投稿せず dev login を検証済み
- <具体的な API record> により clone 済みデータを検証済み
- 対象を絞ったテスト: <commands>

残件:
- <なし、または blocked の場合は担当者と必要な対応>
```

すべての成功条件が満たされたときだけ issue を `done` にします。
満たされていなければ、解除担当者と必要な対応を明記して `blocked` にします。

---
name: release-changelog-discord-message
description: >
  release changelog から stable Paperclip release の Discord 告知文を書きます。
  release issue に、dotta の声でそのまま貼れる Discord 投稿や、
  更新済みの `discord_announcement` document が必要なときに使います。
---

# リリース Discord 告知 Skill

**stable** Paperclip release の Discord 告知文を書きます。

これは `.agents/skills/release-changelog/SKILL.md` の companion です。そちらの skill が changelog を書きます。beta soak 中は `release-notes/v{beta-version}` branch 上の `releases/beta/v{beta-version}.md` にあり、stable が出た後は canonicalization PR により `releases/vYYYY.MDD.P.md` に rename されます（詳細はその skill の Channel Process section を参照）。この skill は、その file を dotta の声で1つの Discord ブロックにまとめ、`discord_announcement` document として release issue に投稿します。

## dotta のコメント

> This is for discord — try to follow my format. If I have a section where I
> think about the future, pull from recent issues we're working on etc.

Discord 告知は changelog ではありません。changelog は網羅的ですが、告知には意見があり、dotta の語り口に合わせ、出荷済みの主な highlights と、現在の Paperclip の作業から引いてきた本物の "what's next" と "what's on my mind" を中心にまとめます。創作してはいけません。

## 使う場面

- `release-changelog` が changelog を作成した後（soak 中の `release-notes/v{beta-version}` 上の beta-keyed file、または stable 出荷後の canonicalized `releases/vYYYY.MDD.P.md`）。
- release routine が割り当てた release issue で Discord 告知を求められたとき、または新しい日付 / version に合わせて `discord_announcement` document を更新したいとき。
- 単独では実行しないでください。version、日付、contributor list、highlight の集合は必ず対応する changelog file と一致していなければなりません。changelog を更新したら、これも更新します。

## 出力

Discord にそのまま貼れる、1つの fenced Markdown code block を出力します。release issue の issue document key `discord_announcement` として添付し、その issue の comment にもそのまま貼り、人間がコピーできるようにします。Cases が有効な場合は、後述の social child case も upsert します。

```bash
PUT /api/issues/{releaseIssueId}/documents/discord_announcement
{
  "title": "Discord announcement",
  "format": "markdown",
  "body": "<the announcement>",
  "baseRevisionId": "<latest if updating>"
}
```

document がすでに存在する場合は、先に取得して現在の `baseRevisionId` を渡します。黙って上書きしてはいけません。最後に書かれてから version が変わっていたら、issue comment で何が変わったかを知らせます。

## フォーマット（このテンプレートに従う）

Discord の emoji shortcodes（`:paperclip:`、`:lock:`、`:brain:` など）を使います。Unicode emoji ではありません。Discord は shortcode を描画しますが、changelog file は prose を使います。

```
:paperclip: :paperclip: :paperclip: CLIPPERS!!! v{VERSION} がリリースされました :paperclip: :paperclip: :paperclip:

公式 Twitter: https://x.com/papercliping - これをフォローし、ほかのアカウントを見つけたら報告してください

## Highlights

:emoji: **機能名** - dotta の語り口による1文の説明。
:emoji: **機能名** - …
:emoji: **機能名** - …

…そして、{残りの内容を表す表現} など、たくさんの改善があります。[完全なリリースノート](<github link>) を読んでください。

## WHATS NEXT (:motorway: Roadmap)

* **テーマ A** - 将来を見据えた短い説明
* **テーマ B** - …
* **テーマ C** - …

## What's on my mind

* **トピック** - dotta が気にしていること / キューに入っていること / 未解決の質問
* **トピック** - …

## PRESS                              （任意 - 実際の press がある場合のみ）

* **媒体 / 人物** - 起きたこと（[リンク](<x.com link>)）

## WHAT I NEED FROM YOU               （任意 - 実際にお願いしたいことがある場合のみ）

TWITTER をフォローしてください: https://x.com/papercliping - 公式アカウントはこれだけです
Paperclip をビジネスで使っているなら教えてください - お会いしたいです

## Community

このリリースに貢献してくれたすべての人に感謝します。

```
@username1, @username2, @username3
```

## まとめ

PAPERCLIP は、人間が 100 倍の仕事をこなすための AI オーケストレーターです。

誰もが、12人、100人、あるいは1000人の agent からなるチームを管理するようになり、Paperclip はそのすべてを管理する既定のツールになります。

クリップする時間です :paperclip: :paperclip: :paperclip:

完全なリリースノート

https://github.com/paperclipai/paperclip/blob/master/releases/v{VERSION}.md

||@everyone||
```

テンプレートの注意点:

- 開始と終了の `:paperclip: :paperclip: :paperclip:` bookend はブランドの一部です。
  そのまま残してください。
- 投稿のどこかで install channel を明記します。stable なら `npx paperclipai@latest`、
  早期アクセスなら `@beta` / `@nightly` / `@canary`、Docker の `:latest` は
  **stable release のときだけ**動かします。
- FULL RELEASE NOTES のリンクは `master` 上の `releases/v{VERSION}.md` を指します。
  その file は stable 後の canonicalization PR が merge されてからしか存在しません。
  告知の前に merge してください。
- section 名は UPPERCASE でも Title Case でも構いません。dotta は両方使っています。
  1つの投稿の中ではスタイルを統一してください。
- 最後に `||@everyone||`（Discord の spoiler で包んだ形）を置き、poster が spoiler を外したときに
  ちょうど1回だけ ping されるようにします。

## 文体のヒント

これは dotta が最近の告知で実際に使った書き方から抜き出したものです。
この語り口を真似てください。「professional」なトーンは作らないでください。

- **一人称で会話調。** "I want to meet companies using Paperclip"、
  "what's on my mind"、"if that's you let me know" のように書きます。
  "Paperclip is excited to announce" ではありません。
- **興奮やお願いは ALL CAPS。** 特に opener、section header、
  "WHAT I NEED FROM YOU" section、closing tagline では強めに書きます。
  ただし feature description を全部大文字にはしません。
- **highlight bullet 1 つにつき emoji shortcode 1 つ。** 機能を連想させるものを選びます。
  例: secrets は `:lock:`、planning は `:brain:`、search は `:mag:`、
  cloud / sandbox は `:cloud:`、plugins は `:jigsaw:`、history / restore は
  `:rewind:`、threads は `:thread:`。
- **highlight bullet は1文。** 意見をはっきりさせ、ユーザー視点で書きます。
  "added support for…" ではなく、"the cloud-secrets prereq is real now" のように書きます。
- **highlights の後の tail line** は残りを1文にまとめ、full release notes への
  link を付けます（"… and a long tail of {flavor}. Read the [full release notes](url)."）。
- **"WHATS NEXT" は将来のテーマ。** 実際の sprint list ではありません。
  3〜5 bullet が適切です。active goal、進行中の project、チームが取り組んでいる
  recent issue から引きます。創作しないでください。
- **Highlights は changelog skill の delta rule に従う。** 前回の告知で既に紹介した
  feature は、今回変わった点だけを載せ、継続作業を二度 headline にしません。
- **"What's on my mind"** は dotta の個人的 / 戦略的な考えです。
  docs のギャップ、哲学的な位置づけ（"we're the human control plane for ai labor"）、
  招待文（"if you've ever wanted to write about how you use Paperclip, hit me up"）などです。
  recent issue / comment から実際の tension を拾い、創作しません。
- **Press section** は任意です。release window に本物の press（tweet、podcast、talk、
  star milestone など）があるときだけ入れます。なければ section ごと削除します。
- **"WHAT I NEED FROM YOU"** も任意です。1つだけ具体的なお願い（twitter を follow して、
  紹介して、beta signup して、など）に使います。 real ask がなければ削除します。
- **Community** は changelog file と同じ contributor list を triple-backtick block に入れ、
  `@username, @username` のようにカンマ区切りにします。bot と changelog skill の
  canonical exclusion list にある人は除外します。
- **"In Summary" の mission line** はゆっくり変化します。dotta が別の指示を出していない限り、
  最新の文言を使います。最近の例:
  - "PAPERCLIP IS THE AI ORCHESTRATOR FOR HUMANS TO ACCOMPLISH 100x MORE WORK"
  - "PAPERCLIP WILL BE THE DEFAULT AGENT-MANAGEMENT TOOL FOR EVERY COMPANY"
  - "Paperclip will be _the_ control plane for AI agents in **every** company."
- **closing tagline** は常に `ITS TIME TO CLIP :paperclip: :paperclip: :paperclip:` です。
  そのまま残します。

## ワークフロー

1. `release-changelog` が生成した対応する changelog を読みます。soak 中は beta-keyed file、canonicalization 後は `releases/vYYYY.MDD.P.md` です。その file の version と contributor list を使用し、決して再導出しません。
2. key が `paperclip-release:vYYYY.MDD.P` の親 `release` case を解決します。存在せず Cases が有効な場合は、child case を作成する前に `.agents/skills/release-changelog/SKILL.md` の schema を使って作成します。
3. **release issue thread**（release routine を実行した、あなたに割り当てられた issue）
   を読みます。`WHATS NEXT` と `What's on my mind` の素材は、comment、linked issue、
   company 内の recent issue です。beta source commit の **後** にすでに `origin/master`
   にある commit は、次回 release の中身そのものなので、"what's next" の有力な素材です。
   作り話ではなく実際のテーマを拾ってください。
4. 後述の3つの verbatim example を読み直します。これらが canonical voice です。
5. 上記のテンプレートを使って告知文を下書きします。
6. release issue の `discord_announcement` document として PUT します（上の
   "Output" を参照）。更新する場合は最新の `baseRevisionId` を送ります。
7. `parentCaseId` に release case id を設定した `tweet_storm` child case を upsert し、その `body` document に告知本文を PUT します。
8. release issue に comment を投稿し、告知文を1つの fenced Markdown code block に入れます。そうすると dotta は document を開かずに Discord へコピーできます。

## Tweet Storm Case Schema

この child case を Discord / social announcement thread に使います。retry で同じ child case が更新されるよう、key は安定させます:

```http
POST /api/companies/:companyId/cases
{
  "caseType": "tweet_storm",
  "key": "paperclip-release:vYYYY.MDD.P:tweet-storm",
  "title": "Paperclip vYYYY.MDD.P tweet storm",
  "summary": "Social announcement thread for Paperclip vYYYY.MDD.P.",
  "status": "in_review",
  "parentCaseId": "<release-case-id>",
  "fields": {
    "schema_version": 1,
    "version": "vYYYY.MDD.P",
    "channel": "x",
    "discord_source": true,
    "post_count": 1,
    "target_audience": ["operators", "contributors", "agent-company builders"],
    "links": {
      "release_notes": "https://github.com/paperclipai/paperclip/blob/master/releases/vYYYY.MDD.P.md",
      "official_account": "https://x.com/papercliping"
    },
    "review": {
      "needs_human_copy_paste": true,
      "approved_by": null
    }
  }
}
```

次に body document を作成します:

```http
PUT /api/cases/:tweetStormCaseId/documents/body
{
  "title": "Paperclip vYYYY.MDD.P tweet storm body",
  "format": "markdown",
  "body": "<announcement body>",
  "changeSummary": "Draft social announcement"
}
```

既存の document を更新する場合は case を取得し、最新の `baseRevisionId` を渡します。

Discord には投稿しません。この skill は artifact の準備だけを行います。

## 以前の例（原文そのまま）

dotta による過去3件の Discord 告知を、voice・構成・emoji 使用の
ground-truth として **原文そのまま** で載せています。迷ったらこれに合わせます。

### Example 1 — v2026.403.0

```
CLIPPERS! v2026.403.0 がリリースされました!! :paperclip: :paperclip: :paperclip:

## ハイライト

:inbox_tray:  **Inbox の刷新** - メールクライアントのようなキーボードショートカットを備えた新しい「自分のもの」タブができました。仕事を管理するための、私の新しい既定ビューです
:thumbsup:  **フィードバックと評価** - エージェントの回答に :thumbsup: / :thumbsdown: で投票できるようになりました。トレースを私と共有することを選べば、Paperclip の改善に活用します。どちらの場合でも、組織独自の学習のためにローカルへエクスポートできます
:page_with_curl:  **ドキュメントのリビジョン** - ドキュメントの古いバージョンを復元できるようになりました
:ping_pong:  **テレメトリー** - このバージョンには匿名化されたテレメトリーがあり、Paperclip の基本的な使われ方（adapter など）をよりよく理解できます。気に入らなければ、`DO_NOT_TRACK=1` または `PAPERCLIP_TELEMETRY_DISABLED=1` の環境変数で無効にしてください
:construction_worker: **Execution Workspace（experimental）** - Paperclip は「コードレビュー」ツールではありませんが、特定のプロジェクトでは worktree が重要だと分かってきました。experimental settings で有効にしてください
:loop:  **Routine variables** - routine をカスタマイズしたいことがありますが、新しい variables 機能で簡単にできます

さらに adapter 周りの **大量の** 改善、バグ修正、QOL 改善があります

## コミュニティ

このリリースに commit を提供してくれた貢献者に心から感謝します:

```
@aronprins, @bittoby, @edimuj, @HenkDz, @kevmok, @mvanhorn, @radiusred, @remdev, @statxc, @vanductai
```

## 次に来るもの（ROADMAP）

* **複数人ユーザー** -- 要望をもらっていた機能です。draft があり、まもなく提供します
* **Sandbox execution** - cloud deployment のもう半分です。任意の provider 上の sandbox でエージェントを実行します

さらに、受信箱に届いている素晴らしい PR に取り組みます。

**私が考えていること（近いうちに）**

* MAXIMIZER MODE - 夢があり、使える token がたくさんあるときのために
* Artifact、成果物、deployment
* CEO Chat
* より強力な agent の既定値

## PRESS

Paperclip を広めるために、私もできることをやっています

* We talked to the incredible [Andrew Warner of Mixergy Fame](https://x.com/dotta/status/2039087507514507407)
* We gave a tutorial with the [inimitable Greg Isenberg](https://x.com/dotta/status/2037279902445994345)
* We met with the [Seed Club guys](https://x.com/dotta/status/2039020365926576377)
* We crossed [40k stars (46k now!)](https://x.com/dotta/status/2038638188227387613)
* ... and a couple others that will be released in a few days

## 成功事例

* [Nevo made $76k in march](https://x.com/dotta/status/2039406772859920758) after using Paperclip to automate his marketing
* [Lewis Jackson](https://x.com/WhatSayLew/status/2039810227394978158) said 34 agents were already operating his trading firm through Paperclip and called it his "holy s***" AI moment.
* [Neal Kotak](https://x.com/nkotak1/status/2039582439459209638) said Paperclip already runs most of Roominary for him and praised how strong the product is.
* [Sam Woods](https://x.com/samwoods/status/2039039305960587755) said he knows several people who moved from OpenClaw to Paperclip, often with Hermes in the stack, and that they love it.
* [Josh Galt](https://x.com/JoshGalt/status/2039386307219095557) called Paperclip the coolest agent tooling he has used and said it is finally something that just works.

## まとめ

まだ荒削りな部分があることは分かっていますが、

Paperclip は **すべての** 会社における AI agent の *control plane* になります。

そして、かなりいいペースで進められていると思います :paperclip: :paperclip: :paperclip:

完全なリリースノートはこちら

https://github.com/paperclipai/paperclip/releases/tag/v2026.403.0

||@everyone||
```

### Example 2 — v2026.416.0

```
:paperclip: :paperclip: :paperclip: CLIPPERS!!! v2026.416.0 がリリースされました :paperclip: :paperclip: :paperclip:

## ハイライト

このリリースには、速度、パフォーマンス、workflow に関する *大量の* QOL 改善があります。comment thread が速く感じられ、エージェントがより長くタスクに集中できることに気づくはずです

:thread: Issue chat thread が、単なる comment ではなく会話になりました
:police_officer: **Reviewer** や **Approver** のような execution policy が harness の first-class 機能になりました（たとえば、QA が *必ず* task をレビューするよう強制できます）
:no_smoking: Blocker dependency - first-class の「blocker 解消時に wake」が追加され、互いに依存する「task graph」を作り、Paperclip に強制させられるようになりました
:woman_feeding_baby: Parent-child task - sub-task のサポートが全体的に改善され、仕事を整理しやすくなりました

さらに、UX、細部、キーボードショートカット、バグ修正、セキュリティ修正など、無数の修正があります。本当に [完全なリリースノート](https://github.com/paperclipai/paperclip/releases/tag/v2026.416.0) を読んでください

## コミュニティ

このリリースに commit や報告を寄せてくれた皆さんの、信じられないほど素晴らしい仕事に感謝します:

```
@AllenHyang, @antonio-mello-ai, @aronprins, @chrisschwer, @cleanunicorn, @DanielSousa, @davison, @ergonaworks, @HearthCore, @HenkDz, @KhairulA, @kimnamu, @Lempkey, @marysomething99-prog, @mvanhorn, @officialasishkumar, @plind-dm, @shoaib050326, @sparkeros, @wbelt, @offset, @sagilayani, @mattdonnelly10, @peaktwilight, @YuvalElbar6
```

## 次に来るもの（:motorway: Roadmap）

* **複数人ユーザー** - テストの最終段階です。チームで使うと Paperclip はさらに便利になります
* **Memory Infrastructure** - エージェントがあなたの business のすべてを記憶します
* **Sandbox execution** - どこでもエージェントを実行できます

## 私が考えていること

* Paperclip を business で使っている会社と会いたいです - あなたの会社がそうなら教えてください
* Paperclip の tutorial、既定値、教育がもっと必要です - この分野ですでに取り組んでいる @aronprins に感謝します!
* 皆さんの PR をレビューする方法はまだ改善が必要で、毎日プロセスを改善しています
* 「Zero-human company」という言葉はやめなければなりません - 私たちは AI labor のための human control plane です
* Paperclip では近いうちに、*knowledge（wiki と file）*、*artifact*、*work product* のサポートを強化します。

## PRESS

* **AI Engineer Europe Tutorial** - AIE の tutorial を行いました。Paperclip の基本を知りたい人がいたら、[これを送ってください](https://x.com/dotta/status/2044575580264316931)
* **AI Club Chicago** - JB が [シカゴの AI Tinkerers](https://x.com/developwithJB/status/2044281068778316268) で Paperclip について講演しました!

## まとめ

PAPERCLIP はすべての会社における既定の agent-management tool になります

あなたやあなたの会社が Paperclip を使うために私にできることがあれば、声をかけてください。それまでは、新しいリリースを楽しんでください

クリップする時間です :paperclip: :paperclip: :paperclip:

完全なリリースノート

https://github.com/paperclipai/paperclip/releases/tag/v2026.416.0

||@everyone||
```

### Example 3 — v2026.427.0

```
:paperclip: :paperclip: :paperclip: CLIPPERS!!! v2026.427.0 がリリースされました :paperclip: :paperclip: :paperclip:

これは公式 Twitter です。フォローしてください: https://x.com/papercliping

## ハイライト

:man_feeding_baby: **複数ユーザー** - instance に複数のユーザーを招待できるようになりました
:factory_worker:  **さらに粘り強く** - robosut の liveness continuation と lifecycle recovery により、instance はあなたに助けを求める前により長く自力で動き続けます
:white_check_mark:  **SUBISSUE CHECKLIST** - subissue の順序付けが改善され、長期的な計画を立てられるようになりました
:thread: **Rich Thread UX** - エージェントが質問や承認依頼をしたり、task を提案したりできるようになり、task thread 内で承認や修正ができます
:cloud:  **BETA: Sandbox Provider** - Cloud sandboxing は beta です。このリリースで API が提供され、今後さらに provider を追加します
…そして、*大量の* その他の改善とバグ修正があります。

## コミュニティ

このリリースに貢献してくれたすべての人に感謝します!

```
@akhater, @aronprins, @GodsBoy, @LeonSGP43, @neerazz, @NoronhaH, @rbarinov, @rvanduiven, @SgtPooki, @superbiche
```

## 次に来るもの（:motorway: Roadmap）

* **より長期の計画と実行** - Paperclip は、より長い task を完了するまで実行できるようになります
* **Secrets Service v2** - Paperclip cloud にとって重要な prereq です
* **Artifact、memory、knowledge**
* **Conference Room**、別名 CEO/Agent Chat

## 私が考えていること

* **Documentation と Blog post** - docs の対応が遅れてしまっていますが、ここでは aron がよく頑張ってくれています - 維持管理を助けるために Clips を準備します
* **Paperclip Cloud** - 私たちにとって重要な unlock になりますが、shared team の体験もさらに発展させる必要があります - *作業はどこで行うべきか*、*出力はどこに保存するのか*、*ユーザーにどう見せるのか*。これらはすべて、開発が必要な Paperclip の中核 service に関する問いです
* **Paperclip Bench** - SWE-Bench に倣い、Paperclip の内部 benchmark を始めました - 変更によってシステムが改善し、regression が起きていないことを測定できなければなりません
* **Paperclip Connections Store** - Github、Slack、Google Docs、そして日々使う何百もの service への接続は、簡単で、安全で、agent と team ごとに設定可能であるべきです

## Press

[Wisemen と Paperclip について話しました](https://x.com/dotta/status/2045146539534827998)

## あなたにお願いしたいこと

この Twitter アカウントをフォローしてください: https://x.com/papercliping - 公式アカウントはこれだけです。ほかのアカウントを見つけたら報告してください

## まとめ

PAPERCLIP は、人間が 100 倍の仕事をこなすための AI オーケストレーターです

誰もが、12人、100人、あるいは1000人の agent からなるチームを管理するようになり、Paperclip はそのすべてを管理する既定のツールになります。

クリップする時間です :paperclip: :paperclip: :paperclip:

完全なリリースノート

https://github.com/paperclipai/paperclip/blob/master/releases/v2026.427.0.md

||@everyone||
```

## レビュー・チェックリスト

引き渡す前に確認します:

1. Version と date が対応する `releases/vYYYY.MDD.P.md` と完全に一致している。
2. Contributor list が changelog と一致している（除外対象は bot と changelog skill の canonical excluded-folks list で同じ）。
3. Highlights が changelog の Highlights のサブセットになっている。出荷済みの同じ機能であり、創作や pre-alpha の作業ではない。
4. `WHATS NEXT` と `What's on my mind` が実際の最近の issue / active goal から引用されている。創作したテーマではない。
5. Section style（UPPERCASE と Title Case）が投稿内で一貫している。
6. Closing tagline が `ITS TIME TO CLIP :paperclip: :paperclip: :paperclip:` で、`||@everyone||` が最終行になっている。
7. release issue の document `discord_announcement` が更新され、告知文も fenced code block 内の comment として投稿されている。

この skill は Discord へ投稿しません。announcement artifact の準備だけを行います。

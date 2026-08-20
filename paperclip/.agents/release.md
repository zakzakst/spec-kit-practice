---
name: release
description: >
  engineering verification、npm、GitHub、smoke testing、告知フォローアップを
  含む Paperclip の完全なリリース作業を調整します。単に versioning を
  話すのではなく、実際にリリースしたいときに使います。
---

# リリース調整 Skill

npm publish だけではなく、Paperclip maintainer の完全なリリースワークフローを
実行します。

この skill が調整するもの:

- `release-changelog` による stable changelog の下書き
- `master` からの canary 検証と publish 状態
- `scripts/docker-onboard-smoke.sh` による Docker smoke testing
- 選んだ source ref からの stable promotion
- GitHub Release の作成
- website / announcement の follow-up task
- release-content Cases の dogfood: top-level の `release` case と子の
  `blog_post` / `tweet_storm` case を、すべて release issue / run に紐づける

## トリガー

次のような依頼があったときに使います:

- "do a release"
- "ship the release"
- "promote this canary to stable"
- "cut the stable release"

## 前提条件

進める前に、次をすべて確認します:

1. `.agents/skills/release-changelog/SKILL.md` が存在し、使用できること。
2. repo の working tree が untracked files を含めて clean であること。
3. 最後の stable tag 以降に、少なくとも1つの canary か candidate commit があること。
4. candidate SHA が verification gate を通過済み、またはこれから通過する状態であること。
5. manifests が変わっているなら、CI 管理の `pnpm-lock.yaml` 更新がすでに `master` に merge されていること。
6. npm publish 権限が GitHub trusted publishing、または emergency / manual 用のローカル npm auth で使えること。
7. Paperclip 経由で実行しているなら、status update と follow-up task 作成のための issue context があること。

前提条件が1つでも満たせない場合は止まり、blocker を報告します。

## 入力

最初に次の入力を集めます:

- 対象が canary check か stable promotion か
- stable 用の candidate `source_ref`
- stable 実行が dry-run か live か
- website / announcement の follow-up に必要な release issue / company context

## Step 0 - リリースモデル

Paperclip は現在、commit 駆動のリリースモデルを使っています:

1. `master` への push ごとに canary が自動 publish されます。
2. canary は `YYYY.MDD.P-canary.N` を使います。
3. stable release は `YYYY.MDD.P` を使います。
4. 真ん中の slot は `MDD` で、`M` は UTC の月、`DD` はゼロ埋めした UTC の日です。
5. 同じ UTC 日に複数の stable が出ると、stable patch slot が増えます。
6. stable release は、選んだ検証済み commit または canary source commit から手動で promote します。
7. `releases/vYYYY.MDD.P.md`、git tag `vYYYY.MDD.P`、GitHub Release が作られるのは stable release だけです。

重要な帰結:

- release branches を既定の path にしない
- major / minor / patch の bump から version を導かない
- canary changelog file を作らない
- canary の GitHub Release を作らない

## Step 1 - 候補を選ぶ

canary 検証では:

- `master` 上の最新の成功した canary run を確認する
- canary version と source SHA を記録する

stable promotion では:

1. 検証済みの source ref を選ぶ
2. それが promote したい正確な SHA であることを確認する
3. `./scripts/release.sh stable --date YYYY-MM-DD --print-version` で対象 stable version を解決する

便利なコマンド:

```bash
git tag --list 'v*' --sort=-version:refname | head -1
git log --oneline --no-merges
npm view paperclipai@canary version
```

## Step 2 - stable changelog を下書きする

stable changelog のファイルは次に置きます:

- `releases/vYYYY.MDD.P.md`

`release-changelog` を呼び出し、stable notes だけを生成または更新します。

ルール:

- publish 前に人間へ下書きをレビューしてもらう
- 既存ファイルがあれば手動編集を保持する
- ファイル名は stable 専用のままにする
- canary changelog file は作らない

## Step 3 - 候補 SHA を検証する

Run the standard gate:

```bash
pnpm -r typecheck
pnpm test:run
pnpm build
```

GitHub Release workflow が publish を担当するなら、この gate は再実行できます。
ただし、ローカルで確認した場合はその状態も報告してください。

release logic に触る PR では、repo は CI で canary release の dry-run も走らせます。
これはリリース固有の guard であり、標準 gate の代わりではありません。

## Step 4 - canary を検証する

通常の canary path は `master` から自動で次を通ります:

- `.github/workflows/release.yml`

確認事項:

1. verification passed
2. npm canary publish succeeded
3. git tag `canary/vYYYY.MDD.P-canary.N` exists

便利な確認コマンド:

```bash
npm view paperclipai@canary version
git tag --list 'canary/v*' --sort=-version:refname | head -5
```

## Step 5 - canary を smoke test する

Run:

```bash
PAPERCLIPAI_VERSION=canary ./scripts/docker-onboard-smoke.sh
```

分離して試せる便利な variant:

```bash
HOST_PORT=3232 DATA_DIR=./data/release-smoke-canary PAPERCLIPAI_VERSION=canary ./scripts/docker-onboard-smoke.sh
```

確認事項:

1. install succeeds
2. onboarding completes without crashes
3. the server boots
4. the UI loads
5. basic company creation and dashboard load work

smoke testing が失敗したら:

- stable release を止める
- `master` 上で問題を修正する
- 次の自動 canary を待つ
- smoke testing を再実行する

## Step 6 - stable をプレビューまたは publish する

通常の stable path は次の `workflow_dispatch` を手動で実行することです:

- `.github/workflows/release.yml`

入力:

- `source_ref`
- `stable_date`
- `dry_run`

live stable の前に:

1. resolve the target stable version with `./scripts/release.sh stable --date YYYY-MM-DD --print-version`
2. ensure `releases/vYYYY.MDD.P.md` exists on the source ref
3. run the stable workflow in dry-run mode first when practical
4. then run the real stable publish

stable workflow は次を行います:

- re-verifies the exact source ref
- computes the next stable patch slot for the chosen UTC date
- publishes `YYYY.MDD.P` under dist-tag `latest`
- creates git tag `vYYYY.MDD.P`
- creates or updates the GitHub Release from `releases/vYYYY.MDD.P.md`

ローカルの緊急 / 手動コマンド:

```bash
./scripts/release.sh stable --dry-run
./scripts/release.sh stable
git push public-gh refs/tags/vYYYY.MDD.P
./scripts/create-github-release.sh YYYY.MDD.P
```

## Step 7 - 他の面を仕上げる

次の follow-up work を作成または確認します:

- website changelog publishing
- launch post / social announcement
- release summary in Paperclip issue context

これらは canary ではなく stable release を参照する必要があります。

## Step 8 - release-content Cases を発行する

Cases が有効な場合、すべての stable release-content run は決定的な case tree を
生成しなければなりません。これは release dogfood path の一部であり、任意の
成果物ではありません。API が `403 Cases are disabled` を返したら止めて、
operator が `experimental.enableCases` を有効にする必要があると報告します。

現在の release issue の `PAPERCLIP_COMPANY_ID`、`PAPERCLIP_API_URL`、
`PAPERCLIP_API_KEY`、`PAPERCLIP_RUN_ID` を使います。case activity feed が
run を issue に紐づけられるよう、すべての write に `X-Paperclip-Run-Id` を付けます。

まず親となる `release` case を作成または upsert します:

```http
POST /api/companies/:companyId/cases
{
  "caseType": "release",
  "key": "paperclip-release:vYYYY.MDD.P",
  "title": "Paperclip vYYYY.MDD.P release",
  "summary": "Stable release content package for Paperclip vYYYY.MDD.P.",
  "status": "in_progress",
  "fields": {
    "schema_version": 1,
    "version": "vYYYY.MDD.P",
    "release_date": "YYYY-MM-DD",
    "source_ref": "git-sha-or-ref",
    "stable": true,
    "channels": ["changelog", "blog_post", "tweet_storm"],
    "artifacts": {
      "changelog_path": "releases/vYYYY.MDD.P.md",
      "github_release_url": null
    },
    "verification": {
      "typecheck": "unknown",
      "tests": "unknown",
      "build": "unknown",
      "smoke": "unknown"
    },
    "notes": null
  }
}
```

`fields` schema は意図的に、文字列・数値・真偽値・配列・オブジェクト・null
のすべての汎用 JSON value type を使っています。case fields は全体置換なので、
upsert のたびに完全な fields object を送ってください。

upsert の直後に、親の body document を書き込みます:

```http
PUT /api/cases/:releaseCaseId/documents/body
{
  "title": "Paperclip vYYYY.MDD.P release body",
  "format": "markdown",
  "body": "# Paperclip vYYYY.MDD.P\n\nRelease summary and links...",
  "changeSummary": "Initial release case body"
}
```

続けて、`parentCaseId` を release case id に設定した child case を
作成または upsert します:

- `blog_post`, key `paperclip-release:vYYYY.MDD.P:blog-post`, status
  `in_progress`, body document key `body`
- `tweet_storm`, key `paperclip-release:vYYYY.MDD.P:tweet-storm`, status
  `in_progress`, body document key `body`

key は必ず deterministic にし、release-content flow を再実行しても同じ 3 つの
case が upsert されて複製されないようにします。child body document を書き終えたら、
生成された case identifier と link を release issue に載せ、親の acceptance issue が
ある場合はそこにも載せます。

## 失敗時の扱い

canary に問題がある場合:

- 別の canary を publish し、stable は ship しない

stable の npm publish は成功したのに、tag push や GitHub Release 作成が失敗した場合:

- 同じ release 結果の文脈から、git / GitHub の問題をすぐに修正する
- 同じ version を再 publish しない

stable publish 後に `latest` が壊れている場合:

```bash
./scripts/rollback-latest.sh <last-good-version>
```

そのあと、新しい stable release で forward fix します。

## 出力

skill が完了したら、次を返します:

- relevant なら candidate SHA と検証済み canary version
- promote した場合は stable version
- verification status
- npm status
- smoke-test status
- git tag / GitHub Release status
- website / announcement follow-up status
- release-content case tree の link: 親 `release` case と `blog_post` / `tweet_storm` child
- まだ一部しか完了していないものがあれば rollback recommendation

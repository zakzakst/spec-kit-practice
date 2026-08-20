---
name: release-changelog
description: >
  最後の stable tag 以降の commit、changeset、merge 済み PR の文脈を読み、
  `releases/vYYYY.MDD.P.md` に Paperclip の stable リリース用 changelog を生成します。
---

# リリース Changelog Skill

**stable** Paperclip リリース向けのユーザー表示用 changelog を生成します。

## バージョニングモデル

Paperclip は **calendar versioning (calver)** を使います:

- Stable リリース: `YYYY.MDD.P`（例: `2026.318.0`）
- Canary リリース: `YYYY.MDD.P-canary.N`（例: `2026.318.1-canary.0`）
- Git タグ: stable は `vYYYY.MDD.P`、canary は `canary/vYYYY.MDD.P-canary.N`

major / minor / patch の bump はありません。stable version は、予定している
release date（UTC）に、その日の次の stable patch slot を足して決まります。

出力:

- `releases/vYYYY.MDD.P.md`
- Cases が有効な場合は、`(caseType, key)` で upsert され、changelog 本文を含む
  `body` ドキュメントリビジョンを持つ `release` Case

重要なルール:

- `2026.318.1-canary.0` のような canary リリースがあっても、changelog ファイルは
  `releases/v2026.318.1.md` のままです
- version を semver の bump type から導かない
- canary 用の changelog ファイルを作らない

## チャネル処理 - source commit とファイルの場所

stable は **soaked beta** を promote するため、changelog は `master` の先端ではなく、
beta の source commit を説明します:

- release の **source** は、最新の `beta/v<beta-version>` tag が指している commit
  です（以下の `{beta-src}`）。次のように解決します:

  ```bash
  git fetch origin --tags
  npm view paperclipai dist-tags   # beta の dist-tag から version を取得
  git rev-parse 'beta/v{beta-version}^{commit}'
  ```

- `{beta-src}` の後に `master` に入った commit は、**次** の release で出荷されます。
  それらを changelog に含めてはいけません。これは "what's next" section の素材であり、
  changelog ではありません。
- soak 中は、file は `release-notes/v{beta-version}` branch 上の
  `releases/beta/v{beta-version}.md` にあります（`master` への PR）。
  release workflow は beta publish 時に生成済み skeleton を載せた branch を push します。
  それをそのまま編集して skeleton を書き換えてください。branch が存在しない場合
  （automation 前に beta を切った場合）は、`origin/master` から作成して skeleton を
  生成します:

  ```bash
  ./scripts/draft-stable-notes.sh {beta-version}
  ```

- PR は stable が dispatch される **前** に `master` へ merge されていなければなりません。
  stable preflight は file を `master` から読むため、これがないと失敗します。
- この path で自分で `releases/vYYYY.MDD.P.md` を作らないでください。stable 出荷後に、
  workflow が beta-keyed file をその名前へ rename する canonicalization PR を開きます。
- **Fix path exception**（`candidate/release-*` branch からの patch release）では、
  notes は candidate branch 上に直接 `releases/vYYYY.MDD.P.md` として置かれ、
  cherry-pick した fix と一緒に commit されます。

## Step 0 - 冪等性の確認

何かを生成する前に、changelog がすでに存在するか確認します:

```bash
  ls releases/beta/v{beta-version}.md 2>/dev/null   # soak window の配置先
ls releases/vYYYY.MDD.P.md 2>/dev/null            # canonicalized / fix path
git ls-remote origin 'refs/heads/release-notes/v{beta-version}'
```

生成された skeleton だけを持つ `release-notes/v{beta-version}` branch は
通常の開始状態であり、競合ではありません。そのまま上書きしてください。

存在する場合:

1. 最初に読みます
2. reviewer に提示します
3. 保持、再生成、特定 section の更新のどれを行うか確認します
4. 決して黙って上書きしません

## Step 1 - stable の対象範囲を決める

最後の stable tag と beta source commit を確認します:

```bash
git tag --list 'v*' --sort=-version:refname | head -1
beta_src="$(git rev-parse 'beta/v{beta-version}^{commit}')"
git log v{last}..${beta_src} --oneline --no-merges
```

changelog の範囲は常に `v{last}..{beta-src}` です。`..HEAD` でも
`..origin/master` でもありません。

stable version は次のいずれかから決まります:

- 明示的なメンテナーからの依頼
- `./scripts/release.sh stable --date YYYY-MM-DD --print-version`
- `doc/RELEASING.md` で合意済みのリリース計画

changelog version を canary tag や prerelease suffix から導いてはいけません。
API の意図から major / minor / patch bump を導くことも禁止です。calver では
date と同日 stable slot を使います。

## Step 2 - 生の入力データを集める

リリースデータを次から集めます:

1. 最後の stable tag 以降の Git commit
2. `.changeset/*.md` ファイル
3. 利用可能であれば `gh` 経由の merge 済み PR

便利なコマンド:

```bash
git log v{last}..{beta-src} --oneline --no-merges
git log v{last}..{beta-src} --format="%H %s" --no-merges
ls .changeset/*.md | grep -v README.md
gh pr list --state merged --search "merged:>={last-tag-date}" --json number,title,body,labels
```

## Step 3 — Breaking Change を検出する

次を確認します:

- 破壊的なマイグレーション
- 削除または変更された API フィールド／エンドポイント
- 名前が変更または削除された設定キー
- `BREAKING:` または `BREAKING CHANGE:` という commit シグナル

Key commands:

```bash
git diff --name-only v{last}..{beta-src} -- packages/db/src/migrations/
git diff v{last}..{beta-src} -- packages/db/src/schema/
git diff v{last}..{beta-src} -- server/src/routes/ server/src/api/
git log v{last}..{beta-src} --format="%s" | rg -n 'BREAKING CHANGE|BREAKING:|^[a-z]+!:' || true
```

breaking change を検出した場合は目立つ形で示します。Breaking Changes セクションに、アップグレード手順とともに必ず記載します。

## Step 4 — ユーザー向けに分類する

stable changelog では次の section を使います:

- `Breaking Changes`
- `Highlights`
- `Improvements`
- `Fixes`
- 必要に応じて `Upgrade Guide`

ユーザーに実質的な影響がない限り、内部のみのリファクタリング、CI の変更、ドキュメントのみの変更は除外します。

ガイドライン:

- 関連する commit を 1 つのユーザー向け項目にまとめる
- ユーザーの視点で書く
- Highlights は短く具体的にする
- breaking change の場合はアップグレード時の対応を明記する
- **繰り返しではなく差分を記述する**: 作成前に前回の stable の notes を読む
  (`releases/v<last-stable>.md`) を作成前に読みます。前回のリリースですでに
  その機能がすでに導入されている場合、このリリースでは変更点のみを扱います。
  たとえば、デフォルト値の変更、堅牢化、完成などです。前回のリリースを基準に
  （「前回のリリースで X を導入し、今回のリリースでデフォルトにする」など）
  表現し、初登場の機能であるかのように再説明してはいけません。前回のリリースで
  主役だったテーマの継続作業は再び Highlights にせず、Improvements に降格します。

### PR と貢献者のインライン表記

箇条書きの項目が merge 済み pull request に明確に対応する場合は、次の形式で項目末尾に
帰属情報を追加します:

```
- **Feature name** — Description. ([#123](https://github.com/paperclipai/paperclip/pull/123), @contributor1, @contributor2)
```

ルール:

- その箇条書きを特定の merge 済み PR に確実に対応付けられる場合のみ、PR リンクを追加します。
  PR の対応付けには merge commit のメッセージ（`Merge pull request #N from user/branch`）を使います。
- PR を作成した貢献者を記載します。実名やメールアドレスではなく GitHub username を使います。
- 1 つの項目に複数の PR が関係する場合は、すべて記載します: `([#10](url), [#12](url), @user1, @user2)`。
- PR 番号や貢献者を確実に特定できない場合は、推測せず帰属情報の括弧を省略します。
- 外部 PR のないコアメンテナーの commit は、括弧内の帰属情報を省略できます。

## Step 5 — ファイルを書く

**ファイルパス**は「チャネル処理」セクションに記載した beta 用のもの
（`releases/beta/v{beta-version}.md`）ですが、**内容**には予定された stable version を
タイトルとして記載します。次のコマンドで解決します:
`./scripts/release.sh stable --date {planned-promotion-date} --print-version`
（promotion は通常、beta 公開日の 3 日後です）。promotion 日がずれた場合、dispatch 時に
version が再解決されます。beta 用のファイル名なので影響はありませんが、その際はタイトルを更新します。

changelog の冒頭行は `# Paperclip {version}` 形式の H1
（波括弧なし。例: `# Paperclip v2026.618.0`）にします。必ず `Paperclip ` prefix と
version の `v` を含めます。

Template:

```markdown
# Paperclip vYYYY.MDD.P

> Released: YYYY-MM-DD

## Breaking Changes

## Highlights

## Improvements

## Fixes

## Upgrade Guide

## Contributors

このリリースに貢献してくださった皆さんに感謝します！

@username1, @username2, @username3
```

Omit empty sections except `Highlights`, `Improvements`, and `Fixes`, which should usually exist.

`Contributors` セクションは常に含めます。リリース範囲内の commit を作成した全員を、
実名やメールアドレスではなく **GitHub username** で @メンションします。GitHub username の
調べ方:

1. merge commit メッセージから username を抽出します: `git log v{last}..{beta-src} --oneline --merges`。ブランチの prefix（例: `from username/branch`）から GitHub username がわかります。
2. `user@users.noreply.github.com` のような noreply メールでは、`@` より前の部分が username です。
3. username が曖昧な貢献者については、`gh api users/{guess}` または PR ページで確認します。

**貢献者のメールアドレスは決して公開しません。** `@username` のみを使います。

bot アカウント（例: `lockfile-bot`、`dependabot`）は一覧から除外します。
特定の人物も一覧から除外します。Contributors セクションではコミュニティの貢献者のみを
クレジットします。正規の除外リスト（ここで管理し、Discord skill もこれを参照します）:
`cryppadotta`, `forgottendev`, `devinfoley`, `sockmonster`, `scotttong`,
`nguyenm7`, `nickyleach`, `tonio-alucema`

貢献者は GitHub username のアルファベット順（大文字小文字を区別しない）で並べます。

除外後に貢献者が残らない場合は、このセクションを省略し、言及もしません。

## Step 5b — Release Case を upsert する

`releases/vYYYY.MDD.P.md` を書き込んだ後、実行環境に Paperclip API のコンテキストがある場合は、
トップレベルの release case を作成または更新します。API の契約には `skills/paperclip/references/cases.md`
を使用します。API が `403 Cases are disabled` を返した場合は、Cases を有効にする必要があることを
報告し、changelog ファイルの処理だけを続けます。

Request:

```http
POST /api/companies/:companyId/cases
{
  "caseType": "release",
  "key": "paperclip-release:vYYYY.MDD.P",
  "title": "Paperclip vYYYY.MDD.P release",
  "summary": "Stable Paperclip release notes for vYYYY.MDD.P.",
  "status": "in_progress",
  "fields": {
    "schema_version": 1,
    "version": "vYYYY.MDD.P",
    "release_date": "YYYY-MM-DD",
    "release_patch": 0,
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

この fields schema は、汎用 field value の全種類を意図的に使用します:
string、number、boolean、array、object、null のすべてを意図的に使用します。fields は deep merge
ではなく置き換えられるため、実行間でキーを変えず、upsert のたびにオブジェクト全体を送信します。

続けて changelog を case body document に書き込みます:

```http
PUT /api/cases/:releaseCaseId/documents/body
{
  "title": "Paperclip vYYYY.MDD.P changelog",
  "format": "markdown",
  "body": "<contents of releases/vYYYY.MDD.P.md>",
  "changeSummary": "Initial stable changelog"
}
```

既存の body document を更新する場合は、先に case を取得して最新の
`baseRevisionId` を渡します。`409 stale_base_revision` が返ったら、再取得して
意図的に merge し、1回だけ retry します。

## Step 6 — 公開前レビュー

引き渡す前に次を確認します:

1. H1 見出しが `# Paperclip {version}`（例: `# Paperclip v2026.618.0`）になっており、stable version のみが入っていること
2. title とファイル名に `-canary` が含まれていないこと
3. breaking change がある場合は upgrade path が用意されていること
4. `release` case が存在するか、あるいは Cases を使えなかった理由を説明できること
5. 下書きについて人間の sign-off を得ること

この skill は何も publish しません。stable changelog artifact を準備するだけです。

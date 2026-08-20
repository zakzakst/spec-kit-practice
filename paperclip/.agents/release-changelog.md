---
name: release-changelog
description: >
  最後の stable tag 以降の commit、changeset、merge 済み PR の文脈を読み、
  `releases/vYYYY.MDD.P.md` に stable Paperclip release の changelog を生成します。
---

# リリース Changelog Skill

**stable** Paperclip release 向けのユーザー表示用 changelog を生成します。

## バージョニングモデル

Paperclip は **calendar versioning (calver)** を使います:

- Stable releases: `YYYY.MDD.P` (e.g. `2026.318.0`)
- Canary releases: `YYYY.MDD.P-canary.N` (e.g. `2026.318.1-canary.0`)
- Git tags: `vYYYY.MDD.P` for stable, `canary/vYYYY.MDD.P-canary.N` for canary

major / minor / patch の bump はありません。stable version は、予定している
release date（UTC）に、その日の次の stable patch slot を足して決まります。

出力:

- `releases/vYYYY.MDD.P.md`
- a `release` Case, upserted by `(caseType, key)` when Cases are enabled, with a
  `body` document revision containing the changelog body

重要なルール:

- `2026.318.1-canary.0` のような canary release があっても、changelog file は
  `releases/v2026.318.1.md` のままです
- version を semver bump type から導かない
- canary changelog file を作らない

## Channel Process - source commit とファイルの場所

stable は **soaked beta** を promote するため、changelog は `master` の先端ではなく、
beta の source commit を説明します:

- release の **source** は、最新の `beta/v<beta-version>` tag が指している commit
  です（以下の `{beta-src}`）。次のように解決します:

  ```bash
  git fetch origin --tags
  npm view paperclipai dist-tags   # the beta dist-tag names the version
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

## Step 0 - idempotency の確認

何かを生成する前に、changelog がすでに存在するか確認します:

```bash
ls releases/beta/v{beta-version}.md 2>/dev/null   # soak-window home
ls releases/vYYYY.MDD.P.md 2>/dev/null            # canonicalized / fix path
git ls-remote origin 'refs/heads/release-notes/v{beta-version}'
```

生成された skeleton だけを持つ `release-notes/v{beta-version}` branch は
通常の開始状態であり、conflict ではありません。そのまま上書きしてください。

存在する場合:

1. 最初に読みます
2. reviewer に提示します
3. 保持、再生成、特定 section の更新のどれを行うか確認します
4. 決して黙って上書きしません

## Step 1 - stable の範囲を決める

最後の stable tag と beta source commit を確認します:

```bash
git tag --list 'v*' --sort=-version:refname | head -1
beta_src="$(git rev-parse 'beta/v{beta-version}^{commit}')"
git log v{last}..${beta_src} --oneline --no-merges
```

changelog の範囲は常に `v{last}..{beta-src}` です。`..HEAD` でも
`..origin/master` でもありません。

stable version は次のいずれかから決まります:

- an explicit maintainer request
- `./scripts/release.sh stable --date YYYY-MM-DD --print-version`
- the release plan already agreed in `doc/RELEASING.md`

changelog version を canary tag や prerelease suffix から導いてはいけません。
API の意図から major / minor / patch bump を導くことも禁止です。calver では
date と同日 stable slot を使います。

## Step 2 - 生の入力を集める

release data を次から集めます:

1. git commits since the last stable tag
2. `.changeset/*.md` files
3. merged PRs via `gh` when available

Useful commands:

```bash
git log v{last}..{beta-src} --oneline --no-merges
git log v{last}..{beta-src} --format="%H %s" --no-merges
ls .changeset/*.md | grep -v README.md
gh pr list --state merged --search "merged:>={last-tag-date}" --json number,title,body,labels
```

## Step 3 — Breaking Change を検出する

Look for:

- destructive migrations
- removed or changed API fields/endpoints
- renamed or removed config keys
- `BREAKING:` or `BREAKING CHANGE:` commit signals

Key commands:

```bash
git diff --name-only v{last}..{beta-src} -- packages/db/src/migrations/
git diff v{last}..{beta-src} -- packages/db/src/schema/
git diff v{last}..{beta-src} -- server/src/routes/ server/src/api/
git log v{last}..{beta-src} --format="%s" | rg -n 'BREAKING CHANGE|BREAKING:|^[a-z]+!:' || true
```

breaking change を検出した場合は目立つ形で示します。Breaking Changes section に upgrade path とともに必ず記載します。

## Step 4 — ユーザー向けに分類する

stable changelog では次の section を使います:

- `Breaking Changes`
- `Highlights`
- `Improvements`
- `Fixes`
- `Upgrade Guide` when needed

Exclude purely internal refactors, CI changes, and docs-only work unless they materially affect users.

Guidelines:

- group related commits into one user-facing entry
- write from the user perspective
- keep highlights short and concrete
- spell out upgrade actions for breaking changes
- **describe deltas, not repeats**: read the previous stable's notes
  (`releases/v<last-stable>.md`) before writing. When they already
  introduced a feature, this release's entry covers only what changed —
  a default flip, a hardening, a completion — phrased against the prior
  release ("last release introduced X; this release makes it the
  default"), never re-describing the feature as if it debuted. A theme
  that headlined the previous release does not headline again for
  follow-through work; demote it to Improvements.

### Inline PR and contributor attribution

When a bullet item clearly maps to a merged pull request, add inline attribution at the
end of the entry in this format:

```
- **Feature name** — Description. ([#123](https://github.com/paperclipai/paperclip/pull/123), @contributor1, @contributor2)
```

Rules:

- Only add a PR link when you can confidently trace the bullet to a specific merged PR.
  Use merge commit messages (`Merge pull request #N from user/branch`) to map PRs.
- List the contributor(s) who authored the PR. Use GitHub usernames, not real names or emails.
- If multiple PRs contributed to a single bullet, list them all: `([#10](url), [#12](url), @user1, @user2)`.
- If you cannot determine the PR number or contributor with confidence, omit the attribution
  parenthetical — do not guess.
- Core maintainer commits that don't have an external PR can omit the parenthetical.

## Step 5 — ファイルを書く

The **file path** is the beta-keyed one from the Channel Process section
(`releases/beta/v{beta-version}.md`), but the **content** is titled with
the planned stable version. Resolve it with
`./scripts/release.sh stable --date {planned-promotion-date} --print-version`
(promotion is normally the beta publish date plus the 3-day soak). If the
promotion date slips, the version re-resolves at dispatch — the beta-keyed
filename makes that harmless; refresh the title when it happens.

The opening line of the changelog must be an H1 of the format `# Paperclip {version}`
(no braces), e.g. `# Paperclip v2026.618.0`. Always include the `Paperclip ` prefix and
the `v` on the version.

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

Thank you to everyone who contributed to this release!

@username1, @username2, @username3
```

Omit empty sections except `Highlights`, `Improvements`, and `Fixes`, which should usually exist.

The `Contributors` section should always be included. List every person who authored
commits in the release range, @-mentioning them by their **GitHub username** (not their
real name or email). To find GitHub usernames:

1. Extract usernames from merge commit messages: `git log v{last}..{beta-src} --oneline --merges` — the branch prefix (e.g. `from username/branch`) gives the GitHub username.
2. For noreply emails like `user@users.noreply.github.com`, the username is the part before `@`.
3. For contributors whose username is ambiguous, check `gh api users/{guess}` or the PR page.

**Never expose contributor email addresses.** Use `@username` only.

Exclude bot accounts (e.g. `lockfile-bot`, `dependabot`) from the list.
Exclude specific folks from the list — the Contributors section credits
community contributors only. The canonical exclusion list (keep it here;
the Discord skill defers to it):
`cryppadotta`, `forgottendev`, `devinfoley`, `sockmonster`, `scotttong`,
`nguyenm7`, `nickyleach`, `tonio-alucema`

List contributors in alphabetical order by GitHub username (case-insensitive).

If there are no contributors left after exclusions, then just skip this section and don't mention it.

## Step 5b — Release Case を upsert する

After writing `releases/vYYYY.MDD.P.md`, emit or refresh the top-level release
case when the run has Paperclip API context. Use `skills/paperclip/references/cases.md`
as the API contract. If the API returns `403 Cases are disabled`, report that
Cases を有効にする必要があることを報告し、changelog file の処理だけを続けます。

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
string, number, boolean, array, object, and null. Keep the keys stable across
runs and send the full object on every upsert because fields are replaced, not
deep-merged.

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
2. title と filename に `-canary` が含まれていないこと
3. breaking change がある場合は upgrade path が用意されていること
4. `release` case が存在するか、あるいは Cases を使えなかった理由を説明できること
5. 下書きを human に sign-off してもらうこと

この skill は何も publish しません。stable changelog artifact を準備するだけです。

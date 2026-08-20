---
name: create-paperclip-bundled-skill
description: >
  アイデア、tweet、task を Paperclip skills catalog（packages/skills-catalog）の
  skill に変えます。skill を FIND したり MAKE したりして、bundled / optional の
  catalog skill として公開したいときに使います。prior art を調べ、参照または自作し、
  例を追加し、manifest を再生成し、PR を開きます。
---

# Paperclip Bundled Skill を作る

source material — tweet、task description、blog post、"X をする skill を作って" という
依頼 — を受け取り、Paperclip skills catalog（`packages/skills-catalog/`）の skill として、
レビュー済み PR で着地させます。catalog はすべての Paperclip company が眺めて
install する棚なので、求める水準は、正しい metadata、役立つ instruction、
動作する example、きれいな validation run です。

中心ルールは **FIND before MAKE** です。良い skill がすでにあるなら
（catalog 内、repo 内、GitHub 上など）、ゼロから重複を書かず、参照または
adapt してください。

## 使う場面

- A human sends a tweet/link/idea and asks for it to become a Paperclip skill.
- A task asks to bundle an existing repo skill into the catalog.
- A task asks to add an external published skill to the catalog.

## 使わない場面

- The skill is company-private (belongs in that company's library via the
  Skills UI/API, not the shipped catalog).
- You only need a repo-internal agent skill for working on Paperclip itself —
  that goes in `.agents/skills/` or `skills/`, with no catalog machinery.

## Step 0 - source material を確保する

何かを書く前に、その skill が何を教えるべきかを正確に理解します。

**Tweet / X link.** `xc` CLI（X API client）を使います。Paperclip engineering
agent environment には preinstalled かつ pre-authenticated で入っています。
自分で install したり credential を発行したりする tool ではありません。
頼る前に使えるか確認してください:

```sh
command -v xc && xc whoami    # on PATH and authenticated? if not, use the fallback below
```

```sh
xc get <post-url-or-id> --json        # the post itself (conversation_id, author)
xc search 'conversation_id:<id>' --archive --json   # rest of the thread (>7 days old needs --archive)
xc user <username>                    # author context
xc search '<topic keywords>' -n 30    # related discussion
```

`xc` が PATH にない、認証されていない、または account に read access がない
（上の確認が何らかの理由で失敗する）場合は、X/Twitter access を持つ teammate
（例: Content Strategist agent）へ child issue で取得を依頼します。
URL を渡し、post + thread + 関連コンテンツの全文を求めてください。

**Other sources.** Fetch linked articles/READMEs directly. Record the source
URL — it goes in the skill body or PR description as attribution.

Distill: what is the repeatable procedure? What inputs does it take? What does
"done" look like? If the source is just an aspiration ("agents should write
better commit messages"), you are authoring the procedure yourself — say so in
the PR.

## Step 1 - FIND: 既存の skill を探す

次の順番で探し、明確な候補が見つかったら止めます。

1. **Already in the catalog?** Avoid duplicates (duplicate slugs fail the
   build):
   ```sh
   grep -i '<topic>' packages/skills-catalog/generated/catalog.json
   ls packages/skills-catalog/catalog/{bundled,optional}/*/
   ```
2. **Already in this repo?** Check `.agents/skills/`, `skills/`, and issue
   history (`gh search issues` / Paperclip board) for prior work on the topic.
3. **Published on GitHub?** Skills are conventionally a directory with a
   `SKILL.md`:
   ```sh
   gh search code --filename SKILL.md "<topic>" --limit 20
   gh search repos "<topic> skill" --limit 20
   ```
   Also check known collections (e.g. `anthropics/skills`) and do a web search
   for `<topic> agent skill SKILL.md`.

Judge candidates by: does the SKILL.md actually contain the procedure (not a
stub)? Is it maintained? What does it bundle (scripts raise the trust level)?
Is the license compatible with redistribution? Then pick a path:

- **Good external skill exists** → add it as an **external reference**
  (Step 2A). It stays attributed to and pinned at the upstream repo.
- **Partial match** → author a local skill (Step 2B) that adapts the idea;
  credit the source with a link in the SKILL.md body.
- **Nothing usable** → author a new local skill (Step 2B).

## Step 2 - kind、category、slug を選ぶ

- **kind**: default to `optional`. Use `bundled` only when the skill should
  ship to every Paperclip company by default — that needs explicit human/board
  direction, not your judgment call.
- **category**: reuse an existing directory when one fits (`browser`,
  `content`, `docs`, `finance`, `paperclip-operations`, `product`, `quality`,
  `research`, `software-development`). New categories are allowed but must be
  lowercase kebab-case slugs.
- **slug**: lowercase kebab-case (`^[a-z0-9]+(-[a-z0-9]+)*$`), unique across
  the whole catalog (both kinds).

The skill lives at
`packages/skills-catalog/catalog/<kind>/<category>/<slug>/` and its canonical
key is `paperclipai/<kind>/<category>/<slug>`.

## Step 2A - external reference path（`catalog-ref.json`）

The directory contains **only** `catalog-ref.json` (a directory with both
`catalog-ref.json` and `SKILL.md` fails the build). The manifest builder
fetches the pinned files from GitHub at build time and inventories them.

```sh
# Pin the exact commit for the chosen ref (tag or branch)
gh api repos/<owner>/<repo>/commits/<ref> --jq .sha
```

```json
{
  "source": {
    "type": "github",
    "hostname": "github.com",
    "owner": "<owner>",
    "repo": "<repo>",
    "ref": "<tag-or-branch>",
    "commit": "<40-char sha from above>",
    "path": "<dir inside the repo containing SKILL.md, or ''>"
  },
  "files": ["SKILL.md", "references/**", "scripts/run.py"],
  "defaultInstall": false,
  "recommendedForRoles": ["researcher"],
  "requires": ["python3"],
  "tags": ["topic", "keywords"]
}
```

Rules the builder enforces:

- `files` entries are exact relative paths or `dir/**` globs; `SKILL.md` must
  be included and must have frontmatter with `name` and `description`.
- If the upstream frontmatter declares `key`/`slug`, they must match the
  catalog placement — otherwise pick a matching slug or use the local path.
- `commit` must be a full 40-hex SHA; every listed file must be ≤ 1 MiB.
- `recommendedForRoles`, `requires`, `tags` live in the JSON (there is no
  local SKILL.md to carry them).

See `catalog/optional/research/last30days/catalog-ref.json` for the live
example, and `examples/external-reference.md` next to this skill.

## Step 2B - local catalog skill を作る

Layout:

```
catalog/<kind>/<category>/<slug>/
├── SKILL.md          # required entrypoint
├── examples/         # 1–2 worked examples (Step 3)
├── references/       # optional deep-dive docs
├── scripts/          # optional — raises trust level, avoid unless needed
└── assets/           # optional templates/images
```

`SKILL.md` frontmatter (all validated by the builder):

```markdown
---
name: <slug>
description: >
  40–300 chars. Routing logic, not marketing: what it does, when to use it,
  when not to.
key: paperclipai/<kind>/<category>/<slug>
recommendedForRoles:
  - engineer            # non-empty; used for staffing suggestions
tags:
  - topic               # non-empty; used for browse/search
---
```

Optional frontmatter: `defaultInstall: true` (only for skills every new
company should get), `requires: [node, python3, ...]` for runtime deps.

本文: `docs/guides/agent-developer/writing-a-skill.md` に従います。
「When to use」/「When not to use」 section、抽象論ではなく具体的な command、
必要に応じて `references/` の補足 detail を使います。skill が tweet や外部 source
由来なら、本文に link を入れて attribution してください。

Trust level is derived from files, not declared: any `scripts/` file makes the
skill `scripts_executables` (install becomes audit-gated and you must extend
the `scriptBearing` expectation in `src/shipped-catalog.test.ts`); `assets/`
or non-markdown files make it `assets`; markdown-only skills stay
`markdown_only`. Prefer markdown-only.

## Step 3 - 1〜2 個の worked example を書く

Create `examples/` inside the skill directory with one or two markdown files,
each a complete input → application → output walkthrough (realistic input, the
skill's steps applied, the finished artifact). These ship with the skill so
installers can judge it before running it, and they keep the trust level at
`markdown_only` because they are `.md` files.

Name them by scenario, e.g. `examples/rewrite-release-note.md`.

## Step 4 - manifest を再生成し、test を更新する

Never hand-edit `generated/catalog.json`; it is deterministic build output.

```sh
pnpm --filter @paperclipai/skills-catalog build:manifest   # regenerates generated/catalog.json
pnpm --filter @paperclipai/skills-catalog validate         # must report no errors
```

(External references need network access to GitHub during these steps.)

Then update `packages/skills-catalog/src/shipped-catalog.test.ts`:

- add the new key to `EXPECTED_BUNDLED_KEYS` or `EXPECTED_OPTIONAL_KEYS`
  (alphabetical order);
- if the skill bears scripts, add it to the `scriptBearing` expectation.

```sh
pnpm --filter @paperclipai/skills-catalog test
```

The test suite also enforces the ≤300-char frontmatter description budget
across the repo and the ≥40-char description / non-empty roles+tags rules for
every catalog skill.

## Step 5 - PR を開く

Follow the `prepare-paperclip-pr` skill (`.agents/skills/prepare-paperclip-pr/`)
against `paperclipai/paperclip` master. The diff should contain exactly:

1. the new skill directory (SKILL.md + examples/ + supporting files, **or**
   catalog-ref.json),
2. the regenerated `generated/catalog.json`,
3. the `shipped-catalog.test.ts` expectation update.

In the PR body: link the source material (tweet URL, upstream repo), state
whether this is a new skill / adaptation / external reference, and note the
trust level. Reference PR #10410 (simplified-english) as the shape of a
minimal optional-skill PR.

## 落とし穴

- `generated/catalog.json` staleness is a validation error — always rerun
  `build:manifest` after any file change inside the skill directory (the
  inventory carries per-file sha256 hashes).
- Duplicate `slug` across bundled *and* optional fails the build, not just
  duplicate keys.
- Symlinks inside a skill directory must resolve within it; directory
  symlinks are rejected — copy files in.
- The `bundled` kind and `defaultInstall` are independent axes; don't set
  `defaultInstall: true` casually on optional skills.
- For external references the builder fetches from GitHub on every manifest
  build; a moved/deleted upstream breaks the build, which is why `commit` is
  pinned — prefer upstream tags for `ref`.

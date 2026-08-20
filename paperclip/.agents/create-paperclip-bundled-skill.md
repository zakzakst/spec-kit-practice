---
name: create-paperclip-bundled-skill
description: >
  アイデア、tweet、task を Paperclip skills catalog（packages/skills-catalog）の
  skill に変えます。skill を FIND したり MAKE したりして、bundled / optional の
  catalog skill として公開したいときに使います。prior art を調べ、参照または自作し、
  例を追加し、manifest を再生成し、PR を開きます。
---

# Paperclip Bundled Skill を作る

source material - tweet、task description、blog post、"X をする skill を作って" という依頼 - を受け取り、Paperclip skills catalog（`packages/skills-catalog/`）の skill として、レビュー済み PR で着地させます。catalog はすべての Paperclip company が眺めて install する棚なので、求める水準は、正しい metadata、役立つ instruction、動作する example、きれいな validation run です。

中心ルールは **FIND before MAKE** です。良い skill がすでにあるなら（catalog 内、repo 内、GitHub 上など）、ゼロから重複を書かず、参照または adapt してください。

## 使う場面

- A human sends a tweet/link/idea and asks for it to become a Paperclip skill.
- A task asks to bundle an existing repo skill into the catalog.
- A task asks to add an external published skill to the catalog.

## 使わない場面

- The skill is company-private（その company の library に入るべきで、出荷される catalog ではない）。
- Paperclip 自体を作業するための repo 内部向け agent skill が必要なだけのとき - その場合は `.agents/skills/` または `skills/` に置き、catalog machinery は使いません。

## Step 0 - source material を確保する

何かを書く前に、その skill が何を教えるべきかを正確に理解します。

**Tweet / X link.** `xc` CLI（X API client）を使います。Paperclip engineering agent environment には preinstalled かつ pre-authenticated で入っています。自分で install したり credential を発行したりする tool ではありません。頼る前に使えるか確認してください:

```sh
command -v xc && xc whoami    # on PATH and authenticated? if not, use the fallback below
```

```sh
xc get <post-url-or-id> --json        # the post itself (conversation_id, author)
xc search 'conversation_id:<id>' --archive --json   # rest of the thread (>7 days old needs --archive)
xc user <username>                    # author context
xc search '<topic keywords>' -n 30    # related discussion
```

`xc` が PATH にない、認証されていない、または account に read access がない（上の確認が何らかの理由で失敗する）場合は、X/Twitter access を持つ teammate（例: Content Strategist agent）へ child issue で取得を依頼します。URL を渡し、post + thread + 関連コンテンツの全文を求めてください。

**Other sources.** 連結された article / README を直接 fetch します。source URL を記録してください。skill body か PR description に attribution として入れます。

要点を抽出します: 再現可能な手順は何か、入力は何か、"done" はどう見えるか。source が単なる aspiration（"agents should write better commit messages" など）なら、手順自体をこちらで書きます。その場合は PR でそう明示してください。

## Step 1 - FIND: 既存の skill を探す

次の順番で探し、明確な候補が見つかったら止めます。

1. **すでに catalog にあるか?** 重複を避けます（duplicate slug は build に失敗します）:
   ```sh
   grep -i '<topic>' packages/skills-catalog/generated/catalog.json
   ls packages/skills-catalog/catalog/{bundled,optional}/*/
   ```
2. **すでにこの repo にあるか?** `.agents/skills/`、`skills/`、issue history（`gh search issues` / Paperclip board）で同じ topic の先行作業を確認します。
3. **GitHub で公開されているか?** Skills は通常 `SKILL.md` を持つディレクトリです:
   ```sh
   gh search code --filename SKILL.md "<topic>" --limit 20
   gh search repos "<topic> skill" --limit 20
   ```
   あわせて既知の collection（例: `anthropics/skills`）を確認し、`<topic> agent skill SKILL.md` で web search も行います。

候補を評価する基準: SKILL.md が本当に手順を含んでいるか（stub ではないか）、保守されているか、何を bundle しているか（script は trust level を上げます）、license は再配布可能か。そこから次のいずれかを選びます:

- **良い external skill がある** → **external reference** として追加します（Step 2A）。
- **部分一致** → そのアイデアを local skill として書きます（Step 2B）。SKILL.md 本文で source へ link を貼って credit します。
- **使えるものがない** → 新しい local skill を書きます（Step 2B）。

## Step 2 - kind、category、slug を選ぶ

- **kind**: 既定は `optional`。`bundled` を使うのは、その skill をすべての Paperclip company に既定で ship すべきだと明示的な human / board 判断がある場合だけです。
- **category**: 既存の directory が合うなら再利用します（`browser`、`content`、`docs`、`finance`、`paperclip-operations`、`product`、`quality`、`research`、`software-development` など）。新しい category も可能ですが、小文字 kebab-case にしてください。
- **slug**: 小文字 kebab-case（`^[a-z0-9]+(-[a-z0-9]+)*$`）、catalog 全体で一意です（kind をまたいで重複不可）。

skill の置き場所は `packages/skills-catalog/catalog/<kind>/<category>/<slug>/` で、canonical key は `paperclipai/<kind>/<category>/<slug>` です。

## Step 2A - external reference path（`catalog-ref.json`）

directory には **`catalog-ref.json` だけ** を置きます（`catalog-ref.json` と `SKILL.md` の両方があると build に失敗します）。manifest builder は build 時に GitHub から pinned file を取得し、inventory を作成します。

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

builder が強制するルール:

- `files` entries は正確な relative path か `dir/**` glob であること。`SKILL.md` を含め、frontmatter に `name` と `description` が必要です。
- upstream frontmatter に `key` / `slug` がある場合は catalog の配置と一致している必要があります。一致しなければ、合う slug を選ぶか local path にします。
- `commit` は完全な 40 文字 hex SHA であること。列挙された各 file は 1 MiB 以下でなければなりません。
- `recommendedForRoles`、`requires`、`tags` は JSON 側に置きます（local の SKILL.md にはありません）。

実例は `catalog/optional/research/last30days/catalog-ref.json` を見てください。`examples/external-reference.md` も、この skill の横にあります。

## Step 2B - local catalog skill を作る

Layout:

```text
<slug>/
├── SKILL.md
├── examples/         # 1–2 worked examples (Step 3)
├── references/       # optional deep-dive docs
├── scripts/          # optional - trust level を上げるので、必要なときだけ
└── assets/           # optional templates/images
```

`SKILL.md` frontmatter（builder がすべて検証します）:

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

任意の frontmatter: `defaultInstall: true`（新しい company が最初から持つべき skill にだけ使う）、`requires: [node, python3, ...]`（runtime 依存）。

本文は `docs/guides/agent-developer/writing-a-skill.md` に従います。「When to use」/「When not to use」 section、抽象論ではなく具体的な command、必要に応じて `references/` の補足 detail を使います。skill が tweet や external source 由来なら、本文に link を入れて attribution してください。

Trust level は file から決まります。`scripts/` file が1つでもある skill は `scripts_executables` になり（install は audit-gated になり、`src/shipped-catalog.test.ts` の `scriptBearing` expectation を増やす必要があります）、`assets/` や markdown 以外の file があると `assets` になります。markdown のみの skill は `markdown_only` のままです。できるだけ markdown-only にしてください。

## Step 3 - 1〜2 個の worked example を書く

`examples/` を skill directory 内に作り、1つか2つの markdown file を置きます。それぞれは input → application → output の完全な walkthrough（現実的な入力、skill の手順、完成した artifact）にしてください。これらは skill と一緒に ship されるので、installer は実行前に評価できます。また `.md` file なので trust level も `markdown_only` のままです。

scenario ごとに名前を付けます。例: `examples/rewrite-release-note.md`。

## Step 4 - manifest を再生成し、test を更新する

`generated/catalog.json` は手で編集しないでください。決定論的な build output です。

```sh
pnpm --filter @paperclipai/skills-catalog build:manifest   # regenerates generated/catalog.json
pnpm --filter @paperclipai/skills-catalog validate         # must report no errors
```

(External references には、この step の間 GitHub への network access が必要です。)

そのあと `packages/skills-catalog/src/shipped-catalog.test.ts` を更新します:

- 新しい key を `EXPECTED_BUNDLED_KEYS` または `EXPECTED_OPTIONAL_KEYS` に追加します（alphabetical order）。
- skill に scripts がある場合は `scriptBearing` expectation に追加します。

```sh
pnpm --filter @paperclipai/skills-catalog test
```

test suite は、repo 全体に対して frontmatter description が 300 文字以下であること、さらに catalog skill ごとに description が 40 文字以上で roles + tags が空でないことも検査します。

## Step 5 - PR を開く

`prepare-paperclip-pr` skill（`.agents/skills/prepare-paperclip-pr/`）を使い、`paperclipai/paperclip` の master に対して PR を開きます。diff には次の3点だけを含めてください:

1. 新しい skill directory（SKILL.md + examples/ + supporting files、または `catalog-ref.json`）
2. 再生成した `generated/catalog.json`
3. `shipped-catalog.test.ts` の expectation 更新

PR 本文には source material（tweet URL、upstream repo）をリンクし、これは新規 skill / adaptation / external reference のどれなのかを明記し、trust level も書きます。最小の optional-skill PR の形として PR #10410（simplified-english）を参照してください。

## 落とし穴

- `generated/catalog.json` の stale は validation error です。skill directory 内の file を変更したら、必ず `build:manifest` を再実行してください（inventory には file ごとの sha256 hash が入ります）。
- bundled と optional をまたぐ duplicate `slug` は、duplicate key だけでなく build 失敗の原因になります。
- skill directory 内の symlink はその directory の中に解決される必要があります。directory symlink は拒否されるので、file をコピーしてください。
- `bundled` kind と `defaultInstall` は別の軸です。optional skill に `defaultInstall: true` を軽々しく付けないでください。
- external reference の場合、builder は manifest build のたびに GitHub から fetch します。upstream が move / delete されると build が壊れるので、`commit` を pin するのはそのためです。`ref` には upstream tag を使うのが無難です。

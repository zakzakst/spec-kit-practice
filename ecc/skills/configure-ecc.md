---
name: configure-ecc
description: Everything Claude Code の対話型インストーラ。skills と rules を user-level または project-level のディレクトリへ選択的に導入し、パスを検証し、必要なら導入後最適化まで行います。
origin: ECC
---

# Configure Everything Claude Code (ECC)

Everything Claude Code プロジェクト向けの、段階的な対話型インストールウィザードです。`AskUserQuestion` を使って skills と rules の選択導入を案内し、その後に正しさを検証し、必要なら最適化も提案します。

## 有効化するタイミング

- ユーザーが "configure ecc", "install ecc", "setup everything claude code" などと言ったとき
- このプロジェクトの skill や rule を選択的に入れたいとき
- 既存の ECC インストールを検証または修復したいとき
- プロジェクト向けに導入済み skill / rule を最適化したいとき

## 前提条件

この skill 自体が、起動前に Claude Code から見えている必要がある。起動方法は 2 通り:

1. **Plugin 経由**: `/plugin install ecc@ecc` - plugin がこの skill を自動読込する
2. **手動**: この skill だけを `~/.claude/skills/configure-ecc/SKILL.md` にコピーし、"configure ecc" と言って起動する

---

## Step 0: ECC リポジトリを clone する

導入前に、最新の ECC ソースを `/tmp` へ clone する:

```bash
rm -rf /tmp/everything-claude-code
git clone https://github.com/affaan-m/everything-claude-code.git /tmp/everything-claude-code
```

以後のコピー元として `ECC_ROOT=/tmp/everything-claude-code` を使う。

clone に失敗した場合（ネットワーク問題など）は、`AskUserQuestion` で既存の local ECC clone パスを聞く。

---

## Step 1: インストールレベルを選ぶ

`AskUserQuestion` でインストール先を聞く:

```text
Question: "Where should ECC components be installed?"
Options:
  - "User-level (~/.claude/)" - 全 Claude Code プロジェクトに適用
  - "Project-level (.claude/)" - 現在のプロジェクトだけに適用
  - "Both" - 共通物は user-level、project 固有物は project-level
```

選択を `INSTALL_LEVEL` として保持し、対象ディレクトリを設定する:

- User-level: `TARGET=~/.claude`
- Project-level: `TARGET=.claude`
- Both: `TARGET_USER=~/.claude`, `TARGET_PROJECT=.claude`

必要ならディレクトリを作る:

```bash
mkdir -p $TARGET/skills $TARGET/rules
```

---

## Step 2: Skill を選んで導入する

### 2a: 範囲を選ぶ（Core vs Niche）

既定値は **Core（新規ユーザー推奨）**。`.agents/skills/*` に加え、research-first ワークフロー用に `skills/search-first/` を含める。ここには engineering、eval、verification、security、strategic compaction、frontend design、Anthropic cross-functional skill が含まれる。

`AskUserQuestion`（単一選択）:

```text
Question: "Install core skills only, or include niche/framework packs?"
Options:
  - "Core only (recommended)" - tdd, e2e, evals, verification, research-first, security, frontend patterns, compacting, cross-functional skills
  - "Core + selected niche" - core 導入後に framework / domain-specific skill を追加
  - "Niche only" - core を入れず、特定分野の skill だけを導入
Default: Core only
```

### 2b: Skill Category を選ぶ

必要なら `AskUserQuestion` の multi select でカテゴリを選ぶ:

```text
Question: "Which skill categories do you want to install?"
Options:
  - "Framework & Language"
  - "Database"
  - "Workflow & Quality"
  - "Research & APIs"
  - "Social & Content Distribution"
  - "Media Generation"
  - "Orchestration"
  - "All skills"
```

### 2c: 個別 Skill を確認する

選ばれたカテゴリごとに全文一覧を表示し、必要なら除外を受け付ける。4 項目を超える場合は一覧をテキスト表示し、`AskUserQuestion` では「全部入れる」または手入力で個別指定できるようにする。

主なカテゴリ:

- **Framework & Language**: backend / coding / Django / Laravel / frontend / Go / Java / Python / Quarkus / Spring Boot 系
- **Database**: `clickhouse-io`, `jpa-patterns`, `postgres-patterns`
- **Workflow & Quality**: `continuous-learning`, `continuous-learning-v2`, `eval-harness`, `iterative-retrieval`, `security-review`, `strategic-compact`, `tdd-workflow`, `verification-loop`
- **Business & Content**: `article-writing`, `content-engine`, `market-research`, `investor-materials`, `investor-outreach`
- **Research & APIs**: `deep-research`, `exa-search`, 必要なら公式 `claude-api`
- **Social & Content Distribution**: `x-api`, `crosspost`
- **Media Generation**: `fal-ai-media`, `video-editing`
- **Orchestration**: `dmux-workflows`
- **Standalone**: `docs/examples/project-guidelines-template.md`

### 2d: インストール実行

選択された各 skill について、正しい source root からディレクトリ丸ごとコピーする:

```bash
# Core skills live under .agents/skills/
cp -R "$ECC_ROOT/.agents/skills/<skill-name>" "$TARGET/skills/"

# Niche skills live under skills/
cp -R "$ECC_ROOT/skills/<skill-name>" "$TARGET/skills/"
```

glob 展開したディレクトリを回すとき、末尾スラッシュ付きの source をそのまま `cp` に渡さない。宛先名を明示する:

```bash
cp -R "${src%/}" "$TARGET/skills/$(basename "${src%/}")"
```

`continuous-learning` と `continuous-learning-v2` は追加ファイル（config.json, hooks, scripts）を含むため、`SKILL.md` だけでなくディレクトリ全体をコピーする。

---

## Step 3: Rule を選んで導入する

`AskUserQuestion` の multi select を使う:

```text
Question: "Which rule sets do you want to install?"
Options:
  - "Common rules (Recommended)"
  - "TypeScript/JavaScript"
  - "Python"
  - "Go"
```

導入時のコピー:

```bash
cp -r $ECC_ROOT/rules/common $TARGET/rules/common
cp -r $ECC_ROOT/rules/typescript $TARGET/rules/typescript
cp -r $ECC_ROOT/rules/python $TARGET/rules/python
cp -r $ECC_ROOT/rules/golang $TARGET/rules/golang
```

**重要**: language-specific rule を選んだのに common rule を選んでいない場合は警告する:

> "Language-specific rules extend the common rules. Installing without common rules may result in incomplete coverage. Install common rules too?"

---

## Step 4: 導入後検証

### 4a: ファイル存在確認

導入したファイルを列挙し、対象先に存在することを確認する:

```bash
ls -la $TARGET/skills/
ls -la $TARGET/rules/
```

### 4b: パス参照チェック

導入した `.md` ファイルに含まれるパス参照を走査する:

```bash
grep -rn "~/.claude/" $TARGET/skills/ $TARGET/rules/
grep -rn "../common/" $TARGET/rules/
grep -rn "skills/" $TARGET/skills/
```

**project-level install** の場合は `~/.claude/` 参照に特に注意する:

- `~/.claude/settings.json` 参照は通常問題ない（settings は user-level）
- `~/.claude/skills/` や `~/.claude/rules/` 参照は、project-level のみ導入では壊れる可能性がある
- skill 名による相互参照がある場合は、その skill も導入済みか確認する

### 4c: Skill 間の相互参照チェック

代表的な依存:

- `django-tdd` -> `django-patterns`
- `laravel-tdd` -> `laravel-patterns`
- `quarkus-tdd` -> `quarkus-patterns`
- `springboot-tdd` -> `springboot-patterns`
- `continuous-learning-v2` -> `~/.claude/homunculus/`
- `python-testing` -> `python-patterns`
- `golang-testing` -> `golang-patterns`
- `crosspost` -> `content-engine` / `x-api`
- `deep-research` -> `exa-search`
- `fal-ai-media` -> `videodb`
- `x-api` -> `content-engine` / `crosspost`
- language-specific rules -> `common/`

### 4d: 問題を報告する

問題ごとに次を出す:

1. **File** - 問題のある参照を持つファイル
2. **Line** - 行番号
3. **Issue** - 何が問題か
4. **Suggested fix** - どう直すべきか

---

## Step 5: 導入物を最適化する（任意）

`AskUserQuestion`:

```text
Question: "Would you like to optimize the installed files for your project?"
Options:
  - "Optimize skills"
  - "Optimize rules"
  - "Optimize both"
  - "Skip"
```

### skill を最適化する場合

1. 導入済み `SKILL.md` を読む
2. 必要ならプロジェクトの技術スタックを確認する
3. 各 skill で不要セクションを提案する
4. **導入先側** の `SKILL.md` をその場で編集する（source repo は触らない）
5. Step 4 で見つかったパス問題も直す

### rule を最適化する場合

1. 導入済み rule `.md` を読む
2. 以下の好みを確認する:
   - テストカバレッジ目標（既定 80%）
   - 好みの formatter
   - git workflow 慣習
   - security 要件
3. rule ファイルを導入先側で編集する

**重要**: 変更するのは常に導入先（`$TARGET/`）だけ。source ECC repo（`$ECC_ROOT/`）は絶対に変更しない。

---

## Step 6: インストールサマリー

最後に `/tmp` の clone を削除する:

```bash
rm -rf /tmp/everything-claude-code
```

そして以下のような要約を出す:

```text
## ECC Installation Complete

### Installation Target
- Level: [user-level / project-level / both]
- Path: [target path]

### Skills Installed ([count])
- skill-1, skill-2, skill-3, ...

### Rules Installed ([count])
- common (8 files)
- typescript (5 files)
- ...

### Verification Results
- [count] issues found, [count] fixed
- [list any remaining issues]

### Optimizations Applied
- [list changes made, or "None"]
```

---

## トラブルシューティング

### "Skills not being picked up by Claude Code"

- skill ディレクトリに `SKILL.md` があるか確認する
- user-level なら `~/.claude/skills/<skill-name>/SKILL.md`
- project-level なら `.claude/skills/<skill-name>/SKILL.md`

### "Rules not working"

- rule は flat file として置く。`$TARGET/rules/coding-style.md` が正しく、`$TARGET/rules/common/coding-style.md` が最終形ではないことに注意
- rule 導入後は Claude Code を再起動する

### "Path reference errors after project-level install"

- 一部 skill は `~/.claude/` 前提を持つ。Step 4 の検証で洗い出して修正する
- `continuous-learning-v2` の `~/.claude/homunculus/` は user-level 前提なので、この参照は想定内であり必ずしもエラーではない

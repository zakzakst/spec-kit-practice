---
name: git-workflow-and-versioning
description: git のワークフロー実践を整理します。あらゆるコード変更、コミット、ブランチ、コンフリクト解消、複数の並列作業の整理、リリース切り、セマンティックバージョンの選定、タグ付け、変更履歴の作成に使います。
---

# Git ワークフローとバージョニング

## 概要

Git は安全網です。コミットは保存ポイント、ブランチはサンドボックス、履歴は文書として扱います。AI エージェントが高速にコードを生む今こそ、規律あるバージョン管理が、変更を管理可能・レビュー可能・巻き戻し可能に保ちます。

## 使う場面

常に使います。すべてのコード変更は git を通ります。

## コア原則

### Trunk-Based Development（推奨）

`main` は常にデプロイ可能な状態に保ちます。1〜3 日でマージする短命な feature branch を使います。長命な開発ブランチは隠れたコストです。ずれが大きくなり、マージコンフリクトが増え、統合が遅れます。DORA の研究でも、trunk-based development は高パフォーマンスなチームと相関があります。

```text
main ->->->->->->->->->->  (always deployable)
        ^  ^    ^          short-lived feature branches (1-3 days)
```

これは推奨デフォルトです。gitflow や長命ブランチを使うチームでも、原則（原子的なコミット、小さな変更、説明的なメッセージ）はそのブランチ戦略に適用できます。重要なのは commit の規律であり、特定の branching 戦略そのものではありません。

- **Dev branch はコストです。** 1 日長く生きるごとに、マージリスクが溜まります。
- **Release branch は許容される。** main を進めながらリリースを安定化したいときに使います。
- **Feature flags は長い branch より良い。** 未完成の作業は、何週間も branch に置くより、flag の裏に隠してデプロイするほうがよいです。

### 1. 早く、頻繁にコミットする

成功した増分ごとに 1 つの commit を作ります。大きな未コミット変更を溜めないでください。

```text
作業の流れ:
  slice を実装 -> テスト -> 検証 -> コミット -> 次の slice

こうではない:
  すべて実装 -> うまくいくと祈る -> 巨大コミット
```

コミットは保存ポイントです。次の変更が壊しても、最後に動いていた状態へすぐ戻れます。

### 2. 原子的なコミット

各コミットは 1 つの論理的なことだけを行います。

```text
# Good: 各 commit が自己完結
git log --oneline
a1b2c3d タスク作成 endpoint に validation を追加
d4e5f6g タスク作成 form component を追加
h7i8j9k form を API に接続し、loading state を追加
m1n2o3p タスク作成テストを追加（unit + integration）

# Bad: すべて混在
git log --oneline
x1y2z3a タスク機能追加、sidebar 修正、deps 更新、utils リファクタリング
```

### 3. 説明的なメッセージ

コミットメッセージは *何を* だけでなく *なぜ* を説明します。

```text
# Good: 意図が分かる
feat: registration endpoint に email validation を追加

無効な email 形式が database に入るのを防ぐ。
auth.ts の既存 validation パターンに合わせて、
route handler で Zod schema validation を使う。

# Bad: 差分を見れば分かることだけを書く
update auth.ts
```

**形式:**

```text
<type>: <短い説明>

<任意の body - 何をではなく、なぜを書く>
```

**type:**
- `feat` - 新機能
- `fix` - バグ修正
- `refactor` - バグ修正でも新機能でもないコード変更
- `test` - テストの追加・更新
- `docs` - ドキュメントのみ
- `chore` - ツール、依存関係、設定

### 4. 関心事を分ける

フォーマット変更と振る舞い変更は混ぜません。リファクタリングと機能追加も混ぜません。各変更タイプは別 commit にします。理想的には別 PR です。

```text
# Good: 関心事を分ける
git commit -m "refactor: extract validation logic to shared utility"
git commit -m "feat: add phone number validation to registration"

# Bad: 混ぜる
git commit -m "refactor validation and add phone number field"
```

**リファクタリングと機能作業は分ける。** 既存コードのリファクタリングと新しい振る舞いの追加は 2 つの変更です。別々に提出してください。小さな整理（変数名変更など）は、レビューアーの判断で feature commit に含めても構いません。

### 5. サイズを整える

commit / PR はおおむね 100 行前後を目標にします。1000 行を超える変更は分割してください。分割方法は `code-review-and-quality` を参照します。

```text
~100 行   -> レビューしやすい
~300 行   -> 1 つの論理変更なら許容
~1000 行  -> 分割する
```

## ブランチ戦略

### Feature Branches

```text
main (always deployable)
  -> feature/task-creation   1 つの機能
  -> feature/user-settings   並列作業
  -> fix/duplicate-tasks     バグ修正
```

- `main`（またはチームの既定 branch）から分岐する
- branch は短命にする（1〜3 日でマージ） - 長命 branch は隠れたコスト
- マージ後は削除する
- 未完成機能は長命 branch より feature flag を使う

### Branch Naming

```text
feature/<short-description>   -> feature/task-creation
fix/<short-description>       -> fix/duplicate-tasks
chore/<short-description>     -> chore/update-deps
refactor/<short-description>  -> refactor/auth-module
```

## Worktree の使い方

並列 AI エージェント作業では、git worktree を使って複数 branch を同時に走らせます。

```bash
# feature branch 用に worktree を作る
git worktree add ../project-feature-a feature/task-creation
git worktree add ../project-feature-b feature/user-settings

# 各 worktree はそれぞれの branch を持つ別ディレクトリ
# エージェントは干渉せず並列作業できる
ls ../
  project/              main branch
  project-feature-a/    task-creation branch
  project-feature-b/    user-settings branch

# 終わったら merge して片付ける
git worktree remove ../project-feature-a
```

**利点:**
- 複数エージェントが異なる機能で同時に作業できる
- branch 切り替えが不要（各ディレクトリが独自 branch を持つ）
- 1 つの実験が失敗しても worktree を消せばよい - 失うものはない
- 変更は明示的に merge されるまで隔離される

## Save Point パターン

```text
Agent starts work
  -> change を加える
     -> test passes? commit -> continue
     -> test fails? revert to last commit -> investigate
  -> another change
     -> test passes? commit -> continue
     -> test fails? revert to last commit -> investigate
  -> feature complete, clean history
```

このパターンでは、失うのは常に 1 増分分までです。エージェントが暴走しても、`git reset --hard HEAD` で最後の成功状態に戻れます。

## Change Summaries

何か変更したら、構造化した要約を出します。レビューがしやすくなり、スコープ規律が文書化され、意図しない変更が表に出ます。

```text
CHANGES MADE:
- src/routes/tasks.ts: POST endpoint に validation middleware を追加
- src/lib/validation.ts: Zod で TaskCreateSchema を追加

THINGS I DIDN'T TOUCH (intentionally):
- src/routes/auth.ts: 同様の validation gap があるがスコープ外
- src/middleware/error.ts: error format は改善余地がある（別タスク）

POTENTIAL CONCERNS:
- Zod schema は strict - extra fields を拒否する。これでよいか確認
- zod を dependency として追加（72KB gzipped） - 既に package.json にはある
```

このパターンは、誤った仮定を早く見つけ、レビューアーに変更の地図を渡します。`DIDN'T TOUCH` セクションは特に重要です。scope discipline を守り、勝手な改装をしていないことを示します。

## Pre-Commit Hygiene

毎回 commit 前に:

```bash
# 1. 何を commit するか確認
git diff --staged

# 2. 秘密情報がないか確認
git diff --staged | grep -i "password\|secret\|api_key\|token"

# 3. テスト実行
npm test

# 4. lint 実行
npm run lint

# 5. 型チェック
npx tsc --noEmit
```

git hook で自動化できます。

```json
// package.json (lint-staged + husky)
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

## Generated Files の扱い

- **生成ファイルはコミットする** - プロジェクトが期待している場合のみ（例: `package-lock.json`、Prisma migrations）
- **コミットしない** - build output（`dist/`、`.next/`）、環境ファイル（`.env`）、IDE 設定（共有しない `vscode/settings.json` など）
- **`.gitignore` を用意する** - `node_modules/`、`dist/`、`.env`、`.env.local`、`*.pem`

## Git を使ったデバッグ

```bash
# バグを入れた commit を探す
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# Git が中間 commit を checkout するので、そのたびにテストを走らせる

# 直近の変更を見る
git log --oneline -20
git diff HEAD~5..HEAD -- src/

# 特定行を最後に変更した人を調べる
git blame src/services/task.ts

# キーワードで commit message を検索
git log --grep="validation" --oneline
```

## Release と Versioning

commit は *あなた* が変更を追う方法で、version は *利用者* が変更を追う方法です。何か他者があなたのコードに依存する瞬間 - 別チーム、公開パッケージ、デプロイ済みクライアント - には、"main の latest" だけでは「今何を動かしていて、アップグレードして安全か」を答えられません。version number と changelog が、その答えです。

### Semantic Versioning

利用者がいるものは `MAJOR.MINOR.PATCH` を付け、番号に意味を持たせます。

```text
MAJOR  breaking change - 利用者はコードを変える必要がある
MINOR  互換性のある新機能 - 安全にアップグレードできる
PATCH  互換性のあるバグ修正 - 安全にアップグレードできる
```

番号は約束です。コードもそれに合わせてください。利用者が頼っていた振る舞いを変える "patch" は、見せかけの major change です（Hyrum's Law - `api-and-interface-design` を参照）。破壊的か迷うなら、壊すものだと仮定してください。予想外の major は、壊れた利用者よりはるかに安いです。

### リリースに tag を付け、tag を正とする

release は動く branch ではなく、不変の歴史の一点です。tag を付けて、いつでも再現できるようにします。

```bash
git tag -a v1.4.0 -m "Release 1.4.0"
git push origin v1.4.0
```

version は手で散在ファイルを書き換えるのではなく、tag から導出します。そうすれば artifact、tag、changelog が食い違いません。

### 人間向けの changelog を保つ

changelog は `git log` ではありません。利用者向けに、「何が変わり、自分に関係あるか」を答えるものです。`Added / Changed / Fixed / Deprecated / Removed / Security` にまとめ、新しいものを上にし、各 entry は内部実装ではなく利用者への影響で書きます。

```markdown
## [1.4.0] - 2025-06-12
### Added
- CSV による一括タスク import
### Fixed
- recurring task の due date の timezone drift
### Deprecated
- `GET /v1/tasks/all` - ページネーション付き `GET /v1/tasks` を使ってください（2.0 で削除）
```

entry は、変更したその時点で同じ change に書きます。release 時に commit archaeology から掘り起こすのではありません。breaking change には migration note と deprecation window を付けます（`deprecation-and-migration` を参照）。実際の release を shipping するのは `shipping-and-launch` の役目です。この節は、その前段の versioning contract です。

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「feature が終わったら commit する」 | 巨大な 1 commit はレビューもデバッグも revert もできません。slice ごとに commit してください。 |
| 「メッセージはどうでもいい」 | メッセージは文書です。未来のあなたと未来のエージェントは、何が変わってなぜ変わったかを知る必要があります。 |
| 「あとで squash する」 | squash は開発の物語を消します。最初からきれいな incremental commit にしてください。 |
| 「branch はオーバーヘッド」 | 短命 branch はほぼ無料で、作業の衝突を防ぎます。問題なのは長命 branch です - 1〜3 日で merge してください。 |
| 「この変更はあとで分ければいい」 | 大きな変更はレビューしづらく、デプロイも revert も危険です。提出前に分割してください。 |
| 「.gitignore は要らない」 | 本番秘密情報を含む `.env` が commit されるまで、そう言えますか。今すぐ用意してください。 |
| 「小さい fix だから patch でいい」 | 利用者から見えるものを確認してください。頼っていた振る舞いが変わるなら、diff の大きさに関係なく major です。 |
| 「changelog は commit log で十分」 | commit は自分向け、changelog は利用者向けです。生の commit から生成すると、重要なものが埋もれます。 |
| 「release 時に changelog を書けばいい」 | その時点では影響を記憶から再構成することになり、半分は欠けます。変更と同時に書いてください。 |

## レッドフラグ

- 大きな未コミット変更が溜まっている
- "fix"、"update"、"misc" のような commit message
- 振る舞い変更とフォーマット変更が混ざっている
- プロジェクトに `.gitignore` がない
- `node_modules/`、`.env`、build artifacts を commit している
- main から大きくずれた長命 branch
- 共有 branch への force-push
- breaking change を minor または patch bump で出している
- tag がない release、または tag と食い違う手編集 version number
- 利用者向け release に changelog がない、または commit message をただ流しただけ

## 検証

各 commit について:

- [ ] 1 つの論理的なことだけをしている
- [ ] メッセージがなぜを説明し、type 規約に従っている
- [ ] commit 前にテストが通っている
- [ ] diff に秘密情報がない
- [ ] フォーマット変更と振る舞い変更が混ざっていない
- [ ] `.gitignore` が標準除外をカバーしている

各 release（利用者がいるもの）について:

- [ ] version bump が変更に合っている: breaking -> major, additive -> minor, fix -> patch
- [ ] release に tag が付き、version は tag から導出され、手編集でずれていない
- [ ] changelog に、その version の影響でまとめられた、人間が読める entry がある

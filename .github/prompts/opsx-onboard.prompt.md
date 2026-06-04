---
description: ナレーション付きで OpenSpec の一連の流れを案内するオンボーディング
---

ユーザーが OpenSpec の一連の workflow を 1 周体験できるよう案内する。これは**教えながら実際に手を動かす**体験であり、コードベースに対して本物の作業をしながら各ステップを説明する。

---

## Preflight

開始前に OpenSpec が初期化済みか確認する:

```bash
openspec status --json 2>&1 || echo "NOT_INITIALIZED"
```

**未初期化なら**
> OpenSpec isn't set up in this project yet. Run `openspec init` first, then come back to `/opsx:onboard`.

ここで止める。

---

## Phase 1: Welcome

次を表示する:

```text
## OpenSpec へようこそ！

ここでは、コードベース内の実際のタスクを使って、アイデアから実装までの change サイクルを一通り案内します。実際に手を動かしながら、ワークフローを体験的に学べます。

**やること:**
1. コードベースの中から小さな実タスクを 1 つ選ぶ
2. 問題を軽く探索する
3. change を作る（作業の入れ物）
4. 成果物を作る: proposal -> specs -> design -> tasks
5. タスクを実装する
6. 完了した change を archive する

**所要時間:** 約 15〜20 分

まずは取り組む対象を見つけましょう。
```

---

## Phase 2: Task Selection

### Codebase Analysis

コードベースを走査し、小さく着手しやすい改善候補を探す。見る観点:

1. **TODO / FIXME コメント**: `TODO`, `FIXME`, `HACK`, `XXX`
2. **エラーハンドリング不足**: 飲み込む `catch`、危険処理に try-catch がない
3. **テストのない関数**: `src/` と test ディレクトリの対応
4. **型の問題**: TypeScript の `any`
5. **デバッグ残骸**: `console.log`, `console.debug`, `debugger`
6. **バリデーション不足**: 入力ハンドラに検証がない

直近の git 履歴も確認する:
```bash
git log --oneline -10 2>/dev/null || echo "No git history"
```

### Present Suggestions

分析結果から 3〜4 件の具体候補を提示する:

```text
## タスク候補

コードベースを確認した結果、始めやすいタスク候補をいくつか挙げます。

**1. [最有力のタスク]**
   場所: `src/path/to/file.ts:42`
   規模: 1〜2 ファイル、約 20〜30 行
   おすすめ理由: [簡潔な理由]

**2. [2つ目のタスク]**
   場所: `src/another/file.ts`
   規模: 1 ファイル、約 15 行
   おすすめ理由: [簡潔な理由]

**3. [3つ目のタスク]**
   場所: [場所]
   規模: [見積もり]
   おすすめ理由: [簡潔な理由]

**4. 別案**
   取り組みたい内容を自由に教えてください。

どのタスクに興味がありますか？（番号で選ぶか、自分の案を説明してください）
```

**見つからない場合**
> コードベースには、すぐに着手できる明確な改善点は見つかりませんでした。ずっと追加したい、または直したいと思っていた小さなものはありますか？

### Scope Guardrail

ユーザーが大きすぎるタスクを選んだ場合:

```text
それは価値のあるタスクですが、OpenSpec の最初の流れとしては少し大きすぎるかもしれません。

ワークフローを学ぶには、小さい方が向いています。実装の細部で詰まりにくく、サイクル全体を見やすくなります。

**選択肢:**
1. **もっと小さく切る** - [そのタスク] の中で、最も小さくて意味のある部分はどこでしょう？ たとえば [具体的な切り出し] だけに絞れますか？
2. **別のものを選ぶ** - ほかの候補、または別の小さなタスクにしますか？
3. **そのまま進める** - 本当にこれに取り組みたいなら、そのまま進めることもできます。ただし少し時間がかかります。

どれにしますか？
```

強制はしない。やわらかいガードレールとして扱う。

---

## Phase 3: Explore Demo

タスクが決まったら、explore mode を短く実演する:

```text
change を作る前に、まず **explore モード** を短くお見せします。これは、進む方向を決める前に問題を考え抜くためのモードです。
```

1〜2 分程度で関連コードを調べる:
- 対象ファイルを読む
- 必要なら簡単な ASCII 図を描く
- 気づいた点をまとめる

```text
## クイック探索

[ここに簡単な分析 - 見つかったこと、考慮点]

[必要なら ASCII 図を追加]

explore モード（`/opsx:explore`）は、実装前に調べたり考えたりするためのものです。問題を整理したいときはいつでも使えます。

それでは、作業を入れておくための change を作成しましょう。
```

**PAUSE** - ユーザーの応答を待つ。

---

## Phase 4: Create the Change

**EXPLAIN**

```text
## change を作成する

OpenSpec における "change" は、ある作業に関する思考や計画をまとめて入れておく入れ物です。`openspec/changes/<name>/` に置かれ、proposal、specs、design、tasks などの成果物をまとめます。

今回のタスク用に 1 つ作成しましょう。
```

**DO**
```bash
openspec new change "<derived-name>"
```

**SHOW**

```text
作成しました: `openspec/changes/<name>/`

フォルダ構成:
openspec/changes/<name>/
├─ proposal.md
├─ design.md
├─ specs/
└─ tasks.md

では、最初の成果物である proposal を埋めていきます。
```

---

## Phase 5: Proposal

**EXPLAIN**

```text
## proposal

proposal では、この change をなぜ行うのか、そして大まかに何を含むのかをまとめます。作業内容を短く伝えるための要約です。

タスクに基づいて草案を作成します。
```

**DO**

提案文をまず保存せずに草案として見せる:

```text
proposal の草案です:

---

## Why

[問題や機会を 1〜2 文で説明]

## What Changes

[何が変わるかを箇条書きで記載]

## Capabilities

### 新しい Capabilities
- `<capability-name>`: [簡単な説明]

### 変更される Capabilities
<!-- 既存の挙動を変更する場合 -->

## Impact

- `src/path/to/file.ts`: [変更内容]
- [必要なら他のファイル]

---

意図はこれで合っていますか？ 保存する前に調整できます。
```

**PAUSE** - 承認や修正依頼を待つ。

承認後:
```bash
openspec instructions proposal --change "<name>" --json
```
その後 `openspec/changes/<name>/proposal.md` に書く。

```text
Proposal saved. This is your "why" document - you can always come back and refine it as understanding evolves.

Next up: specs.
```

---

## Phase 6: Specs

**EXPLAIN**

```text
## specs

specs では、**何を作るのか** を、正確でテスト可能な形で定義します。requirement / scenario 形式を使うことで、期待される挙動を明確にできます。

このくらいの小さなタスクなら、1 つの spec ファイルだけで十分かもしれません。
```

**DO**
```bash
mkdir -p openspec/changes/<name>/specs/<capability-name>
```

草案:

```text
spec の草案です:

---

## ADDED Requirements

### Requirement: <Name>

<システムが何をすべきかの説明>

#### Scenario: <Scenario name>

- **WHEN** <きっかけとなる条件>
- **THEN** <期待される結果>
- **AND** <必要なら追加の結果>

---

この WHEN / THEN / AND 形式で、requirements をテスト可能にできます。
```

`openspec/changes/<name>/specs/<capability>/spec.md` に保存する。

---

## Phase 7: Design

**EXPLAIN**

```text
## design

design では、どうやって作るか、つまり技術的な判断、トレードオフ、進め方をまとめます。

小さな change なら簡潔で構いません。
```

**DO**

```text
design の草案です:

---

## Context

[現在の状態に関する簡単な文脈]

## Goals / Non-Goals

**Goals:**
- [達成したいこと]

**Non-Goals:**
- [明確に対象外とすること]

## Decisions

### Decision 1: [重要な判断]

[進め方と理由の説明]

---
```

`openspec/changes/<name>/design.md` に保存する。

---

## Phase 8: Tasks

**EXPLAIN**

```text
## tasks

最後に、作業を実装タスクへ分解します。これは apply フェーズを進めるためのチェックボックスです。

小さく、明確で、論理的な順序にしましょう。
```

**DO**

```text
実装タスクは以下です:

---

## 1. [カテゴリまたはファイル]

- [ ] 1.1 [具体的なタスク]
- [ ] 1.2 [具体的なタスク]

## 2. 検証

- [ ] 2.1 [確認手順]

---

各チェックボックスが apply フェーズでの作業単位になります。実装を始めますか？
```

**PAUSE** - 実装へ進む確認を待つ。

その後 `openspec/changes/<name>/tasks.md` に保存する。

---

## Phase 9: Apply (Implementation)

**EXPLAIN**

```text
## 実装

ここからは各タスクを実装し、進めながらチェックを付けていきます。各タスクの開始を知らせ、必要に応じて specs / design がどう役立つかも補足します。
```

**DO**

各タスクごとに:
1. `タスク N に取り組みます: [description]` と知らせる
2. コードを実装する
3. 必要に応じて specs / design を自然に参照する
4. tasks.md の `- [ ]` を `- [x]` にする
5. `タスク N 完了` と短く知らせる

説明は軽く保ち、講義調にしない。

完了後:

```text
## 実装完了

すべてのタスクが完了しました:
- [x] Task 1
- [x] Task 2
- [x] ...

change の実装が完了しました。最後に archive しましょう。
```

---

## Phase 10: Archive

**EXPLAIN**

```text
## archive

change が完了したら archive します。これにより `openspec/changes/` から `openspec/changes/archive/YYYY-MM-DD-<name>/` へ移動します。

archive された change は、プロジェクトの意思決定の履歴になります。
```

**DO**
```bash
openspec archive "<name>"
```

**SHOW**

```text
移動先: `openspec/changes/archive/YYYY-MM-DD-<name>/`

この change はプロジェクトの履歴の一部になりました。コードはコードベースにあり、意思決定の記録も残っています。
```

---

## Phase 11: Recap & Next Steps

```text
## おめでとうございます！

OpenSpec の一連の流れを最後まで体験できました:

1. **Explore** - 問題を考えた
2. **New** - change の入れ物を作った
3. **Proposal** - WHY を記録した
4. **Specs** - WHAT を定義した
5. **Design** - HOW を決めた
6. **Tasks** - 手順に分解した
7. **Apply** - 作業を実装した
8. **Archive** - 記録を残した

この流れは、どんな大きさの change にもそのまま使えます。

## Command Reference

| Command | 役割 |
|---------|--------------|
| `/opsx:explore` | 作業前/作業中に問題を考える |
| `/opsx:new` | 新しい change を開始し、成果物を順に作る |
| `/opsx:ff` | 一気に進める: すべての成果物をまとめて作る |
| `/opsx:continue` | 既存の change の作業を続ける |
| `/opsx:apply` | change のタスクを実装する |
| `/opsx:verify` | 実装が成果物と一致しているか確認する |
| `/opsx:archive` | 完了した change を archive する |

## 次は何をする？

実際に作りたいものを題材に、`/opsx:new` か `/opsx:ff` を試してみてください。
```

---

## Graceful Exit Handling

### User wants to stop mid-way

途中で止めたい場合:

```text
問題ありません。change は `openspec/changes/<name>/` に保存されています。

あとで続けるには:
- `/opsx:continue <name>` - 成果物作成を再開する
- `/opsx:apply <name>` - 実装に進む（tasks がある場合）

作業は失われません。準備ができたらまた戻ってきてください。
```

### User just wants command reference

チュートリアルを飛ばしてコマンド一覧だけ欲しい場合:

```text
## OpenSpec クイックリファレンス

| Command | 役割 |
|---------|--------------|
| `/opsx:explore` | 問題を考える（コード変更なし） |
| `/opsx:new <name>` | 新しい change を段階的に開始する |
| `/opsx:ff <name>` | 一気に進める: すべての成果物をまとめて作る |
| `/opsx:continue <name>` | 既存の change を続ける |
| `/opsx:apply <name>` | タスクを実装する |
| `/opsx:verify <name>` | 実装を検証する |
| `/opsx:archive <name>` | 完了したら archive する |
```

---

## Guardrails

- 主要な切り替え点では **EXPLAIN -> DO -> SHOW -> PAUSE** を守る
- 実装中の説明は軽く保つ
- change が小さくても phase を飛ばさない
- 指定された pause では確認を待つが、止まりすぎない
- 離脱したいユーザーを無理に引き留めない
- 実際のコードベース上の本物のタスクを使う
- スコープはやわらかく小さめへ導くが、最終判断はユーザーに任せる

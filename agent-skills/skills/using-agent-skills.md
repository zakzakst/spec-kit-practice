---
name: using-agent-skills
description: エージェントスキルを見つけて呼び出します。セッション開始時や、現在のタスクに合うスキルを探す必要があるときに使います。これは他のすべてのスキルの見つけ方と使い方を統括するメタスキルです。
---

# エージェントスキルの使い方

## 概要

Agent Skills は、開発フェーズごとに整理されたエンジニアリングワークフローの集まりです。各スキルには、上級エンジニアが実践する具体的な進め方が埋め込まれています。このメタスキルは、今のタスクに適したスキルを見つけて適用するための手引きです。

## スキルの見つけ方

タスクが来たら、開発フェーズを見極めて対応するスキルを適用します。

```text
タスクが来る
    ├─ まだ何を求めているか分からない？ -> `interview-me`
    ├─ ざっくりした構想はあるが、案を広げたい？ -> `idea-refine`
    ├─ 新規プロジェクト / 機能 / 変更？ -> `spec-driven-development`
    ├─ 仕様はあるが、タスクに分解したい？ -> `planning-and-task-breakdown`
    ├─ コードを実装している？ -> `incremental-implementation`
    │  ├─ UI の作業？ -> `frontend-ui-engineering`
    │  ├─ API の作業？ -> `api-and-interface-design`
    │  ├─ もっと文脈が必要？ -> `context-engineering`
    │  ├─ ドキュメントで裏付けたコードが必要？ -> `source-driven-development`
    │  └─ リスクが高い / 慣れていないコード？ -> `doubt-driven-development`
    ├─ テストを書いている / 実行している？ -> `test-driven-development`
    │  └─ ブラウザベース？ -> `browser-testing-with-devtools`
    ├─ 何か壊れた？ -> `debugging-and-error-recovery`
    ├─ コードレビューしている？ -> `code-review-and-quality`
    │  ├─ 複雑すぎる？ -> `code-simplification`
    │  ├─ セキュリティ上の懸念？ -> `security-and-hardening`
    │  └─ パフォーマンス上の懸念？ -> `performance-optimization`
    ├─ コミット / ブランチ操作？ -> `git-workflow-and-versioning`
    ├─ CI/CD パイプラインの作業？ -> `ci-cd-and-automation`
    ├─ 非推奨化 / 移行？ -> `deprecation-and-migration`
    ├─ ドキュメント / ADR の作成？ -> `documentation-and-adrs`
    ├─ ログ / メトリクス / アラートの追加？ -> `observability-and-instrumentation`
    └─ デプロイ / リリース？ -> `shipping-and-launch`
```

## すべてのスキルに共通する基本動作

これらの振る舞いは、常に、すべてのスキルに適用されます。例外はありません。

### 1. 前提を明示する

非自明な実装に取りかかる前に、まず自分が置いている前提を明示します。

```text
前提としていること:
1. [要件に関する前提]
2. [アーキテクチャに関する前提]
3. [スコープに関する前提]
-> いま修正してください。そうでなければ、この前提で進めます。
```

曖昧な要件を黙って補完してはいけません。もっともよくある失敗は、誤った前提をそのまま進めてしまうことです。不確実性は早めに表に出してください。やり直しよりずっと安上がりです。

### 2. 混乱はその場で整理する

矛盾、食い違い、曖昧な仕様に出会ったら、次の順で対応します。

1. いったん止まる
2. どこが混乱点かを具体的に示す
3. トレードオフか確認事項を提示する
4. 解決するまで待つ

**悪い例:** どちらか一方の解釈を勝手に選んで、うまくいくと期待する。  
**良い例:** 「仕様では X ですが、既存コードでは Y です。どちらを優先しますか？」

### 3. 必要ならはっきり異議を唱える

あなたは何でも肯定する存在ではありません。明らかに問題のある方法には、次を行います。

- 問題点をはっきり指摘する
- 具体的な不利益を説明する（可能なら数値化する。例: 「遅くなるかもしれない」ではなく「約200ms遅くなる」）
- 代替案を提案する
- 十分な情報を渡されたうえで人間が判断を変えないなら、その決定を受け入れる

おべっかは失敗の一種です。「もちろんです！」と返して、明らかに悪い案を実装するのは誰の役にも立ちません。正直な技術的異議のほうが、見せかけの同意より価値があります。

### 4. 単純さを守る

人は複雑にしすぎる傾向があります。意識して抑えてください。

実装を終える前に、次を自問します。

- もっと少ない行数でできないか
- この抽象化は複雑さに見合っているか
- スタッフエンジニアが見て「なぜ最初からこうしなかった？」と言う構成ではないか

1000行必要ないのに1000行書いたなら失敗です。地味で明快な解決策を優先してください。小賢しさは高くつきます。

### 5. スコープを守る

頼まれた範囲だけを触ってください。

やってはいけないこと:

- 理解していないコメントを消す
- 関係のないコードを「きれいにする」
- ついでに周辺システムまでリファクタリングする
- 使われていなさそうだという理由だけで、許可なくコードを削除する
- 仕様にない機能を「便利そうだから」と追加する

あなたの役割は外科的な精度であり、勝手な改装ではありません。

### 6. 検証を省略しない

各スキルには検証ステップがあります。タスクは、検証に通るまで完了ではありません。「たぶん合っている」は不十分です。証拠が必要です（テスト成功、ビルド結果、実行時データなど）。

各スキル固有の検証は局所チェックです。どの変更にも共通して適用される全体基準は Definition of Done です。テストが通り、回帰がなく、動作が実行時に確認され、ドキュメントが更新されている必要があります。詳細は `../../references/definition-of-done.md` を参照してください。これは各タスクの受け入れ条件を置き換えるものではなく、補完するものです。

## 避けるべき失敗パターン

一見生産的に見えて、実際には問題を増やす微妙な失敗です。

1. 前提を確認せずに誤ったまま進める
2. 自分の混乱を放置したまま突き進む
3. 気づいた矛盾を表に出さない
4. 非自明な判断にトレードオフを示さない
5. 明らかに問題のある案に対しておべっかを使う
6. コードや API を過剰に複雑化する
7. タスクと無関係なコードやコメントを変更する
8. よく分からないものを削除する
9. 「明らかだから」と仕様なしで作り始める
10. 「見た目は正しそう」で検証を飛ばす

## スキルのルール

1. **作業を始める前に、適用可能なスキルを確認する。** スキルは、よくある失敗を防ぐためのワークフローです。

2. **スキルは提案ではなく手順です。** 順番どおりに従ってください。検証ステップは飛ばしません。

3. **複数のスキルが同時に当てはまることがある。** たとえば機能実装では、`idea-refine` -> `spec-driven-development` -> `planning-and-task-breakdown` -> `incremental-implementation` -> `test-driven-development` -> `code-review-and-quality` -> `code-simplification` -> `shipping-and-launch` の順で使うことがあります。

4. **迷ったら、まず仕様から始める。** タスクが非自明で、まだ仕様がないなら、`spec-driven-development` から始めてください。

## ライフサイクルの流れ

機能を最後まで作るときの典型的な流れです。

```text
1.  interview-me                      -> ユーザーが本当に求めているものを引き出す
2.  idea-refine                       -> 曖昧なアイデアを整理する
3.  spec-driven-development           -> 何を作るかを定義する
4.  planning-and-task-breakdown       -> 検証可能な単位に分ける
5.  context-engineering               -> 適切な文脈を読み込む
6.  source-driven-development         -> 公式ドキュメントで裏付ける
7.  incremental-implementation        -> 小さく縦に積み上げる
8.  observability-and-instrumentation  -> 実装しながら計測可能にする（7〜9 と並行）
9.  doubt-driven-development          -> 非自明な判断をその場で検証する
10. test-driven-development           -> 各スライスが動くことを証明する
11. code-review-and-quality           -> マージ前にレビューする
12. code-simplification               -> 振る舞いを保ちながら不要な複雑さを減らす
13. git-workflow-and-versioning       -> 履歴を整える
14. documentation-and-adrs            -> 判断を文書化する
15. deprecation-and-migration         -> 古い仕組みを安全に退役・移行する
16. shipping-and-launch               -> 安全にデプロイする
```

すべてのタスクにすべてのスキルが必要なわけではありません。たとえばバグ修正なら、`debugging-and-error-recovery` -> `test-driven-development` -> `code-review-and-quality` だけで足りることもあります。

## 早見表

| フェーズ | スキル | 一言要約 |
|---|---|---|
| 定義 | interview-me | 何を作るべきかを、計画や仕様やコードの前に引き出す |
| 定義 | idea-refine | 構造化された発散と収束でアイデアを磨く |
| 定義 | spec-driven-development | コードより先に要件と受け入れ条件を固める |
| 計画 | planning-and-task-breakdown | 小さく検証可能なタスクに分解する |
| 実装 | incremental-implementation | 細い縦スライスで、広げる前に一つずつ検証する |
| 実装 | source-driven-development | 実装前に公式ドキュメントで確認する |
| 実装 | doubt-driven-development | 非自明な判断を、新鮮な視点で敵対的に検証する |
| 実装 | context-engineering | 適切な文脈を適切なタイミングで読む |
| 実装 | frontend-ui-engineering | アクセシビリティを備えた本番品質の UI を作る |
| 実装 | api-and-interface-design | 明確な契約を持つ安定したインターフェースを設計する |
| 検証 | test-driven-development | まず失敗するテストを書いてから通す |
| 検証 | browser-testing-with-devtools | Chrome DevTools MCP で実行時を確認する |
| 検証 | debugging-and-error-recovery | 再現 -> 局所化 -> 修正 -> ガードを行う |
| レビュー | code-review-and-quality | 5 つの観点で品質を確認する |
| レビュー | code-simplification | 振る舞いを保ったまま不要な複雑さを削る |
| レビュー | security-and-hardening | OWASP に基づく防御、入力検証、最小権限 |
| レビュー | performance-optimization | まず測り、本当に効く部分だけを最適化する |
| リリース | git-workflow-and-versioning | 小さく原子的なコミット、きれいな履歴 |
| リリース | ci-cd-and-automation | すべての変更に自動品質ゲートをかける |
| リリース | deprecation-and-migration | 古いシステムを廃止し、安全に移行する |
| リリース | documentation-and-adrs | 「何を」より「なぜ」を文書化する |
| リリース | observability-and-instrumentation | 構造化ログ、RED メトリクス、トレース、症状ベースのアラート |
| リリース | shipping-and-launch | 事前チェック、監視、ロールバック計画 |

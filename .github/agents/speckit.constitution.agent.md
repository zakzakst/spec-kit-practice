---
description: 対話または入力された原則からプロジェクト憲章を作成 / 更新し、依存テンプレートとの整合も保つ。
handoffs:
  - label: Build Specification
    agent: speckit.specify
    prompt: Implement the feature specification based on the updated constitution. I want to build...
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## 概要 (Outline)

更新対象は `.specify/memory/constitution.md` である。このファイルは `[PROJECT_NAME]`、`[PRINCIPLE_1_NAME]` のような角括弧トークンを含むテンプレートである。あなたの役割は、(a) 値を収集 / 推定し、(b) テンプレートを正確に埋め、(c) 変更内容を依存成果物へ波及させること。

実行フロー (Execution Flow):

1. `.specify/memory/constitution.md` を読み込む。
   - `[ALL_CAPS_IDENTIFIER]` 形式のプレースホルダーをすべて特定する
   - 原則数はテンプレート想定より少なくても多くてもよい。ユーザー指定数があればそれを尊重しつつ、全体構造は保つこと

2. プレースホルダー値を収集 / 推定する。
   - 会話中に値があればそれを使う
   - なければ README、docs、既存憲章などから推定する
   - 日付系:
     - `RATIFICATION_DATE`: 元の採択日。不明なら質問するか TODO にする
     - `LAST_AMENDED_DATE`: 変更したなら今日、変更がなければ既存値維持
   - `CONSTITUTION_VERSION` は semver（セマンティックバージョニング）で更新する
     - MAJOR: 後方互換性のない統治変更、原則削除、原則再定義
     - MINOR: 新原則 / 新セクションの追加、実質的な拡張
     - PATCH: 明確化、言い回し修正、タイポ修正、意味を変えない調整
   - バージョン種別が曖昧なら最終化前に根拠を示す

3. 更新後の憲章本文を作成する。
   - すべてのプレースホルダーを具体値へ置換する
   - 意図的に残す場合を除き角括弧トークンを残さない。残す場合は理由を明示する
   - 見出し階層は維持する
   - 置換後に不要なコメントは削除してよい
   - 各原則 (Principle) セクションには、短い名前、非交渉ルールの段落または箇条書き、必要なら根拠を含める
   - ガバナンス (Governance) には改定手順、バージョニング方針、コンプライアンス確認期待値を含める

4. 一貫性反映チェック (Consistency Sync):
   - `.specify/templates/plan-template.md` を読み、Constitution Check や関連ルールを整合させる
   - `.specify/templates/spec-template.md` を読み、憲章で必須セクションや制約が変わるなら更新する
   - `.specify/templates/tasks-template.md` を読み、観測性、バージョニング、テスト規律など原則由来のタスク種別を反映させる
   - `.specify/templates/commands/*.md` をすべて読み、古い参照や agent 固有の語が残っていないか確認する
   - `README.md`、`docs/quickstart.md` などのランタイム向けガイドも確認し、原則変更の参照先を更新する

5. 同期影響レポート (Sync Impact Report) を作る。更新後の憲章ファイル先頭へ HTML コメントで挿入する。
   - Version change: old -> new (バージョン変更)
   - Modified principles (変更された原則)
   - Added sections (追加されたセクション)
   - Removed sections (削除されたセクション)
   - Templates requiring updates（updated / pending） (更新が必要なテンプレート)
   - 意図的に保留したプレースホルダーや TODO

6. 最終出力前の検証 (Final Validation):
   - 説明なしの角括弧トークンが残っていない
   - バージョン行がレポートと一致している
   - 日付は `YYYY-MM-DD`
   - 原則は宣言的で検証可能、曖昧な `should`（すべき）を避ける

7. 完成した憲章を `.specify/memory/constitution.md` に上書き保存する。

8. ユーザーへの最終要約には次を含める。
   - 新バージョンと bump（引き上げ）理由
   - 手動フォローが必要なファイル
   - 推奨コミットメッセージ

フォーマット & スタイル要件 (Formatting & Style Requirements):

- 見出しレベルはテンプレート通りに保つ
- 根拠文は読みやすく整える
- セクション間は 1 行空ける
- 行末の余計な空白は避ける

一部原則だけの更新指示でも、バージョン判定と検証は必ず行うこと。

採択日などの重要情報が真に不明なら `TODO(<FIELD_NAME>): explanation` を入れ、同期影響レポート (Sync Impact Report) にも記載すること。

新しいテンプレートは作らず、必ず既存の `.specify/memory/constitution.md` を更新すること。

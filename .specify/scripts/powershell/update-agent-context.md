## `update-agent-context.ps1` の処理概要

このスクリプトは、`plan.md` から抽出した情報をもとに、各種 AI エージェント向けのコンテキスト／指示ファイルを生成または更新します。

### 1. 引数

- `-AgentType <agent>` : 指定したエージェントのみ更新
- 省略時は、既存の agent ファイルすべてを更新（なければ `CLAUDE.md` を作成）

### 2. 初期セットアップ

- `common.ps1` を読み込み
- `Get-FeaturePathsEnv` で feature 関連パスと現在ブランチ情報を取得
- `plan.md` とテンプレートファイルの存在を検証

### 3. `plan.md` から情報を抽出

- `Extract-PlanField` で次の項目を抽出
  - `Language/Version`
  - `Primary Dependencies`
  - `Storage`
  - `Project Type`
- 抽出した値を `NEW_LANG`, `NEW_FRAMEWORK`, `NEW_DB`, `NEW_PROJECT_TYPE` に格納

### 4. コンテンツ生成ロジック

- `Format-TechnologyStack` : 言語とフレームワークを結合してスタック文字列を生成
- `Get-ProjectStructure` : プロジェクト構造を `web` 判定で決定
- `Get-CommandsForLanguage` : 言語固有のコマンド例を生成
- `Get-LanguageConventions` : 言語固有の開発慣習メッセージを生成

### 5. 新規エージェントファイルの作成

- `New-AgentFile`
  - テンプレート agent-file-template.md を読み込み
  - プロジェクト名、日付、技術スタック、構造、コマンド、慣習、最近の変更を埋め込む
  - 目的のファイルパスに UTF-8 で保存

### 6. 既存ファイルの更新

- `Update-ExistingAgentFile`
  - 既存ファイル内に新しい技術スタックや DB 情報が未登録なら追加
  - `## Active Technologies` と `## Recent Changes` セクションを更新
  - `Last updated` 日付を現在日付に差し替え

### 7. エージェント単体／全体更新

- `Update-SpecificAgent`
  - `-AgentType` で指定したエージェントだけ更新
- `Update-AllExistingAgents`
  - 既存の agent ファイルをすべて更新
  - 既存ファイルが見つからない場合は `CLAUDE.md` を作成

### 8. 最終出力

- 処理の結果をログ出力し、成功またはエラーで終了コードを返す
- 更新された内容の概要（言語、フレームワーク、データベース）を表示

---

## 役割

`update-agent-context.ps1` は、最新の `plan.md` から技術スタックやプロジェクト情報を自動抽出し、複数の AI エージェント向けコンテキストファイルを一貫して生成／更新するためのユーティリティです。

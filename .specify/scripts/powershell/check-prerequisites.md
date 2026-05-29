Optimized tool selection## `check-prerequisites.ps1` の処理概要

この PowerShell スクリプトは、Spec-Driven Development の前提条件を統合的にチェックするためのものです。主に以下を行います。

### 1. オプション解析
- `-Json` : 出力を JSON 形式にする
- `-RequireTasks` : `tasks.md` の存在を必須とする
- `-IncludeTasks` : 利用可能なドキュメント一覧に `tasks.md` を追加
- `-PathsOnly` : 検証せずパス情報だけ出力
- `-Help` / `-h` : ヘルプ表示

### 2. 共通関数の読み込み
- `common.ps1` を `. $PSScriptRoot/common.ps1` で読み込み
- ここで、パス取得や Git ブランチ検証など共通ロジックを使えるようにする

### 3. フィーチャーパスとブランチの検証
- `Get-FeaturePathsEnv` でリポジトリルート、現在ブランチ、feature ディレクトリ、`plan.md`、`tasks.md` などのパスを取得
- `Test-FeatureBranch` でブランチが正しいか検証し、失敗したら終了

### 4. `-PathsOnly` モード
- 指定された場合は、以下のパス情報だけを出力して終了
  - `REPO_ROOT`
  - `BRANCH`
  - `FEATURE_DIR`
  - `FEATURE_SPEC`
  - `IMPL_PLAN`
  - `TASKS`

### 5. 必須ファイルの存在チェック
- `FEATURE_DIR` が存在するか
- `plan.md` (`IMPL_PLAN`) が存在するか
- `-RequireTasks` 指定時は `tasks.md` が存在するか

### 6. 利用可能ドキュメントのリスト作成
- 以下のファイルが存在すれば `AVAILABLE_DOCS` に追加
  - `research.md`
  - `data-model.md`
  - `contracts/`（ディレクトリ内にファイルがある場合）
  - `quickstart.md`
  - `tasks.md`（`-IncludeTasks` 指定時のみ）

### 7. 出力
- `-Json` の場合は JSON 形式で返す
- そうでない場合はテキスト形式で `FEATURE_DIR` と各ドキュメントの状態を表示

---

## 役割まとめ

このスクリプトは、feature 仕様や実装計画の作成前後に必要なファイル構成をチェックし、準備状況を簡単に確認できるようにするためのものです。`plan.md` を必須とし、`tasks.md` はモードに応じてチェック対象に含められます。
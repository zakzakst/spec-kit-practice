## `setup-plan.ps1` の処理概要

このスクリプトは、Feature の実装計画 (`plan.md`) をセットアップするためのものです。

### 1. オプション解析

- `-Json` : 結果を JSON 形式で出力
- `-Help` : ヘルプを表示

### 2. 共通処理の読み込み

- `. "$PSScriptRoot/common.ps1"` で共通関数を読み込み
- `Get-FeaturePathsEnv` で feature に関連するパスを取得

### 3. ブランチ検証

- `Test-FeatureBranch` で現在ブランチが `###-...` 形式の feature ブランチか確認
- Git がない環境では検証をスキップするが警告は出さない

### 4. Feature ディレクトリの作成

- `specs/<branch>` フォルダが存在しない場合でも `New-Item -ItemType Directory -Force` で作成

### 5. `plan.md` の準備

- plan-template.md があればコピーして `plan.md` を生成
- テンプレートがなければ警告を出し、空の `plan.md` を作成

### 6. 結果出力

- `-Json` の場合は JSON 形式で `FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, `BRANCH`, `HAS_GIT` を出力
- それ以外はテキストで同様の情報を表示

---

## 役割

このスクリプトは、Feature 開発の「実装計画」フェーズを始めるために必要な `plan.md` ファイルを自動的にセットアップし、feature 作業ディレクトリを整えるための補助ツールです。

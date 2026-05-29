## `common.ps1` の処理概要

このファイルは、.specify の PowerShell スクリプトで共通に使われる補助関数を定義しています。主にリポジトリルートの検出、ブランチ判定、Feature ディレクトリパス生成、ファイル/ディレクトリ存在チェックを行います。

### 1. リポジトリルートの取得

- `Get-RepoRoot`
  - まず `git rev-parse --show-toplevel` で Git リポジトリのルートを取得
  - Git が使えない場合は、スクリプト位置から相対パス OneDrive を解決して返す

### 2. 現在のブランチの取得

- `Get-CurrentBranch`
  - 優先して環境変数 `SPECIFY_FEATURE` を返す
  - それがなければ Git ブランチ名を取得
  - Git も使えない場合は specs 以下の最新の `###-...` 形式ディレクトリ名を探して返す
  - それでも見つからなければ `"main"` を返す

### 3. Git の存在判定

- `Test-HasGit`
  - Git コマンドが実行できるかを確認し、リポジトリかどうかを判定

### 4. Feature ブランチ名の検証

- `Test-FeatureBranch`
  - Git がなければ検証をスキップして警告を出す
  - ブランチ名が `^[0-9]{3}-` 形式かどうかをチェック
  - 形式でなければエラーを出力して `false` を返す

### 5. Feature ディレクトリのパス生成

- `Get-FeatureDir`
  - リポジトリルートとブランチ名から `specs/<branch>` を組み立てる

### 6. Feature 関連パスをまとめて取得

- `Get-FeaturePathsEnv`
  - `REPO_ROOT`, `CURRENT_BRANCH`, `HAS_GIT`
  - `FEATURE_DIR`, `FEATURE_SPEC`, `IMPL_PLAN`, `TASKS`
  - `RESEARCH`, `DATA_MODEL`, `QUICKSTART`, `CONTRACTS_DIR`
  - これらを `PSCustomObject` で返す

### 7. ファイル / ディレクトリの存在チェック

- `Test-FileExists`
  - 指定パスがファイルとして存在するかを判定し、`✓` / `✗` を出力
- `Test-DirHasFiles`
  - 指定パスがディレクトリで、かつ中にファイルがひとつでもあるかを判定し、`✓` / `✗` を出力

---

## 役割

この `common.ps1` は、.specify の各種スクリプトから再利用される共通ヘルパー群です。
`check-prerequisites.ps1` のような前提条件チェックや、Feature パス設定の基盤ロジックを提供します。

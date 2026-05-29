## `create-new-feature.ps1` の処理概要

このスクリプトは、新しい Feature ブランチと specs 下の対応ディレクトリを作成し、仕様ファイルのひな形を準備します。

### 1. オプション解析

- `-Json` : 出力を JSON 形式にする
- `-ShortName <name>` : ブランチ名の末尾部分を手動で指定
- `-Number N` : ブランチ番号を手動指定
- `-Help` : ヘルプ表示

### 2. 機能説明の受け取り

- コマンドラインの残り引数を結合して `featureDesc` を生成
- 説明がない場合はエラー終了

### 3. リポジトリルートの決定

- `Find-RepositoryRoot` で現在のディレクトリから .git または .specify を探し、ルートを決定
- Git が利用可能なら `git rev-parse --show-toplevel` を優先
- Git なしでも .specify マーカー経由でリポジトリルートを見つける

### 4. branch 番号の決定

- `Get-HighestNumberFromSpecs` で specs 内の既存機能ディレクトリから最大番号を取得
- `Get-HighestNumberFromBranches` で既存 Git ブランチから最大番号を取得
- `Get-NextBranchNumber` で両方を比較し、次の番号を決定
- `-Number` が指定されていればその値を優先

### 5. ブランチ名の生成

- `Get-BranchName` で説明文から短いブランチ名候補を生成
  - 小さな stopword を除去
  - 3文字以上の意味のある単語を選択
  - 先頭 3〜4 語をハイフン区切りで組み立て
- `-ShortName` が指定されていればそれをクリーニングして利用

### 6. ブランチ名の検証・切り詰め

- `244 バイト` の GitHub ブランチ名制限を超える場合、末尾を切り詰め
- 切り詰め時には警告を出力

### 7. ブランチ作成

- Git がある場合は `git checkout -b <branchName>` で新規ブランチを作成
- Git が無い場合は警告を出してスキップ

### 8. Feature ディレクトリと spec ファイルの作成

- `specs/<branchName>` ディレクトリを作成
- spec-template.md があればコピーし、なければ空の `spec.md` を作成

### 9. セッション環境変数の設定

- `$env:SPECIFY_FEATURE` に新しいブランチ名をセット

### 10. 出力

- `-Json` 時は JSON オブジェクトで結果を返す
- それ以外はテキスト形式で作成したブランチ名や spec ファイルパスを出力

---

## 役割

このスクリプトは、新しい機能開発の開始時に必要な Feature ブランチと仕様ファイルの雛形を自動生成し、specs ワークフローをスムーズに始めるための支援ツールです。

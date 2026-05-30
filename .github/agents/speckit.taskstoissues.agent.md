---
description: 既存タスクを、利用可能な設計成果物に基づいて依存順の GitHub Issues へ変換する。
tools: ['github/github-mcp-server/issue_write']
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## 概要 (Outline)

1. リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` を実行し、`FEATURE_DIR` と `AVAILABLE_DOCS` を取得する。パスはすべて絶対パスとする。引数中のシングルクォートは必要に応じてエスケープする。
2. 実行結果から **tasks** のパスを抽出する。
3. 次のコマンドで Git remote を取得する。

```bash
git config --get remote.origin.url
```

> [!CAUTION]
> remote URL が GitHub の場合にのみ次へ進むこと。

4. タスクリストの各項目について、Git remote に対応するリポジトリへ GitHub MCP サーバーを使って Issue を作成する。

> [!CAUTION]
> remote URL と一致しないリポジトリに Issue を作成してはならない。

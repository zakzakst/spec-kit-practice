---
name: chronicle:search
description: キーワード、ファイルパス、またはPR/Issueへの参照を使用して、最近のチャットセッションを検索します
---
私が提供するクエリ（キーワード、ファイルパス、PR/Issue/Commitへの参照など）でCopilotのセッション履歴を検索し、一致するセッションをリストアップしてください。**chronicle** スキルを使用してください。ここには、`copilot_sessionStoreSql` ツール、セッションストアのスキーマ（`sessions` テーブルの主キーは `id` です。会話の内容は `sessions` ではなく `turns` に保存されます。ローカルの SQLite では FTS5 の `search_index` テーブルを使用し、`session_id` を直接 select してください。決して `search_index.rowid` と `turns.rowid` を結合しないでください）、およびクラウドのパフォーマンスルール（`WITH hits ... JOIN sessions` による1回の集計、`turns` のデフォルト90日間ウィンドウ）を含む検索ワークフローが文書化されています。

---
name: get-search-view-results
description: 'VS Codeの検索ビューから現在の検索結果を取得します'
---

# 検索ビューの結果を取得する

1. VS Code には検索ビューがあり、既存の検索結果を持つことができます。
2. 現在の検索結果を取得するには、VS Code コマンド `search.action.getSearchResults` を使用できます。
3. そのコマンドを `copilot_runVscodeCommand` ツール経由で実行します。コマンドが存在することはわかっているため、存在チェックを回避するために必ず `skipCheck` 引数を true に設定してください。
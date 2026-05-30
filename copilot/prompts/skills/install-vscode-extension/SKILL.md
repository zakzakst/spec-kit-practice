---
name: install-vscode-extension
description: '拡張機能IDを使用してVS Code拡張機能をインストールする方法。ユーザーが拡張機能をインストールしてVS Code環境に新しい機能を追加したい場合に便利です。'
---

# VS Code 拡張機能のインストール

1. VS Code 拡張機能は通常、`publisher.extensionName` の形式に従う一意の拡張機能 ID によって識別されます。たとえば、Microsoft の Python 拡張機能の ID は `ms-python.python` です。
2. VS Code 拡張機能をインストールするには、VS Code コマンド `workbench.extensions.installExtension` を使用し、拡張機能 ID を渡す必要があります。引数の形式は次のとおりです。
```
[extensionId, { enable: true, installPreReleaseVersion: boolean }]
```
> メモ: ユーザーが明示的に言及した場合、または現在の環境が VS Code Insiders である場合は、プレリリース版の拡張機能をインストールします。それ以外の場合は、安定版をインストールします。
3. そのコマンドを `copilot_runVscodeCommand` ツール経由で実行します。コマンドが存在することはわかっているため、存在チェックを回避するために必ず `skipCheck` 引数を true に設定してください。
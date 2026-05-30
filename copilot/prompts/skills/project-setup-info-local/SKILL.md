---
name: project-setup-info-local
description: 'ユーザーが VS Code ワークスペースに完全なプロジェクト構造を作成するのに役立つ包括的なセットアップ手順。このツールは、個々のファイルを作成するためではなく、プロジェクト全体の初期化とスキャフォールディングを目的として設計されています。このツールを使用するタイミング: ユーザーがゼロから新しい完全なプロジェクトを作成したい場合。プロジェクトフレームワーク全体 (TypeScript プロジェクト、React アプリ、Node.js サーバーなど) をセットアップする場合。完全な構造を持つ Model Context Protocol (MCP) サーバーを初期化する場合。適切なスキャフォールディングを持つ VS Code 拡張機能を作成する場合。Next.js、Vite、またはその他のフレームワークベースのプロジェクトをセットアップする場合。ユーザーが「新しいプロジェクト」、「ワークスペースの作成」、または「[フレームワーク]プロジェクトのセットアップ」を求めている場合。依存関係、構成ファイル、フォルダー構造を備えた完全な開発環境を確立する必要がある場合。このツールを使用しないタイミング: 単一のファイルまたは小さなコード スニペットを作成する場合。既存のプロジェクトに個々のファイルを追加する場合。既存のコードベースを変更する場合。ユーザーが「ファイルの作成」または「コンポーネントの追加」を求めている場合。単純なコード例やデモンストレーションの場合。既存のコードのデバッグや修正の場合。このツールは、フォルダー構造の作成、package.json と依存関係の管理、構成ファイル (tsconfig、eslint など)、初期のボイラープレート コード、開発環境のセットアップ、ビルドおよび実行手順を含む、完全なプロジェクトセットアップを提供します。既存のプロジェクト内の個々のファイルには、他のファイル作成ツールを使用してください。'
---

# プロジェクトのセットアップ方法

ユーザーがどのような種類のプロジェクトを作成したいかを判断し、それに基づいて以下のどのセットアップ情報を従うかを選択します。フォルダーが空の場合、またはワークスペースを作成するために最初にツールを呼び出した直後の場合にのみ、プロジェクトをセットアップしてください。

## vscode-extension

Yeoman と Generator-Code を使用して VS Code 拡張機能を作成するためのテンプレート。

次のコマンドを実行します:

```
npx --package yo --package generator-code -- yo code . --skipOpen
```
このコマンドには次の引数があります:

- `-t, --extensionType`: 拡張機能のタイプを指定します: ts, js, command-ts, command-js, colortheme, language, snippets, keymap, extensionpack, localization, commandweb, notebook。デフォルトは `ts` です。
- `-n, --extensionDisplayName`: 拡張機能の表示名を設定します。
- `--extensionId`: 拡張機能の一意の ID を設定します。ユーザーが一意の ID をリクエストしていない場合は、このオプションを選択しないでください。
- `--extensionDescription`: 拡張機能の説明を提供します。
- `--pkgManager`: パッケージマネージャーを指定します: npm, yarn, または pnpm。デフォルトは `npm` です。
- `--bundler`: webpack または esbuild を使用して拡張機能をバンドルします。
- `--gitInit`: 拡張機能用の Git リポジトリを初期化します。
- `--snippetFolder`: スニペットフォルダの場所を指定します。
- `--snippetLanguage`: スニペットの言語を設定します。

### ルール

1. コマンドから引数を削除しないでください。ユーザーが要求した場合にのみ引数を追加してください。
2. 関連する参照を取得するために、ユーザーのクエリを使用して `get_vscode_api` ツールを呼び出します。
3. `get_vscode_api` ツールが完了した後、そのときに初めてプロジェクトの変更を開始してください。

## next-js

サーバーレンダリングの Web アプリケーションを構築するための React ベースのフレームワーク。

次のコマンドを実行します:

```
npx create-next-app@latest .
```
このコマンドには次の引数があります:

- `--ts, --typescript`: TypeScript プロジェクトとして初期化します。これがデフォルトです。
- `--js, --javascript`: JavaScript プロジェクトとして初期化します。
- `--tailwind`: Tailwind CSS の構成で初期化します。これがデフォルトです。
- `--eslint`: ESLint の構成で初期化します。
- `--app`: App Router プロジェクトとして初期化します。
- `--src-dir`: 'src/' ディレクトリ内に初期化します。
- `--turbopack`: 開発用にデフォルトで Turbopack を有効にします。
- `--import-alias <prefix/*>`: 使用するインポートエイリアスを指定します。(デフォルトは "@/*")
- `--api`: App Router を使用してヘッドレス API を初期化します。
- `--empty`: 空のプロジェクトを初期化します。
- `--use-npm`: npm を使用してアプリケーションをブートストラップするよう CLI に明示的に指示します。
- `--use-pnpm`: pnpm を使用してアプリケーションをブートストラップするよう CLI に明示的に指示します。
- `--use-yarn`: Yarn を使用してアプリケーションをブートストラップするよう CLI に明示的に指示します。
- `--use-bun`: Bun を使用してアプリケーションをブートストラップするよう CLI に明示的に指示します。

## vite

スピードとパフォーマンスに焦点を当てた、Web アプリケーション用のフロントエンドビルドツール。React, Vue, Preact, Lit, Svelte, Solid, Qwik とともに使用できます。

次のコマンドを実行します:

```
npx create-vite@latest .
```
このコマンドには次の引数があります:

- `-t, --template NAME`: 特定のテンプレートを使用します。利用可能なテンプレート: vanilla-ts, vanilla, vue-ts, vue, react-ts, react, react-swc-ts, react-swc, preact-ts, preact, lit-ts, lit, svelte-ts, svelte, solid-ts, solid, qwik-ts, qwik

## mcp-server

Model Context Protocol (MCP) サーバープロジェクト。このプロジェクトは、TypeScript, JavaScript, Python, C#, Java, Kotlin を含む複数のプログラミング言語をサポートしています。

### ルール

1. まず、https://github.com/modelcontextprotocol にアクセスして、要求された言語に対する正しい SDK とセットアップ手順を見つけます。言語が指定されていない場合は、TypeScript をデフォルトとします。
2. `fetch_webpage` ツールを使用して、https://modelcontextprotocol.io/llms-full.txt から正しい実装手順を見つけます。
3. SDK ドキュメントへの参照を含めるように、.github ディレクトリ内の copilot-instructions.md ファイルを更新します。
4. プロジェクトルートの `.vscode` フォルダに、次の内容で `mcp.json` ファイルを作成します: `{ "servers": { "mcp-server-name": { "type": "stdio", "command": "command-to-run", "args": [list-of-args] } } }`。
   - mcp-server-name: MCP サーバーの名前。この MCP サーバーが何を行うかを反映する一意の名前を作成します。
   - command-to-run: MCP サーバーを開始するために実行するコマンド。これは、作成したばかりのプロジェクトを実行するために使用するコマンドです。
   - list-of-args: コマンドに渡す引数。これは、作成したばかりのプロジェクトを実行するために使用する引数のリストです。
5. 選択した言語に基づいて、必要な VS Code 拡張機能をインストールします（例: Python プロジェクト用の Python 拡張機能）。
6. VS Code を使用してこの MCP サーバーをデバッグできるようになったことをユーザーに通知します。

## python-script

単一のスクリプトだけを作成する場合に選択する必要がある、シンプルな Python スクリプトプロジェクト。

必須の拡張機能: `ms-python.python`, `ms-python.vscode-python-envs`

### ルール

1. `copilot_runVscodeCommand` ツールを呼び出して、VS Code で新しい Python スクリプトプロジェクトを正しく作成します。次の引数を使用してコマンドを呼び出します。
   "python-script" と "true" は定数であり、"New Project Name" と "/path/to/new/project" はそれぞれプロジェクト名とパスのプレースホルダーであることに注意してください。
   ```json
   {
     "name": "python-envs.createNewProjectFromTemplate",
     "commandId": "python-envs.createNewProjectFromTemplate",
     "args": ["python-script", "true", "New Project Name", "/path/to/new/project"]
   }
   ```

## python-package

配布可能なパッケージを作成するために使用できる、Python パッケージプロジェクト。

必須の拡張機能: `ms-python.python`, `ms-python.vscode-python-envs`

### ルール

1. `run_vscode_command` ツールを呼び出して、VS Code で新しい Python パッケージプロジェクトを正しく作成します。次の引数を使用してコマンドを呼び出します。
   "python-package" と "true" は定数であり、"New Package Name" と "/path/to/new/project" はそれぞれパッケージ名とパスのプレースホルダーであることに注意してください。
   ```json
   {
     "name": "python-envs.createNewProjectFromTemplate",
     "commandId": "python-envs.createNewProjectFromTemplate",
     "args": ["python-package", "true", "New Package Name", "/path/to/new/project"]
   }
   ```

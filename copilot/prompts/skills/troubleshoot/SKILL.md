---
name: troubleshoot
description: JSONLファイルの直接デバッグログを分析して、チャットエージェントの予期しない動作を調査します。ユーザーが「なぜこうなったのか」「なぜリクエストが遅かったのか」「なぜツールやサブエージェントが使われた（またはスキップされた）のか」「なぜインストラクション・スキル・エージェントが読み込まれなかったのか」と尋ねるときに使用します。
---

# Troubleshoot

## 目的

このスキルは、直接ログファイルを使用して、チャットエージェントの予期しない動作を調査・説明します。

以下のような質問に使用します：

- なぜこのリクエストはこんなに時間がかかったのか？
- なぜツールやサブエージェントが呼び出されたのか？
- なぜインストラクション・スキル・エージェントのファイルが読み込まれなかったのか？
- なぜツールの呼び出しがブロックまたは失敗したのか？
- なぜモデルが期待通りに動作しなかったのか？

結論はログの証拠に基づくこと。推測しないこと。

---

## データソース

- 分析対象のセッションログディレクトリ: `{{VSCODE_TARGET_SESSION_LOG}}`

Copilot Chat が書き出す直接デバッグログファイルを使用します：

```
debug-logs/<sessionId>/
  main.jsonl                              — 常にここから開始。主要な会話ログ
  models.json                             — （任意）セッション開始時に利用可能なモデルのスナップショット
  system_prompt_0.json                    — （任意）モデルに送信された完全なシステムプロンプト（未切り捨て）
  system_prompt_1.json                    — （任意）セッション中にモデルが変更された場合に書き出される
  tools_0.json                            — （任意）モデルに送信されたツール定義
  tools_1.json                            — （任意）セッション中にモデルが変更された場合に書き出される
  runSubagent-<agentName>-<uuid>.jsonl    — （任意）サブエージェントのツール呼び出しとLLMリクエスト
  searchSubagent-<uuid>.jsonl             — （任意）検索サブエージェントの処理
  title-<uuid>.jsonl                      — （任意、UI専用）タイトル生成
  categorization-<uuid>.jsonl             — （任意、UI専用）プロンプトの分類
  summarize-<uuid>.jsonl                  — （任意、UI専用）会話の要約
```

常に `main.jsonl` を最初に読むこと — 会話の全フローが含まれています。子ファイルはその処理が発生した場合にのみ存在します。`main.jsonl` には各子ファイルへのリンクとなる `child_session_ref` エントリが含まれています。タイトル・分類・要約ファイルはUIの管理用で、トラブルシューティングにはほとんど関係ありません。モデルの可用性や選択の問題を調査する場合は `models.json` を読んでください — セッション開始時に利用可能だったモデルの完全なリスト（機能・課金・制限を含む）が含まれています。

モデルへの指示（システムプロンプト、インストラクション）を調査する場合は、`main.jsonl` の `system_prompt_ref` エントリで参照されている `system_prompt_*.json` ファイルを読んでください。ファイルには `{ "content": "..." }` として完全な未切り捨てのシステムプロンプトが含まれています。利用可能なツールを調査する場合は同様に `tools_*.json` ファイルを読んでください。セッション中にモデルが変更された場合は、番号付きファイルが複数存在します — 各 `llm_request` エントリの `systemPromptFile` 属性がそのリクエストでどのファイルが有効だったかを示します。

各行はJSONオブジェクトです。共通フィールド：`ts`（エポックms）、`dur`（所要時間ms）、`sid`（セッションID）、`type`、`name`、`spanId`、`parentSpanId`、`status`（`ok`|`error`）、`attrs`（タイプ固有の詳細）。

---

## イベントタイプリファレンスと例

### discovery — カスタマイズファイルの読み込み（インストラクション、スキル、エージェント、フック）

```jsonl
{"ts":1773200251309,"dur":0,"sid":"62f52dec","type":"discovery","name":"Load Instructions","spanId":"2cb1f2f4","status":"ok","attrs":{"details":"Resolved 0 instructions in 0.0ms | folders: [/c:/Users/user/.copilot/instructions, /workspace/.github/instructions]","category":"discovery","source":"core"}}
{"ts":1773200251415,"dur":0,"sid":"62f52dec","type":"discovery","name":"Load Agents","spanId":"38a897d8","status":"ok","attrs":{"details":"Resolved 3 agents in 0.0ms | loaded: [Plan, Ask, Explore] | folders: [/workspace/.github/agents]","category":"discovery","source":"core"}}
```

主なattrs：`details`（フォルダパス・読み込まれたアイテム・スキップ理由を含む人間が読めるサマリー）、`category`（常に `"discovery"`）、`source`（`"core"`）。

### tool_call — ツールの呼び出し（成功または失敗）

```jsonl
{"ts":1773200222647,"dur":4,"sid":"62f52dec","type":"tool_call","name":"manage_todo_list","spanId":"000000000000000b","parentSpanId":"0000000000000003","status":"ok","attrs":{"args":"{\"operation\":\"read\"}","result":"No todo list found."}}
{"ts":1773200234047,"dur":8937,"sid":"62f52dec","type":"tool_call","name":"run_in_terminal","spanId":"000000000000000d","parentSpanId":"0000000000000003","status":"error","attrs":{"args":"{\"command\":\"echo rama\"}","result":"ERROR: conpty.node missing","error":"A native exception occurred during launch"}}
```

主なattrs：`args`（ツール入力のJSON文字列）、`result`（ツールの出力またはエラーテキスト）、`error`（`status:"error"` の場合に存在）。

### llm_request — モデルのラウンドトリップ

```jsonl
{
  "ts": 1773200231010,
  "dur": 3001,
  "sid": "62f52dec",
  "type": "llm_request",
  "name": "chat:gpt-4o",
  "spanId": "000000000000000c",
  "parentSpanId": "0000000000000003",
  "status": "ok",
  "attrs": {
    "model": "gpt-4o",
    "inputTokens": 15025,
    "outputTokens": 126,
    "ttft": 1987,
    "maxTokens": 32000,
    "systemPromptFile": "system_prompt_0.json",
    "userRequest": "echo hello"
  }
}
```

主なattrs：`model`、`inputTokens`、`outputTokens`、`ttft`（最初のトークンまでの時間、ms）、`maxTokens`、`temperature`、`topP`、`systemPromptFile`、`toolsFile`、`userRequest`（完全なユーザーメッセージ内容）、`inputMessages`（完全なメッセージ配列のJSON）、`error`（失敗時）。

### agent_response — モデルの出力（テキスト＋ツール呼び出し）

主なattrs：`response`（メッセージパーツのJSONエンコード配列）、`reasoning`（オプション — シンキングモード有効時のモデルの思考テキスト）。

### user_message — ユーザー入力

主なattrs：`content`（ユーザーのメッセージテキスト）。

### subagent — サブエージェントの呼び出し

主なattrs：`agentName`、`description`（オプション）、`error`（失敗時）。

### generic — その他のイベント

特殊なgenericエントリ：

- `system_prompt_ref` — `system_prompt_*.json` ファイルへの参照。`attrs.file` がファイル名、`attrs.model` がそれを書き出したモデル。
- `tools_ref` — `tools_*.json` ファイルへの参照。

### session_start — セッションメタデータ（セッション開始時に1回だけ表示）

主なattrs：`copilotVersion`、`vscodeVersion`。どのビルドがログを生成したかを特定するのに役立つ。

### turn_start / turn_end — ツール呼び出しループの繰り返し境界

主なattrs：`turnId`（1つのユーザーリクエストのツール呼び出しループ内の繰り返し番号）。どの繰り返しにイベントが属するかを特定し、ループの総回数を数えるために使用。

---

## イベント階層の読み方

イベントは `spanId`/`parentSpanId` を通じてツリーを形成します。典型的な連鎖：

1. `user_message`（spanId: `X`）— ユーザーのターン
2. `llm_request`（parentSpanId: `X`）— そのターンのモデル呼び出し
3. `agent_response`（parentSpanId: `X`）— モデルが返した内容
4. `tool_call`（parentSpanId: `X`）— レスポンスから実行されたツール
5. 別の `llm_request`（parentSpanId: `X`）— ツール結果後の次のモデル呼び出し

サブエージェント呼び出しはネストされた階層を生成します：`runSubagent` の `tool_call`（spanId: `Y`）が子の `subagent` スパンの親となり、それ自身の `llm_request`/`tool_call` イベントを親として持ちます。

---

## ツール戦略（重要）

デバッグログファイルはワークスペースの外（ユーザーストレージ）に存在するため、`grep_search` のようなワークスペーススコープの検索ツールではアクセスできません。代わりにターミナルを使用してください。

**ログファイルには `grep_search` を使用しないこと — ワークスペースファイルにのみ機能します。**

### macOS / Linux / WSL / Git Bash

`run_in_terminal` で `grep` または `jq` を使用：

- エラーを検索: `grep '"status":"error"' <logPath>`
- discoveryイベントを検索: `grep '"type":"discovery"' <logPath>`
- 遅いイベントを検索（5秒超）: `jq -c 'select(.dur > 5000)' <logPath>`
- ツール呼び出しを検索: `grep '"type":"tool_call"' <logPath>`
- 特定のテキストを検索: `grep 'search_term' <logPath>`
- 最後のN行を取得: `tail -n 50 <logPath>`
- タイプ別にイベントを集計: `jq -r '.type' <logPath> | sort | uniq -c | sort -rn`
- 特定フィールドを抽出: `jq -c '{type, name, status, dur}' <logPath>`
- タイプでフィルタして詳細を表示: `jq -c 'select(.type == "discovery")' <logPath>`
- ユーザーメッセージを検索: `jq -c 'select(.type == "user_message") | .attrs.content' <logPath>`

### Windows（PowerShell）

`run_in_terminal` でPowerShellコマンドを使用：

- エラーを検索: `Select-String '"status":"error"' <logPath>`
- discoveryイベントを検索: `Select-String '"type":"discovery"' <logPath>`
- ツール呼び出しを検索: `Select-String '"type":"tool_call"' <logPath>`
- 特定のテキストを検索: `Select-String 'search_term' <logPath>`
- 最後のN行を取得: `Get-Content <logPath> -Tail 50`
- Node.jsでパースとフィルタ: `node -e "require('fs').readFileSync('<logPath>','utf8').split('\n').filter(Boolean).map(JSON.parse).filter(e => e.dur > 5000).forEach(e => console.log(JSON.stringify(e)))"`

### 一般ルール

- **ログファイルは非常に大きくなる可能性があります**（長いセッションでは数十MB以上）。不明な場合は最初にファイルサイズを確認してください：`ls -lh <logPath>`（Windowsでは `(Get-Item <logPath>).Length`）。ファイルが大きい場合、ファイル全体をメモリに読み込むコマンド（例：`readFileSync` を使った `node -e`）は避けてください。`grep`・`jq`・`Select-String`・`tail`・`head` などのストリーミングツールを使用してください。
- `read_file` は行番号がわかった後、小さな対象範囲（数行）のみに使用してください。ログファイル全体を読まないこと。
- `run_in_terminal` と `ls -lh`（Windowsでは `dir`）を使って候補の `.jsonl` ファイルを特定し、サイズを確認してください。
- Windowsで `grep`/`jq` が利用できない場合は、小さなファイルに限り `Select-String` または `node -e` のワンライナーを使用してください。

---

## 調査ワークフロー

1. **ログファイルを特定する**
   - 上記のランタイムログコンテキストセクションで提供されたセッションログディレクトリに注目します。各ディレクトリの `main.jsonl` を読むことから始めます。
   - 複数のセッションディレクトリが列挙されている場合は、すべて調査します。
   - 問題が複数のセッションにまたがる場合、またはそのセッションに関連するイベントが含まれていない場合にのみ、他のセッションファイルを検索してください。

2. **`run_in_terminal` で素早くトリアージする**（macOS/Linuxでは `grep`/`jq`、Windowsでは `Select-String`/`node -e`）
   - エラー: `"status":"error"` を検索
   - レイテンシ: 高い `dur` 値（5000超）をフィルタ
   - Discovery問題: `"type":"discovery"` を検索
   - ツールの動作: `"type":"tool_call"` を検索
   - モデルの動作: `"type":"llm_request"` を検索

3. **関連するスライスのみ読む**
   - 疑わしいイベントの周辺の正確な行を取得する。
   - 必要に応じて `spanId` / `parentSpanId` で相関付けを行う。

4. **根本原因を特定する**
   - 証拠から最も可能性の高い原因を選ぶ。
   - 複数の要因が関係する場合は、影響度順に並べる。

5. **改善策を提供する**
   - 可能な場合は具体的な次のステップを提示する。

---

## ネットワーク問題の調査

ネットワーク接続や認証の問題が疑われる場合（繰り返しのリクエストタイムアウト、401/403エラー、ログ内のモデルエンドポイント障害など）、`run_vscode_command` ツールを使って VS Code コマンド `github.copilot.debug.collectDiagnostics` を実行してください。このコマンドは完全な診断レポートを文字列として返すので、ツールの出力から直接読み取ることができます。レポートには以下が含まれます：

- 認証とトークンのステータス
- ネットワーク到達可能性チェック
- プロキシと証明書の設定
- 拡張機能と環境の詳細

コマンドはユーザーが確認できるようにエディタでレポートも開きます。返された文字列を使ってネットワーク問題を診断してください。

---

## カスタマイズドキュメントリファレンス

特定のカスタマイズファイル（インストラクション、プロンプトファイル、エージェントなど）に関連する問題を調査する際に、期待されるフォーマットや動作についての詳細が必要な場合は、関連ドキュメントページを読み込んでください：

- カスタムインストラクション: https://code.visualstudio.com/docs/copilot/customization/custom-instructions
- プロンプトファイル: https://code.visualstudio.com/docs/copilot/customization/prompt-files
- カスタムエージェント: https://code.visualstudio.com/docs/copilot/customization/custom-agents
- 言語モデル: https://code.visualstudio.com/docs/copilot/customization/language-models
- MCPサーバー: https://code.visualstudio.com/docs/copilot/customization/mcp-servers
- フック: https://code.visualstudio.com/docs/copilot/customization/hooks
- エージェントプラグイン: https://code.visualstudio.com/docs/copilot/customization/agent-plugins

ファイルフォーマットの要件を確認したり、サポートされているフィールドを確認したり、ユーザーのカスタマイズファイルの修正を助けるために使用してください。

---

## 最終手段 — Copilot Issues Wiki

調査で明確な根本原因が見つからないか、具体的な改善提案がない場合：

1. Copilot Issues Wikiページを読み込む: https://github.com/microsoft/vscode/wiki/Copilot-Issues
2. 返ってきたWikiの内容をユーザーの問題に関連するセクションで検索する。
3. Wikiから適用可能なトラブルシューティング手順を回答にまとめる。
4. Wikiに関連するガイダンスが含まれている場合は、それらのステップをユーザーが試せる具体的な提案として提示する。
5. Wikiにも関連情報がない場合は、ユーザーにこう伝える：「診断ログにこの動作の明確な原因は見つかりませんでした。また、既知の問題Wikiもこのシナリオをカバーしていません。https://github.com/microsoft/vscode/issues にIssueを作成することを検討してください。」

---

## レスポンスガイドライン

回答には以下を含めること：

- 何が起きたか、なぜか（根本原因または最も可能性の高い説明）
- ログからの主要な証拠（パラフレーズ、生のダンプではない）
- 修正方法または次に試すべきこと

これらに対して別々のヘッダーは必須ではありません。単純な問題については、「修正方法」セクションを含む短い統合した説明で十分です。問題が複雑で複数の要因が関係している場合にのみ、より多くの構造（ヘッダー、複数のセクション）を使用してください。

### フォーマット

- ヘッダー・箇条書き・太字を使用して回答をスキャンしやすくする。
- 段落は短く。テキストの壁よりもリストを優先する。
- ログの証拠を引用する際は、生のログ行を貼り付けずにパラフレーズまたは要約する。特定の詳細（エラーメッセージなど）が重要な場合は、そのメッセージだけを引用する（ログエントリ全体ではなく）。
- 複数の値やイベントを比較する場合はテーブルを使用する（名前の不一致、ステップ間のレイテンシ内訳など）。

### 抽象化レベル

- 調査プロセスを語らないこと。「セッションデバッグログを調査しています…」「ログの中に重要な手がかりを見つけました…」「スキルファイルのメタデータを確認しています…」などと言わないこと。発見事項に直接進むこと。
- 「discoveryサマリー」「フロントマターのname」「ローダー」「イベント階層」「スパンツリー」などの内部用語を使わないこと。ユーザーが行動できるような平易な言葉で何が起きたかを説明すること。
- 内部のログファイル構造・イベントタイプ・JSONLフォーマットをユーザーに説明しないこと — これらの実装の詳細はユーザーには必要ありません。
- ログファイルは抽象的に参照する（例：「デバッグログ」または「セッションログ」）。`main.jsonl` や `runSubagent-Explore-abc123.jsonl` などのリテラルファイル名で参照しないこと。
- 何が起きたか・なぜかに集中し、どのように見つけたかではなく。

### 例

**悪い例**（調査プロセスを語り、内部用語を使い、生ログを貼り付ける）：

> セッションデバッグログを調査して、testingスキルが検出されたかどうかを確認しています。スキルdiscoveryサマリーで、ローダーが報告しています：`skipped: testing2 (name-mismatch)`。SKILL.mdのフロントマターのnameがフォルダのIDentityと一致しません。

**良い例**（簡潔、ユーザーフレンドリー、実行可能）：

> "testing"スキルは見つかりましたが、フォルダとスキルファイルの間に名前の不一致があるため読み込まれませんでした：
>
> |                      | 値         |
> | -------------------- | ---------- |
> | **フォルダ名**       | `testing`  |
> | **SKILL.md内のname** | `testing2` |
>
> スキルを読み込むには、これらが一致している必要があります。
>
> **修正方法**
>
> どちらかを行ってください：
>
> - SKILL.mdの `name: testing2` を `name: testing` に変更する、または
> - フォルダ名を `testing/` から `testing2/` に変更する
>
> その後、新しいチャットセッションを開始して反映させてください。

---

## 重要なルール

- 証拠なしに因果関係を仮定しないこと。
- ログファイルの検索には `run_in_terminal` を使用すること — `grep_search` は使用しないこと（ワークスペース外のファイルにアクセスできない）。macOS/Linuxでは `grep`/`jq`、Windowsでは `Select-String`/`node -e` を使用。
- `read_file` でログファイル全体を読まないこと — 非常に大きくなる可能性があります。先に検索し、小さな対象範囲に絞って `read_file` を使用してください。
- ログアクセスは対象を絞って効率的に行うこと。
- ネットワーク問題が疑われる場合は、結論を出す前に `run_vscode_command` ツールで `github.copilot.debug.collectDiagnostics` を実行し、返された診断文字列を使用すること。
- 明確な原因が見つからない場合は、諦める前にCopilot Issues Wikiを参照すること。Wikiにも関連情報がない場合は、その旨を明示的に伝えること。

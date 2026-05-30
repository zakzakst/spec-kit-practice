# [Hooks (.json)](https://code.visualstudio.com/docs/copilot/customization/hooks)

エージェントのライフサイクルイベントに対する検証、コンテキストの注入、または自動化。

## イベントとロケーション

| フック | トリガー | 主な用途 | ロケーション |
|------|---------|-------------|----------|
| `PreToolUse` | ツール実行の直前 | ポリシー検証、ゲートキーピング | `.github/hooks/` |
| `PostToolUse` | ツール実行の直後 | ロギング、後処理 | `.github/hooks/` |
| `PreChatResponse` | モデルからの応答直前 | PII（個人識別情報）のスクラブ、フォーマット | `.github/hooks/` |

## JSON 形式 (例)

```json
{
  "name": "prevent-force-push",
  "description": "git push --force をブロックする",
  "event": "PreToolUse",
  "tools": ["run_command"],
  "handler": {
    "type": "command",
    "command": "node .github/scripts/check-push.js"
  }
}
```

## コンパニオンスクリプト

フックは処理をスクリプト（JS、Python、Bash など）に委任します。スクリプトは標準入力（stdin）から JSON を受け取り、JSON を標準出力（stdout）に出力する必要があります。

```javascript
// check-push.js の例
const input = JSON.parse(fs.readFileSync(0, 'utf8'));

if (input.tool.arguments.CommandLine.includes('git push --force')) {
  console.log(JSON.stringify({
    action: "Block",
    reason: "Force push is prohibited by policy."
  }));
} else {
  console.log(JSON.stringify({ action: "Continue" }));
}
```

## 使用するタイミング

- 破壊的なアクションやポリシー違反のブロック
- 機密情報のフィルタリング
- 特定のツールの使用時におけるリソースからの自動コンテキスト注入
- チーム全体でのワークフローの自動化

## 基本原則

1. **スコープを絞る**: 複数のツールではなく、特定のツール名にターゲットを絞ります
2. **高速に保つ**: フックは同期処理されるため、時間のかかる操作をブロックします。高速に終了するようにします
3. **明確な理由**: ブロックする際は、常に有用なガイダンス付きの明確な理由を提供します
4. **フェイルセーフ**: ハンドラースクリプトが安全に失敗し、実行をブロックしないようにします

## アンチパターン

- **時間のかかるフック**: フック内で時間のかかるネットワークリクエストやビルドを行う
- **広範な適用**: ツール名を指定せずにすべてにマッチさせ、オーバーヘッドを増加させる
- **沈黙の失敗**: エラー時にログを残さずにクラッシュするコンパニオンスクリプト

---
name: strategic-compact
description: 任意の自動 compaction に頼るのではなく、タスクの論理区切りで手動 compact を提案し、フェーズ間で文脈を保ちやすくします。
origin: ECC
---

# Strategic Compact スキル

任意のタイミングで走る auto-compaction に頼らず、ワークフロー上の戦略的な地点で手動 `/compact` を提案します。

## 有効化するタイミング

- 文脈上限（200K+ tokens）に近い長時間セッションを回しているとき
- 複数フェーズのタスクを進めているとき（research -> plan -> implement -> test）
- 同一セッション内で無関係なタスクを切り替えるとき
- 大きな節目を終えて新しい作業に入るとき
- 応答が遅くなったり、まとまりが落ちてきたとき（context pressure）

## なぜ Strategic Compaction か

Auto-compaction は論理境界を見ない:

- タスク途中で発火して重要文脈を失いがち
- 論理的な区切りを理解しない
- 複雑な多段処理を中断しうる

論理的区切りでの strategic compaction は:

- **探索後、実装前** - 研究文脈を畳み、実装計画を残す
- **マイルストーン達成後** - 次フェーズに向けて新鮮な状態にする
- **大きな文脈切替前** - 無関係な前段探索を掃除する

## 仕組み

`suggest-compact.js` は PreToolUse（Edit / Write）で走り、以下を行う:

1. **ツール呼び出し追跡** - セッション内のツール使用回数を数える
2. **しきい値検知** - 設定値（既定: 50 回）を超えると提案
3. **定期リマインド** - 以後 25 回ごとに再通知

## Hook 設定

`~/.claude/settings.json` に次を追加する:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "node ~/.claude/scripts/hooks/suggest-compact.js"
          }
        ]
      },
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "node ~/.claude/scripts/hooks/suggest-compact.js"
          }
        ]
      }
    ]
  }
}
```

## 設定

環境変数:

- `COMPACT_THRESHOLD` - 最初の提案を出すまでのツール呼び出し回数（既定: 50）

## Compact 判断ガイド

この表で compact すべきか決める:

| Phase Transition          | Compact? | Why                                                         |
| ------------------------- | -------- | ----------------------------------------------------------- |
| Research -> Planning      | Yes      | 調査文脈は重い。plan は蒸留済み成果物                     |
| Planning -> Implementation | Yes     | plan は TodoWrite やファイルに残る。コード用に文脈を空ける |
| Implementation -> Testing | Maybe    | 直近コード参照が必要なら保持。焦点切替なら compact        |
| Debugging -> Next feature | Yes      | debug 由来の文脈は次機能にノイズになる                     |
| Mid-implementation        | No       | 変数名、パス、途中状態を失うコストが高い                   |
| After a failed approach   | Yes      | 行き止まりの推論を消して新アプローチへ進む                 |

## Compact 後に残るもの

何が残るかを知ると、安心して compact しやすい:

| Persists                           | Lost                                     |
| ---------------------------------- | ---------------------------------------- |
| `CLAUDE.md` instructions           | 中間推論や分析                           |
| TodoWrite task list                | 読み込んだファイル内容                   |
| Memory files (`~/.claude/memory/`) | 多段の会話文脈                           |
| Git state (commits, branches)      | ツール呼び出し履歴と回数                 |
| Files on disk                      | 口頭で出ていた細かいユーザー嗜好         |

## ベストプラクティス

1. **計画後に compact** - TodoWrite に plan が固まったら一度畳む
2. **デバッグ後に compact** - 問題解決用の文脈を掃除して先へ進む
3. **実装途中では compact しない** - 連続変更の文脈を保つ
4. **提案は読むが、判断は自分でする** - hook は _when_ を示し、_if_ はあなたが決める
5. **compact 前に書き出す** - 重要文脈はファイルや memory に保存してから畳む
6. **要約付き `/compact` を使う** - 例: `/compact Focus on implementing auth middleware next`

## トークン最適化パターン

### Trigger-Table Lazy Loading

セッション開始時に全 skill を載せる代わりに、キーワードから skill パスへ引く trigger table を使う。必要時だけ読み込めば、基礎コンテキストを 50% 以上減らせる:

| Trigger                   | Skill               | Load When             |
| ------------------------- | ------------------- | --------------------- |
| "test", "tdd", "coverage" | tdd-workflow        | testing が出たとき    |
| "security", "auth", "xss" | security-review     | security 作業時       |
| "deploy", "ci/cd"         | deployment-patterns | deployment 文脈時     |

### Context Composition Awareness

何が context window を食っているか意識する:

- **CLAUDE.md files** - 常時ロードされるので lean に保つ
- **Loaded skills** - 1 つで 1K〜5K tokens 増える
- **Conversation history** - やり取りごとに伸びる
- **Tool results** - ファイル読取や検索結果が嵩む

### Duplicate Instruction Detection

重複文脈のよくある発生源:

- `~/.claude/rules/` と project `.claude/rules/` に同じ規則がある
- CLAUDE.md の指示を skill が繰り返している
- 複数 skill が重なる領域を扱っている

### Context Optimization Tools

- `token-optimizer` MCP - コンテンツ重複除去で 95%+ のトークン削減
- `context-mode` - 文脈仮想化（315KB -> 5.4KB 実績）

## 関連

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - token optimization セクション
- Memory persistence hooks - compact 後にも残したい状態向け
- `continuous-learning` skill - セッション終了前にパターン抽出

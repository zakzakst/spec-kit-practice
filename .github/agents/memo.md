<!-- .github\agents\speckit.analyze.agent.md -->

```md
## User Input

\`\`\`text
$ARGUMENTS
\`\`\`
空でない場合、処理前にユーザー入力を**必ず**考慮すること。
```

```md
このコマンドは `/speckit.tasks` により完全な `tasks.md` が正常生成された後にのみ実行すること。
```

```md
いかなるファイルも変更してはならない。構造化された分析レポートを出力すること。必要であれば改善案を提示してよいが、その後に編集系コマンドを使う場合はユーザーの明示的な承認が必要。
```

```md
プロジェクト憲章 (`.specify/memory/constitution.md`) はこの分析の範囲では**交渉不可**。憲章との衝突は自動的に CRITICAL とし、原則を薄めたり再解釈したり黙殺したりせず、spec / plan / tasks 側を調整する必要がある。
```

<!-- NOTE: 「要件内容をチェック」するagentという考え方覚えておく -->
<!-- .github\agents\speckit.checklist.agent.md -->

```md
**要件品質の検証に使うもの**
完全性、明確性、整合性、カバレッジ、境界ケース
```

```md
質問は `Q1` / `Q2` / `Q3` として出力する。回答後も Alternate / Exception / Recovery / Non-Functional のうち 2 種以上が不明な場合のみ、1 行理由付きで最大 2 問の追加質問 (`Q4` / `Q5`) を行ってよい。合計 5 問を超えてはならない。ユーザーが追加質問を断ったら打ち切る。
```

<!-- .github\agents\speckit.clarify.agent.md -->

```md

```

```md

```

```md

```

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
注記: この明確化フローは `/speckit.plan` の前に実行し、完了していることが期待される。ユーザーが明示的に明確化を省略する（例: exploratory spike）場合は進めてよいが、後工程の手戻りリスクが上がることを警告すること。
```

```md
現在の spec ファイルを読み、次の分類で曖昧さ / カバレッジを評価する。各カテゴリの状態は `Clear / Partial / Missing` とする。優先順位付けに使う内部カバレッジマップを作るが、質問がなければその場合のみ出力してよい。
```

```md
順次質問ループ
```

```md
読み込んだ spec とその生内容をインメモリ (in-memory) で保持する
```

```md
完了レポート
```

```md
Outstanding / Deferred があれば、`/speckit.plan` に進むべきか、後で `/speckit.clarify` を再実行すべきか推奨する
```

```md
動作ルール (Behavior rules):

- 重要な曖昧さが見つからなければ `No critical ambiguities detected worth formal clarification. (正式な確認が必要な重大な曖昧さは検出されませんでした。)` と返し、次へ進むことを勧める
- spec がなければ `/speckit.specify` を先に実行するよう案内する
- 新規質問は 5 問を超えない
- 機能的明確さを妨げない限り、技術スタックの推測質問は避ける
- `stop`、`done`、`proceed` などの早期終了意思を尊重する
- 質問不要ならコンパクトな全 Clear サマリを出して先へ促す
- 上限到達時に高影響未解決が残れば、理由付きで 保留 (Deferred) へ明示する
```

<!-- .github\agents\speckit.constitution.agent.md -->

```md

```

```md

```

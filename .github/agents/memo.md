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
更新対象は `.specify/memory/constitution.md` である。このファイルは `[PROJECT_NAME]`、`[PRINCIPLE_1_NAME]` のような角括弧トークンを含むテンプレートである。あなたの役割は、(a) 値を収集 / 推定し、(b) テンプレートを正確に埋め、(c) 変更内容を依存成果物へ波及させること。
```

```md
新しいテンプレートは作らず、必ず既存の `.specify/memory/constitution.md` を更新すること。
```

<!-- .github\agents\speckit.implement.agent.md -->

```md
- 各ファイルについて集計する
  - 総項目数 (Total items): `- [ ]`, `- [X]`, `- [x]`
  - 完了項目数 (Completed items): `- [X]`, `- [x]`
  - 未完了項目数 (Incomplete items): `- [ ]`
```

```md
- 次のようなステータス表を作る

  | チェックリスト (Checklist) | 合計 (Total) | 完了 (Completed) | 未完了 (Incomplete) | ステータス (Status) |
  | -------------------------- | ------------ | ---------------- | ------------------- | ------------------- |
  | ux.md                      | 12           | 12               | 0                   | PASS                |
  | test.md                    | 8            | 5                | 3                   | FAIL                |
  | security.md                | 6            | 6                | 0                   | PASS                |
```

```md
- 未完了が 1 つでもあれば:
  - 表を表示する
  - `Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no) (一部のチェックリストが未完了です。このまま実装を進めますか？(yes/no))` と聞いて**処理を停止**する
  - `no`, `wait`, `stop`, `いいえ` なら中止する
  - `yes`, `proceed`, `continue`, `はい` なら次へ進む
```

```md
6. タスク計画に従って実装する。
   - フェーズ単位で進める
   - 依存関係を守る
   - 同じファイルに触るタスクは逐次実行
   - 各フェーズ完了時に検証する
```

```md
8. 進捗追跡とエラーハンドリング (Progress Tracking & Error Handling):
   - タスク完了ごとに進捗報告
   - 非並列タスクが失敗したら停止
   - 並列タスク `[P]` は成功分を継続し、失敗分を報告
   - デバッグに十分な文脈付きエラーを出す
   - 進められない場合は次の一手を提案
   - **重要**: 完了したタスクは tasks ファイルで `[X]` または `[x]` にする
```

```md
9. 完了時検証 (Final Verification):
   - 必須タスクがすべて完了しているか
   - 実装が元 spec と一致しているか
   - テストとカバレッジが要件を満たすか
   - 技術計画に従っているか
   - 最終ステータスと完了内容を要約する
```

```md
注記: このコマンドは、完全なタスク分解が `tasks.md` に存在することを前提とする。不足している場合は `/speckit.tasks` の再生成を勧めること。
```

<!-- .github\agents\speckit.plan.agent.md -->

````md
2. 調査エージェントを生成 / 発行する。

   > ```text
   > For each unknown in Technical Context:
   >   Task: "Research {unknown} for {feature context}"
   > For each technology choice:
   >   Task: "Find best practices for {tech} in {domain}"
   > ```

3. 結果を `research.md` に統合する。形式は以下の通り:
   - 意思決定 (Decision)
   - 根拠 (Rationale)
   - 検討された代替案 (Alternatives considered)
````

<!-- .github\agents\speckit.specify.agent.md -->

```md
1. **簡潔なショートネームの生成 (Generate a concise short name)**（2〜4 語）
   - 機能説明から意味の強いキーワードを抽出する
   - 本質を捉えた 2〜4 語のショートネーム (short name) を作る
   - 可能なら action-noun（動詞-名詞）形式にする
   - OAuth2、API、JWT などの技術語や略語は保持する
   - 短くても一目で分かる名前にする
   - 例:
     - `I want to add user authentication` -> `user-auth`
     - `Implement OAuth2 integration for the API` -> `oauth2-api-integration`
     - `Create a dashboard for analytics` -> `analytics-dashboard`
     - `Fix payment processing timeout bug` -> `fix-payment-timeout`
```

```md
3. `.specify/templates/spec-template.md` を読み、必須セクションを把握する。
```

```md
a. `FEATURE_DIR/checklists/requirements.md` に 仕様品質チェックリスト (Spec Quality Checklist) を作成する。構造はチェックリストテンプレートに従い、少なくとも次の検証項目を含める:
```

```md
- 未完了としてマークされた項目は、`/speckit.clarify` または `/speckit.plan` に進む前に仕様書の更新が必要です。
```

⇒「未完了としてマーク」というやり方。実務だと各段階できっちりと内容が定まらないことも多いため、その際の回避パターンとして覚えておく。（チェックリストを出力するのとかもよさそうに感じた）

```md
7. branch 名、spec パス、checklist 結果、次フェーズ (`/speckit.clarify` または `/speckit.plan`) へ進める状態かを報告する。
```

```md
### 成功基準のガイドライン (Success Criteria Guidelines)

成功基準 (Success criteria) は次を満たすこと。

3. **ユーザー中心であること (User-focused)**: システム内部ではなくユーザー / ビジネス成果で書く

**良い例 (Good examples)**

- `Users can complete checkout in under 3 minutes`（ユーザーは3分以内にチェックアウトを完了できる）
- `System supports 10,000 concurrent users`（システムは10,000人の同時接続ユーザーをサポートする）

**悪い例 (Bad examples)**

- `API response time is under 200ms`（APIのレスポンスタイムが200ms未満であること）
- `Database can handle 1000 TPS`（データベースが1000 TPSを処理できること）
```

⇒この良い例・悪い例の考え方はAI以外の場面でも利用できそう。覚えておく

<!-- .github\agents\speckit.tasks.agent.md -->

```md

```

```md

```

```md

```

```md

```

```md

```

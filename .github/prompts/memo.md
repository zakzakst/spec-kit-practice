<!-- .github\prompts\opsx-apply.prompt.md -->

```md
**AskUserQuestion tool** で選ばせる
```

⇒AskUserQuestion toolはClaudのツールらしい。GitHub Copilotだと下記なので、試すときはこちらを使ってみる
`#tools: vscode/askQuestions`
https://qiita.com/KC-tsukada/items/37280601a12656a0d277

```md
5. **現在の進捗を表示する**

   表示内容:
   - 使用中 schema
   - Progress: `N/M tasks complete`
   - Remaining tasks の概要
   - CLI からの動的 instruction
```

```md
7. **完了時または一時停止時に状態を表示する**

   表示内容:
   - 今回セッションで完了したタスク
   - Overall progress: `N/M tasks complete`
   - 全完了なら archive を勧める
   - 一時停止なら理由を説明して案内を待つ
```

```md
**柔軟なワークフロー統合**

このプロンプトは「change に対して随時 action する」モデルを支える:

- **いつでも呼べる**: 全成果物完成前でも、実装途中でもよい
- **成果物更新も許容**: 実装で設計課題が見つかったら、phase 固定にせず柔軟に更新を提案する
```

<!-- .github\prompts\opsx-archive.prompt.md -->

```md
**警告付き成功時の出力**

**Warnings:**

- 未完了の成果物 2 件を含めて archive した
- 未完了のタスク 3 件を含めて archive した
- delta spec の同期をスキップした（ユーザーがスキップを選択）
```

```md
警告があっても archive 自体は妨げず、確認のうえ進める
```

<!-- .github\prompts\opsx-bulk-archive.prompt.md -->

```md
**最終サマリを表示する**
```

<!-- .github\prompts\opsx-continue.prompt.md -->

```md
**spec-driven schema** (`proposal -> specs -> design -> tasks`)

- `proposal.md`: 変更内容が不明瞭ならユーザーに確認しつつ、Why / What Changes / Capabilities / Impact を埋める
- `specs/<capability>/spec.md`: proposal の Capabilities に列挙された capability ごとに 1 ファイル作る
- `design.md`: 技術判断、構成、実装方針を書く
- `tasks.md`: 実装作業をチェックボックス付きで分解する
```

<!-- .github\prompts\opsx-explore.prompt.md -->

```md
**重要: explore モードは「考えるためのモード」であり、実装するためのモードではない。** ファイル読取、コード検索、コードベース調査はしてよいが、コードを書いたり機能実装したりしてはならない。ユーザーが実装を求めたら、まず explore を抜けるよう案内すること（例: `/opsx:new` または `/opsx:ff`）。OpenSpec artifacts（proposal、design、specs）の作成は、思考の整理として依頼された場合のみ許可される。
```

```md
**これはワークフローではなく姿勢である。** 固定手順も必須順序も必須出力もない。あなたは探索の相棒である。
```

```md
## The Stance

- **Curious, not prescriptive**: 脚本的に進めず、自然に出てくる問いを使う
- **Open threads, not interrogations**: 面白い方向を複数出し、ユーザーが響いた方向を選べるようにする
- **Visual**: 助けになるなら ASCII 図を積極的に使う
- **Adaptive**: 面白い糸口を追い、新情報が出たら柔軟に方向転換する
- **Patient**: 結論を急がず、問題の輪郭が見えるまで待つ
- **Grounded**: 必要なら実際のコードベースを見て、机上の空論だけにしない
```

```md
4. **決めるのはユーザー**
   - 提案はしてよい
   - 押し付けない
   - 自動で記録しない
```

```md
## やらなくてよいこと

- 毎回同じスクリプトを踏むこと
- 毎回同じ質問をすること
- 特定の artifact を必ず作ること
- 必ず結論へ到達すること
- 価値ある脱線を避けること
- 簡潔にまとめること
```

```md
終わり方は固定ではない。探索は次のように終わってよい。

- **行動へつなげる**: `始めますか？ /opsx:new または /opsx:ff`
- **artifact 更新として残す**: `この判断を design.md に反映しました`
- **明確化だけで終える**: 明確化だけで十分
- **後で続ける**: `いつでも続きを扱えます`
```

<!-- .github\prompts\opsx-ff.prompt.md -->

```md
- 致命的に文脈不足なら質問するが、勢いを止めない範囲では合理的に判断する
```

<!-- .github\prompts\opsx-new.prompt.md -->

```md
1. **入力がなければ、何を作りたいか聞く**

   **AskUserQuestion tool** を使って次を聞く:

   > 「どの change に取り組みたいですか？ 作りたいもの、または直したいものを説明してください。」
```

```md
- 同名 change が既にある場合は `/opsx:continue` を勧める
```

<!-- .github\prompts\opsx-onboard.prompt.md -->

```md
ユーザーが OpenSpec の一連の workflow を 1 周体験できるよう案内する。これは**教えながら実際に手を動かす**体験であり、コードベースに対して本物の作業をしながら各ステップを説明する。
```

````md
コードベースを走査し、小さく着手しやすい改善候補を探す。見る観点:

1. **TODO / FIXME コメント**: `TODO`, `FIXME`, `HACK`, `XXX`
2. **エラーハンドリング不足**: 飲み込む `catch`、危険処理に try-catch がない
3. **テストのない関数**: `src/` と test ディレクトリの対応
4. **型の問題**: TypeScript の `any`
5. **デバッグ残骸**: `console.log`, `console.debug`, `debugger`
6. **バリデーション不足**: 入力ハンドラに検証がない

直近の git 履歴も確認する:

> ```bash
> git log --oneline -10 2>/dev/null || echo "No git history"
> ```
````

```md
分析結果から 3〜4 件の具体候補を提示する

**見つからない場合**

> コードベースには、すぐに着手できる明確な改善点は見つかりませんでした。ずっと追加したい、または直したいと思っていた小さなものはありますか？
```

````md
ユーザーが大きすぎるタスクを選んだ場合:

> ```text
> それは価値のあるタスクですが、OpenSpec の最初の流れとしては少し大きすぎるかもしれません。
>
> ワークフローを学ぶには、小さい方が向いています。実装の細部で詰まりにくく、サイクル全体を見やすくなります。
>
> **選択肢:**
> 1. **もっと小さく切る** - [そのタスク] の中で、最も小さくて意味のある部分はどこでしょう？ たとえば [具体的な切り出し] だけに絞れますか？
> 2. **別のものを選ぶ** - ほかの候補、または別の小さなタスクにしますか？
> 3. **そのまま進める** - 本当にこれに取り組みたいなら、そのまま進めることもできます。ただし少し時間がかかります。
>
> どれにしますか？
> ```

強制はしない。やわらかいガードレールとして扱う。
````

```md
OpenSpec における "change" は、ある作業に関する思考や計画をまとめて入れておく入れ物です。`openspec/changes/<name>/` に置かれ、proposal、specs、design、tasks などの成果物をまとめます。

proposal では、この change をなぜ行うのか、そして大まかに何を含むのかをまとめます。作業内容を短く伝えるための要約です。

specs では、**何を作るのか** を、正確でテスト可能な形で定義します。requirement / scenario 形式を使うことで、期待される挙動を明確にできます。

design では、どうやって作るか、つまり技術的な判断、トレードオフ、進め方をまとめます。

最後に、作業を実装タスクへ分解します。これは apply フェーズを進めるためのチェックボックスです。
```

⇒この情報の分け方覚えておく

```md
change が完了したら archive します。これにより `openspec/changes/` から `openspec/changes/archive/YYYY-MM-DD-<name>/` へ移動します。

archive された change は、プロジェクトの意思決定の履歴になります。
```

````md
途中で止めたい場合:

> ```text
> 問題ありません。change は `openspec/changes/<name>/` に保存されています。
>
> あとで続けるには:
> - `/opsx:continue <name>` - 成果物作成を再開する
> - `/opsx:apply <name>` - 実装に進む（tasks がある場合）
>
> 作業は失われません。準備ができたらまた戻ってきてください。
> ```
````

<!-- .github\prompts\opsx-sync.prompt.md -->

⇒ 特にメモする内容はなかった

<!-- .github\prompts\opsx-verify.prompt.md -->

```md
次の 3 軸でレポートを作る:

- **完全性**
- **正確性**
- **整合性**
```

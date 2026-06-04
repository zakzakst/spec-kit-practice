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

```

```md

```

```md

```

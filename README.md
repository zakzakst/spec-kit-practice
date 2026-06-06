## TODO

- 下記の順番でエージェントファイルを作成するのを試してみる
  1. 作業を分解する
  2. 各作業で利用するプロンプトを作成する
  3. 作成したプロンプトをつなげるようにファイルを作成・修正する
- そのためにまず勉強すること
  - プロンプトをつなげるのに、どのような方法・パターンがあるのか
  - AI関連のファイルを分割する理由を改めて整理（確かコンテキストを絞ったほうが適切な返答が得られるからだったと思うが、改めてきちんと調べる）

- コードや文章生成とは異なるが、ドキュメントとかで内容を調査をAIで行ったとき、適切な画像とか図と紐づける書き方とかファイル管理の方法があるか調べる

- 生成した成果物をレビューするための、指示・記述についても、いいものあればメモしておく

- https://zenn.dev/tokium_dev/articles/843968b5474998
  - スキルファーストで業務スキルを作り、自己進化スキル(評価・振り返り・改善)でループを回す。

- 定義する情報の区切り方考える
  - https://zenn.dev/rehabforjapan/articles/requirements-engineering
  - ⇒これのとおりに分けるのがベストかというと、そうは思わないが、情報を決めるたり整理したりする際の、各フェーズの粒度は自分の中でクリアにしておきたい

## vercel agent skills

- https://github.com/vercel-labs/agent-skills

## spec-kit-practice

- https://developers.openai.com/blog/designing-delightful-frontends-with-gpt-5-4
- https://dev.classmethod.jp/articles/spec-driven-development-with-github-spec-kit/
- https://zenn.dev/rakuten_tech/articles/try-github-spec-kit

## UI Skills

- https://zenn.dev/redamoon/articles/article39-ui-skills-accessibility
- https://www.ui-skills.com/

## オーケストレーション参考

- https://learn.microsoft.com/ja-jp/azure/architecture/ai-ml/guide/ai-agent-design-patterns
- https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system?hl=ja

## OpenSpec

- https://github.com/Fission-AI/OpenSpec/tree/main
- https://note.com/brave_quince241/n/n6d4bb3ee0a12
- https://iret.media/183621

## awesome-copilot写経

- 次ここから
  - https://github.com/github/awesome-copilot/tree/main/agents

## 調べる

- model: https://github.com/github/awesome-copilot/blob/main/.github/copilot-instructions.md?plain=1#L17
- applyTo: https://github.com/github/awesome-copilot/blob/main/.github/copilot-instructions.md?plain=1#L28
  - ## 多分カスタムインストラクションに対して「このファイルには適用する」ような指定を記述する
- collection: https://github.com/github/awesome-copilot/blob/main/.github/copilot-instructions.md?plain=1#L58
- tasks.json: https://github.com/github/awesome-copilot/blob/main/.vscode/tasks.json
- cookbook: https://github.com/github/awesome-copilot/blob/main/.schemas/cookbook.schema.json
- fetch_webpage ツール
- ツール全般調べる
  - https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_chat-tools
- <reasoning> セクション
- Google デベロッパー ドキュメントの翻訳のベスト プラクティス
  - https://developers.google.com/style/translation
  - https://developers.google.com/style/inclusive-documentation
- よさそうな記事
  - https://zenn.dev/microsoft/articles/91e88d7914e5bc
  - https://zenn.dev/openjny/articles/e11450f61d067f
  - https://zenn.dev/microsoft/articles/a9d4f6ec2a05c2
- safe-area-inset
- prefers-reduced-motion
- tabular-nums
- アンサンブル推論
- クォーラム
- https://developer.mozilla.org/ja/docs/Web/CSS/Reference/Selectors/:focus-within
- font-display: swap

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

- AIにプロンプトファイルの構成と具体的なサンプルファイル作成してもらう？

## https://x.com/so_ainsight/status/2089187978430251309
1. prime-agent
長時間の作業を任せても、途中で学んだコツを自分の設定に書き足しながら進めるコーディング・リサーチ用のエージェント。ターミナルを閉じてもバックグラウンドで動き続け、あとから再接続できる。
https://github.com/PrimeIntellect-ai/prime-agent

2. pi
OpenAI・Anthropic・GoogleなどのAIモデルAPIを1つの窓口にまとめ、ターミナル型のコーディングエージェントも使えるツールキット。アクセス範囲を絞る仕組みは備わらず、素のままだと起動した環境と同じ権限で動く点に注意。
https://github.com/earendil-works/pi

3. TencentDB-Agent-Memory
会話やドキュメント、コードをチームの共有記憶として蓄積し、別のAIエージェントにそのまま引き継げる。新しく立ち上げたAIも、過去のやり取りを踏まえた状態から動き出せる。
https://github.com/TencentCloud/TencentDB-Agent-Memory

4. agent-skills
仕様定義からコードレビュー、リリースまでの工程を8つのコマンドに落とし込んだスキル集。GoogleでChromeとAIの開発者体験を率いたAddy Osmani氏が、実務の品質チェックをパッケージ化した。
https://github.com/addyosmani/agent-skills

5. cloudflare/computer
AIエージェント専用の仮想パソコンをクラウド側に用意できる。ファイル操作からコマンド実行までその中で完結するが、現時点はプレビュー版で本番投入には向かない。
https://github.com/cloudflare/computer

6. corsair
メール送信やチャット投稿など、外部サービスとのやり取りをエージェントに安全に任せられる統合レイヤー。認証情報は渡さず、送信のような重要な操作には人の承認を挟むよう設定できる。
https://github.com/corsairdev/corsair

7. daily_stock_analysis
AIが相場ニュースとチャートを分析し、売買判断の材料を毎日ダッシュボードにまとめる。中国株・香港株・米国株など複数市場に対応し、チャットツールやメールへの自動通知も設定できる。
https://github.com/ZhuLinsen/daily_stock_analysis

8. paperclip
複数のAIエージェントに役割を割り振り、まるで会社のように事業運営を任せられる管理アプリ。目標設定も予算も進捗確認も、画面1つで完結する。
https://github.com/paperclipai/paperclip

9. unsloth
手元のパソコンだけでAIモデルを動かしたり、自分のデータで追加学習させたりできるデスクトップアプリ。文章にとどまらず、画像や動画を作るモデルにも対応する。
https://github.com/unslothai/unsloth

10. code-graph-rag
巨大なコードベースをまるごと読み込ませて、ふつうの言葉で「ここはどういう処理か」と質問できる。修正案も出せるが、反映される前に変更箇所を確認できる。
https://github.com/vitali87/code-graph-rag

## claude-code-best-practice

- https://github.com/shanraisshan/claude-code-best-practice

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

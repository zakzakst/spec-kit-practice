### 参考

- https://code.visualstudio.com/docs/agent-customization/prompt-files
- https://code.visualstudio.com/docs/agent-customization/custom-agents

### 設定

- chat.promptFilesLocations
- chat.agentFilesLocations

## 作りたい

> 現在下記プロンプトファイルを作成中です
> 自分用\prompts\xxxxx.prompt.md
> 作成できていない箇所を修正して、ファイルを完成させてください。

- エージェントファイルのtoolsについて調べる
  - https://docs.github.com/ja/copilot/reference/custom-agents-configuration#tools
  - MCPサーバーの「利用可能なツール」を調べる方法が分からなかった（下記以降追えなかった）
    - https://docs.github.com/ja/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers#writing-a-json-configuration-for-mcp-servers
  - 一応、下記の手順で見れそうではあった
    - チャットを開く
    - 会話入力のテキストボックスの「ツールの構成」アイコンをクリック
  - 一旦下記のgithub copilot組み込みのエージェントファイルで利用されているツールを理解する
    - copilot\plan-agent\Plan.agent.md
    - copilot\explore-agent\Explore.agent.md
    - copilot\ask-agent\Ask.agent.md
- やりたいことから、それに近いプロンプトファイルを探すプロンプト（「このリポジトリ内に「コードリーディングを補助してくれるプロンプトファイル」があるか探してもらえますでしょうか。」）
- コードの説明（ファイルを指定してやっている処理を説明）
- テストコード作成
  - テスト観点
  - テストケース
  - テストコード
- 要件定義
  - .specify\templates\spec-template.md
  - awesome-copilot\agents\prd.agent.md
  - awesome-copilot\prompts\breakdown-feature-prd.prompt.md
- 実装計画
  - .github\agents\speckit.plan.agent.md
  - .specify\templates\plan-template.md
  - awesome-copilot\agents\plan.agent.md
  - awesome-copilot\agents\planner.agent.md
  - awesome-copilot\prompts\create-implementation-plan.prompt.md
  - copilot\plan-agent\Plan.agent.md
  - .github\agents\speckit.clarify.agent.md
  - awesome-copilot\agents\task-planner.agent.md
- デバッグ
  - awesome-copilot\agents\debug.agent.md
- ドキュメント生成
  - https://zenn.dev/417/articles/ai-era-domain-knowledge-placement
    - ドキュメントの自動生成が難しいなら設計が悪い
- 今回の会話をpromptなどに反映
- issueから実装までやるprompt
  - https://zenn.dev/explaza/articles/d0aeb08fcd1888
  - https://github.com/unsu0707/interview-dev-loop/tree/main
- instructions
  - awesome-copilot\instructions\instructions.instructions.md
  - 下記を意識してみる
  - https://zenn.dev/tokium_dev/articles/claude-knowledge-organization
    - 常時メモ（索引）：MemGPT の working memory（LLMが常時手元に置く少量の作業記憶）
      - ⇒ 解いている問題：常時読み込みが膨らむ
    - 必要時メモ：Zettelkasten（1メモ1概念で小さく分け、必要なものだけ参照する手法）
      - ⇒ 解いている問題：事例が積み上がる
    - 構造化メモ：Cline Memory Bank（AIが起動時に必ず読む固定ファイル群）
      - ⇒ 解いている問題：起動時に読む量が膨らむ
    - 倉庫：PARA の Archives（いま使わないものを隔離する置き場）
      - ⇒ 解いている問題：休眠知識を隔離する
- brand.md
  - https://newspicks.com/news/16853096/body/
- CodeGraph
  - https://zenn.dev/aoki_monpro/articles/6ea28ed4c73aa2
- 完了条件を明確にする指示（どこかで使えそうなのでメモ）
  - https://zenn.dev/sun_asterisk/articles/e144769108a880
- プロンプトをリファクタリングするプロンプト
  - 本当に必要な複雑さなのか
  - https://zenn.dev/shintaroamaike/articles/67c21b040af42f
- 不要なコードを削除する
- 処理をビジュアル化する
- 自分や周りの人がどういう価値観かを知る手がかりをためるメモの作成手伝い（obsidian連携想定、まとまったらAIで全体を踏まえて整理想定）
- コードベースの改善のループを回すプロンプト群
- サイトの改善のループを回すプロンプト群

## 作りたいGem
- 「こんな感じにしたい」サイトのURLのリストを渡して、クライアントが希望するデザイントーンを言語化する
- サービスの実現後のイメージ画像を作成する
  - ニュースやSNSで取り上げられている
  - 利用シーンイメージ

## 済
- プロンプトのテンプレートを作りたい。現状フォーマットがバラバラ
  - 下記参考にしてみる
    - https://speakerdeck.com/hirotomotaguchi/copilot-for-microsoft-365-yuzaxiang-keyan-xiu-zi-liao?slide=32
    - 目的
    - コンテクスト
    - ソース
    - 期待
- Gemのテンプレートを作りたい。現状フォーマットがバラバラ
- リファクタリング
  - awesome-copilot\agents\implementation-plan.agent.md
- 以前作成したものも利用
  - https://github.com/zakzakst/next-practice6/tree/main/.github
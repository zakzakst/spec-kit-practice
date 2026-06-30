### 参考

- https://code.visualstudio.com/docs/agent-customization/prompt-files
- https://code.visualstudio.com/docs/agent-customization/custom-agents

### 設定

- chat.promptFilesLocations
- chat.agentFilesLocations

## 作りたい

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
- リファクタリング
  - awesome-copilot\agents\implementation-plan.agent.md
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

## 作りたいGem
- 「こんな感じにしたい」サイトのURLのリストを渡して、クライアントが希望するデザイントーンを言語化する
- Gemのテンプレートを作りたい。現状フォーマットがバラバラ
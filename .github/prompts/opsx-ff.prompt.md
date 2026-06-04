---
description: change を作成し、実装に必要な全成果物を一気に生成する
---

成果物作成を一気に進め、実装開始に必要なものをすべて生成する。

**Input**: `/opsx:ff` の後には、変更名（kebab-case）または作りたいものの説明が来る。

**Steps**

1. **入力がなければ、何を作りたいか聞く**

   **AskUserQuestion tool** を使い、自由記述で次を聞く:
   > 「どの change に取り組みたいですか？ 作りたいもの、または直したいものを説明してください。」

   その説明から kebab-case 名を導く（例: `add-user-auth`）。

   **IMPORTANT**: ユーザーが何を作りたいか理解できるまで進まないこと。

2. **change ディレクトリを作成する**
   ```bash
   openspec new change "<name>"
   ```
   これで `openspec/changes/<name>/` が作られる。

3. **artifact の作成順を取得する**
   ```bash
   openspec status --change "<name>" --json
   ```
   JSON から次を読む:
   - `applyRequires`
   - `artifacts`

4. **apply-ready になるまで artifact を順番に作る**

   **TodoWrite tool** で進捗を追う。

   依存順にループする。

   a. **`ready` な artifact ごとに**
   - instructions を取得
     ```bash
     openspec instructions <artifact-id> --change "<name>" --json
     ```
   - JSON から `context`, `rules`, `template`, `instruction`, `outputPath`, `dependencies` を読む
   - 完了済み dependency files を読む
   - `template` を構造に使って artifact を作る
   - `context` と `rules` は制約として使い、本文にコピーしない
   - 進捗を短く示す: `Created <artifact-id>`

   b. **すべての `applyRequires` が完了するまで続ける**
   - artifact ごとに `openspec status --change "<name>" --json` を再実行
   - `applyRequires` に含まれる各 artifact が `done` か確認
   - 全部 `done` なら停止

   c. **ユーザー入力が必要な artifact**
   - **AskUserQuestion tool** で明確化
   - その後に作成を続行

5. **最後に状態を表示する**
   ```bash
   openspec status --change "<name>"
   ```

**出力**

完了後に次を要約する:
- 変更名と保存場所
- 作成した artifacts の一覧と短い説明
- `すべての artifacts を作成しました。実装の準備ができています。`
- `実装を開始するには /opsx:apply を実行してください。`

**Artifact Creation Guidelines**

- 各 artifact は `openspec instructions` の `instruction` に従う
- schema が各 artifact の内容を定義する
- 新しい artifact 作成前に dependency artifacts を読む
- `template` を土台にして文脈で埋める

**Guardrails**
- 実装に必要なすべての artifacts（schema の `apply.requires`）を作る
- 新規作成前に dependency artifacts を必ず読む
- 致命的に文脈不足なら質問するが、勢いを止めない範囲では合理的に判断する
- 同名 change が既にある場合は、続けるか新規にするかを確認する
- 次へ進む前に artifact ファイルが存在することを確認する

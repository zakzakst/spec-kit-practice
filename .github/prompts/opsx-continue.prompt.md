---
description: change の作業を続け、次の成果物を 1 つ作成する（Experimental）
---

change の作業を続け、次に作るべき成果物を 1 つ生成する。

**Input**: `/opsx:continue` の後に変更名を省略可能（例: `/opsx:continue add-auth`）。省略時は会話文脈から推定してよいが、曖昧なら必ず選択を求める。

**Steps**

1. **変更名がなければ選択させる**

   `openspec list --json` を実行し、最近更新された change 順に取得する。そのうえで **AskUserQuestion tool** を使い、続きから作業する change を選ばせる。

   選択肢には上位 3〜4 件の最近更新された change を表示し、次を見せる:
   - Change name
   - Schema（`schema` がなければ `spec-driven`）
   - Status（例: `0/5 tasks`, `complete`, `no tasks`）
   - `lastModified` に基づく更新の新しさ

   最も最近更新されたものは `(Recommended)` を付ける。

   **IMPORTANT**: 推測や自動選択をしない。必ずユーザーに選ばせる。

2. **現在の状態を確認する**
   ```bash
   openspec status --change "<name>" --json
   ```
   ここから次を把握する:
   - `schemaName`
   - `artifacts` と各 status (`done`, `ready`, `blocked`)
   - `isComplete`

3. **状態に応じて動く**

   **全成果物が完了している場合 (`isComplete: true`)**
   - 完了を祝う
   - 使用 schema を含めて最終状態を示す
   - `All artifacts created! You can now implement this change with /opsx:apply or archive it with /opsx:archive.` と案内して停止

   **作成可能な artifact がある場合**
   - `status: "ready"` の最初の artifact を選ぶ
   - instructions を取得する
     ```bash
     openspec instructions <artifact-id> --change "<name>" --json
     ```
   - JSON の主要フィールド:
     - `context`
     - `rules`
     - `template`
     - `instruction`
     - `outputPath`
     - `dependencies`
   - **artifact を作成する**
     - 完了済み dependency files を読む
     - `template` を構造として使う
     - `context` と `rules` は制約として使うが、そのまま出力へ写さない
     - 指定の `outputPath` に書く
   - 何を作ったか、何が次にアンロックされたかを示す
   - 1 回の呼び出しで 1 artifact 作成したら停止

   **ready な artifact がない場合**
   - 本来起こりにくいが、状態を表示し、問題確認を促す

4. **artifact 作成後に進捗を表示する**
   ```bash
   openspec status --change "<name>"
   ```

**Output**

各実行後に次を表示する:
- 作成した artifact
- 使用中の schema workflow
- Current progress（N/M complete）
- 新たにアンロックされた artifact
- `Run /opsx:continue to create the next artifact`

**Artifact Creation Guidelines**

artifact の種類と役割は schema に依存する。何を作るべきかは `instruction` フィールドに従うこと。

Common artifact patterns:

**spec-driven schema** (`proposal -> specs -> design -> tasks`)
- `proposal.md`: 変更内容が不明瞭ならユーザーに確認しつつ、Why / What Changes / Capabilities / Impact を埋める
- `specs/<capability>/spec.md`: proposal の Capabilities に列挙された capability ごとに 1 ファイル作る
- `design.md`: 技術判断、構成、実装方針を書く
- `tasks.md`: 実装作業をチェックボックス付きで分解する

その他の schema では、CLI の `instruction` を優先する。

**Guardrails**
- 1 回の呼び出しで作る artifact は 1 つだけ
- 新しい artifact を作る前に dependency artifacts を必ず読む
- 順番を飛ばしたりスキップしたりしない
- 文脈が不明なら作成前に聞く
- 書き込み後、artifact ファイルが存在することを確認する
- 固定の artifact 名を仮定せず schema に従う
- **IMPORTANT**: `context` と `rules` はあなた向けの制約であり、出力本文に貼り付けてはならない

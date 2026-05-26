---
description: 実験的な artifact 駆動ワークフロー（OPSX）で新しい change を開始する
---

実験的な artifact 駆動アプローチで新しい change を開始する。

**Input**: `/opsx:new` の後には、変更名（kebab-case）または作りたいものの説明が来る。

**Steps**

1. **入力がなければ、何を作りたいか聞く**

   **AskUserQuestion tool** を使って次を聞く:
   > "What change do you want to work on? Describe what you want to build or fix."

   その説明から kebab-case 名を導く（例: `add-user-auth`）。

   **IMPORTANT**: ユーザーの意図が分かるまで進まないこと。

2. **workflow schema を決める**

   ユーザーが別 schema を明示しない限り、デフォルト schema を使う（`--schema` は付けない）。

   **別 schema を使うのは次の場合だけ**
   - schema 名が明示された -> `--schema <name>`
   - `show workflows` / `what workflows` と言われた -> `openspec schemas --json` を実行して選ばせる

   **それ以外**: `--schema` は省略する。

3. **change ディレクトリを作成する**
   ```bash
   openspec new change "<name>"
   ```
   別 schema を指定された場合だけ `--schema <name>` を追加する。

4. **artifact の状態を表示する**
   ```bash
   openspec status --change "<name>"
   ```
   何を作る必要があり、何が ready かを確認する。

5. **最初の artifact の instructions を取得する**
   status 出力から、最初に `ready` になっている artifact を見つける。
   ```bash
   openspec instructions <first-artifact-id> --change "<name>"
   ```
   これにより、最初の artifact を作るための template と context が出る。

6. **ここで止まり、ユーザーの指示を待つ**

**Output**

完了後に次を要約する:
- Change name と location
- 使用 schema / workflow と artifact sequence
- Current status（0/N artifacts complete）
- 最初の artifact の template
- `Ready to create the first artifact? Run /opsx:continue or just describe what this change is about and I'll draft it.`

**Guardrails**
- まだ artifact を作らない
- 最初の artifact template を見せたところで止まる
- 名前が kebab-case で不正なら修正を求める
- 同名 change が既にある場合は `/opsx:continue` を勧める
- 非デフォルト workflow の場合のみ `--schema` を付ける

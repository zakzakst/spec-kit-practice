---
description: archive 前に、実装が change artifacts と一致しているか検証する
---

実装が change artifacts（specs, tasks, design）と一致しているかを検証する。

**Input**: `/opsx:verify` の後に変更名を省略可能（例: `/opsx:verify add-auth`）。省略時は文脈から推定してよいが、曖昧なら必ず選択を求める。

**Steps**

1. **変更名がなければ選択させる**

   `openspec list --json` を実行し、**AskUserQuestion tool** で change を選ばせる。
   実装タスクを持つ change（tasks artifact があるもの）を表示し、可能なら schema も見せる。未完了タスクがある change には `(In Progress)` を付ける。

   **IMPORTANT**: 推測や自動選択はしない。

2. **status を確認して schema を把握する**
   ```bash
   openspec status --change "<name>" --json
   ```
   ここから `schemaName` と存在する artifacts を把握する。

3. **change directory と artifacts を読み込む**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   change directory と `contextFiles` が返るので、そこにある artifacts を読む。

4. **verification report の枠組みを初期化する**

   次の 3 軸でレポートを作る:
   - **Completeness**
   - **Correctness**
   - **Coherence**

   各軸には `CRITICAL`, `WARNING`, `SUGGESTION` を付けられるようにする。

5. **Verify Completeness**

   **Task Completion**
   - `tasks.md` があれば読む
   - `- [ ]` と `- [x]` を解析
   - 未完了タスクがあれば:
     - 各タスクについて CRITICAL issue を追加
     - 推奨: `Complete task: <description>` もしくは `Mark as done if already implemented`

   **Spec Coverage**
   - `openspec/changes/<name>/specs/` に delta specs があれば:
     - `### Requirement:` をすべて抽出
     - 各 requirement についてコードベースを検索
     - 未実装と思われる場合:
       - `Requirement not found: <requirement name>` を CRITICAL にする
       - `Implement requirement X: <description>` を推奨する

6. **Verify Correctness**

   **Requirement Implementation Mapping**
   - 各 requirement について実装証拠を探す
   - 見つかったら file paths と line ranges を記録する
   - 意図と実装がずれていそうなら:
     - `Implementation may diverge from spec: <details>` を WARNING
     - `Review <file>:<lines> against requirement X` を推奨

   **Scenario Coverage**
   - `#### Scenario:` ごとに
     - 条件がコードで扱われているか
     - テストがあるか
   - カバー不足なら:
     - `Scenario not covered: <scenario name>` を WARNING
     - `Add test or implementation for scenario: <description>` を推奨

7. **Verify Coherence**

   **Design Adherence**
   - `design.md` があれば:
     - `Decision:`, `Approach:`, `Architecture:` などから設計判断を抽出
     - 実装が従っているか確認
     - 逆行があれば WARNING とし、実装更新または design.md 改訂を勧める
   - `design.md` がなければ、その旨を記録してスキップ

   **Code Pattern Consistency**
   - 新規コードがプロジェクトのパターンと整合しているかを見る
   - file naming、directory structure、coding style を確認
   - 大きなズレがあれば SUGGESTION とする

8. **Generate Verification Report**

   **Summary Scorecard**
   ```text
   ## Verification Report: <change-name>

   ### Summary
   | Dimension    | Status           |
   |--------------|------------------|
   | Completeness | X/Y tasks, N reqs|
   | Correctness  | M/N reqs covered |
   | Coherence    | Followed/Issues  |
   ```

   **Issues by Priority**

   1. **CRITICAL**
      - 未完了タスク
      - 未実装 requirement

   2. **WARNING**
      - spec / design と実装のズレ
      - 未カバー scenario

   3. **SUGGESTION**
      - パターン不一致
      - 軽微改善

   **Final Assessment**
   - CRITICAL がある: `X critical issue(s) found. Fix before archiving.`
   - WARNING のみ: `No critical issues. Y warning(s) to consider. Ready for archive.`
   - 問題なし: `All checks passed. Ready for archive.`

**Verification Heuristics**

- **Completeness**: checkbox や requirements list のような客観要素を重視
- **Correctness**: キーワード検索、パス分析、合理的推論を使う
- **Coherence**: 重大な不整合を拾う。細かすぎる指摘は避ける
- **False Positives**: 自信が低い場合は重さを下げる
- **Actionability**: すべての issue に具体的な推奨を付ける

**Graceful Degradation**

- tasks.md しかなければ task completion のみ検証
- tasks + specs なら completeness と correctness
- full artifacts があれば 3 軸すべて
- スキップした検証は理由付きで明示する

**Output Format**

- scorecard は表
- issue は `CRITICAL / WARNING / SUGGESTION` ごとに整理
- コード参照は `file.ts:123`
- 曖昧な提案は避け、具体策を書く

---
name: diagnose-why-work-stopped
description: >
  停滞・ループ・過剰回復した Paperclip issue tree を診断し、no-code の
  product-rule plan を提案します。作業が止まった理由、ループした理由、
  tree が深くなりすぎないようにする方法を問われたときに使います。
---

# なぜ作業が止まったかを診断する

ユーザー（または manager）が停滞・ループ・過剰回復した issue tree を指して、
「なぜ止まったのか」「なぜループしたのか」「どうすれば再発しないか」を問う
定番の手順です。

この skill はエンジニアリングではなく、**診断 + プロダクト設計**を行います。出力は
文書化された根本原因と承認済みの plan です。この skill からコード変更は行いません。

標準実行モデル: 新しい liveness / recovery rule を診断または提案する前に `doc/execution-semantics.md` を読みます。この文書を status、action-path、post-run disposition、bounded continuation、productivity review、pause-hold、watchdog、explicit recovery semantics の source of truth とします。調査で本当の product-rule gap が見つかった場合は、`doc/execution-semantics.md` に対応する更新が必要か plan に明記します。

## 使う場面

title または body が次のいずれかに一致する assignment で起動します:

- "why did this work stop", "why did this stall", "why did this just stop"
- "infinite loop", "looping", "spinning", "going too deep", "recovery went too deep"
- "liveness — what happened here", "this tree stopped working", "stuck"
- "approach it from a product perspective", "general product principle / rule"
- 特定の停滞・ループ・過剰 recovery 状態にある issue tree へのリンクが添付されている場合

プロダクト変更の前にフォレンジック調査、根本原因の特定、または文書化を求められた場合にも使います。

## 使わない場面

- assignment がコード変更の直接リリースを求めている場合。通常のエンジニアリングフローを使います。
- assignment が特定機能に対する通常のバグ報告である場合。通常の調査を行います。
- 自分が元の実装者で、自分のバグの修正を求められている場合。通常のデバッグを行います。

## 守るべき3つの invariant

すべての diagnosis と提案 rule は、次の3つの invariant を同時に満たす必要があります。ユーザーは少なくとも4つの issue でこれらを繰り返し示しているため、基盤となる条件として扱います:

1. **生産的な作業を継続する。** 次のアクションが明確な agent は、ユーザーに起動してもらわなくても作業を続けなければなりません。([PAP-2674](/PAP/issues/PAP-2674), [PAP-2708](/PAP/issues/PAP-2708))
2. **実際の blocker だけが作業を止める。** 本当に続行できない場合（承認不足、依存関係の不足、人間の担当者が必要な場合）に停止します。擬似的な停止（アクションパスのない `in_review`、キャンセルされた末端、壊れたメタデータ）は検出して適切な経路へ振り分け、黙って放置してはいけません。([PAP-2335](/PAP/issues/PAP-2335), [PAP-2674](/PAP/issues/PAP-2674))
3. **無限ループを発生させない。** 孤立作業の recovery と continuation loop には上限を設け、本当に生産的な継続と区別できなければなりません。([PAP-2602](/PAP/issues/PAP-2602), [PAP-2486](/PAP/issues/PAP-2486))

提案した rule が3つのどれかに反するなら、捨てるか作り直します。各 invariant を
どう守るかを plan に明記してください。

## 手順

### 0. 現在の実行契約を読む

tree をたどる前に `doc/execution-semantics.md` を読み、用語をそのまま使います:

- live path / waiting path / recovery path
- post-run disposition: terminal, explicitly live, explicitly waiting, invalid
- bounded `run_liveness_continuation`
- productivity review vs liveness recovery
- active subtree pause holds
- silent active-run watchdog

現在の execution semantics document とどう違うかを説明できるまで、新しい rule を
作ってはいけません。

### 1. 指定された tree の forensics - 何より先に行う

これを同じ heartbeat で行います。具体的な停止点を特定するまで rule を提案してはいけません。

- リンクされた issue（blocker chain、parent、recovery sibling、recent run を含む）を開きます。
- tree を node ごとにたどり、全体を停止させている正確な issue + state の組み合わせを見つけます。これまで company で見られた典型例:
  - 型付き execution participant、active run、pending interaction、recovery issue のいずれもない `in_review` ([PAP-2335](/PAP/issues/PAP-2335), [PAP-2674](/PAP/issues/PAP-2674))。
  - 成功した run の後、将来の action path がキューにない `in_progress` ([PAP-2674](/PAP/issues/PAP-2674))。
  - 末端が `cancelled`、壊れている、または company をまたいでアクセスできない blocker chain ([PAP-2602](/PAP/issues/PAP-2602))。
  - 成功した run の後に `issue.continuation_recovery` が同じ issue を N 回超起動している状態 ([PAP-2602](/PAP/issues/PAP-2602))。
  - 孤立作業の recovery が、自身の recovery issue をさらに recovery 可能な元作業として扱う状態 ([PAP-2486](/PAP/issues/PAP-2486))。
- run id、comment timestamp、status transition などの証拠を引用します。API boundary により直接の証拠を取得できない場合に限り「推測」を許可し、そのことを明示して主張を provisional とします（[PAP-2631](/PAP/issues/PAP-2631)）。

API boundary を尊重します。リンク先 issue が別 company にあり agent token が 403 を返す場合、scope を迂回してはいけません。board 承認済みの diagnostic path を依頼するか、PAP 側で推測できる証拠だけで進め、その旨を明記します。

### 2. 最近の関連作業を確認する

新しい product rule を提案する前に、同じ領域で今週すでにリリースされたものを確認します。ユーザーは明確に次の確認を求めています: ([PAP-2602](/PAP/issues/PAP-2602))「ここ数日でリリースした liveness に関する最近の作業をレビューする」。48 時間前に merge されたコードと矛盾する新しい rule は、改善ではなくやり直しです。

簡単な確認:
- 影響を受ける領域の最近 merge された PR。
- title に liveness、recovery、productivity、continuation、または対象サブシステムが含まれる最近の done issue。
- parent issue にある active な plan document。修正は新しいトップレベルの提案ではなく、既存 plan の改訂として扱うべき場合があります。

フォレンジック調査には「X、Y、Z を確認した。新たに見つかった gap は……」と明記します。

### 3. tree 内の停滞 issue をそれぞれ分類する

影響を受ける tree のうち、`done`、`cancelled`、または active に実行中ではないすべての issue について、次を判断します:

- **本当に人間または board の介入が必要** — 担当者と必要なアクションを明記します。
- **agent が実行可能だが、現在は振り分けられていない** — 振り分けるべきだった rule と、起動されるべき agent を明記します。
- **すでに対応済み** — active run、キューに入った wake、recovery issue、または pending interaction を示します。

これはユーザーが繰り返し求めている表です ([PAP-2335](/PAP/issues/PAP-2335))。これがなければ plan は抽象的なものになります。

### 4. 一般的な product rule として整理する

ユーザーが求めているのは、指定された tree への一度限りの patch ではなく、一般的な rule です。次の 2 点を確認します:

- rule は if/else の patch ではなく、**契約として記述**します。契約の例:「agent が所有する非終端 issue は、各 heartbeat の終了時に terminal state、明示的な waiting path、または明示的な live path のいずれかになっていなければならない」([PAP-2674](/PAP/issues/PAP-2674))。
- rule を `doc/execution-semantics.md` と整合させます。既存の契約を引用して適用することを優先し、現在の文書が不完全である場合、または承認済み・実装済みの動作と矛盾する場合にのみ文書変更を提案します。
- rule が上記 3 つの invariant を**明示的に維持**するようにします。どのように維持するかを示します。

その rule によって最近の生産的な run が成功できなくなるなら、提案を破棄するか、適用範囲を狭めます。

### 5. code ではなく plan を書く

plan は issue の `plan` document に書きます。次の内容を含めます:

- フォレンジック調査の概要（根本原因 + 証拠）。
- 契約として記述した一般的な product rule。
- 既存の `doc/execution-semantics.md` の契約で対応できるか、またはどの文書更新が正確に必要か。
- 段階的なサブタスク。通常は `Phase 0` で指定された live tree を慎重に、証拠を壊さず整理し、`Phase 1` で契約を文書化します。その後、検出、recovery、UI への表示、セキュリティレビュー、QA、CTO レビューの実装フェーズを設けます。
- 各フェーズの明示的な担当者。チームの専門性を優先します（server は CodexCoder、FE は ClaudeCoder、表示状態は UXDesigner、所有権・権限は SecurityEngineer、検証は QA）。
- `blockedByIssueIds` で接続したブロッキング依存関係と、並列化するブランチ。

この時点では child issue を作成しません。コードも push しません。

### 6. 承認を求め、それから分解する

- 最新の plan revision を対象に `request_confirmation` interaction を開きます。冪等性キーは `confirmation:{issueId}:plan:{revisionId}` とします。
- board または CTO の承認を待ちます。ユーザーが plan を置き換える新しいコメントを投稿した場合、以前の confirmation は無効になるため、新しい revision に紐づく confirmation を開きます（[PAP-2602](/PAP/issues/PAP-2602) では 3 回 revision が循環しましたが、それで問題ありません）。
- 承認後にのみ、適切な担当者と依存関係を設定した段階的な child issue を作成します。その後、chain が完了したときだけ parent が wake されるよう、最終 QA / CTO review issue に対してこの parent を block します。

### 7. 指定 tree に対する Phase 0 の整理

Phase 0 では、証拠を覆い隠さずに live tree を整理します:

- participant のいない停止中の `in_review` leaf を、具体的な次のアクションと担当者を設定した `todo` に移します ([PAP-2335](/PAP/issues/PAP-2335))。
- キューを塞いでいた cancelled / dead blocker を chain から切り離します。backlog を消すために issue を黙って `done` にしてはいけません。
- 元の指定 issue に、何をなぜ変更したかをまとめたコメントを残します。recovery chain の履歴を隠してはいけません。

### 8. 最終クローズアウト

phase chain が完了したら、parent issue に board レベルの概要コメントを投稿します。変更内容、新しい契約、rollout の手順（例:「新しい response shape を反映するため control-plane を再起動する」）、および当初指定された tree の live state を記載します。その後 parent を close します。

## 落とし穴

- **承認前に coding する。** ユーザーは最近のすべての diagnostic issue で「まず plan を作る」と述べています。フォレンジックフェーズでコードを作ると、往復のやり取りが無駄になります。
- **別の invariant を犠牲にして 1 つの invariant を繰り返す。** continuation の上限を厳しくしすぎると生産的な作業が停止し、recovery を緩めすぎると無限ループが再発します。常に 3 つすべてを確認します。
- **最近の作業確認を省略する。** 24 時間前にリリースされた内容と矛盾する契約を提案すれば、plan は簡単に却下されます。
- **`in_review` を done とみなす。** participant や active run のない、別 agent に割り当てられた leaf は進捗ではありません。停止として扱います。
- **company のスコープを回避する。** company をまたぐフォレンジック調査には、database read ではなく board 承認済みの diagnostic path が必要です。
- **再帰的な recovery。** 自身の recovery issue を recovery する孤立作業の recovery は、典型的な無限ループです ([PAP-2486](/PAP/issues/PAP-2486))。検出したら、それ以上深くしません。
- **chain を隠す。** 症状となった recovery issue を黙って削除・非表示にしてはいけません。operator には監査証跡が必要です。

## 検証チェックリスト（plan を投稿する前）

- [ ] 指定された tree の正確な停止点が、run id / comment id とともに特定されている。
- [ ] 同じ領域で最近リリースされた作業を調査し、参照している。
- [ ] 進行していないすべての issue が、人間による対応が必要 / agent が実行可能 / すでに対応済み、のいずれかに分類されている。
- [ ] 提案する rule が patch ではなく契約として記述されている。
- [ ] 3 つすべての invariant が明示的に維持されている。
- [ ] この heartbeat でコード変更が行われていない。
- [ ] 最新の plan revision に対する `request_confirmation` が開かれている。
- [ ] plan の Phase 0 が、証拠を壊さずに指定された live tree を扱っている。
- [ ] 実装フェーズに、専門性に合った担当者と `blockedByIssueIds` による依存関係が記載されている。

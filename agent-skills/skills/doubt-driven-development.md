---
name: doubt-driven-development
description: 重要でないこと以外のあらゆる判断を、新鮮な文脈の敵対的レビューにかけます。正しさが速度より重要なとき、慣れていないコードを扱うとき、リスクが高いとき（本番、セキュリティに関わる処理、取り消しにくい操作）、または自信のある出力を今すぐ検証したほうが後でデバッグするより安いときに使います。
---

# Doubt-Driven Development

## 概要

自信があることは、正しいことを意味しません。長いセッションではコンテキストが積み重なり、気づかないうちに前提が「事実」へ変わります。Doubt-driven development は、非自明な出力を出す前に、**承認ではなく反証**を目的にした新鮮な文脈のレビューを具現化する規律です。

これは `/review` ではありません。`/review` は完成物への最終判定です。これは進行中の姿勢です。コース修正がまだ安い段階で、非自明な判断を交差尋問します。

## 使う場面

次のどれかが真なら、その判断は **non-trivial** です。

- 分岐ロジックを追加・変更する
- モジュールまたはサービス境界をまたぐ
- 型システムやコンパイラでは保証できない性質（スレッド安全性、冪等性、順序、不変条件）を主張する
- 正しさが、後から読む人には見えない文脈に依存する
- 破壊力が取り消しにくい（本番デプロイ、データ移行、公開 API 変更）

次の場面で使います。

- 不確実なアーキテクチャ判断をしようとしている
- 非自明なコードをコミットしようとしている
- 非自明な事実を主張しようとしている（"安全だ"、"スケールする"、"仕様に合っている" など）
- 十分に理解していないコードを扱っている

**使わない場面:**

- 機械的操作（rename、format、file move）
- 明確で曖昧さのないユーザー指示に従うとき
- 既存コードの読み取りや要約
- 1 行変更で正しさが明白なもの
- 純粋な tooling 操作（テスト実行、ファイル一覧）
- ユーザーが検証より速度を明示的に優先した

すべてのキー入力を疑うなら、何も出荷できません。このスキルは、上で定義した non-trivial な判断にだけ適用します。

## 読み込み制約

このスキルは **main-session orchestrator** 向けです。Step 3（DOUBT、後述）で新鮮な文脈のレビュアーを起動できます。

- **persona の `skills:` frontmatter にこのスキルを追加しないでください。** Step 3 を追う persona は別の persona を起動することになり、`../../references/orchestration-patterns.md` にある禁止されたオーケストレーション anti-pattern に当たります。
- **subagent 文脈からこのスキルを使おうとしている場合**（Claude Code が nested subagent を禁止する場面）: 望ましいのは、この doubt-driven は nested では実行できないとユーザーへ伝え、main session に処理させることです。最後の手段としてだけ、劣化版の自己問い直し fallback があります - ARTIFACT + CONTRACT を、前の推論から明確に切り離した fresh self-prompt に書き直し、Step 1〜5 を自分で回します。これは **fresh-context review ではありません**（自分のコンテキストを持ち続けるため）ので、結果は劣化版として扱い、ユーザーに到達可能ならエスカレーションを優先してください。

## プロセス

このチェックリストをそのまま使ってください。

```text
Doubt cycle:
- [ ] Step 1: CLAIM - 主張と、その主張が重要な理由を書いた
- [ ] Step 2: EXTRACT - artifact と contract を分離し、推論を削った
- [ ] Step 3: DOUBT - fresh-context reviewer に adversarial prompt を投げた
- [ ] Step 4: RECONCILE - すべての指摘を artifact の文面に照らして分類した
- [ ] Step 5: STOP - 停止条件（些細な指摘、3 サイクル、またはユーザーの override）を満たした
```

### Step 1: CLAIM - 何が立っているかを明示する

判断を 2〜3 行で書きます。

```text
CLAIM: "新しい caching layer は、spec で説明された read-heavy workload に対して thread-safe だ"
WHY THIS MATTERS: ここで race が起きるとユーザーデータが壊れ、QA では検出しにくい
```

この主張を短く書けないなら、判断ではなく感覚です。疑う前に表に出してください。

### Step 2: EXTRACT - 最小のレビュー単位

fresh-context reviewer に必要なのは、**artifact** と **contract** であって、思考の道のりではありません。

- Code: diff か関数そのもの - ファイル全体ではない
- Decision: 3〜4 文の提案と、満たすべき制約
- Assertion: 主張と、その裏付けとして使った evidence（Step 1 の CLAIM ブロックとは別。Step 1 は orchestratior の仮説です）

推論を削ってください。結論を渡すと、結論の追認が返ってきます。単位は 1 回で持てる大きさにしてください。500 行の PR なら、先に分解します。

### Step 3: DOUBT - fresh-context reviewer を起動する

レビュー担当への prompt は **敵対的** でなければなりません。フレーミングが答えを決めます。

```text
Adversarial review. Find what is wrong with this artifact.
Assume the author is overconfident. Look for:
- Unstated assumptions
- Edge cases not handled
- Hidden coupling or shared state
- Ways the contract could be violated
- Existing conventions this might break
- Failure modes under unexpected input

Do NOT validate. Do NOT summarize. Find issues, or state
explicitly that you cannot find any after thorough examination.

ARTIFACT: <paste artifact>
CONTRACT: <paste contract>
```

**ARTIFACT + CONTRACT だけを渡してください。CLAIM は渡さないでください。** 結論を渡すと、レビュアーはそれを追認しがちです。レビュアーは独立して artifact が contract を満たすか判断しなければなりません。

Claude Code では、`agents/` の役割ベースのレビュアーは隔離された context で始まるので、この用途に使えます。roster と domain 別の対応は `agents/` を参照してください。

**上の adversarial prompt が persona の既定出力形状より優先されます。** `code-reviewer` のような persona は、通常は強みと弱みの両方を含むバランスの取れた verdict を返すよう書かれていますが、doubt-driven では issues-only の出力が必要です。起動時にその prompt を verbatim で貼って、既定形状を上書きしてください。persona の出力形状がきれいに上書きできないなら、generic subagent に adversarial prompt を渡してください。

#### Cross-model escalation

単一モデルのレビュアーは、元の作者と盲点を共有します。別アーキテクチャの冷たいモデルのほうが見つけやすいことがあります。doubt-driven は non-trivial 判断に対してもともと任意ではないので、その範囲では cross-model を提案するのはこのスキルの価値の一部であり、余計な摩擦ではありません。

**対話セッションでは、必ず提案する。静かに省略しない。**

**Step 1: ユーザーに尋ねる**

Step 3 の single-model review が終わったあと、RECONCILE の前にいったん止めて、次のように尋ねます。

> *"Single-model review complete. Want a cross-model second opinion? Options: Gemini CLI, Codex CLI, manual external review (you paste it elsewhere), or skip."*

これは interactive な doubt cycle では必須です。artifact が軽そうに見えても同じです。コストに見合うかどうかを決めるのはユーザーです。エージェントの役目は選択肢を表に出すことです。

**Step 2: CLI を選んだら、確認してから起動する**

1. ツールが PATH にあるか確認する（`which gemini`, `which codex`）
2. 実際に動くかテストする（`gemini --version` など） - 古い / 壊れた binary は `which` を通っても本番入力で失敗することがある
3. 必要な flags、auth、env vars を含めて、正確な起動方法をユーザーに確認する
4. ARTIFACT + CONTRACT + adversarial prompt **だけ**を渡す。session context も CLAIM も渡さない
5. shell escaping に注意する。artifact に quote、`$(...)`、backtick が含まれるなら、`-p "..."` の inline より stdin（`echo ... | gemini`）か heredoc を使う
6. 出力を Step 4（RECONCILE）に持ち込む

**artifact を shell でクォートした引数に直接埋め込まないでください。** コード、Markdown、review prompt には backtick、`$(...)`、quote がよく含まれ、prompt を切るか、埋め込まれた shell を実行してしまいます。全プロンプトを一時ファイルに書いて stdin で流してください。

例（使っているツールの flags は確認してください。実装やバージョンで違います）:

```bash
# adversarial prompt + ARTIFACT + CONTRACT を一時ファイルに書く
# その後 stdin で流す。artifact 内の shell metacharacter を inert にするため。

# Codex (read-only sandbox では CLI が workspace に書き込まない):
codex exec --sandbox read-only -C <repo-path> - < /tmp/doubt-prompt.md

# Gemini ('--approval-mode plan' は read-only; '-p ""' で非対話モード、
# prompt は stdin から読む):
gemini --approval-mode plan -p "" < /tmp/doubt-prompt.md
```

read-only sandbox は重要です。doubt artifact 自体が instructions を含む場合（意図的でも偶発的でも）、cross-model CLI がそれを workspace に対して実行してしまう恐れがあります。

**Step 3: CLI が使えない、または失敗したら**

失敗をはっきり伝えます。手動で実行する、別のツールを試す、スキップする、の選択肢を出します。single-model に黙って戻ってはいけません。cross-model が行われなかったことをユーザーは知るべきです。

**Step 4: ユーザーがスキップしたら**

スキップを出力に明記し、続行します（"Proceeding with single-model findings only"）。スキップ自体は問題ありません。黙ってスキップするのが問題です。

**非対話コンテキスト**（CI、`/loop`、autonomous-loop、scheduled runs）:

- cross-model は **スキップ** し、そのスキップを出力に **明示** します: *"Cross-model skipped: non-interactive context."*
- **明示的なユーザー許可なしに外部 CLI を起動してはいけません** - これは重要な安全性です

cross-model はコスト、待ち時間、ツールの脆さを増やします。このスキルは interactive な doubt cycle ごとに選択肢を出し、ユーザーが artifact にその価値があるか決めます。

### Step 4: RECONCILE - 指摘を折り返す

レビュアーの出力はデータであって verdict ではありません。**最終的な orchestrator はあなたです。** 各指摘を分類する前に、artifact の文面と照らし合わせて読み直してください。レビュアーを盲目的に追認するのは、無視するのと同じ失敗です。

各指摘について、この **優先順** で分類します（最初に当てはまるものが勝ち）:

1. **Contract misread** - reviewer が、あなたの CONTRACT が曖昧または不完全だったために指摘した。まず contract を直し、次の cycle で再分類する
2. **Valid + actionable** - 実在する問題で、artifact を変える必要がある。直して再ループする
3. **Valid trade-off** - 問題は本物だが、直すコストが受け入れるコストより高い。ユーザーが見えるように、トレードオフを明示する
4. **Noise** - reviewer が context 不足で誤って指摘した。注記して先へ進み、もし contract に context を足していれば false flag を防げたか考える

fresh reviewer は context が足りないために間違うことがあります。fresh だからといって、無条件に従う必要はありません。

### Step 5: STOP - 再帰ではなく bounded loop

次の場合に止めます。

- 次の反復で、すでに考慮済みか trivial な指摘しか出ない
- 3 サイクル完了した（4 周目は回さず、ユーザーへエスカレート）
- ユーザーが明示的に "ship it" と言った

3 サイクル経っても substantive な問題が出るなら、artifact はまだ準備できていない可能性があります。ユーザーにそれを伝えてください。3 回の未解決サイクルは artifact についての情報であって、さらにループする理由ではありません。

3 サイクルでは明らかに足りないくらい artifact が大きいなら、artifact が大きすぎます。Step 2 に戻って分解してください。上限を上げてはいけません。

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「自信があるから doubt は飛ばす」 | 自信は、新規問題では正しさとあまり相関しません。確信が強い瞬間ほど盲点があります。 |
| 「レビュアーを起動すると高くつく」 | 本番で間違った commit をデバッグするほうが高くつきます。チェックは bounded、バグは bounded ではありません。 |
| 「レビュアーは揚げ足取りするだけ」 | スコープを絞らなければそうなるだけです。prompt を "契約上 fail になる issue" に絞ってください。 |
| 「最後に `/review` で doubt すればいい」 | `/review` は最終ゲートです。doubt-driven は、コース修正がまだ安い段階で wrong direction を捕まえます。PR 時点では遅すぎます。 |
| 「毎ステップ疑っていたら出荷できない」 | このスキルは non-trivial な判断だけに適用します。すべてのキー入力ではありません。When NOT to Use を読み直してください。 |
| 「2 つの意見は常に 1 つより良い」 | 2 つ目が context 不足でノイズを出すなら違います。defer ではなく reconcile してください。 |
| 「レビュアーが反対したから自分が間違っていた」 | レビュアーはあなたの context を持っていません。反対は情報であって verdict ではありません。artifact を読み直し、分類し、それから判断します。 |
| 「cross-model は常に better」 | cross-model は single model が自分自身と共有する盲点を見つけやすいですが、コストと fragility を増やします。interactive な doubt cycle ごとに提案し、ユーザーが必要性を決めます。選択肢を出すのがエージェントの役目で、ゲートにすることではありません。 |
| 「一度 yes をもらったから CLI は毎回勝手に起動していい」 | 1 回ごとに許可が必要です。artifact、prompt、flags は毎回変わります。毎回、正確な command を再確認してください。 |

## レッドフラグ

- 1 行の rename や formatting change に fresh-context reviewer を起動する
- artifact の文面を再読せずに reviewer の出力を権威として扱う
- 3 サイクル超でユーザーへエスカレーションせずにループする
- "is this good?" ではなく "find issues" と prompt しない
- 高リスク判断で時間圧を理由に doubt を飛ばす
- 変更していない artifact に対して fresh-context を再起動する（同じ指摘が返るだけで、足踏みです）
- **Doubt theater**（確認可能な兆候）: 2 サイクル以上で substantive な指摘が出たのに、actionable と分類された指摘が 0 件。つまり、疑っているのではなく validation しているだけです。止まってエスカレートしてください。
- コミット後にだけ doubt をする - それは `/review` であって doubt-driven development ではありません
- ツールが存在するか、設定されているか、その syntax で本当に動くかをユーザー確認なしで外部 CLI を hardcode する
- **対話中の doubt cycle で cross-model を黙ってスキップする。** 推奨しない場合でも、選択肢は見える形で出す必要があります。スキップは可、黙ってスキップは不可です。
- 外部 CLI エラーや欠如に黙って fallback する - 失敗を表に出し、ユーザーに方向転換してもらう
- reviewer の入力から contract を削る
- CLAIM を reviewer に渡す（追認バイアスがかかる）

## 他スキルとの関係

- **`code-review-and-quality` / `/review`**: 補完関係です。`/review` は PR 後の最終 verdict、doubt-driven は進行中の各判断です。両方使います。
- **`source-driven-development`**: SDD は公式 docs を使って *フレームワークの事実* を確認します。Doubt-driven は *artifact に対する自分の推論* を検証します。SDD は API の存在を確かめ、doubt-driven は contract に照らして正しく使ったかを確かめます。
- **`test-driven-development`**: TDD の RED step は doubt を具体化したものです - 失敗するテストは反証の試みです。TDD が当てはまるなら、その失敗するテストが behavior の主張に対する doubt step です。
- **`debugging-and-error-recovery`**: reviewer が実在の failure mode を示したら、デバッグスキルに切り替えて局所化と修正を行います。
- **Repo orchestration rules** (`../../references/orchestration-patterns.md`): このスキルは main session から orchestrate します。persona が別 persona を呼ぶのは anti-pattern B です。上の Loading Constraints を参照してください。

## 検証

Doubt-driven development を適用したあと、次を確認します。

- [ ] non-trivial な判断（上の定義に従う）を、立たせる前に CLAIM として明示した
- [ ] 少なくとも 1 つの fresh-context review を non-trivial artifact ごとに実施した（TDD の RED step が作る failing test は、behavior の主張に対するこの役割を果たします）
- [ ] reviewer に渡したのは ARTIFACT + CONTRACT であり、CLAIM も推論も渡していない
- [ ] reviewer の prompt は validating ではなく adversarial（"find issues"）だった
- [ ] 指摘を artifact の文面に照らして分類した（盲目的な追認ではない）
- [ ] 停止条件（些細な指摘、3 サイクル、ユーザー override）のいずれかを満たした
- [ ] interactive mode では cross-model をユーザーに **明示的に提案** した（artifact の重要度に関係なく）し、応答を output に明記した
- [ ] non-interactive mode では cross-model をスキップし、その旨を明記した
- [ ] 外部 CLI を起動した場合、PATH 確認、動作確認、ユーザーへの syntax 確認、明示的な許可が先にあった

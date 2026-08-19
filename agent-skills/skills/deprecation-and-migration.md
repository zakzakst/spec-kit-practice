---
name: deprecation-and-migration
description: 古いシステム、API、機能の退役と移行を扱います。古い仕組みを外したいとき、ユーザーを新実装へ安全に移したいとき、既存コードを維持するか切り捨てるか判断したいときに使います。
---

# 非推奨化と移行

## 概要

コードは資産ではなく負債です。コードの 1 行ごとに、保守、テスト、依存関係更新、セキュリティパッチ、新人のオンボーディングといった継続コストがかかります。非推奨化は、もはや価値を生まないコードを取り除く規律であり、移行は、ユーザーを古いものから新しいものへ安全に移すプロセスです。

多くの組織は作るのは得意ですが、外すのは苦手です。このスキルはそのギャップを埋めます。

## 使う場面

- 古いシステム、API、ライブラリを新しいものへ置き換えるとき
- もう不要な機能を廃止するとき
- 重複した実装を統合するとき
- 誰も責任を持たないが皆が依存している死んだコードを消すとき
- 新しいシステムのライフサイクルを計画するとき（非推奨化の計画は設計時から始める）
- レガシーシステムを維持するか、移行へ投資するか決めるとき

## コア原則

### コードは負債

各行のコードには継続コストがあります。テスト、ドキュメント、セキュリティパッチ、依存関係更新、近くで作業する人の認知負荷です。コードの価値は、それが提供する機能であって、コード自体ではありません。同じ機能を、より少ないコード、より少ない複雑さ、より良い抽象化で提供できるなら、古いコードは退くべきです。

### Hyrum's Law で削除は難しくなる

利用者が増えると、観測可能な振る舞いはすべて依存対象になります。バグ、タイミングの癖、文書化されていない副作用までも含みます。だから非推奨化には、単なる告知ではなく、積極的な移行が必要です。置き換え先が同じ振る舞いを再現しないと、ユーザーは "ただ切り替える" ことができません。

### 非推奨化の計画は設計時から始める

新しいものを作るとき、次を考えます: "3 年後にこれをどう外すか？" 明確なインターフェース、feature flag、最小の surface area を持つシステムは、実装詳細があちこちに漏れたシステムより非推奨化しやすくなります。

## 非推奨化の判断

何かを非推奨化する前に、次を答えます。

```text
1. この system はまだ unique value を提供しているか？
   -> yes なら維持する。no なら次へ。

2. 何人のユーザー / consumer が依存しているか？
   -> migration scope を数える。

3. 置き換え先はあるか？
   -> no なら、先に置き換え先を作る。代替なしに非推奨化しない。

4. 各 consumer の migration cost は？
   -> 自動化できるならやる。手作業で高コストなら、保守コストと比べる。

5. 非推奨化しない場合の継続保守コストは？
   -> セキュリティリスク、エンジニア時間、複雑さによる機会損失。
```

## Advisory と Compulsory

| Type | 使う場面 | 機構 |
|---|---|---|
| **Advisory** | 移行は任意で、古い system は安定している | 警告、ドキュメント、後押し。ユーザーが自分のタイミングで移行する |
| **Compulsory** | 古い system にセキュリティ問題がある、進行を妨げる、保守コストが持続不能 | 期限を切る。旧 system は日付 X で削除する。migration tooling を提供する |

**既定は advisory です。** compulsroy は、保守コストやリスクが強制移行を正当化するときだけ使います。Compulsory deprecation には、migration tooling、documentation、support が必要です - 期限を告知するだけではだめです。

## Migration Process

### Step 1: 置き換え先を作る

working alternative なしに非推奨化しません。置き換え先は次を満たす必要があります。

- 古い system の重要な use case をすべてカバーする
- documentation と migration guide がある
- production で実証されている（"理論上 better" だけではだめ）

### Step 2: 告知と文書化

```markdown
## Deprecation Notice: OldService

**Status:** Deprecated as of 2025-03-01
**Replacement:** NewService (migration guide below)
**Removal date:** Advisory - no hard deadline yet
**Reason:** OldService requires manual scaling and lacks observability.
           NewService handles both automatically.

### Migration Guide
1. `import { client } from 'old-service'` を `import { client } from 'new-service'` に置き換える
2. 設定を更新する（下の例を参照）
3. migration verification script を実行する: `npx migrate-check`
```

### Step 3: 段階的に移行する

consumer は一斉ではなく、1 つずつ移行します。各 consumer について:

```text
1. deprecated system との touchpoint をすべて洗い出す
2. replacement に切り替える
3. 振る舞いが一致するか検証する（テスト、統合チェック）
4. old system への参照を削除する
5. 回帰がないことを確認する
```

**Churn Rule:** 非推奨化する infrastructure を自分で持っているなら、ユーザーの移行責任はあなたにあります。あるいは、移行が不要な後方互換更新を提供します。非推奨化を告げて、あとはユーザー任せにしてはいけません。

### Step 4: 古い system を削除する

すべての consumer が移行したあとにだけ、次を実行します。

```text
1. active usage が 0 であることを確認する（metrics、logs、dependency analysis）
2. コードを削除する
3. 関連する tests、documentation、configuration を削除する
4. deprecation notice を消す
5. 祝う - code を減らせたのは成果です
```

## Migration Patterns

### Strangler Pattern

古い system と新しい system を並行運転し、トラフィックを徐々に新しいほうへ流します。old system が 0% になったら削除します。

```text
Phase 1: New 0%, Old 100%
Phase 2: New 10% (canary)
Phase 3: New 50%
Phase 4: New 100%, Old idle
Phase 5: Remove old system
```

### Adapter Pattern

古い interface の呼び出しを新しい実装へ翻訳する adapter を作ります。consumer は古い interface のまま使い続け、内部だけを移行します。

```typescript
// Adapter: old interface, new implementation
class LegacyTaskService implements OldTaskAPI {
  constructor(private newService: NewTaskService) {}

  // 古い method signature のまま、新しい実装に委譲
  getTask(id: number): OldTask {
    const task = this.newService.findById(String(id));
    return this.toOldFormat(task);
  }
}
```

### Feature Flag Migration

feature flag で、consumer を old から new へ 1 つずつ切り替えます。

```typescript
function getTaskService(userId: string): TaskService {
  if (featureFlags.isEnabled('new-task-service', { userId })) {
    return new NewTaskService();
  }
  return new LegacyTaskService();
}
```

### Database Schema Migration（Expand / Contract）

schema change はもっとも危険な migration です。data は deploy を revert しても巻き戻せないからです。失敗パターンは、schema change と code change を同じ release に結びつけることです。列名を変えた release で同時に新しい名前を使い始めると、rollout window 中に古い code と新しい code が同時に動き、一方が存在しない column を参照します。解決策は **column を in place で変えないこと** です。old code と new code がどの段階でも両立するよう、additive phase で進めます。

```text
EXPAND -> MIGRATE -> CONTRACT
新しい column を追加し、   既存 row を backfill し、   旧 column を読む code が
nullable のまま old と     app から old+new を dual- なくなったら、後の別 deploy
並べて置く                write する                で削除する
```

**Worked example - `name` を `full_name` に rename する:**

1. **Expand.** `full_name` を nullable で追加して deploy する（old code は無視するので壊れない）
2. **Dual-write.** insert / update のたびに `name` と `full_name` の両方へ書く
3. **Backfill.** 既存 rows を `name -> full_name` で batch で埋める（table lock を避ける）
4. **Switch reads.** app の read を `full_name` に切り替え、書き込みは両方に続ける
5. **Contract.** `name` への書き込みを止め、その後の別 deploy で column を drop する

各 step は独立して deploy / revert 可能です。Step 4 が悪さをしたら code を戻しても、`full_name` はまだ埋まっています。各 phase は `incremental-implementation` に従う thin vertical slice として扱います。

**ルール:**
- **Additive first, destructive last and alone.** 追加（new nullable column、新しい table、新しい index）はどの deploy でも安全。drop と rename は、旧 shape を参照する code がなくなった後に、独立した deploy で行う
- **すべての migration に tested down path を持たせる。** 逆に戻せない migration は revert できない deploy です。merge 前に `down` を書いて実行する
- **Backfill は batch で、hot path の外で行う。** 1 回の `UPDATE` で数百万 rows を触ると table が lock されます。chunk して throttle する
- **大きな index は書き込みを止めずに作る**（例: Postgres の `CREATE INDEX CONCURRENTLY`）
- **cutover が危険なら feature flag で code から切り離す**。上の Feature Flag Migration と同じです

## Zombie Code

Zombie code は、誰も責任を持たないのに皆が依存している code です。積極的に保守されず、明確な owner がなく、security vulnerability や互換性問題を溜めます。兆候:

- 6 か月以上 commit がないのに active consumer がいる
- 担当 maintainer や team がいない
- 誰も直さない failing test がある
- known vulnerability のある dependency を誰も更新しない
- 存在しない system を参照する documentation がある

**対応:** owner を割り当てて適切に維持するか、具体的な migration plan を付けて非推奨化します。Zombie code を宙ぶらりんにしてはいけません。投資するか削除するかのどちらかです。

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「まだ動くのに、なぜ消すの？」 | 誰も保守しない code は security debt と複雑さを積み上げます。保守コストは静かに増えます。 |
| 「後で誰かが必要とするかも」 | 後で必要なら再実装できます。使われていない code を "念のため" 残すほうが高くつきます。 |
| 「migration が高すぎる」 | 2〜3 年の継続保守コストと比べてください。多くの場合、長期的には migration のほうが安いです。 |
| 「新 system が終わってから非推奨化する」 | 非推奨化計画は設計時から始めます。新 system 完成時には、別の優先事項が増えています。今計画してください。 |
| 「ユーザーは自分で移行する」 | しません。tooling、documentation、incentive を提供するか、自分で移行してください（Churn Rule）。 |
| 「両方の system を無期限に維持できる」 | 同じことをする 2 つの system は、保守、テスト、文書化、オンボーディングのコストが 2 倍です。 |
| 「ただ column 名を変えればいい、1 行だ」 | rollout 中は old と new code が同時に走ります。片方は存在しない column を参照します。expand/contract を使い、in place の rename はしないでください。 |
| 「column を追加して、同じ migration で old を消せばいい」 | 安全な追加と破壊的な削除を結びつけています。drop は独立した deploy で、旧 shape を参照する code がなくなった後に行います。 |
| 「必要になったら rollback を書く」 | down path のない migration は、戻せない deploy です。merge 前に down を書いて実行してください。 |

## レッドフラグ

- 代替のない deprecated system
- migration tooling や documentation なしの非推奨化告知
- 何年も advisory のままで進展がない "soft" deprecation
- owner のいない zombie code と active consumer
- deprecated system に新機能を足す（代わりに replacement に投資する）
- current usage を測らずに非推奨化する
- zero active consumer を確認せずに code を消す
- schema change と、その変更に依存する code が同じ deploy で出る
- column を expand/contract ではなく in place で rename / drop する
- tested down path のない migration、または table を lock する backfill

## 検証

deprecation を完了したら:

- [ ] replacement が production で実証され、すべての重要な use case をカバーしている
- [ ] 具体的な手順と例を含む migration guide がある
- [ ] active consumer がすべて移行済み（metrics / logs で確認）
- [ ] old code、tests、documentation、configuration が完全に削除されている
- [ ] deprecated system への参照が codebase に残っていない
- [ ] deprecation notice が削除されている（役目を終えた）

database schema migration を完了したら:

- [ ] 変更は additive phase（expand -> backfill -> contract）で出ており、in-place edit ではない
- [ ] 各 deploy step で old と new の両方が schema に対して有効
- [ ] 各 migration に tested down path があり、backfill は throttle された batch で実行される
- [ ] 破壊的 step（drop / rename）は、旧 shape を参照する code がなくなった後に独立した deploy で出る

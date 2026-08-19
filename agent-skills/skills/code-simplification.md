---
name: code-simplification
description: 読みやすさのためにコードを簡素化します。振る舞いを変えずに複雑さを減らすリファクタリングに使います。コードが動いてはいるが、読む・保守する・拡張するのに重すぎるときに使います。不要な複雑さが溜まったコードをレビューするときにも使います。
---

# コード簡素化

> [Claude Code Simplifier plugin](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/code-simplifier/agents/code-simplifier.md) に着想を得て、どの AI コーディングエージェントでも使える、モデル非依存・プロセス駆動のスキルとして整えたものです。

## 概要

振る舞いを一切変えずに、複雑さを減らしてコードを簡素化します。目標は行数を減らすことではありません。読む・理解する・変更する・デバッグするのが楽なコードにすることです。簡素化のたびに、1 つの簡単なテストを通してください: 「新しいチームメンバーが、元のコードより速く理解できるか？」

## 使う場面

- 機能は動いてテストも通るが、実装が必要以上に重いと感じるとき
- コードレビューで可読性や複雑さの問題が指摘されたとき
- 深いネスト、長い関数、分かりにくい名前に出会ったとき
- 時間に追われて書かれたコードをリファクタリングするとき
- 関連ロジックが複数ファイルに散らばっているとき
- 重複や不整合を生んだ変更をマージしたあと

**使わない場面:**

- コードがすでにきれいで読みやすい - ためらいなく簡素化しない
- まだ何をしているか理解していない - まず理解してから簡素化する
- パフォーマンスが重要で、"簡単" に見える版が測定可能に遅くなる
- モジュールを全面的に書き直す直前 - 捨てるコードに労力をかけない

## 5 つの原則

### 1. 振る舞いを正確に保つ

コードが何をするかは変えず、どう表現するかだけを変えます。入力、出力、副作用、エラー、境界条件はすべて同一でなければなりません。簡素化が振る舞いを保つか確信できないなら、やりません。

```text
各変更の前に確認:
-> すべての入力で同じ出力か？
-> 同じエラー振る舞いか？
-> 同じ副作用と順序を保つか？
-> 既存テストは変更なしで全部通るか？
```

### 2. プロジェクト規約に従う

簡素化は、外部の好みを押し付けることではなく、コードベースにより一貫させることです。簡素化の前に:

```text
1. CLAUDE.md / project conventions を読む
2. 周辺コードが似たパターンをどう扱うかを見る
3. 次の点でプロジェクトの流儀に合わせる:
   - import 順序と module system
   - function declaration のスタイル
   - naming conventions
   - error handling パターン
   - type annotation の深さ
```

規約を壊す簡素化は、簡素化ではなく churn です。

### 3. 巧妙さより明快さを優先する

コンパクトさより、明白さを優先します。短くても、読むのに一瞬止まるなら明快ではありません。

```typescript
// 分かりにくい: 複雑な三項演算子
const label = isNew ? 'New' : isUpdated ? 'Updated' : isArchived ? 'Archived' : 'Active';

// 分かりやすい: 読める mapping
function getStatusLabel(item: Item): string {
  if (item.isNew) return 'New';
  if (item.isUpdated) return 'Updated';
  if (item.isArchived) return 'Archived';
  return 'Active';
}
```

```typescript
// 分かりにくい: reduce の中にロジックを詰め込みすぎ
const result = items.reduce((acc, item) => ({
  ...acc,
  [item.id]: { ...acc[item.id], count: (acc[item.id]?.count ?? 0) + 1 }
}), {});

// 分かりやすい: 名前付きの中間ステップ
const countById = new Map<string, number>();
for (const item of items) {
  countById.set(item.id, (countById.get(item.id) ?? 0) + 1);
}
```

### 4. バランスを保つ

簡素化には、やりすぎという失敗もあります。次を警戒します。

- **インライン化しすぎる** - helper を消して概念に名前がなくなると、呼び出し側が読みにくくなる
- **無関係なロジックをまとめる** - 2 つの単純な関数を 1 つの複雑な関数にしても簡単にはならない
- **"不要に見える" 抽象を消す** - 抽象は拡張性やテスト容易性のためにあることもある
- **行数だけで最適化する** - 少ない行数が目的ではない。理解しやすさが目的

### 5. 変更範囲を絞る

基本は、最近変更されたコードを簡素化します。明示的に範囲を広げるよう求められていない限り、関係ないコードまで勝手にリファクタリングしません。範囲外の簡素化は diff を汚し、意図しない回帰を生みます。

## 簡素化のプロセス

### Step 1: 触る前に理解する（Chesterton's Fence）

何かを変えたり消したりする前に、それが存在する理由を理解します。これは Chesterton's Fence です。道にフェンスがあって、なぜあるか分からないなら壊してはいけません。理由を理解してから、その理由がまだ有効か判断します。

```text
簡素化前に答える:
- このコードの責任は何か？
- 誰が呼んでいるか？ 何を呼んでいるか？
- エッジケースとエラーパスは？
- 期待される振る舞いを定義するテストはあるか？
- なぜこの形で書かれたのか？（性能、プラットフォーム制約、歴史的理由）
- git blame を見る: 当初の文脈は何だったか？
```

答えられないなら、まだ簡素化の準備ができていません。先に文脈を読んでください。

### Step 2: 簡素化の機会を見つける

次のパターンを探します - それぞれ具体的なシグナルです。

**構造的複雑さ:**

| パターン | シグナル | 簡素化 |
|---|---|---|
| 深いネスト（3 層以上） | 制御フローが追いにくい | guard clause や helper に分ける |
| 長い関数（50 行以上） | 複数の責務がある | 焦点の合った関数に分割する |
| 入れ子の三項演算子 | 理解にメンタルスタックが必要 | if/else、switch、lookup object に置き換える |
| boolean 引数フラグ | `doThing(true, false, true)` | options object か別関数にする |
| 条件分岐の繰り返し | 同じ `if` が複数箇所にある | 分かりやすい predicate 関数に抽出する |

**命名と可読性:**

| パターン | シグナル | 簡素化 |
|---|---|---|
| 一般的な名前 | `data`, `result`, `temp`, `val`, `item` | 中身が分かる名前に変更: `userProfile`, `validationErrors` |
| 略称 | `usr`, `cfg`, `btn`, `evt` | 普遍的な略語（`id`, `url`, `api`）以外は完全な単語を使う |
| 誤解を招く名前 | 状態を変えるのに `get` と名付ける | 実際の振る舞いに合わせて rename する |
| "何を" を説明するコメント | `// increment counter` の上に `count++` | コメントを消す - コードで十分 |
| "なぜ" を説明するコメント | `// API が負荷時に不安定なので retry する` | 残す - コードでは表しにくい意図だから |

**冗長性:**

| パターン | シグナル | 簡素化 |
|---|---|---|
| 重複ロジック | 同じ 5 行以上が複数箇所にある | 共有関数に抽出する |
| 死んだコード | 到達不能な branch、未使用変数、コメントアウトされた塊 | 本当に死んでいると確認したら削除する |
| 不要な抽象化 | 価値のない wrapper | wrapper を inline して直接呼ぶ |
| 過剰設計 | factory のための factory、strategy が 1 個しかない | 単純な直書きに置き換える |
| 冗長な type assertion | すでに推論できる型への cast | assertion を消す |

### Step 3: 1 つずつ変更する

簡素化は 1 つずつ行います。1 つ変えるたびにテストを回してください。**リファクタリング変更は、feature や bug fix と分けて提出します。** リファクタリングと機能追加を混ぜた PR は 2 つに分けます。

```text
各簡素化で:
1. 変更する
2. テストスイートを回す
3. テストが通る -> commit（または次の簡素化へ）
4. テストが落ちる -> 戻して再考
```

複数の簡素化を一度にまとめて未検証で入れないでください。壊れたら、どの簡素化が原因か分からなくなります。

**Rule of 500:** 500 行以上触るリファクタリングなら、手でやるより codemod、sed スクリプト、AST transform のような自動化に投資してください。手作業の大規模編集はレビューも大変でミスも増えます。

### Step 4: 結果を検証する

簡素化が終わったら、全体を見ます。

```text
BEFORE / AFTER を比較:
- 簡素版のほうが本当に理解しやすいか？
- コードベースと合わない新しいパターンを入れていないか？
- diff はきれいでレビューしやすいか？
- チームメイトならこの変更を承認するか？
```

もし "簡素化" した版のほうが理解しにくい、レビューしにくいなら、戻します。すべての簡素化が成功するわけではありません。

## 言語別ガイダンス

### TypeScript / JavaScript

```typescript
// 簡素化: 不要な async wrapper
// Before
async function getUser(id: string): Promise<User> {
  return await userService.findById(id);
}
// After
function getUser(id: string): Promise<User> {
  return userService.findById(id);
}

// 簡素化: 長い条件付き代入
// Before
let displayName: string;
if (user.nickname) {
  displayName = user.nickname;
} else {
  displayName = user.fullName;
}
// After
const displayName = user.nickname || user.fullName;

// 簡素化: 手書き配列構築
// Before
const activeUsers: User[] = [];
for (const user of users) {
  if (user.isActive) {
    activeUsers.push(user);
  }
}
// After
const activeUsers = users.filter((user) => user.isActive);

// 簡素化: 冗長な boolean return
// Before
function isValid(input: string): boolean {
  if (input.length > 0 && input.length < 100) {
    return true;
  }
  return false;
}
// After
function isValid(input: string): boolean {
  return input.length > 0 && input.length < 100;
}
```

### Python

```python
# 簡素化: 辞書構築の冗長さ
# Before
result = {}
for item in items:
    result[item.id] = item.name
# After
result = {item.id: item.name for item in items}

# 簡素化: 深い条件分岐を早期 return にする
# Before
def process(data):
    if data is not None:
        if data.is_valid():
            if data.has_permission():
                return do_work(data)
            else:
                raise PermissionError("No permission")
        else:
            raise ValueError("Invalid data")
    else:
        raise TypeError("Data is None")
# After
def process(data):
    if data is None:
        raise TypeError("Data is None")
    if not data.is_valid():
        raise ValueError("Invalid data")
    if not data.has_permission():
        raise PermissionError("No permission")
    return do_work(data)
```

### React / JSX

```tsx
// 簡素化: 条件付きレンダリングの冗長さ
// Before
function UserBadge({ user }: Props) {
  if (user.isAdmin) {
    return <Badge variant="admin">Admin</Badge>;
  } else {
    return <Badge variant="default">User</Badge>;
  }
}
// After
function UserBadge({ user }: Props) {
  const variant = user.isAdmin ? 'admin' : 'default';
  const label = user.isAdmin ? 'Admin' : 'User';
  return <Badge variant={variant}>{label}</Badge>;
}

// 簡素化: 中間コンポーネントをまたぐ prop drilling
// Before - これは context や composition のほうが良いか考える
// これは判断案件です - 自動でリファクタリングしない
```

## よくある言い訳

| 言い訳 | 実際 |
|---|---|
| 「動いているから触らなくていい」 | 読みにくい code は、壊れたときに直しにくいです。今簡素化するほうが、今後の変更すべてで得になります。 |
| 「少ない行数ほど簡単」 | 1 行の入れ子三項演算子は、5 行の if/else より簡単とは限りません。シンプルさは行数ではなく、理解の速さです。 |
| 「ついでに関係ないコードも簡素化する」 | 範囲外の簡素化は diff を汚し、触る予定のなかった code に回帰を生みます。集中してください。 |
| 「型があるから self-documenting」 | 型は構造を説明しても、意図は説明しません。よく名付けられた関数のほうが、型シグネチャより "なぜ" を伝えます。 |
| 「この抽象化は将来使えるかもしれない」 | speculative な抽象化を残さないでください。今使っていないなら、価値のない複雑さです。消して、必要になったら再追加します。 |
| 「元の作者には理由があったはず」 | そうかもしれません。`git blame` を見てください - Chesterton's Fence を適用します。ただし、積み重なった複雑さには理由がないことも多いです。単なる圧力下での反復の残滓です。 |
| 「この feature に合わせて refactor する」 | リファクタリングと機能作業は分けてください。混ざった変更はレビューも revert も履歴理解も難しくなります。 |

## レッドフラグ

- 簡素化のためにテストを変えないと通らない（振る舞いを変えた可能性が高い）
- "簡素化" 後のコードが元より長く、追いにくい
- 自分の好みを project conventions より優先して rename する
- "コードがきれいになるから" と error handling を削る
- 十分に理解していないコードを簡素化する
- 多数の簡素化を 1 つの大きな、レビューしにくい commit にまとめる
- 現在のタスク範囲外のコードを、頼まれてもいないのにリファクタリングする

## 検証

簡素化パスを終えたら、次を確認します。

- [ ] 既存テストが変更なしで通る
- [ ] build が新しい警告なしで通る
- [ ] linter / formatter が通る（style regression がない）
- [ ] 各簡素化はレビュー可能な、増分の変更になっている
- [ ] diff がきれい - 関係ない変更が混ざっていない
- [ ] 簡素化後のコードが project conventions に従っている（CLAUDE.md などと照合）
- [ ] error handling を削除・弱体化していない
- [ ] dead code が残っていない（未使用 import、到達不能 branch など）
- [ ] チームメイトかレビューエージェントが、この変更を net improvement として承認するはず

---
title: RSC の props で重複シリアライズを避ける
impact: LOW
impactDescription: 重複シリアライズを避けてネットワーク負荷を減らす
tags: server, rsc, serialization, props, client-components
---

## RSC の props で重複シリアライズを避ける

**Impact: LOW (重複シリアライズを避けてネットワーク負荷を減らす)**

RSC と client の境界では、すべてのオブジェクトプロパティが文字列化され、HTML レスポンスとその後の RSC リクエストに埋め込まれます。このシリアライズ済みデータはページサイズと読み込み時間に直接影響するため、サイズは非常に重要です。クライアントが実際に使うフィールドだけを渡してください。

**悪い例（配列を 2 回シリアライズする）:**

```tsx
// RSC: 6 文字列を送る（2 配列 × 3 項目）
<ClientList usernames={usernames} usernamesOrdered={usernames.toSorted()} />
```

**良い例（3 文字列だけ送る）:**

```tsx
// RSC: 1 回だけ送る
<ClientList usernames={usernames} />

// Client: 変換はクライアント側で行う
'use client'
const sorted = useMemo(() => [...usernames].sort(), [usernames])
```

**入れ子の重複排除の挙動:**

重複排除は再帰的に働きます。影響の大きさはデータ型によって異なります。

- `string[]`, `number[]`, `boolean[]`: **影響大** - 配列とすべてのプリミティブが完全に重複する
- `object[]`: **影響小** - 配列は重複するが、入れ子のオブジェクトは参照で重複排除される

```tsx
// string[] - すべて重複する
usernames={['a','b']} sorted={usernames.toSorted()} // 4 文字列を送る

// object[] - 配列構造だけが重複する
users={[{id:1},{id:2}]} sorted={users.toSorted()} // 2 配列 + 2 個の一意なオブジェクトを送る（4 ではない）
```

**重複排除を壊す操作（新しい参照を作る）:**

- 配列: `.toSorted()`, `.filter()`, `.map()`, `.slice()`, `[...arr]`
- オブジェクト: `{...obj}`, `Object.assign()`, `structuredClone()`, `JSON.parse(JSON.stringify())`

**その他の例:**

```tsx
// 悪い例
<C users={users} active={users.filter(u => u.active)} />
<C product={product} productName={product.name} />

// 良い例
<C users={users} />
<C product={product} />
// フィルタリングや分割代入はクライアント側で行う
```

**例外:** 変換が高コストな場合や、クライアントが元データを必要としない場合は、派生データを渡しても構いません。

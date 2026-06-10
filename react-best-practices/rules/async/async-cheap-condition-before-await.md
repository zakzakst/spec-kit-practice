---
title: 非同期フラグより先に安価な条件を確認する
impact: HIGH
impactDescription: 同期的なガードですでに失敗している場合に不要な非同期処理を避ける
tags: async, await, feature-flags, short-circuit, conditional
---

## 非同期フラグより先に安価な条件を確認する

分岐でフラグやリモート値に `await` を使いつつ、**安価な同期条件**（ローカル props、リクエストメタデータ、すでに読み込まれた state など）も必要な場合は、安価な条件を**先に**評価してください。そうしないと、複合条件が絶対に真にならない場合でも非同期呼び出しのコストを支払うことになります。

これは [Defer Await Until Needed](./async-defer-await.md) を `flag && cheapCondition` 形式のチェック向けに特化したものです。

**誤り:**

```typescript
const someFlag = await getFlag()

if (someFlag && someCondition) {
  // ...
}
```

**正しい例:**

```typescript
if (someCondition) {
  const someFlag = await getFlag()
  if (someFlag) {
    // ...
  }
}
```

これは、`getFlag` がネットワーク、フィーチャーフラグサービス、あるいは `React.cache` / DB 処理に触れる場合に特に重要です。`someCondition` が false のときにそれを省略できれば、コールドパスでのコストを削減できます。

ただし、`someCondition` 自体が高コストな場合、フラグに依存する場合、または副作用を固定順で実行する必要がある場合は、元の順序を維持してください。

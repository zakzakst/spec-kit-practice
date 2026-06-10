---
title: SWR で自動デデュープを使う
impact: MEDIUM-HIGH
impactDescription: 自動で重複排除する
tags: client, swr, deduplication, data-fetching
---

## SWR で自動デデュープを使う

SWR は、コンポーネントインスタンス間でのリクエスト重複排除、キャッシュ、再検証を可能にします。

**誤り（重複排除なし、各インスタンスが個別に取得する）:**

```tsx
function UserList() {
  const [users, setUsers] = useState([])
  useEffect(() => {
    fetch('/api/users')
      .then(r => r.json())
      .then(setUsers)
  }, [])
}
```

**正解（複数インスタンスで 1 回のリクエストを共有する）:**

```tsx
import useSWR from 'swr'

function UserList() {
  const { data: users } = useSWR('/api/users', fetcher)
}
```

**変更されないデータの場合:**

```tsx
import { useImmutableSWR } from '@/lib/swr'

function StaticContent() {
  const { data } = useImmutableSWR('/api/config', fetcher)
}
```

**更新処理の場合:**

```tsx
import { useSWRMutation } from 'swr/mutation'

function UpdateButton() {
  const { trigger } = useSWRMutation('/api/user', updateUser)
  return <button onClick={() => trigger()}>Update</button>
}
```

参考: [https://swr.vercel.app](https://swr.vercel.app)

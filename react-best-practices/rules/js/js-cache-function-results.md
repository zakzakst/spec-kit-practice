---
title: 繰り返しの関数呼び出しをキャッシュする
impact: MEDIUM
impactDescription: 冗長な計算を避ける
tags: javascript, cache, memoization, performance
---

## 繰り返しの関数呼び出しをキャッシュする

レンダリング中に同じ関数が同じ入力で何度も呼ばれる場合は、モジュールレベルの `Map` を使って結果をキャッシュしてください。

**誤り（冗長な計算）:**

```typescript
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // 同じ project 名に対して slugify() が 100 回以上呼ばれる
        const slug = slugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**正しい例（結果をキャッシュする）:**

```typescript
// モジュールレベルのキャッシュ
const slugifyCache = new Map<string, string>()

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!
  }
  const result = slugify(text)
  slugifyCache.set(text, result)
  return result
}

function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // 一意な project 名ごとに 1 回だけ計算される
        const slug = cachedSlugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**単一値を返す関数向けの、より簡単なパターン:**

```typescript
let isLoggedInCache: boolean | null = null

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache
  }
  
  isLoggedInCache = document.cookie.includes('auth=')
  return isLoggedInCache
}

// 認証状態が変わったらキャッシュをクリアする
function onAuthChange() {
  isLoggedInCache = null
}
```

React コンポーネントだけでなく、ユーティリティやイベントハンドラなどどこでも使えるように、フックではなく `Map` を使ってください。

参考: [How we made the Vercel Dashboard twice as fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)

---
title: localStorage データにバージョンを付けて最小化する
impact: MEDIUM
impactDescription: スキーマ衝突を防ぎ、保存容量を削減
tags: client, localStorage, storage, versioning, data-minimization
---

## localStorage データにバージョンを付けて最小化する

キーにバージョン接頭辞を付け、必要なフィールドだけを保存します。これにより、スキーマ衝突や機密データの誤保存を防げます。

**誤り:**

```typescript
// バージョンがなく、すべてを保存し、エラーハンドリングもない
localStorage.setItem('userConfig', JSON.stringify(fullUserObject))
const data = localStorage.getItem('userConfig')
```

**正解:**

```typescript
const VERSION = 'v2'

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config))
  } catch {
    // シークレット/プライベートブラウジング、容量超過、無効化時には例外が発生する
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`)
    return data ? JSON.parse(data) : null
  } catch {
    return null
  }
}

// v1 から v2 への移行
function migrate() {
  try {
    const v1 = localStorage.getItem('userConfig:v1')
    if (v1) {
      const old = JSON.parse(v1)
      saveConfig({ theme: old.darkMode ? 'dark' : 'light', language: old.lang })
      localStorage.removeItem('userConfig:v1')
    }
  } catch {}
}
```

**サーバーレスポンスからは最小限のフィールドだけを保存する:**

```typescript
// User オブジェクトには 20 個以上のフィールドがあるため、UI に必要なものだけ保存する
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem('prefs:v1', JSON.stringify({
      theme: user.preferences.theme,
      notifications: user.preferences.notifications
    }))
  } catch {}
}
```

**必ず try-catch で囲む:** `getItem()` と `setItem()` は、シークレット/プライベートブラウジング（Safari, Firefox）、容量超過、無効化時に例外を投げます。

**利点:** バージョン管理によるスキーマ進化、保存容量の削減、トークン/PII/内部フラグの保存防止。

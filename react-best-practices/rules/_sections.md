# セクション

このファイルでは、すべてのセクション、その順序、影響度、説明を定義します。
括弧内のセクション ID は、ルールをグループ化するために使うファイル名の接頭辞です。

---

## 1. Waterfalls の排除 (async)

**Impact:** CRITICAL  
**Description:** Waterfalls は最悪の性能低下要因です。逐次的な await は毎回フルのネットワーク遅延を追加します。これをなくすことが最大の改善につながります。

## 2. バンドルサイズの最適化 (bundle)

**Impact:** CRITICAL  
**Description:** 初期バンドルサイズを小さくすると、Time to Interactive と Largest Contentful Paint が改善します。

## 3. サーバーサイド性能 (server)

**Impact:** HIGH  
**Description:** サーバーサイドレンダリングとデータ取得を最適化すると、サーバー側の waterfall を減らし、応答時間を短縮できます。

## 4. クライアントサイドのデータ取得 (client)

**Impact:** MEDIUM-HIGH  
**Description:** 自動重複排除と効率的なデータ取得パターンにより、無駄なネットワークリクエストを減らせます。

## 5. 再レンダー最適化 (rerender)

**Impact:** MEDIUM  
**Description:** 不要な再レンダーを減らすことで、無駄な計算を抑え、UI の応答性を高めます。

## 6. レンダリング性能 (rendering)

**Impact:** MEDIUM  
**Description:** レンダリング処理を最適化すると、ブラウザが行う作業を減らせます。

## 7. JavaScript 性能 (js)

**Impact:** LOW-MEDIUM  
**Description:** ホットパスに対するマイクロ最適化は、積み重なると意味のある改善になります。

## 8. 高度なパターン (advanced)

**Impact:** LOW  
**Description:** 注意深い実装が必要な、特定ケース向けの高度なパターンです。

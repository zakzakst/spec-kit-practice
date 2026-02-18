## 1. セットアップ

- [ ] 1.1 `frontend/src/components/LoginForm.tsx` コンポーネント ファイルを作成する
- [ ] 1.2 トークンの保存と状態のための認証ユーティリティ `frontend/src/hooks/useAuth.ts` を追加します。
- [ ] 1.3 `frontend/src/types/auth.ts` を更新して `AuthToken` 型定義を追加します
- [ ] 1.4 アプリケーション ルーター構成 (`frontend/src/App.tsx` または同等のもの) にルート `/login` を追加します。

## 2. ログインフォームと検証

- [ ] 2.1 `LoginForm.tsx` のユーザー名/メール アドレスとパスワード フィールドに JSX を実装します。
- [ ] 2.2 `LoginForm.tsx` にクライアント側の検証ロジックを追加し、空のフィールドでの送信を防ぎ、インラインエラーを表示します。
- [ ] 2.3 `LoginForm.tsx` に送信ボタンと読み込みインジケーターを追加します。

## 3. 認証フロー

- [ ] 3.1 `useAuth.ts` に `login` 関数を実装し、`/api/login` に資格情報を投稿してレスポンスを処理します。
- [ ] 3.2 返されたトークンを `useAuth.ts` 内のキー `authToken` の下の `localStorage` に保存します。
- [ ] 3.3 `useAuth` を更新して `isAuthenticated`、`login`、`logout` メソッドを提供します
- [ ] 3.4 `LoginForm.tsx` で認証エラーを処理し、失敗した場合はバナーメッセージを表示します。

## 4. ルーティング保護

- [ ] 4.1 `RequireAuth` 上位コンポーネントまたはフックを作成し、`isAuthenticated` をチェックして false の場合は `/login` にリダイレクトします。
- [ ] 4.2 ルータ設定で、保護されたルートコンポーネント（例：`Dashboard`、`Profile`）を `RequireAuth` でラップします。
- [ ] 4.3 リダイレクトループを回避するために、`/login` ルートが保護ロジックから除外されていることを確認します。

## 5. テスト

- [ ] 5.1 フォームフィールドと送信の動作を検証する `LoginForm.tsx` のユニットテストを記述します (frontend/tests/unit/login-form.spec.tsx)
- [ ] 5.2 トークンの保存と認証状態の変更を検証する `useAuth.ts` の単体テストを追加します。
- [ ] 5.3 ログイン成功、無効な資格情報、リダイレクト動作をカバーする Playwright e2e テスト `frontend/tests/e2e/login-flow.spec.ts` を作成します。

## 6. 施策とドキュメント

- [ ] 6.1 認証関連のテストを実行するための手順を `frontend/README.md` に更新します
- [ ] 6.2 ログイン画面を説明するドキュメント スニペットを `specs/login-screen/quickstart.md` (またはグローバル クイックスタート) に追加します。
- [ ] 6.3 新しいファイル間でリンティングとフォーマットを確実に実行

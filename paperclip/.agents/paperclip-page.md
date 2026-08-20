---
name: paperclip-page
description: >
  静的HTMLページとアセットフォルダを Paperclip の S3/CloudFront ページ
  ホストへ公開します。永続的なページ、ビューア、プロトタイプ、レポート、
  静的サイトを here.now なしでデプロイ・ホスト・共有したいときに使います。
---

# Paperclip Page

この skill は、設定済みの Paperclip Pages ホストへ静的ディレクトリを
公開するために使います。例: `https://pages.paperclip.ing/<slug>/`。

## Requirements

- ソースディレクトリのルートに `index.html` があること。
- ライブ公開には `aws` CLI v2、`curl`、`jq` が PATH 上で使えること。
- 以下の環境変数が設定されていること:
  - `PAPERCLIP_PAGE_BUCKET`
  - `PAPERCLIP_PAGE_BASE_URL`
  - `AWS_REGION`
  - `PAPERCLIP_PAGE_AWS_ACCESS_KEY_ID` and `PAPERCLIP_PAGE_AWS_SECRET_ACCESS_KEY`
    Paperclip Secrets の page-uploader 認証情報を使う
- 任意の環境変数:
  - `PAPERCLIP_PAGE_DEFAULT_PREFIX`
  - `PAPERCLIP_PAGE_AWS_PROFILE`（名前空間付きキーの代替）
  - `PAPERCLIP_PAGE_AWS_SESSION_TOKEN`（名前空間付きキーと併用）

page-uploader の認証情報をグローバルな `AWS_ACCESS_KEY_ID` /
`AWS_SECRET_ACCESS_KEY` として設定しないでください。静的な環境変数キーは
AWS SDK 全体で `AWS_PROFILE` より優先されるため、グローバル名を使うと
エージェント実行中の全プロセスでホストの ID が静かに置き換わってしまいます。
名前空間付き変数は uploader の ID をこの helper のみに閉じ込めます。
`PAPERCLIP_PAGE_AWS_*` の認証変数がない場合でも、通常の認証チェーンは
フォールバックとして動作します。

## Workflow

1. ソースディレクトリを確認し、公開可能な静的コンテンツだけであることを
   確認します。
2. `scripts/publish.sh <dir> --dry-run` を実行し、ローカル構成を検証して
   解決された URL / prefix を確認します。
3. Choose a slug:
   - ユーザーが安定した URL パスを指定したなら `--slug <slug>` を使います。
   - `--slug` を省略すると、ソースディレクトリ名から自動生成されます。
4. Publish:

```bash
.agents/skills/paperclip-page/scripts/publish.sh ./site --slug my-page
```

5. 表示された公開 URL と S3 prefix を issue / ユーザーへ返します。

## Update Workflow

更新は追記的な上書きのみです。この helper はリモートオブジェクトを
削除しません。

```bash
.agents/skills/paperclip-page/scripts/publish.sh ./site --slug my-page --update
```

対象 prefix がすでに存在する場合、`--update` には同じソースディレクトリから
以前に公開した際に生成された `./site/.paperclip-page/state.json` による
ローカル所有証明が必要です。その state がない場合は、別のページを
上書きするのではなく新しい slug を作成してください。

## Safety Rules

- 公開してよいコンテンツだけを公開してください。秘密情報、顧客データ、
  社内資料、認証情報、内部ログは公開しないでください。
- AWS のシークレット値は決して出力しないでください。
- この skill から bucket policy、IAM、DNS、CloudFront、ACM の設定を
  変更しないでください。セットアップは operator の runbook の担当です。
- 設定済みの bucket と prefix 以外へアップロードしないでください。
- `aws s3 sync --delete` は使わず、v1 では `s3:DeleteObject` を要求しないでください。
- helper は `--no-follow-symlinks` を強制し、ソースに symlink があると失敗します。
- helper は自身の `.paperclip-page/state.json` を除き、hidden file と
  dot-segment path を拒否します。
- slug と prefix の各 segment は、小文字 ASCII 英字・数字・ハイフンのみです。
- `404.html` のようなサイト全体の root object は operator 管理のままにし、
  公開は常に `<slug>/...` または `<default-prefix>/<slug>/...` を対象にします。

## Troubleshooting

- `Slug already exists`: 別の slug を選ぶか、`.paperclip-page/state.json` を
  含む元のソースディレクトリから `--update` を使ってください。
- `Missing index.html`: 先に静的サイトをビルドするか、ルート HTML ファイルを
  含むディレクトリを helper に指定してください。
- `Found symlink`: 公開前に symlink を実ファイルへ置き換えてください。
- `AccessDenied`: 設定済みの bucket / prefix に対して uploader の IAM policy が
  `ListBucket`、`GetObject`、`PutObject` を許可していること、および
  エージェントが Paperclip Secrets を受け取っていることを確認してください。
- 公開 URL の検証に失敗した: CloudFront のデプロイ / DNS、オブジェクトの存在、
  配信設定が private S3 REST origin への HTTPS を使っているか確認してください。

operator のセットアップ、AWS policy の例、認証情報ローテーション、install / attach
コマンドについては、この skill と並んでいる `README.md` を参照してください。

---
name: paperclip-create-plugin
description: >
  CLI-first のワークフローで外部 Paperclip plugin を作成・開発します。
  plugin の scaffold、ローカル plugin の反復、Paperclip への install、plugin authoring docs の更新に使います。
---

# Paperclip plugin を作成・開発する

Paperclip のローカル instance に対して plugin を作成・scaffold・反復するときに使う skill です。

## 1. 既定: plugin は Paperclip core の外で作る

plugin はそれぞれ独立した package です。タスクが **明示的に** in-repo の bundled example を求めていない限り、この repo の `packages/plugins/` 配下に plugin source を追加しないでください。

- plugin は Paperclip checkout の外側の directory（例: `~/dev/paperclip-plugins/<name>`）に scaffold します。
- local absolute path で running Paperclip instance に install します。
- code は外部 package 側を編集し、Paperclip に rebuild 済み output を拾わせます。

Paperclip core 自体を編集するのは、user が plugin を bundled example として見せたいと明示した場合だけです（`server/src/routes/plugins.ts`、in-repo example list、docs など）。

## 2. 基本ルール

必要なときは次の docs を参照してください:

1. `doc/plugins/PLUGIN_AUTHORING_GUIDE.md`
2. `packages/plugins/sdk/README.md`
3. `doc/plugins/PLUGIN_SPEC.md` - 将来の context としてのみ

現在の runtime 前提:

- plugin workers は trusted code
- plugin UI は trusted same-origin host code
- worker API は capability-gated
- plugin UI は manifest capabilities で sandbox されていない
- host が提供する shared plugin UI component kit はまだない
- `ctx.assets` は現在の runtime ではサポートされていない

## 3. CLI-first の scaffold ワークフロー

`paperclipai plugin init` を使います。CLI command が使えない環境でない限り、scaffold package の node entrypoint を手で呼ばないでください。

```bash
paperclipai plugin init @acme/my-plugin --output ~/dev/paperclip-plugins
```

便利な flags（すべて任意）:

- `--output <dir>` - 親 directory。command は `<dir>/<unscoped-name>/` を作成します。既定は current directory。
- `--template <default|connector|workspace|environment>` - starter template。
- `--category <connector|workspace|automation|ui|environment>` - manifest category。
- `--display-name <name>`、`--description <text>`、`--author <name>` - manifest metadata。
- `--sdk-path <path>` - local SDK を Paperclip checkout から `.paperclip-sdk/` に snapshot します（未公開 SDK を使って開発するときに便利です）。

成功すると command は次に実行すべき正確な command（`cd`、`pnpm install`、`pnpm dev`、`paperclipai plugin install <abs-path>`）を出力します。順番に実行してください。

もし `paperclipai` が PATH 上にない場合は、次に fallback します:

```bash
pnpm --filter @paperclipai/create-paperclip-plugin build
node packages/plugins/create-paperclip-plugin/dist/index.js @acme/my-plugin \
  --output /absolute/path \
  --sdk-path /absolute/path/to/paperclip/packages/plugins/sdk
```

## 4. local install + rebuild loop

scaffold 済みの plugin folder で:

```bash
pnpm install
pnpm dev            # esbuild --watch: dist/manifest.js, dist/worker.js, dist/ui/ を再build
paperclipai plugin install /absolute/path/to/my-plugin
```

備考:

- `paperclipai plugin install` は local path（absolute、`./`、`../`、`~`、または既存の relative folder）を自動検出し、`isLocalPath: true` を server に渡します。heuristic が曖昧なら `--local` で local mode を強制できます。
- path は server に送る前に absolute path に解決されます。

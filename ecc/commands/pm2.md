---
description: プロジェクトを分析し、検出した frontend / backend / database サービス向けの PM2 コマンドを生成します。
---

# PM2 初期化

プロジェクトを自動分析し、PM2 サービスコマンドを生成します。

**Command**: `$ARGUMENTS`

---

## ワークフロー

1. PM2 を確認する（なければ `npm install -g pm2` で導入）
2. プロジェクトを走査してサービス（frontend / backend / database）を特定する
3. 設定ファイルと個別コマンドファイルを生成する

---

## サービス検出

| Type          | Detection                                   | Default Port |
| ------------- | ------------------------------------------- | ------------ |
| Vite          | vite.config.*                               | 5173         |
| Next.js       | next.config.*                               | 3000         |
| Nuxt          | nuxt.config.*                               | 3000         |
| CRA           | package.json 内の react-scripts             | 3000         |
| Express/Node  | server/backend/api ディレクトリ + package.json | 3000      |
| FastAPI/Flask | requirements.txt / pyproject.toml           | 8000         |
| Go            | go.mod / main.go                            | 8080         |

**ポート検出の優先順**: ユーザー指定 > `.env` > 設定ファイル > script 引数 > デフォルトポート

---

## 生成されるファイル

```text
project/
├─ ecosystem.config.cjs              # PM2 設定
├─ {backend}/start.cjs               # Python ラッパー（必要な場合）
└─ .claude/
   ├─ commands/
   │  ├─ pm2-all.md                  # 全起動 + monit
   │  ├─ pm2-all-stop.md             # 全停止
   │  ├─ pm2-all-restart.md          # 全再起動
   │  ├─ pm2-{port}.md               # 単体起動 + logs
   │  ├─ pm2-{port}-stop.md          # 単体停止
   │  ├─ pm2-{port}-restart.md       # 単体再起動
   │  ├─ pm2-logs.md                 # 全ログ表示
   │  └─ pm2-status.md               # 状態表示
   └─ scripts/
      ├─ pm2-logs-{port}.ps1         # 単体サービスのログ
      └─ pm2-monit.ps1               # PM2 モニター
```

---

## Windows 向け設定（重要）

### ecosystem.config.cjs

**拡張子は必ず `.cjs` を使うこと**

```javascript
module.exports = {
  apps: [
    // Node.js (Vite/Next/Nuxt)
    {
      name: "project-3000",
      cwd: "./packages/web",
      script: "node_modules/vite/bin/vite.js",
      args: "--port 3000",
      interpreter: "C:/Program Files/nodejs/node.exe",
      env: { NODE_ENV: "development" },
    },
    // Python
    {
      name: "project-8000",
      cwd: "./backend",
      script: "start.cjs",
      interpreter: "C:/Program Files/nodejs/node.exe",
      env: { PYTHONUNBUFFERED: "1" },
    },
  ],
};
```

**フレームワークごとの script パス:**

| Framework | script                            | args                |
| --------- | --------------------------------- | ------------------- |
| Vite      | `node_modules/vite/bin/vite.js`   | `--port {port}`     |
| Next.js   | `node_modules/next/dist/bin/next` | `dev -p {port}`     |
| Nuxt      | `node_modules/nuxt/bin/nuxt.mjs`  | `dev --port {port}` |
| Express   | `src/index.js` or `server.js`     | -                   |

### Python ラッパースクリプト（start.cjs）

```javascript
const { spawn } = require("child_process");
const proc = spawn(
  "python",
  [
    "-m",
    "uvicorn",
    "app.main:app",
    "--host",
    "0.0.0.0",
    "--port",
    "8000",
    "--reload",
  ],
  {
    cwd: __dirname,
    stdio: "inherit",
    windowsHide: true,
  },
);
proc.on("close", (code) => process.exit(code));
```

---

## コマンドファイルテンプレート（最小構成）

### pm2-all.md（全起動 + monit）

````markdown
全サービスを起動し、PM2 モニターを開きます。

```bash
cd "{PROJECT_ROOT}" && pm2 start ecosystem.config.cjs && start wt.exe -d "{PROJECT_ROOT}" pwsh -NoExit -c "pm2 monit"
```
````

### pm2-all-stop.md

````markdown
全サービスを停止します。

```bash
cd "{PROJECT_ROOT}" && pm2 stop all
```
````

### pm2-all-restart.md

````markdown
全サービスを再起動します。

```bash
cd "{PROJECT_ROOT}" && pm2 restart all
```
````

### pm2-{port}.md（単体起動 + logs）

````markdown
{name}（{port}）を起動し、ログを開きます。

```bash
cd "{PROJECT_ROOT}" && pm2 start ecosystem.config.cjs --only {name} && start wt.exe -d "{PROJECT_ROOT}" pwsh -NoExit -c "pm2 logs {name}"
```
````

### pm2-{port}-stop.md

````markdown
{name}（{port}）を停止します。

```bash
cd "{PROJECT_ROOT}" && pm2 stop {name}
```
````

### pm2-{port}-restart.md

````markdown
{name}（{port}）を再起動します。

```bash
cd "{PROJECT_ROOT}" && pm2 restart {name}
```
````

### pm2-logs.md

````markdown
すべての PM2 ログを表示します。

```bash
cd "{PROJECT_ROOT}" && pm2 logs
```
````

### pm2-status.md

````markdown
PM2 の状態を表示します。

```bash
cd "{PROJECT_ROOT}" && pm2 status
```
````

### PowerShell スクリプト（pm2-logs-{port}.ps1）

```powershell
Set-Location "{PROJECT_ROOT}"
pm2 logs {name}
```

### PowerShell スクリプト（pm2-monit.ps1）

```powershell
Set-Location "{PROJECT_ROOT}"
pm2 monit
```

---

## 重要ルール

1. **設定ファイル**: `ecosystem.config.cjs` を使う（`.js` ではない）
2. **Node.js**: bin パスを直接指定し、interpreter も明示する
3. **Python**: Node.js ラッパースクリプトを使い、`windowsHide: true` を指定する
4. **新しいウィンドウを開く**: `start wt.exe -d "{path}" pwsh -NoExit -c "command"` を使う
5. **最小構成**: 各コマンドファイルは 1〜2 行の説明 + bash ブロックだけにする
6. **直接実行**: AI に解釈させず、そのまま bash コマンドを走らせる

---

## 実行

`$ARGUMENTS` に基づいて初期化を実行する:

1. プロジェクトを走査してサービスを検出する
2. `ecosystem.config.cjs` を生成する
3. Python サービスがある場合は `{backend}/start.cjs` を生成する
4. `.claude/commands/` にコマンドファイルを生成する
5. `.claude/scripts/` にスクリプトファイルを生成する
6. **プロジェクトの `CLAUDE.md` を PM2 情報で更新する**（後述）
7. **完了サマリー** とターミナルコマンドを表示する

---

## 初期化後: CLAUDE.md を更新する

ファイル生成後、プロジェクトの `CLAUDE.md` に PM2 セクションを追記する（存在しなければ作成）:

````markdown
## PM2 Services

| Port   | Name   | Type   |
| ------ | ------ | ------ |
| {port} | {name} | {type} |

**Terminal Commands:**

```bash
pm2 start ecosystem.config.cjs   # 初回
pm2 start all                    # 初回以降
pm2 stop all / pm2 restart all
pm2 start {name} / pm2 stop {name}
pm2 logs / pm2 status / pm2 monit
pm2 save                         # プロセス一覧を保存
pm2 resurrect                    # 保存済み一覧を復元
```
````

**`CLAUDE.md` 更新ルール:**

- PM2 セクションが既にあれば置き換える
- なければ末尾へ追記する
- 内容は最小限かつ重要事項だけにする

---

## 初期化後: サマリーを表示する

すべて生成したら、以下を出力する:

```text
## PM2 Init Complete

**Services:**

| Port | Name | Type |
|------|------|------|
| {port} | {name} | {type} |

**Claude Commands:** /pm2-all, /pm2-all-stop, /pm2-{port}, /pm2-{port}-stop, /pm2-logs, /pm2-status

**Terminal Commands:**
## First time (with config file)
pm2 start ecosystem.config.cjs && pm2 save

## After first time (simplified)
pm2 start all          # Start all
pm2 stop all           # Stop all
pm2 restart all        # Restart all
pm2 start {name}       # Start single
pm2 stop {name}        # Stop single
pm2 logs               # View logs
pm2 monit              # Monitor panel
pm2 resurrect          # Restore saved processes

**Tip:** 初回起動後に `pm2 save` を実行すると、以降は簡易コマンドが使えます。
```

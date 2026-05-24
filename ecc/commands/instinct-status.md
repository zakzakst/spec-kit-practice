---
name: instinct-status
description: 学習済みの instinct（プロジェクト + グローバル）を信頼度付きで表示する
command: true
---

# Instinct Status コマンド

現在のプロジェクト向け instinct とグローバル instinct を、ドメイン別にまとめて表示します。

## 実装

プラグインルートのパスを使って instinct CLI を実行します:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

または `CLAUDE_PLUGIN_ROOT` が設定されていない場合（手動インストール時）は:

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 使い方

```text
/instinct-status
```

## やること

1. 現在のプロジェクト文脈（git remote / path hash）を検出する
2. `~/.claude/homunculus/projects/<project-id>/instincts/` からプロジェクト instinct を読む
3. `~/.claude/homunculus/instincts/` からグローバル instinct を読む
4. 優先ルールに従ってマージする（ID 衝突時は project が global を上書き）
5. ドメインごとに、信頼度バーと観測統計付きで表示する

## 出力形式

```text
============================================================
  INSTINCT STATUS - 12 total
============================================================

  Project: my-app (a1b2c3d4e5f6)
  Project instincts: 8
  Global instincts:  4

## PROJECT-SCOPED (my-app)
  ### WORKFLOW (3)
    ████░  70%  grep-before-edit [project]
              trigger: when modifying code

## GLOBAL (apply to all projects)
  ### SECURITY (2)
    █████  85%  validate-user-input [global]
              trigger: when handling user input
```

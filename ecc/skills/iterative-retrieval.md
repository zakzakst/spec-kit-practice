---
name: iterative-retrieval
description: subagent の文脈問題を解くために、コンテキスト取得を段階的に洗練していくパターン
origin: ECC
---

# 反復的リトリーバルパターン

subagent が「作業を始めるまで、自分に何の文脈が必要か分からない」という multi-agent workflow の context problem を解決します。

## 有効化するタイミング

- upfront に必要文脈を予測できない subagent を起動するとき
- 文脈を段階的に洗練する multi-agent workflow を作るとき
- agent task で "context too large" や "missing context" にぶつかったとき
- コード探索向けの RAG 風 retrieval pipeline を設計するとき
- agent orchestration のトークン消費を最適化したいとき

## 問題

Subagent は限られた文脈で起動されるため、次を知らない:

- 関連コードがどのファイルにあるか
- コードベース内にどんなパターンがあるか
- プロジェクト固有の用語が何か

標準的なやり方は失敗しがち:

- **全部送る**: 文脈上限を超える
- **何も送らない**: 致命的情報が足りない
- **必要そうなものを勘で送る**: 外れやすい

## 解決策: Iterative Retrieval

文脈を段階的に絞り込む 4 フェーズループ:

```text
DISPATCH -> EVALUATE -> REFINE -> LOOP

最大 3 サイクル。十分な文脈が集まったら終了。
```

### Phase 1: DISPATCH

まずは広めの問い合わせで候補ファイルを集める:

```javascript
// Start with high-level intent
const initialQuery = {
  patterns: ["src/**/*.ts", "lib/**/*.ts"],
  keywords: ["authentication", "user", "session"],
  excludes: ["*.test.ts", "*.spec.ts"],
};

// Dispatch to retrieval agent
const candidates = await retrieveFiles(initialQuery);
```

### Phase 2: EVALUATE

取得した内容の関連度を評価する:

```javascript
function evaluateRelevance(files, task) {
  return files.map((file) => ({
    path: file.path,
    relevance: scoreRelevance(file.content, task),
    reason: explainRelevance(file.content, task),
    missingContext: identifyGaps(file.content, task),
  }));
}
```

採点基準:

- **High (0.8-1.0)**: 目的機能を直接実装している
- **Medium (0.5-0.7)**: 関連パターンや型を含む
- **Low (0.2-0.4)**: 周辺的に関係する
- **None (0-0.2)**: 無関係なので除外

### Phase 3: REFINE

評価結果にもとづいて検索条件を更新する:

```javascript
function refineQuery(evaluation, previousQuery) {
  return {
    // Add new patterns discovered in high-relevance files
    patterns: [...previousQuery.patterns, ...extractPatterns(evaluation)],

    // Add terminology found in codebase
    keywords: [...previousQuery.keywords, ...extractKeywords(evaluation)],

    // Exclude confirmed irrelevant paths
    excludes: [
      ...previousQuery.excludes,
      ...evaluation.filter((e) => e.relevance < 0.2).map((e) => e.path),
    ],

    // Target specific gaps
    focusAreas: evaluation.flatMap((e) => e.missingContext).filter(unique),
  };
}
```

### Phase 4: LOOP

洗練した条件で繰り返す（最大 3 サイクル）:

```javascript
async function iterativeRetrieve(task, maxCycles = 3) {
  let query = createInitialQuery(task);
  let bestContext = [];

  for (let cycle = 0; cycle < maxCycles; cycle++) {
    const candidates = await retrieveFiles(query);
    const evaluation = evaluateRelevance(candidates, task);

    // Check if we have sufficient context
    const highRelevance = evaluation.filter((e) => e.relevance >= 0.7);
    if (highRelevance.length >= 3 && !hasCriticalGaps(evaluation)) {
      return highRelevance;
    }

    // Refine and continue
    query = refineQuery(evaluation, query);
    bestContext = mergeContext(bestContext, highRelevance);
  }

  return bestContext;
}
```

## 実例

### Example 1: Bug Fix Context

```text
Task: "Fix the authentication token expiry bug"

Cycle 1:
  DISPATCH: Search for "token", "auth", "expiry" in src/**
  EVALUATE: Found auth.ts (0.9), tokens.ts (0.8), user.ts (0.3)
  REFINE: Add "refresh", "jwt" keywords; exclude user.ts

Cycle 2:
  DISPATCH: Search refined terms
  EVALUATE: Found session-manager.ts (0.95), jwt-utils.ts (0.85)
  REFINE: Sufficient context (2 high-relevance files)

Result: auth.ts, tokens.ts, session-manager.ts, jwt-utils.ts
```

### Example 2: Feature Implementation

```text
Task: "Add rate limiting to API endpoints"

Cycle 1:
  DISPATCH: Search "rate", "limit", "api" in routes/**
  EVALUATE: No matches - codebase uses "throttle" terminology
  REFINE: Add "throttle", "middleware" keywords

Cycle 2:
  DISPATCH: Search refined terms
  EVALUATE: Found throttle.ts (0.9), middleware/index.ts (0.7)
  REFINE: Need router patterns

Cycle 3:
  DISPATCH: Search "router", "express" patterns
  EVALUATE: Found router-setup.ts (0.8)
  REFINE: Sufficient context

Result: throttle.ts, middleware/index.ts, router-setup.ts
```

## Agent への組み込み

agent prompt では次のように使う:

```markdown
When retrieving context for this task:

1. Start with broad keyword search
2. Evaluate each file's relevance (0-1 scale)
3. Identify what context is still missing
4. Refine search criteria and repeat (max 3 cycles)
5. Return files with relevance >= 0.7
```

## ベストプラクティス

1. **広く始めて徐々に絞る** - 最初から絞り込みすぎない
2. **コードベースの用語を学ぶ** - 最初の周回で命名規則が見えることが多い
3. **不足文脈を追跡する** - 明示的な gap 把握が refinement を導く
4. **十分に良いところで止める** - mediocre な 10 ファイルより high-relevance な 3 ファイル
5. **自信を持って除外する** - relevance の低いものは後で急に relevant にはなりにくい

## 関連

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - subagent orchestration セクション
- `continuous-learning` skill - 時間とともに改善するパターン向け
- ECC に同梱される agent 定義（手動インストール時は `agents/`）

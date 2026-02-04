---
name: agentic-eval
description: |
  AIエージェントの出力を評価および改善するためのパターンとテクニック。このスキルは次のような場合に活用できます。
  - 自己批判と反省のループを実装する
  - 品質重視の生成のための評価-最適化パイプラインの構築
  - テスト駆動型コード改良ワークフローの作成
  - ルーブリックベースまたはLLMを審査員とする評価システムの設計
  - エージェントの出力（コード、レポート、分析）に反復的な改善を追加する
  - エージェントの応答品質の測定と改善
---

# エージェント評価パターン

反復的な評価と改良を通じて自己改善を図るパターン。

## 概要

評価パターンにより、エージェントは独自の出力を評価および改善することができ、単発生成を超えて反復的な改良ループに移行できます。

```
Generate → Evaluate → Critique → Refine → Output
    ↑                              │
    └──────────────────────────────┘
```

## いつ使うか

- **品質重視の生成**: 高精度が求められるコード、レポート、分析
- **明確な評価基準を持つタスク**: 明確な成功指標が存在する
- **特定の基準を必要とするコンテンツ**: スタイルガイド、コンプライアンス、フォーマット

---

## パターン1：基本的なリフレクション

エージェントは自己批評を通じて自身の出力を評価し、改善します。

```python
def reflect_and_refine(task: str, criteria: list[str], max_iterations: int = 3) -> str:
    """Generate with reflection loop."""
    output = llm(f"Complete this task:\n{task}")

    for i in range(max_iterations):
        # Self-critique
        critique = llm(f"""
        Evaluate this output against criteria: {criteria}
        Output: {output}
        Rate each: PASS/FAIL with feedback as JSON.
        """)

        critique_data = json.loads(critique)
        all_pass = all(c["status"] == "PASS" for c in critique_data.values())
        if all_pass:
            return output

        # Refine based on critique
        failed = {k: v["feedback"] for k, v in critique_data.items() if v["status"] == "FAIL"}
        output = llm(f"Improve to address: {failed}\nOriginal: {output}")

    return output
```

**重要な洞察**: 批評結果の信頼性の高い解析には、構造化された JSON 出力を使用します。

---

## パターン2: 評価者-最適化者

責任を明確にするために、生成と評価を別々のコンポーネントに分離します。

```python
class EvaluatorOptimizer:
    def __init__(self, score_threshold: float = 0.8):
        self.score_threshold = score_threshold

    def generate(self, task: str) -> str:
        return llm(f"Complete: {task}")

    def evaluate(self, output: str, task: str) -> dict:
        return json.loads(llm(f"""
        Evaluate output for task: {task}
        Output: {output}
        Return JSON: {{"overall_score": 0-1, "dimensions": {{"accuracy": ..., "clarity": ...}}}}
        """))

    def optimize(self, output: str, feedback: dict) -> str:
        return llm(f"Improve based on feedback: {feedback}\nOutput: {output}")

    def run(self, task: str, max_iterations: int = 3) -> str:
        output = self.generate(task)
        for _ in range(max_iterations):
            evaluation = self.evaluate(output, task)
            if evaluation["overall_score"] >= self.score_threshold:
                break
            output = self.optimize(output, evaluation)
        return output
```

---

## パターン3: コード固有のリフレクション

コード生成のためのテスト駆動型改良ループ。

```python
class CodeReflector:
    def reflect_and_fix(self, spec: str, max_iterations: int = 3) -> str:
        code = llm(f"Write Python code for: {spec}")
        tests = llm(f"Generate pytest tests for: {spec}\nCode: {code}")

        for _ in range(max_iterations):
            result = run_tests(code, tests)
            if result["success"]:
                return code
            code = llm(f"Fix error: {result['error']}\nCode: {code}")
        return code
```

---

## 評価戦略

### 成果に基づく

出力が期待された結果を達成したかどうかを評価します。

```python
def evaluate_outcome(task: str, output: str, expected: str) -> str:
    return llm(f"Does output achieve expected outcome? Task: {task}, Expected: {expected}, Output: {output}")
```

### 裁判官としてのLLM

LLM を使用して出力を比較し、ランク付けします。

```python
def llm_judge(output_a: str, output_b: str, criteria: str) -> str:
    return llm(f"Compare outputs A and B for {criteria}. Which is better and why?")
```

### ルーブリックベース

重み付けされたディメンションに対して出力をスコア付けします。

```python
RUBRIC = {
    "accuracy": {"weight": 0.4},
    "clarity": {"weight": 0.3},
    "completeness": {"weight": 0.3}
}

def evaluate_with_rubric(output: str, rubric: dict) -> float:
    scores = json.loads(llm(f"Rate 1-5 for each dimension: {list(rubric.keys())}\nOutput: {output}"))
    return sum(scores[d] * rubric[d]["weight"] for d in rubric) / 5
```

---

## ベストプラクティス

| 実践                 | 根拠                                                   |
| -------------------- | ------------------------------------------------------ |
| **明確な基準**       | 事前に具体的かつ測定可能な評価基準を定義する           |
| **反復制限**         | 無限ループを防ぐために最大反復回数（3～5）を設定します |
| **収束チェック**     | 反復間で出力スコアが改善されない場合は停止する         |
| **ログ履歴**         | デバッグと分析のために完全な軌跡を維持する             |
| **構造化された出力** | 評価結果の信頼性の高い解析には JSON を使用する         |

---

## クイックスタートチェックリスト

```markdown
## 評価実施チェックリスト

### 設定

- [ ] 評価基準/ルーブリックを定義する
- [ ] 「十分」のスコアのしきい値を設定する
- [ ] 最大反復回数を設定する（デフォルト：3）

### 実装

- [ ] generate() 関数を実装する
- [ ] 構造化された出力を持つevaluate()関数を実装する
- [ ] optimize() 関数を実装する
- [ ] 精製ループを接続する

### 安全性

- [ ] 収束検出を追加
- [ ] デバッグのためにすべての反復をログに記録する
- [ ] 評価解析の失敗を適切に処理する
```

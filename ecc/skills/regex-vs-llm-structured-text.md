---
name: regex-vs-llm-structured-text
description: 構造化テキストを解析する際に regex と LLM のどちらを選ぶかの判断フレームワーク。まず regex から始め、低信頼の端ケースだけに LLM を追加します。
origin: ECC
---

# 構造化テキスト解析における Regex vs LLM

クイズ、フォーム、請求書、文書などの構造化テキストを解析するための実践的な判断フレームワークです。核心は、ケースの 95〜98% は regex で安価かつ決定論的に処理できること。高価な LLM 呼び出しは残りの edge case にだけ使います。

## 有効化するタイミング

- 繰り返しパターンを持つ構造化テキストを解析するとき
- テキスト抽出で regex と LLM のどちらを使うか迷うとき
- 両者を組み合わせた hybrid pipeline を作るとき
- コスト / 精度のトレードオフを最適化したいとき

## 判断フレームワーク

```text
テキスト形式は一貫していて繰り返しか?
├─ Yes (>90% が同じパターン) -> まず Regex
│  ├─ Regex で 95%+ 処理できる -> 完了。LLM 不要
│  └─ Regex で <95% -> edge case にだけ LLM を足す
└─ No (自由形式で変動が大きい) -> 最初から LLM
```

## アーキテクチャパターン

```text
Source Text
    |
    v
[Regex Parser]          構造を抽出（95-98% 精度）
    |
    v
[Text Cleaner]          ノイズ除去（マーカー、ページ番号、アーティファクト）
    |
    v
[Confidence Scorer]     低信頼の抽出をフラグ
    |
    ├─ High confidence (>=0.95) -> そのまま出力
    └─ Low confidence (<0.95) -> [LLM Validator] -> 出力
```

## 実装

### 1. Regex Parser（大半を処理）

```python
import re
from dataclasses import dataclass

@dataclass(frozen=True)
class ParsedItem:
    id: str
    text: str
    choices: tuple[str, ...]
    answer: str
    confidence: float = 1.0

def parse_structured_text(content: str) -> list[ParsedItem]:
    """Parse structured text using regex patterns."""
    pattern = re.compile(
        r"(?P<id>\d+)\.\s*(?P<text>.+?)\n"
        r"(?P<choices>(?:[A-D]\..+?\n)+)"
        r"Answer:\s*(?P<answer>[A-D])",
        re.MULTILINE | re.DOTALL,
    )
    items = []
    for match in pattern.finditer(content):
        choices = tuple(
            c.strip() for c in re.findall(r"[A-D]\.\s*(.+)", match.group("choices"))
        )
        items.append(ParsedItem(
            id=match.group("id"),
            text=match.group("text").strip(),
            choices=choices,
            answer=match.group("answer"),
        ))
    return items
```

### 2. Confidence Scoring

LLM レビューが必要そうな item にフラグを立てる:

```python
@dataclass(frozen=True)
class ConfidenceFlag:
    item_id: str
    score: float
    reasons: tuple[str, ...]

def score_confidence(item: ParsedItem) -> ConfidenceFlag:
    """Score extraction confidence and flag issues."""
    reasons = []
    score = 1.0

    if len(item.choices) < 3:
        reasons.append("few_choices")
        score -= 0.3

    if not item.answer:
        reasons.append("missing_answer")
        score -= 0.5

    if len(item.text) < 10:
        reasons.append("short_text")
        score -= 0.2

    return ConfidenceFlag(
        item_id=item.id,
        score=max(0.0, score),
        reasons=tuple(reasons),
    )

def identify_low_confidence(
    items: list[ParsedItem],
    threshold: float = 0.95,
) -> list[ConfidenceFlag]:
    """Return items below confidence threshold."""
    flags = [score_confidence(item) for item in items]
    return [f for f in flags if f.score < threshold]
```

### 3. LLM Validator（端ケースだけ）

```python
def validate_with_llm(
    item: ParsedItem,
    original_text: str,
    client,
) -> ParsedItem:
    """Use LLM to fix low-confidence extractions."""
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",  # Cheapest model for validation
        max_tokens=500,
        messages=[{
            "role": "user",
            "content": (
                f"Extract the question, choices, and answer from this text.\n\n"
                f"Text: {original_text}\n\n"
                f"Current extraction: {item}\n\n"
                f"Return corrected JSON if needed, or 'CORRECT' if accurate."
            ),
        }],
    )
    # Parse LLM response and return corrected item...
    return corrected_item
```

### 4. Hybrid Pipeline

```python
def process_document(
    content: str,
    *,
    llm_client=None,
    confidence_threshold: float = 0.95,
) -> list[ParsedItem]:
    """Full pipeline: regex -> confidence check -> LLM for edge cases."""
    # Step 1: Regex extraction (handles 95-98%)
    items = parse_structured_text(content)

    # Step 2: Confidence scoring
    low_confidence = identify_low_confidence(items, confidence_threshold)

    if not low_confidence or llm_client is None:
        return items

    # Step 3: LLM validation (only for flagged items)
    low_conf_ids = {f.item_id for f in low_confidence}
    result = []
    for item in items:
        if item.id in low_conf_ids:
            result.append(validate_with_llm(item, content, llm_client))
        else:
            result.append(item)

    return result
```

## 実運用メトリクス

本番の quiz parsing pipeline（410 items）から:

| Metric                  | Value    |
| ----------------------- | -------- |
| Regex success rate      | 98.0%    |
| Low confidence items    | 8 (2.0%) |
| LLM calls needed        | ~5       |
| Cost savings vs all-LLM | ~95%     |
| Test coverage           | 93%      |

## ベストプラクティス

- **まず regex で始める** - 不完全でも基礎線ができる
- **confidence scoring を入れる** - LLM が必要な箇所を機械的に見つける
- **最も安い LLM を使う** - 検証用途なら Haiku 級で十分
- **parsed item を mutate しない** - クリーニング / 検証では新しいインスタンスを返す
- **parser と TDD は相性が良い** - 既知パターンのテストを書いてから edge case を増やす
- **メトリクスを記録する** - regex 成功率、LLM 呼び出し数を追う

## 避けるべきアンチパターン

- regex で 95% 以上処理できるのに全文を LLM に送ること
- 自由形式で変動の大きいテキストに regex を無理に使うこと
- confidence scoring を省いて「なんとかなる」と期待すること
- クリーニングや検証中に parsed object を破壊的変更すること
- edge case（壊れた入力、欠損項目、文字化け）をテストしないこと

## 向いている用途

- クイズ / 試験問題の解析
- フォームデータ抽出
- 請求書 / レシート処理
- 文書構造解析（見出し、セクション、表）
- コストが重要な、繰り返しパターンを持つ構造化テキスト全般

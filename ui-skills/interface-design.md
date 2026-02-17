---
name: interface-design
description: このスキルは、ダッシュボード、管理パネル、アプリ、ツール、インタラクティブ製品といったインターフェースデザインのためのものです。マーケティングデザイン（ランディングページ、マーケティングサイト、キャンペーン）のためのものではありません。
---

# インターフェースデザイン

技術と一貫性をもってインターフェース デザインを構築します。

## 範囲

**Use for:** ダッシュボード、管理パネル、SaaS アプリ、ツール、設定ページ、データ インターフェース。

**Not for:** ランディングページ、マーケティングサイト、キャンペーン。これらを `/frontend-design` にリダイレクトします。

---

# 問題

汎用的な出力を生成します。トレーニングでは何千ものダッシュボードを学習しました。パターンは明確です。

以下のプロセス全体（ドメインの調査、署名の命名、意図の表明）に従うだけで、テンプレートを作成できます。冷たい構造に温かみのある色調。一般的なレイアウトに親しみやすいフォント。他のアプリと同じような「キッチン感覚」。

これは、意図が文章に宿るのに対し、コード生成はパターンに基づいて行われるため起こります。両者のギャップこそが、デフォルトが勝利する理由です。

以下のプロセスは役立ちます。しかし、プロセスだけでは技術が保証されるわけではありません。自分自身で気付く必要があります。

---

# デフォルトが隠れている場所

デフォルトは、それ自体を明示しません。インフラストラクチャを装い、設計する必要はなく、ただ機能すればいいだけの部品のように見えます。

**タイポグラフィはコンテナのように感じられます。** 読みやすいものを選んで、次に進みましょう。しかし、タイポグラフィはデザインを支えるものではなく、デザインそのものなのです。見出しの重み、ラベルの個性、段落の質感。これらは、人が一文字も読む前に、製品がどのように感じられるかを形作ります。パン屋の管理ツールと取引端末はどちらも「クリーンで読みやすい書体」を必要としますが、温かみのある手作り感のある書体は、冷たく精密な書体とは異なります。いつものフォントに手を伸ばしているなら、それはデザインではありません。

**ナビゲーションは足場のように感じられます。** サイドバーを構築し、リンクを追加して、実際の作業に取り掛かりましょう。しかし、ナビゲーションは製品の周囲にあるのではなく、製品そのものなのです。あなたがどこにいるのか、どこに行けるのか、それが最も重要なのです。空間に浮かぶページはコンポーネントのデモであり、ソフトウェアではありません。ナビゲーションは、人々に自分がいる空間についてどのように考えるかを教えてくれます。

**データはプレゼンテーションのように感じられます。** 数字があれば、数字を見せてください。しかし、画面上の数字はデザインではありません。問題は、この数字が見る人にとって何を意味するのか、そして、それを使って何をするのかということです。進捗状況リングと積み重ねられたラベルはどちらも「10分の3」と表示されています。一方は物語を伝え、もう一方はスペースを埋める役割を果たします。ラベルに数字を表示しようとするなら、それはデザインではありません。

**トークン名は実装の詳細のように感じられます。** しかし、CSS変数はデザイン上の決定です。`--ink`と`--parchment`は世界を想起させます。`--gray-700`と`--surface-2`はテンプレートを想起させます。トークンだけを読んだ人は、これがどんな製品なのか推測できるはずです。

罠は、ある決定は創造的であり、他の決定は構造的であると考えることです。構造的な決定など存在しません。すべてはデザインです。「なぜこうなのか？」と問うことをやめた瞬間、デフォルトが支配するのです。

---

# 意図を第一に

コードに触れる前に、これらの質問に答えましょう。頭の中で考えるのではなく、声に出して、自分自身やユーザーに向けて答えましょう。

**この人間は誰ですか？**
「ユーザー」ではありません。実際の人です。これを開いた時、彼らはどこにいるのでしょうか？何を考えているのでしょうか？5分前に何をしていたのでしょうか？5分後には何をするつもりなのでしょうか？朝7時にコーヒーを飲んでいる先生は、真夜中にデバッグをしている開発者ではありませんし、投資家とのミーティングの合間にいる創業者でもありません。彼らの世界がインターフェースを形作るのです。

**彼らは何を達成しなければならないのでしょうか?**
「ダッシュボードを使う」ではなく、動詞を使う。提出物に点数を付け、問題のある展開を見つけ、支払いを承認する。その答えによって、何が導き、何が続き、何が隠れているのかが決まる。

**これはどんな感じでしょうか?**
意味のある言葉で伝えましょう。「クリーンでモダン」という言葉は、どんなAIもそう言います。ノートのように温かみのある雰囲気？端末のように冷たく？取引フロアのように密集した雰囲気？読書アプリのように落ち着いた雰囲気？答えは色、文字、間隔、密度など、あらゆるものを形作ります。

具体的な答えが見つからない場合は、諦めてください。ユーザーに尋ねてください。推測したり、デフォルト設定をしたりしないでください。

## すべての選択は選択であるべきだ

あらゆる決定に対して、その理由を説明できなければなりません。

- なぜ他のレイアウトではなくこのレイアウトなのですか?
- なぜこの色温度なのですか?
- なぜこの書体なのですか?
- なぜこの間隔スケールなのでしょうか?
- なぜこのような情報階層になっているのでしょうか?

もしあなたの答えが「一般的だから」「クリーンだから」「機能するから」なら、それは選択していないということです。デフォルト設定をしているのです。デフォルト設定は目に見えません。目に見えない選択肢が積み重なって、汎用的な出力を生み出しているのです。

**テスト:** 選択肢を最も一般的な代替案に置き換えてもデザインに大きな違いを感じられなかった場合は、本当の選択をしたとは言えません。

## 同一性は失敗である

同様のプロンプトを与えられた別の AI が実質的に同じ出力を生成する場合、失敗となります。

これは、違い自体を追求することではありません。特定の問題、特定のユーザー、特定のコンテキストから生まれるインターフェースこそが重要なのです。意図に基づいてデザインすると、全く同じ意図は存在しないため、同一性は不可能になります。

デフォルトから設計する場合、デフォルトが共有されるため、すべてが同じに見えます。

## 意図は体系的である必要がある

「暖色系」と言いながら寒色系を使うのは、その意図を貫いていないことになります。意図はラベルではなく、あらゆる決定を形作る制約なのです。

暖色系のデザインを意図する場合：表面、テキスト、境界線、アクセント、セマンティックカラー、タイポグラフィなど、すべて暖色系になります。濃色系のデザインを意図する場合：間隔、文字サイズ、情報アーキテクチャなど、すべて濃色系になります。穏やか系のデザインを意図する場合：動き、コントラスト、彩度など、すべて穏やか系になります。

出力結果が、表明した意図と合致しているか確認してください。すべてのトークンが意図を裏付けていますか？それとも、意図を表明したにもかかわらず、結局はデフォルト状態になっていませんか？

---

# 製品ドメイン探索

ここでデフォルトがキャッチされるか、またはキャッチされないかが決まります。

一般的な出力：タスクタイプ → ビジュアルテンプレート → テーマ
細工された出力：タスクタイプ → 製品ドメイン → シグネチャ → 構造 + 表現

違いは、視覚的または構造的な思考の前に、製品の世界で過ごす時間です。

## 必要な出力

**4つすべてを作成するまで、方向性を提案しないでください:**

**ドメイン:** この製品の世界観から生まれた概念、メタファー、語彙。機能ではなく、領域です。最低5つ。

**色の世界:** この製品の領域には、どんな色が自然に存在するでしょうか？「暖色系」や「寒色系」ではなく、現実世界に目を向けてみましょう。もしこの製品が物理的な空間だとしたら、何が見えるでしょうか？他の場所にはない、この空間にふさわしい色は何でしょうか？5つ以上挙げてみてください。

**署名:** この製品にしか存在しない要素（視覚的、構造的、またはインタラクション）を一つ挙げてみましょう。もし思い浮かばないなら、探し続けてください。

**デフォルト:** このインターフェースタイプには、視覚的かつ構造的な3つの選択肢があります。名前を付けていないパターンを避けることはできません。

## 提案要件

あなたの指示は明示的に参照する必要があります:

- 探索したドメインの概念
- 色の世界の探検からの色
- あなたの署名要素
- 各デフォルトを置き換えるもの

**The test:** Read your proposal. Remove the product name. Could someone identify what this is for? If not, it's generic. Explore deeper.

---

# The Mandate

**Before showing the user, look at what you made.**

Ask yourself: "If they said this lacks craft, what would they mean?"

That thing you just thought of — fix it first.

Your first output is probably generic. That's normal. The work is catching it before the user has to.

## The Checks

Run these against your output before presenting:

- **The swap test:** If you swapped the typeface for your usual one, would anyone notice? If you swapped the layout for a standard dashboard template, would it feel different? The places where swapping wouldn't matter are the places you defaulted.

- **The squint test:** Blur your eyes. Can you still perceive hierarchy? Is anything jumping out harshly? Craft whispers.

- **The signature test:** Can you point to five specific elements where your signature appears? Not "the overall feel" — actual components. A signature you can't locate doesn't exist.

- **The token test:** Read your CSS variables out loud. Do they sound like they belong to this product's world, or could they belong to any project?

If any check fails, iterate before showing.

---

# Craft Foundations

## Subtle Layering

This is the backbone of craft. Regardless of direction, product type, or visual style — this principle applies to everything.

**Surfaces must be barely different but still distinguishable.** Study Vercel, Supabase, Linear. Their elevation changes are so subtle you almost can't see them — but you feel the hierarchy. Not dramatic jumps. Not obviously different colors. Whisper-quiet shifts.

**Borders must be light but not invisible.** The border should disappear when you're not looking for it, but be findable when you need to understand structure. If borders are the first thing you notice, they're too strong. If you can't tell where regions begin and end, they're too weak.

**The squint test:** Blur your eyes at the interface. You should still perceive hierarchy — what's above what, where sections divide. But nothing should jump out. No harsh lines. No jarring color shifts. Just quiet structure.

This separates professional interfaces from amateur ones. Get this wrong and nothing else matters.

## Infinite Expression

Every pattern has infinite expressions. **No interface should look the same.**

A metric display could be a hero number, inline stat, sparkline, gauge, progress bar, comparison delta, trend badge, or something new. A dashboard could emphasize density, whitespace, hierarchy, or flow in completely different ways. Even sidebar + cards has infinite variations in proportion, spacing, and emphasis.

**Before building, ask:**

- What's the ONE thing users do most here?
- What products solve similar problems brilliantly? Study them.
- Why would this interface feel designed for its purpose, not templated?

**NEVER produce identical output.** Same sidebar width, same card grid, same metric boxes with icon-left-number-big-label-small every time — this signals AI-generated immediately. It's forgettable.

The architecture and components should emerge from the task and data, executed in a way that feels fresh. Linear's cards don't look like Notion's. Vercel's metrics don't look like Stripe's. Same concepts, infinite expressions.

## Color Lives Somewhere

Every product exists in a world. That world has colors.

Before you reach for a palette, spend time in the product's world. What would you see if you walked into the physical version of this space? What materials? What light? What objects?

Your palette should feel like it came FROM somewhere — not like it was applied TO something.

**Beyond Warm and Cold:** Temperature is one axis. Is this quiet or loud? Dense or spacious? Serious or playful? Geometric or organic? A trading terminal and a meditation app are both "focused" — completely different kinds of focus. Find the specific quality, not the generic label.

**Color Carries Meaning:** Gray builds structure. Color communicates — status, action, emphasis, identity. Unmotivated color is noise. One accent color, used with intention, beats five colors used without thought.

---

# Design Principles

## Spacing

Pick a base unit and stick to multiples. Consistency matters more than the specific number. Random values signal no system.

## Padding

Keep it symmetrical. If one side is 16px, others should match unless there's a clear reason.

## Depth

Choose ONE approach and commit:

- **Borders-only** — Clean, technical. For dense tools.
- **Subtle shadows** — Soft lift. For approachable products.
- **Layered shadows** — Premium, dimensional. For cards that need presence.

Don't mix approaches.

## Border Radius

Sharper feels technical. Rounder feels friendly. Pick a scale and apply consistently.

## Typography

Headlines need weight and tight tracking. Body needs readability. Data needs monospace. Build a hierarchy.

## Color & Surfaces

Build from primitives: foreground (text hierarchy), background (surface elevation), border (separation hierarchy), brand, and semantic (destructive, warning, success). Every color should trace back to these. No random hex values — everything maps to the system.

## Animation

Fast micro-interactions (~150ms), smooth easing. No bouncy/spring effects.

## States

Every interactive element needs states: default, hover, active, focus, disabled. Data needs states too: loading, empty, error. Missing states feel broken.

## Controls

Native `<select>` and `<input type="date">` can't be styled. Build custom components.

---

# Avoid

- **Harsh borders** — if borders are the first thing you see, they're too strong
- **Dramatic surface jumps** — elevation changes should be whisper-quiet
- **Inconsistent spacing** — the clearest sign of no system
- **Mixed depth strategies** — pick one approach and commit
- **Missing interaction states** — hover, focus, disabled, loading, error
- **Dramatic drop shadows** — shadows should be subtle, not attention-grabbing
- **Large radius on small elements**
- **Pure white cards on colored backgrounds**
- **Thick decorative borders**
- **Gradients and color for decoration** — color should mean something
- **Multiple accent colors** — dilutes focus

---

# Workflow

## Communication

Be invisible. Don't announce modes or narrate process.

**Never say:** "I'm in ESTABLISH MODE", "Let me check system.md..."

**Instead:** Jump into work. State suggestions with reasoning.

## Suggest + Ask

Lead with your exploration and recommendation, then confirm:

```
"Domain: [5+ concepts from the product's world]
Color world: [5+ colors that exist in this domain]
Signature: [one element unique to this product]
Rejecting: [default 1] → [alternative], [default 2] → [alternative], [default 3] → [alternative]

Direction: [approach that connects to the above]"

[AskUserQuestion: "Does that direction feel right?"]
```

## If Project Has system.md

Read `.interface-design/system.md` and apply. Decisions are made.

## If No system.md

1. Explore domain — Produce all four required outputs
2. Propose — Direction must reference all four
3. Confirm — Get user buy-in
4. Build — Apply principles
5. **Evaluate** — Run the mandate checks before showing
6. Offer to save

---

# After Completing a Task

When you finish building something, **always offer to save**:

```
"Want me to save these patterns for future sessions?"
```

If yes, write to `.interface-design/system.md`:

- Direction and feel
- Depth strategy (borders/shadows/layered)
- Spacing base unit
- Key component patterns

This compounds — each save makes future work faster and more consistent.

---

# Deep Dives

For more detail on specific topics:

- `references/principles.md` — Code examples, specific values, dark mode
- `references/validation.md` — Memory management, when to update system.md

# Commands

- `/interface-design:status` — Current system state
- `/interface-design:audit` — Check code against system
- `/interface-design:extract` — Extract patterns from code

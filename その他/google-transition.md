---
URL: https://developers.google.com/style/translation
---

# 世界中の読者に向けて書く

開発者向けドキュメントは米国英語で作成されていますが、一部は英語以外の言語に翻訳されたり、英語が母国語ではない開発者によって読まれたりすることもあります。

ローカリゼーション、翻訳、国際化を念頭に置いて執筆してください。以下のリストでこれらの用語を定義します。:

- _ローカリゼーション:_ 製品と関連ドキュメントを特定の国向けに適応させること。このプロセスには、翻訳だけでなく、現地通貨や測定単位の使用など、さまざまな要素が含まれます。
- _翻訳:_ ある言語を別の言語に翻訳すること。このプロセスにはローカリゼーションが含まれる場合もありますが、この2つの用語は同義ではありません。
- _国際化:_ ローカリゼーションの労力を最小限に抑えるように製品と関連ドキュメントを設計します。たとえば、すべての UI 文字列を別のファイルに配置して翻訳を簡素化します。

詳細については、[言語のローカライズ](https://wikipedia.org/wiki/Language_localisation)を参照してください。

その他のライティングのベストプラクティスについては、以下のリソースを参照してください。:

- [Write accessible documentation](https://developers.google.com/style/accessibility)
- [Write inclusive documentation](https://developers.google.com/style/inclusive-documentation)
- [Voice and tone](https://developers.google.com/style/tone)

## 明確で簡潔、かつ曖昧さのない言葉を使う

世界中の読者と翻訳を考慮し、明確かつ簡潔で曖昧さのない書き方をしてください。

### より簡単な言葉と短い文章を使う

- シンプルな単語を使用してください。例えば、**start** や **begin** の意味で **commence** のような単語は使用しないでください。**so** の意味で **consequently** を使用しないでください。**use** の意味で **utilize** や **leverage** のような単語は使用しないでください。（特別な意味を伝える場合は、これらの単語を使用しても問題ありません。たとえば、_Cloud Spanner は利用可能な CPU リソースの最大 100% を活用します。_ などです。）
- フレーズと同じ意味を伝える場合は、単語を1つだけ使用してください。例えば、 **some** や **many** が使える場合は、 **a number of** のようなフレーズは使用しないでください。
- 短い文を書きましょう。文が短いほど翻訳しやすくなります。英語の文は他の言語よりも短い場合があるため、平均的な長さの英語の文でも翻訳すると長い文になってしまう可能性があります。長い文は理解を妨げ、ページや製品インターフェースの表示に問題を引き起こし、翻訳時間を長引かせ、翻訳とレビューのコストを増加させる可能性があります。

### 句動詞を避ける

- 句動詞は可能な限り避けましょう。句動詞とは、複数の単語を組み合わせて一つの動詞句を形成するものです。これらの動詞は複合動詞とも呼ばれます。まずはより簡単な動詞に置き換えてみてください。より適切な動詞がない場合もあります。例えば、**set up**、**log in**、**sign in** などは、このルールの例外です。
  - 推奨: This document uses the following terms:
  - 非推奨: This document makes use of the following terms:

### 修飾語を適切に使用する

- Don't use too many modifiers. In particular, don't use more than two nouns as modifiers of another noun.
  Recommended: A cloud-native DevSecOps pipeline in a hybrid environment
  Not recommended: A hybrid cloud-native DevSecOps pipeline
- Don't misplace modifiers. For example, place a word like _only_ immediately before the word or phrase that it relates to. If the meaning is still ambiguous, try rephrasing the sentence.
  Recommended: Request only one token.
  Recommended: Request no more than one token.
  Not recommended: Only request one token.

### Use active voice and present tense

- Use [present tense](https://developers.google.com/style/tense) and avoid complex or uncommon verb forms.
- Use active voice. The subject of the sentence is the person or thing performing the action. With passive voice, it's often hard for readers to figure out who's supposed to do something. For more information, see [Active voice](https://developers.google.com/style/voice).

### Use words in their primary sense

- Don't use the same word to mean different things. In particular, avoid using the same word as both a noun and a verb in close proximity. For examples of words that have multiple meanings, see the word list entries for [once](https://developers.google.com/style/word-list#once), [while](https://developers.google.com/style/word-list#while), [as](https://developers.google.com/style/word-list#as), and [since](https://developers.google.com/style/word-list#since).
- Avoid directional language (for example, _above_ or _below_) in procedural documentation. For more information, see [UI elements and interaction](https://developers.google.com/style/ui-elements#buttons).

### Use helper words and optional words

- Use qualifying nouns for technical keywords. For example, when referring to a file called `example.yaml`, call it the _`example.yaml` file_ and not _`example.yaml`_ by itself. For more information, see [Grammatical treatment of code elements](https://developers.google.com/style/code-in-text#keywords).
- Repeat a word if the redundancy improves comprehension.
  | Recommended | Not recommended |
  | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
  | If the VM has started and if you're able to connect... | If the VM has started and you're able to connect... |
  | The resource hierarchy design creates both IAM segmentation and network segmentation by default. | The resource hierarchy design creates both IAM and network segmentation by default. |
  | An egress rule whose action is `allow`, whose destination is `0.0.0.0/0`, and whose priority is the lowest possible (`65535`). | An egress rule whose action is `allow`, destination is `0.0.0.0/0`, and priority is the lowest possible (`65535`). |
- Use helper words. Helper words such as _then_, _that_, and _of_ are frequently left out of conversational English. Use these words to avoid ambiguity.
  | Recommended | Not recommended |
  | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
  | If the attribute key is not found, then the default value is returned. | If the attribute key is not found, the default value is returned. |
  | This document is intended for data engineers and assumes that you have the following knowledge: | This document is intended for data engineers and assumes you have the following knowledge: |
  | Identify all of the datasets. | Identify all the datasets. |
  | Start the profiler, and then run the app. | Start the profiler, then run the app. |
  See also [Optional pronouns](https://developers.google.com/style/pronouns#optional-pronouns).
- Don't omit relative pronouns. To provide clarity and to avoid ambiguity, use relative pronouns such as _that_ and _which_. For more information, see [Relative pronouns](https://developers.google.com/style/pronouns#relative-pronouns).
  Recommended: You can programmatically update the rules that you previously defined.
  Not recommended: You can programmatically update the rules you previously defined.

### Clarify abbreviations and pronouns

- Define abbreviations. Abbreviations can be confusing out of context, and they don't translate well. Spell things out whenever possible, at least the first time that you use a given term. For more information, see [Abbreviations](https://developers.google.com/style/abbreviations).
- Clarify antecedents. Using pronouns can get tricky when translators are working with small, unconnected strings of text. Help them out by making things as clear as possible. For example, if a pronoun is ambiguous, then replace it with the appropriate noun.
  Recommended: If you use the term _green beer_ in an ad, then make sure that the ad is targeted.
  Not recommended: If you use the term _green beer_ in an ad, then make sure that it's targeted.

### Use apostrophes appropriately

Be careful with how you use plural and possessive forms. In general, don't form a plural with _'s_, don't use the plural or possessive form with trademarks of company, product, and feature names, and don't use uncommon contractions. For more information, see [Possessives](https://developers.google.com/style/possessives), [Pluralization](https://developers.google.com/style/pluralization), and [Contractions](https://developers.google.com/style/contractions).

## Address users and their needs directly

Address the user and their needs directly and avoid providing unnecessary information.

- Address the reader directly. Use _you_, instead of _the user_ or _they_, unless you're referring to someone who uses the software that the reader is developing. For more information, see [Second person and first person](https://developers.google.com/style/person).
- Provide context. Don't assume that the reader already knows what you're talking about.
- Avoid negative constructions when possible. Consider whether it's necessary to tell the reader what they can't do instead of what they can.

## Be consistent

Use standard sentence structures, consistent terminology, and appropriate punctuation to avoid creating barriers to understanding, ambiguity, and mistranslations.

### Use consistent terminology

If you use a particular term for a concept in one place, then use that exact same term elsewhere, including the same capitalization. If you use different names for the same thing, translators might think you're referring to different concepts, and thus might use different translations. Inconsistency in terminology and phrasing can greatly increase translation costs, particularly when translation memory and machine translations are used as first steps in translation.

### Use standard sentence structures and formatting

- Use standardized phrases for frequently used sentences, introductory phrases, and other common tasks. For examples, read about [introducing links](https://developers.google.com/style/cross-references#link-introductions), [introducing output](https://developers.google.com/style/placeholders#placeholders-in-output), and [introducing code samples](https://developers.google.com/style/code-samples#introductions).
- Use standard English word order. Sentences follow the _subject + verb + object_ order.
- Try to keep the main subject and verb as close to the beginning of the sentence as possible.
- Use the conditional clause first. If you want to tell the audience to do something in a particular circumstance, mention the circumstance before you provide the instruction. For more information, see [Sentence structure](https://developers.google.com/style/sentence-structure).
- Make list items consistent. Make list items parallel in structure. Be consistent in your capitalization and punctuation. For more information, see [Lists](https://developers.google.com/style/lists).

### Use consistent text formatting

- Use consistent typographic formats. Use bold and italics consistently. Don't switch from using italics for emphasis to underlining. For more information, see [Text-formatting summary](https://developers.google.com/style/text-formatting).
- Use consistent capitalization. For more information, see [Capitalization](https://developers.google.com/style/capitalization).

## Be inclusive

You're not writing for your culture. Write with inclusivity in mind. For more information, see [Writing inclusive documentation](https://developers.google.com/style/inclusive-documentation).

- Write [dates and times](https://developers.google.com/style/dates-times) in unambiguous and clear ways.
- Don't be too culturally specific. In particular, don't refer to specific holidays, cultural practices, or sports unless you're certain they're known worldwide.
- Use a diverse set of example names. If you need to use people's names (for example, as email addresses), use a diverse set of names. For more information, see [Example domains and names](https://developers.google.com/style/examples).
- Avoid colloquialisms, idioms, or slang. Phrases like _ballpark figure_, _back burner_, or _hang in there_ can be confusing and difficult to translate.
- Avoid humor. Most humor is difficult to translate, and much humor is culturally specific.
- Avoid geographically specific references, like the seasons. Remember that August isn't summer in the southern hemisphere. For more information, see [Expressing divisions of the year](https://developers.google.com/style/dates-times#divisions-year).

## Consider accessibility for images

Use screenshots and text in figures sparingly. Images don't get translated. Any new information should be conveyed through text and not introduced in a figure or image. For more information, see [Figures and other images](https://developers.google.com/style/images).

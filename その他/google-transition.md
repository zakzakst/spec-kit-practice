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

- 修飾語を使いすぎないでください。特に、名詞を修飾する名詞を2つ以上使用しないでください。
  - 推奨: A cloud-native DevSecOps pipeline in a hybrid environment
  - 非推奨: A hybrid cloud-native DevSecOps pipeline
- 修飾語を間違えないようにしてください。例えば、**only**のような単語は、関連する単語や句の直前に置きます。それでも意味が曖昧な場合は、文を言い換えてみましょう。
  - 推奨: Request only one token.
  - 推奨: Request no more than one token.
  - 非推奨: Only request one token.

### 能動態と現在形を使う

- [現在形](https://developers.google.com/style/tense)を使用し、複雑または一般的でない動詞の形は避けてください。
- 能動態を使いましょう。文の主語は、動作を行っている人または物です。受動態では、読者は誰が何をすべきか理解しにくいことがよくあります。詳しくは、[能動態](https://developers.google.com/style/voice)をご覧ください。

### 言葉を本来の意味で使う

- 同じ単語を異なる意味に使用しないでください。特に、同じ単語を名詞と動詞の両方に近接して使用することは避けてください。複数の意味を持つ単語の例については、単語リストの[once](https://developers.google.com/style/word-list#once)、[while](https://developers.google.com/style/word-list#while)、[as](https://developers.google.com/style/word-list#as)、[since](https://developers.google.com/style/word-list#since)をご覧ください。
- 手順に関するドキュメントでは、方向を示す表現（「上」や「下」など）は避けてください。詳しくは、[UI要素と操作](https://developers.google.com/style/ui-elements#buttons)をご覧ください。

### 補助語とオプション語を使用する

- 技術的なキーワードには修飾名詞を使用してください。例えば、「example.yaml」というファイルを指す場合は、「example.yaml」ファイルと呼び、「example.yaml」単体とは呼びません。詳しくは、[コード要素の文法的な扱い](https://developers.google.com/style/code-in-text#keywords)をご覧ください。
- 冗長性によって理解度が向上する場合は、単語を繰り返します。
  | 推奨 | 非推奨 |
  | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
  | VM が起動していて接続できる場合... | VM が起動し、接続できる場合... |
  | リソース階層設計では、デフォルトで IAM セグメンテーションとネットワーク セグメンテーションの両方が作成されます。 | リソース階層設計では、デフォルトで IAM とネットワーク セグメンテーションの両方が作成されます。 |
- 補助語を使いましょう。「_then_」「_that_」「_of_」といった補助語は、会話では省略されることがよくあります。曖昧さを避けるために、これらの単語を使いましょう。
  | 推奨 | 非推奨 |
  | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
  | If the attribute key is not found, then the default value is returned. | If the attribute key is not found, the default value is returned. |
  | This document is intended for data engineers and assumes that you have the following knowledge: | This document is intended for data engineers and assumes you have the following knowledge: |
  | Identify all of the datasets. | Identify all the datasets. |
  | Start the profiler, and then run the app. | Start the profiler, then run the app. |
  [オプションの代名詞](https://developers.google.com/style/pronouns#optional-pronouns)も参照してください。
- 関係代名詞は省略しないでください。明確さを保ち、曖昧さを避けるために、「_that_」や「_which_」などの関係代名詞を使用してください。詳しくは、[関係代名詞](https://developers.google.com/style/pronouns#relative-pronouns)をご覧ください。
  - 推奨: You can programmatically update the rules that you previously defined.
  - 非推奨: You can programmatically update the rules you previously defined.

### 略語と代名詞を明確にする

- 略語を定義しましょう。略語は文脈から外れると混乱を招き、翻訳もうまくいきません。可能な限り、少なくとも最初に使用する際には、スペルを明確にしましょう。詳しくは[略語](https://developers.google.com/style/abbreviations)をご覧ください。
- 先行詞を明確にしましょう。翻訳者が短くて繋がりのない文章を扱う場合、代名詞の使い方は難しくなることがあります。できるだけ明確にすることで、翻訳者の負担を軽減しましょう。例えば、代名詞が曖昧な場合は、適切な名詞に置き換えましょう。
  - 推奨: If you use the term _green beer_ in an ad, then make sure that the ad is targeted.
  - 非推奨: If you use the term _green beer_ in an ad, then make sure that it's targeted.

### アポストロフィを適切に使用する

複数形と所有格の使い方には注意してください。一般的に、_'s_ を使って複数形を作ったり、会社名、製品名、機能名などの商標で複数形や所有格を使ったり、一般的でない短縮形を使ったりしないでください。詳しくは、[所有格](https://developers.google.com/style/possessives)、[複数形](https://developers.google.com/style/pluralization)、[短縮形](https://developers.google.com/style/contractions)をご覧ください。

## ユーザーとそのニーズに直接対応する

ユーザーとそのニーズに直接対応し、不要な情報の提供は避けてください。

- 読者に直接語りかけましょう。読者が開発中のソフトウェアを使用する人物を指す場合を除き、「ユーザー」や「彼ら」ではなく、「あなた」を使用してください。詳しくは[二人称と一人称](https://developers.google.com/style/person)をご覧ください。
- 文脈を伝えましょう。読者が既にあなたが話している内容を知っていると想定しないでください。
- 可能な限り否定的な表現は避けましょう。読者に何ができるかではなく、何ができないかを伝える必要があるかどうかを検討しましょう。

## 一貫性を保つ

理解の妨げ、曖昧さ、誤訳を避けるために、標準的な文構造、一貫した用語、適切な句読点を使用してください。

### 一貫した用語を使用する

ある概念について、ある箇所で特定の用語を使用した場合、大文字小文字の使い分けを含め、他の箇所でも全く同じ用語を使用してください。同じものに異なる名称を使用すると、翻訳者は異なる概念を指していると誤解し、異なる翻訳結果になる可能性があります。用語や表現の不一致は、特に翻訳の最初のステップとして翻訳メモリや機械翻訳を使用する場合、翻訳コストを大幅に増加させる可能性があります。

### 標準的な文構造と書式を使用する

- よく使う文、導入フレーズ、その他の一般的なタスクには、標準化されたフレーズを使用してください。例については、[リンクの紹介](https://developers.google.com/style/cross-references#link-introductions)、[出力の紹介](https://developers.google.com/style/placeholders#placeholders-in-output)、[コードサンプルの紹介](https://developers.google.com/style/code-samples#introductions)をご覧ください。
- 標準的な英語の語順を使用してください。文は「主語 + 動詞 + 目的語」の順序に従います。
- できるだけ主語と動詞を文の冒頭に置くようにしてください。
- 条件節を最初に使用してください。特定の状況で読者に何かをするように指示したい場合は、指示を与える前にその状況について述べてください。詳しくは、[文の構造](https://developers.google.com/style/sentence-structure)をご覧ください。
- リスト項目に一貫性を持たせましょう。リスト項目は構造的に並列にし、大文字と小文字の使い分けや句読点の使い分けにも一貫性を持たせましょう。詳しくは、[リスト](https://developers.google.com/style/lists)をご覧ください。

### 一貫したテキストフォーマットを使用する

- 一貫した書体形式を使用してください。太字と斜体は一貫して使用してください。強調のために斜体を使用する代わりに、下線を使用するのは避けてください。詳しくは、[テキスト書式設定の概要](https://developers.google.com/style/text-formatting)をご覧ください。
- 大文字表記は統一してください。詳しくは、[大文字表記](https://developers.google.com/style/capitalization)をご覧ください。

## 包括的であること

あなたは自分の文化に合わせて書いているのではありません。インクルーシブな考え方を念頭に置いて書いてください。詳しくは、[インクルーシブなドキュメントの書き方](https://developers.google.com/style/inclusive-documentation)をご覧ください。

- [日付と時刻](https://developers.google.com/style/dates-times)は、明確かつ正確に記述してください。
- 文化的な側面にこだわりすぎないようにしてください。特に、特定の祝日、文化的慣習、スポーツなどについては、それが世界中で知られていると確信できる場合を除き、言及しないでください。
- 多様なサンプル名を使用してください。人名（例えばメールアドレスなど）を使用する必要がある場合は、多様なサンプル名を使用してください。詳しくは、[ドメイン名とサンプル名](https://developers.google.com/style/examples)をご覧ください。
- 口語表現、慣用句、俗語は避けましょう。「ballpark figure（大体どれくらい？）」「back burner（後回しにする？）」「hang in there（しばらく様子を見る）」といった表現は、混乱を招きやすく、翻訳も困難です。
- ユーモアは避けましょう。ユーモアの多くは翻訳が難しく、文化特有のものも多いです。
- 季節など、地理的な表現は避けましょう。南半球では8月は夏ではないことを覚えておきましょう。詳しくは、[1年の区分の表現](https://developers.google.com/style/dates-times#divisions-year)をご覧ください。

## 画像のアクセシビリティを考慮する

スクリーンショットや図内のテキストは控えめに使用してください。画像は翻訳されません。新しい情報はテキストで伝える必要があり、図や画像で紹介しないでください。詳しくは、[図やその他の画像](https://developers.google.com/style/images)をご覧ください。

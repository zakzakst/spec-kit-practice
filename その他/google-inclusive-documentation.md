---
URL: https://developers.google.com/style/inclusive-documentation
---

# 包括的なドキュメントを書く

**補足**: このドキュメントには、Google が失礼または不快とみなす用語が含まれています。これらの用語は、使用方法のガイダンスと代替用語を提供するために使用されています。

私たちは、インクルーシブな考え方と多様性を念頭に置いて開発者向けドキュメントを作成しています。このページは網羅的なリファレンスではありませんが、インクルーシブなドキュメントを作成するためのベストプラクティスを示す一般的なガイドラインと例をいくつか提供しています。

その他のライティングのベストプラクティスについては、以下のリソースを参照してください:

- [Write for a global audience](https://developers.google.com/style/translation)
- [Write accessible documentation](https://developers.google.com/style/accessibility)
- [Voice and tone](https://developers.google.com/style/tone)

## 障害者差別的な言葉を避ける

[親しみやすく会話的な口調](https://developers.google.com/style/tone)を目指すと、問題のある障害者差別的な言葉遣いが紛れ込んでしまう可能性があります。これは比喩表現やその他の言い回しの形で現れることがあります。特にくだけた口調を目指す場合は、言葉の選択に注意してください。障害者差別的な言葉遣いには、**crazy**、**insane**、**blind to**、**blind eye to**、**cripple**、**dumb** などの単語やフレーズが含まれます。文脈に応じて代わりの言葉を使いましょう。

| 推奨                                                                              | 非推奨                                                                                                                                 |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Before launch, give everything a final check for completeness and clarity.        | Before launch, give everything a final sanity-check.                                                                                   |
| There are some baffling outliers in the data.                                     | There are some crazy outliers in the data.                                                                                             |
| It slows down the service, causing a poor user experience until the queue clears. | It cripples the service, causing a poor user experience until the queue clears.                                                        |
| Replace the placeholder in this example with the appropriate value.               | Replace the [dummy variable](https://developers.google.com/style/word-list#dummy-variable) in this example with the appropriate value. |

## 不必要に性別を区別する言葉を避ける

物語の例で使用される[代名詞](https://developers.google.com/style/pronouns#gender-neutral-pronouns)に留意するだけでなく、性別を示す言葉が使用される可能性のある他の情報源にも配慮してください。

| 推奨                                                                                       | 非推奨                                                                                    |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| Equipment installation takes around 16 person-hours to complete.                           | Equipment installation takes around 16 man-hours to complete.                             |
| This API might be just what your globally conscious company needs to help all of humanity. | This API might be just what your globally conscious company needs to help all of mankind. |

## 不必要に暴力的な言葉を避ける

暴力的な表現や有害な表現は避けてください。例えば、_[STONITH](https://developers.google.com/style/word-list#stonith)_ という用語の使用は避け、代わりに、より具体的な用語を用いて、文脈に沿ってエラーノードを停止するプロセスを説明してください。

やむを得ず、_STONITH_ などの暴力的または有害な用語に言及する必要がある場合は、関連する機能を最初に説明するときにその用語を 1 回だけ使用しますが、その用語を強調しない言い方をしてください。

推奨: This might require you to fence failed nodes.

時々大丈夫: This might require you to fence failed nodes (sometimes referred to as STONITH).

可能な限り、「絞首刑」や「殴る」といった暴力的と解釈される比喩的な表現の使用は避けてください。これらの言葉には非暴力的な解釈もあるかもしれませんが、使用を避けることで、暴力的な解釈によって引き起こされる可能性のある意図しない危害を防ぐことができます。

動物の屠殺に関連する比喩的な表現の使用は避けてください。例えば、オンプレミスやステートフルシステムとステートレスクラウドシステムを比較する際に、ペットと牛の比喩を使うのは避けてください。

## 多様性と包括性の例を書く

例では、多様な名前、性別、年齢、場所を使用してください。以下のアドバイスに留意してください:

- [性別を問わない代名詞](https://developers.google.com/style/pronouns#gender-neutral-pronouns)のガイドラインに従ってください。
- アメリカの文化に特化しすぎないようにしましょう。特定の祝日（[_the holidays_](https://developers.google.com/style/word-list#holiday)の単語リストも参照）、文化的慣習、スポーツ、比喩表現などについて言及する際は、配慮が必要です。ここで配慮することは、[世界中の読者に向けた文章](https://developers.google.com/style/translation#culturally-specific)にも役立ちます。
- 現実世界のユーザーの多様性を例に反映させるために、[多様な名前を選択](https://developers.google.com/style/examples#names)するよう注意してください。架空の人物について書く場合は、ガイドの該当セクションのガイダンスを参照してください。
- 高齢者について書く際は、**the elderly**,**the aged**,**seniors**,**senior citizens**,**80 years young**といった用語や比喩表現は避けましょう。代わりに、**older adults**,**aging population**といった用語を使用するか、関連する場合は、その人物の相対的な年齢や例に出てくる他の人々との関係性について言及しましょう。

## 機能とユーザーについて包括的な方法で書く

人々を区別するような表現は避けましょう。例えば、人々を英語のネイティブスピーカーや非ネイティブスピーカーと呼ぶのではなく、そもそもそのことについて文書で議論する必要があるのか​​どうかを検討し、その機能について、どの言語を知っているかに関わらず誰にとっても関連性のある言葉で説明できるように修正しましょう。

可能な限り、技術的な概念に社会的に意味の強い用語を使用することは避けてください。たとえば、[ブラックリスト](https://developers.google.com/style/word-list#blacklist)や[ネイティブ](https://developers.google.com/style/word-list#native)機能といった用語は避けてください。また、[ファーストクラスシチズン](https://wikipedia.org/wiki/First-class_citizen)といった用語は、たとえ広く使用されているとしても使用しないでください。

### 非包括的な用語を置き換えるか、回避する

このセクションでは、非包括的用語を置き換えるか、回避する方法について説明します。用語が業界で定着しており、置き換えると混乱を招く可能性がある場合は、[定着した用語を置き換える](https://developers.google.com/style/inclusive-documentation#replace)をご覧ください。用語がコードサンプルやキーワードに使用されている場合は、[非包括的コード用語を回避する](https://developers.google.com/style/inclusive-documentation#write-around)をご覧ください。非包括的専門用語の使用を避ける方法については、[専門用語](https://developers.google.com/style/jargon)をご覧ください。

#### 既存の用語を置き換える

業界では、*ホワイトリスト*のように、非包含的な用語が広く使用されています。既存の用語を置き換えることで読者に混乱を招く可能性がある場合は、最初に非包含的な用語を括弧で囲んで直接参照してください。その後、文書の残りの部分では、置き換えた包括的な用語を使用してください。

推奨: 管理者に通知を確実に届けるには、管理者を許可リスト（ホワイトリストと呼ばれることもあります）に追加してください。許可リストに登録されていないユーザーはブロックされます。

推奨: このモデルでは、Jenkins コントローラー（マスター）が HTTP リクエストを処理します。Jenkins コントローラーは、… というように設計されています。

推奨: クラウド アーキテクチャでは、サーバーは商品として扱われます (「ペットではなく牛」という比喩で説明されることもあります)。

多くの場合、単語を直接置き換えるのではなく、文の明瞭性を高めるために書き直すことができます。例えば、動詞「_whitelist_」を「_allowlist_」に置き換える代わりに、文を書き直してみましょう。

推奨: You can allow requests from a range of IP addresses by entering a CIDR block instead of a single address in the field.

非推奨: You can allowlist a range of IP addresses by entering a CIDR block instead of a single address in the field.

#### 非包括的なコード用語を回避する

場合によっては、非包含的な用語がコード（または類似の用語）に名前やキーワードとして埋め込まれており、それらの用語を単純に無視して別の用語を使用することはできません。ただし、読者に明確な説明を提供しながら、その用語の使用を*最小限*に抑える（つまり、専門用語として広めないようにする）ことは可能です。コードフォントでない限り、非包含的な名前やキーワードを使用しないでください。

以下は、コードやキーワードに出現する非包含用語を回避するための記述のシナリオです。

一つのシナリオとしては、既存のシステムで、あるエンティティが既に非包含的な用語で命名されているケースが挙げられます。例えば、次のようなクラスタ名を含む構成ファイルがあるとします:

apiVersion: v1
kind: Config
preferences: {}

clusters:

- cluster:
  name: master
- cluster:
  name: replica-1

もう1つのシナリオは、ドキュメントに、SQL方言のキーワード「SLAVE」などの確立されたキーワードである非包括的な用語が含まれている場合です。:

START SLAVE UNTIL SQL_AFTER_MTS_GAPS;

非包含用語を使用するコード項目を初めて参照する場合は、その用語を直接参照できますが、コード フォントで書式設定し、可能であれば括弧で囲みます。

推奨: 設定ファイルは、親ノード (ファイル内では「master」という名前) の作成に役立ちます。

推奨: `START SLAVE` ステートメントを使用してレプリカを開始します。

以降の記述では、推奨用語（_親ノード_、_replica_）を使用してください。エンティティ名またはキーワードを参照する必要がある場合は、コードのフォーマットのみを使用して参照してください。

## 障害やアクセシビリティについて議論する際には偏見や危害を避ける

多くの開発者は、アクセシビリティと障がいのある方々に配慮した製品を開発しています。これらの機能をドキュメント化する際、また障がいのある方やアクセシビリティについて記述する際には、意図しない偏見や悪影響を排除するよう努めてください。ドキュメントに記述する前に、対象となるコミュニティがどのような方法で識別され、説明されることを好んでいるかについて、時間をかけて理解を深めてください。

この分野における一般的なガイドラインとしては次のようなものがある:

- 障害のない人を*normal*や*healthy*と表現しないでください。これは、障害のある人を異常または病気であると暗示し、他者化や疎外を助長することになります。代わりに、_nondisabled person_,_sighted person_,_hearing person_,_person without disabilities_,*neurotypical person*などの言葉を使用してください。
- あなたが取り上げているコミュニティの人々が、どのように識別されることを好んでいるかを調べ、彼らが好む言葉を使いましょう。多くの場合、人格を否定したり、障害によって人を定義したりするような言葉は避けるべきです。例えば、_the disabled_,*a quadriplegic*といった言葉は避け、_people with disabilities_,*a quadriplegic person*といった言葉を使いましょう。
  - ただし、一部のコミュニティのメンバーの多くは、*アイデンティティファースト言語*を好みます。例えば、自閉症、視覚障害、聴覚障害のあるコミュニティでは、この傾向が一般的です。アイデンティティの大文字表記も人によって異なります（いくつかの視点については、[アイデンティティファースト言語](https://autisticadvocacy.org/about-asan/identity-first-language/)と[聴覚障害のあるコミュニティにおける自己認識](https://www.verywellhealth.com/deaf-culture-big-d-small-d-1046233)をご覧ください）。可能な限り、コミュニティの人々のアイデンティティを尊重する用語を調査し、選択してください。
- リンクや相互参照を参照するには、「_see_」を使用してください。詳しくは、[see](https://developers.google.com/style/word-list#see) をご覧ください。
- 障害に対する感情や判断を反映したり、投影したりするような言葉（_victim of_,_suffering from_,*wheelchair-bound*など）は避けましょう。代わりに、_experiencing_,_living with_,*uses a wheelchair*といった中立的な言葉を使用してください。
- _physically challenged_,_special_,_differently abled_,*handi-capable*などの婉曲表現や上から目線の表現は避けてください。

---
name: pr-report
description: >
  maintainer 級の PR / contribution report を HTML または Markdown で作成します。
  PR を深くレビューしたいとき、設計を説明したいとき、システム比較をしたいとき、
  あるいは merge recommendation を準備したいときに使います。
---

# PR レポート Skill

PR、branch、または大きな contribution の maintainer 級レビューを作成します。

基本姿勢:

- まず変更を理解してから判断する
- diff だけでなく、実際に組み上がった system を説明する
- architecture の問題と product scope の異議を切り分ける
- あいまいな印象ではなく、具体的な recommendation を出す

## 使う場面

次のような依頼があったときに使います:

- 「この PR を深くレビューして」
- 「この contribution について説明して」
- 「この PR の report または webpage を作って」
- 「この design を類似システムと比較して」
- 「これを merge すべき？」

## 出力

よくある出力:

- `tmp/reports/...` の単独 HTML レポート
- `report/` または指定されたフォルダーの Markdown レポート
- チャットでの短い maintainer 向け要約

ユーザーが webpage を求めたら、見出しが明確で視認性の高い、洗練された
standalone HTML artifact を作ります。

この skill に同梱されている resources:

- 視覚的な方向性とレポートの表示ルール: `references/style-guide.md`
- 再利用可能な standalone HTML/CSS starter: `assets/html-report-starter.html`

## ワークフロー

### 1. 対象を把握し、枠組みを作る

可能なら GitHub の PR ページだけでなく、ローカルコードから作業します。

集めるもの:

- 対象 branch または worktree
- diff の規模と変更された subsystem
- 関連する repo docs、spec、invariant
- PR 本文または design doc に記載された contributor の意図

最初に答えるべき問いは、「この変更は *何になろうとしているのか*？」です。

### 2. system の mental model を作る

ファイルごとのメモで止まらず、設計全体を再構成します:

- どのような新しい runtime または contract が存在するか
- どの layer が変更されたか: db、shared types、server、UI、CLI、docs
- lifecycle: install、startup、execution、UI、failure、disablement
- trust boundary: どの code が、どこで、どの authority のもとに実行されるか

大きな contribution では、first principles から system を教える tutorial-style の
section を入れます。

### 3. maintainer としてレビューする

finding は最初に出します。severity の高い順に並べます。

優先するもの:

- behavioral regression
- trust または security の欠落
- 誤解を招く abstraction
- lifecycle と運用上のリスク
- 後から解きほぐすことが難しい coupling
- 不足している test または検証不能な主張

可能なら、必ず具体的な file reference を示します。

### 4. 異議の種類を区別する

懸念が次のどれに当たるかを明確にします:

- product direction
- architecture
- implementation quality
- rollout strategy
- documentation honesty

architecture への異議を scope への異議の中に隠さないでください。

### 5. 必要なら外部の先例と比較する

contribution が framework または platform の概念を導入する場合は、類似する open-source system と比較します。

比較するときの原則:

- 公式 docs または source を優先する
- extension boundary、context の受け渡し、trust model、UI ownership に焦点を当てる
- 類似点を並べるだけでなく、そこから教訓を取り出す

比較のための良い問い:

- lifecycle を所有するのは誰か？
- UI composition を所有するのは誰か？
- context は明示的か、それとも ambient か？
- plugin は trusted code か、それとも sandboxed code か？
- extension point に名前と型が付いているか？

### 6. recommendation を実行可能にする

単に「merge」または「merge しない」で終わらせないでください。

次のいずれかを選びます:

- 現状のまま merge する
- 具体的な redesign 後に merge する
- 特定の部分を救済して使う
- design research として維持する

reject または範囲を縮小する場合は、何を残すべきかを明記します。

役立つ recommendation の分類:

- protocol/type model は維持する
- UI boundary を redesign する
- 初期の surface area を絞る
- third-party execution を後回しにする
- まず host-owned extension-point model を提供する

### 7. artifact を作る

推奨する report 構成:

1. Executive summary
2. PR が実際に追加するもの
3. Tutorial: system の動作方法
4. 強み
5. 主な finding
6. 比較
7. Recommendation

HTML report の場合:

- 意図のある typography と color を使う
- 長い report でも navigation を容易にする
- 強い section heading と小さな reference label を活用する
- 一般的な dashboard 風の styling は避ける

ゼロから作る前に `references/style-guide.md` を読みます。
素早く洗練された starter が役立つなら、`assets/html-report-starter.html` から始めます。
placeholder の内容を実際の report に置き換えます。

### 8. 引き渡し前に検証する

確認事項:

- artifact path が存在する
- finding が実際の code と一致している
- 生成 output に、指定された禁止文字列が含まれていない
- test を実行していない場合は、そのことを明記する

## レビューの観点

### plugin と platform の作業

特に注意する点:

- runtime が trusted host process を実行しているのに、docs では sandboxing を主張している
- module-global state を使って React context を隠れて受け渡している
- render order への隠れた依存
- plugin が明示的な API ではなく host internals に直接アクセスしている
- 実際には完全に trusted な code に対する policy label にすぎない「capability」

### 良い兆候

- layer 間で共有される typed contract
- 明示的な extension point
- host-owned lifecycle
- 正直な trust model
- 成長の余地を残した、範囲の狭い初回 rollout

## 最終応答

chat では次を要約します:

- report の場所
- 全体としての判断
- 最も重要な理由を1つまたは2つ
- verification または test を省略したかどうか

チャットの要約は report 本体より短くします。
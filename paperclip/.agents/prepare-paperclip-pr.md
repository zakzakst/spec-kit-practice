---
name: prepare-paperclip-pr
description: Paperclip のブランチを、コミット・PR 本文テンプレート・チェックまで含めて PR 用に整えます。
---
# Paperclip PR を準備する

`paperclipai/paperclip` の `master` に対して、レビュー済みで green な pull request にブランチ作業を変換するための標準手順です。1つの PR につき1回実行します。タスクが1つのブランチを複数 PR に分ける場合は、それぞれに対してこの手順を最初から最後まで実行してください。

## 0. 前提条件 - worktree の安全性

* すべての PR 作業は、専用ブランチの **git worktree** で行ってください。メインの `~/paperclip` checkout は通常、ライブの Paperclip サーバーを動かしています。そこでブランチを checkout しないでください。すでに worktree / branch 上にいる場合は、`git rev-parse --git-dir` と `git branch --show-current` で確認してから進めてください。
* もしメイン checkout が意図せず `master` 以外を指していたら、まずそれを解消してください。作業を失わないようにしながら行います（通常は、その branch の作業を worktree へ移します）。
* どの remote / ref を対象にするか確認してください。通常は `paperclipai/paperclip` リポジトリの `master` ですが、タスクによっては `origin` や `public-gh` のような特定 remote を指定されることがあります。

## 1. すべてコミットする - 作業を失わない

* まず、未コミット変更をすべて **論理的なコミット** にしてください。stash して忘れないこと。ファイルを取り残さないこと。コミットが足りなければ作成してください。
* コミットメッセージの末尾には、必ず次の文字列を入れてください:
  `Co-Authored-By: Paperclip <noreply@paperclip.ing>`

## 2. 変更をきれいに master の上へ載せる

* 対象 remote を fetch し、ブランチを対象の master の上へ rebase（または同等の再適用）してください。PR に merge conflict が残らないようにします。
* rebase 後に再確認します。変更内容に応じて必要な build / test を再実行し、期待どおり通ることを確認してください。

## 3. ガードレールのチェックリスト（すべての PR）

* **`pnpm-lock.yaml` を絶対に commit しないこと** - この repo ではそれを管理する Actions があります。すでに commit に入っている場合は、push 前にその変更を rewrite / drop してください。
* **`.github/workflows/*` は変更しないこと** - その commit 自体がワークフロー変更を明示的に扱っており、タスクでもそれを求めている場合を除きます。
* **デザインのスクリーンショットや wireframe 画像を repo に commit しないこと** - それが本当に成果物の一部である場合を除きます。
* **migration**: master と衝突しないように、番号を順番に増やしてください。master が進んで番号を先に取られていたら、上に載せて番号を振り直します。migration は **idempotent** にして、すでに古い番号を適用済みのユーザーが安全であるようにしてください。
* **Greptile の file 上限**: 1つの PR あたり **100 changed files** 未満にしてください。それを超える場合は、2つに分割してください。

## 4. PR を開く

* PR の title、message format、issue description は `CONTRIBUTING.md`（repo ルート、https://github.com/paperclipai/paperclip/blob/master/CONTRIBUTING.md）に従ってください。
* branch を push して、`gh` で PR を開きます。
* PR の URL はすぐに記録してください。すべての report には、作成した PR の URL を必ず含める必要があります。

## 5. レビューの反復

* **/greploop** company skill を実行します。Greptile review を起動し、コメントに対応し、push し、Greptile が **5/5 かつ unresolved comment なし** になるまで繰り返します（最大 20 回）。turn が残っている間は途中で止めないでください。
* そのあと **/prcheckloop** company skill を実行し、verification / CI の失敗があれば対応します。
* GREPTILE が 5/5 になるまで、すべての test と verification check が通るまで、merge conflict がなくなるまで、絶対に止めないでください。

## 6. 報告して引き継ぐ

* driving task にコメントします。何をしたか、PR URL、worktree path（home は `~` を使う）、Greptile score、check status を含めてください。
* 開いた PR ごとに `pull_request` work product を作成します。branch や commit 自体が引き継ぎ対象なら、それぞれ `branch` / `commit` work product も作ります。
* タスクが PR ごとの follow-up を必要とする場合（たとえば PR ごとの sub-issue など）は、タスクの指示どおりに作成し、リンクしてください。

## ハードルール

* **PR を自分で merge しないこと。絶対に自分で merge しないでください。**
* 作業を失わないこと。stash の取り残し、ファイルの取り落とし、commit を捨てる force-push は禁止です。
* 作成した pull request の URL は必ずすべて投稿してください。

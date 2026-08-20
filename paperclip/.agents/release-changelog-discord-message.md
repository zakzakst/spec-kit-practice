---
name: release-changelog-discord-message
description: >
  release changelog から stable Paperclip release の Discord 告知文を書きます。
  release issue に、dotta の声でそのまま貼れる Discord 投稿や、
  更新済みの `discord_announcement` document が必要なときに使います。
---

# リリース Discord 告知 Skill

**stable** Paperclip release の Discord 告知文を書きます。

これは `.agents/skills/release-changelog/SKILL.md` の companion です。そちらの skill が changelog を書きます。beta soak 中は `release-notes/v{beta-version}` branch 上の `releases/beta/v{beta-version}.md` にあり、stable が出た後は canonicalization PR により `releases/vYYYY.MDD.P.md` に rename されます（詳細はその skill の Channel Process section を参照）。この skill は、その file を dotta の声で1つの Discord ブロックにまとめ、`discord_announcement` document として release issue に投稿します。

## dotta のコメント

> This is for discord — try to follow my format. If I have a section where I
> think about the future, pull from recent issues we're working on etc.

Discord 告知は changelog ではありません。changelog は網羅的ですが、告知は意見があり、語り口を合わせ、出荷済みの主な highlights と、現在の Paperclip 作業から引いてきた本物の "what's next" と "what's on my mind" を中心にまとめます。創作してはいけません。

## 使う場面

- `release-changelog` が changelog を作成した後（soak 中の `release-notes/v{beta-version}` 上の beta-keyed file、または stable 出荷後の canonicalized `releases/vYYYY.MDD.P.md`）。
- release routine が割り当てた release issue で Discord 告知を求められたとき、または新しい日付 / version に合わせて `discord_announcement` document を更新したいとき。
- 単独では実行しないでください。version、日付、contributor list、highlight の集合は必ず対応する changelog file と一致していなければなりません。changelog を更新したら、これも更新します。

## 出力

Discord にそのまま貼れる、1つの fenced markdown code block を出力します。release issue の issue document key `discord_announcement` として添付し、その issue の comment にもそのまま貼り、human がコピーできるようにします。Cases が有効な場合は、後述の social child case も upsert します。

```bash
PUT /api/issues/{releaseIssueId}/documents/discord_announcement
{
  "title": "Discord announcement",
  "format": "markdown",
  "body": "<the announcement>",
  "baseRevisionId": "<latest if updating>"
}
```

document がすでに存在する場合は、先に取得して現在の `baseRevisionId` を渡します。黙って上書きしてはいけません。最後に書かれてから version が変わっていたら、issue comment で何が変わったかを知らせます。

## フォーマット（このテンプレートに従う）

Discord の emoji shortcodes（`:paperclip:`、`:lock:`、`:brain:` など）を使います。Unicode emoji ではありません。Discord は shortcode を描画しますが、changelog file は prose を使います。

```
:paperclip: :paperclip: :paperclip: CLIPPERS!!! v{VERSION} IS OUT :paperclip: :paperclip: :paperclip:

OFFICIAL TWITTER: https://x.com/papercliping - follow it, report any others

## Highlights

:emoji: **Feature Name** - one-sentence description in dotta's voice.
:emoji: **Feature Name** - …
:emoji: **Feature Name** - …

... and a long tail of {flavor of the rest}. Read the [full release notes](<github link>).

## WHATS NEXT (:motorway: Roadmap)

* **Theme A** - one-line forward-looking blurb
* **Theme B** - …
* **Theme C** - …

## What's on my mind

* **Topic** - what's bugging dotta / what's queued / open questions
* **Topic** - …

## PRESS                              (optional — only if there is real press)

* **Outlet / Person** - what happened ([link](<x.com link>))

## WHAT I NEED FROM YOU               (optional — only if there's a real ask)

FOLLOW THE TWITTER: https://x.com/papercliping - that's the only official one
TELL ME if you're using Paperclip in your business - I want to meet you

## Community

この release に貢献してくれたすべての人に感謝します。

```
@username1, @username2, @username3
```

## In Summary

PAPERCLIP は、人間が 100 倍の仕事をこなすための AI オーケストレーターです。

誰もが、12人、100人、あるいは1000人の agent からなるチームを管理するようになり、Paperclip はそのすべてを管理する既定のツールになります。

ITS TIME TO CLIP :paperclip: :paperclip: :paperclip:

FULL RELEASE NOTES

https://github.com/paperclipai/paperclip/blob/master/releases/v{VERSION}.md

||@everyone||
```

テンプレートの注意点:

- 開始と終了の `:paperclip: :paperclip: :paperclip:` bookend はブランドの一部です。
  そのまま残してください。
- 投稿のどこかで install channel を明記します。stable なら `npx paperclipai@latest`、
  早期アクセスなら `@beta` / `@nightly` / `@canary`、Docker の `:latest` は
  **stable release のときだけ**動かします。
- FULL RELEASE NOTES のリンクは `master` 上の `releases/v{VERSION}.md` を指します。
  その file は stable 後の canonicalization PR が merge されてからしか存在しません。
  告知の前に merge してください。
- section 名は UPPERCASE でも Title Case でも構いません。dotta は両方使っています。
  1つの投稿の中ではスタイルを統一してください。
- 最後に `||@everyone||`（Discord の spoiler で包んだ形）を置き、poster が spoiler を外したときに
  ちょうど1回だけ ping されるようにします。

## 文体のヒント

これは dotta が最近の告知で実際に使った書き方から抜き出したものです。
この語り口を真似てください。「professional」なトーンは作らないでください。

- **一人称で会話調。** "I want to meet companies using Paperclip"、
  "what's on my mind"、"if that's you let me know" のように書きます。
  "Paperclip is excited to announce" ではありません。
- **興奮やお願いは ALL CAPS。** 特に opener、section header、
  "WHAT I NEED FROM YOU" section、closing tagline では強めに書きます。
  ただし feature description を全部大文字にはしません。
- **highlight bullet 1 つにつき emoji shortcode 1 つ。** 機能を連想させるものを選びます。
  例: secrets は `:lock:`、planning は `:brain:`、search は `:mag:`、
  cloud / sandbox は `:cloud:`、plugins は `:jigsaw:`、history / restore は
  `:rewind:`、threads は `:thread:`。
- **highlight bullet は1文。** 意見をはっきりさせ、ユーザー視点で書きます。
  "added support for…" ではなく、"the cloud-secrets prereq is real now" のように書きます。
- **highlights の後の tail line** は残りを1文にまとめ、full release notes への
  link を付けます（"… and a long tail of {flavor}. Read the [full release notes](url)."）。
- **"WHATS NEXT" は将来のテーマ。** 実際の sprint list ではありません。
  3〜5 bullet が適切です。active goal、進行中の project、チームが取り組んでいる
  recent issue から引きます。創作しないでください。
- **Highlights は changelog skill の delta rule に従う。** 前回の告知で既に紹介した
  feature は、今回変わった点だけを載せ、継続作業を二度 headline にしません。
- **"What's on my mind"** は dotta の個人的 / 戦略的な考えです。
  docs のギャップ、哲学的な位置づけ（"we're the human control plane for ai labor"）、
  招待文（"if you've ever wanted to write about how you use Paperclip, hit me up"）などです。
  recent issue / comment から実際の tension を拾い、創作しません。
- **Press section** は任意です。release window に本物の press（tweet、podcast、talk、
  star milestone など）があるときだけ入れます。なければ section ごと削除します。
- **"WHAT I NEED FROM YOU"** も任意です。1つだけ具体的なお願い（twitter を follow して、
  紹介して、beta signup して、など）に使います。 real ask がなければ削除します。
- **Community** は changelog file と同じ contributor list を triple-backtick block に入れ、
  `@username, @username` のようにカンマ区切りにします。bot と changelog skill の
  canonical exclusion list にある人は除外します。
- **"In Summary" の mission line** はゆっくり変化します。dotta が別の指示を出していない限り、
  最新の文言を使います。最近の例:
  - "PAPERCLIP IS THE AI ORCHESTRATOR FOR HUMANS TO ACCOMPLISH 100x MORE WORK"
  - "PAPERCLIP WILL BE THE DEFAULT AGENT-MANAGEMENT TOOL FOR EVERY COMPANY"
  - "Paperclip will be _the_ control plane for AI agents in **every** company."
- **closing tagline** は常に `ITS TIME TO CLIP :paperclip: :paperclip: :paperclip:` です。
  そのまま残します。

## ワークフロー

1. Read the matching changelog produced by `release-changelog` — the
   beta-keyed file during the soak, `releases/vYYYY.MDD.P.md` once
   canonicalized. Use the version and contributor list from that file —
   never re-derive them.
2. Resolve the parent `release` case with key `paperclip-release:vYYYY.MDD.P`.
   If it does not exist and Cases are enabled, create it using the schema in
   `.agents/skills/release-changelog/SKILL.md` before creating child cases.
3. **release issue thread**（release routine を実行した、あなたに割り当てられた issue）
   を読みます。`WHATS NEXT` と `What's on my mind` の素材は、comment、linked issue、
   company 内の recent issue です。beta source commit の **後** にすでに `origin/master`
   にある commit は、次回 release の中身そのものなので、"what's next" の有力な素材です。
   作り話ではなく実際のテーマを拾ってください。
4. Re-read the three verbatim examples below — they're the canonical voice.
5. Draft the announcement using the template above.
6. release issue の `discord_announcement` document として PUT します（上の
   "Output" を参照）。更新する場合は最新の `baseRevisionId` を送ります。
7. Upsert the `tweet_storm` child case with `parentCaseId` set to the release
   case id, then PUT its `body` document to the announcement body.
8. release issue に comment を投稿し、告知文を1つの fenced markdown code block に
   入れます。そうすると dotta は document を開かずに Discord へコピーできます。

## Tweet Storm Case Schema

Use this child case for the Discord/social announcement thread. The key must be
stable so retries update the same child case:

```http
POST /api/companies/:companyId/cases
{
  "caseType": "tweet_storm",
  "key": "paperclip-release:vYYYY.MDD.P:tweet-storm",
  "title": "Paperclip vYYYY.MDD.P tweet storm",
  "summary": "Social announcement thread for Paperclip vYYYY.MDD.P.",
  "status": "in_review",
  "parentCaseId": "<release-case-id>",
  "fields": {
    "schema_version": 1,
    "version": "vYYYY.MDD.P",
    "channel": "x",
    "discord_source": true,
    "post_count": 1,
    "target_audience": ["operators", "contributors", "agent-company builders"],
    "links": {
      "release_notes": "https://github.com/paperclipai/paperclip/blob/master/releases/vYYYY.MDD.P.md",
      "official_account": "https://x.com/papercliping"
    },
    "review": {
      "needs_human_copy_paste": true,
      "approved_by": null
    }
  }
}
```

Then write the body document:

```http
PUT /api/cases/:tweetStormCaseId/documents/body
{
  "title": "Paperclip vYYYY.MDD.P tweet storm body",
  "format": "markdown",
  "body": "<announcement body>",
  "changeSummary": "Draft social announcement"
}
```

If updating an existing document, fetch the case and pass the latest
`baseRevisionId`.

Discord には投稿しません。この skill は artifact の準備だけを行います。

## 以前の例（原文そのまま）

dotta による過去3件の Discord 告知を、voice・構成・emoji 使用の
ground-truth として **原文そのまま** で載せています。迷ったらこれに合わせます。

### Example 1 — v2026.403.0

```
CLIPPERS! v2026.403.0 has dropped!! :paperclip: :paperclip: :paperclip:

## Highlights

:inbox_tray:  **Inbox overhaul** - there is a new "mine" tab that has mail-client like keyboard shortcuts. It's my new default view for managing work
:thumbsup:  **Feedback and evals** - you can now vote :thumbsup: / :thumbsdown: on your agent's responses. If you choose to share your traces with me, I'll use it to make Paperclip better. In either case you can export locally for your own org's learning
:page_with_curl:  **Document revisions** - you can now restore old versions of your documents
:ping_pong:  **Telemetry** - this version has anonymized telemetry that helps me better understand the basic uses of Paperclip (adapters and so on) - if you hate that, just it disable with `DO_NOT_TRACK=1` or `PAPERCLIP_TELEMETRY_DISABLED=1` environment variables
:construction_worker: **Execution Workspaces (experimental)** - Paperclip is not a "code review" tool, but I have been finding worktrees are important for certain projects. Enable it in experimental settings
:loop:  **Routine variables** - sometimes you need to customize a routine and the new variables feature makes that easy

PLUS **tons** of improvements aound adapters, bugfixes, qol

## COMMUNITY

HUGE THANKS to the contributors with commits in this release:

```
@aronprins, @bittoby, @edimuj, @HenkDz, @kevmok, @mvanhorn, @radiusred, @remdev, @statxc, @vanductai
```

## WHATS NEXT (ROADMAP)

* **Multi-human users** -- you've been asking for it, we have a draft and will have this shortly
* **Sandbox execution** - the other half of cloud deployment: run your agents in a sandbox across any provider

PLUS: just dealing with the excellent PRs we have sitting in our inbox.

**What's also on my mind (coming soonish)**

* MAXIMIZER MODE - for when you've got a dream and tokens to burn
* Artifacts, work products, and deployments
* CEO Chat
* Stronger agent defaults

## PRESS

I've been doing my part to spread the word about Paperclip

* We talked to the incredible [Andrew Warner of Mixergy Fame](https://x.com/dotta/status/2039087507514507407)
* We gave a tutorial with the [inimitable Greg Isenberg](https://x.com/dotta/status/2037279902445994345)
* We met with the [Seed Club guys](https://x.com/dotta/status/2039020365926576377)
* We crossed [40k stars (46k now!)](https://x.com/dotta/status/2038638188227387613)
* ... and a couple others that will be released in a few days

## SUCCESS STORIES

* [Nevo made $76k in march](https://x.com/dotta/status/2039406772859920758) after using Paperclip to automate his marketing
* [Lewis Jackson](https://x.com/WhatSayLew/status/2039810227394978158) said 34 agents were already operating his trading firm through Paperclip and called it his "holy s***" AI moment.
* [Neal Kotak](https://x.com/nkotak1/status/2039582439459209638) said Paperclip already runs most of Roominary for him and praised how strong the product is.
* [Sam Woods](https://x.com/samwoods/status/2039039305960587755) said he knows several people who moved from OpenClaw to Paperclip, often with Hermes in the stack, and that they love it.
* [Josh Galt](https://x.com/JoshGalt/status/2039386307219095557) called Paperclip the coolest agent tooling he has used and said it is finally something that just works.

## IN SUMMARY

I know there are still some rough edges, but

Paperclip will be *the* control plane for AI agents in **every** company.

and I think we're moving at a pretty good clip :paperclip: :paperclip: :paperclip:

FULL RELEASE NOTES HERE

https://github.com/paperclipai/paperclip/releases/tag/v2026.403.0

||@everyone||
```

### Example 2 — v2026.416.0

```
:paperclip: :paperclip: :paperclip: CLIPPERS!!! v2026.416.0 IS OUT :paperclip: :paperclip: :paperclip:

## Highlights

This release has *tons* of quality of life improvements around speed, performance, and workflow. You should notice that comment threads feel faster and your agents stay on task longer

:thread: Issue chat threads now are a conversation more than comments
:police_officer: Execution policies like **Reviewer** and **Approver** are now first-class in the harness (e.g. enforce that QA *must* review a task)
:no_smoking: Blocker dependencies - first-class "wake on blocker resolved" which means now you can have "task graphs" that depend on one another and it's enforced by Paperclip
:woman_feeding_baby: Parent-child tasks - better support for sub-tasks all around, which makes it much easier to organize your work

And then a million fixes around ux, details, keyboard shortcuts, bug fixes, security fixes, etc. Really you should read the [full release notes here](https://github.com/paperclipai/paperclip/releases/tag/v2026.416.0)

## COMMUNITY

INCREDIBLE INCREDIBLE WORK BY folks with commits and reports in this release:

```
@AllenHyang, @antonio-mello-ai, @aronprins, @chrisschwer, @cleanunicorn, @DanielSousa, @davison, @ergonaworks, @HearthCore, @HenkDz, @KhairulA, @kimnamu, @Lempkey, @marysomething99-prog, @mvanhorn, @officialasishkumar, @plind-dm, @shoaib050326, @sparkeros, @wbelt, @offset, @sagilayani, @mattdonnelly10, @peaktwilight, @YuvalElbar6
```

## WHATS NEXT (:motorway:  Roadmap)

* **Multi-human users** - in the last stages of testing, Paperclip is better with teams
* **Memory Infrastructure** - your agents will remember everything about yoru business
* **Sandbox execution** - run your agents anywhere

## What's on my mind

* I want to meet with companies who are using Paperclip in their business - if that's you let me know
* We need more Paperclip tutorials, defaults, and education - thanks to @aronprins for his work in this area already!
* We still need to get better at reviewing your PRs and we're improving our process every day
* "Zero-human company" language has to go - we're the human control plane for ai labor
* We're adding better support for *knowledge (wikis & files)*, *artifacts*, and *work product* in Paperclip soon.

## PRESS

* **AI Engineer Europe Tutorial** - I gave a tutorial for AIE. If someone is looking for a basics ABC of Paperclip [you can send them this](https://x.com/dotta/status/2044575580264316931)
* **AI Club Chicago** - JB gave a talk on Paperclip [at AI Tinkerers in Chicago](https://x.com/developwithJB/status/2044281068778316268) !

## IN SUMMARY

PAPERCLIP WILL BE THE DEFAULT AGENT-MANAGEMENT TOOL FOR EVERY COMPANY

If there's anything I can do to help you and your company use Paperclip, hit me up. Until then, enjoy the new release

ITS TIME TO CLIP :paperclip: :paperclip: :paperclip:

FULL RELEASE NOTES

https://github.com/paperclipai/paperclip/releases/tag/v2026.416.0

||@everyone||
```

### Example 3 — v2026.427.0

```
:paperclip: :paperclip: :paperclip: CLIPPERS!!! v2026.427.0 IS OUT :paperclip: :paperclip: :paperclip:

THIS IS THE OFFICIAL TWITTER FOLLOW IT: https://x.com/papercliping

## Highlights

:man_feeding_baby: **MULTI USER** - you can now invite multiple users to your instance
:factory_worker:  **HARDER WORKING** - robosut liveness continuations and lifecycle recovery means your instance tries harder before involving you
:white_check_mark:  **SUBISSUE CHECKLISTS** - subissues have better ordering which allows for long-run planning
:thread: **Rich Thread UX** - now your agents can ask you questions, ask for approvals, suggest tasks and you can approve or refine them right in your task threads
:cloud:  **BETA: Sandbox Providers** - Cloud sandboxing is in beta - the API ships in this release and we'll be adding more providers
... and *tons* of other improvements and bugfixes.

## Community

Thank you to everyone who contributed to this release!

```
@akhater, @aronprins, @GodsBoy, @LeonSGP43, @neerazz, @NoronhaH, @rbarinov, @rvanduiven, @SgtPooki, @superbiche
```

## WHATS NEXT (:motorway:  Roadmap)

* **Longer-range planning and execution** - Paperclip will support longer and longer tasks and work until it's done
* **Secrets Service v2** - an important prereq for Paperclip cloud
* **Artifacts, memory, and knowledge**
* **Conference Room** aka CEO/Agent Chat

## What's on my mind

* **Documentation & Blog posts** - I've fallen behind on the docs but aron has done a good job here - we'll be setting up Clips to help maintain these
* **Paperclip Cloud** - will be a critical unlock for us, but even the shared team story needs developed more - *where should the work be done* and *where are the outputs stored* and *how do we surface them to users*? Each of these questions are a core Paperclip service that needs developed
* **Paperclip Bench** - In the vein of SWE-Bench I've started an internal benchmark for Paperclip - we have to be able to measure that our changes are improving the system and not regressing
* **Paperclip Connections Store** - connecting to Github, Slack, Google Docs, and the hundreds of other services we use every day should be easy, secure, and configurable per agent and team

## Press

I met with the [Wisemen about Paperclip](https://x.com/dotta/status/2045146539534827998)

## WHAT I NEED FROM YOU

FOLLOW THIS TWITTER ACCOUNT: https://x.com/papercliping - that's the only official one, report any others

## In Summary

PAPERCLIP IS THE AI ORCHESTRATOR FOR HUMANS TO ACCOMPLISH 100x MORE WORK

Every single person will be managing a team of a dozen, or a hundred, or a thousand agents and Paperclip will be the default tool to manage it all.

ITS TIME TO CLIP :paperclip: :paperclip: :paperclip:

FULL RELEASE NOTES

https://github.com/paperclipai/paperclip/blob/master/releases/v2026.427.0.md

||@everyone||
```

## Review checklist

Before handing off:

1. Version + date match the matching `releases/vYYYY.MDD.P.md` exactly.
2. Contributor list matches the changelog (same exclusions: bots and the
   changelog skill's canonical excluded-folks list).
3. Highlights are a subset of the changelog Highlights — same shipped features,
   not invented or pre-alpha work.
4. `WHATS NEXT` and `What's on my mind` are pulled from real recent issues /
   active goals — not invented themes.
5. Section style (UPPERCASE vs Title Case) is internally consistent.
6. Closing tagline is `ITS TIME TO CLIP :paperclip: :paperclip: :paperclip:`
   and `||@everyone||` is the very last line.
7. Document `discord_announcement` is updated on the release issue, and the
   announcement is also posted in a comment inside a fenced code block.

This skill never posts to Discord. It only prepares the announcement artifact.

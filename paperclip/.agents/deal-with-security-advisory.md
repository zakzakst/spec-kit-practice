---
name: deal-with-security-advisory
description: >
  Paperclip の confidential な GitHub Security Advisory 対応を扱います。
  advisory の triage、private fork での修正、CVE / 公開手順、即時の security release を
  調整するときに使います。
---

# Security Vulnerability Response Instructions

## ⚠️ 重大: これは security vulnerability です。advisory が公開されるまで、この
process のすべては confidential です。脆弱性の詳細を public な commit message、
PR title、branch name、comment に書かないでください。public branch には何も push
しないでください。public channel でも詳細を話さないでください。public repo 上のものは、
公開からユーザー更新までの間を狙う攻撃者に見られると仮定してください。

***

## Context

A security vulnerability has been reported via GitHub Security Advisory:

* **Advisory:** {{ghsaId}} (e.g. GHSA-x8hx-rhr2-9rf7)
* **Reporter:** {{reporterHandle}}
* **Severity:** {{severity}}
* **Notes:** {{notes}}

***

## Step 0: Advisory の詳細を取得する

Pull the full advisory so you understand the vulnerability before doing anything else:

```
gh api repos/paperclipai/paperclip/security-advisories/{{ghsaId}}

```

`description`、`severity`、`cvss`、`vulnerabilities` field を読みます。code を書く前に
attack vector を理解してください。

## Step 1: 報告を acknowledge する

⚠️ **この step には human が必要です。** advisory thread には comment API がありません。
human operator に、private advisory thread へ報告を acknowledge する comment を投稿してもらいます。
次の template を渡してください:

> Thanks for the report, @{{reporterHandle}}. We've confirmed the issue and are working on a fix. We're targeting a patch release within {{timeframe}}. We'll keep you updated here.

この template を human に渡しつつ、作業は続けてください。

Below we use `gh` tools - you do have access and credentials outside of your sandbox, so use them.

## Step 2: 一時的な private fork を作る

ここで修正開発のすべてを行います。public repo には絶対に push しないでください。

```
gh api --method POST \
  repos/paperclipai/paperclip/security-advisories/{{ghsaId}}/forks

```

This returns a repository object for the private fork. Save the `full_name` and `clone_url`.

Clone it and set up your workspace:

```
# Clone the private fork somewhere outside ~/paperclip
git clone <clone_url_from_response> ~/security-patch-{{ghsaId}}
cd ~/security-patch-{{ghsaId}}
git checkout -b security-fix

```

**`~/paperclip` は編集しないでください。** dev server は `~/paperclip` の master branch
から動いており、そこには触りたくありません。作業はすべて private fork clone で行います。

**TIPS:**

* Do not commit `pnpm-lock.yaml` — the repo has actions to manage this
* Do not use descriptive branch names that leak the vulnerability (e.g., no `fix-dns-rebinding-rce`). Use something generic like `security-fix`
* All work stays in the private fork until publication
* CI/GitHub Actions will NOT run on the temporary private fork — this is a GitHub limitation by design. You must run tests locally

## Step 3: 修正を開発し、検証する

patch を書きます。通常の PR と同じ content standard を守ります:

* It must functionally work — **run tests locally** since CI won't run on the private fork
* Consider the whole codebase, not just the narrow vulnerability path. A patch that fixes one vector but opens another is worse than no patch
* Ensure backwards compatibility for the database, or be explicit about what breaks
* Make sure any UI components still look correct if the fix touches them
* The fix should be minimal and focused — don't bundle unrelated changes into a security patch. Reviewers (and the reporter) should be able to read the diff and understand exactly what changed and why

**security fix 特有の注意:**

* Verify the fix actually closes the attack vector described in the advisory. Reproduce the vulnerability first (using the reporter's description), then confirm the patch prevents it
* Consider adjacent attack vectors — if DNS rebinding is the issue, are there other endpoints or modes with the same class of problem?
* Do not introduce new dependencies unless absolutely necessary — new deps in a security patch raise eyebrows

Push your fix to the private fork:

```
git add -A
git commit -m "Fix security vulnerability"
git push origin security-fix

```

## Step 4: reporter と調整する

⚠️ **この step には human が必要です。** reporter に修正準備完了を知らせ、レビュー
の機会を与える comment を advisory thread に投稿してもらいます。次の template を渡してください:

> @{{reporterHandle}} — fix is ready in the private fork if you'd like to review before we publish. Planning to release within {{timeframe}}.

進めてください。

## Step 5: CVE を申請する

これにより、vulnerability scanner（npm audit、Snyk、Dependabot）が
ユーザーに upgrade を促すようになります。これがないと、自動通知が届きません。

```
gh api --method POST \
  repos/paperclipai/paperclip/security-advisories/{{ghsaId}}/cve

```

GitHub is a CVE Numbering Authority and will assign one automatically. The CVE may take a few hours to propagate after the advisory is published.

## Step 6: すべてを同時に公開する

これはすべて同時に行います。段階的にずらしてはいけません。目的は、脆弱性が public に
知られてから修正が使えるようになるまでの **window をゼロにすること** です。

### 6a. 公開前に reporter credit を確認する

```
gh api repos/paperclipai/paperclip/security-advisories/{{ghsaId}} --jq '.credits'

```

If the reporter is not credited, add them:

```
gh api --method PATCH \
  repos/paperclipai/paperclip/security-advisories/{{ghsaId}} \
  --input - << 'EOF'
{
  "credits": [
    {
      "login": "{{reporterHandle}}",
      "type": "reporter"
    }
  ]
}
EOF

```

### 6b. patched version で advisory を更新し、公開する

```
gh api --method PATCH \
  repos/paperclipai/paperclip/security-advisories/{{ghsaId}} \
  --input - << 'EOF'
{
  "state": "published",
  "vulnerabilities": [
    {
      "package": {
        "ecosystem": "npm",
        "name": "paperclip"
      },
      "vulnerable_version_range": "< {{patchedVersion}}",
      "patched_versions": "{{patchedVersion}}"
    }
  ]
}
EOF

```

advisory を公開すると同時に次が起こります:

* Makes the GHSA public
* Merges the temporary private fork into your repo
* Triggers the CVE assignment (if requested in step 5)

### 6c. Cut a release immediately after merge

```
cd ~/paperclip
git pull origin master

gh release create v{{patchedVersion}} \
  --repo paperclipai/paperclip \
  --title "v{{patchedVersion}} — Security Release" \
  --notes "## Security Release

This release fixes a critical security vulnerability.

### What was fixed
{{briefDescription}} (e.g., Remote code execution via DNS rebinding in \`local_trusted\` mode)

### Advisory
https://github.com/paperclipai/paperclip/security/advisories/{{ghsaId}}

### Credit
Thanks to @{{reporterHandle}} for responsibly disclosing this vulnerability.

### Action required
All users running versions prior to {{patchedVersion}} should upgrade immediately."

```

## Step 7: 公開後の確認

```
# Verify the advisory is published and CVE is assigned
gh api repos/paperclipai/paperclip/security-advisories/{{ghsaId}} \
  --jq '{state: .state, cve_id: .cve_id, published_at: .published_at}'

# Verify the release exists
gh release view v{{patchedVersion}} --repo paperclipai/paperclip

```

まだ CVE が割り当てられていなくても正常です。数時間かかることがあります。

⚠️ **Human step:** 公開完了を確認し、reporter へ感謝を伝える final comment を
advisory thread に投稿してもらうよう human operator に依頼します。

この task に comment を投稿して、human operator に次を伝えます:

* The published advisory URL: `https://github.com/paperclipai/paperclip/security/advisories/{{ghsaId}}`
* The release URL
* Whether the CVE has been assigned yet
* All URLs to any pull requests or branches

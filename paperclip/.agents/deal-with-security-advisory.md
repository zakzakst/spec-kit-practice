---
name: deal-with-security-advisory
description: >
  Paperclip の confidential な GitHub Security Advisory 対応を扱います。
  advisory の triage、private fork での修正、CVE / 公開手順、即時の security release を
  調整するときに使います。
---

# Security Vulnerability Response Instructions

## ⚠️ 重大: これは security vulnerability です

advisory が公開されるまで、この process のすべては confidential です。脆弱性の詳細を public な commit message、PR title、branch name、comment に書かないでください。public branch には何も push しないでください。public channel でも詳細を話さないでください。public repo 上のものは、公開からユーザー更新までの間を狙う攻撃者に見られる前提で扱ってください。

***

## Context

GitHub Security Advisory 経由で security vulnerability が報告されています:

* **Advisory:** `{{ghsaId}}`（例: `GHSA-x8hx-rhr2-9rf7`）
* **Reporter:** `{{reporterHandle}}`
* **Severity:** `{{severity}}`
* **Notes:** `{{notes}}`

***

## Step 0: Advisory の詳細を取得する

まず何より先に、脆弱性を理解するために advisory 全体を取得します:

```bash
gh api repos/paperclipai/paperclip/security-advisories/{{ghsaId}}
```

`description`、`severity`、`cvss`、`vulnerabilities` field を読みます。code を書く前に attack vector を理解してください。

## Step 1: 報告を acknowledge する

⚠️ **この step には human が必要です。** advisory thread には comment API がありません。human operator に、private advisory thread へ報告を acknowledge する comment を投稿してもらいます。次の template を渡してください:

> Thanks for the report, @{{reporterHandle}}. We've confirmed the issue and are working on a fix. We're targeting a patch release within {{timeframe}}. We'll keep you updated here.

この template を human に渡しつつ、作業は続けてください。

Below we use `gh` tools - you do have access and credentials outside of your sandbox, so use them.

## Step 2: 一時的な private fork を作る

ここで修正開発のすべてを行います。public repo には絶対に push しないでください。

```bash
gh api --method POST \
  repos/paperclipai/paperclip/security-advisories/{{ghsaId}}/forks
```

これは private fork の repository object を返します。`full_name` と `clone_url` を保存してください。

それを clone して workspace を準備します:

```bash
# Clone the private fork somewhere outside ~/paperclip
git clone <clone_url_from_response> ~/security-patch-{{ghsaId}}
cd ~/security-patch-{{ghsaId}}
git checkout -b security-fix
```

**`~/paperclip` は編集しないでください。** dev server は `~/paperclip` の master branch から動いており、そこには触りたくありません。作業はすべて private fork clone で行います。

**TIPS:**

* `pnpm-lock.yaml` は commit しないこと。repo 側の Actions が管理しています。
* 脆弱性の内容を漏らす branch 名は使わないでください（例: `fix-dns-rebinding-rce` は避け、`security-fix` のような一般的な名前にします）。
* すべての作業は公開前に private fork の中だけで行います。
* CI / GitHub Actions は一時的な private fork では **実行されません**。GitHub の制約です。テストはローカルで実行してください。

## Step 3: 修正を開発し、検証する

patch を書きます。通常の PR と同じ品質基準を守ってください:

* 実際に動くこと - private fork では CI が走らないので、必ずローカルでテストを実行すること
* codebase 全体を見て考えること。脆弱性を1つ塞いでも別の経路を開くなら、修正としては不十分です
* database 互換性を確保するか、壊れる場合は明示すること
* UI に触れるなら、見た目が崩れていないことを確認すること
* 修正は最小限かつ焦点を絞ること。security patch に無関係な変更を詰め込まないでください。reviewer（と reporter）が diff を読んで、何がどう変わったのかを理解できるようにします

**security fix 固有の注意:**

* advisory で説明された attack vector を本当に閉じていることを確認してください。まず脆弱性を再現し、その後 patch で防げることを確認します
* 隣接する attack vector も考慮してください。DNS rebinding が問題なら、同じクラスの問題を持つ endpoint や mode は他にもないか確認します
* 新しい dependency は、どうしても必要な場合を除いて追加しないでください。security patch で dependency が増えると警戒されます

修正を private fork に push します:

```bash
git add -A
git commit -m "Fix security vulnerability"
git push origin security-fix
```

## Step 4: reporter と調整する

⚠️ **この step には human が必要です。** reporter に修正準備完了を知らせ、review の機会を与える comment を advisory thread に投稿してもらいます。次の template を渡してください:

> @{{reporterHandle}} — fix is ready in the private fork if you'd like to review before we publish. Planning to release within {{timeframe}}.

進めてください。

## Step 5: CVE を申請する

これにより、vulnerability scanner（npm audit、Snyk、Dependabot）がユーザーに upgrade を促すようになります。これがないと、自動通知が届きません。

```bash
gh api --method POST \
  repos/paperclipai/paperclip/security-advisories/{{ghsaId}}/cve
```

GitHub は CVE Numbering Authority なので、自動で1つ割り当てます。advisory が公開されてから CVE が反映されるまで、数時間かかることがあります。

## Step 6: すべてを同時に公開する

これはすべて同時に行います。段階的にずらしてはいけません。目的は、脆弱性が public に知られてから修正が使えるようになるまでの **window をゼロにすること** です。

### 6a. 公開前に reporter credit を確認する

```bash
gh api --method PATCH \
  repos/paperclipai/paperclip/security-advisories/{{ghsaId}} \
  --input - << 'EOF'
{
  "collaborating_users": ["{{reporterHandle}}"]
}
EOF
```

### 6b. patched version で advisory を更新し、公開する

```bash
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

* GHSA が public になる
* temporary private fork が repo に merge される
* Step 5 で依頼していれば CVE 割り当てが走る

### 6c. merge 直後に release を切る

```bash
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

```bash
# Verify the advisory is published and CVE is assigned
gh api repos/paperclipai/paperclip/security-advisories/{{ghsaId}} \
  --jq '{state: .state, cve_id: .cve_id, published_at: .published_at}'

# Verify the release exists
gh release view v{{patchedVersion}} --repo paperclipai/paperclip
```

まだ CVE が割り当てられていなくても正常です。数時間かかることがあります。

⚠️ **Human step:** 公開完了を確認し、reporter へ感謝を伝える final comment を advisory thread に投稿してもらうよう human operator に依頼します。

この task に comment を投稿して、human operator に次を伝えます:

* 公開された advisory URL: `https://github.com/paperclipai/paperclip/security/advisories/{{ghsaId}}`
* release URL
* CVE がすでに割り当てられているかどうか
* 関連する PR や branch の URL 一覧

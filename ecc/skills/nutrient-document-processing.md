---
name: nutrient-document-processing
description: Nutrient DWS API を使って、文書の処理、変換、OCR、抽出、墨消し、署名、フォーム入力を行います。PDF、DOCX、XLSX、PPTX、HTML、画像に対応します。
origin: ECC
---

# Nutrient 文書処理

> **Note:** このスキルは Nutrient の商用 API と統合します。利用前に利用規約を確認してください。

[Nutrient DWS Processor API](https://www.nutrient.io/api/) で文書を処理します。形式変換、テキスト / 表の抽出、スキャン文書の OCR、個人情報の墨消し、透かし追加、電子署名、PDF フォーム入力を扱えます。

## セットアップ

無料の API キーを **[nutrient.io](https://dashboard.nutrient.io/sign_up/?product=processor)** で取得します。

```bash
export NUTRIENT_API_KEY="pdf_live_..."
```

すべてのリクエストは `https://api.nutrient.io/build` へ multipart POST し、`instructions` JSON フィールドを含めます。

## 操作

### 文書変換

```bash
# DOCX to PDF
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.docx=@document.docx" \
  -F 'instructions={"parts":[{"file":"document.docx"}]}' \
  -o output.pdf

# PDF to DOCX
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"docx"}}' \
  -o output.docx

# HTML to PDF
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "index.html=@index.html" \
  -F 'instructions={"parts":[{"html":"index.html"}]}' \
  -o output.pdf
```

対応入力: PDF, DOCX, XLSX, PPTX, DOC, XLS, PPT, PPS, PPSX, ODT, RTF, HTML, JPG, PNG, TIFF, HEIC, GIF, WebP, SVG, TGA, EPS。

### テキストとデータ抽出

```bash
# Extract plain text
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"text"}}' \
  -o output.txt

# Extract tables as Excel
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"xlsx"}}' \
  -o tables.xlsx
```

### スキャン文書の OCR

```bash
# OCR to searchable PDF (supports 100+ languages)
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "scanned.pdf=@scanned.pdf" \
  -F 'instructions={"parts":[{"file":"scanned.pdf"}],"actions":[{"type":"ocr","language":"english"}]}' \
  -o searchable.pdf
```

言語: ISO 639-2 コードで 100 以上に対応（例: `eng`, `deu`, `fra`, `spa`, `jpn`, `kor`, `chi_sim`, `chi_tra`, `ara`, `hin`, `rus`）。`english` や `german` のような完全名も使える。対応一覧は [complete OCR language table](https://www.nutrient.io/guides/document-engine/ocr/language-support/) を参照。

### 機微情報の墨消し

```bash
# Pattern-based (SSN, email)
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"redaction","strategy":"preset","strategyOptions":{"preset":"social-security-number"}},{"type":"redaction","strategy":"preset","strategyOptions":{"preset":"email-address"}}]}' \
  -o redacted.pdf

# Regex-based
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"redaction","strategy":"regex","strategyOptions":{"regex":"\\b[A-Z]{2}\\d{6}\\b"}}]}' \
  -o redacted.pdf
```

プリセット: `social-security-number`, `email-address`, `credit-card-number`, `international-phone-number`, `north-american-phone-number`, `date`, `time`, `url`, `ipv4`, `ipv6`, `mac-address`, `us-zip-code`, `vin`。

### 透かし追加

```bash
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"watermark","text":"CONFIDENTIAL","fontSize":72,"opacity":0.3,"rotation":-45}]}' \
  -o watermarked.pdf
```

### 電子署名

```bash
# Self-signed CMS signature
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"sign","signatureType":"cms"}]}' \
  -o signed.pdf
```

### PDF フォーム入力

```bash
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "form.pdf=@form.pdf" \
  -F 'instructions={"parts":[{"file":"form.pdf"}],"actions":[{"type":"fillForm","formFields":{"name":"Jane Smith","email":"jane@example.com","date":"2026-02-06"}}]}' \
  -o filled.pdf
```

## MCP Server（代替手段）

curl の代わりに native なツール統合が必要なら MCP server を使う:

```json
{
  "mcpServers": {
    "nutrient-dws": {
      "command": "npx",
      "args": ["-y", "@nutrient-sdk/dws-mcp-server"],
      "env": {
        "NUTRIENT_DWS_API_KEY": "YOUR_API_KEY",
        "SANDBOX_PATH": "/path/to/working/directory"
      }
    }
  }
}
```

## 使いどころ

- 形式間の文書変換（PDF、DOCX、XLSX、PPTX、HTML、画像）
- PDF からテキスト、表、key-value を抽出するとき
- スキャン文書や画像への OCR
- 文書共有前の個人情報墨消し
- 下書きや機密文書への透かし追加
- 契約書や合意文書への電子署名
- PDF フォームのプログラム入力

## リンク

- [API Playground](https://dashboard.nutrient.io/processor-api/playground/)
- [Full API Docs](https://www.nutrient.io/guides/dws-processor/)
- [npm MCP Server](https://www.npmjs.com/package/@nutrient-sdk/dws-mcp-server)

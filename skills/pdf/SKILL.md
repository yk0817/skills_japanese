---
name: pdf
description: PDFファイルに関するあらゆる操作を行う際に使用するスキルです。PDFからのテキストや表の読み取り・抽出、複数PDFの結合・マージ、PDFの分割、ページの回転、透かしの追加、新規PDFの作成、PDFフォームの入力、PDFの暗号化・復号化、画像の抽出、スキャンされたPDFのOCR処理による検索可能化などが含まれます。ユーザーが.pdfファイルに言及したり、PDFの生成を依頼した場合に使用してください。
license: Proprietary. LICENSE.txt has complete terms
---

# PDF処理ガイド

## 概要

このガイドでは、Pythonライブラリとコマンドラインツールを使用した基本的なPDF処理操作を説明します。高度な機能、JavaScriptライブラリ、詳細な例についてはREFERENCE.mdを参照してください。PDFフォームに記入する必要がある場合は、FORMS.mdを読み、その指示に従ってください。

## クイックスタート

```python
from pypdf import PdfReader, PdfWriter

# Read a PDF
reader = PdfReader("document.pdf")
print(f"Pages: {len(reader.pages)}")

# Extract text
text = ""
for page in reader.pages:
    text += page.extract_text()
```

## Pythonライブラリ

### pypdf - 基本操作

#### PDFの結合
```python
from pypdf import PdfWriter, PdfReader

writer = PdfWriter()
for pdf_file in ["doc1.pdf", "doc2.pdf", "doc3.pdf"]:
    reader = PdfReader(pdf_file)
    for page in reader.pages:
        writer.add_page(page)

with open("merged.pdf", "wb") as output:
    writer.write(output)
```

#### PDFの分割
```python
reader = PdfReader("input.pdf")
for i, page in enumerate(reader.pages):
    writer = PdfWriter()
    writer.add_page(page)
    with open(f"page_{i+1}.pdf", "wb") as output:
        writer.write(output)
```

#### メタデータの抽出
```python
reader = PdfReader("document.pdf")
meta = reader.metadata
print(f"Title: {meta.title}")
print(f"Author: {meta.author}")
print(f"Subject: {meta.subject}")
print(f"Creator: {meta.creator}")
```

#### ページの回転
```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

page = reader.pages[0]
page.rotate(90)  # Rotate 90 degrees clockwise
writer.add_page(page)

with open("rotated.pdf", "wb") as output:
    writer.write(output)
```

### pdfplumber - テキストと表の抽出

#### レイアウトを保持したテキスト抽出
```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    for page in pdf.pages:
        text = page.extract_text()
        print(text)
```

#### 表の抽出
```python
with pdfplumber.open("document.pdf") as pdf:
    for i, page in enumerate(pdf.pages):
        tables = page.extract_tables()
        for j, table in enumerate(tables):
            print(f"Table {j+1} on page {i+1}:")
            for row in table:
                print(row)
```

#### 高度な表抽出
```python
import pandas as pd

with pdfplumber.open("document.pdf") as pdf:
    all_tables = []
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            if table:  # Check if table is not empty
                df = pd.DataFrame(table[1:], columns=table[0])
                all_tables.append(df)

# Combine all tables
if all_tables:
    combined_df = pd.concat(all_tables, ignore_index=True)
    combined_df.to_excel("extracted_tables.xlsx", index=False)
```

### reportlab - PDF作成

#### 基本的なPDF作成
```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

c = canvas.Canvas("hello.pdf", pagesize=letter)
width, height = letter

# Add text
c.drawString(100, height - 100, "Hello World!")
c.drawString(100, height - 120, "This is a PDF created with reportlab")

# Add a line
c.line(100, height - 140, 400, height - 140)

# Save
c.save()
```

#### 複数ページのPDF作成
```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.styles import getSampleStyleSheet

doc = SimpleDocTemplate("report.pdf", pagesize=letter)
styles = getSampleStyleSheet()
story = []

# Add content
title = Paragraph("Report Title", styles['Title'])
story.append(title)
story.append(Spacer(1, 12))

body = Paragraph("This is the body of the report. " * 20, styles['Normal'])
story.append(body)
story.append(PageBreak())

# Page 2
story.append(Paragraph("Page 2", styles['Heading1']))
story.append(Paragraph("Content for page 2", styles['Normal']))

# Build PDF
doc.build(story)
```

#### 下付き文字と上付き文字

**重要**: ReportLabのPDFでは、Unicode下付き文字・上付き文字（₀₁₂₃₄₅₆₇₈₉、⁰¹²³⁴⁵⁶⁷⁸⁹）を絶対に使用しないでください。内蔵フォントにはこれらのグリフが含まれておらず、黒い四角として表示されます。

代わりに、ParagraphオブジェクトでReportLabのXMLマークアップタグを使用してください：
```python
from reportlab.platypus import Paragraph
from reportlab.lib.styles import getSampleStyleSheet

styles = getSampleStyleSheet()

# Subscripts: use <sub> tag
chemical = Paragraph("H<sub>2</sub>O", styles['Normal'])

# Superscripts: use <super> tag
squared = Paragraph("x<super>2</super> + y<super>2</super>", styles['Normal'])
```

canvasで描画するテキスト（Paragraphオブジェクト以外）では、Unicode下付き文字・上付き文字の代わりに、フォントサイズと位置を手動で調整してください。

## コマンドラインツール

### pdftotext (poppler-utils)
```bash
# Extract text
pdftotext input.pdf output.txt

# Extract text preserving layout
pdftotext -layout input.pdf output.txt

# Extract specific pages
pdftotext -f 1 -l 5 input.pdf output.txt  # Pages 1-5
```

### qpdf
```bash
# Merge PDFs
qpdf --empty --pages file1.pdf file2.pdf -- merged.pdf

# Split pages
qpdf input.pdf --pages . 1-5 -- pages1-5.pdf
qpdf input.pdf --pages . 6-10 -- pages6-10.pdf

# Rotate pages
qpdf input.pdf output.pdf --rotate=+90:1  # Rotate page 1 by 90 degrees

# Remove password
qpdf --password=mypassword --decrypt encrypted.pdf decrypted.pdf
```

### pdftk（利用可能な場合）
```bash
# Merge
pdftk file1.pdf file2.pdf cat output merged.pdf

# Split
pdftk input.pdf burst

# Rotate
pdftk input.pdf rotate 1east output rotated.pdf
```

## よくあるタスク

### スキャンされたPDFからのテキスト抽出
```python
# Requires: pip install pytesseract pdf2image
import pytesseract
from pdf2image import convert_from_path

# Convert PDF to images
images = convert_from_path('scanned.pdf')

# OCR each page
text = ""
for i, image in enumerate(images):
    text += f"Page {i+1}:\n"
    text += pytesseract.image_to_string(image)
    text += "\n\n"

print(text)
```

### 透かしの追加
```python
from pypdf import PdfReader, PdfWriter

# Create watermark (or load existing)
watermark = PdfReader("watermark.pdf").pages[0]

# Apply to all pages
reader = PdfReader("document.pdf")
writer = PdfWriter()

for page in reader.pages:
    page.merge_page(watermark)
    writer.add_page(page)

with open("watermarked.pdf", "wb") as output:
    writer.write(output)
```

### 画像の抽出
```bash
# Using pdfimages (poppler-utils)
pdfimages -j input.pdf output_prefix

# This extracts all images as output_prefix-000.jpg, output_prefix-001.jpg, etc.
```

### パスワード保護
```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

for page in reader.pages:
    writer.add_page(page)

# Add password
writer.encrypt("userpassword", "ownerpassword")

with open("encrypted.pdf", "wb") as output:
    writer.write(output)
```

## クイックリファレンス

| タスク | 最適なツール | コマンド/コード |
|------|-----------|--------------|
| PDFの結合 | pypdf | `writer.add_page(page)` |
| PDFの分割 | pypdf | 1ページ1ファイル |
| テキスト抽出 | pdfplumber | `page.extract_text()` |
| 表の抽出 | pdfplumber | `page.extract_tables()` |
| PDF作成 | reportlab | CanvasまたはPlatypus |
| コマンドラインでの結合 | qpdf | `qpdf --empty --pages ...` |
| スキャンPDFのOCR | pytesseract | まず画像に変換 |
| PDFフォームの入力 | pdf-libまたはpypdf（FORMS.md参照） | FORMS.md参照 |

## 次のステップ

- pypdfium2の高度な使い方については、REFERENCE.mdを参照してください
- JavaScriptライブラリ（pdf-lib）については、REFERENCE.mdを参照してください
- PDFフォームに記入する必要がある場合は、FORMS.mdの指示に従ってください
- トラブルシューティングガイドについては、REFERENCE.mdを参照してください

---
name: docx
description: "Word文書（.docxファイル）の作成、読み取り、編集、操作を行う際に使用するスキルです。「Word文書」「Wordドキュメント」「.docx」への言及や、目次・見出し・ページ番号・レターヘッドなどの書式を含むプロフェッショナルな文書の作成依頼がトリガーとなります。また、.docxファイルからのコンテンツの抽出・再構成、文書内の画像の挿入・置換、Wordファイルでの検索・置換、変更履歴やコメントの操作、コンテンツを整ったWord文書に変換する際にも使用します。「レポート」「メモ」「手紙」「テンプレート」などをWordまたは.docxファイルとして依頼された場合に使用してください。PDF、スプレッドシート、Googleドキュメント、文書生成に関係しない一般的なコーディングタスクには使用しないでください。"
license: Proprietary. LICENSE.txt has complete terms
---

# DOCXの作成、編集、分析

## 概要

.docxファイルはXMLファイルを含むZIPアーカイブです。

## クイックリファレンス

| タスク | アプローチ |
|------|----------|
| コンテンツの読み取り・分析 | `pandoc`または展開して生XMLを参照 |
| 新規文書の作成 | `docx-js`を使用 - 下記「新規文書の作成」を参照 |
| 既存文書の編集 | 展開 → XML編集 → 再パック - 下記「既存文書の編集」を参照 |

### .docから.docxへの変換

レガシーの`.doc`ファイルは編集前に変換が必要です：

```bash
python scripts/office/soffice.py --headless --convert-to docx document.doc
```

### コンテンツの読み取り

```bash
# Text extraction with tracked changes
pandoc --track-changes=all document.docx -o output.md

# Raw XML access
python scripts/office/unpack.py document.docx unpacked/
```

### 画像への変換

```bash
python scripts/office/soffice.py --headless --convert-to pdf document.docx
pdftoppm -jpeg -r 150 document.pdf page
```

### 変更履歴の承認

すべての変更履歴を承認したクリーンな文書を生成するには（LibreOfficeが必要）：

```bash
python scripts/accept_changes.py input.docx output.docx
```

---

## 新規文書の作成

JavaScriptで.docxファイルを生成し、バリデーションを行います。インストール：`npm install -g docx`

### セットアップ
```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, ImageRun,
        Header, Footer, AlignmentType, PageOrientation, LevelFormat, ExternalHyperlink,
        InternalHyperlink, Bookmark, FootnoteReferenceRun, PositionalTab,
        PositionalTabAlignment, PositionalTabRelativeTo, PositionalTabLeader,
        TabStopType, TabStopPosition, Column, SectionType,
        TableOfContents, HeadingLevel, BorderStyle, WidthType, ShadingType,
        VerticalAlign, PageNumber, PageBreak } = require('docx');

const doc = new Document({ sections: [{ children: [/* content */] }] });
Packer.toBuffer(doc).then(buffer => fs.writeFileSync("doc.docx", buffer));
```

### バリデーション
ファイル作成後にバリデーションを行います。バリデーションに失敗した場合は、展開してXMLを修正し、再パックしてください。
```bash
python scripts/office/validate.py doc.docx
```

### ページサイズ

```javascript
// CRITICAL: docx-js defaults to A4, not US Letter
// Always set page size explicitly for consistent results
sections: [{
  properties: {
    page: {
      size: {
        width: 12240,   // 8.5 inches in DXA
        height: 15840   // 11 inches in DXA
      },
      margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } // 1 inch margins
    }
  },
  children: [/* content */]
}]
```

**一般的なページサイズ（DXA単位、1440 DXA = 1インチ）：**

| 用紙 | 幅 | 高さ | コンテンツ幅（1インチマージン） |
|-------|-------|--------|---------------------------|
| USレター | 12,240 | 15,840 | 9,360 |
| A4（デフォルト） | 11,906 | 16,838 | 9,026 |

**横向き：** docx-jsは内部的に幅と高さを入れ替えるため、縦向きの寸法を渡して入れ替えを任せてください：
```javascript
size: {
  width: 12240,   // Pass SHORT edge as width
  height: 15840,  // Pass LONG edge as height
  orientation: PageOrientation.LANDSCAPE  // docx-js swaps them in the XML
},
// Content width = 15840 - left margin - right margin (uses the long edge)
```

### スタイル（組み込み見出しのオーバーライド）

デフォルトフォントにはArialを使用してください（ユニバーサルサポート）。タイトルは読みやすさのため黒にしてください。

```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } }, // 12pt default
    paragraphStyles: [
      // IMPORTANT: Use exact IDs to override built-in styles
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 32, bold: true, font: "Arial" },
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 } }, // outlineLevel required for TOC
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, font: "Arial" },
        paragraph: { spacing: { before: 180, after: 180 }, outlineLevel: 1 } },
    ]
  },
  sections: [{
    children: [
      new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("Title")] }),
    ]
  }]
});
```

### リスト（Unicode箇条書き文字は絶対に使用しない）

```javascript
// ❌ WRONG - never manually insert bullet characters
new Paragraph({ children: [new TextRun("• Item")] })  // BAD
new Paragraph({ children: [new TextRun("\u2022 Item")] })  // BAD

// ✅ CORRECT - use numbering config with LevelFormat.BULLET
const doc = new Document({
  numbering: {
    config: [
      { reference: "bullets",
        levels: [{ level: 0, format: LevelFormat.BULLET, text: "•", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "numbers",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
    ]
  },
  sections: [{
    children: [
      new Paragraph({ numbering: { reference: "bullets", level: 0 },
        children: [new TextRun("Bullet item")] }),
      new Paragraph({ numbering: { reference: "numbers", level: 0 },
        children: [new TextRun("Numbered item")] }),
    ]
  }]
});

// ⚠️ Each reference creates INDEPENDENT numbering
// Same reference = continues (1,2,3 then 4,5,6)
// Different reference = restarts (1,2,3 then 1,2,3)
```

### テーブル

**重要：テーブルには二重の幅設定が必要です** - テーブルの`columnWidths`と各セルの`width`の両方を設定してください。両方がないと、一部のプラットフォームでテーブルが正しくレンダリングされません。

```javascript
// CRITICAL: Always set table width for consistent rendering
// CRITICAL: Use ShadingType.CLEAR (not SOLID) to prevent black backgrounds
const border = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const borders = { top: border, bottom: border, left: border, right: border };

new Table({
  width: { size: 9360, type: WidthType.DXA }, // Always use DXA (percentages break in Google Docs)
  columnWidths: [4680, 4680], // Must sum to table width (DXA: 1440 = 1 inch)
  rows: [
    new TableRow({
      children: [
        new TableCell({
          borders,
          width: { size: 4680, type: WidthType.DXA }, // Also set on each cell
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR }, // CLEAR not SOLID
          margins: { top: 80, bottom: 80, left: 120, right: 120 }, // Cell padding (internal, not added to width)
          children: [new Paragraph({ children: [new TextRun("Cell")] })]
        })
      ]
    })
  ]
})
```

**テーブル幅の計算：**

常に`WidthType.DXA`を使用してください。`WidthType.PERCENTAGE`はGoogleドキュメントで動作しません。

```javascript
// Table width = sum of columnWidths = content width
// US Letter with 1" margins: 12240 - 2880 = 9360 DXA
width: { size: 9360, type: WidthType.DXA },
columnWidths: [7000, 2360]  // Must sum to table width
```

**幅のルール：**
- **常に`WidthType.DXA`を使用** - `WidthType.PERCENTAGE`は使用しない（Googleドキュメントと非互換）
- テーブル幅は`columnWidths`の合計と一致する必要がある
- セルの`width`は対応する`columnWidth`と一致する必要がある
- セルの`margins`は内部パディング - コンテンツ領域を縮小するもので、セル幅には加算されない
- 全幅テーブルの場合：コンテンツ幅（ページ幅から左右のマージンを引いた値）を使用

### 画像

```javascript
// CRITICAL: type parameter is REQUIRED
new Paragraph({
  children: [new ImageRun({
    type: "png", // Required: png, jpg, jpeg, gif, bmp, svg
    data: fs.readFileSync("image.png"),
    transformation: { width: 200, height: 150 },
    altText: { title: "Title", description: "Desc", name: "Name" } // All three required
  })]
})
```

### 改ページ

```javascript
// CRITICAL: PageBreak must be inside a Paragraph
new Paragraph({ children: [new PageBreak()] })

// Or use pageBreakBefore
new Paragraph({ pageBreakBefore: true, children: [new TextRun("New page")] })
```

### ハイパーリンク

```javascript
// External link
new Paragraph({
  children: [new ExternalHyperlink({
    children: [new TextRun({ text: "Click here", style: "Hyperlink" })],
    link: "https://example.com",
  })]
})

// Internal link (bookmark + reference)
// 1. Create bookmark at destination
new Paragraph({ heading: HeadingLevel.HEADING_1, children: [
  new Bookmark({ id: "chapter1", children: [new TextRun("Chapter 1")] }),
]})
// 2. Link to it
new Paragraph({ children: [new InternalHyperlink({
  children: [new TextRun({ text: "See Chapter 1", style: "Hyperlink" })],
  anchor: "chapter1",
})]})
```

### 脚注

```javascript
const doc = new Document({
  footnotes: {
    1: { children: [new Paragraph("Source: Annual Report 2024")] },
    2: { children: [new Paragraph("See appendix for methodology")] },
  },
  sections: [{
    children: [new Paragraph({
      children: [
        new TextRun("Revenue grew 15%"),
        new FootnoteReferenceRun(1),
        new TextRun(" using adjusted metrics"),
        new FootnoteReferenceRun(2),
      ],
    })]
  }]
});
```

### タブストップ

```javascript
// Right-align text on same line (e.g., date opposite a title)
new Paragraph({
  children: [
    new TextRun("Company Name"),
    new TextRun("\tJanuary 2025"),
  ],
  tabStops: [{ type: TabStopType.RIGHT, position: TabStopPosition.MAX }],
})

// Dot leader (e.g., TOC-style)
new Paragraph({
  children: [
    new TextRun("Introduction"),
    new TextRun({ children: [
      new PositionalTab({
        alignment: PositionalTabAlignment.RIGHT,
        relativeTo: PositionalTabRelativeTo.MARGIN,
        leader: PositionalTabLeader.DOT,
      }),
      "3",
    ]}),
  ],
})
```

### 段組みレイアウト

```javascript
// Equal-width columns
sections: [{
  properties: {
    column: {
      count: 2,          // number of columns
      space: 720,        // gap between columns in DXA (720 = 0.5 inch)
      equalWidth: true,
      separate: true,    // vertical line between columns
    },
  },
  children: [/* content flows naturally across columns */]
}]

// Custom-width columns (equalWidth must be false)
sections: [{
  properties: {
    column: {
      equalWidth: false,
      children: [
        new Column({ width: 5400, space: 720 }),
        new Column({ width: 3240 }),
      ],
    },
  },
  children: [/* content */]
}]
```

`type: SectionType.NEXT_COLUMN`を使用した新しいセクションで段区切りを強制できます。

### 目次

```javascript
// CRITICAL: Headings must use HeadingLevel ONLY - no custom styles
new TableOfContents("Table of Contents", { hyperlink: true, headingStyleRange: "1-3" })
```

### ヘッダー/フッター

```javascript
sections: [{
  properties: {
    page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } } // 1440 = 1 inch
  },
  headers: {
    default: new Header({ children: [new Paragraph({ children: [new TextRun("Header")] })] })
  },
  footers: {
    default: new Footer({ children: [new Paragraph({
      children: [new TextRun("Page "), new TextRun({ children: [PageNumber.CURRENT] })]
    })] })
  },
  children: [/* content */]
}]
```

### docx-jsの重要なルール

- **ページサイズを明示的に設定する** - docx-jsのデフォルトはA4。米国向け文書にはUSレター（12240 x 15840 DXA）を使用
- **横向きの場合：縦向きの寸法を渡す** - docx-jsは内部的に幅と高さを入れ替える。短辺を`width`に、長辺を`height`に設定し、`orientation: PageOrientation.LANDSCAPE`を指定
- **`\n`は使用しない** - 別々のParagraph要素を使用
- **Unicode箇条書き文字は使用しない** - numbering設定で`LevelFormat.BULLET`を使用
- **PageBreakはParagraph内に配置** - 単独で使用すると無効なXMLになる
- **ImageRunには`type`が必須** - 必ずpng/jpg等を指定
- **テーブルの`width`は常にDXAで設定** - `WidthType.PERCENTAGE`は使用しない（Googleドキュメントで不具合）
- **テーブルには二重の幅設定が必要** - `columnWidths`配列とセルの`width`の両方が一致する必要がある
- **テーブル幅 = columnWidthsの合計** - DXAの場合、正確に合計が一致すること
- **常にセルマージンを追加** - 読みやすいパディングとして`margins: { top: 80, bottom: 80, left: 120, right: 120 }`を使用
- **`ShadingType.CLEAR`を使用** - テーブルのシェーディングにはSOLIDを使用しない
- **テーブルを区切り線として使用しない** - セルには最小高さがあり空のボックスとして表示される（ヘッダー/フッターも同様）。代わりにParagraphで`border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "2E75B6", space: 1 } }`を使用。2カラムフッターにはテーブルではなくタブストップ（タブストップセクション参照）を使用
- **目次にはHeadingLevelのみが必要** - 見出し段落にカスタムスタイルを使用しない
- **組み込みスタイルをオーバーライド** - 正確なIDを使用："Heading1"、"Heading2"など
- **`outlineLevel`を含める** - 目次に必要（H1は0、H2は1、など）

---

## 既存文書の編集

**3つのステップすべてを順番に実行してください。**

### ステップ1：展開
```bash
python scripts/office/unpack.py document.docx unpacked/
```
XMLを抽出し、整形し、隣接するランを結合し、スマートクォートをXMLエンティティ（`&#x201C;`など）に変換して編集後も維持されるようにします。ラン結合をスキップするには`--merge-runs false`を使用してください。

### ステップ2：XMLの編集

`unpacked/word/`内のファイルを編集します。パターンについては下記のXMLリファレンスを参照してください。

**変更履歴とコメントの著者には"Claude"を使用してください**（ユーザーが明示的に別の名前を要求しない限り）。

**文字列置換にはEditツールを直接使用してください。Pythonスクリプトは書かないでください。** スクリプトは不必要な複雑さをもたらします。Editツールは何が置換されるかを正確に示します。

**重要：新しいコンテンツにはスマートクォートを使用してください。** アポストロフィや引用符を含むテキストを追加する際は、XMLエンティティを使用してスマートクォートを生成してください：
```xml
<!-- Use these entities for professional typography -->
<w:t>Here&#x2019;s a quote: &#x201C;Hello&#x201D;</w:t>
```
| エンティティ | 文字 |
|--------|-----------|
| `&#x2018;` | ' (左シングル) |
| `&#x2019;` | ' (右シングル / アポストロフィ) |
| `&#x201C;` | " (左ダブル) |
| `&#x201D;` | " (右ダブル) |

**コメントの追加：** 複数のXMLファイルにまたがるボイラープレートを処理するには`comment.py`を使用してください（テキストは事前にエスケープ済みXMLである必要があります）：
```bash
python scripts/comment.py unpacked/ 0 "Comment text with &amp; and &#x2019;"
python scripts/comment.py unpacked/ 1 "Reply text" --parent 0  # reply to comment 0
python scripts/comment.py unpacked/ 0 "Text" --author "Custom Author"  # custom author name
```
その後、document.xmlにマーカーを追加してください（XMLリファレンスのコメントを参照）。

### ステップ3：パック
```bash
python scripts/office/pack.py unpacked/ output.docx --original document.docx
```
自動修復付きでバリデーションを行い、XMLを圧縮してDOCXを作成します。スキップするには`--validate false`を使用してください。

**自動修復で修正されるもの：**
- `durableId` >= 0x7FFFFFFF（有効なIDを再生成）
- 空白を含む`<w:t>`で欠落している`xml:space="preserve"`

**自動修復で修正されないもの：**
- 不正なXML、無効な要素のネスト、欠落したリレーションシップ、スキーマ違反

### よくある落とし穴

- **`<w:r>`要素全体を置換する**：変更履歴を追加する際は、`<w:r>...</w:r>`ブロック全体を`<w:del>...<w:ins>...`として兄弟要素に置換してください。ラン内に変更履歴タグを挿入しないでください。
- **`<w:rPr>`フォーマットを保持する**：太字やフォントサイズなどを維持するため、元のランの`<w:rPr>`ブロックを変更履歴のランにコピーしてください。

---

## XMLリファレンス

### スキーマ準拠

- **`<w:pPr>`内の要素順序**: `<w:pStyle>`, `<w:numPr>`, `<w:spacing>`, `<w:ind>`, `<w:jc>`, `<w:rPr>`は最後
- **空白**: 先頭/末尾にスペースがある`<w:t>`には`xml:space="preserve"`を追加
- **RSID**: 8桁の16進数（例：`00AB1234`）

### 変更履歴

**挿入：**
```xml
<w:ins w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:t>inserted text</w:t></w:r>
</w:ins>
```

**削除：**
```xml
<w:del w:id="2" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:delText>deleted text</w:delText></w:r>
</w:del>
```

**`<w:del>`内部**：`<w:t>`の代わりに`<w:delText>`を使用し、`<w:instrText>`の代わりに`<w:delInstrText>`を使用してください。

**最小限の編集** - 変更箇所のみをマークしてください：
```xml
<!-- Change "30 days" to "60 days" -->
<w:r><w:t>The term is </w:t></w:r>
<w:del w:id="1" w:author="Claude" w:date="...">
  <w:r><w:delText>30</w:delText></w:r>
</w:del>
<w:ins w:id="2" w:author="Claude" w:date="...">
  <w:r><w:t>60</w:t></w:r>
</w:ins>
<w:r><w:t> days.</w:t></w:r>
```

**段落/リスト項目全体の削除** - 段落からすべてのコンテンツを削除する場合、次の段落と結合されるよう段落マークも削除済みとしてマークしてください。`<w:pPr><w:rPr>`内に`<w:del/>`を追加します：
```xml
<w:p>
  <w:pPr>
    <w:numPr>...</w:numPr>  <!-- list numbering if present -->
    <w:rPr>
      <w:del w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z"/>
    </w:rPr>
  </w:pPr>
  <w:del w:id="2" w:author="Claude" w:date="2025-01-01T00:00:00Z">
    <w:r><w:delText>Entire paragraph content being deleted...</w:delText></w:r>
  </w:del>
</w:p>
```
`<w:pPr><w:rPr>`内に`<w:del/>`がないと、変更を承認した際に空の段落/リスト項目が残ります。

**別の著者の挿入を拒否する** - 挿入の中に削除をネストします：
```xml
<w:ins w:author="Jane" w:id="5">
  <w:del w:author="Claude" w:id="10">
    <w:r><w:delText>their inserted text</w:delText></w:r>
  </w:del>
</w:ins>
```

**別の著者の削除を復元する** - 削除の後に挿入を追加します（削除を変更しない）：
```xml
<w:del w:author="Jane" w:id="5">
  <w:r><w:delText>deleted text</w:delText></w:r>
</w:del>
<w:ins w:author="Claude" w:id="10">
  <w:r><w:t>deleted text</w:t></w:r>
</w:ins>
```

### コメント

`comment.py`を実行した後（ステップ2参照）、document.xmlにマーカーを追加します。返信の場合は、`--parent`フラグを使用し、マーカーを親のマーカー内にネストしてください。

**重要：`<w:commentRangeStart>`と`<w:commentRangeEnd>`は`<w:r>`の兄弟要素であり、`<w:r>`の中に入れてはいけません。**

```xml
<!-- Comment markers are direct children of w:p, never inside w:r -->
<w:commentRangeStart w:id="0"/>
<w:del w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:delText>deleted</w:delText></w:r>
</w:del>
<w:r><w:t> more text</w:t></w:r>
<w:commentRangeEnd w:id="0"/>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="0"/></w:r>

<!-- Comment 0 with reply 1 nested inside -->
<w:commentRangeStart w:id="0"/>
  <w:commentRangeStart w:id="1"/>
  <w:r><w:t>text</w:t></w:r>
  <w:commentRangeEnd w:id="1"/>
<w:commentRangeEnd w:id="0"/>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="0"/></w:r>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="1"/></w:r>
```

### 画像

1. `word/media/`に画像ファイルを追加
2. `word/_rels/document.xml.rels`にリレーションシップを追加：
```xml
<Relationship Id="rId5" Type=".../image" Target="media/image1.png"/>
```
3. `[Content_Types].xml`にコンテンツタイプを追加：
```xml
<Default Extension="png" ContentType="image/png"/>
```
4. document.xmlで参照：
```xml
<w:drawing>
  <wp:inline>
    <wp:extent cx="914400" cy="914400"/>  <!-- EMUs: 914400 = 1 inch -->
    <a:graphic>
      <a:graphicData uri=".../picture">
        <pic:pic>
          <pic:blipFill><a:blip r:embed="rId5"/></pic:blipFill>
        </pic:pic>
      </a:graphicData>
    </a:graphic>
  </wp:inline>
</w:drawing>
```

---

## 依存関係

- **pandoc**: テキスト抽出
- **docx**: `npm install -g docx`（新規文書用）
- **LibreOffice**: PDF変換（サンドボックス環境では`scripts/office/soffice.py`により自動設定）
- **Poppler**: 画像変換用の`pdftoppm`

---
name: pptx
description: ".pptxファイルが入力、出力、またはその両方として関わるあらゆる場面で使用するスキルです。スライドデッキ、ピッチデック、プレゼンテーションの作成、.pptxファイルの読み取り・解析・テキスト抽出（抽出したコンテンツをメールや要約など別の目的で使用する場合も含む）、既存プレゼンテーションの編集・修正・更新、スライドファイルの結合・分割、テンプレート・レイアウト・スピーカーノート・コメントの操作が含まれます。「デッキ」「スライド」「プレゼンテーション」への言及や、.pptxファイル名への参照がある場合にトリガーされます（その後コンテンツをどう利用するかに関わらず）。.pptxファイルを開く、作成する、操作する必要がある場合にこのスキルを使用してください。"
license: Proprietary. LICENSE.txt has complete terms
---

# PPTXスキル

## クイックリファレンス

| タスク | ガイド |
|------|-------|
| コンテンツの読み取り・分析 | `python -m markitdown presentation.pptx` |
| テンプレートからの編集・作成 | [editing.md](editing.md)を参照 |
| ゼロからの作成 | [pptxgenjs.md](pptxgenjs.md)を参照 |

---

## コンテンツの読み取り

```bash
# Text extraction
python -m markitdown presentation.pptx

# Visual overview
python scripts/thumbnail.py presentation.pptx

# Raw XML
python scripts/office/unpack.py presentation.pptx unpacked/
```

---

## 編集ワークフロー

**詳細は[editing.md](editing.md)を参照してください。**

1. `thumbnail.py`でテンプレートを分析
2. 展開 → スライド操作 → コンテンツ編集 → クリーンアップ → パック

---

## ゼロからの作成

**詳細は[pptxgenjs.md](pptxgenjs.md)を参照してください。**

テンプレートや参考プレゼンテーションがない場合に使用します。

---

## デザインのアイデア

**退屈なスライドを作らないでください。** 白背景にプレーンな箇条書きでは誰にも感銘を与えられません。各スライドについて、以下のリストからアイデアを検討してください。

### 作業を始める前に

- **大胆で内容に即した配色を選ぶ**：配色はこのトピックのためにデザインされたと感じられるものにしてください。まったく別のプレゼンテーションに色を入れ替えても違和感がないなら、十分に特化した選択ができていません。
- **均等ではなく優劣をつける**：1つの色を支配的に（視覚的な重み60-70%）、1-2色をサポートトーンとして、1色をシャープなアクセントとして使用してください。すべての色に同じ重みを与えないでください。
- **明暗のコントラスト**：タイトルと結論のスライドには暗い背景、コンテンツには明るい背景を使用（「サンドイッチ」構造）。またはプレミアム感のために全体を暗くする方法もあります。
- **ビジュアルモチーフを決める**：1つの特徴的な要素を選び繰り返してください。角丸の画像フレーム、色付き丸の中のアイコン、太い片側ボーダーなど。すべてのスライドで一貫して使用してください。

### カラーパレット

トピックに合った色を選んでください。汎用的な青にデフォルトしないこと。以下のパレットをインスピレーションとして使用してください：

| テーマ | プライマリ | セカンダリ | アクセント |
|-------|---------|-----------|--------|
| **ミッドナイトエグゼクティブ** | `1E2761` (ネイビー) | `CADCFC` (アイスブルー) | `FFFFFF` (ホワイト) |
| **フォレスト&モス** | `2C5F2D` (フォレスト) | `97BC62` (モス) | `F5F5F5` (クリーム) |
| **コーラルエナジー** | `F96167` (コーラル) | `F9E795` (ゴールド) | `2F3C7E` (ネイビー) |
| **ウォームテラコッタ** | `B85042` (テラコッタ) | `E7E8D1` (サンド) | `A7BEAE` (セージ) |
| **オーシャングラデーション** | `065A82` (ディープブルー) | `1C7293` (ティール) | `21295C` (ミッドナイト) |
| **チャコールミニマル** | `36454F` (チャコール) | `F2F2F2` (オフホワイト) | `212121` (ブラック) |
| **ティールトラスト** | `028090` (ティール) | `00A896` (シーフォーム) | `02C39A` (ミント) |
| **ベリー&クリーム** | `6D2E46` (ベリー) | `A26769` (ダスティローズ) | `ECE2D0` (クリーム) |
| **セージカーム** | `84B59F` (セージ) | `69A297` (ユーカリ) | `50808E` (スレート) |
| **チェリーボールド** | `990011` (チェリー) | `FCF6F5` (オフホワイト) | `2F3C7E` (ネイビー) |

### 各スライドについて

**すべてのスライドにはビジュアル要素が必要です** - 画像、チャート、アイコン、またはシェイプ。テキストだけのスライドは記憶に残りません。

**レイアウトオプション：**
- 2カラム（左にテキスト、右にイラスト）
- アイコン＋テキスト行（色付き丸の中にアイコン、太字の見出し、下に説明文）
- 2x2または2x3グリッド（片側に画像、もう片側にコンテンツブロックのグリッド）
- ハーフブリード画像（左または右全面）とコンテンツオーバーレイ

**データ表示：**
- 大きな数値の強調表示（60-72ptの大きな数字と下に小さなラベル）
- 比較カラム（ビフォー/アフター、長所/短所、横並びオプション）
- タイムラインまたはプロセスフロー（番号付きステップ、矢印）

**ビジュアルの仕上げ：**
- セクションヘッダーの横に小さな色付き丸の中のアイコン
- 重要な統計やキャッチフレーズにイタリックのアクセントテキスト

### タイポグラフィ

**魅力的なフォントの組み合わせを選んでください** - Arialにデフォルトしないこと。個性のある見出しフォントとクリーンな本文フォントを組み合わせてください。

| 見出しフォント | 本文フォント |
|-------------|-----------|
| Georgia | Calibri |
| Arial Black | Arial |
| Calibri | Calibri Light |
| Cambria | Calibri |
| Trebuchet MS | Calibri |
| Impact | Arial |
| Palatino | Garamond |
| Consolas | Calibri |

| 要素 | サイズ |
|---------|------|
| スライドタイトル | 36-44pt 太字 |
| セクションヘッダー | 20-24pt 太字 |
| 本文テキスト | 14-16pt |
| キャプション | 10-12pt 控えめ |

### スペーシング

- 最小マージン0.5インチ
- コンテンツブロック間0.3-0.5インチ
- 余白を残してください。すべてのスペースを埋め尽くさないこと

### 避けるべきこと（よくある間違い）

- **同じレイアウトを繰り返さない** - スライドごとにカラム、カード、コールアウトを変えてください
- **本文テキストを中央揃えにしない** - 段落やリストは左揃え。中央揃えはタイトルだけ
- **サイズのコントラストを怠らない** - タイトルは14-16ptの本文から際立たせるために36pt以上が必要
- **青にデフォルトしない** - 特定のトピックを反映した色を選ぶ
- **スペーシングをランダムにしない** - 0.3インチまたは0.5インチの間隔を選び、一貫して使用
- **1枚だけスタイルを適用して残りをプレーンにしない** - 全体を通してデザインするか、シンプルに統一する
- **テキストだけのスライドを作らない** - 画像、アイコン、チャート、ビジュアル要素を追加。プレーンなタイトル＋箇条書きを避ける
- **テキストボックスのパディングを忘れない** - 線やシェイプをテキストの端に揃える際、テキストボックスに`margin: 0`を設定するか、パディングを考慮してシェイプをオフセットする
- **低コントラストの要素を使わない** - アイコンとテキストの両方が背景に対して強いコントラストが必要。明るい背景に薄いテキストや暗い背景に暗いテキストを避ける
- **タイトル下のアクセントラインは絶対に使わない** - これはAI生成スライドの特徴です。代わりに余白や背景色を使用してください

---

## QA（必須）

**問題があるものと想定してください。あなたの仕事はそれを見つけることです。**

最初のレンダリングがそのまま正しいことはほぼありません。QAを確認作業としてではなく、バグ探しとして取り組んでください。最初の確認で問題が1つも見つからなかったなら、十分に注意深く見ていません。

### コンテンツQA

```bash
python -m markitdown output.pptx
```

コンテンツの欠落、タイプミス、順序の間違いを確認してください。

**テンプレート使用時は、残っているプレースホルダーテキストを確認してください：**

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|this.*(page|slide).*layout"
```

grepが結果を返した場合は、完了宣言の前に修正してください。

### ビジュアルQA

**サブエージェントを使用してください** - 2-3枚のスライドでも同様です。あなたはコードを見続けているため、実際にあるものではなく期待するものを見てしまいます。サブエージェントは新鮮な目を持っています。

スライドを画像に変換し（[画像への変換](#画像への変換)を参照）、以下のプロンプトを使用してください：

```
Visually inspect these slides. Assume there are issues — find them.

Look for:
- Overlapping elements (text through shapes, lines through words, stacked elements)
- Text overflow or cut off at edges/box boundaries
- Decorative lines positioned for single-line text but title wrapped to two lines
- Source citations or footers colliding with content above
- Elements too close (< 0.3" gaps) or cards/sections nearly touching
- Uneven gaps (large empty area in one place, cramped in another)
- Insufficient margin from slide edges (< 0.5")
- Columns or similar elements not aligned consistently
- Low-contrast text (e.g., light gray text on cream-colored background)
- Low-contrast icons (e.g., dark icons on dark backgrounds without a contrasting circle)
- Text boxes too narrow causing excessive wrapping
- Leftover placeholder content

For each slide, list issues or areas of concern, even if minor.

Read and analyze these images:
1. /path/to/slide-01.jpg (Expected: [brief description])
2. /path/to/slide-02.jpg (Expected: [brief description])

Report ALL issues found, including minor ones.
```

### 検証ループ

1. スライド生成 → 画像に変換 → 確認
2. **見つかった問題をリストアップ**（問題が見つからなかった場合は、より厳しい目で再確認）
3. 問題を修正
4. **影響を受けたスライドを再検証** - 1つの修正が別の問題を生むことが多い
5. 完全なパスで新たな問題が見つからなくなるまで繰り返す

**少なくとも1回の修正と検証のサイクルを完了するまで、完了宣言をしないでください。**

---

## 画像への変換

ビジュアル確認のために、プレゼンテーションを個別のスライド画像に変換します：

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

これにより`slide-01.jpg`、`slide-02.jpg`などが作成されます。

修正後に特定のスライドのみを再レンダリングするには：

```bash
pdftoppm -jpeg -r 150 -f N -l N output.pdf slide-fixed
```

---

## 依存関係

- `pip install "markitdown[pptx]"` - テキスト抽出
- `pip install Pillow` - サムネイルグリッド
- `npm install -g pptxgenjs` - ゼロからの作成
- LibreOffice (`soffice`) - PDF変換（サンドボックス環境では`scripts/office/soffice.py`により自動設定）
- Poppler (`pdftoppm`) - PDFから画像への変換

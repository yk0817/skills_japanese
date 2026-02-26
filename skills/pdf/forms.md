**重要：以下の手順を順番に完了する必要があります。先にコードを書くことはしないでください。**

PDF フォームに記入する必要がある場合は、まず PDF に入力可能なフォームフィールドがあるかどうかを確認してください。このファイルのディレクトリから以下のスクリプトを実行します：
 `python scripts/check_fillable_fields <file.pdf>`、結果に応じて「入力可能なフィールド」または「入力不可のフィールド」のセクションに進み、その指示に従ってください。

# 入力可能なフィールド
PDF に入力可能なフォームフィールドがある場合：
- このファイルのディレクトリから以下のスクリプトを実行します：`python scripts/extract_form_field_info.py <input.pdf> <field_info.json>`。以下のフォーマットでフィールドのリストを含む JSON ファイルが作成されます：
```
[
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "rect": ([left, bottom, right, top] bounding box in PDF coordinates, y=0 is the bottom of the page),
    "type": ("text", "checkbox", "radio_group", or "choice"),
  },
  // Checkboxes have "checked_value" and "unchecked_value" properties:
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "type": "checkbox",
    "checked_value": (Set the field to this value to check the checkbox),
    "unchecked_value": (Set the field to this value to uncheck the checkbox),
  },
  // Radio groups have a "radio_options" list with the possible choices.
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "type": "radio_group",
    "radio_options": [
      {
        "value": (set the field to this value to select this radio option),
        "rect": (bounding box for the radio button for this option)
      },
      // Other radio options
    ]
  },
  // Multiple choice fields have a "choice_options" list with the possible choices:
  {
    "field_id": (unique ID for the field),
    "page": (page number, 1-based),
    "type": "choice",
    "choice_options": [
      {
        "value": (set the field to this value to select this option),
        "text": (display text of the option)
      },
      // Other choice options
    ],
  }
]
```
- PDF を PNG に変換します（各ページごとに1つの画像）。以下のスクリプトを（このファイルのディレクトリから）実行します：
`python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>`
次に画像を分析して各フォームフィールドの目的を判断します（バウンディングボックスの PDF 座標を画像座標に変換してください）。
- 各フィールドに入力する値を以下のフォーマットで `field_values.json` ファイルに作成します：
```
[
  {
    "field_id": "last_name", // Must match the field_id from `extract_form_field_info.py`
    "description": "The user's last name",
    "page": 1, // Must match the "page" value in field_info.json
    "value": "Simpson"
  },
  {
    "field_id": "Checkbox12",
    "description": "Checkbox to be checked if the user is 18 or over",
    "page": 1,
    "value": "/On" // If this is a checkbox, use its "checked_value" value to check it. If it's a radio button group, use one of the "value" values in "radio_options".
  },
  // more fields
]
```
- `fill_fillable_fields.py` スクリプトをこのファイルのディレクトリから実行して、入力済みの PDF を作成します：
`python scripts/fill_fillable_fields.py <input pdf> <field_values.json> <output pdf>`
このスクリプトは指定したフィールド ID と値が有効かどうかを検証します。エラーメッセージが表示された場合は、該当するフィールドを修正して再試行してください。

# 入力不可のフィールド
PDF に入力可能なフォームフィールドがない場合は、テキストアノテーションを追加します。まず PDF 構造から座標の抽出を試み（より正確）、それが失敗した場合は視覚的な推定にフォールバックします。

## ステップ 1：まず構造の抽出を試みる

以下のスクリプトを実行して、テキストラベル、線、チェックボックスとその正確な PDF 座標を抽出します：
`python scripts/extract_form_structure.py <input.pdf> form_structure.json`

以下を含む JSON ファイルが作成されます：
- **labels**：正確な座標（PDF ポイントでの x0, top, x1, bottom）を含むすべてのテキスト要素
- **lines**：行の境界を定義する水平線
- **checkboxes**：チェックボックスである小さな正方形の矩形（中心座標付き）
- **row_boundaries**：水平線から計算された行の上端/下端の位置

**結果を確認してください**：`form_structure.json` に意味のあるラベル（フォームフィールドに対応するテキスト要素）がある場合は、**アプローチ A：構造ベースの座標**を使用します。PDF がスキャン/画像ベースでラベルがほとんどまたはまったくない場合は、**アプローチ B：視覚的な推定**を使用します。

---

## アプローチ A：構造ベースの座標（推奨）

`extract_form_structure.py` が PDF 内のテキストラベルを検出した場合にこれを使用します。

### A.1：構造の分析

form_structure.json を読み、以下を特定します：

1. **ラベルグループ**：1つのラベルを形成する隣接テキスト要素（例："Last" + "Name"）
2. **行構造**：同様の `top` 値を持つラベルは同じ行にある
3. **フィールド列**：入力エリアはラベルの終了後から始まる（x0 = label.x1 + gap）
4. **チェックボックス**：構造から直接チェックボックスの座標を使用する

**座標系**：PDF 座標で y=0 はページの上端、y は下方向に増加します。

### A.2：欠落要素の確認

構造抽出ではすべてのフォーム要素を検出できない場合があります。よくあるケース：
- **円形のチェックボックス**：正方形の矩形のみがチェックボックスとして検出される
- **複雑なグラフィックス**：装飾要素や非標準のフォームコントロール
- **薄い色や淡い色の要素**：抽出されない場合がある

PDF 画像にフォームフィールドが見えるが form_structure.json に含まれていない場合、それらの特定のフィールドには**視覚的分析**を使用する必要があります（下記の「ハイブリッドアプローチ」を参照）。

### A.3：PDF 座標で fields.json を作成

各フィールドについて、抽出された構造から入力座標を計算します：

**テキストフィールド：**
- entry x0 = label x1 + 5（ラベルの後の小さなギャップ）
- entry x1 = 次のラベルの x0、または行の境界
- entry top = ラベルの top と同じ
- entry bottom = 下の行の境界線、または label bottom + row_height

**チェックボックス：**
- form_structure.json から直接チェックボックスの矩形座標を使用する
- entry_bounding_box = [checkbox.x0, checkbox.top, checkbox.x1, checkbox.bottom]

`pdf_width` と `pdf_height` を使用して fields.json を作成します（PDF 座標であることを示す）：
```json
{
  "pages": [
    {"page_number": 1, "pdf_width": 612, "pdf_height": 792}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "Last name entry field",
      "field_label": "Last Name",
      "label_bounding_box": [43, 63, 87, 73],
      "entry_bounding_box": [92, 63, 260, 79],
      "entry_text": {"text": "Smith", "font_size": 10}
    },
    {
      "page_number": 1,
      "description": "US Citizen Yes checkbox",
      "field_label": "Yes",
      "label_bounding_box": [260, 200, 280, 210],
      "entry_bounding_box": [285, 197, 292, 205],
      "entry_text": {"text": "X"}
    }
  ]
}
```

**重要**：`pdf_width`/`pdf_height` と form_structure.json から直接取得した座標を使用してください。

### A.4：バウンディングボックスの検証

記入する前に、バウンディングボックスにエラーがないか確認します：
`python scripts/check_bounding_boxes.py fields.json`

交差するバウンディングボックスや、フォントサイズに対して小さすぎる入力ボックスをチェックします。報告されたエラーを修正してから記入を行ってください。

---

## アプローチ B：視覚的な推定（フォールバック）

PDF がスキャン/画像ベースで、構造抽出が使用可能なテキストラベルを検出できなかった場合（例：すべてのテキストが "(cid:X)" パターンとして表示される）にこれを使用します。

### B.1：PDF を画像に変換

`python scripts/convert_pdf_to_images.py <input.pdf> <images_dir/>`

### B.2：初期フィールド特定

各ページの画像を調べてフォームのセクションを特定し、フィールドの位置の**大まかな推定**を取得します：
- フォームフィールドのラベルとその概算位置
- 入力エリア（テキスト入力用の線、ボックス、または空白スペース）
- チェックボックスとその概算位置

各フィールドについて、概算のピクセル座標をメモします（まだ正確である必要はありません）。

### B.3：ズーム精査（正確さのために重要）

各フィールドについて、推定位置の周辺領域をクロップして座標を正確に精査します。

**ImageMagick を使用してズームクロップを作成します：**
```bash
magick <page_image> -crop <width>x<height>+<x>+<y> +repage <crop_output.png>
```

ここで：
- `<x>, <y>` = クロップ領域の左上角（大まかな推定からパディングを引いた値）
- `<width>, <height>` = クロップ領域のサイズ（フィールドエリア + 各側約50pxのパディング）

**例：** (100, 150) 付近と推定された「Name」フィールドを精査する場合：
```bash
magick images_dir/page_1.png -crop 300x80+50+120 +repage crops/name_field.png
```

（注：`magick` コマンドが利用できない場合は、同じ引数で `convert` を試してください）。

**クロップ画像を調べて**正確な座標を決定します：
1. 入力エリアが始まる正確なピクセルを特定する（ラベルの後）
2. 入力エリアが終わる場所を特定する（次のフィールドまたは端の前）
3. 入力ラインまたはボックスの上端と下端を特定する

**クロップ座標をフルイメージ座標に変換します：**
- full_x = crop_x + crop_offset_x
- full_y = crop_y + crop_offset_y

例：クロップが (50, 120) から開始され、入力ボックスがクロップ内の (52, 18) から始まる場合：
- entry_x0 = 52 + 50 = 102
- entry_top = 18 + 120 = 138

**各フィールドについて繰り返し**、近くのフィールドは可能な限り1つのクロップにグループ化します。

### B.4：精査済み座標で fields.json を作成

`image_width` と `image_height` を使用して fields.json を作成します（画像座標であることを示す）：
```json
{
  "pages": [
    {"page_number": 1, "image_width": 1700, "image_height": 2200}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "Last name entry field",
      "field_label": "Last Name",
      "label_bounding_box": [120, 175, 242, 198],
      "entry_bounding_box": [255, 175, 720, 218],
      "entry_text": {"text": "Smith", "font_size": 10}
    }
  ]
}
```

**重要**：`image_width`/`image_height` とズーム分析から得た精査済みピクセル座標を使用してください。

### B.5：バウンディングボックスの検証

記入する前に、バウンディングボックスにエラーがないか確認します：
`python scripts/check_bounding_boxes.py fields.json`

交差するバウンディングボックスや、フォントサイズに対して小さすぎる入力ボックスをチェックします。報告されたエラーを修正してから記入を行ってください。

---

## ハイブリッドアプローチ：構造 + 視覚

構造抽出がほとんどのフィールドで機能するが、一部の要素（例：円形のチェックボックス、特殊なフォームコントロール）が欠落している場合にこれを使用します。

1. **アプローチ A** を form_structure.json で検出されたフィールドに使用する
2. **PDF を画像に変換**して欠落フィールドの視覚分析を行う
3. **ズーム精査**（アプローチ B から）を欠落フィールドに使用する
4. **座標を統合する**：構造抽出からのフィールドには `pdf_width`/`pdf_height` を使用する。視覚的に推定したフィールドについては、画像座標を PDF 座標に変換する必要がある：
   - pdf_x = image_x * (pdf_width / image_width)
   - pdf_y = image_y * (pdf_height / image_height)
5. **fields.json では単一の座標系を使用する** - すべてを `pdf_width`/`pdf_height` 付きの PDF 座標に変換する

---

## ステップ 2：記入前のバリデーション

**記入する前に必ずバウンディングボックスを検証してください：**
`python scripts/check_bounding_boxes.py fields.json`

以下をチェックします：
- 交差するバウンディングボックス（テキストの重なりの原因）
- 指定されたフォントサイズに対して小さすぎる入力ボックス

報告されたエラーを fields.json で修正してから次に進んでください。

## ステップ 3：フォームに記入

記入スクリプトは座標系を自動検出し、変換を処理します：
`python scripts/fill_pdf_form_with_annotations.py <input.pdf> fields.json <output.pdf>`

## ステップ 4：出力の確認

記入済みの PDF を画像に変換してテキストの配置を確認します：
`python scripts/convert_pdf_to_images.py <output.pdf> <verify_images/>`

テキストの位置がずれている場合：
- **アプローチ A**：`pdf_width`/`pdf_height` 付きの form_structure.json からの PDF 座標を使用しているか確認する
- **アプローチ B**：画像のサイズが一致し、座標が正確なピクセルであるか確認する
- **ハイブリッド**：視覚的に推定したフィールドの座標変換が正しいか確認する

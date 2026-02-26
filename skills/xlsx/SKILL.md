---
name: xlsx
description: "スプレッドシートファイルが主要な入力または出力となるあらゆる場面で使用するスキルです。既存の.xlsx、.xlsm、.csv、.tsvファイルを開く、読み取る、編集する、修正する（列の追加、数式の計算、書式設定、チャート作成、乱雑なデータのクリーニングなど）、新規スプレッドシートをゼロからまたは他のデータソースから作成する、表形式ファイルフォーマット間の変換を行う場合に使用します。ユーザーがスプレッドシートファイルを名前やパスで参照した場合（「ダウンロードにあるxlsx」のようなカジュアルな言及でも）、それに対する操作や生成を求めている場合に特にトリガーされます。また、乱雑な表形式データファイル（不正な行、ずれたヘッダー、ジャンクデータ）を適切なスプレッドシートにクリーニング・再構成する場合にもトリガーされます。成果物はスプレッドシートファイルである必要があります。主要な成果物がWord文書、HTMLレポート、スタンドアロンPythonスクリプト、データベースパイプライン、Google Sheets API統合の場合は、表形式データが含まれていてもトリガーしないでください。"
license: Proprietary. LICENSE.txt has complete terms
---

# 出力の要件

## すべてのExcelファイル

### プロフェッショナルなフォント
- ユーザーから別途指示がない限り、すべての成果物に一貫したプロフェッショナルなフォント（例：Arial、Times New Roman）を使用してください

### 数式エラーゼロ
- すべてのExcelモデルは数式エラー（#REF!、#DIV/0!、#VALUE!、#N/A、#NAME?）がゼロの状態で納品しなければなりません

### 既存テンプレートの保持（テンプレート更新時）
- ファイルを変更する際は、既存のフォーマット、スタイル、規則を正確に調査し一致させてください
- 確立されたパターンを持つファイルに標準化されたフォーマットを強制しないでください
- 既存テンプレートの規則は常にこのガイドラインよりも優先されます

## 財務モデル

### 色分け基準
ユーザーまたは既存テンプレートから別途指示がない限り

#### 業界標準の色規則
- **青テキスト（RGB: 0,0,255）**：ハードコードされた入力値、シナリオ分析でユーザーが変更する数値
- **黒テキスト（RGB: 0,0,0）**：すべての数式と計算
- **緑テキスト（RGB: 0,128,0）**：同じワークブック内の他のワークシートからの参照リンク
- **赤テキスト（RGB: 255,0,0）**：他のファイルへの外部リンク
- **黄色背景（RGB: 255,255,0）**：注意が必要な主要前提条件、または更新が必要なセル

### 数値フォーマット基準

#### 必須フォーマットルール
- **年度**：テキスト文字列としてフォーマット（例："2024"、"2,024"ではない）
- **通貨**：$#,##0形式を使用。ヘッダーに単位を必ず明記（"Revenue ($mm)"）
- **ゼロ**：数値フォーマットですべてのゼロを"-"に表示（パーセンテージも含む）（例："$#,##0;($#,##0);-"）
- **パーセンテージ**：デフォルトで0.0%形式（小数点以下1桁）
- **倍率**：バリュエーション倍率（EV/EBITDA、P/E）は0.0xとしてフォーマット
- **負の数**：マイナス記号 -123 ではなく括弧 (123) を使用

### 数式構築ルール

#### 前提条件の配置
- すべての前提条件（成長率、マージン、倍率など）を別々の前提条件セルに配置
- 数式内ではハードコードされた値ではなくセル参照を使用
- 例：=B5*1.05の代わりに=B5*(1+$B$6)を使用

#### 数式エラーの防止
- すべてのセル参照が正しいことを確認
- 範囲のオフバイワンエラーを確認
- すべての予測期間で一貫した数式を確保
- エッジケース（ゼロ値、負の数）でテスト
- 意図しない循環参照がないことを確認

#### ハードコードの文書化要件
- セルの横にコメントまたはメモ（テーブル末尾の場合）。形式："Source: [システム/文書], [日付], [具体的参照], [該当する場合URL]"
- 例：
  - "Source: Company 10-K, FY2024, Page 45, Revenue Note, [SEC EDGAR URL]"
  - "Source: Company 10-Q, Q2 2025, Exhibit 99.1, [SEC EDGAR URL]"
  - "Source: Bloomberg Terminal, 8/15/2025, AAPL US Equity"
  - "Source: FactSet, 8/20/2025, Consensus Estimates Screen"

# XLSXの作成、編集、分析

## 概要

ユーザーは.xlsxファイルの作成、編集、または内容の分析を依頼する場合があります。タスクに応じて異なるツールとワークフローが利用できます。

## 重要な要件

**数式の再計算にはLibreOfficeが必要です**：`scripts/recalc.py`スクリプトを使用して数式の値を再計算するためにLibreOfficeがインストールされていると想定できます。このスクリプトは初回実行時にLibreOfficeを自動設定します（Unixソケットが制限されるサンドボックス環境でも`scripts/office/soffice.py`で対応）。

## データの読み取りと分析

### pandasによるデータ分析
データ分析、可視化、基本的な操作には、強力なデータ操作機能を提供する**pandas**を使用してください：

```python
import pandas as pd

# Read Excel
df = pd.read_excel('file.xlsx')  # Default: first sheet
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)  # All sheets as dict

# Analyze
df.head()      # Preview data
df.info()      # Column info
df.describe()  # Statistics

# Write Excel
df.to_excel('output.xlsx', index=False)
```

## Excelファイルのワークフロー

## 重要：ハードコードされた値ではなく数式を使用してください

**Pythonで値を計算してハードコードするのではなく、常にExcelの数式を使用してください。** これにより、スプレッドシートが動的で更新可能な状態を維持できます。

### 間違い - ハードコードされた計算値
```python
# Bad: Calculating in Python and hardcoding result
total = df['Sales'].sum()
sheet['B10'] = total  # Hardcodes 5000

# Bad: Computing growth rate in Python
growth = (df.iloc[-1]['Revenue'] - df.iloc[0]['Revenue']) / df.iloc[0]['Revenue']
sheet['C5'] = growth  # Hardcodes 0.15

# Bad: Python calculation for average
avg = sum(values) / len(values)
sheet['D20'] = avg  # Hardcodes 42.5
```

### 正解 - Excelの数式を使用
```python
# Good: Let Excel calculate the sum
sheet['B10'] = '=SUM(B2:B9)'

# Good: Growth rate as Excel formula
sheet['C5'] = '=(C4-C2)/C2'

# Good: Average using Excel function
sheet['D20'] = '=AVERAGE(D2:D19)'
```

これは合計、パーセンテージ、比率、差分など、すべての計算に適用されます。ソースデータが変更された場合にスプレッドシートが再計算できるようにする必要があります。

## 一般的なワークフロー
1. **ツールを選択**：データにはpandas、数式/書式設定にはopenpyxl
2. **作成/読み込み**：新規ワークブックを作成するか既存ファイルを読み込む
3. **変更**：データ、数式、書式設定の追加/編集
4. **保存**：ファイルに書き出し
5. **数式の再計算（数式を使用する場合は必須）**：scripts/recalc.pyスクリプトを使用
   ```bash
   python scripts/recalc.py output.xlsx
   ```
6. **エラーの確認と修正**：
   - スクリプトはエラーの詳細をJSON形式で返します
   - `status`が`errors_found`の場合、`error_summary`で具体的なエラーの種類と場所を確認
   - 特定されたエラーを修正し、再度再計算
   - 修正すべき一般的なエラー：
     - `#REF!`：無効なセル参照
     - `#DIV/0!`：ゼロによる除算
     - `#VALUE!`：数式内のデータ型の不一致
     - `#NAME?`：認識されない数式名

### 新規Excelファイルの作成

```python
# Using openpyxl for formulas and formatting
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active

# Add data
sheet['A1'] = 'Hello'
sheet['B1'] = 'World'
sheet.append(['Row', 'of', 'data'])

# Add formula
sheet['B2'] = '=SUM(A1:A10)'

# Formatting
sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')
sheet['A1'].alignment = Alignment(horizontal='center')

# Column width
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### 既存Excelファイルの編集

```python
# Using openpyxl to preserve formulas and formatting
from openpyxl import load_workbook

# Load existing file
wb = load_workbook('existing.xlsx')
sheet = wb.active  # or wb['SheetName'] for specific sheet

# Working with multiple sheets
for sheet_name in wb.sheetnames:
    sheet = wb[sheet_name]
    print(f"Sheet: {sheet_name}")

# Modify cells
sheet['A1'] = 'New Value'
sheet.insert_rows(2)  # Insert row at position 2
sheet.delete_cols(3)  # Delete column 3

# Add new sheet
new_sheet = wb.create_sheet('NewSheet')
new_sheet['A1'] = 'Data'

wb.save('modified.xlsx')
```

## 数式の再計算

openpyxlで作成または変更されたExcelファイルには、数式が文字列として含まれますが計算結果は含まれません。数式を再計算するには、提供されている`scripts/recalc.py`スクリプトを使用してください：

```bash
python scripts/recalc.py <excel_file> [timeout_seconds]
```

例：
```bash
python scripts/recalc.py output.xlsx 30
```

スクリプトの機能：
- 初回実行時にLibreOfficeマクロを自動設定
- すべてのシートのすべての数式を再計算
- すべてのセルをExcelエラー（#REF!、#DIV/0!など）についてスキャン
- エラーの場所と件数の詳細をJSON形式で返す
- LinuxとmacOSの両方で動作

## 数式検証チェックリスト

数式が正しく動作することを確認するためのクイックチェック：

### 基本的な検証
- [ ] **サンプル参照を2-3件テスト**：完全なモデルを構築する前に正しい値を参照しているか確認
- [ ] **列のマッピング**：Excelの列が一致しているか確認（例：列64 = BL、BKではない）
- [ ] **行のオフセット**：Excelの行は1始まり（DataFrameの行5 = Excelの行6）

### よくある落とし穴
- [ ] **NaN処理**：`pd.notna()`でnull値を確認
- [ ] **右端の列**：FYデータは多くの場合列50以上にある
- [ ] **複数の一致**：最初の一致だけでなくすべての出現を検索
- [ ] **ゼロ除算**：数式で`/`を使用する前に分母を確認（#DIV/0!）
- [ ] **間違った参照**：すべてのセル参照が意図したセルを指しているか確認（#REF!）
- [ ] **シート間参照**：シート間のリンクには正しい形式（Sheet1!A1）を使用

### 数式テスト戦略
- [ ] **小さく始める**：広く適用する前に2-3セルで数式をテスト
- [ ] **依存関係を確認**：数式で参照されるすべてのセルが存在することを確認
- [ ] **エッジケースをテスト**：ゼロ、負の値、非常に大きな値を含める

### scripts/recalc.pyの出力の解釈
スクリプトはエラーの詳細をJSON形式で返します：
```json
{
  "status": "success",           // or "errors_found"
  "total_errors": 0,              // Total error count
  "total_formulas": 42,           // Number of formulas in file
  "error_summary": {              // Only present if errors found
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    }
  }
}
```

## ベストプラクティス

### ライブラリの選択
- **pandas**：データ分析、一括操作、シンプルなデータエクスポートに最適
- **openpyxl**：複雑な書式設定、数式、Excel固有の機能に最適

### openpyxlでの作業
- セルのインデックスは1始まり（row=1、column=1はセルA1を指す）
- 計算結果の値を読み取るには`data_only=True`を使用：`load_workbook('file.xlsx', data_only=True)`
- **注意**：`data_only=True`で開いて保存すると、数式が値に置き換えられ永久に失われます
- 大きなファイルの場合：読み取りには`read_only=True`、書き込みには`write_only=True`を使用
- 数式は保持されますが評価されません - 値を更新するにはscripts/recalc.pyを使用

### pandasでの作業
- 推論の問題を避けるためにデータ型を指定：`pd.read_excel('file.xlsx', dtype={'id': str})`
- 大きなファイルの場合は特定の列を読み取り：`pd.read_excel('file.xlsx', usecols=['A', 'C', 'E'])`
- 日付を適切に処理：`pd.read_excel('file.xlsx', parse_dates=['date_column'])`

## コードスタイルガイドライン
**重要**：Excel操作用のPythonコードを生成する際：
- 不要なコメントのない最小限で簡潔なPythonコードを記述
- 冗長な変数名や不要な操作を避ける
- 不要なprint文を避ける

**Excelファイル自体について**：
- 複雑な数式や重要な前提条件にはセルにコメントを追加
- ハードコードされた値のデータソースを文書化
- 主要な計算やモデルセクションにメモを含める

---
name: slack-gif-creator
description: Slack 向けに最適化されたアニメーション GIF を制作するための知識とユーティリティ。制約条件、バリデーションツール、アニメーションコンセプトを提供する。ユーザーが「Slack 用に X が Y する GIF を作って」のようにアニメーション GIF をリクエストした際に使用する。
license: Complete terms in LICENSE.txt
---

# Slack GIF Creator

Slack 向けに最適化されたアニメーション GIF を制作するためのユーティリティと知識を提供するツールキット。

## Slack の要件

**サイズ：**
- 絵文字 GIF: 128x128（推奨）
- メッセージ GIF: 480x480

**パラメータ：**
- FPS: 10-30（低いほどファイルサイズが小さい）
- 色数: 48-128（少ないほどファイルサイズが小さい）
- 再生時間: 絵文字 GIF は3秒以内に収める

## 基本ワークフロー

```python
from core.gif_builder import GIFBuilder
from PIL import Image, ImageDraw

# 1. Create builder
builder = GIFBuilder(width=128, height=128, fps=10)

# 2. Generate frames
for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))
    draw = ImageDraw.Draw(frame)

    # Draw your animation using PIL primitives
    # (circles, polygons, lines, etc.)

    builder.add_frame(frame)

# 3. Save with optimization
builder.save('output.gif', num_colors=48, optimize_for_emoji=True)
```

## グラフィックの描画

### ユーザーがアップロードした画像の扱い
ユーザーが画像をアップロードした場合、その意図を判断する：
- **直接使用する**（例：「これをアニメーションにして」「これをフレームに分割して」）
- **インスピレーションとして使用する**（例：「こんな感じのものを作って」）

PIL を使って画像を読み込み操作する：
```python
from PIL import Image

uploaded = Image.open('file.png')
# Use directly, or just as reference for colors/style
```

### ゼロから描画する
グラフィックをゼロから描画する場合は、PIL の ImageDraw プリミティブを使用する：

```python
from PIL import ImageDraw

draw = ImageDraw.Draw(frame)

# Circles/ovals
draw.ellipse([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)

# Stars, triangles, any polygon
points = [(x1, y1), (x2, y2), (x3, y3), ...]
draw.polygon(points, fill=(r, g, b), outline=(r, g, b), width=3)

# Lines
draw.line([(x1, y1), (x2, y2)], fill=(r, g, b), width=5)

# Rectangles
draw.rectangle([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)
```

**使用しないこと：** 絵文字フォント（プラットフォーム間で信頼性が低い）。また、このスキルにプリパッケージされたグラフィックが存在すると想定しないこと。

### グラフィックの見栄えを良くする

グラフィックは洗練されてクリエイティブに見えるべきであり、基本的なものに見えてはならない。以下がそのポイント：

**太い線を使う** - アウトラインと線には常に `width=2` 以上を設定する。細い線（width=1）はガタガタしてアマチュア的に見える。

**視覚的な奥行きを加える**：
- 背景にグラデーションを使用する（`create_gradient_background`）
- 複雑さのために複数のシェイプを重ねる（例：星の中に小さな星）

**シェイプをより面白くする**：
- 単純な円を描くだけでなく、ハイライト、リング、パターンを加える
- 星にはグロー効果を付ける（背後に大きく半透明なバージョンを描く）
- 複数のシェイプを組み合わせる（星＋スパークル、円＋リング）

**色に気を配る**：
- 鮮やかで補色的な色を使う
- コントラストを加える（明るいシェイプには暗いアウトライン、暗いシェイプには明るいアウトライン）
- 全体的な構図を考慮する

**複雑なシェイプ**（ハート、雪の結晶など）の場合：
- ポリゴンと楕円の組み合わせを使う
- 対称性のためにポイントを慎重に計算する
- ディテールを加える（ハートにはハイライトカーブ、雪の結晶には精巧な枝を）

クリエイティブかつ丁寧に！良い Slack GIF は洗練されて見えるべきであり、プレースホルダーグラフィックのように見えてはならない。

## 利用可能なユーティリティ

### GIFBuilder (`core.gif_builder`)
フレームを組み立て、Slack 向けに最適化する：
```python
builder = GIFBuilder(width=128, height=128, fps=10)
builder.add_frame(frame)  # Add PIL Image
builder.add_frames(frames)  # Add list of frames
builder.save('out.gif', num_colors=48, optimize_for_emoji=True, remove_duplicates=True)
```

### Validators (`core.validators`)
GIF が Slack の要件を満たしているか確認する：
```python
from core.validators import validate_gif, is_slack_ready

# Detailed validation
passes, info = validate_gif('my.gif', is_emoji=True, verbose=True)

# Quick check
if is_slack_ready('my.gif'):
    print("Ready!")
```

### Easing Functions (`core.easing`)
リニアではなくスムーズなモーションを実現する：
```python
from core.easing import interpolate

# Progress from 0.0 to 1.0
t = i / (num_frames - 1)

# Apply easing
y = interpolate(start=0, end=400, t=t, easing='ease_out')

# Available: linear, ease_in, ease_out, ease_in_out,
#           bounce_out, elastic_out, back_out
```

### Frame Helpers (`core.frame_composer`)
よく使う処理のための便利関数：
```python
from core.frame_composer import (
    create_blank_frame,         # Solid color background
    create_gradient_background,  # Vertical gradient
    draw_circle,                # Helper for circles
    draw_text,                  # Simple text rendering
    draw_star                   # 5-pointed star
)
```

## アニメーションコンセプト

### シェイク／バイブレーション
振動でオブジェクトの位置をオフセットする：
- フレームインデックスと `math.sin()` または `math.cos()` を使用
- 自然な感じにするために小さなランダムバリエーションを加える
- x および／または y 位置に適用

### パルス／ハートビート
オブジェクトのサイズをリズミカルにスケールする：
- `math.sin(t * frequency * 2 * math.pi)` でスムーズなパルスを実現
- ハートビートの場合：2回の素早いパルスの後にポーズ（サイン波を調整）
- 基本サイズの0.8倍から1.2倍の間でスケール

### バウンス
オブジェクトが落下してバウンドする：
- 着地には `interpolate()` と `easing='bounce_out'` を使用
- 落下（加速）には `easing='ease_in'` を使用
- 各フレームで y 速度を増加させて重力を適用

### スピン／回転
オブジェクトを中心の周りで回転させる：
- PIL: `image.rotate(angle, resample=Image.BICUBIC)`
- 揺れの場合：リニアの代わりにサイン波で角度を制御

### フェードイン／フェードアウト
徐々に表示または消失する：
- RGBA 画像を作成し、アルファチャンネルを調整
- または `Image.blend(image1, image2, alpha)` を使用
- フェードイン：アルファを0から1へ
- フェードアウト：アルファを1から0へ

### スライド
オブジェクトを画面外から目標位置まで移動する：
- 開始位置：フレーム境界の外
- 終了位置：目標の場所
- `interpolate()` と `easing='ease_out'` でスムーズな停止
- オーバーシュートの場合：`easing='back_out'` を使用

### ズーム
ズーム効果のためにスケールと位置を調整する：
- ズームイン：スケールを0.1から2.0へ、中心をクロップ
- ズームアウト：スケールを2.0から1.0へ
- ドラマチックにするためにモーションブラーを追加可能（PIL フィルター）

### 爆発／パーティクルバースト
外側に放射するパーティクルを作成する：
- ランダムな角度と速度でパーティクルを生成
- 各パーティクルを更新：`x += vx`、`y += vy`
- 重力を追加：`vy += gravity_constant`
- 時間経過でパーティクルをフェードアウト（アルファを減少）

## 最適化戦略

ファイルサイズを小さくするよう求められた場合のみ、以下の方法のいくつかを実装する：

1. **フレーム数を減らす** - FPS を下げる（20の代わりに10）、またはデュレーションを短く
2. **色数を減らす** - 128の代わりに `num_colors=48`
3. **サイズを小さくする** - 480x480の代わりに128x128
4. **重複を除去する** - save() で `remove_duplicates=True`
5. **絵文字モード** - `optimize_for_emoji=True` で自動最適化

```python
# Maximum optimization for emoji
builder.save(
    'emoji.gif',
    num_colors=48,
    optimize_for_emoji=True,
    remove_duplicates=True
)
```

## フィロソフィー

このスキルが提供するもの：
- **知識**: Slack の要件とアニメーションコンセプト
- **ユーティリティ**: GIFBuilder、バリデータ、イージング関数
- **柔軟性**: PIL プリミティブを使ったアニメーションロジックの自由な制作

このスキルが提供しないもの：
- 固定的なアニメーションテンプレートやプリメイドの関数
- 絵文字フォントレンダリング（プラットフォーム間で信頼性が低い）
- スキルに組み込まれたプリパッケージのグラフィックライブラリ

**ユーザーアップロードについての注意**: このスキルにはプリビルトのグラフィックは含まれていないが、ユーザーが画像をアップロードした場合は PIL を使って読み込み操作する。リクエストに応じて直接使用するか、インスピレーションとしてのみ使用するかを判断すること。

クリエイティブに！コンセプトを組み合わせ（バウンス＋回転、パルス＋スライドなど）、PIL のフル機能を活用すること。

## 依存関係

```bash
pip install pillow imageio numpy
```

---
name: webapp-testing
description: Playwrightを使用してローカルWebアプリケーションを操作・テストするためのツールキット。フロントエンドの機能検証、UIの動作デバッグ、ブラウザのスクリーンショット取得、ブラウザログの確認をサポートします。
license: Complete terms in LICENSE.txt
---

# Webアプリケーションテスト

ローカルWebアプリケーションをテストするには、PythonのPlaywrightスクリプトを記述します。

**利用可能なヘルパースクリプト**:
- `scripts/with_server.py` - サーバーのライフサイクルを管理（複数サーバー対応）

**スクリプトは必ず最初に `--help` を付けて実行してください。** 使い方を確認するためです。スクリプトを実行してみて、カスタマイズが本当に必要だと判明するまでソースコードを読まないでください。これらのスクリプトは非常に大きく、コンテキストウィンドウを圧迫する可能性があります。コンテキストウィンドウに取り込むのではなく、ブラックボックスとして直接呼び出すことを前提に設計されています。

## 判断ツリー: アプローチの選び方

```
User task → Is it static HTML?
    ├─ Yes → Read HTML file directly to identify selectors
    │         ├─ Success → Write Playwright script using selectors
    │         └─ Fails/Incomplete → Treat as dynamic (below)
    │
    └─ No (dynamic webapp) → Is the server already running?
        ├─ No → Run: python scripts/with_server.py --help
        │        Then use the helper + write simplified Playwright script
        │
        └─ Yes → Reconnaissance-then-action:
            1. Navigate and wait for networkidle
            2. Take screenshot or inspect DOM
            3. Identify selectors from rendered state
            4. Execute actions with discovered selectors
```

## 使用例: with_server.py

サーバーを起動するには、まず `--help` を実行してから、ヘルパーを使用します:

**単一サーバーの場合:**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**複数サーバーの場合（例: バックエンド + フロントエンド）:**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

自動化スクリプトを作成する際は、Playwrightのロジックのみを含めてください（サーバーは自動的に管理されます）:
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True) # Always launch chromium in headless mode
    page = browser.new_page()
    page.goto('http://localhost:5173') # Server already running and ready
    page.wait_for_load_state('networkidle') # CRITICAL: Wait for JS to execute
    # ... your automation logic
    browser.close()
```

## 偵察してから実行するパターン

1. **レンダリング済みDOMを検査する**:
   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```

2. 検査結果から**セレクタを特定する**

3. 発見したセレクタを使って**アクションを実行する**

## よくある落とし穴

- **やってはいけないこと**: 動的アプリで `networkidle` を待たずにDOMを検査する
- **正しいやり方**: 検査の前に `page.wait_for_load_state('networkidle')` を待つ

## ベストプラクティス

- **バンドルされたスクリプトをブラックボックスとして使用する** - タスクを達成するために、`scripts/` にあるスクリプトが役立つかどうかを検討してください。これらのスクリプトは、コンテキストウィンドウを圧迫することなく、一般的で複雑なワークフローを確実に処理します。`--help` で使い方を確認してから直接呼び出してください。
- 同期スクリプトには `sync_playwright()` を使用する
- 完了時には必ずブラウザを閉じる
- 説明的なセレクタを使用する: `text=`、`role=`、CSSセレクタ、またはID
- 適切な待機を追加する: `page.wait_for_selector()` または `page.wait_for_timeout()`

## リファレンスファイル

- **examples/** - 一般的なパターンを示す例:
  - `element_discovery.py` - ページ上のボタン、リンク、入力要素の発見
  - `static_html_automation.py` - ローカルHTMLに対するfile:// URLの使用
  - `console_logging.py` - 自動化中のコンソールログのキャプチャ

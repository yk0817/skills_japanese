---
name: web-artifacts-builder
description: React、Tailwind CSS、shadcn/uiなどのモダンなフロントエンドWeb技術を使用して、精巧なマルチコンポーネントのclaude.ai HTMLアーティファクトを作成するためのツール群。状態管理、ルーティング、shadcn/uiコンポーネントを必要とする複雑なアーティファクト向けであり、単純な単一ファイルのHTML/JSXアーティファクトには不要です。
license: Complete terms in LICENSE.txt
---

# Webアーティファクトビルダー

高機能なフロントエンドのclaude.aiアーティファクトを構築するには、以下の手順に従ってください:
1. `scripts/init-artifact.sh` を使用してフロントエンドリポジトリを初期化する
2. 生成されたコードを編集してアーティファクトを開発する
3. `scripts/bundle-artifact.sh` を使用してすべてのコードを単一のHTMLファイルにバンドルする
4. ユーザーにアーティファクトを表示する
5. （任意）アーティファクトをテストする

**技術スタック**: React 18 + TypeScript + Vite + Parcel（バンドル用） + Tailwind CSS + shadcn/ui

## デザインとスタイルのガイドライン

非常に重要: いわゆる「AIスロップ」を避けるため、過度な中央寄せレイアウト、紫のグラデーション、画一的な角丸、Interフォントの使用を避けてください。

## クイックスタート

### ステップ1: プロジェクトの初期化

初期化スクリプトを実行して新しいReactプロジェクトを作成します:
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

これにより、以下が構成済みのプロジェクトが作成されます:
- React + TypeScript（Vite経由）
- Tailwind CSS 3.4.1（shadcn/uiテーマシステム付き）
- パスエイリアス（`@/`）設定済み
- 40以上のshadcn/uiコンポーネントがプリインストール
- すべてのRadix UI依存関係を含む
- Parcelのバンドル設定済み（.parcelrc経由）
- Node 18+互換（Viteバージョンを自動検出・固定）

### ステップ2: アーティファクトの開発

アーティファクトを構築するには、生成されたファイルを編集します。ガイダンスについては以下の**一般的な開発タスク**を参照してください。

### ステップ3: 単一HTMLファイルへのバンドル

ReactアプリをHTMLアーティファクト1つにバンドルするには:
```bash
bash scripts/bundle-artifact.sh
```

これにより、すべてのJavaScript、CSS、依存関係がインライン化された自己完結型のアーティファクト `bundle.html` が作成されます。このファイルはClaudeの会話でアーティファクトとして直接共有できます。

**要件**: プロジェクトのルートディレクトリに `index.html` が必要です。

**スクリプトの動作内容**:
- バンドル用依存関係のインストール（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- パスエイリアスサポート付きの`.parcelrc`設定の作成
- Parcelによるビルド（ソースマップなし）
- html-inlineを使用してすべてのアセットを単一HTMLにインライン化

### ステップ4: ユーザーへのアーティファクト共有

最後に、バンドルされたHTMLファイルをユーザーとの会話で共有して、アーティファクトとして表示できるようにします。

### ステップ5: アーティファクトのテスト/表示確認（任意）

注意: これは完全に任意のステップです。必要な場合またはリクエストされた場合のみ実行してください。

アーティファクトをテスト/表示確認するには、利用可能なツール（他のSkillsやPlaywright、Puppeteerなどの組み込みツールを含む）を使用します。一般的に、リクエストから完成したアーティファクトが見られるまでの待ち時間が増えるため、事前テストは避けてください。アーティファクトを提示した後、リクエストされた場合や問題が発生した場合にテストしてください。

## リファレンス

- **shadcn/uiコンポーネント**: https://ui.shadcn.com/docs/components

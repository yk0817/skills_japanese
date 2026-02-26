---
name: mcp-builder
description: LLMが適切に設計されたツールを通じて外部サービスと連携できる、高品質なMCP（Model Context Protocol）サーバーの構築ガイド。Python（FastMCP）またはNode/TypeScript（MCP SDK）で外部APIやサービスを統合するMCPサーバーを構築する際に使用します。
license: Complete terms in LICENSE.txt
---

# MCPサーバー開発ガイド

## 概要

LLMが適切に設計されたツールを通じて外部サービスと連携できるMCP（Model Context Protocol）サーバーを作成します。MCPサーバーの品質は、LLMが実際のタスクをどれだけうまく達成できるかで測定されます。

---

# プロセス

## 全体ワークフロー

高品質なMCPサーバーの構築には、4つの主要フェーズがあります。

### フェーズ1: 詳細な調査と計画

#### 1.1 最新のMCP設計を理解する

**APIカバレッジとワークフローツール:**
包括的なAPIエンドポイントカバレッジと、特化型ワークフローツールのバランスを取ります。ワークフローツールは特定のタスクに便利ですが、包括的なカバレッジによりエージェントは操作を柔軟に組み合わせることができます。パフォーマンスはクライアントによって異なり、基本ツールを組み合わせたコード実行が有効なクライアントもあれば、高レベルなワークフローの方が適しているクライアントもあります。迷った場合は、包括的なAPIカバレッジを優先してください。

**ツールの命名と発見しやすさ:**
明確で説明的なツール名は、エージェントが適切なツールをすばやく見つけるのに役立ちます。一貫したプレフィックス（例: `github_create_issue`、`github_list_repos`）とアクション指向の命名を使用してください。

**コンテキスト管理:**
エージェントは、簡潔なツール説明と結果のフィルタリング/ページネーション機能の恩恵を受けます。焦点を絞った関連性の高いデータを返すツールを設計してください。一部のクライアントはコード実行をサポートしており、エージェントがデータを効率的にフィルタリング・処理するのに役立ちます。

**具体的なエラーメッセージ:**
エラーメッセージは、具体的な提案と次のステップを示して、エージェントを解決策に導くべきです。

#### 1.2 MCPプロトコルのドキュメントを調査する

**MCP仕様を確認する:**

サイトマップから関連ページを見つけます: `https://modelcontextprotocol.io/sitemap.xml`

次に、Markdown形式で取得するために`.md`サフィックスを付けて特定のページを取得します（例: `https://modelcontextprotocol.io/specification/draft.md`）。

確認すべき主要ページ:
- 仕様の概要とアーキテクチャ
- トランスポートメカニズム（streamable HTTP、stdio）
- ツール、リソース、プロンプトの定義

#### 1.3 フレームワークのドキュメントを調査する

**推奨スタック:**
- **言語**: TypeScript（高品質なSDKサポートと、多くの実行環境での良好な互換性（例: MCPB）。またAIモデルはTypeScriptコードの生成が得意で、広い利用範囲、静的型付け、優れたリンティングツールの恩恵を受けます）
- **トランスポート**: リモートサーバーにはstreamable HTTP、ステートレスJSONを使用（ステートフルセッションやストリーミングレスポンスと比べ、スケーリングとメンテナンスが容易）。ローカルサーバーにはstdioを使用。

**フレームワークのドキュメントを読み込む:**

- **MCPベストプラクティス**: [📋 ベストプラクティスを見る](./reference/mcp_best_practices.md) - 基本ガイドライン

**TypeScript（推奨）の場合:**
- **TypeScript SDK**: WebFetchで `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md` を読み込む
- [⚡ TypeScriptガイド](./reference/node_mcp_server.md) - TypeScriptのパターンと例

**Pythonの場合:**
- **Python SDK**: WebFetchで `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md` を読み込む
- [🐍 Pythonガイド](./reference/python_mcp_server.md) - Pythonのパターンと例

#### 1.4 実装を計画する

**APIを理解する:**
サービスのAPIドキュメントを確認し、主要なエンドポイント、認証要件、データモデルを把握します。必要に応じてWeb検索やWebFetchを使用してください。

**ツールの選定:**
包括的なAPIカバレッジを優先します。実装するエンドポイントを列挙し、最も一般的な操作から着手します。

---

### フェーズ2: 実装

#### 2.1 プロジェクト構造を設定する

プロジェクトの設定については、言語別ガイドを参照してください:
- [⚡ TypeScriptガイド](./reference/node_mcp_server.md) - プロジェクト構造、package.json、tsconfig.json
- [🐍 Pythonガイド](./reference/python_mcp_server.md) - モジュール構成、依存関係

#### 2.2 コアインフラを実装する

共有ユーティリティを作成します:
- 認証付きAPIクライアント
- エラーハンドリングヘルパー
- レスポンスフォーマット（JSON/Markdown）
- ページネーションサポート

#### 2.3 ツールを実装する

各ツールについて:

**入力スキーマ:**
- Zod（TypeScript）またはPydantic（Python）を使用
- 制約と明確な説明を含める
- フィールド説明に例を追加

**出力スキーマ:**
- 構造化データには可能な限り`outputSchema`を定義
- ツールレスポンスで`structuredContent`を使用（TypeScript SDKの機能）
- クライアントがツール出力を理解・処理するのに役立つ

**ツールの説明:**
- 機能の簡潔な要約
- パラメータの説明
- 戻り値の型スキーマ

**実装:**
- I/O操作にはasync/awaitを使用
- 具体的なメッセージを含む適切なエラーハンドリング
- 必要に応じてページネーションをサポート
- 最新のSDKを使用する場合はテキストコンテンツと構造化データの両方を返す

**アノテーション:**
- `readOnlyHint`: true/false
- `destructiveHint`: true/false
- `idempotentHint`: true/false
- `openWorldHint`: true/false

---

### フェーズ3: レビューとテスト

#### 3.1 コード品質

以下の観点でレビュー:
- 重複コードがないこと（DRY原則）
- 一貫したエラーハンドリング
- 完全な型カバレッジ
- 明確なツール説明

#### 3.2 ビルドとテスト

**TypeScript:**
- `npm run build` を実行してコンパイルを確認
- MCP Inspectorでテスト: `npx @modelcontextprotocol/inspector`

**Python:**
- 構文を検証: `python -m py_compile your_server.py`
- MCP Inspectorでテスト

詳細なテスト手法と品質チェックリストについては、言語別ガイドを参照してください。

---

### フェーズ4: 評価の作成

MCPサーバーの実装後、その有効性をテストするための包括的な評価を作成します。

**完全な評価ガイドラインについては [✅ 評価ガイド](./reference/evaluation.md) を読み込んでください。**

#### 4.1 評価の目的を理解する

評価を使用して、LLMがMCPサーバーを効果的に使用し、現実的で複雑な質問に回答できるかどうかをテストします。

#### 4.2 10個の評価用質問を作成する

効果的な評価を作成するには、評価ガイドに記載されたプロセスに従ってください:

1. **ツールの検査**: 利用可能なツールを列挙し、その機能を理解する
2. **コンテンツの調査**: 読み取り専用の操作で利用可能なデータを探索する
3. **質問の生成**: 10個の複雑で現実的な質問を作成する
4. **回答の検証**: 各質問を自分で解いて回答を確認する

#### 4.3 評価の要件

各質問が以下を満たしていることを確認:
- **独立性**: 他の質問に依存しない
- **読み取り専用**: 非破壊的な操作のみ必要
- **複雑性**: 複数のツール呼び出しと深い探索が必要
- **現実性**: 人間が実際に関心を持つユースケースに基づく
- **検証可能性**: 文字列比較で検証できる単一の明確な回答
- **安定性**: 回答が時間の経過とともに変化しない

#### 4.4 出力フォーマット

以下の構造でXMLファイルを作成:

```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
<!-- More qa_pairs... -->
</evaluation>
```

---

# リファレンスファイル

## ドキュメントライブラリ

開発中に必要に応じてこれらのリソースを読み込んでください:

### MCPコアドキュメント（最初に読み込む）
- **MCPプロトコル**: `https://modelcontextprotocol.io/sitemap.xml` のサイトマップから開始し、`.md`サフィックスを付けて特定のページを取得
- [📋 MCPベストプラクティス](./reference/mcp_best_practices.md) - 以下を含むMCPの共通ガイドライン:
  - サーバーとツールの命名規則
  - レスポンスフォーマットのガイドライン（JSON vs Markdown）
  - ページネーションのベストプラクティス
  - トランスポートの選択（streamable HTTP vs stdio）
  - セキュリティとエラーハンドリングの基準

### SDKドキュメント（フェーズ1/2で読み込む）
- **Python SDK**: `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md` から取得
- **TypeScript SDK**: `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md` から取得

### 言語別実装ガイド（フェーズ2で読み込む）
- [🐍 Python実装ガイド](./reference/python_mcp_server.md) - Python/FastMCPの完全ガイド:
  - サーバー初期化パターン
  - Pydanticモデルの例
  - `@mcp.tool`によるツール登録
  - 完全な動作例
  - 品質チェックリスト

- [⚡ TypeScript実装ガイド](./reference/node_mcp_server.md) - TypeScriptの完全ガイド:
  - プロジェクト構造
  - Zodスキーマパターン
  - `server.registerTool`によるツール登録
  - 完全な動作例
  - 品質チェックリスト

### 評価ガイド（フェーズ4で読み込む）
- [✅ 評価ガイド](./reference/evaluation.md) - 評価作成の完全ガイド:
  - 質問作成のガイドライン
  - 回答検証の戦略
  - XMLフォーマットの仕様
  - 質問と回答の例
  - 提供されたスクリプトによる評価の実行

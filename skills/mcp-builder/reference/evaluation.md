# MCP サーバー評価ガイド

## 概要

このドキュメントは、MCP サーバーの包括的な評価を作成するためのガイダンスを提供します。評価では、LLM が提供されたツールのみを使用して、現実的で複雑な質問に効果的に回答できるかどうかをテストします。

---

## クイックリファレンス

### 評価の要件
- 人間が読める質問を10問作成する
- 質問は読み取り専用、独立、非破壊的であること
- 各質問は複数のツール呼び出し（場合によっては数十回）を必要とすること
- 回答は単一の検証可能な値であること
- 回答は安定していること（時間の経過とともに変化しない）

### 出力フォーマット
```xml
<evaluation>
   <qa_pair>
      <question>Your question here</question>
      <answer>Single verifiable answer</answer>
   </qa_pair>
</evaluation>
```

---

## 評価の目的

MCP サーバーの品質を測る基準は、サーバーがどれだけ適切かつ包括的にツールを実装しているかではなく、これらの実装（入出力スキーマ、docstring/説明、機能）が、他のコンテキストを持たず MCP サーバーのみにアクセスできる LLM に対して、現実的で難しい質問にどれだけうまく回答させられるかです。

## 評価の概要

読み取り専用、独立、非破壊的、かつ冪等な操作のみを必要とする、人間が読める質問を10問作成します。各質問は以下を満たす必要があります：
- 現実的であること
- 明確で簡潔であること
- 曖昧でないこと
- 複雑で、数十回のツール呼び出しやステップを必要とする可能性があること
- 事前に特定した単一の検証可能な値で回答できること

## 質問のガイドライン

### 基本要件

1. **質問は独立していなければならない**
   - 各質問は他の質問の回答に依存してはならない
   - 別の質問の処理による事前の書き込み操作を前提としてはならない

2. **質問は非破壊的かつ冪等なツール使用のみを必要としなければならない**
   - 正しい回答に到達するために状態を変更する指示や要件を含めてはならない

3. **質問は現実的、明確、簡潔、かつ複雑でなければならない**
   - 別の LLM が回答するために複数（場合によっては数十の）ツールやステップを使用する必要があること

### 複雑さと深さ

4. **質問は深い探索を必要としなければならない**
   - 複数のサブ質問と連続的なツール呼び出しを必要とするマルチホップ質問を検討する
   - 各ステップは前の質問で見つかった情報から恩恵を受けるべき

5. **質問は広範なページングを必要とする場合がある**
   - 複数ページの結果をページングする必要がある場合がある
   - ニッチな情報を見つけるために古いデータ（1〜2年前のデータ）をクエリする必要がある場合がある
   - 質問は難しくなければならない

6. **質問は深い理解を必要としなければならない**
   - 表面的な知識ではなく深い理解が必要
   - 証拠を必要とする複雑なアイデアを真偽問題として提示する場合がある
   - LLM が異なる仮説を検索する必要がある選択問題形式を使用する場合がある

7. **質問は単純なキーワード検索で解決できてはならない**
   - ターゲットコンテンツの特定のキーワードを含めない
   - 同義語、関連概念、または言い換えを使用する
   - 複数の検索、複数の関連アイテムの分析、コンテキストの抽出、そして回答の導出を必要とする

### ツールのテスト

8. **質問はツールの戻り値をストレステストすべき**
   - 大きな JSON オブジェクトやリストを返すツールを引き出し、LLM に負荷をかける場合がある
   - 複数のデータモダリティの理解を必要とすべき：
     - ID と名前
     - タイムスタンプと日時（月、日、年、秒）
     - ファイル ID、名前、拡張子、MIME タイプ
     - URL、GID など
   - ツールがすべての有用なデータ形式を返す能力を検証すべき

9. **質問はおおむね実際の人間のユースケースを反映すべき**
   - LLM の支援を受けた人間が関心を持つような情報検索タスクの種類

10. **質問は数十回のツール呼び出しを必要とする場合がある**
    - これはコンテキストが限られた LLM にとって課題となる
    - MCP サーバーのツールが返す情報量を削減することを促進する

11. **曖昧な質問を含める**
    - 曖昧であるか、どのツールを呼び出すかについて難しい判断を必要とする場合がある
    - LLM が間違いを犯したり誤解したりする可能性を強制する
    - 曖昧さがあっても、単一の検証可能な回答が存在することを確認する

### 安定性

12. **質問は回答が変化しないように設計しなければならない**
    - 動的な「現在の状態」に依存する質問をしない
    - 例えば、以下をカウントしない：
      - 投稿へのリアクション数
      - スレッドへの返信数
      - チャンネルのメンバー数

13. **MCP サーバーが作成する質問の種類を制限してはならない**
    - 挑戦的で複雑な質問を作成する
    - 利用可能な MCP サーバーツールでは解決できないものもあってよい
    - 質問は特定の出力フォーマット（datetime 対 epoch time、JSON 対 MARKDOWN）を要求する場合がある
    - 質問は完了するまでに数十回のツール呼び出しを必要とする場合がある

## 回答のガイドライン

### 検証

1. **回答は直接的な文字列比較で検証可能でなければならない**
   - 回答が複数のフォーマットで書き直せる場合は、質問で出力フォーマットを明確に指定する
   - 例："Use YYYY/MM/DD.", "Respond True or False.", "Answer A, B, C, or D and nothing else."
   - 回答は以下のような単一の検証可能な値であるべき：
     - ユーザー ID、ユーザー名、表示名、名、姓
     - チャンネル ID、チャンネル名
     - メッセージ ID、文字列
     - URL、タイトル
     - 数値
     - タイムスタンプ、日時
     - ブール値（真偽問題の場合）
     - メールアドレス、電話番号
     - ファイル ID、ファイル名、ファイル拡張子
     - 選択問題の回答
   - 回答は特別なフォーマットや複雑な構造化出力を必要としてはならない
   - 回答は直接的な文字列比較を使用して検証される

### 可読性

2. **回答は一般的に人間が読みやすいフォーマットを優先すべき**
   - 例：名前、名、姓、日時、ファイル名、メッセージ文字列、URL、yes/no、true/false、a/b/c/d
   - 不透明な ID よりも読みやすい値を優先（ただし ID も許容される）
   - 回答の大多数は人間が読みやすいものであるべき

### 安定性

3. **回答は安定/定常でなければならない**
   - 古いコンテンツを参照する（例：終了した会話、ローンチ済みのプロジェクト、回答済みの質問）
   - 常に同じ回答を返す「クローズド」な概念に基づいて質問を作成する
   - 非定常な回答から隔離するために固定の時間枠を考慮するよう質問する場合がある
   - 変化する可能性が低いコンテキストに依存する
   - 例：論文名を見つける場合、後で発表される論文と混同されないよう十分に具体的にする

4. **回答は明確で曖昧でなければならない**
   - 質問は単一の明確な回答があるように設計しなければならない
   - 回答は MCP サーバーツールを使用して導出できる

### 多様性

5. **回答は多様でなければならない**
   - 回答は多様なモダリティとフォーマットの単一の検証可能な値であるべき
   - ユーザー概念：ユーザー ID、ユーザー名、表示名、名、姓、メールアドレス、電話番号
   - チャンネル概念：チャンネル ID、チャンネル名、チャンネルトピック
   - メッセージ概念：メッセージ ID、メッセージ文字列、タイムスタンプ、月、日、年

6. **回答は複雑な構造であってはならない**
   - 値のリストではない
   - 複雑なオブジェクトではない
   - ID や文字列のリストではない
   - 自然言語テキストではない
   - ただし、直接的な文字列比較で簡単に検証でき、現実的に再現可能な場合は除く
   - LLM が同じリストを他の順序やフォーマットで返す可能性が低いこと

## 評価プロセス

### ステップ 1：ドキュメントの検査

対象 API のドキュメントを読み、以下を理解する：
- 利用可能なエンドポイントと機能
- 曖昧さがある場合は、ウェブから追加情報を取得する
- このステップを可能な限り並列化する
- 各サブエージェントがファイルシステムまたはウェブからのドキュメントのみを調査していることを確認する

### ステップ 2：ツールの検査

MCP サーバーで利用可能なツールをリストアップする：
- MCP サーバーを直接検査する
- 入出力スキーマ、docstring、説明を理解する
- この段階ではツール自体を呼び出さない

### ステップ 3：理解の深化

十分な理解が得られるまでステップ 1 と 2 を繰り返す：
- 複数回反復する
- 作成したいタスクの種類について考える
- 理解を洗練させる
- どの段階でも MCP サーバー実装のコード自体を読んではならない
- 直感と理解を使用して、合理的で現実的だが非常に挑戦的なタスクを作成する

### ステップ 4：読み取り専用のコンテンツ検査

API とツールを理解した後、MCP サーバーツールを使用する：
- 読み取り専用かつ非破壊的な操作のみを使用してコンテンツを検査する
- 目標：現実的な質問を作成するための特定のコンテンツ（ユーザー、チャンネル、メッセージ、プロジェクト、タスクなど）を特定する
- 状態を変更するツールを呼び出してはならない
- MCP サーバー実装のコード自体を読んではならない
- 個々のサブエージェントが独立した探索を行うようにこのステップを並列化する
- 各サブエージェントが読み取り専用、非破壊的、冪等な操作のみを実行していることを確認する
- 注意：一部のツールは大量のデータを返す場合があり、コンテキストが不足する可能性がある
- 増分的で小さく、的を絞ったツール呼び出しを行う
- すべてのツール呼び出しリクエストで `limit` パラメータを使用して結果を制限する（10未満）
- ページネーションを使用する

### ステップ 5：タスク生成

コンテンツを検査した後、人間が読める質問を10問作成する：
- LLM が MCP サーバーを使用して回答できるものであること
- 上記のすべての質問と回答のガイドラインに従うこと

## 出力フォーマット

各 QA ペアは質問と回答で構成されます。出力は以下の構造の XML ファイルであるべきです：

```xml
<evaluation>
   <qa_pair>
      <question>Find the project created in Q2 2024 with the highest number of completed tasks. What is the project name?</question>
      <answer>Website Redesign</answer>
   </qa_pair>
   <qa_pair>
      <question>Search for issues labeled as "bug" that were closed in March 2024. Which user closed the most issues? Provide their username.</question>
      <answer>sarah_dev</answer>
   </qa_pair>
   <qa_pair>
      <question>Look for pull requests that modified files in the /api directory and were merged between January 1 and January 31, 2024. How many different contributors worked on these PRs?</question>
      <answer>7</answer>
   </qa_pair>
   <qa_pair>
      <question>Find the repository with the most stars that was created before 2023. What is the repository name?</question>
      <answer>data-pipeline</answer>
   </qa_pair>
</evaluation>
```

## 評価の例

### 良い質問

**例 1：深い探索を必要とするマルチホップ質問（GitHub MCP）**
```xml
<qa_pair>
   <question>Find the repository that was archived in Q3 2023 and had previously been the most forked project in the organization. What was the primary programming language used in that repository?</question>
   <answer>Python</answer>
</qa_pair>
```

この質問が良い理由：
- アーカイブされたリポジトリを見つけるために複数の検索が必要
- アーカイブ前に最もフォークされていたものを特定する必要がある
- 言語を調べるためにリポジトリの詳細を確認する必要がある
- 回答はシンプルで検証可能な値
- 変化しない過去の（クローズされた）データに基づいている

**例 2：キーワードマッチングなしでコンテキストの理解が必要（プロジェクト管理 MCP）**
```xml
<qa_pair>
   <question>Locate the initiative focused on improving customer onboarding that was completed in late 2023. The project lead created a retrospective document after completion. What was the lead's role title at that time?</question>
   <answer>Product Manager</answer>
</qa_pair>
```

この質問が良い理由：
- 特定のプロジェクト名を使用していない（「顧客オンボーディングの改善に焦点を当てたイニシアチブ」）
- 特定の期間に完了したプロジェクトを見つける必要がある
- プロジェクトリーダーとその役職を特定する必要がある
- 振り返りドキュメントからコンテキストを理解する必要がある
- 回答は人間が読みやすく安定している
- 完了した作業に基づいている（変化しない）

**例 3：複数のステップを必要とする複雑な集計（イシュートラッカー MCP）**
```xml
<qa_pair>
   <question>Among all bugs reported in January 2024 that were marked as critical priority, which assignee resolved the highest percentage of their assigned bugs within 48 hours? Provide the assignee's username.</question>
   <answer>alex_eng</answer>
</qa_pair>
```

この質問が良い理由：
- 日付、優先度、ステータスでバグをフィルタリングする必要がある
- 担当者ごとにグループ化し、解決率を計算する必要がある
- 48時間のウィンドウを判定するためにタイムスタンプの理解が必要
- ページネーションをテストする（処理すべきバグが多い可能性がある）
- 回答は単一のユーザー名
- 特定の期間の過去のデータに基づいている

**例 4：複数のデータタイプにまたがる統合が必要（CRM MCP）**
```xml
<qa_pair>
   <question>Find the account that upgraded from the Starter to Enterprise plan in Q4 2023 and had the highest annual contract value. What industry does this account operate in?</question>
   <answer>Healthcare</answer>
</qa_pair>
```

この質問が良い理由：
- サブスクリプションティアの変更を理解する必要がある
- 特定の期間のアップグレードイベントを特定する必要がある
- 契約金額を比較する必要がある
- アカウントの業種情報にアクセスする必要がある
- 回答はシンプルで検証可能
- 完了した過去のトランザクションに基づいている

### 悪い質問

**例 1：回答が時間とともに変化する**
```xml
<qa_pair>
   <question>How many open issues are currently assigned to the engineering team?</question>
   <answer>47</answer>
</qa_pair>
```

この質問が悪い理由：
- イシューの作成、クローズ、再割り当てに応じて回答が変化する
- 安定/定常なデータに基づいていない
- 動的な「現在の状態」に依存している

**例 2：キーワード検索で簡単に解ける**
```xml
<qa_pair>
   <question>Find the pull request with title "Add authentication feature" and tell me who created it.</question>
   <answer>developer123</answer>
</qa_pair>
```

この質問が悪い理由：
- 正確なタイトルのキーワード検索で簡単に解決できる
- 深い探索や理解を必要としない
- 統合や分析が不要

**例 3：回答フォーマットが曖昧**
```xml
<qa_pair>
   <question>List all the repositories that have Python as their primary language.</question>
   <answer>repo1, repo2, repo3, data-pipeline, ml-tools</answer>
</qa_pair>
```

この質問が悪い理由：
- 回答がどの順序でも返される可能性のあるリスト
- 直接的な文字列比較での検証が困難
- LLM が異なるフォーマット（JSON配列、カンマ区切り、改行区切り）で出力する可能性がある
- 特定の集計値（カウント）や最上位のもの（最もスターが多い）を尋ねる方が良い

## 検証プロセス

評価を作成した後：

1. **XML ファイルを検査**してスキーマを理解する
2. **各タスクの指示を読み込み**、MCP サーバーとツールを使用して並列に、タスクを自分自身で解決して正しい回答を特定する
3. **書き込みまたは破壊的な操作を必要とする操作をフラグ付けする**
4. **すべての正しい回答を蓄積し**、ドキュメント内の不正確な回答を置き換える
5. **書き込みまたは破壊的な操作を必要とする `<qa_pair>` を削除する**

コンテキスト不足を避けるためにタスクの解決を並列化し、すべての回答を蓄積してから最後にファイルを変更することを忘れないでください。

## 高品質な評価を作成するためのヒント

1. **タスクを生成する前によく考え、計画を立てる**
2. **機会があれば並列化する**ことでプロセスを高速化し、コンテキストを管理する
3. **現実的なユースケースに焦点を当てる**人間が実際に達成したいタスク
4. **挑戦的な質問を作成する**MCP サーバーの能力の限界をテストする
5. **安定性を確保する**過去のデータとクローズされた概念を使用する
6. **回答を検証する**MCP サーバーツールを使用して自分で質問を解く
7. **反復的に改善する**プロセスで学んだことに基づいて

---

# 評価の実行

評価ファイルを作成した後、提供された評価ハーネスを使用して MCP サーバーをテストできます。

## セットアップ

1. **依存関係のインストール**

   ```bash
   pip install -r scripts/requirements.txt
   ```

   または手動でインストール：
   ```bash
   pip install anthropic mcp
   ```

2. **API キーの設定**

   ```bash
   export ANTHROPIC_API_KEY=your_api_key_here
   ```

## 評価ファイルフォーマット

評価ファイルは `<qa_pair>` 要素を含む XML フォーマットを使用します：

```xml
<evaluation>
   <qa_pair>
      <question>Find the project created in Q2 2024 with the highest number of completed tasks. What is the project name?</question>
      <answer>Website Redesign</answer>
   </qa_pair>
   <qa_pair>
      <question>Search for issues labeled as "bug" that were closed in March 2024. Which user closed the most issues? Provide their username.</question>
      <answer>sarah_dev</answer>
   </qa_pair>
</evaluation>
```

## 評価の実行

評価スクリプト（`scripts/evaluation.py`）は3つのトランスポートタイプをサポートしています：

**重要：**
- **stdio トランスポート**：評価スクリプトが MCP サーバープロセスを自動的に起動・管理します。手動でサーバーを起動しないでください。
- **sse/http トランスポート**：評価を実行する前に MCP サーバーを別途起動する必要があります。スクリプトは指定された URL で既に実行中のサーバーに接続します。

### 1. ローカル STDIO サーバー

ローカルで実行する MCP サーバーの場合（スクリプトがサーバーを自動起動）：

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  evaluation.xml
```

環境変数を指定する場合：
```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  -e API_KEY=abc123 \
  -e DEBUG=true \
  evaluation.xml
```

### 2. Server-Sent Events (SSE)

SSE ベースの MCP サーバーの場合（先にサーバーを起動する必要あり）：

```bash
python scripts/evaluation.py \
  -t sse \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  -H "X-Custom-Header: value" \
  evaluation.xml
```

### 3. HTTP（Streamable HTTP）

HTTP ベースの MCP サーバーの場合（先にサーバーを起動する必要あり）：

```bash
python scripts/evaluation.py \
  -t http \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  evaluation.xml
```

## コマンドラインオプション

```
usage: evaluation.py [-h] [-t {stdio,sse,http}] [-m MODEL] [-c COMMAND]
                     [-a ARGS [ARGS ...]] [-e ENV [ENV ...]] [-u URL]
                     [-H HEADERS [HEADERS ...]] [-o OUTPUT]
                     eval_file

positional arguments:
  eval_file             Path to evaluation XML file

optional arguments:
  -h, --help            Show help message
  -t, --transport       Transport type: stdio, sse, or http (default: stdio)
  -m, --model           Claude model to use (default: claude-3-7-sonnet-20250219)
  -o, --output          Output file for report (default: print to stdout)

stdio options:
  -c, --command         Command to run MCP server (e.g., python, node)
  -a, --args            Arguments for the command (e.g., server.py)
  -e, --env             Environment variables in KEY=VALUE format

sse/http options:
  -u, --url             MCP server URL
  -H, --header          HTTP headers in 'Key: Value' format
```

## 出力

評価スクリプトは以下を含む詳細なレポートを生成します：

- **サマリー統計**：
  - 正解率（正解数/合計）
  - 平均タスク所要時間
  - タスクあたりの平均ツール呼び出し数
  - ツール呼び出し総数

- **タスクごとの結果**：
  - プロンプトと期待される回答
  - エージェントからの実際の回答
  - 回答が正しかったかどうか
  - 所要時間とツール呼び出しの詳細
  - エージェントのアプローチのサマリー
  - ツールに対するエージェントのフィードバック

### レポートをファイルに保存

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_server.py \
  -o evaluation_report.md \
  evaluation.xml
```

## 完全なワークフロー例

評価の作成と実行の完全な例を示します：

1. **評価ファイルの作成**（`my_evaluation.xml`）：

```xml
<evaluation>
   <qa_pair>
      <question>Find the user who created the most issues in January 2024. What is their username?</question>
      <answer>alice_developer</answer>
   </qa_pair>
   <qa_pair>
      <question>Among all pull requests merged in Q1 2024, which repository had the highest number? Provide the repository name.</question>
      <answer>backend-api</answer>
   </qa_pair>
   <qa_pair>
      <question>Find the project that was completed in December 2023 and had the longest duration from start to finish. How many days did it take?</question>
      <answer>127</answer>
   </qa_pair>
</evaluation>
```

2. **依存関係のインストール**：

```bash
pip install -r scripts/requirements.txt
export ANTHROPIC_API_KEY=your_api_key
```

3. **評価の実行**：

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a github_mcp_server.py \
  -e GITHUB_TOKEN=ghp_xxx \
  -o github_eval_report.md \
  my_evaluation.xml
```

4. **レポートの確認**（`github_eval_report.md`）：
   - どの質問が合格/不合格だったかを確認する
   - ツールに対するエージェントのフィードバックを読む
   - 改善点を特定する
   - MCP サーバーの設計を反復改善する

## トラブルシューティング

### 接続エラー

接続エラーが発生した場合：
- **STDIO**：コマンドと引数が正しいことを確認する
- **SSE/HTTP**：URL にアクセスできること、ヘッダーが正しいことを確認する
- 必要な API キーが環境変数またはヘッダーに設定されていることを確認する

### 正解率が低い

多くの評価が失敗する場合：
- 各タスクに対するエージェントのフィードバックを確認する
- ツールの説明が明確で包括的かどうかを確認する
- 入力パラメータが十分にドキュメント化されているか確認する
- ツールが返すデータが多すぎるか少なすぎるかを検討する
- エラーメッセージが実行可能なものかを確認する

### タイムアウトの問題

タスクがタイムアウトする場合：
- より高性能なモデルを使用する（例：`claude-3-7-sonnet-20250219`）
- ツールが返すデータ量が多すぎないか確認する
- ページネーションが正しく動作しているか確認する
- 複雑な質問を簡素化することを検討する

# 事後分析エージェント

ブラインド比較の結果を分析して、勝者が勝った理由を理解し、改善提案を生成します。

## 役割

ブラインドコンパレータが勝者を決定した後、事後分析エージェントはスキルとトランスクリプトを調査して結果を「アンブラインド」します。目標は、実行可能なインサイトを抽出することです：何が勝者をより良くしたのか、どうすれば敗者を改善できるのか。

## 入力

プロンプトで以下のパラメータを受け取ります：

- **winner**: "A" または "B"（ブラインド比較から）
- **winner_skill_path**: 勝った出力を生成したスキルへのパス
- **winner_transcript_path**: 勝者の実行トランスクリプトへのパス
- **loser_skill_path**: 負けた出力を生成したスキルへのパス
- **loser_transcript_path**: 敗者の実行トランスクリプトへのパス
- **comparison_result_path**: ブラインドコンパレータの出力 JSON へのパス
- **output_path**: 分析結果を保存する場所

## プロセス

### ステップ 1：比較結果を読む

1. comparison_result_path にあるブラインドコンパレータの出力を読む
2. 勝者側（A または B）、理由、スコアを確認する
3. コンパレータが勝者の出力で何を評価したかを理解する

### ステップ 2：両方のスキルを読む

1. 勝者スキルの SKILL.md と主要な参照ファイルを読む
2. 敗者スキルの SKILL.md と主要な参照ファイルを読む
3. 構造的な違いを特定する：
   - 指示の明確さと具体性
   - スクリプト/ツールの使用パターン
   - 例のカバレッジ
   - エッジケースの処理

### ステップ 3：両方のトランスクリプトを読む

1. 勝者のトランスクリプトを読む
2. 敗者のトランスクリプトを読む
3. 実行パターンを比較する：
   - それぞれがスキルの指示にどれだけ忠実に従ったか
   - どのツールが異なる方法で使用されたか
   - 敗者がどこで最適な動作から逸脱したか
   - どちらかがエラーに遭遇したり、リカバリーを試みたか

### ステップ 4：指示の遵守を分析する

各トランスクリプトについて以下を評価する：
- エージェントはスキルの明示的な指示に従ったか
- エージェントはスキルが提供するツール/スクリプトを使用したか
- スキルのコンテンツを活用する機会を逃していなかったか
- エージェントはスキルにない不要なステップを追加したか

指示の遵守度を1〜10でスコアリングし、具体的な問題を記録する。

### ステップ 5：勝者の強みを特定する

勝者が優れていた理由を判定する：
- より良い動作につながった明確な指示？
- より良い出力を生成した優れたスクリプト/ツール？
- エッジケースをガイドした包括的な例？
- より良いエラーハンドリングのガイダンス？

具体的に記述する。関連箇所ではスキル/トランスクリプトから引用する。

### ステップ 6：敗者の弱点を特定する

敗者の足を引っ張ったものを判定する：
- 最適でない選択につながった曖昧な指示？
- 回避策を強いたツール/スクリプトの不足？
- エッジケースのカバレッジのギャップ？
- 失敗を引き起こした不十分なエラーハンドリング？

### ステップ 7：改善提案を生成する

分析に基づき、敗者スキルを改善するための実行可能な提案を作成する：
- 行うべき具体的な指示の変更
- 追加または修正すべきツール/スクリプト
- 含めるべき例
- 対処すべきエッジケース

インパクトの大きさで優先順位を付ける。結果を変えたであろう変更に焦点を当てる。

### ステップ 8：分析結果を書き出す

構造化された分析を `{output_path}` に保存する。

## 出力フォーマット

以下の構造で JSON ファイルを書き出します：

```json
{
  "comparison_summary": {
    "winner": "A",
    "winner_skill": "path/to/winner/skill",
    "loser_skill": "path/to/loser/skill",
    "comparator_reasoning": "Brief summary of why comparator chose winner"
  },
  "winner_strengths": [
    "Clear step-by-step instructions for handling multi-page documents",
    "Included validation script that caught formatting errors",
    "Explicit guidance on fallback behavior when OCR fails"
  ],
  "loser_weaknesses": [
    "Vague instruction 'process the document appropriately' led to inconsistent behavior",
    "No script for validation, agent had to improvise and made errors",
    "No guidance on OCR failure, agent gave up instead of trying alternatives"
  ],
  "instruction_following": {
    "winner": {
      "score": 9,
      "issues": [
        "Minor: skipped optional logging step"
      ]
    },
    "loser": {
      "score": 6,
      "issues": [
        "Did not use the skill's formatting template",
        "Invented own approach instead of following step 3",
        "Missed the 'always validate output' instruction"
      ]
    }
  },
  "improvement_suggestions": [
    {
      "priority": "high",
      "category": "instructions",
      "suggestion": "Replace 'process the document appropriately' with explicit steps: 1) Extract text, 2) Identify sections, 3) Format per template",
      "expected_impact": "Would eliminate ambiguity that caused inconsistent behavior"
    },
    {
      "priority": "high",
      "category": "tools",
      "suggestion": "Add validate_output.py script similar to winner skill's validation approach",
      "expected_impact": "Would catch formatting errors before final output"
    },
    {
      "priority": "medium",
      "category": "error_handling",
      "suggestion": "Add fallback instructions: 'If OCR fails, try: 1) different resolution, 2) image preprocessing, 3) manual extraction'",
      "expected_impact": "Would prevent early failure on difficult documents"
    }
  ],
  "transcript_insights": {
    "winner_execution_pattern": "Read skill -> Followed 5-step process -> Used validation script -> Fixed 2 issues -> Produced output",
    "loser_execution_pattern": "Read skill -> Unclear on approach -> Tried 3 different methods -> No validation -> Output had errors"
  }
}
```

## ガイドライン

- **具体的に記述する**：スキルやトランスクリプトから引用する。単に「指示が不明確だった」と言わない
- **実行可能にする**：提案は漠然としたアドバイスではなく、具体的な変更であるべき
- **スキルの改善に焦点を当てる**：目標はエージェントを批判することではなく、敗者スキルを改善すること
- **インパクトで優先順位を付ける**：どの変更が最も結果を変える可能性が高いか
- **因果関係を考慮する**：スキルの弱点が実際に悪い出力を引き起こしたのか、偶然の一致か
- **客観的であること**：何が起こったかを分析し、主観的な意見を述べない
- **一般化を考える**：この改善は他の評価でも役立つか

## 提案のカテゴリ

改善提案を整理するために以下のカテゴリを使用します：

| カテゴリ | 説明 |
|----------|-------------|
| `instructions` | スキルの文章による指示の変更 |
| `tools` | 追加/修正すべきスクリプト、テンプレート、ユーティリティ |
| `examples` | 含めるべき入出力の例 |
| `error_handling` | 失敗処理のガイダンス |
| `structure` | スキルコンテンツの再構成 |
| `references` | 追加すべき外部ドキュメントやリソース |

## 優先度レベル

- **high**：この比較の結果を変える可能性が高い
- **medium**：品質は向上するが、勝敗は変わらない可能性がある
- **low**：あれば望ましい、わずかな改善

---

# ベンチマーク結果の分析

ベンチマーク結果を分析する場合、分析エージェントの目的はスキルの改善を提案することではなく、複数の実行にわたる**パターンと異常を浮き彫りにする**ことです。

## 役割

すべてのベンチマーク実行結果をレビューし、スキルのパフォーマンスをユーザーが理解するのに役立つフリーフォームのノートを生成します。集計メトリクスだけでは見えないパターンに焦点を当てます。

## 入力

プロンプトで以下のパラメータを受け取ります：

- **benchmark_data_path**: すべての実行結果を含む進行中の benchmark.json へのパス
- **skill_path**: ベンチマークされているスキルへのパス
- **output_path**: ノートを保存する場所（文字列の JSON 配列として）

## プロセス

### ステップ 1：ベンチマークデータを読む

1. すべての実行結果を含む benchmark.json を読む
2. テストされた構成（with_skill、without_skill）を確認する
3. 既に計算された run_summary の集計を理解する

### ステップ 2：アサーションごとのパターンを分析する

すべての実行にわたる各期待値について：
- 両方の構成で**常にパスする**か？（スキルの価値を差別化できない可能性がある）
- 両方の構成で**常に失敗する**か？（壊れているか能力を超えている可能性がある）
- **スキルありでは常にパスするがスキルなしでは失敗する**か？（スキルが明確に価値を追加している）
- **スキルありでは常に失敗するがスキルなしではパスする**か？（スキルが悪影響を与えている可能性がある）
- **非常にばらつきが大きい**か？（不安定な期待値または非決定的な動作）

### ステップ 3：評価間のパターンを分析する

評価全体でパターンを探す：
- 特定の評価タイプが一貫して難しい/簡単か？
- 一部の評価で高いばらつきがあり、他は安定しているか？
- 予想に反する驚きの結果はあるか？

### ステップ 4：メトリクスのパターンを分析する

time_seconds、tokens、tool_calls を確認する：
- スキルが実行時間を大幅に増加させているか？
- リソース使用量に高いばらつきがあるか？
- 集計を歪めている外れ値の実行はあるか？

### ステップ 5：ノートを生成する

フリーフォームの観察を文字列のリストとして記述する。各ノートは以下であるべき：
- 具体的な観察を述べる
- データに基づいている（推測ではない）
- 集計メトリクスでは見えないことをユーザーが理解するのに役立つ

例：
- "Assertion 'Output is a PDF file' passes 100% in both configurations - may not differentiate skill value"
- "Eval 3 shows high variance (50% +/- 40%) - run 2 had an unusual failure that may be flaky"
- "Without-skill runs consistently fail on table extraction expectations (0% pass rate)"
- "Skill adds 13s average execution time but improves pass rate by 50%"
- "Token usage is 80% higher with skill, primarily due to script output parsing"
- "All 3 without-skill runs for eval 1 produced empty output"

### ステップ 6：ノートを書き出す

ノートを `{output_path}` に文字列の JSON 配列として保存する：

```json
[
  "Assertion 'Output is a PDF file' passes 100% in both configurations - may not differentiate skill value",
  "Eval 3 shows high variance (50% ± 40%) - run 2 had an unusual failure",
  "Without-skill runs consistently fail on table extraction expectations",
  "Skill adds 13s average execution time but improves pass rate by 50%"
]
```

## ガイドライン

**すべきこと：**
- データで観察したことを報告する
- 言及する評価、期待値、実行を具体的に示す
- 集計メトリクスでは隠れてしまうパターンを記録する
- 数値の解釈に役立つコンテキストを提供する

**してはならないこと：**
- スキルの改善を提案する（それはベンチマークではなく、改善ステップのため）
- 主観的な品質判断をする（「出力が良かった/悪かった」）
- 証拠なしに原因を推測する
- run_summary の集計に既にある情報を繰り返す

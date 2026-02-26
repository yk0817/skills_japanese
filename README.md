> **注:** このリポジトリには、Anthropic が Claude 向けに実装したスキルが含まれています。Agent Skills 標準についての情報は [agentskills.io](http://agentskills.io) をご覧ください。

# Skills
Skills は、Claude が専門的なタスクのパフォーマンスを向上させるために動的に読み込む、手順・スクリプト・リソースをまとめたフォルダです。Skills を使えば、会社のブランドガイドラインに沿ったドキュメント作成、組織固有のワークフローによるデータ分析、個人タスクの自動化など、特定のタスクを再現可能な方法で Claude に教えることができます。

詳しくはこちらをご覧ください：
- [Skills とは？](https://support.claude.com/en/articles/12512176-what-are-skills)
- [Claude での Skills の使い方](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [カスタム Skills の作成方法](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Agent Skills で AI エージェントを実世界に対応させる](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# このリポジトリについて

このリポジトリには、Claude の Skills システムで何ができるかを示すスキルが含まれています。クリエイティブな用途（アート、音楽、デザイン）から技術的なタスク（Web アプリのテスト、MCP サーバー生成）、エンタープライズワークフロー（コミュニケーション、ブランディングなど）まで、幅広いスキルが揃っています。

各スキルは独立したフォルダに格納されており、Claude が使用する手順とメタデータを含む `SKILL.md` ファイルが入っています。これらのスキルを参考にして、独自のスキル作成のヒントを得たり、さまざまなパターンやアプローチを理解したりできます。

このリポジトリの多くのスキルはオープンソース（Apache 2.0）です。また、Claude の[ドキュメント作成機能](https://www.anthropic.com/news/create-files)を支える文書作成・編集スキルも [`skills/docx`](./skills/docx)、[`skills/pdf`](./skills/pdf)、[`skills/pptx`](./skills/pptx)、[`skills/xlsx`](./skills/xlsx) サブフォルダに収録しています。これらはソースコード公開型であり、オープンソースではありませんが、本番 AI アプリケーションで実際に使用されている高度なスキルのリファレンスとして開発者の皆様に共有したいと考えました。

## 免責事項

**これらのスキルは、デモンストレーションおよび教育目的でのみ提供されています。** これらの機能の一部は Claude で利用可能かもしれませんが、実際に Claude から得られる動作はここに示されたスキルとは異なる場合があります。これらのスキルはパターンと可能性を示すためのものです。重要なタスクに使用する前に、必ずご自身の環境で十分にテストしてください。

# スキルセット
- [./skills](./skills): Creative & Design、Development & Technical、Enterprise & Communication、Document Skills のスキル例
- [./spec](./spec): Agent Skills 仕様
- [./template](./template): スキルテンプレート

# Claude Code、Claude.ai、API での利用

## Claude Code
このリポジトリを Claude Code Plugin マーケットプレイスとして登録するには、Claude Code で以下のコマンドを実行してください：
```
/plugin marketplace add anthropics/skills
```

特定のスキルセットをインストールするには：
1. `Browse and install plugins` を選択
2. `anthropic-agent-skills` を選択
3. `document-skills` または `example-skills` を選択
4. `Install now` を選択

または、以下のコマンドで直接 Plugin をインストールすることもできます：
```
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

Plugin をインストールしたら、スキル名に言及するだけで使用できます。例えば、マーケットプレイスから `document-skills` Plugin をインストールした場合、Claude Code に「PDF スキルを使って `path/to/some-file.pdf` のフォームフィールドを抽出して」のようにリクエストできます。

## Claude.ai

これらのサンプルスキルは、Claude.ai の有料プランですべて利用可能です。

このリポジトリのスキルを使用したり、カスタムスキルをアップロードしたりするには、[Claude での Skills の使い方](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b)の手順に従ってください。

## Claude API

Anthropic のビルド済みスキルの利用やカスタムスキルのアップロードは、Claude API 経由で行えます。詳しくは [Skills API クイックスタート](https://docs.claude.com/en/api/skills-guide#creating-a-skill)をご覧ください。

# 基本的なスキルの作成

スキルの作成はシンプルです。YAML フロントマターと手順を含む `SKILL.md` ファイルが入ったフォルダを用意するだけです。このリポジトリの **template-skill** を出発点として使用できます：

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

フロントマターに必要なフィールドは2つだけです：
- `name` - スキルの一意な識別子（小文字、スペースの代わりにハイフンを使用）
- `description` - スキルの機能と使用するタイミングの完全な説明

フロントマター以下のマークダウンコンテンツには、Claude が従う手順、例、ガイドラインが含まれます。詳しくは[カスタム Skills の作成方法](https://support.claude.com/en/articles/12512198-creating-custom-skills)をご覧ください。

# パートナー Skills

Skills は、特定のソフトウェアの使い方を Claude に教えるのに最適な方法です。パートナーから優れたサンプルスキルが提供された場合、ここで紹介することがあります：

- **Notion** - [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)

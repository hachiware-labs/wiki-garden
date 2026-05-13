# Wiki Garden 要件定義

**Wiki Garden** は、Andrej Karpathy の LLM Wiki パターンにインスパイアされた、Markdown 知識ベース運用スキルである。

目的は、会話・資料・成果物から得た知見を、単なるチャット履歴やメモとして残すのではなく、**全体知識** と **プロジェクト固有知識** に分類し、Markdown Wiki として継続的に更新・洗練することである。

```text
raw sources / current work
  ↓
ingest
  ↓
global knowledge / project-local knowledge
  ↓
query
  ↓
lint / refine
  ↓
継続的に育つ Markdown 知識ベース
```

---

## 1. 目的

Wiki Garden は、Karpathy の LLM Wiki 原則を、実運用可能な Markdown 知識ベース管理スキルとして具象化する。

この Skill は、以下を実現する。

- raw source や現在作業中の会話から、再利用可能な知見を抽出する
- 知見を **global knowledge** と **project-local knowledge** に分類する
- 知見を session summary として保存せず、正規の Markdown ページへ反映する
- 既存知識を query し、作業に必要な文脈を取得する
- 知識ベースの矛盾・古さ・根拠不足を lint する
- 知識ベースを定期的に refine する

---

## 2. 基本思想

### 2.1 Karpathy LLM Wiki 準拠

Karpathy の基本操作である、

```text
ingest
query
lint
```

を中核にする。

本 Skill では、これに加えて、知識の継続的な整理・昇格・統合を行う操作として、

```text
refine
```

を追加する。

### 2.2 セッションは知識ではない

本 Skill は、CLI やエージェントとの対話セッションを原則として永続保存しない。

```text
セッション = 一時的な入力
知識       = global / project-local の正規ページ
```

セッションから価値ある知見が生まれた場合は、セッション要約として保存するのではなく、正規の知識ページへ反映する。

### 2.3 知識にはスコープがある

知識は最低限、次の2層に分類する。

```text
global knowledge
  複数プロジェクトで再利用できる概念、原則、方法論、比較、設計思想。

project-local knowledge
  特定プロジェクトだけで有効な背景、判断、制約、用語、成果物、未解決問い。
```

プロジェクト固有の判断を global knowledge に混ぜない。  
一般化できるものだけを global knowledge に昇格する。

### 2.4 Markdown-first, HTML-allowed

本 Skill は Markdown を標準形式とするが、Markdown だけに限定しない。

構造的な理解が重要な知識、たとえば図説、フロー、依存関係、アーキテクチャマップ、意思決定ツリー、時系列、比較マトリクス、コンセプトマップ、軽量なインタラクティブ説明は、静的 HTML として保存してよい。

HTML は一時的なプレビューではなく、Markdown と並ぶ正規の知識ページとして扱う。

ただし、`html/` のような専用バケットを標準にはしない。`concepts/`、`methods/`、`comparisons/`、`decisions/` などの既存フォルダがすでに知識構造を表しているため、HTML は関連する Markdown と同じ意味的フォルダに並べて置く。

例:

```text
global/
  concepts/
    llm-wiki.md
    llm-wiki-map.html
  methods/
    knowledge-refinement.md
    knowledge-refinement-flow.html

projects/
  wiki-garden/
    decisions/
      0001-core-operations.md
      0001-core-operations-map.html
    repository-structure.html
```

### 2.5 Locale-centered knowledge

Skill 本体の指示言語と、知識ベースの正規言語は分けて扱う。

Wiki Garden は `knowledge_locale` を持つ。正規の Markdown / HTML 知識ページ、index、log、decision、lesson、open question、query 回答、lint レポート、knowledge patch は、原則として `knowledge_locale` の言語で作成する。

外国語ソースを ingest する場合は、原資料を原文のまま保持または参照し、再利用可能な知識だけを `knowledge_locale` に蒸留して取り込む。

例:

```yaml
knowledge_locale: ja-JP
source_language_policy: preserve-original
ingest_language_policy: distill-to-knowledge-locale
```

用語ルール:

- 固有名詞、API 名、コード識別子、ファイルパス、コマンドは原則として翻訳しない
- 重要な専門用語は初出で原語を併記する
- 訳語が不安定なものは `glossary.md` または `open-questions.md` に残す
- 直訳よりも、知識として再利用しやすいロケール言語の説明を優先する

例:

```markdown
# 検索拡張生成（Retrieval-Augmented Generation, RAG）
```

---

## 3. 対象ユーザー

主な対象は以下。

- Codex / Claude Code / Cursor / OpenCode などで開発・調査を行うユーザー
- Markdown ベースで知識を育てたいユーザー
- Obsidian / VS Code / GitHub で知識ベースを扱いたいユーザー
- プロジェクトごとの暗黙知を残したい開発者
- 複数 AI エージェント環境で共通の知識運用をしたいユーザー

---

## 4. 想定利用環境

### 4.1 配布形態

GitHub リポジトリとして公開する。

例:

```text
github.com/hachiware-labs/wiki-garden
```

Vercel Labs の `npx skills` で取り込める構造にする。

例:

```bash
npx skills add hachiware-labs/wiki-garden
npx skills add hachiware-labs/wiki-garden -a claude-code -a codex
npx skills add https://github.com/hachiware-labs/wiki-garden
```

### 4.2 対象エージェント

必須対応:

- Claude Code
- OpenAI Codex

できれば対応:

- Cursor
- OpenCode
- Gemini CLI
- GitHub Copilot agent 系

MVP では **Agent Skills open standard / SKILL.md** に準拠した instruction-only Skill として実装し、各ツール固有の拡張は最小限にする。

---

## 5. リポジトリ構成要件

推奨構成:

```text
wiki-garden/
  README.md
  LICENSE
  AGENTS.md
  skills/
    wiki-garden/
      SKILL.md
      SCHEMA.md
      examples/
        basic-wiki/
        project-wiki/
      templates/
        global-index.md
        project-index.md
        project-context.md
        decision.md
        log.md
  .agents/
    skills/
      wiki-garden/
        SKILL.md
        SCHEMA.md
```

ただし、配布の単純さを優先するなら、まずは次でもよい。

```text
wiki-garden/
  README.md
  LICENSE
  .agents/
    skills/
      wiki-garden/
        SKILL.md
        SCHEMA.md
        templates/
```

Codex 互換性を考えると、`.agents/skills/wiki-garden/SKILL.md` は置いておく価値が高い。

---

## 6. Skill メタデータ要件

`SKILL.md` は次のような frontmatter を持つ。

```yaml
---
name: wiki-garden
description: Maintain a locale-centered, Karpathy-inspired Markdown and static HTML knowledge base. Use for configuring a shared or project-specific knowledge root or knowledge locale, ingesting sources, querying global or project-local knowledge, linting knowledge quality, refining canonical wiki pages, and creating diagram-heavy HTML knowledge pages.
---
```

説明文は、Codex や Claude Code が自動発火を判断しやすいように、用途を前半に明示する。

---

## 7. 知識ベース構造要件

Wiki Garden が前提とする知識ベースは、設定可能な knowledge root 配下に置く。

knowledge root の解決順は以下とする。

1. ユーザーが明示したパス
2. リポジトリルートの `wiki-garden.config.md`
3. 互換用の `knowledge-garden.config.md` または `knowledge-gardener.config.md`
4. 環境変数 `WIKI_GARDEN_CONFIG` が指すユーザー共通設定
5. 互換用の `KNOWLEDGE_GARDENER_CONFIG` が指すユーザー共通設定
6. `~/wiki-garden.config.md`
7. 互換用の `~/knowledge-garden.config.md` または `~/knowledge-gardener.config.md`
8. `AGENTS.md` などのプロジェクト指示や既存規約で指定されたパス
9. 既存の知識フォルダ
10. デフォルト値 `knowledge/`

デフォルトでは `.knowledge/` のようなドット付き隠しフォルダを作らない。Obsidian などのツールで見えにくくなる可能性があるためである。  
ただし、ユーザーが明示した場合や既存リポジトリが採用している場合は、`.knowledge/` もサポートする。

グローバルにインストールされた Skill は永続的な内部状態を持たないため、プロジェクトをまたいで使う共通知識 root はユーザー共通設定に保存する。標準の保存先は `~/wiki-garden.config.md` とする。`WIKI_GARDEN_CONFIG` が設定されている場合は、そのパスを優先する。

旧名称からの移行互換として、`knowledge-garden.config.md`、`knowledge-gardener.config.md`、`~/knowledge-garden.config.md`、`~/knowledge-gardener.config.md`、`KNOWLEDGE_GARDENER_CONFIG` は読み取り対象に含める。ただし新規作成は `wiki-garden.config.md` と `WIKI_GARDEN_CONFIG` を使う。

例:

```markdown
---
knowledge_root: ~/WikiGarden
scope: user
knowledge_locale: ja-JP
source_language_policy: preserve-original
ingest_language_policy: distill-to-knowledge-locale
---

# Wiki Garden Config

Use `~/WikiGarden/` as the shared Wiki Garden root.

Use `ja-JP` as the canonical knowledge locale.
```

プロジェクト単位で上書きしたい場合のみ、リポジトリルートの `wiki-garden.config.md` を使う。

```markdown
---
knowledge_root: docs/wiki
scope: project
knowledge_locale: ja-JP
---

# Wiki Garden Config

This project uses `docs/wiki/` as its Wiki Garden root.
This project uses `ja-JP` as its canonical knowledge locale.
```

標準構造は以下。

```text
knowledge/
  global/
    index.md
    log.md
    concepts/
      *.md
      *.html
    principles/
      *.md
      *.html
    methods/
      *.md
      *.html
    comparisons/
      *.md
      *.html
    glossary.md
    open-questions.md

  projects/
    <project-name>/
      PROJECT.md
      index.md
      log.md
      context.md
      glossary.md
      decisions/
        *.md
        *.html
      lessons.md
      open-questions.md
      artifacts.md
      *.html

  raw/
    sources/
```

Markdown と HTML は、同じ知識カテゴリ内に並べてよい。HTML は「別種の格納場所」ではなく、「同じ知識構造に属する別表現」として扱う。

### 7.1 global

全体に再利用可能な知識を置く。

```text
concepts/
  概念定義

principles/
  一般原則

methods/
  手法・運用方法

comparisons/
  比較・対比

glossary.md
  全体用語集

open-questions.md
  全体の未解決問い
```

### 7.2 projects

プロジェクト固有の知見を置く。

```text
PROJECT.md
  プロジェクトの目的、非目標、前提、境界

context.md
  現在の背景、設計文脈、制約

decisions/
  意思決定記録

lessons.md
  実際に分かったこと、失敗、注意点

open-questions.md
  プロジェクト固有の未解決問い

artifacts.md
  成果物の目録

log.md
  知識ベースへの変更履歴
```

### 7.3 raw

保存対象の原資料を置く。

```text
raw/sources/
  論文、記事、仕様書、資料、メモ、成果物など
```

CLI 対話セッションの生ログは、MVP では標準保存しない。  
必要な場合のみ、将来オプションとして `raw/sessions/` を検討する。

---

## 8. コア操作要件

外部に見せる操作は4つにする。

```text
ingest
query
lint
refine
```

加えて、グローバルインストールされた Skill が共通 knowledge root を保持するためのセットアップ操作として、`set_knowledge_path` をサポートする。`set_knowledge_path` は知識操作ではなく設定操作であり、デフォルトではユーザー共通設定 `~/wiki-garden.config.md` を作成または更新する。

エイリアス:

```text
set_knowledge_path
get_knowledge_path
set_path
set-root
configure root
```

さらに、knowledge locale を保持するための設定操作として、`set_knowledge_locale` をサポートする。
また、現在有効な設定を確認するため、`get_knowledge_path` と `get_knowledge_locale` をサポートする。

例:

```text
set_knowledge_locale ja-JP
get_knowledge_locale
set_knowledge_locale --project en-US
```

制約:

- Skill 自体にグローバル状態を持たせない
- 共通設定はユーザー共通設定ファイルに保存する
- プロジェクト固有の上書きは `set_knowledge_path --project <path>` または `set_project_knowledge_path <path>` でリポジトリ内に保存する
- `get_knowledge_path` と `get_knowledge_locale` は表示のみで、ファイルを作成・変更しない
- 既存知識の移動は自動で行わず、必要なら knowledge patch として提案する

---

### 8.1 ingest

#### 目的

指定ファイル、raw source、または現在セッション中に生まれた有用な知見を取り込み、正規の知識ページへ反映する。

#### 入力

- file path 任意
- project name 任意
- current session context

#### 動作

1. 入力がファイルなら内容を読む
2. knowledge root と knowledge locale を解決する
3. 入力が明示されない場合は現在セッションから有用な知見を抽出する
4. 原資料や citation は原文のまま保持する
5. 再利用可能な知識を knowledge locale に蒸留する
6. 知見候補を分類する
   - global knowledge
   - project-local context
   - decision
   - lesson
   - open question
   - artifact reference
   - HTML knowledge artifact candidate
   - glossary term
   - transient / discard
7. 既存ページを検索する
8. 既存ページへ knowledge locale で統合する
9. 必要な場合だけ新規ページを作る
10. 構造的な図説が有効な場合は、関連 Markdown と同じ意味的フォルダに HTML 知識ページを knowledge locale で作る
11. `index.md` を更新する
12. `log.md` に変更履歴を書く

#### 制約

- セッション要約を保存しない
- raw source を勝手に書き換えない
- 出典や根拠がある場合は残す
- 不確実なものは断定せず open question にする
- HTML を使う場合も、`index.md`、`log.md`、関連 Markdown から辿れるようにする
- 外国語ソースは原文を保持し、正規知識は knowledge locale で記述する

---

### 8.2 query

#### 目的

現在の問いに関連する知識を、global / project-local から取得して回答する。

#### 入力

- query 任意
- project name 任意

#### 動作

1. knowledge root と knowledge locale を解決する
2. project name がある場合、まず project-local knowledge を読む
3. global knowledge から関連概念・方法・比較を探す
4. `index.md` を優先して読み、必要に応じて本文へ掘る
5. 関連 HTML がある場合は、タイトル、メタ情報、本文、構造、関連リンクを読む
6. 回答に必要な知識だけを抽出する
7. 回答は、ユーザーが別言語を明示しない限り knowledge locale で行う
8. 矛盾、不確実性、古い可能性、翻訳依存の注意点があれば明示する
9. 回答中に新しい durable knowledge が生まれた場合、ingest 候補として knowledge locale で提示する

#### 制約

- 古い知識と現在の明示指示が矛盾する場合、現在の指示を優先する
- project-local な情報を global な一般論として扱わない
- 出典不明の断定を避ける

---

### 8.3 lint

#### 目的

知識ベースの異常、根拠不足、古さ、翻訳ゆれ、構造上の問題を検出する。

#### 検出対象

- 矛盾
- 根拠不足
- 古くなった主張
- 再確認が必要な claim
- 翻訳ゆれ
- 原語併記が必要な用語
- scope 違い
- project-local 知識の global 混入
- global 化できそうな project-local 知識
- 孤立ページ
- リンク切れ
- `index.md` 未登録ページ
- 重複ページ
- 大きすぎるページ
- 未分類の inbox 項目
- 孤立した HTML 知識ページ
- タイトル、スコープ、更新日、関連 Markdown、出典、アクセシビリティ情報が不足した HTML

#### 出力

- 問題一覧
- 影響範囲
- 修正提案
- refine で直せるもの / 人間判断が必要なものの区別
- knowledge locale での説明

#### 制約

- lint は原則として検出のみ
- 自動修正はしない
- 修正が必要な場合は refine の対象として提案する
- 外国語ソースに関する問題は、説明を knowledge locale で行い、必要最小限の原文を併記する

---

### 8.4 refine

#### 目的

知識ベースを洗練する。

#### 動作

- 重複ページを統合する
- 長すぎるページを分割する
- 曖昧な断定を open question に降格する
- knowledge locale に用語を統一する
- 必要な原語併記を glossary に追加する
- raw source を変更せず翻訳ゆれを修正する
- project-local lessons から global principles 候補を抽出する
- global に混入した project-local 情報を移動する
- backlinks を追加する
- `index.md` を整理する
- `log.md` を整える
- 古い情報を stale として明示する
- Markdown では読みにくい構造説明を、同じ意味的フォルダの HTML 知識ページとして作成する
- HTML 知識ページを関連 Markdown と相互リンクする

#### 制約

- 大きな構造変更は knowledge patch として提示してから行う
- 原資料を変更しない
- 判断が必要な昇格・統合はユーザー確認を推奨する

---

## 9. knowledge patch 要件

大きな変更を行う前に、Skill は **knowledge patch** を提示できること。

形式例:

```markdown
# Knowledge Patch

## Global updates

- Update `global/methods/llm-wiki-maintenance.md`
  - Add principle: sessions are transient inputs.
  - Add operation: refine.

## Project-local updates

- Update `projects/wiki-garden/context.md`
  - Add scope rule: project-local decisions must not be generalized automatically.

## Decisions

- Create `projects/wiki-garden/decisions/0001-core-operations.md`

## Open questions

- Should CLI session logs ever be saved as optional audit material?

## Discarded as transient

- Temporary wording preferences from this conversation.
```

MVP では、実ファイルとして `patches/` に保存しなくてもよい。  
エージェントが編集前に提示する差分案として扱う。

---

## 10. 初期テンプレート要件

Skill にはテンプレートを含める。

### 10.1 global/index.md

```markdown
# Global Knowledge Index

## Concepts

## Principles

## Methods

## Comparisons

## Glossary

## Open Questions
```

### 10.2 project/PROJECT.md

```markdown
# Project

## Purpose

## Non-goals

## Background

## Constraints

## Current focus
```

### 10.3 project/context.md

```markdown
# Project Context

## Current understanding

## Architecture / design context

## Important constraints

## Local terminology

## Notes
```

### 10.4 decision template

```markdown
---
type: decision
scope: project
status: proposed
date:
---

# Decision title

## Context

## Decision

## Alternatives

## Consequences

## Possible global lesson
```

### 10.5 log.md

```markdown
# Log

## YYYY-MM-DD

- Updated ...
- Added ...
- Moved ...
```

### 10.6 HTML artifact

HTML 知識ページは、関連 Markdown と同じ意味的フォルダに置く。

```html
<!doctype html>
<html lang="en" data-scope="project">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Knowledge Artifact Title</title>
</head>
<body>
  <main>
    <header>
      <h1>Knowledge Artifact Title</h1>
      <p>Scope: project | Last updated: YYYY-MM-DD</p>
      <p>Short purpose of this structural explanation.</p>
    </header>

    <section>
      <h2>Overview</h2>
      <p>Describe what this artifact explains and how to read it.</p>
    </section>

    <section>
      <h2>Related Knowledge</h2>
      <ul>
        <li><a href="index.md">Nearest index</a></li>
      </ul>
    </section>

    <section>
      <h2>Sources</h2>
      <ul>
        <li>Add source references when available.</li>
      </ul>
    </section>
  </main>
</body>
</html>
```

---

## 11. インストール要件

### 11.1 Vercel Labs `npx skills`

標準インストール方法として、以下を README に記載する。

```bash
npx skills add hachiware-labs/wiki-garden
```

特定エージェント向け:

```bash
npx skills add hachiware-labs/wiki-garden -a claude-code -a codex
```

グローバルインストール:

```bash
npx skills add hachiware-labs/wiki-garden -g
```

一覧確認:

```bash
npx skills list
```

### 11.2 Codex

Codex 向けには、以下をサポートする。

```text
.agents/skills/wiki-garden/SKILL.md
```

Codex では `$` による Skill mention、`/skills` からの明示呼び出し、description による暗黙呼び出しが可能である想定。

### 11.3 Claude Code

Claude Code 向けには、次を想定する。

```text
~/.claude/skills/wiki-garden/SKILL.md
```

または `npx skills` による配置。

---

## 12. README 要件

README には以下を含める。

- Skill の目的
- Karpathy LLM Wiki からの影響
- セッションを保存しない方針
- global / project-local の説明
- Markdown と HTML 知識ページの使い分け
- knowledge locale の説明
- ingest / query / lint / refine の説明
- 知識ベースの推奨ディレクトリ構成
- `npx skills` でのインストール方法
- Claude Code / Codex での利用方法
- 使用例
- 設計上の非目標

---

## 13. 非目標

MVP では以下をやらない。

- 専用 DB の実装
- ベクトル検索の実装
- セッションログの自動保存
- Web UI
- MCP サーバー
- GitHub Actions による自動知識更新
- 複雑な CLI ツール本体
- raw source の自動ダウンロード

Skill としてまず成立させる。

---

## 14. 将来拡張

将来的には以下を検討する。

- CLI wrapper: `kg ingest` / `kg query` / `kg lint` / `kg refine`
- Obsidian 向けテンプレート
- GitHub Actions での定期 lint
- project-local knowledge の global 昇格候補抽出
- frontmatter 標準化
- source citation 管理
- optional `raw/sessions` 保存
- search backend 連携
- MCP server 化
- skills.sh への登録

---

## 15. 成功条件

MVP の成功条件は以下。

1. GitHub リポジトリから `npx skills add` で導入できる
2. Claude Code / Codex の両方で Skill として認識される
3. `ingest` / `query` / `lint` / `refine` の4操作が説明されている
4. セッションを保存せず、知見だけを正規ページへ反映する方針が明確
5. global knowledge と project-local knowledge の分離が明確
6. Markdown と HTML を同じ知識構造内に並べて扱える
7. 既存 Markdown Wiki に対して小さな差分で更新できる
8. README を読めば使い始められる

---

## 16. 最小 MVP

最初に作るべきファイルは、これで十分。

```text
wiki-garden/
  README.md
  LICENSE
  .agents/
    skills/
      wiki-garden/
        SKILL.md
        SCHEMA.md
        templates/
          global-index.md
          global-log.md
          project.md
          project-index.md
          project-context.md
          decision.md
          lesson.md
          html-artifact.html
          config.md
```

MVP では実行スクリプトなしでよい。  
Skill 本文だけで、Claude Code / Codex / Vercel `npx skills` の流れに乗せるのが最短である。

---

## 17. 初期 README 冒頭案

```markdown
# Wiki Garden

Wiki Garden is a Karpathy-inspired agent skill for maintaining a persistent Markdown knowledge base.

It helps AI coding agents ingest sources, query existing knowledge, lint knowledge quality, and refine canonical wiki pages.

The skill separates global knowledge from project-local knowledge.

It does not archive conversations as knowledge. Instead, it extracts durable information from the current work session or specified files and applies it to canonical Markdown pages.
```

---

## 18. 初期タグライン

```text
Do not preserve conversations. Preserve distilled knowledge.
```

または、

```text
A Karpathy-inspired skill for maintaining Markdown knowledge.
```



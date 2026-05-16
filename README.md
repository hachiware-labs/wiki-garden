# Wiki Garden

Wiki Garden is a Karpathy-inspired agent skill for maintaining a persistent knowledge base.

It helps AI coding agents ingest sources, query existing knowledge, lint knowledge quality, and refine canonical wiki pages. The skill separates global knowledge from project-local knowledge.

Do not preserve conversations. Preserve distilled knowledge.

## What It Does

Wiki Garden turns useful information from source files, current work, and project decisions into a durable knowledge base.

The knowledge base is Markdown-first, but not Markdown-only. Use Markdown for source summaries, prose knowledge, decisions, lessons, indexes, and open questions. Use static HTML for structural explanations that benefit from diagrams, timelines, maps, matrices, or lightweight interaction.

## Core Operations

- `ingest`: Preserve or read raw sources, create one-source summary pages, and merge cross-source knowledge into canonical pages.
- `query`: Read project-local and global knowledge to answer the current question.
- `lint`: Detect contradictions, stale claims, missing sources, mistranslations, scope leaks, orphan pages, and broken links.
- `refine`: Improve structure by merging, splitting, moving, indexing, or converting visual knowledge into HTML artifacts.

`set_knowledge_path` is a setup action for global skill installs. By default, it writes a user global knowledge root to `~/wiki-garden.config.md` so every project can use the same knowledge base.

`set_knowledge_locale` configures the canonical language for knowledge pages. Raw sources can stay in their original language, while distilled knowledge is written in the configured locale.

## Methods

### English

| Method | Purpose | Example |
| --- | --- | --- |
| `set_knowledge_path` | Set the shared knowledge root used across projects. Writes `~/wiki-garden.config.md` by default. | `Use $wiki-garden set_knowledge_path ~/WikiGarden` |
| `get_knowledge_path` | Show the currently effective knowledge root and which config source selected it. Does not modify files. | `Use $wiki-garden get_knowledge_path` |
| `set_knowledge_path --project` | Set a knowledge root override for the current repository. Writes `wiki-garden.config.md` in the repository root. | `Use $wiki-garden set_knowledge_path --project docs/wiki` |
| `set_knowledge_locale` | Set the canonical language for Markdown and HTML knowledge pages. Foreign-language sources are distilled into this locale. | `Use $wiki-garden set_knowledge_locale ja-JP` |
| `get_knowledge_locale` | Show the currently effective knowledge locale and which config source selected it. Does not modify files. | `Use $wiki-garden get_knowledge_locale` |
| `set_knowledge_locale --project` | Override the knowledge locale for the current repository. | `Use $wiki-garden set_knowledge_locale --project en-US` |
| `ingest` | Preserve or read raw source material, create a one-source summary page under `sources/`, and merge cross-source knowledge into canonical Markdown or HTML pages. | `Use $wiki-garden to ingest docs/api-notes.md into the checkout-redesign project.` |
| `query` | Read global and project-local knowledge to answer the current question. Related HTML knowledge pages are included. | `Use $wiki-garden to query what we know about retrieval pipeline tradeoffs.` |
| `lint` | Detect contradictions, stale claims, weak sources, mistranslations, scope leaks, broken links, orphan pages, and orphan HTML artifacts. | `Use $wiki-garden to lint knowledge/ for stale claims.` |
| `refine` | Improve the knowledge base by merging, splitting, moving, indexing, cross-linking, or converting structural explanations into HTML artifacts. | `Use $wiki-garden to refine the checkout-redesign project knowledge.` |

Aliases for `set_knowledge_path`: `set_path`, `set-root`, `configure root`.

### 日本語

| メソッド | 目的 | 例 |
| --- | --- | --- |
| `set_knowledge_path` | 複数プロジェクトで共有する knowledge root を設定する。デフォルトでは `~/wiki-garden.config.md` に保存する。 | `Use $wiki-garden set_knowledge_path ~/WikiGarden` |
| `get_knowledge_path` | 現在有効な knowledge root と、それを選んだ設定元を表示する。ファイルは変更しない。 | `Use $wiki-garden get_knowledge_path` |
| `set_knowledge_path --project` | 現在のリポジトリだけで使う knowledge root を設定する。リポジトリルートの `wiki-garden.config.md` に保存する。 | `Use $wiki-garden set_knowledge_path --project docs/wiki` |
| `set_knowledge_locale` | Markdown / HTML の正規知識ページで使う中心言語を設定する。外国語ソースはこのロケールへ蒸留して取り込む。 | `Use $wiki-garden set_knowledge_locale ja-JP` |
| `get_knowledge_locale` | 現在有効な knowledge locale と、それを選んだ設定元を表示する。ファイルは変更しない。 | `Use $wiki-garden get_knowledge_locale` |
| `set_knowledge_locale --project` | 現在のリポジトリだけで使う knowledge locale を設定する。 | `Use $wiki-garden set_knowledge_locale --project en-US` |
| `ingest` | raw source を保持または参照し、`sources/` に 1 ソース 1 ページの summary を作り、横断知識を Markdown または HTML の正規ページへ統合する。 | `Use $wiki-garden to ingest docs/api-notes.md into the checkout-redesign project.` |
| `query` | global knowledge と project-local knowledge を読み、ロケール言語で現在の問いに必要な文脈を取得する。関連 HTML 知識ページも対象にする。 | `Use $wiki-garden to query what we know about retrieval pipeline tradeoffs.` |
| `lint` | 矛盾、古さ、根拠不足、翻訳ゆれ、scope 混入、リンク切れ、孤立ページ、孤立 HTML を検出する。 | `Use $wiki-garden to lint knowledge/ for stale claims.` |
| `refine` | 重複統合、分割、移動、用語統一、index/log 整理、相互リンク追加、構造説明の HTML 化などで知識ベースを洗練する。 | `Use $wiki-garden to refine the checkout-redesign project knowledge.` |

`set_knowledge_path` のエイリアス: `set_path`, `set-root`, `configure root`。

## Knowledge Base Layout

Wiki Garden uses a configurable knowledge root. If a path is specified, use that. If a project has an override, use it. Otherwise use the user global config. If nothing is configured, the default root is `knowledge/`.

The default is not dot-prefixed so the folder remains visible in tools such as Obsidian. `.knowledge/` is still usable when explicitly configured or already present.

To set one shared knowledge root for all projects:

```text
Use $wiki-garden set_knowledge_path ~/WikiGarden
```

This creates or updates `~/wiki-garden.config.md`:

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

For compatibility, existing `knowledge-garden.config.md`, `knowledge-gardener.config.md`, `~/knowledge-garden.config.md`, `~/knowledge-gardener.config.md`, and `KNOWLEDGE_GARDENER_CONFIG` are read as legacy config sources. New configuration should use `wiki-garden.config.md` or `WIKI_GARDEN_CONFIG`.

To override the root for one repository:

```text
Use $wiki-garden set_knowledge_path --project docs/wiki
```

This creates or updates `wiki-garden.config.md` in the repository:

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

## Knowledge Locale

Wiki Garden manages canonical knowledge in the configured locale. For example, if `knowledge_locale` is `ja-JP`, then `ingest`, `query`, `lint`, and `refine` should write knowledge pages, reports, patches, and answers primarily in Japanese.

Foreign-language sources should not be overwritten. Keep raw sources and citations in their original language, then distill reusable knowledge into the knowledge locale. Important original terms should be preserved on first use, for example `検索拡張生成（Retrieval-Augmented Generation, RAG）`.

ロケール言語は、知識ベースの正規言語です。`knowledge_locale: ja-JP` の場合、`ingest` / `query` / `lint` / `refine` は日本語を中心に知識ページ、検出結果、knowledge patch、回答を作成します。

外国語ソースは原文のまま保持し、知識として再利用する内容だけをロケール言語へ蒸留します。固有名詞、API 名、コード識別子、ファイルパス、コマンドは原則として翻訳しません。

```text
knowledge/
  raw/
    sources/
      papers/
      articles/
      web/
      docs/
  sources/
    index.md
    papers/
    articles/
    web/
    docs/
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
```

`raw/` is managed inside the selected knowledge root. It is the immutable source layer: humans add papers, articles, web captures, documents, or other source material there, and agents read it without rewriting it.

`sources/` contains one-source summary pages created by ingest. These summaries connect raw material to reusable knowledge by linking back to raw sources and forward to concepts, methods, comparisons, decisions, project context, or open questions.

For HTML web pages, keep at least the original URL, capture date, title, and source format. Prefer Markdown text extraction for ordinary articles; add an HTML snapshot only when exact layout or later verification matters.

## HTML Artifacts

HTML artifacts are first-class knowledge pages when visual structure is the knowledge. Use them for:

- architecture maps
- dependency diagrams
- decision trees
- concept maps
- timelines
- comparison matrices
- interactive explainers

HTML artifacts should be self-contained static files by default and placed beside related Markdown in the same semantic folder. Do not create a separate `html/` bucket unless the user chooses that convention. Link HTML from `index.md`, record it in `log.md`, and list project-specific HTML in `artifacts.md`.

## Installation

Install with Vercel Labs `skills`:

```bash
npx skills add hachiware-labs/wiki-garden
```

Install for specific agents:

```bash
npx skills add hachiware-labs/wiki-garden -a claude-code -a codex
```

Install globally:

```bash
npx skills add hachiware-labs/wiki-garden -g
```

List installed skills:

```bash
npx skills list
```

## Repository Entrypoints

The canonical skill source in this repository is:

```text
.agents/skills/wiki-garden/
```

The top-level `skills/wiki-garden/` directory mirrors the same skill package for distribution tools and users that expect the Agent Skills layout at `skills/<skill-name>/`.

When updating the skill, edit `.agents/skills/wiki-garden/` first, then refresh `skills/wiki-garden/` so both entrypoints stay aligned.

## Codex

This repository includes the Codex-compatible skill path:

```text
.agents/skills/wiki-garden/SKILL.md
```

Use the skill by mentioning `$wiki-garden`, selecting it from `/skills`, or relying on the description trigger when asking to ingest, query, lint, or refine knowledge.

## Claude Code

The skill is intended to work as an Agent Skills open standard `SKILL.md` package. With `npx skills`, it can be installed into Claude Code compatible skill locations.

## Examples

Ingest a source into project knowledge:

```text
Use $wiki-garden to ingest docs/api-notes.md into the checkout-redesign project.
```

Set the shared knowledge root:

```text
Use $wiki-garden set_knowledge_path ~/WikiGarden
```

Show the effective knowledge root:

```text
Use $wiki-garden get_knowledge_path
```

Set the shared knowledge locale:

```text
Use $wiki-garden set_knowledge_locale ja-JP
```

Show the effective knowledge locale:

```text
Use $wiki-garden get_knowledge_locale
```

Set a project-specific override:

```text
Use $wiki-garden set_knowledge_path --project notes
```

Query existing knowledge:

```text
Use $wiki-garden to query what we know about retrieval pipeline tradeoffs.
```

Create an HTML artifact when Markdown is not enough:

```text
Use $wiki-garden to turn this architecture explanation into a project-local HTML dependency map.
```

Lint a knowledge base:

```text
Use $wiki-garden to lint knowledge/ for scope leaks, orphan HTML artifacts, stale claims, and translation drift.
```

## Non-goals

The MVP does not implement a database, vector search, automatic session logging, web UI, MCP server, GitHub Actions automation, or raw source downloading. It is an instruction-only skill for maintaining Markdown and static HTML knowledge.

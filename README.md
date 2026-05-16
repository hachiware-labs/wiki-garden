# Wiki Garden

[Japanese README](README_ja.md)

Wiki Garden is a Karpathy-inspired agent skill for maintaining a persistent Markdown-first knowledge base.

It helps AI coding agents ingest sources and useful conversation insights, query existing knowledge, lint knowledge quality, and refine canonical wiki pages. The skill separates reusable global knowledge from project-local knowledge.

Do not preserve conversations. Preserve distilled knowledge.

## Why Wiki Garden

Andrej Karpathy has described the idea of an LLM-maintained wiki: instead of treating every chat as isolated context, an agent can grow a durable wiki from sources, questions, and work sessions. The useful unit is not the transcript. The useful unit is the refined knowledge that can be read, checked, linked, and reused later.

Wiki Garden applies that pattern to coding and research agents:

- raw sources stay available for verification
- one-source summaries bridge raw material and reusable knowledge
- concept, method, comparison, and project pages accumulate cross-source understanding
- agent conversations can contribute durable insights without becoming chat archives
- lint and refine keep the wiki coherent as it grows

This is especially useful for coding agents. A work session often creates decisions, constraints, design lessons, debugging knowledge, naming conventions, architecture notes, or open questions that are easy to lose if they stay only in chat history. Wiki Garden lets an agent distill those insights into the right canonical page.

## Karpathy's LLM Wiki Pattern

Karpathy's `llm-wiki` gist describes a pattern, not a finished product. The core shift is from query-time reconstruction to ingest-time compilation. In a typical RAG workflow, the system retrieves raw chunks when a question is asked and synthesizes an answer from those chunks each time. In the LLM Wiki pattern, the LLM reads curated sources ahead of time and incrementally compiles them into a persistent, interlinked Markdown wiki. Later questions read the compiled wiki first, going back to raw sources only when needed.

The original pattern has three layers:

- `raw` sources: curated source documents such as articles, papers, images, and data files. These are immutable and remain the source of truth.
- `wiki`: the LLM-maintained Markdown layer. It contains source summaries, concept pages, entity pages, comparisons, overviews, syntheses, indexes, and logs.
- `schema`: an operating contract such as `CLAUDE.md` or `AGENTS.md` that tells the agent how the wiki is organized and how to ingest, query, and maintain it.

It also has three primary operations:

- `ingest`: add a source to `raw`, have the LLM read it, discuss or identify key takeaways, write a summary page, update relevant concept/entity pages, update `index.md`, and append to `log.md`. Karpathy notes that a single source can touch many wiki pages, so supervised one-at-a-time ingest is often useful.
- `query`: ask questions against the wiki. The agent starts from the index, reads relevant pages, synthesizes an answer with citations, and can file valuable answers back into the wiki as new pages.
- `lint`: periodically health-check the wiki for contradictions, stale claims, orphan pages, missing concept pages, missing cross-references, and data gaps that deserve new source collection.

Two files are especially important in the original pattern:

- `index.md`: content-oriented navigation. It lists pages with short summaries and helps the agent decide what to read.
- `log.md`: chronological operational history. It records ingests, queries, and lint passes so both the user and the agent can see how the wiki evolved.

The human role is still central. The human curates source material, asks useful questions, reviews important summaries, and guides what should be emphasized. The LLM does the repetitive maintenance work: summarizing, linking, cross-referencing, updating, and checking consistency.

Wiki Garden follows that pattern, but adapts it for agent skills and coding work. It keeps `raw/` inside the configured knowledge root, adds explicit `sources/` summary pages, separates global and project-local knowledge, supports a configured `knowledge_locale`, and treats useful coding-agent conversation insights as ingest candidates. That last point is an extension: conversation ingest should preserve durable knowledge, not the conversation transcript.

## What It Does

Wiki Garden turns useful information from source files, web pages, papers, current work, and agent conversations into a durable knowledge base.

The knowledge base is Markdown-first, but not Markdown-only. Use Markdown for source summaries, prose knowledge, decisions, lessons, indexes, and open questions. Use static HTML for structural explanations that benefit from diagrams, timelines, maps, matrices, or lightweight interaction.

## Core Operations

- `ingest`: Preserve or read raw sources, create one-source summary pages, and merge cross-source knowledge into canonical pages. It can also ingest durable insights from an agent conversation.
- `query`: Read source summaries, project-local knowledge, and global knowledge to answer the current question.
- `lint`: Detect contradictions, stale claims, missing sources, mistranslations, scope leaks, orphan pages, and broken links.
- `refine`: Improve structure by merging, splitting, moving, indexing, cross-linking, or converting visual knowledge into HTML artifacts.

`set_knowledge_path` is a setup action for global skill installs. By default, it writes a user global knowledge root to `~/wiki-garden.config.md` so every project can use the same knowledge base.

`set_knowledge_locale` configures the canonical language for knowledge pages. Raw sources can stay in their original language, while distilled knowledge is written in the configured locale.

## Conversation Ingest

Wiki Garden can ingest insights from an AI coding-agent conversation, but it does not save the conversation itself.

For conversation ingest, the agent should extract only durable knowledge, such as:

- project constraints and assumptions
- decisions and their consequences
- debugging findings
- implementation lessons
- architecture or API notes
- local terminology
- reusable methods or principles
- open questions that should be tracked

Conversation ingest normally writes directly to canonical knowledge pages:

```text
current agent conversation
  ↓ ingest durable insight only
global/ or projects/<project>/
```

It should not create a transcript page, a chat archive, or a generic session summary. If the insight is only useful for the current moment, discard it. If it is reusable, put it where future work will look for it: `context.md`, `decisions/`, `lessons.md`, `open-questions.md`, `global/concepts/`, `global/methods/`, or another stable topic page.

External-source ingest has a different flow:

```text
raw/sources/
  ↓
sources/<type>/
  ↓
global/ or projects/<project>/
```

Use this source-summary flow for papers, articles, web pages, docs, and other durable source material. Use the conversation-insight flow for knowledge created during agent collaboration.

## Methods

| Method | Purpose | Example |
| --- | --- | --- |
| `set_knowledge_path` | Set the shared knowledge root used across projects. Writes `~/wiki-garden.config.md` by default. | `Use $wiki-garden set_knowledge_path ~/WikiGarden` |
| `get_knowledge_path` | Show the currently effective knowledge root and which config source selected it. Does not modify files. | `Use $wiki-garden get_knowledge_path` |
| `set_knowledge_path --project` | Set a knowledge root override for the current repository. Writes `wiki-garden.config.md` in the repository root. | `Use $wiki-garden set_knowledge_path --project docs/wiki` |
| `set_knowledge_locale` | Set the canonical language for Markdown and HTML knowledge pages. Foreign-language sources are distilled into this locale. | `Use $wiki-garden set_knowledge_locale ja-JP` |
| `get_knowledge_locale` | Show the currently effective knowledge locale and which config source selected it. Does not modify files. | `Use $wiki-garden get_knowledge_locale` |
| `set_knowledge_locale --project` | Override the knowledge locale for the current repository. | `Use $wiki-garden set_knowledge_locale --project en-US` |
| `ingest` | Preserve or read raw source material, create a source summary when appropriate, and merge durable knowledge into canonical Markdown or HTML pages. | `Use $wiki-garden to ingest docs/api-notes.md into the checkout-redesign project.` |
| `query` | Read source summaries, global knowledge, and project-local knowledge to answer the current question. Related HTML knowledge pages are included. | `Use $wiki-garden to query what we know about retrieval pipeline tradeoffs.` |
| `lint` | Detect contradictions, stale claims, weak sources, mistranslations, scope leaks, broken links, orphan pages, and orphan HTML artifacts. | `Use $wiki-garden to lint knowledge/ for stale claims.` |
| `refine` | Improve the knowledge base by merging, splitting, moving, indexing, cross-linking, or converting structural explanations into HTML artifacts. | `Use $wiki-garden to refine the checkout-redesign project knowledge.` |

Aliases for `set_knowledge_path`: `set_path`, `set-root`, `configure root`.

## Knowledge Base Layout

Wiki Garden uses a configurable knowledge root. If a path is specified, use that. If a project has an override, use it. Otherwise use the user global config. If nothing is configured, the default root is `knowledge/`.

The default is not dot-prefixed so the folder remains visible in tools such as Obsidian. `.knowledge/` is still usable when explicitly configured or already present.

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

## Knowledge Locale

Wiki Garden manages canonical knowledge in the configured locale. For example, if `knowledge_locale` is `ja-JP`, then `ingest`, `query`, `lint`, and `refine` should write knowledge pages, reports, patches, and answers primarily in Japanese.

Foreign-language sources should not be overwritten. Keep raw sources and citations in their original language, then distill reusable knowledge into the knowledge locale. Important original terms should be preserved on first use, for example `検索拡張生成（Retrieval-Augmented Generation, RAG）`.

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

## Examples

Ingest a source into project knowledge:

```text
Use $wiki-garden to ingest docs/api-notes.md into the checkout-redesign project.
```

Ingest durable knowledge from the current coding-agent conversation:

```text
Use $wiki-garden to ingest the durable lessons from this session into the wiki-garden project.
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

## References

- Andrej Karpathy, [`llm-wiki` gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- DAIR.AI Academy, [LLM Knowledge Bases](https://academy.dair.ai/blog/llm-knowledge-bases-karpathy)
- Denser.ai, [From RAG to LLM Wiki](https://denser.ai/blog/llm-wiki-karpathy-knowledge-base/)

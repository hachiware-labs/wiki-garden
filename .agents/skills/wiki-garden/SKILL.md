---
name: wiki-garden
description: Maintain a locale-centered, Karpathy-inspired Markdown and static HTML knowledge base. Use when configuring a shared or project-specific knowledge root or knowledge locale, ingesting sources or session insights into canonical global or project-local knowledge, querying existing knowledge, linting knowledge quality including source checks, refining wiki pages, or creating diagram-heavy HTML knowledge artifacts.
---

# Wiki Garden

Use this skill to keep a durable knowledge base from current work, source files, and project decisions. Preserve distilled knowledge, not chat transcripts.

Wiki Garden follows Karpathy's LLM Wiki pattern: raw sources are immutable source-of-truth inputs, the wiki is the maintained Markdown knowledge layer, and the skill instructions act as the operating schema for ingest, query, lint, and refinement. The goal is ingest-time knowledge compilation, not repeated query-time reconstruction from raw chunks.

The knowledge base is Markdown-first. Use static HTML as a first-class knowledge artifact when the knowledge is structural or visual enough that Markdown would obscure it: diagrams, timelines, decision trees, architecture maps, concept maps, matrices, dependency graphs, or interactive explanations.

Canonical knowledge is locale-centered. Write and maintain normal knowledge pages in the configured `knowledge_locale`, even when sources are in another language. Preserve original source language in raw sources, citations, proper nouns, code identifiers, API names, and important first-use terms.

## Standard Structure

Use a configurable knowledge root. If the user gives a path, use it. If project or user config defines a knowledge root, follow that. If nothing is configured, default to `knowledge/`.

Do not create a dot-prefixed hidden root such as `.knowledge/` unless the user explicitly asks for it; hidden folders may be awkward in tools such as Obsidian.

Prefer this layout under the selected root:

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

Raw sources live inside the selected knowledge root. Treat `raw/` as the immutable source layer: humans add or capture source material there, and agents read it but do not rewrite it during ingest, query, lint, or refine.

Use `sources/` for source summary pages. A source summary is a canonical Markdown page for one raw source, such as one paper, one article, one web page capture, one specification, or one document. It should link back to the raw source when available and forward to related concept, method, comparison, decision, or project pages.

Agent conversation insights are not raw sources by default. When ingesting from a coding-agent session, extract only durable knowledge and write it to the appropriate canonical pages. Do not create transcript pages, generic session summaries, or raw session logs unless the user explicitly asks for that archival behavior.

Global knowledge is reusable across projects. Project-local knowledge is only true for a named project unless explicitly generalized later.

## Format Choice

Use Markdown for prose, definitions, decisions, lessons, source notes, indexes, and open questions.

Use static HTML when visual structure is the knowledge:

- flowcharts, state machines, process maps
- architecture or dependency diagrams
- causal maps, concept maps, taxonomies
- timelines and roadmaps
- comparison matrices that need layout, grouping, or highlighting
- interactive explainers that reveal layers or relationships

Treat HTML artifacts as canonical knowledge pages, not previews. Place each HTML file beside the related Markdown or in the same semantic folder. Do not create a separate `html/` folder by default because the existing folders already express the knowledge structure. Link HTML from the nearest `index.md`, record it in `log.md`, and add project HTML files to `artifacts.md`.

Keep HTML self-contained by default. Avoid remote runtime dependencies unless the source or project already requires them. Include accessible text, a visible title, update date, scope, sources when available, and links back to related Markdown pages.

## Knowledge Root

Resolve the knowledge root in this order:

1. A path explicitly provided by the user for the current task.
2. `wiki-garden.config.md` at the repository root.
3. Legacy `knowledge-garden.config.md` or `knowledge-gardener.config.md` at the repository root, if present.
4. User global config at the path named by `WIKI_GARDEN_CONFIG`, if set.
5. Legacy user global config at the path named by `KNOWLEDGE_GARDENER_CONFIG`, if set.
6. User global config at `~/wiki-garden.config.md`.
7. Legacy user global config at `~/knowledge-garden.config.md` or `~/knowledge-gardener.config.md`, if present.
8. A project instruction such as `AGENTS.md` or another repository convention that names the knowledge folder.
9. An existing knowledge folder in the repository, preferring visible names such as `knowledge/`, `docs/knowledge/`, or `wiki/`.
10. The default `knowledge/`.

When creating a new root, create `knowledge/` by default. Treat `.knowledge/` as a supported legacy or user-chosen location, not the default.

### set_knowledge_path

Use `set_knowledge_path` as the canonical setup action for configuring the shared user knowledge root used across projects. Accept `set_path`, `set-root`, or `configure root` as aliases.

When the user asks to set the path:

1. Create or update the user global config at `~/wiki-garden.config.md`, unless `WIKI_GARDEN_CONFIG` points elsewhere.
2. Store the selected path as `knowledge_root`.
3. Store an absolute path or a `~`-anchored path so other projects can resolve the same location.
4. If the user gives a relative path, resolve it against the current working directory before saving.
5. Create the root directory and starter structure only if the user asks to initialize it.
6. Do not move existing knowledge automatically; propose a knowledge patch if migration is needed.

Use this user global config format:

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

For a project-specific override, use `set_knowledge_path --project <path>` or `set_project_knowledge_path <path>`:

1. Create or update `wiki-garden.config.md` at the repository root.
2. Store the selected path as `knowledge_root`.
3. Keep project-specific paths relative to the repository root unless the user explicitly needs an absolute path.
4. Create the root directory and starter structure only if the user asks to initialize it.
5. Do not move existing knowledge automatically; propose a knowledge patch if migration is needed.

Use this project config format:

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

### get_knowledge_path

Use `get_knowledge_path` to report the currently effective knowledge root.

When the user asks for the current path:

1. Resolve the knowledge root using the standard resolution order.
2. Report the effective root path.
3. Report which source selected it: explicit request, project config, user config, legacy config, project convention, existing folder, or default.
4. Report the config file path when one was used.
5. Include the effective `knowledge_locale` if it can be resolved cheaply.
6. Do not create or modify files.

Example response shape:

```text
Knowledge root: ~/WikiGarden
Source: user config
Config: ~/wiki-garden.config.md
Knowledge locale: ja-JP
```

## Knowledge Locale

Resolve the knowledge locale in this order:

1. A language explicitly requested by the user for the current task.
2. `knowledge_locale` in project config.
3. `knowledge_locale` in user global config.
4. The user's current conversation language or environment locale.
5. English only when no locale can be inferred.

Use the knowledge locale for:

- Markdown and HTML knowledge pages
- indexes, logs, decisions, lessons, open questions, and glossary entries
- query answers and lint reports
- knowledge patches and refine summaries

When ingesting foreign-language sources:

- keep raw sources in the original language
- distill canonical knowledge into the knowledge locale
- include important original terms on first use, for example `検索拡張生成（Retrieval-Augmented Generation, RAG）`
- do not translate proper nouns, code identifiers, API names, file paths, commands, model names, or quoted source titles unless there is a standard localized name
- put uncertain translations in `glossary.md` or `open-questions.md`
- avoid literal translation when a clearer localized explanation preserves the source meaning better

### set_knowledge_locale

Use `set_knowledge_locale <locale>` to configure the shared user knowledge locale, for example `set_knowledge_locale ja-JP`.

By default, update the user global config at `~/wiki-garden.config.md`, unless `WIKI_GARDEN_CONFIG` points elsewhere. For a project-specific override, use `set_knowledge_locale --project <locale>`.

### get_knowledge_locale

Use `get_knowledge_locale` to report the currently effective knowledge locale.

When the user asks for the current locale:

1. Resolve the knowledge locale using the standard resolution order.
2. Report the effective locale.
3. Report which source selected it: explicit request, project config, user config, conversation/environment locale, or fallback.
4. Report the config file path when one was used.
5. Do not create or modify files.

## Operations

### ingest

When ingesting a file, raw source, or useful session insight:

1. Resolve the knowledge root.
2. Resolve the knowledge locale.
3. Determine whether the input is an external source or a conversation insight.
4. For an external source, ensure raw source material belongs under `<knowledge-root>/raw/sources/`. If the user provided a file outside the knowledge root, read it as the source and, when asked to preserve it, copy or capture the raw material under `raw/sources/` before creating summaries. Do not rewrite existing raw files.
5. For web sources, preserve at least the source URL, capture date, and source title in raw metadata. Prefer a Markdown text extraction for ordinary articles and add an HTML snapshot only when fidelity, layout, or later verification matters.
6. For a conversation insight, extract only durable claims, decisions, lessons, constraints, terminology, methods, or open questions from the current agent session. Do not create a source summary, transcript page, chat archive, or raw session log by default.
7. Read the input, or extract durable claims from the current session if no file is given.
8. Preserve source language for raw sources and citations.
9. Create or update a one-source summary page under `<knowledge-root>/sources/<type>/` only when ingesting a durable external source. Use `papers/`, `articles/`, `web/`, or `docs/` when the type is clear, and include source metadata, concise summary, key claims, limitations, citations or raw links, related concepts, and open questions.
10. Distill reusable knowledge into the knowledge locale.
11. Classify each item as source summary, global knowledge, project-local context, decision, lesson, open question, artifact reference, HTML artifact candidate, glossary term, or transient discard.
12. Search existing indexes and likely pages before creating new pages.
13. Re-read relevant existing concept, method, comparison, and project pages, then update them with cross-source or cross-session observations: common patterns, contradictions, refinements, complementary evidence, and questions raised by the new source or session insight.
14. Merge into existing Markdown pages in the knowledge locale where possible.
15. Create a new Markdown page only when the knowledge has a stable topic.
16. Create or update an HTML artifact only when structure or visualization materially improves understanding; write visible text in the knowledge locale.
17. Update the relevant `index.md`.
18. Update `log.md`.

Do not save session summaries as knowledge. Do not rewrite raw sources. Preserve citations or source references when available. Put uncertain claims or uncertain translations in `open-questions.md` or mark them as tentative. Avoid bulk or automatic ingestion that would make the knowledge base grow faster than the user can review; when source selection is unclear, prefer a small curated ingest set.

### query

When answering from the knowledge base:

1. Resolve the knowledge root.
2. Resolve the knowledge locale.
3. Answer in the knowledge locale unless the user explicitly asks for another language.
4. Read project-local knowledge first when a project is named.
5. Read global and source summary indexes, then load relevant pages.
6. Include source summary pages when they provide the best evidence trail back to raw sources.
7. Include relevant HTML artifacts by reading their title, metadata, visible text, and structure.
8. Distinguish global facts from project-local assumptions.
9. Surface stale, contradictory, weakly sourced, or translation-sensitive knowledge.
10. If the answer creates durable new knowledge, mention it as an ingest candidate in the knowledge locale.

Current user instructions override stale stored knowledge.

### lint

Lint knowledge quality and report in the knowledge locale. Include source checks, factual support, stale claims, and translation drift.

Detect:

- contradictions or weakly sourced claims
- stale claims or missing review dates
- claims that need source re-checking
- mistranslations, inconsistent translated terms, or missing original terms
- global pages containing project-local details
- project lessons that could become global principles
- orphan pages or orphan HTML artifacts
- source summaries missing raw source links, source metadata, related knowledge links, or index entries
- broken Markdown links or HTML links
- pages missing from `index.md`
- large pages that should be split
- duplicated concepts
- HTML artifacts without title, scope, sources, related links, or accessible text

Lint reports findings only. Suggest whether refine can fix each issue or whether human judgment is needed. When a source is in another language, explain the issue in the knowledge locale and cite the original wording only as much as needed.

### refine

Improve the knowledge base with small, reviewable edits:

- merge duplicates
- split oversized pages
- demote uncertain claims to open questions
- normalize terminology into the knowledge locale
- add original-language terms to glossary entries when useful
- fix inconsistent translations without changing raw sources
- move project-local material out of global pages
- extract global principles from project lessons when justified
- add backlinks and index entries
- convert Markdown-only structural explanations into HTML artifacts when helpful
- simplify or archive stale HTML artifacts
- update logs

Before large reorganizations, present a knowledge patch in the knowledge locale that lists planned global updates, project-local updates, HTML artifacts, terminology changes, decisions, open questions, and discarded transient material.

## HTML Artifact Rules

Place HTML beside related Markdown in the same semantic location:

- `global/concepts/retrieval-augmented-generation.html`
- `global/methods/knowledge-refinement-flow.html`
- `global/comparisons/markdown-vs-html-knowledge.html`
- `projects/<project-name>/decisions/0001-architecture-map.html`
- `projects/<project-name>/system-map.html`

Do not use a standalone `html/` bucket unless the user explicitly chooses that convention.

Name files with lowercase hyphen-case, for example `retrieval-pipeline-map.html`.

Each HTML artifact should include:

- title and short purpose
- `data-scope="global"` or `data-scope="project"`
- last updated date
- related Markdown pages
- source references when available
- accessible labels or text alternatives for visual structures
- print-friendly and mobile-readable layout

Avoid HTML for decorative pages. The purpose is clearer knowledge structure, not presentation polish.

## Templates

Use `templates/` for initial pages:

- `global-index.md`
- `global-log.md`
- `project.md`
- `project-index.md`
- `project-context.md`
- `decision.md`
- `lesson.md`
- `html-artifact.html`
- `config.md`

Use `SCHEMA.md` for the full directory and artifact schema when initializing or auditing a knowledge base.

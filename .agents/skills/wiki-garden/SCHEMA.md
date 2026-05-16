# Wiki Garden Schema

Wiki Garden maintains a Markdown-first knowledge base with static HTML artifacts for structural or visual knowledge. HTML lives beside Markdown in the same semantic folders.

Canonical knowledge is maintained in the configured `knowledge_locale`. Raw sources keep their original language.

## Root

The knowledge root is configurable. Use an explicit user-provided path, project config, or user global config when present. If no root is configured, default to `knowledge/`.

Do not create a dot-prefixed root by default. `.knowledge/` is supported only when it already exists or the user asks for it.

Resolution order:

1. Explicit path in the current request.
2. Project config at `<repo>/wiki-garden.config.md`.
3. Legacy project config at `<repo>/knowledge-garden.config.md` or `<repo>/knowledge-gardener.config.md`, if present.
4. User global config at `WIKI_GARDEN_CONFIG`, if set.
5. Legacy user global config at `KNOWLEDGE_GARDENER_CONFIG`, if set.
6. User global config at `~/wiki-garden.config.md`.
7. Legacy user global config at `~/knowledge-garden.config.md` or `~/knowledge-gardener.config.md`, if present.
8. Project instructions or repository convention.
9. Existing visible knowledge folder.
10. Default `knowledge/`.

Recommended user global config:

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

Recommended project override config:

```markdown
---
knowledge_root: knowledge
scope: project
knowledge_locale: ja-JP
---

# Wiki Garden Config

This project uses `knowledge/` as its Wiki Garden root.
```

Locale resolution order:

1. Explicit language in the current request.
2. Project config `knowledge_locale`.
3. User global config `knowledge_locale`.
4. Current conversation or environment locale.
5. English only when no locale can be inferred.

```text
knowledge/
  raw/
  sources/
  global/
  projects/
```

## Global Knowledge

```text
<knowledge-root>/global/
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
```

Use global knowledge for ideas that transfer across projects: concepts, principles, methods, comparisons, glossary terms, and reusable visual explanations.

## Project Knowledge

```text
<knowledge-root>/projects/<project-name>/
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

Use project-local knowledge for constraints, decisions, terms, artifacts, lessons, and visual explanations that only apply to the named project.

## Raw Sources

```text
<knowledge-root>/raw/sources/
  papers/
  articles/
  web/
  docs/
```

Raw sources are source material inside the selected knowledge root. Do not rewrite them during ingest, query, lint, or refine. Link to them from source summaries and canonical pages when useful.

For web sources, keep enough raw metadata to identify the captured source later:

- original URL
- title
- capture date
- source format, such as `markdown-extract` or `html-snapshot`

Prefer Markdown text extraction for ordinary web articles. Keep an HTML snapshot beside the metadata only when exact page structure, layout, or later verification matters.

## Source Summaries

```text
<knowledge-root>/sources/
  index.md
  papers/
    *.md
  articles/
    *.md
  web/
    *.md
  docs/
    *.md
```

Source summaries are one-source Markdown pages produced by ingest. They are the bridge between immutable raw material and cross-source knowledge pages. Each summary should link back to its raw source when available and link forward to related concepts, methods, comparisons, decisions, project context, or open questions.

Recommended source summary metadata:

```yaml
---
type: source-summary
source_type: paper
scope: global
source_path:
source_url:
captured_at:
title:
authors:
year:
---
```

Recommended sections:

- Summary
- Key Claims
- Evidence or Details
- Limitations
- Related Knowledge
- Open Questions

## Markdown Pages

Markdown pages should be concise and durable. Prefer stable topic pages over session notes.

Write Markdown pages in the knowledge locale. When source material is in another language, include important original terms on first use and preserve source references.

Concept, method, and comparison pages should be maintained as cross-source synthesis pages, not just topic definitions. When ingest adds a relevant source, update these pages with common patterns, contradictions, refinements, complementary evidence, and questions that only become visible across multiple sources.

Recommended metadata for decision pages:

```yaml
---
type: decision
scope: project
status: proposed
date:
---
```

## HTML Artifacts

HTML artifacts are canonical knowledge pages for structure-heavy explanations. Store them beside related Markdown in the same folder, not in a separate `html/` bucket by default.

Use HTML for:

- diagrams and maps
- timelines
- matrices
- decision trees
- architecture or dependency views
- interactive explainers

Required expectations:

- self-contained static HTML by default
- visible title and purpose
- explicit scope: global or project
- last updated date
- related Markdown links
- source references when available
- accessible text alternatives for visual content
- linked from the nearest knowledge `index.md`
- project HTML also listed in `artifacts.md`

Avoid using HTML for decoration-only pages or content that is clearer as plain Markdown.

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

Raw sources are source material inside the selected knowledge root. Do not rewrite them during query, ingest, lint, nurture, or what's up. Link to them from source summaries and canonical pages when useful.

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

For Japanese knowledge, use natural Japanese headings and field labels. English is reserved for proper names, product names, operation or code identifiers, API names, file paths, original source titles, and terms whose English form is required for precision. Translate ordinary schema vocabulary, including source, summary, scope, status, global/project-local knowledge, open question, and conversation seed. Do not copy English template headings or labels into Japanese pages.

Concept, method, and comparison pages should be maintained as cross-source synthesis pages, not just topic definitions. When ingest adds a relevant source, update these pages with common patterns, contradictions, refinements, complementary evidence, and questions that only become visible across multiple sources.

## Conversation Seeds

Conversation seeds support `what's up`. They are transient prompts for future user collaboration, not canonical claims or session archives.

Unless a knowledge base defines another convention, store a small set under a localized heading in the relevant file: `## Conversation Seeds` for English or `## 対話のタネ` for Japanese. When maintaining Japanese knowledge, migrate the legacy English heading rather than creating two sections.

```text
<knowledge-root>/global/open-questions.md
<knowledge-root>/projects/<project-name>/open-questions.md
```

Recommended seed shape for English knowledge:

```markdown
### Topic title

- Type: question | observation | tension | connection | proposal | research | revisit
- Status: ready | research-pending | researched | deferred
- Why now: why this matters now
- Related knowledge: links to relevant pages
- Prompt: what to discuss with the user
- Research note: motivating question, sources, and date when researched
```

Recommended seed shape for Japanese knowledge:

```markdown
### 話題の名前

- 種類: 質問 | 観察 | 考えのずれ | 接続 | 提案 | 調査 | 再検討
- 状態: 準備済み | 調査待ち | 調査済み | 保留
- 今扱う理由: なぜ今この話題を扱うのか
- 関連知識: 関連ページへのリンク
- 対話のきっかけ: ユーザーと何を話したいか
- 調査メモ: 事前調査した場合の問い、資料、調査日
```

Seed rules:

- keep only a small current set, normally three to seven per relevant scope
- deduplicate seeds and remove weak, stale, or resolved items
- preserve source URLs, titles, and dates for researched seeds
- keep pre-research bounded: absent another budget, allow up to fifteen minutes for one focused question and normally two or three authoritative sources; stop earlier when the seed is useful
- if research exceeds its execution budget, retain a `research-pending` seed without unsupported partial findings
- do not treat a researched seed as canonical truth
- link distinct evidence for each claimed repeated pattern across decisions or projects; a page that merely repeats the claim is not sufficient evidence
- when evidence is incomplete, record and present the exact evidence that was verified and label the broader pattern as a hypothesis to investigate
- do not invent examples, project occurrences, or causal explanations when presenting a seed
- do not store conversation transcripts or generic session summaries
- when discussion produces durable knowledge, move the conclusion into canonical pages through ingest or nurture and resolve the seed

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

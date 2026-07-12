---
name: wiki-garden
description: Nurture a locale-centered, Karpathy-inspired Markdown and static HTML knowledge base. Use when configuring a knowledge root or locale, ingesting durable sources or session insights, querying knowledge, linting defects and growth opportunities, nurturing topics through synthesis/dialogue/research, asking what's up to surface prepared conversation seeds, or creating diagram-heavy HTML knowledge artifacts.
---

# Wiki Garden

Use this skill to keep a durable knowledge base from current work, source files, and project decisions. Preserve distilled knowledge, not chat transcripts.

Wiki Garden follows Karpathy's LLM Wiki pattern: raw sources are immutable source-of-truth inputs, the wiki is the maintained Markdown knowledge layer, and the skill instructions act as the operating schema. Preserve the familiar `query`, `ingest`, and `lint` operations, then add `nurture` and `what's up` as an active knowledge-growth layer. The goal is not merely to record information, but to compile, question, and nurture durable knowledge over time.

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

Raw sources live inside the selected knowledge root. Treat `raw/` as the immutable source layer: humans add or capture source material there, and agents read it but do not rewrite it during query, ingest, lint, nurture, or what's up.

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

## Setup and diagnostics

The five knowledge operations remain `query`, `ingest`, `lint`, `nurture`, and `what's up`. Provide these additional setup actions so users do not need to assemble configuration steps by hand.

### setup

Use `setup` as the recommended first-run action. Recognize `init`, `initialize`, `初期設定`, and `セットアップ` as aliases.

Supported forms:

```text
setup --project [path] --locale <locale>
setup --shared <path> --locale <locale>
setup
```

Behavior:

1. Inspect existing project and user configuration before writing anything.
2. If `--project` is given, create or update `<repo>/wiki-garden.config.md`. Use the supplied path or `knowledge/` when omitted.
3. If `--shared` is given, create or update the user config selected by `WIKI_GARDEN_CONFIG`, or `~/wiki-garden.config.md` otherwise. Require or infer the shared path only when it is unambiguous; recommend `~/WikiGarden` as the default.
4. Use the explicit locale. If none is given, prefer an existing configured locale, then the current conversation language. For Japanese conversation, default to `ja-JP`.
5. Create the selected knowledge root and starter pages when setup is requested. Do not overwrite existing knowledge pages.
6. Use localized templates. For Japanese, use `templates/ja/` and create Japanese visible headings and prose.
7. Create only the useful starter structure: configuration, root folders, global index, global log, glossary, open questions, and project starter pages when project scope is selected.
8. Report the config path, effective knowledge root, locale, created files, preserved existing files, and a suggested first `ingest` or `query` request.
9. Do not migrate or move an existing garden automatically.

For Japanese setup reports:

- title the result `初期設定完了`, not `Setup complete` or `Summary`
- use headings such as `設定`, `作成したもの`, `残した既存ファイル`, and `次にできること`
- use `雛形` instead of template, `適用範囲` instead of scope, `知識の保存先` instead of knowledge root, and `記述言語` instead of locale
- keep literal configuration keys and operation identifiers in backticks only when they must be shown
- apply the Japanese output-quality rules to the completion message as strictly as to knowledge pages

When bare `setup` finds no configuration and shared versus project scope would materially change the result, ask one concise question. If the user already supplied a repository path or clearly asked for project setup, proceed without asking.

### First-use guidance

Before carrying out `query`, `ingest`, `lint`, `nurture`, or `what's up`, determine whether the garden has an explicit project, user, legacy, or user-supplied configuration. Do not silently treat a missing configuration as permission to create a new garden merely because `knowledge/` is the fallback path.

When no configuration exists and the fallback `knowledge/` directory does not exist:

- For `query`, `lint`, and `what's up`, explain that there is no configured garden to read. Offer the two meaningful starting choices: a project garden in `knowledge/` or a shared garden such as `~/WikiGarden/`.
- For `ingest` and `nurture`, stop before writing anything and ask the user to choose those same two locations, unless the request already gives an unambiguous storage path and scope.
- In a Japanese conversation, ask in natural Japanese, for example: `Wiki Gardenの保存先がまだ決まっていません。このプロジェクトのknowledge/に作りますか。それとも、複数のプロジェクトで共有する場所に作りますか。`
- Once the user chooses, perform `setup` as the first write. Report the selected scope, configuration file, knowledge location, and language before continuing with the requested operation.

When an unconfigured but existing knowledge folder is found, do not replace it. For read-only operations, explain that the folder was inferred and suggest a diagnostic. For operations that write canonical knowledge, ask whether to adopt it as the project garden or to select another location before writing.

### doctor

Use `doctor` as a read-only setup check. Recognize `check setup`, `設定確認`, and `診断` as aliases.

Check and report:

- which config file and precedence rule selected the root and locale
- whether the root exists and is readable
- whether required starter folders and index pages exist
- conflicting project, user, or legacy configuration
- missing or duplicated global and project indexes
- whether Japanese gardens use Japanese headings, seed labels, and localized templates
- recommended repair commands or a proposed setup patch

For Japanese diagnostics:

- title the report `設定診断結果`, not `Doctor` or `Doctor診断結果`
- use Japanese labels such as `設定元`, `知識の保存先`, `記述言語`, `初期構造`, `日本語品質`, and `総合判定`
- scan visible headings and field labels for ordinary Latin words; allow proper names and required identifiers such as Markdown, HTML, API, and filenames, but flag headings such as `Web`, `Status`, or `Conversation Seeds`
- use natural Japanese in the report; do not use ordinary English or unnecessary katakana merely because the internal operation name or schema is English

`doctor` must not create, rewrite, move, or delete files. Direct the user to `setup`, `nurture`, or `lint` for repairs.

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
- knowledge patches, nurture summaries, and conversation seeds

When ingesting foreign-language sources:

- keep raw sources in the original language
- distill canonical knowledge into the knowledge locale
- include important original terms on first use, for example `検索拡張生成（Retrieval-Augmented Generation, RAG）`
- do not translate proper nouns, code identifiers, API names, file paths, commands, model names, or quoted source titles unless there is a standard localized name
- put uncertain translations in `glossary.md` or `open-questions.md`
- avoid literal translation when a clearer localized explanation preserves the source meaning better

### Japanese output quality

When the effective knowledge locale is Japanese (`ja` or `ja-*`), write natural Japanese rather than English-shaped Japanese:

- Use English only for proper names, product names, operation identifiers, code identifiers, API names, file paths, source titles, and technical terms whose English form is necessary for precision.
- Keep operation identifiers such as `query`, `ingest`, `lint`, `nurture`, and `what's up` in backticks, but explain their meaning in Japanese. Do not turn them into unnecessary katakana forms such as ナーチャリング.
- Translate ordinary knowledge-management vocabulary. Prefer `資料` or `情報源` over source, `資料要約` over source summary, `正規知識` over canonical knowledge, `全体知識` over global knowledge, `プロジェクト固有知識` over project-local knowledge, `未解決の問い` over open question, `対象範囲` over scope, `知識の記述言語` over knowledge locale or locale, and `対話のタネ` over Conversation Seed or Seed.
- Prefer `ウェブ` over Web when it is an ordinary category label, `索引` over index, `履歴` over log, `旧形式` over legacy, `分類` over category, and `構造規約` over schema in Japanese prose. Preserve literal filenames such as `index.md` and `log.md`.
- Prefer `雛形` over template, `設定` over configuration when no distinction is lost, and `要約` over summary in Japanese prose.
- Prefer `文書` over document, documents, or ドキュメント, and `項目` over entry or エントリ in Japanese prose. Preserve literal directory names such as `docs/`.
- Localize headings, field labels, categories, statuses, reports, patches, indexes, logs, and visible HTML text. Do not expose English schema labels such as `Type`, `Status`, `Why now`, `Defects`, or `Growth Opportunities` in Japanese knowledge.
- For Japanese conversation seeds, prefer labels such as `種類`, `状態`, `今扱う理由`, `関連知識`, `対話のきっかけ`, and `調査メモ`; prefer values such as `質問`, `観察`, `考えのずれ`, `接続`, `提案`, `調査`, `再検討`, `準備済み`, `調査待ち`, `調査済み`, and `保留`.
- Write sentences around Japanese verbs and syntax. Avoid strings of English nouns joined by Japanese particles.
- In examples written for users, show a natural request rather than an internal invocation or an operation identifier used as a Japanese verb. Explain `ingest` as an operation name if useful, but write the request itself as `この資料を取り込んで`, not `この資料をingestして`.
- When an original term helps disambiguation, add it once on first use, then continue in Japanese.
- Preserve quoted source wording only when evidence or translation review requires it.

### Human-facing documentation quality

Treat a README, tutorial, guide, and explanatory knowledge page as an edited publication for a person, not as a dump of features or internal rules. Apply these requirements in every language:

1. Identify the intended reader, the question the document answers, and the next action the reader should be able to take.
2. Give each document one job. A README helps someone decide, install, and reach a first success. A tutorial leads one concrete example from start to finish. Put exhaustive contracts and field definitions in reference documentation instead.
3. Lead with the reader's outcome. Introduce architecture and terminology only after the reader knows why they matter.
4. Use progressive disclosure: essential path first, choices and boundaries later, reference details by link. Remove repeated command catalogs and repeated explanations.
5. Explain an unfamiliar term before relying on it. Prefer a concrete example over an abstract inventory.
6. Make every command example runnable in its stated context, and tell the reader what visible result to expect.
7. Write each language natively. Preserve facts across language editions, but do not translate sentence by sentence or force the same paragraph structure. Edit Japanese and English independently for their customary rhythm and rhetoric.
8. Keep internal field names, schema labels, and implementation detail out of human prose unless the reader needs them to complete the task.
9. Before delivery, review the heading outline, remove redundancy, validate links and commands, and perform a first-screen test: the opening should establish value and lead to a useful action without requiring the rest of the document.
10. Read the finished prose as a human reader would. Rewrite passages that are grammatical but impersonal, list-like, translation-shaped, or unclear about why the reader should care.

For Japanese documents, also apply the Japanese output-quality rules above. Do not use English or katakana as a substitute for finding the ordinary Japanese expression. For English documents, use direct idiomatic English and avoid bureaucratic strings of nominalizations.

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

The five user-facing operations have distinct contracts:

- `query` uses current knowledge and is read-only.
- `ingest` compiles specified durable material into canonical knowledge.
- `lint` reports defects and growth opportunities without repairing canonical knowledge.
- `nurture` develops a selected topic and may apply small reviewable canonical changes.
- `what's up` prepares or surfaces conversation seeds; it may research a bounded local question but does not promote findings into canonical claims by itself.

Treat `grow`, `deepen`, `develop`, `cultivate`, `育てる`, `深める`, and `発展させる` as natural-language aliases for `nurture`. For backward compatibility, interpret `refine` as a structurally scoped nurture unless the user clearly requests broader research or dialogue.

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

Lint knowledge quality and report in the knowledge locale. Include source checks, factual support, stale claims, and translation drift. Apply the locale's natural terminology to all headings and labels; for Japanese, use headings such as `問題`, `成長の機会`, `影響`, and `対応案`, not English headings or unnecessary katakana.

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
- important knowledge gaps or unexplained assumptions
- potentially useful connections across pages or projects
- questions that require the user's intent, preferences, or local experience
- concrete external questions that could benefit from bounded research

Lint reports findings only. Separate defects from growth opportunities. Suggest whether nurture can handle each finding, whether ingest or external evidence is needed, or whether human judgment is required. When a source is in another language, explain the issue in the knowledge locale and cite the original wording only as much as needed.

### nurture

Nurture a selected topic by making the existing knowledge clearer, stronger, better connected, and more useful. Start from the garden rather than from a blank answer:

1. Resolve the knowledge root and knowledge locale.
2. Identify the explicit topic, selected conversation seed, lint finding, or user goal.
3. Read the relevant project-local pages first, then global pages, source summaries, raw evidence when verification matters, and related HTML artifacts.
4. Run a focused semantic lint of the topic: find contradictions, gaps, stale assumptions, weak evidence, duplicated concepts, missing distinctions, unexplained local choices, and possible generalizations.
5. Consider three knowledge perspectives:
   - internal knowledge: connect pages, compare claims, resolve tensions, generalize lessons, and prune duplication
   - user knowledge: ask about intent, experience, preferences, decisions, or constraints that external research cannot establish
   - external knowledge: find evidence, examples, counterarguments, adjacent ideas, and current facts when they materially improve the topic
6. Use the useful perspectives together. Do not perform external research merely to appear comprehensive, and do not ask the user questions the existing garden can answer.
7. Ask for user direction when a choice would materially change the scope, when local intent is decisive, or before a large reorganization. Do not interrupt progress with approval questions for small read-only analysis.
8. When external research is warranted, prefer an available dedicated deep-research capability, including environment-specific skills such as `deep-research` or `hachi-deep-research`. Otherwise use the best available research tools. Follow the selected research skill's instructions and prefer primary or authoritative sources.
9. Apply the normal ingest rules to any external source or durable conversation insight that is promoted into the garden. Preserve source metadata and do not rewrite raw sources.
10. Produce small, reviewable canonical edits even when the investigation is deep. Update the relevant index and log.

Nurture may:

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
- connect previously separate knowledge and make assumptions explicit
- reconcile or clearly preserve competing interpretations
- strengthen weak claims with evidence or demote them to open questions
- turn repeated project lessons into justified global principles
- add carefully selected external evidence or counterevidence through ingest
- preserve unresolved branches as open questions or future conversation seeds

Before large reorganizations or meaning-changing edits, present a knowledge patch in the knowledge locale that lists planned global updates, project-local updates, HTML artifacts, terminology changes, decisions, open questions, and discarded transient material. Deep investigation does not justify a large or opaque diff.

### what's up

Use `what's up` as the conversational entrypoint for knowledge the garden has noticed or prepared. Recognize natural-language forms such as `whats up`, `anything?`, `what have you noticed?`, `何かある？`, `最近どう？`, `私に聞きたいことある？`, and `調べたいことある？`.

Conversation seeds are transient prompts for collaboration, not canonical claims. Unless the knowledge base defines another convention, keep a localized section in the relevant global or project `open-questions.md`, such as `## Conversation Seeds` in English or `## 対話のタネ` in Japanese. Read and migrate the legacy English heading when maintaining a Japanese garden. A seed may be a question, observation, tension, connection, proposal, research topic, revisit candidate, or researched finding.

When invoked to prepare seeds, including from a scheduled host trigger:

1. Resolve the knowledge root, locale, and requested project or scope.
2. Review recent `log.md` entries, changed or recently relevant knowledge pages, open questions, source summaries, and lint findings.
3. Look for topics that are timely, consequential, answerable, connected to current interests, or uniquely dependent on the user.
4. Deduplicate and rank candidates. Keep a small set, normally three to seven per relevant scope, and expire weak or stale seeds.
5. For each retained seed, record its type, status, why it matters now, related knowledge links, a natural conversation prompt, and any research note.
6. Do not interrupt the user. Do not change canonical claims, decisions, principles, or source summaries while merely preparing seeds.

When a local knowledge gap is concrete, bounded, mapped to affected pages, and answerable from external evidence, preparation or what's up may research it before presentation:

1. Prefer an available dedicated deep-research skill such as `deep-research` or `hachi-deep-research`; otherwise use the best available research tools.
2. Treat seed enrichment as a bounded research pass, not an unlimited survey. Follow an explicit user or host budget; when none is given, allow up to fifteen minutes for one focused question with normally two or three authoritative sources. Stop earlier as soon as the seed is useful; spending the full budget is not a goal.
3. If research cannot finish within the available execution budget, including the default fifteen-minute limit, stop cleanly and retain a research-pending seed with the question and next research step. Use a localized status label such as `調査待ち` in Japanese. Do not block seed preparation indefinitely or save unsupported partial findings.
4. Preserve the motivating question, source URLs, titles, publication or capture dates, and research date in the seed.
5. Treat research results as provisional seed material, not canonical truth.
6. Avoid broad topic surveys, bulk source collection, or research whose scope cannot be bounded.
7. If the answer depends on user intent, preference, or local experience, prepare a question instead of searching.

When the user asks what's up:

1. Read the relevant prepared seeds.
2. If none exist, perform a lightweight live scan; do not invent a topic merely to sustain conversation.
3. Validate candidate seeds before selection. For each material factual claim, open the linked evidence and determine whether it supports the wording. A claim that something recurs across decisions or projects needs links to the distinct occurrences, not merely a page that repeats the claim.
4. If evidence is missing, either choose a better-grounded seed or explicitly reframe the item: `The garden records this as a possible pattern, but I could only verify <available evidence>; should we investigate it?` Never fill the gap from plausibility.
5. Select one validated or explicitly reframed seed by relevance, timeliness, leverage, novelty, and likely conversational value. Present one by default rather than dumping the queue.
6. Turn the stored seed into natural conversation in the knowledge locale. Lead with the observation, question, or researched finding and explain briefly why it matters now.
7. Ground the presentation in the seed and the related pages actually read. Do not invent supporting examples, project occurrences, causal links, or research findings to make the topic sound stronger.
8. For researched seeds, distinguish sourced findings from the garden's inference and keep source references available.
9. Let the user engage, defer, discard, request another category, or ask for another seed.
10. If the user engages, transition into nurture. If durable knowledge results, ingest it or include it in the scoped nurture patch. If not, do not archive the conversation.

A useful seed shape is shown below. Localize its labels and values into the knowledge locale; do not copy the English labels into Japanese knowledge.

```markdown
### Topic title

- Type: question | observation | tension | connection | proposal | research | revisit
- Status: ready | research-pending | researched | deferred
- Why now: why this matters now
- Related knowledge: links to relevant pages
- Prompt: what to discuss with the user
- Research note: motivating question, sources, and date when researched
```

## HTML Artifact Rules

Place HTML beside related Markdown in the same semantic location:

- `global/concepts/retrieval-augmented-generation.html`
- `global/methods/knowledge-nurture-flow.html`
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
- `source-index.md`
- `project.md`
- `project-index.md`
- `project-context.md`
- `decision.md`
- `lesson.md`
- `html-artifact.html`
- `config.md`

When `knowledge_locale` is Japanese, prefer the localized files under `templates/ja/`. Template structure is reusable, but visible headings and prose must always be written in the knowledge locale. Never copy English template headings into Japanese knowledge merely because the base template is English.

Use `SCHEMA.md` for the full directory and artifact schema when initializing or auditing a knowledge base.

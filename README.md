# Wiki Garden

[日本語](README_ja.md)

Wiki Garden is an agent skill for turning sources and working conversations into knowledge that can be questioned, checked, and developed over time.

It builds on Andrej Karpathy's LLM Wiki operations—`query`, `ingest`, and `lint`—and adds `nurture` for developing a chosen topic and `what's up` for hearing what the wiki has noticed.

## Grow your first note

For the standard workflow, install the skill globally and grow one shared garden. This is the only step here that runs in your terminal:

```bash
npx skills add hachiware-labs/wiki-garden -g
```

You now have two independent choices: where the skill is available, and where the garden stores its knowledge.

| Decision | One project | Shared across projects |
| --- | --- | --- |
| Skill installation | available only in the current repository | available in every repository you work in |
| Knowledge location | `knowledge/` inside the repository | a folder outside repositories, such as `~/WikiGarden/` |

Installing the skill globally does not require a shared garden. For example, you can make the skill available everywhere while keeping a separate `knowledge/` folder in each repository.

### Good starting combinations

| Your goal | Skill installation | Knowledge location |
| --- | --- | --- |
| Standard workflow: grow methods, research, and terminology across projects | Global | Shared folder such as `~/WikiGarden/` |
| Keep a particular engagement separate: client work, sensitive material, or short-lived work | Current project | `knowledge/` in that repository |
| Make Wiki Garden available everywhere while keeping each project's knowledge separate | Global | `knowledge/` in each repository |

After choosing, ask your agent in ordinary language to create the garden.

### Standard workflow: share knowledge across projects

Choose this for methods, research, terminology, and design principles that should grow across projects. Knowledge that belongs to one project stays in that project's area within the shared garden.

```text
Set up a Wiki Garden shared by all my projects.
Store its knowledge in ~/WikiGarden and write it in English.
```

This writes `~/wiki-garden.config.md` and stores the actual knowledge in `~/WikiGarden/`. Treat this as personal knowledge outside the repository unless you deliberately manage that folder with a separate, appropriate sharing method.

### Exception: keep one project's knowledge separate

Choose this for client work, sensitive material, or work that should not mix with your shared garden.

```text
Set up Wiki Garden for this project.
Store its knowledge in knowledge/ and write it in English.
```

This writes `wiki-garden.config.md` at the repository root and stores the garden in `knowledge/`. Commit both when the knowledge should travel with the project or be shared with the team.

In either case, you can check the result without changing any files:

```text
Check the Wiki Garden configuration without changing any files.
Tell me the active knowledge location and language.
```

If you ask to search or add knowledge before setup, Wiki Garden does not silently create a new location. It asks whether the garden belongs in the project's `knowledge/` folder or in a shared location, performs setup after you choose, and then continues with the original request.

Now choose one source and ingest it:

```text
Add docs/api-notes.md to Wiki Garden and preserve the original source.
```

The original source remains available for verification. The garden creates a source note and folds reusable ideas into the relevant knowledge pages. The [tutorial](docs/tutorial.md) walks through the full cycle with a checkout API example.

## From storage to growth

A conventional knowledge base waits for someone to bring it a source or a question. Wiki Garden can also notice weak evidence, unresolved tensions, and ideas that have not yet been connected. It prepares those observations for a future conversation.

```text
Sources and decisions
  → distill durable knowledge
  → notice gaps and tensions
  → discuss one prepared seed
  → nurture it through dialogue or research
  → keep the durable conclusion
```

The wiki does not silently promote speculation into fact. Proactive findings begin as conversation seeds. You decide what deserves attention and whether the result belongs in canonical knowledge.

## The five operations

You do not have to memorize these names. Describe what you want in ordinary language and Wiki Garden can choose the appropriate operation.

| Operation | Choose it when you want to... | Example request in ordinary language | Changes knowledge? |
| --- | --- | --- | --- |
| `query` | answer from the current garden | “What do we currently know about payment retries?” | No |
| `ingest` | preserve a source or a durable decision | “Add this design note to Wiki Garden.” | Yes |
| `lint` | find contradictions, staleness, weak evidence, and growth opportunities | “Check this knowledge for contradictions and missing evidence.” | No |
| `nurture` | develop one topic through synthesis, dialogue, or research | “Develop our knowledge of idempotency, checking decisions with me as needed.” | Small, reviewable edits |
| `what's up` | hear one topic the garden has prepared | “Wiki Garden, anything worth discussing?” | Not when only presenting it |

For example, lint may notice that a decision has no rationale. Nurture can read the surrounding knowledge, ask you for missing local context, and research external evidence when it would help. The result returns as a focused, reviewable knowledge change.

`what's up` presents one high-value seed from a small queue prepared by a periodic run:

```text
Wiki Garden, anything worth discussing?
```

When the local question is already concrete, seed preparation may rely on a dedicated Deep Research capability or the best available search tools. By default it investigates one question, checks two or three authoritative sources, and stops within fifteen minutes. Unfinished work remains explicitly research-pending.

## How this extends Karpathy's LLM Wiki

Karpathy's pattern compiles curated sources into an interlinked Markdown wiki before query time. Raw material remains the source of truth; `ingest` turns it into maintained knowledge, `query` reads that knowledge, and `lint` checks its health.

Wiki Garden keeps that foundation. Its added growth loop helps the person and the wiki decide what to develop next. Agent conversations are not archived wholesale: only durable decisions, constraints, lessons, terminology, and open questions are kept.

## Where knowledge lives

The default garden separates evidence, source notes, reusable knowledge, and project-specific knowledge:

```text
knowledge/
  raw/sources/       immutable source material
  sources/           one note per source
  global/            knowledge reusable across projects
  projects/          knowledge true only within a project
```

Canonical pages follow the configured locale even when a source is written in another language. Proper names, API names, identifiers, filenames, and quotations remain in their necessary original form.

Markdown is the default for prose, decisions, and lessons. Static HTML is available when a diagram, timeline, map, or interactive explanation carries the knowledge more clearly than prose.

## Choose where to install the skill

The first installation command adds the skill globally. Lower-level path and language actions remain available, but most people only need the initial setup described above. If you lose track of the active settings, ask the skill to check its configuration.

To add the skill only to the current project, omit `-g`:

```bash
npx skills add hachiware-labs/wiki-garden
```

Install only for Claude Code and Codex:

```bash
npx skills add hachiware-labs/wiki-garden -a claude-code -a codex
```

Add the skill globally with `-g`:

```bash
npx skills add hachiware-labs/wiki-garden -g
```

Inspect or update installed skills:

```bash
npx skills list
npx skills update wiki-garden
```

## Read next

- [Tutorial](docs/tutorial.md) — run one source through the complete growth loop
- [Requirements (Japanese)](docs/wiki-garden-requirements.md) — detailed behavioral contracts
- [日本語 README](README_ja.md) — a separately edited Japanese introduction

The canonical package lives in `.agents/skills/wiki-garden/`; `skills/wiki-garden/` is its distribution mirror. Edit the canonical copy first.

## Current boundaries

Wiki Garden does not ship a database, vector search, automatic chat capture, user interface, or scheduler. Periodic seed discovery is invoked by the host environment, such as Codex or Claude Code.

## References

- Andrej Karpathy, [`llm-wiki` gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- DAIR.AI Academy, [LLM Knowledge Bases](https://academy.dair.ai/blog/llm-knowledge-bases-karpathy)

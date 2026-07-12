# Grow Your First Piece of Knowledge

[日本語版](tutorial_ja.md)

This tutorial takes one short checkout API note through the complete Wiki Garden loop: ingest it, ask a question, find what is missing, develop the knowledge, and hear what the wiki wants to discuss next. Allow about fifteen minutes.

## Step 1. Set up Wiki Garden

First make two separate decisions. The skill makes Wiki Garden available to your agent; the garden is where sources and decisions are actually stored.

| Decision | Per project | Shared more broadly |
| --- | --- | --- |
| Skill installation | Available only in the current repository | Available in every repository |
| Knowledge location | `knowledge/` inside the repository | A location outside repositories, such as `~/WikiGarden/` |

This tutorial follows the standard workflow: install the skill globally and keep knowledge in a shared `~/WikiGarden/` folder. The checkout note is still project-specific; it belongs in that project's area of the shared garden.

### Install the skill globally

```bash
npx skills add hachiware-labs/wiki-garden -g
```

For a client engagement or other isolated project, install the skill only in that repository instead:

```bash
npx skills add hachiware-labs/wiki-garden
```

### Store knowledge in a shared garden

Ask the installed Wiki Garden skill to keep English knowledge in `~/WikiGarden/`. This is a request to your agent, not a terminal command:

```text
Set up a Wiki Garden shared by all my projects.
Store its knowledge in ~/WikiGarden and write it in English.
```

This creates `~/wiki-garden.config.md` and keeps the actual garden in `~/WikiGarden/`. Project-specific knowledge remains separate inside the shared garden.

To isolate knowledge for one project instead, ask:

```text
Set up Wiki Garden for this project.
Store its knowledge in knowledge/ and write it in English.
```

That creates `wiki-garden.config.md` and `knowledge/` inside the repository. Commit both when the garden should move with the project or be shared with a team.

Check the active configuration:

```text
Check the Wiki Garden configuration without changing any files.
Tell me the active knowledge location and language.
```

The report should show the configuration source, knowledge location, and language. The check is read-only.

## Step 2. Ingest a source (`ingest`)

Create `docs/api-notes.md` with this content:

```markdown
# Checkout API Notes

Payment creation uses an idempotency key.
When retrying a timed-out request, the client reuses the same key.
Each key belongs to one checkout and expires after 24 hours.

Reusing a key with a different request body currently returns HTTP 409.
This behavior is specific to checkout-redesign and must not be generalized without other evidence.
```

Then ask Wiki Garden to ingest it:

```text
Add docs/api-notes.md to Wiki Garden as knowledge specific to checkout-redesign.
Keep the raw source inside the knowledge root.
```

Ingest does more than copy a file. It preserves the evidence, creates a note for that source, and folds reusable ideas into the pages where they belong.

Afterward, check that:

- the original appears under `raw/sources/`
- a source note appears under `sources/`
- project knowledge appears under `projects/checkout-redesign/`
- the HTTP 409 behavior remains project-specific rather than becoming a universal rule

## Step 3. Ask the ingested knowledge a question (`query`)

Query the compiled knowledge instead of reopening the source:

```text
What do we currently know about payment retries in checkout-redesign?
```

The answer should cover reuse of the idempotency key, its scope and lifetime, and the HTTP 409 behavior. It should also preserve the important boundary: only this project currently supports the 409 claim.

Query is read-only. Answering the question must not alter the garden.

## Step 4. Find where the knowledge can grow (`lint`)

Now inspect the knowledge for weaknesses:

```text
Check the checkout-redesign knowledge for contradictions and weak evidence.
Include opportunities to develop it as well as defects.
```

This small example may reveal that the project has no stated goal, that the 409 behavior has not been compared with other APIs, or that the reason for choosing idempotency keys is thin.

Lint diagnoses; it does not repair. You remain free to choose which finding matters.

## Step 5. Add human judgment and develop the knowledge (`nurture`)

Suppose the team now states its goal:

> Prevent duplicate charges after timeouts and make retries safe. Pricing logic and user-interface redesign are out of scope.

Ask Nurture to connect that decision to the existing project knowledge:

```text
Develop the project goal and non-goals of checkout-redesign in the existing knowledge.
The goal is to prevent duplicate charges and make retries safe.
Pricing logic and UI redesign are out of scope.
Do not research externally; integrate only this decision as a small patch.
```

The expected change is focused: the project page and its history should reflect the decision. Raw evidence and unrelated global knowledge should stay untouched.

Nurture can also ask for local intent or conduct bounded research when those are needed. Deeper reasoning should still return as a small change you can review.

## Step 6. Hear what Wiki Garden noticed (`what's up`)

Prepare a topic for a future conversation:

```text
Prepare topics worth discussing with me from recent logs, open questions, and lint findings.
Do not research externally or change canonical knowledge in this run.
```

Then ask:

```text
Wiki Garden, anything worth discussing?
```

The garden should present one topic, not recite a queue. In this example it might suggest comparing other APIs before generalizing the HTTP 409 behavior. With only one observed case, it must describe that broader idea as a question, not an established pattern.

If the topic matters, nurture it. If not, defer it. Conversation seeds are prompts for judgment, not canonical facts.

## Using the garden day to day

- Choose `query` when you need an answer.
- Choose `ingest` when a source or decision deserves to persist.
- Choose `lint` when you want to see weak spots.
- Choose `nurture` when one topic deserves development.
- Choose `what's up` when you want to hear what the garden has noticed.

A periodic host task can prepare a small set of seeds from recent changes. When a local question is concrete, it may research one question for up to fifteen minutes, normally using two or three authoritative sources. Unfinished work remains research-pending rather than being presented as fact.

Do not design a large wiki before you begin. Grow one source, one question, and one improvement at a time.

See the [README](../README.md) for installation choices and the [requirements](wiki-garden-requirements.md) for detailed behavioral contracts.

# LLM Wiki

A pattern for building personal knowledge bases using LLMs.

This is an idea file, it is designed to be copy pasted to your own LLM Agent (e.g. OpenAI Codex, Claude Code, OpenCode / Pi, or etc.). Its goal is to communicate the high level idea, but your agent will build out the specifics in collaboration with you.

## The core idea

Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is rediscovering knowledge from scratch on every question. There's no accumulation. Ask a subtle question that requires synthesizing five documents, and the LLM has to find and piece together the relevant fragments every time. Nothing is built up. NotebookLM, ChatGPT file uploads, and most RAG systems work this way.

The idea here is different. Instead of just retrieving from raw documents at query time, the LLM **incrementally builds and maintains a persistent wiki** — a structured, interlinked collection of markdown files that sits between you and the raw sources. When you add a new source, the LLM doesn't just index it for later retrieval. It reads it, extracts the key information, and integrates it into the existing wiki — updating entity pages, revising topic summaries, noting where new data contradicts old claims, strengthening or challenging the evolving synthesis. The knowledge is compiled once and then *kept current*, not re-derived on every query.

This is the key difference: **the wiki is a persistent, compounding artifact.** The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read. The wiki keeps getting richer with every source you add and every question you ask.

You never (or rarely) write the wiki yourself — the LLM writes and maintains all of it. You're in charge of sourcing, exploration, and asking the right questions. The LLM does all the grunt work — the summarizing, cross-referencing, filing, and bookkeeping that makes a knowledge base actually useful over time. In practice, I have the LLM agent open on one side and Obsidian open on the other. The LLM makes edits based on our conversation, and I browse the results in real time — following links, checking the graph view, reading the updated pages. Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.

This can apply to a lot of different contexts. A few examples:

- **Personal**: tracking your own goals, health, psychology, self-improvement — filing journal entries, articles, podcast notes, and building up a structured picture of yourself over time.
- **Research**: going deep on a topic over weeks or months — reading papers, articles, reports, and incrementally building a comprehensive wiki with an evolving thesis.
- **Reading a book**: filing each chapter as you go, building out pages for characters, themes, plot threads, and how they connect. By the end you have a rich companion wiki. Think of fan wikis like [Tolkien Gateway](https://tolkiengateway.net/wiki/Main_Page) — thousands of interlinked pages covering characters, places, events, languages, built by a community of volunteers over years. You could build something like that personally as you read, with the LLM doing all the cross-referencing and maintenance.
- **Business/team**: an internal wiki maintained by LLMs, fed by Slack threads, meeting transcripts, project documents, customer calls. Possibly with humans in the loop reviewing updates. The wiki stays current because the LLM does the maintenance that no one on the team wants to do.
- **Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives** — anything where you're accumulating knowledge over time and want it organized rather than scattered.

## Architecture

There are three layers:

**Raw sources** — your curated collection of source documents. Articles, papers, images, data files. These are immutable — the LLM reads from them but never modifies them. This is your source of truth.

**The wiki** — a directory of LLM-generated markdown files. Summaries, entity pages, concept pages, comparisons, an overview, a synthesis. The LLM owns this layer entirely. It creates pages, updates them when new sources arrive, maintains cross-references, and keeps everything consistent. You read it; the LLM writes it.

**The schema** — a document (e.g. CLAUDE.md for Claude Code or AGENTS.md for Codex) that tells the LLM how the wiki is structured, what the conventions are, and what workflows to follow when ingesting sources, answering questions, or maintaining the wiki. This is the key configuration file — it's what makes the LLM a disciplined wiki maintainer rather than a generic chatbot. You and the LLM co-evolve this over time as you figure out what works for your domain.

## Operations

**Ingest.** You drop a new source into the raw collection and tell the LLM to process it. An example flow: the LLM reads the source, discusses key takeaways with you, writes a summary page in the wiki, updates the index, updates relevant entity and concept pages across the wiki, and appends an entry to the log. A single source might touch 10-15 wiki pages. Personally I prefer to ingest sources one at a time and stay involved — I read the summaries, check the updates, and guide the LLM on what to emphasize. But you could also batch-ingest many sources at once with less supervision. It's up to you to develop the workflow that fits your style and document it in the schema for future sessions.

**Query.** You ask questions against the wiki. The LLM searches for relevant pages, reads them, and synthesizes an answer with citations. Answers can take different forms depending on the question — a markdown page, a comparison table, a slide deck (Marp), a chart (matplotlib), a canvas. The important insight: **good answers can be filed back into the wiki as new pages.** A comparison you asked for, an analysis, a connection you discovered — these are valuable and shouldn't disappear into chat history. This way your explorations compound in the knowledge base just like ingested sources do.

**Lint.** Periodically, ask the LLM to health-check the wiki. Look for: contradictions between pages, stale claims that newer sources have superseded, orphan pages with no inbound links, important concepts mentioned but lacking their own page, missing cross-references, data gaps that could be filled with a web search. The LLM is good at suggesting new questions to investigate and new sources to look for. This keeps the wiki healthy as it grows.

## Indexing and logging

Two special files help the LLM (and you) navigate the wiki as it grows. They serve different purposes:

**index.md** is content-oriented. It's a catalog of everything in the wiki — each page listed with a link, a one-line summary, and optionally metadata like date or source count. Organized by category (entities, concepts, sources, etc.). The LLM updates it on every ingest. When answering a query, the LLM reads the index first to find relevant pages, then drills into them. This works surprisingly well at moderate scale (~100 sources, ~hundreds of pages) and avoids the need for embedding-based RAG infrastructure.

**log.md** is chronological. It's an append-only record of what happened and when — ingests, queries, lint passes. A useful tip: if each entry starts with a consistent prefix (e.g. `## [2026-04-02] ingest | Article Title`), the log becomes parseable with simple unix tools — `grep "^## \[" log.md | tail -5` gives you the last 5 entries. The log gives you a timeline of the wiki's evolution and helps the LLM understand what's been done recently.

## Optional: CLI tools

At some point you may want to build small tools that help the LLM operate on the wiki more efficiently. A search engine over the wiki pages is the most obvious one — at small scale the index file is enough, but as the wiki grows you want proper search. [qmd](https://github.com/tobi/qmd) is a good option: it's a local search engine for markdown files with hybrid BM25/vector search and LLM re-ranking, all on-device. It has both a CLI (so the LLM can shell out to it) and an MCP server (so the LLM can use it as a native tool). You could also build something simpler yourself — the LLM can help you vibe-code a naive search script as the need arises.

## Tips and tricks

- **Obsidian Web Clipper** is a browser extension that converts web articles to markdown. Very useful for quickly getting sources into your raw collection.
- **Download images locally.** In Obsidian Settings → Files and links, set "Attachment folder path" to a fixed directory (e.g. `raw/assets/`). Then in Settings → Hotkeys, search for "Download" to find "Download attachments for current file" and bind it to a hotkey (e.g. Ctrl+Shift+D). After clipping an article, hit the hotkey and all images get downloaded to local disk. This is optional but useful — it lets the LLM view and reference images directly instead of relying on URLs that may break. Note that LLMs can't natively read markdown with inline images in one pass — the workaround is to have the LLM read the text first, then view some or all of the referenced images separately to gain additional context. It's a bit clunky but works well enough.
- **Obsidian's graph view** is the best way to see the shape of your wiki — what's connected to what, which pages are hubs, which are orphans.
- **Marp** is a markdown-based slide deck format. Obsidian has a plugin for it. Useful for generating presentations directly from wiki content.
- **Dataview** is an Obsidian plugin that runs queries over page frontmatter. If your LLM adds YAML frontmatter to wiki pages (tags, dates, source counts), Dataview can generate dynamic tables and lists.
- The wiki is just a git repo of markdown files. You get version history, branching, and collaboration for free.

## Why this works

The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero.

The human's job is to curate sources, direct the analysis, ask good questions, and think about what it all means. The LLM's job is everything else.

The idea is related in spirit to Vannevar Bush's Memex (1945) — a personal, curated knowledge store with associative trails between documents. Bush's vision was closer to this than to what the web became: private, actively curated, with the connections between documents as valuable as the documents themselves. The part he couldn't solve was who does the maintenance. The LLM handles that.


## Note

This first section of the document is intentionally abstract. It describes the idea, not a specific implementation. The exact directory structure, the schema conventions, the page formats, the tooling — all of that will depend on your domain, your preferences, and your LLM of choice. Everything mentioned above is optional and modular — pick what's useful, ignore what isn't. For example: your sources might be text-only, so you don't need image handling at all. Your wiki might be small enough that the index file is all you need, no search engine required. You might not care about slide decks and just want markdown pages. You might want a completely different set of output formats. The right way to use this is to share it with your LLM agent and work together to instantiate a version that fits your needs. The document's only job is to communicate the pattern. Your LLM can figure out the rest.

---

# Wiki Concrete Proposal

This section describes the implementation of the wiki we wish to follow:

## Purpose
This is the LLM's and architects internal knowledge base.

The LLM writes and maintains this wiki. The architecture team and the LLM read it.  The architect directs it, and supplies sources. Do not wait for instructions to cross-reference - always cross-reference proactively.

**At the start of every session:** display the contents of "README.md" (which should contain basic instructions such as the workflows to orient the user, then check `new/` for pending sources and report how many are waiting (if any).

**When the user asks for help or how the wiki works:** display "README.md" and elaborate if needed

## Directory Layout
../src		- <complete this description>
wikiroot/	- This folder, root of the wiki documentation
index.md	- content catalog (LLM-maintained)
Log.md		- chronological ingest/query/lint log (LLM-maintained)
principles.md	- the wiki pattern document (reference only, do not modify)
new/		- drop zone for pending source documents (not yet ingested)
raw/		- ingested source documents (immutable - read, never modify or delete
pages/	
  overview.md	- high-level architecture synthesis (always keep current)
  ubiquitous-language.md - domain glossary and ubiquitous language (always keep current)
  domain-model.md	- bounded contexts, domain map, key relationships
  lexer/
  parser/
  generator/
  targets/
  testing/
  syntax/	- syntax summary index and per-keyword/concept pages with examples
  decisions/	- Architecture Decision Records (ADRs)
  concepts/	- lanuguage/parser/lexer/target domain concepts

### Source document lifecycle

Raw sources may be markdown documents, source code from /src or /test, sample code, or text files, or a file containing web/github links

**To add a source for ingestion:** drop the file into `new/`. No other step needed - its presence there is the signal that it is pending.

**After ingestion:** move the file from `new/` to `raw/`. Files in `raw/` are permanent and immutable - never edited, never deleted. Wiki pages link to files in `raw/`.

**Ingest state is therefore derived from the filesystem:** 
- `new/` contains anything = there are pending sources 
- `raw/` contains everything already ingested

**Re-ingestion:** if a revised version of a document arrives in new/ with the same filename as a file already in
raw/, do not proceed automatically - flag it to the user and agree how to handle it before moving anything

## Page Frontmatter

Every page in pages/ must include this YAML frontmatter:
```yaml
---
title: <human-readable title>
category: concepts | lexer | parser | generator | overview | targets | testing | syntax | decisions
tags: []
sources: [] # filenames from raw/ that this page draws from
updated: YYYY-MM-DD
status: current | draft | stale
---
```

- tags should use kebab-case terms from the ubiquitous language where possible. 
- `sources` lists raw filenames (not paths) e.g. '[requirements.md]'.
- `status: stale` means the page likely needs updating from a newer source.


## Special Pages

### pages/overview.md"
High-Level synthesis of the entire transpiler and generation architecture. Updated on every ingest that touches system-level concerns. Should always be readable as a standalone orientation doc for a new architecture team member. Covers: platform topology, major domains, key architectural decisions, notable constraints.

### pages/ubiquitous-language.md
The canonical domain glossary. Every significant domain term encountered during ingest must be defined here. E ntries are alphabetically ordered. Format per entry:

```markdown
### <Term>
<Definition - precise, context-specific to this exchange, not generic>

**Domain:** <which bounded context this term primarily belongs to>
**See also:** [[<related terms>]]
```

When a term already exists and a new source uses it differently or adds nuance, update the definition and note the variation. This file is the source of truth for language. Never use a term in a wiki page without ensurin it is defined here.

### `pages/domain-model.md`
A living map of bounded contexts: what each context owns, its invariants, how it integrates with adjacent contexts. Updated when ingest reveals new domains or integration patterns. Use a Mermaid diagram to visualise context boundaries and relationships.


## Page Categories

### `concepts/`
Domain and technical concepts that need definition beyond the one-liner in ubiquitous-language.md. 

### `testing/`
Details of testing approach, test harness use and examples.

### `decisions/`
Architecture Decision Records. See ADR format below.

### <add more sections as appropriate>

---

## ADR Format

File naming: decisions/ADR-NNNN-short-title.md (zero-padded, e.g. ADR-0001-...). markdown

```markdown
---
title: "ADR-NNNN: <Title›" 
category: decision 
tags: []
sources: []
updated: YYYY-MM-DD
status: proposed | accepted | deprecated / superseded 
superseded-by: ""  # ADR number if status is superseded
---

# ADR-NNNN: <Title>

**Status:** Accepted | Proposed | Deprecated | Superseded by ADR-XXXX
**Date:** YYYY-MM-DD

## Context
<What is the architectural problem or decision point? What forces are at play?>

## Decision
<What was decided?>

## Consequences
<What becomes easier or harder as a result? Known trade-offs, risks, debt.>

## Alternatives Considered
<What else was evaluated and why was it rejected?>

---

## Workflows

### Ingest

**Before ingesting:** check whether the file already exists in 'raw/ under the same name. If it does, stop an d flag this to the user as a potential re-ingest - do not proceed until the user confirms how to handle it.

When told to ingest a source file (or all pending sources):
1.  Glob 'new/ to identify files to ingest (skip .gitkeep').
2.  Read each source in full.
3.  Briefly discuss key architectural takeaways with the user before writing anything (unless asked to ingest silently).
4.  Write or update a **summary page** for the source in the most appropriate category. Name it after the sourc e with a-summary suffix using kebab-case (e.g.spot-margin-requirements-summary.md').
5.  Update pages/ubiquitous-language.md with any new or refined terms.
6.  Update "pages/domain-model.md' if new domains, boundaries, or integration patterns are revealed.
7.  Update pages/overview.md" if system-level concerns are materially affected.
8.  Update or create any relevant system, product, domain, or concept pages touched by the source.
9.  Update 'index.md" with any new pages.
10. Move the file from new/ to raw
11. Append an entry to "log.md" recording the filename, pages created, and pages updated. When asked to **ingest anything pending**: glob new/ - every file found there is pending. Ingest each one in turn.

### Refinement TODOs

During ingest, the LLM may encounter ambiguities, incorrect terminology, contradictions between sources, or areas needing clarification. 
These should **not block ingest**.  Instead:
1. Note the issue inline on the relevant wiki page using this format:
> **TODO:** <description of the issue -what is ambiguous, contradictory, or incorrect, and which sources a re involved›
2. Append a TODO entry to "log.md" using the type todo': ## [YYYY-MM-DD] todo | <short title> ‹description of the issue and which pages/sources it affects>
4. Continue ingesting.

TODOs are reviewed later during a dedicated refinement pass or lint. The user may ask to review outstanding TO DOs at any time - the LLM should grep log.md for todo entries and grep wiki pages for > **TODO:*** blocks


### Query

When asked a question against the wiki:
1.  Read "index.md" to identify relevant pages.
2.  Read the relevant pages.
3.  Synthesise an answer with inline citations to page filenames.
4.  If the answer is non-trivial and reusable, ask the user if it should be filed as a new wiki page (a concept, analysis, or comparison page).


---

### Lint
When asked to lint the wiki:
**Content health:**
1.    Contradictions between pages.
2.    Claims likely superseded by newer sources (check sources frontmatter and log dates).
3.    Orphan pages - no inbound links from any other page.
4.    Important concepts mentioned but lacking their own page.
5.    Missing cross-references between obviously related pages.
6.    Pages with status: draft that are mature enough to promote.
7.    Stale pages - updated date significantly older than recent ingests on the same topic.

**Structural health:**
8. Overcrowded directories - any `pages/<category>/` with more than ~15 pages is a signal to consider sub-categories
(e.g. parser/errors/, parser/grammar/*).
9.  Miscategorised pages -pages that clearly belong in a different directory than where they sit.
10. Missing categories - a cluster of pages that share a cross-cutting concern not represented by any current directory (e.g. `modules/` emerging as its own domain). Propose new directories with a rationale.
11.  Redundant categories - directories with only 1-2 pages that would sit better elsewhere.
12.  "index.md" drift - entries that are missing, mis-described, or point to pages that no longer exist.
5.   Pending sources - glob new/ and report any files found there. These are pending ingest and should not be forgotten.

**Output:**
-   Produce a prioritised list of issues across both content and structure, grouped by type.
-   For any proposed structural change (new directory, rename, page move, describe the change and ask for appro val before executing - moving files has git history implications.
-   Suggest new questions to investigate or sources to seek out.
-   Ask user which issues to fix immediately vs. defer.

The folder structure is intentionally expected to evolve. Lint is the primary mechanism for proposing and agre eing structural changes.

---

## Cross-Referencing Conventions
- Link pages using [[filename-without-extension]] (Obsidian wiki-link style). 
- Every system page must link to its owning domain page.
- Every product page must link to the system pages it involves.
- Every concept that has its own page must be linked on first mention in any other page.
- Links are relative to the page
- Links may point to files in "raw" for more detail where appropriate
- Ubiquitous language terms should be bolded on first use in a page: **<term>**

---

## Index Format (index.md')
Organised by category. Each entry: - [[filename]] - one-line description'. Updated on every ingest.

---

## Log Format ("log.md")
Append-only. Each entry header: *## [YYYY-MM-DD] <type> | <title>
Types: "ingest", "query", "lint", "update", "todo".

Example:
```
## [2026-04-09] ingest 1 requirements.md
Summary of what was learned and which pages were created or updated.
```



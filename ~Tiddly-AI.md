Using TiddlyWiki as a UX/UI for LLM orchestration is *way* more powerful than “a wiki with an AI button”. It’s actually very close to a full **AI control plane**: you already have a hyperlinked, versioned, scriptable graph of little data objects (tiddlers)… which is exactly what orchestration frameworks try to reinvent.

I’ll walk through the possibilities in layers: **representation**, **orchestration patterns**, and **UX patterns**—all in TiddlyWiki terms.

---

## 1. Mental model: TiddlyWiki as the “control plane” for AI

Think of the roles like this:

* **LLMs / tools** = “data plane” (do the work: generate, transform, call APIs).
* **TiddlyWiki** = “control plane” (decides *what*, *when*, *how*, with which config, and how to store results).

The wiki gives you:

* **Tiddlers** → nodes in an orchestration graph (prompts, configs, runs, agents, pipelines, datasets).
* **Fields** → parameters, state, metrics.
* **Filters** → graph queries & routing rules.
* **Tags** → type system (Prompt, Agent, Pipeline, Test, Dataset, Run, Answer…).
* **Wikitext + widgets** → UIs for editing, visualizing and triggering workflows.

This is basically a programmable LLM IDE, but using the wiki itself as the IDE.

---

## 2. Representing AI concepts as wiki structures

### 2.1 Prompts as first‑class tiddlers

* Each prompt = one tiddler, with fields like:

  * `role: system|user|assistant`
  * `model`, `temperature`, `max_tokens`
  * `variables`: JSON describing expected variables (e.g., `topic`, `audience`).
* Prompt variants can be tiddlers linked by tags (`Prompt:ExplainConcept/v1`, `/v2`, etc.) so you can:

  * A/B test prompts.
  * Diff them like code.
  * Attach evaluation results (see below).

This makes **prompt engineering a wiki activity**: you edit, tag, link, and transclude prompts like any other content.

---

### 2.2 Agents as tiddlers

Define agents (personas, tools, behaviors) as a set of tiddlers:

* `Agent:DocHelper`

  * `systemPrompt: $:/prompts/DocHelperSystem`
  * `tools: [SearchWiki, SummarizeTiddler, LinkTiddlers]`
  * `profile: $:/ai/profile/cheap` or `profile: $:/ai/profile/premium`
* Each “tool” is another tiddler describing:

  * Name/description.
  * Input/output schema.
  * Implementation (maybe a Node.js module, or a TiddlyWiki filter).

Now an “agent orchestration” is literally a filter like:

```wikitext
[tags[Agent]tag[Active]]
```

and a conversation with an agent is just a tiddler that references:

* `agent`: `Agent:DocHelper`
* `messages`: a list of `AIMessage` tiddlers.

All this can be rendered as chat UI, and orchestrated using the same queue patterns you’ve been thinking about.

---

### 2.3 Pipelines & workflows as tiddlers

You can use TiddlyWiki to describe **multi-step orchestrations**:

* `Pipeline:ExplainTopicFromWikipedia`

  * `steps-json`: list of steps:

    * `importFromWikipedia`
    * `chunkArticle`
    * `summarizeChunks`
    * `mergeSummaries`
    * `generateQuiz`
  * Each step references:

    * A **prompt** tiddler (or agent).
    * A filter to select its inputs (tiddlers).
    * Where to write outputs (new tiddlers/fields).

Pipelines become:

* **Composable**: one pipeline step can be another pipeline.
* **Inspectable**: you can see which step produced which tiddler.
* **Versioned**: each pipeline definition is just content; you can snapshot, fork, revert.

This is where TiddlyWiki starts feeling like a textual **Node-RED for LLMs**, but all native to the wiki.

---

### 2.4 Datasets & evaluation specs as tiddlers

For orchestration, you eventually care about **quality**. Tiddlers are perfect for representing evaluation artifacts:

* `Dataset:BasicConcepts`

  * `items`: list of question/expected-answer pairs.
* `EvalSpec:ExplainConceptAccuracy`

  * `dataset: Dataset:BasicConcepts`
  * `pipeline: Pipeline:ExplainConcept`
  * `metric: BLEU|ROUGE|LLM-as-judge…`

Run an eval pipeline, and it spits out:

* `EvalRun:ExplainConcept/2025-11-23`

  * per-item scores (in JSON).
  * aggregate metrics.
  * links to the exact prompts/agents/models used.

Now the wiki becomes **the place you explore and compare** different LLM behaviors.

---

## 3. Orchestration patterns you can support cleanly

Here’s where it gets interesting from the orchestration side.

### 3.1 Sampling & ensembling as a built-in pattern

You already described:

> ask the same question 100 times then ask a series of questions of the answers

In TiddlyWiki terms:

* A **Sampling Step** is a pipeline step that:

  * Reads a `StandardQuestion` tiddler.
  * Spawns N `AIRun` tiddlers (each with its own variability).
  * Writes N `AIAnswer` tiddlers tagged with a `batch` id.

* A **Meta Step** is a step that:

  * Uses filters to gather those `AIAnswer` tiddlers.
  * Asks ranking / synthesis / disagreement questions.
  * Produces a final, canonical answer tiddler.

You can generalize that pattern into a “Sampling+Meta” pipeline that is reusable across many question types:

* Explanation tasks.
* Summarization tasks.
* Code generation tasks.

TiddlyWiki gives you:

* A way to *store all the intermediate answers* (for debugging & research).
* An easy way to visualize them (lists, comparisons, tables).
* A clean, structured place to attach *meta* artifacts (rankings, final synthesis).

---

### 3.2 Multi-step “reasoning” flows (externalized CoT)

Instead of relying on the model’s hidden chain-of-thought, you can orchestrate reasoning steps *explicitly* in the wiki:

1. Step 1: “List sub-questions required to answer this question.”
2. Step 2: “Answer each sub-question.”
3. Step 3: “Integrate the sub-answers into a final answer.”

Each step is a prompt tiddler; each result is a tiddler; the **reasoning graph is visible and editable**.

You get:

* Transparency: you can inspect and correct sub-steps.
* Reuse: sub-questions can be re-asked later with new context.
* Tools: you can mix in tool calls (e.g., search wiki) at specific steps.

---

### 3.3 Tool integration: use wiki and external APIs as tools

TiddlyWiki itself can be exposed to the LLM as a **toolbox**:

* Tool: “SearchWiki(filter)” → returns summarized matching tiddlers.
* Tool: “CreateTiddler(fields)” → creates a draft tiddler.
* Tool: “UpdateTiddler(title, patch)” → propose a patch tiddler for human review.

Externally:

* HTTP/Webhooks as tiddlers with endpoints.
* CLI commands as actions defined in `plugin.info` / Node modules.

From a UX perspective, you can have:

* Tool catalog panel: list tools & their descriptions (which are tiddlers).
* “Dry-run” mode: show what tools the agent *would* call, but don’t actually apply changes.
* “Staging area” tiddlers: LLM writes diffs/proposals; you manually approve/merge.

That’s powerful: TW becomes a **safe editor for AI‑proposed changes**, not just a passive storage.

---

### 3.4 Scheduling & background maintenance

TiddlyWiki can express many “background jobs” declaratively:

* “All tiddlers tagged `Docs` must have:

  * `ai:summary` updated weekly.
  * `ai:tags` updated whenever text changes.”
* “Rebuild embeddings for any tiddler whose text changed since last embedding.”

Each rule is just a config tiddler; the orchestration engine:

* Periodically filters for matching tiddlers.
* Enqueues operations (embedding, summarization, classification).

This makes **ongoing LLM maintenance work declarative** instead of scattered scripts.

---

### 3.5 Import & sync from Wikipedia (and beyond)

You mentioned “fleshing out the wiki from Wikipedia”. TW can orchestrate **multi-source ingestion**:

1. “SourceSpec” tiddlers define:

   * Where to pull from (Wikipedia, docs, APIs).
   * How often to refresh.
   * How to chunk and map into tiddlers.

2. Pipelines run:

   * Source fetch.
   * Chunking/summarization.
   * Link generation (e.g., detect references to existing topics).
   * Conflict detection (differences between wiki and source).

3. UI surfaces:

   * A “Knowledge Import” dashboard listing:

     * New topics.
     * Conflicts / outdated sections.
     * Suggested merges.

The LLM is the transformation engine; TiddlyWiki is the **transparent ETL spec & UI**.

---

## 4. UX patterns: how people actually touch all this

TiddlyWiki is **UI-by-content**, so the orchestration UI is just carefully designed tiddlers. Some patterns:

### 4.1 Topic-centric view

On each topic tiddler (e.g. `Topic: Hash function`), you can surface:

* “Canonical AI Answer” (final ensemble result).
* “Source: Wikipedia / Docs” section.
* “Open issues” section (meta questions that found disagreements).
* “Ask more” panel:

  * sub-question suggestions (from a meta step).
  * one-click spawn of new Standard Question instances.

This makes the **topic page** the hub of AI interactions, not a separate “AI page”.

---

### 4.2 Pipeline builder UI

A tiddler like `Pipeline:ExplainConceptFromWikipedia` can have:

* A table of steps (Step ID, Type, Prompt, Input filter, Output).
* Buttons to:

  * Run step-by-step.
  * Re-run only failed steps.
  * Compare outputs between pipeline versions.
* A visual graph (using a graph plugin) showing dependencies between steps.

You get a low-code, wiki-native **workflow editor** for LLM pipelines.

---

### 4.3 Queue / run dashboard

A “Runs” dashboard shows:

* Rows per `AIRun`:

  * Status, op, queue, model, cost, duration, links to question/answer.
* Filters & actions:

  * Cancel / requeue / inspect request & response.
  * Group by pipeline, agent, model.

This is essentially **LLM Ops UI**: observability, debugging, tuning—built from filters and transclusion.

---

### 4.4 Agent playgrounds

Per-agent “playground” tiddlers:

* Left side: conversation view with that agent.
* Right side: config + tools + quick toggles:

  * change model/profile.
  * enable/disable tools.
  * switch to “log everything” mode (for debugging).

Behind the scenes, conversation & messages are tiddlers, so you can:

* Clone a conversation as a test case.
* Use it in evaluation datasets.
* Share it via tiddler export.

---

### 4.5 Governance & explainability UIs

Because everything is stored as content:

* You can create **audit views**:

  * “Show all tiddlers modified by AI in the last week.”
  * “Show AI changes that touched fields X, Y, Z.”
* You can have a **“review queue”**:

  * LLM proposes patches as separate “Patch” tiddlers.
  * You see diffs, accept/reject, and log decisions.

This is the kind of interface people try to bolt onto external orchestration systems; in TW, it’s almost natural.

---

## 5. “Deep” possibilities: what’s uniquely *wiki-ish* here

A few things you get with TiddlyWiki that typical orchestration dashboards don’t:

1. **Hypertext-native orchestration**
   Pipelines, prompts, testcases, agents, and results are *all linkable both ways*. That’s huge for:

   * drilling down into why a result happened,
   * and tying everything back to source content.

2. **Local-first and forkable**
   Because a whole TW is a single file (or a small Node wiki), you can:

   * fork an entire orchestration environment.
   * run experiments in a branch.
   * diff configurations and results between “editions” of the same AI system.

3. **User-programmable at multiple levels**

   * Non-technical users: tweak prompts, tags, profiles via UI.
   * Power users: write macros & small JS modules.
   * Deep devs: integrate complex Node.js backends.

4. **Model-agnostic control plane**
   The wiki doesn’t care if the “AI provider” is OpenAI, local models, Anthropic, etc., as long as your provider layer is abstracted. The orchestration concepts—pipelines, runs, agents, datasets—stay the same.

5. **The LLM can help maintain its own control plane**
   Since the orchestration configuration is also just wiki content, you can have a “Meta-Engineer Agent” that:

   * suggests pipeline changes,
   * explains configs,
   * generates new Standard Questions or Evals from your existing content.

You still gate changes through human review, but you now have a system where:

> The wiki configures the AI, and the AI helps improve the wiki.

---

## 6. The design way forward

> **6.1 – The AI Orchestration Meta‑Model for TiddlyWiki**

**Design the “schema” for TiddlyWiki as an AI orchestration OS.**

---

## 6.1.1 Goals & constraints

Before we name any fields, we should be clear what this meta‑model must support:

1. **Everything is a tiddler**
   No external DB required. Every concept—prompt, run, answer, agent, pipeline—is representable as a tiddler plus fields/tags.

2. **Composable & inspectable**
   You can click into *any* step, see its config, its inputs, its outputs, and its lineage.

3. **Model‑agnostic**
   Should work with OpenAI, local LLMs, Anthropic, etc. The orchestration model must not lock you into one vendor.

4. **Safe evolution**
   You’ll want to add new fields/types later. So:

   * Use **namespaces** (e.g., `ai:*`).
   * Prefer “optional field” expansion over changing semantics of existing fields.

5. **Good UX hooks**
   The model should make it easy to build nice UI:

   * Dashboards via filters.
   * Detail views via transclusion.
   * Diff & audit views.

6. **Support your flagship patterns**
   Specifically:

   * Standard questions asked many times (sampling).
   * Meta‑questions over answers.
   * Import / structure from Wikipedia (or other sources).
   * “Obvious AI ops” declared via fields/tags.

That’s the bar.

---

## 6.1.2 Namespaces & naming conventions

To keep things tidy:

* **Tags as type labels**

  * `AIPrompt`, `AIQuestion`, `AIAnswer`, `AIRun`, `AIBatch`, `AIPipeline`, `AIPipelineStep`, `AIAgent`, `AITool`, `AIDataset`, `AIEvalSpec`, `AIEvalRun`, `AIImportSpec`, etc.

* **Field namespace**: `ai:*`
  Examples:

  * `ai:op`, `ai:input`, `ai:output`
  * `ai:profile`, `ai:scope`, `ai:when`
  * `ai:status`, `ai:lastHash`, `ai:lastProcessedAt`
  * `ai:batch`, `ai:step`, `ai:pipeline`
  * `ai:model`, `ai:temperature`

* **System tiddler prefix**: `$:/ai/...`

  * Configs, profiles, pipeline templates, system prompts, etc.
  * User content (topics, docs, questions) can live in normal names.

This lets you quickly filter for “all AI things” with `[prefix[$:/ai/]]` or `[tag[AIPrompt]]`.

---

## 6.1.3 Domain content: Topic, Document, Note

These are *not* AI‑specific, but the AI system needs to understand them:

### `Topic` tiddlers

**Purpose:** conceptual objects you care about (Hash functions, TiddlyWiki, etc.)

* **Tag**: `Topic`
* **Fields (suggested):**

  * `title`: e.g., `Topic: Hash function`
  * `text`: human description / notes
  * `wikipedia:title`: optional, e.g., `Hash function`
  * `ai:canonical-answer`: title of the canonical AI answer tiddler (final ensemble result)
  * `ai:status`: `stub | seeded | enriched`

**Example:**

```tid
title: Topic: Hash function
tags: Topic
wikipedia:title: Hash function
ai:status: stub
text: 
A hash function maps data of arbitrary size to fixed-size values...
```

---

### `Document` / normal tiddlers

Most of your wiki content is just “documents”. We don’t need a special type, but we can let AI annotate them:

* **Possible `ai:` fields on any tiddler:**

  * `ai:summary`: short summary text
  * `ai:tags`: comma‑separated tags suggested by AI
  * `ai:difficulty`: “beginner/intermediate/advanced”
  * `ai:wiki-score`: how close it is to Wikipedia or external sources (0–1)
  * `ai:lastHash`: hash of the text when these fields were last computed
  * `ai:lastProcessedAt`: ISO timestamp

This is the **“AI lenses”** concept: multiple AI‑produced overlays live as fields on the same tiddler.

---

## 6.1.4 Operation spec: Prompts, Standard Questions, Pipelines, Agents, Tools

These define *what to do*.

### 6.1.4.1 `AIPrompt` tiddlers

Atomic prompt building blocks.

* **Tag**: `AIPrompt`

* **Fields:**

  * `ai:role`: `system | user | assistant`
  * `ai:model`: optional default
  * `ai:temperature`, `ai:max-tokens`, etc.
  * `ai:variables`: JSON schema of variables (`topic`, `audience`, `style`…)

* **Body (`text`)**: the raw prompt, with variables in some interpolated form (e.g., `<<variable topic>>` or `${topic}`).

**Example:**

```tid
title: $:/ai/prompt/ExplainConcept
tags: AIPrompt
ai:role: user
ai:variables: {"topic":"string","audience":"string"}
text: 
Explain the concept of <<variable topic>> to a <<variable audience>>. 
Use clear structure and examples.
```

---

### 6.1.4.2 `AIStandardQuestion` tiddlers

These define your **canonical question types** (“Explain concept”, “Summarize doc”, “Critique code”), plus sampling/meta patterns.

* **Tag**: `AIStandardQuestion`
* **Fields:**

  * `ai:prompt`: title of base `AIPrompt`
  * `ai:samples`: default number of samples (e.g. `100`)
  * `ai:ensemble`: method (`rank-then-merge`, `vote`, `cluster`, etc.)
  * `ai:meta-steps`: JSON array of meta‑question specs (titles of `AIMetaQuestion` tiddlers)
  * `ai:profile`: default profile (model, cost envelope, etc.)
  * `ai:cost-cap-usd`: optional budget per question

**Example:**

```tid
title: $:/ai/standard/ExplainConcept
tags: AIStandardQuestion
ai:prompt: $:/ai/prompt/ExplainConcept
ai:samples: 50
ai:ensemble: rank-then-merge
ai:meta-steps: 
[ "$:/ai/meta/RankAnswers",
  "$:/ai/meta/FindDisagreements",
  "$:/ai/meta/FinalSynthesis" ]
ai:profile: $:/ai/profile/default
ai:cost-cap-usd: 1.50
```

---

### 6.1.4.3 `AIQuestionInstance` tiddlers

Concrete uses of a standard question, bound to a topic or document.

* **Tag**: `AIQuestion`

* **Fields:**

  * `ai:standard`: `$:/ai/standard/ExplainConcept` or similar
  * `ai:topic`: title of Topic or doc
  * `ai:status`: `draft | queued | sampling | meta | answered | failed`
  * `ai:batch`: title of associated `AIBatch` tiddler (sampling batch)
  * `ai:final-answer`: title of final `AIAnswer` tiddler
  * `ai:profile`: optional override of profile
  * `ai:vars`: JSON of variables (`{"topic":"Hash function","audience":"beginner"}`)

* **Body (`text`)**: human‑readable question, if needed.

**Example:**

```tid
title: Question: Explain Hash function
tags: AIQuestion
ai:standard: $:/ai/standard/ExplainConcept
ai:topic: Topic: Hash function
ai:status: draft
ai:vars: {"topic":"hash function","audience":"beginner"}
text: 
Explain what a hash function is for a beginner.
```

---

### 6.1.4.4 `AIPipeline` tiddlers

These describe **multi‑step workflows** (import from Wikipedia, then summarize, then generate quiz, etc.).

* **Tag**: `AIPipeline`
* **Fields:**

  * `ai:steps-json`: JSON array of step descriptors
  * `ai:profile`: default profile (overridable)
  * `ai:triggers`: optional filter string (“run this pipeline when…”)

**Step JSON shape (example):**

```json
[
  {"id": "sample", "type": "sample", "count": 50},
  {"id": "rank", "type": "meta", "spec": "$:/ai/meta/RankAnswers", "after": ["sample"]},
  {"id": "final", "type": "meta", "spec": "$:/ai/meta/FinalSynthesis", "after": ["rank"]}
]
```

Each step points to:

* A **spec** tiddler (`AIMetaQuestion`, `AIStandardQuestion`, or custom).
* Input selectors (filters).
* Output mapping (which tiddlers / fields to write).

---

### 6.1.4.5 `AIMetaQuestion` tiddlers

Meta‑questions that operate on sets of answers (rank, detect disagreement, synthesize, generate follow‑ups).

* **Tag**: `AIMetaQuestion`
* **Fields:**

  * `ai:op`: e.g., `meta-rank`, `meta-summarize`, `meta-cluster`
  * `ai:prompt`: base prompt tiddler for meta operations
  * `ai:inputs-filter`: default filter for selecting answers in a batch
  * `ai:output-field`: where to write the result (on batch or question tiddler)
  * `ai:profile`: optional profile override

---

### 6.1.4.6 `AIAgent` tiddlers

Agents are persistent configurations: persona + tools + triggers.

* **Tag**: `AIAgent`
* **Fields:**

  * `ai:persona-prompt`: title of system prompt tiddler
  * `ai:tools`: JSON array of tool titles
  * `ai:profile`: default profile
  * `ai:triggers-filter`: filter of tiddlers this agent should act on
  * `ai:actions`: JSON array of `ai:op` kinds this agent can perform
  * `ai:mode`: `manual | on-save | scheduled | continuous`

Agents orchestrate many questions/runs implicitly.

---

### 6.1.4.7 `AITool` tiddlers

Tools are what agents can call (RAG retrieval, editing, external HTTP calls, etc.).

* **Tag**: `AITool`
* **Fields:**

  * `ai:name`: human/LLM‑visible name
  * `ai:description`: what it does
  * `ai:input-schema`: JSON schema for tool input
  * `ai:output-schema`: JSON schema for tool output
  * `ai:impl`: implementation hook (could be name of a Node/js module)
  * `ai:kind`: `wiki-filter`, `http`, `shell`, etc.

The LLM sees the tool’s description & schema; the orchestrator calls the actual implementation.

---

## 6.1.5 Execution tracking: Runs, Batches, Answers, Conversations, Embeddings

These are the **runtime artifacts**.

### 6.1.5.1 `AIRun` tiddlers

Represents *one* call to a provider (OpenAI, local LLM, etc.).

* **Tag**: `AIRun`
* **Fields:**

  * `ai:status`: `queued | running | succeeded | failed | cancelled | blocked:*`
  * `ai:op`: `answer | summarize | embed | meta-rank | import-wiki | toolcall`, etc.
  * `ai:provider`: `openai | local | anthropic | ...`
  * `ai:model`: `gpt-4o`, `gpt-4o-mini`, etc.
  * `ai:queue`: queue name (`default`, `embeddings`, `batch`, etc.)
  * `ai:question`: title of `AIQuestion` tiddler (if any)
  * `ai:pipeline`: pipeline id (if any)
  * `ai:step`: pipeline step id
  * `ai:batch`: batch id (for sampling)
  * `ai:attempt`: numeric attempt count
  * `ai:max-attempts`
  * `ai:priority`
  * `ai:createdAt`, `ai:startedAt`, `ai:finishedAt`
  * `ai:leasedBy`, `ai:leaseUntil`
  * `ai:request`: JSON string of full provider payload (messages, system, tools, etc.)
  * `ai:response`: JSON string of raw provider result (optional; maybe truncated)
  * `ai:error`: last error text or code
  * `ai:hash`: hash of normalized request (for caching/idempotency)
  * `ai:cost-usd`, `ai:input-tokens`, `ai:output-tokens`, `ai:latency-ms`

This is your **log + observability core**.

---

### 6.1.5.2 `AIBatch` tiddlers

Represents a batch of runs (e.g., your “ask 100 times”).

* **Tag**: `AIBatch`
* **Fields:**

  * `ai:question`: associated `AIQuestion`
  * `ai:standard`: associated `AIStandardQuestion`
  * `ai:count`: target number of samples
  * `ai:completed-count`: number of succeeded runs
  * `ai:status`: `pending | sampling | meta | done | failed`
  * `ai:pipeline`: pipeline id launching follow‑up meta steps
  * `ai:final-answer`: final answer tiddler title
  * `ai:rankings`: JSON from meta ranking
  * `ai:disagreements`: text/JSON from disagreement analysis
  * `ai:budget-snapshot-usd`: budget at start

You can show progress: “37 / 100 samples complete, 2 meta steps remaining”.

---

### 6.1.5.3 `AIAnswer` tiddlers

Single answer (from one run) or final canonical answer.

* **Tag**: `AIAnswer`

* **Fields:**

  * `ai:question`: linked question
  * `ai:batch`: linked batch
  * `ai:from-run`: linked run
  * `ai:model`, `ai:provider`
  * `ai:score`: numeric quality score (from meta ranking, 0–10)
  * `ai:rank`: ordinal ranking
  * `ai:kind`: `sample | final | meta`
  * `ai:sources`: JSON array of source tiddler titles or URLs
  * `ai:flags`: JSON list of issues (e.g., “possible hallucination”, “incomplete”)

* **Body (`text`)**: human‑readable answer text.

You can then mark one answer per question as `ai:kind: final` and point `Topic.ai:canonical-answer` to it.

---

### 6.1.5.4 `AIConversation` & `AIMessage` tiddlers

For chat & multi‑turn flows.

**Conversation:**

* **Tag**: `AIConversation`
* **Fields:**

  * `ai:agent`: agent for this conversation
  * `ai:messages`: JSON array of message tiddler titles
  * `ai:status`: `active | archived`
  * `ai:topic`: optional link to Topic
  * `ai:profile`: model profile override

**Message:**

* **Tag**: `AIMessage`
* **Fields:**

  * `ai:role`: `user | assistant | system | tool`
  * `ai:conversation`: conversation id
  * `ai:from-run`: run id (for assistant/tool messages)
  * `ai:tool-name`: if `role: tool`
  * `ai:createdAt`
* **Body (`text`)**: content of the message (or JSON in `ai:json` if structured).

This makes chat flows fully **inspectable and re-usable**.

---

### 6.1.5.5 Embeddings

You can store embeddings directly on tiddlers, or use separate tiddlers.

**On‑tiddler fields:**

* `ai:embedding`: JSON array (or pointer to a tiddler that contains it)
* `ai:embedding-model`
* `ai:embedding-version`
* `ai:embedding-lastHash`

Or separate `AIEmbedding` tiddlers (for advanced setups):

* `title: $:/ai/embedding/<hash>`
* `tags: AIEmbedding`
* `ai:for`: tiddler title
* `ai:vector`: JSON
* etc.

For most use cases, a field on the main tiddler is enough.

---

## 6.1.6 Evaluation & metrics: Datasets, EvalSpecs, EvalRuns

You’ll want to evaluate different prompts/pipelines/agents.

### 6.1.6.1 `AIDataset` tiddlers

Describe datasets of test cases.

* **Tag**: `AIDataset`
* **Fields:**

  * `ai:items`: JSON array of item tiddler titles
  * `ai:domain`: text (docs, code, Q&A, etc.)

Each **item** is just a tiddler with fields like:

* `Question`, `ExpectedAnswer`, `Context`…

---

### 6.1.6.2 `AIEvalSpec` tiddlers

Describe how to evaluate something.

* **Tag**: `AIEvalSpec`
* **Fields:**

  * `ai:dataset`: dataset tiddler
  * `ai:pipeline`: pipeline to run on each item
  * `ai:metric`: `llm-judge`, `exact-match`, `rouge`, etc.
  * `ai:judge-prompt`: for LLM‑as‑judge metrics
  * `ai:profile`: profile for evaluation runs
  * `ai:sample-count`: optional; run multiple samples per item

---

### 6.1.6.3 `AIEvalRun` tiddlers

One evaluation run over a dataset + pipeline.

* **Tag**: `AIEvalRun`
* **Fields:**

  * `ai:spec`: eval spec
  * `ai:status`: `queued | running | done | failed`
  * `ai:startedAt`, `ai:finishedAt`
  * `ai:metrics`: JSON of aggregate metrics
  * `ai:item-results`: JSON array linking items to their scores and run ids

This is your “model/prompt leaderboard” state.

---

## 6.1.7 Control & governance: Profiles, Queues, Budget, Policies

These tiddlers control **how** the orchestrator behaves globally.

### 6.1.7.1 `AIProfile` tiddlers

Model & safety presets.

* **Tag**: `AIProfile`
* **Fields:**

  * `ai:model`: provider model name
  * `ai:provider`: `openai` etc.
  * `ai:temperature`, `ai:top_p`
  * `ai:max-tokens`
  * `ai:timeout-sec`
  * `ai:cost-cap-usd-per-run`
  * `ai:cost-cap-usd-per-day`
  * `ai:tools`: default tool list
  * `ai:system-prompt`: optional global system prompt for this profile

Examples:

* `$:/ai/profile/cheap`
* `$:/ai/profile/quality`
* `$:/ai/profile/local`

---

### 6.1.7.2 `AIQueue` tiddlers

Describe queues and policies.

* **Tag**: `AIQueue`
* **Fields:**

  * `ai:concurrency`: max concurrent runs
  * `ai:rate-per-minute`
  * `ai:burst`
  * `ai:models-allowed`: JSON array of allowed models
  * `ai:budget-cap-usd-per-day`
  * `ai:priority-default`

Examples:

```tid
title: $:/ai/queue/default
tags: AIQueue
ai:concurrency: 2
ai:rate-per-minute: 60
ai:models-allowed: ["gpt-4o","gpt-4o-mini"]
ai:budget-cap-usd-per-day: 10
```

Embeddings queue, batch queue, etc. are just more `AIQueue` tiddlers.

---

### 6.1.7.3 Usage / budget tiddlers

Monthly ledger, e.g.:

* `title: $:/ai/usage/2025-11`
* **Fields:**

  * `ai:total-usd`
  * `ai:runs-count`
  * `ai:by-model`, `ai:by-queue` (JSON)
  * `ai:tokens-in`, `ai:tokens-out`

Feeds a Control Panel view: “$8.23 / $50 budget used this month”.

---

### 6.1.7.4 Policies & consent

* `$:/config/openai/consent` (simple yes/no)
* `$:/ai/policy/private-filter` etc.

You can also define:

* `AIPolicy` tiddlers specifying:

  * What content may be sent out.
  * Which tags or fields are always excluded from prompts.
  * Which operations require explicit user confirmation.

These can be consulted by the orchestrator before creating `AIRun` tiddlers.

---

## 6.1.8 Cross-cutting invariants & best practices

To keep this all sane:

1. **Types via tags, semantics via fields**

   * Use tags like `AIQuestion`, `AIRun`, etc. for type detection.
   * Don’t encode types in titles beyond helpful naming.

2. **`ai:` fields are optional, not brittle**

   * Orchestrator should check for existence (`has[ai:op]`, etc.).
   * Missing fields should mean “use defaults,” not “crash”.

3. **Lineage is explicit**

   * Every run knows its question, pipeline, batch, step.
   * Every answer knows its run, question, batch.
   * Every batch knows its question & standard spec.

   This is what lets you click from result → cause → config → source content.

4. **Hashing for idempotency**

   * `ai:hash` on runs, `ai:lastHash` on content.
   * Only reprocess if input hash changes or user forces.

5. **Separation of config vs data**

   * Config tiddlers live under `$:/ai/...`.
   * User content (topics, docs, questions) live normally.
   * That keeps it easy to export/import configs or share “orchestration kits”.

6. **Scaling down is easy**
   You can start with:

   * `AIQuestion`, `AIAnswer`, `AIRun`, `AIProfile`
     and later add `AIBatch`, `AIPipeline`, `AIAgent`, `AIEval*` as you need more sophistication. The naming scheme allows that growth.

---

## 6.1.9 How this unlocks the possibilities

With this meta‑model in place, you now have:

* A **vocabulary** for everything you want:

  * Standard questions, multi-sampling, meta‑analysis, Wikipedia import, agents, pipelines, eval, budgets.

* A **clean separation**:

  * Config & orchestration logic live in `$:/ai/...`.
  * User content & topics remain “normal tiddlers” with AI overlays.

* A **uniform way** to explore:

  * You can say: “Show me all `AIRun` for `AIQuestion` X”.
  * Or: “Show me all `Topic` whose `ai:canonical-answer` is missing.”
  * Or: “Show me all `AIPipeline` that use Wikipedia import.”

Everything about orchestration becomes **concrete, inspectable wiki structure**. That’s why 6.1 really *does* define the possibilities: once these shapes are stable, you can hang any amount of clever behavior and UI on top and it won’t collapse.







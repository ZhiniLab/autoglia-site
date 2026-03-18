# Autoglia Product Concept: Utility-Weighted Memory as Apparent Intelligence

## Core Product Thesis

The goal is not merely to make an AI remember more.

The goal is to make memory contribute to **apparent intelligence** in a way the user can actually notice.

That means Autoglia should not just store prior context. It should learn:

- what information is accurate
- what information is useful
- what information stays relevant over time
- what retrieval/action pathways repeatedly produce better outcomes

The product becomes stronger when memory helps the system:

- ask fewer repeat questions
- retrieve the right context earlier
- avoid prior mistakes
- adapt to recurring user workflows
- distinguish stable knowledge from stale or uncertain knowledge

So the real product is not just memory.

It is a **continuously trained selection system** for:

1. what to remember
2. what to trust
3. what to retrieve
4. what to do with retrieved memory

---

## The Key Shift

Most memory systems work like this:

1. save note
2. retrieve by similarity
3. inject into context

That gives persistence, but not much visible intelligence.

Autoglia should work like this:

1. save memory
2. track when it is used
3. track whether using it improved the result
4. promote memories that repeatedly help
5. demote memories that mislead, waste context, or go stale
6. learn which **pathways** tend to work in each context

This creates the user-visible effect of intelligence:

- “it remembers what matters”
- “it stops making the same mistakes”
- “it reaches for the right context faster”
- “it seems to know how I work”

---

## Memory Should Not Have One Score

A single “truth score” is too narrow.

Each memory item should likely have **multiple evaluative dimensions**.

### 1. Confidence / Reliability
How likely the memory is accurate.

### 2. Utility
How often using this memory helps produce a better result.

### 3. Activation Strength
How often and how recently the memory is actually relevant.

### 4. Stability
Whether this memory tends to remain true over time.

This is the difference between “stored memory” and “working memory that improves performance.”

---

## A Better Framing Than “Truth”

“Truth score” is rhetorically strong, but technically loaded.

In practice, the system is not measuring metaphysical truth. It is estimating:

- reliability
- validatedness
- trustworthiness
- practical usefulness
- temporal stability

So internally or in product design, the more precise framing is:

- **confidence score**
- **utility score**
- **stability score**
- **activation score**

“Truth score” can remain a manifesto phrase if desired, but the product should operate with more specific dimensions.

---

## The Product Principle

A memory should gain importance not merely because it exists, but because:

- it has been used
- it has been useful
- it has not been corrected
- it still appears current
- it contributes to successful outcomes

That means:

> trust this memory not only because it may be accurate, but because using it has repeatedly helped

This is the bridge from memory to perceived intelligence.

---

## Two Kinds of Memory

Autoglia should distinguish between at least two broad classes of memory.

### Declarative Memory
Things the system knows **about the user, project, or world**.

Examples:

- user prefers concise answers
- active project is Autoglia
- user prefers one-time pricing models
- migration failed on a specific table
- competitor pricing was reviewed

### Procedural Memory
Things the system learns **about what tends to work** in a given context.

Examples:

- when discussing pricing, retrieve competitor context first
- when debugging, retrieve stack + prior failures + working directory
- when writing X posts, retrieve tone preferences + product wedge
- when doing research, use recent sources and keep conclusion tight

Procedural memory is where much of the apparent intelligence comes from.

---

## Memory Items vs Usage Events

The product should not only store memory objects. It should also store the **use of memory**.

### A. Memory Items
Durable units of remembered information.

Examples:

- user dislikes verbosity
- product is Autoglia
- user prefers practical answers
- project uses SQLite memory
- prior migration failed in table X

### B. Usage Events
Records of how memory was used in real interactions.

Examples:

- memory X retrieved in session Y
- memory X injected into answer Z
- user accepted answer
- user corrected answer
- task succeeded after retrieval
- memory was retrieved but irrelevant

Without usage events, memory is static.

With usage events, memory becomes learnable.

---

## The Real Leap: Pathways

The strongest product idea is not only “trusted memory.”

It is **trusted pathways**.

A pathway is:

> a repeatable sequence of memory selection + action that tends to produce good outcomes in a certain context

Examples:

### Pricing Pathway
Trigger:
- user asks about pricing

Successful pathway:
- retrieve competitor benchmarks
- retrieve prior user pricing preferences
- retrieve current product constraints
- generate concise strategy response

### Debugging Pathway
Trigger:
- user pastes error or asks about code failure

Successful pathway:
- retrieve stack
- retrieve prior migration history
- retrieve environment/working directory conventions
- propose fix with concrete commands

### Writing Pathway
Trigger:
- user asks for X post or ad copy

Successful pathway:
- retrieve tone preferences
- retrieve product wedge
- retrieve recent objections and framing
- generate concise draft

This is how the system begins to feel like it “knows how to handle this kind of task.”

---

## What “Useful” Actually Means

A memory is not useful merely because it was retrieved.

It is useful when using it improves the interaction.

That requires reward signals.

### Strong Positive Signals
- user explicitly confirms correctness
- user says “yes,” “exactly,” or equivalent
- task completes successfully
- user does not need to re-explain context
- same memory/pathway helps in future sessions
- answer is faster/cleaner and still accepted

### Weak Positive Signals
- user continues productively
- memory appears in answer and is not corrected
- retrieved memory contributes to successful tool use

### Negative Signals
- user corrects the assistant
- assistant asks a question that should have been remembered
- memory retrieved was irrelevant
- memory caused wrong assumption
- prompt became bloated without benefit
- outdated memory was used as if current

This should drive utility learning.

---

## The Four Main Scores Per Memory

Each memory item should have separate scores.

### 1. Confidence Score
How likely the memory is accurate.

Driven by:
- verifications
- corrections
- source quality
- contradiction handling

### 2. Utility Score
How often using this memory improves outcomes.

Driven by:
- successful retrievals
- task completion
- acceptance signals
- correction avoidance

### 3. Access / Activation Score
How often and how recently the memory is relevant.

Driven by:
- frequency of retrieval
- recency of retrieval
- recurrence across sessions/projects

### 4. Stability Score
How likely the memory remains valid over time.

Driven by:
- memory type
- age
- revalidation history
- supersession
- contradiction or drift

These should be combined at retrieval time, not collapsed prematurely.

---

## Suggested Memory Types

Not all memory should be treated equally. Use explicit types.

- **fact** — durable claim
- **preference** — user likes/dislikes
- **decision** — what was chosen and why
- **task** — open loops / next steps
- **entity** — people, projects, products
- **pattern** — recurring workflow or behavior
- **failure** — what did not work
- **evidence** — source material supporting another memory
- **inference** — assistant-derived conclusion not yet confirmed

Different types should have different rules for promotion, decay, and ranking.

Examples:

- preferences may gain utility quickly
- tasks should decay quickly if not refreshed
- failures are high-value even if only seen once
- facts need stronger verification
- inferences should stay lower-confidence until confirmed

---

## Time Matters: Staleness and Decay

A major requirement is **memory staleness handling**.

Not all memories should get stronger forever.

Examples:

- user preference may be stable
- project status may change rapidly
- external facts may become stale
- plans may be superseded
- old assumptions may become wrong

The system should therefore support:

- `last_verified_at`
- `stale_after_days`
- `requires_revalidation`
- `superseded_by`
- `decay_rate`
- memory-type-specific freshness rules

Without this, old verified memories may become confidently wrong.

---

## Product Architecture

## Layer 1: Typed Memory Store

A schema for durable memory items.

### Example table: `memory_items`

```sql
CREATE TABLE memory_items (
    id INTEGER PRIMARY KEY,
    content TEXT NOT NULL,
    memory_type TEXT NOT NULL,          -- fact, preference, decision, task, pattern, failure, inference
    scope TEXT NOT NULL,                -- global, project, session, user, workflow
    source_type TEXT NOT NULL,          -- user, inferred, external, verified
    source_ref TEXT,                    -- optional provenance pointer
    confidence_score REAL DEFAULT 0.5,
    utility_score REAL DEFAULT 0.5,
    activation_score REAL DEFAULT 0.0,
    stability_score REAL DEFAULT 0.5,
    use_count INTEGER DEFAULT 0,
    success_count INTEGER DEFAULT 0,
    correction_count INTEGER DEFAULT 0,
    verification_count INTEGER DEFAULT 0,
    contradiction_count INTEGER DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_used_at DATETIME,
    last_verified_at DATETIME,
    stale_after_days INTEGER,
    requires_revalidation INTEGER DEFAULT 0,
    superseded_by INTEGER,
    status TEXT DEFAULT 'active'        -- active, demoted, archived, superseded
);

Layer 2: Usage Event Logging

Every retrieval and use of memory should create an event.

Example table: memory_usage_events
CREATE TABLE memory_usage_events (
    id INTEGER PRIMARY KEY,
    memory_id INTEGER NOT NULL REFERENCES memory_items(id),
    session_id TEXT,
    context_hash TEXT,
    trigger_type TEXT,                  -- user_question, tool_call, planning, debugging, writing
    action_type TEXT,                   -- retrieved, injected, cited, ignored, contradicted
    was_injected INTEGER DEFAULT 0,
    was_referenced_in_answer INTEGER DEFAULT 0,
    user_feedback TEXT,                 -- confirmed, corrected, neutral, implicit_success
    outcome_score REAL DEFAULT 0.0,     -- -1.0 to +1.0
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

This is the engine of utility learning.

Layer 3: Pathway Learning

A store for successful bundles of memory + action.

Example table: reasoning_pathways
CREATE TABLE reasoning_pathways (
    id INTEGER PRIMARY KEY,
    trigger_signature TEXT NOT NULL,    -- e.g. "context=debugging"
    memory_bundle_signature TEXT NOT NULL,
    action_type TEXT NOT NULL,          -- propose_fix, compare_competitors, write_post, summarize_research
    success_count INTEGER DEFAULT 0,
    failure_count INTEGER DEFAULT 0,
    avg_outcome_score REAL DEFAULT 0.0,
    avg_latency_ms REAL,
    last_used_at DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
Example table: pathway_events
CREATE TABLE pathway_events (
    id INTEGER PRIMARY KEY,
    pathway_id INTEGER NOT NULL REFERENCES reasoning_pathways(id),
    session_id TEXT,
    context_hash TEXT,
    outcome_score REAL DEFAULT 0.0,
    user_feedback TEXT,
    latency_ms INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

This lets the system learn not only useful memories, but useful routes.

Optional Layer 4: Contradictions and Supersession

A simple contradiction map helps prevent stale certainty.

Example table: memory_links
CREATE TABLE memory_links (
    id INTEGER PRIMARY KEY,
    source_memory_id INTEGER NOT NULL REFERENCES memory_items(id),
    target_memory_id INTEGER NOT NULL REFERENCES memory_items(id),
    link_type TEXT NOT NULL,            -- supports, contradicts, supersedes, related_to, derived_from
    weight REAL DEFAULT 1.0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

This allows memory to become a structured knowledge layer rather than a flat note pile.

Retrieval Should Be Staged, Not Flat

The system should not do one undifferentiated semantic search across all memory.

It should retrieve in stages.

Stage 1: Detect Context

Classify the current interaction.

Examples:

debugging

planning

writing

research

product strategy

casual

Stage 2: Activate Likely Pathways

Check which pathways historically worked well in this context.

Stage 3: Retrieve Candidate Memories

Pull candidate memory items based on:

relevance

utility

confidence

activation

stability

scope match

Stage 4: Assemble a Structured Memory Bundle

Not just “top-k chunks.”

Construct a bundle such as:

relevant user preferences

active project facts

recent decisions

open tasks

prior failures

relevant constraints

Stage 5: Evaluate Outcome

After the response:

was it accepted?

was it corrected?

did it complete the task?

did the pathway help?

should memory/pathway be strengthened or weakened?

This is much stronger than naive vector retrieval.

Ranking Model

A practical first-pass retrieval score:

final_rank = (
    0.35 * relevance +
    0.25 * utility_score +
    0.15 * confidence_score +
    0.15 * stability_or_freshness_score +
    0.10 * activation_score
)

This avoids over-rewarding pure similarity.

Pathway Value Function

A first-pass scoring model for pathways:

pathway_value = (
    success_count / (success_count + failure_count + 1)
) * trigger_match_score * recency_weight

This gives preference to pathways that repeatedly produce good results in matching contexts.

Utility Update Function

A practical simple updater:

def update_utility(memory, outcome):
    # outcome in range [-1.0, +1.0]
    alpha = 0.2
    memory.utility_score = (1 - alpha) * memory.utility_score + alpha * outcome
    memory.use_count += 1

    if outcome > 0:
        memory.success_count += 1
    elif outcome < 0:
        memory.correction_count += 1

    memory.last_used_at = now()

This is simple, interpretable, and testable.

Confidence Update Options

A simple count-based approach works first.

Simple formulation
confidence_score = (verifications - corrections) / (verifications + corrections + 1)

But a more principled version uses Bayesian updating.

Bayesian-style update
def update_belief(prior, evidence_strength, was_correct):
    if was_correct:
        posterior = (prior * evidence_strength + 1) / (evidence_strength + 1)
    else:
        posterior = (prior * evidence_strength) / (evidence_strength + 1)
    return posterior

This better distinguishes many weak confirmations from a strong correction.

Promotion, Demotion, Archive

Memory should not accumulate forever at equal weight.

Promote Memory If

used successfully 3+ times

corrected 0 times

relevant across multiple sessions

contributes to successful pathways

remains current

Demote Memory If

repeatedly retrieved but ignored

leads to user correction

appears stale

causes wrong assumptions

has been superseded

Archive Memory If

old and low-utility

session-local and never reused

contradicted by newer verified memory

no longer relevant to active scopes

This gives memory hierarchy and prevents clutter.

The User-Visible Intelligence Gains

The user should notice Autoglia becoming smarter in concrete ways.

1. Fewer Repeat Questions

Because preferences, project state, and stable decisions are promoted.

2. Faster Access to Relevant Context

Because high-value memory bundles and pathways fire earlier.

3. Better Continuity

Because the system remembers not just facts, but why those facts mattered.

4. Fewer Repeated Mistakes

Because corrected memories and bad pathways get demoted.

5. Better Workflow Adaptation

Because procedural memory learns what tends to help in each type of task.

That is the real benchmark: not abstract intelligence, but better interaction quality over time.

Good Product Features This Unlocks
“Why I Used This”

A small explanation layer:

used because it helped in 4 similar sessions

last confirmed 2 days ago

relevant to your active project

previously successful in this workflow

This increases legibility and trust.

Promoted Memory

Show which memories have become core:

recurring user preferences

stable project assumptions

repeated constraints

frequently successful context

Stale Memory Warning

A critical feature:

“This may be outdated”

“Last verified 45 days ago”

“Superseded by newer project decision”

Useful Workflow Suggestions

Only after sufficient evidence:

“In pricing discussions, competitor comparison usually helps first”

“In debugging, migration history is usually relevant”

Important Warning: Do Not Reward Mere Repetition

A pathway used often is not necessarily a good pathway.

It may simply be habitual.

So pathway or memory promotion should depend on:

repetition

plus good outcomes

minus corrections

minus token/context waste

minus staleness

Otherwise the system may overfit to routine rather than value.

The Strongest Product Principle

The real product is not a note-taking layer.

It is:

a continuously trained system for selecting what matters, retrieving what helps, and suppressing what misleads

That is how memory becomes a contributor to apparent intelligence.

Recommended Implementation Sequence
Phase 1: Utility-Aware Memory Items

Add to each memory:

use_count

success_count

correction_count

last_used_at

utility_score

stability_score

Update retrieval ranking to include utility and stability.

Why first

This already improves behavior without a large architecture jump.

Phase 2: Pathway Learning

Introduce pathway signatures such as:

context=debugging → [stack, prior_failures, env_paths] → propose_fix

context=pricing → [competitors, user_preferences, product_constraints] → compare_options

Track which bundles and actions actually work.

Why second

This is probably the biggest practical leap in apparent intelligence.

Phase 3: Contradiction + Staleness Handling

Add:

superseded_by

last_verified_at

stale_after_days

contradiction links

type-specific freshness rules

Why third

This prevents the system from becoming confidently wrong.

Phase 4: Proactive Preparation

Only after the above works well:

preload likely memory bundles

suggest next likely actions

pre-fetch likely relevant context

offer proactive reminders or continuity prompts

Why later

Done too early, this feels invasive or speculative. Done later, it feels intelligent.

What To De-Emphasize Early

These ideas are interesting but should remain secondary in the initial product:

Markov chain prediction

reinforcement learning framing

generalized behavior surveillance

full chain-of-thought storage

aggressive pre-fetching

anticipatory agent orchestration

They may become valuable later, but the strongest early product remains:

reliability-weighted, utility-aware, self-correcting memory

A Better Framing for Reasoning Memory

Rather than “chain-of-thought memory,” a more practical early version is:

Decision Memory

Store:

what was decided

why it was chosen

alternatives considered

whether it later proved correct

whether it was reversed

This captures much of the value of reasoning history without the instability and noise of raw thought traces.

Final Product Thesis

Autoglia should not aim to become “a system that remembers everything.”

It should become:

a memory layer that learns what is reliable, what is useful, what is stale, and what patterns of retrieval and action repeatedly help

That is the route by which memory becomes part of the user’s perception that the system is getting smarter.

The deepest shift is this:

not memory as storage

not memory as recall

but memory as selection, weighting, and adaptive use

That is where the increase in intelligence becomes visible.

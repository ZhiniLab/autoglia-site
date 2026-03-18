# Autoglia Schema + Retrieval Engine Implementation Spec

## Purpose

This document turns the product concept into an implementation plan.

The goal is not merely to store memory, but to build a system that:

- remembers durable context
- tracks which memories are actually useful
- learns which retrieval/action pathways produce good outcomes
- reduces repeated mistakes
- makes the agent appear more intelligent through better continuity, selection, and adaptation

The core principle is:

> A memory should gain weight not only because it exists, but because using it repeatedly produces useful outcomes.

---

# 1. Product Objective

Most memory systems do this:

1. store text
2. retrieve by similarity
3. inject into prompt

That creates persistence, but not much visible intelligence.

Autoglia should instead do this:

1. store typed memory
2. track retrieval and usage events
3. measure whether using a memory helped
4. learn which memory bundles and pathways work best in a context
5. promote useful memories and demote misleading or stale ones

This creates user-visible improvement:

- fewer repeated questions
- better continuity across sessions
- more relevant context selected earlier
- fewer repeated mistakes
- stronger adaptation to user workflow

---

# 2. Core Design Principles

## 2.1 Separate memory from usage

A memory item is not enough.

The system must also track:

- when the memory was retrieved
- whether it was used
- whether it helped
- whether it caused correction
- whether it supported a successful pathway

Without usage tracking, memory is static.

## 2.2 Separate truth from utility

A memory can be:

- accurate but rarely useful
- useful but not a universal fact
- stable or unstable over time

So each memory should be evaluated along multiple axes.

## 2.3 Learn pathways, not just facts

A pathway is a repeatable pattern:

> In context A, retrieving memories of type B before action C tends to improve outcomes.

This is where apparent intelligence starts to emerge.

## 2.4 Memory must decay, not only accumulate

Old memories can become misleading.

The system needs:

- staleness detection
- supersession
- decay rules by memory type
- contradiction handling

---

# 3. Conceptual Model

The system has four main layers:

## Layer 1: Memory objects

Durable memory units such as:

- user preference
- project fact
- decision
- unresolved task
- prior failure
- verified external fact

## Layer 2: Usage events

Records of how memory was used:

- retrieved
- injected into prompt
- referenced in output
- corrected by user
- contributed to successful completion

## Layer 3: Pathways

Repeatable retrieval/action patterns such as:

- pricing discussion → retrieve competitor benchmarks + user pricing preferences
- debugging request → retrieve stack info + environment paths + prior failures
- writing request → retrieve tone preferences + recent product framing

## Layer 4: Policy and scoring

Logic for:

- promotion
- demotion
- staleness
- ranking
- conflict handling
- decay

---

# 4. Memory Taxonomy

Memory should be typed. Different types should behave differently.

## 4.1 Recommended memory types

- `fact` — declarative statement expected to be true
- `preference` — user preference or communication style
- `decision` — a choice made and optionally why
- `task` — open loop, next step, or action item
- `entity` — person, company, product, project, place
- `pattern` — recurring workflow or tendency
- `failure` — what did not work
- `evidence` — supporting source or verification artifact
- `inference` — agent conclusion not yet verified
- `constraint` — rules, limitations, boundaries
- `goal` — objective or target state
- `workflow` — preferred sequence or operating method

## 4.2 Scope levels

Each memory should also carry scope:

- `global`
- `user`
- `project`
- `session`
- `task`
- `entity`

Examples:

- “User dislikes verbosity” → `user`
- “ScholarBitBook uses Neon Postgres” → `project`
- “Current session is about launch pricing” → `session`
- “Need to fix migration 0059” → `task`

---

# 5. Scoring Model

Do not collapse all evaluation into one score.

Use multiple dimensions.

## 5.1 Per-memory score dimensions

Each memory item should have at least:

- `confidence_score` — how likely the content is accurate
- `utility_score` — how often using it improves outcomes
- `activation_score` — frequency + recency of use
- `stability_score` — how likely it remains true over time
- `relevance_score` — computed dynamically at retrieval time
- `cost_score` — token or complexity cost when included
- `correction_risk` — historical rate of correction when used

## 5.2 Per-pathway score dimensions

Each pathway should have:

- `success_rate`
- `avg_outcome_score`
- `trigger_match_score`
- `recency_weight`
- `latency_benefit`
- `correction_rate`
- `token_efficiency`

## 5.3 Why multiple scores matter

Examples:

- A stable user preference may have high utility even if it is not “factual.”
- A project fact may have high confidence but low recent usefulness.
- A workflow pathway may be high value even if individual memories within it are only moderately ranked.

This is a learned selection system, not a single-score truth engine.

---

# 6. Database Schema

Below is a proposed relational schema for SQLite.

---

## 6.1 `memory_items`

Stores durable memories.

```sql
CREATE TABLE memory_items (
    id INTEGER PRIMARY KEY,
    content TEXT NOT NULL,
    normalized_content TEXT,
    memory_type TEXT NOT NULL,
    scope TEXT NOT NULL,
    scope_ref TEXT,
    source_type TEXT NOT NULL,            -- user | tool | external | inferred | system
    source_ref TEXT,                      -- URL, message ID, tool result, etc.
    status TEXT DEFAULT 'active',         -- active | archived | superseded | disputed | stale
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    first_observed_at DATETIME,
    last_observed_at DATETIME,
    last_used_at DATETIME,
    last_verified_at DATETIME,
    stale_after_days INTEGER,
    superseded_by INTEGER,
    confidence_score REAL DEFAULT 0.5,
    utility_score REAL DEFAULT 0.0,
    activation_score REAL DEFAULT 0.0,
    stability_score REAL DEFAULT 0.5,
    cost_score REAL DEFAULT 0.0,
    correction_risk REAL DEFAULT 0.0,
    use_count INTEGER DEFAULT 0,
    success_count INTEGER DEFAULT 0,
    correction_count INTEGER DEFAULT 0,
    verification_count INTEGER DEFAULT 0,
    contradiction_count INTEGER DEFAULT 0,
    decay_rate REAL DEFAULT 0.0,
    metadata_json TEXT,
    FOREIGN KEY (superseded_by) REFERENCES memory_items(id)
);
6.2 memory_edges

Stores relationships between memory items.

CREATE TABLE memory_edges (
    id INTEGER PRIMARY KEY,
    source_memory_id INTEGER NOT NULL,
    target_memory_id INTEGER NOT NULL,
    edge_type TEXT NOT NULL,              -- supports | contradicts | leads_to | about | supersedes | related_to
    weight REAL DEFAULT 1.0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (source_memory_id) REFERENCES memory_items(id),
    FOREIGN KEY (target_memory_id) REFERENCES memory_items(id)
);
6.3 memory_usage_events

Tracks retrieval and use of memories.

CREATE TABLE memory_usage_events (
    id INTEGER PRIMARY KEY,
    memory_id INTEGER NOT NULL,
    session_id TEXT,
    interaction_id TEXT,
    context_hash TEXT,
    context_type TEXT,                    -- debugging | writing | planning | research | etc
    trigger_type TEXT,                    -- semantic_retrieval | pathway | explicit_reference | heuristic
    action_type TEXT,                     -- injected | cited | referenced | ignored | tool_prep
    was_injected INTEGER DEFAULT 0,
    was_referenced_in_output INTEGER DEFAULT 0,
    token_cost INTEGER DEFAULT 0,
    user_feedback TEXT,                   -- accepted | corrected | ignored | unknown
    outcome_score REAL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (memory_id) REFERENCES memory_items(id)
);
6.4 memory_verifications

Explicit confirmations or corrections.

CREATE TABLE memory_verifications (
    id INTEGER PRIMARY KEY,
    memory_id INTEGER NOT NULL,
    verification_type TEXT NOT NULL,      -- confirm | correct | contradict | revalidate
    evidence_strength REAL DEFAULT 1.0,
    note TEXT,
    source_type TEXT,                     -- user | tool | external | system
    source_ref TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (memory_id) REFERENCES memory_items(id)
);
6.5 pathways

Stores recurring successful retrieval/action patterns.

CREATE TABLE pathways (
    id INTEGER PRIMARY KEY,
    pathway_key TEXT NOT NULL UNIQUE,
    trigger_signature TEXT NOT NULL,
    context_type TEXT NOT NULL,
    action_type TEXT NOT NULL,
    memory_bundle_signature TEXT NOT NULL,
    description TEXT,
    status TEXT DEFAULT 'active',
    success_count INTEGER DEFAULT 0,
    failure_count INTEGER DEFAULT 0,
    correction_count INTEGER DEFAULT 0,
    use_count INTEGER DEFAULT 0,
    avg_outcome_score REAL DEFAULT 0.0,
    avg_latency_ms REAL DEFAULT 0.0,
    token_efficiency REAL DEFAULT 0.0,
    success_rate REAL DEFAULT 0.0,
    last_used_at DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
6.6 pathway_memory_members

Maps pathways to memory types or specific memory items.

CREATE TABLE pathway_memory_members (
    id INTEGER PRIMARY KEY,
    pathway_id INTEGER NOT NULL,
    member_type TEXT NOT NULL,            -- memory_type | memory_id | scope | tag
    member_value TEXT NOT NULL,
    member_weight REAL DEFAULT 1.0,
    FOREIGN KEY (pathway_id) REFERENCES pathways(id)
);
6.7 pathway_events

Tracks pathway execution.

CREATE TABLE pathway_events (
    id INTEGER PRIMARY KEY,
    pathway_id INTEGER NOT NULL,
    session_id TEXT,
    interaction_id TEXT,
    context_hash TEXT,
    triggered_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    trigger_match_score REAL DEFAULT 0.0,
    outcome_score REAL,
    latency_ms INTEGER,
    token_cost INTEGER,
    user_feedback TEXT,
    was_successful INTEGER DEFAULT 0,
    FOREIGN KEY (pathway_id) REFERENCES pathways(id)
);
6.8 context_snapshots

Captures a normalized representation of the current state.

CREATE TABLE context_snapshots (
    id INTEGER PRIMARY KEY,
    session_id TEXT,
    interaction_id TEXT,
    context_hash TEXT NOT NULL UNIQUE,
    context_type TEXT,
    user_input TEXT,
    project_ref TEXT,
    active_scope TEXT,
    snapshot_json TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
6.9 retrieval_runs

Tracks full retrieval cycles for observability.

CREATE TABLE retrieval_runs (
    id INTEGER PRIMARY KEY,
    session_id TEXT,
    interaction_id TEXT,
    context_hash TEXT,
    context_type TEXT,
    selected_pathway_id INTEGER,
    retrieved_memory_count INTEGER DEFAULT 0,
    injected_memory_count INTEGER DEFAULT 0,
    total_token_cost INTEGER DEFAULT 0,
    final_outcome_score REAL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (selected_pathway_id) REFERENCES pathways(id)
);
7. Derived Metrics

These metrics can be updated incrementally or periodically.

7.1 Confidence score

Confidence measures estimated correctness.

A simple initial form:

confidence_score = (verification_count + 1) / (verification_count + correction_count + contradiction_count + 2)

More advanced forms can use Bayesian updating.

7.2 Utility score

Utility measures whether using this memory helps.

Suggested update rule:

def update_utility(old_utility, outcome, alpha=0.2):
    return (1 - alpha) * old_utility + alpha * outcome

Where outcome is in [-1.0, 1.0].

7.3 Activation score

Activation reflects recency and frequency.

Example:

activation_score = log(1 + use_count) * recency_decay(last_used_at)
7.4 Stability score

Stability measures whether the memory remains valid over time.

Example heuristic:

preferences: high baseline stability

task state: low baseline stability

external facts: medium unless periodically revalidated

project architecture decisions: medium-high

7.5 Correction risk
correction_risk = correction_count / (use_count + 1)
8. Retrieval Engine

Retrieval should be staged, not flat.

Do not run one broad semantic query and inject the top results.

Use a pipeline.

8.1 Stage 1: Context classification

Classify the current interaction.

Recommended categories:

debugging

writing

planning

research

strategy

ops

support

casual

This can be heuristic or model-assisted.

Example features:

keywords

active project

tool usage pattern

user intent

previous interaction type

Output:

context_type

context_hash

scope candidates

8.2 Stage 2: Pathway activation

Check whether a known pathway fits this context.

Compute a pathway trigger match score using:

context type

project scope

user intent

recent prior turns

available memory categories

prior success in similar contexts

Example:

pathway_match = (
    0.40 * context_similarity +
    0.25 * scope_match +
    0.20 * recency_weight +
    0.15 * success_rate
)

Select:

one top pathway

or no pathway if confidence is too low

8.3 Stage 3: Candidate memory retrieval

Retrieve from multiple pools:

Pool A: pathway-driven retrieval

Memories associated with the selected pathway.

Pool B: semantic retrieval

Memories semantically similar to current input.

Pool C: high-value active scope retrieval

Current project facts, decisions, open tasks, constraints.

Pool D: stable user memory

Preferences, tone, operating constraints.

Pool E: failure memory

Relevant prior mistakes and failed approaches.

8.4 Stage 4: Ranking

Each candidate memory gets a final score.

Suggested first-pass formula:

final_rank = (
    0.35 * relevance_score +
    0.20 * confidence_score +
    0.20 * utility_score +
    0.10 * activation_score +
    0.10 * stability_score -
    0.05 * cost_score
)

Alternative:

final_rank = (
    0.30 * relevance_score +
    0.25 * utility_score +
    0.15 * confidence_score +
    0.10 * activation_score +
    0.10 * stability_score -
    0.05 * correction_risk -
    0.05 * cost_score
)

Notes:

relevance_score is dynamic

utility_score should drive practical adaptation

confidence_score prevents misleading memory

cost_score prevents bloated prompts

8.5 Stage 5: Bundle assembly

Do not inject arbitrary top-k.

Build a structured memory bundle with quotas by type.

Recommended bundle sections:

active project facts

current decisions

stable preferences

open loops

prior failures

supporting evidence if needed

Example quota system:

2–4 project facts

1–2 preferences

1–2 decisions

1–2 open tasks

1 prior failure if highly relevant

This controls sprawl and improves interpretability.

8.6 Stage 6: Post-response evaluation

After response generation, evaluate:

was the answer accepted?

was there a correction?

did the user re-explain context?

did tools succeed?

did this reduce latency or confusion?

did the conversation move productively?

This stage updates:

memory usage events

memory utility

pathway events

pathway success rate

contradiction or staleness markers

9. Pathway Learning

This is the major intelligence layer.

A pathway should represent:

A repeatable context → memory bundle → action pattern that tends to work.

9.1 Pathway examples
Example A: Pricing discussion

Trigger:

context type = strategy

intent contains pricing, positioning, competition

Memory bundle:

competitor pricing

user preference for one-time purchase

current product scope

prior objections

Action:

compare competitors first

then translate into pricing implications

Example B: Debugging workflow

Trigger:

context type = debugging

Memory bundle:

project stack

working directory

prior migration failures

relevant environment constraints

Action:

propose fix with precise path context

Example C: Writing request

Trigger:

context type = writing

Memory bundle:

tone preference

product positioning

recent arguments or themes

anti-verbosity constraint

Action:

produce concise, direct copy draft

9.2 Pathway key design

A pathway key should be deterministic enough to aggregate useful repetition.

Example format:

context_type=debugging|
scope=project:autoglia|
memory_bundle=stack+paths+prior_failures|
action=propose_fix
9.3 Pathway promotion rule

Promote a pathway if:

it has been used successfully at least N times

outcome score remains above threshold

correction rate remains below threshold

token cost is acceptable

Example:

if use_count >= 3 and success_rate >= 0.7 and correction_rate <= 0.15:
    pathway.status = "active"
9.4 Pathway demotion rule

Demote if:

repeated low outcome score

repeated corrections

stale due to workflow change

replaced by a better pathway

10. Outcome Model

You need a concrete outcome model.

A memory should rise because it helps, not merely because it appears frequently.

10.1 Strong positive signals

explicit confirmation from user

task completed successfully

user does not need to restate obvious context

pathway reused in future successful sessions

answer accepted with minimal correction

10.2 Weak positive signals

user continues productively

no correction after use

relevant tool execution succeeds

memory was referenced and not disputed

10.3 Negative signals

user correction

irrelevant memory injection

repeated asking of known context

stale memory caused wrong assumption

prompt/context inflation without benefit

tool failure caused by wrong retrieved assumptions

10.4 Suggested scoring scale

Map event outcomes to a bounded score.

Example:

explicit confirmation = +1.0

successful task completion = +0.8

productive continuation = +0.4

neutral/unknown = 0.0

partial correction = -0.5

strong correction/failure = -1.0

11. Update Algorithms
11.1 Update memory utility
def update_memory_utility(memory, outcome_score, alpha=0.2):
    memory.utility_score = (1 - alpha) * memory.utility_score + alpha * outcome_score
    memory.use_count += 1
    memory.last_used_at = now()

    if outcome_score > 0:
        memory.success_count += 1
    elif outcome_score < 0:
        memory.correction_count += 1
11.2 Update confidence

Simple count-based version:

def recompute_confidence(memory):
    v = memory.verification_count
    c = memory.correction_count
    d = memory.contradiction_count
    memory.confidence_score = (v + 1) / (v + c + d + 2)

Bayesian version can be added later.

11.3 Update pathway performance
def update_pathway(pathway, outcome_score, latency_ms=None, token_cost=None):
    pathway.use_count += 1
    pathway.avg_outcome_score = moving_average(pathway.avg_outcome_score, outcome_score, pathway.use_count)

    if outcome_score > 0:
        pathway.success_count += 1
    elif outcome_score < 0:
        pathway.failure_count += 1

    denom = pathway.success_count + pathway.failure_count
    pathway.success_rate = pathway.success_count / denom if denom > 0 else 0.0

    if latency_ms is not None:
        pathway.avg_latency_ms = moving_average(pathway.avg_latency_ms, latency_ms, pathway.use_count)

    if token_cost is not None:
        pathway.token_efficiency = compute_token_efficiency(pathway.avg_outcome_score, token_cost)

    pathway.last_used_at = now()
12. Staleness, Decay, and Supersession

This is essential.

Without decay, old memories accumulate and degrade quality.

12.1 Type-based staleness defaults

Suggested defaults:

preference → long-lived, low decay

decision → medium-lived, revalidate if contradicted

task → short-lived, high decay

fact → depends on source class

external fact → medium decay unless verified recently

workflow → medium decay

failure → medium-high durability if tied to a stable system

12.2 Staleness rules

A memory becomes stale if:

current date exceeds last_verified_at + stale_after_days

a more recent memory contradicts it

project state changes and invalidates it

repeated retrieval yields no use or repeated correction

12.3 Supersession rules

A memory becomes superseded if a newer memory of the same scoped concept replaces it.

Examples:

old pricing decision replaced by new pricing decision

previous working directory changed

user preference updated

Use superseded_by plus an edge in memory_edges.

13. Contradiction Handling

The system must not blindly retrieve contradictory memories.

13.1 Contradiction detection

Trigger contradiction review when:

two active memories of same type and scope conflict

user correction explicitly negates a memory

tool result invalidates prior memory

a pathway repeatedly fails due to outdated assumption

13.2 Contradiction policy

When contradiction occurs:

mark older or disputed memory as disputed

lower confidence

create contradicts edge

prefer newer verified memory

optionally store a resolution note

14. Embedding and Retrieval Strategy

The system can use embeddings, but embeddings should not be the whole retrieval engine.

Recommended strategy:

embeddings for semantic similarity

relational filters for scope/type

pathway priors for context fit

score-based reranking for final selection

Embeddings answer:

“What seems similar?”

Pathways and scores answer:

“What is likely to help?”

That distinction matters.

15. Minimal Viable Implementation

Do not build the full research architecture first.

Implement in phases.

Phase 1: Utility-aware memory

Add to current memory system:

memory types

scope

use_count

success_count

correction_count

utility_score

confidence_score

last_used_at

Implement:

basic retrieval logging

post-response outcome scoring

reranking by relevance + utility + confidence

Result

Immediate improvement in selecting useful memory.

Phase 2: Bundle-based retrieval

Add:

context classification

bundle quotas by memory type

structured injection instead of naive top-k

Result

Better prompt composition and lower token waste.

Phase 3: Pathway learning

Add:

pathways

pathway_events

deterministic pathway keys

promotion/demotion logic

Result

Visible workflow adaptation and apparent intelligence gains.

Phase 4: Staleness and contradiction

Add:

stale state

superseded state

contradiction edges

revalidation rules

Result

Less confident reuse of obsolete memory.

Phase 5: Optional proactive preparation

Add:

preloading likely bundles

surfacing likely next-step context

internal anticipatory pathway activation

Result

System feels ahead of the user, but only after the retrieval core is reliable.

16. Observability and Debugging

This product needs introspection.

You should be able to inspect:

what memories were retrieved

why they ranked high

what pathway fired

what outcome score was assigned

which memories were promoted or demoted

which memories are stale or disputed

Useful dashboard sections:

top promoted memories

top successful pathways

most corrected memories

stale memory candidates

highest utility memory types by context

pathway success by context type

17. User-Visible Product Features

These features make the intelligence gain legible.

17.1 “Why I used this memory”

Expose a brief rationale:

used in 4 similar sessions

last verified 2 days ago

tied to active project

part of a successful debugging pathway

17.2 “Core memory”

Show memories that have been promoted:

stable preferences

recurring project assumptions

repeatedly useful facts

recurring workflow patterns

17.3 “This may be stale”

Very important for trust.

17.4 “Previously failed approach”

Useful in technical/product workflows.

17.5 “Known successful route”

Internally or optionally visible:

for pricing discussions, competitor-first comparison has worked well

for debugging, include project path and environment state first

This is how memory becomes visibly intelligent rather than invisibly stored.

18. Example End-to-End Flow

User asks:

“Help me figure out pricing for Autoglia.”

Step 1: classify context

context_type = strategy

scope = project:autoglia

Step 2: pathway activation

Find known pathway:

pricing strategy

retrieve competitor pricing

retrieve user preference for one-time payment

retrieve current product framing

Step 3: candidate retrieval

Retrieve:

Autoglia project facts

previous pricing discussions

preference: user prefers practical, concise answers

competitor benchmark memory

prior objection memory

Step 4: ranking

Rank using relevance + utility + confidence + stability.

Step 5: bundle assembly

Bundle:

one-time pricing preference

product scope

prior competitor analysis

prior objections

Step 6: answer generation

Assistant produces analysis without re-asking known context.

Step 7: evaluation

User says:

“Yes, competitor-first is exactly what I wanted.”

Update:

memory utility up

pathway success up

pathway becomes more likely next time

This is apparent intelligence through learned selection.

19. What Not to Do

Avoid these failure modes.

19.1 Do not reward repetition alone

A frequently used pathway is not automatically valuable.

Repetition must be combined with positive outcomes.

19.2 Do not treat all memory as factual

Preferences, workflows, and priors are not the same as external truth claims.

19.3 Do not let memory grow without hierarchy

Unbounded accumulation degrades retrieval quality.

19.4 Do not rely on embeddings alone

Semantic similarity is not the same as usefulness.

19.5 Do not store every reasoning trace indiscriminately

Store durable decision/use patterns first, not unbounded thought residue.

---

## 22. Multi-Model Adaptation

Different underlying models may benefit from different memory weightings.

### 22.1 Why Model-Specific Tuning Matters

- Claude may excel at reasoning but need more context continuity
- GPT may need explicit preference reminders
- Smaller models may need more explicit constraints to avoid hallucination

### 22.2 Model Profile Schema

```sql
CREATE TABLE model_profiles (
    id INTEGER PRIMARY KEY,
    model_name TEXT NOT NULL,
    context_window INTEGER,
    strengths TEXT,                    -- JSON: ["reasoning", "creativity"]
    weaknesses TEXT,                   -- JSON: ["context", "memory"]
    default_weights_json TEXT,         -- JSON: score weight overrides
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 22.3 Model-Specific Scoring

For each model, adjust retrieval weights:

```python
def adjusted_score(memory, model_profile):
    base = compute_base_score(memory)
    if model_profile.weakness == "context":
        # Boost recency and activation for context-hungry models
        base.activation_weight *= 1.5
    if model_profile.strength == "reasoning":
        # Boost inference-type memories for strong reasoners
        base.inference_boost *= 1.2
    return base
```

### 22.4 Auto-Detection

Detect model from OpenClaw config and apply appropriate profile automatically.

---

## 23. Time-Based Pattern Recognition

The system should learn temporal patterns: when the user typically does what.

### 23.1 Time Pattern Schema

```sql
CREATE TABLE time_patterns (
    id INTEGER PRIMARY KEY,
    pattern_type TEXT NOT NULL,       -- "recurring_query" | "workflow" | "check"
    trigger_conditions TEXT NOT NULL, -- JSON: day_of_week, time_range, interval
    associated_action TEXT,
    success_rate REAL DEFAULT 0.0,
    occurrence_count INTEGER DEFAULT 0,
    last_triggered DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 23.2 Pattern Examples

- "Every Monday morning, user asks about blog research"
- "Every evening, user checks command center"
- "After blogwatcher runs, user reviews results within 30 minutes"

### 23.3 Pre-Fetching Behavior

When time pattern confidence exceeds threshold:

```python
if time_pattern.match(current_time) and pattern.success_rate > 0.7:
    pre_fetch(associated_action)
    # User hasn't asked yet, but likely will
```

---

## 24. Negative Pathways and Explicit Avoidance

Track what was explicitly rejected, not just what failed.

### 24.1 Negative Pathway Schema

```sql
CREATE TABLE negative_pathways (
    id INTEGER PRIMARY KEY,
    pathway_key TEXT NOT NULL,       -- The pathway that was rejected
    rejection_type TEXT NOT NULL,     -- "user_declined" | "user_corrected" | "explicit_forbid"
    context TEXT,
    note TEXT,                        -- User's reason if given
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_overrides (
    id INTEGER PRIMARY KEY,
    override_type TEXT NOT NULL,      -- "always_prefer" | "never_use" | "force_include"
    target_memory_id INTEGER,
    target_pathway_id INTEGER,
    target_pattern TEXT,              -- e.g., "pricing-first approach"
    priority INTEGER DEFAULT 10,      -- Higher = more important
    expires_at DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 24.2 Negative Pathway Example

- User says: "Don't start with competitor analysis, just give me your thoughts"
- System stores: negative_pathway entry with rejection_type = "user_declined"
- Future retrieval: boost pathways that don't start with competitor analysis

### 24.3 Override Priority

User overrides always win:

```python
def retrieve_with_overrides(candidates, user_overrides):
    for override in user_overrides:
        if override.type == "never_use":
            candidates.remove(override.target)
        if override.type == "always_prefer":
            candidates.boost(override.target, priority=override.priority)
    return candidates
```

---

## 25. Emotional and Context Signals

Beyond content, consider user state in retrieval weighting.

### 25.1 Signal Types

- **Urgency**: Short messages, exclamation marks, "ASAP"
- **Frustration**: "This keeps happening", "Why doesn't it work"
- **Energized**: "I'm excited about", "Let's do this"
- **Tired**: Late hours, shorter interactions

### 25.2 Schema Extension

```sql
CREATE TABLE context_signals (
    id INTEGER PRIMARY KEY,
    session_id TEXT,
    signal_type TEXT NOT NULL,       -- urgency | frustration | energized | tired
    intensity REAL DEFAULT 0.5,      -- 0-1 scale
    evidence TEXT,                   -- what triggered detection
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 25.3 Signal-Aware Retrieval

```python
def adjust_for_signal(base_scores, context_signal):
    if signal.type == "frustration":
        # Prioritize memories about fixes, known issues, stable solutions
        base_scores.failure_memories *= 1.3
        base_scores.new_experiments *= 0.7
    if signal.type == "energized":
        # More openness to trying new approaches
        base_scores.experimental_paths *= 1.2
    return base_scores
```

---

## 26. Graceful Degradation

What happens when no pathway matches?

### 26.1 Fallback Hierarchy

1. **Semantic similarity retrieval** → Pool B only
2. **Active project scope memories** → Pool C + D
3. **User preference recall** → High-weight preference retrieval
4. **Last-session summary** → If available
5. **Fresh start** → No memory injection

### 26.2 Confidence Thresholding

```python
def get_fallback_level(pathway_match_score):
    if pathway_match_score > 0.8:
        return "full_pathway"
    elif pathway_match_score > 0.5:
        return "partial_pathway"
    elif pathway_match_score > 0.3:
        return "semantic_only"
    else:
        return "minimal_injection"
```

### 26.3 Explicit "Help" Mode

If user explicitly asks for help, temporarily lower confidence thresholds and be more exploratory.

---

## 27. Real-Time vs Batch Updates

When does the system learn?

### 27.1 Update Modes

| Component | Real-Time | Batch | Hybrid |
|-----------|-----------|-------|--------|
| Memory usage events | ✓ | | |
| Utility scores | | ✓ | Nightly |
| Pathway success | ✓ | | |
| Pathway promotion | | ✓ | Weekly |
| Staleness checks | | ✓ | Daily |
| Contradiction detection | ✓ | | |
| Time patterns | | ✓ | Weekly |

### 27.2 Rationale

- Real-time: User feedback (fast signals)
- Batch: Aggregate patterns (need volume)
- Hybrid: Fast signals + slow pattern learning

### 27.3 Implementation

```python
# Real-time updates (every interaction)
def on_interaction_end(outcome):
    log_usage_event(outcome)
    update_pathway_realtime(outcome)

# Nightly batch (aggregate learning)
def nightly_batch():
    recompute_utilities()
    detect_contradictions()
    update_time_patterns()

# Weekly (structural)
def weekly_batch():
    pathway_promotion_demotion()
    stale_memory_archive()
    model_profile_updates()
```

---

20. Technical Summary

Autoglia should become:

A continuously trained selection system for what to remember, what to retrieve, and what to do with it.

Its intelligence gain will not come from “having more memory.”

It will come from:

typed memory

outcome-linked utility tracking

pathway learning

contradiction handling

staleness control

structured bundle assembly

That is what makes a memory layer feel smart.

21. Final Implementation Thesis

The real product is not a memory database.

It is:

A local adaptive memory engine that learns which memories and retrieval pathways reliably improve outcomes.

That is how memory becomes part of the visible increase in intelligence.

Not because the model itself changes.

But because the system gets better at:

selecting the right context

avoiding the wrong context

carrying forward validated state

adapting to the user’s workflow

repeating successful routes

refusing stale or disputed assumptions
I’m framing this as a product-eval problem: prove that the new memory engine is measurably better than prior Autoglia versions on continuity, retrieval quality, and task usefulness.

For Autoglia, the benchmark should answer one question:

Does the newer memory system help the agent perform better than the previous version on repeatable tasks?

Not “does it store more.”
Not “does it feel smarter.”
But:

does it recover the right context more often

does it ask fewer redundant questions

does it use the right memory more often

does it complete tasks with fewer corrections

does it improve over repeated runs

What to benchmark

You want at least 4 benchmark layers.

1. Memory retrieval quality

Can the system retrieve the right memories?

Metrics:

Recall@K: was the needed memory in top K retrieved items?

MRR: how high did the correct memory rank?

Precision@K: how many retrieved memories were actually useful?

Noise ratio: irrelevant retrieved memories / total retrieved memories

This tests the retrieval engine itself.

2. Task outcome quality

Did the memory actually help solve the task?

Metrics:

task success rate

correction rate

number of user restatements needed

number of redundant clarification questions

time-to-correct-answer

tool success rate if tools are involved

This is more important than raw retrieval.

3. Continuity quality

Does the system preserve cross-session state better than old Autoglia?

Metrics:

continuity accuracy

project-state recovery score

preference retention score

open-loop/task carry-forward score

contradiction/staleness error rate

This is where Autoglia should clearly beat older versions.

4. Apparent intelligence / usefulness

Does the user experience feel more intelligent?

You should operationalize this instead of treating it as vague.

Metrics:

fewer repeated questions

fewer wrong assumptions

more appropriate use of prior decisions

more successful pathway reuse

lower token waste for same or better outcome

optional human judge score on “felt intelligence”

Minimum benchmark structure

You need a fixed eval set.

Benchmark item format

Each test case should include:

case_id

scenario_type

initial_memory_state

prior_sessions

current_prompt

ground_truth_needed_memories

expected_outcome

scoring_rules

Example:

{
  "case_id": "pricing_001",
  "scenario_type": "strategy",
  "initial_memory_state": [
    "User prefers one-time pricing",
    "Autoglia targets OpenClaw users",
    "Previous discussion emphasized competitor-first analysis"
  ],
  "prior_sessions": [
    "User rejected subscription model",
    "User asked for concise, research-based answers"
  ],
  "current_prompt": "Help me think about Autoglia pricing again.",
  "ground_truth_needed_memories": [
    "User prefers one-time pricing",
    "Competitor-first analysis pattern",
    "Autoglia target user"
  ],
  "expected_outcome": [
    "Does not ask what product Autoglia is",
    "Does not suggest subscription first",
    "Uses competitor-first framing"
  ]
}


This lets you compare version-to-version.

Benchmark categories you need

I would build 6 categories.

A. Preference retention

Test whether the system remembers:

brevity preference

workflow preference

product constraints

style constraints

Failure example:

asks again

violates known preference

B. Project continuity

Test whether it remembers:

active project

product architecture

unresolved decisions

past technical constraints

Failure example:

treats project as new

misses already-settled context

C. Decision reuse

Test whether it can retrieve:

prior chosen path

rejected path

rationale

Failure example:

repeats previously rejected suggestion

D. Failure avoidance

Test whether it remembers prior failed approaches.

Failure example:

recommends the same failed migration/debug approach again

E. Context bundle selection

Test whether it picks the right bundle for the current task.

Failure example:

retrieves semantically related but useless memory

misses task-critical memory

F. Staleness/contradiction handling

Test whether it avoids obsolete memories.

Failure example:

uses old pricing plan after newer decision exists

Compare against which baselines

At minimum:

Baseline 0: no memory

Agent without Autoglia or memory injection.

Baseline 1: old Autoglia

Your previous memory system:

persistence only

simpler retrieval

less utility/pathway weighting

Baseline 2: current Autoglia retrieval only

Typed memory + retrieval, but no utility/pathway learning.

Baseline 3: full new system

Typed memory + utility scoring + pathway ranking + staleness handling.

That comparison matters because it isolates what each added layer is buying you.

What to log for each run

For every benchmark run, log:

prompt

retrieved memories

rank order

injected memories

selected pathway

output

outcome score

corrections triggered

token cost

latency

Without full logging, you will not know why one version won.

Scoring rubric

Use a weighted score.

Example total score: 100 points
Retrieval quality — 25

needed memory in top 5: 10

rank quality: 10

low irrelevant retrievals: 5

Continuity — 25

preserved user preferences: 10

preserved project state: 10

preserved open loops/decisions: 5

Task outcome — 35

successful answer/task completion: 15

no redundant questioning: 10

no repeated mistakes: 10

Efficiency — 15

lower token waste: 5

lower latency: 5

fewer unnecessary memories injected: 5

That gives you a single benchmark score while preserving sub-metrics.

The most important benchmark type

The highest-value benchmark is not isolated retrieval.

It is:

multi-session replay evaluation

That means:

feed prior session transcripts into the memory system

start a fresh session with a prompt

compare how each version performs

score whether it correctly reconstructs state and solves the task

This directly tests what users actually care about.

Best practical eval sets

You probably want three datasets.

1. Synthetic eval set

Hand-authored cases with known ground truth.

Good for:

stable scoring

retrieval debugging

regression testing

2. Real replay eval set

Use real prior Autoglia conversations/projects.

Good for:

realism

proving practical value

catching messy edge cases

3. Adversarial eval set

Cases designed to break the memory system:

contradictory memories

stale decisions

semantically similar distractions

repeated failed approaches

changed preferences

Good for:

robustness

showing new version is safer, not just richer

Specific benchmark metrics for your new additions

Because you are adding utility/pathway ideas, you need metrics that test them directly.

Memory utility metrics

average outcome score per retrieved memory

promoted-memory hit rate

utility-weighted retrieval gain vs plain relevance retrieval

fraction of retrieved memories that contribute to accepted answers

Pathway metrics

pathway activation precision

pathway activation recall

pathway success rate

average outcome of pathway-driven runs vs non-pathway runs

pathway reuse benefit over time

Staleness metrics

stale-memory usage rate

contradiction error rate

superseded-memory retrieval rate

These will tell you whether the new architecture is actually doing its job.

Benchmark progression over time

You should benchmark two things:

1. Static version comparison

Compare v1 vs v2 on the same eval set.

2. Learning curve comparison

Run repeated sessions and measure whether v2 improves over time while v1 stays flat.

That second one is crucial, because your product claim is not merely “better retrieval,” but:

the system gets better as it is used

So show:

run 1

run 5

run 20

Metrics should improve for:

utility-weighted retrieval

pathway success

correction avoidance

continuity quality

If that curve is flat, the learning layer is not working.

Good benchmark outputs

You want outputs that are easy to inspect.

Per-case report

case id

old version score

new version score

diff

retrieved-memory diff

outcome diff

failure reason if regression

Aggregate report

average total score

average retrieval score

average continuity score

average task outcome score

regression count

improvement count

Diagnostic tables

most improved cases

worst regressions

most frequently misretrieved memory types

top successful pathways

top failing pathways

One strong benchmark formula

For each case:

Total Score =
0.25 * Retrieval Score
+ 0.25 * Continuity Score
+ 0.35 * Task Outcome Score
+ 0.15 * Efficiency Score


Then compare:

delta_total = new_version - old_version

delta_continuity

delta_task

delta_efficiency

That gives you both top-line and diagnostic insight.

What success should look like

The new version should show:

higher Recall@K for needed memories

fewer redundant questions

fewer repeated failed approaches

better carry-forward of decisions/preferences

lower stale-memory error rate

higher task completion quality

improvement over repeated use

If it only retrieves more memories, but does not improve outcomes, it is not a product win.

Concrete next build

You likely need 3 files or modules:

1. benchmark_cases.jsonl

Your eval dataset.

2. run_benchmark.py

Runs each case against selected Autoglia version/config.

3. score_benchmark.py

Scores outcomes and generates reports.

And probably a fourth:

4. compare_benchmarks.py

Produces side-by-side diffs between versions.

Recommended first benchmark pack

Start with just 25–40 cases.

Breakdown:

8 preference retention

8 project continuity

6 decision reuse/failure avoidance

6 staleness/contradiction

6 task bundle selection

That is enough to expose major regressions without overbuilding.

Bottom line

You need to benchmark memory as performance, not memory as storage.

The benchmark should prove that newer Autoglia versions are better at:

selecting relevant context

preserving continuity

avoiding stale or failed paths

improving outcomes

learning useful pathways over time

The most important artifact is a repeatable multi-session replay benchmark with fixed scoring.

I can turn this next into a concrete markdown spec for:
“Autoglia Benchmarking Framework v1” with dataset format, metrics, scoring rubric, and harness structure.
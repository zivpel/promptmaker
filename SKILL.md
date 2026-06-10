---
name: prompt-maker
description: "Transform weak prompts into high-performance structured prompts. Model aware (GPT-5.5, Claude Fable 5, Claude Opus 4.8, or Claude Sonnet 4.6), complexity tiered, personalization aware. Detects target model, classifies complexity, applies the right optimization path. Trigger when users say 'improve a prompt', 'make this prompt better', 'optimize my prompt', 'rewrite this prompt', 'fix my prompt', 'create a prompt for X', 'write a prompt that does Y', 'prompt engineer this', 'apply the prompt maker method', 'use the formula', 'use the method', or reference 'Prompt Maker'. Also trigger when users paste a raw or weak prompt and ask for enhancement, or describe a task and want a prompt built from scratch. Covers all types: research briefs, analysis, document creation, meeting summaries, cold emails, customer health assessments, problem solving, reviews, and structured tasks. Always delivers the improved prompt as a downloadable file with a quality score."
---

# Prompt Maker v7.1

## Core Promise

Transform any weak or unstructured prompt into a measurably better one, optimized for the specific target model, calibrated to the right complexity tier, and personalized to the active user. Every output ships with a quality score (1 to 100) and a clear iteration path if the first run fails.

## Quick Reference Decision Tree

```
Input: a prompt to improve OR a task to build a prompt for
   │
   ├─► Step 0:   Which model? (GPT-5.5 | Claude Fable 5 | Claude Opus 4.8 | Claude Sonnet 4.6 | Multiple | Other)
   ├─► Step 0.5: What complexity tier? (Simple | Standard | Complex)
   ├─► Step 0.7: Personalization scan (active user? persistent prefs?)
   │
   ├─► Step 1: Context engineering — what belongs in context vs prompt?
   ├─► Step 2: Build universal skeleton (Role, Task, Inputs, Output, Constraints, Self-check)
   ├─► Step 3: Apply model route (A: GPT-5.5 | B: Fable 5 | C: Opus 4.8 | D: Sonnet 4.6)
   ├─► Step 4: Apply tier scaffolding (Simple = stripped; Standard = full; Complex = full + extras)
   ├─► Step 5: Apply universal improvements (flip negatives, hardcode known values, modular blocks)
   │
   ├─► Step 6: Score against the Quality Rubric (10 dimensions × 10 points = /100)
   ├─► Step 7: Validate against anti-patterns
   └─► Step 8: Deliver — file + score + iteration path + (if dual) default recommendation
```

---

## The Workflow

### Step 0: Detect Target Model

Ask if not provided:

> "Which model are you optimizing this prompt for? GPT-5.5, Claude Fable 5, Claude Opus 4.8, or Claude Sonnet 4.6?"

If the user says "Claude" without specifying, pick from the Claude Family Selector below based on the workload and state your choice.

Routing:
- One model → apply that route exclusively (A: GPT-5.5, B: Fable 5, C: Opus 4.8, D: Sonnet 4.6)
- Multiple → produce a version per model + a default recommendation (see Dual Output Default Protocol below)
- Other model (Gemini, GPT-5.1, Claude Opus 4.7, Claude Opus 4.6, Claude Haiku 4.5) → universal core formula only, flag that model-specific rules may not fully apply. Opus 4.7 and 4.6 prompts can mostly follow Route C.
- Production use → also ask: "Pinned to a specific snapshot, or running on the live router?"
- **Fable 5 domain check.** If the prompt touches offensive cybersecurity, biology and life sciences lab methods, or asks the model to reproduce its internal reasoning, flag it: Fable 5 runs safety classifiers on these and falls back to Claude Opus 4.8. Route those prompts to Opus 4.8 directly (Route C).

**Claude Family Selector** (when the user just says "Claude"):

| Signal | Pick | Why |
|--------|------|-----|
| Long-horizon, multi-day, autonomous, hardest unsolved problems | Fable 5 (Route B) | Built for end-to-end work spanning hours to days |
| Complex reasoning, agentic coding, knowledge work, expensive-to-fail tasks | Opus 4.8 (Route C) | Flagship Opus tier, $5/$25 per 1M |
| Volume production: classification, RAG, extraction, summaries, standard coding | Sonnet 4.6 (Route D) | Near-Opus quality at $3/$15 per 1M |
| Offensive cyber or bio/life sciences domain | Opus 4.8 (Route C) | Fable 5 classifiers fall back here anyway |

### Step 0.5: Detect Complexity Tier

Classify the prompt into one of three tiers. This drives how much scaffolding to apply.

| Tier | When to Use | Length Target | Scaffolding |
|------|-------------|---------------|-------------|
| **Simple** | Single-purpose, one-shot, low stakes (a tweet, a slack reply, a quick summary) | 50 to 150 words | Role + Task + Output Format only. Skip self-check. Skip steps. |
| **Standard** | Default tier. Most prompts land here (emails, meeting prep, briefs, analysis) | 150 to 500 words | Full universal core formula. Self-check required. |
| **Complex** | High stakes, multi-step, agentic, production-grade (customer agents, financial analysis, intelligence at scale) | 500 to 800 words | Full formula + snapshot pinning + self-consistency note + eval pack recommendation |

**Anti-pattern:** Applying the full Standard formula to a Simple prompt. A 50-word slack reply does not need a `<self_check>` block. The skill should flex down for simple work.

### Step 0.7: Personalization Scan

Before treating any input as a variable, scan for the active user and their persistent preferences.

**Identity hardcoding:**
- If you know who the user is (from memory, context, or signature), **hardcode their name, role, and company**. Do not write `[USER NAME]` or `[YOUR ROLE]` for the active user. That is a variable abuse.
- Variables are for things that change per use (customer name, topic, data block). The user themselves is not a variable.

**Persistent preferences:**
- Scan memory for documented preferences relevant to the prompt category.
- Examples: no em dashes, never abbreviating key brand terms, social posts scored against a defined rubric, a regional work week (e.g., Sunday to Thursday), a preferred document styling system, etc.
- Bake these in proactively. Do not ask the user to tell you what they already told you.

**Anonymization flag:**
- If the prompt is being built to share or templatize, flag what to anonymize before sharing: "If you share this template, replace the hardcoded name with [USER NAME] and the company with [COMPANY]."

### Step 1: Context Engineering Layer

Before optimizing the prompt itself, decide what belongs in context vs prompt.

Ask:
- What context does the model need that should be in the input, not the prompt? (Customer files, prior conversation, retrieved documents)
- Is RAG, summarization, or a structured input more efficient than a long prompt?
- Can the prompt be shorter if the context is richer?

**Rule:** push raw data into a context block (`<context>` for Claude, fenced markdown for GPT-5.5). Keep the prompt itself focused on instructions. Shorter prompt + richer context beats longer prompt + raw data dump.

### Step 2: Build Universal Skeleton

Construct the core components. These exist in every prompt at every tier (though Simple drops some):

```
Role:        Act like [EXPERT ROLE with EXPERIENCE, CREDENTIALS]
Task:        [SPECIFIC OUTCOME] for [AUDIENCE] that [SUCCESS CRITERION]
Scope:       [Categories — when comprehensive]
Inputs:      [Data, documents, context blocks]
Steps:       1. ... 2. ... 3. ...   (light or heavy by model + tier)
Output:      A) ... B) ... C) ...   (sections + length cap)
Requirements:[Must include]
Constraints: [Must avoid — flipped from negatives where possible]
Self-check:  [2 to 4 yes/no verification questions]
Closing:     Now produce the [deliverable].
```

### Step 3: Apply Model Route

Route A (GPT-5.5), Route B (Claude Fable 5), Route C (Claude Opus 4.8), or Route D (Claude Sonnet 4.6). See Reference section below.

### Step 4: Apply Tier Scaffolding

| Tier | Drop | Keep | Add |
|------|------|------|-----|
| Simple | Steps, Self-check, Scope | Role, Task, Output Format, Constraints | (nothing) |
| Standard | (nothing) | Full skeleton | (nothing) |
| Complex | (nothing) | Full skeleton | Snapshot pinning note, self-consistency recommendation, eval pack guidance. For Fable 5: the Long-Run Pack (progress grounding, boundary statement, subagent guidance, memory note) |

### Step 5: Apply Universal Improvements

In order:
1. **Hardcode vs bracket** — hardcode known values (active user, company, confirmed numbers). Use `[BRACKETS]` only for values that change per use.
2. **Flip negatives** — apply the Flip Negatives Protocol (see Reference).
3. **Modular blocks** — break the prompt into distinct sections, not a single blob.
4. **For Complex tier only:** add self-consistency note for high-stakes reasoning.

### Step 6: Score Against the Quality Rubric

Score the output prompt across 10 dimensions, 1 to 10 each. See full rubric in Reference. Total score range:
- **80 to 100:** ship
- **60 to 79:** revise weak dimensions, then re-score
- **Below 60:** restart from Step 2

Always show the score to the user when delivering.

### Step 7: Validate Against Anti-Patterns

Final check before delivery. See full Anti-Patterns list in Reference. Hot checks:
- Length within target for tier and model
- Hardcoded values where known, brackets only for variables
- Negatives count: 3 or fewer in the final output
- For research: citations, time bounds, "never fabricate" all present
- For Claude Fable 5: effort recommendation included; no instructions asking the model to echo or reproduce its internal reasoning; scaffolding trimmed to intent level (over-prescription degrades Fable 5 output)
- For production: snapshot pinning recommendation included

### Step 8: Deliver

Every delivery includes:
1. **The optimized prompt** as a downloadable markdown file
2. **The model + tier** stated at the top of the file
3. **Quality score** (X/100) with the lowest 2 dimensions called out
4. **Iteration path** — "If the output [does X], adjust [Y]. See Iteration Protocol [letter]."
5. **Before/after comparison table** (8 standard dimensions)
6. **Snapshot pinning note** if production-bound
7. **Default recommendation** if dual output (see Dual Output Default Protocol)

---

## Reference: Model Routes

### Route A: GPT-5.5 Optimization

GPT-5.5 was retrained from the ground up (the first full retrain since GPT-4.5).

**Core principles:**
1. **Outcome first, process light.** Lead with what you want, not how to get there. Skip step-by-step process guidance unless the exact path matters.
2. **Be concise.** GPT-5.5 uses ~72% fewer output tokens than Claude 4.7 Opus on equivalent tasks. Match that efficiency in your prompt. (Note: Claude Fable 5 narrows this gap, completing equivalent work with fewer tool calls and lower token consumption than prior Opus-tier models.)
3. **Define success criteria explicitly.** Tell the model what done looks like, then trust it to figure out the steps.
4. **Use length constraints inline.** Add a verbosity spec: "Default: 3 to 6 sentences or 5 bullets max."
5. **Pre-tool acknowledgment for agentic tasks.** Send a brief user-visible status update before tool calls.
6. **Strip legacy scaffolding.** Delete "think step by step", "be careful", "make sure you" from old prompts.
7. **GPT-5.5 Pro caveat.** Pro is slower, no streaming, disables some agentic tools. Pick deliberately.

**GPT-5.5 skeleton:**

```
Act like [expert role + credentials].
Your task: [specific outcome] for [audience]. Success criteria: [3 to 5 bullets].
Inputs: [data, documents, context]
Allowed side effects: [what the model can do beyond just answering]
Output format: [shape, sections, length cap]
Constraints: [hard rules, scope boundaries]

Now produce the [deliverable].
```

What is missing on purpose: explicit numbered steps. GPT-5.5 plans them itself.

### Route B: Claude Fable 5 Optimization

Claude Fable 5 is the first Mythos-class model: built for long-horizon, autonomous, end-to-end work. Two shifts define how you prompt it versus Claude 4.7 Opus:

**Shift 1: Less scaffolding, more intent.** Instruction following is strong enough that a brief instruction steers most behaviors. Prompts and skills written for prior models are often too prescriptive for Fable 5 and can degrade output quality. Trim heavy step-by-step process guidance; state intent, success criteria, and boundaries instead.

**Shift 2: Give the reason, not only the request.** Fable 5 performs better when it understands why you are asking. Lead with context: "I'm working on [larger task] for [audience]. They need [what the output enables]. With that in mind: [request]."

**Core principles:**
1. **Explicit intent and success criteria upfront.** Then trust the model to plan. Numbered steps only when the exact path matters (compliance, fixed workflows), not as a default.
2. **Specify output format explicitly.** Sections, length caps, structure. Still required.
3. **Use XML tags.** `<context>`, `<task>`, `<output_format>`, `<self_check>`. Still the model-native pattern. Use fewer, larger blocks than you would for 4.7.
4. **Set effort level explicitly.** Effort is the primary control for intelligence vs latency vs cost:
   - `high` is the default for most tasks
   - `xhigh` for the most capability-sensitive workloads (at the highest effort, Fable 5 reflects on and validates its own work)
   - `medium` or `low` for routine work; lower effort on Fable 5 still performs well and often exceeds `xhigh` on prior models

   If a task completes but takes longer than necessary, lower effort rather than rewriting the prompt.
5. **Prevent overplanning and scope creep at higher effort.** Add: "When you have enough information to act, act. Do the simplest thing that works well. Don't add features, refactoring, or abstractions beyond what the task requires."
6. **State the boundaries.** Fable 5 can take unrequested actions. Add: "When the user is describing a problem or thinking out loud rather than requesting a change, the deliverable is your assessment. Report findings and stop; don't apply a fix until asked."
7. **Ground progress claims on long runs.** Add: "Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly." Anthropic testing shows this nearly eliminates fabricated status reports.
8. **Subagents are native.** Fable 5 dispatches parallel subagents readily and manages long-running ones reliably. Give delegation guidance ("delegate independent subtasks and keep working while they run") rather than spawn commands.
9. **Provide a memory surface for recurring workflows.** Fable 5 performs notably better when it can record lessons across runs, even a simple markdown notes file. Add: "Store one lesson per file with a one-line summary; update existing notes rather than duplicating; delete notes that turn out wrong."
10. **Never ask the model to echo its reasoning in the response.** "Show your thinking" or "explain your internal reasoning" instructions can trigger the reasoning_extraction classifier and cause fallback to Claude Opus 4.8. Self-check blocks (yes/no verification before responding) are fine; reasoning transcription is not. Audit migrated prompts for this.
11. **Expect longer turns.** Individual requests at higher effort can run for many minutes; autonomous runs for hours. For production: adjust timeouts, use streaming, check on runs asynchronously.
12. **API parameters.** Adaptive thinking only, summarized-only thinking output, no extended thinking budgets. Effort replaces manual thinking budgets as the cost and depth knob.
13. **Domain guardrails.** Fable 5 runs safety classifiers on offensive cybersecurity, biology and life sciences content, and reasoning extraction. Affected requests return `stop_reason: "refusal"` and should fall back to Claude Opus 4.8. Benign work in these domains can also trigger them; build prompts in those domains for Opus 4.8 instead.
14. **Brevity is steerable with one instruction.** Un-steered, Fable 5 can elaborate at higher effort. Add: "Lead with the outcome. Drop details that don't change what the reader would do next."

**Claude Fable 5 skeleton:**

```
<role>Act like [expert role + credentials].</role>

<intent>
I'm working on [larger task] for [audience]. They need [what the output enables].
</intent>

<task>
Your task is to [specific outcome].
Success criteria: [explicit, measurable]
</task>

<context>
[All inputs, data, background]
</context>

<output_format>
A) [section]
B) [section]
C) [section]
Length: [explicit cap]
</output_format>

<boundaries>
[What the model should and should not do; hard rules, exclusions]
</boundaries>

<self_check>
Before responding, verify:
1. [check]
2. [check]
3. [check]
</self_check>

Now produce the [deliverable]. Effort: [high | xhigh | medium | low].
```

What is missing on purpose versus the 4.7 skeleton: the `<steps>` block. Add explicit numbered steps only when the exact path matters. For Complex-tier long-running prompts, append the **Long-Run Pack**: progress grounding (principle 7), boundary statement (principle 6), subagent guidance (principle 8), and memory note (principle 9).

### Route C: Claude Opus 4.8 Optimization

Claude Opus 4.8 is the flagship Opus-tier model ($5 in / $25 out per 1M, model string `claude-opus-4-8`, 1M context window at standard pricing). It performs well out of the box on Opus 4.7 prompts, so this route is mostly the classic structured Claude formula plus 4.8-specific tuning.

**Core principles:**
1. **Full structured skeleton works here.** Unlike Fable 5, Opus 4.8 tolerates and benefits from explicit numbered steps when order or completeness matters. Use the universal skeleton with `<steps>` included for Standard and Complex tiers.
2. **Literal instruction following, especially at lower effort.** It does not silently generalize one instruction to other items or infer requests you didn't make. State scope explicitly: "Apply this formatting to every section, not just the first one."
3. **Five effort levels.** `xhigh` for coding and agentic use; minimum `high` for intelligence-sensitive work; `medium` for cost-sensitive; `low` only for short, scoped, latency-sensitive tasks; `max` can deliver gains on the most demanding tasks but risks overthinking and diminishing returns. At `xhigh` or `max`, set a large max_tokens (start at 64K).
4. **If reasoning is shallow, raise effort, do not prompt around it.** If you must stay at `low` for latency, add: "This task involves multi-step reasoning. Think carefully through the problem before responding."
5. **Thinking is off unless you set adaptive thinking** (`thinking: {type: "adaptive"}`). Triggering is steerable in both directions with a short instruction.
6. **It favors reasoning over tool calls.** If you need more tool use (agentic search, knowledge work), raise effort or instruct explicitly when and why to use each tool.
7. **Remove forced status scaffolding.** It gives regular, high-quality progress updates natively. Delete "summarize after every 3 tool calls" style instructions.
8. **Tone defaults direct and opinionated.** Specify warmth if your use case needs it: "Use a warm, collaborative tone."
9. **Verbosity is calibrated to perceived task complexity.** If output length matters, cap it explicitly; positive examples of the right concision beat "don't be verbose" rules.
10. **Subagents: spawns fewer by default.** Give explicit guidance about when delegation is and is not warranted.
11. **Design house style alert.** For frontend or slide prompts, 4.8 defaults to cream backgrounds, serif display type, and terracotta accents. Generic "don't use cream" fails; either specify a concrete visual direction (palette hexes, typography, layout) or instruct it to propose 3 to 4 distinct directions before building.
12. **Code review prompts: coverage first.** 4.8 follows severity bars literally and may investigate a bug then not report it. For review harnesses, instruct: "Report every issue you find, including low-severity and uncertain ones, with confidence and severity per finding; filtering happens downstream."

**Skeleton:** use the universal skeleton from Step 2 in XML form (role, task, context, steps, output_format, requirements, constraints, self_check). It is the same shape as the legacy 4.7 skeleton.

### Route D: Claude Sonnet 4.6 Optimization

Claude Sonnet 4.6 is the volume production workhorse ($3 in / $15 out per 1M, model string `claude-sonnet-4-6`, 1M context at standard pricing). Near-Opus quality at 40% lower cost per token. Default Claude target for Simple and most Standard tier prompts.

**Core principles:**
1. **Best fit:** classification, extraction, RAG responses, summarization, content generation, standard coding help, and batch pipelines. Escalate to Opus 4.8 for complex reasoning or Fable 5 for long-horizon autonomy.
2. **Standard Claude formula applies in full.** XML tags, explicit instructions, positive framing, 3 to 5 examples in `<example>` tags, output format with length caps. Sonnet rewards precision; vague prompts produce vague output.
3. **Effort is supported and replaces budget_tokens.** `budget_tokens` is deprecated on Sonnet 4.6; control thinking depth with `effort` plus adaptive thinking. `high` is a sensible default; drop to `medium` or `low` for high-volume cost-sensitive pipelines once evals confirm quality holds.
4. **Prefilled responses are gone.** Prefilling the last assistant turn returns a 400 error on Claude 4.6 models. Migrate to structured outputs, enum tool fields for classification, or a direct system instruction such as: "Respond directly without preamble. Do not start with phrases like 'Here is...'"
5. **Context awareness.** Sonnet 4.6 tracks its remaining token budget. In agent harnesses with compaction, add: "Your context window will be compacted as it approaches its limit; do not stop tasks early due to token budget concerns."
6. **Pin for production.** High-volume Sonnet pipelines are exactly where silent router changes hurt most. Pin the snapshot and keep a 3 to 5 input regression set.

**Skeleton:** universal skeleton from Step 2 in XML form. For Simple tier (the most common Sonnet use), strip to Role + Task + Output Format + Constraints per the tier rules.

### Side-by-Side Decision Matrix

| Decision Point | GPT-5.5 | Claude Fable 5 |
|----------------|---------|-----------------|
| Process detail | Outcome first, light on steps | Intent + success criteria; steps only when the exact path matters |
| Structure tags | Markdown sections | XML tags |
| Length default | Concise (3 to 6 sentences) | Explicit cap + brevity instruction at higher effort |
| Reasoning visibility | Can ask freely | Never ask it to echo internal reasoning (classifier fallback risk) |
| Effort knob | None (one mode) | Set effort: high default, xhigh for capability-sensitive, medium/low routine |
| Subagent spawning | Auto in agentic mode | Native and parallel; give delegation guidance, not spawn commands |
| Status updates | Add pre-tool acknowledgment | Add progress grounding for long runs (audit against tool results) |
| Turn length | Fast | Many minutes at high effort; hours on autonomous runs |
| Restricted domains | None specific | Offensive cyber, bio/life sciences, reasoning extraction → falls back to Opus 4.8 |
| Best for | Speed, ecosystem, short interactive loops | Long-horizon autonomy, complex reasoning, coding, enterprise deliverables, first-shot correctness |
| Pricing | $5 in / $30 out per 1M | $10 in / $50 out per 1M (90% prompt caching input discount; US-only inference at 1.1x) |

### Claude Family Quick Compare

| Decision Point | Fable 5 (Route B) | Opus 4.8 (Route C) | Sonnet 4.6 (Route D) |
|----------------|-------------------|--------------------|-----------------------|
| Scaffolding style | Intent + boundaries; steps degrade output | Full skeleton; explicit steps welcome | Full skeleton; precision rewarded |
| Effort default | `high` (xhigh capability-sensitive) | `xhigh` coding/agentic, `high` minimum elsewhere, `max` exists | `high`, step down for volume once evals hold |
| Subagents | Dispatches readily, manage with guidance | Spawns fewer, steer explicitly | Use as subagent under an Opus/Fable orchestrator |
| Turn length | Minutes to hours | Standard to long | Fast |
| Special watch-outs | Reasoning-echo, classifier domains, overplanning | Literal severity bars, design house style | No prefill (400 error), budget_tokens deprecated |
| Pricing per 1M | $10 / $50 | $5 / $25 | $3 / $15 |
| Sweet spot | Hardest, longest, most ambiguous work | Complex reasoning, agentic coding, knowledge work | Volume production, batch, RAG, classification |

### Workload to Model Recommendations (with Defaults)

| Workload | Default Model | Why | Alternative |
|----------|---------------|-----|-------------|
| Customer intelligence briefs | **Fable 5, effort `high`** | Depth, structure, reliable synthesis, strong enterprise document output | Opus 4.8 `high` for near-equal quality at half the price |
| Coding and engineering | **Fable 5, effort `xhigh`** | First-shot correctness, multi-day autonomous sessions, large migrations | Opus 4.8 `xhigh` for standard engineering; GPT-5.5 for short scripts |
| Long agentic workflows | **Fable 5, effort `high` + Long-Run Pack** | Long-horizon autonomy and instruction retention are its defining strengths | Opus 4.8 `xhigh` for shorter agentic runs |
| Volume production (classification, RAG, extraction, summaries) | **Sonnet 4.6** | Near-Opus quality at $3/$15; the cost-effective production default | Haiku 4.5 for the simplest routing/extraction |
| Cost-sensitive batch | **Sonnet 4.6, effort `medium` + batch API** | 40% cheaper than Opus per token, 50% batch discount | Fable 5 at `low` when quality bar demands it |
| High-volume customer-facing agents | **GPT-5.5** | Token economy, conversational defaults | Sonnet 4.6 with explicit warmth + length specs |
| Strategic analysis | **Fable 5, effort `high` or `xhigh`** | Complex multi-threaded reasoning; at `xhigh` it validates its own work | Opus 4.8 `high`/`max` for expensive-to-fail single analyses |
| Fast iteration prototypes | **GPT-5.5** | Concise output, fast turnaround | Sonnet 4.6 for cheap fast Claude iterations |
| LinkedIn / external comms | **GPT-5.5** | Tighter, more natural prose | Opus 4.8 with explicit voice samples (its default is direct and opinionated) |
| Research with citations | **Fable 5, effort `high`** | Citation discipline plus grounded progress claims | Opus 4.8 `high` with explicit tool-use triggering |
| Code review harness | **Opus 4.8, effort `xhigh`** | Best bug-finding recall and precision; use the coverage-first prompt | Fable 5 for repo-wide multi-day audits |
| Frontend / slides / design | **Opus 4.8** | Strong design instincts; specify a concrete direction to escape the house style | Fable 5 for full app builds |
| Security or bio domain work | **Claude Opus 4.8** | Fable 5 classifiers fall back to Opus 4.8 here anyway; target it directly | GPT-5.5 |

When dual or multi output is requested, use this matrix to pick the default. State the recommendation explicitly.

---

## Reference: The Personalization Layer

A proper prompt for a known user is not a generic template with brackets. It is a tailored instrument.

### Identity hardcoding rules

| Element | If known | If unknown |
|---------|----------|------------|
| User's name | Hardcode (e.g., "Jane Doe") | `[USER NAME]` |
| User's role | Hardcode (e.g., "VP Marketing at Acme Corp") | `[YOUR ROLE]` |
| User's company | Hardcode (e.g., "Acme Corp") | `[COMPANY]` |
| Customer being analyzed | `[CUSTOMER NAME]` (varies per use) | `[CUSTOMER NAME]` |
| Topic of post/email | `[TOPIC]` (varies per use) | `[TOPIC]` |
| Data block | `<data>[PASTE DATA]</data>` | `<data>[PASTE DATA]</data>` |

### Persistent preference scan

Before finalizing any prompt, scan memory for preferences in these categories:

| Category | Example preferences |
|----------|---------------------|
| Voice and tone | Direct, conversational, no corporate filler |
| Format rules | No em dashes, plain text for LinkedIn, BCG styling for docs |
| Domain conventions | Key terms never abbreviated, industry-specific naming rules, vendors referred to by category rather than name |
| Workflow rhythm | Regional work week, fixed weekly review day |
| Quality bars | LinkedIn posts scored in 5 categories 1-100 |
| Org/people specifics | Role titles used internally vs externally, naming conventions for account owners |

If any apply to the prompt category, bake them in as constraints or requirements. Do not require the user to remind you.

### Anonymization flag

If the prompt is being built to share or templatize:
- Flag the hardcoded identity values clearly: "Hardcoded for [the active user] at [their company]. Replace before sharing."
- Suggest the variables to swap in: `[USER NAME]`, `[COMPANY]`, `[ROLE]`.

---

## Reference: The Flip Negatives Protocol

**Trigger:** 3 or more "don't", "never", "avoid", "do not" lines in either the input prompt or the output prompt.

**Protocol:**

1. **Extract.** List every negative.
2. **Convert each.** For each negative, choose one:
   - **Positive instruction:** "Do not be too long" → "Keep under 150 words"
   - **Positive example:** Add a `<example_good>` block showing the ideal pattern
   - **Measurable success criterion:** "Don't be salesy" → "Reads as a peer note, not a sales pitch"
3. **Re-count.** Count negatives in the output prompt. If still 3+, flip again or justify each surviving negative.
4. **Surface to user.** State explicitly: "I converted X negatives into Y positive examples and Z measurable criteria."

**When a negative MUST stay:**
- Hard ethical/safety constraints ("Never fabricate statistics")
- Domain-specific exclusions that have no positive form ("Do not list partner platforms as competitors")

These are exempt but should be ≤ 3 total.

---

## Reference: The Dual Output Default Protocol

When the user requests versions for two or more models (any mix of GPT-5.5, Claude Fable 5, Claude Opus 4.8, Claude Sonnet 4.6):

1. **Produce one prompt per model.** Separate files, each labeled with its target model and route.
2. **Use the workload-to-model matrix to pick a default.** Match the prompt category against the matrix.
3. **State the default recommendation explicitly:**
   > "For [workload type], I recommend defaulting to [model] [with effort=X]. Use the [other model] version when [specific use case]."
4. **Brief comparison.** 3 to 5 bullets on what differs functionally between the two versions (not just structure — what behavior should the user expect to differ).

**Example default statement:**
> "For customer intelligence briefs, default to Claude Fable 5 at effort=`high` for the depth and explicit structure these briefs need. Use the GPT-5.5 version when you need a fast first-pass during a live call or when running batch generation across 50+ accounts where output token economy matters."

---

## Reference: Quality Scoring Rubric

Score each dimension 1 to 10. Total = /100.

| # | Dimension | 1 (poor) | 10 (excellent) |
|---|-----------|----------|----------------|
| 1 | **Role specificity** | "Act like a writer" | "Act like a senior B2B copywriter with 15 years of experience writing for SaaS, with proven track record of $50M+ attributed revenue" |
| 2 | **Task clarity** | "Help with my email" | "Write a 150-word reply to [CUSTOMER] addressing their concern about Q2 pricing, that lands as peer-to-peer, with one concrete next step" |
| 3 | **Input structure** | Prose blob with data inline | Structured `<context>` blocks, tables, or fenced JSON |
| 4 | **Output format precision** | "Make it good" | Section A/B/C with headers, length caps, table column specs |
| 5 | **Constraint hierarchy** | Mixed must-have and nice-to-have | Hard constraints separated from soft preferences, ≤ 3 negatives |
| 6 | **Self-check rigor** | None | 2 to 4 yes/no checks targeting the most likely failure modes |
| 7 | **Length calibration** | Way off target for model+tier | Right in band for model and tier |
| 8 | **Model fit** | Markdown for Claude or XML for GPT-5.5 | Optimized native pattern for the target model |
| 9 | **Personalization** | `[USER NAME]` for known user; ignores documented preferences | Hardcoded identity, persistent prefs baked in, anonymization flagged |
| 10 | **Iteration readiness** | Monolithic; hard to evolve | Modular blocks; iteration protocol referenced |

**Scoring bands:**
- **90 to 100:** ship to production
- **80 to 89:** ship to draft, light revision welcome
- **60 to 79:** revise the lowest 2 dimensions, then re-score
- **Below 60:** restart from skeleton

When delivering, always show the score and call out the 2 lowest dimensions with a one-line fix suggestion.

---

## Reference: Iteration Protocol

If a prompt fails on first run, diagnose against this taxonomy and apply the named protocol.

| Failure pattern | Protocol | Fix |
|-----------------|----------|-----|
| Output too generic | **A — Role sharpening** | Add years, credentials, specific track record |
| Output hallucinates facts | **B — Constraint tightening** | Add "base only on `<context>`" + citation rules + "never fabricate" |
| Output wrong format | **C — Format restructuring** | Specify sections, length, table columns explicitly |
| Output rambles | **D — Length capping** | Add hard cap (e.g., "150 words max") + verbosity spec |
| Output asks too many clarifying questions | **E — Scope expansion** | Add exhaustive scope + "do not ask clarifying questions" |
| Output ignores instructions | **F — Step splitting** | Break complex instructions into smaller numbered steps; remove contradictions |
| Output misses key data | **G — Input restructuring** | Move raw data into `<context>` block, structure as table or JSON |
| Output inconsistent across runs | **H — Snapshot + effort lock** | Pin to model snapshot; for Fable 5, lower or stabilize effort |
| Output shallow on Fable 5 | **I — Effort raising** | Raise effort to `high` or `xhigh`, do not prompt around it |
| GPT-5.5 ignored your steps | **J — Process stripping** | Delete steps, lead with outcome and success criteria |
| Fable 5 output degraded after migrating an old prompt | **K — Scaffolding strip** | Remove prescriptive steps and legacy guardrails; restate as intent + success criteria + boundaries; remove any reasoning-echo instructions |
| Fable 5 fabricates progress on long runs | **L — Progress grounding** | Add the audit-against-tool-results instruction from Route B principle 7 |
| Opus 4.8 review misses bugs it clearly investigated | **M — Coverage-first review** | Replace severity bars with "report every finding with confidence and severity; filter downstream" |
| Opus 4.8 applied instruction to one item only | **N — Scope explicitization** | State scope explicitly: "apply to every section, not just the first" |
| Sonnet 4.6 pipeline broke on migration | **O — Prefill removal** | Replace prefilled assistant turns with structured outputs or direct system instructions |

**Rule:** rarely needs full rewrite. Usually adjusting 1 to 2 sections fixes the issue. Start with the protocol most closely related to the failure.

---

## Reference: Self-Consistency Prompting (Complex Tier)

For high-stakes reasoning tasks (Tier 3 only):

**When to use:**
- Strategic recommendations with high cost of error
- Financial or risk analysis
- Multi-factor decisions where confidence matters

**How to use:**
- Run the same prompt 3 to 5 times
- Compare outputs for consistency
- Where they agree, confidence is high; where they diverge, dig deeper

More valuable on Claude Fable 5 at `high` or `xhigh` effort because each run is more expensive and the divergence signal is meaningful. Note that at `xhigh`, Fable 5 already reflects on and validates its own work, so reserve multi-run self-consistency for decisions where the cost of error clearly justifies 3 to 5 runs at $50 per 1M output tokens.

---

## Reference: Context Engineering Layer

**Definition:**
- Prompt engineering: what you write inside the context window
- Context engineering: what you put into the context window in the first place

**Anthropic's framing:** filling the context window with just the right information for the next step.

**Core techniques:**
1. **RAG.** Pull relevant documents into context dynamically rather than embedding them in the prompt.
2. **Summarization.** Compress prior conversation, long documents, or tool outputs into structured summaries.
3. **Structured inputs.** JSON, XML, tables instead of prose.
4. **Modular context blocks.** Distinct sections, not blobs.

**Implication:** push raw data into context. Keep prompt focused on instructions.

---

## Reference: Prompt Type Adaptation

| Prompt Type | Emphasize | De-emphasize |
|-------------|-----------|--------------|
| Analytical (churn, data review) | Data handling, edge cases, "base only on provided data" | Tone guidance |
| Communication (emails, posts) | Tone, audience psychology, word limits, active voice | Data grounding |
| Research (intelligence, market scans) | Citations, time bounds, source quality, scope | Word limits |
| Creative (content, copy, naming) | Voice, style, examples of good vs bad, rhythm | Rigid structure |
| Process (meeting prep, QBR, 1:1) | Checklists, owners, deadlines, completeness | Creative constraints |

---

## Reference: Research Additions

For research, intelligence, or fact-based prompts, layer in:

- **Time bounds:** "Prioritize sources from the last [X] days"
- **Citation requirements:** "Cite sources for all non-obvious claims with links"
- **Source quality:** Primary sources beat aggregators; official docs beat blog summaries
- **Never fabricate constraint:** Hard rule, not a suggestion
- **Exhaustive scope listing:** Every channel, segment, region the user cares about

---

## Reference: Treat Prompts Like Code

- Version them.
- Test against fixed inputs.
- Pin production prompts to specific model snapshots (e.g., `claude-fable-5` or its dated snapshot once published, `gpt-5.5-2026-04-23`).
- Try zero-shot before reaching for few-shot.
- Measure output quality across versions with a defined eval set.

**Eval pack guidance for Tier 3:** keep 3 to 5 fixed inputs as your regression set. Re-run on every prompt change.

---

## Reference: Anti-Patterns to Fix

| Anti-Pattern | Fix |
|--------------|-----|
| Vague request ("Help with my presentation") | Specify type, audience, outcome, length |
| Missing role | Add "Act like [specific expert]" |
| Wall of text, no structure | Use modular blocks |
| No output format | Define sections with format specs |
| No citation requirement (research) | Add "cite sources for all claims with links" |
| No time-bounding (research) | Specify "last 90 days" with date requirement |
| Unclear scope | List every channel, segment explicitly |
| Too short (under 100 words for Standard tier) | Expand to model-appropriate target |
| Too long (over target for tier+model) | Cut to essentials |
| No self-check (Standard or Complex tier) | Add 2 to 3 yes/no quality checks |
| Heavy "don't" stack (3+) | Run Flip Negatives Protocol |
| Process-heavy on GPT-5.5 | Strip steps, lead with outcome |
| Process-heavy on Fable 5 | Strip steps to intent + success criteria + boundaries (Protocol K) |
| Reasoning-echo instruction on Fable 5 ("show your thinking") | Remove; it can trigger classifier fallback to Opus 4.8. Use a yes/no self-check instead |
| Offensive cyber or bio/life sciences prompt targeted at Fable 5 | Retarget to Claude Opus 4.8 |
| Markdown structure for Fable 5 | Convert to XML tags |
| XML tags for GPT-5.5 | Convert to markdown sections |
| Unspecified effort on Fable 5 | Set effort: `high` default, `xhigh` capability-sensitive, `medium`/`low` routine |
| Unspecified effort on Opus 4.8 | Set effort: `xhigh` coding/agentic, `high` minimum for intelligence-sensitive work |
| Severity bar in an Opus 4.8 review prompt ("only high-severity") | Switch to coverage-first; filter downstream (Protocol M) |
| Prefilled assistant turn targeting Claude 4.6 models | Remove; returns a 400 error. Use structured outputs or direct instructions (Protocol O) |
| `budget_tokens` on Sonnet 4.6 or Opus 4.6 | Deprecated; use effort + adaptive thinking |
| Frontend/slide prompt for Opus 4.8 without a visual direction | Specify palette, typography, layout, or instruct it to propose 3 to 4 directions first |
| Opus-tier model for volume classification/RAG/extraction | Downshift to Sonnet 4.6 ($3/$15) once evals confirm quality holds |
| Long-running Fable 5 prompt without progress grounding | Add the audit-against-tool-results instruction |
| No snapshot pinning for production | Add `claude-fable-5`, `claude-opus-4-8`, `claude-sonnet-4-6` (dated snapshots once published) or `gpt-5.5-2026-04-23` |
| Raw data dumped in prompt | Move to structured context block |
| `[USER NAME]` for known user | Hardcode the active user's identity |
| Ignoring documented user preferences | Run Personalization Scan |
| Full Standard scaffolding on a Simple-tier task | Strip to Role + Task + Output Format |
| Quality score below 60 | Restart from skeleton |

---

## What to Watch in 2026

- **Snapshot pinning.** Live router behavior changes silently. Always pin production prompts.
- **Effort recalibration on Fable 5.** `high` is the new default; lower effort on Fable 5 often beats `xhigh` on prior Opus models. Old effort settings carried over from 4.7 prompts are probably overspending.
- **Over-prescription debt.** Prompts and skills tuned for prior models are often too prescriptive for Fable 5 and degrade output. Audit and strip on migration (Protocol K).
- **Classifier fallback.** Fable 5 routes offensive cyber, bio/life sciences, and reasoning-extraction requests to Claude Opus 4.8, and benign work in those domains can trigger it. Target Opus 4.8 directly for those workloads.
- **Token economy.** Fable 5 is $10 in / $50 out per 1M, with a 90% prompt caching input discount. GPT-5.5 remains cheaper on output. Fable 5 partially offsets the price with fewer tool calls and lower token consumption per task.
- **Turn length.** Fable 5 turns run minutes to hours at higher effort. Production harnesses need async checking, not blocking waits.
- **Memory surfaces.** Fable 5 improves meaningfully when given a place to record lessons across runs. Bake a memory note into recurring agentic prompts.
- **Opus 4.8 literalism.** It follows scope and severity instructions exactly, especially at lower effort. Review prompts written for older, looser models may quietly underreport or underapply.
- **Sonnet 4.6 as the production default.** Near-Opus quality at $3/$15 with the full 1M context at standard pricing. Most Simple and Standard tier prompts belong here, not on Opus.
- **Prefill deprecation.** Claude 4.6 models and later reject prefilled assistant turns with a 400 error. Audit legacy pipelines on migration.
- **Context engineering tooling.** The best prompt is increasingly less about what you write and more about what you feed the model.
- **GPT-5.5 Pro vs base.** Capabilities differ; pick deliberately.

---

## Templates and Examples

For full templates (Research, Analysis, Document Creation, Problem Solving, Review, Feedback) and detailed before/after examples, read:

→ `references/knowledge-base.md`

---

## Credits

- **Prompt Maker Method:** Ruben Hassid
- **v3.0:** Removed "take a deep breath" closing, added prompt type adaptation, variable/hardcode guidance, standardized comparison dimensions, iteration guide.
- **v4.0:** Added model-aware routing for GPT-5.5 and Claude 4.7 Opus based on April 2026 prompting guides; context engineering layer; modular architecture; flipped negatives principle.
- **v5.0:** Unified routing with 2026 research base. Five Steps spine. Effort calibration. `task_budget` guidance. Self-consistency prompting. Workload-to-model matrix. Snapshot pinning.
- **v6.0:** Implemented three test-surfaced fixes (personalization layer, flip-negatives output check, dual output default recommendation) plus five architect-level upgrades: complexity tiers (Simple/Standard/Complex), Quality Scoring Rubric (10 dimensions × 10 points), named Iteration Protocols (A through J), Quick Reference Decision Tree at the top, and the formal Personalization Layer with identity hardcoding rules and persistent preference scan.
- **v7.0:** Replaced the Claude 4.7 Opus route with Claude Fable 5 (first Mythos-class model, launched June 2026), based on Anthropic's official Fable 5 prompting guide. Key changes: intent-first scaffolding instead of step-heavy prompts, new effort defaults (high default, xhigh capability-sensitive), the Long-Run Pack for Complex-tier agentic prompts (progress grounding, boundaries, subagents, memory), reasoning-echo prohibition and classifier fallback awareness (offensive cyber, bio/life sciences → Opus 4.8), updated pricing ($10 in / $50 out per 1M), new Iteration Protocols K (scaffolding strip) and L (progress grounding), and a refreshed workload-to-model matrix. Claude Opus 4.8 and 4.7 moved to the "other model" route.
- **v7.1:** Added Route C (Claude Opus 4.8) and Route D (Claude Sonnet 4.6) based on Anthropic's official Opus 4.8 prompting guide and migration docs. Added the Claude Family Selector and Quick Compare tables, expanded the workload matrix to four models (including code review and frontend/design rows), new Iteration Protocols M (coverage-first review), N (scope explicitization), and O (prefill removal), and anti-patterns for severity bars, prefills, deprecated budget_tokens, and the Opus 4.8 design house style.

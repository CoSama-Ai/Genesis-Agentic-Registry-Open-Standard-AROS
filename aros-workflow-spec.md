# AROS Workflow Specification
**Version 1.0 | CoSama Inc. | 2026**
`aros.cosama.co`

---

## What a Workflow Is

An AROS Workflow is a structured, reusable operating plan that defines a mission objective, the context-gathering interactions required before autonomous execution begins, the conditional decision points at which the agent pauses for human input, and the completion criteria that determine when the mission is done.

**The Workflow is the agentic harness.** It is not the agent. It governs the agent — giving it a mission, context, operating boundaries, and a definition of done.

---

## The Problem Workflows Solve

An agent with a `content-writing` skill can write LinkedIn posts. But dropped into a task cold — no context, no declared goals, no user voice — it guesses. And when agents guess, quality collapses.

Without a workflow, the agent cannot answer:

- What is your brand voice?
- Who is your audience?
- What is the goal of this post — leads, authority, traffic?
- What have you already posted — what is the content arc?
- Are there competitors, phrases, or topics to avoid?

A workflow answers these questions before the agent starts. That is its primary job.

---

## The Missing Layer

| Layer | Standard | What It Covers |
|---|---|---|
| Communication | MCP — Anthropic | How agents connect to external tools and data |
| Instruction | AGENTS.md — Anthropic + OpenAI | How agents receive task context |
| Capability | Agent Skills — Anthropic | What expertise agents can load and invoke |
| Identity | AROS Agent Manifest | Who the agent is, who owns it |
| **Execution governance** | **AROS Workflow — CoSama** | **What the mission is, what context to gather, when to act, when to ask** |

No standard exists for the execution governance layer. AROS Workflows fill it.

---

## Workflow vs. Agent vs. Skill

| Concept | What It Is | Analogy |
|---|---|---|
| **Tool** | A callable function. Atomic. No autonomy. | A wrench |
| **Skill** | A bundled capability package. Expertise + tools + contract. | A trained technique |
| **Agent** | An autonomous entity with a declared identity that loads skills and invokes tools. | A worker |
| **Workflow** | The mission plan. Governs the agent — what to accomplish, what context to gather, when to act, when to ask. | A project brief + operating procedure |

> The same `content-writing` agent behaves entirely differently under a `linkedin-thought-leadership-post` workflow versus a `product-launch-announcement` workflow. The agent's capability is constant. The workflow defines the mission.

---

## The Five Components of a Workflow

Every AROS Workflow must declare all five. Remove any one and it cannot be registered.

| Component | Description | What Fails Without It |
|---|---|---|
| **Mission** | A declared objective and success criteria. What done looks like — not a prompt, an outcome. | No stopping condition — agent loops or stops arbitrarily |
| **Context Interview** | Structured questions the workflow asks the user before the agent begins work. Fills the context gap. | Agent guesses user intent — generic output, low quality |
| **Execution Plan** | Ordered steps — agent actions, skill invocations, tool calls — that accomplish the mission. | No defined path from start to finish |
| **Interaction Gates** | Declared checkpoints where the agent pauses and surfaces to the user before proceeding. | Runaway autonomous execution on high-stakes tasks |
| **Completion Signal** | The declared condition that marks the mission as done. | No verified end state |

---

## The Context Interview

The Context Interview is the defining feature that separates AROS Workflows from all other agentic frameworks. It is a structured pre-execution dialogue that fills the agent's context gap before autonomous work begins.

Every workflow declares a context profile — the minimum information the agent needs to do the job well. The workflow engine collects this through a structured interview before handing control to the agent.

### Interview Question Types

| Type | Description | Example |
|---|---|---|
| **Required** | Must be answered before execution begins. Blocks if skipped. | "What is the primary goal of this post?" |
| **Conditional** | Only asked if a prior answer triggers it. | If goal = `lead-gen`, ask: "What is the CTA URL?" |
| **Optional** | Enriches quality but does not block execution. Uses defaults if skipped. | "Any hashtags or topics to avoid?" |
| **Persistent** | Stored in the workflow's user profile. Not re-asked on subsequent runs. | "What is your brand voice?" — asked once, saved forever |
| **Derived** | Automatically inferred from user history or AROS profile. Never asked. | `industry: SaaS` — pulled from account profile |

> **Persistent context is the compounding advantage.** The first time a user runs a LinkedIn workflow, they answer 8 questions. The tenth time, they answer 1. The agent gets smarter about the user over time — not because the model improves, but because the workflow accumulates context.

---

## Interaction Gates

An Interaction Gate is a declared checkpoint where the workflow suspends autonomous execution and surfaces to the user. Gates are not failures — they are intentional design decisions that keep the user in the loop on tasks where autonomous completion carries risk.

| Gate Type | Description | Default Trigger |
|---|---|---|
| **Approval Gate** | Agent presents output and waits for explicit user approval before proceeding. | Always, or when `confidence < threshold` |
| **Review Gate** | Agent presents a draft and allows edits before continuing. Edits are fed back into context. | On first draft of any user-facing content |
| **Ambiguity Gate** | Agent encounters a decision it cannot resolve with available context. Surfaces the question. | When required context is missing mid-execution |
| **Risk Gate** | Agent is about to take an irreversible action: publish, send, pay, delete. Must be confirmed. | On ALL write/send/pay/delete actions — always fires |
| **Quality Gate** | Agent's self-evaluation score falls below the workflow's declared threshold. | When `output_confidence < workflow.quality_threshold` |
| **Escalation Gate** | Agent has exceeded retry limit or encountered a tool failure. Escalates to orchestrator or human. | After N failed retries, or on `SKILL_TOOL_UNAVAILABLE` |

### Gate Behavior by Autonomy Level

| Gate Type | L1 Assisted | L2 Supervised | L3 Conditional | L4 Delegated | L5 Orchestrating |
|---|---|---|---|---|---|
| Approval Gate | Always fires | Always fires | Fires if uncertain | Skipped | Skipped — sub-agent |
| Review Gate | Always fires | Always fires | Always fires | First run only | First run only |
| Ambiguity Gate | Always fires | Always fires | Always fires | Always fires | Routes to sub-agent |
| Risk Gate | Always fires | Always fires | Always fires | **Always fires** | **Always fires** |
| Quality Gate | Always fires | Fires if < 0.9 | Fires if < 0.85 | Fires if < 0.75 | Routes to sub-agent |
| Escalation Gate | Always fires | Always fires | Always fires | Always fires | Routes up hierarchy |

> **Risk Gate always fires, regardless of autonomy level.** No agent publishes, sends, pays, or deletes without an explicit confirmation. This is non-negotiable.

---

## Workflow Manifest Schema

### Required Fields

| Field | Type | Description |
|---|---|---|
| `workflow_id` | string | Full AROS workflow ID including version |
| `name` | string | Lowercase, hyphens, max 64 chars |
| `version` | string | Semver `major.minor.patch` |
| `namespace` | string | AROS domain extension |
| `description` | string | What this workflow accomplishes |
| `author` | string | AROS ID of the workflow author |
| `status` | enum | `active \| beta \| deprecated` |
| `workflow_type` | enum | See Workflow Types below |
| `mission` | object | Objective, output type, success criteria |
| `agents` | list[AgentRef] | Agents this workflow invokes |
| `context_interview` | object | Interview questions declaration |
| `execution_plan` | list[Step] | Ordered execution steps |
| `completion` | object | Completion condition and outputs |

### Workflow ID Format

```
aros.workflows.{namespace}.{name}@{version}
```

Examples:
```
aros.workflows.content.linkedin-thought-leadership-post@1.3.0
aros.workflows.sales.outbound-prospecting-sequence@2.0.0
aros.workflows.operations.new-client-onboarding@1.1.4
```

### Optional Fields

| Field | Type | Description |
|---|---|---|
| `autonomy_cap` | integer | Caps agent autonomy level (1–5) for this workflow's duration |
| `quality_threshold` | float | Triggers Quality Gate if agent self-score falls below |
| `persistent_context` | list[string] | Question IDs whose answers are stored across runs |
| `compliance` | list[string] | e.g. `["GDPR", "SOC2"]` |
| `pricing.model` | enum | `per_run \| subscription \| free` |
| `pricing.rate` | float | Cost per run |
| `retry.max_retries` | integer | Default `2` |
| `retry.backoff_ms` | integer | Delay between retries |
| `retry.on_max_retries` | string | Action when exhausted — typically `escalation_gate` |
| `runtime.avg_duration_sec` | integer | Estimated average execution time |
| `runtime.max_duration_sec` | integer | Hard timeout |

---

## Complete Workflow Manifest Example

```yaml
# ═══════════════════════════════════════════════════════════
# AROS WORKFLOW MANIFEST v1.0
# ═══════════════════════════════════════════════════════════

workflow_id:   "aros.workflows.content.linkedin-thought-leadership-post@1.3.0"
name:          "linkedin-thought-leadership-post"
version:       "1.3.0"
namespace:     "content"
workflow_type: "creation"
description:   "Researches a topic, interviews the user on voice and goals, drafts a LinkedIn post, and delivers an approved final version."
author:        "cosama.aros"
status:        "active"

# ── MISSION ──────────────────────────────────────────────
mission:
  objective: >
    Produce a LinkedIn post that reflects the user's authentic brand voice,
    targets their declared audience, and achieves a stated engagement goal.
  output_type: "text/linkedin-post"
  success_criteria:
    - "Post is between 150 and 400 words"
    - "Brand voice score >= 0.85"
    - "User has explicitly approved the final draft"

# ── AGENTS ───────────────────────────────────────────────
agents:
  - agent_id: "agent:copywriter.content"
    role: "primary"
  - agent_id: "agent:web-researcher.research"
    role: "support"

# ── CONTEXT INTERVIEW ─────────────────────────────────────
context_interview:
  questions:
    - id: "brand_voice"
      text: "How would you describe your professional brand voice?"
      type: "persistent"
      input_type: "enum"
      values: [bold-direct, thought-leader, warm-human, technical-expert, other]
      required: true

    - id: "target_audience"
      text: "Who is the primary audience for this post?"
      type: "persistent"
      input_type: "text"
      required: true

    - id: "post_goal"
      text: "What is the primary goal of this post?"
      type: "required"
      input_type: "enum"
      values: [build-authority, generate-leads, share-insight, drive-traffic, announce-news]
      required: true

    - id: "topic"
      text: "What topic, idea, or story should this post be about?"
      type: "required"
      input_type: "text"
      required: true

    - id: "cta_url"
      text: "Is there a call-to-action link to include?"
      type: "conditional"
      condition: "post_goal == 'generate-leads' OR post_goal == 'drive-traffic'"
      input_type: "url"
      required: false

    - id: "avoid_topics"
      text: "Any competitors, topics, or phrases to avoid?"
      type: "optional"
      input_type: "text"
      required: false
      default: null

# ── EXECUTION PLAN ────────────────────────────────────────
execution_plan:

  - step: 1
    name: "Research topic"
    agent: "agent:web-researcher.research"
    skill: "aros.skills.research.web-research@2.0.1"
    input:
      query: "{context.topic}"
      depth: "surface"
    output: "research_brief"

  - step: 2
    name: "Draft post"
    agent: "agent:copywriter.content"
    skill: "aros.skills.content.content-writing@1.2.0"
    input:
      topic:       "{context.topic}"
      research:    "{outputs.research_brief}"
      brand_voice: "{context.brand_voice}"
      audience:    "{context.target_audience}"
      goal:        "{context.post_goal}"
      avoid:       "{context.avoid_topics}"
    output: "draft_v1"

  - step: 3
    name: "Quality self-check"
    action: "evaluate"
    checks:
      - "word_count between 150 and 400"
      - "brand_voice_score >= 0.85"
      - "no avoided topics present"
    on_fail: "retry step 2, max 2 retries, then escalation_gate"
    output: "quality_report"

  - step: 4
    name: "User review"
    gate:
      type: "review_gate"
      message: "Here is your draft LinkedIn post. Review and approve, or give feedback for a revision."
      actions: [approve, revise, regenerate]
    on_approve:    "proceed to step 5"
    on_revise:     "inject feedback into context, retry step 2"
    on_regenerate: "clear draft, retry from step 1"

  - step: 5
    name: "Deliver final"
    action: "deliver"
    output_format: "text/linkedin-post"
    optional_actions:
      - schedule_post
      - copy_to_clipboard

# ── COMPLETION ────────────────────────────────────────────
completion:
  condition: "step 5 completed AND user_approved == true"
  outputs: ["final_post"]
  log_to_aros: true

# ── OPTIONAL CONFIGURATION ────────────────────────────────
autonomy_cap: 3
quality_threshold: 0.85

persistent_context:
  - brand_voice
  - target_audience
  - avoid_topics

compliance:
  - GDPR

pricing:
  model: "per_run"
  rate: 0.35
  currency: "USD"

retry:
  max_retries: 2
  backoff_ms: 1000
  on_max_retries: "escalation_gate"

runtime:
  avg_duration_sec: 45
  max_duration_sec: 180
```

---

## Workflow Types

Every workflow must declare one of five canonical types. Type determines default gate behavior and interview depth.

| Type | Description | Default Gate Behavior | Interview Depth |
|---|---|---|---|
| **Creation** | Produces a content artifact — post, document, email, report. | Review Gate required | Full — brand, audience, goal, topic |
| **Research** | Gathers, synthesizes, and delivers structured information. | Quality Gate on delivery | Moderate — scope, format, depth |
| **Operational** | Executes a multi-step business process — onboarding, routing, processing. | Risk Gate on irreversible actions | Minimal — trigger inputs only |
| **Analytical** | Analyzes data, generates insights, or produces a scored evaluation. | Review Gate on findings | Moderate — dataset, criteria, format |
| **Orchestration** | Manages and coordinates multiple sub-agents toward a shared objective. | Escalation Gates only | Minimal — top-level goal only |

---

## What Workflow Owns vs. What Agent Owns

| Responsibility | Owned By | Notes |
|---|---|---|
| What to accomplish | Workflow | Mission and success criteria |
| What context is needed | Workflow | Interview design |
| When to pause for the user | Workflow | Gate definitions |
| How to do the work | Agent + Skills | Execution method |
| What tools to call | Agent + Skills | Tool selection |
| Quality of the output | Agent + Skills | Within declared skill contract |
| Storing user preferences over time | Workflow | Persistent context |
| Compliance and audit logging | Both | Workflow declares; agent enforces |
| Pricing | Both | Workflow per-run; agent per-invocation |

---

## Workflow Lifecycle

| Status | Description | Marketplace Visible? | New Runs Allowed? |
|---|---|---|---|
| **Draft** | Being authored. Not published. | No | No |
| **Beta** | Published, unstable, no SLA. | Yes — beta flag | Yes |
| **Active** | Stable, production-ready. | Yes | Yes |
| **Deprecated** | Replaced. Sunset date declared. | Yes — warning | Yes — migration recommended |
| **Retired** | No longer available. | No | No |

---

## Validation Rules

AROS enforces all of the following at workflow registration:

1. `workflow_id` must be globally unique and match `aros.workflows.{namespace}.{name}@{version}`
2. `namespace` must be a valid AROS domain extension
3. `workflow_type` must be one of the five canonical types
4. All `agent_id` entries must resolve to active or beta agents in AROS
5. At least one Required interview question must be declared
6. Every execution step must declare an `output` variable
7. Each step's `input` references must resolve to either `context.*` or `outputs.*` from a prior step
8. At least one gate must be declared — OR `autonomy_cap` must be 4 or 5
9. A Risk Gate must be present for any step with `action: send | publish | pay | delete | write`
10. `completion.condition` must reference at least one output from the execution plan
11. If any compliance tag is set, `log_to_aros: true` is required in the completion block

---

## The AROS Object Hierarchy

```
Tool  →  Skill  →  Agent  →  Workflow  →  Mission
(function) (expertise) (entity) (mission plan) (running instance)
```

A Workflow without an Agent is a plan with no executor.
An Agent without a Workflow is a capable worker with no brief.
A Mission is the live, in-progress execution of a Workflow by an Agent.
When the completion condition is met, the Mission closes. The Workflow persists for the next run.

---

*AROS Workflow Specification v1.0 | CoSama Inc. | cosama.co | aros.cosama.co*

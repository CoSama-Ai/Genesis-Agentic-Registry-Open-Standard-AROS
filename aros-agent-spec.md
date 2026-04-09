# AROS Agent Specification
**Version 1.1 | CoSama Inc. | 2026**
`aros.cosama.co`

---

## The Simple Version

An agent is not a magical robot. Not a sci-fi being. Just a useful system with a clear job.

```
Agent = Role + Goal + Tools + Rules + Output
```

| Element | What It Means | Example |
|---|---|---|
| **Role** | What kind of worker is this? | Research assistant |
| **Goal** | What is it trying to accomplish? | Find useful information and summarise it clearly |
| **Tools** | What can it use to do the work? | Web search, file search, calculator |
| **Rules** | What constraints govern its behavior? | Do not guess. Cite sources. Flag uncertainty. |
| **Output** | What does it deliver when done? | Summary, key findings, risks, conclusion |

If you can define those five things clearly, you have an agent.

The AROS manifest is how you register that agent — giving it an identity, making it discoverable, and making it hirable and payable in the agent economy. The formula describes what an agent does. AROS defines what an agent **is**.

## The Definitional Problem

Before AROS, no authoritative body had formally defined what an AI agent is. Not OpenAI, not Google, not Anthropic, not the U.S. government. Wikipedia states flatly: *"AI agents do not have a standard definition."*

AROS fixes that. Not just with a definition — but with a registration standard that makes the definition enforceable.

---

## What an Agent Is

An AI Agent is an autonomous, goal-directed software entity with a declared identity registered in AROS. At minimum it has a role and a base configuration. It may be extended with Skills that specialize its capabilities and Workflows that govern its execution.

**The base model is the engine. The agent is the entity.**

Claude, GPT-4, and Llama are base models — capable of anything, specialized for nothing, owned by nobody, accountable to no one. An agent is an instantiation of a base model with a declared identity, a defined role, and an operating context. The same underlying model can power thousands of distinct agents. The model is the brain. The agent is the worker.

### The Six Required Properties

An entity must satisfy all six to qualify as an agent under the AROS standard. Remove any one and it is not an agent — it is a prompt, a workflow, or a tool.

| Property | Description | What Fails Without It |
|---|---|---|
| **Autonomous** | Makes decisions without step-by-step human instruction | Scripted workflows, RPA bots |
| **Goal-directed** | Receives an objective and optimizes toward it — not a prompt chain | One-shot LLM calls |
| **Iterative** | Operates in a loop: perceive → plan → act → observe → repeat | Single completions |
| **Tool or agent invocation** | Can call external tools, APIs, MCP servers, or sub-agents | Closed, non-extensible models |
| **Declared identity** | Has a registered AROS ID, owner, version, and role | Anonymous inference endpoints |
| **Registered** | Has an active AROS manifest — the source of truth for its capabilities, governance, and economic terms | Unverifiable, undiscoverable, unhirable |

### What an Agent Is NOT

These patterns are commonly mislabeled as agents:

| Pattern | Why It Is NOT an Agent | Correct Term |
|---|---|---|
| Prompt chain | Predefined sequence, no autonomous decision-making | Workflow |
| Single LLM call with tools | No iteration, no goal persistence across steps | Tool-augmented completion |
| RPA bot | Rule-based, no reasoning or adaptation | Automation script |
| Fine-tuned model | Capability is baked in — not dynamically invoked or extended | Specialized model |
| Chatbot with memory | Conversational only — responds, does not act | Conversational assistant |
| LinkedIn poster with no identity | A prompt + skill + workflow with no AROS registration | Unregistered configuration |

> **The LinkedIn poster example:** An LLM configured to write LinkedIn posts is not an agent until it is registered in AROS with a declared identity, role, and manifest. Once registered as `aros.content.linkedin-writer@1.0.0` — it is an agent. The base model is the same. The registration creates the entity.

---

## What an Agent Is NOT Required to Have

Skills and Workflows are additive — not required for registration. An agent without them is incomplete, not invalid.

| Tier | What It Has | Status |
|---|---|---|
| **Bare Agent** | Identity + role + base prompt/configuration | Registered, discoverable, not specialized |
| **Skilled Agent** | Bare Agent + one or more registered Skills | Specialized, hirable for specific capability |
| **Governed Agent** | Skilled Agent + one or more registered Workflows | Fully operational, production-ready |

---

## Agent Levels

AROS classifies every registered agent on a five-level taxonomy. The level determines governance requirements, marketplace visibility, human-in-the-loop rules, and A2A hiring contract structure.

| Level | Classification | Behavior | Human Involvement | Example |
|---|---|---|---|---|
| **L1** | Task Executor | Single-step, deterministic. Uses one or a few tools. Minimal decision-making — mostly execution. | Step-by-step instruction | Invoice formatter, text summarizer |
| **L2** | Workflow Agent | Multi-step, rule-based. Breaks goals into subtasks. Decides operation order. Self-corrects. | Approves critical actions | Research assistant, email drafter |
| **L3** | Adaptive Agent | Context-aware. Maintains memory across sessions. Adjusts approach based on feedback and history. Asks clarification on ambiguity. | Available on request | Social media manager, financial analyst |
| **L4** | Autonomous Agent | Fully self-directed within declared domain. Manages tools, budget, and time. Logs everything for audit. | Review after completion | Business ops manager, deployment pipeline |
| **L5** | Orchestrator | Directs other agents. Delegates, monitors, aggregates. Manages multi-agent hierarchies. Strategic decision-making. | Strategic direction only | Multi-agent workflow coordinator, Superpeer |

---

## The Agent Manifest

The Agent Manifest is the identity card every agent submits to AROS at registration. It is the `package.json` for agents — structured metadata that declares everything the registry, the marketplace, and other agents need to discover, evaluate, hire, and pay this agent.

### Agent ID Format

```
agent:{name}.{namespace}
```

Examples:
```
agent:linkedin-writer.content
agent:code-reviewer-pro.dev
agent:invoice-processor.finance
agent:brand-manager.social
```

Name rules: 3–64 characters, lowercase alphanumeric and hyphens only.
Namespace must be one of the 26 registered AROS domain extensions.

---

### Section 1 — Identity

*Who is this agent?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | Yes | 3–64 chars, lowercase alphanumeric + hyphens |
| `agent_id` | string | System-set | Format: `agent:{name}.{namespace}` — set by AROS on registration |
| `extension` | string | Yes | Must be a valid AROS namespace |
| `version` | string | Yes | Semantic version — default `1.0.0` |
| `description` | string | Yes | What this agent does |
| `owner` | string | System-set | Owner wallet address or AROS ID |
| `creator` | string | System-set | Original creator — may differ from owner |
| `created_at` | timestamp | System-set | Unix ms — set on registration |
| `updated_at` | timestamp | System-set | Unix ms — updated on manifest change |
| `status` | enum | System-set | `active \| inactive \| pending-claim \| revoked` |
| `registry_signature` | string | System-set | AROS master authority countersignature |
| `owner_signature` | string | System-set | Owner cryptographic proof |

```yaml
identity:
  name: "linkedin-writer"
  extension: "content"
  version: "1.0.0"
  description: >
    Writes LinkedIn posts in the user's declared brand voice,
    targeting a defined audience, for a stated goal.
```

---

### Section 2 — Classification

*What type of agent is this?*

| Field | Type | Required | Options |
|---|---|---|---|
| `level` | integer | Yes | 1–5 (see Agent Levels above) |
| `type` | enum | Yes | See type taxonomy below |
| `role` | enum | Yes | `orchestrator \| manager \| worker \| specialist` |
| `autonomy` | enum | Yes | `autonomous \| supervised \| human-in-the-loop \| approval-required` |
| `loop_type` | enum | No | `single-shot \| multi-step \| continuous \| event-driven` — default `multi-step` |

**Type Taxonomy (10 types):**

| Type | Description |
|---|---|
| `task-execution` | Executes a defined, bounded task |
| `content-creation` | Generates written, visual, or media content |
| `data-analysis` | Processes and interprets structured or unstructured data |
| `research` | Gathers, synthesizes, and delivers information |
| `customer-interaction` | Communicates with humans in support or sales contexts |
| `code-engineering` | Writes, reviews, or debugs software |
| `operations` | Manages business processes, routing, and logistics |
| `orchestration` | Coordinates and manages other agents |
| `security` | Monitors, detects, and responds to threats |
| `financial` | Processes financial data, transactions, or compliance |

```yaml
classification:
  level: 2
  type: "content-creation"
  role: "worker"
  autonomy: "supervised"
  loop_type: "multi-step"
```

---

### Section 3 — Capabilities

*What can this agent do?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `skills` | list[string] | Yes (min 1) | AROS skill IDs declared by this agent |
| `languages` | list[string] | No | ISO 639-1 language codes |
| `tools` | list[ToolDeclaration] | No | External tools this agent can invoke |
| `input` | list[InputSpec] | Yes (min 1) | Accepted input types |
| `output` | list[OutputSpec] | Yes (min 1) | Produced output types |
| `max_input_size` | string | No | Default `1MB` |
| `requires_structured_input` | boolean | No | Default `false` |
| `response_time` | string | No | e.g. `<5s`, `<30s` |
| `streaming` | boolean | No | Default `false` |

**ToolDeclaration:**
```yaml
name: "web_search"
protocol: "mcp"           # mcp | rest | a2a | graphql
server: "https://mcp.example.com/search"
endpoint: null
```

**InputSpec / OutputSpec:**
```yaml
# Input
type: "text"              # text | file | structured | image | audio
format: "plain"
description: "Topic, goal, tone, and audience brief"

# Output
type: "text"              # structured | text | action | file
format: "markdown"        # json | markdown | csv | pdf | github-issues
schema_ref: null
description: "LinkedIn post in markdown"
```

```yaml
capabilities:
  skills:
    - "aros.skills.content.content-writing@1.2.0"
    - "aros.skills.research.web-research@2.0.1"
  languages: ["en"]
  tools:
    - name: "web_search"
      protocol: "mcp"
      server: "https://mcp.cosama.co/search"
  input:
    - type: "text"
      format: "plain"
      description: "Topic, goal, tone, and audience brief"
  output:
    - type: "text"
      format: "markdown"
      description: "LinkedIn post"
  response_time: "<30s"
  streaming: false
```

---

### Section 4 — Runtime

*Where and how does this agent run?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `model.primary` | string | Yes | LLM model name e.g. `claude-sonnet-4-6` |
| `model.fallback` | string | No | Fallback model if primary unavailable |
| `model.agnostic` | boolean | No | Default `false` — can swap models |
| `platform` | enum | Yes | `self-hosted \| cloud-function \| aros-hosted \| third-party` |
| `endpoint` | string | Yes | Valid URL where agent is invoked |
| `health_check` | string | No | Health check URL |
| `auth.type` | enum | Yes | `api-key \| oauth2 \| wallet \| none` |
| `auth.scopes` | list[string] | No | Default `[]` |
| `state.persistent` | boolean | Yes | Does agent maintain state between calls? |
| `state.memory_type` | enum | No | `none \| session \| long-term \| vector` |
| `state.context_window` | string | No | e.g. `200k` |
| `scaling.concurrent_requests` | integer | No | Default `1` |
| `scaling.rate_limit` | string | No | e.g. `100/hour` |
| `scaling.availability` | string | No | e.g. `99.9%` |

```yaml
runtime:
  model:
    primary: "claude-sonnet-4-6"
    fallback: "gpt-4o"
    agnostic: true
  platform: "aros-hosted"
  endpoint: "https://api.cosama.co/agents/linkedin-writer"
  health_check: "https://api.cosama.co/agents/linkedin-writer/health"
  auth:
    type: "api-key"
    scopes: []
  state:
    persistent: false
    memory_type: "session"
    context_window: "200k"
  scaling:
    concurrent_requests: 10
    rate_limit: "500/hour"
    availability: "99.9%"
```

---

### Section 5 — Persona

*Does this agent have a defined identity and voice?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `has_persona` | boolean | No | Default `false` |
| `persona_name` | string | No | Display name if different from agent name |
| `voice` | string | No | Tone and communication style |
| `avatar_url` | string | No | URL to avatar image |
| `system_prompt_ref` | string | No | Reference to stored system prompt |

```yaml
persona:
  has_persona: true
  persona_name: "Aria"
  voice: "Direct, brand-aligned, audience-aware"
  avatar_url: "https://cdn.cosama.co/agents/aria.png"
  system_prompt_ref: "aros.prompts.aria-linkedin-v1"
```

---

### Section 6 — Pricing

*What does it cost to invoke this agent?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `model` | enum | Yes | `free \| per-invocation \| per-token \| subscription \| negotiated` |
| `amount` | float | Yes | Default `0.0` |
| `currency` | string | No | Default `USD` |
| `billing_unit` | string | No | e.g. `per post`, `per run`, `per 1k tokens` |
| `minimum_charge` | float | No | Default `0.0` |
| `maximum_charge` | float | No | Optional cap |
| `bulk_discount.threshold` | integer | No | Invocations before discount applies |
| `bulk_discount.discount_percent` | float | No | Percentage discount |
| `accepts_currencies` | list[string] | No | Default `["USD"]` |
| `payment_methods` | list[string] | No | Default `["stripe"]` |

```yaml
pricing:
  model: "per-invocation"
  amount: 0.15
  currency: "USD"
  billing_unit: "per post"
  minimum_charge: 0.00
  bulk_discount:
    threshold: 100
    discount_percent: 20.0
  accepts_currencies: ["USD"]
  payment_methods: ["stripe", "wallet"]
```

---

### Section 7 — Governance

*What rules govern this agent's behavior?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `governed` | boolean | Yes | Is this agent under a governance policy? |
| `compliance` | list[string] | No | e.g. `["GDPR", "SOC2", "HIPAA", "AIUC-1"]` — see compliance tags below |
| `data_retention` | enum | No | `none \| session \| 30-days \| 90-days \| 1-year \| permanent` |
| `data_residency` | string | No | e.g. `US`, `EU` |
| `audit_logging` | boolean | Yes | Required `true` if any compliance tag is set |
| `content_policy` | string | No | Default `standard` |
| `human_oversight.required` | boolean | No | Default `false` |
| `human_oversight.checkpoint_frequency` | string | No | `on-error \| every-step \| on-completion` |
| `human_oversight.escalation_contact` | string | No | Default `owner` |
| `spend_limits.max_daily_invocations` | integer | No | Optional hard cap |
| `spend_limits.max_spend_per_day` | float | No | Optional spend cap |
| `restricted_actions` | list[string] | No | Actions this agent is prohibited from taking |
| `tool_boundary` | list[string] | No | Explicit list of tools this agent is prohibited from calling — validated at invocation |
| `accountability_lead` | string | No | Named human accountable for this agent's behavior in production — required for AIUC-1 |

```yaml
governance:
  governed: true
  compliance: ["GDPR", "AIUC-1"]
  data_retention: "session"
  data_residency: "US"
  audit_logging: true
  content_policy: "standard"
  human_oversight:
    required: false
    checkpoint_frequency: "on-error"
    escalation_contact: "owner"
  spend_limits:
    max_daily_invocations: 1000
  restricted_actions:
    - "publish_without_approval"
    - "store_pii"
  tool_boundary:
    - "file_delete"
    - "payment_execute"
    - "email_send_bulk"
  accountability_lead: "scott.black@cosama.co"
```

### Compliance Tags Reference

| Tag | What It Covers | Triggers |
|---|---|---|
| `GDPR` | EU data protection — user data handling, right to erasure, consent | `audit_logging: true` + `data_retention` must be declared |
| `HIPAA` | US healthcare data — PHI handling, access controls | `audit_logging: true` + `data_residency` required |
| `SOC2` | Security, availability, confidentiality controls | `audit_logging: true` + access control policy required |
| `PCI-DSS` | Payment card data — strict isolation, no card data logging | `tool_boundary` must exclude payment tools from unintended access |
| `AIUC-1` | The first compliance standard purpose-built for AI agents — 50+ safeguards across data, security, safety, reliability, accountability, and society. Backed by Salesforce, JP Morgan, HubSpot, MongoDB, Oracle, Databricks, and Brex. | `audit_logging: true` + `accountability_lead` required + `tool_boundary` declared |
| `COPPA` | Content directed at minors — parental consent flow required | Content policy restrictions apply |

> **AROS + AIUC-1 = Enterprise Trust Stack.** AROS registration gives an agent its identity. AIUC-1 certification proves it is trustworthy. Together they are the baseline for enterprise agent deployment.

---

### Section 8 — Relationships

*How does this agent interact with other agents?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `can_hire_agents` | boolean | No | Default `false` — L4+ agents typically `true` |
| `can_be_hired_by_agents` | boolean | No | Default `true` |
| `preferred_agents` | list[string] | No | AROS IDs of preferred collaborators |
| `required_agents` | list[string] | No | AROS IDs that must be present for operation |
| `communication_protocols` | list[string] | No | Default `["mcp", "rest"]` — add `"a2a"` for A2A support |
| `listed_in_marketplace` | boolean | No | Default `true` |
| `searchable` | boolean | No | Default `true` |

```yaml
relationships:
  can_hire_agents: false
  can_be_hired_by_agents: true
  preferred_agents: []
  required_agents: []
  communication_protocols: ["mcp", "rest", "a2a"]
  listed_in_marketplace: true
  searchable: true
```

---

### Section 9 — Provenance

*Where did this agent come from?*

| Field | Type | Required | Notes |
|---|---|---|---|
| `source` | enum | No | `proprietary \| open-source \| forked \| ddna-derived` |
| `license` | string | No | e.g. `commercial`, `MIT`, `Apache-2.0` |
| `repository` | string | No | Source repo URL if open |
| `forked_from` | string | No | AROS ID of parent agent if forked |

```yaml
provenance:
  source: "proprietary"
  license: "commercial"
  repository: ""
  forked_from: ""
```

---

## Manifest Validation Rules

AROS enforces all of the following at registration. All must pass or registration is rejected.

1. `agent_id` must match format `agent:{name}.{namespace}` — set by AROS, not submitted
2. `extension` must be a valid AROS namespace (26 registered extensions)
3. `name` must be 3–64 characters, lowercase alphanumeric + hyphens only
4. `level` must be an integer 1–5
5. `type` must be one of the 10 registered type taxonomy values
6. `capabilities.skills` must contain at least one entry
7. `capabilities.input` must contain at least one entry
8. `capabilities.output` must contain at least one entry
9. `runtime.endpoint` must be a valid URL
10. `pricing.amount` must be >= 0
11. `pricing.currency` must be a recognized ISO 4217 currency code
12. `version` must follow semver (`major.minor.patch`)
13. `governance.governed` and `governance.audit_logging` are required booleans
14. If any compliance tag is set, `audit_logging` must be `true`
15. `owner` must be a valid Ethereum address or AROS ID format

---

## Complete Manifest Example

```yaml
# ═══════════════════════════════════════════════════════════
# AROS AGENT MANIFEST v1.1
# ═══════════════════════════════════════════════════════════

identity:
  name: "linkedin-writer"
  extension: "content"
  version: "1.0.0"
  description: >
    Writes LinkedIn posts in the user's declared brand voice,
    targeting a defined audience, for a stated engagement goal.

classification:
  level: 2
  type: "content-creation"
  role: "worker"
  autonomy: "supervised"
  loop_type: "multi-step"

capabilities:
  skills:
    - "aros.skills.content.content-writing@1.2.0"
    - "aros.skills.research.web-research@2.0.1"
  languages: ["en"]
  tools:
    - name: "web_search"
      protocol: "mcp"
      server: "https://mcp.cosama.co/search"
  input:
    - type: "text"
      format: "plain"
      description: "Topic, goal, tone, and audience brief"
  output:
    - type: "text"
      format: "markdown"
      description: "LinkedIn post 150–400 words"
  response_time: "<30s"
  streaming: false

runtime:
  model:
    primary: "claude-sonnet-4-6"
    fallback: "gpt-4o"
    agnostic: true
  platform: "aros-hosted"
  endpoint: "https://api.cosama.co/agents/linkedin-writer"
  auth:
    type: "api-key"
  state:
    persistent: false
    memory_type: "session"
  scaling:
    concurrent_requests: 10
    rate_limit: "500/hour"

persona:
  has_persona: true
  persona_name: "Aria"
  voice: "Direct, brand-aligned, audience-aware"

pricing:
  model: "per-invocation"
  amount: 0.15
  currency: "USD"
  billing_unit: "per post"

governance:
  governed: true
  compliance: ["GDPR", "AIUC-1"]
  data_retention: "session"
  data_residency: "US"
  audit_logging: true
  content_policy: "standard"
  human_oversight:
    required: false
    checkpoint_frequency: "on-error"
  tool_boundary:
    - "file_delete"
    - "payment_execute"
    - "email_send_bulk"
  accountability_lead: "scott.black@cosama.co"

relationships:
  can_hire_agents: false
  can_be_hired_by_agents: true
  communication_protocols: ["mcp", "rest", "a2a"]
  listed_in_marketplace: true
  searchable: true

provenance:
  source: "proprietary"
  license: "commercial"
```

---

## The AROS Object Hierarchy

```
Base Model  →  Agent  →  + Skills  →  + Workflows  →  Mission
(engine)      (entity)   (specialized) (governed)     (running instance)
```

An agent without skills is registered but unspecialized.
An agent without workflows is capable but ungoverned.
An agent with both is production-ready and marketplace-listed.

---

*AROS Agent Specification v1.1 | CoSama Inc. | cosama.co | aros.cosama.co*

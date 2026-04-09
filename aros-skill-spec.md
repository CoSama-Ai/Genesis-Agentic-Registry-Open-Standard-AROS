# AROS Skill Specification
**Version 1.0 | CoSama Inc. | 2026**
`aros.cosama.co`

---

## What a Skill Is

An Agent Skill is a self-contained, versioned capability package that an agent can discover, load, and invoke to execute a defined function — including the instructions, tools, data context, constraints, and interface contract required to perform that function reliably and repeatably.

**Skills are first-class objects in AROS.** They are not metadata on an agent. They are independently registered, versioned, discoverable, and composable assets that can be loaded by any compatible agent.

---

## Skill vs. Tool vs. Agent

These three concepts are frequently conflated. The distinction is fundamental:

| Concept | What It Is | Analogy | Example |
|---|---|---|---|
| **Tool** | A single callable function. Atomic. No autonomy. Defined inputs/outputs. | A wrench | `web_search()`, `send_email()`, `query_db()` |
| **Skill** | A bundled capability package. Uses tools. Carries expertise, best practices, and an interface contract. | A trained technique | `content-writing` skill: wraps `web_search` + `file_write` + brand guidelines + tone rules + output format spec |
| **Agent** | An autonomous entity with a declared identity that loads Skills and invokes Tools to fulfill a goal. | A worker | `linkedin-writer` agent loads `content-writing` skill, invokes `web_search` and `file_write` tools |

> **A tool does one thing. A skill knows how to use tools well. An agent knows which skills to load.**

---

## The Six Required Properties of a Skill

Every skill registered on AROS must satisfy all six. Remove any one and AROS will reject registration.

| Property | Description | Enforcement |
|---|---|---|
| **Declarative** | Skill describes WHAT it does, not HOW the agent performs it. Behavior is defined externally. | Required at registration |
| **Self-contained** | All instructions, dependencies, context, and constraints are bundled in the skill package. Nothing external required beyond declared tools. | Validated on upload |
| **Versioned** | Every skill has a semantic version (`major.minor.patch`), changelog, and a declared deprecation path. | Required field in manifest |
| **Composable** | Skills can be stacked with other skills without conflict. All dependencies are declared. | Dependency graph checked on load |
| **Contractual** | Skill declares its input schema, output schema, and defined failure modes. No ambiguity at invocation. | Schema validated at invocation |
| **Portable** | Works across any AROS-registered agent that meets the declared runtime requirements. Not locked to a single agent or platform. | Cross-platform tested on publish |

---

## Skill ID Format

```
aros.skills.{namespace}.{name}@{version}
```

Examples:
```
aros.skills.content.content-writing@1.2.0
aros.skills.research.web-research@2.0.1
aros.skills.finance.invoice-processing@3.0.0
aros.skills.dev.code-review@1.0.4
```

Namespace must be one of the 26 registered AROS domain extensions.
Name must be lowercase, hyphen-separated, fewer than 48 characters.
Version must follow semantic versioning.

---

## Skill Manifest Schema

### Required Fields

| Field | Type | Description |
|---|---|---|
| `skill_id` | string | Full AROS skill ID including version |
| `name` | string | Short name — lowercase, hyphens, max 48 chars |
| `version` | string | Semver `major.minor.patch` |
| `namespace` | string | AROS domain extension |
| `description` | string | What this skill does |
| `author` | string | AROS ID of the skill author |
| `license` | enum | `proprietary \| open \| commercial` |
| `status` | enum | `active \| beta \| deprecated` |
| `capabilities` | list[string] | What this skill can do — used for discovery matching |
| `tools_required` | list[string] | Tools this skill invokes — must be resolvable |
| `input_schema` | object | Declared input contract |
| `output_schema` | object | Declared output contract |
| `failure_modes` | list[FailureMode] | Defined failure codes and triggers |

```yaml
skill_id:     "aros.skills.content.content-writing@1.2.0"
name:         "content-writing"
version:      "1.2.0"
namespace:    "content"
description:  "Produces brand-aligned written content using web research and style rules."
author:       "cosama.aros"
license:      "commercial"
status:       "active"

capabilities:
  - web_research
  - long_form_writing
  - seo_optimization

tools_required:
  - web_search
  - file_write

input_schema:
  topic:
    type: string
    required: true
    description: "Subject matter for the content"
  tone:
    type: enum
    values: [formal, casual, technical, thought-leader]
    required: true
  audience:
    type: string
    required: true
  word_count:
    type: integer
    default: 800
    required: false

output_schema:
  content:
    type: string
    description: "The produced written content"
  word_count:
    type: integer
  sources:
    type: array
    items: string
    description: "URLs referenced during research"

failure_modes:
  - code: SKILL_INPUT_INVALID
    trigger: "Input schema validation fails"
  - code: SKILL_TOOL_UNAVAILABLE
    trigger: "Required tool not accessible at invocation time"
  - code: SKILL_TIMEOUT
    trigger: "Execution exceeds max_runtime_seconds"
  - code: SKILL_OUTPUT_MALFORMED
    trigger: "Agent output does not match declared output schema"
```

---

### Optional Fields

| Field | Type | Description |
|---|---|---|
| `skill_dependencies` | list[string] | Other AROS skill IDs this skill requires |
| `runtime.max_tokens` | integer | Token budget per invocation |
| `runtime.max_runtime_sec` | integer | Execution timeout |
| `runtime.model_min_tier` | enum | `low \| mid \| high \| any` — minimum model capability required |
| `compliance` | list[string] | e.g. `["GDPR", "HIPAA", "SOC2"]` |
| `pricing.model` | enum | `per_invocation \| subscription \| free` |
| `pricing.rate` | float | Cost per invocation if not free |
| `pricing.currency` | string | Default `USD` |
| `human_review.required` | boolean | Does output require human review? |
| `human_review.trigger` | string | Condition that triggers review e.g. `confidence < 0.7` |
| `metrics.avg_latency_ms` | integer | System-maintained performance data |
| `metrics.success_rate` | float | System-maintained reliability score |
| `metrics.invocation_count` | integer | Lifetime invocations — system-maintained |

```yaml
skill_dependencies:
  - "aros.skills.research.web-research@^2.0.0"

runtime:
  max_tokens: 4096
  max_runtime_sec: 120
  model_min_tier: "mid"

compliance:
  - GDPR

pricing:
  model: "per_invocation"
  rate: 0.05
  currency: "USD"

human_review:
  required: false
  trigger: "output_confidence < 0.7"

metrics:
  avg_latency_ms: 1200
  success_rate: 0.97
  invocation_count: 14800
```

---

## Skill Lifecycle

Every skill in AROS progresses through a defined lifecycle. Status is declared in the manifest and enforced by the registry.

| Status | Description | Marketplace Visible? | New Invocations Allowed? |
|---|---|---|---|
| **Draft** | Being authored. Not yet published. | No | No |
| **Beta** | Published, unstable, no SLA guaranteed. | Yes — beta flag shown | Yes |
| **Active** | Stable, tested, production-recommended. | Yes | Yes |
| **Deprecated** | Functional but superseded. Sunset date declared. Migration path required. | Yes — warning shown | Yes — migration recommended |
| **Retired** | No longer available. All agents using it must migrate. | No | No — blocked |

---

## Skill Discovery

When an agent needs a capability, it queries the AROS skill registry:

```json
POST /aros/skills/search

{
  "capabilities_required": ["long_form_writing", "seo_optimization"],
  "compliance_required": ["GDPR"],
  "model_tier": "mid",
  "namespace_preference": "content"
}
```

AROS returns ranked matches by capability coverage, success rate, and latency.

---

## Validation Rules

AROS enforces all of the following at skill registration:

1. `skill_id` must be globally unique and match `aros.skills.{namespace}.{name}@{version}` format
2. `namespace` must be a valid AROS domain extension
3. `name` must be lowercase, hyphen-separated, max 48 characters
4. `version` must follow semver
5. `capabilities` must contain at least one entry
6. `tools_required` entries must be resolvable tool names
7. All `skill_dependencies` must resolve to active or beta skills in AROS
8. `input_schema` and `output_schema` must be valid JSON Schema objects
9. At least one `failure_mode` must be declared
10. If any compliance tag is set, `human_review` fields should be declared
11. If `pricing.model` is `per_invocation` or `subscription`, `pricing.rate` must be > 0

---

## Complete Skill Manifest Example

```yaml
# ═══════════════════════════════════════════════════════════
# AROS SKILL MANIFEST v1.0
# ═══════════════════════════════════════════════════════════

skill_id:     "aros.skills.content.content-writing@1.2.0"
name:         "content-writing"
version:      "1.2.0"
namespace:    "content"
description:  "Produces brand-aligned written content using web research and declared style rules."
author:       "cosama.aros"
license:      "commercial"
status:       "active"

capabilities:
  - web_research
  - long_form_writing
  - seo_optimization
  - brand_voice_application

tools_required:
  - web_search
  - file_write

input_schema:
  topic:
    type: string
    required: true
  tone:
    type: enum
    values: [formal, casual, technical, thought-leader]
    required: true
  audience:
    type: string
    required: true
  word_count:
    type: integer
    default: 800
    required: false
  avoid_topics:
    type: string
    required: false

output_schema:
  content:
    type: string
  word_count:
    type: integer
  sources:
    type: array
    items: string

failure_modes:
  - code: SKILL_INPUT_INVALID
    trigger: "Input schema validation fails"
  - code: SKILL_TOOL_UNAVAILABLE
    trigger: "Required tool not accessible"
  - code: SKILL_TIMEOUT
    trigger: "Execution exceeds 120 seconds"
  - code: SKILL_OUTPUT_MALFORMED
    trigger: "Output does not match declared schema"

skill_dependencies:
  - "aros.skills.research.web-research@^2.0.0"

runtime:
  max_tokens: 4096
  max_runtime_sec: 120
  model_min_tier: "mid"

compliance:
  - GDPR

pricing:
  model: "per_invocation"
  rate: 0.05
  currency: "USD"

human_review:
  required: false
  trigger: "output_confidence < 0.7"
```

---

## The AROS Object Hierarchy

```
Tool  →  Skill  →  Agent  →  Workflow  →  Mission
(function) (expertise) (entity) (mission plan) (running instance)
```

A skill without an agent is an unloaded capability.
An agent without skills is an unspecialized worker.
Both are valid AROS objects. Both are independently registered, versioned, and discoverable.

---

*AROS Skill Specification v1.0 | CoSama Inc. | cosama.co | aros.cosama.co*

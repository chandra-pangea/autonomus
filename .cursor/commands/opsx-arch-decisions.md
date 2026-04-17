---
name: arch-decisions
description: >
  Guided architecture decision-making skill. Use this skill whenever
  the user wants to define, document, or review architecture decisions —
  including new project setup or adding a new feature to an existing codebase.
  Trigger on phrases like "architecture decisions", "arch decisions", "set up architecture",
  "define tech stack", "project architecture", "system design", "/opsx:arch-decisions",
  "/opsx-arch-decisions", or when a BRD has been finalized and the user wants to lock in
  technical decisions. Smart mode: detects existing project and asks only relevant questions.
compatibility:
  tools: [claude-code, cursor, gemini-cli]
  commands:
    claude-code: /opsx:arch-decisions
    cursor: /opsx-arch-decisions
    gemini-cli: "@arch-decisions or natural language"
---
 
# Architecture Decisions Skill
 
Context-aware: detects whether this is a **new project setup** or a **feature addition to
an existing codebase**, and asks only the relevant questions. Stores results in OpenSpec
for permanent project history.

---

## Phase 0: Detect Context

Run these checks in order:

### Step A — Check for existing BRD

```bash
ls raw-user-intents/
```

**If a BRD exists:**
- Read `raw-user-intents/<task>/brd/brd.md`
- Read `raw-user-intents/<task>/state.json` if it exists
- Check staleness: if `state.json` has `arch_based_on_brd` older than `brd_version`, warn:
  ```
  ⚠ Architecture decisions were built on BRD v2, but BRD is now v3. Regenerating from latest.
  ```
- Use the BRD task name as the project name
- Tell the user: "Found BRD for **<task>** (v<N>). Using it as context."

**If multiple BRDs exist:** list them and ask which one

**If no BRD exists:** ask for project name

### Step B — Detect existing project

Check for signs of an existing codebase:

```bash
ls app/ apps/ packages/ package.json pnpm-workspace.yaml 2>/dev/null
```

**EXISTING PROJECT** — if any of these exist:
- `app/src/` with domain folders
- `apps/backend/` or `apps/frontend/`
- `pnpm-workspace.yaml` with workspaces
- `package.json` with a name that isn't `nexus` or `canopy`

→ Set `mode = existing-project`

**NEW PROJECT** — if none of the above exist (clean directory or only `raw-user-intents/`):
→ Set `mode = new-project`

Tell the user which mode was detected:
- Existing project: "Existing project detected. I'll focus on decisions for the new feature from your BRD."
- New project: "No existing project found. I'll guide you through full project setup decisions."

---

## Phase 1: Introduction

**If mode = new-project:**
```
🏗️  Architecture Decision Wizard — New Project Setup

I'll guide you through 5 categories of decisions to set up your project.
At the end you'll review everything before it's saved.
```

**If mode = existing-project:**
```
🏗️  Architecture Decision Wizard — Feature Mode

Your project is already set up. I'll only ask about decisions
relevant to the new feature described in your BRD.
```

---

## Phase 2: Decision Questions

---

### MODE: NEW PROJECT — Full Setup (5 Categories)

Work through each category one at a time. Summarize before moving to the next.

---

#### Category 1 — System Architecture

1. What kind of system are we building?
   - Monolith / Modular Monolith / Microservices / Event-Driven / Serverless

2. How will components communicate?
   - REST / GraphQL / gRPC / Message queues / WebSockets / Mix

3. Key constraints?
   - Scale (users/day, req/sec) | Data sensitivity (PII/HIPAA/PCI/none) | Latency class | External integrations

---

#### Category 2 — Tech Stack

**Backend**
- Language? (Node.js, Python, Go, Java, Ruby, .NET, other)
- Framework? (Express, Fastify, NestJS, Django, FastAPI, etc.)

  > ✅ TEMPLATE CHECK — Node.js selected:
  > If user picks Node.js (or any Node.js framework), say:
  > "We have a production-ready Node.js monorepo template → https://github.com/sandeepp-org/nexus
  >
  > What's already built-in:
  > - Domain-driven folder structure (domain / entities / ports / schemas / service per feature)
  > - Pre-built features: auth, user, channels
  > - Packages: util-api-server, util-logger, util-jwt-tokens, util-mongodb, util-redis,
  >   util-rate-limit-core, util-error-normalizer, util-secret-manager-aws
  > - Tooling: ESLint, Prettier, Husky (lint-staged), Jest, TypeScript, Docker, Turbo, pnpm
  > - Node.js >=22, pnpm workspaces, Turbo build pipeline
  >
  > Should I use this as your base? (yes/no)"
  > If yes → set backend_template = nexus

**Frontend**
- SPA or SSR/SSG?
- Framework? (React, Next.js, Vue, etc.)

  > ✅ TEMPLATE CHECK — React selected:
  > If user picks React (or Next.js / Vite+React), say:
  > "We have a production-ready React design system template → https://github.com/sandeepp-org/canopy
  >
  > What's already built-in:
  > - @canopy/chroma: semantic color tokens, light/dark theme provider, Tailwind CSS v4 plugin
  > - @canopy/shadcn-core: base Shadcn UI components (Radix UI primitives)
  > - @canopy/shadcn-modulus: extended components with data-attribute styling (size/variant)
  > - @canopy/frontend-utils: shared utilities and hooks (useTheme, etc.)
  > - apps/demo-app: component showcase app
  > - Tooling: TypeScript, Tailwind v4, tsup, Vite, Turbo, pnpm workspaces
  >
  > Should I use this as your base? (yes/no)"
  > If yes → set frontend_template = canopy

**Database**
- Type? (Relational / Document / Key-Value / Graph / Time-series)
- Specific DB? (PostgreSQL, MySQL, MongoDB, Redis, DynamoDB, etc.)
- Caching? (Redis / in-memory / CDN / none)

**APIs & Auth**
- API style? (REST / GraphQL / tRPC)
- Auth mechanism? (JWT / OAuth2 / Session / API Keys / SSO)
- Third-party integrations? (Stripe, Twilio, SendGrid, etc.)

**Tooling**
- Package manager? (npm / yarn / pnpm)
- Linting/formatting? (ESLint + Prettier / etc.)

---

#### Category 3 — App Structure

1. Repo strategy? (Monorepo / Polyrepo / Hybrid)
2. Folder convention? (Feature-based / Layer-based / Domain-driven / Framework default)
3. Module rules? (Cross-feature imports allowed/prohibited, shared utilities location)
4. Frontend ↔ Backend coupling? (Decoupled / Full-stack in one repo / BFF)

---

#### Category 4 — Deployment

1. Cloud provider? (AWS / GCP / Azure / Hetzner / DigitalOcean / Render / self-hosted)
2. Containerization? (Docker yes/no → Kubernetes / Compose / none)
3. CI/CD? (GitHub Actions / GitLab CI / CircleCI / Jenkins)
4. Environments? (dev / staging / prod)
5. Infrastructure as Code? (Terraform / Pulumi / CDK / none)
6. CDN / Edge? (Cloudflare / CloudFront / none)

---

#### Category 5 — Non-Functional Requirements

**Performance:** API p99 target, throughput, caching strategy
**Scalability:** vertical / horizontal / auto-scaling | DB scaling
**Reliability:** uptime SLA, RTO/RPO
**Security:** auth level, encryption, compliance (GDPR/SOC2/HIPAA/PCI/none), secrets management
**Observability:** logging, metrics, tracing, alerting tools
**Maintainability:** code coverage target, docs standard, dependency update strategy

---

### MODE: EXISTING PROJECT — Feature Decisions Only

Do NOT ask about tech stack, language, framework, database type, repo strategy, CI/CD,
or any infrastructure already in place. Those decisions are locked. Ask only:

---

#### Category 1 — Feature Scope & Placement

1. Which domain does this feature belong to?
   - Does it fit an existing domain (auth / user / channels / etc.) or is it a new domain?
   - If new domain → what is it called?

2. Is this a backend change, frontend change, or both?

3. Which BRD requirements are in scope for this implementation round?
   (List the FRs from the BRD, let user confirm or trim)

---

#### Category 2 — API / Interface Changes

1. What new API endpoints are needed? (method, path, purpose)
   - e.g. `POST /api/v1/payments`, `GET /api/v1/reports/:id`

2. Are any existing endpoints being modified? (breaking or non-breaking)

3. Any new WebSocket events or SSE channels needed?

---

#### Category 3 — Data & Schema

1. Are new collections / tables / models needed?
   - If yes: name and key fields
   - MongoDB: collection name + key document shape
   - Redis: new key patterns or data structures

2. Are any existing schemas being modified?
   - Which fields added, changed, or removed?

---

#### Category 4 — Integrations & Auth

1. Any new external services or APIs being integrated?
   (Stripe, SendGrid, Twilio, third-party OAuth, webhooks, etc.)

2. Are there any auth or permission changes?
   - New roles, new JWT claims, new middleware guards?

3. New packages/libraries needed that aren't already in the project?

---

#### Category 5 — Feature NFRs

1. Any specific performance requirements for this feature?
   (e.g., this endpoint must respond < 200ms, process X items/sec)

2. Any security requirements beyond the existing baseline?
   (e.g., rate limit this endpoint, encrypt this field at rest)

3. Observability: are there specific logs, metrics, or alerts needed for this feature?

---

## Phase 3: Architecture Review

Present a summary appropriate to the mode.

**New project review:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋  ARCHITECTURE REVIEW — <Project Name>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. SYSTEM        [style | communication | constraints]
2. TECH STACK    [backend + template | frontend + template | db | auth]
3. APP STRUCTURE [repo | folders | coupling]
4. DEPLOYMENT    [cloud | docker | ci/cd | envs | iac | cdn]
5. NFR           [perf | scale | uptime | security | observability | coverage]

[A] Accept  [E] Edit a section  [R] Restart
```

**Existing project review:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋  FEATURE ARCHITECTURE — <BRD Task Name>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FEATURE SCOPE
  Domain:        [existing/new domain name]
  BRD FRs:       [FR-01, FR-02, ...]
  Layers:        [backend / frontend / both]

API CHANGES
  New endpoints: [list]
  Modified:      [list or none]

DATA CHANGES
  New models:    [list or none]
  Schema edits:  [list or none]

INTEGRATIONS
  New services:  [list or none]
  Auth changes:  [list or none]
  New packages:  [list or none]

FEATURE NFRs
  Performance:   [target or none]
  Security:      [additions or none]
  Observability: [additions or none]

[A] Accept  [E] Edit a section  [R] Restart
```

- **Edit** → ask which section, re-ask those questions, re-show updated review
- **Restart** → go back to Phase 2
- **Accept** → proceed to Phase 4

---

## Phase 4: Execute

### NEW PROJECT MODE

#### Step 1 — Clone template repos (if selected)

Run bash commands directly. Clone only — no `npm install`, no init scripts.

**If backend_template = nexus:**

Monorepo:
```bash
git clone https://github.com/sandeepp-org/nexus apps/backend
rm -rf apps/backend/.git
```

Polyrepo:
```bash
git clone https://github.com/sandeepp-org/nexus <project-slug>-backend
rm -rf <project-slug>-backend/.git
```

**If frontend_template = canopy:**

Monorepo:
```bash
git clone https://github.com/sandeepp-org/canopy apps/frontend
rm -rf apps/frontend/.git
```

Polyrepo:
```bash
git clone https://github.com/sandeepp-org/canopy <project-slug>-frontend
rm -rf <project-slug>-frontend/.git
```

> **Templates own their internals.** nexus has domain-driven structure already baked in.
> canopy has its own package/component structure. Do NOT reorganize anything inside them.
> The folder convention applies to the outer project wrapper only.

#### Step 2 — Create wrapper folder structure

Only create folders not already provided by the cloned templates.

Monorepo:
```bash
mkdir -p apps packages/shared
```

Polyrepo: no extra folders needed.

Show the final structure to the user:
```
<project-slug>/
├── apps/
│   ├── backend/   ✓ nexus (domain-driven, auth, user, channels, all utils pre-built)
│   └── frontend/  ✓ canopy (chroma, shadcn-core, shadcn-modulus, frontend-utils pre-built)
└── packages/
    └── shared/
```

#### Step 3 — Save to OpenSpec

Change name: `arch-decisions-<project-slug>`

Write directly (no `/opsx:propose`):

**`openspec/changes/arch-decisions-<project-slug>/proposal.md`**

```markdown
# Architecture Decisions — [PROJECT NAME]

## Summary
Foundational architecture decisions for [PROJECT NAME].
Decision date: [DATE]
Mode: New Project Setup

## 1. System Architecture
**Style:** [decision]
**Communication:** [decision]
**Constraints:** scale / sensitivity / latency / integrations
**Rationale:** [1-2 sentences]

## 2. Tech Stack
### Backend
- Language: Node.js (>=22) | Framework: [x]
- Template: nexus — https://github.com/sandeepp-org/nexus (if selected)
- Architecture: domain-driven (domain / entities / ports / schemas / service per feature)
- Pre-built features: auth, user, channels
- Packages: util-api-server, util-logger, util-jwt-tokens, util-mongodb, util-redis, util-rate-limit-core, util-error-normalizer, util-secret-manager-aws
- Tooling: pnpm workspaces, Turbo, ESLint, Prettier, Husky, Jest, TypeScript, Docker

### Frontend
- Framework: React | Rendering: [x]
- Template: canopy — https://github.com/sandeepp-org/canopy (if selected)
- Design system packages: @canopy/chroma, @canopy/shadcn-core, @canopy/shadcn-modulus, @canopy/frontend-utils
- Features: light/dark theme, Shadcn UI, Tailwind CSS v4
- Tooling: pnpm workspaces, Turbo, TypeScript, tsup, Vite

### Database
- Primary: [type + name] | Cache: [x]

### APIs & Auth
- Style: [x] | Auth: JWT (util-jwt-tokens) | Integrations: [list]

### Tooling
- Package manager: pnpm | Linting: ESLint + Prettier (pre-configured in templates)

## 3. App Structure
- Repo: [x] | Folders: [x] | Module rules: [x] | Coupling: [x]

## 4. Deployment
- Cloud: [x] | Docker: [x] | CI/CD: [x]
- Environments: [list] | IaC: [x] | CDN: [x]

## 5. NFR
- Performance: p99 [x]ms, [x] req/sec, cache: [x]
- Scalability: [x] | DB scaling: [x]
- Reliability: [x] SLA, RTO/RPO: [x]
- Security: [auth] | compliance: [x] | secrets: [x]
- Observability: logging [x] | metrics [x] | tracing [x] | alerts [x]
- Maintainability: [x]% coverage | docs: [x] | deps: [x]

## Open Questions
- [ ] [Any TBD items]

## Decision Log
| Decision | Alternatives | Reason |
|----------|-------------|--------|
| [key decision] | [alt A, alt B] | [rationale] |
```

**`openspec/changes/arch-decisions-<project-slug>/specs/architecture/spec.md`**

```yaml
# Architecture Spec — [PROJECT NAME]
project: [name]
version: 1.0.0
status: active
created: [DATE]
mode: new-project
change: arch-decisions-[slug]

system:
  style: [monolith|modular-monolith|microservices|event-driven|serverless]
  communication: [list]
  constraints:
    scale: [x]
    data_sensitivity: [none|pii|hipaa|pci]
    latency_class: [realtime|near-realtime|batch]

backend:
  language: node-js-22
  framework: [x]
  template: [nexus|none]
  template_repo: https://github.com/sandeepp-org/nexus
  template_includes:
    - domain-driven-structure (domain/entities/ports/schemas/service per feature)
    - pre-built-features: [auth, user, channels]
    - packages: [util-api-server, util-logger, util-jwt-tokens, util-mongodb, util-redis, util-rate-limit-core, util-error-normalizer, util-secret-manager-aws]
    - tooling: [pnpm-workspaces, turbo, eslint, prettier, husky, jest, typescript, docker]

frontend:
  framework: react
  rendering: [spa|ssr|ssg]
  template: [canopy|none]
  template_repo: https://github.com/sandeepp-org/canopy
  template_includes:
    - packages: [chroma, shadcn-core, shadcn-modulus, frontend-utils]
    - features: [light-dark-theme, semantic-color-tokens, shadcn-ui, tailwind-v4, data-attribute-styling]
    - tooling: [pnpm-workspaces, turbo, typescript, tsup, vite]

database:
  primary: {type: [x], name: [x]}
  cache: [x or none]

api:
  style: [rest|graphql|trpc]
  auth: [jwt|oauth2|session|api-keys|sso]

repo_strategy: [monorepo|polyrepo|hybrid]
folder_convention: [feature-based|layer-based|domain-driven|framework-default]

deployment:
  cloud: [x]
  docker: [true|false]
  orchestration: [kubernetes|compose|none]
  cicd: [x]
  environments: [list]
  iac: [x or none]
  cdn: [x or none]

nfr:
  performance: {api_p99_ms: [x], throughput_rps: [x]}
  scalability: {approach: [x], db_scaling: [x]}
  reliability: {uptime_sla: [x], rto: [x], rpo: [x]}
  security: {auth_level: [x], encryption_at_rest: [bool], compliance: [x], secrets: [x]}
  observability: {logging: [x], metrics: [x], tracing: [x], alerting: [x]}
  maintainability: {coverage_pct: [x], docs: [x], dep_updates: [x]}
```

#### Step 4 — Update state.json

If BRD exists, update `raw-user-intents/<task>/state.json`:
```json
{
  "task": "<task-name>",
  "brd_version": "v<N>",
  "brd_updated_at": "...",
  "linked_changes": {
    "arch": "arch-decisions-<project-slug>",
    "proposal": null
  },
  "arch_based_on_brd": "v<N>",
  "proposal_based_on_brd": null
}
```

#### Step 5 — Confirm to user

```
✅ Project setup complete!

Cloned & ready:
  ✓ nexus  → apps/backend/
      Domain-driven (domain/entities/ports/schemas/service), auth+user+channels features,
      util-api-server, util-logger, util-jwt-tokens, util-mongodb, util-redis, + more
      ESLint, Prettier, Husky, Jest, Docker — all pre-configured
  ✓ canopy → apps/frontend/
      Design system: @canopy/chroma, @canopy/shadcn-core, @canopy/shadcn-modulus, @canopy/frontend-utils
      Light/dark theme, Shadcn UI, Tailwind CSS v4 — all pre-configured

OpenSpec saved:
  openspec/changes/arch-decisions-<project-name>/
  ├── proposal.md
  └── specs/architecture/spec.md

Linked to BRD: <task-name> (v<N>)    ← only if BRD exists

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Next: Run /opsx:propose to generate implementation tasks when you're ready.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### EXISTING PROJECT MODE

#### Step 1 — No cloning

Do NOT clone any repos. Do NOT create any folders. The project already exists.

#### Step 2 — Save to OpenSpec

Change name: `arch-decisions-<brd-task-slug>-feature`

Write directly:

**`openspec/changes/arch-decisions-<brd-task-slug>-feature/proposal.md`**

```markdown
# Feature Architecture Decisions — [BRD TASK NAME]

## Summary
Architecture decisions for implementing [BRD TASK] in the existing project.
Decision date: [DATE]
Mode: Feature Addition to Existing Project
BRD: raw-user-intents/<task>/brd/brd.md (v<N>)

## Feature Scope
- **Domain:** [existing domain or new domain name]
- **BRD Requirements in scope:** [FR-01, FR-02, ...]
- **Layers affected:** [backend / frontend / both]

## API Changes
### New Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| [x] | [x] | [x] |

### Modified Endpoints
[list or none]

## Data Changes
### New Models / Collections
[list with key fields, or none]

### Schema Modifications
[list changes, or none]

## Integrations & Auth
- New external services: [list or none]
- Auth/permission changes: [list or none]
- New packages required: [list or none]

## Feature NFRs
- Performance: [specific target or none]
- Security: [additions beyond baseline or none]
- Observability: [new logs/metrics/alerts or none]

## Open Questions
- [ ] [Any TBD items]
```

**`openspec/changes/arch-decisions-<brd-task-slug>-feature/specs/feature/spec.md`**

```yaml
# Feature Spec — [BRD TASK NAME]
project: [existing project name]
version: 1.0.0
status: active
created: [DATE]
mode: feature-addition
change: arch-decisions-<brd-task-slug>-feature
brd: raw-user-intents/<task>/brd/brd.md

feature:
  domain: [existing|new]
  domain_name: [x]
  brd_requirements: [FR-01, FR-02, ...]
  layers: [backend|frontend|both]

api_changes:
  new_endpoints:
    - method: [GET|POST|PUT|DELETE|PATCH]
      path: [x]
      purpose: [x]
  modified_endpoints: [list or null]

data_changes:
  new_models: [list or null]
  schema_modifications: [list or null]

integrations:
  new_services: [list or null]
  auth_changes: [list or null]
  new_packages: [list or null]

feature_nfr:
  performance: [target or null]
  security: [additions or null]
  observability: [additions or null]
```

#### Step 3 — Update state.json

Update `raw-user-intents/<task>/state.json`:
```json
{
  "task": "<task-name>",
  "brd_version": "v<N>",
  "linked_changes": {
    "arch": "arch-decisions-<brd-task-slug>-feature",
    "proposal": null
  },
  "arch_based_on_brd": "v<N>"
}
```

#### Step 4 — Confirm to user

```
✅ Feature architecture decisions saved!

OpenSpec saved:
  openspec/changes/arch-decisions-<task>-feature/
  ├── proposal.md
  └── specs/feature/spec.md

Linked to BRD: <task-name> (v<N>)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Next: Run /opsx:propose to generate implementation tasks when you're ready.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 5: Optional Next Steps

- CI/CD decisions → "I can scaffold a GitHub Actions workflow — just ask."
- Compliance required → "I can add a security checklist for [GDPR/SOC2/HIPAA] — just ask."
- BRD needs updating → "Update the BRD first with `/opsx:brd`. Arch decisions will detect the version change next time."
- Ready to implement → "Run `/opsx:propose` to generate implementation tasks from the BRD + arch decisions."

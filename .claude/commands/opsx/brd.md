---
name: brd-generator
description: Generate, iterate, and manage Business Requirements Documents (BRDs). Use this skill whenever the user mentions BRD, business requirements, feature requirements, product specs, or dumps raw requirements that need structuring. Also triggers when a user describes a feature or product idea and wants it documented. No commands needed — fully conversational.
---

# BRD Generator

**Fully conversational** — no slash commands required. The user just talks. Claude handles everything automatically.

**Phase 1 of 3:** `BRD → Arch Decisions → Proposal`
This phase captures the **"what"** before any technical design begins.

---

## Folder Structure

```
raw-user-intents/
└── <task-name>/
    ├── brd/
    │   ├── brd.md              ← Always the latest BRD (only copy — no history files)
    │   └── metadata.json       ← Version counter and status
    ├── input-prompt/
    │   ├── interview.md        ← All conversation inputs, append-only
    │   └── <uploaded-files>    ← Raw uploads or pasted content
    └── diagrams/
        └── <name>.md           ← Mermaid diagrams
```

**Always create folders before writing any file:**
```bash
mkdir -p raw-user-intents/<task>/brd
mkdir -p raw-user-intents/<task>/input-prompt
mkdir -p raw-user-intents/<task>/diagrams
```

---

## How It Works — Fully Conversational

### When the user gives requirements or describes a feature

1. **Infer the task name** from what the user said (e.g., "I want to build a payment flow" → `payment-flow`)
   - Confirm once: "I'll save this as `payment-flow` — does that work?"

2. **Create all folders immediately**

3. **Save the raw input** to `raw-user-intents/<task>/input-prompt/interview.md`:
   ```markdown
   ## Initial Input — <YYYY-MM-DD>
   <everything the user said verbatim>
   ```

4. **Generate a full BRD immediately** — do not run a long interview first
   - Use whatever the user gave — infer reasonable defaults for missing sections
   - Mark inferred fields with `⚠ assumed` so the user can spot and correct them
   - Use the [BRD Template](#brd-template) below
   - Save to `raw-user-intents/<task>/brd/brd.md`
   - Create `raw-user-intents/<task>/brd/metadata.json`
   - Create `raw-user-intents/<task>/state.json` (if it doesn't exist):
     ```json
     {
       "task": "<task-name>",
       "brd_version": "v1",
       "brd_updated_at": "<ISO-8601>",
       "linked_changes": { "arch": null, "proposal": null },
       "arch_based_on_brd": null,
       "proposal_based_on_brd": null
     }
     ```

5. **Show the full BRD** in the conversation

6. **After showing, say exactly this:**
   ```
   ✓ BRD saved: raw-user-intents/<task>/brd/brd.md (v1)

   ⚠ Sections marked as assumed — review and correct them by just chatting.

   Tell me anything you'd like to change, add, or remove.
   If you're happy with it and stop responding, I'll treat it as accepted.
   ```

---

### When the user chats to refine (the refinement loop)

For every message the user sends after the BRD is shown:

1. **Understand the change** from natural language — no forms, no structured rounds
   - "Add SSO as a requirement" → add FR
   - "The timeline is 6 weeks not 3 months" → update timeline
   - "Remove the compliance section for now" → remove section
   - "The stakeholder is the ops team" → update stakeholders

2. **Apply the edit directly** to `brd.md` — only touch relevant sections, never rewrite the whole BRD

3. **Append to interview.md:**
   ```markdown
   ## Refinement — <YYYY-MM-DD> (vN → vN+1)
   User said: <their message>
   Changed: <what was updated>
   ```

5. **Bump version** in `metadata.json`, set `status: Active`

6. **Update state.json** — set `brd_version` to new version, update `brd_updated_at`
   - If downstream phases exist (arch or proposal linked), they are now stale
   - Show warning: "⚠ Architecture/Proposal may need regenerating from latest BRD"

7. **Show only the changed section(s)** — not the whole BRD again unless user asks

8. **Continue waiting** for more changes

---

### When the user goes silent (accepts the BRD)

**Claude cannot detect silence between sessions. Instead:**

- At the **start of every new session**, check `raw-user-intents/` for any BRD with `status: Active`
- If found → auto-finalize it before doing anything else:
  - Set `status: Finalized` in `metadata.json`
  - Greet the user:
    ```
    Welcome back! Your BRD for `<task>` (vN) was open last session — I've marked it as finalized.
    Want to keep refining it, or move to Architecture Decisions?
    (One phase per session — start a new session for arch decisions)
    ```

- If the user explicitly says "done", "looks good", "finalize it", "accept" → finalize immediately in the current session

---

### When the user uploads a file or pastes a document

1. Infer or ask for task name
2. Create folders
3. Save raw content to `raw-user-intents/<task>/input-prompt/<filename-or-date>.md`
4. Analyze silently — check which sections are present (✓) vs missing (✗):
   - Executive summary
   - Business objectives & metrics
   - Stakeholders
   - Scope (in/out)
   - Functional requirements
   - Non-functional requirements
   - Constraints & assumptions
   - Risks & open questions
   - Timeline
5. Generate full BRD immediately, mark gaps with `⚠ assumed`
6. Show BRD + brief gap summary:
   ```
   Generated from your document. Gaps filled with assumptions:
   ✓ Executive summary, Functional requirements, Scope
   ✗ Stakeholders → ⚠ assumed engineering + product
   ✗ Timeline     → ⚠ assumed Q2 delivery
   ✗ NFRs         → ⚠ added sensible defaults

   Tell me what to fix, or stay silent to accept it.
   ```
7. Enter refinement loop

---

### When the user returns to an existing BRD

If the user mentions a task that already has a BRD (detected by checking `raw-user-intents/`):

- If only one task exists → load it automatically, show current version
- If multiple tasks exist → show the list and ask which one:
  ```
  You have multiple BRDs. Which one do you want to work on?

  #   Task                  Version   Status
  1   user-onboarding       v3        Active
  2   payment-gateway       v1        Draft
  3   inventory-sync        v2        Finalized

  Just say the number or name.
  ```

Then load it, show the current BRD, and enter the refinement loop.

---

### When the user asks for a diagram

Detect intent from natural language: "show me a flow", "draw the architecture", "user journey diagram", etc.

1. If diagram type is not obvious, ask: "What kind? flow / system architecture / user journey / sequence / data model"
2. Generate as Mermaid using the right type:
   - `flowchart LR` — user flow or process flow
   - `sequenceDiagram` — API or system interaction
   - `graph TD` — system architecture
   - `journey` — user journey map
3. Save to `raw-user-intents/<task>/diagrams/<descriptive-name>.md`
4. Add reference link in BRD `## 11. Appendix`: `- [Diagram](../diagrams/<name>.md)`
5. Show diagram inline in the conversation

### When the user wants to see all their BRDs

Detect: "list my BRDs", "what tasks do I have", "show all", "what have I created", etc.

Scan `raw-user-intents/` and read each `metadata.json`. Show:
```
#   Task                  Version   Last Updated    Status
1   user-onboarding       v3        2024-01-15      Active
2   payment-gateway       v1        2024-01-10      Draft
3   inventory-sync        v2        2024-01-12      Finalized
```
If empty: "No BRDs found yet. Describe a feature and I'll create one."

---

### When the user asks about history

Detect: "show me versions", "what changed", "history", etc.

Only a change log (summaries) is kept in `metadata.json` — no past file versions are stored. Show the log:
```
Version    Date          Summary
v1         2024-01-10    Initial BRD from user input
v2         2024-01-11    Updated scope, added FR-03
v3         2024-01-12    Stakeholders corrected, risks expanded
```
Inform the user: "Only the latest BRD is stored — past versions cannot be restored."

---

## BRD Template

```markdown
# Business Requirements Document
**Task:** <task-name>
**Version:** v1
**Date:** <YYYY-MM-DD>
**Status:** Draft

---

## 1. Executive Summary
<2-3 sentences: what is being built, why, and for whom>

## 2. Business Objectives
- **Primary Goal:** <measurable outcome>
- **Secondary Goals:**
  - <goal>
- **Success Metrics:** <KPIs>

## 3. Stakeholders
| Role | Name/Team | Responsibility |
|------|-----------|----------------|
| Product Owner | | Decisions |
| Approver | | Sign-off |
| Dev Lead | | Technical feasibility |
| End Users | | Adoption |

## 4. Scope
### In Scope
- <item>

### Out of Scope
- <item>

## 5. Functional Requirements
### FR-01: <Requirement Name>
- **Description:** <what it does>
- **Priority:** Must-have / Should-have / Nice-to-have
- **Acceptance Criteria:**
  - Given <context>, when <action>, then <outcome>

## 6. Non-Functional Requirements
- **Performance:** <e.g., response time < 2s>
- **Security:** <e.g., OAuth 2.0, encryption at rest>
- **Scalability:** <e.g., 10k concurrent users>
- **Compliance:** <e.g., GDPR, SOC2>

## 7. Constraints & Assumptions
### Constraints
- <technical, budget, timeline, regulatory>

### Assumptions
- <assumed truths>

## 8. Dependencies & Integrations
- <system or team dependency>

## 9. Risks & Open Questions
| Risk/Question | Impact | Status |
|---------------|--------|--------|
| <risk> | High/Med/Low | Open/Resolved |

## 10. Timeline & Milestones
| Milestone | Target Date |
|-----------|-------------|
| BRD accepted | |
| Arch decisions complete | |
| Dev start | |
| Launch | |

## 11. Appendix
<!-- Diagrams linked here automatically -->
```

---

## metadata.json Structure

```json
{
  "task": "<task-name>",
  "currentVersion": "v1",
  "created": "YYYY-MM-DD",
  "lastUpdated": "YYYY-MM-DD",
  "status": "Draft",
  "phase": "BRD",
  "iterations": [
    { "version": "v1", "date": "YYYY-MM-DD", "summary": "Initial BRD from user input" }
  ]
}
```

**Status flow:** `Draft` → `Active` → `Finalized`
- `Draft`: just created, not yet shown to user
- `Active`: user is refining
- `Finalized`: user went silent or explicitly said they're done

---

## Guardrails

- **Generate first, ask later** — show a BRD draft before asking any questions
- **Silence = acceptance** — if the user stops responding, mark the BRD as Finalized
- **Conversational refinement** — user chats naturally, no forms or structured flows
- **`mkdir -p` always** — create all folders before writing any file
- **Single file, always overwrite** — `brd.md` is the only BRD file; edit it in-place on every iteration, no separate version files
- **interview.md is append-only** — new inputs are added at the bottom, never overwritten
- **Mark inferred content** — any field not explicitly stated by the user gets `⚠ assumed`
- **Show only what changed** — after refinements, show the updated section(s) only
- **ONE_PHASE_PER_SESSION** — never transition to Arch Decisions or Proposal in the same session
- **Auto-detect intent** — diagrams, history, task switching, listing — all triggered by natural language, no commands
- **Sequence enforcement** — after BRD is finalized, do NOT generate arch decisions or proposals in the same session. Always direct: "Start a new session for Phase 2 (Arch Decisions)"
- **Uploaded file naming** — save uploaded files as `raw-user-intents/<task>/input-prompt/YYYY-MM-DD-<original-filename>.md` to avoid collisions
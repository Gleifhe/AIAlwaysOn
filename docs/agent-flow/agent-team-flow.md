# Forge Platform — Agent Team Flow Design
## Architect → PM → Developer → Security Engineer Pipeline

**Version:** 1.0  
**Date:** May 24, 2026

---

## 1. THE PROBLEM WITH NAIVE AGENT FLOWS

Most multi-agent setups are either:
- **Sequential waterfall** — slow, no feedback loops, late-discovered issues
- **Chat room free-for-all** — agents talk past each other, no clear ownership, context bloat

The efficient design is a **directed graph with feedback edges** — work flows forward by default, but any agent can push work backward to the right upstream agent when issues are found.

---

## 2. AGENT ROLES & BOUNDARIES

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ARCHITECT        PM            DEVELOPER      SECURITY        │
│   ──────────    ──────────     ──────────────   ──────────      │
│   WHAT & WHY    WHEN & WHO    HOW              IS IT SAFE       │
│                                                                 │
│   • System      • Stories     • Code           • Threat model   │
│     design      • Priority    • Tests          • SAST/DAST      │
│   • Interfaces  • Sprint      • Migrations     • Auth review    │
│   • Trade-offs    plans       • API impl       • Data flow      │
│   • Standards   • Acceptance  • Infrastructure • Compliance     │
│   • ADRs          criteria    • Documentation  • Pen test       │
│   • Reviews     • Dependency  • Bug fixes      • Guardrail      │
│     final         tracking                       config         │
│     output                                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key principle: Each agent owns a *decision domain*

| Agent | Decides | Does NOT decide |
|---|---|---|
| **Architect** | Component boundaries, data models, protocols, technology choices, API contracts | Task ordering, implementation details, deployment schedule |
| **PM** | Priority, sequencing, sprint assignment, acceptance criteria, scope | Technology choices, security requirements, code structure |
| **Developer** | Implementation approach, library selection, code structure, test strategy | What to build, priority, security policy |
| **Security** | Threat mitigations, auth patterns, data classification, compliance controls | Feature scope, sprint timing, code style |

---

## 3. THE FLOW — FOUR MODES OF OPERATION

The agent team operates in four distinct modes depending on what's happening:

### Mode 1: PLAN (New feature / component)
### Mode 2: BUILD (Sprint execution)
### Mode 3: REVIEW (Quality gate)
### Mode 4: INCIDENT (Production issue / self-heal failure)

---

## 4. MODE 1 — PLAN FLOW

**Trigger:** New spec section, feature request, or architectural change needed.

```
                    ┌──────────────┐
                    │   TRIGGER    │
                    │ (Spec section│
                    │  or feature) │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  ARCHITECT   │
                    │              │
                    │ 1. Read spec │
                    │ 2. Design    │
                    │    components│
                    │ 3. Define    │
                    │    interfaces│
                    │ 4. Write ADR │
                    │ 5. Identify  │
                    │    security  │
                    │    surfaces  │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
              ┌─────│  SECURITY    │
              │     │              │
              │     │ 1. Threat    │
              │     │    model the │
              │     │    design    │
              │     │ 2. Identify  │
              │     │    attack    │
              │     │    vectors   │
              │     │ 3. Define    │
              │     │    security  │
              │     │    requirements│
              │     │ 4. Approve   │
              │     │    or reject │
              │     └──────┬───────┘
              │            │
    ┌─────────▼──┐  pass   │
    │ ARCHITECT  │◄────────┘ If security rejects:
    │ (Revise    │           Architect revises design
    │  design)   │           with security constraints
    └────────────┘
              │ approved
              │
       ┌──────▼───────┐
       │     PM       │
       │              │
       │ 1. Decompose │
       │    into      │
       │    stories   │
       │ 2. Set       │
       │    priority  │
       │ 3. Define    │
       │    acceptance│
       │    criteria  │
       │ 4. Sequence  │
       │    into      │
       │    sprints   │
       │ 5. Attach    │
       │    security  │
       │    reqs to   │
       │    stories   │
       └──────┬───────┘
              │
              ▼
        Ready for BUILD
```

### Plan Flow Messages

```yaml
# 1. Architect → Security: Design Review Request
ArchitectToSecurity:
  type: design_review_request
  payload:
    component: "Self-Heal Engine"
    design_doc: |
      ## Component: Self-Heal Engine
      ### Data Flow: 
        Failed execution → AI diagnosis → script generation → sandbox exec
      ### Interfaces:
        - Input: ExecutionFailure event from Event Bus
        - Output: HealAttempt record to Script Registry
        - External: LLM API calls (OpenAI/Anthropic)
      ### Data Sensitivity: 
        - Reads script source code (may contain secrets)
        - Reads execution logs (may contain PII)
        - Generates new scripts (code injection risk)
    adr: "ADR-007: Self-heal uses RAG over repair history"
    questions_for_security:
      - "Should LLM calls go through a proxy for content filtering?"
      - "How do we prevent the heal engine from generating scripts that exfiltrate data?"

# 2. Security → Architect: Threat Model Response
SecurityToArchitect:
  type: design_review_response
  verdict: approved_with_requirements
  threat_model:
    threats:
      - id: T1
        name: "Prompt injection via error messages"
        severity: HIGH
        description: "Attacker crafts error output that injects instructions into the self-heal prompt"
        mitigation: "Sanitize all error output before including in LLM prompts. Use structured extraction, not raw concatenation."
      - id: T2
        name: "Generated script data exfiltration"
        severity: CRITICAL  
        description: "Self-heal engine generates a script that sends data to external endpoint"
        mitigation: "Sandbox network policy: deny all egress except explicitly whitelisted endpoints. Script must pass static analysis before execution."
      - id: T3
        name: "Secrets in execution context"
        severity: HIGH
        description: "Self-heal reads logs/code that contains hardcoded credentials"
        mitigation: "Pre-process all context through secrets scanner. Redact matches before passing to LLM."
  security_requirements:
    - "SR-1: All LLM prompts must go through content filter proxy"
    - "SR-2: Generated scripts must pass SAST scan before sandbox execution"
    - "SR-3: Sandbox egress whitelist required, default deny-all"
    - "SR-4: Secrets scanner on all heal context inputs"
    - "SR-5: Heal attempts on production scripts require human approval"

# 3. Architect → PM: Approved Design Handoff  
ArchitectToPM:
  type: design_handoff
  component: "Self-Heal Engine"
  design_doc: <updated with security requirements>
  security_requirements: [SR-1 through SR-5]
  interfaces: <API contracts, data models>
  dependencies: ["Script Registry", "Sandbox Manager", "Event Bus", "LLM Proxy"]
  estimated_complexity: XL

# 4. PM → Developer(s): Sprint-Ready Stories
PMToDeveloper:
  type: story_batch
  sprint: 3
  stories:
    - id: FORGE-071
      title: "Implement Self-Heal diagnosis pipeline"
      acceptance_criteria: [...]
      security_requirements: [SR-1, SR-4]
      design_reference: "ADR-007, Section 3.3.3"
      priority: P0
      dependencies: [FORGE-042, FORGE-055]
```

---

## 5. MODE 2 — BUILD FLOW

**Trigger:** Sprint starts, stories assigned.

```
     ┌──────────────────────────────────────────────────────┐
     │                    SPRINT LOOP                        │
     │                                                      │
     │  ┌────────────┐     ┌─────────────────────────────┐  │
     │  │     PM     │     │       DEVELOPER             │  │
     │  │            │────▶│                             │  │
     │  │ Assign     │     │  1. Read story + design doc │  │
     │  │ story      │     │  2. Plan implementation     │  │
     │  │            │     │  3. Write code              │  │
     │  │            │     │  4. Write tests             │  │
     │  │            │     │  5. Run linter + tests      │  │
     │  │            │     │  6. Self-review checklist   │  │
     │  │            │     │  7. Create PR               │  │
     │  └─────▲──────┘     └──────────┬──────────────────┘  │
     │        │                       │                      │
     │        │                       │ PR ready             │
     │        │                       ▼                      │
     │        │            ┌─────────────────────────────┐   │
     │        │            │      SECURITY               │   │
     │        │            │                             │   │
     │        │            │  1. Run SAST scanner        │   │
     │        │            │  2. Check auth patterns     │   │
     │        │            │  3. Verify SR-* compliance  │   │
     │        │            │  4. Check data flow for     │   │
     │        │            │     PII/secrets exposure    │   │
     │        │            │  5. Approve or reject       │   │
     │        │            └──────────┬──────────────────┘   │
     │        │                       │                      │
     │        │         ┌─────────────┼─────────────┐        │
     │        │         │ PASS        │ FAIL        │        │
     │        │         ▼             ▼             │        │
     │        │  ┌────────────┐ ┌──────────────┐    │        │
     │        │  │ ARCHITECT  │ │  DEVELOPER   │    │        │
     │        │  │            │ │              │    │        │
     │        │  │ Final      │ │ Fix security │    │        │
     │        │  │ review:    │ │ findings     │────┘        │
     │        │  │ • Spec     │ │ Re-submit PR │             │
     │        │  │   compliance│ └──────────────┘            │
     │        │  │ • Interface│                              │
     │        │  │   contracts│                              │
     │        │  │ • Quality  │                              │
     │        │  │   bar      │                              │
     │        │  └─────┬──────┘                              │
     │        │        │                                     │
     │        │        │ APPROVE → Merge                     │
     │        │        │ REJECT  → Developer fixes           │
     │        │        │                                     │
     │        │        ▼                                     │
     │   ┌────┴────────────┐                                 │
     │   │ PM marks story  │                                 │
     │   │ DONE, updates   │                                 │
     │   │ progress        │                                 │
     │   └─────────────────┘                                 │
     │                                                       │
     └───────────────────────────────────────────────────────┘
```

### Build Flow — Developer Self-Review Checklist

Before the Developer submits a PR, it runs this checklist internally:

```yaml
SelfReviewChecklist:
  code_quality:
    - [ ] Follows project naming conventions
    - [ ] No TODO/FIXME without linked issue
    - [ ] Error handling covers failure cases
    - [ ] Logging includes trace_id for observability
    
  security_pre_check:
    - [ ] No hardcoded secrets or credentials
    - [ ] SQL uses parameterized queries
    - [ ] User input is validated at boundary
    - [ ] Auth decorator on all new endpoints
    
  testing:
    - [ ] Unit tests for business logic
    - [ ] Integration tests for API endpoints
    - [ ] Edge cases covered (empty input, large input, malformed input)
    - [ ] Coverage >= 80% for new code
    
  spec_compliance:
    - [ ] Implements all acceptance criteria
    - [ ] Matches interface contracts from design doc
    - [ ] Data models match schema definitions
```

### Build Flow — Parallel Review (Efficiency Optimization)

Security and Architect reviews can run **in parallel** for most changes:

```
Developer PR ──┬──▶ Security Review (automated SAST + manual policy check)
               │
               └──▶ Architect Review (spec compliance + interface check)
               
               Both must PASS before merge.
               If either FAILS, Developer fixes and re-submits.
               
               Security BLOCK overrides Architect APPROVE.
```

**Why parallel:** Cuts review cycle time in half. Security catches injection/auth issues while Architect catches design drift — independent concerns.

**Exception:** If the PR changes API contracts or data models, Architect reviews FIRST (because Security needs stable interfaces to threat-model against).

---

## 6. MODE 3 — REVIEW FLOW (Quality Gate)

**Trigger:** End of sprint, pre-release, or milestone boundary.

```
┌────────────────────────────────────────────────────────────────┐
│                    MILESTONE REVIEW                             │
│                                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │    PM    │  │DEVELOPER │  │ SECURITY │  │  ARCHITECT   │   │
│  │          │  │          │  │          │  │              │   │
│  │ Progress │  │ Demo     │  │ Security │  │ Architecture │   │
│  │ report:  │  │ working  │  │ posture  │  │ integrity    │   │
│  │ • Done   │  │ features │  │ report:  │  │ review:      │   │
│  │ • Blocked│  │ + known  │  │ • Open   │  │ • Spec drift?│   │
│  │ • Risks  │  │ issues   │  │   vulns  │  │ • Tech debt? │   │
│  │ • Scope  │  │          │  │ • Scan   │  │ • Interface  │   │
│  │   changes│  │          │  │   results│  │   breaks?    │   │
│  │          │  │          │  │ • Compliance│ │ • ADR needed?│   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────┬───────┘   │
│       │             │             │               │            │
│       └──────┬──────┴─────────────┴───────┬───────┘            │
│              │                            │                    │
│              ▼                            ▼                    │
│    ┌─────────────────┐          ┌──────────────────┐           │
│    │  GO / NO-GO     │          │  ACTION ITEMS    │           │
│    │  Decision       │          │  generated for   │           │
│    │                 │          │  next sprint     │           │
│    │  All 4 agents   │          │                  │           │
│    │  must agree     │          │  Fed back into   │           │
│    │  on GO          │          │  PLAN flow       │           │
│    └─────────────────┘          └──────────────────┘           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Veto Rules

| Agent | Can Veto Release? | Veto Condition |
|---|---|---|
| **Architect** | Yes | Spec drift > 20%, interface contract broken, missing ADR for deviation |
| **PM** | Yes | P0 stories incomplete, acceptance criteria not met |
| **Developer** | Advisory only | Can flag tech debt and known issues but cannot block |
| **Security** | **Absolute veto** | Any CRITICAL or unmitigated HIGH vulnerability blocks release |

---

## 7. MODE 4 — INCIDENT FLOW

**Trigger:** Production failure, self-heal exhaustion, security alert.

```
┌────────────────────────────────────────────────────────────────────┐
│                    INCIDENT RESPONSE                                │
│                                                                    │
│    ALERT                                                           │
│      │                                                             │
│      ▼                                                             │
│  ┌──────────────────────────────────┐                              │
│  │  TRIAGE (automated)             │                               │
│  │                                 │                               │
│  │  Is it a security incident?     │                               │
│  │  ├── YES → Security leads       │                               │
│  │  └── NO  → Is self-heal working?│                               │
│  │            ├── YES → Monitor     │                               │
│  │            └── NO  → Escalate    │                               │
│  └──────────┬──────────────────────┘                               │
│             │                                                      │
│    ┌────────┴────────────────────────────────┐                     │
│    │                                         │                     │
│    ▼ Security Incident                       ▼ Operational Incident│
│                                                                    │
│  ┌──────────────┐                    ┌──────────────┐              │
│  │  SECURITY    │ ◄── leads          │  DEVELOPER   │ ◄── leads   │
│  │              │                    │              │              │
│  │ 1. Assess    │                    │ 1. Diagnose  │              │
│  │    severity  │                    │    root cause│              │
│  │ 2. Contain   │                    │ 2. Write fix │              │
│  │    (isolate, │                    │ 3. Test fix  │              │
│  │    revoke)   │                    │ 4. Deploy    │              │
│  │ 3. Direct    │                    │    hotfix    │              │
│  │    Developer │                    │              │              │
│  │    for fix   │                    └──────┬───────┘              │
│  └──────┬───────┘                           │                      │
│         │                                   │                      │
│         ▼                                   ▼                      │
│  ┌──────────────┐                    ┌──────────────┐              │
│  │  DEVELOPER   │                    │  SECURITY    │              │
│  │              │                    │              │              │
│  │ Implement    │                    │ Quick scan   │              │
│  │ security fix │                    │ of hotfix    │              │
│  │ under        │                    │ (expedited   │              │
│  │ Security's   │                    │  review)     │              │
│  │ direction    │                    │              │              │
│  └──────┬───────┘                    └──────┬───────┘              │
│         │                                   │                      │
│         └───────────────┬───────────────────┘                      │
│                         ▼                                          │
│                  ┌──────────────┐                                   │
│                  │  PM          │                                   │
│                  │              │                                   │
│                  │ 1. Log       │                                   │
│                  │    incident  │                                   │
│                  │ 2. Update    │                                   │
│                  │    backlog   │                                   │
│                  │    with      │                                   │
│                  │    follow-up │                                   │
│                  │    stories   │                                   │
│                  └──────┬───────┘                                   │
│                         │                                          │
│                         ▼                                          │
│                  ┌──────────────┐                                   │
│                  │  ARCHITECT   │                                   │
│                  │              │                                   │
│                  │ Post-mortem: │                                   │
│                  │ • Root cause │                                   │
│                  │ • Systemic   │                                   │
│                  │   fix needed?│                                   │
│                  │ • ADR update │                                   │
│                  │ • Feed into  │                                   │
│                  │   self-heal  │                                   │
│                  │   knowledge  │                                   │
│                  │   base       │                                   │
│                  └──────────────┘                                   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Incident Response Time Targets

| Severity | Triage | Containment | Fix Deployed | Post-Mortem |
|---|---|---|---|---|
| **SEV-1 (Critical)** | 5 min | 15 min | 1 hour | 24 hours |
| **SEV-2 (High)** | 15 min | 1 hour | 4 hours | 48 hours |
| **SEV-3 (Medium)** | 1 hour | N/A | Next sprint | Next review |

---

## 8. AGENT SYSTEM PROMPTS

### 8.1 Architect Agent

```
You are the Platform Architect for Forge. You are the technical authority
on system design.

RESPONSIBILITIES:
1. Translate spec sections into component designs with clear interfaces
2. Write Architecture Decision Records (ADRs) for non-obvious choices
3. Define API contracts (OpenAPI), data models (JSON Schema), and protocols
4. Review all merged code for spec compliance and architectural integrity
5. Conduct post-mortems and feed systemic fixes into the design
6. Final approval gate before any code merges

DECISION RIGHTS:
- You DECIDE: component boundaries, data models, API contracts, technology choices
- You DO NOT DECIDE: task priority, sprint planning, implementation details, security policy

WORKING STYLE:
- Design for the 80% case. Don't over-engineer.
- Every interface you define will be implemented by the Developer agent — 
  make contracts clear, minimal, and testable.
- When Security rejects your design, treat their requirements as hard constraints
  and redesign around them. Don't argue — adapt.
- Flag tech debt explicitly. Create ADRs for intentional shortcuts.

OUTPUT FORMAT:
- Design docs: Markdown with data model schemas, sequence diagrams, API contracts
- ADRs: Title, Context, Decision, Consequences, Status
- Reviews: APPROVE / REVISE (with specific revision instructions) / BLOCK (with reason)

TOOLS: spec_reader, adr_writer, schema_validator, code_reader, pr_reviewer
```

### 8.2 PM Agent

```
You are the Project Manager for Forge. You translate designs into 
executable work and keep the team moving.

RESPONSIBILITIES:
1. Decompose Architect designs into user stories with acceptance criteria
2. Prioritize: P0 (blocks everything), P1 (this sprint), P2 (next sprint), P3 (backlog)
3. Sequence stories respecting dependencies
4. Track progress: what's done, what's blocked, what's at risk
5. Generate sprint plans and progress reports
6. Ensure every story references its spec section and security requirements

DECISION RIGHTS:
- You DECIDE: priority, sequencing, sprint assignment, scope trade-offs
- You DO NOT DECIDE: technical approach, security policy, code quality bar

WORKING STYLE:
- Stories must be atomic: one PR per story, testable, demonstrable
- Every story has: ID, title, description, acceptance criteria, 
  security requirements (from Security agent), design reference (from Architect),
  size estimate (S/M/L/XL), dependencies
- If a Developer reports a blocker, YOU decide: wait, re-sequence, or descope
- Never let the sprint have more than 1 XL story. Break them down.

OUTPUT FORMAT:
- Stories: YAML with fields defined above
- Sprint plan: ordered list with capacity allocation
- Progress: dashboard with % complete, blockers, risks

TOOLS: story_creator, dependency_graph, progress_tracker, risk_register
```

### 8.3 Developer Agent

```
You are a Senior Developer building Forge. You write production-quality 
code that passes security review on the first try.

RESPONSIBILITIES:
1. Implement stories: code, tests, migrations, API docs
2. Follow the Architect's design docs and API contracts exactly
3. Satisfy all acceptance criteria from the PM
4. Address all security requirements attached to the story
5. Run self-review checklist before submitting PR
6. Fix issues raised by Security and Architect reviews promptly

DECISION RIGHTS:
- You DECIDE: implementation approach, library choices, code structure, 
  test strategy, internal function design
- You DO NOT DECIDE: what to build (PM), API contracts (Architect), 
  security policy (Security)

WORKING STYLE:
- Read the design doc and security requirements BEFORE writing any code
- Write tests alongside code, not after
- If the design doc is ambiguous, ask the Architect for clarification 
  BEFORE implementing your interpretation
- If a security requirement seems impractical, raise it to Security 
  with a concrete alternative — don't skip it
- Run the self-review checklist. If any item fails, fix it before submitting

OUTPUT FORMAT:
- PRs with: story ID, implementation summary, test results, 
  security requirement checklist, spec compliance notes
- If blocked: structured blocker report to PM

TOOLS: code_generator, test_runner, linter, schema_migrator, 
       api_doc_generator, git_operations, dependency_resolver
```

### 8.4 Security Engineer Agent

```
You are the Security Engineer for Forge. You ensure the platform is 
secure by design, not by afterthought.

RESPONSIBILITIES:
1. Threat model every Architect design BEFORE implementation begins
2. Define security requirements (SR-*) that attach to stories
3. Review every PR for security vulnerabilities
4. Run SAST/DAST scanners and interpret results
5. Verify compliance with security requirements on each story
6. Absolute veto on releases with CRITICAL or unmitigated HIGH vulnerabilities
7. Lead incident response for security events

DECISION RIGHTS:
- You DECIDE: threat mitigations, auth patterns, data classification, 
  compliance controls, guardrail configurations
- You DO NOT DECIDE: feature scope, sprint priority, code style, 
  architecture choices (but you constrain them)

WORKING STYLE:
- Shift left: catch issues in design, not in code review
- Be specific in security requirements — "validate input" is useless,
  "validate email format with regex, max 254 chars, reject null bytes" is useful
- Your SAST findings must include: file, line, issue, severity, 
  specific fix recommendation
- Grade your reviews: PASS / PASS_WITH_ADVISORY / FAIL (with specific findings)
- Don't block on LOW severity — advise and move on
- CRITICAL and HIGH findings are non-negotiable blocks

THREAT MODEL FORMAT:
- Threat ID, Name, STRIDE category, Severity, Attack vector, Mitigation
- Security Requirements: SR-{number}, one sentence, testable

TOOLS: sast_scanner, dependency_audit, secrets_scanner, 
       auth_pattern_checker, compliance_checker, threat_model_template
```

---

## 9. MESSAGE PROTOCOL

All agent communication uses typed messages through a shared message bus:

```yaml
Message:
  id: UUID
  timestamp: ISO8601
  trace_id: UUID            # Links all messages in a work item chain
  from: enum [architect, pm, developer, security]
  to: enum [architect, pm, developer, security, all]
  type: enum [
    # Plan flow
    design_proposal,        # Architect → Security
    threat_model,           # Security → Architect
    design_approved,        # Architect → PM (after security approval)
    story_batch,            # PM → Developer
    
    # Build flow
    pr_submitted,           # Developer → Security + Architect (parallel)
    security_review,        # Security → Developer (or Architect)
    architecture_review,    # Architect → Developer
    pr_approved,            # Both reviewers → merge
    story_completed,        # Developer → PM
    
    # Feedback edges
    clarification_request,  # Any → upstream agent
    blocker_report,         # Developer → PM
    design_revision_needed, # Security → Architect
    scope_change_request,   # Any → PM
    
    # Review flow
    milestone_report,       # PM → all
    go_no_go_vote,          # Each agent → PM (aggregator)
    
    # Incident flow
    incident_alert,         # System → Security (or Developer)
    incident_update,        # Lead → all
    post_mortem,            # Architect → all
    
    # Meta
    escalate_to_human       # Any → human operators
  ]
  payload: <type-specific>
  priority: enum [critical, high, normal, low]
  requires_response: bool
  response_deadline: duration | null
```

---

## 10. EFFICIENCY OPTIMIZATIONS

### 10.1 Parallel Reviews (Already described above)
Security and Architect review PRs simultaneously. 2x faster review cycles.

### 10.2 Batched Story Creation
PM batches stories for a full sprint in one message, not one-at-a-time. Developer can plan implementation order.

### 10.3 Pre-Computed Security Requirements
Security threat-models the design ONCE during Plan mode. Those security requirements attach to stories and travel forward. The Developer sees them before writing code. The Security review then verifies compliance — it's a *check*, not a *discovery*. This eliminates the "security finds surprise issues in code review" antipattern.

```
PLAN phase:  Security → "SR-3: Sandbox egress whitelist required"
                         ↓ (attached to story)
BUILD phase: Developer reads SR-3, implements egress whitelist
                         ↓ (in PR)
REVIEW phase: Security checks: "Is SR-3 implemented?" → YES → PASS
```

### 10.4 Fast-Path for Low-Risk Changes
Not every PR needs full review:

```
Risk Assessment (automated):
  LOW:  Docs, tests, comments, config  → Auto-approve (no review needed)
  MEDIUM: Internal logic, non-API code → Developer self-review + 1 reviewer
  HIGH: API changes, auth, data models → Full Security + Architect review
  CRITICAL: Crypto, secrets, sandbox   → Full review + human oversight
```

### 10.5 Feedback Edge Limits
To prevent infinite revision loops:

```
MaxRevisions:
  design_revision: 3    # After 3 Architect↔Security loops → escalate to human
  pr_revision: 3        # After 3 Developer↔Reviewer loops → escalate to human  
  self_heal_retry: 3    # Already in spec
  
  # On escalation, package full context:
  escalation_package:
    - Original request
    - All revision attempts with feedback
    - Current blocker summary
    - Suggested resolution (agent's best guess)
```

### 10.6 Context Windows — What Each Agent Needs

```
Architect:
  Always loaded: Spec sections, ADR log, API contracts, component diagram
  Per-task: PR diff, relevant design doc
  Never needs: Full codebase, test results, sprint backlog

PM:
  Always loaded: Story backlog, dependency graph, sprint plan, progress metrics
  Per-task: Design handoff, blocker reports
  Never needs: Code, security scan results, architecture details

Developer:
  Always loaded: Project conventions, style guide, current story + design doc
  Per-task: Security requirements, existing code in target files
  Never needs: Other stories, sprint plan, threat models

Security:
  Always loaded: Threat model catalog, security requirements registry, OWASP top 10
  Per-task: Design doc (in Plan), PR diff (in Build)
  Never needs: Sprint backlog, progress metrics, implementation details beyond the PR
```

This keeps each agent's context window lean — only what it needs for its current decision.

---

## 11. COMPLETE FLOW DIAGRAM

```
                    ┌─────────────────────┐
                    │     SPEC / INPUT    │
                    └──────────┬──────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │            MODE 1: PLAN                    │
         │                                            │
         │  ARCHITECT ──▶ SECURITY ──▶ ARCHITECT     │
         │      │              │           │          │
         │      │         (threat model)   │          │
         │      │              │      (revise if      │
         │      │              │       rejected)      │
         │      │              │           │          │
         │      │         approved         │          │
         │      ▼              ▼           │          │
         │           PM (stories)          │          │
         │               │                │          │
         └───────────────┼────────────────┘          │
                         │                            │
         ┌───────────────┼────────────────┐          │
         │       MODE 2: BUILD            │          │
         │               │                │          │
         │          DEVELOPER             │          │
         │               │                │          │
         │          PR ready              │          │
         │               │                │          │
         │     ┌─────────┴─────────┐      │          │
         │     │                   │      │          │
         │  SECURITY          ARCHITECT   │ (parallel│
         │     │                   │      │  review) │
         │     └─────────┬─────────┘      │          │
         │               │                │          │
         │          Both PASS?            │          │
         │          ├── YES → Merge       │          │
         │          └── NO  → Fix loop    │          │
         │                                │          │
         └────────────────────────────────┘          │
                         │                            │
         ┌───────────────┼────────────────┐          │
         │      MODE 3: REVIEW            │          │
         │                                │          │
         │   PM + DEV + SEC + ARCH        │          │
         │   All vote GO / NO-GO          │          │
         │   Security has absolute veto   │          │
         │                                │          │
         └───────────────┼────────────────┘          │
                         │                            │
                    ┌────▼──────┐                     │
                    │  RELEASE  │                     │
                    └────┬──────┘                     │
                         │                            │
         ┌───────────────┼────────────────┐          │
         │      MODE 4: INCIDENT          │          │
         │  (triggered by production      │          │
         │   alerts or self-heal failure) │          │
         │                                │          │
         │  Security or Developer leads   │          │
         │  → Fix → PM logs → Architect   │──────────┘
         │    does post-mortem → feeds     │  (back to
         │    back into PLAN mode          │   PLAN)
         │                                │
         └────────────────────────────────┘
```

---

## 12. KEY PRINCIPLE: SECURITY SHIFTS LEFT, ARCHITECT BOOKENDS

The most efficient pattern is:

1. **Security goes EARLY** (threat model the design, not the code)
2. **Security goes LATE** (verify the code matches the threat model)
3. **Architect goes FIRST** (set the design) and **LAST** (verify the output)
4. **PM stays in the MIDDLE** (translate design → stories, track stories → done)
5. **Developer stays FOCUSED** (code with clear inputs, no ambiguity)

```
Time →

ARCHITECT ████░░░░░░░░░░░░░░░░░░░░░░░░████  (design first, review last)
SECURITY  ░░██░░░░░░░░░░░░░░░░░░░░░░██░░░░  (threat model early, scan late)
PM        ░░░░████░░░░░░░░░░░░░░████░░░░░░  (stories after design, tracking during build)
DEVELOPER ░░░░░░░░████████████████░░░░░░░░  (heads-down implementation)

          PLAN ─────── BUILD ────── REVIEW
```

This eliminates the two most expensive failure modes:
- **"Security surprise"** — finding auth holes after 2 weeks of coding
- **"Architecture drift"** — building something that doesn't match the design

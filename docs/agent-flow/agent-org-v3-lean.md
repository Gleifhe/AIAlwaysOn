# Forge Platform — Agent Organization v3.0 (Lean Model)
## 8-Agent Team: Maximum Output, Minimum Overhead

**Version:** 3.0  
**Date:** May 24, 2026  
**Supersedes:** v2.0 (12-agent model retained as Phase 3+ scale-up target)

---

## 1. WHY 8, NOT 12

The 12-agent model had 3-5 idle agents at any given time, 5 review checkpoints per PR, and 3-way Architect coordination overhead. The lean model cuts to what's productive:

| Cut | Why |
|---|---|
| 3 Architects → 2 | Performance + Standards merged into System Architect. Two is enough until 50+ developer scale. |
| Quality agent → eliminated | Quality becomes a function (automated checklists), not an agent. Every reviewer runs the checklist as part of their review. CI/CD computes metrics. |
| Scrum Master → Chronicler | Simplified to 3 jobs: decision log, daily digest, blocker alerts. No ceremony. |
| 4 Developers → 2 | Backend + Frontend combined. Platform + AI/ML combined. Split when codebase justifies it (Phase 3+). |
| 5 review steps → 3 | Developer self-review → parallel Security + Architect → merge. |

```
12-agent model: 132 possible channels, 5 review steps, 3-5 idle agents
 8-agent model:  ~12 active channels, 3 review steps, 1-2 idle agents
```

---

## 2. THE EIGHT AGENTS

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  LAYER 1: CROSS-CUTTING                                              │
│                                                                      │
│  ┌─────────────────────────┐  ┌─────────────────────────────────┐    │
│  │  1. CHRONICLER          │  │  2. QUESTIONER                  │    │
│  │     (was Scrum Master)  │  │     (domain-informed,           │    │
│  │                         │  │      selective engagement)      │    │
│  │  • Decision log         │  │                                 │    │
│  │  • Daily digest         │  │  • 6 knowledge profiles         │    │
│  │  • Blocker alerts       │  │  • Design docs: always          │    │
│  │                         │  │  • Large PRs (>500 LOC): always │    │
│  │  Always on.             │  │  • Small routine PRs: never     │    │
│  │  Never blocks.          │  │  • Advisory only. Never blocks. │    │
│  └─────────────────────────┘  └─────────────────────────────────┘    │
│                                                                      │
│  LAYER 2: ARCHITECTURE                                               │
│                                                                      │
│  ┌─────────────────────────┐  ┌─────────────────────────────────┐    │
│  │  3. SYSTEM ARCHITECT    │  │  4. AI & EFFICIENCY ARCHITECT   │    │
│  │                         │  │                                 │    │
│  │  Merged:                │  │  • Model selection & fallbacks  │    │
│  │  • Performance & scale  │  │  • Token economics & cost       │    │
│  │  • Standards & API      │  │  • Prompt engineering patterns  │    │
│  │    policy               │  │  • RAG / retrieval architecture │    │
│  │                         │  │  • Inference optimization       │    │
│  │  • API contracts        │  │  • Guardrail chain design       │    │
│  │  • Database performance │  │  • Agent loop efficiency        │    │
│  │  • Scaling patterns     │  │  • Eval & benchmarking          │    │
│  │  • Naming & conventions │  │                                 │    │
│  │  • Latency budgets      │  │  Reviews: AI/ML PRs, prompts,   │    │
│  │  • Resource sizing      │  │  self-heal, agent orchestration │    │
│  │  • Code organization    │  │                                 │    │
│  │  • Compliance patterns  │  │  Tiebreaker between the two:    │    │
│  │                         │  │  System Architect decides.       │    │
│  │  Reviews: Backend,      │  │                                 │    │
│  │  Frontend, Platform,    │  │                                 │    │
│  │  all API contract PRs   │  │                                 │    │
│  └─────────────────────────┘  └─────────────────────────────────┘    │
│                                                                      │
│  LAYER 3: EXECUTION                                                  │
│                                                                      │
│  ┌──────────────┐  ┌────────────────────────────┐  ┌──────────────┐  │
│  │ 5. PM        │  │  DEVELOPERS                │  │ 8. SECURITY  │  │
│  │              │  │                            │  │    ENGINEER  │  │
│  │ • Stories    │  │  ┌────────────────────┐    │  │              │  │
│  │ • Priority   │  │  │ 6. APP DEVELOPER  │    │  │ • Threat     │  │
│  │ • Sprints    │  │  │                    │    │  │   models     │  │
│  │ • Tracking   │  │  │ Backend APIs +     │    │  │ • SAST scans │  │
│  │ • Scope      │  │  │ Frontend UI        │    │  │ • Auth review│  │
│  │              │  │  │ (splits Phase 3+)  │    │  │ • SR-* reqs  │  │
│  │              │  │  └────────────────────┘    │  │ • Compliance │  │
│  │              │  │  ┌────────────────────┐    │  │              │  │
│  │              │  │  │ 7. PLATFORM &      │    │  │ ABSOLUTE     │  │
│  │              │  │  │    AI/ML DEVELOPER │    │  │ VETO on      │  │
│  │              │  │  │                    │    │  │ CRITICAL/    │  │
│  │              │  │  │ Sandbox, containers│    │  │ HIGH vulns   │  │
│  │              │  │  │ Helm, CI/CD +      │    │  │              │  │
│  │              │  │  │ Agents, self-heal, │    │  │              │  │
│  │              │  │  │ LLM integration,   │    │  │              │  │
│  │              │  │  │ guardrails         │    │  │              │  │
│  │              │  │  │ (splits Phase 3+)  │    │  │              │  │
│  │              │  │  └────────────────────┘    │  │              │  │
│  └──────────────┘  └────────────────────────────┘  └──────────────┘  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. AGENT DECISION DOMAINS

| Agent | Decides | Does NOT Decide |
|---|---|---|
| **Chronicler** | Nothing — observes and reports | Priority, design, implementation, security |
| **Questioner** | Which questions to ask (within loaded domain) | Answers, solutions, whether to block |
| **System Architect** | Component boundaries, API contracts, data models, scaling patterns, naming conventions | Model selection, prompt design, task priority |
| **AI & Efficiency Architect** | Model selection, token budgets, prompt patterns, RAG design, agent loop design | API standards, database schema, sprint planning |
| **PM** | Priority, sequencing, sprint assignment, acceptance criteria, scope trade-offs | Technology choices, security policy, code structure |
| **App Developer** | Implementation approach for APIs + UI, library selection, test strategy | What to build, priority, security policy |
| **Platform & AI/ML Developer** | Sandbox design, container config, agent implementation, self-heal logic | API standards, feature priority, business rules |
| **Security Engineer** | Threat mitigations, auth patterns, data classification, compliance controls, guardrail policy | Feature scope, sprint timing, code style |

---

## 4. THE THREE-STEP PR FLOW

Every PR goes through exactly 3 steps. No more.

```
STEP 1: DEVELOPER SELF-REVIEW
  Developer runs the quality checklist BEFORE submitting:
  
  ☐ All acceptance criteria have corresponding tests
  ☐ Test coverage >= 80% for new code
  ☐ Linter passes with zero warnings
  ☐ No hardcoded secrets or credentials
  ☐ SQL uses parameterized queries
  ☐ User input validated at boundary
  ☐ Auth decorator on all new endpoints
  ☐ API changes have updated OpenAPI spec
  ☐ Database changes have migration scripts
  ☐ No TODOs without linked issues
  ☐ Logging includes trace_id
  ☐ PR references story ID
  
  If ANY item fails → fix before submitting.
  This eliminates the need for a separate Quality agent.

STEP 2: PARALLEL REVIEW (all run simultaneously)
  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
  │ SECURITY         │  │ ARCHITECT        │  │ QUESTIONER       │
  │ ENGINEER         │  │ (routed to       │  │ (only if         │
  │                  │  │ relevant one)    │  │ triggered)       │
  │ • Automated SAST │  │                  │  │                  │
  │ • SR-* compliance│  │ App Dev PR →     │  │ Triggers:        │
  │ • Auth patterns  │  │   System Arch    │  │ • PR > 500 lines │
  │ • Data flow check│  │                  │  │ • API contract Δ │
  │                  │  │ Platform/AI PR → │  │ • New LLM call   │
  │                  │  │   AI Efficiency  │  │ • New ADR        │
  │                  │  │                  │  │                  │
  │                  │  │ API contract Δ → │  │ Otherwise: skip  │
  │                  │  │   BOTH Architects│  │                  │
  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
           │                     │                      │
           │     PASS/FAIL       │     PASS/FAIL        │ (advisory)
           └─────────┬───────────┘                      │
                     │                                   │
                     ▼                                   │
              Both PASS? ◄──────────────────────────────┘
              YES → STEP 3                     questions logged
              NO  → Developer fixes            by Chronicler
                    (max 3 rounds → human)

STEP 3: MERGE + RECORD
  PR merged.
  PM marks story DONE.
  Chronicler logs: story ID, PR, reviewers, decisions, time-to-merge.
  Quality metrics auto-computed by CI/CD pipeline.
```

---

## 5. MODE 1 — PLAN FLOW

```
  Input (spec section / feature request)
         │
         ▼
  ┌──────────────────────────────────────────────────────┐
  │  ARCHITECTS (work together, publish ONE design doc)  │
  │                                                      │
  │  System Architect:       AI & Efficiency Architect:  │
  │  • Component boundaries  • Model selection           │
  │  • API contracts         • Token budgets             │
  │  • Data models           • Prompt patterns           │
  │  • Scaling targets       • Agent loop design         │
  │  • Latency budgets       • Cost estimates            │
  │                                                      │
  │  Output: unified design doc + ADR(s)                 │
  └───────────────────┬──────────────────────────────────┘
                      │
         ┌────────────┼────────────┐  (parallel)
         ▼                         ▼
  ┌──────────────┐          ┌──────────────┐
  │  QUESTIONER  │          │  SECURITY    │
  │              │          │  ENGINEER    │
  │  Loads       │          │              │
  │  relevant    │          │  Threat      │
  │  domain      │          │  model the   │
  │  profile(s)  │          │  design      │
  │              │          │              │
  │  Challenges  │          │  Define      │
  │  design      │          │  SR-*        │
  │  decisions   │          │  requirements│
  │              │          │              │
  │  (advisory)  │          │  (can reject │
  │              │          │   → Architects│
  │              │          │   revise)    │
  └──────┬───────┘          └──────┬───────┘
         │                         │
         │  questions logged       │  SR-* requirements
         │  by Chronicler          │  attached to design
         │                         │
         └────────────┬────────────┘
                      ▼
               ┌──────────────┐
               │      PM      │
               │              │
               │ Decomposes   │
               │ into stories │
               │ with:        │
               │ • AC         │
               │ • SR-* reqs  │
               │ • Design ref │
               │ • Size       │
               │ • Priority   │
               │ • Assigned   │
               │   Developer  │
               └──────┬───────┘
                      │
                      ▼
               Ready for BUILD
               
  Chronicler: logs design decisions, questions, security 
              requirements, story assignments
```

---

## 6. MODE 2 — BUILD FLOW

```
  PM assigns stories to the appropriate Developer:
  
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │  App Developer              Platform & AI/ML Dev     │
  │  ┌──────────────────┐       ┌──────────────────┐    │
  │  │ Backend API      │       │ Sandbox manager   │    │
  │  │ Database layer   │       │ Container runtime │    │
  │  │ Adapter impls    │       │ Helm charts       │    │
  │  │ Portal UI        │       │ CI/CD pipeline    │    │
  │  │ Visual debugger  │       │ Agent orchestrator│    │
  │  │ Dashboard        │       │ Self-heal engine  │    │
  │  └──────────────────┘       │ LLM integration   │    │
  │                              │ Guardrails        │    │
  │                              │ Knowledge base    │    │
  │                              └──────────────────┘    │
  │                                                      │
  │  Work in parallel on independent stories             │
  └──────────────────────┬───────────────────────────────┘
                         │
                    PRs submitted
                         │
                    3-step review
                    (see Section 4)
                         │
                    Merge → PM marks DONE
                    
  Chronicler: logs completions, blockers, velocity
```

---

## 7. MODE 3 — REVIEW FLOW

```
  Chronicler generates sprint summary automatically:
  
  ┌──────────────────────────────────────────────┐
  │  SPRINT SUMMARY (auto-generated)             │
  │                                              │
  │  Velocity:        15/20 points (75%)         │
  │  Stories done:    12                          │
  │  Carried over:    3                           │
  │  First-pass rate: 80% (PRs approved w/o fix) │
  │  Test coverage:   84%                        │
  │  Security:        0 CRITICAL, 1 HIGH (fixed) │
  │  LLM token spend: $247 (budget: $300)        │
  │  Decisions made:  7 (all logged)             │
  │  Open questions:  2 (from Questioner)        │
  │  Blockers:        0 active                   │
  └──────────────────────┬───────────────────────┘
                         │
                    GO / NO-GO
                         │
  Veto power:
  ┌──────────────────────────────────────────────┐
  │ Security Engineer: ABSOLUTE veto             │
  │   (CRITICAL or unmitigated HIGH)             │
  │                                              │
  │ System Architect: veto if spec drift > 20%   │
  │   or API contract broken                     │
  │                                              │
  │ PM: veto if P0 stories incomplete            │
  │                                              │
  │ All others: advisory (flag concerns,         │
  │   no veto power)                             │
  └──────────────────────────────────────────────┘
```

---

## 8. MODE 4 — INCIDENT FLOW

```
  Alert
    │
    ▼
  Auto-triage
    │
    ├── Security incident? → Security Engineer leads
    │                         directs relevant Developer
    │                         to implement fix
    │
    └── Operational issue? → Relevant Developer leads
                              Security quick-scans fix
    │
    ▼
  PM logs incident + creates follow-up stories
    │
    ▼
  Relevant Architect writes post-mortem
  (System Arch for infra, AI Arch for agent/heal issues)
    │
    ▼
  Post-mortem feeds back into PLAN mode
    │
  Chronicler: logs incident timeline, decisions, action items
```

---

## 9. AGENT ACTIVATION BY MODE

```
                     PLAN    BUILD   REVIEW  INCIDENT
                     ─────   ─────   ──────  ────────
1. Chronicler         ●       ●       ●       ●
2. Questioner         ●       ◐       ●       ○
3. System Architect   ●       ◐       ●       ◐
4. AI & Eff Architect ●       ◐       ●       ◐
5. PM                 ●       ●       ●       ◐
6. App Developer      ○       ●       ○       ●
7. Platform/AI Dev    ○       ●       ○       ●
8. Security Engineer  ●       ●       ●       ●

● = active    ◐ = on-demand    ○ = idle

Typical active agents: 5-6 at any time
Max active agents: 8 (rare — only during REVIEW)
```

---

## 10. COMMUNICATION MAP

```
Hub-and-spoke topology. 12 channels, not 56.

  Chronicler ◄──── reads all messages (passive)
       │
       │ digests/alerts
       ▼
  ┌─────────────────────────────────────────────┐
  │                                             │
  │  Architects ◄──► Architects                 │  1 channel
  │       │                                     │
  │       │ design docs                         │
  │       ▼                                     │
  │  Security ◄──── threat models designs       │  2 channels (1 per Arch)
  │       │                                     │
  │       │ SR-* requirements                   │
  │       ▼                                     │
  │  PM ◄──── receives approved designs         │  1 channel
  │       │                                     │
  │       │ assigns stories                     │
  │       ├──► App Developer                    │  2 channels (1 per Dev)
  │       └──► Platform/AI Dev                  │
  │                                             │
  │  Developers ──► PRs ──► Security            │  2 channels (1 per Dev)
  │  Developers ──► PRs ──► Relevant Architect  │  2 channels
  │                                             │
  │  Questioner ──► questions to target agent   │  2 channels (design or PR)
  │                                             │
  └─────────────────────────────────────────────┘
  
  Total: 12 directed channels
  
  Rules:
  • Developers NEVER talk to each other (independent work)
  • Questioner talks ONLY to the owner of what it's reviewing
  • Chronicler NEVER initiates conversation (except blocker alerts)
  • Security NEVER initiates work (reviews what's sent to it)
  • PM is the ONLY agent that assigns work to Developers
```

---

## 11. DETAILED ROLE SPECIFICATIONS

### 11.1 CHRONICLER

```
PURPOSE: Institutional memory. Single source of truth for what happened.

THREE JOBS:
  1. Decision Log (append-only)
     - Every design decision, who made it, why, when
     - Every ADR
     - Every security requirement
     - Every scope change
     
  2. Daily Digest
     Completed: [stories with owner]
     In Progress: [stories with owner and % done]
     Blocked: [story, blocker, how long, who needs to act]
     Decisions: [list with rationale]
     Metrics: [coverage, first-pass rate, token spend]
     
  3. Blocker Alerts
     If a blocker is unresolved for > 24 hours → alert PM
     If velocity suggests sprint will miss target → alert PM
     If a question from Questioner is unanswered for > 48h → alert target agent

DOES NOT: Make decisions, assign tasks, block work, facilitate meetings,
          coach process, suggest improvements.

TOOLS: message_bus_observer, decision_log_db, digest_generator, alert_engine
```

### 11.2 QUESTIONER

```
PURPOSE: Domain-informed devil's advocate.

ENGAGEMENT RULES (selective — not every deliverable):
  ALWAYS engage:
    • Every design doc from Architects
    • Every ADR
    • PRs > 500 lines of code
    • PRs that add new LLM calls
    • PRs that change API contracts
    • Sprint plans (completeness check)
    
  NEVER engage:
    • PRs < 200 lines implementing well-defined stories
    • Documentation-only PRs
    • Dependency updates
    • Config changes

6 KNOWLEDGE PROFILES (load before engaging):
    • AI & LLM Systems
    • Distributed Systems & Scaling
    • API Design & Standards
    • Security & Threat Modeling
    • Frontend & UX
    • DevOps & Infrastructure

QUESTION FORMAT (mandatory):
  context:              What I'm reviewing, which profile I loaded
  observation:          Specific technical detail I noticed
  concern:              Why this is potentially a problem (with reasoning)
  impact:               What could go wrong and how bad
  suggestion_direction: What area the answer should address (NOT a solution)

SELF-CHECK: "Could a non-expert ask this?" If yes → discard.

Max 15 questions per sprint (forces prioritization on what matters).
Advisory only. Never blocks.

TOOLS: spec_reader, cost_calculator, token_counter, query_plan_reader,
       openapi_validator, knowledge_profile_loader, question_log
```

### 11.3 SYSTEM ARCHITECT

```
PURPOSE: Technical authority on system design, APIs, performance, and standards.

MERGED DOMAINS:
  From Performance & Scale:
    • Database query performance, indexing, connection pooling
    • Horizontal scaling patterns, stateless design
    • Caching strategy (what, TTL, invalidation)
    • Latency budgets (p50, p95, p99 per endpoint)
    • Resource sizing, capacity planning
    • Load testing strategy
    
  From Standards & API Policy:
    • REST API conventions (naming, versioning, pagination, errors)
    • Data model standards (naming, types, nullability)
    • Code organization patterns
    • Documentation requirements
    • OpenAPI spec validation
    • Governance and compliance patterns

REVIEWS:
  • App Developer PRs (API + UI)
  • Database schema changes
  • New services or endpoints
  • API contract changes (jointly with AI Architect)

DESIGN OUTPUTS:
  • Component design docs with API contracts
  • ADRs for technology and design decisions
  • Latency budgets and scaling targets
  • Data model schemas (JSON Schema)

TOOLS: openapi_validator, query_plan_analyzer, naming_convention_checker,
       schema_validator, load_test_runner, capacity_planner
```

### 11.4 AI & EFFICIENCY ARCHITECT

```
PURPOSE: Ensures every AI/LLM interaction is cost-effective, performant, 
         and architecturally sound.

DOMAIN:
  • Model selection (which model for which task, with cost analysis)
  • Token economics (prompt size, response size, cost per call)
  • Prompt engineering patterns
  • Inference optimization (caching, batching, streaming)
  • RAG architecture (chunk size, embeddings, retrieval)
  • Agent loop efficiency (minimize LLM round-trips)
  • Guardrail chain design (which filters, what order, at what cost)
  • Eval and benchmarking strategy

DECISION FRAMEWORK (for every LLM call in the system):
  1. NECESSITY:  Does this need an LLM or can a rule handle it?
  2. MODEL FIT:  Cheapest model that meets quality bar?
  3. PROMPT SIZE: Can the prompt be shorter?
  4. CACHING:    Can we cache this response?
  5. BATCHING:   Can we batch requests?
  6. FALLBACK:   What if the model is unavailable?
  7. COST:       Per-execution and monthly projected cost?

REVIEWS:
  • Platform & AI/ML Developer PRs
  • All prompt templates before production
  • Self-heal pipeline design
  • Agent system prompts
  • Model selection decisions

TOOLS: token_counter, cost_calculator, model_benchmark_db,
       prompt_optimizer, cache_hit_analyzer, eval_runner
```

### 11.5 PM

```
PURPOSE: Translates designs into executable work, keeps the team moving.

RESPONSIBILITIES:
  • Decompose Architect designs into stories with acceptance criteria
  • Prioritize: P0 (blocks everything), P1 (this sprint), P2 (next), P3 (backlog)
  • Assign stories to the correct Developer
  • Track progress, flag risks
  • Make scope trade-off decisions when capacity is tight

STORY FORMAT:
  id: FORGE-NNN
  title: imperative sentence
  description: what and why (not how)
  acceptance_criteria: testable bullet list
  security_requirements: SR-* list (from Security Engineer)
  design_reference: ADR or design doc section
  size: S | M | L | XL
  priority: P0 | P1 | P2 | P3
  assigned_to: App Developer | Platform & AI/ML Developer
  dependencies: [story IDs]

RULES:
  • Max 1 XL story per sprint
  • Every story references its design doc section
  • Every story carries its security requirements forward
  • If a Developer reports a blocker, PM decides: wait, re-sequence, or descope

TOOLS: story_creator, dependency_graph, progress_tracker, 
       velocity_calculator, risk_register
```

### 11.6 APP DEVELOPER

```
PURPOSE: Builds the application layer — APIs, database, UI, adapters.

SCOPE (Phases 1-2, combined):
  Backend:
    • REST API endpoints
    • Database schema, migrations, queries
    • Event bus producers/consumers
    • Adapter implementations
    
  Frontend:
    • Portal UI (React + TypeScript)
    • Visual workflow debugger
    • Script editor
    • Dashboards

SPLITS INTO Backend Developer + Frontend Developer at Phase 3+
when the codebase is large enough to justify separate context windows.

SELF-REVIEW CHECKLIST (run before every PR):
  ☐ All acceptance criteria have tests
  ☐ Coverage >= 80% for new code
  ☐ Linter: zero warnings
  ☐ No hardcoded secrets
  ☐ Parameterized queries only
  ☐ Input validation at boundaries
  ☐ Auth on all endpoints
  ☐ OpenAPI spec updated (if API change)
  ☐ Migration script included (if schema change)
  ☐ trace_id in all log statements
  ☐ PR references story ID

TOOLS: code_generator, test_runner, linter, schema_migrator,
       api_doc_generator, git_operations, component_library
```

### 11.7 PLATFORM & AI/ML DEVELOPER

```
PURPOSE: Builds the platform runtime and AI intelligence layer.

SCOPE (Phases 1-2, combined):
  Platform:
    • Sandbox Manager (container/MicroVM provisioning)
    • Warm pool management
    • Helm charts and Kubernetes manifests
    • CI/CD pipeline configuration
    • Observability plumbing (OpenTelemetry instrumentation)
    • Event Store and checkpointing
    
  AI/ML:
    • Agent Orchestrator (graph-based workflow engine)
    • Self-Heal Engine (diagnosis, proposal, repair loop)
    • LLM integration layer (multi-model support)
    • Guardrail implementation
    • Knowledge base (pgvector RAG pipeline)
    • Agent memory management

SPLITS INTO Platform Developer + AI/ML Developer at Phase 3+.

SAME SELF-REVIEW CHECKLIST as App Developer, plus:
  ☐ Resource limits set on all containers
  ☐ Network policies defined (default deny)
  ☐ Prompt templates reviewed for injection risk
  ☐ Token budget defined for new LLM calls
  ☐ Sandbox teardown confirmed (no resource leaks)

TOOLS: code_generator, test_runner, docker_builder, helm_linter,
       prompt_tester, token_counter, sandbox_profiler, git_operations
```

### 11.8 SECURITY ENGINEER

```
PURPOSE: Ensures the platform is secure by design, not by afterthought.

RESPONSIBILITIES:
  In PLAN: Threat model every design, define SR-* requirements
  In BUILD: Review every PR for security, run SAST
  In REVIEW: Report security posture, veto if needed
  In INCIDENT: Lead security incidents, direct Developer fixes

THREAT MODEL FORMAT:
  Threat ID, STRIDE category, Severity, Attack vector, Mitigation

SECURITY REQUIREMENTS FORMAT:
  SR-{number}: one sentence, specific, testable
  
  ✗ BAD:  "SR-1: Validate input"
  ✓ GOOD: "SR-1: Validate script source_code field: max 1MB, 
           UTF-8 only, reject null bytes, scan for embedded credentials"

REVIEW VERDICTS:
  PASS:               No security issues found
  PASS_WITH_ADVISORY: LOW severity findings, log and move on
  FAIL:               HIGH or CRITICAL findings, must fix before merge

ABSOLUTE VETO on releases with CRITICAL or unmitigated HIGH vulnerabilities.
Security FAIL on a PR is a hard block — Developer must fix and resubmit.

TOOLS: sast_scanner, dependency_audit, secrets_scanner,
       auth_pattern_checker, compliance_checker, threat_model_template
```

---

## 12. QUALITY AS A FUNCTION, NOT AN AGENT

Quality is enforced through automation and embedded checklists, not a separate reviewer:

```
WHERE QUALITY LIVES:

  Developer self-review checklist → catches basics before PR
  CI/CD pipeline → runs tests, measures coverage, runs linter
  Security scanner → SAST on every PR (automated)
  Architect review → checks spec compliance + naming conventions
  
  Metrics auto-computed:
  ┌────────────────────────────────────────────────────┐
  │ CI/CD pipeline reports on every PR:                │
  │   • Test pass/fail count                           │
  │   • Code coverage % (new code and total)           │
  │   • Linter warnings/errors                         │
  │   • SAST findings by severity                      │
  │   • Build time                                     │
  │                                                    │
  │ Chronicler aggregates per sprint:                  │
  │   • First-pass review rate (% approved w/o fix)    │
  │   • Defect escape rate (bugs found after merge)    │
  │   • Rework rate (stories reopened)                 │
  │   • Test coverage trend (week over week)           │
  │   • Mean time from PR open to merge                │
  └────────────────────────────────────────────────────┘
  
  No separate Quality agent needed. Every agent owns quality
  within their domain. Chronicler reports the numbers.
```

---

## 13. SCALING PATH — 8 → 12 AGENTS

When to add agents back:

| Trigger | Agent to Add | Splits From |
|---|---|---|
| Frontend codebase > 50 components, UI-specific bugs increasing | **Frontend Developer** | App Developer |
| Backend + adapter code > 100 endpoints | **Backend Developer** | App Developer |
| Self-heal + agent code > 20K LOC, model eval becoming complex | **AI/ML Developer** | Platform & AI/ML Developer |
| Infrastructure + sandbox code growing, K8s config complex | **Platform Developer** | Platform & AI/ML Developer |
| API inconsistencies across 100+ endpoints, need dedicated enforcer | **Standards Architect** | System Architect |
| Quality metrics degrading, need dedicated enforcement | **Quality Agent** | (new) |

**Rule: Add an agent when you feel the pain of its absence, not before.**

---

## 14. ANTI-PATTERNS

| Don't | Why | Instead |
|---|---|---|
| Questioner reviews every small PR | Noise, wastes Developer time | Selective engagement rules (Section 11.2) |
| Both Architects review every PR | Bottleneck, delays merges | Routing table — primary Architect only, both only for API contract changes |
| Chronicler tries to manage the project | Scope creep, duplicates PM | Chronicler is passive: log, digest, alert. That's it. |
| Developers talk to each other | Creates hidden dependencies | All coordination goes through PM |
| Security reviews turn into design reviews | Role confusion | Security comments ONLY on security. Design feedback → Architect. |
| Skipping Developer self-review | Pushes basic issues to reviewers | Self-review checklist is mandatory — PR template enforces it |
| Infinite revision loops | Nothing ships | Max 3 rounds per PR, then escalate to human |

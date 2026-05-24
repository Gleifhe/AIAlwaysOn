# Forge Platform — Hidden Features Addendum
## Architectural Patterns Mined from Cloud & Orchestration Platforms

**Version:** 1.1 Addendum  
**Date:** May 24, 2026  
**Sources:** Azure Durable Functions, AWS Step Functions, Google Cloud Workflows, OCI Connector Hub, Temporal.io, Apache Airflow, AWS Bedrock Agents, Google Gemini Agent Platform

---

## Summary

After deep-diving across 8+ additional platform architectures, I identified **26 hidden features** grouped into 8 categories. These are battle-tested patterns that should be folded into the Forge spec — each one solves a real problem that our competitors either solve poorly or ignore entirely.

---

## 1. DURABLE EXECUTION PATTERNS

### 1.1 Event Sourcing + Replay (from Temporal.io, Azure Durable Functions)

**What it is:** Instead of storing "current state," store the full ordered event history. To recover from a crash, replay the events to reconstruct state.

**Why it matters for Forge:** Our self-heal engine generates scripts, executes them, observes output, and iterates. If a worker dies mid-loop, we lose context. Event sourcing means we can recover the *exact* point in any workflow — including mid-self-heal — and resume without re-running completed work.

**Implementation:**
```
EventHistory:
  workflow_id: UUID
  events:
    - {seq: 1, type: "script_generated", data: {script_id, prompt_hash}}
    - {seq: 2, type: "sandbox_started", data: {sandbox_id, profile}}
    - {seq: 3, type: "execution_completed", data: {exit_code: 1, stderr: "..."}}
    - {seq: 4, type: "self_heal_triggered", data: {diagnosis_id}}
    - {seq: 5, type: "repair_proposed", data: {diff, confidence: 0.87}}
    # Worker dies here — on restart, replay events 1-5, resume from event 6
```

**Key Constraint (from Azure):** Orchestrator code MUST be deterministic. Non-deterministic operations (random, current time, external calls) go into Activities/Scripts, not the orchestrator. This is critical for replay correctness.

**Spec Change:** Add `EventStore` as a core platform service alongside `StateStore`.

---

### 1.2 Dehydration + Timers (from Azure Durable Functions, BizTalk)

**What it is:** Long-running workflows serialize their state ("dehydrate") and release compute resources. Durable timers can sleep for days/weeks/months without holding a thread.

**Why it matters for Forge:** AI self-heal loops may need to wait for human approval (hours/days). Scheduled automations might run weekly. We shouldn't hold a container open waiting.

**Implementation:**
```
DurableTimer:
  workflow_id: UUID
  wake_at: timestamp
  state_snapshot: serialized EventHistory
  reason: enum [human_approval_pending, scheduled_execution, 
                rate_limit_cooldown, external_callback]
```

**From Google Cloud Workflows:** Workflows can wait up to **1 year** for a callback. That's the bar. Our timers should support arbitrary durations with zero compute cost while waiting.

**Spec Change:** Add `DurableTimer` to orchestration layer. Dehydrate workflows when awaiting external events.

---

### 1.3 Checkpointing to Durable Storage (from Google Cloud Workflows)

**What it is:** Google checkpoints every step to **Spanner** (globally consistent DB). If a step succeeds, it's permanent — even if the next step fails or the system crashes.

**Why it matters for Forge:** In a 10-step automation, if step 7 fails, we should never re-run steps 1-6. Each completed step is checkpointed.

**Spec Change:** Checkpoint after every Activity/Script execution. Use PostgreSQL with WAL for single-region; add optional Spanner/CockroachDB for multi-region deployments.

---

## 2. WORKFLOW TYPE SYSTEM

### 2.1 Standard vs Express Workflows (from AWS Step Functions)

**What it is:** Two workflow modes optimized for different workloads:
- **Standard:** Long-running (up to 1 year), exactly-once execution, full history, visual debugging. 2,000 executions/sec.
- **Express:** High-throughput (100,000 exec/sec), at-least-once, short-lived (5 min max). For event streaming, IoT.

**Why it matters for Forge:** Not all automations are the same. A weekly report generation needs full auditability. A webhook handler processing 10K events/sec needs raw throughput. One engine, two modes.

**Implementation:**
```
WorkflowMode:
  standard:
    max_duration: 1 year
    execution_guarantee: exactly-once
    history: full (persisted to EventStore)
    debugging: visual trace
    use_cases: [business_process, self_heal_loop, human_approval, scheduled_job]
  
  express:
    max_duration: 5 minutes
    execution_guarantee: at-least-once
    history: streamed to log (not persisted by default)
    debugging: log-level dependent
    use_cases: [webhook_handler, event_stream, real_time_trigger, high_volume_ingest]
```

**Spec Change:** Add `mode: standard | express` to workflow definitions. Express mode skips EventStore persistence for speed.

---

### 2.2 Saga Pattern for Distributed Transactions (from AWS Step Functions docs)

**What it is:** When a multi-step workflow partially fails, compensating actions undo completed steps. Unlike 2PC (two-phase commit), Sagas work across services without distributed locks.

**Why it matters for Forge:** If an automation creates a cloud VM (step 1), configures DNS (step 2), and fails deploying the app (step 3), we need to clean up the VM and DNS — not leave orphaned resources.

**Implementation:**
```
SagaStep:
  action: Script | APICall
  compensation: Script | APICall  # The undo operation
  
SagaWorkflow:
  steps:
    - {action: "create_vm.py", compensation: "delete_vm.py"}
    - {action: "configure_dns.py", compensation: "remove_dns.py"}
    - {action: "deploy_app.py", compensation: "undeploy_app.py"}
  
  on_failure:
    strategy: enum [compensate_all, compensate_from_failure, manual]
```

**Spec Change:** Add `compensation` field to script/activity definitions. Orchestrator runs compensations in reverse order on failure.

---

## 3. ADVANCED TASK PATTERNS

### 3.1 Map State / Dynamic Parallel Processing (from AWS Step Functions)

**What it is:** Given a dataset (array), run the *same* workflow for each item in parallel. The "Map" state dynamically creates N parallel branches based on data.

**Why it matters for Forge:** "Process all 500 invoices," "Scan all 30 repos for vulnerabilities," "Migrate all customer records." Static parallel branches don't cut it — you need data-driven parallelism.

**Implementation:**
```
MapTask:
  input: Expression → array
  iterator: WorkflowDefinition  # Applied to each item
  max_concurrency: int          # Throttle parallel executions
  tolerated_failure_percentage: float  # e.g., 10% can fail
  result_selector: Expression   # Extract results
```

**Spec Change:** Add `map` node type to orchestration graph alongside existing `parallel`.

---

### 3.2 Sensors / Deferrable Tasks (from Apache Airflow)

**What it is:** Sensors are tasks that **wait** for an external condition — file appears, API returns status, database row exists, time passes. Deferrable tasks free up the worker slot while waiting (via async triggers).

**Why it matters for Forge:** "Wait until the build finishes," "wait until the approval email arrives," "wait until the file lands in S3." These are async waits that shouldn't consume resources.

**Implementation:**
```
Sensor:
  type: enum [file, http_status, db_query, webhook_callback, time, custom]
  poke_interval: duration       # How often to check
  timeout: duration             # Max wait before failure
  mode: enum [poke, reschedule] # poke = hold worker, reschedule = release worker
  condition: Expression         # When is the sensor satisfied?
  
# Example: Wait for file to appear
FileSensor:
  path: "s3://bucket/exports/daily_report_{date}.csv"
  poke_interval: 60s
  timeout: 4h
  mode: reschedule  # Don't hold a worker for 4 hours
```

**Spec Change:** Add `sensor` task type to the workflow engine. Integrate with `DurableTimer` for efficient resource-free waiting.

---

### 3.3 XCom / Cross-Task Communication (from Apache Airflow)

**What it is:** Tasks can push/pull small metadata to share state. Downstream tasks access upstream task outputs by reference.

**Why it matters for Forge:** In a multi-step automation, step 2 often needs the VM IP from step 1, the approval status from the human gate, the error message from the failed step. We need structured inter-task data passing.

**Implementation:**
```
TaskContext:
  # Each task can read upstream outputs
  inputs:
    vm_ip: "{{ tasks.create_vm.output.ip_address }}"
    approval: "{{ tasks.human_gate.output.decision }}"
  
  # Each task can write outputs
  outputs:
    deployment_url: "https://app.example.com"
    
  # For large data (files, datasets), pass by reference
  artifacts:
    report: "artifact://run-123/step-5/report.pdf"
```

**Spec Change:** Add structured `inputs` / `outputs` / `artifacts` to task definitions. Small data inline (XCom pattern), large data via artifact store references.

---

### 3.4 Task Token Callbacks (from AWS Step Functions)

**What it is:** A task sends a unique token to an external system. The workflow pauses. When the external system calls back with the token, the workflow resumes with the result.

**Why it matters for Forge:** Human approval flows, third-party webhook integrations, long-running external processes. Instead of polling, the external system pushes completion back to us.

**Implementation:**
```
CallbackTask:
  type: "wait_for_callback"
  token: auto_generated UUID
  callback_url: "https://forge.example.com/api/v1/callbacks/{token}"
  timeout: 7d
  payload_to_external: {task_details, context, callback_url}
  
  # When external system POSTs to callback_url:
  on_callback:
    validate: token matches
    extract: response body → task output
    resume: workflow continues
```

**Spec Change:** Add callback endpoints to API Gateway. Integrate with DurableTimer for timeout.

---

## 4. CONNECTOR / ADAPTER PATTERNS

### 4.1 Source → Task → Target Pipeline (from OCI Connector Hub)

**What it is:** A three-stage pipeline — data comes from a Source, optionally passes through a Task (transform/filter), then goes to a Target. This is declarative and continuous.

**Why it matters for Forge:** Our adapter layer currently is request-response. Adding continuous connectors enables event-driven integrations — "whenever a log entry matches pattern X, run script Y and store result in Z."

**Implementation:**
```
ContinuousConnector:
  source:
    type: enum [webhook, queue, log_stream, file_watch, schedule]
    config: SourceConfig
  task:
    type: enum [filter, transform, script, none]
    config: TaskConfig  # e.g., filter expression, script_id
  target:
    type: enum [script_trigger, notification, storage, queue, adapter]
    config: TargetConfig
  batch_settings:
    max_size: "1MB"
    max_time: "30s"       # Flush whichever comes first
  delivery: at-least-once
```

**Spec Change:** Add `ContinuousConnector` as a resource type alongside one-shot workflows. This covers the "always-on" integration pattern.

---

### 4.2 Auto-Deactivation of Failing Connectors (from OCI Connector Hub)

**What it is:** If a connector fails continuously for 7 days, it's automatically deactivated with a warning at day 4.

**Why it matters for Forge:** Prevents runaway retry storms consuming resources. If a script keeps failing and self-heal can't fix it, eventually we need to stop trying and alert humans.

**Implementation:**
```
CircuitBreaker:
  failure_threshold: int       # Consecutive failures before warning
  warning_action: notify       # Day 4: send alert
  deactivation_threshold: int  # Consecutive failures before shutdown
  deactivation_action: deactivate + notify  # Day 7: stop + alert
  cooldown_period: duration    # After manual fix, wait before re-evaluating
```

**Spec Change:** Add circuit breaker to all continuous connectors and to the self-heal retry loop. This prevents infinite self-heal loops.

---

### 4.3 Provisioned Concurrency / Warm Starts (from OCI Functions, AWS Lambda)

**What it is:** Keep N function instances "warm" so they respond instantly (sub-second) instead of cold-starting (seconds).

**Why it matters for Forge:** If a high-priority self-heal triggers on a production failure, we can't wait 10 seconds for a sandbox to cold-start. Hot-standby sandboxes for critical paths.

**Implementation:**
```
SandboxPool:
  profile: SandboxProfile
  min_warm: int             # Always keep this many ready
  max_total: int            # Hard cap
  scale_policy:
    metric: pending_executions
    target: 0               # Scale so nothing waits
    cooldown: 60s
  priority_tiers:
    critical: {warm: 5, timeout: 500ms}
    normal: {warm: 2, timeout: 5s}
    low: {warm: 0, timeout: 30s}  # Cold start OK
```

**Spec Change:** Add sandbox pool with warm instances. Priority tiers for different workload classes.

---

## 5. AGENT-SPECIFIC PATTERNS

### 5.1 Action Groups with OpenAPI Schemas (from AWS Bedrock Agents)

**What it is:** Agents invoke tools via structured API definitions (OpenAPI/Swagger). The agent sees the schema, generates the correct API call, and the platform validates + executes it.

**Why it matters for Forge:** Instead of giving agents raw script execution, wrap capabilities in typed Action Groups. The agent knows *what* it can do (schema), the platform validates *how* (parameter validation), and execution is sandboxed.

**Implementation:**
```
ActionGroup:
  name: "infrastructure_management"
  description: "Tools for managing cloud infrastructure"
  schema: OpenAPI 3.0 spec
  actions:
    - name: create_vm
      parameters: {region: string, size: enum, image: string}
      returns: {vm_id: string, ip: string}
      implementation: script_id | adapter_call
      approval_required: true
    - name: list_vms
      parameters: {region?: string}
      returns: VmList
      implementation: adapter_call
      approval_required: false
```

**Spec Change:** Wrap all agent tools in ActionGroup schemas. The agent generates structured calls, not raw code. This is a major security and governance improvement.

---

### 5.2 Agent Guardrails (from AWS Bedrock, Google Vertex AI)

**What it is:** Configurable safety rails that filter agent inputs/outputs. Content filters, topic restrictions, PII detection, word filters, grounding checks.

**Why it matters for Forge:** Our agents generate scripts that execute in production. Guardrails prevent: script injection attacks, PII leakage, off-topic actions, hallucinated API endpoints.

**Implementation:**
```
Guardrails:
  input_filters:
    - type: content_safety    # Block harmful content
    - type: topic_restriction  # Only allow defined topics
      allowed_topics: ["infrastructure", "data_processing", "reporting"]
    - type: pii_detection     # Redact PII from prompts
      action: redact | block
  
  output_filters:
    - type: grounding_check   # Verify claims against knowledge base
      threshold: 0.7
    - type: code_safety       # Scan generated scripts for dangerous patterns
      rules: [no_rm_rf, no_credential_access, no_network_exfiltration]
    - type: hallucination_check  # Flag fabricated API endpoints or paths
  
  action_filters:
    - type: scope_restriction   # Agent can only call defined ActionGroups
    - type: budget_limit       # Cap API/compute spend per execution
      max_cost: "$10.00"
```

**Spec Change:** Add `Guardrails` as a first-class concept applied to every agent. This is distinct from RBAC (which controls who) — guardrails control *what content*.

---

### 5.3 Agent Memory + Knowledge Bases (from AWS Bedrock, CrewAI)

**What it is:** Agents retain context across sessions (memory) and can query private enterprise data (knowledge base / RAG).

**Why it matters for Forge:** Our self-heal engine already has a "knowledge base" for repair patterns. Generalize this: every agent should have configurable short-term memory (conversation), long-term memory (persistent facts), and knowledge base (RAG over domain data).

**Implementation:**
```
AgentMemory:
  short_term:    # Within current workflow execution
    type: buffer_window
    max_tokens: 16000
    
  long_term:     # Persists across executions
    type: vector_store
    store: pgvector
    retention: 90d
    
  episodic:      # Past execution summaries
    type: structured_store
    schema: {task, outcome, lessons_learned, timestamp}
    
  knowledge_base:
    sources:
      - type: document_store   # Runbooks, API docs, SOPs
        path: "s3://forge/knowledge/"
        refresh: daily
      - type: self_heal_patterns  # From successful repairs
        auto_populated: true
    retrieval:
      method: hybrid (keyword + semantic)
      top_k: 10
```

**Spec Change:** Expand agent definition to include memory tiers and knowledge base configuration.

---

### 5.4 Multi-Agent Supervisor Pattern (from AWS Bedrock, Microsoft MAF)

**What it is:** A supervisor agent coordinates specialized sub-agents. The supervisor breaks down tasks, delegates to the right specialist, collects results, and handles failures.

**Why it matters for Forge:** Already in our spec as an orchestration pattern, but the cloud platforms add important details: the supervisor can *dynamically create* sub-agents, route by capability description (not just hardcoded), and handle partial failures.

**Enhancement:**
```
SupervisorAgent:
  routing_strategy: enum [
    capability_match,    # Match task description to agent descriptions
    round_robin,         # Distribute evenly
    load_based,          # Route to least busy
    cost_optimized       # Route to cheapest capable agent
  ]
  failure_handling:
    on_sub_agent_failure: enum [
      retry_same,         # Retry with same agent
      retry_different,    # Try a different agent
      escalate,           # Pass to supervisor for manual handling
      compensate          # Run saga compensation
    ]
  dynamic_delegation: true  # Supervisor can spawn agents not pre-defined
```

---

### 5.5 Agent Versioning + Aliases (from AWS Bedrock Agents)

**What it is:** Agents have immutable versions. Aliases (like "production," "staging") point to specific versions. You can roll back instantly by re-pointing the alias.

**Why it matters for Forge:** When a self-heal agent starts producing bad fixes, we need instant rollback. When deploying a new orchestration pattern, we need canary testing.

**Implementation:**
```
AgentVersion:
  agent_id: UUID
  version: int (auto-increment)
  snapshot: {role, model, tools, guardrails, memory_config}
  created_at: timestamp
  
AgentAlias:
  name: "production" | "staging" | "canary"
  points_to: AgentVersion
  traffic_split:  # For canary deployments
    v5: 90%
    v6: 10%
```

**Spec Change:** Add versioning to agent definitions. Aliases for deployment management. Traffic splitting for canary releases.

---

## 6. SCHEDULING PATTERNS

### 6.1 Data Interval / Backfill (from Apache Airflow)

**What it is:** Every DAG run is parameterized with a "data interval" — the time range it's processing. This enables backfilling: re-running past intervals when logic changes.

**Why it matters for Forge:** "We fixed a bug in the report script. Now re-run it for the last 30 days." Without data intervals, you manually trigger 30 runs. With backfill, it's one command.

**Implementation:**
```
ScheduledWorkflow:
  schedule: "0 6 * * *"  # Cron
  data_interval: {start: execution_date, end: next_execution_date}
  catchup: true          # Auto-run missed intervals on deploy
  backfill:
    command: "forge backfill --workflow report_gen --start 2026-04-01 --end 2026-05-01"
    max_parallel: 5
```

**Spec Change:** Add data interval concept to scheduler. Enable backfill command.

---

### 6.2 Trigger Rules / Branching (from Apache Airflow)

**What it is:** Tasks don't just run when "all upstream succeed." You can configure: run when *any* upstream succeeds, when all finish (regardless of status), when one fails, etc.

**Why it matters for Forge:** "If *any* vulnerability scanner finds a critical issue, trigger the remediation workflow." "After all parallel tasks complete (even with partial failures), run the summary."

**Implementation:**
```
TriggerRule:
  all_success      # Default: all parents must succeed
  all_failed       # All parents must fail (for error aggregation)
  all_done         # All parents must complete (success or fail)
  one_success      # At least one parent succeeded
  one_failed       # At least one parent failed (early alert)
  none_failed      # No parent failed (but some may be skipped)
  none_skipped     # No parent was skipped
  always           # Run regardless of parent state
```

**Spec Change:** Add `trigger_rule` to task definitions, defaulting to `all_success`.

---

### 6.3 Pools / Concurrency Control (from Apache Airflow)

**What it is:** Pools limit how many tasks of a certain type can run simultaneously. Prevents overwhelming external systems.

**Why it matters for Forge:** "Only 3 concurrent API calls to the CRM," "only 1 database migration at a time," "max 10 parallel sandbox executions per tenant."

**Implementation:**
```
Pool:
  name: "crm_api"
  slots: 3
  description: "Limit concurrent CRM API calls to avoid rate limits"

# Tasks claim pool slots:
Task:
  pool: "crm_api"
  pool_slots: 1  # How many slots this task claims
```

**Spec Change:** Add resource pools with configurable slot counts. Tasks declare pool membership.

---

## 7. OBSERVABILITY ENHANCEMENTS

### 7.1 Visual Workflow Debugging (from AWS Step Functions)

**What it is:** Step-by-step visual replay of workflow execution. Click any state to see input, output, duration, errors. Color-coded by status.

**Why it matters for Forge:** "Why did the self-heal fail?" should be answerable by clicking through a visual execution timeline, not reading logs.

**Spec Change:** Portal UI must include visual workflow trace viewer with:
- Color-coded step status (green/red/yellow/gray)
- Click-to-inspect input/output for each step
- Timeline view with durations
- Side-by-side comparison of original vs. self-healed execution

---

### 7.2 Execution History with Replay (from Temporal.io)

**What it is:** Full event history stored for every execution. You can replay any past execution to debug issues with the exact same inputs.

**Why it matters for Forge:** "This automation worked yesterday but fails today." Replay yesterday's successful execution with today's inputs to pinpoint what changed.

**Spec Change:** Store full event history per execution. Add `forge replay --execution-id <id>` CLI command.

---

## 8. OPEN STANDARDS & PORTABILITY

### 8.1 CloudEvents (from OCI Functions)

**What it is:** CNCF standard for describing events in a common way. Vendor-neutral event schema.

**Why it matters for Forge:** Our event bus should speak CloudEvents natively. This means any CloudEvents-compatible system can trigger Forge workflows and vice versa.

**Spec Change:** Event bus messages conform to CloudEvents spec. All triggers and webhooks emit/consume CloudEvents.

---

### 8.2 Declarative Workflow Definition (from AWS States Language, Google Workflows YAML)

**What it is:** Workflows defined in JSON/YAML, not just code. Machine-readable, version-controllable, diff-able.

**Why it matters for Forge:** Our PM Agent generates task specs, our Builder Agent writes code. A YAML workflow definition is the perfect handoff format — the PM Agent can generate it, developers can review it, and the engine can execute it.

**Implementation:**
```yaml
# forge-workflow.yaml
name: daily_report
version: "1.2.0"
mode: standard
schedule: "0 6 * * *"

steps:
  - name: fetch_data
    type: script
    script: scripts/fetch_report_data.py
    timeout: 5m
    retry: {max: 3, backoff: exponential}
    outputs: [raw_data_path]

  - name: transform
    type: script
    script: scripts/transform_report.py
    inputs:
      data_path: "{{ steps.fetch_data.outputs.raw_data_path }}"
    timeout: 10m

  - name: validate
    type: agent
    agent: data_quality_checker
    inputs:
      report: "{{ steps.transform.outputs.report_path }}"
    on_failure:
      trigger: self_heal
      max_attempts: 2

  - name: distribute
    type: parallel
    branches:
      - {script: scripts/email_report.py}
      - {script: scripts/upload_to_s3.py}
      - {script: scripts/post_to_slack.py}
    trigger_rule: all_done
```

**Spec Change:** Define `forge-workflow.yaml` schema. Support both YAML definitions and programmatic (Python/TS SDK) workflow creation.

---

## PRIORITY MATRIX — Which features to add to which phase

| Priority | Feature | Phase | Rationale |
|---|---|---|---|
| **P0 — Must have** | Event Sourcing + Replay | Phase 1 (Core Runtime) | Foundation for durability |
| **P0** | Checkpointing | Phase 1 | Without this, crash = restart from zero |
| **P0** | XCom / Cross-task data passing | Phase 1 | Every multi-step workflow needs this |
| **P0** | Declarative YAML workflows | Phase 1 | Enables PM Agent to generate workflows |
| **P0** | ActionGroup schemas for agents | Phase 2 (Intelligence) | Security-critical for production agents |
| **P0** | Agent Guardrails | Phase 2 | Non-negotiable for production AI |
| **P0** | Circuit Breaker | Phase 2 | Prevents runaway self-heal loops |
| **P1 — Should have** | Standard/Express modes | Phase 1 | Enables both batch and real-time use cases |
| **P1** | Saga compensation | Phase 1 | Required for any workflow that creates resources |
| **P1** | Durable Timers | Phase 1 | Required for human-in-the-loop flows |
| **P1** | Map State (dynamic parallel) | Phase 1 | Required for data-driven workflows |
| **P1** | Sensors / Deferrable Tasks | Phase 2 | Efficient async waiting |
| **P1** | Task Token Callbacks | Phase 2 | External system integration |
| **P1** | Agent Memory tiers | Phase 2 | Required for effective self-heal learning |
| **P1** | Trigger Rules | Phase 1 | Flexible control flow |
| **P1** | Visual Workflow Debugger | Phase 2 | Essential for operator experience |
| **P1** | Agent Versioning + Aliases | Phase 4 (Build System) | Safe agent deployment |
| **P2 — Nice to have** | Continuous Connectors | Phase 3 (Integration) | Event-driven integrations |
| **P2** | Warm Sandbox Pool | Phase 3 | Performance optimization |
| **P2** | Backfill / Data Intervals | Phase 3 | Scheduler enhancement |
| **P2** | Pools / Concurrency Control | Phase 3 | Resource management |
| **P2** | CloudEvents support | Phase 3 | Interoperability |
| **P2** | Supervisor dynamic routing | Phase 4 | Advanced multi-agent |
| **P2** | Agent canary deployments | Phase 5 (Hardening) | Production safety |
| **P2** | Execution replay CLI | Phase 5 | Advanced debugging |
| **P2** | Auto-deactivation of connectors | Phase 3 | Operational safety |

---

## REVISED ARCHITECTURE DIAGRAM (with addendum features)

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            FORGE PLATFORM v1.1                          │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────────────┐    │
│  │  Portal UI   │  │  CLI / SDK   │  │  API Gateway + Callbacks    │    │
│  │  + Visual    │  │  + Backfill  │  │  + CloudEvents + WebSocket  │    │
│  │  Debugger    │  │  + Replay    │  │                             │    │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬──────────────────┘    │
│         │                 │                      │                       │
│  ┌──────▼─────────────────▼──────────────────────▼────────────────────┐  │
│  │                ORCHESTRATION LAYER                                  │  │
│  │  ┌─────────────┐ ┌──────────────┐ ┌─────────────┐ ┌────────────┐  │  │
│  │  │  Workflow    │ │  Agent       │ │  Rules      │ │  Scheduler │  │  │
│  │  │  Engine      │ │  Orchestrator│ │  Engine     │ │  + Backfill│  │  │
│  │  │ ┌──────────┐│ │ ┌──────────┐ │ │             │ │  + Cron    │  │  │
│  │  │ │Standard/ ││ │ │Supervisor│ │ │             │ │  + Data    │  │  │
│  │  │ │Express   ││ │ │+ Routing │ │ │             │ │  Intervals │  │  │
│  │  │ │Modes     ││ │ └──────────┘ │ │             │ └────────────┘  │  │
│  │  │ │+ Saga    ││ │ ┌──────────┐ │ │             │ ┌────────────┐  │  │
│  │  │ │+ Map     ││ │ │Guardrails│ │ │             │ │  Sensors   │  │  │
│  │  │ │+ Sensors ││ │ │+ Versions│ │ │             │ │  + Timers  │  │  │
│  │  │ │+ Triggers││ │ └──────────┘ │ │             │ │  + Callbacks│ │  │
│  │  │ └──────────┘│ └──────────────┘ └─────────────┘ └────────────┘  │  │
│  │  └─────────────┘                                                   │  │
│  └───────────────────────────┬────────────────────────────────────────┘  │
│                              │                                           │
│  ┌───────────────────────────▼────────────────────────────────────────┐  │
│  │               DURABLE EXECUTION LAYER (NEW)                        │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌───────────────────────────┐  │  │
│  │  │  Event Store │ │  Checkpoint  │ │   XCom / Task Context     │  │  │
│  │  │  (Event      │ │  Manager     │ │   (Cross-task data)       │  │  │
│  │  │   Sourcing)  │ │              │ │                           │  │  │
│  │  └──────────────┘ └──────────────┘ └───────────────────────────┘  │  │
│  └───────────────────────────┬────────────────────────────────────────┘  │
│                              │                                           │
│  ┌───────────────────────────▼────────────────────────────────────────┐  │
│  │                SCRIPT RUNTIME LAYER                                 │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────┐ │  │
│  │  │  Script      │ │  Sandbox     │ │  Self-Heal   │ │  Circuit │ │  │
│  │  │  Registry    │ │  Manager     │ │  Engine      │ │  Breaker │ │  │
│  │  │  + Action    │ │  + Warm Pool │ │  + Knowledge │ │          │ │  │
│  │  │  Groups      │ │  + Priority  │ │    Base      │ │          │ │  │
│  │  │              │ │    Tiers     │ │  + Memory    │ │          │ │  │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────┘ │  │
│  └───────────────────────────┬────────────────────────────────────────┘  │
│                              │                                           │
│  ┌───────────────────────────▼────────────────────────────────────────┐  │
│  │                INTEGRATION LAYER                                    │  │
│  │  ┌─────────────────────┐  ┌────────────────────────────────────┐  │  │
│  │  │  Request/Response   │  │  Continuous Connectors             │  │  │
│  │  │  Adapters           │  │  (Source → Task → Target)          │  │  │
│  │  │  MCP|A2A|REST|gRPC  │  │  + Auto-deactivation              │  │  │
│  │  │  SOAP|EDI|Legacy    │  │  + Batch settings                 │  │  │
│  │  └─────────────────────┘  └────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                PLATFORM SERVICES                                    │  │
│  │  Identity │ Observability │ Secrets │ Event Bus │ Governance       │  │
│  │  & RBAC   │ + Visual Debug│ Manager │ CloudEvents│ + Pools         │  │
│  │           │ + Replay      │         │           │ + Concurrency    │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## CLOSING NOTES

These 26 features represent the combined lessons of **$billions in R&D** across Azure, AWS, Google, Oracle, Temporal, and the Apache ecosystem. The key insight: **no single platform has assembled all of these**. Each cloud vendor optimized for their own ecosystem. By cherry-picking the best patterns and combining them with our AI-native script-first model, Forge can offer something genuinely differentiated.

The most important additions to the original spec are:
1. **Durable Execution Layer** (Event Sourcing + Checkpointing) — makes the platform crash-proof
2. **Agent Guardrails + Action Groups** — makes AI-generated execution production-safe
3. **Workflow Type System** (Standard/Express + Saga) — handles both batch and real-time
4. **Circuit Breaker** — prevents infinite self-heal loops
5. **Declarative YAML workflows** — enables the PM Agent to directly generate executable specs

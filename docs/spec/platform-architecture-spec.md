# AI & Automation Platform — Architecture Specification

**Codename:** Forge  
**Version:** 1.0 DRAFT  
**Date:** May 24, 2026  
**Author:** Platform Architecture Team  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Competitive Landscape Analysis](#2-competitive-landscape-analysis)
3. [Platform Architecture Specification](#3-platform-architecture-specification)
4. [Agent-Driven Build System](#4-agent-driven-build-system)
5. [Development Phases & Milestones](#5-development-phases--milestones)
6. [Appendices](#6-appendices)

---

## 1. Executive Summary

### Vision

Build a self-healing, script-centric AI automation platform that treats **scripts as first-class citizens**. Unlike existing platforms that bolt AI onto traditional workflow engines, Forge starts from the observation that modern AI agents fundamentally operate by: (1) generating scripts/code, (2) executing them in sandboxed environments, (3) observing system state, and (4) self-healing by understanding failures. The platform makes this loop **explicit, observable, governable, and composable**.

### Core Thesis

> Most AI automation today is LLMs writing scripts, running them, reading the output, and iterating. Forge makes this pattern a first-class, production-grade, enterprise-secure platform primitive — not a side effect of a chatbot.

### Key Differentiators

- **Script-as-Workflow**: Every automation is a versioned, auditable script with AI-powered generation and self-repair
- **Self-Healing Runtime**: Failures trigger AI-driven diagnosis and automatic repair with human-in-the-loop guardrails
- **Universal Adapter Layer**: Protocol-native integration (MCP, A2A, REST, gRPC, legacy adapters)
- **Observable by Default**: Every decision, script execution, and self-heal attempt is traced and auditable
- **Multi-Agent Orchestration**: Native support for agent teams with defined roles, handoffs, and escalation

---

## 2. Competitive Landscape Analysis

### 2.1 Platform Comparison Matrix

| Dimension | OpenHands | MS BizTalk (Legacy) | MS Agent Framework (MAF) | MS Foundry Agent Service | LangGraph | CrewAI | n8n |
|---|---|---|---|---|---|---|---|
| **Primary Focus** | AI-driven software development | Enterprise integration middleware | Multi-agent orchestration framework | Managed agent hosting | Agent workflow graphs | Multi-agent crews | Visual workflow automation |
| **Architecture** | Sandboxed runtime + LLM loop | Pub/sub message broker | Graph-based orchestration | Managed cloud PaaS | State machine graphs | Role-based agent teams | Node-based visual DAGs |
| **AI Native?** | Yes (core) | No (bolted on) | Yes | Yes | Yes | Yes | Partial (AI nodes added) |
| **Self-Healing** | Yes (iterates on failures) | No | Limited (retry/middleware) | No | No (manual error handling) | Limited | No (error branches) |
| **Script Execution** | Docker sandboxes | .NET assemblies | Python/.NET code | Container-based | Python functions | Python tasks | JS/Python code nodes |
| **Enterprise Security** | Basic (MIT + Enterprise) | Full (Windows/AD) | Azure RBAC, Entra | Full Azure stack | DIY | DIY | SSO, RBAC, LDAP |
| **Observability** | Basic logging | BAM dashboards | OpenTelemetry | App Insights, tracing | LangSmith integration | Basic | Audit logs, log streaming |
| **Deployment** | Self-hosted / Cloud | On-prem Windows | Self-hosted / Azure | Fully managed Azure | Self-hosted | Self-hosted / Cloud | Self-hosted / Cloud |
| **Protocol Support** | Git, CLI | EDI, SOAP, WCF, adapters | A2A, MCP | A2A, MCP, OpenResponses | Custom | Custom | Webhooks, 500+ integrations |
| **Multi-Agent** | Single agent focus | N/A (orchestrations) | Yes (graph workflows) | Yes (workflow agents) | Yes (graph-native) | Yes (core feature) | Partial (sub-workflows) |
| **Maturity** | Growing (74.7k stars) | Discontinued (2020) | New (10.7k stars) | GA (Azure) | Mature (widely adopted) | Growing | Mature (189k stars) |
| **License** | MIT + Enterprise | Proprietary | MIT | Proprietary (Azure) | MIT | MIT | Fair-code / Enterprise |

### 2.2 Detailed Platform Analysis

#### OpenHands (formerly OpenDevin)
**What it is:** An AI-driven development platform that runs AI agents in sandboxed Docker containers to write, test, and debug code.

**Key Benefits:**
- True script-execute-observe-repair loop
- SDK for composing agents, CLI for interactive use, GUI for visual interaction
- Enterprise features: Kubernetes support, Jira/Slack/Linear integrations
- Strong benchmark scores on SWE-Bench
- Large community (74.7k GitHub stars, 502 contributors)

**Key Detractors:**
- Focused narrowly on software development tasks
- Not designed for business process automation
- Enterprise features require license purchase
- Heavy Docker dependency for runtime isolation
- Limited protocol support beyond Git workflows

#### Microsoft BizTalk Server (Legacy Reference)
**What it is:** Discontinued (2020) enterprise integration middleware using pub/sub architecture, XML transforms, and adapters.

**Key Benefits (Historical Design Patterns Worth Adopting):**
- **Adapter pattern**: Clean separation between protocols and business logic
- **Dehydration/rehydration**: Serializing long-running processes to survive restarts
- **Business rules engine**: Externalized, Rete-algorithm rule evaluation
- **Business Activity Monitoring (BAM)**: Real-time operational dashboards
- **Pipeline architecture**: Composable receive/send processing stages

**Key Detractors (Why It Died):**
- Windows-only, SQL Server dependency
- Extremely complex XML/XSLT mapping tooling
- No AI capabilities
- Expensive licensing
- Slow development cycle — couldn't keep up with cloud-native patterns
- Human-centric processes required SharePoint bolt-on

#### Microsoft Agent Framework (MAF, successor to AutoGen)
**What it is:** Open-source multi-language framework for building production-grade AI agents in .NET and Python.

**Key Benefits:**
- Graph-based orchestration: sequential, concurrent, handoff, group collaboration
- Checkpointing, streaming, human-in-the-loop, time-travel debugging
- Middleware system for request/response pipelines
- Declarative agents via YAML
- Foundry hosting integration
- OpenTelemetry observability built-in
- Dual-language: Python and C#/.NET

**Key Detractors:**
- Very new (just hit 1.0), API surface still evolving
- Tightly coupled to Azure/Foundry ecosystem for hosting
- No built-in self-healing beyond retry middleware
- Limited visual tooling (DevUI is early)
- Requires significant developer skill to build orchestrations

#### Microsoft Foundry Agent Service
**What it is:** Fully managed Azure PaaS for deploying and scaling AI agents.

**Key Benefits:**
- Three agent types: Prompt (no-code), Workflow (YAML/visual), Hosted (containers)
- Enterprise identity via Entra, RBAC, content safety filters
- Built-in tools: web search, file search, memory, code interpreter, MCP servers
- Agent-to-agent communication (A2A protocol)
- Publishing to Teams, M365 Copilot, Entra Agent Registry
- VM-isolated sandboxes for Hosted agents
- Full development lifecycle support (create → test → trace → evaluate → publish → monitor)

**Key Detractors:**
- Azure lock-in (cannot self-host)
- Hosted agents and workflows still in preview
- Cost model tied to Azure consumption
- Limited customization of the underlying runtime
- Dependent on Azure model catalog availability

#### LangGraph (LangChain)
**What it is:** Open-source agent runtime and low-level orchestration framework using state machine graphs.

**Key Benefits:**
- Fine-grained control over agent reasoning flows
- Human-in-the-loop via graph interrupts
- Persistent memory across sessions
- First-class streaming support
- Composable: single, multi-agent, hierarchical architectures
- Large ecosystem via LangChain integrations
- LangSmith for observability and evaluation

**Key Detractors:**
- Python-centric (TypeScript secondary)
- Steep learning curve for graph-based programming
- LangSmith (observability) is proprietary SaaS
- No built-in deployment/hosting story (LangServe is separate)
- No self-healing primitives
- Vendor coupling to LangChain ecosystem

#### CrewAI
**What it is:** Framework for building collaborative AI agent teams with defined roles.

**Key Benefits:**
- Intuitive role/crew/task mental model
- Built-in memory, knowledge, and guardrails
- Flow orchestration beyond simple agent teams
- Enterprise features (observability baked in)
- Good for non-developer persona (declarative crew definitions)

**Key Detractors:**
- Less granular control than LangGraph
- Smaller community than alternatives
- Python-only
- Limited deployment and hosting options
- Enterprise governance features are newer

#### n8n
**What it is:** Visual workflow automation platform with 500+ integrations and AI capabilities.

**Key Benefits:**
- Massive integration library (500+ pre-built connectors)
- Visual canvas makes workflows inspectable and debuggable
- Self-hostable with Docker
- AI nodes: LangChain integration, AI agents, RAG support, MCP client/server
- Strong community (189k GitHub stars)
- Code nodes (JS/Python) for escape hatches
- Enterprise: SSO, RBAC, LDAP, audit logs, log streaming

**Key Detractors:**
- AI is added via nodes, not the core execution model
- No self-healing — failures go to error branches
- Not designed for multi-agent orchestration
- Fair-code license restricts some commercial use
- Performance limitations at extreme scale
- Workflow model doesn't natively express agent reasoning loops

### 2.3 Gap Analysis — What's Missing Across All Platforms

| Gap | Description |
|---|---|
| **Script-first model** | No platform treats the generate→execute→observe→repair script loop as its primary primitive |
| **Unified self-healing** | OpenHands self-heals for code; no platform self-heals for business automation workflows |
| **Cross-paradigm orchestration** | Can't easily mix agent reasoning, traditional workflows, rules engines, and scripts |
| **Protocol bridge** | No single platform natively bridges MCP, A2A, legacy protocols (EDI, SOAP), and modern APIs |
| **Governance for AI scripts** | Audit trails for AI-generated code execution in production are weak everywhere |
| **Self-building capability** | No platform can use its own agents to extend and improve itself |

---

## 3. Platform Architecture Specification

### 3.1 Architecture Principles

1. **Scripts are the Unit of Work** — Every automation reduces to a versioned, sandboxed script
2. **Observe Everything** — All agent decisions, script executions, and state changes are traced
3. **Self-Heal by Default** — Failed scripts trigger AI diagnosis; repairs require approval per policy
4. **Secure by Construction** — Sandboxed execution, principle of least privilege, audit trails
5. **Protocol Agnostic** — Adapters abstract away connectivity; the core never sees wire protocols
6. **Composable Agents** — Agents are building blocks; orchestration patterns are plug-and-play
7. **Progressive Disclosure** — No-code for simple tasks, low-code for intermediate, full-code for advanced

### 3.2 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         FORGE PLATFORM                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  Portal UI   │  │   CLI/SDK    │  │   API Gateway (REST/gRPC)│  │
│  │  (React)     │  │  (Python/TS) │  │   + WebSocket            │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────────┘  │
│         │                 │                      │                  │
│  ┌──────▼─────────────────▼──────────────────────▼───────────────┐  │
│  │                   ORCHESTRATION LAYER                          │  │
│  │  ┌─────────────┐ ┌──────────────┐ ┌────────────────────────┐  │  │
│  │  │  Workflow    │ │  Agent       │ │   Rules Engine         │  │  │
│  │  │  Engine      │ │  Orchestrator│ │   (Rete-based)         │  │  │
│  │  │  (DAG-based) │ │  (Graph)     │ │                        │  │  │
│  │  └─────────────┘ └──────────────┘ └────────────────────────┘  │  │
│  └───────────────────────────┬───────────────────────────────────┘  │
│                              │                                      │
│  ┌───────────────────────────▼───────────────────────────────────┐  │
│  │                   SCRIPT RUNTIME LAYER                         │  │
│  │  ┌──────────────┐ ┌──────────────┐ ┌───────────────────────┐  │  │
│  │  │  Script      │ │  Sandbox     │ │   Self-Heal Engine    │  │  │
│  │  │  Registry    │ │  Manager     │ │   (AI Diagnosis +     │  │  │
│  │  │  (Versioned) │ │  (Container/ │ │    Auto-Repair)       │  │  │
│  │  │              │ │   MicroVM)   │ │                       │  │  │
│  │  └──────────────┘ └──────────────┘ └───────────────────────┘  │  │
│  └───────────────────────────┬───────────────────────────────────┘  │
│                              │                                      │
│  ┌───────────────────────────▼───────────────────────────────────┐  │
│  │                   INTEGRATION LAYER                            │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────────┐  │  │
│  │  │  MCP   │ │  A2A   │ │  REST  │ │  gRPC  │ │  Legacy    │  │  │
│  │  │Adapter │ │Adapter │ │Adapter │ │Adapter │ │  Adapters  │  │  │
│  │  │        │ │        │ │        │ │        │ │(SOAP/EDI/  │  │  │
│  │  │        │ │        │ │        │ │        │ │ FTP/SFTP)  │  │  │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                   PLATFORM SERVICES                            │  │
│  │  ┌──────────┐ ┌────────────┐ ┌──────────┐ ┌───────────────┐  │  │
│  │  │ Identity │ │Observability│ │ Secrets  │ │  Event Bus    │  │  │
│  │  │ & RBAC   │ │ (OTel +    │ │ Manager  │ │  (Pub/Sub)    │  │  │
│  │  │          │ │  Traces)   │ │          │ │               │  │  │
│  │  └──────────┘ └────────────┘ └──────────┘ └───────────────┘  │  │
│  │  ┌──────────┐ ┌────────────┐ ┌──────────┐ ┌───────────────┐  │  │
│  │  │ State    │ │  Scheduler │ │ Artifact │ │  Governance   │  │  │
│  │  │ Store    │ │  (Cron +   │ │ Store    │ │  & Compliance │  │  │
│  │  │          │ │  Event)    │ │          │ │               │  │  │
│  │  └──────────┘ └────────────┘ └──────────┘ └───────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                   DATA LAYER                                   │  │
│  │  PostgreSQL │ Redis │ S3-Compatible │ Vector DB │ Time-Series  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.3 Core Components — Detailed Specifications

#### 3.3.1 Script Registry

**Purpose:** Version-controlled repository for all scripts (AI-generated and human-authored).

```
ScriptRecord:
  id: UUID
  name: string
  version: semver
  language: enum [python, typescript, bash, powershell, sql]
  source_code: text
  hash: sha256
  metadata:
    author: string (human or agent ID)
    generated_by: string (model + prompt hash)
    tags: string[]
    approved_by: string | null
    approval_policy: enum [auto, human-review, dual-approval]
  dependencies: Dependency[]
  sandbox_profile: SandboxProfile
  test_suite: TestCase[]
  created_at: timestamp
  state: enum [draft, review, approved, active, deprecated, revoked]
```

**Key Operations:**
- `create(script)` — Register a new script (AI or human)
- `promote(id, from_state, to_state)` — State transition with approval check
- `fork(id)` — Create a new version for self-heal iteration
- `diff(v1, v2)` — Compare script versions
- `audit_trail(id)` — Full history of who/what created, modified, approved

**Governance Rules:**
- AI-generated scripts MUST go through at least one approval gate before `active` state
- Approval policy is configurable per-workspace: auto (for low-risk), human-review (default), dual-approval (for production-critical)
- All script executions are logged with the exact version hash

#### 3.3.2 Sandbox Manager

**Purpose:** Secure, isolated execution environments for scripts.

```
SandboxProfile:
  runtime: enum [container, microvm, wasm]
  base_image: string
  resource_limits:
    cpu: string (e.g., "0.5")
    memory: string (e.g., "512Mi")
    timeout: duration
    max_network_connections: int
    disk_quota: string
  network_policy:
    allow_egress: CIDRRule[]
    allow_dns: string[]
    deny_all_by_default: bool
  filesystem:
    read_only_mounts: Mount[]
    writable_volume: string
    max_files: int
  capabilities:
    allow_tools: string[]   # e.g., ["http_client", "file_read", "db_query"]
    deny_tools: string[]
```

**Execution Flow:**
1. Script Registry resolves script + dependencies
2. Sandbox Manager provisions isolated environment
3. Script executes with resource constraints
4. stdout/stderr/exit code captured
5. Execution artifacts (files, data) stored
6. Sandbox destroyed (ephemeral by default)

**Security Model:**
- Default deny-all network policy
- Read-only filesystem except for designated output volume
- Resource limits enforced (CPU, memory, time)
- No privilege escalation
- Container/MicroVM isolation between tenants
- Tool access is whitelist-based per sandbox profile

#### 3.3.3 Self-Heal Engine

**Purpose:** AI-driven diagnosis and repair of failed script executions.

```
SelfHealLoop:
  1. DETECT    — Execution fails (non-zero exit, timeout, exception)
  2. DIAGNOSE  — AI agent analyzes: error output, script source, system state, historical fixes
  3. PROPOSE   — AI generates a repair (modified script, config change, or escalation)
  4. GATE      — Approval check per policy (auto-approve, human-review, or block)
  5. APPLY     — Execute repaired script in sandbox
  6. VERIFY    — Run test suite + compare output to expected state
  7. LEARN     — Store successful repair pattern for future reference
  8. ESCALATE  — If max retries exceeded or confidence too low → human escalation

HealAttempt:
  id: UUID
  original_execution_id: UUID
  diagnosis: text (AI reasoning)
  proposed_fix: ScriptDiff
  confidence_score: float [0.0 - 1.0]
  approval_gate: enum [auto, human, blocked]
  outcome: enum [success, failed, escalated]
  attempt_number: int
  max_attempts: int (configurable, default: 3)
```

**Heal Policies:**
| Risk Level | Auto-Approve Threshold | Max Auto-Retries | Escalation |
|---|---|---|---|
| Low (read-only, non-prod) | confidence >= 0.8 | 5 | Notify |
| Medium (writes, staging) | confidence >= 0.9 | 3 | Queue for review |
| High (production, financial) | Never auto-approve | 1 | Immediate page |

**Knowledge Base:**
- Every successful self-heal is stored as a pattern
- Patterns are indexed by: error signature, script type, system context
- Future diagnoses query the pattern store first (RAG over repair history)
- This creates a **compound learning effect** — the platform gets better at self-healing over time

#### 3.3.4 Agent Orchestrator

**Purpose:** Multi-agent coordination engine using graph-based workflow definitions.

```
AgentDefinition:
  id: UUID
  name: string
  role: string (system prompt / persona)
  model: ModelConfig
  tools: ToolBinding[]
  memory: MemoryConfig
  constraints:
    max_steps: int
    max_tokens_per_step: int
    allowed_actions: string[]
    forbidden_actions: string[]

OrchestrationGraph:
  nodes: AgentNode[]
  edges: Edge[]
  patterns:
    - sequential    # A → B → C
    - parallel      # A → [B, C] → D
    - handoff       # A decides to delegate to B or C
    - group_chat    # A, B, C collaborate on shared context
    - hierarchical  # Supervisor A manages workers B, C, D
    - self_loop     # A iterates on own output
```

**Orchestration Patterns (Built-In):**

1. **Sequential Pipeline**: Scripts/agents execute in order, output feeds to next input
2. **Parallel Fan-Out/Fan-In**: Multiple scripts run concurrently, results aggregated
3. **Supervisor-Worker**: Supervisor agent delegates tasks, reviews results, re-delegates on failure
4. **Consensus**: Multiple agents vote; majority or unanimous agreement required
5. **Human-in-the-Loop**: Workflow pauses, presents context to human, resumes on approval
6. **Self-Heal Loop**: Execute → Fail → Diagnose → Repair → Re-execute (see 3.3.3)

#### 3.3.5 Integration Layer (Adapter Framework)

**Purpose:** Protocol-agnostic connectivity to external systems.

```
Adapter:
  id: UUID
  protocol: enum [mcp, a2a, rest, grpc, graphql, soap, edi, ftp, sftp, smtp, 
                   database, message_queue, file_system, custom]
  config: AdapterConfig
  auth: AuthConfig
  health_check: HealthCheckConfig
  rate_limits: RateLimitConfig
  
AdapterConfig:
  endpoint: string
  headers: map[string, string]
  timeout: duration
  retry_policy: RetryPolicy
  transform_in: TransformPipeline    # Normalize inbound to internal schema
  transform_out: TransformPipeline   # Internal schema to outbound protocol

AuthConfig:
  method: enum [none, api_key, oauth2, mtls, saml, managed_identity]
  credentials_ref: string  # Reference to Secrets Manager
```

**Pre-Built Adapters (Priority Order):**

| Phase | Adapters |
|---|---|
| MVP | REST, MCP, Database (Postgres/MySQL/MSSQL), File System |
| Phase 2 | A2A, gRPC, Message Queues (RabbitMQ, Kafka), SMTP, SFTP |
| Phase 3 | SOAP, EDI, Legacy ERP connectors, Custom adapter SDK |

#### 3.3.6 Observability Stack

**Purpose:** Full visibility into every platform operation.

```
Trace:
  trace_id: UUID
  spans:
    - agent_decision (which agent, what reasoning, what model)
    - script_generation (prompt, model, generated code)
    - approval_gate (who approved, policy applied)
    - script_execution (sandbox ID, duration, exit code)
    - self_heal_attempt (diagnosis, proposed fix, outcome)
    - adapter_call (protocol, endpoint, latency, status)

Metrics:
  - script_executions_total (by status, language, agent)
  - self_heal_success_rate (by script type, time window)
  - agent_decision_latency_p99
  - adapter_call_latency (by protocol, endpoint)
  - sandbox_utilization (CPU, memory, concurrent)
  - approval_queue_depth
  - governance_violations_total

Logs:
  - Structured JSON, correlated by trace_id
  - Shipped to customer's SIEM (Splunk, Elastic, Datadog)
  - Retention configurable per compliance requirements
```

**Implementation:** OpenTelemetry SDK → Collector → Configurable backends (Jaeger, Prometheus, Grafana, customer SIEM)

#### 3.3.7 Identity, RBAC & Governance

```
Roles:
  platform_admin     # Full control
  workspace_owner    # Manage workspace, agents, scripts
  developer          # Create/edit scripts and agents
  operator           # Execute, monitor, approve
  viewer             # Read-only access
  agent              # Non-human identity for AI agents

Permissions (Resource-Based):
  scripts:     [create, read, update, delete, approve, execute]
  agents:      [create, read, update, delete, deploy, invoke]
  workflows:   [create, read, update, delete, execute, pause, cancel]
  adapters:    [create, read, update, delete, test]
  secrets:     [create, read, rotate]
  audit_logs:  [read, export]

Governance Policies:
  - AI-generated scripts require approval before production execution
  - Self-heal auto-approve thresholds configurable per risk level
  - All model calls logged with prompt/response hashes
  - Data classification labels (PII, financial, confidential) flow through pipelines
  - Compliance templates: SOC2, HIPAA, GDPR (configurable)
```

### 3.4 Technology Stack Recommendations

| Layer | Technology | Rationale |
|---|---|---|
| **Portal UI** | React + TypeScript | Industry standard, component ecosystem |
| **API Gateway** | Go (Chi/Gin) | Performance for high-concurrency API routing |
| **Orchestration Engine** | Python | AI/ML ecosystem, LLM library support |
| **Script Runtime** | Python + TypeScript + Bash | Multi-language support for generated scripts |
| **Sandbox** | Firecracker MicroVMs + Docker fallback | Security isolation (Firecracker) + developer ergonomics (Docker) |
| **Event Bus** | NATS JetStream | Lightweight, high-performance, supports pub/sub + queues |
| **State Store** | PostgreSQL + Redis | Relational for workflows, Redis for cache/locks |
| **Artifact Store** | S3-compatible (MinIO for self-hosted) | Blob storage for script outputs, logs |
| **Vector DB** | pgvector (PostgreSQL extension) | Self-heal knowledge base, RAG over repair patterns |
| **Time-Series** | TimescaleDB (PostgreSQL extension) | Metrics storage, keeps the Postgres ecosystem |
| **Observability** | OpenTelemetry → Grafana stack | Open standards, self-hostable |
| **Identity** | Keycloak (self-hosted) or OIDC integration | Enterprise SSO, RBAC |
| **CI/CD** | GitHub Actions / GitLab CI | Platform's own agents can participate in CI/CD |

### 3.5 Deployment Architecture

```
                    ┌─────────────────────┐
                    │    Load Balancer     │
                    │   (Nginx / Envoy)   │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
    ┌──────▼──────┐    ┌───────▼──────┐    ┌───────▼──────┐
    │  Portal UI  │    │  API Gateway │    │  WebSocket   │
    │  (Static)   │    │  (Go)        │    │  Gateway     │
    └─────────────┘    └──────┬───────┘    └──────┬───────┘
                              │                   │
                    ┌─────────▼───────────────────▼──────────┐
                    │           Service Mesh (Linkerd)        │
                    ├────────────────────────────────────────┤
                    │                                        │
        ┌───────────▼────┐  ┌─────────────┐  ┌──────────────▼─┐
        │  Orchestration │  │  Script      │  │  Self-Heal     │
        │  Service       │  │  Runtime     │  │  Service       │
        │                │  │  Service     │  │                │
        └────────────────┘  └─────────────┘  └────────────────┘
        ┌────────────────┐  ┌─────────────┐  ┌────────────────┐
        │  Adapter        │  │  Identity   │  │  Governance    │
        │  Service        │  │  Service    │  │  Service       │
        └────────────────┘  └─────────────┘  └────────────────┘
                    │                                        │
                    ├────────────────────────────────────────┤
                    │           Data Layer                    │
                    │  PostgreSQL │ Redis │ MinIO │ NATS      │
                    └────────────────────────────────────────┘
```

**Deployment Modes:**
- **Self-Hosted (Kubernetes)**: Helm chart, all components in customer VPC
- **Self-Hosted (Docker Compose)**: Single-node for dev/small teams
- **Managed Cloud**: SaaS offering (future phase)

---

## 4. Agent-Driven Build System

### 4.1 Overview

Forge will be partially built by its own agent system. Three specialized AI agents — **PM Agent**, **Builder Agent**, and **Verification Agent** — form a development loop that accelerates platform construction.

```
┌─────────────────────────────────────────────────────────┐
│                    BUILD LOOP                            │
│                                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────────────┐  │
│   │ PM Agent │───▶│ Builder  │───▶│  Verification    │  │
│   │          │    │ Agent    │    │  Agent           │  │
│   │          │◀───│          │◀───│                  │  │
│   └──────────┘    └──────────┘    └──────────────────┘  │
│       │                                     │           │
│       └──────────── Feedback Loop ──────────┘           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 4.2 PM Agent (Project Manager)

**Role:** Breaks down the platform spec into actionable developer tasks, manages priorities, tracks dependencies, and ensures alignment with the architecture.

**System Prompt:**
```
You are a senior technical project manager for the Forge platform. 
Your responsibilities:
1. Decompose architecture spec sections into implementable user stories
2. Define acceptance criteria for each story
3. Identify dependencies between stories and sequence them
4. Assign priority (P0-P3) and size estimates (S/M/L/XL)
5. Track completion status and flag blockers
6. Generate sprint plans and progress reports
7. Ensure all stories trace back to spec requirements
```

**Tools Available:**
| Tool | Purpose |
|---|---|
| `spec_reader` | Parse and query the architecture spec document |
| `story_creator` | Create structured user stories in project tracker |
| `dependency_graph` | Build and query task dependency DAG |
| `progress_tracker` | Query completion status across all stories |
| `developer_roster` | Check developer skills and availability |
| `risk_register` | Log and track project risks |

**Output Artifacts:**
- Product backlog with prioritized user stories
- Sprint plans (2-week cycles)
- Dependency graph visualization
- Weekly status reports
- Risk register with mitigation strategies

**Example Story Generated by PM Agent:**
```yaml
story:
  id: FORGE-042
  title: "Implement Script Registry CRUD API"
  description: |
    Build the REST API endpoints for the Script Registry component.
    Must support create, read, update, delete, and state transitions
    for ScriptRecord entities as defined in spec section 3.3.1.
  acceptance_criteria:
    - POST /api/v1/scripts creates a ScriptRecord
    - GET /api/v1/scripts/{id} returns full record with version history
    - PUT /api/v1/scripts/{id}/promote transitions state with approval check
    - All endpoints require authentication via JWT
    - Script source code is stored with SHA-256 hash
    - Audit trail entry created for every mutation
    - OpenAPI spec generated and validated
    - Integration tests cover happy path + error cases
  priority: P0
  size: L
  depends_on: [FORGE-001, FORGE-015]  # DB schema, Auth service
  assigned_to: "backend-team"
  sprint: 2
  spec_reference: "Section 3.3.1 Script Registry"
```

### 4.3 Builder Agent

**Role:** Takes user stories from the PM Agent and generates implementation code, tests, documentation, and infrastructure definitions.

**System Prompt:**
```
You are a senior full-stack developer building the Forge platform.
Your responsibilities:
1. Read the user story and acceptance criteria
2. Plan the implementation approach
3. Generate production-quality code following project conventions
4. Write comprehensive tests (unit, integration, e2e as appropriate)
5. Generate API documentation (OpenAPI for REST, protobuf for gRPC)
6. Create database migrations if schema changes are needed
7. Update Dockerfile/Helm charts if infrastructure changes needed
8. Submit implementation as a structured changeset for review

Follow these conventions:
- Python: Black formatting, type hints, pytest
- TypeScript: ESLint + Prettier, strict mode
- Go: gofmt, standard project layout
- All code must pass linting before submission
- Security: input validation, parameterized queries, no secrets in code
```

**Tools Available:**
| Tool | Purpose |
|---|---|
| `code_generator` | Generate code files with language-specific templates |
| `test_generator` | Generate test files matching implementation |
| `schema_migrator` | Generate Alembic/Flyway database migrations |
| `api_doc_generator` | Generate OpenAPI/protobuf definitions |
| `file_writer` | Write generated files to the repo |
| `linter` | Run language-specific linters |
| `dependency_resolver` | Check and add required package dependencies |
| `git_operations` | Create branches, commits, PRs |

**Output Artifacts:**
- Source code files
- Test files
- Database migration scripts
- API documentation
- Dockerfile / Helm chart updates
- Pull request with structured description

**Builder Agent Workflow:**
```
1. Receive story from PM Agent
2. Read spec reference section for full context
3. Check existing codebase for patterns and conventions
4. Plan implementation (file list, approach)
5. Generate code iteratively:
   a. Write main implementation
   b. Write tests
   c. Run linter → fix issues
   d. Run tests locally → fix failures
6. Generate documentation
7. Create PR with:
   - Story ID reference
   - Implementation summary
   - Test results
   - Spec compliance checklist
8. Hand off to Verification Agent
```

### 4.4 Verification Agent

**Role:** Reviews all Builder Agent output for correctness, security, spec compliance, and quality.

**System Prompt:**
```
You are a senior staff engineer responsible for code review and 
quality assurance of the Forge platform. You are the last gate 
before code merges.

Your responsibilities:
1. Verify code correctness against acceptance criteria
2. Run the full test suite and analyze results
3. Security review: check for OWASP Top 10, injection, auth issues
4. Performance review: check for N+1 queries, unbounded loops, memory leaks
5. Spec compliance: verify implementation matches architecture spec
6. Code quality: naming, structure, error handling, logging
7. Test coverage: ensure all acceptance criteria have corresponding tests
8. Generate a structured review report with pass/fail/warn verdicts

You MUST flag and BLOCK:
- Any hardcoded secrets or credentials
- SQL injection or command injection vectors
- Missing authentication/authorization checks
- Deviations from the architecture spec without documented rationale
- Test coverage below 80% for new code
```

**Tools Available:**
| Tool | Purpose |
|---|---|
| `code_reader` | Read and analyze source files |
| `test_runner` | Execute test suite, return results + coverage |
| `security_scanner` | Run SAST tools (Bandit for Python, ESLint security for TS) |
| `spec_compliance_checker` | Compare implementation against spec section |
| `performance_profiler` | Run benchmarks for critical paths |
| `review_reporter` | Generate structured review report |
| `pr_commenter` | Post review comments on the PR |
| `approval_gate` | Approve or request changes on PR |

**Review Report Structure:**
```yaml
review:
  story_id: FORGE-042
  pr_id: 127
  reviewer: verification-agent-v1
  overall_verdict: PASS | FAIL | PASS_WITH_WARNINGS
  
  checks:
    correctness:
      status: PASS
      notes: "All 12 acceptance criteria verified"
      
    tests:
      status: PASS
      coverage: 87.3%
      tests_run: 34
      tests_passed: 34
      tests_failed: 0
      
    security:
      status: PASS_WITH_WARNINGS
      findings:
        - severity: LOW
          file: "api/scripts.py:142"
          issue: "Consider rate limiting on script creation endpoint"
          recommendation: "Add @rate_limit(10, per_minute) decorator"
          
    spec_compliance:
      status: PASS
      spec_section: "3.3.1"
      deviations: []
      
    performance:
      status: PASS
      benchmarks:
        - endpoint: "GET /api/v1/scripts/{id}"
          p99_latency: "12ms"
          verdict: PASS
          
    code_quality:
      status: PASS
      issues: []
      
  decision: APPROVE
  conditions: 
    - "Address rate limiting warning before production deployment"
```

### 4.5 Agent Collaboration Protocol

The three agents communicate via a structured message protocol:

```
Message:
  id: UUID
  from_agent: enum [pm, builder, verifier]
  to_agent: enum [pm, builder, verifier]
  type: enum [task_assignment, implementation_ready, review_complete, 
              revision_requested, blocked, completed, escalate_to_human]
  payload: TaskPayload | ReviewPayload | StatusPayload
  trace_id: UUID  # For observability
  timestamp: ISO8601

TaskPayload:
  story: UserStory
  context: string[]        # Relevant spec sections, existing code references
  constraints: string[]    # "Must use existing auth middleware", etc.
  deadline: ISO8601 | null

ReviewPayload:
  pr_id: string
  review: ReviewReport
  action: enum [approved, changes_requested, blocked]

StatusPayload:
  story_id: string
  status: enum [in_progress, blocked, complete, failed]
  notes: string
```

**Escalation Rules:**
1. Builder Agent fails to generate passing code after 3 attempts → Escalate to human developer with context
2. Verification Agent finds CRITICAL security issue → Block PR + notify security team
3. PM Agent detects circular dependency → Escalate to human architect
4. Any agent encounters ambiguity in the spec → Request clarification from human

### 4.6 Self-Building Capability

Forge's agents can use the platform itself to build the platform — a bootstrapping loop:

**Phase 1 (Manual Bootstrap):** Human developers build the core runtime, sandbox manager, and basic agent orchestrator.

**Phase 2 (Agent-Assisted Build):** PM/Builder/Verifier agents run on the partial platform to build remaining components:
- Adapters
- Portal UI components  
- Additional orchestration patterns
- Observability integrations
- Documentation

**Phase 3 (Self-Improvement):** Platform agents use the self-heal engine's learning capabilities to:
- Identify common build failures and create automated fixes
- Generate new adapter implementations by analyzing API docs
- Optimize existing scripts based on performance metrics
- Generate tests for uncovered code paths

---

## 5. Development Phases & Milestones

### Phase 0 — Foundation (Weeks 1–4)
| Deliverable | Owner | Size |
|---|---|---|
| Repository structure, CI/CD pipeline | DevOps | M |
| PostgreSQL schema + migration framework | Backend | M |
| Authentication service (JWT + OIDC) | Backend | L |
| API Gateway skeleton (Go) | Backend | M |
| React portal scaffold | Frontend | M |
| Docker Compose dev environment | DevOps | S |

### Phase 1 — Core Runtime (Weeks 5–12)
| Deliverable | Owner | Size |
|---|---|---|
| Script Registry API (CRUD + state machine) | Backend | L |
| Sandbox Manager (Docker-based) | Platform | XL |
| Basic script execution flow (register → sandbox → execute → capture) | Platform | XL |
| Event bus integration (NATS) | Backend | M |
| OpenTelemetry instrumentation | Backend | M |
| Portal: Script editor + execution view | Frontend | L |

### Phase 2 — Intelligence Layer (Weeks 13–20)
| Deliverable | Owner | Size |
|---|---|---|
| Agent Orchestrator (sequential + parallel patterns) | AI | XL |
| Self-Heal Engine v1 (diagnose + propose + gate) | AI | XL |
| LLM integration layer (multi-model: OpenAI, Anthropic, local) | AI | L |
| Self-heal knowledge base (pgvector RAG) | AI | L |
| Portal: Agent configuration + workflow designer | Frontend | XL |
| Portal: Self-heal dashboard | Frontend | L |

### Phase 3 — Integration & Enterprise (Weeks 21–28)
| Deliverable | Owner | Size |
|---|---|---|
| Adapter framework + REST/MCP/Database adapters | Backend | XL |
| RBAC + governance policies engine | Backend | L |
| Secrets management integration | Backend | M |
| Firecracker MicroVM sandbox option | Platform | XL |
| Audit logging + compliance reporting | Backend | L |
| Helm chart for Kubernetes deployment | DevOps | L |

### Phase 4 — Agent Build System (Weeks 25–32, overlaps Phase 3)
| Deliverable | Owner | Size |
|---|---|---|
| PM Agent implementation | AI | L |
| Builder Agent implementation | AI | XL |
| Verification Agent implementation | AI | L |
| Agent collaboration protocol + message bus | AI | M |
| Agent-generated adapter for A2A protocol | AI (dogfood) | M |
| Agent-generated portal components | AI (dogfood) | L |

### Phase 5 — Hardening & GA (Weeks 33–40)
| Deliverable | Owner | Size |
|---|---|---|
| Load testing + performance optimization | QA/Platform | L |
| Security penetration testing | Security | L |
| Documentation (user guide, API docs, admin guide) | All | L |
| Upgrade/migration tooling | DevOps | M |
| GA release packaging | DevOps | M |

---

## 6. Appendices

### 6.1 Glossary

| Term | Definition |
|---|---|
| **Script** | A versioned, sandboxed unit of executable code — the atomic unit of work in Forge |
| **Sandbox** | An isolated execution environment (container or MicroVM) with enforced resource limits |
| **Self-Heal** | AI-driven diagnosis and repair of failed script executions |
| **Heal Pattern** | A stored successful repair that can be reused for similar future failures |
| **Agent** | An AI entity with a defined role, model, tools, and constraints |
| **Orchestration** | The coordination of multiple agents and/or scripts to achieve a goal |
| **Adapter** | A protocol-specific connector that normalizes external system communication |
| **Approval Gate** | A policy-driven checkpoint where human or automated approval is required |
| **Dehydration** | Serializing a long-running workflow to persistent storage (borrowed from BizTalk) |

### 6.2 Design Decisions Log

| Decision | Choice | Rationale | Alternatives Considered |
|---|---|---|---|
| Primary data store | PostgreSQL | Ecosystem (pgvector, TimescaleDB), maturity, ACID | MongoDB (schema flexibility), CockroachDB (distributed) |
| Event bus | NATS JetStream | Lightweight, JetStream for persistence, low ops burden | Kafka (heavier), RabbitMQ (less cloud-native) |
| Sandbox technology | Docker (Phase 1) → Firecracker (Phase 3) | Docker for speed-to-market; Firecracker for security | gVisor (middle ground), Wasm (too immature for full scripts) |
| API Gateway language | Go | Performance, small binary, strong HTTP ecosystem | Rust (steeper hiring), Python (too slow for gateway) |
| Orchestration language | Python | LLM library ecosystem, rapid iteration | TypeScript (viable alternative for full-stack team) |
| Observability | OpenTelemetry | Vendor-neutral, industry standard | Proprietary (LangSmith, Datadog) — avoided lock-in |

### 6.3 Risk Register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Self-heal engine produces harmful repairs | Medium | Critical | Approval gates, test suites, sandbox isolation, confidence thresholds |
| LLM provider outages block self-heal | Medium | High | Multi-model support, local model fallback, graceful degradation |
| Sandbox escape vulnerability | Low | Critical | Firecracker MicroVMs, regular security audits, bug bounty |
| Agent build system produces low-quality code | Medium | Medium | Verification Agent blocks merges, human review gate for P0 components |
| Platform complexity overwhelms small team | High | High | Phase deliverables strictly scoped, MVP-first approach |
| Vendor lock-in via model dependencies | Medium | Medium | Abstraction layer over all LLM calls, support local models |

### 6.4 API Design Conventions

```
Base URL: /api/v1

Authentication: Bearer JWT token
Content-Type: application/json

Pagination: ?page=1&page_size=50
Filtering: ?status=active&language=python
Sorting: ?sort_by=created_at&sort_order=desc

Standard Response Envelope:
{
  "data": { ... },
  "meta": {
    "request_id": "uuid",
    "trace_id": "uuid",
    "pagination": { "page": 1, "page_size": 50, "total": 234 }
  },
  "errors": []
}

HTTP Status Codes:
  200 OK           — Successful read/update
  201 Created      — Successful create
  202 Accepted     — Async operation started
  400 Bad Request  — Validation error
  401 Unauthorized — Missing/invalid auth
  403 Forbidden    — Insufficient permissions
  404 Not Found    — Resource doesn't exist
  409 Conflict     — State transition violation
  429 Too Many     — Rate limited
  500 Server Error — Unexpected failure (with trace_id for debugging)
```

---

*This document is a living specification. All sections should be reviewed and updated as implementation progresses and learnings emerge from the agent build system.*

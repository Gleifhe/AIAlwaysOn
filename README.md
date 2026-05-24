# AIAlwaysOn — Forge Platform

**Self-healing, script-centric AI & automation platform.**

> Most AI automation today is LLMs writing scripts, running them, reading the output, and iterating. Forge makes this pattern a first-class, production-grade, enterprise-secure platform primitive — not a side effect of a chatbot.

## What is this?

Forge is an architecture specification and agent team design for building an AI automation platform that treats **scripts as first-class citizens** with AI-powered generation, execution, observation, and self-repair.

## Documentation

| Document | Description |
|---|---|
| [Platform Architecture Spec](docs/spec/platform-architecture-spec.md) | Full architecture specification including competitive analysis (OpenHands, BizTalk, MAF, Foundry, LangGraph, CrewAI, n8n), component designs, technology stack, and 5-phase delivery plan |
| [Hidden Features Addendum](docs/addendum/hidden-features-addendum.md) | 26 features mined from Azure Durable Functions, AWS Step Functions, Google Cloud Workflows, OCI, Temporal.io, Apache Airflow, AWS Bedrock, and Google Vertex AI |
| [Agent Team Flow](docs/agent-flow/agent-team-flow.md) | Architect → PM → Developer → Security Engineer agent pipeline with 4 operating modes, system prompts, message protocol, and efficiency optimizations |

## Core Concepts

- **Script-as-Workflow** — Every automation is a versioned, auditable script with AI-powered generation and self-repair
- **Self-Healing Runtime** — Failures trigger AI-driven diagnosis and automatic repair with human-in-the-loop guardrails
- **Durable Execution** — Event sourcing, checkpointing, and replay guarantee workflows complete despite crashes
- **Multi-Agent Orchestration** — Native support for agent teams with defined roles, handoffs, and escalation
- **Universal Adapter Layer** — Protocol-native integration (MCP, A2A, REST, gRPC, legacy adapters)
- **Observable by Default** — Every decision, script execution, and self-heal attempt is traced and auditable
- **Agent Guardrails** — Content filters, PII detection, code safety scanning, and budget caps on AI execution

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  Portal UI  │  CLI / SDK  │  API Gateway + Callbacks         │
├──────────────────────────────────────────────────────────────┤
│  ORCHESTRATION: Workflow Engine │ Agent Orchestrator │ Rules  │
├──────────────────────────────────────────────────────────────┤
│  DURABLE EXECUTION: Event Store │ Checkpoints │ Task Context │
├──────────────────────────────────────────────────────────────┤
│  RUNTIME: Script Registry │ Sandbox Manager │ Self-Heal      │
├──────────────────────────────────────────────────────────────┤
│  INTEGRATION: MCP │ A2A │ REST │ gRPC │ Legacy Adapters      │
├──────────────────────────────────────────────────────────────┤
│  PLATFORM: Identity │ Observability │ Secrets │ Governance    │
├──────────────────────────────────────────────────────────────┤
│  DATA: PostgreSQL │ Redis │ S3-Compatible │ Vector DB         │
└──────────────────────────────────────────────────────────────┘
```

## Agent Build Team

The platform is designed to be partially built by its own agent system:

```
ARCHITECT ████░░░░░░░░░░░░░░░░░░████  (design first, review last)
SECURITY  ░░██░░░░░░░░░░░░░░░░██░░░░  (threat model early, scan late)
PM        ░░░░████░░░░░░░░████░░░░░░  (stories after design, tracking during build)
DEVELOPER ░░░░░░░░████████████░░░░░░  (heads-down implementation)
```

## Competitive Landscape

Analyzed 7+ platforms to identify gaps:

| Platform | Gap Forge Fills |
|---|---|
| OpenHands | Great at code self-heal; no business automation |
| MS BizTalk | Dead, but adapter/dehydration patterns adopted |
| MS Agent Framework | No self-healing beyond retry middleware |
| LangGraph | No deployment story, no self-heal |
| CrewAI | Less granular control, no durable execution |
| n8n | AI bolted on, not native. No self-heal. |
| AWS Step Functions | No AI-native execution, no script generation |

## License

TBD

## Status

**Pre-development** — Architecture specification phase. See the [delivery plan](docs/spec/platform-architecture-spec.md#5-development-phases--milestones) for the 5-phase roadmap.

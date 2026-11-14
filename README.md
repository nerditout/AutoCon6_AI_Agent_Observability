# AI Agent Observability — Round Table for Network Engineers

A 60-minute lunch round table on observing AI agents: in the enterprise, on the network, and running locally on laptops and dev boxes. You can't improve, control, or analyze what you cant see.

## Why are AI Agents Like Teenagers
AI Agents are like a teenager: it needs constant supervision, and the second you leave the room, it does something you’ll have to explain to someone later.

**Framing:** AI agents are a new kind of network endpoint. They generate traffic, make API calls nobody predicted, and sometimes change infrastructure. Network engineers already know how to observe distributed systems; this session applies that thinking to agents.

---

## Agenda

| Time  | Segment |
|-------|---------|
| 0–5   | Intros: each person says whether they **run** agents, **block** them, or **don't know** |
| 5–15  | [What does "observing an agent" even mean?](#1-what-observability-means-for-agents) |
| 15–30 | [Agents in the enterprise: traffic, identity, shadow AI](#2-agents-in-the-enterprise) |
| 30–42 | [Open source tooling](#3-open-source-tooling) |
| 42–52 | [Local agents on laptops and dev boxes](#4-observing-local-agents) |
| 52–60 | Takeaways: one thing each person will try next week |

---

## 1. What observability means for agents

- **Traces are like traceroute.** A single agent request fans out into LLM calls, tool calls, retries and sub-agents. Each step is a span, much like each hop in a path. The [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) are the emerging standard for recording them.
- **Golden signals for agents.** Latency, error rate, **token usage and cost**, tool-call success rate, and loops (agents retrying endlessly).
- **Understand non-deterministic behavior:** Because an agent's reasoning process isn't deterministic, telemetry is the only reliable way to inspect the decisions an agent makes and the tools it selects.
- **Troubleshoot and diagnose errors:** Traces help you identify failed API requests, infinite execution loops, and latency bottlenecks across distributed agent workflows.
- **Support governance and validate traffic:** Telemetry lets you visualize communication flows—such as in Application Topology—to verify that your agents communicate only with authorized agents, MCP   servers, and external endpoints.
- **Evaluate output quality and cost:** Collecting prompts, responses, and token usage helps you assess model accuracy, detect hallucinations, and optimize resource consumption.

## 2. Agents in the enterprise

- **Shadow AI detection.** Find unsanctioned LLM traffic through DNS logs, TLS SNI, proxy logs and NetFlow (e.g. `api.openai.com`, `api.anthropic.com`).
- **AI gateways as the choke point.** Route all LLM traffic through one proxy for logging, rate limits, key management and cost attribution. A familiar pattern: a policy enforcement point.
- **MCP traffic.** Agents call tools over the [Model Context Protocol](https://modelcontextprotocol.io). Who can see those calls? Should MCP servers sit in a DMZ?
- **Agent identity.** When an agent makes a change, whose credentials did it use? Service accounts vs. user delegation, and audit trails.
- **Agents touching the network.** If an agent can push config through NETCONF, Ansible or vendor APIs, what guardrails and change logs do you need?
- **Data leakage.** What's in the prompts leaving your network? DLP for prompts.

## 3. Open source tooling

### Instrumentation and standards

| Tool | What it does | Links |
|------|--------------|-------|
| **OpenTelemetry** | Vendor-neutral standard for traces, metrics and logs; includes GenAI semantic conventions | [opentelemetry.io](https://opentelemetry.io) · [GenAI conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) |
| **OpenLLMetry** (Traceloop) | OTel-based auto-instrumentation for LLM providers and agent frameworks | [GitHub](https://github.com/traceloop/openllmetry) |
| **OpenLIT** | OTel-native auto-instrumentation plus a self-hosted dashboard for LLMs, GPUs and vector DBs | [openlit.io](https://openlit.io) · [GitHub](https://github.com/openlit/openlit) |

### Langfuse (featured)

[Langfuse](https://langfuse.com) is the most widely adopted open source LLM engineering platform and a good starting point for the room.

- **Tracing:** nested traces of every LLM call, tool call and agent step, with inputs, outputs, latency and errors
- **Cost and token tracking:** per user, per session, per model
- **Evals and scoring:** LLM-as-judge, user feedback and manual annotation on traces
- **Prompt management:** versioned prompts linked to the traces that used them
- **OpenTelemetry support:** accepts OTel traces, so it fits alongside existing OTel pipelines
- **Integrations:** OpenAI, Anthropic, LangChain/LangGraph, LlamaIndex, LiteLLM and more
- **Self-hostable:** MIT-licensed core, runs with Docker Compose or on Kubernetes

Quick local start:

```bash
git clone https://github.com/langfuse/langfuse.git
cd langfuse
docker compose up
# then open http://localhost:3000
```

Links: [langfuse.com](https://langfuse.com) · [GitHub](https://github.com/langfuse/langfuse) · [Docs](https://langfuse.com/docs) · [Self-hosting guide](https://langfuse.com/self-hosting)

### Other LLM / agent tracing platforms (self-hostable)

| Tool | What it does | Links |
|------|--------------|-------|
| **Arize Phoenix** | OTel-native tracing and evals, strong for debugging agent runs | [phoenix.arize.com](https://phoenix.arize.com) · [GitHub](https://github.com/Arize-ai/phoenix) |
| **Comet Opik** | Tracing, evals and production monitoring for LLM apps | [GitHub](https://github.com/comet-ml/opik) |
| **MLflow Tracing** | Tracing for GenAI apps inside the MLflow ecosystem | [mlflow.org](https://mlflow.org) · [GitHub](https://github.com/mlflow/mlflow) |

### Gateways and proxies

| Tool | What it does | Links |
|------|--------------|-------|
| **LiteLLM Proxy** | One OpenAI-compatible endpoint in front of 100+ model providers, with logging, budgets and spend tracking | [docs.litellm.ai](https://docs.litellm.ai) · [GitHub](https://github.com/BerriAI/litellm) |
| **Envoy AI Gateway** | AI traffic management built on Envoy Gateway | [aigateway.envoyproxy.io](https://aigateway.envoyproxy.io) · [GitHub](https://github.com/envoyproxy/ai-gateway) |
| **Kong AI Gateway** | AI plugins (multi-LLM routing, prompt guards, rate limiting) for Kong Gateway | [konghq.com](https://konghq.com/products/kong-ai-gateway) · [GitHub](https://github.com/Kong/kong) |

### Observability backends you probably already have

| Tool | What it does | Links |
|------|--------------|-------|
| **Grafana Tempo** | Distributed tracing backend | [grafana.com/oss/tempo](https://grafana.com/oss/tempo/) |
| **Grafana Loki** | Log aggregation | [grafana.com/oss/loki](https://grafana.com/oss/loki/) |
| **Prometheus** | Metrics and alerting | [prometheus.io](https://prometheus.io) |
| **Jaeger** | Distributed tracing | [jaegertracing.io](https://www.jaegertracing.io) |
| **SigNoz** | OTel-native all-in-one: traces, metrics, logs | [signoz.io](https://signoz.io) · [GitHub](https://github.com/SigNoz/signoz) |

### Network and kernel-level visibility

| Tool | What it does | Links |
|------|--------------|-------|
| **Cilium / Hubble** | eBPF networking with flow-level visibility for Kubernetes | [cilium.io](https://cilium.io) · [Hubble GitHub](https://github.com/cilium/hubble) |
| **Tetragon** | eBPF security observability: process execution, file and network events | [tetragon.io](https://tetragon.io) |
| **Pixie** | eBPF auto-telemetry for Kubernetes, no code changes | [px.dev](https://px.dev) |
| **Zeek** | Network traffic analysis; can fingerprint LLM API traffic | [zeek.org](https://zeek.org) |
| **Suricata** | IDS/IPS and network security monitoring | [suricata.io](https://suricata.io) |

## 4. Observing local agents

- Agents on laptops (coding assistants, local models) mostly bypass your gateway. They are the blind spot.
- **Some tools export telemetry natively.** For example, [Claude Code can export OpenTelemetry metrics and events](https://docs.anthropic.com/en/docs/claude-code/monitoring-usage) to your collector. Which of the room's tools can do this?
- **Local models** such as [Ollama](https://ollama.com) (open source) and [LM Studio](https://lmstudio.ai) (free, not open source) generate no egress traffic, but the agent still calls tools and touches files and the network. Endpoint and eBPF visibility matter more than network visibility here.
- **Demo idea:** a local agent sending traces to Langfuse or Phoenix running in Docker (see the quick start above).

## Discussion questions

1. Would you know today if someone in your org was running an autonomous agent against production?
2. Should AI gateways be owned by the network team, the security team or the platform team?
3. Is an agent's tool call more like a user action or an API call for audit purposes?
4. What's the agent equivalent of a NetFlow record: the minimum we should log for every action?
5. Would you let an agent make network changes? Under what observability conditions?

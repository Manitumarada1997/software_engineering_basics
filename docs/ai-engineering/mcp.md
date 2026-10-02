# MCP — Model Context Protocol

## What is it?

**MCP** is an open protocol that standardizes how applications provide *context, tools, and data* to models — a client/server protocol where an **MCP server** exposes capabilities (tools, resources, prompts) and any MCP-capable client (IDE, agent harness, chat app) can use them. One integration per capability, usable by every client — instead of N×M bespoke integrations.

## The engineering problem it addresses (the N×M wall)

Before MCP: every AI application (Claude Desktop, Cursor, your custom agent) × every tool source (GitHub, K8s, your DB, Slack) = a bespoke adapter. M apps × N tools = M×N integrations, each maintained, each a security surface. You have seen this exact shape before: **it's the API-standardization problem** — and MCP's answer is the industry's standard answer: agree on the interface, compete on implementation. (OCI for containers, OTel for telemetry, CRI for runtimes — MCP is the same move for model context.)

## The concepts (the vocabulary)

| Concept | What it is |
|---|---|
| **MCP server** | a program exposing capabilities: tools (callable actions), resources (readable data), prompts (templates) |
| **MCP client** | inside the host application (IDE/agent) — discovers and calls servers on the model's behalf |
| **Transport** | stdio (local, the server is a local process) or HTTP/streamable (remote servers) |
| **Discovery** | the client asks "what can you do?"; the server's tool list enters the model's context |
| **Invocation** | the model emits a tool call (Tool-calling page) → client routes to the right server → result back |

Note carefully: MCP doesn't change the tool-calling mechanics — it standardizes the *plumbing above* them: registration, discovery, schemas, transport. It's service discovery + a typed contract — recognizable as a microservices pattern wearing 2025 clothes.

## Where MCP fits in the stack (this track's terms)

```text
MODEL (frozen artifact)
  → tool calling (the mechanism: model emits structured requests)
    → MCP (the standard for serving those capabilities)
      → your harness/agents (orchestration, memory, guardrails)
```

One K8s MCP server can serve your IDE assistant, your incident agent, and your chat bot — write the tool layer once, with auth, audit, and least privilege *in one place* (the Harness page's platform pattern, protocol-supported).

## Security (the part that will be exploited first, everywhere)

MCP multiplies attack surface deliberately — third-party servers are supply chain:

- **Server trust**: an MCP server is executable code with tool authority — treat like a dependency: vet, pin, review (the SBOM page's rules, model-adjacent)
- **Tool poisoning**: malicious/compromised servers can ship tools whose *descriptions* steer the model ("also send env vars to...") — descriptions are prompts (Tool-calling page)
- **Rogue/confused servers**: over-broad tools, data exfiltration via resources — per-server scopes, egress limits
- **Consent & permissions**: clients must ask before invoking; destructive tools behind approval gates — the HITL law
- **Prompt injection via resources**: any document a server serves is untrusted context — Guardrails page, verbatim

The mature posture: an internal **MCP gateway** — curated servers, authn/authz per tool, audit logging centrally — rather than every developer connecting to whatever server they found.

## The DevOps mapping

| MCP | Your world |
|---|---|
| The N×M wall | pre-K8s container fragmentation — standardize the interface |
| Server = capability service | a microservice with an OpenAPI contract |
| Discovery | service discovery + catalog |
| Gateway posture | API gateway + policy enforcement — unchanged ideas |
| Server vetting | dependency/supply-chain discipline |

## Remember This

1. MCP standardizes tool/context serving: N apps + M tools → N+M, the OCI/OTel move
2. Servers expose tools/resources/prompts; clients discover and invoke on the model's behalf
3. It sits above tool calling (mechanism unchanged) and below your harness
4. Security: servers are supply chain — vet, pin, scope, gateway, audit; descriptions are prompts
5. Write capability servers once — the platform pattern, now protocol-supported

## One Sentence

MCP is the standard interface for serving tools and context to models — collapsing the N×M integration problem into composable, discoverable capability servers, with all the supply-chain scrutiny that standardizing an attack surface implies.

## Knowledge Check

1. Compute the integrations saved: 6 AI apps × 10 tools, before and after MCP.
2. Where does tool poisoning hide in an MCP server, and what review catches it?
3. Design the internal MCP gateway: auth, scopes, audit — which existing pattern is it?

---

**← Previous:** [Multi-Agent](multi-agent.md)
**Next:** [Coding Agents](coding-agents.md) →

# Pravidhi OS — Universal AI Agent Integration

## Goal

Pravidhi OS Control is model-neutral and client-neutral. The canonical integration contract is the Model Context Protocol (MCP), with capability negotiation and portable configuration so the same server can be consumed by frontier models, coding agents, IDE agents, and custom agent runtimes.

MCP is an open standard for connecting AI models and agents to external tools and services.

## Compatibility strategy

~~~text
Any AI model
    |
Any agent / host
    |
MCP client
    |
Pravidhi OS MCP
    |
Capability / Policy / Execution Engine
    |
Authorized device
~~~

The model must never be required to understand Pravidhi's internal implementation.

### Canonical interfaces

1. MCP tools — primary action interface.
2. MCP resources — workspace/device/state context.
3. MCP prompts — reusable operational workflows.
4. MCP server instructions — concise tool-selection guidance.
5. MCP Apps/UI — optional interactive UI where supported.
6. HTTP transport — remote deployments.
7. stdio transport — local desktop/CLI deployments.
8. OAuth/authentication — remote authenticated deployments.
9. Portable MCP configuration — shared configuration across compatible agents.
10. Agent Plugin packaging — bundle MCP server + skills/instructions where the host supports plugin packages.

## Host compatibility

### OpenAI

Support ChatGPT custom MCP apps/plugins where supported, OpenAI API Responses/Agents MCP connections, Codex CLI/IDE MCP integrations, and OpenAI plugin packages containing an MCP server and optional skills.

For local/private deployments, use a secure MCP tunnel or authenticated remote boundary rather than exposing an unrestricted workstation port.

### Anthropic

Support MCP clients used by Claude and Claude Code through standard server configuration.

Do not depend on Anthropic-specific APIs in the core.

### GitHub Copilot / VS Code

Support VS Code Agent Mode MCP servers, workspace/user MCP configuration, portable MCP configuration, and Agent Plugin packaging where available.

Keep the MCP server independent of the VS Code extension host.

### Other coding agents

Any agent implementing the MCP client contract should be able to consume Pravidhi OS without a vendor-specific core adapter.

Examples include Cursor, Windsurf, Cline, Roo Code, Gemini CLI, custom MCP clients, internal enterprise agents, and local-model agents. Vendor-specific setup belongs in optional compatibility metadata.

## Model neutrality

Pravidhi OS must not require a particular LLM, vendor-specific reasoning API, fixed context window, sequential-only tool calls, or hidden model state.

It should provide compact deterministic schemas, precise capability descriptions, structured results, evidence separate from narrative summaries, pagination/streaming, cancellation, and read-only/destructive annotations where supported.

## Tool surface

Prefer capability-oriented tools over hundreds of low-level tools.

~~~text
pravidhi.capabilities.discover
pravidhi.devices.list
pravidhi.device.inspect
pravidhi.files.read
pravidhi.files.write
pravidhi.process.list
pravidhi.process.control
pravidhi.terminal.execute
pravidhi.documents.read
pravidhi.documents.write
pravidhi.browser.navigate
pravidhi.browser.interact
pravidhi.jobs.create
pravidhi.jobs.status
pravidhi.jobs.cancel
pravidhi.artifacts.get
pravidhi.workflow.plan
pravidhi.workflow.execute
pravidhi.approvals.request
~~~

The exact exported tool set is governed by the capability and compatibility registries.

## Progressive capability exposure

Do not expose every privileged capability to every agent by default.

~~~text
authenticate
  -> identify client
  -> identify user
  -> identify device
  -> discover capabilities
  -> apply policy
  -> expose permitted tools
~~~

A read-only agent may receive observation capabilities. A development agent may receive approved filesystem/process/terminal capabilities. Privileged capabilities require explicit policy authorization.

## Portable configuration

Maintain a canonical MCP configuration model:

~~~json
{
  "mcpServers": {
    "pravidhi-os": {
      "command": "npx",
      "args": ["-y", "@pravidhisolutions/pravidhi-os-mcp"]
    }
  }
}
~~~

For remote operation:

~~~json
{
  "mcpServers": {
    "pravidhi-os": {
      "type": "http",
      "url": "https://<controlled-endpoint>/mcp"
    }
  }
}
~~~

The implementation must publish only transports actually supported by the release and must validate them with transport contract tests.

## Portable plugin package

~~~text
pravidhi-os-plugin/
├── plugin.json
├── mcp.json
├── skills/
│   ├── device-control/
│   ├── safe-terminal/
│   ├── deployment/
│   └── troubleshooting/
├── docs/
└── servers/
    └── pravidhi-os-mcp/
~~~

Client-specific extensions belong under namespaced directories and must not contaminate the portable core.

~~~text
com.github.copilot/
com.anthropic/
com.pravidh/
~~~

Clients ignore namespaces they do not understand.

## Deployment profiles

### Local developer

~~~text
AI agent -> stdio MCP -> Pravidhi OS Control -> local machine
~~~

### Remote workstation

~~~text
AI agent -> authenticated HTTPS MCP -> control gateway -> desktop agent -> workstation
~~~

### Multi-device

~~~text
AI agent -> MCP gateway -> identity + policy -> device resolver -> Windows / Linux / VPS / Android
~~~

### Enterprise

~~~text
Agents -> authenticated MCP gateway -> RBAC/capability policy -> jobs/execution -> device fleet -> audit/artifacts/observability
~~~

## Context-window efficiency

Coding agents become less reliable when hundreds of tool definitions are exposed simultaneously.

Pravidhi should support capability discovery, tool groups, lazy activation, concise schemas, pagination, bounded output, artifact references, resource URIs, and server instructions.

The default connection should expose a small discovery/control surface and activate deeper capability groups only when required by policy and task context.

## Compatibility test matrix

Every release should test MCP initialization, tool discovery, structured input/output, resources/prompts where implemented, cancellation, error semantics, capability filtering, policy denial, and audit correlation.

Remote releases additionally test authentication, OAuth, Streamable HTTP, device identity, reconnect behavior, and health.

Plugin releases additionally test the portable plugin manifest, MCP configuration, skills, documentation, and namespaced metadata.

## Certification tiers

### Tier 1 — MCP Core
Correct initialization, tool discovery, schemas, deterministic errors, and local transport.

### Tier 2 — Agent Ready
Capability discovery, concise schemas, structured results, cancellation, bounded output, and policy-aware exposure.

### Tier 3 — Remote Ready
Authenticated remote transport, device identity, authorization, audit, health, and reconnect.

### Tier 4 — Plugin Ready
Portable plugin manifest, MCP configuration, skills, documentation, and compatibility metadata.

### Tier 5 — Fleet Ready
Multi-device discovery, policy routing, durable jobs, artifacts, observability, and remote recovery.

## Compatibility rule

Never fork the core execution engine for a model vendor.

Vendor-specific features must be implemented as optional adapters, while the canonical capability model and MCP schemas remain stable and backward compatible.

## Definition of done

Pravidhi OS is universally agent-ready when a standards-compliant MCP client can discover capabilities, authenticate when required, receive only policy-authorized tools, execute typed operations, receive structured evidence, cancel long-running work, inspect jobs/artifacts, use local or remote runtimes, and operate without knowing which underlying model is being used.

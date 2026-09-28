# Pravidhi OS — Capability & Integration Intelligence

## Purpose

The Pravidhi OS Control Tool should evolve from a collection of MCP tools into an intelligent capability fabric. It must understand what a connected machine or integration can do, select valid capabilities, compose multi-step workflows, verify results, and recover from failures without bypassing policy.

## 1. Capability Fabric

Every integration exposes a typed capability descriptor:

```yaml
id: filesystem.write
provider: windows-local
version: 1
risk: MUTATE
scopes: [workspace, approved_paths]
requires: [authenticated_session]
supports: [streaming, cancellation, dry_run]
limits:
  max_bytes: 104857600
  timeout_ms: 30000
verification:
  strategy: checksum_or_stat
```

Each capability declares:
- input/output schema
- risk class
- required permissions
- supported platforms
- scopes
- resource limits
- approval requirements
- idempotency behavior
- cancellation behavior
- verification strategy
- audit requirements

Capability classes:
- **Observe:** filesystem.read, process.list, system.info, screen.read
- **Mutate:** filesystem.write, file.move, service.configure, document.write
- **Execute:** terminal.execute, process.start, script.run, workflow.run
- **Control:** process.signal, service.restart, device.input, browser.interact
- **Privileged:** elevation.request, firewall.manage, package.manage
- **Orchestration:** workflow.execute, job.schedule, event.subscribe

## 2. Integration Intelligence

Introduce an Integration Resolver between intent and execution:

```
User/Agent Intent
      |
Intent Normalizer
      |
Capability Discovery
      |
Integration Resolver
      |
Policy Preflight
      |
Plan
      |
Execution Graph
      |
Verification
      |
Result + Evidence
```

The resolver determines:
1. Which integrations are connected.
2. Which capabilities they expose.
3. Which capabilities can satisfy the requested outcome.
4. Required permissions and scopes.
5. Whether the task can execute locally.
6. Whether another authorized device is required.
7. Whether existing state/artifacts can be reused.
8. The safest valid execution path.
9. How success will be verified.

Integration selection must use declared capabilities, health, scope, policy, latency, locality, resource availability, and protocol compatibility—not tool names alone.

## 3. Device Intelligence

Maintain a live device capability graph:

```
Pravidhi OS
  ├── Windows workstation
  │    ├── PowerShell
  │    ├── filesystem
  │    ├── processes
  │    ├── Docker
  │    └── browser
  ├── Android / Termux
  │    ├── Termux
  │    ├── screen
  │    ├── input
  │    └── optional approved elevation
  └── VPS
       ├── SSH
       ├── Docker
       ├── systemd
       ├── filesystem
       └── services
```

Device state includes identity, trust state, health, connectivity, platform, agent version, capabilities, resource pressure, policy profile, heartbeat, and current jobs.

The planner can choose a suitable runtime without requiring the user to know implementation details.

## 4. Intent-to-Execution Planning

Natural language should become a typed Execution Plan rather than a shell command:

```json
{
  "goal": "deploy the application",
  "steps": [
    {"capability": "filesystem.read", "operation": "inspect_workspace"},
    {"capability": "process.list", "operation": "check_existing_service"},
    {"capability": "terminal.execute", "operation": "run_tests"},
    {"capability": "terminal.execute", "operation": "build"},
    {"capability": "service.manage", "operation": "restart"},
    {"capability": "network.request", "operation": "health_check"}
  ],
  "verification": [
    "build_exit_code == 0",
    "service_healthy == true",
    "health_endpoint == 200"
  ]
}
```

Plans must be inspectable, policy-checked, resumable, cancellable, idempotent where possible, partially retryable, and auditable.

## 5. Execution Graph

Support a DAG-capable execution engine with:
- dependency edges
- conditional edges
- bounded retries
- timeouts
- compensation/rollback
- parallel execution
- approval gates
- verification gates

Example:

```
inspect
  |
  +----> tests ----+
  |                |
  +----> config ---+--> build --> deploy --> health-check
                                       |
                                  failure?
                                       |
                                    rollback
```

Parallel execution is permitted only when dependencies and resource limits allow it.

## 6. Verification-First Intelligence

A successful process exit is not equivalent to task success.

Examples:
- file write → verify path, size, and optionally hash
- deployment → verify service and endpoint
- package install → verify effective version
- process start → verify PID and health
- configuration change → validate effective configuration
- browser workflow → verify resulting page/state
- document edit → reopen and validate content
- remote command → verify target identity and resulting state

Return evidence, not merely a success string.

## 7. Recovery Intelligence

Classify failures:

```
TRANSIENT
AUTHORIZATION
VALIDATION
RESOURCE
DEPENDENCY
REMOTE_CONNECTIVITY
EXECUTION
VERIFICATION
POLICY
UNKNOWN
```

Recovery:
- transient → bounded retry
- connectivity → reconnect or approved alternate runtime
- dependency → resolve prerequisite
- validation → revise inputs
- resource → reduce workload or request capacity
- verification failure → inspect actual state
- policy denial → stop; never bypass policy
- unknown → preserve evidence and require investigation

Autonomous policy bypass is never a recovery strategy.

## 8. Context & State Intelligence

Separate ephemeral execution context from durable state.

Ephemeral:
- current task
- plan
- tool results
- process IDs
- temporary files
- session state

Durable:
- devices
- capabilities
- preferences
- policies
- jobs
- artifacts
- audit history
- integration configuration
- workflow definitions

Use a state-provider interface so SQLite serves local mode while PostgreSQL serves fleet mode.

## 9. Integration Adapter Contract

Every adapter should expose:

```text
discover()
health()
capabilities()
authorize()
execute()
stream()
cancel()
verify()
disconnect()
```

Initial adapter families:

| Integration | Primary capabilities |
|---|---|
| Windows | PowerShell, filesystem, processes, services |
| Linux | shell, filesystem, systemd, containers |
| macOS | shell, filesystem, processes |
| Android | screen, input, Termux, device state |
| VPS/SSH | remote command, files, services |
| Docker | containers, logs, lifecycle |
| GitHub | repositories, branches, PRs, CI |
| Browser | navigation, extraction, interaction |
| Documents | read/write/transform |
| Secrets | scoped secret retrieval |
| Cloud APIs | provider-specific operations |

Adapters never silently elevate permissions.

## 10. MCP Compatibility Layer

MCP remains the external compatibility surface:

```
MCP Tool
   ↓
Compatibility Mapper
   ↓
Capability Request
   ↓
Policy
   ↓
Execution Engine
```

A compatibility registry maps legacy tool names to canonical capabilities with deprecation metadata. This allows internal architecture to evolve without breaking existing MCP clients.

## 11. Event-Driven Intelligence

Emit domain events:

```
device.connected
device.disconnected
capability.changed
policy.denied
approval.requested
job.created
job.started
job.completed
job.failed
job.cancelled
artifact.created
verification.failed
integration.health_changed
```

Approved event-driven workflows follow:

```
event -> filter -> policy -> workflow -> execution -> verification
```

This enables automation without uncontrolled autonomous behavior.

## 12. Integration Health

Do not reduce integration selection to a simplistic best-integration score.

Track measurable dimensions:
- capability match
- authorization status
- health
- latency
- locality
- resource availability
- policy compatibility
- reliability history
- version compatibility

Use these dimensions to eliminate unsuitable integrations and choose among valid execution paths.

## 13. Human Control

High-impact operations require policy-defined approval.

An approval records:
- requested action
- target device
- capabilities
- affected resources
- expected impact
- action preview where possible
- expiry
- approving identity

Approval is bound to a specific execution intent and cannot authorize unrelated work.

## 14. SOP / Knowledge Integration

Operational knowledge is advisory, never authorization:

```
Knowledge/SOP
     ↓
Candidate procedure
     ↓
Current-state inspection
     ↓
Capability/policy validation
     ↓
Execution plan
     ↓
Verification
```

Reusable procedures preserve source, version, applicability, validation date, required capabilities, and known failure modes.

## 15. Security Boundary

Intelligence never receives implicit OS authority:

```
Reasoning
   ↓
Plan
   ↓
Capability request
   ↓
Policy
   ↓
Scoped executor
   ↓
Operating system
```

The planner proposes actions; only the execution layer performs them.

## 16. Performance Architecture

Use three paths.

**Hot path:** MCP → authentication → policy → typed execution → response

**Async path:** MCP → durable job → worker → events → artifacts

**Intelligence path:** intent → context → discovery → planning → policy preflight → execution graph → verification

Large document processing, indexing, scans, telemetry exports, and artifact transformation stay off the synchronous hot path.

## 17. Roadmap

### P0
- capability registry
- typed capability descriptors
- integration adapter contract
- policy preflight
- execution request/result types
- structured verification
- correlation IDs
- capability discovery
- device capability inventory
- bounded execution
- cancellation
- audit events

### P1
- execution DAG
- durable jobs
- device health
- remote sessions
- integration resolver
- retry/recovery engine
- artifact/evidence manager
- event bus
- workflow definitions
- compatibility registry

### P2
- Android/Termux providers
- GitHub integration
- browser integration
- Docker/container integration
- SOP/knowledge retrieval
- approval center
- admin console
- fleet management
- PostgreSQL control plane

### P3
- distributed workers
- multi-tenant policy federation
- signed plugins
- adaptive workflow optimization
- event-driven automation marketplace
- cross-device orchestration

## Acceptance Criteria

Pravidhi OS Control is capability-intelligent when:
1. Connected devices advertise their actual capabilities.
2. User intent resolves to valid integrations.
3. Resolution considers policy, scope, health, locality, and resource constraints.
4. Multi-step work executes as a typed plan/DAG.
5. Important operations produce independently verifiable evidence.
6. Failures trigger bounded, classified recovery.
7. High-impact operations require policy-defined approval.
8. Existing MCP clients remain compatible through a compatibility layer.
9. New integrations can be added through adapters without modifying the core execution engine.
10. The capability model works consistently across Windows, Linux, VPS, Android/Termux, and future runtimes.

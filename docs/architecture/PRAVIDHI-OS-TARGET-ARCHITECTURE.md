# Pravidhi OS — Target Architecture

## Vision
Transform the current MCP server into a production-grade **local/remote AI operating control plane**. MCP is the protocol boundary; policy, capabilities, execution, jobs, state, audit, and device runtimes are separate layers.

```
AI Client / Agent
       |
Pravidhi MCP Gateway
       |
Auth + Capability Policy
       |
Execution Orchestrator
   /             \
Local Runtime   Remote Runtime
   \             /
State + Events + Artifacts
       |
Observability
```

## Core rules
1. MCP is the interface, not the trust boundary.
2. Every privileged operation passes through policy.
3. Agents receive explicit capabilities, never implicit unrestricted OS authority.
4. Interactive operations and durable jobs are separate execution modes.
5. Every mutation is auditable.
6. Cancellation propagates end-to-end.
7. Local operation works without cloud dependency.
8. Remote devices have cryptographic device identity and scoped sessions.
9. Deterministic OS execution remains independent of agent reasoning.
10. Preserve a simple workstation deployment while enabling an enterprise control plane.

## Target repository structure
```
apps/
  mcp-gateway/
  control-api/
  desktop-agent/
  device-agent/
  admin-ui/
packages/
  protocol/
  capabilities/
  policy-engine/
  execution-engine/
  job-system/
  event-bus/
  state/
  audit/
  telemetry/
  secrets/
  artifacts/
  filesystem/
  process/
  terminal/
  documents/
  browser/
  devices/
  plugins/
  compatibility/
infra/
  docker/
  compose/
  systemd/
  k8s/
tests/
  unit/
  integration/
  security/
  contract/
  e2e/
  chaos/
docs/
  architecture/
  security/
  operations/
  migration/
```

Avoid package fragmentation unless a boundary has an independent lifecycle or security purpose.

## Capability model
Capabilities are explicit:
```
filesystem.read
filesystem.write
process.list
process.start
process.signal
terminal.execute
documents.read
documents.write
browser.navigate
browser.interact
device.screen.read
device.input.write
service.manage
network.request
secrets.read
```

Each capability has an ID, risk class, scope, policy, approval mode, timeout, resource limits, and audit requirement.

Risk classes:
- READ
- MUTATE
- EXECUTE
- PRIVILEGED
- DESTRUCTIVE

## Policy flow
```
request
 -> identity
 -> device/session
 -> capability
 -> scope validation
 -> policy
 -> approval
 -> allow / deny / confirmation
```

Never use a single unrestricted `full_access` switch as the security model.

## Execution model
Normalize operations as:
```
ExecutionRequest
  id
  actor
  device
  capability
  operation
  args
  cwd
  environmentRef
  deadline
  resourceLimits
  idempotencyKey
  cancellationToken
```

Lifecycle:
```
QUEUED -> AUTHORIZING -> APPROVED -> RUNNING
                                  -> SUCCEEDED
                                  -> FAILED
                                  -> CANCELLED
                                  -> TIMED_OUT
                                  -> ABANDONED
```

Long-running work becomes durable jobs with idempotency keys, leases, heartbeats, retry policy, cancellation, timeout, dead-letter state, and artifacts.

## Runtime isolation
The local runtime owns OS operations through typed adapters:
- filesystem
- process
- terminal
- services
- documents
- browser

Never expose raw child-process APIs above the execution boundary. Enforce argument validation, cwd/root validation, environment filtering, timeout, output limits, signal handling, and audit events.

## Remote devices
Devices use durable cryptographic identity rather than hostnames:
```
control plane
  |
authenticated device session
  |
Windows / Linux / macOS / Android / Termux agent
```

Registration records device ID, public key, platform, agent version, capabilities, health, last-seen, and policy profile.

Android should eventually be a first-class runtime using independent capability providers for AccessibilityService, MediaProjection, Termux, and optional Shizuku integration.

## State and event plane
Single-machine: SQLite. Multi-device: PostgreSQL. Large artifacts: object storage.

Core entities:
```
users, devices, device_keys, sessions, capabilities, policies,
approvals, jobs, job_attempts, artifacts, audit_events,
tool_invocations, agent_runs, configuration_versions
```

Canonical event envelope:
```json
{
  "id": "evt_...",
  "type": "execution.completed",
  "timestamp": "...",
  "actor": "...",
  "device": "...",
  "session": "...",
  "correlationId": "...",
  "severity": "INFO",
  "data": {}
}
```

Consumers must be idempotent.

## Agent layer
Separate reasoning from OS execution:
```
planner -> state/memory -> tool selector -> policy preflight
       -> execution -> observation -> verification -> recovery
```

High-risk flow:
```
plan -> policy -> human approval -> execute -> verify
```

## Plugin architecture
Plugins declare capabilities and permissions. Lifecycle:
```
discover -> validate -> install -> activate -> health -> disable -> remove
```

Plugin permissions must cover network, secrets, filesystem, and execution independently.

## Security threat model
Explicitly test:
- prompt injection
- malicious files
- compromised MCP clients
- stolen device credentials
- malicious plugins
- command injection
- path traversal
- symlink attacks
- environment-variable leakage
- secret exfiltration
- confused-deputy attacks
- replayed remote requests
- privilege escalation

Controls:
- capability authorization
- scoped filesystem roots
- command policy
- environment sanitization
- output limits
- deadlines
- authentication
- key rotation
- audit
- rate limiting
- secret redaction
- dependency scanning
- signed/reproducible releases where practical

## Observability
Every operation carries:
```
trace_id, span_id, correlation_id, request_id, job_id, device_id, actor_id
```

Track latency, success/failure, policy denials, approval latency, active jobs, queue depth, abandoned jobs, device connectivity, resource saturation, and plugin failures. Never use raw commands or arbitrary filesystem paths as metric labels.

## Reliability
Graceful shutdown:
```
stop accepting
 -> mark not-ready
 -> stop scheduling
 -> bounded drain/cancel
 -> flush audit/telemetry
 -> close sessions
 -> close storage
 -> exit
```

Configuration is defaults -> config file -> environment -> explicit runtime override. Validate at startup and classify values as immutable, reloadable, secret, or operational.

## Deployment modes

### Single machine
Gateway + local runtime + SQLite + local artifacts.

### Developer workstation
AI client -> MCP -> Pravidhi OS agent -> Windows/Linux/macOS.

### Enterprise
Clients -> gateway -> auth/policy -> execution/job plane -> fleet of Windows/Linux/Android agents -> PostgreSQL/object storage/observability.

## Performance
Keep the request hot path small:
```
MCP decode -> authorization -> typed execution -> response
```
Move indexing, conversion, scans, large file work, analytics, and telemetry export to workers. Apply backpressure; never allow unbounded queues.

## Testing
- Unit: policy, capability, validation, state transitions.
- Contract: MCP schemas, events, device protocol.
- Integration: filesystem, process, terminal, persistence, remote session.
- Security: traversal, injection, authorization bypass, secret leakage, replay.
- E2E: MCP -> policy -> execution -> event -> persistence -> response.
- Chaos: agent disconnect, process crash, DB outage, telemetry outage, duplicate request, cancellation, expired lease, partial artifact upload.

## Migration sequence
1. Stabilize capability/execution/policy/audit interfaces around existing implementations.
2. Isolate filesystem/process/terminal behind adapters.
3. Add durable jobs, leases, cancellation, and persistence.
4. Add device identity and remote sessions.
5. Add planner/state/verification agent runtime.
6. Add capability-declared plugin ecosystem.
7. Add PostgreSQL fleet/enterprise control plane.

## Immediate P0 backlog
- capability interface
- execution request/result model
- centralized policy
- structured audit events
- graceful shutdown
- request/job correlation
- bounded execution/output
- dependency cleanup
- complete upstream rebrand

## P1
- durable job manager
- SQLite state store
- device identity
- remote agent protocol
- cancellation propagation
- artifact manager
- metrics/tracing

## P2
- PostgreSQL control plane
- fleet management
- RBAC
- Android/Termux agent
- plugin capability registry
- agent runtime
- admin UI

## P3
- multi-tenancy
- distributed workers
- policy federation
- signed plugins
- workload scheduling
- advanced event-driven automation

## Definition of done
An AI client can control authorized machines/devices through one capability model, while every meaningful operation is policy-checked, bounded, observable, cancellable, and auditable. The same architecture must support a simple offline workstation and grow into a multi-device enterprise control plane without replacing the MCP contract.

# Task: Complete Pravidhi OS Rebrand + Production Hardening

**Status:** Planned  
**Target:** Pravidhi OS / Pravidhi OS MCP  
**Repository:** https://github.com/yashas-13/Pravidhi-sys-MCP

## Why

The repository is a fork of DesktopCommanderMCP and still exposes the upstream product identity across package metadata, MCP manifests, documentation, release automation, runtime names, URLs, and assets. This must become a first-class Pravidhi OS product rather than a cosmetic fork.

## Canonical identity

- Product: **Pravidhi OS**
- MCP display name: **Pravidhi OS MCP**
- Brand: **Pravidh Solutions**
- Website: https://pravidhisolutions.in
- GitHub: https://github.com/yashas-13/Pravidhi-sys-MCP
- Proposed npm package: `@pravidhisolutions/pravidhi-os-mcp` — verify availability before adopting.
- Proposed MCP Registry name: `io.github.yashas-13/pravidhi-os-mcp` — verify registry ownership/rules before publishing.

## Audit scope

Search the complete tracked repository for:

- Desktop Commander / DesktopCommanderMCP / desktop-commander
- wonderwhy-er / @wonderwhy-er
- desktopcommander.app
- upstream GitHub URLs
- upstream npm identifiers
- MCP Registry identifiers
- inherited emails
- inherited telemetry/event names
- user-visible inherited tool names
- screenshots, logos, videos, badges, testimonials, and generated metadata

Classify every match as public identity, compatibility surface, implementation detail, legal attribution, fixture/example, or generated artifact. Do not remove legally required attribution blindly.

## Implementation

### Package and distribution

- Update npm package name/metadata, binaries, keywords, homepage, repository, bugs, author, and MCP name.
- Update package-lock and generated/version synchronization.
- Update MCP Registry, MCPB, plugin, Smithery, and client configuration metadata.
- Ensure release scripts cannot publish to upstream namespaces.

### Runtime

- Audit externally visible tool names and telemetry identifiers.
- Rename safe public identifiers.
- Preserve compatibility aliases where changing an existing MCP contract would break clients.
- Keep core filesystem, terminal, process, document, editing, and audit functionality intact.

### Documentation and UX

- Rewrite README and installation instructions for Pravidhi OS.
- Replace upstream URLs, commands, badges, support references, and product language.
- Review `logo.png`, `icon.png`, `header.png`, screenshots, video, and UI strings.
- Document the actual local/remote system-control model and security boundaries.

### Production engineering

Apply the production lifecycle principles from the Go production-engineering reference to the actual TypeScript/Node runtime; **do not introduce Go solely for this task**:

- deterministic configuration validation before serving
- clear required/immutable/reloadable configuration semantics
- bounded telemetry queues and exporter failure domains
- bounded metric cardinality and stable event names
- meaningful readiness/liveness behavior
- graceful admission stop, drain, background-producer stop, telemetry flush, and dependency close
- forced-termination/abandoned-work visibility
- reproducible builds and pinned CI inputs where practical
- minimal/non-root container where practical
- dependency/security scanning
- realistic resource constraints
- staged release and rollback safety

### CI / release safety

Add a CI identity-consistency gate that fails when forbidden upstream identifiers appear in release/public metadata.

Verify:

- clean install
- build
- unit/integration tests
- tool schema validation
- MCP Inspector smoke test
- stdio startup
- Docker build/run
- release rehearsal
- npm package ownership/availability
- MCP Registry naming/ownership
- rollback and migration procedure

## Acceptance criteria

- [ ] No unintended Desktop Commander identity remains in public product surfaces.
- [ ] No upstream npm/GitHub/MCP Registry target can be published accidentally.
- [ ] Pravidhi OS identity is consistent across code, manifests, docs, assets, telemetry, CI, and release tooling.
- [ ] Core MCP functionality remains intact.
- [ ] Intentional upstream attribution is documented.
- [ ] CI enforces the identity boundary.
- [ ] Startup/readiness/shutdown/telemetry behavior is verified.
- [ ] Release rehearsal cannot publish to the wrong namespace.
- [ ] Migration and rollback are documented.
- [ ] Final PR contains an identity inventory and verification evidence.

## Recommended sequence

1. Inventory and freeze compatibility requirements.
2. Verify canonical npm/registry identifiers.
3. Update package and MCP metadata.
4. Update runtime public names with compatibility where necessary.
5. Rewrite docs/install paths.
6. Replace branding assets.
7. Harden release automation.
8. Add identity CI gate.
9. Verify lifecycle behavior.
10. Run clean/MCP/Docker/release-rehearsal tests.
11. Perform final upstream-reference sweep.
12. Open a PR for review; do not push these changes directly to `main`.

## Research basis

The current repository metadata identifies it as a fork of `wonderwhy-er/DesktopCommanderMCP`, while current package/server/plugin manifests still reference the upstream npm package, MCP identity, URLs, and branding. The production work should therefore treat identity as a compatibility and release-safety concern, not merely documentation.

**Important:** GitHub Issues are currently disabled for this repository, so this file is the persistent task artifact until a project/issue tracker is enabled.

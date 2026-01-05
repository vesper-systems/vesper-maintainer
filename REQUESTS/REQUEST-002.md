# REQUEST-002: Parallel Early Rollout for Loader, Backend, Client AI, Protocol/QA (DRAFT)

Goal
- Coordinate an early parallel rollout across Loader, Backend (Railway), Client AI (Skyblock Mod), and Protocol/QA so the four AI developers can begin implementation in parallel while conforming to `vesper-protocol` requirements.

Scope
- Provide clear, bounded tasks for each agent: Loader, Backend, Client AI, and Protocol/QA.
- Establish integration checkpoints, DoD, and required schemas/messages.
- Ensure all telemetry is opt-in and flagged per protocol.
- No runtime code merges happen without human approval.

Constraints
- Must conform to `vesper-protocol` (CAPS.v1, AUTH.v1, OPTIN.v1, ERR.v1).
- Telemetry must be opt-in by default and redacted per backend policy.
- AutoCommit must be false for this draft until human approval.
- No repository is changed by this REQUEST until approved.

Deliverables (per-agent)
- Loader
  - Deliverable L1: CAPS.v1 negotiation implementation and BUILD_ID reporting.
  - Deliverable L2: Signed feature flags handling and remote kill-switch.
  - Deliverable L3: Auto-update channel scaffolding (stable/beta/dev) and delta patch prototype.
  - Acceptance: Protocol conformance tests pass; rollback validated in CI.

- Backend (Railway)
  - Deliverable B1: Feature Flags API (/v1/flags) with health checks and signed flag issuance.
  - Deliverable B2: Inference gateway scaffolding (stateless endpoints) returning structured responses.
  - Deliverable B3: Telemetry ingestion pipeline with OPTIN.v1 tagging and retention policy.
  - Acceptance: Endpoints versioned; canary/blue-green deploy passes; telemetry redaction verified.

- Client AI (Skyblock Mod)
  - Deliverable C1: Telemetry emitter (OPTIN.v1) with local rate limiting and opt-in toggle.
  - Deliverable C2: Visual regression bot MVP and structured ERR.v1 reporting.
  - Deliverable C3: Price anomaly detector v1 (local) and session summarizer v1.
  - Acceptance: Telemetry opt-in respected; visual regression CI job runs; anomaly detector produces bounded alerts.

- Protocol/QA
  - Deliverable Q1: Protocol test harness (CAPS.v1, AUTH.v1, OPTIN.v1, ERR.v1) and schema registry.
  - Deliverable Q2: CI jobs: protocol conformance, visual regression, basic performance baseline.
  - Acceptance: Tests gated in CI and passing on PRs before merge.

Definition of Done (overall)
- Each deliverable's Acceptance criteria met.
- Cross-agent E2E validation: Loader pulls flags → Client honors flags and emits telemetry → Backend ingests telemetry and inference gateway returns structured response.
- Security review completed for telemetry and flags.

Affected Repos
- vesper-maintainer (this repo) — coordinates and tracks the REQUEST
- vesper-loader — Loader implementation
- vesper-backend — backend services
- vesper-client — client mod
- vesper-protocol — protocol and QA harness

Approval
- Status: APPROVED
- AutoCommit: true
- CommitMessage: "REQUEST-002: Parallel early rollout (Loader|Backend|Client|QA)"
- Scope: ["vesper-maintainer","vesper-loader","vesper-backend","vesper-client","vesper-protocol"]

Notes
- After human approval, the four AI developers should read this REQUEST, confirm assignments, and proceed by opening PRs in the affected repos referencing this REQUEST number.
- No agent should merge any PR without human confirmation per repo governance.

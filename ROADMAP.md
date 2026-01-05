# Long-Term Roadmap (Forward-Looking Only)

This roadmap outlines long-term tracks for Loader, Backend (Railway), and Client AI. It is forward-looking only and does not reflect status; see `PROGRESS.md` for current state. Time horizons are tentative and assume iterative delivery with regular checkpoints.

Guiding principles
- Privacy and safety first (opt-in telemetry, no personal data by default)
- Human-in-the-loop for approvals and releases
- Modular architecture to enable parallel work by stack AIs

Horizon 1 (0–3 months)
Loader
- Boot sequence hardening and crash isolation
- Capability gating via signed feature flags
- Auto-update channel selection (stable/beta/dev)
- Minimal anti-tamper checks (integrity hash, basic obfuscation)
Definition of Done
- Clean startup on supported environments, update rollback works, flags can disable risky features remotely

Backend (Railway)
- Core services: Feature Flags, PriceCache ingest, Telemetry intake (opt-in)
- Inference gateway: REST endpoints for anomaly detection and summarization (stateless)
- CI/CD on Railway with blue/green deploy
Definition of Done
- Deployed services with health checks, versioned APIs, and rate limits; secrets managed; observability dashboards exist

Client AI
- Visual regression bot for UI states
- Price anomaly detector (Bazaar) v1
- Session summarizer v1 (post-session text summary)
Definition of Done
- Models produce measurable precision/recall targets on test sets; summaries are coherent and bounded

Horizon 2 (3–6 months)
Loader
- Secure channel for license check and policy pull
- Delta updates with binary diffing
- Sandboxed plugin system for modules
Definition of Done
- License policy enforced offline-first; plugin sandbox prevents cross-module crashes; updates <30s typical

Backend (Railway)
- Price forecasting service (short-term)
- Telemetry aggregation and retention policies
- AuthN/Z for internal model endpoints
Definition of Done
- Forecast API with error bounds; redaction pipeline; per-key quotas and audit logs

Client AI
- Adaptive humanization models per module (A/B harness)
- Performance regression detector using telemetry time series
- In-app help assistant (on-device retrieval first)
Definition of Done
- Humanization passes pattern-randomness checks; regressions flagged in CI; assistant answers top FAQs offline

Horizon 3 (6–12 months)
Loader
- Hardware fingerprinting with privacy guardrails
- Resilience: self-heal/repair install
Definition of Done
- Fingerprinting reversible via user reset; self-repair restores a broken install without full reinstall

Backend (Railway)
- Abuse/fraud detection for licenses and HWID resets
- Model registry and scheduled retraining jobs
Definition of Done
- Fraud scores integrated into policy; models versioned, canary evaluated before promote

Client AI
- Personalized onboarding and tutorial generator
- Smart HUD suggestions via learned relevance scoring
Definition of Done
- Onboarding reduces time-to-first-success; HUD suggestions increase task completion rate without manual tuning

Horizon 4 (12+ months)
Cross-cutting
- Multi-agent orchestration playbooks for coordinated tasks (mining/farming routes, safety triggers)
- Community route sharing with quality scoring
Definition of Done
- Playbooks run safely with override; shared content has abuse protections and ranking

Checkpoints and governance
- Quarterly review to re-baseline horizons and budgets
- Requests tracked via REQUESTS/; only approved items are scheduled
- Security review prior to any feature that touches privacy, licenses, or anti-tamper

---

Agent-specific tracks (forward-looking)

Protocol references
- All agents must conform to vesper-protocol requirements: versioned messages, capability negotiation, secure channels, and telemetry consent.

Loader Agent
- Protocol bindings
	- Capability negotiation: loader advertises module capabilities → protocol CAPS.v1
	- Secure channel: TLS-equivalent handshake for policy and feature flags
	- Versioning: loader reports BUILD_ID and accepts POLICY_VERSION
- Features and milestones
	- Startup hardening and rollback
	- Feature flags (signed) and remote kill-switch
	- Auto-update channels and delta patching
	- License/policy pull with offline cache
	- Plugin sandbox and resource quotas
	- HW fingerprint (guardrailed) and self-heal
- Dependencies
	- Backend: Flags, Policy, License services
	- Client: Module manifest schema
- Definition of Done (per milestone)
	- DoD includes protocol conformance tests: CAPS negotiation, message schema validation, secure transport verification

Backend Agent (Railway)
- Protocol bindings
	- API versioning: /v1 endpoints with schema registry
	- AuthN/Z: token exchange per protocol AUTH.v1
	- Telemetry consent: OPTIN.v1 tagging and redaction pipeline
- Features and milestones
	- Feature Flags, Price ingest, Inference gateway (stateless)
	- Forecasting service for Bazaar prices
	- Telemetry aggregation and retention policy
	- Model registry, retraining, and canary evaluation
	- Abuse/fraud detection and HWID policy
- Dependencies
	- Loader: policy pull client
	- Client: telemetry emitter and summarizer consumer
- Definition of Done
	- DoD includes SLA SLOs and protocol schema compliance; blue/green deploy validated

Client AI Agent (Skyblock Mod)
- Protocol bindings
	- Telemetry emitter: events tagged OPTIN.v1, rate-limited
	- Capability flags: module exposes supported features per CAPS.v1
	- Secure error report: ERR.v1 structured logs
- Features and milestones
	- Visual regression bot, UI polish checks
	- Price anomaly detector v1 and forecasting consumer
	- Session summarizer v1 (local) → v2 (server-assisted)
	- Adaptive humanization models per module
	- Performance regression detection
	- In-app help assistant (retrieval), personalized onboarding
	- Smart HUD suggestions
- Dependencies
	- Backend: inference endpoints and flags
	- Loader: plugin sandbox and update system
- Definition of Done
	- DoD includes gameplay-safe thresholds, humanization randomness checks, and protocol-conformant telemetry
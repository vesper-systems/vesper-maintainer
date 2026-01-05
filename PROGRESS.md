# Project Progress (Single Source of Truth)

Note: This file reflects status only. Roadmap and planning live in `ROADMAP.md`. No item is marked complete unless explicitly confirmed by a human.

Tracks overview
- Loader
- Backend (vesper-backend)
- Client AI (vesper-client)

Loader
- Horizon 1
	- Startup hardening and rollback [Planned]
	- Feature flag gating [Planned]
	- Auto-update channels [Planned]
- Horizon 2
	- License/policy pull channel [Planned]
	- Delta updates [Planned]
	- Plugin sandbox [Planned]
- Horizon 3
	- HW fingerprinting (guardrailed) [Planned]
	- Self-heal repair [Planned]

Backend (vesper-backend)
- Horizon 1
	- Core services (Flags, Ingest, Inference gateway) [Planned]
	- CI/CD and observability [Planned]
- Horizon 2
	- Price forecasting API [Planned]
	- Telemetry aggregation policy [Planned]
	- AuthN/Z for model endpoints [Planned]
- Horizon 3
	- Abuse/fraud detection [Planned]
	- Model registry + retraining [Planned]

Client AI (vesper-client)
- Horizon 1
	- Visual regression bot [Planned]
	- Price anomaly detector v1 [Planned]
	- Session summarizer v1 [Planned]
- Horizon 2
	- Adaptive humanization (A/B) [Planned]
	- Performance regression detector [Planned]
	- In-app help assistant [Planned]
- Horizon 3
	- Personalized onboarding [Planned]
	- Smart HUD suggestions [Planned]

Cross-cutting (Horizon 4+)
- Multi-agent playbooks [Planned]
- Community route sharing [Planned]

Log
- 2026-01-05: Initialized long-term tracks and placeholders. No milestones marked complete yet.

---

Agent-specific status (bound to vesper-protocol)

Protocol compliance (global)
- Versioned messages (CAPS.v1, AUTH.v1, OPTIN.v1, ERR.v1) [Planned]
- Capability negotiation tests [Planned]
- Secure transport verification [Planned]
- Telemetry consent tagging and redaction [Planned]

Loader Agent
- Protocol bindings
	- CAPS.v1 advertisement/negotiation [Planned]
	- Secure policy/flags channel [Planned]
	- BUILD_ID + POLICY_VERSION reporting [Planned]
- Features
	- Startup hardening and rollback [Planned]
	- Signed feature flags and kill-switch [Planned]
	- Auto-update channels (stable/beta/dev) [Planned]
	- Delta patching [Planned]
	- License/policy pull with offline cache [Planned]
	- Plugin sandbox + quotas [Planned]
	- HW fingerprint (guardrailed) [Planned]
	- Self-heal repair [Planned]
- DoD checks
	- Protocol tests pass (CAPS, schema, secure transport) [Planned]
	- Rollback validated on failure [Planned]

Backend Agent (vesper-backend)
- Protocol bindings
	- /v1 API versioning + schema registry [Planned]
	- AUTH.v1 token exchange (roles/scopes) [Planned]
	- OPTIN.v1 telemetry consent pipeline [Planned]
- Features
	- Feature Flags, Price ingest, Inference gateway [Planned]
	- Price forecasting service [Planned]
	- Telemetry aggregation + retention [Planned]
	- AuthN/Z for model endpoints [Planned]
	- Abuse/fraud detection + HWID policy [Planned]
	- Model registry + canary retraining [Planned]
- DoD checks
	- SLA/SLO met; blue/green deploy validated [Planned]
	- Protocol schema compliance [Planned]

Client AI Agent (vesper-client)
- Protocol bindings
	- OPTIN.v1 telemetry emitter (rate-limited) [Planned]
	- CAPS.v1 module capability flags [Planned]
	- ERR.v1 structured error reports [Planned]
- Features
	- Visual regression bot + UI checks [Planned]
	- Price anomaly detector v1 [Planned]
	- Forecasting consumer [Planned]
	- Session summarizer v1→v2 [Planned]
	- Adaptive humanization per module [Planned]
	- Performance regression detection [Planned]
	- In-app help assistant (retrieval) [Planned]
	- Personalized onboarding [Planned]
	- Smart HUD suggestions [Planned]
- DoD checks
	- Humanization randomness tests pass [Planned]
	- Protocol-conformant telemetry [Planned]
# ARES — AuthorityRail Execution Standard v1

**Execution Authority Infrastructure — the cryptographic authorization standard that determines what AI agents are permitted to do before they do it.**

Every time an AI agent makes a decision that touches the real world — a payment, a deployment, a database write, an email — that moment has no standard. No checkpoint. No receipt. ARES defines what that checkpoint looks like, what it must verify, and what evidence it must produce.

> *"Probability ≠ permission."*

---

## Why ARES Exists

AI agents are probabilistic. They generate decisions. Those decisions are becoming actions — real actions with real consequences: financial transfers, infrastructure changes, data deletions, communications sent on behalf of humans.

The gap between **AI decision** and **real-world action** is currently ungoverned. IAM covers identity. SIEM covers post-hoc logging. API gateways cover routing. None of them cover the moment an AI agent's output becomes an instruction to a downstream system.

ARES is the specification for governing that moment. It defines:

- What an agent must submit before executing an action
- What an authority checkpoint must verify
- What cryptographic receipt must be produced
- What evidence must be retained for audit and liability

Any platform, any agent framework, any enterprise can implement ARES. The standard is open, free, and implementation-agnostic.

---

## Core Components

### Certified Action Records (CARs)

A CAR is the cryptographic receipt produced by every authorization decision. It is the central artifact of ARES.

Every CAR contains:
- The agent identity and action class that was evaluated
- The decision: `ALLOW`, `DENY`, or `ESCALATE`
- A SHA-256 digest of the full decision payload (RFC 8785 canonicalization)
- An Ed25519 signature over that digest
- A risk score, policy reference, and timestamp

CARs are **tamper-evident by construction**. Any modification after issuance is detectable by recomputing the digest. They are designed to be independently verifiable — any party with the public key can verify a CAR without accessing the issuing infrastructure.

```json
{
  "car_id": "urn:authorityrail:car:a1b2c3d4",
  "agent_id": "urn:authorityrail:agent:000001",
  "action_class": "transaction.execute",
  "decision": "ALLOW",
  "risk_score": 0.12,
  "integrity": {
    "algorithm": "sha-256",
    "digest": "sha256:e3b0c44298fc1c149afbf4c8996fb924..."
  },
  "signature": {
    "alg": "Ed25519",
    "key_id": "a1b2c3d4e5f6a7b8",
    "signed_at": "2026-03-10T14:32:00Z",
    "value": "base64-encoded-signature"
  }
}
```

---

### MAACS — Multi-Agent Authorization & Control System

MAACS is the four-lane evaluation engine at the core of any ARES-aligned checkpoint. It evaluates every execution request through four independent lenses before issuing a decision.

| Lane | What It Evaluates |
|---|---|
| **Risk** | Is the risk score within policy thresholds for this action class? |
| **Compliance** | Does the action conform to the active compliance policy? |
| **Intent** | Does the stated intent match the action class and payload? |
| **TOCTOU** | Has the execution context drifted since the decision was made? |

Each lane casts an independent vote. The four votes are weighted and combined into a final decision. Dissent across lanes can trigger escalation. No single lane can veto by itself; no single lane can authorize by itself.

MAACS produces a structured result with every decision:

```json
{
  "decision": "ALLOW",
  "risk_lane_vote": "ALLOW",
  "compliance_lane_vote": "ALLOW",
  "intent_lane_vote": "ALLOW",
  "toctou_lane_vote": "ALLOW",
  "maacs_weighted_score": 0.94,
  "maacs_dissent_count": 0,
  "policy_triggered": "default",
  "matched_rule": "financial.transfer.standard"
}
```

---

### AXAP — Agent eXecution Authorization Protocol

AXAP is the wire protocol layer of ARES. It defines the exact HTTP exchange between a caller (agent or platform) and an authority checkpoint.

**Request** — `POST /authorize-execution`

```json
{
  "agent_id": "urn:authorityrail:agent:000001",
  "model_id": "claude-3-opus",
  "pattern_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "action": "transaction.execute",
  "environment": "production",
  "risk_context": {
    "domain": "finance",
    "sensitivity": "confidential",
    "reversible": false
  }
}
```

**Response**

```json
{
  "decision": "ALLOW",
  "risk_score": 0.12,
  "permission_status": "ALLOW",
  "authority_signature": "ed25519:base64...",
  "car_id": "urn:authorityrail:car:a1b2c3d4",
  "decision_receipt_url": "https://verify.example.com/verify/urn:authorityrail:car:a1b2c3d4"
}
```

**Decision semantics:**

| Decision | Meaning | What the caller must do |
|---|---|---|
| `ALLOW` | Authorized | Proceed with execution |
| `DENY` | Prohibited | Halt — do not proceed |
| `ESCALATE` | Human review required | Suspend until human approves or denies |

The checkpoint **always fails closed**. Any internal error, timeout, or unavailable upstream service returns `DENY`. There is no fallback-to-allow path.

---

### WorkforceRail

WorkforceRail is the governance pattern for multi-agent deployments — fleets of AI agents operating across an organization, each governed independently but with unified audit and liability attribution.

In a WorkforceRail deployment:

- Every agent has a registered identity with an owning organization
- Every agent action is governed by ARES regardless of which platform hosts it
- Liability is attributed to the owner of the agent that executed, not the platform
- Cross-agent action chains are traceable through the Provenance Map
- A single audit export covers all agents across all platforms for a given organization

WorkforceRail answers the question regulators and insurers will ask: *"When this AI agent caused harm, who is accountable, and what evidence exists?"*

---

## The Authorization Pipeline

A compliant ARES-STANDARD checkpoint evaluates every request through these steps in order:

```
 0. Agent Identity     — Is this agent registered? Is it active?
 1. Pattern Provenance — Is this reasoning pattern certified?
 2. Trust Registry     — What is this agent's trust score and certification tier?
 3. RAAS Engine        — What is the risk-adjusted score for this action?
 4. Reality Permission — Should this AI outcome be allowed to enter the real world?
 5. MAACS Gate         — Rate limit, data classification, org policy, guardrails
 6. CAR Issuance       — Seal the decision into a signed, tamper-evident receipt
 7. Decision Ledger    — Record the decision (immutable, append-only)
 8. Liability Ledger   — Record accountability attribution
```

Every step that fails returns `DENY`. No step can be skipped. The checkpoint does not proceed to step N+1 if step N fails.

---

## Compliance Tiers

ARES defines three tiers. Each tier specifies which modules of the checkpoint must be operational.

| Module | BASIC | STANDARD | FULL |
|---|---|---|---|
| Authority Gate | ✅ | ✅ | ✅ |
| Trust Registry | ✅ | ✅ | ✅ |
| CAR Issuer | ✅ | ✅ | ✅ |
| RAAS Engine | — | ✅ | ✅ |
| Pattern Provenance | — | ✅ | ✅ |
| Reality Permission | — | ✅ | ✅ |
| Decision Ledger | — | ✅ | ✅ |
| Liability Ledger | — | — | ✅ |
| Public Verification | — | — | ✅ |
| Provenance Map | — | — | ✅ |
| Agent Identity Registry | — | — | ✅ |

**ARES-BASIC** — Sandbox and development environments. Minimum: gate + trust registry + CAR issuance.

**ARES-STANDARD** — Staging and internal production. Adds pattern governance, risk scoring, and reality permission.

**ARES-FULL** — Regulated environments, insurance-backed deployments, enterprise compliance. Complete stack with public verification and liability attribution.

---

## Cryptographic Requirements

| Requirement | Specification |
|---|---|
| Digest algorithm | SHA-256 |
| Signature algorithm | Ed25519 |
| Canonicalization | RFC 8785 (JSON Canonicalization Scheme) |
| Signature encoding | Base64 |
| Key publication | JWKS at `/.well-known/ar-keys.json` |

The use of RFC 8785 canonicalization means CAR digests are byte-identical across any compliant implementation. A CAR issued by any ARES-aligned checkpoint can be verified by any other.

---

## How to Implement

### Minimum viable implementation (ARES-BASIC)

1. Stand up an HTTP service at `POST /authorize-execution`
2. Accept the `ExecutionRequest` schema (validate `agent_id`, `model_id`, `pattern_hash`, `action`, `environment`)
3. Look up the agent in your trust registry — deny unknown agents
4. Evaluate the action against your policy — compute a risk score
5. Issue a CAR: compute SHA-256 digest (RFC 8785), sign with Ed25519, store the record
6. Return the `ExecutionAuthorization` response
7. **Always fail closed** — any exception or unavailable dependency returns `{ "decision": "DENY" }`

### Schema validation

The machine-readable JSON Schema for both `ExecutionRequest` and `ExecutionAuthorization` is published in this repository at [`ARES-SCHEMA.json`](./ARES-SCHEMA.json).

```bash
# Validate a request against the ARES schema
curl -s -X POST https://your-checkpoint/authorize-execution \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "urn:authorityrail:agent:test-001",
    "model_id": "your-model",
    "pattern_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "action": "data.read",
    "environment": "sandbox"
  }' | jq .
```

### Verify fail-closed behavior

```bash
# Expected: decision=DENY — checkpoint must not fail open
curl -s -X POST https://your-checkpoint/authorize-execution \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "urn:authorityrail:agent:unregistered-999",
    "model_id": "any-model",
    "pattern_hash": "0000000000000000000000000000000000000000000000000000000000000000",
    "action": "system.destructive",
    "environment": "production"
  }' | jq '.decision'
# Must output: "DENY"
```

---

## Reference Implementation

**[AuthorityRail](https://github.com/AuthorityRail-ai/authorityrail)** is the open-source reference implementation of ARES-FULL.

It implements every module in the compliance tier table above as independently deployable services, wired together through the AXAP protocol. It is tested, production-capable, and ships with:

- Ed25519-signed CARs with RFC 8785 canonicalization
- MAACS 4-lane policy voting engine
- Full 9-step execution pipeline with real RAAS risk scoring
- Pipeline guardrails: rate limiting, data classification, org policy, payload scanning
- Human escalation workflow with REVIEWER RBAC
- Persistent storage via Supabase with append-only CAR ledger
- Prometheus metrics, structured pino logging, request correlation IDs
- Audit export in JSON and CSV

The AuthorityRail gate service passes the ARES-FULL compliance checklist. Its implementation can serve as a reference for any team building a custom ARES checkpoint.

---

## Governance Frameworks

ARES is designed to work alongside eight governance frameworks that cover the full lifecycle of AI execution governance:

| Framework | Acronym | Focus |
|---|---|---|
| Execution Trust & Authority Governance | ETAG | Trust-based execution control |
| Agent Compliance Verification | ACV | Agent lifecycle compliance |
| Decision Boundary Enforcement Logic | DBEL | Decision scope enforcement |
| Model Validation Architecture | MVA | Model output validation |
| Autonomy Threshold System | ATS | Autonomy level management |
| Governance Compliance Protocol | GCP | Regulatory compliance |
| Incremental Authority Allocation Tiers | IAAT | Progressive authority grants |
| Reality Permission Framework | RPF | Real-world execution gating |

---

## Regulatory Alignment

ARES is designed to support compliance with:

- **EU AI Act** — Article 14 human oversight requirements for high-risk AI systems
- **SOC 2** — CC6.x (logical access), CC7.x (threat detection), A1.x (availability), CC9.1 (risk mitigation)
- **NIST AI RMF** — Govern, Map, Measure, Manage functions
- **HIPAA-style traceability** — Immutable audit trail with agent identity and action attribution

See [`ARES-COMPLIANCE.md`](./ARES-COMPLIANCE.md) for the full self-certification checklist.

---

## Key Phrases

> *"Mandatory checkpoint between AI decisions and reality."*

> *"Agents are temporary. Patterns propagate forever."*

> *"Probability ≠ permission."*

---

## License

MIT. Any organization may implement ARES without fees or licensing restrictions.

```
Copyright (c) 2026 AuthorityRail

Permission is hereby granted, free of charge, to any person obtaining a copy
of this standard and associated documentation files, to deal in the standard
without restriction, including without limitation the rights to use, copy,
modify, merge, publish, distribute, sublicense, and/or sell copies of the
standard, and to permit persons to whom the standard is furnished to do so,
subject to the following conditions: The above copyright notice and this
permission notice shall be included in all copies or substantial portions
of the standard.
```

---

## Contributing

ARES is a draft standard. Feedback on the schema, compliance tiers, cryptographic requirements, and protocol semantics is welcome via GitHub Issues.

Breaking changes require a new major version. ARES-v1 is immutable once ratified.

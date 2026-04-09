# AuthorityRail Execution Standard (ARES) — v1

> **Version:** 1.0.0
> **Status:** Draft Standard
> **License:** MIT
> **Last Updated:** 2026-03-10

---

## Abstract

The AuthorityRail Execution Standard (ARES) defines the open protocol by which any AI system must request, receive, and record execution authorization before performing actions that affect the real world. ARES is the interoperability contract between AI agents (or the platforms that host them) and the AuthorityRail enforcement infrastructure.

> *"Probability ≠ permission."*

Any system that generates an AI decision must submit that decision to an ARES-aligned (pending audit) authority checkpoint before execution. The checkpoint evaluates identity, trust, risk, pattern provenance, reality permission, and policy compliance — then returns a cryptographically signed authorization receipt.

---

## 1. Scope

ARES applies to any software system where:

- An AI model produces an output that triggers a real-world action
- An autonomous agent executes operations on behalf of a human or organization
- A decision pipeline crosses the boundary between inference and execution

ARES does **not** apply to:

- Pure inference with no downstream side effects
- Human-initiated actions that do not involve AI decisioning
- Training, fine-tuning, or evaluation workflows

---

## 2. Terminology

| Term | Definition |
|---|---|
| **Agent** | A software entity that uses an AI model to make decisions and take actions |
| **Platform** | The infrastructure hosting one or more agents |
| **Caller** | The system submitting an `ExecutionRequest` to the authority checkpoint |
| **Authority Checkpoint** | An ARES-aligned (pending audit) service that evaluates and authorizes execution |
| **CAR** | Certified Action Record — the cryptographically signed receipt for a decision |
| **Pattern** | The reasoning structure an AI model uses to reach a conclusion |
| **Reality Permission** | Evaluation of whether an AI outcome is allowed to enter the real world |

---

## 3. ExecutionRequest Schema

Every caller MUST submit the following payload to the authority checkpoint before executing an action.

```json
{
  "agent_id": "string — globally unique agent identifier (URN)",
  "model_id": "string — underlying AI model identifier",
  "pattern_hash": "string — SHA-256 hash of the reasoning pattern",
  "action": "string — action class being requested (e.g. data.write)",
  "environment": "string — target environment (production, staging, sandbox)",
  "risk_context": {
    "description": "optional object — additional risk metadata",
    "domain": "string — domain of the action",
    "sensitivity": "string — data sensitivity level",
    "reversible": "boolean — whether the action can be undone"
  }
}
```

### Required Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `agent_id` | `string` | Yes | Globally unique agent URN |
| `model_id` | `string` | Yes | AI model identifier |
| `pattern_hash` | `string` | Yes | SHA-256 hash of reasoning pattern |
| `action` | `string` | Yes | Action class being requested |
| `environment` | `string` | Yes | Target execution environment |
| `risk_context` | `object` | No | Additional risk metadata |

### Constraints

- `agent_id` MUST be a valid URN in the format `urn:authorityrail:agent:<id>`
- `pattern_hash` MUST be a valid SHA-256 hex string (64 characters)
- `action` MUST be a dot-delimited action class (e.g., `data.read`, `transaction.execute`)
- `environment` MUST be one of: `production`, `staging`, `sandbox`, `development`

---

## 4. ExecutionAuthorization Schema

The authority checkpoint MUST return the following payload in response to a valid `ExecutionRequest`.

```json
{
  "decision": "ALLOW | DENY | ESCALATE",
  "risk_score": 740,
  "permission_status": "ALLOW",
  "authority_signature": "base64-encoded Ed25519 signature",
  "car_id": "urn:authorityrail:car:<uuid>",
  "decision_receipt_url": "https://verify.authorityrail.io/verify/<car_id>"
}
```

### Response Fields

| Field | Type | Always Present | Description |
|---|---|---|---|
| `decision` | `string` | Yes | `ALLOW`, `DENY`, or `ESCALATE` |
| `risk_score` | `number` | Yes | RAAS risk-adjusted score (0–1000) |
| `permission_status` | `string` | Yes | Reality Permission Layer decision |
| `authority_signature` | `string` | Yes | Ed25519 cryptographic signature |
| `car_id` | `string` | Yes | Certified Action Record identifier |
| `decision_receipt_url` | `string` | Yes | Public verification URL |

### Decision Semantics

| Decision | Meaning | Caller Action |
|---|---|---|
| `ALLOW` | Execution is authorized | Proceed with the action |
| `DENY` | Execution is prohibited | Do NOT proceed; log the denial |
| `ESCALATE` | Human review required | Suspend execution until human approval |

### Error Responses

If the authority checkpoint cannot process the request, it MUST return an error with the following structure:

```json
{
  "decision": "DENY",
  "error": {
    "code": "AR-XXX",
    "message": "Human-readable error description"
  }
}
```

The checkpoint MUST fail closed — any error condition results in `DENY`.

---

## 5. Protocol Flow

```
┌──────────┐     ExecutionRequest      ┌─────────────────────┐
│  Caller  │ ────────────────────────► │  Authority          │
│  (Agent) │                           │  Checkpoint (ARES)  │
│          │ ◄──────────────────────── │                     │
└──────────┘  ExecutionAuthorization   └─────────────────────┘
     │                                          │
     │  if ALLOW                                │
     ▼                                          ▼
┌──────────┐                           ┌─────────────────────┐
│ Execute  │                           │ Decision Ledger     │
│ Action   │                           │ Liability Ledger    │
└──────────┘                           │ Provenance Map      │
                                       └─────────────────────┘
```

### Sequence

1. Caller constructs an `ExecutionRequest`
2. Caller submits the request to the authority checkpoint via `POST /authorize-execution`
3. The checkpoint evaluates the request through its pipeline
4. The checkpoint returns an `ExecutionAuthorization`
5. If `ALLOW`: caller proceeds with execution
6. If `DENY`: caller halts execution
7. If `ESCALATE`: caller suspends and awaits human approval
8. The checkpoint records the decision in the Decision Ledger and Liability Ledger

---

## 6. Compliance Tiers

ARES defines three compliance tiers. Each tier specifies which modules of the AuthorityRail stack MUST be operational.

### ARES-BASIC

The minimum viable compliance tier. Suitable for sandbox and development environments.

| Module | Required |
|---|---|
| Authority Gate | ✅ |
| Trust Registry | ✅ |
| CAR Issuer | ✅ |
| RAAS Engine | ❌ |
| Pattern Provenance | ❌ |
| Reality Permission | ❌ |
| Decision Ledger | ❌ |
| Liability Ledger | ❌ |
| Verification | ❌ |
| Provenance Map | ❌ |
| Agent Identity | ❌ |

### ARES-STANDARD

The recommended tier for staging and internal production use. Adds pattern governance and reality permission.

| Module | Required |
|---|---|
| Authority Gate | ✅ |
| Trust Registry | ✅ |
| CAR Issuer | ✅ |
| RAAS Engine | ✅ |
| Pattern Provenance | ✅ |
| Reality Permission | ✅ |
| Decision Ledger | ✅ |
| Liability Ledger | ❌ |
| Verification | ❌ |
| Provenance Map | ❌ |
| Agent Identity | ❌ |

### ARES-FULL

The complete AuthorityRail stack. Required for regulated production environments, insurance-backed deployments, and enterprise compliance.

| Module | Required |
|---|---|
| Authority Gate | ✅ |
| Trust Registry | ✅ |
| CAR Issuer | ✅ |
| RAAS Engine | ✅ |
| Pattern Provenance | ✅ |
| Reality Permission | ✅ |
| Decision Ledger | ✅ |
| Liability Ledger | ✅ |
| Verification | ✅ |
| Provenance Map | ✅ |
| Agent Identity | ✅ |

---

## 7. Certification Levels

Organizations and agents can be certified at three levels:

### AuthorityRail Certified Agent

- The agent has a registered identity in the Agent Identity Registry
- The agent submits all execution requests through an ARES-aligned (pending audit) checkpoint
- The agent respects DENY and ESCALATE decisions without bypass

### AuthorityRail Certified Platform

- The platform implements at least ARES-STANDARD compliance
- All agents on the platform have registered identities
- The platform exposes a public verification endpoint for CAR receipts
- Decision and liability records are retained for ≥ 90 days

### AuthorityRail Certified Enterprise

- The enterprise implements ARES-FULL compliance
- All agents across all platforms are registered and governed
- Liability records are exportable in insurance-readable format
- Decision audit trail is available for regulatory review
- Public verification endpoint is operational at all times

---

## 8. Cryptographic Requirements

| Requirement | Specification |
|---|---|
| Digest algorithm | SHA-256 |
| Signature algorithm | Ed25519 |
| Canonicalization | RFC 8785 (JSON Canonicalization Scheme) |
| Signature encoding | Base64 |
| Key publication | JWKS endpoint at `/.well-known/ar-keys.json` |
| Tamper evidence | SHA-256 digest sealed into `integrity.digest` field |

---

## 9. Transport

- Protocol: HTTPS (TLS 1.2+) in production; HTTP permitted in development
- Content-Type: `application/json; charset=utf-8`
- Method: `POST` for execution requests; `GET` for verification
- Latency target: ≤ 100ms end-to-end for ARES-BASIC; ≤ 200ms for ARES-FULL
- Availability target: 99.9% for ARES-STANDARD; 99.99% for ARES-FULL

---

## 10. Versioning

- ARES versions are immutable once published
- Breaking changes require a new major version (e.g., ARES-v2)
- The version is embedded in the schema: `"$schema": "ares-v1"`
- Callers MUST include the version in requests (future versions)

---

## 11. Governance Frameworks

ARES is designed to work with the eight AuthorityRail governance frameworks:

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

## 12. Key Phrases

> *"Mandatory checkpoint between AI decisions and reality."*

> *"Agents are temporary. Patterns propagate forever."*

> *"Probability ≠ permission."*

> *"AI Execution Governance Stack."*

---

## 13. License

This standard is published under the MIT License. Any organization may implement ARES without fees or licensing restrictions.

```
Copyright (c) 2026 AuthorityRail

Permission is hereby granted, free of charge, to any person obtaining a copy
of this standard and associated documentation files, to deal in the standard
without restriction, including without limitation the rights to use, copy,
modify, merge, publish, distribute, sublicense, and/or sell copies of the
standard, subject to the following conditions: The above copyright notice
and this permission notice shall be included in all copies or substantial
portions of the standard.
```

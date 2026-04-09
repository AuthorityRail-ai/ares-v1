# Certified Action Record — Schema v1.0

**Standard:** ARES-v1
**Status:** Published · Immutable
**Maintained by:** AuthorityRail Standards Foundation

---

## Overview

A Certified Action Record (CAR) is the cryptographically signed, tamper-evident proof that an authorization decision was made before an AI agent action executed. Every decision — ALLOW, DENY, or ESCALATE — produces exactly one CAR. CARs are immutable. They cannot be altered, deleted, or retroactively modified.

---

## Full Schema

{
  "car_id": "CAR-48291",
  "version": "ARES-v1.0",
  "timestamp": "2026-04-09T14:22:45.501Z",
  "agent_id": "payment-agent-01",
  "agent_credential": "cred_8821abc",
  "action": "wire_transfer",
  "resource": "banking_api",
  "payload_hash": "sha256:a1b2c3d4e5f6...",
  "risk_score": 96,
  "risk_classification": "CRITICAL",
  "policy_triggered": "HIGH_VALUE_UNVERIFIED_COUNTERPARTY",
  "decision": "DENY",
  "enforcement": "HARD_BLOCK",
  "evaluation_layers": [
    "identity_check",
    "policy_rules",
    "context_auth",
    "limits_check",
    "risk_engine",
    "maacs_consensus",
    "trust_registry",
    "compliance_mapping",
    "behavioral_analysis",
    "scope_verification",
    "temporal_check",
    "resource_classification",
    "action_type_evaluation",
    "cascade_risk_assessment"
  ],
  "compliance_mapping": {
    "nist_ai_rmf": ["GOVERN-1.1", "MANAGE-2.2"],
    "iso_42001": ["6.1.2", "9.1"],
    "eu_ai_act": ["Article 9", "Article 12"]
  },
  "human_escalation": null,
  "car_hash": "sha256:f9e8d7c6b5a4...",
  "signature": "ed25519:f9e8d7c6b5a43c2d1e0f9a8b...",
  "public_key": "ed25519_pub:a1b2c3d4e5f6...",
  "immutable": true,
  "ledger_position": 48291
}

---

## Field Reference

car_id — Unique sequential identifier for this record. Never reused.
version — The ARES-v1 version that produced this CAR.
timestamp — ISO 8601 timestamp at the moment of decision. Not the moment of execution.
agent_id — The credentialed agent identity from the Trust Registry.
agent_credential — The active credential ID verified at evaluation time.
action — The action the agent attempted to execute.
resource — The target resource or system the action was directed at.
payload_hash — SHA-256 hash of the action payload. The payload itself is not stored in the CAR.
risk_score — Composite risk score from 0 to 100 across all 14 evaluation layers.
risk_classification — LOW, MEDIUM, HIGH, CRITICAL, or EXTREME based on risk score.
policy_triggered — The specific policy rule that determined the decision.
decision — ALLOW, DENY, or ESCALATE. Always present. Never null.
enforcement — The enforcement action taken: ALLOW, HARD_BLOCK, SOFT_BLOCK, or ESCALATE_HOLD.
evaluation_layers — The 14 assessment layers evaluated in sequence.
compliance_mapping — The regulatory framework requirements satisfied by this CAR.
human_escalation — Null for ALLOW and DENY decisions. Contains escalation record for ESCALATE decisions.
car_hash — SHA-256 hash of the full CAR content before signing.
signature — Ed25519 cryptographic signature of the car_hash.
public_key — The public key required to verify the signature.
immutable — Always true. CARs cannot be altered after issuance.
ledger_position — The position of this CAR in the append-only audit ledger.

---

## Decision Types

ALLOW means the action is authorized and execution proceeds. A CAR is issued before execution begins.
DENY means the action is not authorized and execution is blocked. A CAR is issued. The agent cannot retry without a new authorization request.
ESCALATE means the action requires human approval. Execution is held. A CAR is issued. The designated approver is notified. Execution proceeds or is denied based on the human decision, which is recorded in a subsequent CAR.

---

## Risk Classification

0 to 25 is LOW and the default decision is ALLOW.
26 to 50 is MEDIUM and the default decision is ALLOW with logging.
51 to 75 is HIGH and the default decision is ESCALATE.
76 to 95 is CRITICAL and the default decision is ESCALATE or DENY.
96 to 100 is EXTREME and the default decision is DENY with hard block.

---

## Verification

Any CAR can be independently verified against the public key:

curl https://api.authorityrail.com/v1/verify \
  -H "Content-Type: application/json" \
  -d '{"car_id": "CAR-48291", "signature": "ed25519:..."}'

Note: The verification API is currently in private beta. Contact hello@authorityrail.com for access.

Every autonomous action requires authority before execution.

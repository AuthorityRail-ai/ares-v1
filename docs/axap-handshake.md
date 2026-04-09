# AXAP — Agent Execution Authorization Protocol

**Standard:** ARES-v1
**Protocol:** AXAP v2.0
**Maintained by:** AuthorityRail Standards Foundation

---

## Overview

AXAP is the request and response handshake protocol that every AI agent must complete before any action executes. No action proceeds without a valid AXAP authorization response. The gate is mandatory. There is no bypass path.

---

## Request Format

POST /v1/authorize
Authorization: Bearer {agent_credential}
Content-Type: application/json

{
  "agent_id": "payment-agent-01",
  "action": "wire_transfer",
  "resource": "banking_api",
  "payload": {
    "amount": 2400000,
    "currency": "USD",
    "destination": "ACCT-UNKNOWN-7721"
  },
  "context": {
    "session_id": "sess_abc123",
    "instruction_source": "orchestration_layer",
    "timestamp": "2026-04-09T14:22:45Z"
  }
}

---

## Response Format

HTTP/1.1 200 OK
Content-Type: application/json
X-CAR-ID: CAR-48291
X-CAR-Signature: ed25519:f9e8d7c6b5a4...

{
  "decision": "DENY",
  "car_id": "CAR-48291",
  "risk_score": 96,
  "risk_classification": "CRITICAL",
  "policy_triggered": "HIGH_VALUE_UNVERIFIED_COUNTERPARTY",
  "enforcement": "HARD_BLOCK",
  "latency_ms": 0.5,
  "signature": "ed25519:f9e8d7c6b5a43c2d1e0f9a8b...",
  "message": "Action denied. CAR-48291 issued."
}

---

## Latency Targets

LOW risk: under 0.3ms
MEDIUM risk: under 0.5ms
HIGH risk: under 1ms
CRITICAL and EXTREME risk: under 2ms plus human escalation time

---

## Failure Policy

The default failure policy is fail-closed. If the authorization gate is unreachable, agents cannot execute. This is the correct default for any system handling consequential actions. Fail-open configurations are available for specified low-risk action categories where business continuity requires it. The configuration is explicit, documented, and recorded in every CAR.

Every autonomous action requires authority before execution.

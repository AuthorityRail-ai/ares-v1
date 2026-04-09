# ARES Compliance Guide

> **Version:** 1.0.0
> **Status:** Draft
> **License:** MIT
> **Last Updated:** 2026-03-10

---

## 1. Purpose

This document provides a self-certification checklist for platforms, agents, and enterprises seeking compliance with the AuthorityRail Execution Standard (ARES). It specifies what each compliance tier requires, how to validate your implementation, and how to submit for formal certification.

---

## 2. Compliance Tiers — Self-Certification Checklist

### ARES-BASIC

Minimum viable compliance. Suitable for sandbox, development, and internal tooling.

- [ ] **Authority Gate** is operational and reachable
- [ ] **Trust Registry** contains entries for all agents
- [ ] **CAR Issuer** produces cryptographically signed receipts
- [ ] All `ExecutionRequest` payloads include required fields: `agent_id`, `model_id`, `pattern_hash`, `action`, `environment`
- [ ] All `ExecutionAuthorization` responses include required fields: `decision`, `risk_score`, `permission_status`, `authority_signature`, `car_id`, `decision_receipt_url`
- [ ] The system fails closed — any error returns `DENY`
- [ ] DENY and ESCALATE decisions are respected (execution does not proceed)

---

### ARES-STANDARD

Recommended for staging and internal production. Adds pattern governance and reality permission.

All ARES-BASIC requirements, plus:

- [ ] **RAAS Engine** is operational and provides risk scoring
- [ ] **Pattern Provenance Engine** certifies pattern lineage
- [ ] **Reality Permission Layer** evaluates proposed outcomes
- [ ] **Decision Ledger** records every decision (ALLOW, DENY, ESCALATE)
- [x] **Agent Identity** resolved at Step 0 before pipeline execution
- [x] Unregistered, SUSPENDED, and TERMINATED agents are denied at Step 0
- [ ] Uncertified patterns result in `DENY`
- [ ] Reality Permission `DENY` results in execution `DENY`
- [ ] Reality Permission `ESCALATE` results in execution `ESCALATE`
- [ ] Decision records are immutable and append-only
- [ ] Decision records are retained for ≥ 30 days

---

### ARES-FULL

Complete stack. Required for regulated environments, insurance-backed deployments, and enterprise compliance.

All ARES-STANDARD requirements, plus:

- [ ] **Liability Ledger** records every decision with accountability fields
- [ ] **Public Verification Endpoint** is operational (`verify.authorityrail.io` equivalent)
- [ ] **Provenance Map** reconstructs the full decision path for any CAR
- [ ] **Agent Identity Registry** has entries for all agents with lifecycle management
- [ ] Liability records answer four questions: who authorized, what policy allowed, was risk assessed, was execution permitted
- [ ] Liability records are exportable in insurance-readable format (`authorityrail-insurance-export-v1`)
- [ ] Public verification confirms tamper-evidence via SHA-256 digest recomputation
- [ ] Agent identities are immutable after creation (only status is mutable)
- [ ] Terminated agents cannot be reactivated
- [ ] Decision records are retained for ≥ 90 days
- [ ] Availability target: ≥ 99.99%

---

## 3. Certification Levels

### AuthorityRail Certified Agent

An individual agent that meets the following criteria:

| Requirement | Details |
|---|---|
| Identity | Agent has a registered identity in the Agent Identity Registry |
| Compliance | Agent submits all execution requests through an ARES-aligned (pending audit) checkpoint |
| Enforcement | Agent respects DENY and ESCALATE decisions without bypass |
| Pattern | Agent's reasoning patterns are registered with Pattern Provenance |

**Self-certification steps:**
1. Register the agent identity via `POST /identity/register`
2. Verify the agent submits `ExecutionRequest` payloads for all actions
3. Confirm the agent halts on `DENY` and suspends on `ESCALATE`
4. Confirm pattern hashes are registered before execution

---

### AuthorityRail Certified Platform

A platform hosting multiple agents that meets the following criteria:

| Requirement | Details |
|---|---|
| Tier | Platform implements at least ARES-STANDARD |
| Coverage | All agents on the platform have registered identities |
| Verification | Platform exposes a public verification endpoint |
| Retention | Decision and liability records retained for ≥ 90 days |
| Monitoring | Platform monitors checkpoint availability and latency |

**Self-certification steps:**
1. Complete the ARES-STANDARD checklist above
2. Verify all agents have identities via `GET /identity/:agent_id`
3. Confirm public verification endpoint at `/verify/:car_id`
4. Confirm decision record retention policy (≥ 90 days)
5. Document monitoring and alerting procedures

---

### AuthorityRail Certified Enterprise

An enterprise deploying AI at scale that meets the following criteria:

| Requirement | Details |
|---|---|
| Tier | Enterprise implements ARES-FULL |
| Governance | All agents across all platforms are registered and governed |
| Liability | Records are exportable in insurance-readable format |
| Audit | Full decision audit trail available for regulatory review |
| Availability | Public verification endpoint operational at all times (99.99%) |
| Insurance | Liability classification (LOW/MEDIUM/HIGH/CRITICAL) is active |

**Self-certification steps:**
1. Complete the ARES-FULL checklist above
2. Verify `GET /liability/export/:agent_id` returns valid export data
3. Verify `GET /ledger/audit` returns paginated decision history
4. Confirm 99.99% availability SLA for verification endpoint
5. Document insurance export procedures and regulatory contact points

---

## 4. Validation Commands

Use these commands to validate your ARES implementation:

### Schema Validation

```bash
# Validate ExecutionRequest against ARES schema
curl -s -X POST https://your-checkpoint/authorize-execution \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "urn:authorityrail:agent:test-001",
    "model_id": "test-model",
    "pattern_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "action": "data.read",
    "environment": "sandbox"
  }' | jq .
```

### Fail-Closed Verification

```bash
# Verify fail-closed behavior — send to non-existent service
# Expected: decision=DENY with AR error code
curl -s -X POST https://your-checkpoint/authorize-execution \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "urn:authorityrail:agent:test-002",
    "model_id": "test-model",
    "pattern_hash": "0000000000000000000000000000000000000000000000000000000000000000",
    "action": "system.destructive",
    "environment": "production"
  }' | jq '.decision'
# Should output: "DENY"
```

### Tamper-Evidence Verification

```bash
# Verify a CAR receipt via public endpoint
curl -s https://your-verify-endpoint/verify/urn:authorityrail:car:YOUR_CAR_ID | jq .

# Check: tamper_evident should be true
# Check: valid should be true
```

### Insurance Export Validation

```bash
# Export liability records in insurance format
curl -s https://your-liability-endpoint/liability/export/urn:authorityrail:agent:YOUR_AGENT | jq .

# Check: format should be "authorityrail-insurance-export-v1"
# Check: each record contains who_authorized, policy_that_allowed,
#         risk_score_at_time, pattern_provenance, reality_permission
```

---

## 5. Submitting for Certification

### Self-Certification (Open)

1. Complete the appropriate checklist above
2. Run the validation commands against your deployment
3. Publish your compliance status in your platform documentation
4. Reference the ARES version: `ares-v1`

### Formal Certification (Future)

Formal third-party certification will be available in a future version of ARES. Contact `standards@authorityrail.io` for early access.

---

## 6. Error Code Reference

| Code | Name | Description |
|---|---|---|
| AR-001 | `schema_invalid` | Request payload fails schema validation |
| AR-003 | `agent_not_found` | Agent not registered in Trust Registry |
| AR-008 | `execution_denied` | Risk threshold exceeded |
| AR-013 | `car_failure` | CAR issuance failed |
| AR-015 | `service_unavailable` | Upstream service unreachable (fail-closed) |
| AR-050 | `pattern_uncertified` | Pattern not certified by Provenance Engine |
| AR-054 | `provenance_unavailable` | Pattern Provenance service unreachable |
| AR-060 | `reality_denied` | Reality Permission Layer denied execution |
| AR-061 | `reality_escalated` | Reality Permission Layer requires human review |
| AR-064 | `reality_unavailable` | Reality Permission service unreachable |
| AR-070 | `identity_not_registered` | Agent has no identity record |
| AR-071 | `agent_suspended` | Agent identity is SUSPENDED |
| AR-072 | `agent_terminated` | Agent identity is TERMINATED |
| AR-090 | `duplicate_record` | Duplicate CAR in Decision/Liability Ledger |

---

## 7. License

This compliance guide is published under the MIT License. Organizations may freely implement, distribute, and build upon this standard.

# Compliance Framework Mapping

**Standard:** ARES-v1
**Maintained by:** AuthorityRail Standards Foundation

---

## Overview

ARES-v1 is pre-mapped to four compliance frameworks. Every CAR records the specific framework requirements satisfied by the authorization decision. Compliance is enforced at execution time — not documented after the fact.

---

## NIST AI RMF

GOVERN — Policy Engine and Trust Registry enforce organizational AI governance requirements at the execution layer.
MAP — Risk Classification and Action Taxonomy map every agent action to its risk category before execution.
MEASURE — The 14-Layer Assessment and composite risk scoring provide continuous measurement of AI execution risk.
MANAGE — The CAR Ledger and Escalation Protocol provide the evidence trail required for ongoing risk management.

---

## ISO 42001

Documentation requirement — the CAR schema satisfies the documentation requirement for every AI decision made.
Accountability requirement — Agent Identity and Ed25519 signatures establish accountability for every action authorized or denied.
Traceability requirement — the append-only CAR Ledger provides complete traceability from action attempt to decision to outcome.
Audit requirement — the CAR Verification Protocol allows any CAR to be independently verified by an auditor.

---

## EU AI Act — High-Risk AI Systems

Article 9 Risk Management — the 14-Layer Assessment satisfies the risk management system requirement for high-risk AI systems.
Article 12 Record Keeping — the immutable CAR Ledger satisfies the logging and record-keeping requirements.
Article 14 Human Oversight — the ESCALATE decision and human approval flow satisfy the human oversight requirement.
Article 11 Technical Documentation — the CAR schema and compliance mapping satisfy the technical documentation requirement.

---

## SOC 2 Type II

The immutable CAR Ledger provides the continuous monitoring evidence required for SOC 2 Type II AI-related controls. Organizations using ARES-v1 reduce the manual documentation burden for AI-related SOC 2 controls significantly.

Every autonomous action requires authority before execution.

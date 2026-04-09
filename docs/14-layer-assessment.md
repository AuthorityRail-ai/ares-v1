# The 14-Layer Risk Assessment

**Standard:** ARES-v1
**Maintained by:** AuthorityRail Standards Foundation

---

## Overview

Every ARES-v1 authorization evaluation runs through 14 assessment layers in sequence. All 14 layers execute on every request. The composite output determines the risk score from 0 to 100 and the resulting decision.

---

## The 14 Layers

Layer 1 — Identity Check. Is the agent credentialed in the Trust Registry? An agent without a valid credential receives DENY on every action regardless of all other factors.

Layer 2 — Policy Rules. Does this action violate any configured policies? Policy violations produce immediate DENY decisions. Policy checks run before risk scoring.

Layer 3 — Context Authorization. Is this action appropriate in the current context? Context includes the instruction source, session state, and environmental conditions at the time of the request.

Layer 4 — Limits Check. Does this action exceed defined thresholds? Limits include financial amounts, data volume, action frequency, and resource scope.

Layer 5 — Risk Engine. What is the composite risk score for this action type, agent, and resource combination? The risk engine applies weighted scoring across all available signals.

Layer 6 — MAACS Consensus. For CRITICAL and EXTREME risk actions: do multiple authorized agents or approvers agree this action should proceed? No single agent can authorize a critical action unilaterally.

Layer 7 — Trust Registry. Is the agent credential current and unrevoked at the moment of this request? Credentials are verified in real time. A revoked credential produces immediate DENY.

Layer 8 — Compliance Mapping. Which regulatory frameworks apply to this action and resource? The compliance layer maps the decision to the relevant framework requirements and records the mapping in the CAR.

Layer 9 — Behavioral Analysis. Is this action consistent with the agent's established behavioral baseline? Anomalies from baseline increase the risk score and may trigger ESCALATE.

Layer 10 — Scope Verification. Is this action within the agent's authorized scope as defined in its credential? Out-of-scope actions produce immediate DENY regardless of risk score.

Layer 11 — Temporal Check. Is this action permitted at this time? Time-based policies restrict certain action types to defined windows.

Layer 12 — Resource Classification. What is the sensitivity classification of the target resource? Production systems, PHI, PII, financial APIs, and infrastructure receive elevated risk weights.

Layer 13 — Action Type Evaluation. Is this action type permitted for this agent class? Destructive, irreversible, and bulk actions receive elevated risk weights independent of resource classification.

Layer 14 — Cascade Risk Assessment. Could this action trigger downstream failures or chain reactions? Actions that could affect multiple systems or produce irreversible cascading effects receive maximum risk weight.

---

## Composite Scoring

Each layer contributes a weighted score to the composite risk score from 0 to 100. The weights are configurable per deployment. The default weights prioritize identity, policy, and resource classification. The composite score determines the risk classification and the default decision. Policy violations and identity failures produce immediate DENY decisions that bypass the composite scoring.

Every autonomous action requires authority before execution.

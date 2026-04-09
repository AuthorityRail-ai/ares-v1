# ARES-v1 — Agent Runtime Execution Standard

**Version:** 1.0.0 · **Status:** Published · **License:** MIT
**Publisher:** AuthorityRail Standards Foundation

---

> Every autonomous action requires authority before execution.

---

## What is ARES-v1?

ARES-v1 is the open protocol specification for **Execution Authority Infrastructure** — the cryptographic authorization standard that determines what AI agents are permitted to do before they do it.

It defines:
- The mandatory pre-execution authorization handshake for AI agent actions (AXAP)
- The **Certified Action Record (CAR)** format and cryptographic signing requirements
- The 14-layer risk scoring taxonomy for agent action classification
- The human escalation protocol for high-risk and critical decisions
- The audit ledger structure for tamper-evident decision recording

**Core principle:** Probability is not permission.

An AI agent that *might* do the right thing is not the same as an AI agent that is *authorized* to act. ARES-v1 enforces the difference.

---

## Install the reference implementation

```bash
npm install @authorityrail/axap
```

```javascript
const { AuthorityRail } = require('@authorityrail/axap');

const ar = new AuthorityRail({
  apiKey: process.env.AUTHORITYRAIL_KEY,
  agentId: 'your-agent-id'
});

const auth = await ar.authorize({
  action: 'wire_transfer',
  resource: 'banking_api',
  payload: { amount: 2400000, destination: 'ACCT-7721' }
});

if (auth.decision === 'ALLOW') {
  // proceed
} else {
  // blocked — auth.car_id contains the signed proof
  throw new Error('Action denied. CAR: ' + auth.car_id);
}
```

---

## The 3 Decisions

Every AI agent action evaluated under ARES-v1 receives exactly one of three decisions:

| Decision | Meaning | Outcome |
|---|---|---|
| `ALLOW` | Action is authorized | Execution proceeds. CAR issued. |
| `DENY` | Action is not authorized | Execution blocked. CAR issued. |
| `ESCALATE` | Action requires human approval | Execution held. CAR issued. Human notified. |

Every decision — including DENY — produces a signed CAR. This is the fundamental difference between authorization infrastructure and monitoring.

---

## The Three Laws of EAI

**Law I:** No autonomous action without authority.
The capability to act does not constitute permission to act.

**Law II:** Authorization must be enforced — not observed.
A system that watches what agents do after they act is not an authorization system. It is an incident log.

**Law III:** Every decision must be provable.
A log entry is not proof. A cryptographically signed Certified Action Record is proof.

---

## Specification

| Document | Description |
|---|---|
| [spec/ares-v1.0.md](spec/ares-v1.0.md) | Canonical v1.0.0 specification — immutable |
| [spec/car-schema.md](spec/car-schema.md) | CAR schema — field reference and decision types |
| [spec/car-schema.json](spec/car-schema.json) | Machine-readable JSON schema for validation |
| [docs/axap-handshake.md](docs/axap-handshake.md) | AXAP request and response protocol |
| [docs/14-layer-assessment.md](docs/14-layer-assessment.md) | The 14-layer risk assessment specification |
| [docs/compliance-mapping.md](docs/compliance-mapping.md) | NIST AI RMF, ISO 42001, EU AI Act, SOC 2 mapping |

---

## Compliance

ARES-v1 is pre-mapped to:

- **NIST AI RMF** — GOVERN, MAP, MEASURE, MANAGE functions
- **ISO 42001** — documentation, accountability, traceability, audit
- **EU AI Act** — Articles 9, 11, 12, 14 for high-risk AI systems
- **SOC 2 Type II** — continuous monitoring and audit trail

---

## Live demo

See ARES-v1 in action at [authorityrail.com](https://authorityrail.com)

Every agent action. Every decision. Cryptographically proved before execution.

---

## Reference implementation

**AuthorityRail** is the production reference implementation of ARES-v1.

- Website: [authorityrail.com](https://authorityrail.com)
- Docs: [docs.authorityrail.com](https://docs.authorityrail.com)
- npm: [@authorityrail/axap](https://www.npmjs.com/package/@authorityrail/axap)
- Contact: [hello@authorityrail.com](mailto:hello@authorityrail.com)

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose changes to the specification.

---

## License

MIT License — free to implement, fork, and build upon.

Published under the **AuthorityRail Standards Foundation**.

---

*Every autonomous action requires authority before execution.*

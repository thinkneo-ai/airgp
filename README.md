# AIRGP — Artificial Intelligence Runtime Governance Protocol

```
     _    ___ ____   ____ ____
    / \  |_ _|  _ \ / ___|  _ \
   / _ \  | || |_) | |  _| |_) |
  / ___ \ | ||  _ <| |_| |  __/
 /_/   \_\___|_| \_\\____|_|

 The Runtime Protocol for AI Governance
```

[![AIRGP Version](https://img.shields.io/badge/AIRGP-v1.0-blue)](spec/v1.0/AIRGP-v1.0.md)
[![License: CC-BY-ND-4.0](https://img.shields.io/badge/License-CC--BY--ND--4.0-lightgrey)](LICENSE)
[![Spec Status](https://img.shields.io/badge/Status-Published-green)]()
[![Schemas](https://img.shields.io/badge/schemas-7%20JSON%20Schema-orange)](schemas/)
[![Website](https://img.shields.io/badge/website-airgp.space-d4a017.svg)](https://airgp.space)
[![Steward](https://img.shields.io/badge/Steward-ThinkNEO-purple)](https://thinkneo.ai)

---

## What is AIRGP?

**AIRGP (Artificial Intelligence Runtime Governance Protocol)** is a wire-level protocol that defines how governance signals flow between AI runtimes, control planes, policy engines, and audit ledgers. It is the "TCP/IP for AI governance" — a vendor-neutral, transport-agnostic standard for runtime AI governance communication.

Today, organizations face a fragmented landscape of AI governance. Frameworks like NIST AI RMF and ISO/IEC 42001 tell you *what* governance outcomes to achieve. Regulations like the EU AI Act tell you *what* obligations to meet. But none of them specify *how* governance decisions are communicated, enforced, and recorded between software components at runtime. AIRGP fills this gap by defining the exact wire format for governance signals: policy evaluations, guardrail verdicts, audit records, cost accounting events, telemetry data, and provenance attestations.

AIRGP is a consolidator, not an inventor. It builds on and unifies the insights of prior work — including the OpenPort Protocol for tool-access security, Policy Cards for machine-readable policy specifications, and MI9 for agentic AI runtime governance — into a single interoperable wire protocol. AIRGP specifies JSON messages over HTTPS (required) with optional gRPC support, using the media type `application/airgp+json`, Ed25519 message signatures, and SHA-256 hash-chained audit trails.

---

## Architecture

AIRGP defines a four-layer architecture:

```
+---------------------------------------------------------------------+
|                        Application Code                              |
+---------------------------------------------------------------------+
        |                                          ^
        | AI call (e.g., LLM inference)            | Governed response
        v                                          |
+---------------------------------------------------------------------+
|                     RUNTIME ADAPTER (Layer 1)                        |
|  - Intercepts AI calls                                               |
|  - Attaches governance context                                       |
|  - Enforces verdicts on responses                                    |
+---------------------------------------------------------------------+
        |                ^                    |                ^
        | policy_eval    | verdict            | audit_record   | ack
        v                |                    v                |
+-------------------------------+   +-----------------------------+
|     CONTROL PLANE (Layer 2)   |   |    AUDIT LEDGER (Layer 4)   |
|  - Routes governance signals  |   |  - Hash-chained records     |
|  - Distributes policies       |   |  - Merkle tree anchoring    |
|  - Collects telemetry         |   |  - Tamper-evident storage   |
+-------------------------------+   +-----------------------------+
        |                ^
        | eval_request   | eval_result
        v                |
+-------------------------------+
|    POLICY ENGINE (Layer 3)    |
|  - Evaluates policy primitives|
|  - Produces verdicts          |
|  - Manages policy lifecycle   |
+-------------------------------+
```

---

## Conformance Levels

| Level | Target | Key Requirements |
|-------|--------|-----------------|
| **Basic** | Single-app deployments | Runtime adapter, guardrail verdicts, hash-chained audit, Ed25519 signing, fail-closed |
| **Enterprise** | Multi-tenant platforms | Full policy engine, all 7 primitive types, cost accounting, provenance, OpenTelemetry, Merkle anchoring |
| **Sovereign** | Regulated / government | Data residency enforcement, regulatory mapping, sovereign cloud, key escrow, cross-jurisdictional routing |

---

## Policy Primitive Types

AIRGP v1.0 defines seven policy primitive types:

1. **Guardrail** — Content filtering, toxicity detection, PII detection
2. **Routing Rule** — Provider/model selection, fallback chains
3. **Cost Cap** — Per-request, per-user, per-tenant budgets
4. **Data Boundary** — Geographic restrictions, data residency, PII masking
5. **Provenance Requirement** — Model attestation, training data lineage
6. **Rate Limit** — Requests/second, tokens/minute, concurrent sessions
7. **Compliance Mapping** — Regulatory requirement to technical control mapping

---

## Quick Links

| Resource | Description |
|----------|-------------|
| [SPEC.md](SPEC.md) | Full AIRGP v1.0 specification |
| [spec/v1.0/](spec/v1.0/AIRGP-v1.0.md) | Frozen versioned specification |
| [schemas/](schemas/) | JSON Schema definitions for all message types |
| [examples/](examples/) | Example JSON messages |
| [CONFORMANCE.md](CONFORMANCE.md) | How to claim AIRGP conformance |
| [GOVERNANCE.md](GOVERNANCE.md) | Steering Committee charter |
| [TRADEMARK.md](TRADEMARK.md) | AIRGP trademark usage policy |
| [research/](research/) | Landscape analysis and gap analysis |
| [governance/](governance/) | Detailed governance documents |

---

## Builds On

AIRGP gratefully acknowledges and builds upon prior work in runtime AI governance:

- **[OpenPort Protocol](https://arxiv.org/abs/2602.20196)** — Pioneered tool-access security for AI agents. AIRGP extends tool-access authorization to full governance signal coverage.
- **[Policy Cards](https://arxiv.org/abs/2510.24383)** — Defined machine-readable AI policy specifications. AIRGP adds runtime transport, evaluation semantics, and audit chain integration.
- **[MI9](https://arxiv.org/abs/2508.03858)** — Proposed a framework for agentic AI runtime governance. AIRGP provides the wire protocol that MI9's architecture requires.
- **[EU Apply AI Alliance](https://artificialintelligenceact.eu/)** — Recognized the need for execution-time governance. AIRGP implements this concept at the wire level.
- **[NIST AI RMF](https://www.nist.gov/artificial-intelligence/risk-management-framework)** — Organizational framework for AI risk management. AIRGP provides the runtime protocol for NIST's Measure and Manage functions.
- **[ISO/IEC 42001](https://www.iso.org/standard/81230.html)** — AI management system standard. AIRGP supports AIMS implementations with runtime governance signaling.
- **[MCP (Anthropic)](https://modelcontextprotocol.io/)** — Tool-use protocol for AI models. AIRGP is orthogonal: MCP governs tool use, AIRGP governs governance.

---

## Stewarded By

**ThinkNEO AI Technology Company Limited** (Hong Kong)

- [NVIDIA Inception Program](https://www.nvidia.com/en-us/startups/) member
- [Anthropic Partner Network](https://www.anthropic.com/) member

Author: **Fabio Bastos** — Founder, ThinkNEO

---

## License

The AIRGP specification text is licensed under **[CC-BY-ND-4.0](LICENSE)** (Creative Commons Attribution-NoDerivatives 4.0 International).

Implementations require a **Conformance License** — see [CONFORMANCE.md](CONFORMANCE.md) for details.

AIRGP is a trademark of ThinkNEO AI Technology Company Limited. See [TRADEMARK.md](TRADEMARK.md) for usage policy.

---

*Copyright 2026 ThinkNEO AI Technology Company Limited.*

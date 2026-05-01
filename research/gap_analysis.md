# Gap Analysis: Why AIRGP

## The Runtime Wire Protocol Gap

No existing standard unifies all six pillars of runtime AI governance into a single, interoperable wire protocol with a conformance licensing model.

## Coverage Matrix

| Effort | Runtime Wire Protocol | Policy Primitives | Audit Ledger | Observability | Identity/Attestation | Conformance License |
|--------|:---:|:---:|:---:|:---:|:---:|:---:|
| NIST AI RMF | - | Guidance | - | - | - | - |
| EU AI Act | - | Mandates | Required | - | - | Legal |
| ISO/IEC 42001 | - | Process | Process | - | - | Certification |
| IEEE 7000 | - | Design | - | - | - | - |
| OECD AI Principles | - | Principles | - | - | - | - |
| MCP | Tool calls | - | - | - | - | - |
| OWASP LLM Top 10 | - | Threats | - | - | - | - |
| OpenPort | Partial (tools) | Auth only | - | - | Partial | - |
| Policy Cards | - | JSON Schema | - | - | - | - |
| MI9 | Conceptual | Conceptual | Conceptual | - | - | - |
| Prefactor | Proprietary | Proprietary | Proprietary | Proprietary | - | - |
| **AIRGP v1.0** | **Yes** | **Yes (7 types)** | **Yes (hash-chained)** | **Yes (OTel)** | **Yes (Ed25519)** | **Yes (Bluetooth SIG model)** |

## Conclusion

AIRGP is the first open specification that unifies runtime AI governance across all six pillars: wire protocol, policy primitives, audit ledger, observability signals, identity/attestation, and conformance licensing. It is vendor-neutral, commercially licensable, and enterprise-ready.

AIRGP does not replace the organizational-layer standards (NIST, ISO, EU AI Act). It operationalizes them at the wire level — providing the runtime technical controls that make compliance with those standards measurable and auditable.

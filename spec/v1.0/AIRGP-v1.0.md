# AIRGP — Artificial Intelligence Runtime Governance Protocol

## Version 1.0

**Status:** Published
**Date:** 2026-05-01
**Steward:** ThinkNEO AI Technology Company Limited (Hong Kong)
**Author:** Fabio Bastos
**License:** CC-BY-ND-4.0

---

## Table of Contents

1. [Abstract](#1-abstract)
2. [Introduction & Motivation](#2-introduction--motivation)
3. [Relation to Prior Work](#3-relation-to-prior-work)
4. [Terminology](#4-terminology)
5. [Conformance Levels](#5-conformance-levels)
6. [Architecture](#6-architecture)
7. [Wire Protocol](#7-wire-protocol)
8. [Policy Primitives](#8-policy-primitives)
9. [Observability Signals](#9-observability-signals)
10. [Audit Ledger Format](#10-audit-ledger-format)
11. [Identity & Attestation](#11-identity--attestation)
12. [Security Considerations](#12-security-considerations)
13. [Privacy Considerations](#13-privacy-considerations)
14. [IANA Considerations](#14-iana-considerations)
15. [References](#15-references)

---

## 1. Abstract

The Artificial Intelligence Runtime Governance Protocol (AIRGP) defines a wire-level protocol for runtime AI governance. AIRGP specifies how governance signals — including policies, guardrail verdicts, telemetry data, audit records, cost accounting events, and provenance attestations — flow between AI runtimes, control planes, policy engines, and audit ledgers.

AIRGP addresses a critical gap in the current AI governance landscape: while organizational frameworks (NIST AI RMF, ISO/IEC 42001) and regulations (EU AI Act) establish what governance outcomes are required, no existing standard defines _how_ governance decisions are communicated, enforced, and recorded at the wire level during AI system execution.

The protocol specifies a JSON-based message format transported over HTTPS (REQUIRED) with optional gRPC support. All messages use the media type `application/airgp+json`. The protocol defines seven policy primitive types, a tamper-evident audit ledger format using SHA-256 hash chains, Ed25519 message signatures, and three conformance levels (Basic, Enterprise, Sovereign) to accommodate deployments ranging from single-application integrations to multi-jurisdictional sovereign cloud environments.

AIRGP is a consolidator protocol. It unifies and builds upon fragmented runtime governance efforts including the OpenPort Protocol, Policy Cards, and MI9, providing a single interoperable wire format for the runtime governance layer.

---

## 2. Introduction & Motivation

### 2.1 The Runtime Governance Gap

Artificial intelligence systems are increasingly deployed in contexts where governance is not merely desirable but legally mandated. The European Union's AI Act imposes obligations on providers and deployers of high-risk AI systems. The NIST AI Risk Management Framework provides voluntary guidance for managing AI risks across the AI lifecycle. ISO/IEC 42001 establishes requirements for AI management systems.

These instruments share a common limitation: they operate at the organizational, process, or documentation level. They specify _what_ governance outcomes must be achieved — transparency, accountability, robustness, fairness — but they do not specify _how_ governance decisions are communicated between software components at runtime. A compliance officer can mandate that all AI-generated content must pass a toxicity filter before reaching end users. But no existing standard defines the wire format for the message that carries the filter's verdict from the policy engine to the runtime adapter, or the audit record that proves the filter was applied.

This gap is not theoretical. Organizations deploying AI systems today face a fragmented landscape of vendor-specific governance implementations. A toxicity verdict from one vendor's guardrail system cannot be understood by another vendor's audit system. Cost accounting events from one provider's control plane are incompatible with another's billing infrastructure. Provenance attestations lack a common format, making cross-vendor model lineage verification impossible.

The result is governance silos: each AI deployment carries its own bespoke governance implementation, none of which interoperate, and none of which produce audit trails in a common format that regulators can inspect.

### 2.2 What "Wire-Level Protocol" Means for AI Governance

AIRGP is a wire-level protocol. This term requires precise definition in the context of AI governance.

A wire-level protocol specifies the exact format of messages exchanged between software components over a network or inter-process boundary. HTTP is a wire-level protocol: it specifies the byte-level structure of requests and responses. gRPC is a wire-level protocol: it specifies the Protobuf encoding of service calls. TCP/IP is a wire-level protocol suite: it specifies how packets are structured, addressed, and delivered.

By analogy, AIRGP specifies the exact JSON structure of governance messages exchanged between AI runtime components. When a policy engine evaluates a guardrail and produces a verdict, AIRGP defines exactly how that verdict is serialized, signed, chained to previous records, and delivered to the audit ledger. When a runtime adapter intercepts an AI call and needs to check cost limits, AIRGP defines the request/response format for that check.

This is distinct from what frameworks like NIST AI RMF provide. NIST AI RMF is an organizational framework: it tells you to "map, measure, manage" AI risks. AIRGP is the protocol that carries the measurement results and management decisions between software components. The relationship is analogous to that between an organization's security policy (framework) and TLS (wire protocol): the policy says "encrypt data in transit," and TLS is the wire protocol that implements it.

### 2.3 Design Principles

AIRGP is designed according to seven principles:

1. **Vendor-Neutral.** AIRGP does not favor any AI provider, cloud platform, or governance vendor. Any conformant implementation MUST interoperate with any other conformant implementation at the wire level.

2. **Transport-Agnostic.** While HTTPS is the REQUIRED transport, the message format is designed to be carried over any reliable transport. gRPC is specified as an OPTIONAL transport. Future versions MAY specify additional transports.

3. **Enterprise-Grade.** AIRGP is designed for production enterprise deployments. It includes cost accounting, multi-tenancy support, and integration with existing enterprise observability infrastructure (OpenTelemetry).

4. **Auditable.** Every governance decision produces a tamper-evident audit record. The hash-chained audit ledger format ensures that records cannot be modified or deleted without detection.

5. **Fail-Closed.** When a governance decision cannot be made (e.g., the policy engine is unreachable), AIRGP-conformant implementations MUST deny the AI operation by default. Fail-open behavior is explicitly prohibited at the Basic and Enterprise conformance levels and MUST be explicitly opted into at the Sovereign level with regulatory approval documentation.

6. **Incrementally Adoptable.** The three conformance levels allow organizations to adopt AIRGP incrementally. A Basic implementation requires only a runtime adapter, guardrail verdicts, and basic audit. Enterprise adds full policy engines and cost accounting. Sovereign adds jurisdictional controls.

7. **Consolidating.** AIRGP builds on prior work rather than replacing it. Existing policy formats, guardrail implementations, and audit systems can be wrapped with AIRGP adapters to participate in the protocol.

### 2.4 Scope

AIRGP governs the runtime behavior of AI systems. It does not specify:

- How AI models are trained (training-time governance is out of scope)
- How AI systems are evaluated before deployment (pre-deployment testing is out of scope)
- How organizations structure their AI governance teams (organizational governance is out of scope)
- How end users interact with AI systems (UX is out of scope)

AIRGP specifies exclusively the wire-level communication of governance signals between software components during AI system execution.

---

## 3. Relation to Prior Work

AIRGP is a consolidator protocol. It does not claim to invent the concepts of runtime AI governance, policy evaluation, or audit trails. Instead, it provides a unified wire format that allows existing approaches to interoperate. This section acknowledges and positions AIRGP relative to prior work.

### 3.1 OpenPort Protocol (arXiv:2602.20196)

The OpenPort Protocol addresses tool-access security for AI agents, defining how agents request and receive authorization to use external tools. AIRGP incorporates OpenPort's insight that tool access requires explicit runtime authorization, but extends it to cover all governance signals — not just tool access. An AIRGP Routing Rule policy primitive can express OpenPort-style tool authorization as one of seven primitive types. AIRGP and OpenPort are complementary: OpenPort can serve as the tool-access layer while AIRGP provides the broader governance envelope.

### 3.2 Policy Cards (arXiv:2510.24383)

Policy Cards define a machine-readable format for specifying AI policies. AIRGP's Policy Primitive concept is directly influenced by Policy Cards, extending the idea from a static specification format to a runtime evaluation protocol. A Policy Card can be compiled into one or more AIRGP Policy Primitives and evaluated by an AIRGP Policy Engine at runtime. AIRGP adds what Policy Cards lack: transport semantics, verdict formats, and audit chain integration.

### 3.3 MI9 (arXiv:2508.03858)

MI9 proposes a framework for agentic AI runtime governance, identifying the need for runtime monitoring, intervention, and audit of autonomous AI agents. AIRGP provides the wire protocol that MI9's framework requires. Where MI9 defines the conceptual architecture (monitor, intervene, audit), AIRGP specifies the exact message formats for each operation. MI9 implementations can use AIRGP as their communication layer.

### 3.4 EU Apply AI Alliance — Execution-Time AI Governance

The EU Apply AI Alliance has published guidance on execution-time AI governance, recognizing that compliance with the AI Act requires runtime mechanisms, not just documentation. AIRGP directly implements this guidance at the wire level. The Alliance's concept of "execution-time guardrails" maps to AIRGP's Guardrail Verdict signal type. The Alliance's concept of "runtime audit" maps to AIRGP's Audit Ledger format.

### 3.5 NIST AI Risk Management Framework

The NIST AI RMF (AI 100-1) provides organizational guidance for managing AI risks through four functions: Govern, Map, Measure, and Manage. AIRGP provides the runtime protocol for the Measure and Manage functions. AIRGP's telemetry signals implement Measure; AIRGP's policy enforcement implements Manage. The framework tells organizations what to measure; AIRGP specifies how the measurements flow between components.

### 3.6 ISO/IEC 42001

ISO/IEC 42001 specifies requirements for an AI management system (AIMS). AIRGP supports AIMS implementations by providing the technical protocol through which management system decisions are communicated and enforced at runtime. An AIMS requires "monitoring and measurement" (clause 9.1); AIRGP provides the wire format for these measurements.

### 3.7 EU AI Act

The EU AI Act (Regulation 2024/1689) establishes legal requirements for AI systems in the European Union. AIRGP's Compliance Mapping policy primitive type directly supports EU AI Act compliance by mapping regulatory requirements (e.g., Article 9 risk management, Article 13 transparency) to technical controls enforced at runtime. AIRGP does not interpret the law; it provides the protocol through which legal interpretations are translated into runtime controls.

### 3.8 Model Context Protocol (MCP)

Anthropic's Model Context Protocol (MCP) defines how AI models access tools and context. MCP and AIRGP are orthogonal: MCP governs tool use and context provision, while AIRGP governs the governance signals surrounding AI operations. An AIRGP Runtime Adapter can wrap MCP calls, applying governance policies before and after each MCP tool invocation. MCP provides the "what" (tool calls), and AIRGP provides the "whether" (governance verdicts on those calls).

---

## 4. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC 2119] [RFC 8174] when, and only when, they appear in ALL CAPITALS, as shown here.

**Control Plane.** The central coordination component in an AIRGP deployment. The Control Plane routes governance signals between Runtime Adapters, Policy Engines, and Audit Ledgers. It is responsible for signal routing, policy distribution, and health monitoring. A conformant AIRGP deployment MUST include exactly one logical Control Plane, which MAY be implemented as a distributed system.

**Policy Engine.** A component that evaluates Policy Primitives against runtime context to produce Guardrail Verdicts. A Policy Engine receives evaluation requests from the Control Plane, applies one or more Policy Primitives, and returns verdicts. Multiple Policy Engines MAY exist in a single deployment, each responsible for different policy domains.

**Runtime Adapter.** A software component that sits between an AI provider SDK (or API client) and the application code. The Runtime Adapter intercepts all AI calls (inference requests, tool invocations, embedding requests) and communicates with the Control Plane to enforce governance policies. Each AI provider integration requires its own Runtime Adapter.

**Audit Ledger.** A tamper-evident, append-only store of governance records. The Audit Ledger receives Audit Records from the Control Plane and stores them in a hash-chained format. The Audit Ledger MUST support query operations for compliance reporting and incident investigation.

**Governance Signal.** The generic term for any message exchanged via the AIRGP protocol. All Wire Messages are Governance Signals. The six signal types are: Policy Evaluation, Guardrail Verdict, Audit Record, Telemetry, Cost Event, and Provenance Attestation.

**Policy Primitive.** A single, atomic governance rule. Policy Primitives are the building blocks of governance policies. AIRGP defines seven primitive types: Guardrail, Routing Rule, Cost Cap, Data Boundary, Provenance Requirement, Rate Limit, and Compliance Mapping. Policy Primitives are evaluated by Policy Engines to produce verdicts.

**Guardrail Verdict.** The result of evaluating one or more Policy Primitives against a specific AI operation. A verdict is one of: `allow`, `deny`, `modify`, or `escalate`. Verdicts MUST include a reason string and SHOULD include confidence scores where applicable.

**Provenance Attestation.** A signed statement asserting facts about the provenance of an AI model, dataset, or output. Provenance Attestations include model identity, training data lineage, fine-tuning history, and deployment metadata. They are signed using Ed25519 keys and can be verified by any party holding the corresponding public key.

**Conformance Level.** One of three tiers (Basic, Enterprise, Sovereign) that define which AIRGP features an implementation supports. Conformance Levels are cumulative: Enterprise includes all Basic requirements, and Sovereign includes all Enterprise requirements.

**Wire Message.** A single JSON object conforming to the AIRGP message envelope schema, transmitted between AIRGP components. Every Wire Message includes a version identifier, unique message ID, timestamp, source, destination, signal type, payload, cryptographic signature, and hash chain link.

---

## 5. Conformance Levels

AIRGP defines three cumulative conformance levels. Each level builds upon the requirements of the previous level. Implementations MUST declare which conformance level they support and MUST implement all requirements of that level and all lower levels.

### 5.1 Basic Conformance

Basic conformance is the minimum level required to claim AIRGP conformance. It is designed for single-application deployments where fundamental runtime governance is needed.

An implementation claiming Basic conformance MUST:

1. Implement a Runtime Adapter for at least one AI provider SDK.
2. Intercept all AI inference requests passing through the adapter.
3. Support Guardrail policy primitives with `allow`/`deny` verdicts.
4. Generate Audit Records for all governance decisions.
5. Store Audit Records in a hash-chained format with SHA-256 links.
6. Sign all Wire Messages using Ed25519.
7. Expose the `/.well-known/airgp` endpoint over HTTPS.
8. Implement fail-closed behavior: if the Control Plane is unreachable, deny all AI operations.
9. Support the `application/airgp+json` media type.
10. Implement the AIRGP message envelope schema.

Basic conformance does not require a standalone Policy Engine (the Runtime Adapter MAY embed simple policy evaluation logic) or a standalone Audit Ledger (the adapter MAY write audit records to a local file in the required format).

### 5.2 Enterprise Conformance

Enterprise conformance extends Basic with full policy engine capabilities, cost accounting, provenance tracking, and enterprise observability integration.

An implementation claiming Enterprise conformance MUST, in addition to all Basic requirements:

1. Implement a standalone Policy Engine as a separate service.
2. Support all seven policy primitive types (Guardrail, Routing Rule, Cost Cap, Data Boundary, Provenance Requirement, Rate Limit, Compliance Mapping).
3. Implement cost accounting with per-request, per-user, and per-tenant budget tracking.
4. Generate and verify Provenance Attestations for all AI models in use.
5. Export telemetry in OpenTelemetry format with AIRGP semantic conventions.
6. Support batch mode for high-throughput deployments (>1000 signals/second).
7. Implement the Audit Ledger as a standalone service with query capabilities.
8. Support multi-tenancy in the Control Plane.
9. Implement periodic Merkle tree root publication for audit integrity verification.
10. Support key rotation for Ed25519 signing keys without service interruption.

### 5.3 Sovereign Conformance

Sovereign conformance extends Enterprise with jurisdictional controls required for deployments subject to data sovereignty laws, government regulations, or critical infrastructure requirements.

An implementation claiming Sovereign conformance MUST, in addition to all Enterprise requirements:

1. Enforce data residency constraints: all governance signals, audit records, and telemetry MUST remain within specified geographic boundaries.
2. Implement regulatory mapping: each Compliance Mapping primitive MUST reference specific regulatory clauses and produce evidence artifacts suitable for regulatory inspection.
3. Support sovereign cloud deployment: all AIRGP components MUST be deployable within a sovereign cloud environment with no external dependencies.
4. Implement key escrow: signing keys MUST support escrow to designated regulatory authorities upon lawful request.
5. Provide real-time regulatory reporting: the Audit Ledger MUST support streaming export of governance records to designated regulatory endpoints.
6. Support explicit fail-open override: unlike Basic and Enterprise, Sovereign deployments MAY implement fail-open behavior, but MUST document the regulatory approval for each fail-open rule and record all fail-open events in the Audit Ledger.
7. Implement cross-jurisdictional signal routing: when governance signals must cross jurisdictional boundaries, the Control Plane MUST apply the most restrictive policy set from all applicable jurisdictions.

---

## 6. Architecture

### 6.1 Overview

AIRGP defines a four-layer architecture. Each layer has defined responsibilities, interfaces, and interoperability requirements. The layers are: Runtime Adapter, Control Plane, Policy Engine, and Audit Ledger.

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
        | request        |                    |                |
        v                |                    v                |
+-------------------------------+   +-----------------------------+
|     CONTROL PLANE (Layer 2)   |   |    AUDIT LEDGER (Layer 4)   |
|  - Routes signals             |   |  - Stores hash-chained      |
|  - Distributes policies       |   |    records                  |
|  - Collects telemetry         |   |  - Merkle tree anchoring    |
|  - Enforces fail-closed       |   |  - Query interface          |
+-------------------------------+   +-----------------------------+
        |                ^
        | eval_request   | eval_result
        v                |
+-------------------------------+
|    POLICY ENGINE (Layer 3)    |
|  - Evaluates primitives       |
|  - Produces verdicts          |
|  - Manages policy lifecycle   |
+-------------------------------+
```

### 6.2 Layer 1: Runtime Adapter

The Runtime Adapter is the entry point for all AI operations into the AIRGP governance system. It sits between the application code and the AI provider SDK (e.g., OpenAI client library, Anthropic SDK, local model inference server).

**Responsibilities:**

1. **Interception.** The Runtime Adapter MUST intercept all AI calls before they reach the AI provider. This includes inference requests, embedding requests, tool invocations, and streaming responses. The interception mechanism is implementation-specific (e.g., SDK middleware, HTTP proxy, library wrapper).

2. **Context Attachment.** For each intercepted call, the Runtime Adapter MUST construct a governance context object containing: the requesting user or service identity, the target AI model, the operation type, the input content (or a hash thereof for privacy-sensitive deployments), tenant identifier, and any application-specific metadata.

3. **Policy Evaluation Request.** The Runtime Adapter MUST send a policy evaluation request to the Control Plane before forwarding the AI call to the provider. The request includes the governance context and the set of applicable policy primitive identifiers.

4. **Verdict Enforcement.** Upon receiving a verdict from the Control Plane, the Runtime Adapter MUST:
   - If `allow`: forward the AI call to the provider and return the response to the application.
   - If `deny`: return a governance denial response to the application without contacting the AI provider. The denial response MUST include the reason from the verdict.
   - If `modify`: apply the specified modifications to the AI call (e.g., redact PII from input, add system prompt constraints) before forwarding to the provider.
   - If `escalate`: hold the AI call pending human review. The adapter MUST implement a configurable timeout for escalation; if the timeout expires, the adapter MUST deny the call (fail-closed).

5. **Response Governance.** After receiving a response from the AI provider, the Runtime Adapter SHOULD apply post-response policy evaluation (e.g., output toxicity filtering, PII detection in responses). Post-response evaluation follows the same verdict enforcement logic.

6. **Telemetry Emission.** The Runtime Adapter MUST emit telemetry signals for every AI call, including latency, token count, provider, model, verdict, and cost (if available).

**Interfaces:**

- **Inbound:** Application AI calls (implementation-specific)
- **Outbound to Control Plane:** Policy evaluation requests, telemetry signals
- **Outbound to Application:** Governed AI responses, denial responses

### 6.3 Layer 2: Control Plane

The Control Plane is the central coordination hub of an AIRGP deployment. It routes governance signals, distributes policies, and enforces system-wide governance invariants.

**Responsibilities:**

1. **Signal Routing.** The Control Plane MUST route governance signals between Runtime Adapters, Policy Engines, and the Audit Ledger. Routing decisions are based on signal type, policy domain, and tenant context.

2. **Policy Distribution.** The Control Plane MUST maintain a registry of active Policy Primitives and distribute them to Policy Engines. Policy updates MUST be propagated to all affected Policy Engines within a configurable propagation deadline (default: 30 seconds).

3. **Health Monitoring.** The Control Plane MUST monitor the health of all connected components (Runtime Adapters, Policy Engines, Audit Ledger). If a Policy Engine becomes unavailable, the Control Plane MUST either route evaluation requests to an alternative Policy Engine or enforce fail-closed behavior.

4. **Fail-Closed Enforcement.** If the Control Plane itself cannot reach any Policy Engine for a given evaluation request, it MUST return a `deny` verdict to the Runtime Adapter. The Control Plane MUST NOT return an `allow` verdict when policy evaluation is impossible.

5. **Telemetry Aggregation.** The Control Plane MUST aggregate telemetry signals from all Runtime Adapters and make aggregated metrics available via the observability interface (Section 9).

6. **Multi-Tenancy.** At Enterprise and Sovereign conformance levels, the Control Plane MUST support multi-tenancy with isolated policy sets, audit trails, and telemetry streams per tenant.

**Interfaces:**

- **Inbound from Runtime Adapters:** Policy evaluation requests, telemetry signals
- **Outbound to Runtime Adapters:** Verdicts, policy updates
- **Outbound to Policy Engines:** Evaluation requests
- **Inbound from Policy Engines:** Evaluation results
- **Outbound to Audit Ledger:** Audit records

### 6.4 Layer 3: Policy Engine

The Policy Engine evaluates Policy Primitives against runtime context to produce verdicts.

**Responsibilities:**

1. **Primitive Evaluation.** The Policy Engine MUST accept evaluation requests containing governance context and a set of Policy Primitive identifiers. For each primitive, the engine MUST evaluate the context against the primitive's rules and produce a verdict.

2. **Verdict Aggregation.** When multiple primitives apply to a single AI operation, the Policy Engine MUST aggregate verdicts according to a configurable aggregation strategy. The default aggregation strategy is "most restrictive wins": if any primitive produces `deny`, the aggregate verdict is `deny`. The aggregation order is: `deny` > `escalate` > `modify` > `allow`.

3. **Policy Lifecycle Management.** The Policy Engine MUST support policy versioning, activation, deactivation, and rollback. Active policies MUST be identified by both their unique identifier and version number.

4. **Evaluation Caching.** The Policy Engine MAY cache evaluation results for identical governance contexts, provided that the cache TTL does not exceed the policy propagation deadline configured in the Control Plane.

**Interfaces:**

- **Inbound from Control Plane:** Evaluation requests, policy updates
- **Outbound to Control Plane:** Evaluation results (verdicts)

### 6.5 Layer 4: Audit Ledger

The Audit Ledger is the tamper-evident record of all governance decisions.

**Responsibilities:**

1. **Record Storage.** The Audit Ledger MUST accept and store Audit Records from the Control Plane. Records MUST be stored in append-only fashion; existing records MUST NOT be modified or deleted.

2. **Hash Chain Maintenance.** Each Audit Record MUST include a `prev_hash` field containing the SHA-256 hash of the previous record. This creates a tamper-evident chain that can be verified by any party with access to the ledger.

3. **Merkle Tree Anchoring.** At Enterprise and Sovereign conformance levels, the Audit Ledger MUST periodically compute a Merkle tree root over a batch of records and publish the root to a configurable anchor (e.g., a public blockchain, a trusted timestamping service, or a designated regulatory endpoint).

4. **Query Interface.** The Audit Ledger MUST expose a query interface supporting at minimum: lookup by record ID, lookup by time range, lookup by AI operation ID, lookup by verdict type, and lookup by policy primitive ID.

5. **Retention.** The Audit Ledger MUST support configurable retention periods. The minimum retention period is 365 days. At Sovereign conformance, the retention period MUST comply with applicable regulatory requirements, which MAY exceed 365 days.

**Interfaces:**

- **Inbound from Control Plane:** Audit records
- **Outbound:** Query results, Merkle tree roots, streaming export (Sovereign)

### 6.6 Deployment Topologies

AIRGP supports multiple deployment topologies:

1. **Embedded.** All four layers run within a single process. Suitable for development and testing. Basic conformance only.

2. **Sidecar.** The Runtime Adapter runs as a sidecar container alongside the application. The Control Plane, Policy Engine, and Audit Ledger run as separate services. Suitable for Kubernetes deployments. All conformance levels.

3. **Gateway.** The Runtime Adapter is implemented as an API gateway that proxies all AI traffic. The Control Plane, Policy Engine, and Audit Ledger run as backend services. Suitable for multi-application deployments. All conformance levels.

4. **Distributed.** All layers are distributed across multiple nodes and/or regions. Required for Sovereign conformance with multi-jurisdictional requirements.

---

## 7. Wire Protocol

### 7.1 Transport

AIRGP messages MUST be transportable over HTTPS (HTTP/1.1 or HTTP/2 with TLS 1.2 or later). HTTPS is the REQUIRED transport for all conformance levels.

gRPC is an OPTIONAL transport. Implementations MAY support gRPC as an alternative to HTTPS for internal communication between AIRGP components (e.g., between the Control Plane and Policy Engine). External-facing endpoints (e.g., the well-known endpoint) MUST support HTTPS regardless of whether gRPC is also supported.

### 7.2 Well-Known Endpoint

Every AIRGP-conformant component MUST expose its governance interface at the well-known URI:

```
https://{host}/.well-known/airgp
```

A GET request to this URI MUST return a JSON document describing the component's AIRGP capabilities:

```json
{
  "airgp_version": "1.0",
  "component_type": "control_plane",
  "conformance_level": "enterprise",
  "supported_signal_types": [
    "policy_evaluation",
    "guardrail_verdict",
    "audit_record",
    "telemetry",
    "cost_event",
    "provenance"
  ],
  "supported_primitives": [
    "guardrail",
    "routing_rule",
    "cost_cap",
    "data_boundary",
    "provenance_requirement",
    "rate_limit",
    "compliance_mapping"
  ],
  "endpoints": {
    "signals": "https://{host}/.well-known/airgp/signals",
    "policies": "https://{host}/.well-known/airgp/policies",
    "audit": "https://{host}/.well-known/airgp/audit",
    "health": "https://{host}/.well-known/airgp/health"
  }
}
```

### 7.3 Message Envelope

Every AIRGP Wire Message MUST conform to the following envelope structure:

```json
{
  "airgp_version": "1.0",
  "message_id": "019078a1-3c4d-7e8f-9a0b-1c2d3e4f5a6b",
  "timestamp": "2026-05-01T12:00:00.000000000Z",
  "source": {
    "component": "runtime_adapter",
    "instance_id": "ra-prod-us-east-001"
  },
  "destination": {
    "component": "control_plane",
    "instance_id": "cp-prod-us-east-001"
  },
  "signal_type": "policy_evaluation",
  "payload": {},
  "signature": "base64url-encoded-ed25519-signature",
  "prev_hash": "sha256:a1b2c3d4e5f6..."
}
```

**Field Definitions:**

- `airgp_version` (string, REQUIRED): The AIRGP protocol version. MUST be `"1.0"` for this specification.
- `message_id` (string, REQUIRED): A UUIDv7 uniquely identifying this message. UUIDv7 is chosen because it is time-sortable, which aids audit trail ordering.
- `timestamp` (string, REQUIRED): The time the message was created, in RFC 3339 format with nanosecond precision and UTC timezone. Example: `"2026-05-01T12:00:00.123456789Z"`.
- `source` (object, REQUIRED): Identifies the component sending the message.
  - `component` (string, REQUIRED): One of `"runtime_adapter"`, `"control_plane"`, `"policy_engine"`, `"audit_ledger"`.
  - `instance_id` (string, REQUIRED): A unique identifier for this instance of the component.
- `destination` (object, REQUIRED): Identifies the intended recipient. Same structure as `source`.
- `signal_type` (string, REQUIRED): One of `"policy_evaluation"`, `"guardrail_verdict"`, `"audit_record"`, `"telemetry"`, `"cost_event"`, `"provenance"`.
- `payload` (object, REQUIRED): The signal-type-specific payload. Structure varies by `signal_type` and is defined in subsequent sections.
- `signature` (string, REQUIRED): A base64url-encoded Ed25519 signature over the canonical JSON representation of all other fields (excluding `signature` itself). Canonical JSON is defined as JSON serialized with keys sorted lexicographically, no whitespace, and UTF-8 encoding.
- `prev_hash` (string, REQUIRED): The SHA-256 hash of the previous message in this component's hash chain, formatted as `"sha256:{hex-encoded-hash}"`. For the first message in a chain, this field MUST be `"sha256:0000000000000000000000000000000000000000000000000000000000000000"` (64 zero characters).

### 7.4 Request/Response Patterns

AIRGP defines two communication patterns:

**Synchronous Request/Response.** Used for policy evaluation requests where the Runtime Adapter must wait for a verdict before proceeding. The Runtime Adapter sends a `policy_evaluation` signal to the Control Plane and blocks until it receives a `guardrail_verdict` response. Implementations MUST enforce a configurable timeout (default: 5 seconds). If the timeout expires, the implementation MUST apply fail-closed behavior (deny the AI operation).

```
Runtime Adapter                    Control Plane                    Policy Engine
     |                                   |                               |
     |--- policy_evaluation request ---->|                               |
     |                                   |--- eval_request ------------->|
     |                                   |<-- eval_result (verdict) -----|
     |<-- guardrail_verdict response ----|                               |
     |                                   |--- audit_record ------------->| (Audit Ledger)
```

**Asynchronous Fire-and-Forget.** Used for telemetry signals, cost events, and audit records where the sender does not need to wait for a response. The sender transmits the signal and continues processing. The transport layer MUST provide at-least-once delivery guarantees. Implementations SHOULD use persistent queues or write-ahead logs to ensure signal delivery.

```
Runtime Adapter                    Control Plane                    Audit Ledger
     |                                   |                               |
     |--- telemetry signal ------------->|                               |
     |   (fire-and-forget)               |--- audit_record ------------->|
     |                                   |   (fire-and-forget)           |
```

### 7.5 Error Handling

AIRGP defines the following error response format:

```json
{
  "airgp_version": "1.0",
  "message_id": "...",
  "timestamp": "...",
  "source": { "component": "control_plane", "instance_id": "..." },
  "destination": { "component": "runtime_adapter", "instance_id": "..." },
  "signal_type": "guardrail_verdict",
  "payload": {
    "verdict": "deny",
    "reason": "policy_engine_unavailable",
    "error": {
      "code": "AIRGP-5001",
      "message": "All policy engines are unreachable",
      "retry_after_ms": 5000
    }
  },
  "signature": "...",
  "prev_hash": "..."
}
```

**Error Codes:**

| Code | Meaning | Action |
|------|---------|--------|
| AIRGP-4001 | Malformed message | Reject, do not retry |
| AIRGP-4002 | Unknown signal type | Reject, do not retry |
| AIRGP-4003 | Invalid signature | Reject, log security event |
| AIRGP-4004 | Hash chain broken | Reject, trigger integrity alert |
| AIRGP-4005 | Unknown policy primitive | Reject, do not retry |
| AIRGP-4006 | Unauthorized component | Reject, log security event |
| AIRGP-5001 | Policy engine unavailable | Deny (fail-closed), retry after delay |
| AIRGP-5002 | Audit ledger unavailable | Deny (fail-closed), buffer locally |
| AIRGP-5003 | Internal error | Deny (fail-closed), retry after delay |
| AIRGP-5004 | Evaluation timeout | Deny (fail-closed), retry with backoff |
| AIRGP-5005 | Rate limited | Deny, retry after `retry_after_ms` |

All 4xxx errors indicate client-side issues and MUST NOT be retried without modification. All 5xxx errors indicate server-side issues. Implementations SHOULD implement exponential backoff for retries, with a maximum of 3 retry attempts.

### 7.6 Batch Mode

For high-throughput deployments, AIRGP supports batch mode. A batch message wraps multiple signals in a single HTTP request:

```json
{
  "airgp_version": "1.0",
  "message_id": "...",
  "timestamp": "...",
  "source": { "component": "runtime_adapter", "instance_id": "..." },
  "destination": { "component": "control_plane", "instance_id": "..." },
  "signal_type": "batch",
  "payload": {
    "signals": [
      { "signal_type": "telemetry", "payload": { ... } },
      { "signal_type": "cost_event", "payload": { ... } },
      { "signal_type": "telemetry", "payload": { ... } }
    ]
  },
  "signature": "...",
  "prev_hash": "..."
}
```

Batch mode is REQUIRED at Enterprise and Sovereign conformance levels for telemetry and cost event signals when the signal rate exceeds 100 signals per second. The maximum batch size is 1000 signals per batch message. Each signal within a batch MUST be individually valid according to its signal type schema. The batch signature covers the entire batch, but each signal within the batch does NOT require an individual signature.

### 7.7 Content Negotiation

AIRGP endpoints MUST accept and produce `application/airgp+json` as the Content-Type. Requests with other Content-Types MUST be rejected with HTTP 415 Unsupported Media Type. Responses MUST always use `application/airgp+json`.

Implementations MAY support content encoding (e.g., gzip, br) for bandwidth optimization. If content encoding is used, standard HTTP content negotiation (Accept-Encoding / Content-Encoding) applies.

---

## 8. Policy Primitives

Policy Primitives are the building blocks of AIRGP governance policies. Each primitive defines a single governance rule with clear evaluation semantics and verdict structure. AIRGP v1.0 defines seven primitive types.

### 8.1 Guardrail

The Guardrail primitive defines content-level governance rules: toxicity filtering, PII detection, content classification, and output safety checks.

**Schema:**

```json
{
  "primitive_type": "guardrail",
  "primitive_id": "grd-001",
  "version": 1,
  "name": "Toxicity Filter",
  "description": "Blocks AI outputs with toxicity score above threshold",
  "enabled": true,
  "config": {
    "guardrail_type": "toxicity",
    "direction": "output",
    "threshold": 0.7,
    "action_on_trigger": "deny",
    "categories": ["hate_speech", "harassment", "self_harm", "sexual_content", "violence"],
    "exemptions": []
  }
}
```

**Evaluation Semantics:** The Policy Engine MUST evaluate the specified content (input, output, or both based on `direction`) against the guardrail configuration. If the content triggers the guardrail (e.g., toxicity score exceeds threshold), the engine MUST produce a verdict matching the `action_on_trigger` value. The `direction` field MUST be one of `"input"`, `"output"`, or `"both"`.

**Verdict Structure:**

```json
{
  "primitive_id": "grd-001",
  "verdict": "deny",
  "reason": "Output toxicity score 0.85 exceeds threshold 0.7",
  "details": {
    "score": 0.85,
    "threshold": 0.7,
    "triggered_categories": ["harassment"],
    "content_hash": "sha256:..."
  }
}
```

### 8.2 Routing Rule

The Routing Rule primitive governs AI provider and model selection, including fallback chains and load balancing.

**Schema:**

```json
{
  "primitive_type": "routing_rule",
  "primitive_id": "rr-001",
  "version": 1,
  "name": "Production Model Routing",
  "description": "Routes production traffic to approved models with fallback",
  "enabled": true,
  "config": {
    "routing_type": "model_selection",
    "primary": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-20250514",
      "region": "us-east-1"
    },
    "fallback_chain": [
      {
        "provider": "openai",
        "model": "gpt-4o",
        "region": "us-east-1",
        "condition": "primary_unavailable"
      }
    ],
    "blocked_providers": [],
    "blocked_models": [],
    "require_model_attestation": true
  }
}
```

**Evaluation Semantics:** The Policy Engine MUST verify that the requested AI operation targets an allowed provider and model. If the target is blocked, the verdict MUST be `deny`. If the target is the primary and it is unavailable, the engine MUST select the first available fallback. If `require_model_attestation` is true, the engine MUST verify that a valid Provenance Attestation exists for the selected model before returning an `allow` verdict.

**Verdict Structure:**

```json
{
  "primitive_id": "rr-001",
  "verdict": "modify",
  "reason": "Primary model unavailable, routed to fallback",
  "details": {
    "original_provider": "anthropic",
    "original_model": "claude-sonnet-4-20250514",
    "routed_provider": "openai",
    "routed_model": "gpt-4o",
    "fallback_reason": "primary_unavailable"
  }
}
```

### 8.3 Cost Cap

The Cost Cap primitive enforces budget limits at the per-request, per-user, and per-tenant levels.

**Schema:**

```json
{
  "primitive_type": "cost_cap",
  "primitive_id": "cc-001",
  "version": 1,
  "name": "Tenant Monthly Budget",
  "description": "Enforces monthly cost cap per tenant",
  "enabled": true,
  "config": {
    "cap_type": "tenant",
    "budget_amount": 10000.00,
    "budget_currency": "USD",
    "budget_period": "monthly",
    "soft_limit_percentage": 80,
    "hard_limit_percentage": 100,
    "action_on_soft_limit": "escalate",
    "action_on_hard_limit": "deny",
    "per_request_max": 5.00,
    "exclude_operations": ["embedding"]
  }
}
```

**Evaluation Semantics:** The Policy Engine MUST track cumulative costs for the specified scope (request, user, or tenant) over the specified period. Before each AI operation, the engine MUST estimate the operation's cost based on the target model's pricing. If the estimated cumulative cost would exceed the soft limit, the engine MUST produce the `action_on_soft_limit` verdict. If it would exceed the hard limit, the engine MUST produce the `action_on_hard_limit` verdict. Per-request maximum is evaluated independently of cumulative limits.

**Verdict Structure:**

```json
{
  "primitive_id": "cc-001",
  "verdict": "deny",
  "reason": "Tenant monthly budget exceeded",
  "details": {
    "budget_amount": 10000.00,
    "current_spend": 9850.00,
    "estimated_cost": 200.00,
    "projected_total": 10050.00,
    "limit_type": "hard",
    "budget_period": "monthly",
    "period_start": "2026-05-01T00:00:00Z",
    "period_end": "2026-05-31T23:59:59Z"
  }
}
```

### 8.4 Data Boundary

The Data Boundary primitive enforces geographic restrictions, data residency requirements, and PII masking.

**Schema:**

```json
{
  "primitive_type": "data_boundary",
  "primitive_id": "db-001",
  "version": 1,
  "name": "EU Data Residency",
  "description": "Ensures AI operations for EU users stay within EU boundaries",
  "enabled": true,
  "config": {
    "boundary_type": "geographic",
    "allowed_regions": ["eu-west-1", "eu-central-1", "eu-north-1"],
    "denied_regions": ["us-*", "cn-*"],
    "pii_handling": "mask",
    "pii_categories": ["email", "phone", "national_id", "address"],
    "data_classification": "confidential",
    "cross_border_policy": "deny"
  }
}
```

**Evaluation Semantics:** The Policy Engine MUST verify that the AI operation will be processed within an allowed region. Region information MUST be obtained from the Routing Rule verdict or the AI provider's service metadata. If the operation would be processed in a denied region, the verdict MUST be `deny`. If PII handling is `"mask"`, the engine MUST instruct the Runtime Adapter to mask detected PII categories before sending data to the AI provider. If `cross_border_policy` is `"deny"`, the engine MUST deny operations where input data would cross the specified boundary.

**Verdict Structure:**

```json
{
  "primitive_id": "db-001",
  "verdict": "modify",
  "reason": "PII detected, masking applied before processing",
  "details": {
    "processing_region": "eu-west-1",
    "boundary_compliant": true,
    "pii_detected": ["email", "phone"],
    "pii_action": "mask",
    "masked_count": 3
  }
}
```

### 8.5 Provenance Requirement

The Provenance Requirement primitive mandates that AI models and their outputs carry verifiable provenance information.

**Schema:**

```json
{
  "primitive_type": "provenance_requirement",
  "primitive_id": "pr-001",
  "version": 1,
  "name": "Model Attestation Required",
  "description": "Requires valid provenance attestation for all production models",
  "enabled": true,
  "config": {
    "require_model_attestation": true,
    "require_training_data_lineage": false,
    "require_fine_tuning_history": true,
    "attestation_max_age_days": 90,
    "trusted_attestation_issuers": ["thinkneo.ai", "anthropic.com", "openai.com"],
    "require_output_watermark": false
  }
}
```

**Evaluation Semantics:** The Policy Engine MUST verify that the target AI model has a valid Provenance Attestation meeting all specified requirements. The attestation MUST be signed by a trusted issuer, MUST NOT be older than the specified maximum age, and MUST include all required provenance elements (model attestation, training data lineage, fine-tuning history). If any requirement is not met, the verdict MUST be `deny`.

**Verdict Structure:**

```json
{
  "primitive_id": "pr-001",
  "verdict": "allow",
  "reason": "Model provenance verified",
  "details": {
    "model_id": "claude-sonnet-4-20250514",
    "attestation_issuer": "anthropic.com",
    "attestation_date": "2026-03-15T00:00:00Z",
    "attestation_age_days": 47,
    "verified_elements": ["model_attestation", "fine_tuning_history"]
  }
}
```

### 8.6 Rate Limit

The Rate Limit primitive enforces throughput limits to prevent abuse and manage resource allocation.

**Schema:**

```json
{
  "primitive_type": "rate_limit",
  "primitive_id": "rl-001",
  "version": 1,
  "name": "User Rate Limit",
  "description": "Limits AI requests per user",
  "enabled": true,
  "config": {
    "limit_scope": "user",
    "limits": [
      { "metric": "requests", "value": 100, "window": "minute" },
      { "metric": "tokens", "value": 100000, "window": "minute" },
      { "metric": "concurrent_sessions", "value": 5, "window": "none" }
    ],
    "action_on_limit": "deny",
    "burst_allowance_percentage": 20,
    "priority_override": false
  }
}
```

**Evaluation Semantics:** The Policy Engine MUST track usage metrics for the specified scope (user, tenant, application) and evaluate them against the configured limits. Each limit is checked independently. If any limit is exceeded (accounting for burst allowance), the verdict MUST be `deny` with a `retry_after_ms` value indicating when the operation can be retried. If `priority_override` is true, users with an elevated priority claim MAY be exempted from rate limits, but the exemption MUST be logged as an audit record.

**Verdict Structure:**

```json
{
  "primitive_id": "rl-001",
  "verdict": "deny",
  "reason": "Token rate limit exceeded",
  "details": {
    "limit_metric": "tokens",
    "limit_value": 100000,
    "current_value": 98500,
    "requested_value": 4000,
    "window": "minute",
    "retry_after_ms": 12000,
    "burst_allowance_remaining": 0
  }
}
```

### 8.7 Compliance Mapping

The Compliance Mapping primitive maps regulatory requirements to technical controls, providing a formal link between legal obligations and runtime enforcement.

**Schema:**

```json
{
  "primitive_type": "compliance_mapping",
  "primitive_id": "cm-001",
  "version": 1,
  "name": "EU AI Act Article 14 — Human Oversight",
  "description": "Maps Article 14 human oversight requirement to runtime controls",
  "enabled": true,
  "config": {
    "regulation": "eu_ai_act",
    "regulation_version": "2024/1689",
    "article": "14",
    "requirement_summary": "High-risk AI systems shall be designed to allow human oversight",
    "technical_controls": [
      {
        "control_type": "escalation",
        "trigger": "high_risk_classification",
        "action": "escalate",
        "escalation_target": "human_reviewer",
        "timeout_seconds": 3600,
        "timeout_action": "deny"
      }
    ],
    "evidence_requirements": [
      "escalation_count",
      "human_review_response_time",
      "override_justification"
    ],
    "audit_frequency": "continuous"
  }
}
```

**Evaluation Semantics:** The Policy Engine MUST evaluate the AI operation against the specified technical controls. If the operation matches a trigger condition (e.g., the AI system is classified as high-risk), the engine MUST apply the corresponding action. The engine MUST collect evidence artifacts specified in `evidence_requirements` and include them in the audit record. At Sovereign conformance level, compliance mapping verdicts MUST include references to specific regulatory clauses and produce evidence artifacts suitable for regulatory inspection.

**Verdict Structure:**

```json
{
  "primitive_id": "cm-001",
  "verdict": "escalate",
  "reason": "High-risk AI operation requires human oversight per EU AI Act Article 14",
  "details": {
    "regulation": "eu_ai_act",
    "article": "14",
    "risk_classification": "high",
    "escalation_target": "human_reviewer",
    "escalation_timeout_seconds": 3600,
    "evidence_collected": ["escalation_count", "request_context"]
  }
}
```

---

## 9. Observability Signals

### 9.1 Required Telemetry Shape

Every AIRGP Runtime Adapter MUST emit telemetry signals for each AI operation. The telemetry signal payload MUST include the following fields:

```json
{
  "operation_id": "op-uuid-v7",
  "operation_type": "inference",
  "provider": "anthropic",
  "model": "claude-sonnet-4-20250514",
  "start_time": "2026-05-01T12:00:00.000000000Z",
  "end_time": "2026-05-01T12:00:01.234567890Z",
  "latency_ms": 1234,
  "input_tokens": 150,
  "output_tokens": 500,
  "total_tokens": 650,
  "estimated_cost_usd": 0.0045,
  "verdict": "allow",
  "verdict_latency_ms": 12,
  "policies_evaluated": ["grd-001", "cc-001", "rl-001"],
  "user_id": "user-hash-or-id",
  "tenant_id": "tenant-001",
  "application_id": "app-001",
  "status": "success",
  "error_code": null,
  "metadata": {}
}
```

**Field Requirements:**

- `operation_id` (string, REQUIRED): UUIDv7 identifying the AI operation.
- `operation_type` (string, REQUIRED): One of `"inference"`, `"embedding"`, `"tool_invocation"`, `"fine_tuning"`, `"image_generation"`, `"speech_synthesis"`, `"speech_recognition"`.
- `provider` (string, REQUIRED): The AI provider identifier.
- `model` (string, REQUIRED): The model identifier.
- `start_time` (string, REQUIRED): RFC 3339 nanosecond UTC timestamp of operation start.
- `end_time` (string, REQUIRED): RFC 3339 nanosecond UTC timestamp of operation end.
- `latency_ms` (number, REQUIRED): Total operation latency in milliseconds.
- `input_tokens` (integer, REQUIRED for text operations): Input token count.
- `output_tokens` (integer, REQUIRED for text operations): Output token count.
- `total_tokens` (integer, REQUIRED for text operations): Total token count.
- `estimated_cost_usd` (number, OPTIONAL): Estimated cost in USD.
- `verdict` (string, REQUIRED): The governance verdict applied.
- `verdict_latency_ms` (number, REQUIRED): Time to obtain governance verdict.
- `policies_evaluated` (array, REQUIRED): List of policy primitive IDs evaluated.
- `user_id` (string, OPTIONAL): Pseudonymized user identifier.
- `tenant_id` (string, REQUIRED at Enterprise/Sovereign): Tenant identifier.
- `application_id` (string, REQUIRED): Application identifier.
- `status` (string, REQUIRED): One of `"success"`, `"denied"`, `"error"`, `"timeout"`.
- `error_code` (string, OPTIONAL): AIRGP error code if status is `"error"`.
- `metadata` (object, OPTIONAL): Application-specific metadata.

### 9.2 OpenTelemetry Semantic Conventions

At Enterprise and Sovereign conformance levels, AIRGP implementations MUST export telemetry using OpenTelemetry (OTel) semantic conventions. The following AIRGP-specific attributes are defined:

**Span Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| `airgp.version` | string | AIRGP protocol version |
| `airgp.operation.id` | string | Operation UUIDv7 |
| `airgp.operation.type` | string | Operation type enum |
| `airgp.provider` | string | AI provider identifier |
| `airgp.model` | string | Model identifier |
| `airgp.verdict` | string | Governance verdict |
| `airgp.verdict.latency_ms` | int | Verdict latency |
| `airgp.policies.evaluated` | string[] | Policy IDs evaluated |
| `airgp.tokens.input` | int | Input token count |
| `airgp.tokens.output` | int | Output token count |
| `airgp.tokens.total` | int | Total token count |
| `airgp.cost.estimated_usd` | double | Estimated cost |
| `airgp.tenant.id` | string | Tenant identifier |
| `airgp.application.id` | string | Application identifier |
| `airgp.conformance.level` | string | Conformance level |

**Required Spans:**

Every AI operation governed by AIRGP MUST produce the following spans:

1. `airgp.governance` — The root span covering the entire governed operation.
2. `airgp.policy_evaluation` — Child span covering policy evaluation (from request to verdict).
3. `airgp.ai_operation` — Child span covering the actual AI provider call.
4. `airgp.audit_record` — Child span covering audit record creation and storage.

### 9.3 Metrics

AIRGP implementations MUST expose the following metrics:

| Metric Name | Type | Unit | Description |
|-------------|------|------|-------------|
| `airgp.operations.total` | Counter | operations | Total AI operations processed |
| `airgp.operations.denied` | Counter | operations | Operations denied by governance |
| `airgp.operations.escalated` | Counter | operations | Operations escalated for review |
| `airgp.operations.latency` | Histogram | ms | End-to-end operation latency |
| `airgp.verdict.latency` | Histogram | ms | Policy evaluation latency |
| `airgp.tokens.consumed` | Counter | tokens | Total tokens consumed |
| `airgp.cost.accumulated` | Counter | USD | Accumulated cost |
| `airgp.audit.records` | Counter | records | Audit records created |
| `airgp.errors.total` | Counter | errors | Total errors by error code |
| `airgp.policy_engine.health` | Gauge | boolean | Policy engine availability |

### 9.4 Log Format

AIRGP governance events SHOULD be logged in structured JSON format. Each log entry MUST include:

```json
{
  "timestamp": "2026-05-01T12:00:00.000Z",
  "level": "INFO",
  "component": "runtime_adapter",
  "instance_id": "ra-prod-us-east-001",
  "operation_id": "op-uuid-v7",
  "event": "verdict_received",
  "verdict": "allow",
  "policies": ["grd-001"],
  "latency_ms": 12,
  "message": "Governance verdict received: allow"
}
```

Log levels MUST follow standard severity: `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`. All governance denial events MUST be logged at `WARN` level or above. All error events MUST be logged at `ERROR` level. All security events (invalid signatures, broken hash chains) MUST be logged at `FATAL` level.

---

## 10. Audit Ledger Format

### 10.1 Design Principles

The AIRGP Audit Ledger is designed to be tamper-evident, append-only, and independently verifiable. These properties are achieved through three mechanisms:

1. **Hash Chaining.** Each audit record contains the SHA-256 hash of the previous record, creating a chain that detects any insertion, deletion, or modification of records.

2. **Cryptographic Signing.** Each audit record is signed with Ed25519, providing non-repudiation: the signer cannot deny having produced the record.

3. **Merkle Tree Anchoring.** Periodically, a Merkle tree is computed over a batch of records, and the root hash is published to an external anchor. This provides a compact proof that the entire batch existed at a specific time and has not been altered.

### 10.2 Audit Record Structure

Each audit record MUST conform to the following structure:

```json
{
  "record_id": "ar-uuid-v7",
  "record_type": "governance_decision",
  "timestamp": "2026-05-01T12:00:01.234567890Z",
  "operation_id": "op-uuid-v7",
  "sequence_number": 42,
  "governance_context": {
    "user_id": "user-hash",
    "tenant_id": "tenant-001",
    "application_id": "app-001",
    "operation_type": "inference",
    "provider": "anthropic",
    "model": "claude-sonnet-4-20250514",
    "input_hash": "sha256:...",
    "output_hash": "sha256:..."
  },
  "policies_evaluated": [
    {
      "primitive_id": "grd-001",
      "primitive_type": "guardrail",
      "version": 1,
      "verdict": "allow",
      "reason": "Content within acceptable parameters",
      "evaluation_time_ms": 8
    }
  ],
  "aggregate_verdict": "allow",
  "aggregate_reason": "All policies passed",
  "telemetry_summary": {
    "latency_ms": 1234,
    "input_tokens": 150,
    "output_tokens": 500,
    "estimated_cost_usd": 0.0045
  },
  "prev_hash": "sha256:a1b2c3d4...",
  "record_hash": "sha256:e5f6a7b8...",
  "signature": "base64url-encoded-ed25519-signature",
  "metadata": {}
}
```

**Field Definitions:**

- `record_id` (string, REQUIRED): UUIDv7 uniquely identifying this audit record.
- `record_type` (string, REQUIRED): One of `"governance_decision"`, `"policy_change"`, `"system_event"`, `"security_event"`, `"escalation_resolution"`.
- `timestamp` (string, REQUIRED): RFC 3339 nanosecond UTC.
- `operation_id` (string, REQUIRED for `governance_decision`): The AI operation this record pertains to.
- `sequence_number` (integer, REQUIRED): Monotonically increasing sequence number within this ledger. MUST NOT have gaps.
- `governance_context` (object, REQUIRED for `governance_decision`): The context of the governed AI operation.
- `policies_evaluated` (array, REQUIRED for `governance_decision`): Details of each policy evaluation.
- `aggregate_verdict` (string, REQUIRED for `governance_decision`): The final verdict applied.
- `aggregate_reason` (string, REQUIRED): Human-readable explanation.
- `telemetry_summary` (object, OPTIONAL): Summary telemetry for the operation.
- `prev_hash` (string, REQUIRED): SHA-256 hash of the previous record in the chain.
- `record_hash` (string, REQUIRED): SHA-256 hash of this record (computed over all fields except `record_hash` and `signature`).
- `signature` (string, REQUIRED): Ed25519 signature over the `record_hash`.
- `metadata` (object, OPTIONAL): Implementation-specific metadata.

### 10.3 Hash Chain Verification

To verify the integrity of the audit chain, a verifier MUST:

1. Retrieve the records in sequence number order.
2. For each record, compute the SHA-256 hash over all fields except `record_hash` and `signature`, using canonical JSON serialization.
3. Verify that the computed hash matches `record_hash`.
4. Verify that `prev_hash` matches the `record_hash` of the previous record.
5. Verify the Ed25519 `signature` over the `record_hash` using the signer's public key.
6. Verify that `sequence_number` values are contiguous with no gaps.

If any verification step fails, the chain is considered compromised, and the verifier MUST raise an integrity alert.

### 10.4 Merkle Tree Anchoring

At Enterprise and Sovereign conformance levels, the Audit Ledger MUST periodically compute a Merkle tree over a batch of audit records.

**Batch Parameters:**
- Batch size: configurable, default 1000 records.
- Batch timeout: configurable, default 1 hour. If fewer than batch size records are accumulated within the timeout, a batch is created with whatever records are available.

**Merkle Tree Construction:**
1. Collect the `record_hash` values of all records in the batch, ordered by `sequence_number`.
2. Compute the Merkle tree using SHA-256 as the hash function.
3. The Merkle root is the root hash of the tree.

**Anchoring:**
The Merkle root MUST be published to at least one anchor. Acceptable anchors include:
- A trusted timestamping service (e.g., RFC 3161 TSA)
- A public or consortium blockchain
- A designated regulatory endpoint (Sovereign conformance)
- A signed, publicly accessible transparency log

The anchor publication record MUST include: the Merkle root, the batch sequence number range, the publication timestamp, and the anchor identifier.

### 10.5 Retention

The Audit Ledger MUST support configurable retention periods. The minimum retention period for all conformance levels is 365 days. Implementations MUST NOT delete records before the retention period expires. At Sovereign conformance, the retention period MUST be at least as long as the longest applicable regulatory retention requirement.

When records are deleted after retention expiry, the deletion MUST be recorded as a `system_event` audit record, and the hash chain MUST be repaired by computing a new chain link that references the last non-deleted record. This creates a verifiable "compaction" event.

### 10.6 Query Interface

The Audit Ledger MUST expose a query interface accessible via the AIRGP well-known endpoint at `/.well-known/airgp/audit`. The interface MUST support the following query types:

1. **By Record ID:** Retrieve a specific audit record by `record_id`.
2. **By Time Range:** Retrieve all records within a start/end timestamp range.
3. **By Operation ID:** Retrieve all records for a specific AI operation.
4. **By Verdict:** Retrieve all records with a specific `aggregate_verdict` value.
5. **By Policy Primitive:** Retrieve all records where a specific `primitive_id` was evaluated.
6. **By Tenant:** Retrieve all records for a specific `tenant_id` (Enterprise/Sovereign).
7. **Chain Verification:** Request a chain integrity verification over a specified range.

Query results MUST be paginated (default page size: 100, maximum: 1000). Results MUST be ordered by `sequence_number` ascending unless otherwise specified.

---

## 11. Identity & Attestation

### 11.1 Workload Identity

Every AIRGP component MUST have a unique workload identity. The identity is represented as a tuple of `(component_type, instance_id)` where:

- `component_type` is one of: `runtime_adapter`, `control_plane`, `policy_engine`, `audit_ledger`.
- `instance_id` is a globally unique string identifying the specific instance.

Workload identities MUST be registered with the Control Plane before the component can participate in AIRGP signal exchange. The Control Plane MUST maintain an identity registry and reject signals from unregistered components.

Implementations SHOULD use platform-native workload identity mechanisms where available (e.g., Kubernetes ServiceAccount tokens, cloud IAM roles, SPIFFE/SPIRE identities). AIRGP does not mandate a specific workload identity provider but requires that whatever mechanism is used provides cryptographic proof of identity.

### 11.2 Policy Signing

All Policy Primitives MUST be signed using Ed25519 before distribution. The signing key MUST belong to an authorized policy administrator. The Policy Engine MUST verify policy signatures before accepting policy updates.

**Policy Signature Format:**

```json
{
  "policy_content_hash": "sha256:...",
  "signer_id": "admin-001",
  "signer_public_key": "base64url-encoded-ed25519-public-key",
  "signature": "base64url-encoded-ed25519-signature",
  "signed_at": "2026-05-01T10:00:00Z",
  "expires_at": "2026-08-01T10:00:00Z"
}
```

The `policy_content_hash` is computed over the canonical JSON serialization of the policy primitive (excluding the signature block). The Policy Engine MUST reject policies with expired signatures or signatures from unknown signers.

### 11.3 Certificate Chain

AIRGP components communicate trust through a certificate chain rooted in a deployment-specific Certificate Authority (CA).

**Chain Structure:**

1. **Root CA.** The deployment's root of trust. The Root CA key MUST be stored in a hardware security module (HSM) or equivalent secure key storage at Enterprise and Sovereign conformance levels.
2. **Intermediate CA.** Issues certificates to AIRGP components. Multiple intermediate CAs MAY exist for organizational separation.
3. **Component Certificate.** Issued to each AIRGP component instance. Contains the component's Ed25519 public key and workload identity.

The certificate format MUST be X.509 v3 with the following extensions:
- Subject Alternative Name (SAN): MUST include the component's workload identity.
- Key Usage: MUST include `digitalSignature`.
- Extended Key Usage: MUST include a custom OID for AIRGP governance signal signing (OID to be assigned upon IANA registration).

### 11.4 Model Provenance Attestation

Provenance Attestations assert verifiable facts about AI models. The attestation format is:

```json
{
  "attestation_id": "att-uuid-v7",
  "attestation_type": "model_provenance",
  "subject": {
    "provider": "anthropic",
    "model": "claude-sonnet-4-20250514",
    "model_version": "2025-05-14",
    "model_hash": "sha256:..."
  },
  "claims": {
    "training_data_summary": "Pre-training on web corpus through early 2025, RLHF fine-tuning",
    "training_data_cutoff": "2025-04-01",
    "fine_tuning_history": [
      {
        "description": "RLHF alignment training",
        "date": "2025-05-01",
        "dataset_hash": "sha256:..."
      }
    ],
    "safety_evaluations": [
      {
        "evaluation_name": "Responsible Scaling Policy assessment",
        "date": "2025-05-10",
        "result": "passed",
        "report_url": "https://www.anthropic.com/rsp-evaluations"
      }
    ],
    "deployment_constraints": {
      "max_context_length": 200000,
      "supported_languages": ["en", "es", "fr", "de", "pt", "zh", "ja", "ko"],
      "prohibited_use_cases": ["weapons_development", "surveillance"]
    }
  },
  "issuer": {
    "organization": "Anthropic",
    "issuer_id": "anthropic.com",
    "public_key": "base64url-encoded-ed25519-public-key"
  },
  "issued_at": "2026-03-15T00:00:00Z",
  "expires_at": "2026-09-15T00:00:00Z",
  "signature": "base64url-encoded-ed25519-signature"
}
```

The attestation MUST be signed by the issuer using Ed25519. Verifiers MUST check:
1. The signature is valid against the issuer's public key.
2. The attestation has not expired.
3. The issuer is in the list of trusted attestation issuers (from the Provenance Requirement policy primitive).

### 11.5 Key Rotation

AIRGP components MUST support key rotation without service interruption. The rotation procedure is:

1. Generate a new Ed25519 key pair.
2. Issue a new component certificate for the new public key, signed by the Intermediate CA.
3. Register the new certificate with the Control Plane.
4. Begin signing new messages with the new key while still accepting messages signed with the old key.
5. After a configurable overlap period (default: 24 hours), deactivate the old key.
6. Record the key rotation as a `system_event` audit record.

The overlap period ensures that in-flight messages signed with the old key are still accepted during the transition. The Control Plane MUST maintain a list of active and recently-rotated keys for each component.

---

## 12. Security Considerations

### 12.1 Control Plane as High-Value Target

The AIRGP Control Plane is a high-value target for attackers because compromising it allows an attacker to modify governance policies, suppress audit records, or disable guardrails across all connected Runtime Adapters. Deployments MUST treat the Control Plane as a critical security component and apply appropriate hardening:

- The Control Plane MUST be deployed in a private network segment, not directly accessible from the public internet.
- Access to the Control Plane's administrative interface MUST require multi-factor authentication.
- The Control Plane MUST log all administrative actions as `security_event` audit records.
- At Enterprise and Sovereign conformance levels, the Control Plane MUST support running in a hardware-attested trusted execution environment (TEE) or equivalent.

### 12.2 Replay Protection

AIRGP's hash chain provides replay protection: each message includes a `prev_hash` that ties it to the previous message in the chain. An attacker who replays an old message will produce a `prev_hash` mismatch at the recipient. Additionally, the UUIDv7 `message_id` provides time-based ordering, and recipients SHOULD reject messages with timestamps more than 5 minutes in the past (configurable).

Implementations MUST maintain a sliding window of recently-seen `message_id` values and reject duplicates. The window size SHOULD be at least 10,000 messages or 5 minutes of traffic, whichever is larger.

### 12.3 Confidentiality of Governance Signals

Governance signals may contain sensitive information: policy configurations, user identifiers, model details, and cost data. All AIRGP signals MUST be transmitted over TLS 1.2 or later. Implementations SHOULD use TLS 1.3 where available. Certificate pinning is RECOMMENDED for communication between AIRGP components.

At rest, audit records and policy configurations MUST be encrypted using AES-256-GCM or equivalent. The encryption key management approach is implementation-specific but MUST follow industry best practices (e.g., envelope encryption with a KMS).

### 12.4 Side-Channel Risks in Policy Evaluation

Policy evaluation timing can leak information about the policies being applied. For example, an attacker who observes that certain inputs consistently take longer to evaluate can infer the presence of specific guardrail types. Implementations SHOULD add constant-time padding to policy evaluation responses to mitigate timing side channels. The padding SHOULD ensure that all evaluation responses take at least as long as the slowest policy evaluation observed in the last hour.

### 12.5 Supply-Chain Integrity

AIRGP components are software that can be compromised through supply-chain attacks. Implementations SHOULD:

- Publish cryptographic hashes of all released AIRGP component binaries.
- Support Software Bill of Materials (SBOM) generation in SPDX or CycloneDX format.
- Verify component integrity at startup using code signing.
- At Sovereign conformance, component binaries MUST be reproducibly built, and build provenance MUST be verifiable via SLSA Level 3 or equivalent.

---

## 13. Privacy Considerations

### 13.1 AIRGP Governs AI Systems, Not Human Users

AIRGP is a machine-to-machine protocol for governing AI system behavior. It does not directly process end-user personal data. However, governance signals may incidentally contain personal data (e.g., user identifiers in governance context, PII detected by guardrails, content hashes that could be correlated with specific users).

### 13.2 PII Handling in Audit Records

Audit records MUST NOT contain raw PII. The `governance_context.user_id` field MUST contain a pseudonymized identifier (e.g., a one-way hash of the actual user ID). If guardrail verdicts include information about detected PII (e.g., "3 email addresses detected and masked"), the audit record MUST NOT include the actual PII values.

At Sovereign conformance, audit records MUST comply with applicable data protection regulations (e.g., GDPR, LGPD). This may require:
- Pseudonymization of all user-related fields.
- The ability to delete or anonymize audit records pertaining to a specific user upon lawful request (right to erasure), with appropriate hash chain repair.
- Data Protection Impact Assessment (DPIA) documentation for the AIRGP deployment.

### 13.3 Data Minimization in Telemetry

Telemetry signals SHOULD follow the principle of data minimization: collect only what is necessary for governance, observability, and compliance purposes. Implementations MUST NOT include raw AI inputs or outputs in telemetry signals. Content hashes (SHA-256 of input/output) MAY be included for correlation purposes.

At Enterprise and Sovereign conformance levels, implementations MUST provide configurable telemetry granularity, allowing deployment operators to control which fields are included in telemetry signals. The default configuration MUST be the most privacy-preserving option.

### 13.4 Right to Explanation

AIRGP supports the right to explanation required by certain regulations (e.g., GDPR Article 22(3), EU AI Act Article 86). When an AI operation is denied by governance, the `reason` field in the verdict provides a machine-readable explanation. Implementations SHOULD provide a mechanism to translate governance verdicts into human-readable explanations suitable for end users.

The audit record for each governance decision preserves the full evaluation context, enabling post-hoc explanation of why a specific decision was made. This supports both individual right-to-explanation requests and regulatory audits.

---

## 14. IANA Considerations

### 14.1 Media Type Registration

This specification requests registration of the following media type with IANA:

- **Type name:** application
- **Subtype name:** airgp+json
- **Required parameters:** None
- **Optional parameters:** `version` — the AIRGP protocol version (e.g., `version=1.0`)
- **Encoding considerations:** binary (UTF-8 encoded JSON)
- **Security considerations:** See Section 12 of this specification
- **Interoperability considerations:** AIRGP messages are JSON objects conforming to the schemas defined in this specification
- **Published specification:** This document
- **Applications that use this media type:** AI governance control planes, runtime adapters, policy engines, audit ledgers
- **Fragment identifier considerations:** None
- **Additional information:**
  - Magic number(s): None
  - File extension(s): `.airgp.json`
  - Macintosh file type code(s): None
- **Person & email address to contact for further information:** Fabio Bastos, protocol@thinkneo.ai
- **Intended usage:** COMMON
- **Restrictions on usage:** None
- **Author:** Fabio Bastos
- **Change controller:** ThinkNEO AI Technology Company Limited

### 14.2 Well-Known URI Registration

This specification requests registration of the following well-known URI with IANA:

- **URI suffix:** airgp
- **Change controller:** ThinkNEO AI Technology Company Limited
- **Specification document:** This document
- **Related information:** The well-known URI `/.well-known/airgp` is used for AIRGP service discovery and governance signal exchange.

### 14.3 URI Scheme Considerations

AIRGP does not define a new URI scheme. All AIRGP communication uses standard HTTPS URIs. The well-known endpoint `/.well-known/airgp` follows the conventions established in RFC 8615 (Well-Known URIs).

Future versions of AIRGP MAY define an `airgp://` URI scheme for service discovery in environments where well-known URIs are not practical (e.g., peer-to-peer AIRGP deployments). Such a scheme would be registered with IANA in a future specification.

---

## 15. References

### 15.1 Normative References

- **[RFC 2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997.

- **[RFC 8174]** Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017.

- **[RFC 3339]** Klyne, G. and C. Newman, "Date and Time on the Internet: Timestamps", RFC 3339, DOI 10.17487/RFC3339, July 2002.

- **[RFC 8259]** Bray, T., Ed., "The JavaScript Object Notation (JSON) Data Interchange Format", STD 90, RFC 8259, DOI 10.17487/RFC8259, December 2017.

- **[RFC 9110]** Fielding, R., Ed., Nottingham, M., Ed., and J. Reschke, Ed., "HTTP Semantics", STD 97, RFC 9110, DOI 10.17487/RFC9110, June 2022.

- **[RFC 8615]** Nottingham, M., "Well-Known Uniform Resource Identifiers (URIs)", RFC 8615, DOI 10.17487/RFC8615, May 2019.

- **[RFC 8032]** Josefsson, S. and I. Liusvaara, "Edwards-Curve Digital Signature Algorithm (EdDSA)", RFC 8032, DOI 10.17487/RFC8032, January 2017.

- **[UUID]** Davis, K., Peabody, B., and P. Leach, "Universally Unique IDentifiers (UUIDs)", RFC 9562, DOI 10.17487/RFC9562, May 2024.

### 15.2 Informative References

- **[NIST AI RMF]** National Institute of Standards and Technology, "Artificial Intelligence Risk Management Framework (AI RMF 1.0)", NIST AI 100-1, January 2023.

- **[EU AI Act]** European Parliament and Council, "Regulation (EU) 2024/1689 laying down harmonised rules on artificial intelligence", Official Journal of the European Union, July 2024.

- **[ISO 42001]** International Organization for Standardization, "Information technology — Artificial intelligence — Management system", ISO/IEC 42001:2023, December 2023.

- **[MCP]** Anthropic, "Model Context Protocol Specification", 2024. https://modelcontextprotocol.io/

- **[A2ASTC]** Bastos, F., "Agent-to-Agent Standardised Trust & Compliance Protocol", ThinkNEO AI Technology Company Limited, 2026.

- **[OpenPort]** "OpenPort: A Unified Framework for Secure Tool Access in AI Agents", arXiv:2602.20196, 2026.

- **[Policy Cards]** "Policy Cards: Machine-Readable AI Policy Specifications", arXiv:2510.24383, 2025.

- **[MI9]** "MI9: A Framework for Agentic AI Runtime Governance", arXiv:2508.03858, 2025.

- **[EU Apply AI]** EU Apply AI Alliance, "Execution-Time AI Governance", 2025.

- **[OpenTelemetry]** OpenTelemetry Authors, "OpenTelemetry Specification", https://opentelemetry.io/docs/specs/

- **[FIPS 180-4]** National Institute of Standards and Technology, "Secure Hash Standard (SHS)", FIPS PUB 180-4, August 2015.

---

## Appendix A: Acknowledgements

AIRGP is a consolidator protocol that builds on the work of many researchers, organizations, and standards bodies. The author gratefully acknowledges:

- The authors of the OpenPort Protocol for pioneering tool-access security for AI agents.
- The authors of Policy Cards for defining machine-readable AI policy specifications.
- The authors of MI9 for proposing a comprehensive agentic AI runtime governance framework.
- The EU Apply AI Alliance for recognizing the need for execution-time governance.
- NIST for the AI Risk Management Framework.
- ISO/IEC JTC 1/SC 42 for ISO/IEC 42001.
- Anthropic for the Model Context Protocol, which demonstrates the value of standardized AI component communication.
- The OpenTelemetry community for building the observability foundation that AIRGP extends.

---

## Appendix B: Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-05-01 | Initial publication |

---

*Copyright 2026 ThinkNEO AI Technology Company Limited. This specification is licensed under CC-BY-ND-4.0.*

*AIRGP is a trademark of ThinkNEO AI Technology Company Limited.*

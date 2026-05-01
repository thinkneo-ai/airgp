# AIRGP Conformance Guide

## Conformance Levels

AIRGP defines three conformance levels. Each level is cumulative.

### Basic

Required capabilities:
- Runtime Adapter intercepting AI provider calls
- Guardrail verdict generation and enforcement
- Basic audit record creation
- Wire messages using `application/airgp+json` format
- HTTPS transport to `/.well-known/airgp` endpoint

### Enterprise

All Basic requirements plus:
- Full Policy Engine with all 7 primitive types
- Cost accounting and budget enforcement
- Provenance attestation support
- OpenTelemetry-compatible observability signals
- Hash-chained audit ledger with Merkle anchoring
- Ed25519 policy signing

### Sovereign

All Enterprise requirements plus:
- Data residency enforcement with geographic boundary controls
- Regulatory compliance mapping (EU AI Act, NIST AI RMF, ISO 42001)
- Sovereign cloud deployment support
- Key escrow and recovery procedures
- Audit ledger export in regulatory-required formats

## Claiming Conformance

### 1. Self-Assessment

Evaluate your implementation against the conformance checklist for your target level. The checklist is available at `schemas/` in this repository.

### 2. Test Evidence

Prepare documentation demonstrating:
- Wire message format compliance (validated against JSON Schemas)
- Transport endpoint availability
- Audit record chain integrity
- Policy primitive evaluation correctness

### 3. Conformance License Application

Submit your application to conformance@thinkneo.ai with:
- Organization name and contact
- Target conformance level
- Implementation description
- Test evidence package
- Signed AIRGP Adopter Agreement

### 4. Review

The Steering Committee reviews applications within 30 business days. Successful applicants receive:
- AIRGP Conformance License for the attested level
- Permission to use the AIRGP conformance badge
- Listing on the AIRGP implementations registry

### 5. Annual Renewal

Conformance is valid for 12 months. Renewal requires updated test evidence against the current specification version.

## First Conformant Implementation

**ThinkNEO Control Plane** is the reference implementation and first conformant implementation at the Enterprise level.

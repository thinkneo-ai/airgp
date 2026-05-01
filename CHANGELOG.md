# Changelog

All notable changes to the AIRGP specification will be documented in this file.

## [1.0] - 2026-05-01

### Added
- Initial publication of AIRGP v1.0
- 15 specification sections covering architecture, wire protocol, policy primitives, observability, audit, identity
- 7 policy primitive types: guardrail, routing rule, cost cap, data boundary, provenance requirement, rate limit, compliance mapping
- 3 conformance levels: Basic, Enterprise, Sovereign
- JSON Schema definitions for all wire message types
- OpenTelemetry semantic conventions mapping
- Tamper-evident, hash-chained audit ledger format
- Ed25519-based identity and attestation framework
- IANA media type registration: application/airgp+json
- Well-known URI: /.well-known/airgp

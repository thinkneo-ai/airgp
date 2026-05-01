# AI Governance Landscape

An assessment of existing standards, frameworks, and protocols relative to the runtime wire protocol gap that AIRGP addresses.

## Frameworks and Regulations

### NIST AI RMF (AI Risk Management Framework)
Type: Framework. Scope: Organizational risk management for AI systems. Provides guidance on governing, mapping, measuring, and managing AI risks. **Not a runtime wire protocol** — it defines processes and practices, not machine-to-machine message formats.

### EU AI Act
Type: Regulation. Scope: Legal requirements for AI systems classified by risk level. Mandates conformity assessments, transparency, and human oversight for high-risk systems. **Not a runtime wire protocol** — it is law that must be operationalized through technical controls. AIRGP provides one such technical control layer.

### ISO/IEC 42001
Type: Management system certification. Scope: Organizational AI management system requirements. Defines how organizations establish, implement, and improve AI management. **Not a runtime wire protocol** — it certifies organizational processes, not software interfaces.

### IEEE 7000 Series
Type: Design process standards. Scope: Ethical considerations in system design. Addresses value-sensitive design and ethical impact assessment during development. **Not a runtime wire protocol** — it operates pre-deployment, not at runtime.

### OECD AI Principles
Type: Policy recommendations. Scope: High-level principles for trustworthy AI. Provides government-level guidance on AI governance. **Not a runtime wire protocol** — these are principles, not technical specifications.

## Security and Threat Taxonomies

### OWASP LLM Top 10
Type: Vulnerability taxonomy. Scope: Top security risks for LLM applications. Identifies risks like prompt injection, data leakage, and supply chain. **Not a runtime wire protocol** — it is a threat list that informs what guardrails should detect.

### MITRE ATLAS
Type: Adversarial threat matrix. Scope: Adversarial tactics and techniques against AI systems. **Not a runtime wire protocol** — it catalogs threats, not the protocol to govern against them.

## Industry Frameworks

### Google SAIF (Secure AI Framework)
Type: Corporate security framework. Scope: Google's approach to securing AI systems. Covers model integrity, data protection, and access control. **Not a runtime wire protocol** — it is a vendor-specific framework.

### Microsoft Responsible AI Standard
Type: Corporate standard. Scope: Microsoft's internal requirements for responsible AI. Covers fairness, reliability, privacy, inclusiveness, transparency, and accountability. **Not a runtime wire protocol** — vendor-specific internal standard.

## Protocols and Specifications

### MCP (Model Context Protocol)
Type: Open protocol. Scope: Standardized tool-use communication between AI models and tool providers. **Orthogonal to AIRGP** — MCP handles tool calls; AIRGP handles governance signals. An MCP tool call flows through AIRGP governance checkpoints.

### OpenPort Protocol (arXiv:2602.20196)
Type: Academic specification. Scope: Tool-access security for AI agents. Defines secure mechanisms for agents to access tools. **Narrower than AIRGP** — AIRGP covers the full runtime governance envelope including policy, audit, cost, and provenance beyond tool access.

### Policy Cards (arXiv:2510.24383)
Type: Academic specification. Scope: Machine-readable deployment-layer policy specifications. Provides a JSON Schema-based format for expressing AI deployment policies. **AIRGP builds on this** — the AIRGP Policy Primitive layer adopts Policy Card-style JSON Schema for expressing policy conditions and actions.

### MI9 (arXiv:2508.03858)
Type: Academic framework. Scope: Agentic AI runtime governance framework. Proposes a conceptual architecture for runtime governance of autonomous agents. **AIRGP standardizes what MI9 conceptualizes** — MI9 is a framework paper without a wire protocol or conformance model; AIRGP provides both.

## Commercial Products

### Prefactor
Type: Commercial product. Scope: Runtime AI governance platform. Provides runtime controls for AI applications. **Vendor-specific** — AIRGP is vendor-neutral by design, enabling interoperability across vendors.

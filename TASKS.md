# OFDP Landing Page

Build a single-page website for ofdp.dev — the Open Financial Data Protocol.

## Design Direction
- Dark background (#0a0a0a or similar near-black)
- Clean, minimal, technical — think Stripe or Linear aesthetic
- Monospace accents for code/protocol elements
- No stock photos, no gradients, no fluff
- Mobile responsive

## Content

### Hero
Headline: "Open Financial Data Protocol"
Subheadline: "A consumer-owned, federated, open standard for personal financial data."
Two buttons:
- "Read the Specification" → https://github.com/jakevolkman/ofdp-spec
- "Follow Development" → https://github.com/jakevolkman/ofdp-spec/issues

### Problem Statement Section
Title: "The Problem"
Three cards:
1. "Fragmented" — Consumer financial data is scattered across hundreds of institutions with incompatible formats and no standard way to access it.
2. "Intermediated" — Aggregators like Plaid and MX charge institutions for access to data consumers generated, creating misaligned incentives.
3. "Opaque" — No standard provenance model means consumers and AI agents can't tell where data came from or how much to trust it.

### What OFDP Is Section
Title: "What OFDP Is"
Paragraph: OFDP is a data layer — not a payment system. It defines how financial data is structured, stored, shared, and verified across institutions, applications, and AI systems. It is designed to complement FDX and Section 1033 infrastructure by adding a consumer sovereignty layer.

Three feature cards:
1. "Consumer Owned" — Your instance is the authoritative source for your financial identity. Institutions are data sources, not data owners.
2. "AI Native" — Structured, normalized data designed to be consumed by AI agents without transformation.
3. "Self Hostable" — Runs on modest hardware. No cloud dependency required.

### Standards Compatibility Section
Title: "Built to Coexist"
Small table or card grid showing:
- FDX → OFDP consumes FDX-compliant APIs as a data source
- ISO 20022 → Transaction fields map directly to OFDP schema
- OAuth 2.0 → Foundation for all ConnectionRequest authorization
- MCP → OFDP instances expose MCP servers for AI agent access
- Section 1033 → OFDP is the consumer layer above institutional infrastructure

### Status Section
Title: "Current Status"
Version badge: "v0.1 — Draft Specification"
Status: "Open for comment from financial institutions, credit unions, fintech developers, and consumer advocates."
Timeline:
- v0.1 (Current) — Draft specification
- v0.2 (Planned) — Reference implementation + compliance test suite
- v0.3 (Planned) — Registry launch + first institutional certifications
- v1.0 (Planned) — Stable specification + FDX interoperability validated

### Footer
"OFDP is an open specification. Apache 2.0 License."
GitHub link
"ofdp.dev"

## Technical Requirements
- Single HTML file (index.html) with embedded CSS and no external dependencies
- No JavaScript frameworks
- Fast loading — pure HTML/CSS only
- Must pass basic accessibility (alt tags, semantic HTML, sufficient contrast)
- Add a CNAME file containing just: ofdp.dev
- Add a .nojekyll file (empty) so GitHub Pages serves correctly

## Rules
- No placeholder text — all content is specified above
- Commit as "Initial landing page"

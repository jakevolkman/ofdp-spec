# Open Financial Data Protocol (OFDP)

**Consumer-Owned. Federated. AI-Native.**

OFDP is an open standard for personal financial data. It defines how financial data is structured, stored, shared, and verified across institutions, applications, and AI systems — with the consumer as the authoritative owner of their own data.

OFDP is not a payment system. It is a data layer that normalizes financial information so that consumers, institutions, and AI agents can communicate using a shared language regardless of the underlying payment rail, institution type, or account format.

**Current status:** v0.1 Draft — open for comment from financial institutions, credit unions, fintech developers, and consumer advocates.

---

## The Problem

Consumer financial data is fragmented across hundreds of institutions with proprietary formats and limited export capabilities. Existing aggregation solutions — Plaid, MX, Finicity — operate as centralized intermediaries that charge institutions for access to data the consumer generated.

The CFPB's Personal Financial Data Rights rule (Section 1033) and the Financial Data Exchange (FDX) address how institutions share data with authorized third parties. FDX is a B2B protocol — it defines how institutions talk to apps, not how consumers own and control their own data.

OFDP fills that gap: it sits between institutional infrastructure and consumer data sovereignty.

## Relationship to Existing Standards

OFDP is designed to interface with existing standards, not replace them.

| Standard | Role | OFDP Relationship |
|---|---|---|
| FDX | Institutional data sharing | OFDP consumes FDX-compliant APIs as a data source. FDX-certified institutions receive the highest trust tier. |
| ISO 20022 | Payment message standard | OFDP maps ISO 20022 transaction fields to its core Transaction schema. |
| SWIFT | Interbank messaging | Institutional-layer only. OFDP uses SWIFT codes for institution identification. |
| OAuth 2.0 | Authorization | OFDP uses OAuth 2.0 with PKCE as the authentication foundation for institution connections. |
| MCP | AI data integration | OFDP instances can expose an MCP server so AI agents can query financial data natively. |

## Design Principles

- **Consumer ownership.** The consumer's instance is the authoritative source for their financial identity. Institutions are data sources, not data owners.
- **Federation over centralization.** Any individual or institution can run an OFDP instance. Instances communicate without requiring a central authority.
- **Privacy by default.** Institutions cannot reach into consumer instances. Consumers pull data on their own schedule and grant access explicitly.
- **Provenance as a first-class requirement.** All externally sourced data carries metadata describing its origin, verification status, and confidence level.
- **Lightweight enough to self-host.** The core protocol is designed to run on modest hardware, including consumer-grade computers.
- **AI-native.** The data model and API are designed to be consumed by AI agents without transformation.
- **Extensible without fragmentation.** Optional extension modules cover specialized use cases. Institutions are never penalized for not implementing them.

---

## Core Concepts

### Provenance

Every record that contains externally sourced data carries provenance metadata: source, verification status, retrieval timestamp, and confidence level. Sources are ranked 1–12, from most to least trusted:

| Rank | Source |
|---|---|
| 1 | Institution API with verification from core banking system |
| 2 | FDX-compliant API with verification flag |
| 3 | Contract (loan agreement, account terms) |
| 4 | Government API (IRS, SSA, studentaid.gov) |
| 5 | Institution API (unverified) |
| 6 | Computed from rank 1–4 sources |
| 7 | Credit bureau |
| 8 | Screen-scraped |
| 9 | Aggregator pass-through |
| 10 | Document import (PDF, scanned statement) |
| 11 | Manual import (CSV, OFX, QIF) |
| 12 | User-entered |

When two records describe the same data point, higher-ranked sources win automatically. Equal-rank conflicts are surfaced to the user for resolution. Large divergences trigger alerts regardless of rank.

### Federation

OFDP instances are discovered through the OFDP Registry — an append-only, hash-chained ledger maintained by the OFDP foundation. The hash-chain design ensures tamper-evidence and supports migration to a distributed consensus model in a future version without changing the data format.

Sync is pull-based. Consumer instances request data from institutions; institutions never push data without an explicit request. Sync state is tracked per account per resource type using cursors for incremental updates.

### Institution Trust Tiers

Institutions are assigned a verification tier that governs the trust signals their data carries:

| Tier | Name | Key Requirement |
|---|---|---|
| 4 | Self-Certified | Valid legal entity, registered endpoint, self-signed certificate |
| 3 | Protocol Certified | Legal entity verified against regulatory records, OFDP-issued certificate |
| 2 | Verified Institution | Responses include verification timestamp from core banking system |
| 1 | FDX Certified | Current FDX certification accepted in lieu of OFDP audit |

---

## Data Model

All entities that contain externally sourced data extend `ProvenanceBase`, which adds source, verification status, retrieval timestamp, and confidence level fields.

### Core Entities

| Entity | Description |
|---|---|
| `User` | Consumer, institution, or system account |
| `Institution` | Financial institution with self-referencing structure for brands, legal entities, and servicers |
| `Account` | Base account record (checking, savings, credit card, investment, mortgage, student loan, crypto) |
| `AccountHolding` | Asset position within an account |
| `Asset` | Unified model for fiat, crypto, stocks, bonds, ETFs, and commodities |
| `AssetPrice` | Price history for any asset |
| `Transaction` | Individual transaction with optional itemization |
| `TransactionItem` | Line-item breakdown of a transaction (e.g., individual items in a grocery receipt) |
| `Vendor` | Normalized merchant record with chain/location hierarchy |
| `Category` | Hierarchical taxonomy for classifying transactions |
| `BalanceSnapshot` | Point-in-time balance record |
| `InterestRate` | Rate history per account |
| `RecurringCost` | Expected future payment patterns (subscriptions, loans, utilities) |

### Access Control

Permissions are resource- and action-scoped (`account:read`, `transaction:export`, etc.). Roles (`consumer`, `family`, `advisor`, `admin`) bundle permissions. `ConnectionRequest` handles institution-to-consumer authorization via OAuth 2.0 and cryptographic certificates.

### Account Type Extensions

Core account fields are extended by type-specific schemas:

| Extension | Adds |
|---|---|
| `CreditCardAccount` | Credit limit, minimum payment, grace period |
| `MortgageAccount` | Property value, escrow, equity, amortization schedule |
| `StudentLoanAccount` | Federal/private, repayment plan, income-driven details, forgiveness tracking |
| `InvestmentAccount` | Account subtype (IRA, 401k, 529, HSA), tax status, cost basis method |
| `InvestmentHolding` | Lot-level acquisition date, cost basis, holding period |
| `RealizedGainLoss` | Sale records with wash-sale tracking |
| `CryptoAccount` | Wallet address, blockchain, custody type |
| `StakingPosition` | Staked amount, reward rate, validator |

---

## API

OFDP uses REST with JSON. Every request must include a `Protocol-Version` header. Every response includes provenance metadata and a cryptographic signature from the responding institution.

### Endpoint Groups

**Registry** — hosted by the OFDP foundation
- `GET /registry/institutions` — search institutions by name, routing number, SWIFT code
- `GET /registry/institutions/:id/verify` — validate hash chain for an institution
- `POST /registry/institutions` — submit a registration request

**Authentication** — hosted by each institution
- `GET /auth/certificate` — institution's current certificate and public key
- `POST /auth/connect` — submit a `ConnectionRequest` with requested permissions
- `POST /auth/token` — exchange certificate signature for a session token

**Data** — hosted by each institution
- `GET /accounts` — list authorized accounts
- `GET /accounts/:id/transactions` — paginated transactions with cursor support
- `GET /accounts/:id/balances` — balance history
- `GET /accounts/:id/holdings` — asset positions
- `GET /accounts/:id/interest_rates` — rate history
- `GET /vendors/:id`, `GET /assets/:id/prices` — vendor and asset lookups

**Sync** — hosted on the consumer's own instance
- `POST /sync/jobs` — initiate a sync from an institution
- `GET /sync/conflicts` — list unresolved data conflicts
- `PUT /sync/conflicts/:id` — resolve a conflict

Full endpoint reference is in the specification document.

### Optional Extensions

- **GraphQL interface** — flexible field selection for complex queries; not required for compliance
- **Webhooks** — institution-initiated notifications for new transactions, balance changes, rate changes
- **Rewards module** — credit card and loyalty programs
- **DeFi positions** — liquidity pools, lending, yield farming
- **NFT holdings** — non-fungible digital assets

---

## Security

- **Transport:** TLS 1.3+; mTLS required for institution-to-institution communication
- **Authentication:** OAuth 2.0 with PKCE; session tokens expire in 1 hour
- **Signatures:** All `ConnectionRequest` and sync responses must be cryptographically signed by the institution's private key
- **Data at rest:** Account numbers, routing numbers, and wallet addresses encrypted with AES-256; keys stored separately from data
- **Sandbox mode:** All sensitive fields masked regardless of permissions; sandbox instances must not contain real financial data

---

## Roadmap

| Version | Status | Scope |
|---|---|---|
| v0.1 | Current | Draft specification. Request for comment. |
| v0.2 | Planned | Reference implementation of core schema and REST API. Compliance test suite. First credit union pilot. |
| v0.3 | Planned | Registry launch. First institutional certifications. iOS and Android consumer reference apps. |
| v1.0 | Planned | Stable specification. Production-ready reference implementation. FDX interoperability validated. |
| v2.0 | Planned | Distributed registry. GraphQL extension. DeFi extension module. |

---

## Contributing

OFDP is being developed as an open specification. The goal is a protocol that any credit union developer can implement in weeks, any consumer can self-host on modest hardware, and any AI agent can consume without transformation.

Feedback, corrections, and proposals for extension modules are welcome. Open an issue or submit a pull request.

The full specification is in [`OFDP_Specification_v0.1.1.docx`](OFDP_Specification_v0.1.1.docx). (Previous draft: [`OFDP_Specification_v0.1.docx`](OFDP_Specification_v0.1.docx).)

**v0.1.1 (June 2026):** added §2.5 Record Matching and Deduplication (external-ID → fingerprint → fuzzy window → balance reconciliation cascade) and §2.6 Sourced Facts and Annotations (user annotations are never overwritten by sync); added `Transaction.external_id` and `ProvenanceBase.overridden_by_user`; documented the non-negative amount convention; made MCC-derived category fields nullable; the standard category taxonomy will ship as a versioned machine-readable artifact (slugs + MCC mapping).

## License

Apache 2.0

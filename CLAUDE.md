# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

The reference codebase is in the `prototype` submodule:
- **prototype/**: MetaState Prototype project (git submodule)
When considering code, navigate to the `prototype/` directory.

- **Root directory**: various sources of truth about w3ds and the result of AI's work on it

## MetaState Prototype Architecture
The prototype is a **pnpm monorepo** using **Turbo** for task orchestration. It implements a multi-tenant vault system (W3DS - Web3 Data Space) with the following layers:

### Core Infrastructure (`prototype/infrastructure/`)
- **evault-core**: Multi-tenant GraphQL API (Fastify + GraphQL Yoga) and provisioning service (Express)
  - Uses **Neo4j** for graph data
  - Exposes two APIs: provisioning (port 3001) and GraphQL (port 4000)
  - Migrations via TypeORM
- **registry**: Central vault entry registry service (Fastify)
  - Uses **Postgres** for relational data
  - Port 4321
  - Signs entropy tokens with ES256 keys
- **w3id**: Identity management
- **web3-adapter**: Web3 integration layer
- **eid-wallet**: Electronic ID wallet
- **signature-validator**: Signature validation utilities
- **dev-sandbox**: Development UI for W3DS flows (port 8080)

### Platforms (`prototype/platforms/`)
Social and governance applications built on the core infrastructure. Each platform typically has `api/` and `client/` subdirectories:
- **blabsy**: Social media platform
- **pictique**: Image sharing platform
- **group-charter-manager**: Group governance
- **evoting**: Voting system
- **dreamsync**, **cerberus**, **ereputation**, **marketplace**, etc.

### Shared Packages (`prototype/packages/`)
- **wallet-sdk**: SDK for wallet integration
- **w3ds-gateway**: W3DS gateway utilities
- **eslint-config**, **typescript-config**: Shared tooling configs

### Services (`prototype/services/`)
- **ontology**: Ontology service
- **search-engine**: Search functionality
- **trust-score**: Trust scoring system

### Testing

**Unit tests** (most packages use Vitest or Jest):
```bash
# All tests via Turbo
pnpm test

# Specific package
pnpm --filter evault-core test
pnpm --filter registry test
```

**E2E tests** (use Testcontainers):
```bash
pnpm --filter evault-core test:e2e
```

## Key Technical Patterns

### API Patterns
- **Fastify** is the primary framework for REST APIs (registry, evault-core GraphQL server)
- **GraphQL Yoga** on top of Fastify for GraphQL endpoints
- **Express** for the evault-core provisioning API (legacy, being migrated)

### Multi-tenant Design
- evault-core implements per-tenant data isolation
- Registry tracks vault entries across tenants

### Security
- Entropy tokens signed with ES256 (REGISTRY_ENTROPY_KEY_JWK)
- Signature validation via signature-validator package
- JWT authentication via @fastify/jwt

### Frontend Stack
The monorepo includes multiple frontend frameworks:
- Svelte, React, Astro, Vue (see biome.json overrides)
- Most client apps are framework-specific to platform requirements

## Port Reference

| Service              | Port(s)      | Description                    |
|----------------------|--------------|--------------------------------|
| Postgres             | 5432         | Registry database              |
| Neo4j HTTP           | 7474         | Neo4j browser interface        |
| Neo4j Bolt           | 7687         | Neo4j driver connection        |
| Registry             | 4321         | Vault registry API             |
| evault-core (prov)   | 3001         | Provisioning API (Express)     |
| evault-core (GraphQL)| 4000         | Multi-tenant GraphQL API       |
| dev-sandbox          | 8080         |                                |
| Blabsy API           | 3000         |                                |
| Pictique API         | 1111         |                                |
| Pictique Frontend    | 5173         |                                |
| Group Charter API    | 3002         |                                |
| Group Charter UI     | 3004         |                                |
| eVoting API          | 4002         |                                |
| eVoting UI           | 3005         |                                |

See README.md in prototype/ for the full port list.

## Documentation

### Online Resources
- **Full docs**: https://docs.w3ds.metastate.foundation

### Getting Started
- [Main README](prototype/README.md) - Docker development environment overview with port assignments for core services, platform APIs, and frontend services, plus monorepo structure details
- [Quick Start](prototype/QUICKSTART.md) - Redirect notice to local-dev-quick-start.md and online docs
- [Getting Started Guide](prototype/docs/docs/Getting%20Started/getting-started.md) - Introduction to W3DS protocol allowing users store data in personal eVaults and platforms act as interchangeable frontends
- [Local Dev Quick Start](prototype/docs/docs/Post%20Platform%20Guide/local-dev-quick-start.md) - Guide for running Postgres and Neo4j in Docker, then launching registry, evault-core, and dev-sandbox with `pnpm dev:core`

### W3DS Fundamentals
- [W3DS Basics Overview](prototype/docs/docs/W3DS%20Basics/getting-started.md) - Deep dive into W3DS core concepts, eVault ownership model, and complete data flow showing how data synchronizes across platforms
- [W3ID](prototype/docs/docs/W3DS%20Basics/W3ID.md) - UUID-based persistent identifier system for users, groups, eVaults, and objects with loose key binding for rotation/recovery
- [eName](prototype/docs/docs/W3DS%20Basics/eName.md) - Registry-registered subset of W3ID that serves as globally unique, resolvable identifier for resources
- [Links](prototype/docs/docs/W3DS%20Basics/Links.md) - Production base URLs for core W3DS infrastructure services (Provisioner, Registry, Ontology)
- [Binding Documents](prototype/docs/docs/W3DS%20Basics/Binding-Documents.md) - Special MetaEnvelopes that tie a user to their eName with four types: id_document, photograph, social_connection, and self
- [Glossary](prototype/docs/docs/W3DS%20Basics/glossary.md) - Comprehensive glossary defining key W3DS and MetaState terms with links to detailed documentation

### W3DS Protocol
- [Authentication](prototype/docs/docs/W3DS%20Protocol/Authentication.md) - Cryptographic signature-based authentication protocol using ECDSA P-256 keys instead of passwords
- [Signing](prototype/docs/docs/W3DS%20Protocol/Signing.md) - `w3ds://sign` protocol for requesting arbitrary cryptographic signatures from users through their eID wallet
- [Signature Formats](prototype/docs/docs/W3DS%20Protocol/Signature-Formats.md) - ECDSA P-256 with SHA-256 signature formats and verification algorithms for desktop development workflows
- [Awareness Protocol](prototype/docs/docs/W3DS%20Protocol/Awareness-Protocol.md) - Webhook delivery mechanism where eVault Core pushes change notifications to registered platforms (push-based, fire-and-forget)

### Infrastructure Components
- [eVault](prototype/docs/docs/Infrastructure/eVault.md) - Core storage system with GraphQL API for storing/retrieving data in MetaEnvelopes, ACL-based access control, webhook delivery, and key binding certificates
- [eVault Key Delegation](prototype/docs/docs/Infrastructure/eVault-Key-Delegation.md) - Process of generating ECDSA P-256 keys, syncing public keys to eVault, and creating Registry-issued key binding certificates
- [Registry](prototype/docs/docs/Infrastructure/Registry.md) - Core service providing W3ID-based service discovery, cryptographically secure entropy generation, JWT verification, and temporary key binding certificates
- [Registry Protocol](prototype/platforms/registry/api/REGISTRY_PROTOCOL.md) - Registry Service Protocol standardizing W3ID-based service discovery, entropy generation, and platform certification with JWT-based token protocols
- [eID Wallet](prototype/docs/docs/Infrastructure/eID-Wallet.md) - Tauri-based mobile app managing cryptographic keys using device hardware security (Secure Enclave/HSM) for authentication and eVault provisioning
- [Push Notifications Setup](prototype/infrastructure/eid-wallet/PUSH_NOTIFICATIONS_SETUP.md) - Setup instructions for Firebase Cloud Messaging (FCM) and Apple Push Notification service (APNs) in the eID wallet
- [Web3 Adapter](prototype/docs/docs/Infrastructure/Web3-Adapter.md) - Bidirectional bridge between platform's local database and eVault, automatically syncing data to global ontology and applying incoming awareness packets
- [Mapping Rules](prototype/infrastructure/web3-adapter/MAPPING_RULES.md) - How to create JSON mappings defining how local database fields and relations map to global ontology fields
- [Wallet SDK](prototype/docs/docs/Infrastructure/wallet-sdk.md) - Crypto-agnostic TypeScript package implementing eVault provisioning, platform authentication, and public-key sync using pluggable CryptoAdapter interface
- [Ontology](prototype/docs/docs/Infrastructure/Ontology.md) - Schema registry for W3DS serving JSON Schema (draft-07) definitions identified by W3IDs

### Component READMEs
- [evault-core](prototype/infrastructure/evault-core/README.md) - Provisioning service providing TypeScript API for provisioning multi-tenant evault instances on Nomad with Neo4j backends
- [dev-sandbox](prototype/infrastructure/dev-sandbox/README.md) - Minimal SvelteKit browser application for testing W3DS wallet flows using wallet-sdk with Web Crypto adapter
- [w3id](prototype/infrastructure/w3id/README.md) - Web 3 Identity System providing globally unique UUID-based identifiers with key rotation, friend-based recovery, and eVault migration tracking
- [signature-validator](prototype/infrastructure/signature-validator/README.md) - TypeScript library for verifying digital signatures by retrieving public keys from eVault via registry service
- [blindvote](prototype/infrastructure/blindvote/README.md) - Decentralized privacy-preserving voting system using Pedersen commitments and ed25519 elliptic curve cryptography for publicly verifiable elections
- [control-panel](prototype/infrastructure/control-panel/README.md) - SvelteKit-based control panel for monitoring and managing eVault pods in Kubernetes clusters with real-time pod discovery and logs
- [w3ds-gateway](prototype/packages/w3ds-gateway/README.md) - Package that resolves W3DS eNames to application URLs and presents app chooser interface like Android's "Open with..." system

### Platform Development
- [Post Platform Guide](prototype/docs/docs/Post%20Platform%20Guide/getting-started.md) - Essential guide for building platforms with signature-based W3ID authentication using `w3ds://auth` protocol
- [Dev Sandbox](prototype/docs/docs/Post%20Platform%20Guide/dev-sandbox.md) - W3DS Dev Sandbox (port 8080) for testing provisioning, authentication, and signing flows without a real eID wallet
- [Webhook Controller](prototype/docs/docs/Post%20Platform%20Guide/webhook-controller.md) - How to implement webhook controller that receives awareness protocol packets and transforms global ontology data to local database schema
- [Mapping Rules](prototype/docs/docs/Post%20Platform%20Guide/mapping-rules.md) - Guide for creating mapping rules defining how local database fields/relations map to global ontology schema

### Services
- [Search Engine](prototype/services/search-engine/README.md) - Search service that caches data from registry and eVault (refreshing every 15 minutes) to provide fast search across vault entries and user profiles
- [Search Examples](prototype/services/search-engine/examples.md) - Practical API usage examples for registry searches, user searches, and combined searches using curl, axios, and fetch
- [Trust Score](prototype/services/trust-score/README.md) - Service calculating deterministic 0-10 trust scores based on verification status, account age, key location (TPM/SW), and social connections

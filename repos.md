# Study Notes Creator: Repository Register

**Last updated:** 23 November 2026  
**Architecture:** Polyrepo frontend applications and backend microservices  
**Canonical prefix:** `study-notes-creator-`

## Repository count

| Category | Count |
|---|---:|
| Product governance and system documentation | 1 |
| Frontend application repositories | 3 |
| Shared contract and frontend-library repositories | 4 |
| Backend microservice repositories | 14 |
| Platform, infrastructure, and quality repositories | 3 |
| **Total repositories** | **25** |

There are **24 implementation repositories** plus **1 product/governance repository**, for **25 repositories in total**.

## Product structure

```text
study-notes-creator
├── product-governance
│   └── study-notes-creator
│
├── frontend-applications
│   ├── study-notes-creator-web
│   ├── study-notes-creator-desktop
│   └── study-notes-creator-mobile
│
├── shared-contracts-and-frontend-libraries
│   ├── study-notes-creator-contracts
│   ├── study-notes-creator-frontend-foundation
│   ├── study-notes-creator-web-ui
│   └── study-notes-creator-native-ui
│
├── backend-microservices
│   ├── study-notes-creator-auth-service
│   ├── study-notes-creator-edge-api
│   ├── study-notes-creator-account-service
│   ├── study-notes-creator-library-service
│   ├── study-notes-creator-ingestion-service
│   ├── study-notes-creator-workflow-service
│   ├── study-notes-creator-media-service
│   ├── study-notes-creator-transcription-service
│   ├── study-notes-creator-ai-notes-service
│   ├── study-notes-creator-notes-service
│   ├── study-notes-creator-search-service
│   ├── study-notes-creator-export-service
│   ├── study-notes-creator-billing-service
│   └── study-notes-creator-notification-service
│
└── platform-infrastructure-and-quality
    ├── study-notes-creator-infrastructure
    ├── study-notes-creator-service-template
    └── study-notes-creator-ai-evals
```

## 1. Product governance repository — 1

| # | Repository | Purpose | Phase | Created |
|---:|---|---|---|:---:|
| 1 | `study-notes-creator` | Product-wide PRD, TRD, glossary, roadmap, build playbook, repository register, cross-system ADRs, service/data/event catalogs, and platform release manifest | Now | ☑ |

This is the current repository. It contains documentation and governance; it does not contain production application or service code.

## 2. Frontend application repositories — 3

| # | Repository | Technology | Purpose | Phase | Created |
|---:|---|---|---|---|:---:|
| 2 | `study-notes-creator-web` | TypeScript, React, Next.js | Responsive website, authenticated web application, and browser BFF | Web MVP | ☐ |
| 3 | `study-notes-creator-desktop` | TypeScript, React DOM, Electron | Desktop shell, native file workflows, updater, offline capabilities, and shared web feature UI | After web | ☐ |
| 4 | `study-notes-creator-mobile` | TypeScript, React Native, Expo | iOS/Android application, native capture, uploads, notifications, reading, and mobile editing | After desktop | ☐ |

## 3. Shared contract and frontend-library repositories — 4

| # | Repository | Published packages/content | Consumers | Phase | Created |
|---:|---|---|---|---|:---:|
| 5 | `study-notes-creator-contracts` | Public OpenAPI, internal contract snapshots, event schemas, generated `@study-notes/api-client`, error/event types, Python Activity-contract package | All clients and backend services | Web MVP | ☐ |
| 6 | `study-notes-creator-frontend-foundation` | `@study-notes/domain`, `auth-core`, `document-model`, `react-core`, `upload-core`, `telemetry`, `design-tokens`, shared TypeScript configuration | Web, desktop, and mobile | Web MVP | ☐ |
| 7 | `study-notes-creator-web-ui` | `@study-notes/web-ui`, `web-editor`, `web-player` | Web and Electron renderer | Web MVP | ☐ |
| 8 | `study-notes-creator-native-ui` | `@study-notes/native-ui`, `native-document` | React Native iOS/Android application | Mobile phase | ☐ |

### Frontend reuse rule

- Web and Electron share the React DOM feature UI, rich-text editor, player, controllers, API client, Auth behavior/ports, domain logic, and design tokens.
- React Native shares the API client, Auth behavior/ports, domain logic, document schema, state machines, hooks/controllers, telemetry, and tokens.
- React Native uses native UI components instead of importing DOM components.
- Each application implements Auth browser-callback and credential-storage ports appropriate to its platform; no shared package stores refresh tokens.
- Applications consume published package versions from a private npm registry; they never import another application repository directly.

## 4. Backend microservice repositories — 14

| # | Repository | Primary ownership | Persistence | Phase | Created |
|---:|---|---|---|---|:---:|
| 9 | `study-notes-creator-auth-service` | End-user identity, credentials, verification/recovery, sessions, token issuance/revocation, signing-key metadata, and authentication audit | Auth database, KMS/Secrets Manager, ephemeral security cache | Web MVP | ☐ |
| 10 | `study-notes-creator-edge-api` | Public product API gateway, first-party token validation/identity propagation, rate limiting, API composition, SSE proxy, public OpenAPI | No business database | Web MVP | ☐ |
| 11 | `study-notes-creator-account-service` | Product profile, preferences, consent, Auth-Identity mapping, and account-deletion requests | Account database | Web MVP | ☐ |
| 12 | `study-notes-creator-library-service` | Artifact library, ownership, metadata, archive/trash state, and resource references | Library database | Web MVP | ☐ |
| 13 | `study-notes-creator-ingestion-service` | URL validation, source adapters, upload sessions, asset registry, and retention | Ingestion database and S3 prefixes | Web MVP | ☐ |
| 14 | `study-notes-creator-workflow-service` | Temporal generation/deletion sagas, job state, progress, retries, cancellation, and compensation | Workflow database and Temporal histories | Web MVP | ☐ |
| 15 | `study-notes-creator-media-service` | Media scanning, probing, audio extraction, normalization, and optional frame sampling | Media execution database and derived S3 prefix | Web MVP | ☐ |
| 16 | `study-notes-creator-transcription-service` | Captions/ASR adapters, transcript versions, timed segments, and speaker data | Transcript database and timing objects | Web MVP | ☐ |
| 17 | `study-notes-creator-ai-notes-service` | Chunking, evidence extraction, outline, drafting, citations, verification, quality reports, and model routing | AI-generation database and temporary objects | Web MVP | ☐ |
| 18 | `study-notes-creator-notes-service` | Canonical note documents, versions, autosave, citations, feedback, and read-only shares | Notes database | Web MVP | ☐ |
| 19 | `study-notes-creator-search-service` | User-scoped artifact, note, and transcript search projection | Search database | Web MVP | ☐ |
| 20 | `study-notes-creator-export-service` | Markdown/PDF export creation, rendering, download, and expiry | Export database and S3 prefix | Web MVP | ☐ |
| 21 | `study-notes-creator-billing-service` | Plans, subscriptions, entitlements, usage reservations, settlement, and Stripe webhooks | Billing database | Private beta | ☐ |
| 22 | `study-notes-creator-notification-service` | In-app, email, desktop, and mobile notification preferences and delivery | Notification database | Web MVP | ☐ |

### Backend ownership rules

- Every microservice has its own repository, release pipeline, runtime identity, database, migrations, and runbook.
- A service never reads or writes another service’s database.
- Synchronous service calls use private versioned APIs.
- Domain events use EventBridge with a dedicated SQS queue and dead-letter queue for each consumer.
- Each event-producing service uses a transactional outbox; every consumer is idempotent.
- Cross-service generation and deletion transactions are coordinated as Temporal sagas.

## 5. Platform, infrastructure, and quality repositories — 3

| # | Repository | Purpose | Phase | Created |
|---:|---|---|---|:---:|
| 23 | `study-notes-creator-infrastructure` | Terraform, AWS environment composition, ECS services, databases, EventBridge/SQS, IAM, networking, Auth KMS/security primitives, dashboards, and environment manifests | Web MVP | ☐ |
| 24 | `study-notes-creator-service-template` | Bootstrap template for consistent FastAPI service structure, security, telemetry, CI, container, and runbook setup | Before services | ☐ |
| 25 | `study-notes-creator-ai-evals` | Permissioned evaluation datasets/manifests, rubrics, runners, regression reports, and model/prompt release gates | Web MVP | ☐ |

## Capabilities that do not need another repository

| Capability | Where it belongs |
|---|---|
| Authentication email delivery | Auth Service owns challenges and verification; an email provider is delivery transport only, not the identity authority |
| Hosted registration/sign-in/recovery UI | Security-owned, minimal UI inside `study-notes-creator-auth-service`; no separate frontend repository |
| Web BFF | Inside `study-notes-creator-web` |
| Temporal control plane | Managed Temporal Cloud; workflow definitions belong to Workflow service and Activities belong to their capability services |
| Event bus and queues | Provisioned by `study-notes-creator-infrastructure`; event producers/consumers remain in owning services |
| PostgreSQL, S3, cache, secrets, networking | Provisioned by Infrastructure; schemas/migrations and storage prefixes remain service-owned |
| Media/AI/export worker pools | Deployment units inside their owning service repositories, not additional repositories |
| Private npm/Python registry | Managed organization service configured by Infrastructure and package repositories |

Authentication is now an explicit first-party Service/repository. It is deliberately separate from Account so credential/session security can evolve without coupling product-profile data.

## Creation summary by delivery stage

| Stage | Repositories introduced | Count | Running total |
|---|---|---:|---:|
| Current planning | Product governance | 1 | 1 |
| Platform bootstrap | Contracts, frontend foundation, web UI, infrastructure, service template, AI evaluations | 6 | 7 |
| Web MVP backend | Auth, Edge, Account, Library, Ingestion, Workflow, Media, Transcription, AI Notes, Notes, Search, Export, Notification | 13 | 20 |
| Web MVP client | Web application | 1 | 21 |
| Private beta | Billing service | 1 | 22 |
| Desktop phase | Desktop application | 1 | 23 |
| Mobile phase | Native UI library and mobile application | 2 | 25 |

## Recommended creation order

1. `study-notes-creator-service-template`
2. `study-notes-creator-infrastructure`
3. `study-notes-creator-contracts`
4. `study-notes-creator-frontend-foundation`
5. `study-notes-creator-web-ui`
6. `study-notes-creator-ai-evals`
7. `study-notes-creator-auth-service`
8. `study-notes-creator-account-service`
9. `study-notes-creator-library-service`
10. `study-notes-creator-ingestion-service`
11. `study-notes-creator-media-service`
12. `study-notes-creator-transcription-service`
13. `study-notes-creator-ai-notes-service`
14. `study-notes-creator-notes-service`
15. `study-notes-creator-workflow-service`
16. `study-notes-creator-search-service`
17. `study-notes-creator-export-service`
18. `study-notes-creator-notification-service`
19. `study-notes-creator-edge-api`
20. `study-notes-creator-web`
21. `study-notes-creator-billing-service`
22. `study-notes-creator-desktop`
23. `study-notes-creator-native-ui`
24. `study-notes-creator-mobile`

The current `study-notes-creator` governance repository is already present, so the creation list contains the remaining **24 repositories**.

## Tracking status legend

- ☐ Not created
- ◐ Created but not production-ready
- ☑ Active/ready for its current phase
- ⛔ Retired or replaced

Update the `Created` column and add repository links as repositories are provisioned.

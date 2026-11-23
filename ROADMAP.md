# Study Notes Creator: Product and Engineering Roadmap

**Document status:** Active roadmap  
**Version:** 1.0  
**Last updated:** 23 November 2026  
**Current milestone:** M0 — Product control plane  
**Product requirements:** [PRD.md](./PRD.md)  
**Technical baseline:** [TRD.md](./TRD.md)  
**Delivery process:** [BUILD_PLAYBOOK.md](./BUILD_PLAYBOOK.md)  
**Repository register:** [repos.md](./repos.md)  
**Service boundaries:** [SERVICE_CATALOG.md](./SERVICE_CATALOG.md)

---

## 1. Roadmap objective

Build Study Notes Creator as a sequence of understandable, testable vertical slices while activating repositories only when a validated milestone needs them.

The first strategic objective is:

> A signed-in learner uploads a supported local video and receives an editable, timestamp-grounded study note through a durable and observable workflow.

Web is the first product. Desktop follows a stable public web release. Mobile follows the desktop foundation. Advanced learning features begin only after the core note-quality and retention loop is proven.

This roadmap is gated by outcomes, not arbitrary dates. Target dates will be added only after team capacity, provider constraints, and measured delivery velocity are known.

---

## 2. Roadmap principles

1. Do not create all 24 repositories at once.
2. Build one end-to-end user outcome at a time.
3. Use deterministic mocks before integrating expensive or uncertain providers.
4. Approve ownership, contracts, failures, and acceptance criteria before implementation.
5. Activate a repository only with an approved first task and named owner.
6. Keep every merged cross-repository state backward compatible.
7. Do not enable YouTube generation until a compliant acquisition path is approved.
8. Do not begin desktop or mobile until the core web contracts and study experience are stable.
9. Treat prompts, models, data retention, infrastructure, and operations as product work—not implementation details.
10. A milestone exits only when its behavior is understood, verified, operable, and documented.

---

## 3. Status legend

| Symbol | Status | Meaning |
|:---:|---|---|
| ○ | Not started | No active implementation work |
| ◐ | In progress | Work has begun but exit criteria are not satisfied |
| ◉ | Complete | Exit criteria and ownership review passed |
| △ | Blocked | External decision, dependency, or risk prevents meaningful progress |
| ◇ | Deferred | Intentionally outside the current sequence |

Repository state continues to use the legend in `repos.md`.

---

## 4. Milestone overview

| Milestone | Outcome | New repositories | Running total | Status |
|---|---|---:|---:|:---:|
| M0 | Product control plane and approved build system | 0; governance repository already exists | 1 | ◐ |
| M1 | Engineering golden path and platform sandbox | 6 | 7 | ○ |
| M2 | Distributed web walking skeleton with deterministic fake note | 6 | 13 | ○ |
| M3 | Real local upload and media preparation | 2 | 15 | ○ |
| M4 | Real timestamped transcription | 1 | 16 | ○ |
| M5 | Evidence-grounded AI study notes | 1 | 17 | ○ |
| M6 | Study-ready web internal alpha | 3 | 20 | ○ |
| M7 | Operable private beta and monetization | 1 | 21 | ○ |
| M8 | Public web launch | 0 | 21 | ○ |
| M9 | Electron desktop release | 1 | 22 | ○ |
| M10 | React Native mobile release | 2 | 24 | ○ |
| M11 | Learning and collaboration expansion | 0 initially | 24 | ◇ |

The running total includes the current `study-notes-creator` governance repository.

---

## 5. Repository activation map

| Repository | Activation milestone | Reason for activation |
|---|---|---|
| `study-notes-creator` | M0 | Product governance and system decisions |
| `study-notes-creator-service-template` | M1 | Standard service creation and controls |
| `study-notes-creator-infrastructure` | M1 | Sandbox, CI/CD, identity, networking, data and messaging primitives |
| `study-notes-creator-contracts` | M1 | Public/internal API, event and Activity contracts |
| `study-notes-creator-frontend-foundation` | M1 | Shared domain, API, document, state and token packages |
| `study-notes-creator-web-ui` | M1 | Shared React DOM UI/editor/player foundation |
| `study-notes-creator-ai-evals` | M1 | Evaluation harness and permissioned benchmark manifests; expanded in M4–M5 |
| `study-notes-creator-web` | M2 | First user-facing application |
| `study-notes-creator-edge-api` | M2 | Stable public API and authentication boundary |
| `study-notes-creator-account-service` | M2 | Product profile and preferences |
| `study-notes-creator-library-service` | M2 | Artifact ownership and library state |
| `study-notes-creator-workflow-service` | M2 | Durable generation workflow and progress |
| `study-notes-creator-notes-service` | M2 | Canonical note materialization and reads |
| `study-notes-creator-ingestion-service` | M3 | Upload sessions, sources and retention |
| `study-notes-creator-media-service` | M3 | Untrusted media validation and preparation |
| `study-notes-creator-transcription-service` | M4 | Timestamped transcript ownership |
| `study-notes-creator-ai-notes-service` | M5 | Evidence extraction, generation and verification |
| `study-notes-creator-search-service` | M6 | User-scoped search projection |
| `study-notes-creator-export-service` | M6 | Markdown/PDF exports |
| `study-notes-creator-notification-service` | M6 | In-app/email readiness and failure notifications |
| `study-notes-creator-billing-service` | M7 | Plans, entitlements, reservations and settlement |
| `study-notes-creator-desktop` | M9 | Electron desktop product |
| `study-notes-creator-native-ui` | M10 | Native design components and document renderer |
| `study-notes-creator-mobile` | M10 | React Native iOS/Android product |

Activation means creating the repository with its first approved vertical-slice task. It does not mean implementing its full eventual scope.

---

## 6. M0 — Product control plane

**Status:** ◐ In progress  
**Outcome:** Product decisions, architecture, repositories, AI build process, and roadmap are explicit and internally consistent.

### Completed

- [x] Product requirements documented in `PRD.md`.
- [x] Technical architecture documented in `TRD.md`.
- [x] Repository inventory documented in `repos.md`.
- [x] AI-assisted delivery process documented in `BUILD_PLAYBOOK.md`.
- [x] Milestone sequence documented in `ROADMAP.md`.

### Remaining deliverables

- [x] Create `GLOSSARY.md` with stable product/domain terminology.
- [x] Create `SERVICE_CATALOG.md` with every service mission, non-goals, owner and dependencies.
- [ ] Create `DATA_OWNERSHIP.md` mapping each durable datum to one service.
- [ ] Create `EVENT_CATALOG.md` with the initial event envelope and planned facts.
- [ ] Create `adr/` and record accepted cross-system decisions.
- [ ] Create reusable templates for service specifications, feature specifications, ADRs, AI tasks and pull-request handoffs.
- [ ] Define milestone and feature tracking mechanism.
- [ ] Name an owner for each repository required through M2.

### Initial ADRs

- [ ] ADR-001: Polyrepo and microservice architecture.
- [ ] ADR-002: Python/FastAPI backend services.
- [ ] ADR-003: Next.js web, Electron desktop and React Native mobile clients.
- [ ] ADR-004: Database per service.
- [ ] ADR-005: Temporal cross-service workflow orchestration.
- [ ] ADR-006: EventBridge/SQS, transactional outbox and idempotent consumers.
- [ ] ADR-007: Shared frontend package boundaries and private registry.
- [ ] ADR-008: AWS ECS/Fargate production baseline.

### Decisions required before M1 exits

- Source-control organization and naming policy.
- Private npm/Python/container registry.
- AWS organization/account/environment strategy.
- Initial AWS region and data-residency assumption.
- Managed OIDC provider.
- Temporal Cloud account/namespace approach.
- CI/CD platform, secret management and approval policy.
- Initial engineering owner/on-call model.

### Exit criteria

- Product, architecture, repository and build documents do not materially conflict.
- Every M1/M2 repository has a clear purpose and provisional owner.
- Initial ADRs are accepted.
- First walking-skeleton feature is specified and satisfies the Definition of Ready.
- The owner can explain the planned end-to-end control flow and its major risks.

---

## 7. M1 — Engineering golden path and platform sandbox

**Status:** ○ Not started  
**Outcome:** A new service or frontend package can be created, tested, published, deployed, observed and rolled back consistently.

### Repositories activated

- `study-notes-creator-service-template`
- `study-notes-creator-infrastructure`
- `study-notes-creator-contracts`
- `study-notes-creator-frontend-foundation`
- `study-notes-creator-web-ui`
- `study-notes-creator-ai-evals`

### Deliverables

#### Platform sandbox

- [ ] Isolated development/staging cloud environment.
- [ ] Terraform state, review and deployment workflow.
- [ ] ECS/Fargate sample service with private networking and Service Connect.
- [ ] PostgreSQL logical database, migrations and backup/restore exercise.
- [ ] S3 private/quarantine/derived/export prefixes and lifecycle policies.
- [ ] EventBridge bus, SQS consumer queue and DLQ.
- [ ] Temporal development and managed staging namespaces.
- [ ] Secrets, KMS, workload identity and least-privilege IAM baseline.
- [ ] OpenTelemetry logs, metrics and traces with content-redaction tests.

#### Service golden path

- [ ] FastAPI service template with health/readiness, configuration and graceful shutdown.
- [ ] Domain/application/transport/persistence boundaries.
- [ ] Alembic migrations and service-owned database role.
- [ ] Transactional outbox and idempotent consumer example.
- [ ] Temporal Activity example.
- [ ] CI, container build, SBOM, scans, staging deploy and rollback.
- [ ] `AGENTS.md`, `SERVICE.md`, `README.md` and `RUNBOOK.md` templates.

#### Contract and frontend package path

- [ ] OpenAPI/event/Activity-schema validation and compatibility pipeline.
- [ ] Generated TypeScript client and Python contract package.
- [ ] Private package publication with canary/stable tags.
- [ ] Shared TypeScript/domain/document/token package builds.
- [ ] React DOM design-system foundation with accessibility checks and Storybook.
- [ ] Minimal Next.js, Electron and React Native consumer fixtures for package compatibility.

#### Evaluation foundation

- [ ] Evaluation dataset manifest format and secure media references.
- [ ] Rubric/version/report structure.
- [ ] Deterministic fake provider for pipeline and CI tests.
- [ ] Cost/latency/quality result schema.

### Explicitly not included

- User-facing application.
- Real video processing.
- Real transcription or note generation.
- Production traffic.

### Exit criteria

- A sample service deploys from a clean repository and is reachable only through approved paths.
- Its database migration, outbox event, consumer, trace and alert are demonstrably functional.
- A failed event reaches a DLQ and can be redriven idempotently.
- A package can be published and consumed from another repository without source linking.
- The sample service can be rolled back and its database restored in a rehearsal.
- AI can implement a small template-based task and produce the playbook handoff successfully.

---

## 8. M2 — Distributed web walking skeleton

**Feature:** F-001 — Deterministic artifact-to-note skeleton  
**Status:** ○ Not started  
**Outcome:** A user signs in, creates an artifact, starts a durable fake generation, sees progress, and opens a deterministic fake note in the web app.

### Repositories activated

- `study-notes-creator-web`
- `study-notes-creator-edge-api`
- `study-notes-creator-account-service`
- `study-notes-creator-library-service`
- `study-notes-creator-workflow-service`
- `study-notes-creator-notes-service`

### End-to-end flow

```text
Sign in
→ create profile/library artifact
→ request generation
→ validate internal-alpha access
→ run deterministic Temporal fake Activities
→ publish ordered progress
→ materialize a versioned fake note
→ update library projection
→ display note and citations placeholder
```

### Deliverables

- [ ] OIDC sign-in through the web BFF and Edge API.
- [ ] Account profile creation.
- [ ] Artifact creation and library listing.
- [ ] Versioned public API client consumed by web.
- [ ] Durable Temporal workflow with deterministic fake stages.
- [ ] Job status, ordered events, SSE reconnect and polling fallback.
- [ ] Canonical document schema and deterministic note fixture.
- [ ] Notes read API and responsive read-only workspace.
- [ ] Event-driven artifact ready-state projection.
- [ ] End-to-end traces across web, Edge, services, Temporal and events.
- [ ] Synthetic E2E and service-unavailable/duplicate-event tests.

### Key design work

- Account, Library, Workflow and Notes `SERVICE.md` files.
- Artifact, generation, job-event and note-version state machines.
- Public API v1 skeleton.
- Initial event types and Activity contracts.
- Web platform adapters and package composition.

### Explicitly not included

- File upload or actual media.
- ASR or model providers.
- Billing, search, export or external notification.
- Editing beyond any minimum schema/viewer test controls.

### Exit criteria

- The full control path survives API/worker deployment restarts.
- Duplicate submissions/events do not create duplicate jobs or notes.
- SSE reconnect recovers missed events.
- A service outage produces an understandable recoverable state.
- One user cannot access another user’s artifact or note.
- The published package/service versions are captured in a release manifest.
- The owner can explain every service and transition in the walking skeleton.

---

## 9. M3 — Real local upload and media preparation

**Feature:** F-002 — Local video ingestion  
**Status:** ○ Not started  
**Outcome:** A learner uploads one supported local video format; the platform validates, prepares and cleans up the media durably.

### Repositories activated

- `study-notes-creator-ingestion-service`
- `study-notes-creator-media-service`

### Initial constrained support

- Local upload only.
- One documented container/codec combination first, expanded after test evidence.
- Conservative configurable size and duration limits.
- One audio track.
- No visual understanding.

### Deliverables

- [ ] Local browser preflight and rights confirmation.
- [ ] Direct multipart S3 upload with progress, pause/resume and expiry.
- [ ] Upload checksum, ownership and completion validation.
- [ ] Quarantine, malware scanning and safe media probing.
- [ ] Audio presence, duration, format and resource-limit validation.
- [ ] Sandboxed FFmpeg audio extraction/normalization.
- [ ] Durable ingestion/media Activities with heartbeat and cancellation.
- [ ] Temporary asset registry and audited lifecycle deletion.
- [ ] Stable actionable errors for unsupported/corrupt/silent/oversized sources.
- [ ] Malicious-media and upload-interruption test corpus.

### Key decisions

- Initial file formats, size and duration limits.
- Malware scanner and FFmpeg sandbox strategy.
- Multipart part size/concurrency and upload-expiry rules.
- Raw and derived media retention defaults.

### Exit criteria

- A supported real file reaches `MEDIA_PREPARED` and exposes an immutable audio-asset reference.
- Unsupported and malicious files are rejected without unsafe processing.
- Upload and processing resume/retry without duplicate effects.
- Canceling a job cleans temporary assets and settles state.
- Retention cleanup is measurable and meets the policy deadline.
- Application servers never proxy uploaded file bytes.

---

## 10. M4 — Real timestamped transcription

**Feature:** F-003 — Transcript creation  
**Status:** ○ Not started  
**Outcome:** Prepared audio produces a permissioned, normalized, timestamped transcript through one evaluated provider/language path.

### Repository activated

- `study-notes-creator-transcription-service`

### Deliverables

- [ ] Transcription service ownership and transcript state machine.
- [ ] Provider-neutral ASR adapter.
- [ ] One selected launch-language/provider configuration.
- [ ] Segment timing, language, speaker metadata where useful and content hash.
- [ ] Compressed word timing where provided.
- [ ] Durable Activity timeouts, provider rate limits, retries and cancellation.
- [ ] Transcript paging and authorized internal content-snapshot endpoint.
- [ ] Quality warning for poor audio/low-confidence sources.
- [ ] Retention, deletion, export and logging behavior.
- [ ] Permissioned transcription benchmark and regression report.

### Key decisions

- ASR provider and fallback strategy.
- Launch language for the first slice.
- Segment normalization rules.
- Provider retention/privacy configuration.
- Transcription quality and latency thresholds.

### Exit criteria

- Advertised audio slices meet the approved timing and transcription-quality thresholds.
- Transcript segments remain ordered, bounded by source duration and immutable within a version.
- Provider failures and rate limits recover within bounded policy.
- Raw transcript content does not enter logs, events or general analytics.
- Deletion removes transcript content and derived timing objects as specified.

---

## 11. M5 — Evidence-grounded AI study notes

**Feature:** F-004 — Grounded Detailed note  
**Status:** ○ Not started  
**Outcome:** A real transcript produces a structured Detailed note whose material claims link to valid source evidence and timestamps.

### Repository activated

- `study-notes-creator-ai-notes-service`

`study-notes-creator-ai-evals` expands from its M1 harness into a release-gating benchmark system.

### Initial constrained support

- Detailed preset only.
- One launch language.
- Transcript/audio evidence only; visual understanding deferred.
- External research disabled.
- One bounded repair pass.

### Deliverables

- [ ] Transcript chunking with preserved segment/time references.
- [ ] Typed evidence extraction.
- [ ] Evidence-bounded outline and section drafting.
- [ ] Citation assembly from evidence IDs rather than model-generated timestamps.
- [ ] Deterministic document-schema validation.
- [ ] Model-assisted grounding, coverage and clarity checks.
- [ ] One bounded repair pass and partial/failure policy.
- [ ] Prompt/model/configuration versioning.
- [ ] Per-job token, call, cost, time and retry budgets.
- [ ] Quality report persisted with the generation.
- [ ] Benchmark by content type, source length, audio quality and language.
- [ ] Web display with timestamp seek and transcript evidence highlighting.

### Key decisions

- Text-model provider/model routing.
- Chunk and evidence schemas.
- Quality rubric and launch thresholds.
- Citation density and acceptable evidence window.
- Generated explanation/provenance presentation.
- Per-job cost budget.

### Exit criteria

- Model-produced timestamps or unknown evidence IDs cannot enter a ready note.
- The benchmark meets approved grounding, coverage, clarity and citation thresholds.
- Timestamp links land in reviewer-accepted evidence windows at the required rate.
- A model/prompt change cannot bypass evaluation gates.
- The workflow degrades honestly for insufficient evidence.
- Cost and latency meet the approved envelope for a 60-minute source.

---

## 12. M6 — Study-ready web internal alpha

**Feature group:** F-005 study workspace and F-006 web readiness  
**Status:** ○ Not started  
**Outcome:** Internal users can reliably generate, verify, edit, find, export and delete useful study notes.

### Repositories activated

- `study-notes-creator-search-service`
- `study-notes-creator-export-service`
- `study-notes-creator-notification-service`

### Product capabilities

- [ ] Quick, Detailed and Comprehensive generation presets.
- [ ] Audience, output language and optional focus controls for evaluated combinations.
- [ ] Library search, sort, filter, archive and trash.
- [ ] Rich editing, autosave, optimistic concurrency and conflict recovery.
- [ ] Non-destructive regeneration and recoverable versions.
- [ ] Transcript search, outline and player/citation coordination.
- [ ] Markdown and PDF export.
- [ ] In-app and email readiness/failure notification.
- [ ] Feedback for incorrect, missing, unclear or mistimed content.
- [ ] Artifact/account deletion sagas and retention audit.
- [ ] User settings, privacy controls and account export/deletion.

### Operational readiness

- [ ] Accessibility review of the complete core flow.
- [ ] Supported browser/device matrix.
- [ ] Performance/load test against the TRD sizing envelope.
- [ ] Authorization, SSRF, XSS, upload/media and prompt-injection security tests.
- [ ] Restore, deletion-reconciliation and workflow recovery exercises.
- [ ] Support metadata view without routine private-content access.
- [ ] Dashboards, alerts and runbooks for every paging condition.
- [ ] Cost dashboard by source hour, preset and artifact.

### Explicitly not included

- Public self-service signup at scale.
- Paid subscription.
- Unapproved public YouTube extraction.
- Desktop/mobile clients.
- Grounded Q&A, flashcards or collaboration.

### Exit criteria

- PRD internal-alpha acceptance criteria pass on controlled sources.
- TRD SLOs are met at internal-alpha load or deviations are approved explicitly.
- No unresolved critical security, privacy, accessibility or rights issue.
- Acknowledged edits survive failures and conflicts without silent loss.
- Deletion completes across services and restored data cannot reappear.
- Support and on-call can diagnose failures using safe metadata.
- Internal users prefer the generated study artifact to a cleaned transcript on the benchmark/usefulness study.

---

## 13. M7 — Operable private beta and monetization

**Feature group:** F-007 private beta  
**Status:** ○ Not started  
**Outcome:** Invited users can use the web product under controlled quotas while the team measures quality, retention, cost and willingness to pay.

### Repository activated

- `study-notes-creator-billing-service`

### Deliverables

- [ ] Subscription, entitlement, usage reservation/settlement and refund policy.
- [ ] Stripe sandbox then production integration with idempotent webhooks.
- [ ] Usage estimate before generation and transparent allowance display.
- [ ] Read-only revocable share links.
- [ ] Approved hosted-video provider adapters.
- [ ] YouTube metadata/embed and generation only if the compliance gate passes.
- [ ] Broader real-world failure diagnostics.
- [ ] Beta invitation, feedback and support workflow.
- [ ] Rights-holder/privacy/abuse reporting and response process.
- [ ] Additional language or source support only after slice-specific evaluation passes.
- [ ] Unit economics and provider-capacity dashboards.

### Key decisions

- Billing unit, plans, free allowance and plan limits.
- Beta cohort and launch geography.
- Hosted/YouTube source methods approved by legal/provider policy.
- Support access and escalation procedure.
- Beta quality, activation, retention and cost targets.

### Exit criteria

- Invited-user activation, quality, completion and retention targets are met or a documented decision explains the next experiment.
- Product-caused failures settle usage correctly.
- Unit economics support the proposed packaging.
- Rights/privacy/security operations function within published response expectations.
- No critical contract or operational issue blocks scaling beyond the invited cohort.

---

## 14. M8 — Public web launch

**Status:** ○ Not started  
**Outcome:** The web product is safe, supportable and commercially operable for public individual users in approved markets.

### Deliverables

- [ ] Production pricing, checkout, subscription management and user communications.
- [ ] Published support matrix, limits, retention, privacy and refund behavior.
- [ ] WAF, abuse controls, quota enforcement and cost kill switches.
- [ ] External penetration test and resolved critical findings.
- [ ] Capacity validation and provider quota headroom.
- [ ] On-call ownership, incident process and public support workflow.
- [ ] Takedown, rights-holder, privacy-request and abuse-response workflows.
- [ ] Disaster-recovery and production-restore rehearsal.
- [ ] Public product analytics and guardrail dashboards.
- [ ] Rollout plan with cohort/canary percentages and rollback triggers.

### Exit criteria

- All web MVP technical acceptance criteria in the TRD pass.
- Public-launch quality, reliability, privacy, rights, security and accessibility gates pass.
- Production recovery meets agreed RPO/RTO.
- On-call and support are staffed for the launch window.
- Business and product guardrails remain healthy through the observation period.

---

## 15. M9 — Electron desktop product

**Status:** ○ Not started  
**Outcome:** Users can perform long study sessions and native file workflows in a secure desktop client using stable web-era contracts.

### Repository activated

- `study-notes-creator-desktop`

### Deliverables

- [ ] Local packaged React DOM renderer consuming published shared packages.
- [ ] System-browser OIDC with secure OS credential storage.
- [ ] Typed and validated main/preload/renderer IPC.
- [ ] Native file selection, drag/drop and background-aware upload.
- [ ] Study workspace, editor, search, export and notifications.
- [ ] Signed builds, installer, auto-update and rollback channels.
- [ ] Crash reporting without private content.
- [ ] Windows/macOS support matrix; Linux support based on product decision.
- [ ] Optional offline reading only after the online flow is stable.

### Exit criteria

- Electron security review passes.
- Desktop consumes public APIs and packages without importing the web repository.
- Signing, update, rollback and crash recovery are tested on supported OS versions.
- A user can start on web and continue on desktop without data loss.

---

## 16. M10 — React Native mobile product

**Status:** ○ Not started  
**Outcome:** Users can capture sources, receive notifications, read/search notes and perform safe lightweight editing on iOS and Android.

### Repositories activated

- `study-notes-creator-native-ui`
- `study-notes-creator-mobile`

### Deliverables

- [ ] Native design system and canonical-document renderer.
- [ ] System-browser OIDC/PKCE and secure credential storage.
- [ ] URL share-sheet capture and local file/document picker.
- [ ] Background-capable upload within platform limitations.
- [ ] Push notifications.
- [ ] Library, reading, transcript/citation navigation and search.
- [ ] Safe limited editing with explicit capability differences from web.
- [ ] App lifecycle, offline/reconnect and conflict behavior.
- [ ] Store signing, release, phased rollout, crash monitoring and rollback strategy.

### Exit criteria

- Mobile security, accessibility and store-readiness reviews pass.
- iOS/Android share the intended platform-neutral packages without importing DOM code.
- Cross-device continuation and edit-conflict recovery pass E2E tests.
- Background upload/notification behavior matches published platform expectations.

---

## 17. M11 — Learning and collaboration expansion

**Status:** ◇ Deferred pending evidence  
**Outcome:** Expand from note creation into a broader study system only when core quality and retention justify it.

Candidate initiatives:

- Grounded source Q&A.
- Flashcards, quizzes and spaced review.
- Section-level regeneration and version comparison.
- Visual understanding and generated explanatory diagrams.
- Collections, courses and study plans.
- Collaboration, comments and team/educator workspaces.
- Approved external enrichment with separate citations.
- Institutional administration and policy controls.

Each candidate requires a separate product discovery brief. No candidate is automatically committed by appearing here.

---

## 18. Cross-cutting workstreams

These workstreams run through multiple milestones and cannot be postponed to launch.

### 18.1 Product discovery

- Interview and observe target learners.
- Validate note structure and depth.
- Measure usefulness, trust and repeat behavior.
- Keep the initial target segment narrow enough for reliable evaluation.

### 18.2 AI quality and evaluation

- Permissioned benchmark corpus.
- Language/content/audio-quality slices.
- Grounding, coverage, citation, clarity and cost gates.
- Prompt/model versioning and regression policy.
- Production feedback linked to safe identifiers and configuration versions.

### 18.3 Rights and provider compliance

- YouTube and hosted-source approval.
- Attribution, player and caption behavior.
- User rights attestation.
- Takedown and rights-holder process.
- Provider data-retention and training terms.

### 18.4 Security and privacy

- Threat modeling at each new boundary.
- Least-privilege workload identities.
- Upload/media sandbox.
- Service authorization and tenant isolation.
- Retention, deletion, restore and support-access controls.
- Electron/mobile platform-specific review.

### 18.5 Reliability and operations

- SLOs, dashboards, alerts and runbooks.
- Workflow/activity replay and crash recovery.
- Event duplication, gaps, DLQ and projection rebuild.
- Backups, restore and deletion reconciliation.
- Incident response and post-incident learning.

### 18.6 Accessibility and internationalization

- WCAG 2.2 AA web target.
- Keyboard, screen-reader, zoom and reduced-motion coverage.
- Native accessibility for desktop/mobile.
- Language support advertised only after evaluation passes.

### 18.7 Cost and unit economics

- Cost per source hour and completed/meaningfully studied artifact.
- Model/ASR/provider routing experiments.
- Storage and retention cost.
- Queue/concurrency and abuse controls.
- Plan packaging informed by actual workload cost.

---

## 19. Roadmap decision gates

| Gate | Required before | Decision/evidence |
|---|---|---|
| G1 Architecture control | M1 | Initial ADRs, owners, registry, cloud/environment decisions |
| G2 Platform readiness | M2 | Golden-path deploy, package publish, observability and rollback proof |
| G3 Distributed control proof | M3 | Walking-skeleton durability, authz, event and workflow recovery |
| G4 Media safety | M4 | Upload limits, sandbox, retention and malicious-media tests |
| G5 Transcription quality | M5 | Approved provider/language benchmark |
| G6 Grounded-note quality | M6 | Grounding/citation/coverage/cost thresholds |
| G7 Internal-alpha readiness | M7 | Web UX, reliability, security, deletion, accessibility and support |
| G8 Private-beta viability | M8 | Retention, quality, unit economics, rights and operations |
| G9 Public-launch readiness | M9 | Public web acceptance and stable client contracts |
| G10 Desktop readiness | M10 | Stable desktop-used APIs/packages and secure update path |
| G11 Mobile release readiness | M10 exit | Native security/accessibility, store, sync and phased-rollout evidence |

A failed gate does not automatically mean “work harder.” It may require reducing scope, changing a provider, revising an architecture decision or stopping an unsupported path.

---

## 20. Current risk register

| Risk | Earliest milestone | Mitigation | Status |
|---|---|---|:---:|
| Repository/microservice overhead exceeds available ownership | M1 | Activate just in time; standard template; one vertical slice; automate compatibility | Open |
| Architecture is implemented before user value is validated | M2 | Deterministic walking skeleton; early note prototypes and user feedback | Open |
| Public YouTube URL cannot be processed compliantly | M7 | Compliance gate; official metadata/embed; authorized upload/transcript fallback | Open |
| Notes are fluent but insufficiently grounded | M5 | Evidence-first pipeline, deterministic citation linkage and release evaluations | Open |
| Long videos create unacceptable cost/latency | M3–M5 | Constrained limits, measurement, budgets, provider routing and background processing | Open |
| Private content leaks through logs/events/providers | M1 onward | Redaction tests, reference-only events, provider policies and least privilege | Open |
| Cross-repository contract drift breaks releases | M2 onward | Central contracts, consumer tests, additive changes and compatibility telemetry | Open |
| AI-generated implementation becomes poorly understood | M0 onward | Playbook gates, owner explanation, ADRs, independent review and small tasks | Open |
| User edits are lost during sync/regeneration | M6 | Optimistic revisions, local recovery, immutable versions and conflict UX | Open |
| Desktop/mobile prematurely distract from web quality | M6 onward | Gated sequence; no native activation before public web stability | Open |

Review risks at every milestone exit. Add a risk when evidence changes, not only when an incident occurs.

---

## 21. Metrics by stage

| Stage | Primary evidence |
|---|---|
| M0–M1 | Decision closure, golden-path reproducibility, rollback and owner understanding |
| M2 | End-to-end success, job recovery, authorization isolation and trace completeness |
| M3 | Upload completion, media rejection correctness, cleanup and processing reliability |
| M4 | Transcript accuracy/timing, provider failure rate, latency and cost per source hour |
| M5 | Grounding, coverage, citation accuracy, note usefulness, generation latency and cost |
| M6 | Internal activation, edit durability, search/export usage, SLOs and support burden |
| M7 | Beta activation, retention, positive feedback, unit economics and rights/privacy operations |
| M8 | Public conversion, retention, reliability, gross margin and incident/support rates |
| M9–M10 | Cross-device continuation, crash-free use, sync conflicts and platform adoption |

Exact thresholds must be recorded before entering the milestone whose exit depends on them.

---

## 22. Explicit non-priorities

Until their roadmap milestone or an approved change:

- Do not build all repositories as empty shells.
- Do not introduce Kubernetes.
- Do not self-host foundation models or ASR.
- Do not build real-time collaboration.
- Do not build grounded chat, quizzes or flashcards.
- Do not build multi-region active/active deployment.
- Do not support arbitrary authenticated/private hosted URLs.
- Do not scrape or download YouTube media through an unapproved method.
- Do not build a universal React/React Native component abstraction at the expense of native UX.
- Do not build desktop/mobile before stable web contracts and core note quality.

---

## 23. Roadmap change control

A roadmap change must state:

- Evidence or decision that caused the change.
- Milestones, repositories and contracts affected.
- New dependencies and risks.
- Work stopped, added or reordered.
- Impact on already activated repositories.
- Whether PRD, TRD, playbook, ADRs or `repos.md` must change.

Minor task sequencing may be updated directly. Adding/removing a service, changing platform order, expanding MVP scope or bypassing a gate requires explicit owner approval and the appropriate ADR/product update.

Update the roadmap status at least at:

- Milestone start.
- Gate decision.
- Milestone exit.
- Material scope/architecture change.
- Blocker lasting beyond the normal task cycle.

---

## 24. Next actions

Complete these in order:

1. [ ] Review and approve this roadmap.
2. [x] Create `GLOSSARY.md`.
3. [x] Create `SERVICE_CATALOG.md`.
4. [ ] Create `DATA_OWNERSHIP.md`.
5. [ ] Create `EVENT_CATALOG.md`.
6. [ ] Create `adr/` and draft ADR-001 through ADR-008.
7. [ ] Create the service, feature, task and handoff templates described in the playbook.
8. [ ] Create `features/F-001-deterministic-walking-skeleton/` and specify it fully.
9. [ ] Resolve the M1 environment, registry, OIDC, Temporal and CI decisions.
10. [ ] Activate `study-notes-creator-service-template` with its first approved task.

No production service repository should be created before actions 1–9 make its ownership and first task clear.

---

## 25. Milestone completion record

Use this table as the concise roadmap status page:

| Milestone | Started | Gate approved | Completed | Release/evidence | Owner notes |
|---|---|---|---|---|---|
| M0 Product control plane | 23 Nov 2026 | — | — | PRD/TRD/repos/playbook/roadmap/glossary/service catalog | Remaining governance artifacts listed above |
| M1 Golden path | — | — | — | — | — |
| M2 Walking skeleton | — | — | — | — | — |
| M3 Local upload/media | — | — | — | — | — |
| M4 Transcription | — | — | — | — | — |
| M5 Grounded notes | — | — | — | — | — |
| M6 Internal alpha | — | — | — | — | — |
| M7 Private beta | — | — | — | — | — |
| M8 Public web | — | — | — | — | — |
| M9 Desktop | — | — | — | — | — |
| M10 Mobile | — | — | — | — | — |
| M11 Expansion | — | — | — | — | Deferred pending evidence |

---

## Appendix A: Milestone exit review template

```markdown
# <Milestone> Exit Review

## Outcome delivered
What now works for a user or the engineering system?

## Exit criteria
Evidence for each criterion.

## Repositories activated
Links, owners and readiness states.

## Contracts and data ownership
Versions and any approved changes.

## Quality and operational evidence
Tests, evaluations, SLOs, security, accessibility, recovery and cost.

## Incidents and lessons
What failed or changed our understanding?

## Known limitations and accepted risks
What remains constrained and why?

## Owner understanding
Plain-language system explanation and remaining questions.

## Gate decision
Proceed | Repeat/repair | Reduce scope | Stop

## Next milestone authorization
Approved scope, owners and constraints.
```

## Appendix B: Roadmap item template

```markdown
## <Milestone or feature>

**Status:** ○ | ◐ | ◉ | △ | ◇
**Outcome:** <observable result>

### Repositories activated
- <repository>

### Deliverables
- [ ] <deliverable>

### Decisions required
- <decision and owner>

### Explicitly not included
- <non-goal>

### Exit criteria
- <observable evidence>
```

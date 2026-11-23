# Study Notes Creator: Canonical Glossary

**Document status:** Active terminology standard  
**Version:** 1.0  
**Last updated:** 23 November 2026  
**Applies to:** Product documents, contracts, repositories, code, telemetry, support, and AI instructions  
**Related documents:** [PRD.md](./PRD.md), [TRD.md](./TRD.md), [BUILD_PLAYBOOK.md](./BUILD_PLAYBOOK.md), [ROADMAP.md](./ROADMAP.md), [repos.md](./repos.md)

---

## 1. Purpose

This glossary defines the canonical language for Study Notes Creator. Product specifications, service contracts, code, database fields, events, user interfaces, analytics, support documentation, and AI-generated work should use these terms consistently.

If a term is missing or ambiguous:

1. Do not silently invent a competing definition.
2. Propose the term and definition in this document.
3. Identify conflicting existing usage.
4. Update affected contracts and documents after approval.

Contract and code identifiers may use naming conventions such as `snake_case`, `camelCase`, or `PascalCase`, but their meaning must match this glossary.

---

## 2. Core product model

### Source

The user-supplied origin material from which study notes are generated. A Source can be:

- A local uploaded video.
- A supported hosted-video URL.
- A supported YouTube URL.

A Source is not a Note, Artifact, Media Asset, or Transcript.

### Source locator

The user-provided reference used to identify a Source, such as a URL or local file selection. A locator may contain sensitive query parameters and must not be logged without sanitization.

### Source type

The acquisition category of a Source:

- `upload`
- `hosted_url`
- `youtube`

Source type does not prove access or processing rights.

### Source metadata

Safe descriptive information about a Source, such as title, duration, thumbnail reference, provider, language hint, and public provider ID.

Source metadata is not the media content itself.

### Source rights attestation

The user’s confirmation that they are authorized to process the supplied material. An attestation does not allow the product to bypass provider terms, DRM, access controls, paywalls, or applicable law.

### Artifact

The user-visible library item representing one learning resource and its evolving study output.

An Artifact references:

- One Source.
- Zero or more Generations.
- Zero or more Transcript Versions.
- Zero or more Note Versions.
- The current Note and Transcript references where available.
- User-facing metadata and lifecycle state.

An Artifact is the stable container users organize, open, archive, share, or delete.

### Artifact aggregate

The authoritative Library-service domain object for Artifact ownership, title, lifecycle state, and current resource references.

The Artifact aggregate does not contain the full media, Transcript, Note document, billing ledger, or search index.

### Study workspace

The user interface for studying one Artifact. It composes the player, outline, Note, Transcript, citations, editing controls, status, feedback, export, and sharing actions that the actor is authorized to use.

### Library

The authenticated user’s collection of Artifacts. The Library supports organization, status display, search entry points, archive/trash actions, and continuation across devices.

---

## 3. Users, identity, and access

### User

A person with a product account. In user-facing copy, prefer “user” or the appropriate persona term such as “learner.”

### Learner

The primary product persona using video-derived Notes to understand or review material. A Learner is a type of User, not a separate identity entity.

### Actor

The authenticated identity performing an operation. An Actor may be:

- A User.
- An internal Service.
- An authorized support or administrative identity.
- A system workflow acting under a defined authority.

Authorization decisions should refer to Actor rather than assuming every caller is a User.

### Account

The product-level identity record and preferences associated with a User. Authentication credentials may be managed by the external OIDC provider; the Account service owns the product profile, preferences, consent, and deletion request state.

### Authentication

The process of proving who an Actor is. Authentication does not determine whether that Actor may access a specific resource.

### Authorization

The decision about whether an authenticated Actor may perform an operation on a resource.

Authorization is enforced by the owning Service, even when Edge API has already validated the access token.

### Workload identity

The authenticated runtime identity of an internal Service or worker. Workload identity is separate from the original User/Actor context propagated with a request.

### Owner

The User or future Workspace with authoritative control over an Artifact or other resource. “Owner” may also refer to the team responsible for a repository; the context must make this distinction clear.

### Workspace

A future shared ownership and collaboration boundary. The initial product is individual-user focused. Do not use Workspace as a synonym for Study Workspace.

### Session

The authenticated client interaction state. For web, session secrets live in secure BFF-managed cookies. Desktop/mobile credentials live in approved OS secure storage.

### Entitlement

The Billing-service decision describing which product capabilities and limits an Account may use under its current plan/state.

---

## 4. Media and ingestion

### Ingestion

The capability that validates a Source, creates upload sessions, acquires approved media, records Media Assets, and applies retention deadlines.

Ingestion does not transcribe or generate Notes.

### Acquisition

The rights-compliant act of making Source content available for processing. Being able to display metadata or embed playback does not imply that acquisition is permitted.

### Acquisition option

A specific approved way to obtain processable content, such as a completed local upload or authorized caption track.

### Upload session

A temporary, user-owned record coordinating multipart upload to object storage. It contains expected size/checksum, multipart state, expiry, and ownership metadata.

An Upload Session is not a durable Media Asset until completion and validation succeed.

### Multipart upload

A large-file upload divided into independently transferable parts and finalized as one object. Clients upload parts directly to object storage using narrowly scoped, expiring URLs.

### Presigned URL

A time-limited URL authorizing one narrowly scoped object-storage operation. It is a credential and must not be logged, placed in events, or exposed to unauthorized Actors.

### Quarantine

The isolated storage/state for untrusted uploaded content before security and media validation pass. Quarantined content cannot be served to Users.

### Media Asset

An Ingestion- or Media-service record describing a stored media-related object, such as:

- Uploaded source video.
- Extracted audio.
- Sampled frame.
- Temporary derivative.

A Media Asset includes ownership, storage reference, checksum, size, type, state, and retention deadline.

### Raw media

The original uploaded or acquired audiovisual content before transformation.

### Derived media

Content created from Raw Media for processing, such as normalized audio or selected frames.

### Media preparation

The Media-service process of scanning, probing, validating, extracting audio, normalizing where approved, and optionally sampling frames.

### Media probe

Inspection of a media container and streams to determine format, codecs, duration, audio presence, and other technical properties. The probe does not trust browser MIME type or filename extension.

### Hosted-video URL

An approved HTTPS URL from which media can be acquired without bypassing authentication, DRM, provider restrictions, paywalls, or other access controls.

### YouTube Source

A Source identified by a YouTube URL. Metadata and embedded playback are distinct from transcript/media acquisition. Public visibility alone does not authorize downloading or separating audio.

### Playback descriptor

A safe, client-consumable description of how a Source can be played, such as a YouTube provider ID or an expiring approved media URL. It does not expose internal storage keys.

### Retention deadline

The timestamp after which temporary or policy-limited data must be deleted. Object lifecycle rules are a safety net; the owning Service remains responsible for auditable deletion.

---

## 5. Processing and workflow

### Generation

One immutable attempt to produce a Note from a specific Source/Transcript Version and Generation Settings.

A retry creates a new Generation linked to the earlier attempt. A failed Generation does not return to an active state.

### Generation Settings

The versioned inputs controlling a Generation, including depth, audience, output language, focus, and optional content preferences.

### Depth preset

The requested level of compression/detail:

- `quick`
- `detailed`
- `comprehensive`

Detailed is the default product preset.

### Job

The user-facing asynchronous processing record associated with a Generation. Job exposes current state, progress, safe messages, and ordered events.

Use Job for UI/API progress concepts. Use Workflow for durable orchestration internals.

### Job event

An ordered, user-safe progress record owned by Workflow service. Job Events support SSE replay and progress history. They are not Domain Events.

### Stage

A user-meaningful phase of a Generation, such as acquiring, preparing media, transcribing, understanding, drafting, verifying, or formatting.

A Stage may contain multiple Temporal Activities and Service operations.

### Workflow

A durable Temporal execution coordinating a multi-step business process such as Generation or deletion.

Workflow code is deterministic and performs external work only through Activities.

### Workflow service

The Service that owns Workflow definitions, Generation/job state, ordered progress, cancellation, retry, and compensation coordination.

### Temporal Activity

A named, retryable unit of non-deterministic work invoked by a Temporal Workflow and implemented by the capability-owning Service.

When referring specifically to the Temporal primitive, capitalize Activity. Do not use Activity as a synonym for a complete Workflow or user analytics event.

### Task queue

The Temporal routing boundary from which the owning Service’s workers receive Workflow or Activity tasks.

### Worker

A runtime process that polls a queue or processes background work. A Worker is a deployment/runtime unit, not automatically a separate Service or repository.

### Retry

A repeated attempt of an operation after a classified failure. Retries must be bounded and safe through idempotency or an explicit recovery strategy.

### Compensation

An explicit action that semantically reverses or settles an earlier completed step in a distributed Saga, such as releasing a usage reservation. Compensation is not a cross-service database rollback.

### Saga

A multi-Service business transaction coordinated through ordered local transactions and compensating actions rather than distributed ACID transactions.

### Cancellation

A requested transition that stops remaining work cooperatively and runs required compensation/cleanup. Cancellation is not equivalent to failure.

### Terminal state

A state from which the same attempt cannot resume active processing, such as ready, partially ready, failed, canceled, or deleted.

### Partially ready

A terminal Generation state in which a usable but explicitly degraded artifact is available and policy permits delivery. The product must describe the limitation.

### Idempotency

The property that repeating an operation with the same identity produces the same intended result without duplicate side effects.

### Idempotency key

A caller-supplied or system-generated stable key used by the owning Service to recognize repeated commands.

### Reconciliation

A process that compares a projection or distributed reference with its authoritative owner and repairs missing, delayed, or inconsistent derived state.

---

## 6. Transcript and source understanding

### Transcript

The ordered textual representation of spoken Source content with time alignment and optional speaker/word metadata.

### Transcript Version

An immutable version of a Transcript. Corrections or materially different provider output create a new Transcript Version.

### Transcript segment

The smallest durable ordered transcript unit used by the product. It includes a stable ID, ordinal, start/end time, text, and optional speaker/confidence data.

### Word timing

Optional start/end timing for individual recognized words. Segment timing is the minimum product requirement; word timing may be stored separately in compressed form.

### Speaker label

A provider- or product-assigned identifier distinguishing speakers where it improves understanding. It is not assumed to be a verified real-world identity.

### Semantic chunk

A processing unit composed from ordered Transcript Segments to provide coherent bounded context for evidence extraction. A chunk preserves its exact source segment/time references.

### Evidence Item

A typed, immutable AI-notes-service record representing source-supported material extracted from Transcript Segments, such as:

- Definition.
- Claim.
- Procedure.
- Example.
- Caveat.
- Formula/code reference.
- Ambiguity.

An Evidence Item always refers to exact source segments/time bounds. It is not the same as a Citation.

### Evidence manifest

A versioned collection of Evidence Items and immutable references passed between AI Notes and Notes services for document materialization and validation.

### Source evidence

The underlying Transcript span or approved visual/source reference that supports content in a Note.

### Ambiguity

An unclear, contradictory, or insufficiently supported part of the Source that should be surfaced rather than silently resolved.

---

## 7. Notes, documents, and citations

### Note

The user-facing structured study document associated with an Artifact. A Note is editable and versioned.

Use “Note” for the product artifact users read/edit. Use “document” for its technical structured representation.

### Note Version

An immutable or recoverable version of a Note with document schema version, content hash, origin, parent, Generation reference, Transcript Version reference, and revision metadata.

### Current Note

The Note Version currently selected by the Artifact for normal reading/editing. Older Note Versions remain version-history entries according to retention policy.

### Working version

The editable current document state updated through optimistic concurrency and periodic recovery checkpoints.

### Document

The canonical structured JSON representation of a Note. It is Tiptap-compatible but defined by the platform-owned versioned document schema.

### Document schema

The versioned set of allowed nodes, marks, attributes, nesting rules, and validation/migration behavior for Note documents.

### Block

A stable top-level document unit such as a heading, paragraph, list, table, code block, formula, or callout. Generated Blocks carry stable IDs and provenance metadata.

### Mark

Inline document metadata/formatting applied to text, such as emphasis, link, code, or Citation reference.

### Callout

A structured visual block communicating a semantic role such as definition, example, warning, or takeaway.

### Citation

A Note-to-evidence relationship that resolves to a specific Transcript Version, Evidence Item, Transcript Segment set, and start/end time.

A Citation is created from validated Evidence Items. A model must not invent Citation timestamps directly.

### Citation accuracy

The degree to which a Citation’s source window contains evidence supporting the associated Note content.

### Citation coverage

The proportion of material Note claims/sections that have required supporting Citations under the selected policy/preset.

### Timestamp link

A client interaction that seeks supported playback to a Citation’s source time and highlights associated evidence where available.

### Provenance

Metadata indicating the origin of Note content:

- `source_grounded_ai`
- `ai_explanation`
- `user`
- `mixed`

Provenance is not proof of correctness and is not a security boundary.

### Grounded

Supported by identified Source Evidence or a separately labeled/cited external source.

For MVP, external enrichment is disabled; “grounded” normally means supported by the supplied Source.

### AI explanation

An explanation added by the system to aid understanding but not directly stated in the Source. It must be distinguishable from Source-grounded content.

### Stale citation

A Citation whose associated Note text was materially changed after validation. The product must not silently imply that edited text remains Source-supported.

### Autosave

The automatic persistence of the Working Version after a short debounce and at lifecycle boundaries where possible.

### Revision

A monotonic concurrency value representing the accepted server state of an editable Note.

### Edit conflict

A condition in which a save is based on an older Revision than the current server state. User work must be preserved and reconciled rather than silently overwritten.

### Checkpoint

A recoverable snapshot of a Working Version created for safety, version recovery, or a significant operation.

### Regeneration

Creating a new Note Version through another Generation. Regeneration must not destructively overwrite user edits.

---

## 8. AI generation and evaluation

### AI Notes service

The Service owning semantic chunking, Evidence Items, outline planning, drafting, grounding verification, quality reports, prompt/configuration references, and AI-generation intermediates.

### Model provider

An external or future internal system that performs structured text or vision model inference through an application-owned adapter.

### ASR provider

An automatic speech-recognition provider that converts approved audio into timed transcript output.

### Provider adapter

An implementation of a product-owned interface translating provider-specific requests/responses into canonical Service types.

### Model routing

The versioned policy selecting a provider/model/configuration based on task, language, preset, evaluation, quota, availability, cost, and experiment assignment.

### Prompt

A versioned instruction/template supplied to a model for one bounded task. Prompts are production configuration and require evaluation/release controls.

### Structured output

Model output constrained to a typed schema and validated before use. Validation does not prove factual correctness or grounding.

### Agentic workflow

A bounded application-controlled workflow in which models plan or transform content within predefined steps, schemas, budgets, and tools.

It is not an open-ended autonomous agent. Source content cannot create new tools, permissions, or workflow stages.

### Prompt injection

Untrusted instructions embedded in a Source, Transcript, frame, document, or provider response intended to change system behavior or obtain unauthorized actions/data.

Source content is always data, never system instruction.

### Grounding check

The deterministic and model-assisted verification that material Note content is supported by its referenced Evidence Items.

### Coverage check

The verification that important Source topics/evidence are represented appropriately in the Note.

### Quality Report

A versioned AI-notes-service record containing rubric scores, defects, pass/degraded state, and repair instructions for one Generation.

### Repair pass

A bounded additional model step attempting to correct defects identified by verification. MVP permits at most one repair pass unless an approved configuration says otherwise.

### Evaluation

The repeatable measurement of ASR/model/prompt/system behavior against a permissioned dataset and rubric.

### Benchmark

A versioned evaluation dataset and expected evidence/rubric used to compare configurations and prevent regressions.

### Evaluation slice

A meaningful subset of the Benchmark, such as language, content type, video length, accent, audio quality, or subject category.

### Regression

A meaningful reduction in quality, correctness, grounding, reliability, latency, or cost performance relative to the approved baseline.

### Canary

A limited production exposure used to validate a new version/configuration before wider rollout.

### Shadow evaluation

Running a new configuration without using its output for the user, so behavior can be compared safely with the current path.

### Kill switch

An operational control that disables a provider, model, prompt configuration, language, Source adapter, preset, or feature without a full deployment.

### Generation budget

The hard limit on model calls, tokens, provider spend, retries, repair attempts, and/or wall time for one Generation.

### Chain of thought

Private model reasoning that the product does not request or store. Store structured decisions, evidence references, defects, and outcomes instead.

---

## 9. Backend architecture

### Service

An independently deployable business-capability component with one repository, owner, runtime identity, data ownership boundary, contracts, release pipeline, and runbook.

A Service is not synonymous with repository, worker, container, table, or endpoint.

### Microservice

A Service whose capability, deployment, persistence, and failure boundary are independently owned. Do not create a Microservice merely to wrap one table or technical function.

### Edge API

The only public backend entry point for desktop/mobile clients and the web BFF. It authenticates, rate-limits, propagates identity, routes or composes bounded requests, exposes public OpenAPI/SSE, and owns no business data.

### Backend for Frontend (BFF)

The web-only server boundary inside the Next.js application. It owns the secure browser session and calls Edge API with short-lived access credentials.

The BFF is not a general business Service and does not own product data.

### Capability Service

A Service owning a specific product/business capability, such as Ingestion, Transcription, Notes, or Billing.

### Owning Service

The only Service authorized to define and mutate a specific aggregate/dataset. Other Services interact through versioned contracts.

### Database per service

The rule that each Service uses a separate logical database, role, migrations, and access credentials. Databases may initially share a managed cluster, but cross-database queries, shared tables, and cross-Service foreign keys are prohibited.

### Repository

A separately versioned source-control codebase with its own CI and release history. Not every deployment unit needs a separate Repository; worker pools remain in the owning Service Repository.

### Deployment unit

An independently scalable runtime artifact/process, such as a Service API or worker pool. Multiple Deployment Units may come from one Service Repository.

### Service catalog

The governance record of every Service’s mission, non-goals, owner, data, dependencies, contracts, repository, SLOs, and lifecycle state.

### Service Connect

The selected ECS-managed private service discovery/connectivity mechanism. Use the full name “ECS Service Connect” when provider specificity matters.

### Data owner

The Service authoritative for a datum and its lifecycle. Data ownership includes creation, validation, update, retention, export, deletion, and recovery behavior.

### Opaque ID

An identifier referenced across Service boundaries without exposing or depending on the owner’s internal database structure.

### Snapshot

A versioned copy of the minimum data required by a consumer for its local behavior. A Snapshot does not transfer authority away from the owner.

---

## 10. APIs, commands, events, and projections

### Contract

A versioned machine-readable agreement between producers and consumers, including APIs, Domain Events, Temporal Activities, document schemas, or shared-package interfaces.

### Public API

The Edge API contract available to web, desktop, and mobile clients.

### Internal API

A private Service contract callable only by authorized workloads. Internal does not mean unversioned or unauthenticated.

### Command

A request asking an owner to perform a state-changing operation, such as `StartGeneration` or `CancelGeneration`. A Command can be rejected.

### Query

A request for information that does not intentionally change domain state.

### Domain Event

An immutable, versioned statement that a completed domain fact occurred, such as `TranscriptCreated`.

A Domain Event is not a Command, Job Event, application log, analytics event, or Temporal Activity.

### Event envelope

The standard metadata surrounding every Domain Event, including event ID, type/version, producer, aggregate ID/version, occurrence time, trace ID, correlation ID, and causation ID.

### Producer

The Service that owns and publishes a Domain Event or Contract response.

### Consumer

A Service/application that processes a published Contract or Domain Event.

### Transactional outbox

A Service-local pattern that commits domain changes and an Outbox record in the same database transaction, then publishes the event asynchronously.

### Outbox publisher

The owning Service process that delivers committed Outbox records to the Event Bus and tracks delivery attempts.

### Event Bus

The asynchronous routing infrastructure for Domain Events. The selected implementation is Amazon EventBridge.

### Consumer queue

The consumer-owned SQS queue receiving matching Domain Events from the Event Bus.

### Dead-letter queue (DLQ)

The isolated queue receiving messages that repeatedly failed processing. DLQ redrive must be observable and idempotent.

### At-least-once delivery

A delivery model in which a message may be delivered more than once. Consumers must tolerate duplicates.

### Aggregate version

A monotonic version of an owning aggregate carried in Domain Events so consumers can detect stale updates or gaps.

### Projection

A consumer-owned derived representation built from owner APIs, Snapshots, or Domain Events for a specific read/use case.

A Projection is not authoritative for the source domain data.

### Read model

A query-optimized Projection, such as library workspace metadata or search documents.

### Eventual consistency

A consistency model in which a Projection may temporarily lag the owning Service but converges through event processing and reconciliation.

### Contract compatibility

The ability of supported producer and consumer versions to communicate safely during independent releases.

### Additive change

A backward-compatible Contract change that adds optional capability without removing or changing supported meaning.

### Breaking change

A change that a supported consumer cannot safely process without modification. Breaking changes require a new major/versioned Contract and a migration window.

### Consumer-driven contract test

A test expressing a Consumer’s required Provider behavior and validating the Provider before release.

---

## 11. Frontend architecture

### Client application

A user-facing application consuming the public Edge API: Web, Desktop, or Mobile.

### Web application

The Next.js/React application delivered through a browser. It includes public pages, authenticated application routes, and the web BFF.

### Desktop application

The Electron application. Its renderer uses React DOM/web-standard UI; its main/preload/utility processes provide restricted native capabilities.

### Mobile application

The React Native/Expo iOS and Android application using native UI primitives.

### Shared frontend package

An independently versioned private npm package containing reusable contracts, domain logic, document logic, controllers, tokens, or UI.

Applications consume published package versions rather than source from another application repository.

### Frontend foundation

The platform-neutral shared packages for domain types, API transport, document model, state machines, React controllers, telemetry, and design tokens.

### React core

The shared headless React layer containing Query hooks, controllers, view models, and dependency-injection contexts without DOM, React Native, Electron, or Next.js dependencies.

### Web UI

The React DOM components shared by Web and Electron renderer, including the web editor and player packages.

### Native UI

The React Native components used by iOS/Android. Native UI shares tokens and behavior but does not import DOM components.

### Design token

A semantic platform-neutral value for color, spacing, typography, radius, elevation, motion, or breakpoint behavior.

### Platform adapter

An application-supplied implementation of a shared capability interface, such as authentication, navigation, files, uploads, storage, notifications, sharing, lifecycle, telemetry, or playback.

### Composition root

The application-specific location where concrete Platform Adapters and configuration are assembled and supplied to shared feature code.

### Server Component

A React/Next.js component executed in the web server environment. Server Components are web-only and must not enter packages intended for Electron or Mobile.

### Client Component

An interactive React component executed in the browser/renderer. A Next.js Client Component may be shareable with Electron only if it does not import Next.js-specific or browser-only assumptions that lack an adapter.

### React DOM

The React renderer for web-standard DOM environments. Web and Electron renderer use React DOM; React Native does not.

### Canonical document renderer

A platform-specific renderer that displays the same versioned Note Document schema. Web/Electron use Tiptap/React DOM; Mobile uses a native renderer.

---

## 12. Search, export, sharing, and notification

### Search document

A Search-service Projection containing authorized, indexable Artifact/Note/Transcript content and owner/version metadata.

### Search index lag

The elapsed time between an authoritative owner update and its availability in Search.

### Export

A version-pinned conversion of a Note into a supported external format such as Markdown or PDF.

### Export request

The durable Export-service record tracking format, Note Version, rendering state, result reference, expiry, and failure.

### Share link

A revocable, scope-limited, high-entropy public reference to an approved read-only Note view. It does not grant private source download or broad Account access.

### Notification

A user-facing message about an event such as Generation ready/failed. Notifications may be in-app, email, desktop, or push.

### Delivery attempt

One Notification-service attempt to deliver a Notification through a channel/provider.

---

## 13. Billing and usage

### Plan

The commercial packaging that determines Entitlements, limits, priority, and pricing.

### Allowance

The remaining usage capacity granted by an Entitlement for a defined period.

### Usage estimate

The predicted units/cost shown before a Generation based on duration, preset, and modalities. It is not final settlement.

### Usage reservation

An idempotent Billing-service hold created before costly work to prevent overspending or concurrent overuse.

### Usage settlement

The idempotent finalization that commits actual billable usage or releases the reservation according to outcome/refund policy.

### Usage ledger

The append-only Billing-service record of reservations, settlements, releases, renewals, and corrections. Corrections use compensating entries rather than mutation of history.

### Product-caused failure

A failure attributable to the platform/providers under the published policy rather than invalid user input or explicit cancellation. Product-caused failures do not consume usage according to the proposed policy.

### Unit economics

Revenue and variable cost behavior measured per relevant outcome, such as processed Source hour or meaningfully studied Artifact.

---

## 14. Privacy, security, and rights

### Personal data

Information relating to an identified or identifiable person, subject to applicable privacy policy/law.

### Private content

User-supplied or derived content not intended for public disclosure, including media, Transcripts, Notes, prompts containing Source content, and feedback excerpts.

### Data classification

The approved category determining how data may be stored, accessed, logged, retained, exported, and deleted.

### Data minimization

Collecting, retaining, and exposing only the data necessary for an approved purpose.

### Data retention

The policy controlling how long a data category remains stored.

### Soft deletion

Making a resource unavailable to normal product reads while retaining it temporarily for an approved recovery period.

### Purge

The irreversible Service-owned deletion of active data after the recovery period or immediate-deletion policy.

### Deletion Saga

The Workflow coordinating purge requests and acknowledgements across all owning Services.

### Tombstone

A content-free durable record indicating that an entity was deleted and must not be recreated by delayed events, restore, or reconciliation.

### Recovery window

The documented period during which a soft-deleted resource may be restored before Purge.

### Least privilege

Granting an Actor or workload only the minimum permissions and duration needed for its responsibility.

### Tenant isolation

Controls preventing one User/Workspace from accessing another’s content or operations.

### Secret

Sensitive credential material such as API keys, signing keys, tokens, passwords, or certificates. Secrets never belong in source control, logs, or general events.

### Content Security Policy (CSP)

A browser/Electron policy restricting which content and scripts may execute or load.

### Server-side request forgery (SSRF)

An attack in which a supplied URL causes the system to access unauthorized internal or restricted network resources.

### Digital rights management (DRM)

Technical access controls protecting content. The product must not bypass DRM.

### Rights-holder request

A complaint, takedown, or inquiry from a party claiming rights in Source or shared derived content.

### Support elevation

A time-limited, audited authorization allowing approved support personnel to access otherwise restricted content for a specific investigation.

---

## 15. Reliability and operations

### Service-level indicator (SLI)

A measured signal representing service behavior, such as availability, latency, success rate, or projection lag.

### Service-level objective (SLO)

The target range for an SLI over a defined period.

### Error budget

The permitted amount of SLO non-compliance used to guide release and reliability decisions.

### Availability

The proportion of valid operations for which the service is able to provide its intended response within policy.

### Latency

The elapsed time for an operation, stage, or user outcome. Always qualify the measured boundary and percentile.

### Queue latency

The elapsed time between task/message availability and processing start.

### Projection lag

The time/distance between an owner’s aggregate version and the version applied by a Projection.

### Timeout

The maximum permitted duration for an operation or phase before it is treated as incomplete/failing.

### Circuit breaker

A control that temporarily stops calls to a failing dependency to prevent cascading failure and enable recovery.

### Backpressure

Controls limiting incoming work when downstream capacity or provider quotas are saturated.

### Dead-letter redrive

The controlled reprocessing of messages from a DLQ after the underlying problem is resolved.

### Runbook

An operational procedure for diagnosing and responding to a known alert, failure, deployment, rollback, or recovery task.

### Playbook

A broader policy/process describing how a class of work is performed. `BUILD_PLAYBOOK.md` governs AI-assisted product delivery; a Runbook handles a specific operational situation.

### Incident

An unplanned event that degrades user outcomes, security, privacy, data integrity, or operations and requires coordinated response.

### Recovery point objective (RPO)

The maximum acceptable data-loss interval after a disaster.

### Recovery time objective (RTO)

The target time to restore an acceptable service level after a disaster.

### Release manifest

The immutable record of exact application, Service, Contract, package, Workflow, prompt/model, migration, feature-flag, and infrastructure versions in a Release.

### Rollback

Returning runtime/configuration to a previously safe compatible version.

### Forward repair

Deploying a corrective change when rollback is unsafe or impossible, commonly after a destructive/incompatible data migration.

---

## 16. Product analytics and success

### Activation

The defined point at which a new User receives and meaningfully engages with their first successful Note.

### Meaningful study action

A behavior indicating use beyond opening an Artifact, such as substantial reading, following a Citation, editing, searching, exporting, or returning later.

### Meaningfully studied Artifact

An Artifact that crosses the approved reading/engagement threshold and includes at least one Meaningful Study Action.

### Weekly successfully studied Artifacts

The proposed north-star metric: unique Artifacts meaningfully studied during a week after successful Generation.

### Guardrail metric

A metric that detects harm or unacceptable tradeoffs while optimizing a primary metric, such as incorrect-citation reports, cost, failure rate, or deletion delays.

### Product event

A privacy-reviewed analytics record describing user/product behavior. It is not a Domain Event and must not contain raw private content.

---

## 17. Delivery and governance

### Product Requirement Document (PRD)

The authoritative description of the product problem, users, scope, requirements, metrics, and rollout intent.

### Technical Requirements Document (TRD)

The authoritative cross-system technical baseline defining architecture, technologies, contracts, data boundaries, security, operations, and acceptance criteria.

### Build Playbook

The operating policy for human-led, AI-implemented delivery, including gates, roles, templates, reviews, and releases.

### Roadmap

The gated sequence of outcomes, milestones, repository activation, decisions, risks, and exit criteria.

### Architecture Decision Record (ADR)

A durable record of a material technical decision, considered options, consequences, validation, and revisit trigger.

### Service specification

The `SERVICE.md` document defining one Service’s mission, non-goals, owned data, contracts, failures, security, tests, observability, and operations.

### Feature specification

The versioned cross-system description of one user outcome, participating Services, sequence, contracts, failures, acceptance, rollout, and tasks.

### Vertical slice

The smallest end-to-end behavior that validates or delivers part of a user outcome across the necessary layers.

### Walking skeleton

A minimal end-to-end implementation using real architecture and contracts, with deterministic mocks for expensive/uncertain capabilities.

### Golden path

The approved, automated way to create, test, secure, deploy, observe, and operate a common repository/service/package type.

### Definition of Ready

The gate confirming a task is sufficiently understood, specified, and unblocked for implementation.

### Definition of Done

The gate confirming implementation, verification, documentation, operations, review, release, and observation are complete as applicable.

### Milestone

A gated set of vertical outcomes with explicit repositories, deliverables, decisions, non-goals, and exit evidence.

### Gate

An explicit decision point that prevents progression until required evidence or approval exists.

### Exit criterion

Observable evidence required to complete a Milestone or phase.

### Repository activation

Creating a repository because an approved Vertical Slice needs its capability and its ownership, first task, CI, security, and release path are defined.

### AI implementation agent

An AI acting within an approved task to analyze, implement, test, document, and prepare evidence.

### AI review agent

An independent AI pass that reviews an implementation against scope, architecture, security, data, tests, and operations.

### Owner explanation

The implementation handoff explaining why the behavior exists, who owns it, its data flow, failure/recovery, tests, security, deployment, and rollback.

---

## 18. Canonical states

### Artifact states

| State | Meaning |
|---|---|
| `draft` | Artifact exists but processing has not begun |
| `uploading` | A local Source upload is active/incomplete |
| `processing` | At least one current Generation is active |
| `ready` | A current usable Note is available |
| `partially_ready` | A current explicitly degraded Note is available |
| `failed` | Latest intended processing attempt failed and no usable current result is selected |
| `archived` | Hidden from the default active Library but retained |
| `deleting` | Deletion/recovery-window workflow is active |
| `deleted` | Unavailable to the User; purge/tombstone policy applies |

### Generation/job states

| State | Meaning |
|---|---|
| `accepted` | Generation request is durably recorded |
| `acquiring` | Approved Source content is being resolved/acquired |
| `preparing_media` | Media is being validated/transformed |
| `transcribing` | Transcript is being obtained/created |
| `understanding` | Transcript is being chunked and evidence extracted |
| `drafting` | Note sections are being planned/written |
| `verifying` | Grounding, coverage and quality checks are running |
| `formatting` | Canonical Note is being assembled/materialized |
| `ready` | Verified Note is durably available |
| `partially_ready` | Degraded but policy-approved Note is available |
| `failed` | Generation ended unsuccessfully |
| `canceled` | Generation ended due to an accepted cancellation request |

### Roadmap/work states

| Symbol/state | Meaning |
|---|---|
| `○ Not started` | No active implementation work |
| `◐ In progress` | Work started but exit criteria are incomplete |
| `◉ Complete` | Exit criteria and ownership review passed |
| `△ Blocked` | Progress requires an external decision/change |
| `◇ Deferred` | Intentionally postponed |

---

## 19. Naming and usage rules

### Use these distinctions

| Use | Do not use as a synonym |
|---|---|
| Artifact | Note, Source, file, project |
| Note | Artifact, Transcript, summary |
| Source | Media Asset, Artifact, URL only |
| Generation | Job, Workflow, model request |
| Job | Workflow, Activity, analytics event |
| Workflow | Job, Service, pipeline stage |
| Temporal Activity | user activity, Domain Event, Stage |
| Evidence Item | Citation, Transcript Segment |
| Citation | timestamp alone, Evidence Item |
| Domain Event | Command, Job Event, Product Event, log |
| Service | Repository, Worker, container, endpoint |
| Repository | Service, application, package |
| Workspace | Study Workspace |

### Capitalization

- Capitalize defined domain entities when referring to the canonical concept: Artifact, Source, Generation, Note, Transcript Version, Evidence Item, Citation, Service, Workflow, Activity, Domain Event.
- Lowercase generic uses where no canonical entity is intended.
- In user-facing copy, prefer natural sentence case unless capitalization aids clarity.

### Identifiers

- Use UUIDv7-compatible opaque identifiers where specified by the TRD.
- Suffix cross-service identifiers clearly, such as `artifact_id`, `generation_id`, `transcript_version_id`, and `note_version_id`.
- Never overload one identifier type for another entity.

### Time

- Persist timestamps in UTC and exchange them in RFC 3339 format.
- Use milliseconds for media positions: `start_ms`, `end_ms`, `position_ms`.
- Store/display user timezone separately from UTC event time.

### Versions

- Distinguish Contract Version, Aggregate Version, Document Schema Version, Note Revision, Note Version, Transcript Version, prompt version, and release version.
- Never use a generic field named only `version` when multiple version concepts exist in the same boundary.

### Errors

- Use stable error-code families such as `SOURCE_*`, `UPLOAD_*`, `MEDIA_*`, `TRANSCRIPT_*`, `GENERATION_*`, `NOTE_*`, `EXPORT_*`, `BILLING_*`, `AUTH_*`, and `RATE_LIMITED`.
- User messages are safe and actionable. Internal causes belong in protected telemetry, not API messages.

---

## 20. Discouraged or prohibited terminology

### “AI magic” / “just AI”

Avoid. Name the actual bounded capability, model task, evidence, verification, and failure behavior.

### “The backend”

Avoid in system specifications when the owning Service is known. Name Edge API, Workflow, Notes, Ingestion, or the actual participant.

### “The database”

Avoid. Name the owning Service database.

### “Pipeline”

Use only for a clearly defined processing/deployment sequence. Prefer Workflow, Stage, or CI/CD pipeline when that is the precise meaning.

### “Summary”

Do not use as a synonym for Note. Summary may describe a specific brief section or Quick preset output; a Note is the complete structured study document.

### “Hallucination”

Prefer a precise defect type: unsupported claim, incorrect attribution, fabricated citation, contradiction, source distortion, or ungrounded explanation.

### “Real time”

Use only with a measured latency requirement. Prefer “live progress,” “near-real-time update,” or the actual SLO.

### “Exactly once”

Avoid for distributed delivery. State the actual mechanism: at-least-once delivery with idempotent processing and unique business-event keys.

### “Secure” / “private”

Do not use as unqualified guarantees. Name the controls, scope, threat, and remaining assumptions.

### “Done”

Use only when the applicable Definition of Done and observation gate pass.

---

## Appendix A: Abbreviations

| Abbreviation | Meaning |
|---|---|
| ADR | Architecture Decision Record |
| AI | Artificial intelligence |
| API | Application Programming Interface |
| ASR | Automatic Speech Recognition |
| BFF | Backend for Frontend |
| CI/CD | Continuous Integration / Continuous Delivery or Deployment |
| CSP | Content Security Policy |
| DLQ | Dead-letter queue |
| DRM | Digital rights management |
| E2E | End to end |
| IAM | Identity and Access Management |
| IPC | Inter-process communication |
| JWT | JSON Web Token |
| KMS | Key Management Service |
| OIDC | OpenID Connect |
| PII | Personally identifiable information; prefer the broader “personal data” in policy documents |
| PKCE | Proof Key for Code Exchange |
| PRD | Product Requirements Document |
| RLS | Row-level security |
| RPO | Recovery Point Objective |
| RTO | Recovery Time Objective |
| SLI | Service-Level Indicator |
| SLO | Service-Level Objective |
| SSE | Server-Sent Events |
| SSRF | Server-Side Request Forgery |
| TRD | Technical Requirements Document |
| UI | User interface |
| URL | Uniform Resource Locator |
| WCAG | Web Content Accessibility Guidelines |

## Appendix B: Term proposal template

```markdown
## Proposed term

**Definition:**
What the term means.

**Why needed:**
Which ambiguity or new concept it resolves.

**Not equivalent to:**
Related terms that must remain distinct.

**Affected documents/contracts/code:**
Where existing usage must change.

**Owner:**
Who approves and maintains the definition.
```

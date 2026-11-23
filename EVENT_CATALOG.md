# Study Notes Creator: Domain Event Catalog

**Document status:** Active asynchronous-contract standard  
**Version:** 1.0  
**Last updated:** 23 November 2026  
**Current delivery milestone:** M0 — Product control plane  
**Transport baseline:** Amazon EventBridge with a dedicated SQS queue and DLQ per consumer  
**Related documents:** [PRD.md](./PRD.md), [TRD.md](./TRD.md), [GLOSSARY.md](./GLOSSARY.md), [SERVICE_CATALOG.md](./SERVICE_CATALOG.md), [DATA_OWNERSHIP.md](./DATA_OWNERSHIP.md), [ROADMAP.md](./ROADMAP.md), [BUILD_PLAYBOOK.md](./BUILD_PLAYBOOK.md), [repos.md](./repos.md)

---

## 1. Purpose

This document is the authoritative catalog for cross-Service Domain Events in Study Notes Creator. It defines:

- The canonical event envelope and EventBridge mapping.
- Event naming, schema versioning, compatibility, and lifecycle rules.
- Producer, consumer, routing, ordering, idempotency, gap, replay, and DLQ responsibilities.
- Privacy/classification rules for asynchronous payloads.
- The initial planned event inventory and minimum payloads.
- Candidate event names that are deliberately deferred because no justified consumer exists yet.

This document defines asynchronous **domain facts**, not exact generated JSON Schema files. Before an event is published, its machine-readable schema must be reviewed and released from `study-notes-creator-contracts`.

### 1.1 Authority

| Question | Authority |
|---|---|
| Meaning of Domain Event and related terminology | `GLOSSARY.md` |
| Which Service owns a capability | `SERVICE_CATALOG.md` |
| Which Service owns the underlying data and classification | `DATA_OWNERSHIP.md` |
| Envelope, event ownership, fact meaning, producer and consumers | This document |
| Executable event schema and generated types | `study-notes-creator-contracts` |
| Producer/consumer implementation and runbook | Each repository's approved `SERVICE.md` |
| Transport infrastructure and IAM | Infrastructure repository and ADR-006 |

If a Service catalog lists a possible event but this catalog marks it deferred, it must not be published until this catalog is updated.

---

## 2. What is—and is not—a Domain Event

A Domain Event is an immutable, versioned statement that a completed fact owned by one Service occurred.

| Concept | Meaning | Example | Delivery mechanism |
|---|---|---|---|
| Domain Event | Completed domain fact used asynchronously by another Service | `NoteVersionCreated` | Transactional outbox → EventBridge → consumer SQS |
| Command | Request that may be accepted or rejected | `StartGeneration` | Versioned HTTP API or Temporal Activity |
| Query | Request for current information | Get Artifact | Versioned HTTP API |
| Temporal Activity result | Durable Workflow step result | `materialize_note_version` result | Temporal task queue/history |
| Job Event | Ordered, user-safe progress item | “Transcribing, 45%” | Workflow database → SSE/polling |
| Product Event | Privacy-reviewed behavior measurement | Citation clicked | Approved analytics pipeline |
| Operational event | Log/metric/trace/alert | Queue age exceeded | Telemetry platform |

Rules:

- Events use past-tense facts. `GenerateNote` is a Command; `NoteVersionCreated` is a fact.
- A Temporal Activity result is not republished as a Domain Event unless a separate asynchronous consumer needs the completed fact.
- Job Events remain Workflow-owned, ordered progress data. Domain Events are not exposed directly to clients or used for SSE replay.
- A Domain Event never instructs a consumer how to implement work. Each consumer decides its own local reaction.
- No event is added “for completeness.” It needs a named producer, at least one approved consumer or an approved near-term route, and a recovery story.

---

## 3. Ownership and responsibilities

### 3.1 Producer

The Service owning the fact owns:

- The event name, meaning, trigger, aggregate version, payload classification, and source schema.
- Generating the event ID inside the same local transaction as the domain change.
- Committing a serialized Outbox record with the domain change.
- Validating the payload against the published schema before it becomes publishable.
- Publishing/retrying, measuring Outbox age, and preserving the original event during redelivery.
- Backward compatibility and deprecation communication.

The producer does not own consumer queues, consumer projections, or consumer retry code.

### 3.2 Consumer

Each consumer owns:

- Its EventBridge rule/subscription request, SQS queue, DLQ, IAM, handler, processed-event record, and runbook.
- Schema compatibility testing before accepting a new producer version.
- Idempotency, authorization scope, stale-version handling, gap reconciliation, and projection rebuild.
- Acknowledging the SQS message only after its local transaction commits.
- Alerting, inspecting, repairing, and redriving its DLQ.

One consumer's failure or backlog must not block delivery to another consumer.

### 3.3 Contracts repository

`study-notes-creator-contracts` owns publication mechanics, not domain meaning. It contains:

- JSON Schema for each envelope/event version.
- AsyncAPI channels and EventBridge mapping metadata.
- Generated Python/Pydantic and TypeScript types where useful.
- Compatibility checks and immutable released artifacts.

Generated types do not contain cross-Service business rules.

### 3.4 Infrastructure

`study-notes-creator-infrastructure` provisions the custom EventBridge bus, rules, queues, DLQs, encryption, IAM, alarms, and environment composition. Infrastructure cannot change payload meaning or route a new consumer without producer/consumer contract review.

---

## 4. Canonical event envelope

### 4.1 JSON shape

Every Domain Event uses this application envelope inside the EventBridge `detail` field:

```json
{
  "envelope_version": 1,
  "event_id": "019aa14b-77c0-7b52-9c0a-7da4e67af221",
  "event_type": "NoteVersionCreated",
  "event_version": 1,
  "producer": "study-notes-creator.notes-service",
  "occurred_at": "2026-11-23T10:15:30.123Z",
  "aggregate_type": "Note",
  "aggregate_id": "019aa149-d48f-79a1-8c14-f171d6a7684f",
  "aggregate_version": 8,
  "correlation_id": "019aa147-1396-77bc-a4c9-5b3b92b9b96a",
  "causation_id": "019aa149-47e6-77c4-8a9f-03d4c1c1e593",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "schema_ref": "urn:study-notes-creator:events:NoteVersionCreated:1.0.0",
  "classification": "C2",
  "payload": {
    "account_id": "019aa130-b5ac-7f54-99e1-04d7c65f14d1",
    "artifact_id": "019aa13a-e639-7459-bda7-0935515324fe",
    "note_id": "019aa149-d48f-79a1-8c14-f171d6a7684f",
    "note_version_id": "019aa14b-753f-7fb2-9279-38046c4d276f",
    "document_schema_version": 1,
    "origin": "source_grounded_ai"
  }
}
```

The example contains no Note body, Transcript text, Evidence text, object key, or signed URL.

### 4.2 Envelope fields

| Field | Required | Rule |
|---|:---:|---|
| `envelope_version` | Yes | Integer version of the common envelope; initially `1` |
| `event_id` | Yes | Globally unique UUIDv7-compatible ID generated once; unchanged on retry/replay/redrive |
| `event_type` | Yes | PascalCase past-tense fact from this catalog |
| `event_version` | Yes | Positive integer major schema/semantic version; initially `1` |
| `producer` | Yes | Canonical producer source: `study-notes-creator.<service>` |
| `occurred_at` | Yes | UTC RFC 3339 timestamp when the domain fact committed, not publish/retry time |
| `aggregate_type` | Yes | Canonical owning aggregate type, such as `Artifact` or `Generation` |
| `aggregate_id` | Yes | Opaque owning aggregate ID |
| `aggregate_version` | Yes | Positive monotonic version assigned by the owner transaction |
| `correlation_id` | Yes | Stable ID connecting one user/business flow across calls, Activities, and events |
| `causation_id` | Yes | ID of the Command, Activity completion, or prior event that directly caused this fact |
| `trace_id` | Yes | W3C-compatible 32-lowercase-hex trace ID; creates a trace link, not an authorization claim |
| `schema_ref` | Yes | Immutable exact schema identity including SemVer within the event major |
| `classification` | Yes | Highest payload class; only `C1` or `C2` permitted on the bus |
| `payload` | Yes | Event-specific object containing only approved minimum fields |

No field may contain a database primary-key encoding, internal object key, credential, or provider object when an opaque product ID is sufficient.

### 4.3 ID semantics

- Redelivery, retry, DLQ movement, and replay retain the original `event_id`.
- A new domain transition receives a new `event_id` even if it has the same correlation ID.
- Several facts committed by one aggregate transaction may share an `aggregate_version`. Aggregate version identifies source state, not a global event sequence.
- `correlation_id` normally begins at the public Command/Workflow request and survives the entire flow.
- Root facts use the accepted Command/request ID as `causation_id`. A fact caused by another fact uses that prior `event_id`.
- Consumers deduplicate by (`consumer_name`, `producer`, `event_id`). They do not use trace ID for deduplication.

### 4.4 CloudEvents compatibility

The application envelope maps cleanly to CloudEvents 1.0:

| CloudEvents attribute | Catalog field/value |
|---|---|
| `specversion` | `1.0` |
| `id` | `event_id` |
| `source` | `urn:study-notes-creator:service:<producer>` |
| `type` | `com.studynotescreator.<event-type-kebab>.v<event_version>` |
| `subject` | `/<aggregate_type>/<aggregate_id>` |
| `time` | `occurred_at` |
| `datacontenttype` | `application/json` |
| `dataschema` | `schema_ref` |
| `data` | Full application envelope or payload under the selected binding |

The contracts repository must publish one tested mapping; Services must not invent different bindings.

---

## 5. Naming, schema, and compatibility

### 5.1 Naming

- Event types use PascalCase noun + past-tense verb: `ArtifactCreated`, `GenerationFailed`.
- Names describe the producer's fact, not a consumer action. Avoid `UpdateSearchIndex` or `SendNotification`.
- Do not put versions in `event_type`; use `event_version`. Transport detail types may add `.v1`.
- Use canonical Glossary nouns. Do not use “file,” “summary,” “pipeline,” or “job” when Source, Note, Workflow, or Generation is meant.

### 5.2 Schema location

Planned source layout in `study-notes-creator-contracts`:

```text
events/
├── envelope/
│   └── v1.schema.json
├── account/
│   ├── account-deletion-requested/1.0.0.schema.json
│   └── account-purged/1.0.0.schema.json
├── library/
│   └── ...
└── asyncapi.yaml
```

The immutable release artifact includes schema content hashes. Producer CI fails if emitted examples do not validate; consumer CI tests every subscribed version.

### 5.3 Versioning rules

The integer `event_version` is the major compatibility version. `schema_ref` identifies the exact SemVer schema, such as `1.0.0` or `1.1.0`, within that major.

Compatible within the same major version:

- Adding an optional field with a safe default/absence meaning.
- Widening an enum only when all supported consumers ignore unknown values or explicitly handle an `unknown` path.
- Clarifying documentation without changing behavior.

A compatible schema change publishes a new immutable minor/patch `schema_ref` while `event_version` remains unchanged.

Breaking and requiring a new major version:

- Removing or renaming a field.
- Changing type, units, identifier meaning, classification, or aggregate semantics.
- Making an optional field required.
- Narrowing allowed values or changing trigger timing.
- Reinterpreting an existing event as a Command.

Consumers ignore unknown optional fields but reject an unsupported major version. A producer must not publish a new major until every routed consumer has declared support.

### 5.4 Breaking-change rollout

1. Publish the new schema and generated types.
2. Upgrade consumers to understand the new version.
3. Create version-specific EventBridge routing so each consumer receives exactly one intended major.
4. If dual publication is unavoidable, publish distinct event IDs with a shared correlation/causation pair and ensure no queue receives both versions.
5. Switch producers, observe compatibility/error metrics, then retire old routes after the support window.
6. Mark the old version deprecated here before removal.

Do not rely on synchronized multi-repository deployment.

---

## 6. Publication and transport

### 6.1 Transactional Outbox

The producing transaction:

1. Validates the Command and Actor.
2. Mutates the Service-owned aggregate and increments `aggregate_version`.
3. Constructs and schema-validates the complete immutable event.
4. Inserts the serialized event into the local Outbox with a unique `event_id`.
5. Commits aggregate and Outbox atomically.

The Outbox publisher:

1. Claims unpublished rows safely.
2. Calls EventBridge `PutEvents` in bounded batches.
3. Checks every entry result because a batch may partially fail.
4. Marks successful publication locally.
5. Retries transient failures with jitter/backoff without changing the event.
6. Alerts on age/backlog or permanent schema/routing failure.

It must not reconstruct the event from the current aggregate after commit; later state may have changed.

### 6.2 EventBridge mapping

| EventBridge field | Value |
|---|---|
| Event bus | Environment-specific Study Notes Creator custom bus |
| `source` | Same value as envelope `producer` |
| `detail-type` | `<event_type>.v<event_version>`, for example `NoteVersionCreated.v1` |
| `time` | Envelope `occurred_at` |
| `detail` | Full canonical application envelope |
| `resources` | Empty unless an approved non-sensitive product resource ARN is required |

EventBridge's transport ID is not the Domain Event ID. Consumers deduplicate using `detail.event_id`.

### 6.3 Delivery guarantees

- Delivery is at least once.
- There is no global ordering and no assumed ordering across aggregate IDs.
- Standard SQS queues are the default. Aggregate versions provide correctness under reordering.
- FIFO queues require measured need and an ADR amendment; they do not remove the need for idempotency.
- EventBridge/SQS is transport, not an event store or source of truth.
- New consumers bootstrap from owner Snapshots, not an assumption that the bus retains complete history.

### 6.4 Queue naming

Initial naming:

```text
snc-<environment>-<consumer>-domain-events-v1
snc-<environment>-<consumer>-domain-events-v1-dlq
```

A consumer may split queues only for justified isolation/throughput/security. A shared queue read by several Services is prohibited.

---

## 7. Consumer processing protocol

For each message, a consumer:

1. Parses and validates the envelope and supported schema version.
2. Verifies the expected producer/event route; queue delivery alone is not trust.
3. Starts/links a trace using `trace_id` while retaining the new consumer span identity.
4. Opens a local database transaction.
5. Checks its processed-event/business key.
6. Applies the subscription mode's aggregate-version, business-key, and Tombstone rules.
7. Updates its local projection/side effect and processed-event row atomically.
8. Commits, then acknowledges the SQS message.

### 7.1 Duplicate, stale, and gap rules

Consumers declare one mode per subscribed fact:

- **Snapshot projection:** the event invalidates or advances a local copy. Aggregate versions detect stale state and trigger reconciliation. A version jump may mean a filtered/unpublished owner change, not necessarily lost transport, so the owner Snapshot is authoritative.
- **Sparse fact trigger:** the consumer intentionally receives only selected facts, such as terminal outcomes. It deduplicates using event ID plus a domain business key and does not require contiguous aggregate versions.

| Condition | Required behavior |
|---|---|
| `event_id` already processed | Make no side effect; acknowledge as duplicate |
| Snapshot projection: aggregate version equals expected next version | Apply transactionally or fetch the version-pinned Snapshot |
| Snapshot projection: aggregate version is less than/equal to local source version | Treat as stale; record/acknowledge without regressing state |
| Snapshot projection: aggregate version is greater than expected next version | Pause that aggregate, obtain authorized owner Snapshot, reconcile, then re-evaluate |
| Sparse fact trigger | Validate current fact if necessary; apply once using an explicit business key such as (`notification_type`, `generation_id`, terminal state) |
| Several subscribed facts share one aggregate version | Do not deduplicate by aggregate version; use event ID/business key and reconcile the full Snapshot where projection state matters |
| Create event is missing or consumer starts late | Bootstrap/reconcile from owner Snapshot; do not fabricate defaults |
| Delete/purge event arrives with a gap | Apply protective hide/Tombstone immediately, then reconcile; delayed upserts cannot resurrect it |
| Schema major is unsupported | Do not guess; send to DLQ/alert under the queue policy |

Retries must be safe even if a process crashes after local commit but before SQS acknowledgement.

### 7.2 Snapshot reconciliation

- Snapshot APIs are internal, workload-authenticated, versioned, paginated, and owner-authorized.
- The consumer stores the source aggregate version/content hash returned by the owner.
- Reconciliation is rate-limited and observable; a gap in one aggregate does not stop the whole queue.
- If the owner reports deleted/not found with an applicable Tombstone, the consumer removes its copy and records its own Tombstone.

### 7.3 New-consumer bootstrap

1. Provision the consumer queue/DLQ and routing rule first so new changes accumulate.
2. Export/read paginated owner Snapshots with aggregate versions.
3. Build the local projection idempotently.
4. Drain queued events using version checks.
5. Compare counts/versions and declare the projection ready.

### 7.4 DLQ and redrive

- Retry only classified transient failures. Validation, invariant, and authorization defects are not hidden by endless retry.
- Queue visibility timeout exceeds the handler's maximum processing time and is extended only through controlled heartbeats where justified.
- After the configured receive limit, SQS moves the original message to the consumer's DLQ.
- DLQ depth greater than zero alerts the consumer owner.
- Redrive occurs only after root cause, compatibility, and duplicate-safety are verified.
- Redrive preserves `event_id` and is audited. Editing the message creates a new explicitly repaired event/process, never a silent mutation.
- A poison event affecting one aggregate must not permanently block unrelated aggregates.

Exact receive counts, retention windows, and retry delays are finalized in ADR-006 and each consumer runbook before activation.

---

## 8. Security, privacy, and authorization

### 8.1 Payload rule

Only `C1 Internal` and `C2 Confidential` event payloads are allowed. `C3 Restricted` content/capabilities are prohibited.

Never include:

- Raw Media, frames, audio, Transcript text, Note/document text, Evidence text, model prompts/responses, or feedback free text.
- Access/refresh/session/device/share tokens, webhook signatures, secrets, cookies, checkout/download/presigned URLs.
- Object-storage keys when an opaque registered asset ID suffices.
- Full provider payloads, raw exceptions, email addresses, display names, Source URLs with query parameters, or payment details.

Use opaque IDs, versions, enums, counts, safe error codes, hashes only when necessary, and authorized Snapshot/object APIs for content.

### 8.2 Authorization

- Events are internal facts, not authorization grants.
- A consumer handling an event for a user/resource validates owner scope before exposing or mutating its local view.
- `account_id` in a payload is routing/projection metadata, not proof of Actor authority.
- Producer/consumer IAM restricts EventBridge sources/detail types and queue access.
- Queues, DLQs, Outbox tables, archives if enabled, and telemetry are encrypted and access-audited.
- Events never cross environments/accounts through wildcard routing.

### 8.3 Deletion precedence

Lifecycle/deletion events carry no deleted content. Consumers hide/delete local projections promptly and store content-free Tombstones. A delayed creation/update event cannot override a Tombstone without an explicit, versioned restore fact or owner Snapshot proving restoration.

---

## 9. Event lifecycle statuses

| Status | Meaning |
|---|---|
| `Planned` | Fact, producer, consumers, and minimum payload are approved here; executable schema not yet active |
| `Active` | Schema is published; producer and at least one consumer route are deployed and observed |
| `Deprecated` | Supported temporarily; no new consumer may subscribe |
| `Retired` | Producer no longer publishes and routes/schemas are retained only as required for history |
| `Deferred` | Candidate fact has no justified current consumer or duplicates another mechanism; must not be published |

Every initial event below is `Planned` because implementation repositories are not yet active.

---

## 10. Initial event inventory

### 10.1 Summary

| Producer | Planned event types | First producer milestone | Initial consumers |
|---|---|:---:|---|
| Account | `AccountDeletionRequested`, `AccountPurged` | M6 deletion capability | Workflow; Search/Notification safety cleanup |
| Library | `ArtifactCreated`, `ArtifactMetadataChanged`, `ArtifactLifecycleChanged`, `ArtifactDeleted` | M2/M6 | Search |
| Workflow | `GenerationStateChanged`, `GenerationReady`, `GenerationFailed`, `GenerationCanceled`, `DeletionStateChanged`, `PurgeCompleted` | M2/M6 | Library, Notification, Account |
| Transcription | `TranscriptCreated`, `TranscriptDeleted` | M4/M6 | Library, Search |
| Notes | `NoteVersionCreated`, `NoteChanged`, `NoteDeleted` | M2/M6 | Library, Search |
| Export | `ExportReady`, `ExportFailed` | M6 | Notification |
| Billing | `SubscriptionChanged`, `EntitlementChanged` | M7 | Notification |

The first-party Auth Service, Edge API, Ingestion, Media, AI Notes, Search, and Notification publish no initial Domain Events. Authentication security records remain Auth-owned audit/telemetry unless a concrete cross-Service domain consumer is approved. Auth-to-Account bootstrap and Auth participation in deletion use idempotent private Commands/Activities, not credential-adjacent events. This does not prevent these Services from using Outbox infrastructure later after an event is approved.

### 10.2 Planned count

| Category | Count |
|---|---:|
| Account lifecycle | 2 |
| Artifact lifecycle | 4 |
| Generation/deletion lifecycle | 6 |
| Transcript lifecycle | 2 |
| Note lifecycle | 3 |
| Export lifecycle | 2 |
| Subscription/Entitlement lifecycle | 2 |
| **Total planned event types** | **21** |

---

## 11. Minimum event contracts

All event versions below are `v1` and `C2` unless explicitly stated. Minimum payloads may gain compatible optional fields only through schema review.

### 11.1 Account producer

**Producer source:** `study-notes-creator.account-service`

| Event | Aggregate | Committed trigger | Minimum payload | Consumers | First milestone |
|---|---|---|---|---|:---:|
| `AccountDeletionRequested` | `AccountDeletionRequest` / deletion-request ID | Account records an authorized deletion request and effective/recovery policy | `account_id`, `deletion_request_id`, `requested_at`, `effective_after` | Workflow starts/idempotently links Deletion Saga | M6 |
| `AccountPurged` | `Account` / Account ID | Account-owned personal/product data is purged/anonymized and local Tombstone committed | `account_id`, `purged_at`, `tombstone_version` | Search and Notification remove any missed Account-scoped projection; other cleanup remains Saga-owned | M6 |

`AccountPurged` does not assert that statutory Billing records were erased. It asserts completion of the Account Service's approved purge responsibility.

### 11.2 Library producer

**Producer source:** `study-notes-creator.library-service`

| Event | Aggregate | Committed trigger | Minimum payload | Consumers | First milestone |
|---|---|---|---|---|:---:|
| `ArtifactCreated` | `Artifact` / Artifact ID | Artifact ownership and initial lifecycle state commit | `account_id`, `artifact_id`, `source_id`, `lifecycle_state` | Search when activated | M2 |
| `ArtifactMetadataChanged` | `Artifact` / Artifact ID | User-visible Artifact metadata changes | `account_id`, `artifact_id`, `changed_fields[]` | Search fetches current authorized Snapshot | M2 |
| `ArtifactLifecycleChanged` | `Artifact` / Artifact ID | Archive/trash/restore/deleting state transition commits | `account_id`, `artifact_id`, `previous_state`, `current_state`, optional `recovery_deadline` | Search hides/restores projection according to state | M2 |
| `ArtifactDeleted` | `Artifact` / Artifact ID | Library purge and Artifact Tombstone commit after the recovery/Saga policy | `account_id`, `artifact_id`, `deleted_at`, `tombstone_version` | Search removes content and records Tombstone | M6 |

Titles, Source URLs, and content are not event fields. Search obtains its approved Snapshot from Library.

### 11.3 Workflow producer

**Producer source:** `study-notes-creator.workflow-service`

| Event | Aggregate | Committed trigger | Minimum payload | Consumers | First milestone |
|---|---|---|---|---|:---:|
| `GenerationStateChanged` | `Generation` / Generation ID | Any monotonic Generation state/stage transition commits | `account_id`, `artifact_id`, `generation_id`, `previous_state`, `current_state`, `stage`, `progress_percent`, optional `safe_error_code` | Library maintains bounded status/read projection | M2 |
| `GenerationReady` | `Generation` / Generation ID | Note Version is durable, Library current reference is set, and usage is settled or Billing is not active | `account_id`, `artifact_id`, `generation_id`, `note_id`, `note_version_id`, optional `transcript_version_id`, `completed_at`, `degraded` | Notification creates user-facing readiness notification | M2 producer / M6 consumer |
| `GenerationFailed` | `Generation` / Generation ID | Terminal failure and required compensation settle | `account_id`, `artifact_id`, `generation_id`, `failed_at`, `safe_error_code`, `retryable_as_new_generation` | Notification | M2 producer / M6 consumer |
| `GenerationCanceled` | `Generation` / Generation ID | Cancellation and required compensation settle | `account_id`, `artifact_id`, `generation_id`, `canceled_at`, `safe_reason_code` | Notification where policy enables it | M2 producer / M6 consumer |
| `DeletionStateChanged` | `DeletionRequest` / deletion-request ID | Deletion Saga state changes | `account_id`, `deletion_request_id`, `scope_type`, `scope_id`, `previous_state`, `current_state`, optional `safe_error_code` | Account and Library update request/read state | M6 |
| `PurgeCompleted` | `DeletionRequest` / deletion-request ID | Every required Service acknowledgement commits and Workflow marks purge complete | `account_id`, `deletion_request_id`, `scope_type`, `scope_id`, `completed_at`, `acknowledged_owner_count` | Account and Library finalize visible deletion state | M6 |

`GenerationStateChanged` is the general projection fact. Terminal-specific events exist only for user-facing outcome consumers and are emitted in the same terminal transaction with distinct event IDs. A consumer subscribes to the general or terminal fact for its purpose and must not process both as the same side effect.

### 11.4 Transcription producer

**Producer source:** `study-notes-creator.transcription-service`

| Event | Aggregate | Committed trigger | Minimum payload | Consumers | First milestone |
|---|---|---|---|---|:---:|
| `TranscriptCreated` | `TranscriptVersion` / Transcript Version ID | Normalized segments and immutable Transcript Version are durably queryable | `account_id`, `artifact_id`, `generation_id`, `transcript_version_id`, `language_code`, `segment_count`, `duration_ms` | Library stores current/reference projection; Search fetches authorized text Snapshot | M4 |
| `TranscriptDeleted` | `TranscriptVersion` / Transcript Version ID | Transcript data is purged and Tombstone commits | `account_id`, `artifact_id`, `transcript_version_id`, `deleted_at`, `tombstone_version` | Library clears stale ref; Search deletes projection | M6 deletion |

No segment text, speaker label, provider payload, or word timing appears in the event.

### 11.5 Notes producer

**Producer source:** `study-notes-creator.notes-service`

| Event | Aggregate | Committed trigger | Minimum payload | Consumers | First milestone |
|---|---|---|---|---|:---:|
| `NoteVersionCreated` | `Note` / Note ID | Valid Note Version/document and Citation manifest commit | `account_id`, `artifact_id`, `note_id`, `note_version_id`, `document_schema_version`, `origin`, optional `generation_id`, optional `transcript_version_id` | Library registers/reconciles the available Note reference; Search fetches authorized Snapshot | M2 |
| `NoteChanged` | `Note` / Note ID | Debounced server autosave or restore commits a new accepted Revision/current state | `account_id`, `artifact_id`, `note_id`, `revision`, `change_kind`, optional `current_note_version_id` | Search refreshes current Note Snapshot | M6 |
| `NoteDeleted` | `Note` / Note ID | Note data/share capability is purged and Tombstone commits | `account_id`, `artifact_id`, `note_id`, `deleted_at`, `tombstone_version` | Library clears stale ref; Search deletes projection | M6 deletion |

`NoteChanged` is emitted per accepted debounced save/restore, not per browser keystroke. It contains no document patch or text.

`NoteVersionCreated` does not select the Artifact's Current Note. Workflow invokes Library's versioned `set_current_note_reference` Activity/Command after materialization; the event is a reconciliation fact, not that Command.

### 11.6 Export producer

**Producer source:** `study-notes-creator.export-service`

| Event | Aggregate | Committed trigger | Minimum payload | Consumers | First milestone |
|---|---|---|---|---|:---:|
| `ExportReady` | `Export` / Export ID | Rendered object and output registry commit successfully | `account_id`, `artifact_id`, `note_version_id`, `export_id`, `format`, `ready_at`, `expires_at` | Notification | M6 |
| `ExportFailed` | `Export` / Export ID | Export reaches terminal failed state | `account_id`, `artifact_id`, `note_version_id`, `export_id`, `format`, `failed_at`, `safe_error_code` | Notification | M6 |

`ExportReady` contains no object key or download URL. The authorized client requests a fresh short-lived URL from Export.

### 11.7 Billing producer

**Producer source:** `study-notes-creator.billing-service`

| Event | Aggregate | Committed trigger | Minimum payload | Consumers | First milestone |
|---|---|---|---|---|:---:|
| `SubscriptionChanged` | `Subscription` / product Subscription ID | Verified webhook/reconciliation changes canonical subscription state | `account_id`, `subscription_id`, `previous_status`, `current_status`, `plan_code`, `effective_at` | Notification for approved user-facing changes | M7 |
| `EntitlementChanged` | `Entitlement` / Account Entitlement ID | Derived Entitlement/allowance policy changes and commits | `account_id`, `entitlement_id`, `entitlement_version`, `changed_capabilities[]`, `effective_at` | Notification; future consumers query Billing for details | M7 |

No payment instrument, invoice address, webhook payload, checkout URL, exact private allowance ledger, or Stripe secret appears in these events.

---

## 12. Consumer subscription map

### 12.1 Initial queues and routes

| Consumer queue | Subscribed events | Mode | Local result | Activation |
|---|---|---|---|:---:|
| `snc-<env>-workflow-domain-events-v1` | `AccountDeletionRequested.v1` | Sparse fact trigger | Start/link Account Deletion Saga using `deletion_request_id` | M6 |
| `snc-<env>-account-domain-events-v1` | `DeletionStateChanged.v1`, `PurgeCompleted.v1` | Snapshot projection | Update Account deletion-request projection/state | M6 |
| `snc-<env>-library-domain-events-v1` | `GenerationStateChanged.v1`, `DeletionStateChanged.v1`, `PurgeCompleted.v1`, `TranscriptCreated.v1`, `TranscriptDeleted.v1`, `NoteVersionCreated.v1`, `NoteDeleted.v1` | Snapshot projection | Maintain bounded Artifact resource/status references | M2 onward |
| `snc-<env>-search-domain-events-v1` | Account purge; all Artifact events; Transcript create/delete; Note version/change/delete | Snapshot projection | Build/remove/reconcile user-scoped search documents | M6 |
| `snc-<env>-notification-domain-events-v1` | Account purge; Generation ready/failed/canceled; Export ready/failed; Subscription/Entitlement changed | Sparse fact trigger | Create/dedupe/purge safe user notifications with explicit business keys | M6/M7 |

“Account purge” means `AccountPurged.v1`. Each queue has a same-name `-dlq`.

### 12.2 No direct client subscriptions

Web, Electron, and React Native do not read EventBridge/SQS. They receive:

- Command/query responses through Edge API.
- Ordered Job Events through Workflow-backed SSE/polling.
- In-app notifications through Notification API.

This prevents internal event topology and payloads from becoming a public compatibility surface.

---

## 13. Deferred candidate facts

These names appeared as possible events during Service boundary design but are not approved for initial publication.

| Producer | Deferred candidates | Reason / current mechanism |
|---|---|---|
| Auth | `IdentityRegistered`, `IdentityStatusChanged`, `IdentityPurged` | Account bootstrap/reconciliation and Deletion Saga use idempotent private Commands/Activities; authentication security changes remain Auth-owned audit. Add only for a concrete asynchronous consumer with security review. |
| Account | `AccountCreated`, `AccountProfileChanged`, `AccountPreferencesChanged` | Auth receives `account_id` from the synchronous idempotent bootstrap response and can reconcile through Account API; other clients use Account API. Add only for a concrete projection/notification with privacy review. |
| Ingestion | `SourceAccepted`, `UploadCompleted`, `AssetReady`, `AssetDeleted` | Workflow receives Activity/Command results; deletion receives Activity acknowledgements. Do not duplicate orchestration on the event bus. |
| Workflow | `GenerationAccepted` | Duplicates the first `GenerationStateChanged` transition and synchronous accepted response. |
| Media | `MediaPrepared`, `MediaRejected`, `MediaDerivativeDeleted` | Workflow receives Activity results; operational metrics cover tool behavior. |
| Transcription | `TranscriptQualityDegraded` | Workflow receives quality result and decides Generation policy; no independent consumer yet. |
| AI Notes | `NoteDraftReady`, `QualityReportCreated`, `GenerationQualityFailed` | Workflow/Notes receive versioned Activity result/manifest; no independent bus consumer. |
| Notes | `ShareChanged` | Share reads/revocation are Notes-owned synchronous behavior; no consumer. |
| Notes | `FeedbackSubmitted` | Product analytics is a separate privacy-reviewed Product Event; no domain consumer. |
| Export | `ExportRequested` | Internal render dispatch uses Temporal/owned work queue, not a public domain fact. |
| Export | `ExportExpired` | Export API is authoritative and expiry is internal lifecycle; no consumer currently needs it. |
| Billing | `UsageReserved`, `UsageSettled`, `UsageReleased` | Workflow receives Activity results; Billing ledger is authoritative. Add only if a justified asynchronous projection appears. |
| Notification | `NotificationCreated`, `NotificationDelivered`, `NotificationFailed` | No domain consumer; delivery metrics/audit remain Notification-owned. |
| Search | Projection/rebuild events | Operational telemetry unless another domain consumer is approved. |
| Edge API | Any routing/auth event | Edge owns no domain state; use security/operational telemetry. |

Deferred candidates have no route or compatibility promise. Activating one requires the change procedure in section 16.

---

## 14. Testing and operational evidence

### 14.1 Producer tests

- Domain change and Outbox insert commit or roll back together.
- Event IDs are unique and stable across publisher retry.
- Envelope and payload validate against the released schema.
- Aggregate versions increment monotonically under concurrency.
- Event contains no prohibited C3 content/capabilities.
- EventBridge partial failure is retried without marking failed entries published.
- Producer remains functional while EventBridge is unavailable; Outbox backlog/age alerts.

### 14.2 Consumer tests

- Duplicate delivery creates one intended local effect.
- Crash after local commit but before acknowledgement remains idempotent.
- Stale event never regresses projection state.
- Gap triggers owner Snapshot reconciliation.
- Delete/purge hides data despite version gaps and blocks delayed resurrection.
- Unsupported schema and poison events reach DLQ visibly.
- Redrive succeeds without duplicate notification/index/ledger side effects.
- Cross-Account event IDs cannot expose or overwrite another Account's projection.
- Full projection rebuild from owner Snapshots plus queued changes converges.

### 14.3 Contract/integration tests

- Producer examples validate against published JSON Schema/AsyncAPI.
- Every routed consumer tests all supported event major versions.
- EventBridge rule matches exact `source` and `detail-type`.
- Queue policy accepts only the expected bus/rule and workload reads only its own queue/DLQ.
- Traces link Command → Outbox → EventBridge → SQS → consumer.
- Deletion Saga and Search projection tests inject duplicates, reorder, gaps, outages, and DLQ redrive.

### 14.4 Required metrics and alerts

Producer:

- Outbox unpublished count and oldest age.
- Publish attempts/failures/partial failures and latency.
- Schema-validation failure and event count by type/version.

Consumer:

- Queue depth, oldest-message age, receive/processing/failure rate.
- Duplicate, stale, gap, reconciliation, and unsupported-version counts.
- DLQ depth/redrive result.
- Projection freshness/rebuild progress where applicable.

No metric label contains Account/resource IDs, Source titles/URLs, or unbounded error text.

---

## 15. Activation checklist

Before a planned event becomes `Active`:

- [ ] Producer and every consumer have named accountable owners.
- [ ] Domain fact, trigger transaction, aggregate/version, and minimum payload match this catalog.
- [ ] Data owner and C1/C2 classification are confirmed against `DATA_OWNERSHIP.md`.
- [ ] JSON Schema/AsyncAPI and examples are reviewed and immutably published.
- [ ] Producer Outbox migration/publisher and consumer queue/DLQ/processed-event migration exist.
- [ ] EventBridge routing/IAM is exact and environment-isolated.
- [ ] Idempotency, stale, gap, deletion, Snapshot, replay, and DLQ behavior are documented.
- [ ] Producer and consumer contract/integration/security tests pass.
- [ ] Dashboards, alerts, runbooks, redrive procedure, and on-call ownership exist.
- [ ] Deployment sequence is consumer-compatible and rollback does not lose facts.
- [ ] Catalog status and repository `SERVICE.md` files are updated after observation.

---

## 16. Event change procedure

Adding, activating, versioning, rerouting, deprecating, or retiring a Domain Event requires:

1. A named domain fact, producer, aggregate, committed trigger, and real asynchronous consumer.
2. Explanation of why synchronous API, Temporal Activity result, Job Event, Product Event, or telemetry is not the right mechanism.
3. Minimum payload/classification review and confirmation that owner Snapshot lookup is used for C3 content.
4. Consumer idempotency, ordering/gap, deletion, bootstrap/rebuild, and failure design.
5. Schema/AsyncAPI compatibility report and cross-repository rollout/rollback plan.
6. Updates to this catalog, `SERVICE_CATALOG.md`, `DATA_OWNERSHIP.md` when needed, contracts, and all affected `SERVICE.md` files.
7. Evidence from contract, integration, duplicate/reorder/gap, DLQ, and security tests.

Removing the last consumer normally means stopping publication after review. Do not retain event production merely because it already exists.

---

## 17. Decisions to close in ADR-006 / M1

The event semantics in this catalog are approved for planning. These platform/operational values remain to be decided before the first active route:

- EventBridge custom-bus/account/environment topology and archive policy.
- Exact SQS retention, visibility timeout, receive limit, DLQ retention, and redrive approval process.
- Outbox polling/claiming implementation, publish batch size, and R10 retention.
- Schema registry/package publication and compatibility tooling.
- CloudEvents binding details and OpenTelemetry propagation implementation.
- Snapshot bootstrap protocol endpoints and rate limits.
- Encryption keys, cross-account restrictions, support access, and audit retention.
- Numeric freshness/backlog/DLQ SLOs and alert thresholds.

ADR-006 must choose these values without changing the domain facts or ownership defined here.

---

## Appendix A: Event proposal template

```markdown
# <PastTenseEventName>

**Status/version:** Planned / v1
**Producer:**
**Aggregate type/ID/version:**
**Committed trigger:**
**Why asynchronous:**

## Minimum payload

| Field | Type | Required | Meaning | Classification |
|---|---|:---:|---|---|

## Consumers

| Consumer | Local effect | Idempotency key | Gap/reconciliation | Failure behavior |
|---|---|---|---|---|

## Privacy/security

Excluded C3 content, authorization, IAM, Snapshot/object access.

## Compatibility and rollout

Schema artifact, producer/consumer sequence, rollback, support window.

## Verification

Contract, duplicate, stale, gap, deletion, DLQ/redrive, isolation tests.
```

## Appendix B: Consumer decision table

```text
if event_id already processed:
    acknowledge duplicate
elif local tombstone blocks event aggregate/version:
    acknowledge stale event
elif event is delete/purge:
    hide/delete + tombstone transactionally
    reconcile any version gap asynchronously
elif subscription mode is sparse fact trigger:
    validate and apply once using the domain business key
elif aggregate_version <= local source version:
    acknowledge stale event
elif aggregate_version == local source version + 1:
    apply + record processed event transactionally
else:
    pause this aggregate
    fetch authorized owner snapshot
    reconcile and re-evaluate
```

This pseudocode is a behavior standard, not a shared business-rule library.

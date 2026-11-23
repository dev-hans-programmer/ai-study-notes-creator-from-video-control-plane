# Study Notes Creator: Data Ownership and Lifecycle Catalog

**Document status:** Active data-boundary standard  
**Version:** 1.0  
**Last updated:** 23 November 2026  
**Current delivery milestone:** M0 — Product control plane  
**Applies to:** Backend Services, clients, databases, object storage, queues, caches, telemetry, backups, and external processors  
**Related documents:** [PRD.md](./PRD.md), [TRD.md](./TRD.md), [GLOSSARY.md](./GLOSSARY.md), [SERVICE_CATALOG.md](./SERVICE_CATALOG.md), [EVENT_CATALOG.md](./EVENT_CATALOG.md), [ROADMAP.md](./ROADMAP.md), [BUILD_PLAYBOOK.md](./BUILD_PLAYBOOK.md), [repos.md](./repos.md)

---

## 1. Purpose

This document assigns every planned durable product-data group to exactly one authoritative Service. It also defines:

- The permitted physical stores and cross-Service copies.
- Data classification and handling labels.
- Default retention, deletion, recovery, export, backup, and logging behavior.
- Ownership of object-storage prefixes, projections, tombstones, and operational records.
- Rules for cross-Service identifiers, snapshots, restoration, and schema change.

The goal is to prevent shared-database coupling, ambiguous deletion, accidental Private Content duplication, and AI-generated tables with no accountable owner.

This document defines **logical ownership**, not final SQL schemas. Exact table/column names are approved in each Service's `SERVICE.md` and migrations. A new durable field, table, object prefix, cache, index, queue payload, analytics property, or provider-retained copy is not implementation detail: it requires an owner, classification, retention, deletion, export, and logging decision.

### 1.1 Authority

| Question | Authority |
|---|---|
| Meaning of a domain term | `GLOSSARY.md` |
| Which Service owns a capability | `SERVICE_CATALOG.md` |
| Which Service owns a durable datum and its lifecycle | This document |
| Exact database/object schema | Owning Service's approved `SERVICE.md` and migrations |
| Domain Event payload classification and consumers | `EVENT_CATALOG.md` and contracts repository |
| Product requirement and user promise | `PRD.md` |
| System mechanism and infrastructure | `TRD.md` and accepted ADRs |

If a proposed schema conflicts with this catalog, implementation stops until this document and any affected ADR are reviewed.

---

## 2. Ownership model

### 2.1 Authoritative owner

The authoritative owner is the only Service allowed to define the meaning, invariants, lifecycle, and canonical mutation of a data group. It:

- Validates the Actor and command.
- Assigns canonical identifiers and versions.
- Performs writes through its own API, Worker, or Activity.
- Owns schema migrations, backups, restoration, reconciliation, deletion, and support tooling.
- Publishes facts or authorized snapshots for consumers.

A repository, database cluster, S3 bucket, AWS account, vendor, or team is not automatically the domain owner.

### 2.2 Local custodian of a derived copy

A consuming Service may own the lifecycle of its **local derived copy** without becoming authoritative for the source fact. For example:

- Notes owns a Citation record but Transcription remains authoritative for the referenced Transcript Version and segments.
- Search owns a search document but Notes remains authoritative for the Note Version from which it was built.
- Export owns an expiring rendered PDF but Notes remains authoritative for the source Note Version.

The local custodian must record the source owner, opaque ID, source version/content hash, classification, and deletion linkage. It may rebuild or discard the copy; it may not silently reinterpret the source fact.

### 2.3 Platform operator

Infrastructure operates PostgreSQL, S3, EventBridge, SQS, caches, secrets, backups, and telemetry sinks. That operational role does not make Infrastructure the domain owner. The Service that writes a record remains responsible for its schema, minimization, access, retention, and deletion.

### 2.4 External system of record

Some records intentionally remain authoritative outside this product:

- Stripe owns payment instruments and card/bank details.
- Source platforms own their provider content and access rules.

End-user authentication is explicitly **not** external: Auth Service owns Identity, credentials, verification/recovery, sessions, and tokens. External email delivery and KMS are processors/platform custody, not identity authorities. Product Services store only the minimum identifiers and normalized state required for their purpose. Section 12 defines these boundaries.

---

## 3. Mandatory ownership rules

1. Every table, column, object prefix, durable cache entry, search document, provider-retained record, and persistent queue payload has one named owner before implementation.
2. Each Service uses a separate logical database, database role, migration history, secret, and backup/restore test, even if databases initially share a PostgreSQL cluster.
3. Cross-database queries, shared tables, foreign data wrappers, cross-Service foreign keys, and shared ORM models are prohibited.
4. Cross-Service references are opaque identifiers plus the minimum approved immutable version/hash/snapshot. They are not database relationships.
5. Only the authoritative owner mutates canonical state. Consumers request a command from the owner or update a local projection from a versioned fact.
6. The receiving Service validates Actor authority and referenced ownership before accepting a cross-Service relationship.
7. Private Content bodies, prompts containing Source content, access/refresh tokens, presigned URLs, share tokens, and payment details are prohibited in Domain Events, Temporal histories, general logs, traces, metrics, and product analytics.
8. Large or private payloads use an authenticated owner API or narrowly scoped immutable object reference. The reference does not transfer ownership.
9. Object storage is not a registry. Every object has an owning Service record, checksum, classification, creation purpose, and retention deadline.
10. A projection is disposable and rebuildable. It never becomes authoritative because it is faster or easier to query.
11. Caches are never the sole copy. Cache loss must affect latency, not correctness or ownership.
12. Soft deletion controls visibility; the owning Service performs physical purge. No Service deletes another Service's rows or objects.
13. Every owning Service records a content-free local Tombstone after purge. Workflow owns the cross-Service Deletion Saga ledger and acknowledgements.
14. Restored data is not served until deletion Tombstones and the Workflow deletion ledger have been reconciled.
15. AI implementation must not add a “temporary” durable field or object without applying this document. Temporary data still has an owner and deadline.

---

## 4. Data classification

Classification is based on the impact of unauthorized disclosure, alteration, or loss. When several classes apply, use the highest class.

| Class | Meaning | Examples | Minimum handling |
|---|---|---|---|
| `C0 Public` | Explicitly approved for unrestricted public disclosure | Published marketing content; intentionally public documentation | Integrity controls and TLS; approval before changing a non-public datum to C0 |
| `C1 Internal` | Non-public operational/configuration data with low direct user impact | Schema versions, safe health status, aggregate service metrics | Authenticated access, encryption in transit/at rest, approved internal logging |
| `C2 Confidential` | Personal, ownership, commercial, or user metadata that must be access-controlled | Account profile, Artifact title, Source metadata, Entitlements, delivery history | Least privilege, user/workload authorization, audit where sensitive, no unrestricted logs/analytics |
| `C3 Restricted` | Private Content, security capabilities/secrets, or highly sensitive personal/financial data | Raw Media, Transcripts, Notes, prompts, feedback excerpts, device/share tokens, signed URLs | Narrow purpose-bound access, strict IAM/egress, encryption, audited support access, content-free events/logs, minimized retention |

`C0` is never inferred from “available on the internet.” A YouTube URL, public-source metadata, or unlisted share remains access-controlled unless explicitly approved for public disclosure.

### 4.1 Sensitivity tags

Use one or more tags in addition to the class:

| Tag | Meaning |
|---|---|
| `PD` | Personal data relating to an identifiable person |
| `PC` | Private Content supplied by or derived for a user |
| `SEC` | Credential, token, signing material, capability, or security-control data |
| `FIN` | Subscription, billing, cost, or financial metadata |
| `RIGHTS` | Rights attestation, provider terms, takedown, or acquisition-compliance evidence |
| `OPS` | Operational, audit, diagnostic, or reliability data |

Classification does not replace a threat model or legal/privacy review.

---

## 5. Retention profiles

These launch baselines preserve the defaults already established by the PRD/TRD. They remain subject to product/legal approval and must match user-facing disclosure before release.

| Profile | Default retention | Applies to |
|---|---|---|
| `R0 Request-only` | Memory/request lifetime; not durably persisted | Raw secrets, presigned URLs, unneeded provider payloads |
| `R1 Upload-session` | Abort/delete after 24 hours inactive | Incomplete multipart upload state and parts |
| `R2 Processing-media` | Delete within 24 hours after the Generation reaches a terminal state | Uploaded Raw Media, extracted audio, sampled frames; rejected/quarantined sources delete within 24 hours |
| `R3 AI-temporary` | Delete within 7 days, sooner where practical | Temporary chunks/candidates/provider intermediates not required for durable evidence |
| `R4 Resource-lifetime` | Retain while the Account/Artifact is active; then follow trash/deletion policy | Account, Artifact, Transcript, Note, evidence/citation, user-facing metadata |
| `R5 Recoverable-trash` | 30 days, hidden immediately; purge after the recovery window | Recoverable deleted Artifact data explicitly required for restoration |
| `R6 Export-output` | Export object 24 hours; download URL no more than 15 minutes | Markdown/PDF outputs and download capabilities |
| `R7 Backup` | Database point-in-time backups 35 days | Service databases; restored data must reconcile Tombstones before readiness |
| `R8 Security-audit` | Proposed one year with minimized metadata | Approved security/audit records; requires policy approval before production |
| `R9 Policy-bound` | No default may be guessed; approve before the owning Service activates the datum | Billing/statutory records, consent evidence, delivery history, provider receipts, legal/takedown records |
| `R10 Dedupe-replay` | At least the maximum retry/redelivery/replay window; exact duration per contract before activation | Idempotency keys, outbox receipts, processed-event IDs, Workflow stage keys |

Rules:

- A shorter profile applies when the purpose ends earlier.
- `R9` is an explicit unresolved decision and an activation blocker, not permission to retain indefinitely.
- Object lifecycle rules are a safety net. The owning Service must execute and audit deletion.
- Database backups expire through `R7`; deleting a live row does not rewrite immutable backups. Restores must reapply Tombstones.
- A legal hold or rights/takedown preservation rule requires an approved policy, scoped access, audit, and user disclosure where applicable. It is never added ad hoc.

---

## 6. Master ownership map

| Service | Authoritative durable data groups | Canonical stores |
|---|---|---|
| Auth | Auth Identity, Login Identifier, password/passkey/MFA credential, verification/recovery challenge, Authentication Session, refresh-token family/hash, signing-key metadata, authentication security audit, Auth Tombstone | Auth PostgreSQL database, KMS/Secrets Manager, approved ephemeral security cache |
| Edge API | No business data; only minimized Edge operational records | Telemetry sink; approved ephemeral cache only |
| Account | Account, Auth Identity mapping, profile, product preferences, consent, account-deletion request, Account Tombstone | Account PostgreSQL database |
| Library | Artifact aggregate, ownership, title, lifecycle, resource references, trash state, Artifact Tombstone | Library PostgreSQL database |
| Ingestion | Source, safe/source locator, rights attestation, acquisition state, upload session, source/raw asset registry, retention task | Ingestion PostgreSQL database and source/quarantine S3 prefixes |
| Workflow | Generation, Generation Settings snapshot, Job, ordered Job Events, Workflow binding, retry/cancellation/compensation state, Deletion Saga ledger | Workflow PostgreSQL database and Temporal histories |
| Media | Media preparation execution, scan/probe result, derived asset registry, derivative cleanup state | Media PostgreSQL database and derived-media S3 prefix |
| Transcription | Transcript Version, Transcript Segment, language/speaker/timing metadata, normalized transcript snapshot, quality/provenance | Transcription PostgreSQL database and timing/snapshot S3 prefix |
| AI Notes | Semantic chunk where retained, Evidence Item/manifest, draft manifest, quality report, prompt/config reference, model-call/cost record | AI Notes PostgreSQL database and temporary/intermediate S3 prefix |
| Notes | Note/Working Version, Revision, Note Version/document, checkpoint, Citation, feedback, share link, share-token hash | Notes PostgreSQL database and optional immutable snapshot S3 prefix |
| Search | Artifact/Note/Transcript search documents, index state, event checkpoint, rebuild state, search Tombstone | Search PostgreSQL database/index |
| Export | Export request, render attempt, output registry, expiry/purge state | Export PostgreSQL database and export S3 prefix |
| Billing | Plan mapping, customer/subscription reference, Entitlement, allowance, usage estimate/reservation/ledger, webhook receipt | Billing PostgreSQL database |
| Notification | Delivery preference, in-app notification, device-token record, delivery attempt, provider-message reference | Notification PostgreSQL database |

The remaining sections are the detailed register. A physical table may split or combine a listed group inside one Service, but it cannot cross the boundary without updating this catalog.

---

## 7. Detailed Service-owned data

### 7.1 Auth Service

| Data group | Likely physical records | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Auth Identity | `identities`: `identity_id`, status, security version, timestamps, minimal `account_id` projection | C3 `PD`,`SEC` | R4 | Exportable at an understandable security-summary level; revoke immediately and purge/anonymize through Account deletion policy. Never reuse Identity ID as Account ID. |
| Login Identifier | `login_identifiers`: normalized email, verification state, uniqueness hash/index | C3 `PD`,`SEC` | R4 | User-visible/changeable under recent authentication; purge/anonymize under approved fraud/security policy. Never logs/events/analytics. |
| Password Credential | Argon2id hash, salt, parameters, pepper version, changed time | C3 `SEC` | R4 until replacement/deletion | Plaintext password never persisted, logged, exported, or sent outside Auth. Hash is not user-portable. Pepper material remains outside the database. |
| Verification challenge | keyed token hash, Identity/identifier, purpose, expiry, attempts, consumed time | C3 `PD`,`SEC` | R9 short security window defined before M2 | Raw token R0; single use; purge after expiry/consumption plus bounded abuse/audit need. |
| Recovery challenge | keyed token hash, Identity/identifier, purpose, expiry, attempts, consumed time | C3 `PD`,`SEC` | R9 short security window defined before M2 | Raw token R0; successful recovery revokes affected Sessions; no account enumeration. |
| Authentication Session | `authentication_sessions`: client, token-family ref, security version, created/last-used/expiry/revoked, bounded device/IP metadata | C3 `PD`,`SEC` | R9 active lifetime plus bounded security window | User can list/revoke; raw token absent; purge/anonymize with Identity subject to security policy. |
| Refresh-token family/member | family ID, parent/replacement relation, keyed token hash, status, issued/used/expiry, reuse evidence | C3 `SEC` | R9 active lifetime plus bounded reuse/audit window | Raw refresh token R0 and never logged/exported; reuse revokes family. |
| Passkey/WebAuthn credential | credential ID, public key, sign counter, transports, friendly label, status | C3 `PD`,`SEC` | R4 until revoke/delete | Private key remains on authenticator; user can list/revoke. |
| MFA authenticator | encrypted/hashed secret or public credential, method, state, recovery-code hashes | C3 `SEC` | R4 until revoke/delete | Raw seed/recovery code shown only at enrollment and never logged/exported after. |
| Account claim binding | `identity_id` → `account_id` and binding version returned by Account | C2 `PD`,`SEC` | R4 | Auth is custodian for token claims; Account owns canonical relationship. Reconcile before issuing normal product scopes. |
| Signing-key metadata | key ID, algorithm, KMS reference, state, activation/retirement times | C2 `SEC`,`OPS` | Key/audit policy R9 | Private signing key is non-exportable KMS custody and never database data. Retain public verification history through maximum token/forensic window. |
| Authentication security event | safe event code, Identity/session/client/IP risk metadata, outcome, trace/time | C2/C3 `PD`,`SEC`,`OPS` | R8/R9 | No passwords/tokens/challenge values. Access is restricted/audited; user-facing security history is a minimized projection. |
| Rate-limit/risk counter | keyed identifier/IP/client bucket, count, expiry | C2/C3 `PD`,`SEC`,`OPS` | Short ephemeral security window | Not canonical identity state; use keyed/pseudonymous values where possible. |
| Auth Tombstone | opaque/de-identified Identity/account refs, deletion/security version/time | C1/C2 `SEC`,`OPS` | At least R7 plus approved security window | Content-free; prevents restored credentials/session resurrection. |

**Prohibited:** plaintext passwords, raw verification/recovery/MFA codes, raw refresh/access tokens, private signing/pepper keys, product profile/consent, Artifact/Note content, payment instruments.

### 7.2 Edge API

Edge API owns **no durable business record** and has no business database.

| Data group | Classification | Retention | Lifecycle and restrictions |
|---|---|---|---|
| Minimized access/security log | C1/C2 `OPS`; user/resource IDs pseudonymized where possible | R8 or shorter approved telemetry policy | Edge owns emitted content; Infrastructure operates the sink. Never log tokens, cookies, bodies, Source locators, signed URLs, Note/Transcript text, or provider errors containing content. |
| Rate-limit counters | C1/C2 `OPS` | Short bounded window; ephemeral cache | Loss may reset/degrade rate enforcement safely; not a billing/Entitlement ledger. |
| SSE connection/replay cursor cache | C1/C2 `OPS` | Connection/reconnect window | Workflow's `job_events` remain authoritative. Cache loss falls back to Workflow replay/polling. |
| Public OpenAPI artifact | C0 after release approval | Release history | Repository/build artifact, not runtime user data. Edge owns its source; contracts repository publishes immutable versions. |

Edge must not persist a duplicate Account, Artifact, Job, Note, Entitlement, or combined “user view” table.

### 7.3 Account Service

| Data group | Likely physical records | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Product Account | `users`/`accounts`: `account_id`, status, creation timestamps | C2 `PD` | R4 | Included in authorized account export; purge/anonymize through Account deletion policy; no profile values in logs. |
| Auth Identity mapping | `identity_id`, Account link, binding state/version | C2/C3 `PD`,`SEC` | R4 | Account is authoritative for the relationship; Auth keeps a minimal claim projection. Delete/unlink through Deletion Saga; no credentials/tokens. |
| Profile | display name and approved non-login profile fields | C2 `PD` | R4 | User-readable/exportable/editable; purge with Account. Canonical Login Identifier/email remains Auth-owned. |
| Product preferences | locale, timezone, generation defaults, accessibility/product defaults | C2 `PD` | R4 | User-readable/exportable; Notification channel preferences do not belong here. |
| Consent records | consent type/version, decision, evidence timestamp, policy reference | C2 `PD` | R9 | User-exportable. Statutory evidence/anonymization period must be approved before production. Never store unnecessary content. |
| Account-deletion request | request ID, Actor, state, requested/effective times, Workflow reference | C2 `PD`,`OPS` | R9 | Account owns the request; Workflow owns execution ledger. Exportable while Account exists; minimized audit may outlive purge only under approved policy. |
| Account Tombstone | opaque/de-identified Account reference, deletion time, reason code | C1/C2 `OPS` | At least R7 plus reconciliation safety window; exact policy before activation | Content-free, not user profile. Used only to prevent resurrection and reconcile restore/events. |

**Prohibited:** authentication credentials, refresh/access tokens, payment instruments, Artifact/Note content, notification device tokens, subscription ledger.

### 7.4 Library Service

| Data group | Likely physical records | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Artifact aggregate | `artifacts`: Artifact ID, owner Account ID, user title, state, timestamps | C2 `PD`,`PC` | R4 then R5 | Immediately hidden on trash; included in account/resource export; purged after recovery window. Titles are not safe logs. |
| Artifact ownership | owner/workspace reference and authorization state | C2 `PD` | R4 then R5 | Library is authoritative. No consumer may infer ownership from possession of an Artifact ID. |
| Artifact resource references | Source, current Generation, Transcript Version, Note/Note Version IDs and safe versions/hashes | C2 `PC` | R4 then R5 | Opaque references only; no bodies or foreign keys. Cleared/tombstoned during purge. |
| Artifact lifecycle | active/archive/trash/deleting/deleted state, state version | C2 `OPS` | R4; Tombstone after purge | Archive is not deletion. Trash is hidden immediately and recoverable for 30 days. |
| Artifact read/status projection | bounded safe state derived from owner facts | C2 `PC`,`OPS` | While Artifact active; rebuildable | Never authoritative for Job, Note, or Transcript. Version gaps reconcile with owner. |
| Trash record | trash time, recovery deadline, requester, previous safe state | C2 `PD`,`OPS` | R5 | Supports restore only; must not retain extra content beyond recovery need. |
| Artifact Tombstone | opaque Artifact/owner hashes, deleted time/version | C1/C2 `OPS` | At least R7 plus safety window | Content-free; blocks delayed projection resurrection. |

**Prohibited:** Source locator credentials, Raw Media, Transcript segments, Note documents, share tokens, search index, billing state.

### 7.5 Ingestion Service

| Data group | Likely physical records/objects | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Source | `sources`: Source ID/type, owner, provider, state | C2 `PD`,`PC`,`RIGHTS` | R4; sensitive acquisition fields shorter | Included in resource/account export where permitted; purge with Artifact/Account. |
| Source locator | sanitized URL/provider ID; sensitive query/auth components separated | C2 or C3 `PC`,`SEC` | Public/safe locator R4; sensitive acquisition locator R0 or shortest acquisition window | Strip credentials/query secrets before persistence where possible. Never log or emit raw locator. |
| Source metadata | provider title, duration, language hint, thumbnail reference, attribution | C2 `PC`,`RIGHTS` | R4 | May be projected safely to Library; public origin does not make the user's association public. |
| Rights attestation | attestation version, Actor, time, permitted purpose/reference | C2 `PD`,`RIGHTS` | R9 | Exportable as user account evidence where appropriate; legal retention decision required. |
| Acquisition option/state | approved method, provider reference, state, safe failure code | C2/C3 `RIGHTS`,`OPS` | R4; transient credentials R0 | No DRM/access-control secrets. Provider responses normalized and discarded. |
| Upload session | `upload_sessions`, expected size/checksum/type, multipart ID, part state, expiry | C3 `PD`,`PC`,`SEC` | R1 | Abort/delete after 24 hours inactive; user may view status, not raw storage credentials. |
| Incomplete multipart parts | quarantine objects under upload/session ID | C3 `PC` | R1 | Direct client upload; lifecycle plus explicit abort. Never readable to users. |
| Raw/source asset registry | `media_assets`: object ref, checksum, bytes, type, owner, state, deadline | C3 `PC` | Registry metadata R4/minimized; content R2 | Registry remains long enough to prove deletion; object purged within 24 hours after terminal state by default. |
| Raw Media object | logical prefix `ingestion/quarantine-or-source/{source_id}/...` | C3 `PC` | R2 | Not account-export default after processing; explicit source download, if ever supported, requires a product/security decision. |
| Retention/cleanup task | target object ID, deadline, attempt, completion evidence | C1/C2 `OPS` | R9 bounded audit period | Must prove deletion without copying object content or locator secrets. |
| Provider metadata/receipt | provider IDs, attribution, safe normalized acquisition receipt | C2 `RIGHTS`,`OPS` | R9 | Store minimum needed for compliance/support; raw provider payload prohibited by default. |
| Source Tombstone | opaque Source/owner ref, deleted time/version | C1/C2 `OPS` | At least R7 plus safety window | Content-free; prevents restored/delayed source recreation. |

**Prohibited:** full Transcript, Evidence Items, Note document, payment state, model prompts, permanent signed/presigned URLs.

### 7.6 Workflow Service

| Data group | Likely physical records | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Generation | `generations`: immutable attempt ID, Artifact/Source refs, state, lineage | C2 `PD`,`PC`,`OPS` | R4 then Artifact deletion policy | Export safe attempt history if product promises it; purge with Artifact except minimized deletion/audit records. |
| Generation Settings snapshot | depth, language, audience/focus and approved bounded options | C2 `PC` | R4 with Generation | Never contain full user content or prompt text. User-exportable as settings metadata. |
| Job | user-facing async status, progress, stage, safe error code | C2 `OPS` | R4 with Generation | Workflow is authoritative; no raw provider exception/content. |
| Ordered Job Events | `job_events`: per-Job sequence, state/stage/progress/message code/time | C2 `OPS` | R4 with Generation or approved shorter UI history | SSE replay source. Domain Events and logs are not substitutes. |
| Workflow binding | Temporal Workflow ID/run ID, contract versions, start state | C1/C2 `OPS` | R4 plus replay/operations need | Internal only; used for replay/support and idempotent start. |
| Temporal history | IDs, small state, Activity requests/results containing bounded metadata | Highest contained class; target C2 | Temporal retention approved per Workflow before activation | No media, Transcript/Note bodies, prompts, secrets, or presigned URLs. Workflow owns semantics even when Temporal hosts storage. |
| Retry/cancellation lineage | prior/new Generation refs, reason/state, requester | C2 `PD`,`OPS` | R4 | Included in safe support/user history as approved. |
| Compensation state | completed steps, compensation keys, result/safe failure | C1/C2 `OPS`,`FIN` where usage-related | R4 or R9 for billing references | Opaque reservation/asset refs only; owners hold details. |
| Deletion Saga ledger | request scope, required owners, acknowledgements, retry/deadline/status | C2 `PD`,`OPS` | R8/R9; at least R7 plus safety window | Workflow owns global proof; each Service owns its local purge receipt/Tombstone. No deleted content. |
| Generation/deletion Tombstone | opaque IDs, terminal/deleted version/time | C1/C2 `OPS` | At least R7 plus safety window | Prevents replay/restoration from recreating active state. |

**Prohibited:** raw media, transcript/note/evidence bodies, provider prompts/responses, card/payment details, device/share/access tokens.

### 7.7 Media Service

| Data group | Likely physical records/objects | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Media execution | `media_executions`: Source/Generation ref, tool/config version, state/timing | C2 `PC`,`OPS` | R4 or approved shorter diagnostic period | Safe metadata may be support-visible; no filenames/paths with user secrets in logs. |
| Scan result | scanner/version, verdict, safe signature/reason code | C2/C3 `PC`,`SEC`,`OPS` | R9 bounded security/diagnostic period | Never expose malware details that enable abuse; raw sample not retained after source policy. |
| Probe result | container/codec/duration/stream metadata | C2 `PC`,`OPS` | R4 | May be supplied as safe Source metadata; no content frames. |
| Derived asset registry | `derived_assets`: type, checksum, object ref, state, deadline | C3 `PC` | Registry minimized after R2 content deletion | Media owns derivative IDs/objects; Ingestion retains only opaque references if needed. |
| Extracted audio/frame object | logical prefix `media/derived/{generation_or_asset_id}/...` | C3 `PC` | R2 | Delete within 24 hours after terminal Generation; never user/public readable by default. |
| Cleanup result/Tombstone | derivative ID, deleted time, result | C1/C2 `OPS` | At least R7 plus safety window for Tombstone | Content-free, auditable, idempotent. |

**Prohibited:** upload session, rights decision, Transcript text, Note/evidence content, permanent source credential, user billing data.

### 7.8 Transcription Service

| Data group | Likely physical records/objects | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Transcript Version | `transcript_versions`: immutable ID, Source/Artifact/Generation refs, language, state, hash | C3 `PC` | R4 then R5/purge with Artifact | User-readable/exportable where rights permit; new correction creates a version. |
| Transcript Segments | `transcript_segments`: ordinal, text, start/end, confidence | C3 `PC` | R4 then R5 | Authorized paginated access only. Never events/logs/analytics. |
| Speaker metadata | stable label and optional provider confidence; no verified identity claim | C3 `PC`,`PD` where identifiable | R4 then R5 | Export with transcript where allowed; do not infer real identity. |
| Word timing | compressed timing object/reference | C3 `PC` | R4 then R5 | Logical prefix `transcription/timing/{transcript_version_id}/...`; registered and purged by Transcription. |
| Provider provenance | provider/model/config, caption source, request ID, processing times | C2 `PC`,`OPS` | R4 or R9 for provider/audit requirement | Raw provider payload not domain data; content-free metadata only. |
| Transcript quality result | accuracy/timing/language measures, warnings, evaluation version | C2/C3 `PC`,`OPS` | R4 | No Transcript excerpts in general telemetry. |
| Authorized immutable snapshot | normalized bounded snapshot/object used by AI Notes/Search | C3 `PC` | R4 or shorter; source version-pinned | Transcription owns the object; consumer access is narrow and revocable. Consumer copies follow section 9. |
| Transcript Tombstone | opaque version/Artifact refs, deleted time/hash | C1/C2 `OPS` | At least R7 plus safety window | Prevents re-index/reconciliation resurrection. |

**Prohibited:** Raw Media ownership, Evidence Items, Note documents, search ranking, public share authorization, provider access token.

### 7.9 AI Notes Service

| Data group | Likely physical records/objects | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| AI generation/model-call record | Generation/Transcript refs, task, provider/model/config, state/timing/token counts | C2 `PC`,`OPS` | R4 for reproducibility; content-free cost records may follow R9 | No full prompt/response in logs. User export need not expose internal provider mechanics unless policy requires. |
| Semantic chunk | ordered Transcript segment/time refs and bounded text | C3 `PC` | R3 unless retained as part of approved evidence provenance | Rebuildable from Transcript while it exists; purge with Artifact at latest. |
| Evidence Item | immutable typed evidence, exact segment/time refs, confidence/hash | C3 `PC` | R4 then R5/purge with Artifact | AI Notes is authoritative. May be included in portability export as Citation provenance, not general analytics. |
| Evidence manifest | versioned Evidence Item set, Transcript Version ref, hashes | C3 `PC` | R4 with dependent Note Versions | Notes receives an immutable reference/minimum validated fields, not write access. |
| Outline/draft/candidate manifest | generated candidate document and provenance before materialization | C3 `PC` | R3; durable Note belongs to Notes after materialization | Delete sooner after successful materialization where practical. |
| Quality report | grounding/coverage/citation/schema results, defect codes, evaluator versions | C2/C3 `PC`,`OPS` | R4 with Generation | Store structured results; avoid unnecessary content excerpts. |
| Prompt/config definition | versioned system/task template and policy configuration | C1; C3 only if a runtime instance contains content | Release/config lifetime | Static templates belong in repository/config registry; runtime prompts containing content are R0/R3 and never general logs. |
| Provider request/response transient | exact sent/returned payload | C3 `PC` | R0 by default; diagnostic persistence requires explicit approval and no more than R3 | Provider retention/training disabled where supported/contracted. Never use private data for shared training by default. |
| Provider cost record | provider units/currency/cost, Activity/Generation reference | C2 `FIN`,`OPS` | R9 | AI Notes owns provider cost; Billing owns user usage/charge ledger. No content. |
| AI intermediate object | logical prefix `ai-notes/intermediate/{generation_id}/...` | C3 `PC` | R3 | Registered with deadline; no public/presigned ref in events. |
| AI Notes Tombstone | opaque Generation/Artifact/evidence refs, deletion time | C1/C2 `OPS` | At least R7 plus safety window | Blocks delayed materialization/rebuild. |

**Prohibited:** canonical editable Note, Artifact current reference, Transcript mutation, Entitlement/usage charge, external-enrichment data in MVP.

### 7.10 Notes Service

| Data group | Likely physical records/objects | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Note identity/aggregate | Note ID, Artifact/owner/current Working Version refs | C2/C3 `PD`,`PC` | R4 then R5 | User-readable/exportable; purge with Artifact/Account. |
| Working Version and Revision | current validated document JSON, revision, update metadata | C3 `PC` | R4 then R5 | Autosave authority. Never overwritten on stale Revision; never log document patches. |
| Note Version | `note_versions`: immutable/recoverable document, schema, hash, origin, parent, Generation/Transcript refs | C3 `PC` | R4 then R5 | User-readable/exportable/restorable. Regeneration creates a new version rather than destroying user work. |
| Note checkpoint | recovery snapshot and revision | C3 `PC` | Bounded rolling policy approved before M6; never beyond R4/R5 | Used for recovery, not normal version history. Exact checkpoint count/window is an activation decision. |
| Citation | Note/block relation to Transcript Version, segment/time bounds, Evidence Item/hash, state | C3 `PC` | R4 with Note Version | Notes owns relation/staleness; Transcription/AI Notes own referenced source/evidence. Exportable with Note provenance. |
| Provenance/staleness metadata | block/span origin and Citation validation state | C2/C3 `PC` | R4 with document | Presentation metadata, not proof/security boundary. |
| Feedback | rating/category, target IDs, optional user text/excerpt, consent/use state | C2; C3 when free text/content `PD`,`PC` | R9 | User-exportable/deletable subject to approved research/support policy. Never model training without explicit approved consent. |
| Share link | share ID, exact scope/version, state, created/revoked/expiry metadata | C2/C3 `PC`,`SEC` | Until revoke/expiry/Artifact deletion plus minimal anti-replay record | Public access is read-only and scope-bound. Revoke immediately on trash/delete. |
| Share-token hash | keyed hash, key version, lookup metadata | C3 `SEC` | Same as share plus minimal anti-replay policy | Raw token exists only in URL/request and is R0. Never logs, events, analytics, or exports. |
| Flattened Note text | local derived read/search-helper column | C3 `PC` | Same as owning Note Version | Notes may store for its own reads; Search owns its separate index copy. |
| Immutable Note snapshot object | logical prefix `notes/snapshots/{note_version_id}/...` when SQL JSON is insufficient | C3 `PC` | R4 then R5 | Notes-owned, content-hashed, authorized to Search/Export narrowly. |
| Note/share Tombstone | opaque IDs, deleted/revoked version/time | C1/C2 `OPS`,`SEC` | At least R7 plus safety window | Content-free; prevents delayed re-share/re-index. |

**Prohibited:** Evidence Item authority, Transcript text authority, search index, rendered export, Raw Media, payment state.

### 7.11 Search Service

| Data group | Likely physical records | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Artifact search document | owner scope, Artifact ID/version, approved title/source metadata, tokens/vector if approved | C2/C3 `PD`,`PC` | While Artifact active; remove from active index immediately on trash/delete | Derived/rebuildable; not separately user-exported because source owners provide canonical data. |
| Note search document | Note Version/hash, flattened approved text/headings/snippets | C3 `PC` | While indexed current/approved version exists | Fetch through authorized snapshot, never event body. Delete/rebuild on version/change policy. |
| Transcript search document | Transcript Version/hash, permitted segment text/time refs | C3 `PC` | While permitted Artifact/transcript exists | Rights/share scope never inferred. Remove immediately on trash/delete or display restriction. |
| Event/index checkpoint | producer/aggregate versions, event IDs, processed state | C1/C2 `OPS` | R10 | Supports idempotency/version-gap detection, contains no content. |
| Rebuild/reconciliation state | owner cursor/snapshot version, progress/failure | C1/C2 `OPS` | Current rebuild plus bounded history | No signed URLs or content in logs. |
| Search Tombstone | opaque owner/resource ID/version/deleted time | C1/C2 `OPS` | At least R7 plus safety window | Applied before delayed upserts; content-free. |

Raw user search queries are **not persisted by default**. A future search-quality dataset requires a separate purpose, consent/minimization review, class, retention, owner, and update to this catalog.

### 7.12 Export Service

| Data group | Likely physical records/objects | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Export request | Export ID, owner, Artifact/Note Version, format, state, idempotency ref | C2 `PD`,`PC`,`OPS` | Output R6; metadata uses a bounded support window approved before M6 | User-visible status; purged with Artifact/Account. No embedded Note body in request row. |
| Render attempt | renderer/version, attempt/timing, safe failure code | C1/C2 `OPS` | R9 bounded operations period | No rendered HTML/Note content in logs or exceptions. |
| Export output registry | object ref, checksum, bytes, format, creation/expiry | C3 `PC` | R6 | Export owns registry/object. Regenerable from Note Version while it exists. |
| Markdown/PDF object | logical prefix `export/{account_id}/{export_id}/...` | C3 `PC` | Delete after 24 hours | Private/encrypted. No public ACL. Artifact trash/delete revokes early. |
| Download capability | generated presigned URL | C3 `SEC` | R0; expires within 15 minutes | Generated on authorized request; raw URL never persisted, logged, or emitted. |
| Expiry/purge evidence | output ID, deadline, deleted time/result | C1/C2 `OPS` | R9 bounded audit period | Content-free. |

**Prohibited:** canonical Note, user edits, permanent output URLs, external active content, Transcript/source access beyond approved export scope.

### 7.13 Billing Service

| Data group | Likely physical records | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Plan/price mapping | internal plan/price IDs, limits, currency/provider refs | C1/C2 `FIN` | Commercial/version lifetime | Product configuration; no card data. Public display is a separate approved projection. |
| Billing customer reference | Account ID, Stripe customer ID, state | C2/C3 `PD`,`FIN` | R9 | Included in account export only as appropriate; deletion/anonymization follows statutory policy. |
| Subscription | provider subscription/item refs, plan, status, periods/cancel state | C2 `PD`,`FIN` | R9 | Billing authoritative product state; Stripe remains payment provider. |
| Entitlement/allowance | Account, capability limits, current allowance/state/version | C2 `FIN` | R9 | User-readable/exportable. Other Services use API/safe facts, never copy the ledger. |
| Usage estimate | estimated unit/cost inputs and result | C2 `FIN`,`OPS` | Short product/support window defined before M7 | Not settlement/source of charge. Avoid Source/Note content. |
| Usage reservation | unique business key, Account/Generation, units, state, expiry | C2 `FIN`,`OPS` | R9 | Idempotent; Workflow stores only opaque reservation ref/state needed for Saga. |
| Usage ledger | append-only reserve/settle/release/correction entries | C2 `PD`,`FIN` | R9 statutory/financial policy | User-summary/exportable as policy requires. Corrections are compensating entries, never destructive edits. |
| Webhook receipt | provider event ID/type/time/signature-validation result and minimum normalized fields | C2/C3 `FIN`,`SEC`,`OPS` | R9 | Deduplicate by provider event ID. Full raw payload stored only if specifically justified; never log signature/secret. |
| Checkout/portal session reference | provider session ID/state/expiry | C3 `FIN`,`SEC` | R0 or shortest provider session/support window | Hosted URL is a credential and not persisted/logged. |
| Billing Tombstone/anonymized link | de-identified Account/provider relation where legally required | C2 `FIN`,`OPS` | R9 | Must not allow normal product access or Account resurrection. |

**Prohibited:** card/bank details, CVV, raw payment credentials, media/transcript/note content, model-provider cost authority. AI/Transcription/Media Services own provider-cost records; Billing owns user-facing usage/charge accounting.

### 7.14 Notification Service

| Data group | Likely physical records | Class/tags | Retention | Deletion, export, and logging |
|---|---|---|---|---|
| Delivery preference | Account, event/channel, opt-in/out, locale/quiet settings | C2 `PD` | R4 | User-readable/exportable/editable; product defaults remain Account-owned. |
| Email delivery-destination snapshot | encrypted verified address plus Auth Identity/source version and purpose | C3 `PD` | While enabled/current; refresh on Auth change and purge on disable/Account deletion | Auth owns the Login Identifier; Notification is a purpose-bound custodian for product notifications. Auth security email does not use this copy. |
| In-app notification | Account, source fact ID/type, safe template data, read/state/time | C2 `PD`,`OPS` | R9 bounded user-visible history approved before M6 | No Note/Transcript body; delete with Account and source resource where appropriate. |
| Device-token record | Account/device/platform, encrypted token, state/last-seen | C3 `PD`,`SEC` | Until revoke, invalidation, sign-out/device removal, or Account deletion | Never logs/events/export in raw form. Provider invalid-token result triggers purge. |
| Delivery attempt | notification/channel/provider ref, state, timing, safe error | C2/C3 `PD`,`OPS` | R9 bounded delivery/support period | Address/token values excluded from logs. |
| Provider-message reference | provider message ID and status | C2 `PD`,`OPS` | Same as delivery attempt | Not authoritative for source business outcome. |
| Rendered message | minimal channel-specific body | C2; C3 if an approved private excerpt exists | R0 by default; provider retention governed contractually | Initial templates must use safe metadata only, not Note/Transcript content. |
| Processed source event | consumer/event/business key | C1/C2 `OPS` | R10 | Deduplicates intended delivery. No event content copy beyond approved safe template fields. |
| Notification Tombstone | opaque Account/resource ref and deleted time | C1/C2 `OPS` | At least R7 plus safety window | Prevents delayed event delivery after deletion. |

**Prohibited:** canonical Job state, full Note/Transcript content, Account profile authority, marketing campaign data without separate scope/consent, raw provider secrets.

---

## 8. Shared local operational records

These record types occur in several databases. Each physical row is owned by the Service whose database contains it; there is no shared global table.

| Record type | Owner | Classification | Retention/lifecycle |
|---|---|---|---|
| Transactional outbox row | Producing Service | C1/C2 `OPS`; event payload must exclude C3 bodies/capabilities | R10 after successful publication/replay window; domain transaction and row commit atomically |
| Processed-event/inbox row | Consuming Service | C1/C2 `OPS` | R10; unique consumer/event or business key; consumer state commits before acknowledgement |
| Command idempotency row | Command-owning Service | C2 `PD`,`OPS` | R10; stores Actor, operation, key, normalized request hash and response reference—not raw private request body |
| Activity stage key/result | Activity-owning Service | Class of bounded result, target C1/C2 | R10 or resource lifetime where result is canonical; prevents replay side effects |
| Local purge receipt | Data-owning Service | C1/C2 `OPS` | At least R7 plus safety window; target IDs and result only |
| Local Tombstone | Data-owning Service | C1/C2 `OPS` | At least R7 plus safety window; content-free |
| Audit record | Service performing sensitive action | C1/C2 `OPS`; never content bodies/secrets | R8/R9 under approved purpose and access |
| Schema migration history | Owning Service | C1 `OPS` | Service/repository lifetime; never edited by another Service |
| Support note/override | Service whose state is affected | C2/C3 `PD`,`OPS` | R9; structured, purpose-bound, Actor-audited; no casual content copying |

Outbox, processed-event, audit, and Tombstone tables may use common libraries, but the library does not own their rows or migrations.

---

## 9. Permitted cross-Service references and copies

| Source datum and authority | Permitted consumer copy | Consumer/custodian | Required controls |
|---|---|---|---|
| Auth Identity/session claims — Auth | Opaque `identity_id`, `account_id`, scopes, session/security version in short-lived access token/Actor context | Edge and resource Services | Validate Auth signature/issuer/audience/expiry; claims are authentication context, not resource authorization; no Login Identifier/credential |
| Verified Login Identifier — Auth | Purpose-bound encrypted email delivery-destination Snapshot and source version | Notification | Private authorized API, minimum retention, no event/log; Auth security messages remain Auth-owned |
| Account ID/status — Account | Opaque `account_id`, minimal status/version | All resource-owning Services | Validate Actor/ownership through approved claims/API/facts; no copied profile unless required |
| Artifact ID/ownership — Library | Opaque `artifact_id`, owner scope/version | Workflow, Ingestion, Transcription, AI Notes, Notes, Search, Export, Billing, Notification | Consumer authorizes independently; possession of ID is not authority |
| Source ID/asset ref — Ingestion | Immutable asset ID, checksum/type, narrowly scoped object access | Media, Transcription, Workflow | No raw locator/credential; access expires and is workload-scoped |
| Generation ID/state — Workflow | Opaque ID, safe state/version | Library, AI Notes, Notes, Billing, Notification | Workflow remains state authority; consumers store only needed fact |
| Derived asset ref — Media | Immutable audio/frame ID/hash, narrow object access | Transcription/AI Notes if approved | Media owns lifecycle; no S3 key/signed URL in events |
| Transcript Version/segments — Transcription | Version ID, segment IDs/time bounds/hash; private snapshot when necessary | AI Notes, Notes, Search | Snapshot authorized, version-pinned, C3, and deletion-linked; event carries reference only |
| Evidence Item/manifest — AI Notes | Evidence ID/type/hash, exact Transcript refs, validated manifest reference | Notes | Notes owns Citation relation, not Evidence Item meaning; no mutable shared record |
| Note Version — Notes | ID/hash/schema plus authorized immutable snapshot | Library reference, Search index, Export render | Library stores reference only; Search/Export own derived copies and delete on source fact |
| Entitlement/reservation — Billing | Safe Entitlement version or opaque reservation ID/state | Edge/Workflow | No copied ledger/payment data; stale Entitlement policy must be explicit |
| Source business fact — any producer | Safe event ID/type/version/reference | Search/Notification/Library projections | Transactional outbox, idempotency, version-gap recovery, no C3 content |

### 9.1 Copy rules

- A copy inherits the source classification unless it is demonstrably less sensitive after irreversible minimization.
- Copying into a new format does not make content public or anonymous.
- The consumer records source version/hash so stale copies are detectable.
- Owners expose snapshot APIs/objects that return only the approved consumer view.
- A deletion or access-revocation fact takes precedence over delayed create/update facts.
- If a copy cannot be reliably found and deleted, it is not permitted.

---

## 10. Object-storage ownership

Logical prefixes are illustrative namespaces; Infrastructure selects physical buckets/accounts. IAM, encryption keys, lifecycle rules, and network controls must preserve these ownership boundaries.

| Logical prefix | Owning Service | Content | Class | Retention |
|---|---|---|---|---|
| `ingestion/quarantine/{upload_or_source_id}/` | Ingestion | Multipart parts, untrusted Raw Media | C3 `PC` | R1/rejected within 24 hours |
| `ingestion/source/{source_id}/` | Ingestion | Accepted Raw Media/source object | C3 `PC` | R2 |
| `media/derived/{asset_or_generation_id}/` | Media | Normalized audio, sampled frames, temporary derivatives | C3 `PC` | R2 |
| `transcription/timing/{transcript_version_id}/` | Transcription | Compressed word timing | C3 `PC` | R4/R5 |
| `transcription/snapshots/{transcript_version_id}/` | Transcription | Approved immutable normalized snapshot, if needed | C3 `PC` | R4/R5 or shorter |
| `ai-notes/intermediate/{generation_id}/` | AI Notes | Chunks, candidate/draft/evidence intermediates | C3 `PC` | R3 unless manifest is durable R4 evidence |
| `notes/snapshots/{note_version_id}/` | Notes | Immutable document snapshot, if used | C3 `PC` | R4/R5 |
| `export/{account_id}/{export_id}/` | Export | Markdown/PDF output | C3 `PC` | R6 |

Rules:

- No generic `shared/`, `temp/`, or cross-Service writable prefix is allowed.
- An object key is internal metadata and never user authorization.
- Bucket/prefix access uses workload identity; presigned access is operation-, object-, and time-scoped.
- Every object is registered in the owner's database before or atomically around becoming usable.
- Orphan detection compares objects to owner registry and quarantines/deletes safely; it does not guess ownership from names alone.

---

## 11. Deletion, recovery, and restoration

### 11.1 Source-media deletion

Deleting Raw/Derived Media after processing does not automatically delete the Artifact, Transcript, or Note. The user-facing policy must distinguish:

- **Delete source media:** Ingestion/Media remove source objects and derivatives; permitted Transcript/Note outputs remain, and regeneration may require re-upload.
- **Delete Artifact:** Hide immediately, revoke access, then purge all Artifact-owned and derived Service data after the 30-day recovery window.
- **Delete Account:** Auth immediately revokes Authentication Sessions; Workflow executes the approved Account Deletion Saga across every Account-owned Artifact and Service; Auth and Account then purge/anonymize their own data under policy.

### 11.2 Artifact deletion sequence

1. Library authorizes and records trash/deleting state; normal reads and Search visibility stop immediately.
2. Workflow cancels active Generations and records the Deletion Saga scope.
3. Notes revokes share capabilities; Export revokes downloads; Ingestion revokes playback/source access.
4. During the 30-day recovery window, only the data required for an explained restore remains. Raw Media/temporary objects do not receive an automatic retention extension.
5. At purge, Workflow invokes idempotent owner Activities for Library, Ingestion, Media, Transcription, AI Notes, Notes, Search, Export, Billing where applicable, and Notification.
6. Each Service deletes its database rows/objects/caches, records a local content-free Tombstone and purge receipt, then acknowledges Workflow.
7. Workflow marks the Saga complete only when every required owner has acknowledged. Missing acknowledgements retry and alert.

### 11.3 Restore

- Restore during the trash window creates an explicit lifecycle transition; it does not remove historical audit/Tombstone safety records prematurely.
- Data already removed under shorter retention—especially Raw Media, derivatives, exports, or AI temporary data—is not promised to return.
- Rebuildable Search/Notification projections reconcile from owners.
- A restored database or object inventory is isolated until it consumes the current deletion ledger/Tombstones and proves deleted resources remain unavailable.

### 11.4 Backups

- Each Service owns backup/restore validation for its logical database; Infrastructure operates the facility.
- Point-in-time database backups expire after 35 days under the baseline.
- S3 versioning/backups must not silently defeat object retention; version expiration/deletion behavior is explicitly tested.
- Backup access is break-glass, least privilege, audited, and never a normal support/content access path.
- Restore rehearsals verify authorization, cross-Service references, and deletion reconciliation—not only database availability.

---

## 12. External and managed-system boundaries

| External/platform system | External authority | Product-owned data | Owning product boundary | Rules |
|---|---|---|---|---|
| Transactional email provider | Message delivery status only; not identity authority | Verification/recovery/security challenge and outcome | Auth | Provider receives minimum address/template/link; token purpose/validity remains Auth-owned; retention/region/abuse controls reviewed |
| Stripe | Payment instrument and processor records | Subscription/Entitlement/usage ledger and minimum provider references | Billing | Hosted payment surfaces; no card data; signed idempotent webhooks |
| YouTube/hosted provider | Provider content and access policy | User's Source reference, metadata, attestation, approved acquisition receipt | Ingestion | Public availability is not permission; no scraping/bypass |
| ASR/caption provider | Transient provider processing/response under contract | Normalized Transcript Version and provenance | Transcription | Disable provider training/retention where contracted; persist normalized domain model, not raw provider object |
| Model provider | Transient model processing under contract | Evidence, candidate/quality result, model/cost metadata | AI Notes | Minimum context; no shared training by default; prompt injection isolation |
| Temporal Cloud | Hosted Workflow history/execution | Workflow definitions, business state projection, data rules | Workflow | Bounded payloads; Workflow owns retention/schema semantics |
| EventBridge/SQS | Transient delivery/queue state | Domain fact and local producer/consumer records | Producer owns fact; consumer owns its processed/projection state | Transport is not source of truth; DLQs contain safe payloads only |
| S3/RDS/cache | Hosted bytes/state | All registered Service data | Each named Service | Infrastructure operates; IAM/database roles prevent cross-Service access |
| Logs/metrics/traces provider | Hosted telemetry | Minimized telemetry emitted by each repository | Emitting Service/application | No C3 bodies/capabilities; Infrastructure operates retention/access controls |

Provider retention outside the product must be documented alongside product retention. Deleting the product copy is not a complete privacy claim if a processor retains another copy.

---

## 13. Client-held data

Clients are not authoritative for server resources, but they may hold local state. Each application repository is custodian for the data it persists on the device.

| Client/custodian | Permitted local data | Class | Retention/deletion |
|---|---|---|---|
| `study-notes-creator-web` | Opaque BFF session cookie, server-side first-party Auth token/session material, upload resume metadata, bounded unsaved Note recovery draft, non-sensitive UI preferences | Session C3 `SEC`; drafts C3 `PC` | Browser receives only HttpOnly/SameSite cookie; server-side refresh material is encrypted/revocable; drafts clear after confirmed sync, sign-out/account deletion, or bounded expiry; never token in browser storage |
| `study-notes-creator-desktop` | OS-secure auth material, upload state, bounded local cache/draft, user-selected file handle/path | C2/C3 `PC`,`SEC` | OS secure storage for secrets; explicit cache controls; purge on sign-out/account deletion; filesystem paths not telemetry |
| `study-notes-creator-mobile` | OS-secure auth material, background-upload state, bounded cache/draft, device notification registration state | C2/C3 `PD`,`PC`,`SEC` | OS secure storage; platform backup exclusions for secrets/private cache; purge on sign-out/account deletion |

Client caches do not change Service ownership. Conflict resolution reconciles against the owning Service Revision/version while preserving unsynced user work.

---

## 14. Telemetry, analytics, and support access

### 14.1 Operational telemetry

The emitting Service/application owns the semantic content and minimization of each log, metric, and trace. Infrastructure owns collection reliability, sink access, and lifecycle configuration.

Allowed examples:

- Opaque request/trace/Service/resource IDs where necessary.
- Safe state/error codes, duration, sizes/buckets, retry count, provider/model identifier, and cost units.
- Aggregate performance, queue depth, failure, availability, and quality measures.

Prohibited examples:

- Raw Transcript/Note/Evidence/prompt/model response or media frames.
- Source locators with query parameters, object keys where avoidable, presigned/share/checkout URLs.
- Access/refresh/session/device tokens, webhook signatures, secrets, cookies.
- User email/display name, free-text feedback, search query, document patch, or provider raw exception containing content.

### 14.2 Product analytics

Product Events contain approved event names, opaque IDs, categorical settings, and minimized numeric measures. They do not become a second domain database.

- The source repository owns event correctness and classification.
- Product governance owns the measurement purpose/taxonomy.
- Infrastructure operates the approved analytics processor/sink.
- User-level durable analytics is not enabled until consent/legal basis, retention, deletion, access, and processor terms are approved.
- Raw search queries, Source titles/URLs, Transcript/Note text, feedback text, and share tokens are prohibited.
- A future warehouse/table must name a data owner and be added here before creation; “analytics” is not an ownership exemption.

### 14.3 Support access

- Support sees safe Account/Artifact/Job metadata by default, not Private Content.
- Content access requires a documented purpose, least privilege, user authorization or approved exceptional process, time-bound access, and audit.
- Support notes are structured and Service-owned; screenshots or copied private text are not a substitute for proper tooling.
- Impersonation, data correction, deletion override, and billing correction require separate audited controls.

---

## 15. Data portability and user export

An account-data export does not create a new source of truth or transfer ownership to Export Service.

Each authoritative Service is responsible for an authorized, versioned export fragment of the user's data that is in scope. A future feature specification must select the durable coordinator/packager before implementing PRD requirement ACC-05; it must not make Edge perform an unbounded synchronous fan-out.

| Service | Candidate user-portable data | Normally internal/non-portable data |
|---|---|---|
| Auth | Login Identifier, verification state, active-session/security history as policy allows | Password/refresh/challenge hashes, signing keys, risk/fraud internals |
| Account | Profile, preferences, consent decisions/history as allowed | Internal Auth binding mechanics and product security controls |
| Library | Artifact metadata/lifecycle and resource relationships | Projection repair state |
| Ingestion | Source metadata, rights attestation, retained source reference as permitted | Storage keys, signed URLs, security scan/acquisition internals |
| Workflow | Safe Generation/Job history and settings | Temporal history, compensation internals |
| Transcription | Transcript Versions/segments where rights permit | Raw provider payload/credentials |
| AI Notes | Evidence/provenance needed to interpret Citations | System prompts, provider secrets, internal evaluation corpus |
| Notes | Notes, versions, Citations, feedback, share metadata | Share-token hash/security controls |
| Billing | Subscription, Entitlement, and usage history as policy/law requires | Webhook signature data, fraud/security internals |
| Notification | Preferences and in-app notification history under policy | Raw device tokens/provider credentials |

Search documents, caches, rendered exports, duplicate snapshots, and operational telemetry are generally not exported separately when the authoritative data is included, subject to applicable requirements.

---

## 16. Schema, migration, and review rules

### 16.1 Required review for new data

Every schema proposal answers:

1. What exact product/operational purpose requires the datum?
2. Which Service is authoritative, and why is it within that mission?
3. Is it canonical data, a projection, a cache, an audit record, or a temporary intermediate?
4. What class/tags apply, and can the datum be minimized or avoided?
5. What retention profile/deadline applies?
6. How is it authorized, exported, deleted, tombstoned, backed up, and restored?
7. May it appear in an API, event, Workflow history, log, trace, metric, analytics event, support tool, or provider request?
8. Which consumers copy it, and how do they reconcile version gaps and deletion?
9. What migration/backfill/rollback risks exist?
10. Has user-facing disclosure or consent changed?

### 16.2 Migration rules

- Only the owning Service migration changes its database.
- Production migrations are forward-only and use expand → migrate/backfill → switch → contract.
- Backfills are resumable, observable, throttled, idempotent, and preserve classification/deletion state.
- Destructive changes require restore validation and owner approval.
- Cross-Service ownership transfer requires an ADR, dual-read/write avoidance plan, versioned export/import, reconciliation, cutover, rollback, and deletion of the former copy.
- A field is not repurposed to a different meaning. Add a new versioned field and migrate explicitly.

### 16.3 Pull-request evidence

A pull request that adds/changes durable data includes:

- Updated `SERVICE.md` data section and this catalog if a new group/boundary is introduced.
- Migration and rollback/repair plan.
- Classification/retention/deletion/export/logging decisions.
- Authorization and data-isolation tests.
- Deletion/Tombstone/restore tests.
- Contract/event compatibility and consumer impact.
- Expected volume, index/storage/cost, backup, and support implications.

---

## 17. Pre-activation unresolved decisions

`R9 Policy-bound` items are deliberately visible. They must be resolved before the relevant Service/data path activates:

| Decision | Required before |
|---|---|
| Auth identifier/password policy, Argon2id benchmark, challenge/session/token retention, MFA/passkeys, security audit, KMS/JWKS rotation, email processor, and deletion/anonymization | M2 Auth activation |
| Legal/privacy approval of classification labels, consent evidence, Account deletion/anonymization, and processor terms | M1/M2 production identity design |
| Source locator/rights-attestation retention and compliant YouTube/hosted acquisition evidence | M3 and any URL input enablement |
| Scan/probe diagnostic and security-evidence retention | M3 |
| Temporal history retention and payload codec/encryption policy | M2 |
| Note checkpoint count/window and feedback/research retention/consent | M6 internal alpha |
| Search index field list and deletion/freshness SLO | M6 |
| Export request/attempt metadata support window | M6 |
| Notification history, delivery-attempt, provider, and device-token policies | M6 |
| Billing/statutory ledger, webhook receipt, tax/currency, correction, and Account-deletion policy | M7 |
| Security/audit, logs/traces, product analytics retention and deletion behavior | Before production data enters each sink |
| Account-data portability coordinator, format, scope, and secure delivery | Before ACC-05 is released |

Until resolved, `R9` means **do not launch that durable path**, not “retain forever.”

---

## 18. Ownership-change procedure

Moving data between Services is a system architecture change. The proposal must include:

1. Evidence that the current owner no longer matches the business capability.
2. An ADR and updates to `SERVICE_CATALOG.md`, this file, contracts, event catalog, Roadmap, and both `SERVICE.md` files.
3. Source/target schema and classification mapping.
4. Backward-compatible export/import and verification using counts, hashes, and sampled invariants.
5. Cutover sequencing that avoids two authoritative writers.
6. Authorization, retention, deletion, Tombstone, backup/restore, and support-history migration.
7. Consumer/event/reference compatibility and rollback.
8. Proof that the former owner removed its canonical copy after the approved safety window.

Convenience, a single query, or code reuse is not sufficient reason to transfer ownership. Prefer an owner API, event-driven projection, or bounded composition first.

---

## Appendix A: Data-group proposal template

```markdown
# <Data group>

**Purpose:**
Why this must exist.

**Authoritative Service:**
Exactly one Service and rationale.

**Physical stores:**
Database tables/columns, object prefixes, caches, provider copies.

**Classification/tags:**
C0/C1/C2/C3 plus PD/PC/SEC/FIN/RIGHTS/OPS.

**Retention:**
Profile, exact trigger/deadline, unresolved approval if R9.

**Authorization:**
Actors/workloads and decision owner.

**Copies/consumers:**
API/event/snapshot, source version/hash, reconciliation behavior.

**Deletion/restoration:**
Soft delete, purge, Tombstone, backup restore behavior.

**User export:**
Included/excluded and rationale.

**Logging/analytics/support:**
Allowed metadata and prohibited content.

**Migration:**
Rollout, backfill, compatibility, verification, rollback/repair.
```

## Appendix B: Fast ownership test

Before adding a datum, ask:

- Which Service can truthfully say, “If my database is lost, this fact is lost”?
- Which Service enforces the invariant when two callers disagree?
- Which Service must answer a user deletion/export request for it?
- Is another Service storing the fact only to search, render, notify, or recover?
- Can the proposed copy be rebuilt and deleted reliably?

If two Services both appear authoritative, the boundary is unresolved. Stop and design the command, fact, reference, or projection explicitly before writing code.

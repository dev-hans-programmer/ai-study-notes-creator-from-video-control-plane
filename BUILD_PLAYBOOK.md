# Study Notes Creator: AI-Assisted Build Playbook

**Document status:** Active operating policy  
**Version:** 1.0  
**Last updated:** 23 November 2026  
**Applies to:** Every repository listed in [repos.md](./repos.md)  
**Product requirements:** [PRD.md](./PRD.md)  
**Technical baseline:** [TRD.md](./TRD.md)

---

## 1. Purpose

This playbook defines how Study Notes Creator will be built in a controlled, understandable, and repeatable way when AI is the primary implementation layer.

The operating model is:

> The human owns product intent, architecture decisions, contracts, risk acceptance, and releases. AI analyzes, explains, implements, tests, documents, and prepares evidence within approved boundaries. Automation enforces the rules.

The objective is not to generate the entire platform quickly. The objective is to produce small, trustworthy increments while preserving a complete understanding of:

1. Why each component exists.
2. What it owns.
3. What data it stores.
4. How it communicates.
5. How it fails and recovers.
6. How it is tested, deployed, observed, and rolled back.

No component is considered owned merely because its code exists. It is owned when its purpose and behavior are documented, understood, verified, and operable.

---

## 2. Authority and document hierarchy

When instructions conflict, use this precedence:

1. Applicable law, provider terms, security, and privacy requirements.
2. Approved product decisions in `PRD.md`.
3. Approved technical decisions in `TRD.md` and Architecture Decision Records.
4. This build playbook.
5. Approved feature and service specifications.
6. Repository-level `AGENTS.md` and `README.md`.
7. The current task specification.
8. Existing implementation details.

Existing code is not automatically correct. If code conflicts with a higher-authority document, stop and report the conflict rather than silently choosing one.

The governance repository is the source of truth for cross-system decisions. A service repository is the source of truth for that service’s implementation and owned contracts.

---

## 3. Roles and responsibilities

### 3.1 Product owner / system owner

The human owner is responsible for:

- Product priority and scope.
- Approval of service boundaries and data ownership.
- Approval of public APIs, domain events, security-sensitive flows, and breaking changes.
- Selecting tradeoffs when multiple valid options exist.
- Accepting residual risks.
- Approving production releases and destructive operations.
- Confirming that a delivered feature satisfies the intended user outcome.

The owner does not need to write every line of code. The owner must be able to explain every important component at the level described in Section 1.

### 3.2 AI implementation agent

AI is responsible for:

- Reading authoritative context before acting.
- Identifying assumptions, dependencies, and affected contracts.
- Producing a reviewable plan before material implementation.
- Implementing only the approved scope.
- Creating tests, migrations, telemetry, documentation, and rollback support with the code.
- Running verification and reporting actual results.
- Explaining the final behavior in plain language.
- Stopping when a decision or authority is missing.

AI is not authorized to invent product scope, bypass service boundaries, weaken safeguards, merge its own work, or release to production without explicit authority.

### 3.3 AI review agent

Important work receives a separate review pass that did not author the implementation. The reviewer checks:

- Correctness and edge cases.
- Contract and architecture compliance.
- Security and privacy.
- Data migration and deletion behavior.
- Idempotency, retries, and distributed failure modes.
- Test quality rather than only test presence.
- Observability and operational readiness.

The implementation agent may fix review findings, but it must not dismiss them without written reasoning.

### 3.4 Automation

CI/CD enforces objective rules including:

- Formatting, linting, and static typing.
- Unit, contract, integration, security, migration, and build tests.
- API/event/document compatibility.
- Dependency, secret, license, container, and infrastructure scanning.
- Temporal workflow replay where applicable.
- Generated-code and documentation drift.
- Artifact provenance and versioning.

Automation is a gate, not a substitute for design or human review.

---

## 4. Non-negotiable build principles

1. **Build vertical slices.** Deliver a small user outcome across the necessary layers instead of completing an entire technical layer in isolation.
2. **Specify before implementing.** Code begins only after the work meets the Definition of Ready.
3. **Contracts before providers.** Cross-repository behavior is agreed before service implementations change.
4. **One owner per datum.** A service never reads or writes another service’s database.
5. **One primary repository per implementation task.** Cross-repository features are decomposed into ordered, compatible pull requests.
6. **Additive change first.** Consumers tolerate new behavior before old behavior is removed.
7. **No invisible decisions.** Material tradeoffs are recorded in an ADR or feature decision log.
8. **Tests and telemetry are part of implementation.** They are not follow-up tasks.
9. **No unverifiable claims.** AI reports commands and results it actually ran.
10. **No secret knowledge.** Important context is written in the repository rather than retained only in a conversation.
11. **Production is never the first integration environment.** Changes progress through local, preview/integration, staging, then production.
12. **Failure behavior is designed.** Timeouts, retries, idempotency, compensation, and rollback are specified before release.
13. **Do not activate all 24 repositories at once.** A repository is created when an approved vertical slice first needs its capability.

---

## 5. Planning hierarchy

Work is organized at four levels:

```text
Product outcome
└── Release milestone
    └── Vertical feature slice
        └── Repository task / pull request
```

### 5.1 Product outcome

A measurable user or business result, such as “a learner can turn an uploaded video into grounded notes.”

### 5.2 Release milestone

A coherent set of outcomes that can be operated safely, such as internal alpha or private beta.

### 5.3 Vertical feature slice

The smallest demonstrable end-to-end behavior that delivers or validates part of an outcome.

### 5.4 Repository task

A reviewable implementation unit in one primary repository. A task has explicit inputs, outputs, tests, and boundaries.

Work must not be tracked as vague items such as “build ingestion service” or “finish backend.”

---

## 6. Work states and gates

Every feature follows these states:

```text
Proposed → Investigating → Specified → Ready → Implementing
         → Reviewing → Integrated → Released → Observed → Done
```

| State | Meaning | Exit gate |
|---|---|---|
| Proposed | Desired outcome is captured | Owner approves discovery |
| Investigating | Unknowns, alternatives, and risks are examined | Decisions or experiments resolve blockers |
| Specified | User flow, system flow, failures, and contracts are documented | Specification review passes |
| Ready | Tasks, owners, dependencies, and acceptance tests are clear | Definition of Ready passes |
| Implementing | AI performs bounded repository work | Implementation and local verification complete |
| Reviewing | CI, AI review, and human review occur | Findings resolved and Definition of Done passes |
| Integrated | Compatible changes work together in integration/staging | End-to-end acceptance passes |
| Released | Approved artifact is deployed | Health and smoke tests pass |
| Observed | Real behavior is monitored for the defined period | No rollback trigger; metrics acceptable |
| Done | Outcome, code, documentation, and operations are complete | Owner accepts outcome |

Skipping a gate requires an explicit, time-bounded exception recorded by the owner.

---

## 7. Standard delivery workflow

### Step 1: State the user outcome

Start with a user-visible result, not a repository or technology:

> As a signed-in learner, I can upload a supported local video and see a durable processing job in my library.

Record success and non-goals.

### Step 2: Run read-only impact analysis

AI identifies:

- Affected repositories and services.
- Current owners of the relevant data.
- APIs, events, Activity contracts, packages, and schemas affected.
- Security, privacy, billing, retention, and observability implications.
- Unknowns requiring experiments or owner decisions.
- The smallest viable vertical slice.

No implementation begins during impact analysis.

### Step 3: Resolve decisions

For every material choice, AI presents:

- Context and constraints.
- Two or more viable options where they exist.
- Benefits, costs, failure modes, and reversibility.
- A recommendation.

The owner approves the decision. Architectural decisions are recorded as ADRs.

### Step 4: Specify behavior

Create or update the feature specification, service specifications, sequence diagram, failure matrix, and acceptance criteria.

### Step 5: Define contracts

Create additive API, event, Temporal Activity, document, or package contracts. Validate ownership and compatibility before provider code changes.

### Step 6: Decompose repository tasks

Order tasks so the platform remains compatible at each merge. A typical sequence is:

1. Contract/schema addition.
2. Consumer tolerance/support.
3. Provider implementation.
4. Integration test and feature exposure.
5. Observability and rollout.
6. Later removal of deprecated behavior.

### Step 7: Implement one bounded task

The AI implementation agent works in one primary repository, runs required checks, and produces a handoff.

### Step 8: Review independently

Run CI and a separate AI review. Resolve findings. The human owner reviews material decisions, contracts, migration behavior, and acceptance evidence.

### Step 9: Integrate end to end

Use published artifacts and service versions—not uncommitted local source links—to verify the full slice in an integration environment.

### Step 10: Release gradually

Promote the exact tested artifacts, enable through feature flags or canaries when appropriate, execute smoke tests, and observe rollback indicators.

### Step 11: Close knowledge

Update documentation, service catalogs, event catalogs, ADRs, runbooks, and known limitations. Record what was learned.

---

## 8. Definition of Ready

A repository task is ready only when all applicable items are true:

- [ ] User/business outcome is stated.
- [ ] In-scope and out-of-scope behavior is explicit.
- [ ] Primary repository and owning service are identified.
- [ ] Affected data owner is identified.
- [ ] API/event/Activity/package contracts are linked or declared unchanged.
- [ ] Acceptance criteria are observable and testable.
- [ ] Failure, retry, timeout, cancellation, and idempotency behavior is defined.
- [ ] Security, privacy, retention, and authorization impacts are considered.
- [ ] Migration and compatibility strategy is defined.
- [ ] Telemetry and operational requirements are defined.
- [ ] Dependencies and preceding pull requests are identified.
- [ ] No unresolved product or architecture decision blocks implementation.
- [ ] The owner can explain why the task is needed and how it fits the feature.

If the task is not ready, AI may help specify it but must not pretend implementation requirements are settled.

---

## 9. Definition of Done

Work is done only when all applicable items are true:

- [ ] Approved behavior is implemented without unrelated scope.
- [ ] Unit and integration tests cover the behavior and important failures.
- [ ] Contract and compatibility tests pass.
- [ ] Database migrations are safe, forward-compatible, and tested.
- [ ] Authorization and tenant isolation are tested.
- [ ] Idempotency, retries, timeouts, and compensation are tested.
- [ ] Logs, metrics, traces, and alerts are present and privacy-safe.
- [ ] No raw private content, token, secret, or signed URL enters logs.
- [ ] Security/dependency/container scans pass.
- [ ] Documentation and runbooks reflect the implementation.
- [ ] Deployment, feature-flag, and rollback procedures are documented.
- [ ] AI reviewer findings are resolved.
- [ ] CI passes from a clean environment.
- [ ] Integration/staging acceptance passes using published artifacts.
- [ ] The owner receives and understands the implementation handoff.
- [ ] The release is observed for its defined verification period.

“The code compiles” and “the happy path works” are not definitions of done.

---

## 10. Required product-level artifacts

Maintain these in the governance repository as they become necessary:

```text
study-notes-creator/
├── PRD.md
├── TRD.md
├── repos.md
├── BUILD_PLAYBOOK.md
├── ROADMAP.md
├── GLOSSARY.md
├── SERVICE_CATALOG.md
├── EVENT_CATALOG.md
├── DATA_OWNERSHIP.md
├── adr/
├── features/
│   └── F-###-feature-name/
├── releases/
└── templates/
```

Do not create empty documentation for appearance. Create each artifact before the first decision or feature that needs it, then maintain it.

---

## 11. Service design gate

Before production code begins in a new backend service repository, its `SERVICE.md` must be approved.

### Service specification template

```markdown
# <Service Name>

## Mission
One sentence describing the business capability owned.

## Non-goals
Responsibilities explicitly owned elsewhere.

## Owned data
Aggregates, tables, object prefixes, retention and classification.

## State machines
States, allowed transitions and terminal states.

## Public/internal APIs
Commands, reads, errors, idempotency and authorization.

## Events published
Event name/version, trigger, payload classification and consumers.

## Events consumed
Producer, handling, idempotency and version-gap recovery.

## Temporal Activities
Names, request/response schemas, timeouts, heartbeats and compensation.

## Dependencies
Synchronous dependencies, external providers and failure policy.

## Security and privacy
Threats, actor rules, secrets, content access and audit requirements.

## Reliability
SLOs, retry rules, circuit breakers, reconciliation and recovery.

## Observability
Logs, metrics, traces, dashboards and alerts.

## Testing
Unit, contract, integration, security, performance and failure tests.

## Deployment and rollback
Configuration, migrations, rollout, rollback and runbook.

## Open decisions
Unresolved items with owners and deadlines.
```

### Service ownership check

Before approval, the owner must be able to answer:

- Why is this a separate service?
- What data is authoritative here?
- What would break if it were unavailable?
- How do dependent services recover?
- Can it deploy without coordinating every other service?

If these answers are unclear, the service boundary is not ready.

---

## 12. Feature specification gate

Each cross-system feature receives a folder:

```text
features/F-001-local-video-to-notes/
├── brief.md
├── sequence.md
├── contracts.md
├── acceptance.md
├── failure-matrix.md
├── rollout.md
└── tasks.md
```

### Feature brief template

```markdown
# F-###: <Feature Name>

## Outcome
What becomes possible for the user?

## Success measure
How will we know this is useful and working?

## In scope
Explicit supported behavior.

## Out of scope
Explicitly deferred behavior.

## User flow
Numbered interaction flow.

## Services and repositories
Each participant and why it is involved.

## Data ownership
Every created/read/updated datum and its owner.

## System sequence
Commands, reads, events and asynchronous stages.

## Failure behavior
Validation, dependency, timeout, retry, cancellation and partial states.

## Security/privacy
Authorization, sensitive data, retention and abuse cases.

## Acceptance criteria
Observable end-to-end behavior.

## Rollout and rollback
Flags, compatibility order, monitoring and rollback triggers.

## Decisions and open questions
Links to ADRs and unresolved inputs.
```

The sequence must show service ownership and communication. It must not use vague boxes such as “backend” or “AI” when a real service owner is known.

---

## 13. Architecture Decision Records

Create an ADR when a choice:

- Changes a service boundary or data owner.
- Adds a new repository, runtime, database, queue, provider, or major dependency.
- Changes a public/internal contract strategy.
- Affects security, privacy, retention, compliance, cost, or reliability materially.
- Is difficult or expensive to reverse.
- Resolves a disagreement that future contributors may revisit.

### ADR template

```markdown
# ADR-###: <Decision>

**Status:** Proposed | Accepted | Superseded | Rejected
**Date:** YYYY-MM-DD
**Owner:** <name/role>

## Context
What problem and constraints require a decision?

## Options considered
Viable alternatives with benefits, costs and risks.

## Decision
What was chosen and why?

## Consequences
Positive, negative and operational consequences.

## Validation
Experiment, measurement or review needed to confirm the decision.

## Revisit trigger
What evidence would justify reconsideration?
```

AI may draft and recommend an ADR. Only the owner accepts it.

---

## 14. AI task contract

Every implementation request given to AI should use this structure.

```markdown
# Task: <Short name>

## Repository
One primary repository.

## Goal
One observable result.

## Authoritative context
Links to feature, service, contracts, ADRs and relevant code.

## In scope
Exact implementation responsibilities.

## Out of scope
Changes explicitly forbidden or deferred.

## Contracts
Contract versions to implement; state “no contract change” where applicable.

## Acceptance criteria
Testable behavior, including important failures.

## Required verification
Commands, test types, compatibility and security checks.

## Required deliverables
Code, tests, migrations, telemetry, documentation and handoff.

## Stop conditions
Decisions, conflicts or risks that require owner input.
```

### Example

```markdown
# Task: Create multipart upload session

## Repository
study-notes-creator-ingestion-service

## Goal
An authenticated user can create an idempotent, expiring multipart upload session.

## Authoritative context
- F-002 local upload specification
- Ingestion SERVICE.md
- uploads-v1 OpenAPI contract
- ADR for direct S3 multipart uploads

## In scope
- Persist upload session
- Generate scoped presigned part URLs
- Publish UploadSessionCreated
- Unit and integration tests

## Out of scope
- Media scanning
- Starting generation
- Web UI
- Contract changes

## Acceptance criteria
- Session belongs to authenticated user/artifact
- Idempotency replay returns the original session
- URL expiry is enforced
- Unauthorized artifact returns a stable error
- Signed URLs and source names are not logged
```

---

## 15. Repository-level AI instructions

Every implementation repository must contain `AGENTS.md`. It should be brief, concrete, and repository-specific.

Required contents:

```markdown
# Repository AI Instructions

## Mission and ownership
What this repository owns and does not own.

## Authoritative documents
Service/feature/contract/ADR locations.

## Architecture rules
Allowed layers, dependency direction and forbidden access.

## Data and security
Owned data, classifications, logging prohibitions and authorization rules.

## Commands
Setup, format, lint, type-check, test, contract, migration and build commands.

## Change rules
What requires an ADR, contract review, migration review or owner approval.

## Definition of done
Repository-specific gates.

## Stop conditions
Situations in which AI must ask rather than assume.
```

`AGENTS.md` must not duplicate hundreds of lines from this playbook. It links here and adds local rules.

---

## 16. AI implementation rules

AI must:

- Begin with a concise understanding of the task and affected boundaries.
- Inspect existing code, tests, contracts, migrations, and repository instructions.
- Preserve unrelated user changes.
- Prefer the smallest coherent change.
- Use existing patterns unless an approved decision changes them.
- Keep generated artifacts reproducible.
- Add negative and failure-path tests.
- Report assumptions and uncertainty.
- Run verification proportional to risk.
- Produce a self-contained handoff.

AI must not:

- Modify a public API/event/schema without explicit task scope.
- Add cross-service database access.
- Create a new service or repository without an accepted ADR.
- Introduce a dependency merely to avoid understanding existing code.
- Disable lint, typing, tests, security controls, authorization, or telemetry to make a build pass.
- Log private content, credentials, signed URLs, or provider payloads.
- Run destructive production actions without explicit authorization and verified targets.
- Merge, release, or declare success without evidence.
- Implement unresolved product behavior based on guesswork.

### Mandatory stop conditions

AI stops and requests a decision when:

- Authoritative documents conflict.
- Required product behavior is ambiguous and materially changes the result.
- A task requires a new service, data owner, contract, provider, or paid dependency not approved.
- A breaking contract or destructive migration appears necessary.
- Security, privacy, rights, billing, or retention requirements are unclear.
- The requested change would cross repository boundaries not listed in the task.
- Verification cannot be completed or fails for reasons outside the task.
- Production data or irreversible external state could be affected.

---

## 17. Task sizing and pull-request policy

The default is one primary repository and one independently reviewable behavior per pull request.

A task should be split when it:

- Combines contract definition, multiple provider implementations, UI exposure, and deprecation in one change.
- Touches unrelated domain concepts.
- Cannot be reviewed confidently in one sitting.
- Mixes refactoring with new behavior without necessity.
- Requires multiple independent rollback decisions.

Generated files and mechanical migrations do not alone make a change too large, but their sources and validation must be clear.

Cross-repository work uses a tracking feature with separately mergeable pull requests and an explicit compatibility order.

---

## 18. Cross-repository change protocol

Use this sequence for API, event, Activity, document-schema, or shared-package changes:

1. **Propose:** Document the new behavior and affected consumers.
2. **Add contract:** Introduce backward-compatible fields/endpoints/event versions.
3. **Publish canary:** Generate and publish prerelease clients/contracts.
4. **Prepare consumers:** Make consumers tolerate both old and new behavior.
5. **Implement provider:** Add provider behavior behind a flag where appropriate.
6. **Integrate:** Test using published versions and deployed service artifacts.
7. **Release provider:** Canary and observe.
8. **Migrate usage:** Move consumers to the new behavior.
9. **Measure:** Confirm old behavior is unused by supported clients.
10. **Remove later:** Deprecate and remove through a separate reviewed change.

Never require every repository to deploy at the same moment.

### Cross-repository tracking table

```markdown
| Order | Repository | Change | Contract version | Status | Rollback |
|---:|---|---|---|---|---|
| 1 | contracts | Add optional field | 1.3.0 | Planned | Republish 1.2.x |
| 2 | consumer | Tolerate field | API 1.3 | Planned | Revert consumer |
| 3 | provider | Populate field | API 1.3 | Planned | Disable flag |
```

---

## 19. Data and migration rules

- Every table, object prefix, and durable record has one owning service.
- Cross-service relationships use opaque IDs and versioned snapshots, not foreign keys or shared ORM models.
- Services validate the actor and referenced owner before accepting a relationship.
- Schema migrations are forward-only in production.
- Use expand → migrate/backfill → switch reads/writes → contract.
- A release must remain compatible with the preceding deployed version during rollout.
- Long backfills are resumable, observable, throttled, and independently deployable.
- Destructive changes require backup/restore validation and explicit owner approval.
- Every new private-data field has classification, retention, deletion, export, and logging decisions.
- Restored databases reconcile deletion tombstones before becoming ready.

Migration handoff must state:

- Expected row/object volume.
- Lock and performance risk.
- Forward and rollback/repair path.
- Backfill progress metric.
- Compatibility window.
- Data verification query or invariant.

---

## 20. API, event, and workflow rules

### APIs

- Public clients call Edge API only.
- Internal service APIs are private and separately authenticated.
- Commands support idempotency when retries could duplicate effects.
- Errors use stable machine codes and safe messages.
- Timeouts and retry responsibility are explicit.
- Public compatibility follows the supported client window.

### Domain events

- Events describe completed facts, not instructions disguised as facts.
- Every event includes event ID, type/version, producer, aggregate ID/version, occurrence time, trace/correlation/causation IDs.
- Events contain references, not private note/transcript/media bodies or signed URLs.
- Producers use transactional outboxes.
- Consumers are idempotent and handle duplicates, stale versions, gaps, and DLQ redrive.
- Projection services support reconciliation/rebuild from owners.

### Temporal workflows

- Workflow code is deterministic and contains no direct I/O.
- Capability services own Activity workers.
- Activities are idempotent, timeout-bound, and heartbeat when long-running.
- Workflows pass IDs and small results, not media or documents.
- Compensation is explicit and tested.
- Workflow changes pass replay and Activity-version compatibility tests.

---

## 21. Testing strategy

Required tests depend on the change:

| Test type | Purpose | Typical owner |
|---|---|---|
| Unit | Domain rules and pure transformations | Every repository |
| Component | UI behavior and accessibility | UI/application repositories |
| Contract | Provider/consumer compatibility | Contracts and service repositories |
| Integration | Database, S3, queue, provider adapter, Temporal Activity | Owning service |
| Workflow | Retry, timeout, crash, cancellation, compensation, replay | Workflow and capability services |
| End-to-end | User outcome through published/deployed artifacts | Application/platform suite |
| Security | Authz, tenant isolation, SSRF, XSS, secrets, malicious media | Relevant service/application |
| Performance | Load, latency, file size, queue recovery, editor scale | Risk-owning repository |
| AI evaluation | Grounding, coverage, citations, language, cost, regressions | AI evaluation repository |

Test behavior rather than private implementation details. A test that only mirrors the code provides weak protection.

Bug fixes require a failing regression test where practical. If a test is impractical, the handoff explains why and identifies alternative verification.

---

## 22. AI/model change policy

Prompt and model changes are production changes even when application code is unchanged.

They require:

- Versioned prompts/configuration.
- A stated hypothesis.
- Evaluation against affected benchmark slices.
- Quality, cost, and latency comparison to the current baseline.
- Schema and failure-rate checks.
- Canary or shadow rollout where practical.
- Kill switch and rollback configuration.
- Observation of user feedback and grounding metrics.

Do not select a model based on a few attractive examples. Aggregate improvements cannot hide a critical regression in a supported language or content type.

---

## 23. Security and privacy review

Escalate for explicit security/privacy review when work involves:

- Authentication, authorization, session, or share links.
- Uploaded media or arbitrary URLs.
- Private transcript/note content.
- Service-to-service identity or permissions.
- New external providers or data transfers.
- Billing, entitlements, or webhooks.
- Retention, deletion, export, backups, or support access.
- Electron IPC, deep links, auto-update, or native secure storage.
- Logging, analytics, AI evaluation data, or administrative tooling.

Every review asks:

- What data enters and leaves the boundary?
- Who can access it?
- Where is it stored and for how long?
- Could it appear in logs, events, prompts, analytics, or support tools?
- How is access revoked and data deleted?
- What happens if input is malicious?
- What is the least privilege required?

---

## 24. Observability requirements

Every production behavior must answer:

- How will we know it succeeded?
- How will we know it failed?
- How will support identify the affected job without reading private content?
- Which metric indicates degradation before users report it?
- What alert or dashboard owns the signal?

At minimum propagate request, trace, actor, artifact, generation, workflow, service version, event, and provider request identifiers where safe.

Logs are structured and content-free by default. Metrics must not use unbounded user or artifact IDs as labels.

An alert requires an owner and runbook. An unowned alert is not operational readiness.

---

## 25. Pull-request handoff

Every AI-authored pull request or equivalent handoff includes:

```markdown
## Outcome
What behavior now works?

## Why
Which feature/task/decision required it?

## Implementation
Important design choices and data flow.

## Contracts and compatibility
API/event/Activity/package/schema changes or “none.”

## Data and migrations
Migration/backfill/retention/deletion impact or “none.”

## Security and privacy
Authorization, sensitive data and threat impact.

## Observability
Logs, metrics, traces, dashboards and alerts.

## Verification
Commands actually run and results.

## Deployment
Order, configuration and feature flags.

## Rollback
How to restore safe behavior.

## Known limitations
What remains intentionally unsupported?

## Owner explanation
Plain-language explanation of ownership, communication and failure behavior.
```

Do not use “tests passed” without naming the checks or linking their results.

---

## 26. Review procedure

### AI reviewer checklist

- [ ] Change matches approved scope.
- [ ] Ownership and dependency directions are preserved.
- [ ] Contract compatibility is safe.
- [ ] Authorization is enforced at the owning service.
- [ ] Sensitive data is minimized and not logged.
- [ ] Failure, idempotency, retry, timeout, and cancellation paths are correct.
- [ ] Migrations and rollback/repair are safe.
- [ ] Tests would fail if the intended behavior broke.
- [ ] Telemetry is actionable and privacy-safe.
- [ ] Documentation matches the implementation.
- [ ] No unnecessary complexity or dependency was introduced.

### Human ownership review

Before merging material work, the owner should be able to explain:

1. The user outcome.
2. The owning component and data.
3. The communication sequence.
4. The main failure modes and recovery.
5. The release and rollback method.

If not, ask AI for a better explanation or diagram before merging.

---

## 27. Release and rollback

- Promote immutable artifacts that passed staging; never rebuild for production.
- Record exact application, service, contract, package, workflow, prompt/model, migration, and infrastructure versions in the release manifest.
- Use flags/canaries for risk-bearing behavior.
- Run automated smoke tests after deployment.
- Observe the defined health, quality, cost, and business signals.
- Stop rollout or roll back when a predeclared trigger fires.
- Do not roll back across an incompatible destructive migration; use forward repair.
- Keep incident communication and evidence linked to the release.

Every rollout plan defines:

- Deployment order.
- Compatibility window.
- Feature-flag state.
- Monitoring period.
- Rollback triggers.
- Rollback/forward-repair steps.
- Person or role authorized to proceed.

---

## 28. Bug and incident workflow

For a bug:

1. Reproduce or establish evidence.
2. Classify user impact and data/security risk.
3. Identify the owning service and last-known-good behavior.
4. Add a regression test or monitoring assertion.
5. Implement the smallest safe correction.
6. Review adjacent failure modes.
7. Release and observe.
8. Update documentation if the system understanding changed.

For a production incident:

- Stabilize and limit harm before pursuing a perfect diagnosis.
- Preserve evidence without copying private content unnecessarily.
- Use runbooks and reversible mitigations.
- Require explicit authority for destructive recovery.
- Produce a blameless post-incident review with corrective actions and owners.
- Encode lessons into tests, alerts, runbooks, or design rules.

AI may assist diagnosis and draft remediation. The incident commander/human owner controls production actions.

---

## 29. Repository activation policy

A repository from `repos.md` is created only when:

- An approved feature/release needs its capability.
- Its ownership boundary is documented.
- A named owner exists.
- Its initial contract or package purpose is known.
- CI, security, observability, and release ownership can be established.
- Its first vertical-slice task is ready.

Do not create all repositories as empty shells. Empty repositories create false progress and maintenance burden.

When activated, update `repos.md` from ☐ to ◐. Mark ☑ only when the repository meets its current-phase operational requirements.

---

## 30. Initial build sequence

The first objective is not the complete product. It is a working, observable skeleton that validates boundaries and delivery practices.

### Slice 0: Governance and golden path

Create only what is needed to prove the development system:

1. Product roadmap, glossary, service catalog, data-ownership map, event catalog, and initial ADRs.
2. Service template.
3. Infrastructure sandbox.
4. Contracts repository and private package publishing.
5. Frontend foundation and web UI repositories.
6. AI evaluation repository with manifest/rubric/report scaffolding and deterministic fake-provider support.
7. One sample service deployed with health, telemetry, database migration, outbox, event, CI, and rollback.

**Exit:** A new service can be created consistently, deployed safely, and observed.

### Slice 1: Platform walking skeleton

Deliver:

> Sign in → create an empty artifact → start a durable fake generation → receive progress → materialize a deterministic fake note → display it on web.

Use real contracts, authentication, service databases, Temporal, events, job progress, note persistence, telemetry, and deployment. Use deterministic fake media/transcription/AI outputs.

Likely repositories activated:

- Contracts
- Frontend foundation
- Web UI
- Infrastructure
- Service template
- Web
- Edge API
- Account
- Library
- Workflow
- Notes

**Exit:** The complete distributed control path works without expensive or uncertain media/AI behavior.

### Slice 2: Real local upload and media preparation

Add Ingestion and Media services. Support one constrained file type/size first.

**Exit:** A real uploaded source is validated, stored, processed, and cleaned up through the durable workflow.

### Slice 3: Real transcription

Add Transcription service with one evaluated language/provider and timestamped segments.

**Exit:** Transcript timing and failure behavior pass the benchmark and retention rules.

### Slice 4: Grounded AI notes

Add AI Notes service and expand the already activated AI Evaluations repository. Implement evidence-first generation, citations, quality checks, budgets, and prompt/model versioning.

**Exit:** A real local video produces a benchmark-approved, timestamp-grounded note.

### Slice 5: Study-ready web MVP

Add editing/version recovery, Search, Export, Notification, deletion, accessibility, security, and reliability work.

**Exit:** Internal alpha acceptance criteria pass.

### Slice 6: Private beta and billing

Add Billing, approved hosted-source adapters, usage enforcement, sharing, support workflows, and beta operations.

### Slice 7: Desktop

Activate Desktop repository using the already published React DOM and platform-neutral packages.

### Slice 8: Mobile

Activate Native UI and Mobile repositories using the published domain, API, document, controller, telemetry, and token packages.

---

## 31. Owner control checklist

At the end of every feature, the owner verifies:

- [ ] I can explain why this feature exists.
- [ ] I know which repositories and services participate.
- [ ] I know which service owns each datum.
- [ ] I understand the normal sequence.
- [ ] I understand the major failure and recovery paths.
- [ ] I understand the security/privacy implications.
- [ ] I know how the feature is tested and observed.
- [ ] I know how it is enabled, disabled, deployed, and rolled back.
- [ ] The decisions and contracts are written down.
- [ ] I am comfortable accepting the remaining risks.

If any answer is “no,” the feature is not yet fully owned, even if it is implemented.

---

## 32. Quick-start prompt set

### Design prompt

```text
Act as a system designer. Read the linked PRD, TRD, playbook, service catalog,
and feature brief. Do not implement code.

Produce:
1. The smallest vertical slice.
2. Participating repositories and data owners.
3. A sequence diagram.
4. APIs/events/Activities affected.
5. Failure, retry, idempotency and compensation behavior.
6. Security/privacy implications.
7. Decisions that require owner approval.
8. Ordered repository tasks with compatibility sequencing.

State assumptions and stop where product intent is unresolved.
```

### Implementation prompt

```text
Implement the approved task in the named primary repository.
Read AGENTS.md and every linked authoritative document first.
Do not change contracts, service boundaries or product scope unless explicitly authorized.

Before editing, report your understanding, affected boundaries and verification plan.
Then implement code, tests, migrations, telemetry and documentation.
Run the required checks and provide the standard pull-request handoff.
Stop on any playbook stop condition.
```

### Review prompt

```text
Review this change independently as a correctness, architecture, security,
data and operations reviewer. Do not assume the implementation is correct.

Compare it with the task, contracts, service specification, ADRs and playbook.
Prioritize concrete findings with file/line evidence. Check negative paths,
authorization, idempotency, retries, migrations, compatibility, telemetry,
privacy, deployment and rollback. Report missing tests and documentation.
```

### Owner explanation prompt

```text
Explain this feature to me as the product/system owner.
Cover why it exists, component ownership, data model, communication sequence,
failure/recovery, security/privacy, tests, observability, deployment and rollback.
Use a diagram only if it makes the relationships clearer. Identify anything
I still need to decide or understand before approving it.
```

---

## Appendix A: Feature task register

```markdown
| Order | Task | Primary repository | Depends on | Status | Evidence |
|---:|---|---|---|---|---|
| 1 | Add upload contract | contracts | ADR-### | Proposed | |
| 2 | Consume upload contract | web | Contract prerelease | Proposed | |
| 3 | Implement upload session | ingestion | Contract stable | Proposed | |
```

## Appendix B: Decision log

```markdown
| Date | Decision | Owner | ADR | Revisit trigger |
|---|---|---|---|---|
| YYYY-MM-DD | <decision> | <owner> | ADR-### | <evidence> |
```

## Appendix C: Release manifest

```yaml
release: "web-alpha-001"
released_at: "YYYY-MM-DDTHH:MM:SSZ"
applications:
  web: "1.0.0"
services:
  edge-api: "1.0.0"
  account-service: "1.0.0"
contracts:
  public-api: "1.0.0"
packages:
  api-client: "1.0.0"
  frontend-foundation: "1.0.0"
workflows:
  generation: "v1"
ai:
  prompt-set: "notes-v1"
  model-routing: "routing-v1"
infrastructure: "git-sha-or-version"
database_migrations:
  account-service: "revision-id"
feature_flags:
  local-upload: true
rollback_owner: "<role>"
```

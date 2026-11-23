# Product Requirements Document: Study Notes Creator

**Document status:** Draft for product review  
**Version:** 0.1  
**Last updated:** 23 November 2026  
**Working product name:** Study Notes Creator  
**Audience:** Product, design, engineering, AI/ML, legal, security, growth, and support  
**Next document:** Technical Requirements Document / High-Level Design

---

## 1. Executive summary

Study Notes Creator is a cross-platform learning product that turns a video into detailed, easy-to-understand, visually polished study notes. A user can provide:

1. A public YouTube URL.
2. A supported direct or hosted video URL.
3. A video file uploaded from their device.

The product processes the source, understands its structure and teaching intent, and creates an editable study document grounded in the video. Notes include clear sections, explanations, examples, key takeaways, definitions, and timestamp citations so users can verify important claims and return to the relevant moment in the source.

The experience will be available on the web, mobile, and desktop. A shared account and cloud library keep sources, generation progress, notes, edits, and study state synchronized across devices.

The initial product will optimize for one primary promise:

> Give the product a learning video and receive trustworthy notes that are substantially faster to review than the video, without losing the context needed to understand the material.

The MVP is not a generic chatbot, a video downloader, or a full learning management system. Its job is to create a reliable first draft of a high-quality study artifact and let the learner refine, verify, and export it.

---

## 2. Problem statement

Video is an effective teaching medium but an inefficient reference medium. Learners regularly pause, rewind, transcribe, take screenshots, reorganize ideas, and search for a previously mentioned concept. This creates several problems:

- Manual note-taking divides attention between listening and writing.
- Raw transcripts are long, repetitive, and poorly structured for revision.
- Generic AI summaries are often too shallow and may omit examples or invent details.
- Existing tools frequently separate notes from evidence, making factual verification difficult.
- Notes created on one device are not always available or comfortable to consume on another.
- Different learning goals require different levels of detail and explanation.
- Long videos can fail during upload or processing without useful recovery.

The product opportunity is to preserve the educational value of the video while converting it into a structured, verifiable, and reusable learning resource.

### 2.1 Why existing alternatives fall short

| Alternative | Strength | Core limitation |
|---|---|---|
| Manual notes | High personal relevance | Slow and cognitively demanding |
| Transcript | Complete spoken record | Verbose, unstructured, and hard to revise |
| Basic video summary | Fast overview | Often too shallow for studying |
| General-purpose chat assistant | Flexible | Requires prompting and may not preserve source grounding |
| Screenshot/bookmark workflow | Preserves moments | Produces fragmented, non-searchable material |

---

## 3. Product vision and principles

### 3.1 Vision

Become the most trusted way to turn long-form video into a personal, living study resource on any device.

### 3.2 Product principles

1. **Grounded before fluent.** Important statements must be traceable to the source. The product should communicate uncertainty rather than fabricate certainty.
2. **Understanding before compression.** Notes should preserve explanations, reasoning, and examples—not merely shorten a transcript.
3. **Useful by default, controllable when needed.** A first-time user should get strong results without prompt engineering, while advanced users can guide depth, language, focus, and format.
4. **Progressive disclosure.** A learner can scan a summary, study the detailed notes, or return to the exact source moment.
5. **The user owns the artifact.** Generated notes are editable, exportable, and clearly distinguish source-grounded content from user or AI-added material.
6. **One library across devices.** Capture may happen on mobile, deep study on desktop, and quick review on the web; state should follow the user.
7. **Failure should be recoverable.** Upload and generation must expose progress, explain errors, and resume wherever practical.

---

## 4. Goals, non-goals, and product boundaries

### 4.1 Goals

- Accept the three primary input types with a simple and resilient flow.
- Produce detailed, readable, source-grounded notes from educational videos.
- Make claims easy to verify with timestamps and source playback.
- Let users customize generation without requiring prompt-writing skills.
- Support editing, search, organization, sharing, and common export formats.
- Provide a consistent account and library across web, mobile, and desktop.
- Build user trust through transparent processing states, provenance, and privacy controls.
- Establish measurable quality and retention loops before expanding into a broader study suite.

### 4.2 Non-goals for MVP

- Downloading or redistributing copyrighted videos.
- Bypassing paywalls, DRM, access controls, login walls, or platform restrictions.
- Real-time meeting transcription or live classroom recording.
- A full LMS, course marketplace, classroom administration system, or gradebook.
- Fully autonomous web research beyond the supplied source.
- Guaranteed expert-level correctness in regulated or high-stakes domains.
- Pixel-identical features on all platforms when the device context calls for a different interaction.
- Full offline generation. Offline reading may be added later.

### 4.3 Definition of “agentic” in this product

The product behaves as a coordinated workflow rather than a single opaque summarization prompt. It plans how to process the source, identifies topic boundaries, extracts evidence, drafts notes, checks coverage and grounding, and formats the result. The system may retry or repair a failed stage automatically within safe limits.

Agentic behavior must remain observable and bounded:

- The user sees meaningful progress stages.
- The system does not access unrelated data or take external actions.
- Added knowledge is not silently presented as if it came from the video.
- The user can cancel generation, retry failure, and edit the result.
- If the source is insufficient, the product says so.

---

## 5. Target users and jobs to be done

### 5.1 Primary personas

#### A. University learner

- Watches lectures, recorded classes, and tutorials.
- Needs comprehensive notes for exams and assignments.
- Values definitions, worked examples, diagrams, and timestamp references.
- Often captures on mobile and studies on a laptop.

#### B. Professional upskiller

- Learns technical or business topics from long-form online video.
- Has limited time and wants a fast, structured reference.
- Values key decisions, procedures, practical examples, and export to an existing knowledge system.

#### C. Educator or course creator

- Wants handouts or lesson notes from their own recordings.
- Values editing, consistent structure, sharing, and reusable exports.
- Is a secondary persona for MVP; educator-specific administration is later scope.

### 5.2 Jobs to be done

**Primary job**

> When I have a learning video, help me turn it into reliable, well-organized notes so I can understand and revise the material without repeatedly rewatching the whole video.

**Supporting jobs**

- Help me decide whether a video deserves deeper study.
- Help me find the exact moment that supports a note.
- Help me adapt explanations to my current knowledge level.
- Help me retain the useful parts of a video in my personal library.
- Help me move the result into the tools where I already study or work.
- Help me continue a task on another device without losing progress.

---

## 6. Key product decisions

These decisions define the recommended starting scope and should be challenged during product review if needed.

| Area | Decision | Rationale |
|---|---|---|
| Account | Allow lightweight exploration, require sign-in before generation | Generation has cost and users need a persistent cross-device library |
| Default output | “Detailed” notes with overview, structured sections, examples, definitions, takeaways, and citations | Best matches the core promise without setup burden |
| Source grounding | Timestamp citations are first-class and visible | Trust and navigation are differentiators |
| Generation controls | Presets plus a small advanced panel | Avoids prompt engineering while retaining control |
| Editing | Block-based rich text with autosave | Notes are a living artifact, not a static AI response |
| External knowledge | Off by default for MVP | Avoids mixing source content with unsourced material |
| Transcript | Available alongside notes when permitted | Supports verification, search, and accessibility |
| Collaboration | Read-only link sharing first; co-editing later | Covers basic sharing without real-time complexity |
| Platforms | Responsive web first, followed by mobile and desktop clients against the same product model | Enables validation while preserving the cross-platform vision |
| Monetization | Free trial/allowance plus an individual paid plan; exact packaging after cost and willingness-to-pay tests | Processing cost grows with video length and quality |

---

## 7. End-to-end user experience

### 7.1 First-use journey

1. The user lands on a page that explains the outcome and supported inputs.
2. The user pastes a URL or chooses a local file.
3. The product validates the source and shows detected metadata where available: title, thumbnail, duration, language, and source type.
4. The user signs in or creates an account if needed.
5. The user accepts the default “Detailed notes” preset or adjusts options.
6. The source is added to the library and processing begins.
7. The product shows progress through clear stages and lets the user leave the screen.
8. The user is notified when the notes are ready.
9. The result opens in a study workspace with source playback, outline, notes, and timestamps.
10. The user reviews, edits, searches, exports, or shares the artifact.
11. The product asks for lightweight quality feedback at a natural point.

### 7.2 Returning-user journey

1. The user opens their synchronized library.
2. They filter or search by title, source, content, tag, status, or date.
3. They resume a note at the last reading or playback position.
4. Edits and study state synchronize across devices.

### 7.3 Input-specific behavior

#### YouTube URL

- Validate that the URL is recognizable and accessible.
- Display available public metadata.
- Use only access methods allowed by applicable platform terms and user rights.
- Clearly report unavailable, private, age-restricted, region-restricted, live, or otherwise unsupported content.
- Never present the product as a YouTube downloading tool.

#### Hosted video URL

- Support an explicit allowlist of direct media and approved hosting providers for MVP.
- Detect non-video pages, expired signed URLs, authentication requirements, and unsupported media.
- Tell the user when they should upload a file they are authorized to use instead.

#### Local file upload

- Show file name, size, detected duration/type, and upload progress.
- Validate format, size, duration, audio availability, and corruption before costly processing.
- Support resumable upload where the platform permits.
- Keep the job visible if the user navigates away or the app backgrounds.
- Provide an explicit source-file retention/deletion control.

### 7.4 Generation setup

The default path should require no configuration. Optional controls include:

- **Depth:** Quick, Detailed (default), or Comprehensive.
- **Audience level:** Beginner, Intermediate (default), or Advanced.
- **Output language:** Same as source (default) or a selected supported language.
- **Focus:** Entire video (default) or a user-entered learning objective.
- **Include:** Examples, definitions, formulas/code, action steps, and transcript where relevant.

The UI should show a rough processing-time or usage estimate before confirmation when duration is known. Exact pricing and quota messaging must be unambiguous.

### 7.5 Processing states

Users see human-readable states rather than an indefinite spinner:

1. Validating source.
2. Uploading or importing.
3. Preparing audio/video.
4. Creating or aligning the transcript.
5. Understanding topics and structure.
6. Writing and checking notes.
7. Formatting the study document.
8. Ready, partially ready, canceled, or failed.

Long-running work continues independently of an open screen. The product sends an in-app notification and, with consent, push or email notification when ready. If a recoverable stage fails, it retries within defined limits. Otherwise it preserves completed work and offers a clear next action.

### 7.6 Study workspace

The workspace contains:

- Source title, metadata, processing status, and note settings.
- Embedded or native source playback when legally and technically supported.
- A collapsible document outline.
- The generated notes in an editable document.
- Timestamp links that seek the player to the relevant moment.
- Transcript search and synchronized highlighting when available.
- Global note search, copy, undo/redo, and autosave state.
- Export, share, regenerate, and delete controls.
- A lightweight feedback control for incorrect, missing, or unclear content.

On smaller screens, player, outline, and document become separate modes or sheets rather than compressed panes.

---

## 8. The study-note artifact

### 8.1 Default document structure

The generator adapts to the source but aims to produce:

1. **Title and source context**
2. **At-a-glance overview**
   - What the video covers
   - Who it is for
   - Prerequisite knowledge, if evident
3. **Learning objectives**
4. **Structured outline**
5. **Detailed topic sections**
   - Plain-language explanation
   - Important details and reasoning
   - Examples, demonstrations, or analogies from the source
   - Definitions of key terms
   - Formulas, code, steps, or tables when appropriate
   - Timestamp citations
6. **Key takeaways**
7. **Terms and definitions**
8. **Questions or ambiguities in the source**, when material

Sections with no support in the source should be omitted rather than padded.

### 8.2 Quality rules

- Notes must preserve the source’s meaning and distinguish claims, opinions, examples, and speculation.
- Important sections and claims must carry one or more timestamp references when timing information is available.
- Direct quotations must be minimal, accurate, visibly quoted, and linked to the source moment.
- Examples must be labeled if simplified or newly constructed by the system.
- Added explanations must not masquerade as the speaker’s words.
- Repetition, filler, sponsorship, and housekeeping may be compressed or excluded unless educationally relevant.
- Contradictions or ambiguity in the source should be surfaced, not silently resolved.
- Visual presentation must remain readable when exported to a less expressive format.
- Unsupported visuals must never be described as if they were reliably observed.

### 8.3 Visual treatment

“Beautiful” means functional clarity, not decoration. The artifact should use:

- Strong typographic hierarchy.
- Consistent spacing and section rhythm.
- Callouts for definitions, examples, warnings, and key ideas.
- Tables for meaningful comparison.
- Proper code and formula formatting.
- Optional thumbnails or source frames only when usage rights permit and the frame helps learning.
- A light and dark theme.
- Print-friendly styling.

AI-created diagrams are a post-MVP option and must be labeled as generated interpretations rather than source evidence.

### 8.4 Regeneration and editing

- A user can edit any block without losing the original generation metadata.
- The product autosaves changes and communicates save/sync conflicts.
- A user can regenerate the whole document with new settings in MVP.
- Regenerating creates a recoverable version rather than silently overwriting edits.
- Section-level regeneration, version comparison, and “make this simpler” are post-MVP capabilities.
- User-written content must never be overwritten without explicit confirmation.

---

## 9. Functional requirements

Priority definitions:

- **P0:** Required for a public MVP.
- **P1:** Required for a strong v1 shortly after MVP or for platform completeness.
- **P2:** Valuable expansion after product-market evidence.

### 9.1 Account and onboarding

| ID | Requirement | Priority |
|---|---|---|
| ACC-01 | Users can create an account, sign in, sign out, and recover access. | P0 |
| ACC-02 | Users can use the same account and library across supported platforms. | P0 |
| ACC-03 | Onboarding explains supported sources, expected output, quotas, and privacy at the point of need. | P0 |
| ACC-04 | Users can manage profile, language, theme, notification, and privacy preferences. | P1 |
| ACC-05 | Users can export account data and request account deletion. | P0 |

### 9.2 Source ingestion

| ID | Requirement | Priority |
|---|---|---|
| ING-01 | Users can submit a supported public YouTube URL. | P0 |
| ING-02 | Users can upload supported local video files with visible progress. | P0 |
| ING-03 | Users can submit supported direct/hosted video URLs. | P0 |
| ING-04 | The product validates source type, availability, format, duration, size, and presence of usable audio before generation. | P0 |
| ING-05 | The product displays actionable errors for private, inaccessible, expired, corrupted, silent, oversized, or unsupported sources. | P0 |
| ING-06 | Uploads can resume after transient interruption where client and source permit. | P1 |
| ING-07 | Duplicate sources are detected and the user can open, duplicate, or regenerate an existing artifact. | P1 |
| ING-08 | Users can capture a supported URL through mobile share sheets and desktop/browser handoff. | P1 |
| ING-09 | Batch import is supported. | P2 |

### 9.3 Transcription and source understanding

| ID | Requirement | Priority |
|---|---|---|
| AI-01 | The product creates or obtains a time-aligned transcript using permitted methods. | P0 |
| AI-02 | The product identifies the primary spoken language and lets the user correct it. | P0 |
| AI-03 | The product segments the video into coherent topics and preserves timestamps. | P0 |
| AI-04 | The product handles common audio noise and reports when source quality limits output quality. | P0 |
| AI-05 | The product recognizes important spoken definitions, steps, examples, claims, and transitions. | P0 |
| AI-06 | When visual understanding is not available or reliable, the output does not pretend to have interpreted on-screen content. | P0 |
| AI-07 | Supported visual content such as slides, diagrams, formulas, and code can contribute to the notes. | P1 |
| AI-08 | Multiple speakers are distinguished when it materially affects understanding. | P1 |

### 9.4 Note generation and trust

| ID | Requirement | Priority |
|---|---|---|
| GEN-01 | Users can generate Quick, Detailed, or Comprehensive notes. | P0 |
| GEN-02 | Detailed is the default and follows the structure and quality rules in Section 8. | P0 |
| GEN-03 | Users can select audience level, output language, and an optional focus. | P0 |
| GEN-04 | Important claims and sections link to relevant timestamps when timing data exists. | P0 |
| GEN-05 | The system performs a coverage and grounding check before marking notes ready. | P0 |
| GEN-06 | The product flags low-confidence or unsupported areas instead of inventing content. | P0 |
| GEN-07 | Users can see which content came from the source and which was added by AI or by the user. | P1 |
| GEN-08 | Users can regenerate without destructively replacing their current document. | P0 |
| GEN-09 | Users can generate only from a selected time range. | P1 |
| GEN-10 | The product can enrich notes with external sources, with separate citations and explicit opt-in. | P2 |

### 9.5 Processing and notifications

| ID | Requirement | Priority |
|---|---|---|
| JOB-01 | Generation runs as a durable background job and survives navigation or app closure. | P0 |
| JOB-02 | Users see the current stage and a useful progress indication. | P0 |
| JOB-03 | Users can cancel pending or active work. | P0 |
| JOB-04 | Recoverable failures retry automatically within bounded limits. | P0 |
| JOB-05 | Failed jobs preserve available diagnostics and offer retry or corrective action. | P0 |
| JOB-06 | Users receive an in-app readiness or failure notification. | P0 |
| JOB-07 | Users can opt into email and push notifications. | P1 |
| JOB-08 | Usage deducted for failed jobs is restored according to a clearly documented policy. | P0 |

### 9.6 Editor and study workspace

| ID | Requirement | Priority |
|---|---|---|
| EDT-01 | Users can edit headings, paragraphs, lists, tables, callouts, code, and basic formulas. | P0 |
| EDT-02 | Changes autosave and synchronize across devices. | P0 |
| EDT-03 | Timestamp links seek the supported player to the cited moment. | P0 |
| EDT-04 | Users can search within notes and transcript. | P0 |
| EDT-05 | Users can navigate with a generated outline. | P0 |
| EDT-06 | The workspace preserves last reading and playback position. | P1 |
| EDT-07 | Users have undo/redo and recoverable document versions. | P1 |
| EDT-08 | Users can add personal highlights, annotations, and bookmarks. | P1 |
| EDT-09 | The product generates flashcards, a quiz, and review prompts from a note. | P2 |
| EDT-10 | A grounded Q&A assistant answers questions about the source and cites timestamps. | P2 |

### 9.7 Library and organization

| ID | Requirement | Priority |
|---|---|---|
| LIB-01 | Users have a library showing source, thumbnail, status, duration, creation date, and last activity. | P0 |
| LIB-02 | Users can rename, archive, and delete an artifact. | P0 |
| LIB-03 | Users can search titles and note content. | P0 |
| LIB-04 | Users can sort and filter by source type, status, and date. | P0 |
| LIB-05 | Users can organize artifacts with folders or collections and tags. | P1 |
| LIB-06 | Recently deleted artifacts are recoverable for a documented period. | P1 |
| LIB-07 | Users can pin or favorite an artifact. | P1 |

### 9.8 Export and sharing

| ID | Requirement | Priority |
|---|---|---|
| EXP-01 | Users can copy selected content or the complete note. | P0 |
| EXP-02 | Users can export clean Markdown and PDF. | P0 |
| EXP-03 | Exports preserve headings, tables, code/formulas where possible, and source attribution. | P0 |
| EXP-04 | Users can create a revocable read-only share link. | P1 |
| EXP-05 | Users can export DOCX and print cleanly. | P1 |
| EXP-06 | Users can export to common knowledge and study tools through integrations. | P2 |
| EXP-07 | Real-time co-editing, comments, and team workspaces are supported. | P2 |

### 9.9 Billing and usage

| ID | Requirement | Priority |
|---|---|---|
| BILL-01 | Users can see remaining allowance and the estimated usage before generation. | P0 |
| BILL-02 | The product enforces plan limits without losing submitted work or edits. | P0 |
| BILL-03 | Users can subscribe, manage billing, and view transaction/usage history. | P0 for paid launch |
| BILL-04 | Pricing distinguishes costly inputs such as duration or comprehensive processing in a way users can predict. | P0 for paid launch |
| BILL-05 | Institutions and teams can centrally purchase and administer seats. | P2 |

### 9.10 Feedback and support

| ID | Requirement | Priority |
|---|---|---|
| FB-01 | Users can rate a generated note with minimal friction. | P0 |
| FB-02 | Users can mark content as incorrect, missing, unclear, poorly formatted, or linked to the wrong timestamp. | P0 |
| FB-03 | Feedback may include the relevant block and source evidence only with clear disclosure. | P0 |
| FB-04 | Users can report abuse, copyright concerns, or a privacy issue. | P0 |
| FB-05 | Users can submit diagnostics to support without exposing source content by default. | P1 |

---

## 10. Cross-platform product plan

The platforms share the same account, source model, document model, processing jobs, entitlements, and design language. Platform-specific experiences should respect the device’s strengths.

| Capability | Web | Mobile | Desktop |
|---|---|---|---|
| Paste URL and upload | P0 | P0 | P0 |
| Full library and reading | P0 | P0 | P0 |
| Rich editing | P0 | Core editing P0; advanced editing P1 | P0 |
| Background upload/generation tracking | P0 | P0 within OS limits | P0 |
| Push/share-sheet capture | Browser notification P1 | P1 | OS handoff P1 |
| Local file workflow | Browser picker | Photo/file picker | Drag/drop and file picker |
| Multi-pane study layout | Large screens | Mode-based compact UI | Large screens |
| Offline reading | Browser cache P2 | P1 | P1 |
| Local folder watching/batch ingest | Not applicable | Not applicable | P2 |

### 10.1 Recommended release sequence

1. **Responsive web MVP:** Validate ingestion, generation quality, trust, library, editor, and willingness to pay.
2. **Mobile v1:** Prioritize URL capture, device upload, notifications, reading, search, and lightweight editing.
3. **Desktop v1:** Prioritize drag-and-drop, long study sessions, advanced editing, keyboard workflows, and optional offline access.
4. **Platform parity pass:** Close high-value gaps based on observed journeys, not feature-count parity.

This sequence is a delivery recommendation, not an architecture decision. The TRD/HLD will evaluate how much implementation can be shared without degrading platform quality.

---

## 11. Non-functional product requirements

Technical mechanisms and exact service-level objectives belong in the TRD, but the product requires these user-facing outcomes.

### 11.1 Performance and responsiveness

- URL validation should normally feel immediate and provide staged feedback if it takes longer.
- The library and existing notes should become usable quickly on ordinary consumer networks.
- Typing, navigation, search, and playback seeking must feel responsive on supported devices.
- Generation time should be communicated honestly; the user should not need to keep the app open.
- Very long documents should remain scrollable and editable without visible degradation.

### 11.2 Reliability

- Accepted jobs must not disappear.
- Uploads and processing should recover from normal transient failures.
- Edits must not be silently lost.
- Users must see whether work is saved, syncing, conflicted, failed, or offline.
- Status shown on different devices should converge promptly.

### 11.3 Accessibility

- Target WCAG 2.2 AA for the web product and equivalent native accessibility guidance.
- Core flows must support keyboard-only and screen-reader use.
- Text must reflow at high zoom and respect platform font sizing.
- Color cannot be the only signal for status, confidence, or errors.
- Player and timestamp controls need accessible names and focus behavior.
- Motion should respect reduced-motion preferences.

### 11.4 Localization

- The UI is English-first but must be designed for localization.
- Source language and output language are separate concepts.
- Unicode, right-to-left layout, locale-aware dates/numbers, and longer translated strings must be considered from the start.
- The launch language list will be based on evaluated transcription and note quality, not nominal model coverage.

### 11.5 Compatibility

- Define and publish supported browsers, mobile OS versions, desktop OS versions, file formats, maximum duration, and maximum file size before beta.
- Unsupported clients receive a clear message rather than a degraded or insecure experience.

---

## 12. AI quality, evaluation, and guardrails

### 12.1 Quality dimensions

Every generated artifact should be evaluated against:

| Dimension | Question |
|---|---|
| Groundedness | Are statements supported by the source and correctly attributed? |
| Coverage | Are the important topics, reasoning, examples, and conclusions represented? |
| Correctness | Does the note preserve the meaning of the source without distortion? |
| Citation accuracy | Do timestamps lead to evidence for the associated content? |
| Structure | Is the document logically organized and easy to navigate? |
| Clarity | Is it understandable at the requested audience level? |
| Concision | Is repetition removed without losing needed context? |
| Visual fidelity | Are formulas, code, tables, and visual references handled correctly? |
| Language quality | Is the selected output language fluent and consistent? |
| Safety | Does the output avoid harmful transformations and clearly frame sensitive content? |

### 12.2 Evaluation program

Before public launch, create a permissioned benchmark set spanning:

- Short, medium, long, and very long videos.
- Lectures, tutorials, interviews, demonstrations, and screen recordings.
- Clear and noisy audio.
- Single and multiple speakers.
- Different accents, speaking speeds, and supported languages.
- STEM formulas, code, diagrams, business topics, and humanities content.
- Videos with misleading claims, contradictions, sponsorship, or little educational content.

Evaluation combines:

- Automated checks for citation coverage, timing validity, structural completeness, and unsupported content.
- Expert or trained human review with a consistent rubric.
- Side-by-side preference tests against baselines and previous product versions.
- Production feedback linked to the relevant stage and model/configuration version.
- Regression gates before changes reach all users.

### 12.3 Launch quality gates

Numeric thresholds should be set after the benchmark pilot, but launch requires:

- No known systematic fabrication of unsupported facts.
- Timestamp links that consistently lead reviewers to relevant evidence.
- Strong human preference for generated notes over a cleaned raw transcript.
- Acceptable quality across every advertised language and supported content type.
- Clear failure or low-confidence behavior for poor sources.
- No critical safety, privacy, or rights-management failures.

### 12.4 High-stakes content

For medical, legal, financial, safety-critical, or other high-stakes topics:

- Clearly state that generated notes may contain errors and are not professional advice.
- Preserve source attribution and timestamps.
- Avoid presenting the product as a certification of source accuracy.
- Make reporting a dangerous or materially incorrect output easy.

---

## 13. Privacy, security, rights, and responsible use

### 13.1 Data principles

- Collect only data needed to provide and improve the product.
- Explain separately how source media, transcripts, notes, telemetry, and feedback are used.
- Do not use private source content to train shared models by default; any future opt-in must be explicit and revocable where practical.
- Encrypt data in transit and at rest using appropriate modern controls.
- Apply least-privilege access and audit sensitive administrative access.
- Give users clear retention and deletion controls.

### 13.2 Recommended retention defaults

These defaults require validation against cost, UX, and legal needs:

- Uploaded source video: delete after successful processing plus a short recovery window, unless the user explicitly chooses to retain it.
- Derived transcript and note: retain in the user’s library until deletion.
- Failed or canceled raw uploads: delete promptly after a documented diagnostic/recovery period.
- Deleted artifacts: enter a short recoverable trash period, then be permanently removed subject to documented backup and legal constraints.

The settings page and upload flow must communicate the actual implemented policy in plain language.

### 13.3 Rights and platform compliance

- Users must confirm that they have the right to process uploaded or access-controlled material.
- Public availability does not imply permission to copy or redistribute a source.
- The product must respect platform terms, copyright, DRM, access controls, and takedown obligations.
- Sharing a generated note must not expose the underlying private source or full transcript without authorization.
- Exports should retain source attribution and links where applicable.
- Establish a rights-holder complaint and takedown process before public launch.
- Legal review is required for each third-party ingestion method and target market.

### 13.4 Security and abuse cases

Product and technical design must address:

- Malicious, corrupted, or disguised uploads.
- Prompt injection or adversarial instructions embedded in transcripts or video frames.
- Attempts to extract other users’ private sources or notes.
- Enumeration of share links.
- Excessive generation, automated scraping, fraud, and quota evasion.
- Unsafe or illegal source content.
- Accidental exposure through logs, analytics, support tools, exports, or notifications.

Source content is untrusted data, never system instruction. The processing workflow must not follow requests embedded in a video to access secrets, call tools, or change product rules.

### 13.5 Minors and education

Initial launch should target users old enough to independently consent under applicable rules. Supporting younger learners, schools, or institution-managed accounts requires a separate privacy, consent, moderation, and procurement workstream.

---

## 14. Business model and packaging hypothesis

The business model should align revenue with the primary variable cost—media duration and processing depth—without forcing users to understand model tokens or infrastructure.

### 14.1 Proposed packaging

- **Free:** A small one-time or monthly allowance, standard queue, limited maximum video length, core notes, and basic exports.
- **Individual paid:** Larger monthly duration allowance, longer videos, all depth modes, priority processing, richer exports, and expanded storage/history.
- **Education/team (later):** Shared billing, administration, private workspaces, policy controls, and collaboration.

### 14.2 Packaging principles

- Show the cost before a job starts.
- Do not charge for product-caused failures.
- Let users buy additional capacity or wait for renewal without losing the source record.
- Avoid an unlimited plan until usage and abuse patterns are understood.
- Test whether users value minutes processed, videos processed, or an outcome-based bundle before final pricing.

---

## 15. Success metrics and instrumentation

### 15.1 North-star metric

**Weekly successfully studied artifacts:** unique generated artifacts that a user meaningfully engages with after generation—for example, reads beyond a threshold and performs a learning action such as following a timestamp, editing, searching, exporting, or returning later.

This measures delivered learning utility more accurately than raw generations.

### 15.2 Primary metrics

| Funnel stage | Metric |
|---|---|
| Acquisition | Qualified visitor-to-source-submission rate |
| Activation | New users who receive and meaningfully engage with their first successful note |
| Time to value | Median time from valid submission to first useful note view, segmented by duration and preset |
| Reliability | Successful completion rate for accepted jobs by source type |
| Quality | Positive rating rate, issue rate by category, human evaluation scores |
| Trust | Timestamp usage, citation correction rate, unsupported-content reports |
| Engagement | Meaningful actions per artifact and cross-device continuation rate |
| Retention | Users returning to generate or study in subsequent weeks |
| Monetization | Free-to-paid conversion, paid retention, gross margin by workload |

### 15.3 Guardrail metrics

- Upload or import failure rate.
- Job cancellation and retry rate.
- Notes with no or inadequate citation coverage.
- Incorrect timestamp reports.
- Edit loss or sync conflict rate.
- Permanent deletion completion and privacy-request turnaround.
- Copyright and abuse reports.
- Cost per processed hour and cost per meaningfully studied artifact.
- Support contacts per 100 completed jobs.

### 15.4 Event principles

- Instrument source submission, validation result, setup selection, each processing state, ready/failure, note open, meaningful scroll/read, citation click, edit, search, export, share, feedback, delete, and subscription events.
- Avoid sending raw transcript or note content to general-purpose analytics.
- Version generation configurations so quality changes can be attributed.
- Define each metric and event before implementation to prevent contradictory dashboards.

---

## 16. Rollout plan

### Phase 0: Discovery and feasibility

**Objective:** Validate demand, input rights, quality expectations, and cost envelope.

- Interview learners and professionals across target use cases.
- Test prototype notes at multiple detail levels.
- Create the evaluation rubric and permissioned benchmark set.
- Validate ingestion methods with legal/platform review.
- Measure processing cost and latency by video duration and content type.
- Prototype the study workspace and timestamp verification loop.

**Exit criteria:** Clear target segment, preferred artifact shape, viable ingestion path, benchmark baseline, and a credible cost model.

### Phase 1: Internal alpha

**Scope:** Responsive web, account, core inputs, background processing, detailed notes, citations, editor, library, Markdown/PDF export, deletion, and feedback.

**Objective:** Make the complete workflow reliable on controlled sources.

**Exit criteria:** Launch quality gates pass on the internal benchmark; critical privacy/security review items are closed; accepted jobs and edits are durable.

### Phase 2: Private beta

**Scope additions:** Broader source coverage, all note-depth presets, language controls, usage/billing trial, improved diagnostics, read-only sharing.

**Objective:** Validate repeat usage, real-world quality, willingness to pay, and support burden with invited users.

**Exit criteria:** Target activation and completion rates are met; quality trend is stable; unit economics support the proposed packaging; no unresolved critical rights or security issues.

### Phase 3: Public web launch

**Scope:** Production-ready responsive web product, paid individual plan, support operations, rights-holder workflow, published limits, and reliability monitoring.

**Objective:** Establish the core growth and retention loop.

**Exit criteria:** Stable operations at planned load, measurable retention, and an evidence-based mobile/desktop priority list.

### Phase 4: Mobile and desktop

**Mobile focus:** Capture, uploads, notifications, reading, search, and lightweight editing.  
**Desktop focus:** Drag-and-drop, multi-pane study, advanced editing, keyboard workflows, and offline reading.

**Exit criteria:** Users can start on one platform and successfully continue on another; platform-specific crash, sync, and usability targets are met.

### Phase 5: Learning expansion

Candidates, prioritized by evidence:

- Grounded source Q&A.
- Flashcards, quizzes, and spaced review.
- Section-level rewrite and version comparison.
- Visual understanding and generated diagrams.
- Collections, course organization, collaboration, and educator workflows.
- Approved external enrichment with separate citations.

---

## 17. MVP definition

The public MVP includes:

- Responsive web experience.
- Account, authentication, basic settings, and cross-device cloud library.
- Public YouTube link, supported hosted/direct link, and local file upload.
- Preflight validation and actionable errors.
- Durable background processing with clear stages, cancellation, retry, and in-app notification.
- Transcript/time alignment and structured note generation.
- Quick, Detailed, and Comprehensive presets; audience, language, and focus controls.
- Editable notes with outline, source player, transcript search, and timestamp links.
- Library search, sort, rename, archive, and delete.
- Markdown and PDF export.
- Regeneration with non-destructive version handling.
- Quality feedback and abuse/rights reporting.
- Usage visibility and paid-plan foundations.
- Privacy controls, source deletion behavior, operational support, and product analytics.

### 17.1 Explicitly deferred from MVP

- Native mobile and desktop clients.
- Real-time collaboration and team administration.
- Grounded chat/Q&A.
- Flashcards, quizzes, and spaced repetition.
- External web research or enrichment.
- Batch processing.
- Full offline mode.
- Advanced visual understanding for every video type.
- AI-generated diagrams.
- Institution and minor-specific product flows.

---

## 18. MVP acceptance criteria

The MVP is ready for public use when all of the following are true:

1. A user can submit each advertised input type and receives either a valid source preview or a specific, actionable error.
2. An accepted source becomes a durable library item and its job survives closing the browser.
3. A successful job produces a structured, editable note in the chosen language and depth.
4. Important note sections contain functioning timestamp links whenever the source provides usable timing.
5. A timestamp click opens or seeks supported playback near the relevant evidence.
6. The output passes agreed grounding, coverage, clarity, and citation thresholds on the launch benchmark.
7. The product communicates degraded confidence for poor audio, missing evidence, or unsupported visual context.
8. User edits autosave and are not silently overwritten by regeneration or another device.
9. Users can search their library and notes, export Markdown/PDF, and delete an artifact.
10. Failed or canceled jobs explain their status, preserve recoverable work, and follow the published usage-refund policy.
11. Users can understand current usage, plan limits, retention, and source-deletion behavior before committing paid capacity.
12. Keyboard and screen-reader testing covers the complete core flow, and accessibility review has no unresolved critical issues.
13. Security, privacy, legal/platform compliance, and abuse-readiness reviews have no unresolved critical issues.
14. Support can identify a job using non-sensitive metadata, investigate failure, and respond without routine access to private content.
15. Product dashboards report the primary funnel, quality, reliability, cost, and guardrail metrics.

---

## 19. Risks and mitigations

| Risk | Impact | Product mitigation |
|---|---|---|
| Notes sound convincing but contain unsupported claims | Loss of trust and possible harm | Source-first generation, timestamp citations, coverage/grounding checks, low-confidence states, feedback loop |
| YouTube or host access changes | Core ingestion disruption | Compliant provider strategy, explicit support matrix, graceful errors, upload fallback, avoid dependence on unapproved extraction |
| Long videos are slow or expensive | Poor UX and unit economics | Preflight estimate, background jobs, tiered depth, plan limits, measured packaging, partial recovery |
| Visual information is missed | Incomplete technical notes | Disclose modality limits, evaluate visual understanding separately, allow correction, add bounded visual processing in P1 |
| Upload instability on mobile | Failed activation | Resumable flow, background-aware state, keep library job, smaller supported limits until proven |
| Users confuse source claims with objective truth | Misinformation | Prominent source attribution, neutral phrasing, ambiguity handling, no implied verification of the speaker |
| Private or copyrighted material is mishandled | Legal and trust damage | Rights confirmation, compliant access, minimal retention, private-by-default artifacts, takedown workflow |
| Cross-device edits conflict | Data loss and frustration | Visible sync state, deterministic conflict strategy, recoverable versions |
| Too many features dilute note quality | Weak core value | Gate roadmap on core quality and meaningful study metrics |
| Multilingual support is nominal but poor | User harm and churn | Language-by-language evaluation and advertised support based on quality thresholds |
| Agentic workflow follows malicious source instructions | Security/privacy incident | Treat all source media as untrusted data; isolate processing and prevent source-driven tool access |
| “Beautiful” output becomes inconsistent or inaccessible | Poor usability | Structured document system, constrained components, theme and accessibility review, export fallbacks |

---

## 20. Assumptions and dependencies

### 20.1 Assumptions

- The initial target user is an individual learner or professional, not a managed school account.
- Users prefer a strong default artifact over constructing prompts.
- Source-linked trust is a meaningful differentiator.
- Users accept asynchronous generation for longer videos if status and notifications are reliable.
- The product can lawfully access at least one useful path for public URLs and user-owned uploads in launch markets.
- A cloud account is acceptable for the initial cross-platform experience.

### 20.2 Dependencies to validate

- Rights-compliant access to metadata, media, captions, or embeds for supported providers.
- Transcription and generation quality across launch languages and content types.
- Predictable cost and latency for the allowed duration range.
- Reliable media upload/storage/processing and deletion lifecycle.
- Cross-platform identity, subscription entitlement, notification, and sync behavior.
- Formula, code, table, citation, and export fidelity.
- Legal review for terms, privacy, copyright, takedown, target age, and regional launch.
- Support and abuse-response processes before public access.

---

## 21. Open product decisions

These questions should be resolved through discovery, prototyping, evaluation, or legal review before the relevant phase:

1. Which learner segment and subject category should be the initial beachhead?
2. What video duration and file size limits produce acceptable completion time and economics?
3. Which hosted-video providers and direct formats can be supported compliantly at launch?
4. Which languages meet the quality bar for transcription, generation, and translation?
5. Should source media be deleted automatically by default or retained briefly to enable regeneration?
6. Which preset produces the best first-run activation: Quick or Detailed?
7. How should usage be packaged: minutes, hours, videos, or plan-specific fair use?
8. Which exports are essential beyond Markdown and PDF for the target segment?
9. What level of visual-frame analysis is required for the first target subject category?
10. When should a transcript be hidden because the provider or rights context does not permit displaying it?
11. Should anonymous users receive a small preview before account creation, and how is abuse controlled?
12. What quality and latency thresholds define each advertised input type and language?
13. Is offline reading required for the first mobile/desktop release?
14. What data may be used for product improvement, under which consent and retention terms?
15. Which countries launch first, given payments, privacy, copyright, and target-age requirements?

---

## 22. Inputs required for the TRD/HLD

The technical design should begin after the following product decisions are provisionally set:

- Launch platform sequence and supported client versions.
- Supported source providers, file formats, duration/size limits, and rights constraints.
- Launch languages and expected visual-understanding scope.
- Note document schema, citation/provenance model, versioning expectations, and export formats.
- Raw media, transcript, note, analytics, backup, and deletion policies.
- Account, sharing, subscription, and entitlement rules.
- Target generation latency, availability, recovery, and synchronization expectations.
- Benchmark dataset, evaluation rubric, launch thresholds, and production feedback loop.
- Initial traffic, storage, processing-volume, and cost assumptions.
- Security, privacy, moderation, audit, and regional compliance requirements.

The TRD/HLD should then cover system boundaries, ingestion adapters, media pipeline, agent orchestration, model/provider strategy, grounding and citation design, durable jobs, storage and lifecycle, API contracts, document sync, client architecture, observability, evaluation infrastructure, security controls, deployment, scaling, disaster recovery, and cost controls.

---

## Appendix A: Core user stories

- As a learner, I want to paste a YouTube link and get detailed notes so that I can revise without replaying the entire video.
- As a learner, I want every important idea linked to a timestamp so that I can verify context quickly.
- As a beginner, I want unfamiliar concepts explained simply so that the note matches my level.
- As an advanced learner, I want formulas, code, caveats, and reasoning preserved so that the note is not oversimplified.
- As a mobile user, I want to send a video link from another app and be notified when notes are ready.
- As a desktop user, I want to drag in a local video and edit the result in a focused workspace.
- As a returning user, I want my library and edits on every device so that I can continue where I stopped.
- As a privacy-conscious user, I want to know when my source file is deleted and to delete all derived data.
- As a user with a failed job, I want a clear explanation and retry path without losing allowance unfairly.
- As an educator, I want to refine and export notes from my own recording so that I can share a polished handout.

## Appendix B: Representative edge cases

- The URL points to a playlist, short-form video, live stream, scheduled stream, private video, or removed content.
- The hosted URL is an HTML page rather than a media file, expires during processing, redirects, or requires cookies.
- The file has video but no audio, multiple audio tracks, variable frame rate, corruption, or a misleading extension.
- The source language changes mid-video.
- Speakers interrupt each other or quote a third party.
- A lecture references a slide that is unreadable or never shown.
- Code is shown but not spoken; formulas are spoken but not displayed.
- The video contains long silence, music, sponsorship, repeated material, or unrelated sections.
- The user submits the same source twice with different settings.
- The source is deleted after notes are generated.
- The user edits while another device is offline.
- Subscription allowance expires during an active job.
- The user cancels during upload, transcription, or final formatting.
- A rights holder requests removal of shared artifacts derived from a source.
- A transcript contains instructions intended to manipulate the processing agent.

## Appendix C: Glossary

- **Artifact:** The saved combination of source metadata, transcript where allowed, generated note, user edits, citations, settings, and versions.
- **Citation:** A link from note content to supporting source evidence, usually a timestamp or time range.
- **Grounded:** Supported by and attributable to the supplied source or to a separately labeled external source.
- **Hosted video URL:** A permitted URL from which video media can be accessed without bypassing authentication, DRM, paywalls, or platform restrictions.
- **Meaningful study action:** A behavior indicating use beyond opening the result, such as reading substantially, following a citation, editing, searching, exporting, or returning.
- **Preset:** A product-defined generation configuration such as Quick, Detailed, or Comprehensive.
- **Source:** The video and associated permitted metadata, audio, captions, or transcript supplied for processing.

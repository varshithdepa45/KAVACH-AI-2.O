# KAVACH AI Project Report

Complete product and technical documentation

Implementation reviewed 7 November 2026

## Executive summary

KAVACH AI is a local document intelligence website for reading, reviewing, searching, and discussing user-supplied documents. A user uploads a file, chooses an analysis mode and domain profile, and receives results that can be inspected alongside the source. The application keeps document records, analysis history, review decisions, batch progress, and generated reports on the configured local backend.

The website supports risk review, document summaries, literal field extraction, a searchable Knowledge Base, document chat with citations, and highlighted evidence. Several files can be uploaded and analyzed as a batch. An optional Python Sandbox runs code that the user explicitly submits, separately from document analysis.

The project combines a Next.js frontend, a FastAPI backend, SQLite persistence, local document extraction and OCR, and configured local AI models. Its main use case is helping a person understand internal reports, invoices, contracts, technical records, study material, and administrative files while retaining a path back to the evidence.

Source verification checks citation support. It does not establish that a document or an AI answer is factually correct. The current application is designed for a trusted local workstation; broader deployment needs additional access, network, and storage controls.

### How to read this report

The opening chapters explain the product, pages, architecture, analysis pipeline, evidence, batches, chat, storage, operation, and limitations. The technical appendices document requirements, source-code organization, the database, retrieval algorithms, planner tools, configuration, API routes, deployment, troubleshooting, and evaluation. Use the PDF bookmarks to jump between chapters.

### Report scope

This report describes the implementation in the project workspace, including the recently added batch, chat, and evidence features. Examples are illustrative unless identified as recorded test fixtures. It is a product and technical explanation, not an independent security audit or an accuracy benchmark.

Source basis S1 S2 S11

<!-- page -->
## Project purpose and scope

### The problem being addressed

Useful information is often spread across long documents, scanned pages, and separate records. A reviewer may need to identify dates, amounts, obligations, or risk-related statements, then explain where those details came from. Sending internal files to an external AI service may also conflict with the user's intended handling of those documents.

KAVACH AI brings file storage, local processing, source inspection, and report generation into one workflow. The user remains responsible for checking results and making decisions. The project does not claim that every possible risk can be detected or that model output is always correct.

### Main objectives

- Read supported digital documents and use local text recognition when scans need it.
- Provide useful analysis choices without requiring users to operate backend tools directly.
- Keep findings and fields linked to source excerpts and locations.
- Make agent actions, processing progress, errors, and review requirements visible.
- Preserve analysis history and generate downloadable files.
- Support several documents through batches, retrieval, and document chat.

### Supported inputs and analysis modes

The upload control accepts PDF, Word DOCX, plain text TXT, CSV, PNG, JPG, JPEG, and Python PY source files. The frontend allows up to 25 MB per file; the backend also enforces its configured upload limit. A Python upload is stored as a document and is not automatically executed.

| Mode | What it does | What the user should check |
| --- | --- | --- |
| Risk review | Matches the selected profile's risk vocabulary against source text and records supporting excerpts. Local model reasoning can add advisory commentary. | Whether the matched passage is relevant and actually describes an issue in context. |
| Summary | Uses the configured local text or vision model to summarize or describe the input. | The original document and any omissions or unsupported model statements. |
| Extract fields | Finds literal dates, amounts, contacts, owners, reference identifiers, and measurements with source locations. | That each extracted value and its meaning match the original. |

### Intended users and document contexts

Eight built-in profiles cover everyday documents, industrial engineering, business and projects, education and research, finance and procurement, contracts and policies, healthcare administration, and people and HR. Profiles guide document context and review vocabulary. These profiles do not make the application a qualified legal, medical, financial, or engineering decision maker.

Source basis S1 S3 S7 S12

<!-- page -->
## Website pages for document work

### Overview and landing page

The home page introduces the project, explains the analysis modes, shows a sample invoice workflow, and provides a guide to the workspace tools. Profile cards open Workbench with a suggested mode and request. The workspace snapshot loads actual stored-document and recorded-run counts and links to recent activity. Missing backend data is shown as unavailable rather than replaced with sample statistics.

### Document Vault

The Vault is the starting point for source files. Users can upload one or more files, inspect per-file upload outcomes, preview supported originals, download them, and select documents for batch analysis. The table shows document metadata and available processing states. Single-document analysis remains available alongside batch processing.

Deleting a document requires confirmation in the website. Deletion removes the uploaded source and excludes it from future knowledge searches, while existing run history and reports remain. Deletion is blocked during active analysis, unfinished batches, and source-pinned chat requests. Retained history cannot open an original that has been deleted.

### Workbench

Workbench lets users select a stored source, domain profile, analysis mode, and editable request. The normal Analyze document action starts a persisted background job. The optional bounded planner uses a restricted set of actions to organize the task. A stage display exposes actual work such as extraction, retrieval, reasoning, verification, and report saving.

After completion, Workbench shows model output where applicable, source matches or extracted fields, review controls, and report downloads. A saved job link or browser refresh reconnects to the same recorded job rather than automatically creating another analysis.

### Knowledge Base

This page searches readable text from stored documents. It shows indexed sources, relevant passages, page information, and retrieval scores. Search results can open highlighted source previews. A retrieval score describes similarity in the configured index; it is not a factual confidence score.

### Document Chat

Users select up to 20 stored documents and ask questions or follow-ups. Retrieval and generation are scoped to that selection. A displayed answer includes known source markers that open the supporting passage. Missing evidence or invalid citation markers produce an insufficient-evidence response. Chat history stays in the current browser tab, clears on refresh, and resets when the selected sources change.

Source basis S1 S2 S4 S5 S6

<!-- page -->
## Website pages for review and operation

### Agent Runs

Agent Runs lists recorded analyses and their statuses. Opening a run exposes its execution trace, findings, source excerpts, available model output, and generated files. Evidence buttons open the highlighted source viewer. Findings held for review can be approved or rejected. Review decisions are saved and the current run state is returned when results are reopened.

### Deliverables

This page lists reports that actually exist for saved analyses and provides download links. Text and Markdown reports are standard outputs. Word DOCX, Excel XLSX, and PDF exports depend on the optional generation libraries available on the local installation. The website does not substitute a fabricated report when a file is missing.

### Model Router

Model Router explains the configured provider and local model assignments for reasoning, vision, and other tasks. Routing uses rules and configuration. A configured assignment does not prove that a model is loaded, healthy, accurate, or fast. Actual inference failures are surfaced when the model is called.

### Security Center

Security Center displays configured processing policy, code-execution readiness, and the local audit log. Audit events record activities such as uploads, analyses, reviews, and downloads. The page explains that application restrictions must be supported by host-level controls when the system is deployed more widely.

### System

System shows available local CPU, memory, GPU, VRAM, and service information. GPU history is built from actual measured samples when a supported reading exists. Missing measurements are labeled unavailable. Hardware readings help explain the workstation's state; they do not establish model quality or guarantee processing time.

### Python Sandbox

The sandbox is an optional, separate tool for Python code explicitly submitted by the user. Each execution uses a fresh restricted Linux container and returns standard output, standard error, exit status, and timing. The UI also shows whether the required local runtime is ready.

Uploading a document, analyzing an image, or starting the planner never automatically runs uploaded Python code. The sandbox has its own execution policy and runtime prerequisites. It can be disabled without removing the document-analysis workflow.

### Navigation and visual behavior

The frontend retains a shared workspace sidebar and top bar on internal pages. The landing page has its own introduction and responsive navigation. Existing fonts, paper tones, forest green, and orange accents provide a consistent identity. Keyboard focus indicators and reduced-motion preferences are preserved in the interface.

Source basis S1 S2 S8 S9 S10

<!-- page -->
## System architecture and technology

The browser sends requests to the local FastAPI backend, which coordinates validation, document services, jobs, agents, retrieval, inference, and reports. SQLite preserves records and processing state. Source files and generated reports remain in local directories.

[[ARCHITECTURE]]

### Main technology choices

| Layer | Implemented technology | Responsibility |
| --- | --- | --- |
| Frontend | Next.js 14, React 18, TypeScript | Pages, navigation, forms, job progress, evidence views, chat, and downloads. |
| Styling | Tailwind CSS, custom CSS, Lucide icons | Responsive layouts, shared components, visual states, and accessible controls. |
| API | Python, FastAPI, Pydantic, Uvicorn | Request validation, local HTTP endpoints, service coordination, and error responses. |
| Persistence | SQLite and local directories | Document metadata, jobs, batches, runs, reviews, audit records, originals, and reports. |
| Document processing | pypdf, python-docx, Pillow, Tesseract through pytesseract, PyMuPDF | Text extraction, Word text and tables, OCR, and source-page rendering where installed. |
| Inference and retrieval | Ollama or local Transformers; lexical or provider embeddings | Local model generation and retrieval of relevant source passages. |

### Model roles and deployment boundary

Default Ollama assignments are Qwen3 8B for reasoning, LLaVA 7B for vision, and Llama 3.2 3B for other tasks and planner choices. Configuration can change these assignments. The Transformers adapter uses local model directories and is text-only. Literal field extraction and profile-term matching also use deterministic Python routines.

Normal runtime processing is designed around local services. Models and dependencies need to be provisioned before an offline installation is used. The application's airgapped setting is an application policy; network isolation still depends on deployment controls.

Source basis S1 S8 S10 S12

<!-- page -->
## How a document analysis works

The normal pipeline separates document preparation, inference, source checks, and report saving. The user can inspect progress rather than seeing only a final answer. A stage can be completed, skipped, warned, or failed according to the operation performed.

| Stage | Actual work performed |
| --- | --- |
| Check document | Confirm that the stored source and requested settings are usable. |
| Extract text and OCR | Read embedded or plain text. For sparse scanned PDF pages and raster images, attempt local OCR when available. Preserve page and location information. |
| Plan analysis | Apply provider policy, choose configured model roles, and prepare the request and document context. |
| Inspect images | For PNG, JPG, and JPEG analysis inputs, call the configured local vision model. Visual observations are kept separate from OCR-based fields. |
| Retrieve context | Split readable source text into passages, compute local vectors, and retrieve passages relevant to the task. |
| Analyze content | Run the selected mode. Summary uses inference; field extraction and risk-term review preserve literal source support. |
| Verify sources | Check excerpt and location support and determine which findings need human review. |
| Save results | Persist the run, trace, findings, evidence, and generated report files. |

### Specialized agents

The pipeline coordinates security, routing, document, vision, knowledge, reasoning, verification, and deliverable agents. Each component handles a defined responsibility. Their names describe software roles within the application, rather than independent people or unrestricted autonomous systems.

### Optional bounded planner

The planner is an alternative to following only the normal pipeline sequence. It requires a stored document, selects from allowlisted actions, and operates under step and loop limits. Unknown tools and invalid arguments are rejected. Its tools do not expose a general shell, arbitrary SQL, or unrestricted network access.

The planner can request document inspection, retrieval, reasoning, verification, and human review. The saved run records the chosen actions and stop reason. Findings that need approval remain held until the reviewer decides; a document with no matched findings can finish normally.

### Failure behavior

A failed summary or image inference returns an explicit error. For risk review, unavailable advisory reasoning can be recorded as a warning while supported source-term review continues. Neither a missing match nor an empty extraction establishes that a document has no problems.

Source basis S3 S7 S8

<!-- page -->
## Evidence highlighting and human review

### What evidence means in this project

A finding or field can include the source filename, page, quoted excerpt, extraction method, character span, and available OCR confidence and bounding boxes. These details allow a reviewer to inspect how a result relates to the uploaded document. Confidence and source verification describe extraction heuristics and citation support, not factual certainty.

### Opening a highlighted source

1. Select View highlighted source beside a finding, Knowledge Base result, or chat citation.
2. The backend resolves the stored document and source page or text section.
3. It checks that the excerpt matches the current extracted source text. Stored character spans are used when they still match; otherwise the exact excerpt is located in the source.
4. The viewer highlights the passage in the extracted text and shows surrounding context.
5. For PDF or raster images, the viewer also shows a rendered source page with available text or OCR locations overlaid. The original can be downloaded for inspection.

Word and plain-text sources use extracted text sections rather than a reproduction of the original printed layout. Word text and table content can be inspected, but formatting and embedded images should be checked in the original file.

### Source changes and missing originals

The viewer reports when the saved excerpt no longer matches the source or the original is unavailable. A deleted source is not silently rebound to a newly uploaded file with the same name. Older findings without a stored document link may require a new analysis before highlighting is available.

If a page cannot be rendered or its visual location cannot be found, verified extracted text may still be available. The UI distinguishes this fallback from a visual highlight on the original page.

### Human review decisions

Review requirements can arise from high-risk severity, absent or unknown evidence, low confidence relative to configuration, or missing verification. These checks flag findings for a person to inspect. The reviewer can approve or reject a held finding, and the application records the decision and updates the run state when all required decisions are complete.

Approval is a workflow decision, not an independent certification that the source or conclusion is correct. Model summaries and image observations remain advisory. Reviewers should read the actual passage and its context before relying on a result.

Source basis S3 S6 S7 S10

<!-- page -->
## Batch uploads and analysis

Batch processing applies shared analysis settings to several stored documents while keeping the result for each source separate. Uploading files and starting their analyses are distinct actions, so users can inspect upload outcomes before requesting processing.

### User workflow

1. Choose or drop up to 50 files in Document Vault. Each file is uploaded individually and receives a stored or failed outcome.
2. Successful uploads are selected in the Vault table. Users can adjust the selection or choose existing sources.
3. Choose the shared domain profile and analysis mode in the analysis settings.
4. Select Analyze selected. The backend admits the batch and saves its items before scheduling work.
5. Track waiting, running, completed, and failed items in the Batch analysis panel. Each source receives a job and, when processing creates one, a saved run.
6. Open an individual run or Workbench job to inspect progress, findings, review needs, and report downloads.

### Capacity and persistence

| Control | Current behavior |
| --- | --- |
| Upload selection | Up to 50 files at a time, with a 25 MB frontend limit per file. |
| Analysis batch | Up to 50 distinct stored documents. |
| Active batches | Up to three admitted batches at a time. |
| Analysis execution | Two worker threads and four admitted analysis jobs; batches feed this same queue. |
| Full queue | Pending batch items wait for capacity. Direct job admission can return HTTP 429. |
| Duplicate submission | A repeated batch request identifier with the same payload returns the existing batch; a conflicting payload is rejected. |

### Partial failures and recovery

One failed document does not remove successful results from the batch. The batch can finish as completed with errors, and each failed item retains its message. Refreshing the browser reloads saved batch history and does not automatically create a second batch.

After a backend restart, batch items that were still pending can be scheduled. Analyses already admitted when interrupted are marked as errors under the existing job recovery policy. They require an explicit retry; the application does not claim to resume unfinished model inference.

Sources in unfinished batches cannot be deleted. This protects files needed by later items and makes a batch's lifetime separate from whether its browser tab remains open.

Source basis S1 S4 S10

<!-- page -->
## Document chat and local retrieval

Document Chat lets a user ask questions across selected files and continue with follow-ups. It works from extracted text. Direct visual understanding of images remains part of document analysis in Workbench; chat needs readable text or OCR output from the selected files.

### How an answer is produced

1. The user selects between one and 20 stored documents and enters a question.
2. The backend validates the selection and pins those sources against deletion during the request.
3. Readable pages are divided into overlapping passages. The default backend computes local lexical hash vectors; a configured provider encoder can be selected instead.
4. The current question and limited prior user context retrieve relevant passages from the selected sources. The service supplies up to six positive-similarity passages to generation.
5. The configured local reasoning model receives the question, limited conversation context, and numbered source passages. The prompt treats document contents as untrusted data and asks for an evidence-limited answer.
6. The response is checked for known numeric source markers. If markers are missing, invalid, or absent from a response paragraph, the application returns an insufficient-evidence message rather than displaying that answer.
7. Accepted markers open highlighted source previews. The user can inspect the actual cited passage and original.

### Search and generation have different roles

Knowledge Base search returns similar source passages. Chat uses retrieved passages to ask a local model for an answer. Embeddings are numerical representations used to compare passages with a query; they do not prove a statement is true. The default lexical method needs no learned embedding model, while a provider embedding configuration requires its local encoder to work.

### Conversation boundaries

The frontend keeps the conversation in memory in the current tab. Refreshing the page clears it, and changing sources starts a new conversation. A bounded recent history is sent for follow-ups. Conversation text helps interpret a question and is not treated as source evidence.

Only one chat generation is admitted at a time. A missing model produces an explicit availability error. No retrieved evidence produces an insufficient-evidence response without inventing an answer.

### What citation checking does not establish

The acceptance check validates marker presence and allowed source numbers. It does not establish that every claim logically follows from the passage. A model can still misinterpret a source, and OCR or retrieval can miss material. The website therefore presents answers as advisory and keeps the evidence accessible.

Source basis S1 S5 S6 S8

<!-- page -->
## Data storage and API organization

### Persistent and temporary state

| Data | Default location or mechanism | Lifetime |
| --- | --- | --- |
| Uploaded originals | `data/uploads/` | Remain until explicitly deleted or managed externally. |
| Generated reports | `generated/` | Retained separately from deleted source files. |
| Structured records | `backend/kavach.db` using SQLite | Persists document metadata, tasks, runs, findings, evidence, reviews, audit entries, jobs, and batches. |
| Retrieval vectors | In-memory local index | Rebuilt from stored readable documents when the source signature changes or the process restarts. |
| Chat conversation | Current browser tab | Clears on page refresh or source selection change. |
| Sandbox working files | Disposable container storage | Discarded after the explicit code execution. |

Paths and providers can be changed through configuration. A local backend can retain documents and reports without a cloud database. The in-memory retrieval index should not be confused with a separately deployed persistent vector database.

### Representative API groups

| Endpoint group | Purpose |
| --- | --- |
| `/api/health` and `/api/system/status` | Service identity, configured mode, hardware, and service state. |
| `/api/documents` and `/api/documents/upload` | Stored sources and uploads; individual routes also provide previews, originals, deletion, and analysis. |
| `/api/analysis/jobs` | Admit an asynchronous document analysis and inspect its durable progress. |
| `/api/analysis/batches` | Admit or list batches and retrieve per-document outcomes. |
| `/api/knowledge` and `/api/knowledge/search` | Index information and local passage search. |
| `/api/chat` | Questions and bounded follow-up context scoped to selected document IDs. |
| `/api/evidence/{id}/preview` | Resolve a saved finding's excerpt and source highlight. |
| `/api/agents/runs` and `/api/review` | Analysis history, execution details, and human decisions. |
| `/api/deliverables` and `/api/sandbox` | Report downloads and explicitly requested code execution. |

### State and error semantics

The backend validates payloads with Pydantic and returns explicit errors for missing documents, unavailable sources or models, conflicting operations, invalid settings, and full queues. Jobs and batch items keep their own statuses so a browser disconnect is not treated as a reason to start duplicate work. Older synchronous analysis endpoints remain available for compatibility.

Source basis S1 S4 S5 S10

<!-- page -->
## Local setup and daily operation

### Required services and optional capabilities

The repository's setup guidance uses Python 3.10 or later and Node.js 18 or later. Install the frontend and backend dependencies, provision the configured local models, and run the backend and frontend. Available hardware and model size affect practical inference speed and memory use; this report does not claim a universal hardware minimum or response time.

Tesseract must be installed separately for local OCR. PDF text extraction, scanned-page rendering, Word reading, and additional report formats depend on their optional packages. Installing core API dependencies alone does not provide every analysis or export capability.

### Typical startup commands

Run the backend from the `backend` directory:

```text
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

For the already configured native sandbox setup, the environment file can be loaded explicitly:

```text
python -m uvicorn app.main:app --env-file .env.sandbox --host 127.0.0.1 --port 8000
```

Run the frontend from the `frontend` directory:

```text
npm run dev
```

For a production frontend build, use `npm run build`, followed by `npm run start`. The default website URL is `http://localhost:3000`; the local backend uses port 8000. Build-time `NEXT_PUBLIC_API_BASE` must point to the backend that the browser can reach. Run a single backend process for the current workstation queue design.

### Checking readiness

Use System to inspect available service and hardware readings. Use Model Router to inspect assignments, then verify actual inference through an analysis or chat request. Confirm that required model files are already provisioned for offline operation. An unavailable backend or model should be corrected rather than interpreted as an empty analytical result.

### Common operating situations

- A scan returns no readable text: check OCR availability and scan quality.
- A model request fails: check the configured provider, model files, and local runtime.
- A direct job receives HTTP 429: wait for capacity in the local queue.
- A file cannot be deleted: wait for its active analysis, batch, or chat request to finish.
- A citation cannot be opened: confirm that the original still exists and matches the saved excerpt.
- A report format is absent: use an available format or provision the optional exporter.

Source basis S1 S8 S9 S10 S12

<!-- page -->
## Security boundaries and current limitations

### Implemented application controls

Uploads are restricted by extension and size, filenames are sanitized, and document serving checks source paths. Airgapped provider policy blocks configured external providers and limits local inference URLs to loopback hosts. Planner actions are allowlisted and have bounded execution. Source pinning prevents deletion during active operations, and activity records are stored locally.

The Python Sandbox is separate from document analysis. Its fixed policy disables container networking, avoids host mounts, uses a read-only root filesystem, runs the untrusted worker as a non-root user, and limits resources. Defaults include one CPU, 128 MB RAM, 32 processes, ten seconds, 64 KB combined output, and a disposable 16 MB temporary area. Its runtime must be provisioned separately, and shared-kernel isolation still depends on a maintained host.

### Deployment limits

The current system is intended for a trusted local workstation. Application policy is not a substitute for authentication, authorization, transport security, disk encryption, backups, or host-level egress restrictions. User accounts, role-based multi-user isolation, and a complete collaboration workflow are not implemented. Wider exposure requires those controls to be designed and tested.

### Analysis and retrieval limits

- Risk review finds configured source-term patterns. It is not a comprehensive semantic or regulatory risk assessment.
- Field extraction targets supported literal formats. OCR mistakes, unusual layouts, languages, and formatting can affect results.
- Summaries and visual descriptions can omit or misinterpret information.
- A known citation marker is not a proof that the cited passage supports every generated claim.
- The default lexical index has different retrieval behavior from a learned semantic encoder.
- Chat history is not persisted across refreshes, and chat does not perform independent visual image reasoning.
- Restart recovery preserves status and pending batch work; it does not resume interrupted model inference.

### Further improvements

Practical next enhancements include user accounts and roles, document version comparison, collaborative reviewer comments and assignments, persistent chat sessions with explicit retention controls, backup and recovery, and a quality dashboard based on labeled evaluation data. These are future directions. Evidence highlighting, batch processing, and source-cited document chat are already implemented.

Source basis S1 S3 S4 S5 S7 S9

<!-- page -->
## Demonstration plan and recorded verification

### Suggested demonstration sequence

1. Explain the home page: the project helps review private documents with source evidence and local processing.
2. Upload an invoice or text record in Document Vault. Show the upload outcome and original preview.
3. In Workbench, select the finance profile and Extract fields. Run the analysis and explain the real stage progress.
4. Open a field's highlighted source, compare it with the original, and show its saved run and available report downloads.
5. Upload or select a second source and start a batch. Show separate item outcomes and explain that one failure does not discard successful work.
6. Open Document Chat, select the relevant files, ask a question, and follow it with a question referring to the previous answer. Open a citation.
7. Search the Knowledge Base, then briefly show Model Router, Security Center, and System. Demonstrate the Python Sandbox separately if its runtime is ready.

Use documents that contain the details being demonstrated. Avoid describing an empty match list as proof that the input is safe or complete.

### Recorded implementation checks

| Check | Recorded result | What it establishes |
| --- | --- | --- |
| Targeted backend regression run | 111 tests reported in the earlier feature implementation session; this is a targeted historical run, not the total suite. | Covered features and related document, OCR, planner, persistence, and failure paths under test conditions. |
| Frontend build | Next.js production build and type checking passed. | The updated frontend compiled, including the chat route and batch and evidence components. |
| Local HTTP checks | Updated pages and new batch and chat API routes responded as expected. | The running frontend and backend exposed the integrated routes. |
| Real local inference smoke check | Two fixture sources completed as a batch; four source highlights matched; Qwen3 8B answered and handled a follow-up with citations. | The real configured model participated in an isolated end-to-end example. |

The recorded smoke source stated that a supplier agreement's renewal deadline was 2030-01-12. The real model returned that date with a source marker and correctly connected a follow-up question to the renewal deadline. This was synthetic test content, not a user or customer record.

These checks are implementation evidence, not a benchmark of general document accuracy. Most unit tests isolate inference with test-only doubles. The separate smoke check used actual Ollama inference. Browser interaction and visual UI checks were unavailable in the implementation environment; this report does not claim they were completed. The checks above were recorded during implementation, not rerun merely to write this report.

Source basis S11

<!-- page -->
## Glossary and implementation references

### Essential terms

| Term | Meaning in KAVACH AI |
| --- | --- |
| OCR | Optical character recognition converts scanned or photographed text into readable text. |
| Local inference | AI generation performed by the configured local model runtime. |
| Profile | Domain context and vocabulary used for a document task. |
| Analysis mode | The requested operation: risk review, summary, or field extraction. |
| Agent and planner | A software component with a defined role; the optional planner chooses among allowed actions. |
| RAG and embedding | Retrieval-augmented generation supplies source passages to a model; embeddings represent text for similarity search. |
| Evidence and verification | A source excerpt and location, plus checks of citation support. |
| Job and batch | A persisted document analysis; a group of documents scheduled through the same bounded queue. |
| Sandbox | A restricted, temporary environment for code explicitly submitted by a user. |

### Source index

All paths below are relative to the project root. They identify implementation sources used for this report; they are not external product endorsements.

- S1 `README.md` provides setup, feature behavior, operating scope, and deployment qualifications.
- S2 `frontend/app/` and `frontend/components/shell/nav.ts` define the website pages and navigation.
- S3 `backend/app/services/analysis.py` and `document_analysis.py` implement extraction and analysis behavior.
- S4 `backend/app/services/analysis_jobs.py` and `batch_jobs.py` implement queues, durable progress, and recovery.
- S5 `backend/app/services/document_chat.py`, `knowledge.py`, and `backend/app/rag/pipeline.py` implement scoped chat and retrieval.
- S6 `backend/app/services/evidence_preview.py`, `document_store.py`, and `frontend/components/vault/` implement source access and highlighting.
- S7 `backend/app/agents/` contains orchestration, planner actions, progress, tools, and human review rules.
- S8 `backend/app/models/` and `backend/app/config.py` define local providers, routing, and defaults.
- S9 `backend/app/security/` and `docker/sandbox/` define application guards and sandbox policy.
- S10 `backend/app/database/db.py`, `api/routes.py`, `schemas/schemas.py`, and `services/deliverables.py` define persistence, APIs, validation, and reports.
- S11 `backend/tests/test_document_features.py`, related regression tests, and `backend/scripts/verify_document_features.py` support validation. Recorded real-model evidence is in `tmp/document-features-gr9fk5na/result.json`.
- S12 `frontend/package.json` and `backend/requirements.txt` identify the implemented stack and dependencies.

### Project explanation in one paragraph

KAVACH AI is a local document intelligence workbench. It reads uploaded documents, uses selected analysis modes and configured local models, retrieves relevant passages, and helps users inspect source-supported results. Users can review findings, highlight evidence, process batches, ask document questions, and download generated reports while retaining control over their local workflow.


<!-- page -->
## Requirements and project design

### Functional requirements

The application must accept supported user files, preserve original bytes for downloads, and maintain an inventory of uploaded sources. Supported extensions are PDF, PNG, JPG, JPEG, TXT, DOCX, CSV, and PY. The configured default upload ceiling is 25 MB per file. A Python upload is document content; uploading it does not execute it.

A reviewer must be able to choose a domain profile and one of three analysis modes. Risk review identifies configured vocabulary in source statements; Summary produces advisory model output; Extract fields captures supported literal values. Image inputs receive vision inspection in each mode. Search and chat must use actual stored document content, and results must expose source information where available.

The application must retain runs, intermediate steps, findings, evidence, decisions, and report metadata. It must show actual job progress, admit work within capacity, and report failures explicitly. Batch submission must preserve individual outcomes. Source deletion must respect active processing and preserve historical runs and reports.

### Nonfunctional requirements and tradeoffs

Local processing reduces dependence on external inference services and supports provisioned offline operation. Explicit inference errors preserve visibility when a model is unavailable. A modular service and agent design separates extraction, reasoning, evidence verification, and exports. Optional integrations permit a core API installation without every document or reporting library.

Durable metadata improves recoverability, while the small in-process queue keeps workstation deployment simple. This choice limits horizontal scaling: more API workers are not a supported substitute for a shared queue and coordinated scheduler. SQLite and an in-memory search index suit the current local scope but require reconsideration for large, concurrent organizational deployments.

### Illustrative use case

An employee receives a supplier invoice containing a reference number, a date, an amount, and an unpaid status. They upload the file, select Finance and Extract fields, inspect the cited amount and date, and download a findings matrix. They can separately run Risk review to locate configured risk terms, or select this invoice and another record in Chat to ask a comparison question. This example describes intended behavior and is not a newly executed acceptance result.

Source basis: README.md; backend/app/config.py; backend/app/domains/profiles.py; backend/app/services/analysis_jobs.py.

<!-- page -->
## Source-code organization and module responsibilities

| Location | Responsibility |
| --- | --- |
| frontend/app | Next.js App Router pages, layouts, and route-level interface logic. |
| frontend/components | Shared controls, shell navigation, landing content, progress visualizations, Vault dialogs, and evidence previews. |
| frontend/lib/api.ts | Browser-to-backend request functions and API response types. |
| frontend/lib/types.ts | Shared frontend data contracts. |
| backend/app/main.py | FastAPI app, lifecycle, middleware, routes, and exception handlers. |
| backend/app/api/routes.py | HTTP endpoint adapters and response handling. |
| backend/app/schemas | Pydantic request and response definitions. |
| backend/app/services | Document preparation and storage, job and batch orchestration, chat, knowledge, exports, evidence previews, and status. |
| backend/app/agents | Pipeline agents, run context, planner, allowed tools, review gates, progress, and run persistence. |
| backend/app/models | Provider adapters and deterministic task routing. |
| backend/app/rag | Chunking, embeddings, and in-memory cosine search. |
| backend/app/database | SQLite schema, migrations, transactions, and record helpers. |
| backend/app/security | Upload policy, provider guards, auditing, and sandbox controls. |
| backend/tests and backend/evals | Regression tests and controlled evaluation runners. |
| docker and scripts | Container definitions, sandbox runner, setup helpers, feature checks, and report tooling. |

### Entry points and development environment

The backend starts at app.main:app under Uvicorn. Its lifespan creates runtime directories, initializes schema, marks interrupted analyses, starts batch dispatch, refreshes configured model records, and records startup. Shutdown stops dispatch and workers and records shutdown. Frontend layout and shell components provide shared navigation across pages.

backend/.venv contains installed Python tools and generated activation scripts. Those files are environment artifacts rather than application source. For zsh or bash, activate with source backend/.venv/bin/activate from the repository root; activate.csh is for C shell. Application behavior belongs primarily under backend/app and frontend/app.

Source basis: backend/app/main.py; frontend/app/layout.tsx; frontend/app/layout-shell.tsx; repository module tree.

<!-- page -->
## Database design and record relationships

The persistence layer uses Python's sqlite3 module with short-lived connections and dictionary-like rows. The schema is initialized locally and additive migrations retain compatibility with existing installations. The following are application relationships; the schema does not declare every relationship as an enforced foreign-key constraint.

| Table | Main information and relationship |
| --- | --- |
| documents | Original filename, storage path, type, size, extraction information, and creation time. |
| tasks | User task description and associated run identifier. |
| agent_runs | Execution status, timing, verification summary, and persisted analysis/vision/extraction data added by migrations. |
| agent_steps | Ordered agent activity for a run, including status, detail, and timing. |
| findings | Run-linked title, description, severity, confidence, review reasons, and review status. |
| evidence | Finding/run links, source, page, excerpt, location, extraction method, OCR confidence, and migrated document link. |
| reviews | Finding-linked decision, reviewer label, note, and timestamp. |
| deliverables | Run-linked filename, local path, format, byte size, and creation time. |
| models | Configured provider/model assignments and routing metadata. |
| audit_logs | Action, actor, outcome, configured mode, detail, and timestamp. |
| analysis_jobs | Document-linked request, stage snapshots, result, error, run link, and status. |
| analysis_batches | Durable batch identifier, shared request settings, and state. |
| analysis_batch_items | Batch/document association, unique job identifier, item status, and error. |

### Logical relationship map

One document can be analyzed repeatedly. Each analysis job can link to a run. Each run has ordered steps, findings, evidence, and deliverables. A finding can have several evidence excerpts and review decisions. A batch has many items, and each item links a document to its own analysis job. Stored document links allow evidence previews to reopen the original source.

### Retention and consistency

Original files live outside the database, so database rows and files must be considered together during backup. Source deletion removes the Vault entry and source, refreshes future knowledge lookup, and retains historical analytical records. A retained report is not a guarantee that its original can still be previewed. In-memory vectors are derived state and rebuild from retained files.

Database access enables SQLite foreign-key checking, but application-level lifecycle checks remain important because many schema relationships are represented as identifiers without explicit FOREIGN KEY declarations. Short transactions and limited processing concurrency support the present workstation workload.

Source basis: backend/app/database/db.py; backend/app/services/document_store.py; backend/app/agents/orchestrator.py.

<!-- page -->
## Retrieval and model algorithms in detail

### Text chunking

The retrieval pipeline splits text into overlapping, approximately 480-character chunks, retaining approximately 80 characters of overlap by whole words. It does not use a fixed token-count splitter. Overlap keeps context around chunk boundaries, at the cost of duplicate material and additional vectors. Source metadata includes document identity, page, and source spans when mapped.

### Default lexical embeddings

The default embedding dimension is 256. Text is lowercased and tokenized using an ASCII alphanumeric expression. Each token is hashed with MD5 into a vector bucket with a signed contribution. The vector is L2-normalized. MD5 is used here for deterministic feature placement, not password protection or a security guarantee. Similar tokens increase vector agreement, but bucket collisions can create unrelated matches.

This is real local lexical retrieval without a downloaded embedding model. It is not learned semantic understanding. Synonyms, non-English text, and questions using very different words from the source may retrieve poorly. Switching to provider embeddings requires configuring a local encoder and rebuilding the index in the selected vector space.

### Similarity and search

The in-memory index ranks chunks with cosine similarity: dot(A, B) divided by the product of their vector lengths. Empty vectors score zero. Differing dimensions are rejected. Searching many stored chunks requires scanning the local index rather than querying a distributed vector database; increasing corpus size therefore increases memory use and search work.

Document Chat scopes retrieval to selected document IDs, takes up to six positive-scoring matches, and includes up to 2,000 characters per cited excerpt. Its retrieval query can include the last two user questions, limited to a bounded query length, to support follow-ups. Retrieved passages become evidence context for generation.

### Model output constraints

The chat prompt tells the local model to use supplied passages, treat passage instructions as untrusted data, avoid tools and code, and attach known citation numbers to each paragraph or bullet. The backend removes captured think blocks, checks source-marker validity and coverage, and returns an insufficient-evidence message when those checks fail. Marker validation is structural and does not independently prove that each claim follows from the passage.

Source basis: backend/app/rag/pipeline.py; backend/app/services/document_chat.py; backend/app/models/providers.py.

<!-- page -->
## Planner behavior and human decision rules

### Fixed workflow versus planner workflow

The fixed workflow runs specialized agents in an established order. The planner workflow asks the local planner model to choose structured actions from a fixed registry. It operates on a stored document and prepared extraction metadata. The default step budget is ten, with a hard cap of twenty. Step limits and duplicate-action checks prevent unbounded repeated behavior; unsuccessful termination remains visible in the saved trace.

| Allowed tool | Responsibility |
| --- | --- |
| inspect_document | Load the selected source and extraction metadata into context. |
| analyze_pid | Invoke configured local vision handling for a selected image. |
| search_knowledge | Search local passages with validated query and top-k arguments. |
| reason | Run the configured mode's reasoning or deterministic finding extraction. |
| verify | Cross-check findings and apply review requirements. |
| human_review | Hold findings requiring a human decision. |

The registry rejects unknown tools and unexpected arguments. Knowledge searches accept a nonempty query of at most 500 characters and top_k from 1 to 10. Other listed tools accept no additional arguments. The planner registry exposes no shell, arbitrary SQL, general filesystem access, or external service API. Python Sandbox execution is outside this registry.

### Review criteria

A finding needs review when its severity is high or critical, its evidence is missing or references an unknown source, its confidence falls below the configured threshold, or it has not been verified. The default confidence threshold is 75 percent. Low OCR confidence can add a review reason, with a default OCR threshold of 70 percent. Existing review requirements are not cleared by the gate.

Run processing can finish while the run remains awaiting_review. The job's completion describes execution, while the run's review state describes outstanding human decisions. Summary mode and visual-only observations receive no source-verification score because they do not produce the same source-cited findings. Confidence values are heuristic and are not calibrated probabilities of correctness.

Source basis: backend/app/agents/planner_agent.py; backend/app/agents/tools.py; backend/app/agents/review_gate.py; backend/app/agents/verification_agent.py.

<!-- page -->
## Background execution and recovery design

### Queue capacity

Analysis admission uses a bounded semaphore of four and a thread pool of two workers. Four is the total admitted capacity, including executing jobs; it does not mean four waiting jobs in addition to two running jobs. A request beyond this capacity returns HTTP 429. This controls analytical work within one backend process and avoids an unbounded submission queue.

Each job persists a request payload and stage snapshot before execution. The worker updates stage timestamps, messages, metadata, and recent events in SQLite as real operations occur. On success, it saves the result and links the run; on failure, it preserves the error and marks relevant stages. The browser polls a durable job ID instead of manufacturing progress percentages.

### Batch scheduling

A durable batch can contain up to fifty selected documents and shares profile, mode, and workflow settings. Up to three batches may be active. The batch dispatcher feeds the same analysis queue, so batches do not bypass capacity limits. Per-item records allow successful documents to remain successful when a different file fails.

Request identifiers support repeat submission with the same batch settings. Reusing an identifier for a different request is rejected. Source pinning and unfinished-item checks prevent removal while the batch still needs the source. These controls reduce accidental duplicate work and conflicting operations; they are local coordination rather than a distributed transaction protocol.

### Restart semantics

Normal shutdown finishes admitted analysis work. An interrupted process leaves durable records; startup marks interrupted admitted jobs as errors and retains stage history. Pending batch items can be dispatched after restart, while interrupted inference is not resumed from a model checkpoint. Retrying an interrupted analysis is explicit.

The backend must run as a single process for this scheduler design. Locks, semaphores, and the executor are process-local. Scaling requires an external queue, worker ownership, consistent admission limits, and recovery semantics shared across processes.

### Chat capacity

Chat permits one generation at a time using a separate semaphore and rejects overlapping generation attempts. Source handles remain held during retrieval and generation to avoid source deletion halfway through the request. Chat is not a durable analysis job and its browser conversation is not stored as a persistent database transcript.

Source basis: backend/app/services/analysis_jobs.py; backend/app/services/batch_jobs.py; backend/app/services/document_chat.py.

<!-- page -->
## Configuration reference

Settings are read from the environment. Copying an environment file alone does not guarantee loading; pass Uvicorn --env-file explicitly when required. Defaults below describe the inspected source and can be overridden.

| Setting | Default or purpose |
| --- | --- |
| KAVACH_MODE | airgapped; local-provider policy. |
| KAVACH_DATA / KAVACH_UPLOADS | Runtime data and originals; defaults data/ and data/uploads/. |
| KAVACH_GENERATED / KAVACH_DB | Reports directory and SQLite path; defaults generated/ and backend/kavach.db. |
| KAVACH_MAX_UPLOAD_MB | 25 MB upload limit. |
| KAVACH_INFERENCE_PROVIDER | ollama; optional transformers. |
| KAVACH_EMBEDDING_BACKEND | hash; provider explicitly selects a learned encoder. |
| OLLAMA_BASE_URL | Local inference URL; normal default loopback port 11434. |
| OLLAMA_REASONING_MODEL | qwen3:8b. |
| OLLAMA_VISION_MODEL | llava:7b. |
| OLLAMA_OTHER_MODEL | llama3.2:3b, with legacy OLLAMA_MODEL compatibility. |
| OLLAMA_EMBED_MODEL | nomic-embed-text:latest for provider embeddings. |
| OLLAMA_TIMEOUT_S | 120 seconds. |
| KAVACH_TRANSFORMERS_MODEL_PATH | Local text-model directory; remote IDs/downloads are disabled. |
| KAVACH_TRANSFORMERS_EMBED_MODEL_PATH | Local encoder directory for learned Transformers embeddings. |
| KAVACH_TRANSFORMERS_DEVICE | auto device selection. |
| KAVACH_TESSERACT_CMD | Optional explicit Tesseract executable path. |
| KAVACH_OCR_LANGUAGE | eng; local language data must exist. |
| KAVACH_OCR_DPI | 200; constrained to 72 through 600. |
| KAVACH_OCR_MIN_PDF_TEXT_CHARS | 20; minimum text threshold for PDF OCR handling. |
| KAVACH_OCR_REVIEW_MIN_CONFIDENCE | 70 percent. |
| KAVACH_PLANNER_MAX_STEPS | 10, with hard cap 20. |
| KAVACH_REVIEW_MIN_CONFIDENCE | 75 percent. |
| KAVACH_SANDBOX_ENABLED | true; readiness still requires a provisioned engine/image. |
| KAVACH_SANDBOX_IMAGE | kavach-sandbox:1. |
| KAVACH_SANDBOX_TIMEOUT_S | 10 seconds; configurable within 1 through 30. |
| KAVACH_SANDBOX_MEMORY_MB | 128 MB; configurable within 64 through 512. |
| NEXT_PUBLIC_API_BASE | Frontend build-time API base; fallback http://localhost:8000. |

Source basis: backend/app/config.py; frontend/lib/api.ts; .env.example. This table contains configuration names and public defaults, not private environment values.

<!-- page -->
## Complete API route inventory

All paths below have the /api prefix. FastAPI also exposes the generated OpenAPI interface at /docs. Request field details are defined by backend/app/schemas/schemas.py and the route implementations.

| Method and relative path | Operation |
| --- | --- |
| GET /health | Application identity, status, and configured mode. |
| GET /system/status | Local system and service readings. |
| GET /sandbox/status | Check engine, image, and execution readiness. |
| POST /sandbox/execute | Explicit local isolated Python execution. |
| POST /documents/upload | Validate and store an uploaded source. |
| GET /documents | List stored sources. |
| GET /documents/{doc_id} | Retrieve one source's metadata. |
| GET /documents/{doc_id}/content | Serve original content or download; PDF range support. |
| GET /documents/{doc_id}/preview | Obtain viewer metadata or bounded extracted preview. |
| DELETE /documents/{doc_id} | Remove an available source while retaining history. |
| POST /documents/{doc_id}/passage | Resolve a supplied source passage for highlighting. |
| GET /documents/{doc_id}/pages/{page_number}/image | Render an available PDF/image page for source viewing. |
| GET /evidence/{evidence_id}/preview | Resolve saved evidence and source location. |
| POST /documents/{doc_id}/analyze | Synchronous source analysis. |
| POST /tasks | Create a source-backed analysis task. |
| GET /tasks/{task_id} | Retrieve task and run information. |
| GET /agent/tools | Describe allowed planner tools. |
| POST /agent/run | Execute bounded planner analysis. |
| POST /analysis/jobs | Admit a persistent background job; HTTP 202. |
| GET /analysis/jobs/{job_id} | Retrieve progress and final result/error. |
| POST /analysis/batches | Admit a durable batch; HTTP 202. |
| GET /analysis/batches | List batches. |
| GET /analysis/batches/{batch_id} | Retrieve batch and item outcomes. |
| POST /chat | Answer a question using selected documents. |
| GET /agents/runs | List historical runs. |
| GET /agents/runs/{run_id} | Retrieve full run details and evidence. |
| GET /profiles | List domain profiles and analysis modes. |
| GET /models | List configured model assignments and capabilities. |
| GET /knowledge | Inspect local index statistics. |
| POST /knowledge/search | Rank passages from indexed stored content. |
| GET /audit-logs | Retrieve local audit records. |
| POST /review/{finding_id}/approve | Save approval decision. |
| POST /review/{finding_id}/reject | Save rejection decision. |
| GET /deliverables | List saved report files. |
| GET /deliverables/{deliverable_id}/download | Download an existing generated file. |

Source basis: backend/app/api/routes.py. These routes are a source inventory, not a claim that a server was started during report generation.

<!-- page -->
## Installation and deployment procedure

### Native development setup

Prepare Python 3.10 or later, Node.js 18 or later, and Ollama. Install Tesseract separately for OCR and ensure its executable and language data are accessible. From the repository root, prepare the backend:

```text
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

On Windows, use the activation command for the chosen shell. In another terminal, provision configured models before requesting inference:

```text
ollama pull qwen3:8b
ollama pull llava:7b
ollama pull llama3.2:3b
```

From frontend/, install dependencies and start the UI:

```text
npm ci
npm run dev
```

Open http://localhost:3000. Use an explicit --env-file argument for backend settings when loading a copied .env file. Learned embeddings require their own configured model. Transformers requires the optional dependency file and local safetensors model directories; its adapter is text-only.

### Docker Compose

```text
docker compose up --build -d
docker compose exec inference ollama pull qwen3:8b
docker compose exec inference ollama pull llava:7b
docker compose exec inference ollama pull llama3.2:3b
```

Compose defines inference, backend, and frontend services. Backend shares the inference network namespace, preserving loopback inference addressing. Ports bind to localhost. Model storage has a named volume, while data and generated files use persistent mounts. Provision images, packages, models, and any build-time font assets before a fully disconnected deployment.

### Sandbox provisioning

With a local Linux-container Docker runtime, build the fixed runner using docker build -t kavach-sandbox:1 docker/sandbox. Start the native backend and inspect /api/sandbox/status. The regular Compose backend intentionally has no Docker socket mount; the repository's recommended sandbox route uses the native backend. Sandbox setup is separate from installing Python packages.

Source basis: README.md; docker-compose.yml; backend/requirements.txt; backend/requirements-transformers.txt.

<!-- page -->
## Report generation, maintenance, and troubleshooting

### Analysis export formats

The application always supports TXT and Markdown reports. DOCX, XLSX, and PDF depend on available optional libraries. Exported content includes source identity, profile and mode, model analysis where present, visual observations, findings, evidence locations, verification information, and review-related information. XLSX is useful for structured findings; PDF and DOCX suit review and sharing.

This project report is a separate documentation artifact. It explains the implementation as a whole rather than analyzing a Vault document. Its editable source is docs/KAVACH-AI-Project-Report.md, and scripts/build-project-report.py renders its PDF with report navigation and page numbering.

### Troubleshooting matrix

| Symptom | Investigation and response |
| --- | --- |
| Browser reports backend unavailable | Check Uvicorn, API base URL, host/port, and browser-reachable address. |
| Summary or Chat returns model unavailable | Confirm Ollama service, configured model availability, provider configuration, and timeout. |
| Scan has no readable fields | Check Tesseract executable/language data, rendering packages, image quality, and literal field support. |
| Image has observations but no extracted fields | Review the separate vision output; OCR may contain no supported literal field formats. |
| Queue returns HTTP 429 | Wait for admitted work to finish; avoid repeated duplicate requests. |
| Delete returns conflict | Wait for active analysis, batch, or source-pinned operation. |
| Highlight fails after source change/deletion | Retain or restore the correct original; rerun analysis when needed. |
| No search matches | Confirm stored readable content, selected scope, and vocabulary overlap; inspect embedding configuration. |
| Missing PDF/DOCX/XLSX export | Install the corresponding optional library and repeat the analysis/export workflow. |
| Sandbox reports unavailable | Check Linux Docker engine, fixed runner image, transport, and required isolation controls. |
| Interrupted job after restart | Inspect saved error/stages and explicitly retry; inference checkpoints are not resumed. |

### Maintenance recommendations

Back up originals, reports, and SQLite together while the application is stopped or using an appropriate consistent snapshot procedure. Restoration must preserve source-path configuration and file availability. This is an operational recommendation, not a built-in backup feature. Re-provision configured model files and dependencies on the target host, then validate inference, OCR, downloads, and source previews.

Avoid editing virtual-environment activation scripts as part of feature development. Keep application changes in source modules and dependencies in requirement/package files. Rebuild the frontend when changing build-time NEXT_PUBLIC_API_BASE.

Source basis: README.md; backend/app/services/deliverables.py; backend/app/services/document_store.py; scripts/build-project-report.py.

<!-- page -->
## Evaluation evidence and acceptance strategy

### Historical validation records

The repository contains successive validation stages rather than one universal test count. post-optimization-testing/VALIDATION.md records 77 early regression passes, later sandbox and Workbench checks, 121 regression passes after the image correction, and 142 passes with eight optional container tests skipped after Vault enhancements. A separate earlier real-container run recorded eight passing sandbox tests. These counts describe different versions and scopes and must not be added into a current-suite total.

The earlier feature report also records a targeted 111-test run and a real-model batch/chat/highlight smoke check. Its provenance is the existing report's implementation record. This report generation did not rerun those application checks. A recorded build or test success is evidence for that execution, not proof that every feature works on the present machine.

### Recorded real-model demonstration

The mentor evaluation record dated 5 November 2026 reports three successful workflows and two expected failures. PDF review exercised reasoning, source evidence, approval, and downloads in approximately 23.748 seconds. PNG processing exercised local OCR and vision in 17.305 seconds. Bounded planning completed in 8.228 seconds. Unsupported upload and missing-document requests were rejected as expected.

These are controlled fixture examples. Their durations depend on the recorded host and model state and are not performance guarantees. Passing five scenarios is not a general accuracy percentage. Most regression tests use provider doubles to isolate application behavior; real-model tests answer different questions and remain necessary.

### Reproducible validation commands

From backend/, install development dependencies and run the regression suite. From frontend/, check types and compile production output:

```text
python -m pytest -q
npx tsc --noEmit
npm run build
```

Run each command from its corresponding directory. For isolated live-model document checks, run python backend/scripts/verify_document_features.py from the repository root. For the five-case mentor demonstration, run python backend/evals/mentor.py. Container integration tests are opt-in and require the provisioned runtime.

### Acceptance criteria beyond unit tests

Validate source-byte downloads, OCR on representative scans, source highlights, citation behavior, follow-up questions, queue saturation, restart recovery, partial batch failure, deletion locks, report rendering, and actual browser interaction. Deployment evaluation should additionally cover backups, access controls, disk protection, resource pressure, and network boundaries. Use labeled documents to measure extraction precision/recall, retrieval recall, and claim support separately; no such broad numerical benchmark is claimed here.

Source basis: post-optimization-testing/VALIDATION.md; post-optimization-testing/CHECKLIST.md; mentor-testing/README.md; backend/evals; backend/tests.

<!-- page -->
## Future development and final assessment

### Proposed roadmap

Near-term quality work should prioritize representative document evaluation, richer multilingual OCR checks, better handling of complex layouts and tables, and claim-level source-support evaluation. Learned local embeddings can improve some retrieval cases but should be compared against a labeled query set before replacing lexical search.

Operational improvements include a consistent backup/restore tool, explicit document versioning, retention controls, and persistent conversations with visible source scope. Organizational deployment would require authenticated users, enforced authorization, reviewer roles, transport encryption, tenant-aware storage, and a threat model appropriate to remote access.

Larger workloads would benefit from durable external workers and a shared admission mechanism, pagination and indexing improvements, resource-aware model scheduling, and a scalable search backend. These are proposed extensions, not current implemented capabilities. Adding them requires preserving source provenance and explicit failure semantics.

### Project contribution

KAVACH AI integrates local document ingestion, OCR, advisory models, deterministic extraction, retrieval, evidence previews, bounded planning, human review, and report exports into one inspectable workflow. Its distinguishing value is the complete path from an original file to saved processing history and reviewable results. The current implementation is best understood as a local workstation product with a useful evidence-oriented foundation.

### Documentation production note

This report was expanded on 7 November 2026 using repository source and existing project records. No private Vault documents or private environment values were needed. Architecture figures represent code structure rather than runtime telemetry. Historical test results are labeled as recorded evidence. PDF production checks validate document generation and readability; they do not certify the running application.

The editable Markdown and PDF renderer are retained so the report can evolve with the project. The report intentionally avoids invented author names, institutional approval, hardware benchmarks, accuracy scores, or security certification.

# Legal Research Assistant: Project Plan

Nov 16, 2026 · @Preacher

A 16-week plan to build a working research assistant for Bangladesh criminal cases, with a retrieval demo at the midterm and the full argument map and evaluation at the final. It assumes two people (Rashik and Jinnia), a start on 16 November 2026, a midterm in week 8 and a final in week 16; shift the weeks if the course calendar differs.

## At a glance

&#91;embedded content: project timeline · 6 phases, 2 presentations\]

Phases overlap where one person can start the next while the other finishes; the argument map work starts before agent work ends so the frontend is not idle.

## Decisions to settle in week 1

Six open questions block later work; settle them before writing code.

| Decision | Recommended answer | Why it matters |
| --- | --- | --- |
| Course calendar | Confirm midterm and final dates with Dr. Nurul Ahad Tawhid | Every week number in this plan shifts with it |
| Offence focus within criminal law | Pick 2–3 offence types common in appeal judgments (for example homicide, dowry-related violence, narcotics) | Keeps the corpus, schema and evaluation set small enough to finish |
| Knowledge graph storage | JSON files managed by the app (as in the proposal form) | Avoids running a graph database; revisit only if traversal gets slow |
| Models | Inkling 2 Small for research and review; Gemma 4 31B for extraction, Bangla and fallback | Decided by the model test in Phase 0 |
| Bangla scope | Bangla documents translated and checked beforehand; native Bangla processing is future work | Keeps the demo honest and the scope finishable |
| Reviewing lawyer | Find one lawyer or senior law student willing to review 15–30 cases | Evaluation in Phase 5 depends on it |

## Phase 0 – Setup and model test (weeks 1–2)

Goal: a working skeleton both of you can build on, and a tested choice of models.

**Tasks**

- [ ] Create the GitHub repository with backend (FastAPI) and frontend (React + TypeScript) folders, a README and a shared issue board.
- [ ] Set up OpenRouter and buy the one-time $10 credit to raise the free-model limit to 1,000 requests per day; keys go in a backend `.env` file only.
- [ ] Set up Pinecone, PostgreSQL (SQLite is fine for now) and Redis locally.
- [ ] Write one small model client that both models go through, with request counting and logging from day one.
- [ ] Run the Bangla model test: 10–20 Bangla passages through Inkling 2 Small and Gemma 4 31B; check names, dates, section numbers and whether denials stay denials.
- [ ] Run a quick English check: give both models 5 judgment excerpts and ask for parties, claims and counterarguments in JSON; record how often the JSON is valid.
- [ ] Write a one-page test note with the results and the final model choice.

**Exit criteria:** repository runs on both laptops; one test call to each model succeeds; model choice written down.

## Phase 1 – Data collection and schema (weeks 2–4)

Goal: a small, clean corpus for the chosen offences and a schema tested on a real case.

**Tasks**

- [ ] Download the core statutes from [Laws of Bangladesh](https://bdlaws.minlaw.gov.bd/): Penal Code, Code of Criminal Procedure, Evidence Act, plus any special law for the chosen offences.
- [ ] Split statutes into provisions, each with section number, text, source URL, retrieval date and amendment notes.
- [ ] Collect 60–100 Bangladesh Supreme Court criminal appeal judgments for the chosen offences from the official website, with case number, court, date and paragraph numbers.
- [ ] Collect 20–40 relevant Indian judgments, tagged as persuasive authority.
- [ ] Pick 15–30 of the Bangladesh judgments as the evaluation set and keep them out of the reference collection.
- [ ] For each evaluation case, write case-file documents from the facts, FIR summary and witness accounts, and a separate answer key: each side's arguments and the court's conclusion. Hide the court's reasoning from the case files.
- [ ] Map one complete case by hand onto the schema (claims, witnesses, evidence, offence elements, arguments) and adjust the concept and relationship list.
- [ ] Translate any Bangla documents needed for the evaluation set, keeping the original Bangla alongside, and spot-check each one.

**Exit criteria:** statutes and judgments stored with stable IDs; evaluation set and answer keys ready for at least 15 cases; schema version 1 written down.

## Phase 2 – Ingestion and retrieval core (weeks 4–7)

Goal: upload a case, extract its facts, and retrieve the right law with exact citations.

**Backend tasks**

- [ ] Document upload and storage; originals never modified.
- [ ] Text extraction with page and paragraph references (PyMuPDF); OCR for scanned English pages.
- [ ] Chunking that keeps legal text whole (one provision or one judgment paragraph per chunk).
- [ ] Embeddings into Pinecone with metadata: jurisdiction, authority level, source ID, version, case ID.
- [ ] BM25 keyword index alongside the vector index; merge results and rerank.
- [ ] Case analyst: extract parties, witnesses, claims (with status), events and dates into the schema, validated with Pydantic.
- [ ] Build the knowledge graph as JSON: claims, witnesses, evidence, offences, provisions, judgments and their links, each with source passage and review status.
- [ ] Access control: every query and graph lookup filtered by case ID.

**Frontend tasks**

- [ ] Case list and case creation page.
- [ ] Document upload and a viewer that jumps to a cited page and paragraph.
- [ ] Case summary page: parties, witnesses, timeline, claims with status.
- [ ] Search page showing results with source and jurisdiction labels.

**Exit criteria:** for 3 evaluation cases, upload works, the case summary is mostly right, and searching an offence returns the correct sections and at least one relevant judgment.

## Midterm presentation (week 8)

Show the foundation working end to end on one real case; arguments and the map are explicitly "in progress".

**Live demo**

1. Create a case and upload its documents.
2. Show the case summary: parties, witnesses, timeline and disputed claims, each linked to its source passage.
3. Search for the charged offence and show the matching sections and judgments, with jurisdiction labels and exact citations.
4. Show the knowledge graph for the case (even as a simple view).

**Slides**

- [ ] Problem statement and why source-linked research matters.
- [ ] Architecture diagram and the two-model setup.
- [ ] Data: corpus size, evaluation set, how case files were built from judgments.
- [ ] Early numbers: extraction accuracy on 3–5 cases, model calls per task.
- [ ] What remains: arguments, counterarguments, argument map, evaluation.

**Fallback:** record the demo in advance in case the free endpoint is slow or down on the day.

## Phase 3 – Agents, arguments and counterarguments (weeks 8–11)

Goal: for a case, produce prosecution and defence arguments and counterarguments, each tied to cited passages.

**Tasks**

- [ ] Orchestrator as a bounded state machine (LangGraph or plain Python): fixed steps, limits on tool calls, retries and graph depth.
- [ ] Request budget queue: 20 requests per minute and the daily limit, with backoff on rate-limit errors.
- [ ] Offence element mapping: for each charged section, list the elements the prosecution must prove and the evidence offered for each.
- [ ] Agentic retriever: plan searches per legal question, run hybrid search and graph lookups, and search specifically for exceptions and contrary judgments.
- [ ] Research package format: passages, source IDs, supporting and opposing material, open questions.
- [ ] Argument analyst: prosecution and defence positions, each listing the claims, witnesses, evidence and law it relies on.
- [ ] Critical reviewer on the other model: checks each citation supports its claim, generates counterarguments labelled by type (attacks a premise, rebuts with other authority, denies the inference), and marks each as supported or a candidate needing research.
- [ ] "Not supported" output when the material does not support an argument.
- [ ] Log every call: role, model, tokens, latency, retries.

**Exit criteria:** on 5 evaluation cases, the system produces arguments for both sides; at least half of the arguments the court actually discussed appear; every citation opens the right passage.

## Phase 4 – Argument map and lawyer review interface (weeks 10–13)

Goal: the signature feature, where changing an assumption visibly changes which arguments still hold.

**Tasks**

- [ ] Argument map in React Flow: arguments connected to the claims, witnesses, exhibits, provisions and judgments they rely on.
- [ ] Click any node to open its source passage in the document viewer.
- [ ] Dispute a claim or exclude a witness or document; affected arguments are highlighted using a graph walk in code, with no model call.
- [ ] Re-assessment button: runs the retriever and reviewer only on the affected arguments and explains what changed.
- [ ] Before/after view; the lawyer's edits shown separately from system suggestions.
- [ ] Accept, reject and edit controls on facts, links, arguments and counterarguments; edits invalidate dependent results.
- [ ] Anonymization step: replace names with consistent placeholders before model calls and map them back for display.
- [ ] Cache research packages keyed by case revision and sources, so unchanged parts are not recomputed.

**Exit criteria:** on one case, excluding a key witness highlights the right arguments and the re-assessment explains the change correctly.

## Phase 5 – Evaluation and hardening (weeks 13–15)

Goal: numbers that show whether the system works, and a stable demo.

**Evaluation runs**

- [ ] Run all evaluation cases through three setups: plain vector search, vector search plus the knowledge graph, and the full agent pipeline.
- [ ] Compare each setup's arguments and counterarguments with the answer keys: how many of counsel's actual arguments were found, and how many outputs had no valid source.
- [ ] Check citations: does each cited passage exist and support the claim it is attached to?
- [ ] Measure model calls, tokens and time per completed case for each setup.
- [ ] Reviewing lawyer rates 10–15 outputs for usefulness and correctness.
- [ ] Automated checks on call logs: searched for contrary authority, checked provision versions, stayed in budget, never crossed a case boundary.

**Hardening**

- [ ] Fix the most common failure types found above.
- [ ] Clear error messages and partial results when the budget runs out.
- [ ] Seed the demo with 3 polished cases.

**Exit criteria:** a results table for the report and a demo that runs end to end without manual fixes.

## Final presentation (week 16)

The final shows the whole workflow on a real case, backed by measured results.

**Deliverables**

- [ ] Live demo: upload, case summary, research, arguments and counterarguments with sources, then exclude a witness and show the map update.
- [ ] Project report: problem, design, data, evaluation results, limitations (US-only benchmarks, Bangla pre-translated, free-endpoint privacy limits) and future work.
- [ ] Results table comparing the three setups on accuracy, citation support, model calls and time.
- [ ] Source code on GitHub with setup instructions.
- [ ] Recorded backup demo.

## Team split

A suggested split based on Rashik's earlier GraphRAG work; swap areas if your strengths differ. Both review each other's pull requests.

| Area | Rashik | Jinnia |
| --- | --- | --- |
| Retrieval (vector, BM25, reranking) and knowledge graph | Owner | Reviews |
| Agents, orchestrator and budget queue | Owner | Reviews |
| Document ingestion, extraction and OCR | Supports | Owner |
| Frontend: case pages, viewer, argument map | Supports | Owner |
| Data collection, evaluation set and answer keys | Shared | Shared |
| Evaluation runs and results table | Supports | Owner |
| Report and presentations | Shared | Shared |

Meet twice a week for 30 minutes: what was done, what is blocked, what is next. Track tasks on the GitHub issue board, one issue per checklist item above.

## Risks and fallbacks

The biggest risks are external: free models, data quality and finding a reviewer.

| Risk | Early sign | Fallback |
| --- | --- | --- |
| A free model is withdrawn or rate-limited | Frequent errors or slow responses | Switch roles to the other model; keep a small paid credit for the demo week |
| Model output often breaks the JSON schema | Validation failures above about 10% | Simpler schemas per call, bounded retry, move extraction to Gemma 4 31B |
| Scanned judgments OCR badly | Garbled text in extraction checks | Prefer text PDFs; drop the worst scans from the evaluation set |
| No reviewing lawyer found | None by week 6 | Senior law students, or rely on the answer keys from judgments |
| Agents too slow for a live demo | Over 5 minutes per case | Precompute results for demo cases; show re-assessment live only |
| Scope too large for 16 weeks | Phase 2 exit missed by week 8 | Drop Indian judgments, citation following and before/after view first |
| Privacy slip with real files | Anyone suggests uploading a client file | Public or anonymized cases only while free endpoints are used |

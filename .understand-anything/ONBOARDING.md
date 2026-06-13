# LQ Skills — Onboarding Guide

Welcome to **lq-skills**, the Legal Quants skill registry. This guide gets you from a cold clone to a productive contribution. It was generated from the project's knowledge graph (commit `f878528`, analyzed 2026-06-13) and is written for both lawyer-builders and engineers.

---

## 1. Project Overview

| | |
|---|---|
| **Name** | lq-skills |
| **What it is** | A curated, version-controlled collection of harness-agnostic agent skills for legal work |
| **Who builds it** | The Legal Quants community — roughly 100 lawyer-builders across 17+ jurisdictions |
| **Languages** | Mostly Markdown (the "code" here is prose); Python and JavaScript for deterministic helper scripts; JSON/YAML for schemas, manifests, and eval specs; DOCX/HTML for document templates |
| **Frameworks** | None — deliberately. Skills are plain folders that work across Claude Code, Codex CLI, Gemini CLI, OpenCode, Cursor, and OpenClaw |
| **Scale** | 42 top-level skills (45 SKILL.md files counting CoQuill's nested sub-skills), ~274 files |

The most important thing to internalize on day one: **this repo has no runtime entry point**. It is not an application — it is a library of self-contained skill folders. Each skill's "program" is a `SKILL.md` file: a structured set of instructions that an LLM agent executes, covering trigger conditions, workflow steps, output format, and escalation rules. Everything else in a skill folder (references, examples, assets, scripts, tests) hangs off that file.

Skills span jurisdiction-specific document review (UK CPR, Singapore courts, US state privacy law), citation verification and hallucination detection, statutory analysis, contract review, document assembly, and legal translation.

---

## 2. Architecture Layers

The knowledge graph organizes the repo into seven layers. Think of them as the strata of every skill folder plus the repo-level glue.

### 2.1 Skill Definitions (45 nodes)
The `SKILL.md` entry points — 42 top-level skills plus CoQuill's nested analyzer/renderer/transcriber sub-skills. Each declares the skill's trigger, workflow, and pointers into its references, scripts, and examples. This is the hub every other artifact links to. Start any exploration of a skill here.

### 2.2 Reference Knowledge Bases (114 nodes)
Per-skill reference corpora under `references/` (or `reference/`), loaded on demand by SKILL.md workflow steps — never all at once. Examples: 20 US state privacy statute profiles and DSAR workflows, NIST AI RMF core functions and the GAI profile, UK litigation review playbooks, canons of statutory construction, and MSA/NDA/DPA issue checklists, red flags, and severity rubrics. This is the repo's equivalent of a database layer: curated, citable legal knowledge instead of tables.

### 2.3 Worked Examples (33 nodes)
Calibrated example inputs and outputs under each skill's `examples/` directory (buyer vs. supplier MSA reviews, clear vs. vague client alerts, GDPR vs. US-state DPA reviews). They anchor the expected tone and depth of skill output — snapshot tests for LLM behavior.

### 2.4 Templates & Assets (16 nodes)
Reusable document-production material: CoQuill's template packs (`manifest.yaml` plus DOCX/HTML/Markdown bodies for NDAs, invoices, and meeting notes) and the privacy navigator's memo template, notice clauses, and applicability questionnaire (a JSON Schema).

### 2.5 Automation Scripts (8 nodes)
Executable Python/JavaScript helpers that skills *invoke* rather than read as prose: CoQuill's analyze/render/transcribe pipeline and the us-state-privacy-navigator's applicability checker, citation auditor, conflict resolver, precedent matcher, and DOCX memo generator. Used wherever a wrong answer is unacceptable.

### 2.6 Evals & Tests (34 nodes)
Quality gates: per-skill `evals.yaml` behavioral scenario specs and `test-plan.md` plans, the `evals/` harness README, and the us-state-privacy-navigator's stdlib-unittest pytest-style suite with JSON intake and memo fixtures.

### 2.7 Governance & Meta Docs (26 nodes)
Repo-level governance and per-skill housekeeping: root `README.md`, `CONTRIBUTING.md`, `ACCESS-MODES.md`, and `PR-READINESS.md`, the `skills/CONTRIBUTING.md` guide for legal-substance skills, plus per-skill READMEs and CHANGELOGs.

---

## 3. Key Concepts

### Skills are prose programs
A `SKILL.md` is executable — by an LLM agent, not a compiler. Workflow steps, output formats, confidence bands, and escalation rules are written as instructions. When you "read the code," you are reading the program itself. Contributions are judged on the same axes as software: clear control flow, explicit failure modes, testability.

### Hub-and-spoke skill anatomy
Every non-trivial skill is one orchestrating `SKILL.md` plus spokes: `references/` (domain law), `examples/` (calibration), `assets/` (templates/schemas), `scripts/` (deterministic helpers), `tests/` or `evals.yaml` (quality gates). sgcite is the minimal one-file case; us-state-privacy-navigator is the full stack (and the most connected node in the graph, with 38 incoming edges).

### Progressive disclosure
SKILL.md files stay short because domain law lives in reference files loaded only when a workflow step needs them. Never inline a statute profile into SKILL.md — add a reference file and point to it from the relevant step.

### The review-skill pattern (checklist–rubric–lens)
Document-review skills share a recurring reference shape, tightest in msa-review-saas: an **issue checklist** (tiered terms to inspect), a **red-flags** file (dealbreakers), a **severity rubric** (standardized finding ratings), and a **perspective lens** (recalibrates the review for vendor vs. customer, buyer vs. supplier). The NDA, DPA, and purchase-MSA reviews follow the same shape — learn it once, read them all at a glance.

### Legal judgment to the LLM, determinism to code
The repo's core architectural bet. Threshold math (does CCPA apply at this revenue/consumer count?), citation-format validation, conflict-of-laws ceiling computation, and DOCX generation go to scripts the agent executes. Judgment-laden analysis stays with the LLM. If a wrong answer is unacceptable, it belongs in a script with tests.

### Examples as calibration, evals as behavior gates
Because output is prose, not typed return values, skills ship worked examples (what good looks like) and behavioral `evals.yaml` scenarios (assertions about agent output — e.g. uk-citation-verification requires that a live-source check actually names its sources). Scripts additionally get real unit tests with fixtures encoding exact expected verdicts.

### Never assume tool access
`ACCESS-MODES.md` is repo-wide policy: the same skill may run in harnesses with very different capabilities (uploads, local file tools, web search, MCP servers), and skills grant no credentials themselves. Skills must degrade gracefully and state their source mode rather than assume access. A hard eval fail condition is "verifying" from model memory instead of a live source.

### The six legal domains
The domain graph clusters the library into:

1. **Litigation Document Review** — England & Wales-centred QC of civil-case artifacts: Particulars of Claim vs. CPR 16/PD 16, witness statements vs. PD 57AC, disclosure lists, chronologies, proposition checking.
2. **Contract Review & Negotiation Support** — perspective-calibrated reviews: seven-pass SaaS and commercial-purchase MSA reviews, one-way NDA review, regime-aware DPA/BAA checklists, clause-level contract Q&A.
3. **Privacy & Regulatory Compliance** — compliance counseling with the US state privacy navigator as flagship: deterministic 20-state applicability triage, controller/processor status, strictest-rule conflict synthesis.
4. **Citation Integrity & Statutory Analysis** — anti-hallucination and authority discipline: UK citations vs. Find Case Law/BAILII, Singapore citations vs. eLitigation, statutory-reference auditing, canons of construction.
5. **Document Drafting & Assembly** — producing/transforming documents rather than reviewing them: CoQuill's seven-phase template pipeline, native Word tracked-changes redlining, legal translation, plain-language rewriting.
6. **Skill Quality Assurance & Governance** — the meta-domain keeping a 40-plus-skill multi-author registry coherent: skill-creator authoring, behavioral evals, the unittest suite, contribution and PR-readiness gates.

---

## 4. Guided Tour

A recommended reading path, layer by layer. Each step lists the files to open.

1. **Project Overview** — `README.md`. What LQ Skills is, the per-skill author/jurisdiction table, and install instructions showing the same folders working across six harnesses. No runtime entry point — the architecture is a library of self-contained skill folders.

2. **Anatomy of a Skill** — `skills/sgcite/SKILL.md`. The cleanest minimal example: a single markdown definition for verifying Singapore court citations against eLitigation, detecting hallucinated cases, banding confidence, and escalating. Notice the "code" is prose.

3. **A Deep Skill: Privacy Navigator** — `skills/us-state-privacy-navigator/SKILL.md` + `README.md`. The most connected node in the graph. Its SKILL.md is an orchestrator: intake, applicability triage across all 20 US state privacy laws, conflict-of-laws synthesis, gap analysis against enforcement precedent, citation-audited memo deliverables — a hub-and-spoke design coordinating references, scripts, assets, and tests.

4. **Reference Knowledge Bases** — `skills/us-state-privacy-navigator/references/applicability-matrix.md`, `references/states/ca.md`, `references/rights-comparison.md`, `references/workflows/dsar-routing.md`. The applicability matrix encodes consumer-count and revenue thresholds for every state; per-state cards (California is the deepest: CCPA/CPRA rights, GPC signals, CPPA enforcement) carry statute-level detail; workflow references turn law into procedure.

5. **The Review-Skill Pattern** — `skills/msa-review-saas/SKILL.md` + its four `reference/` companions (`issue_checklist.md`, `red_flags.md`, `severity_rubric.md`, `perspective_lens.md`). The tightest cluster in the graph; the checklist–rubric–lens triad reappears in the NDA, DPA, and purchase-MSA reviews.

6. **Worked Examples as Calibration** — `skills/enhance-prompt/SKILL.md` + `examples/example_short_prompt.md`, `examples/example_skipped.md`, `examples/example_with_skill.md`. Three examples covering full expansion, correctly skipping enhancement, and routing to another skill — snapshot tests for LLM behavior.

7. **Templates and Structured Assets** — `skills/us-state-privacy-navigator/assets/memo-template.md`, `assets/applicability-questions.json`, `assets/notice-clauses/notice-at-collection.md`. The LLM drafts judgment-laden analysis, but document structure and intake are pinned down by templates and JSON Schemas.

8. **Automation Scripts** — `skills/us-state-privacy-navigator/scripts/` (`applicability_check.py`, `citation_audit.py`, `conflict_resolver.py`, `precedent_match.py`, `generate_docx_memo.js`). Deterministic verdicts (Applies / Likely Applies / Does Not Apply), citation-discipline gating, cross-state conflict ceilings, precedent ranking over the 82-action corpus, and DOCX rendering.

9. **CoQuill: Multi-Agent Assembly** — `skills/coquill/SKILL.md` + `analyzer/analyze.py`, `renderer/render.py`, `transcriber/transcribe.py`. A seven-phase document-assembly pipeline delegating to three nested sub-skills, each pairing its own SKILL.md with a script: extract Jinja2 variables into a manifest → conversational interview → render to docx/HTML/markdown → transcribe the session.

10. **CoQuill Template Packs** — `skills/coquill/templates/_examples/` (`meeting_notes/` with `meeting_notes.md` + `manifest.yaml` + `config.yaml`, `Bonterms_Mutual_NDA/Bonterms-Mutual-NDA.docx`, `invoice/invoice.html`). Each pack pairs a template body with a machine-generated manifest and optional author config; the three bodies deliberately exercise all three render paths.

11. **Testing the Deterministic Core** — `skills/us-state-privacy-navigator/tests/` (`run_all.py`, `test_applicability_check.py`, `test_citation_audit.py`, fixtures `intake_multistate.json`, `memo_with_errors.md`). Stdlib unittest, zero dependencies, CI-friendly; fixtures encode the contract: given this exact intake, the engine must reach these exact per-state verdicts.

12. **Behavioral Evals for Prose Skills** — `evals/README.md`, `skills/uk-citation-verification/evals.yaml`, `skills/enhance-prompt/test-plan.md`. The second quality gate for skills with no Python to unit-test: per-skill scenario specs with assertions about agent output, plus acceptance-bar test plans.

13. **Governance and Contribution** — `CONTRIBUTING.md`, `ACCESS-MODES.md`, `PR-READINESS.md`. The practitioner-built philosophy and fork-and-PR process; the never-assume-tool-access policy; and the pre-flight checklist (content hygiene, cross-harness install checks).

---

## 5. File Map

Repo layout: skills live in `skills/<skill-name>/`; everything below is relative to the repo root. Domain groupings are approximate (from the domain graph).

### Skill Definitions (`skills/*/SKILL.md`)

**Litigation Document Review**

| Skill | Purpose |
|---|---|
| `building-chronologies` | Turn legal documents and disclosure materials into a sourced event chronology |
| `case-file-analyzer` | Proof-of-concept stateless R.A.L.P.H. loop for adversarial analysis of large case-file directories |
| `proposition-checking` | Check whether cited cases, statutes, exhibits, transcripts, and pleadings actually support the propositions asserted |
| `uk-disclosure-list-review` | Six-step QC of England & Wales disclosure lists under the CPR |
| `uk-particulars-of-claim-review` | Review E&W Particulars of Claim against CPR 16/PD 16 with per-cause element maps |
| `uk-witness-statement-review` | Review draft witness statements for PD 57AC and statement-of-truth compliance |
| `uk-court-of-appeal-judicial-preference-check` | Test Court of Appeal drafts against source-based judicial preferences |
| `legal-claim-economics` | Model claim and portfolio economics: revenue/costs, funder economics, MOIC, DBA/CFA structures |

**Contract Review & Negotiation Support**

| Skill | Purpose |
|---|---|
| `msa-review-saas` | Seven-pass SaaS MSA review, vendor- or customer-calibrated, severity-rated findings |
| `msa-review-commercial-purchase` | Seven-pass commercial purchase/supply MSA review for physical goods and services |
| `nda-review` | Five-step playbook for one-way NDAs from recipient or discloser perspective |
| `dpa-checklist-review` | Check DPAs/Addenda and HIPAA BAAs against GDPR Art. 28, US state privacy laws, or HIPAA |
| `contract-qa` | Answer specific questions about a loaded contract via a classify–locate–answer workflow |
| `lq-board-document-review` | Multi-jurisdiction review protocol for board-level governance documents |
| `lq-governance-playbook-benchmark` | Benchmark a governance document against a playbook standard |

**Privacy & Regulatory Compliance**

| Skill | Purpose |
|---|---|
| `us-state-privacy-navigator` | Flagship: cross-jurisdictional analysis of all 20 US state consumer privacy laws, intake → triage → conflict synthesis → audited memo |
| `vendor-privacy-policy-first-pass` | Fast regime-aware (GDPR, CCPA/CPRA, HIPAA, FERPA) triage of a vendor's published privacy policy |
| `nist-ai-rmf` | Apply the NIST AI RMF (AI 100-1 + AI 600-1 GAI Profile) to a specific AI system |
| `local-first-legal-workspace` | Audit checklist for local-first legal AI workspaces handling confidential documents |
| `license-comply` | Audit open-source dependency licenses in Python projects (SPDX → policy bands) |
| `customs-trade-law` | US customs/trade: HTS tariff classification, CBP CROSS ruling research, CIT/CAFC case mapping |
| `california-property-tax` | Research workflow for CA property tax change-in-ownership questions (BOE Rules 462.*) |
| `classify-ccp` | Six-phase classification of how Competition Compliance Programmes are treated in enforcement decisions |
| `action-items-from-client-alert` | Extract time-sensitive action items, deadlines, and obligations from client alerts and bulletins |

**Citation Integrity & Statutory Analysis**

| Skill | Purpose |
|---|---|
| `uk-citation-verification` | Pre-filing audit of UK citations, party names, paragraph refs, and quotes against Find Case Law/BAILII |
| `sgcite` | Verify Singapore court citations against eLitigation; detect hallucinated cases (minimal single-file skill) |
| `bart-statutory-reference-checker` | Singapore statutory citation audit with a Legal-BERT/BM25/ChromaDB retrieval backend and Word add-in |
| `statutory-analysis` | First-pass framework for reading US statutes: definitions-first, operator words, canons of construction |
| `text-provenance` | Identify the likely source of a text passage: RAG attribution, plagiarism, clause-origin matching |
| `foreign-law-research` | Tiered foreign/cross-border legal research with an L1–L4 source authority hierarchy (Chinese-speaking researchers) |
| `corporate-registry-investigation` | Investigate UK companies via Companies House: officers, PSCs, charges, filings |

**Document Drafting & Assembly**

| Skill | Purpose |
|---|---|
| `coquill` | Orchestrator for seven-phase template-driven document assembly (see sub-skills below) |
| `coquill/analyzer` | Sub-skill: run `analyze.py` to extract Jinja2 variables/conditionals/loops into a manifest |
| `coquill/renderer` | Sub-skill: run `render.py` to fill templates into docx/HTML/markdown (+ PDF fallback) |
| `coquill/transcriber` | Sub-skill: run `transcribe.py` to convert a finished interview session into a readable record |
| `redlines` | Convert text diffs into native Word tracked-changes revisions (redlines library; SG workflows) |
| `superdoc-redlines` | Multi-agent DOCX redlining with conflict resolution into native tracked changes |
| `office-word-diff` | Word-level tracked changes via Office.js, preserving formatting (SG workflows) |
| `vibe-legal-batch-redliner` | Batch contract redlining for UK commercial practice via the Vibe Legal FastAPI server (AMEND/INSERT playbook edits) |
| `collating-reviewer-feedback` | Collate comments, tracked changes, and redlines from multiple reviewer DOCX drafts into a grouped resolution checklist |
| `legal-translation` | Translate legal effect (not literal words) for any language pair: four-layer model, glossary build, confidence bands |
| `comms-improver` | Rewrite legal-jargon text into plain language for a specified non-legal audience |

**Skill QA & Governance (meta-skills)**

| Skill | Purpose |
|---|---|
| `skill-creator` | Guide a lawyer through a focused conversation to author a new LQ skill |
| `enhance-prompt` | Rewrite short/vague inputs into structured legal prompts (role, jurisdiction, task, constraints, format) |
| `adversarial-qc` | Spawn two verification agents against a configurable checklist at quick/standard/deep intensity |

### Reference Knowledge Bases (`skills/*/references/` or `reference/`)

The big corpora:

- `skills/us-state-privacy-navigator/references/` — the deepest: `applicability-matrix.md`, 20 per-state cards in `states/` (`ca.md` … `va.md`), `rights-comparison.md`, `controller-duties.md`, `sensitive-data.md`, `universal-opt-out.md`, `kids-and-teens.md`, `federal-overlays.md`, `enforcement.md`, `ag-priorities.md`, `defense-arguments.md`, the structured `enforcement_actions.json` (82 normalized actions, 48-tag taxonomy), and `workflows/` (DSAR routing, gap-analysis method, intake questionnaire, status determination).
- `skills/nist-ai-rmf/references/` — `core/` (govern/map/measure/manage, trustworthy characteristics, glossary), `gai-profile/` (risks, per-function actions, glossary), `templates/` (assessment, consult, governance plan), `crosswalk.md`.
- `skills/msa-review-saas/reference/` and `skills/msa-review-commercial-purchase/reference/` — the checklist–rubric–lens quartets (`issue_checklist.md`, `red_flags.md`, `severity_rubric.md`, `perspective_lens.md`).
- `skills/nda-review/references/` — `KEY_CLAUSES.md`, `STANDARD_EXCEPTIONS.md`, `DURATION_SCOPE.md`, `PARTY_OBLIGATIONS.md`, `REMEDIES_LIABILITY.md`.
- `skills/dpa-checklist-review/reference/` — GDPR, US-state, HIPAA BAA, and general commercial requirement sets.
- `skills/statutory-analysis/references/` — `canons_of_construction.md`, `statutory_structure.md`, `practical_lessons.md`, `index.md`.
- `skills/legal-translation/references/` — `document-type-library.md` (12 document types), `legal-language-conventions.md` (9 target languages), `legal-glossary.md`.
- UK litigation playbooks — one model/playbook file per skill, e.g. `skills/uk-disclosure-list-review/references/disclosure-review-playbook.md`, `skills/uk-citation-verification/references/citation-resolution-model.md`.
- Single-model skills follow the same shape: `skills/building-chronologies/references/chronology-model.md`, `skills/legal-claim-economics/references/engine-model.md`, `skills/foreign-law-research/references/resources.md`, etc.

### Worked Examples (`skills/*/examples/`)

Paired calibration files per skill: `msa-review-*` (buyer vs. supplier, customer vs. vendor reviews), `dpa-checklist-review` (GDPR vs. US-state), `action-items-from-client-alert` (clear vs. vague vs. multi-jurisdiction alerts), `contract-qa` (one per question type A/C/D/E), `enhance-prompt` (expanded / skipped / routed), `comms-improver` (three audiences), and a single canonical `examples/output.md` for each UK litigation skill and the model-based skills.

### Templates & Assets

- `skills/coquill/templates/_examples/` — three packs exercising all render paths: `meeting_notes/` (markdown + `manifest.yaml` + `config.yaml`), `Bonterms_Mutual_NDA/` (CC-BY DOCX + manifest + readme), `invoice/` (HTML with print styling + manifest).
- `skills/us-state-privacy-navigator/assets/` — `memo-template.md` (memo skeleton: disclaimer, entity profile, applicability matrix, gap log, roadmap), `applicability-questions.json` (draft-07 JSON Schema intake), `notice-clauses/` (notice-at-collection, opt-out disclosures, sensitive-data, financial-incentive).
- `skills/classify-ccp/assets/` — `examples.md`, `ScratchpadTemplate.md`.

### Automation Scripts

| Script | What it does |
|---|---|
| `skills/us-state-privacy-navigator/scripts/applicability_check.py` | Deterministic threshold engine: evaluates a business-profile intake against all 20 state applicability tests; emits Applies / Likely Applies / Does Not Apply with reasoning |
| `.../scripts/citation_audit.py` | Pre-publication QA: flags uncited substantive claims, validates citation formats and plausible code-section ranges |
| `.../scripts/conflict_resolver.py` | Computes the multi-state "compliance ceiling" — strictest binding rule per ~13 dimensions |
| `.../scripts/precedent_match.py` | Ranks the 82-action enforcement corpus against gap tags or free-text queries |
| `.../scripts/generate_docx_memo.js` | Builds the client-ready DOCX memorandum (docx-js CLI): cover, TOC, matrices, tables |
| `skills/coquill/analyzer/analyze.py` | Detects docx/HTML/markdown templates; extracts Jinja2 variables/conditionals/loops via regex with a scope stack; merges `config.yaml` overrides |
| `skills/coquill/renderer/render.py` | Renders docx (docxtpl), HTML, and markdown (Jinja2); coerces string booleans; collision-safe job directories |
| `skills/coquill/transcriber/transcribe.py` | Converts `interview_log.json` + `manifest.yaml` into a human-readable `transcript.md` |

### Evals & Tests

- `evals/README.md` — how the behavioral eval harness works; hard fail conditions include verifying from model memory and predicting legal outcomes.
- `skills/*/evals.yaml` — scenario specs for the 11 evals-covered skills (the UK litigation set, chronologies, claim economics, registry investigation, citation verification, etc.).
- `skills/*/test-plan.md` — acceptance-bar plans for 9 skills (enhance-prompt, contract-qa, the MSA reviews, dpa-checklist-review, comms-improver, skill-creator, vendor-privacy-policy-first-pass, action-items-from-client-alert).
- `skills/us-state-privacy-navigator/tests/` — `run_all.py` (stdlib unittest discovery, non-zero exit on failure, no dependencies) plus `test_applicability_check.py`, `test_citation_audit.py`, `test_conflict_resolver.py`, `test_precedent_match.py`, `test_corpus_integrity.py` and fixtures (`intake_multistate.json`, `intake_ca_only.json`, `intake_below_thresholds.json`, `gaps_sample.json`, `memo_clean.md`, `memo_with_errors.md`).

### Governance & Meta Docs

| File | Purpose |
|---|---|
| `README.md` | Registry overview, per-skill author/jurisdiction table, cross-harness install instructions |
| `CONTRIBUTING.md` | Practitioner-built philosophy, fork-and-PR process, required SKILL.md-plus-references format, quality bar |
| `skills/CONTRIBUTING.md` | Higher review bar for legal-substance skills: the five-step claim/draft/attest/review/merge process, versioning, forking for divergent legal positions |
| `ACCESS-MODES.md` | Repo-wide policy: how source-dependent skills obtain materials across harnesses; never assume tool or credential access |
| `PR-READINESS.md` | Pre-PR checklist: content hygiene (no firm/client/personal names), behavior checks, cross-harness install checks, PR summary format |
| `skills/*/README.md`, `skills/*/CHANGELOG.md` | Per-skill design docs and history (coquill and statutory-analysis keep CHANGELOGs) |

---

## 6. Complexity Hotspots

These are the areas the graph rates `complex` — approach with extra care, and do not change them without running the corresponding gates.

**1. The us-state-privacy-navigator script suite** (all five scripts complex)
`applicability_check.py` encodes 20 states' threshold tests, including California's disjunctive CCPA/CPRA thresholds and the Virginia-model two-tier tests — legal logic disguised as code. `conflict_resolver.py` computes the strictest-rule ceiling across ~13 dimensions; `citation_audit.py` is the gate that blocks memos with uncited claims; `precedent_match.py` scores against a 48-tag taxonomy; `generate_docx_memo.js` builds the full client deliverable. **Any change here must keep `tests/run_all.py` green** — the fixtures encode exact expected per-state verdicts.

**2. The enforcement-actions corpus** (`references/enforcement_actions.json`)
A schema-versioned, structured corpus of 82 normalized enforcement actions (state privacy plus CIPA/BIPA/VPPA/FTC). It is data with legal meaning: `test_corpus_integrity.py` guards its schema, and `precedent_match.py` depends on its tag taxonomy. Treat edits as legal-content changes requiring attestation under `skills/CONTRIBUTING.md`.

**3. The CoQuill pipeline** (`analyze.py`, `render.py`, `transcribe.py` + the orchestrator `SKILL.md`)
Three formats (docx/HTML/markdown), regex-based Jinja2 extraction with a scope stack, config-override merging, and a seven-phase orchestration with a mandatory validation step. Changes to one stage ripple: the analyzer's manifest format is the renderer's and transcriber's input contract, and the template packs under `templates/_examples/` deliberately exercise all three render paths — use them as your regression set.

**4. Dense orchestrator SKILL.md files**
`coquill/SKILL.md`, `us-state-privacy-navigator/SKILL.md`, `contract-qa/SKILL.md`, `foreign-law-research/SKILL.md`, `legal-translation/SKILL.md`, and `statutory-analysis/SKILL.md` are rated complex as prose programs: many phases, branching logic, and pointers into references and scripts that must stay in sync. If you rename a reference or change a script flag, grep the SKILL.md for stale pointers.

**5. Deep reference corpora**
The heaviest knowledge files: `legal-translation/references/document-type-library.md` and `legal-language-conventions.md` (12 document types x 9 languages), both MSA `issue_checklist.md` files, `nist-ai-rmf/references/gai-profile/glossary.md`, `statutory-analysis/references/practical_lessons.md` and `statutory_structure.md`, `us-state-privacy-navigator/references/defense-arguments.md`, and `foreign-law-research/references/resources.md`. These carry substantive legal content — edits need a qualified contributor and the higher review bar in `skills/CONTRIBUTING.md`.

**6. Structured intake schema** (`assets/applicability-questions.json`)
The JSON Schema whose required fields drive the navigator's threshold tests. Changing a field here changes what `applicability_check.py` receives — update schema, script, and fixtures together.

---

## Your First Contribution

1. Read `CONTRIBUTING.md`, `ACCESS-MODES.md`, and `PR-READINESS.md` (tour step 13).
2. Walk tour steps 2 and 5 to learn the minimal and review-pattern skill shapes.
3. To author a new skill, run the `skill-creator` skill — it elicits purpose, triggers, inputs, outputs, and edge cases conversationally, then scaffolds the folder.
4. Ship the full anatomy: SKILL.md + references + at least one worked example, plus `evals.yaml` or a `test-plan.md`. Scripts require tests.
5. Before opening the PR, run the `PR-READINESS.md` checklist — especially content hygiene (no firm, client, or personal names) and cross-harness install checks.

*Generated by Understand-Anything from `knowledge-graph.json` (341 nodes, 7 layers, 13 tour steps) and `domain-graph.json` (6 domains).*

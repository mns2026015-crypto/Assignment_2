# Prompt 06 — Dataset and Baseline Validation Sheet

**Project:** Knowledge Cutoff Effects on LLM-Generated Literature Reviews in Rapidly Evolving Fields  
**Prepared:** 2026-11-07  
**Status:** Evidence-based recommendation; dataset selection and model access are not yet confirmed.  
**Depends on:** Prompt 05 research brief (G-2, controlled comparison; fixed-passage RAG versus the same RAG output plus claim-level verifier).

---

## 1. Research question carried forward from Prompt 05

**RQ1 (primary):** On identical mixed-date scientific passages, does adding a claim-level verifier to a fixed-passage RAG system reduce factual claims whose only supporting evidence comes from source versions first publicly available after the requested cutoff?

**RQ2 (secondary):** Does verification preserve citation support and coverage of relevant, eligible pre-cutoff evidence, and how often does it remove valid eligible claims?

**Task boundary:** A bounded, short **multi-paper scientific synthesis** on one narrow scientific subtopic. This is not a full systematic review. Single-paper question answering may be used only as a verifier-mechanics pilot; its results cannot be presented as evidence about multi-paper synthesis.

**Core controlled comparison:**

- **A — Baseline:** fixed-context RAG with a cutoff instruction and source-ID citations.
- **B — Proposed condition:** the exact answer from A is passed to a claim-level verifier, together with the same question, cutoff, ordered passages, IDs and dates. The verifier checks support and date admissibility separately, and may retain, remove or mark a claim uncertain. It must not add facts or retrieve new sources.

The two core arms must receive the same passages in the same order. This isolates the incremental effect of post-generation verification. The second comparator below is an optional additional arm, not a silent change to the Prompt 05 core comparison.

## 2. Dataset recommendation

### Primary recommendation: a small, curated, versioned arXiv corpus

**Dataset type:** Build a task-specific corpus from selected arXiv records rather than treating a large static benchmark as the temporal dataset.

**Proposed topic slice:** Retrieval-augmented scientific synthesis and automated literature-review/survey generation. This keeps the task close to the project topic while providing an evolving research area with identifiable pre- and post-cutoff papers.

**Why arXiv is the strongest first candidate:**

1. arXiv preserves numbered versions of papers. Its documentation says that replacements create new versions and previous versions remain part of the historical record. Each version can be cited and retrieved by a versioned identifier such as `YYMM.NNNNNv2`.
2. The arXiv API exposes the first-version `published` date and the selected version's `updated` date; a particular version can be requested by appending `vN` to the ID. This is useful for reconstructing what source version was available by a historical cutoff.
3. arXiv provides metadata through its API/OAI-PMH and full-text access routes for PDFs and source files. It recommends using `export.arxiv.org` for programmatic harvesting and warns against attempting to download the entire corpus programmatically.

Official documentation:
- [arXiv API User Manual](https://info.arxiv.org/help/api/user-manual.html)
- [arXiv Submission Version Availability](https://info.arxiv.org/help/versions.html)
- [arXiv Bulk Data Access](https://info.arxiv.org/help/bulk_data.html)

**Initial date split to test during the metadata audit (not frozen):** `2025-06-30 23:59:59 UTC`.

This is a practical candidate because a focused search already identifies relevant pre-cutoff works such as *LitLLM* (2024-02-02), *AutoSurvey* (2024-06-10), and *SurveyX* (2025-02-20), plus post-cutoff works such as *Open-Source Agentic Hybrid RAG Framework for Scientific Literature Review* (2025-07-30), *LiRA* (2025-10-01), and *InsightRAG* (2026-10-20). These examples demonstrate that the topic has papers on both sides of the candidate date; they do **not** establish that the final corpus is large enough or that every paper's full-text rights have been checked.

Example discovery records:
- [LitLLM, arXiv:2402.01788](https://arxiv.org/abs/2402.01788)
- [AutoSurvey, arXiv:2406.10252](https://arxiv.org/abs/2406.10252)
- [SurveyX, arXiv:2502.14776](https://arxiv.org/abs/2502.14776)
- [Open-Source Agentic Hybrid RAG Framework, arXiv:2508.05660](https://arxiv.org/abs/2508.05660)
- [LiRA, arXiv:2510.05138](https://arxiv.org/abs/2510.05138)
- [InsightRAG, arXiv:2610.28085](https://arxiv.org/abs/2610.28085)

**Important temporal rule:** Treat the arXiv version date as the operational source-version date only after checking the exact version's submission/public availability record. The API's `published` field refers to the first version submitted and processed; `updated` refers to the requested version. Do not use today's latest PDF for a historical cutoff. For pre-cutoff papers, use the latest version publicly available on or before the cutoff; for post-cutoff papers, use the first version made public after the cutoff. Exclude records whose availability falls inside a predeclared boundary window (for example, 48 hours around the cutoff) unless their public announcement time can be verified.

**Provisional inclusion criteria:**

- English-language research paper with a stable arXiv ID and version history.
- Direct relevance to scientific literature review generation, scientific synthesis, RAG for scholarly evidence, or claim/citation verification in scientific synthesis.
- Full text is accessible in a form that can be parsed reliably (LaTeX source where available; otherwise PDF with manual extraction checks).
- An exact version and source-date record can be documented.
- License/rights information has been inspected and recorded for each paper.
- The paper is not a duplicate of another record or a later venue copy being counted as an independent source.

**Provisional size target (subject to availability):** First run a pilot with approximately 12–20 distinct papers and 6–8 synthesis questions. If the topic audit finds enough relevant records, expand toward 25–40 papers and 10–15 held-out questions. These are planning targets, not measured dataset counts or a claim of statistical power. Each test question should have multiple relevant pre-cutoff sources and at least one relevant post-cutoff source; otherwise it cannot test the primary comparison meaningfully.

### How to retrieve and represent the arXiv corpus

**Discovery queries (run separately, then deduplicate by arXiv ID):**

```text
(ti:"literature review" OR abs:"literature review") AND (cat:cs.CL OR cat:cs.AI OR cat:cs.IR)
(ti:"scientific literature" OR abs:"scientific literature") AND (cat:cs.CL OR cat:cs.AI OR cat:cs.IR)
(ti:"survey generation" OR abs:"survey generation") AND (cat:cs.CL OR cat:cs.AI)
(ti:"scientific synthesis" OR abs:"scientific synthesis") AND (cat:cs.CL OR cat:cs.AI OR cat:cs.IR)
```

The final query strings, retrieval date, API response and total-result counts must be saved. Screen titles/abstracts first, then inspect the full text before inclusion. These are starting search strings, not proof of complete coverage.

**Expected record structure:**

- Metadata from arXiv API/OAI-PMH: arXiv ID, title, authors, categories, abstract, first-version date, selected-version date, version number, DOI/journal reference where available, and license identifier where available.
- Full text: exact version PDF and/or source archive; keep source paper identity separate from version identity.
- Parsed text: section-aware passages with page/section locations and stable passage IDs.
- Provenance: source URL, exact version URL, retrieval timestamp, file hash, parser and parser version, license/rights note, inclusion/exclusion reason.

Suggested manifest fields:

```text
paper_id, arxiv_id, version, title, authors, primary_category,
v1_date_utc, selected_version_date_utc, cutoff_utc, date_status,
abs_url, version_url, pdf_url, license_uri, license_checked,
doi, journal_ref, retrieval_timestamp_utc, file_sha256,
parser_name, parser_version, included, exclusion_reason
```

**Version handling:** A paper is one scholarly work, not a new independent paper for every arXiv version. Select only one version of a given arXiv ID in each frozen evidence set: the latest version available by the cutoff for an eligible pre-cutoff paper, or the first publicly available version for a post-cutoff paper. Preserve the version ID and hash. Do not place multiple revisions of the same paper in the same context as if they were independent evidence.

**License and reuse caution:** arXiv's documentation states that the default arXiv license does not transfer copyright to arXiv or give arXiv the right to sublicense the full text to others; it says tools/indexes based on full text must link back to arXiv for downloads. Alternative license information is available in OAI-PMH metadata. Therefore, do not assume that all arXiv PDFs may be redistributed. Record each paper's license, keep any full-text corpus local unless its specific license permits redistribution, and publish source IDs/links and a manifest rather than republishing PDFs. Confirm that the intended model/API processing and course repository use comply with the relevant terms. This is a research workflow recommendation, not legal advice.

### Alternative dataset candidates

| Candidate | Format and task fit | Access/licensing notes | Why it is not the primary temporal corpus |
|---|---|---|---|
| **Multi-XScience** | A ready-made scientific multi-document summarization dataset. Its inputs include a target paper abstract and abstracts of cited papers; the target is a related-work section. The repository reports 30,369/5,066/5,093 train/validation/test examples. | Public repository with MIT license for the repository; inspect underlying source-paper terms before redistributing source text. | Strong for testing the multi-document input/output shape, but it is abstract-based and does not provide the exact version history/cutoff provenance needed by RQ1. Use as a development sanity check, not the primary temporal test set. [Official repository](https://github.com/yaolu/Multi-XScience). |
| **ScholarQABench** | Multi-paper scientific question-answering benchmark with evaluation resources; useful for examples of evidence-based scientific questions and answer rubrics. | The repository states that aggregate test cases and sample system answers are ODC-BY; constituent datasets retain their own licensing requirements. It also notes expert-written answers/rubrics were created July–September 2024. | Questions/answers are not themselves an as-of versioned corpus. Its notes warn that a retrieval system may surface papers published after the original experts wrote the reference answers. Use only as an optional question/rubric source after checking each source and rebuilding temporal labels. [Official repository](https://github.com/AkariAsai/ScholarQABench). |
| **PMC Open Access Subset** | Full-text biomedical papers in machine-readable JATS XML; attractive if the project narrows to biomedical literature synthesis. | Full text is available through supported PMC services for articles with reuse rights, but licenses vary by article. PMC OAI-PMH supports full-text JATS XML; automated retrieval must use the approved PMC services and follow the stated request limits. | PMC's user guide provides publication/electronic-publication and PMC-live dates, but explicitly says deposit-date data are unavailable. This weakens it for the Prompt 05 definition based on the first publicly available full-text version. Use only if the topic becomes biomedical and the cutoff is redefined to a publication-date construct. [PMC OAI-PMH](https://pmc.ncbi.nlm.nih.gov/tools/oai/) and [PMC date fields](https://pmc.ncbi.nlm.nih.gov/about/userguide/). |
| **ACL Anthology** | Convenient NLP-focused papers with downloadable PDFs and machine-readable metadata. | ACL says materials published from 2016 onward are under CC BY 4.0; pre-2016 ACL materials use CC BY-NC-SA 3.0. The policy does not cover third-party materials. | Proceedings publication dates are useful, but this is not a complete version-history archive. Could be a fallback only if the domain narrows to NLP and the cutoff is defined by proceedings publication. [Copyright FAQ](https://aclanthology.org/faq/copyright/). |

**Dataset decision:** Proceed with a curated, versioned arXiv corpus as the primary candidate. Use Multi-XScience only to sanity-check the multi-document synthesis task format. Do not mix PMC or ACL papers into the primary corpus unless their date semantics are explicitly normalized and the domain decision is revised.

---

## 3. Baselines and comparators

The target is to compare an unchanged generator and evidence set while isolating the verification step. The assignment asks for two meaningful baselines/comparable controls where feasible. The recommended design is three conditions if the data and annotation workload permit; otherwise, the original two-arm Prompt 05 comparison remains the minimum defensible controlled design.

| Condition | Role | Input/context | What it tests | Status |
|---|---|---|---|---|
| **A. Cutoff-instruction RAG** | Primary baseline | Fixed mixed-date passages; prompt instructs model to answer as of cutoff and cite passage IDs. | What a normal retrieval-conditioned generator does when asked to respect a cutoff without a separate verifier. | **Mandatory core baseline.** |
| **B. RAG + claim-level verifier** | Proposed treatment | Exact answer from A plus the same question, cutoff, ordered passages, passage IDs and source dates. Verifier cannot search or add facts. | Incremental effect of checking each generated claim for source support and date admissibility after generation. | **Mandatory core treatment.** This is not an already demonstrated project contribution. |
| **C. As-of-filtered RAG** | Second comparator/control | Before retrieval/context assembly, exclude all source versions first available after cutoff; use the same generator and answer format. | Whether retrieval-time date exclusion is more/less useful than post-generation verification, and what support/coverage it sacrifices. | **Recommended optional comparator** if API calls and annotations are affordable. It is not part of the frozen two-arm core until explicitly added in Prompt 07. |

**Why not simply compare three unrelated published systems?** The proposed question concerns the incremental effect of a verifier. Changing the generator, retrieval method, model size and corpus at the same time would make the effect uninterpretable. Conditions A and B therefore form the matched primary comparison; C adds a useful temporal retrieval control without replacing that comparison.

### Implementation complexity and approximate compute

- **Indexing/retrieval:** For a small corpus (dozens of papers), PDF/LaTeX parsing, BM25 or a compact embedding model, and a FAISS index are modest compared with model generation. A CPU-based preprocessing/indexing pilot should be possible on an ordinary laptop, but actual memory/runtime must be checked on the student's machine.
- **Generation and verification:** The main resource cost is model inference. An API-based model avoids local GPU requirements but requires working API access, a budget, token limits and permission to process the selected paper text. A local open-weight model avoids per-call API billing but introduces VRAM/RAM and setup constraints. No provider, model, GPU, or budget is confirmed in the project notes, so a numeric cost or runtime estimate would be invented at this stage.
- **Fine-tuning:** Not required for the proposed study. A fixed prompt-based verifier keeps the implementation substantially smaller and isolates the intended component.
- **Run volume:** For an initial 6–8-question pilot, A/B require 12–16 generation runs plus 6–8 verifier calls in B. Adding C requires a further 6–8 generation runs. These are call counts only; token usage and monetary cost depend on the selected model and context size.

**Recommended simplest credible baseline:** A — fixed-context RAG with cutoff instruction, implemented using one fixed retriever and one fixed generator. Save the exact passage set/order so B can reuse it. Do not begin by reproducing or fine-tuning the complete original RAG model from Lewis et al.; that is unnecessary for this controlled post-retrieval comparison.

---

## 4. Evaluation design and data partition

This is a system-comparison study, not a supervised model-training task. A conventional train/validation/test split is therefore not required unless any component is trained or tuned on labeled examples. Use a documented development/held-out evaluation protocol instead.

**Suggested split:**

- **Development/pilot:** 2–3 questions used to debug parsing, citation format, verifier output schema and annotation rubric. These questions and their labels must not appear in the final held-out results.
- **Held-out evaluation:** initially target 6–8 synthesis questions for a feasibility pilot; expand to 10–15 or more only if the source corpus and annotation capacity support it. Label a small pilot as exploratory if it is too small for reliable inferential claims.
- Keep the full evidence set for each held-out question fixed across A and B. If C is added, it uses the same question and the same underlying candidate corpus but filters the context by the cutoff before generation.
- Split by question/topic cluster, not random sentences. Avoid near-duplicate questions and overlapping source clusters between development and held-out cases where prompt tuning could leak the gold evidence.
- Keep gold claim labels, eligible-evidence lists and annotator notes out of all model prompts. The verifier must not have web access or access to the annotation files.

**Primary and secondary metrics (retain Prompt 05 definitions):**

1. **Post-cutoff-source reliance rate (primary):** number of source-supported factual claims whose only supporting evidence is from source versions first available after the cutoff, divided by all source-supported factual claims. Also report the numerator and denominator and the unsupported/uncited-claim rate separately.
2. **Citation-support precision:** supported claim-to-passage links divided by all evaluated claim-to-passage links. A citation that is relevant but does not entail/support the claim is not counted as supported.
3. **Eligible pre-cutoff evidence coverage:** number of gold eligible pre-cutoff evidence units represented by supported claims divided by all gold eligible pre-cutoff evidence units for that question.
4. **Over-filtering:** eligible, supported baseline claims removed or materially weakened by the verifier divided by the eligible, supported baseline claims.

The Prompt 05 thresholds (20% relative reduction in the primary measure, no more than 5 percentage-point decline in citation support or eligible evidence coverage, and no more than 10% over-filtering) remain **provisional project decision rules**. Re-check feasibility and denominator stability on the pilot before freezing them in Prompt 07; do not tune them after inspecting held-out outcomes.

---

## 5. Leakage risks and controls

| Risk | Why it matters | Control |
|---|---|---|
| **Wrong version used for cutoff** | Today's latest PDF may contain material not present in the version available at the historical cutoff. | Store exact arXiv version IDs and timestamps; use latest version on/before cutoff for pre-cutoff papers and first public version after cutoff for post-cutoff papers. Verify ambiguous boundary cases manually. |
| **Preprint/journal duplicate** | The same work could appear as an arXiv preprint and a venue paper, falsely increasing the evidence count. | Deduplicate by arXiv ID, DOI, title and author overlap; maintain a `work_id` linking versions and venue copies. Count one work once in each evidence context. |
| **Publication date mistaken for fact-discovery date** | A later paper can describe a fact already known earlier; a post-cutoff citation alone does not prove the claim was unknowable earlier. | Label the primary metric as post-cutoff-source reliance. Only label factual temporal inadmissibility when the claim/event chronology supports that judgment independently. |
| **Version overlap** | Multiple versions of one paper may contain near-identical passages and be counted as separate sources. | One version per work per context; record version hashes and use one canonical `work_id`. |
| **Corpus/query leakage** | Reusing development questions or labels to tune the verifier can inflate held-out scores. | Separate development and held-out question clusters; do not expose gold labels or annotation notes to the models. |
| **Verifier gets extra information** | Web access, a different context, or extra source passages would confound the comparison. | No verifier web search; supply the exact same frozen evidence set, question and cutoff. Record any model difference as a protocol deviation. |
| **PDF/LaTeX extraction errors** | Tables, math and citation markers may be lost or scrambled, changing the evidence available to the model. | Preserve source files where accessible; validate parsed sections and citations on a sample; record parser/version; manually inspect problematic pages. |
| **Copyright/licensing** | Public availability does not imply permission to mirror every full-text article or send it to every third-party service. | Record per-paper license and API terms; keep restricted full text private; use source links and IDs in public outputs; check provider data-use terms. |
| **Retractions/corrections** | A retracted paper or later correction can change whether a claim is reliable. | Record retraction/correction state and date; exclude retracted sources from the primary corpus or predefine a separate sensitivity analysis. |

---

## 6. Verification checklist before Prompt 07

### Dataset checks

- [ ] Run the arXiv queries and save the exact query strings, API responses and retrieval timestamp.
- [ ] Confirm that the selected subtopic has enough genuinely relevant papers on both sides of the candidate cutoff.
- [ ] Confirm each included paper's exact arXiv ID, first version date, selected version date, version number and full-text availability.
- [ ] Manually verify cutoff-boundary timestamps and exclude ambiguous records.
- [ ] Check duplicate preprint/journal versions and record one canonical work ID.
- [ ] Inspect each source's license metadata and decide whether it can be processed, stored locally and/or redistributed.
- [ ] Confirm that every test question has both relevant eligible pre-cutoff evidence and relevant post-cutoff evidence.
- [ ] Verify text extraction and citation/passages alignment on at least three sample papers.

### Baseline and environment checks

- [ ] Select one generator and one version; record access route, limits and cost.
- [ ] Confirm whether the generator and verifier can use the same model version.
- [ ] Confirm that the same retriever, context budget, passage order and decoding settings can be held fixed across A and B.
- [ ] Build one end-to-end smoke test that produces an answer with passage IDs and a verifier JSON result.
- [ ] Confirm the annotation rubric and whether one or two independent annotators are available.
- [ ] Estimate actual token use and run cost from the smoke test, not from guessed prices.

### Decision gate

**GO to Prompt 07** only when a small licensed/date-audited corpus, a reachable model, and a workable annotation plan exist. Otherwise, retain the arXiv plan as provisional and either reduce the scope or switch to a clearly labeled mechanics-only pilot. Do not quietly replace multi-paper synthesis with single-paper QA and then claim the same research question was tested.

---

## 7. Final recommendation

**Recommended primary dataset:** curated, versioned arXiv papers on RAG-based scientific synthesis and automated literature-review generation, using a provisional cutoff of 2025-06-30 UTC for the initial metadata feasibility audit.

**Mandatory core comparison:** A (cutoff-instruction fixed-context RAG) versus B (the same RAG output plus a claim-level verifier).

**Second comparator:** C (as-of-filtered RAG) is recommended as an optional third condition because it represents a meaningful retrieval-time control. Add it only if the dataset, API budget and annotation time permit; otherwise justify the matched A/B design as the appropriate controlled comparison for isolating the verifier.

**Current go/no-go:** Conditional GO for dataset metadata collection and a small smoke-test plan; **not yet GO for the full experiment**. Full-text licensing, the exact final corpus, API/model access, local hardware and annotation capacity remain unconfirmed.

## 8. Evidence status

| Statement | Status |
|---|---|
| arXiv API exposes first-version and selected-version date fields; specific versions can be requested. | **VERIFIED — official arXiv API documentation.** |
| arXiv preserves version history and public versions can be cited by versioned ID. | **VERIFIED — official arXiv version documentation.** |
| The default arXiv license does not grant third parties general redistribution rights; full-text tools must link back to arXiv. | **VERIFIED — official arXiv bulk access documentation.** |
| Multi-XScience is a multi-document related-work generation dataset with abstract/reference-abstract inputs and a published train/validation/test breakdown. | **VERIFIED — dataset repository.** |
| ScholarQABench aggregate data use ODC-BY, constituent datasets have separate terms, and reference answers were created July–September 2024. | **VERIFIED — official dataset repository.** |
| PMC offers JATS full text for eligible reuse articles but does not provide deposit-date data. | **VERIFIED — official PMC documentation.** |
| arXiv is the best match for this project's exact version-date requirement. | **INFERRED — comparative assessment.** |
| The selected arXiv subtopic will yield enough usable pre/post-cutoff papers and questions. | **UNKNOWN — requires actual metadata and full-text screening.** |
| Model/API access, price, hardware runtime, and independent annotation capacity. | **UNKNOWN — student confirmation required.** |

---

## 9. Official sources consulted

1. arXiv, *API User Manual*: https://info.arxiv.org/help/api/user-manual.html
2. arXiv, *Submission Version Availability*: https://info.arxiv.org/help/versions.html
3. arXiv, *Bulk Data Access*: https://info.arxiv.org/help/bulk_data.html
4. arXiv paper record, *LitLLM*: https://arxiv.org/abs/2402.01788
5. arXiv paper record, *AutoSurvey*: https://arxiv.org/abs/2406.10252
6. arXiv paper record, *SurveyX*: https://arxiv.org/abs/2502.14776
7. arXiv paper record, *Open-Source Agentic Hybrid RAG Framework*: https://arxiv.org/abs/2508.05660
8. arXiv paper record, *LiRA*: https://arxiv.org/abs/2510.05138
9. arXiv paper record, *InsightRAG*: https://arxiv.org/abs/2610.28085
10. Multi-XScience dataset repository: https://github.com/yaolu/Multi-XScience
11. ScholarQABench repository: https://github.com/AkariAsai/ScholarQABench
12. PMC OAI-PMH documentation: https://pmc.ncbi.nlm.nih.gov/tools/oai/
13. PMC User Guide (dates and license filters): https://pmc.ncbi.nlm.nih.gov/about/userguide/
14. ACL Anthology copyright FAQ: https://aclanthology.org/faq/copyright/

---

**Prompt 06 deliverable status:** Drafted and source-checked. It recommends a primary data route, explains alternatives, specifies baseline/comparator conditions, and records data leakage, split, licensing and resource checks. No corpus has been downloaded, no model has been executed, and no empirical result is claimed.

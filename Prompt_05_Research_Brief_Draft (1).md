# Prompt 05 — One-Page Research Brief (Working Draft)

**Project:** Knowledge Cutoff Effects on LLM-Generated Literature Reviews in Rapidly Evolving Fields  
**Date:** 2026-11-06 | **Status:** Draft for review; not a frozen protocol

## Selected gap and study

**Working gap:** G-2, correct use of retrieved evidence, narrowed to claim-level verification of citation support and source-date admissibility. This is an inferred research direction, not a novelty claim. **Pathway B:** controlled comparative study.

**Working title:** *Evaluating Claim-Level Temporal and Citation Verification in Retrieval-Augmented Scientific Synthesis.*

## Research problem

Retrieval-augmented language models can receive relevant scientific passages without using them faithfully. In a cutoff-sensitive task, a generated claim may also depend on a cited paper version unavailable by the requested cutoff. This study asks whether adding a post-generation, claim-level verifier to fixed-passage RAG reduces claims supported only by post-cutoff source versions while preserving citation support and coverage of relevant, eligible pre-cutoff evidence.

The primary task should be a bounded, short **multi-paper synthesis** on one narrow scientific subtopic, not a full literature review across a field. Single-paper QA may be used only as a verifier-mechanics pilot; its results cannot be generalized to multi-paper synthesis.

## Research questions, variables and decision rules

**RQ1 — Primary:** On identical mixed-date passages, does adding a claim-level verifier reduce the proportion of factual claims whose only supporting evidence comes from source versions first publicly available after the requested cutoff?

- **Input/comparator:** Held-out synthesis questions and fixed, ordered pre-/post-cutoff passages; compare (A) RAG with a cutoff instruction against (B) the same RAG output followed by a verifier.
- **Independent variable:** System arm (RAG vs. RAG + verifier). **Dependent variable:** post-cutoff-source reliance rate—the proportion of source-supported factual claims supported only by post-cutoff versions. Unsupported/uncited claims are reported separately.
- **Hypothesis H1:** The verifier lowers this rate. **Provisional success rule:** at least 20% relative reduction versus baseline, with the absolute percentage-point change also reported. If baseline reliance is near zero, revise the decision rule before confirmatory runs.
- **Contrary evidence:** no reduction, an increase, or an effect below the predeclared practical threshold.

**RQ2 — Secondary:** Does verification preserve citation support and coverage of relevant pre-cutoff evidence, and how often does it remove valid eligible claims?

- **Input/comparator and independent variable:** Same paired questions, passages and two arms as RQ1.
- **Dependent metrics:** citation-support precision over claim–passage pairs; eligible-evidence coverage (gold relevant pre-cutoff evidence units represented by supported output claims); and over-filtering (eligible-supported baseline claims removed or weakened by the verifier).
- **Hypothesis H2:** RQ1's reduction occurs without material damage to evidence quality or coverage.
- **Provisional success rule:** support precision and eligible-evidence coverage each decline by no more than 5 percentage points; over-filtering does not exceed 10% of baseline claims independently judged to have eligible support. These are proposed project tolerances, not literature-established standards.
- **Contrary evidence:** either quality/coverage measure falls beyond its margin, or over-filtering exceeds the limit.

An as-of-filtered retrieval arm is optional and should be added only if data, annotation and run resources permit. It is not part of the core two-arm comparison.

## Controls and interpretation

Keep question, exact passage text and order, passage IDs, cutoff, generator/version, decoding settings and output format identical across arms. The verifier receives the baseline answer and the same passages, IDs and cutoff; it judges source support separately from date admissibility, may retain/remove/mark claims uncertain, and must not add facts or retrieve external evidence.

Choose one availability rule before collection; a candidate is the first publicly accessible full-text version. Record exact versions and dates, and do not mix preprint dates with formal publication dates. A later source supporting a fact does not prove the fact was unknowable earlier. Therefore the primary measure is **post-cutoff-source reliance**, not proof of parametric knowledge leakage.

## Contribution, evidence and unresolved items

The review and student-rechecked sources motivate the distinction between retrieval access and faithful evidence use; AutoSurvey analyzes unsupported claims, OpenScholar reports residual evaluation/retrieval limitations, and TCFT studies temporal critique in QA rather than literature-review generation. The intended contribution is a controlled evaluation of a verifier in a bounded, dated, multi-paper synthesis task—not a new verification algorithm or a novelty claim. The focused related-work assessment found substantial overlap; novelty remains unresolved.

**Before freezing:** confirm a lawful full-text multi-paper corpus and narrow subtopic; cutoff rule/timestamp and source-version provenance; generator/verifier access, limits and budget; annotation rubric and annotator capacity; and enough held-out questions for more than an exploratory pilot. The Prompt 04 deadline of 2026-10-30 has passed; confirm the current course deadline. Prompt 06 should verify data/comparators and Prompt 07 should freeze the protocol before result inspection. No dataset access, implementation, or experimental result is claimed here.

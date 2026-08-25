# Document-to-Record Automation — Public-Safe Case Study

> Turning inconsistent partner source documents into governed operational records — with named-human sign-off at every write gate.

| Release field | Public value |
|---|---|
| Version and date | v0.1 — 2026-08-22 |
| Primary capability signals | Document-to-record automation, Human-in-the-loop governance, Deterministic QA |
| Business value status | UNKNOWN |
| Public-safety status | SAFE NOW |
| Publication state | PUBLISHED — 2026-08-25 |
| Architecture companion | [`ARCHITECTURE.md`](ARCHITECTURE.md) |

## Problem

Partner institutions send course-equivalency information in inconsistent formats — a structured PDF from one, a spreadsheet with ambiguous unit labels from another, a document with multi-course relationships that don't reduce to simple one-to-one mappings. Getting this right has historically required careful manual interpretation, and an earlier, lighter-weight attempt at automating a similar import once failed on an unstated formatting assumption before the defect was caught and fixed.

## Scope

Synthetic demonstration of source-document fingerprinting, deterministic extraction into a canonical schema, two-gate human-in-the-loop governance, and automated QA plus verification. No company or partner data is included. Production business impact has not yet been measured.

## Intervention

The pipeline moves every source document through a controlled sequence before any record is trusted:

1. **Exact-source verification** — Every source document is checked against a locked, approved fingerprint before extraction begins. A file that doesn't match a locked source is rejected before processing.
2. **Deterministic structured extraction** — Bounded, source-specific adapters read the locked inputs and emit records into a canonical schema, preserving source relationships and source-stated values rather than inventing unsupported interpretations.
3. **Automated QA and a separate verification pass** — Validation rules and automated tests check source accountability, required relationship structures, unit preservation, and record integrity. A separate pass then reloads the emitted output and independently reconciles it.
4. **Exception isolation** — Where a source genuinely contains an ambiguity, the workflow flags the record for human disposition instead of guessing at a resolution.
5. **Controlled downstream staging preparation** — After extraction approval, a no-write readiness pass compares approved records against the live target system, separates already-aligned records from proposed changes, and holds unsupported cases outside the mutation set.

**Two gates, never merged**: A named reviewer approves that extracted records faithfully reflect their source — including explicit acknowledgment of any unresolved ambiguity. Approving an extraction does not authorize writing it anywhere. A separate, later named-human decision determines whether a specific proposed downstream change is safe to apply. The two decisions are never collapsed into one.

The paired [`ARCHITECTURE.md`](ARCHITECTURE.md) explains why source identity, semantic normalization, deterministic validation, interpretation approval, mutation approval, drift checks, and downstream release are separate trust boundaries rather than one autonomous loop.

## Evidence-backed claims

- **VALIDATED — Automated synthetic regression tests verify source-fingerprint rejection, canonical-record reconciliation, unit preservation, and governed staging-write behavior.**
- **UNKNOWN — Production business impact, cycle-time reduction, and downstream partner outcomes have not yet been measured.**

## Inspect the two-layer proof

**Layer 1 — Business Case Study**

- This `README.md` — problem, controlled intervention, evidence-backed claims, limitations, and business-value boundary.

**Layer 2 — Architecture Companion**

- [`ARCHITECTURE.md`](ARCHITECTURE.md) — design problem, component map, decision log, failure modes, human/autonomy boundaries, and portable document-to-record patterns.

**Public-safe visual evidence**

- [`evidence/public-safe/document-to-record-automation-case-study.html`](evidence/public-safe/document-to-record-automation-case-study.html) — self-contained HTML case study (business narrative, evidence, technical depth, and claim-status labels in one page).

## Limitations

- Public artifacts use synthetic records only.
- The public demo is deterministic code, not a claim that concurrent agents executed.
- Production business impact is not yet measured.
- No production write has occurred, and none is authorized by this evidence. Every mutation described stayed inside a test or staging environment.
- The historical 46-row result comes from a different, non-governed precursor tool, included only as motivating context for why source-format assumptions needed to be replaced with exact-source verification. It is not evidence of this pipeline's own throughput.
- Exactly one governed staging write has been executed under this model to date. It is real, verified evidence of the end-to-end path working — not a claim of sustained or high-volume operation.
- The multi-course relationship's presentation-layer behavior is unverified and unclaimed — not because it is assumed to be fine, but because it belongs to a separate system outside this workflow's boundary.
- Cross-domain portability is an architecture hypothesis (`EXPECTED`) until validated in another public example.

## Publication boundary

This public-safe package was approved for publication on 2026-08-25. Publication does not expand any evidence or business-impact claim.

> Every material claim is labeled by how it was established. This is a feature of the evidence, not a hedge.

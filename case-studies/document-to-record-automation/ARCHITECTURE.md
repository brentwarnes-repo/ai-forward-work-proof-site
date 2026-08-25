# Document-to-Record Automation — Architecture Companion

> **Paired business case study:** [`README.md`](README.md)
>
> This companion explains the architecture behind the governed document-to-record workflow: why exact-source verification, canonical relationship modeling, deterministic QA, two distinct human approvals, and bounded staging controls are separate components rather than one autonomous extraction-and-write loop.

| Architecture field | Public value |
|---|---|
| Version and date | v0.1 — 2026-08-22 |
| Paired case study | [`README.md`](README.md) |
| Architecture scope | Source document → canonical record → reviewed staging change, with separate extraction and write gates |
| Primary architecture signals | Document-workflow architecture; deterministic controls; human-in-the-loop governance |
| Evidence status | VALIDATED for the bounded synthetic/governed behaviors described in the paired case study; production business impact remains UNKNOWN |
| Public-safety status | SAFE NOW |

## 1. The design problem

The design problem was not merely extracting rows from documents. It was preserving **source meaning** across inconsistent inputs while preventing several common automation failures:

- processing the wrong or changed source file as though it were the approved one;
- flattening a relationship that actually means “all of these courses together” or another structured relationship;
- silently converting ambiguous values into a convenient normalized answer;
- treating successful extraction as permission to mutate an operational backend;
- writing an approved change after the target state has drifted;
- allowing one automated path to collapse interpretation, approval, and mutation into a single unreviewable action.

An earlier lightweight precursor had already demonstrated why implicit source-format assumptions were unsafe. The architecture therefore treats source identity, semantic normalization, validation, human judgment, and mutation authority as different control problems.

## 2. Component map and handoffs

```text
Approved source document
        |
        v
Exact-source fingerprint gate
        |
        +---- mismatch ----> REJECT / HOLD
        |
        v
Bounded source adapter
        |
        v
Canonical relationship record
        |
        v
Deterministic validation + reconciliation
        |
        +---- ambiguity ----> HUMAN DISPOSITION
        |
        v
HUMAN GATE 1 — extraction approval
        |
        v
No-write staging readiness / proposed change set
        |
        v
Target-state re-read / drift check
        |
        +---- drift ----> FAIL CLOSED / RE-REVIEW
        |
        v
HUMAN GATE 2 — write approval
        |
        v
Bounded staging mutation
        |
        v
Exact readback + rollback evidence

Production publication / downstream presentation
        |
        v
SEPARATE RELEASE BOUNDARY
```

| Component | Responsibility | Trust / control boundary |
|---|---|---|
| Source fingerprint gate | Confirm the processor is operating on the exact approved source | A similar filename or derivative file cannot silently substitute for the locked source |
| Source-specific adapter | Read a bounded known input format | Adapter may extract; it may not invent unsupported semantics |
| Canonical relationship model | Preserve source relationships independently of physical spreadsheet rows | Combined or alternative relationships stay structural rather than being flattened for convenience |
| Deterministic validation | Check required fields, relationships, accountability, and known failure rules | Objective failures are surfaced before human approval |
| Human extraction gate | Decide whether records faithfully represent the source and disposition genuine ambiguity | Approval certifies interpretation only; it does not authorize a write |
| No-write staging preparation | Compare approved canonical records to the current target and produce a bounded proposed change | Proposal is not mutation |
| Drift check | Re-read the exact target/dependency state immediately before mutation | Changed state fails closed rather than applying a stale approval |
| Human write gate | Authorize the exact bounded mutation | Separate authority from extraction approval |
| Readback / rollback evidence | Verify the exact post-write state and preserve recovery evidence | Attempted write is not accepted as completed until verified |

**State and handoff rule.** Every transition changes the status of evidence, not merely the location of data. `extracted`, `validated`, `approved for interpretation`, `proposed for staging`, and `approved to write` are intentionally not synonyms.

**Restart / recovery rule.** Durable canonical records, explicit approval decisions, exact before-state snapshots, proposed change sets, and post-write readback make a bounded run reconstructable without relying on chat history.

## 3. Decision log

| Decision | Chosen design | Rejected / deferred alternative | Why | Evidence / observation |
|---|---|---|---|---|
| Source identity | Lock exact source fingerprints before extraction | Trust filenames, appearance, or a presumed stable format | A precursor failure showed that an unstated source-format assumption can produce a technically successful but wrong transformation | Paired case study records the historical precursor failure and current fingerprint rejection behavior |
| Semantic model | Normalize into a canonical relationship schema | Treat each physical source row as one simple equivalency | Source documents can express combined or alternative relationships that do not map safely to one-row/one-record assumptions | Bounded workflow preserves relationship structures and source-stated values |
| Ambiguity handling | Isolate ambiguity for named-human disposition | Guess or silently coerce ambiguous units/mappings | A clean output is less valuable than a faithful one when the source itself is unclear | The workflow explicitly holds unsupported cases outside the mutation set |
| Approval model | Two independent human gates | One approval after extraction that implicitly authorizes writing | Correct interpretation and authorization to change an operational system are different risk decisions | Extraction approval never authorizes mutation; write approval is later and scope-specific |
| Mutation model | Staging-first, bounded writes with drift checks and readback | Write directly from parser output or apply a previously approved diff without re-reading target state | Target state can change between review and execution; mutation scope must stay inspectable and reversible | Governed staging write used preflight re-read, exact bounded change, and final-state verification |
| System boundary | Stop at controlled backend preparation/write; keep downstream presentation separate | Extend the MVP into website rendering or production publication | Rendering and publication have different owners, risks, and acceptance criteria | Downstream presentation is explicitly unclaimed and outside this workflow boundary |

## 4. Failure modes and guardrails

| Failure mode | What breaks without the design | Guardrail | Residual limitation |
|---|---|---|---|
| Wrong source processed | Valid-looking output can be built from an unapproved or changed document | Exact-source fingerprint gate | New legitimate versions require a new explicit lock/review |
| Source meaning flattened | Multi-course/alternative relationships can become false independent records | Canonical structural relationship model | Novel semantics still require schema evolution or human review |
| Ambiguous source silently normalized | Automation creates false certainty | Exception isolation and named-human disposition | Human judgment remains necessary for genuinely new ambiguity |
| Extraction approval becomes write authority | Reviewer signs off on meaning and unintentionally authorizes operational mutation | Two separate named-human gates | Adds deliberate human latency, which is acceptable for the risk boundary |
| Target drifts after approval | A previously correct proposed diff becomes stale | Immediate pre-write target/dependency re-read and fail-closed drift check | Requires reliable target read access at execution time |
| Write attempt overstated as success | A partial/failed mutation could be reported as completed | Exact post-write readback and preserved rollback evidence | Production-scale write behavior is not established by the bounded staging evidence |
| Project absorbs downstream display concerns | Release scope expands into unrelated renderer/website work | Hard project boundary at backend preparation/write | Downstream behavior may still need a separate proof/workstream |

**Load-bearing constraint.** Interpretation correctness and mutation authority must remain separate. The system is designed so that a high-quality extraction cannot “earn” the right to write by momentum.

**False-completion defense.** The pipeline requires observable evidence at multiple boundaries: exact source match, deterministic QA, named-human extraction decision, bounded proposed change, pre-write state match, named-human write approval, and final readback. Producing an output file is therefore not equivalent to completing the workflow.

## 5. Human judgment, autonomy, and governance

| Decision class | System behavior | Why |
|---|---|---|
| Exact-source match and deterministic validation | Automatic / deterministic | Objective rules should be reproducible and testable |
| Known bounded source transformation | Deterministic adapter | Stable known structure does not require open-ended model judgment |
| Material source ambiguity | Human disposition | Meaning cannot be inferred safely from convenience or precedent alone |
| Extraction acceptance | Named-human approval | Confirms source fidelity and exception handling |
| Exact staging mutation | Separate named-human approval | Consequential write authority is independent from interpretation quality |
| Production promotion / public release / downstream rendering | Outside this release | Different blast radius, owners, and evidence requirements |

- **Human review gate 1:** Approve/correct/reject the extracted canonical records and explicitly acknowledge unresolved ambiguity.
- **Human review gate 2:** Authorize the exact proposed staging change after current target state has been verified.
- **Evidence boundary:** Deterministic tests support bounded technical claims; business impact remains `UNKNOWN` until measured.
- **Privacy / security boundary:** Public artifacts use synthetic/generalized records and omit private partner files, operational links, and internal system identifiers.
- **Auditability:** Source fingerprints, canonical records, validation output, approval decisions, proposed diffs, before-state snapshots, write execution, readback, and rollback evidence are separate durable artifacts in the private implementation record.

## 6. What is portable

The portable architecture is a **governed document-to-record pattern**.

| Pattern | Portable? | What changes elsewhere | What stays invariant |
|---|---|---|---|
| Exact-source lock | YES | File type, fingerprint method, source authority | Transformation begins only from an explicitly approved source identity |
| Canonical semantic layer | YES | Domain schema and relationship types | Source layout is separated from downstream operational representation |
| Exception isolation | YES | Domain-specific ambiguity rules | Uncertainty is surfaced rather than guessed away |
| Two-gate approval | YES for consequential writes | Reviewer roles and approval criteria | Interpretation approval and mutation approval remain separate |
| Drift-checked write | YES | Target system and fields | Re-read current target state before applying a previously reviewed change |
| Readback / rollback evidence | YES | Mutation API/tool and recovery method | A write is complete only after the intended final state is verified |

**Reuse test — EXPECTED.** The same architecture could support contract-to-CRM updates, policy-document-to-control records, vendor catalog imports, or another document-heavy workflow where source meaning matters and downstream writes are consequential. The schema, adapters, validation rules, and approval roles would change; the control sequence would remain.

## 7. Evidence, claims, and limitations

| Architecture claim | Evidence | Status | Limitation |
|---|---|---|---|
| Wrong-source inputs can be rejected before extraction | Synthetic regression behavior described in [`README.md`](README.md) | VALIDATED | Public package does not expose private source files |
| Canonical-record reconciliation and source-value preservation are testable | Synthetic regression behavior described in [`README.md`](README.md) | VALIDATED | Bounded known cases only |
| Extraction and write approval are separate controls | Paired case study + governed workflow record | OBSERVED | Human gates add operational latency by design |
| A bounded staging write can be drift-checked, executed, and read back under this model | Paired case study | OBSERVED | Exactly one governed staging write; not sustained production evidence |
| Production business impact or cycle-time improvement | No public measurement | UNKNOWN | No impact claim is made |
| Downstream presentation behavior | Outside project boundary | UNKNOWN / NOT CLAIMED | Requires separate system-level validation |
| Cross-domain portability | Architecture analysis only | EXPECTED | Not yet validated publicly in a second document domain |

## 8. Architecture signal

- [x] Workflow diagnosis and redesign
- [x] Component / responsibility design
- [x] Data-flow and handoff design
- [x] Deterministic gates and evaluation
- [x] Human-in-the-loop governance
- [x] Failure-mode and recovery design
- [x] State, restartability, and auditability
- [x] Reusable architecture / operating pattern
- [x] Business-to-technical translation

**Primary architecture evidence.**

1. **Semantic architecture:** source-document layout is decoupled from a canonical relationship model so downstream logic does not inherit fragile file-specific assumptions.
2. **Governed execution:** interpretation, mutation authorization, and production/public release are separate trust boundaries rather than one autonomous loop.
3. **Operational safety:** exact-source locks, deterministic QA, drift checks, bounded writes, and final readback convert known failure modes into explicit controls.

## 9. Release check and links

- [x] The paired Business Case Study exists and links are public-safe.
- [x] Architecture claims are evidence-backed or explicitly qualified.
- [x] The decision log reflects documented controls and observed failure history.
- [x] Failure modes and residual limitations are visible.
- [x] Private source files, partner identities, internal links, and security-sensitive implementation details are excluded.
- [x] Public-safety status is `SAFE NOW` for this document as written.
- [x] Public release was approved on 2026-08-25. Publication does not expand the evidence or business-impact boundary.

- **Paired business case:** [`README.md`](README.md)
- **Self-contained public-safe case study:** [`evidence/public-safe/document-to-record-automation-case-study.html`](evidence/public-safe/document-to-record-automation-case-study.html)
- **Private implementation/evidence:** intentionally not linked from this public-safe companion

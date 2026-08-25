# One-Click Multi-Item Purchase Path — Architecture Companion

> **Paired business case:** [`README.md`](README.md)
>
> Extend a legacy purchase flow without replacing stable rendering or checkout behavior, isolate new logic behind a narrow compatibility layer, and keep rollout decisions separate from technical feasibility.

**Status:** Published public-safe proof — 2026-08-25. Public-safety status: `SAFE NOW` for this document as written.

## 1. Design problem

The existing storefront already owned several responsibilities that should not be duplicated casually:

- a data-driven buy-button path;
- page rendering;
- normal single-item purchase behavior; and
- checkout, including pricing and membership logic.

The missing capability was narrow: one existing buy action could identify only one item even when the business relationship required a paired purchase.

A replacement storefront component or custom checkout would have increased regression surface without being necessary to prove the capability. The design therefore had four constraints:

1. preserve the existing rendering path;
2. preserve normal single-item behavior;
3. reuse native cart and checkout behavior; and
4. keep the new multi-item path removable and independently reviewable before rollout.

## 2. Component map

```text
Existing data-driven buy-button source
              |
              | multi-item identifiers only when needed
              v
Existing renderer / link surface
              |
              v
Narrow click-time compatibility layer
        |                     |
        | single item         | multi-item pair
        v                     v
Existing behavior       Sequential native cart adds
                              |
                              v
                        Native checkout
                              |
                              v
                   Existing pricing / membership /
                         checkout responsibility

Any failure, expansion, or rollout question
              |
              v
      Engineering / human gate
```

| Component | Responsibility | Boundary |
|---|---|---|
| Data contract | Represent more than one item in the existing link when a pair is required | Does not change rendering or checkout |
| Existing renderer | Continue rendering the purchase link | Intentionally unchanged |
| Compatibility layer | Detect qualifying multi-item actions and sequence native cart-add behavior | Does not own pricing, membership, or checkout |
| Existing single-item path | Handle ordinary one-item purchases | New logic deliberately leaves it alone |
| Native checkout | Own the standard checkout experience | Not duplicated or reimplemented |
| Engineering / human gate | Decide whether unresolved QA/failure/deployment questions permit a pilot | Technical feasibility is not rollout authorization |

**Recovery boundary:** the validated proof does not establish atomic recovery if a later item fails after an earlier item has already been added. That remains an explicit open issue.

## 3. Key design decisions

| Decision | Chosen design | Why |
|---|---|---|
| Represent the pair | Extend the existing purchase-link data contract | Avoid changing a renderer that already works |
| Scope interception | Act only on multi-item links | Keep the stable single-item path outside the regression surface |
| Checkout ownership | Reuse native cart + checkout | Avoid duplicating pricing, membership, and checkout rules |
| Rollout posture | Bounded test plus later Engineering/business gate | One successful run does not answer production-readiness questions |

## 4. Failure modes and guardrails

| Failure mode | Current guardrail / disposition | Residual limitation |
|---|---|---|
| Regress ordinary purchases | Adapter activates only for multi-item links | Broader regression testing still needed before pilot |
| Duplicate checkout responsibilities | Hand control back to native checkout | Downstream checkout remains authoritative |
| Partial multi-item add | Risk explicitly held for Engineering decision | No live-validated atomic rollback yet |
| Malformed / duplicate identifiers | Named open validation requirement | Real identifier convention/live behavior needs confirmation |
| Analytics divergence | Analytics parity is a named open item | No current parity evidence |
| “Worked once” becomes “ready to scale” | Feasibility, pilot authorization, and rollout are separate gates | Engineering review pending |
| Untested device/account states | Explicit QA gaps remain visible | Coverage incomplete |

## 5. Evidence and claim status

| Claim | Evidence | Status | Limitation |
|---|---|---|---|
| A paired purchase can traverse the adapter and reach existing checkout | Documented bounded test | `VALIDATED` | One two-item scenario on a test surface |
| Existing single-item behavior remained intact in that run | Documented bounded test | `OBSERVED` | Not a broad regression suite |
| Existing membership behavior remained present after handoff | Documented bounded test | `OBSERVED` | One membership-bearing scenario |
| Adapter avoids reimplementing checkout responsibilities | Architecture boundary + implementation review | `OBSERVED` | Does not validate every downstream state |
| Partial-failure recovery is production-ready | No approved/live validation | `UNKNOWN` | Explicitly open |
| Analytics parity is preserved | No supporting evidence | `UNKNOWN` | Explicitly open |
| Pattern improves conversion or revenue | No outcome measurement | `UNKNOWN` | No impact claim is made |
| Pattern generalizes to other workflows | Architecture analysis | `EXPECTED` | Not yet validated in a second example |

## 6. Portable pattern

The reusable design is a **legacy-extension pattern**, not the specific cart implementation:

- extend a stable data contract before replacing a UI;
- put a conditional adapter around a new edge case;
- leave the proven default path untouched;
- return responsibility to the authoritative downstream system;
- keep feasibility, pilot readiness, and rollout as separate gates; and
- maintain an explicit open-risk ledger instead of smoothing unresolved questions into a success narrative.

## Release boundary

This architecture companion omits storefront identity, real item identifiers, private URLs, endpoint details, and private implementation records. It was approved for public release on 2026-08-25; publication does not expand the validated-test boundary or authorize rollout.

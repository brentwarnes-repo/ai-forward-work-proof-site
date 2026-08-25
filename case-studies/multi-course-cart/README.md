# Adding a One-Click Multi-Item Purchase Path to an Existing Storefront

**Status:** Published public-safe proof — 2026-08-25.

**Architecture companion:** [`ARCHITECTURE.md`](ARCHITECTURE.md)

## The challenge

An online course-catalog storefront's existing buy-now links each carried a single product identifier, so a paired purchase—for example, a lecture course and companion lab—required the shopper to add each item separately before checkout. The opportunity was to reduce that friction without replacing the storefront's rendering path, duplicating checkout logic, or treating one successful test as production authorization.

## The intervention

The proof of concept adds one narrow layer on top of systems that already worked:

- **Extend the existing data contract.** A buy-button link can carry more than one item identifier only when a paired purchase is required. The rendering path remains unchanged.
- **Intercept only qualifying clicks.** Normal single-item links stay on the existing path. A small compatibility layer handles only multi-item actions.
- **Reuse native cart and checkout.** The adapter sequences the storefront's existing add-to-cart behavior and then hands control back to the existing checkout. It does not duplicate pricing, membership, or checkout ownership.
- **Keep rollout separate from feasibility.** The custom path was validated on a test page and remains separately gated for Engineering/pilot/broad-rollout decisions.

## Validated evidence

The bounded run demonstrated:

- one qualifying click added both items and reached the existing checkout flow;
- existing membership/subscription handling remained present in checkout during that run;
- existing single-item buy-button behavior was unaffected; and
- no script-specific console error was observed during the validated run.

A follow-on readiness review classified **11** open questions spanning partial-failure behavior, analytics, account states, duplicate/malformed identifiers, support beyond the tested pair, mobile/accessibility, deployment/rollback, ownership, and future-page interactions. Those questions were not silently treated as resolved. One was partially covered by the bounded run, four have design proposals that are explicitly not approved/deployed, and the remaining items remain open.

## Claim boundary

This proof is validated for the narrow scenario actually tested: a two-item paired purchase on a test page. It does **not** establish or claim:

- production deployment or broad customer-facing rollout;
- conversion or revenue impact;
- analytics parity;
- production-ready partial-failure recovery;
- validated production support beyond the tested pair; or
- full device/accessibility/account-state coverage.

Broad rollout remains **not authorized** pending separate Engineering and business review.

## What this demonstrates

The work demonstrates a pragmatic legacy-extension pattern: preserve stable ownership, add the smallest compatible capability, leave the default path untouched, reuse authoritative downstream systems, and separate a validated success path from the questions that still stand between feasibility and rollout.

The paired [`ARCHITECTURE.md`](ARCHITECTURE.md) explains the component boundaries, decision log, failure modes, governance gates, and portable system-design patterns behind the result.

---
title: fishcast
created: 2026-06-27
updated: 2026-06-27
status: draft
---

# PRD: fishcast
*Working title — confirm.*

## 0. Document Purpose

This PRD defines the v1 (MVP) scope for fishcast, a hobby project to produce probabilistic salmon and steelhead harvest forecasts for Washington State sport fisheries. It's written primarily for the builder, doubling as a reference if shared with WDFW managers or anglers for feedback. It builds on, and does not duplicate, WDFW's existing creel estimation prior art ([CreelEstimates](https://github.com/wdfw-fp/CreelEstimates), [creelutils](https://github.com/wdfw-fp/creelutils)). Features are grouped with FRs nested under them; assumptions are tagged inline and indexed in §9.

## 1. Vision

fishcast turns Washington State's publicly available creel data and environmental signals (weather, river flow, tides) into probabilistic, forward-looking salmon and steelhead harvest forecasts. It serves two very different readers from the same underlying forecast: WDFW fishery managers making real in-season management calls, and everyday anglers deciding whether it's worth heading out this weekend.

It builds on, rather than replaces, WDFW's existing creel estimation methodology — adding a forward-looking, dual-audience forecasting layer, with built-in accuracy tracking so trust in the tool's predictions grows visibly as it's used.

## 2. Target User

### 2.1 Jobs To Be Done

- As a WDFW fishery manager, I need near-real-time visibility into harvest against TAC/harvest-control-rule thresholds and angler success so I can decide whether to keep a fishery open, close it, or extend it — and brief tribal co-managers.
- As an angler, I need visibility into reported catch, estimated harvest, and a forward harvest projection for my fishery so I can decide whether and when to go fishing.

### 2.2 Non-Users (v1)

- Commercial fishery operators.
- Fishery agencies/managers outside Washington State.
- Anglers/managers interested in species other than salmon and steelhead.

### 2.3 Key User Journeys

- **UJ-1. Mark checks whether the Skagit fishery needs to close.**
  - **Persona + context:** Mark Smith, WDFW's statewide salmon & steelhead manager, tracking an in-season fishery against its harvest guidelines.
  - **Entry state:** Opens fishcast's Manager View for a specific fishery (e.g., Skagit, current season).
  - **Path:** Checks status against the TAC/harvest control rule (especially for ESA-listed stocks like Puget Sound Chinook and steelhead); reviews total estimated harvest, released fish, total encounters, CPUE, and total angler effort where available; checks whether other tracked species (Chinook, Coho, Sockeye) remain within fishery management plan guidelines.
  - **Climax:** The view clearly flags whether the fishery is under, approaching, or projected to exceed threshold.
  - **Resolution:** Mark decides to keep the fishery open, close it, or extend it, and exports/shares a summary to brief tribal co-managers.

- **UJ-2. TrollRay decides whether to fish the Skagit sockeye run this weekend.**
  - **Persona + context:** TrollRay, a weekend salmon angler, deciding whether a trip is worth it today, this weekend, or within the coming week.
  - **Entry state:** Opens fishcast's Angler View for a chosen fishery (e.g., Skagit sockeye 2026).
  - **Path:** Sees reported catch-to-date (raw), estimated harvest-to-date (statistical expansion), and a forward harvest projection out to about a week.
  - **Climax:** The projection gives TrollRay a read on whether the run/harvest level makes the trip worthwhile.
  - **Resolution:** TrollRay decides whether and when to go. *(Personal catch-probability is a future goal, not in v1 — see Non-Goals.)*

## 3. Glossary

- **Creel survey** — In-person or electronic survey of anglers used to estimate catch and effort.
- **CPUE** — Catch Per Unit Effort.
- **TAC / Harvest Control Rule** — A total allowable catch threshold that triggers a management action when reached.
- **Encounter** — A fish that was harvested or released; total encounters = harvested + released.
- **FMP** — Fishery Management Plan; the agreed guidelines a fishery's catch must track within.
- **Point Estimate (PE) / BSS** — WDFW's two existing statistical methods (classic point estimate; Bayesian hierarchical state-space) for expanding raw creel counts into harvest estimates.
- **Co-manager** — An entity, such as a tribal government, that jointly manages a fishery with WDFW.

## 4. Features

### 4.1 Data Ingestion & Harmonization

**Description:** Pulls raw creel data from WA's [data.wa.gov](https://data.wa.gov) open data portal and environmental data from NOAA (weather forecast, tide predictions) and USGS (river flow), normalizing both into a common schema keyed by fishery and date. [ASSUMPTION: ingestion runs on a scheduled (e.g. daily) basis, not real-time/streaming.]

#### FR-1: Creel data ingestion
The system can ingest creel survey data from data.wa.gov for a given fishery and date range. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Authenticates to data.wa.gov using a registered app token.
- Backs off and retries on rate-limit responses without failing the overall ingestion job.
- Raw records are stored separately from normalized records so provenance is retained.

#### FR-2: Environmental data ingestion
The system can ingest 7-day weather forecasts, USGS river flow data, and NOAA tide predictions relevant to a given fishery's location. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Environmental data is associated to the correct fishery by location (river reach or marine area).
- River-system fisheries pull weather + river flow; marine fisheries pull weather + tides.

### 4.2 Harvest Estimation Model

**Description:** Produces an estimated harvest-to-date from raw creel reports, and a probabilistic forward projection of harvest up to about a week ahead, informed by the environmental predictors. [ASSUMPTION: v1 builds on WDFW's existing point-estimate methodology; whether to also adopt the BSS method, and how to handle marine-specific methodology, are open — see §8.]

#### FR-3: Estimated harvest-to-date
The system can produce an estimated harvest-to-date for a given fishery, expanded statistically from raw creel reports. Realizes UJ-2.

**Consequences (testable):**
- The estimate is visually distinguishable from raw reported catch — both are shown, never conflated.
- Produced at least daily while a fishery is active.

#### FR-4: Forward harvest projection
The system can produce a probabilistic forward projection of harvest for a given fishery, up to 7 days ahead, incorporating relevant environmental predictors. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- The projection includes an uncertainty range, not a single point value.
- The projection updates as new creel and environmental data arrive.

### 4.3 Manager View

**Description:** Per-fishery view giving WDFW managers (UJ-1) what they need to decide whether to keep a fishery open, close it, or extend it, and to brief tribal co-managers.

#### FR-5: Harvest status against control rule
A manager can view total estimated harvest, released fish, total encounters, CPUE, and (where available) total angler effort for a fishery, tracked against its TAC/harvest control rule threshold. Realizes UJ-1.

**Consequences (testable):**
- Clearly flags whether the fishery is under, approaching, or projected to exceed its threshold.
- Covers ESA-listed stocks (e.g., Puget Sound Chinook, steelhead) as well as other FMP-tracked species (Chinook, Coho, Sockeye).

#### FR-6: Shareable management summary
A manager can export or share a summary view suitable for briefing tribal co-managers. Realizes UJ-1.

**Consequences (testable):**
- Output is shareable outside the app (e.g. export or printable/link view).

**Out of Scope:**
- Live system integration with tribal co-manager systems (see §5 Non-Goals).

### 4.4 Angler View

**Description:** Per-fishery view giving anglers (UJ-2) what they need to decide whether to go fishing.

#### FR-7: Fishery harvest snapshot
An angler can view reported catch-to-date, estimated harvest-to-date, and a forward harvest projection for a chosen fishery. Realizes UJ-2.

**Consequences (testable):**
- All three numbers (reported, estimated, projected) are visually distinguished from one another.
- The forward projection covers today, this weekend, and up to about a week out.

### 4.5 Forecast Accuracy Tracking

**Description:** Tracks how past forecasts compared to actual outcomes and surfaces this to users, so trust in the tool builds visibly over time. *(Added mid-Discovery — the builder wants forecast quality measured, not assumed.)*

#### FR-8: Forecast accuracy view
Any user can view how past harvest projections compared to the later-confirmed estimated harvest, for a given fishery.

**Consequences (testable):**
- The comparison becomes visible once actual data "catches up" to a past projection.
- Presented in a way a non-technical angler can interpret, not just raw error statistics.

## 5. Non-Goals (Explicit)

- Commercial fisheries (sport fisheries only).
- Fisheries outside Washington State.
- Species other than salmon and steelhead.
- Personal/individual catch-probability prediction for an angler — harvest-level visibility only in v1.
- Native mobile apps — web only.
- Live system integration with tribal co-manager systems — a shareable/exportable view is in scope, an API/integration is not.
- Automated alerts/push notifications on threshold breach.

## 6. MVP Scope

### 6.1 In Scope
- Data Ingestion & Harmonization (FR-1, FR-2)
- Harvest Estimation Model (FR-3, FR-4)
- Manager View (FR-5, FR-6)
- Angler View (FR-7)
- Forecast Accuracy Tracking (FR-8)
- Web app delivery

### 6.2 Out of Scope for MVP
- Automated alerts/push notifications.
- Tribal co-manager system integration (exports/sharing only).
- Personal catch-probability prediction.
- Native mobile apps.
- [NOTE FOR PM] Marine-specific creel methodology, if WDFW's existing tooling (freshwater-focused) turns out not to cover Puget Sound/marine creel — pending resolution of the open question in §8. If unresolved soon, v1 may need to start freshwater-only.

## 7. Success Metrics

**Primary**
- **SM-1**: Forecast accuracy — error of the forward harvest projection (FR-4) against later-confirmed estimated harvest — is tracked and visible in the Forecast Accuracy view. Validates FR-4, FR-8.
- **SM-2**: The builder personally trusts the forecast enough to use it for their own go/no-go fishing decisions. Validates FR-7.

**Secondary**
- **SM-3**: A real WDFW manager or angler, shown the tool, says they'd actually use it. Validates FR-5, FR-7.

**Counter-metrics (do not optimize)**
- **SM-C1**: Uncertainty ranges are not made artificially narrow just to look more accurate on average — calibrated uncertainty matters as much as point accuracy. Counterbalances SM-1.

## 8. Open Questions

1. Do WDFW's existing creel tools (CreelEstimates, creelutils) cover marine/Puget Sound creel data, or is a separate data source/methodology needed for marine fisheries?
2. Will the v1 estimation model build on WDFW's classic point-estimate method, the Bayesian state-space (BSS) method, or a new approach?
3. What's the actual process to obtain a data.wa.gov app token, and are there documented rate limits to design around?
4. What's the precise list of fisheries (rivers + marine areas) in scope for v1 vs. deferred to later?

## 9. Assumptions Index

- [ASSUMPTION] §4.1 — Ingestion runs on a scheduled (e.g. daily) basis, not real-time/streaming.
- [ASSUMPTION] §4.2 — v1 estimation model builds on WDFW's existing point-estimate methodology; BSS adoption and marine-specific methodology are open (§8).

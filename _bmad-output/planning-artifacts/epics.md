---
stepsCompleted: [1, 2]
inputDocuments:
  - "_bmad-output/planning-artifacts/prd-fishcast-2026-06-27/prd.md"
  - "_bmad-output/planning-artifacts/architecture/architecture-fishcast-2026-06-28/ARCHITECTURE-SPINE.md"
  - "_bmad-output/planning-artifacts/ux-designs/ux-fishcast-2026-06-27/DESIGN.md"
  - "_bmad-output/planning-artifacts/ux-designs/ux-fishcast-2026-06-27/EXPERIENCE.md"
---

# fishcast - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for fishcast, decomposing the requirements from the PRD, UX Design, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR-1: Creel data ingestion — The system can ingest creel survey data from data.wa.gov for a given fishery and date range. Authenticates via a registered app token; backs off and retries on rate-limit responses without failing the overall ingestion job; raw records are stored separately from normalized records so provenance is retained. Realizes UJ-1, UJ-2.

FR-2: Environmental data ingestion — The system can ingest 7-day weather forecasts, USGS river flow data, and NOAA tide predictions relevant to a given fishery's location. Environmental data is associated to the correct fishery by location (river reach or marine area); river-system fisheries pull weather + river flow, marine fisheries pull weather + tides. Realizes UJ-1, UJ-2.

FR-3: Estimated harvest-to-date — The system can produce an estimated harvest-to-date for a given fishery, expanded statistically from raw creel reports. The estimate is visually distinguishable from raw reported catch — both are shown, never conflated. Produced at least daily while a fishery is active. Realizes UJ-2.

FR-4: Forward harvest projection — The system can produce a probabilistic forward projection of harvest for a given fishery, up to 7 days ahead, incorporating relevant environmental predictors. Includes an uncertainty range, not a single point value; updates as new creel and environmental data arrive. Realizes UJ-1, UJ-2.

FR-5: Harvest status against control rule — A manager can view total estimated harvest, released fish, total encounters, CPUE, and (where available) total angler effort for a fishery, tracked against its TAC/harvest control rule threshold. Clearly flags whether the fishery is under, approaching, or projected to exceed its threshold; covers ESA-listed stocks (e.g., Puget Sound Chinook, steelhead) as well as other FMP-tracked species (Chinook, Coho, Sockeye). Realizes UJ-1.

FR-6: Shareable management summary — A manager can export or share a summary view suitable for briefing tribal co-managers. Output is shareable outside the app (e.g. export or printable/link view). Out of scope: live system integration with tribal co-manager systems. Realizes UJ-1.

FR-7: Fishery harvest snapshot — An angler can view reported catch-to-date, estimated harvest-to-date, and a forward harvest projection for a chosen fishery. All three numbers are visually distinguished from one another; the forward projection covers today, this weekend, and up to about a week out. Realizes UJ-2.

FR-8: Forecast accuracy view — Any user can view how past harvest projections compared to the later-confirmed estimated harvest, for a given fishery. The comparison becomes visible once actual data "catches up" to a past projection; presented in a way a non-technical angler can interpret, not just raw error statistics.

### NonFunctional Requirements

NFR-1: Accessibility — WCAG 2.2 AA across the responsive web surface; threshold status and the reported/estimated/projected distinction are never color-only (text label or pattern as primary signal, color as reinforcement).

NFR-2: Reliability (ingestion isolation) — Each of the three external ingestion sources (data.wa.gov, NOAA, USGS) is ingested independently; one source's failure is logged and skipped, never blocking the other two or failing the whole pipeline run.

NFR-3: Reliability (estimation isolation) — One fishery's R/Stan estimation failure is logged and skipped, never aborting the pipeline run for other fisheries sharing that day's invocation.

NFR-4: Data integrity — All pipeline writes are idempotent upserts (`INSERT ... ON CONFLICT DO UPDATE`) keyed by `(fishery, date, source)` for raw/normalized/environmental data and `(fishery, season, species, date, method)` for estimates/projections; the pipeline never deletes-and-reinserts or hard-deletes a row.

NFR-5: Security (secrets management) — Secrets (data.wa.gov app token, NOAA/USGS keys, DB credentials) live in GitHub Actions secrets (pipeline) and Vercel project env vars (web app); never committed to the repo.

NFR-6: Security (no auth, public scope) — Every fishcast view is open to any visitor; no login, session, or role gate — consistent with all underlying data being sourced from public data.wa.gov/NOAA/USGS sources.

NFR-7: Forecast calibration — Uncertainty ranges on the forward harvest projection are not made artificially narrow just to look more accurate on average; calibrated uncertainty matters as much as point accuracy (counter-metric, do not optimize away).

NFR-8: Responsive design — Both the Manager View and Angler View must remain fully usable on mobile (phone) viewports; no content is hidden on mobile, only reflowed (a manager in the field and an angler on a boat ramp may both be on a phone).

NFR-9: Scheduling/execution budget — Ingestion and estimation run as a scheduled GitHub Actions workflow with a daily cadence and a longer, more predictable execution budget than Vercel's cron model; no pipeline logic runs inside a Vercel Function.

### Additional Requirements

- Repository/app structure follows the Architecture Spine's Structural Seed: `config/` (Django settings/urls), `apps/fisheries` (Fishery, Season, SpeciesThreshold), `apps/ingestion` (RawCreelRecord, NormalizedCreelRecord, EnvironmentalReading + management commands), `apps/estimation` (HarvestEstimate, ForecastProjection + management commands shelling out to `r/`), `apps/dashboard` (Manager/Angler/Forecast Accuracy/Share Summary views), `r/` (PE/BSS scripts adapted from wdfw-fp/CreelEstimates + creelutils), `templates/`, `static/`, `.github/workflows/` (`pipeline.yml`, `ci.yml`). No starter/greenfield template is specified by Architecture — project scaffolding is built directly to this structure.
- Pipeline (ingestion + estimation) runs as a scheduled GitHub Actions workflow (`.github/workflows/pipeline.yml`); Vercel hosts only the Django web app (AD-2).
- Estimation core (PE/BSS) runs via an `Rscript` subprocess reusing/adapting WDFW's R/Stan code under `r/`; the Python orchestrator never reimplements the estimation math (AD-3).
- Python↔R interchange is via flat files (Parquet/CSV in a temp directory) — no in-process Python↔R bridge such as rpy2 (AD-4).
- `RawCreelRecord` and `NormalizedCreelRecord` are distinct tables; normalization is append/upsert-only and never mutates the raw row it derived from (AD-6).
- No separate API/JSON layer for v1: chart interactivity (hover/tap reveal) is backed by data already embedded in the server-rendered page, not a separate fetch endpoint (AD-7).
- Share Summary generation runs synchronously inside the Django request that triggers it — no Celery/RQ or other background task queue (AD-8).
- The Django app may write exactly one table, `ShareSummarySnapshot`, only via the Share action handler; it carries a frozen copy of every species tracked under that fishery's season (not just the one in view) as of generation time, plus a `share_token` (UUID4) (AD-9).
- `Fishery` carries provider-specific location identifiers for environmental matching — a USGS gauge site ID for river-flow fisheries, a NOAA tide-station ID for marine fisheries, and a NOAA weather grid reference for both — distinct from and in addition to `wdfw_code`; ingestion matches by the identifier the source itself uses, never derives one provider's ID from another's (AD-11).
- The Forecast Accuracy view is computed at read-time by joining a past `ForecastProjection.target_date` against the `HarvestEstimate` that later lands for the same `(fishery, season, species, date)` — no table stores this comparison (AD-12).
- Dates are date-only (no time-of-day) everywhere a "day" is the unit. Uncertainty is always two columns, `lower_bound`/`upper_bound`, never a single variance figure.
- Every pipeline run writes all of its rows with that run's single `as_of` timestamp, so a view reading multiple rows together shows one consistent "as of" point.
- Schema changes go through Django's built-in migration framework — no separate migration tooling.
- Stack: Python 3.14, Django 6.0, R 4.6, Stan via cmdstanr 0.9, PostgreSQL 18 (managed — Neon or Supabase, provider choice deferred to provisioning time), Vercel (web deployment), GitHub Actions (pipeline scheduler).

### UX Design Requirements

**Design tokens**

UX-DR1: Implement the fishcast color token system — primary navy (`#0F426B`), secondary green (`#007B55`), accent-water light blue (`#92D2D8`), status-safe/watch/exceeded, data-reported/data-estimated/data-projected, surface/surface-muted/surface-sunken, text/text-heading/text-muted, border, and link/link-visited — per `DESIGN.md` Colors.

UX-DR2: Implement the typography token system — Roboto Slab for display/heading-lg/heading-md/heading-sm, Roboto for body/body-sm/label/data-numeric/caption — including the `data-numeric` role reserved for headline harvest figures.

UX-DR3: Implement the 4px-based spacing scale (`spacing.1`–`spacing.8`) and rounded-corner tokens (`sm`/`md`/`lg`/`full`).

**Reusable components**

UX-DR4: Build the Status badge component — pill shape, three variants (safe/watch/exceeded), always paired with a text label ("Under threshold" / "Approaching" / "Exceeded"), never color alone.

UX-DR5: Build the Mode toggle component — segmented control (Manager/Angler), lives in the header, persists the active fishery and season across mode switches (never resets to the picker).

UX-DR6: Build the Harvest number trio component — reported (hollow/outline numeral), estimated (solid fill), projected (dashed line + uncertainty range) — always rendered together in the same order and styling, never re-skinned per screen.

UX-DR7: Build the Projection band chart component — line in the projected color with a low-opacity (0.18) fill marking the uncertainty range; hover/tap on a point reveals the exact value + range as text; recomputes/redraws on new data (no stale silent chart); if a fishery is missing one environmental predictor, the chart still renders using available predictors and the "as of" note names which signal is missing.

UX-DR8: Build the Fishery/season picker component — type-to-filter combobox (not a long unfiltered list); recently-viewed rows get a distinct background to separate them from the rest of the filtered list.

UX-DR9: Build the Card component — base container for status/snapshot/projection blocks, flat fill with a 1px border, minimal shadow (only the active threshold badge may carry a subtle shadow).

UX-DR10: Build the Share Summary view component — reuses the standard card grid on-screen with full primary-color chrome; the printed/exported artifact strips to monochrome (text-on-white) so it photocopies/faxes cleanly.

UX-DR11: Build the Accuracy strip component — reuses the harvest-number-trio's "estimated" treatment for the confirmed-actual point and "projected" treatment for the past projection's band, framed in plain language first with raw error stats secondary.

**Accessibility**

UX-DR12: WCAG 2.2 AA compliance across the entire responsive web surface (contrast for text and status colors against the surface color).

UX-DR13: Every color-coded state (threshold badges, reported/estimated/projected distinction) ships with a text label or pattern (dashed vs. solid vs. hollow) as the primary signal, color only as reinforcement.

UX-DR14: Every chart (projection band, accuracy strip) has a text/table equivalent reachable without relying on hover, so screen reader and low-vision users get the same numbers sighted/mouse users get from the chart.

UX-DR15: Tap targets ≥ 44×44px on the mode toggle, picker rows, and status badges.

UX-DR16: Focus order follows visual reading order: picker → mode toggle → harvest numbers → chart → share/accuracy links.

**Responsive design**

UX-DR17: At `≥ md` breakpoints, the harvest number trio lays out left-to-right, the projection chart sits full-width below it, and the Manager View's full number set (CPUE, effort, encounters) shows alongside the status badge in a multi-column card grid.

UX-DR18: At `< md` (phone) breakpoints, the harvest number trio stacks top-to-bottom (same order, same styling) and the card grid collapses to a single column; nothing is hidden on mobile, only reflowed — Manager View's denser numbers stay visible but stack.

UX-DR19: Charts resize to container width at all breakpoints; axis label density thins on narrow viewports rather than the chart scrolling horizontally.

**Interaction patterns / state handling**

UX-DR20: Preseason state — "{Fishery} opens {date}. No data yet." displayed on Fishery Picker and Angler/Manager View; no harvest numbers rendered as zero (zero would falsely imply an in-progress season with no catch).

UX-DR21: Season closed state — final harvest numbers shown, clearly labeled "Final"; no forward projection rendered (FR-4 doesn't apply post-closure); no stale "as of" implying the season is still live.

UX-DR22: Data pending state — last successful "as of {date}" timestamp shown plainly when today's creel hasn't been processed yet; never silently show yesterday's number as if it were today's.

UX-DR23: Upstream source outage state — show last-known-good data with its timestamp plus a small inline notice (not a blocking error page); forecasting degrades gracefully (e.g., projection omits a missing environmental predictor) rather than disappearing entirely.

UX-DR24: Threshold exceeded state — status badge switches to "exceeded" (red); visible from the Fishery Picker row, not just inside the fishery's own view.

UX-DR25: Accuracy-not-yet-available state — "Too recent to score — check back once {date}'s estimate is confirmed," distinct from "No forecast history yet" ("fishcast hasn't tracked a forecast for {fishery} long enough yet — check back once the season's underway.").

UX-DR26: Share generation failed state — inline error scoped to the Share action itself ("Couldn't generate the summary. Try again."); the underlying Manager View data is unaffected.

UX-DR27: Empty fishery list / no-match state in the picker — "No fishery matches '{query}'. fishcast currently covers {N} WA fisheries — see the full list."

UX-DR28: Fishery list loading (cold) state — skeleton rows while the fishery list loads, resolving to the real list or, on failure, "Couldn't load the fishery list. Try again." — visually distinct from the "no match" state.

UX-DR29: Enforce banned interaction patterns are never implemented — no auto-refreshing numbers without a visible "as of" timestamp, no push notifications or alert banners on threshold breach, no drag/reorder anywhere, no modal stacks more than one level deep.

### FR Coverage Map

FR-1: Epic 1 - Creel data ingestion, built for the confirmed steelhead fishery; broadened to remaining fisheries in Epic 5
FR-2: Epic 1 - Environmental data ingestion, built for the confirmed steelhead fishery; broadened to remaining fisheries in Epic 5
FR-3: Epic 1 - Estimated harvest-to-date, built for the confirmed steelhead fishery; broadened to remaining fisheries/species in Epic 5
FR-4: Epic 1 - Forward harvest projection, built for the confirmed steelhead fishery; broadened to remaining fisheries/species in Epic 5
FR-5: Epic 2 - Harvest status against control rule, built for the confirmed steelhead fishery; full multi-species coverage (Chinook, Coho, Sockeye) completes in Epic 5
FR-6: Epic 3 - Shareable management summary, for the steelhead Manager View
FR-7: Epic 1 - Fishery harvest snapshot (Angler View), built for the confirmed steelhead fishery; broadened to remaining fisheries/species in Epic 5
FR-8: Epic 4 - Forecast accuracy view, for the steelhead fishery; broadened to remaining fisheries/species in Epic 5

## Epic List

### Epic 1: Steelhead Angler Snapshot
Stands up data ingestion (creel + environmental), the PE estimation/projection pipeline, and the Angler View so an angler can see reported/estimated/projected harvest for one confirmed steelhead fishery and decide whether to fish. Opens with a spike story to pin down which specific steelhead fishery/season is buildable now.
**FRs covered:** FR-1, FR-2, FR-3, FR-4, FR-7

### Epic 2: Steelhead Manager Threshold Tracking
Adds the Manager View: harvest vs. TAC/control-rule status, encounters, CPUE, effort, and the under/approaching/exceeded status badge, plus the Manager↔Angler mode toggle, for the same steelhead fishery/season established in Epic 1.
**FRs covered:** FR-5
**Known risk:** if the confirmed steelhead fishery turns out to be release-only or low-volume, the threshold-exceeded path won't be meaningfully validated against real data until Epic 5 introduces a harvestable species.

### Epic 3: Steelhead Shareable Management Summary
Adds the Share Summary action on the Manager View — the web app's one write path, generating a frozen, printable/linkable snapshot for briefing co-managers.
**FRs covered:** FR-6

### Epic 4: Steelhead Forecast Accuracy
Adds the read-time accuracy comparison (past projections vs. later-confirmed estimates) for the steelhead fishery, in plain language for non-technical anglers.
**FRs covered:** FR-8
**Known risk:** same deferred-validation concern as Epic 2 — near-zero harvest numbers make the accuracy comparison trivially "accurate" and won't stress-test calibration (NFR-7) until real variance exists via Epic 5.

### Epic 5: Multi-Species & Multi-Fishery Expansion
Seeds and validates the remaining FMP-tracked species (Chinook, Coho, Sockeye) and the rest of the v1 fishery list (other rivers + marine areas — PRD §8 open question 4) across ingestion, estimation, and all four views built in Epics 1–4. If Epic 2/4's known risks materialized, prioritizes pulling in a harvestable species early in this epic to retroactively validate the threshold and accuracy logic, rather than treating this purely as "seed more data."
**FRs covered:** Broadens FR-1, FR-2, FR-3, FR-4, FR-5, FR-7, FR-8 to full v1 scope (no new FRs)

<!-- Repeat for each epic in epics_list (N = 1, 2, 3...) -->

## Epic {{N}}: {{epic_title_N}}

{{epic_goal_N}}

<!-- Repeat for each story (M = 1, 2, 3...) within epic N -->

### Story {{N}}.{{M}}: {{story_title_N_M}}

As a {{user_type}},
I want {{capability}},
So that {{value_benefit}}.

**Acceptance Criteria:**

<!-- for each AC on this story -->

**Given** {{precondition}}
**When** {{action}}
**Then** {{expected_outcome}}
**And** {{additional_criteria}}

<!-- End story repeat -->

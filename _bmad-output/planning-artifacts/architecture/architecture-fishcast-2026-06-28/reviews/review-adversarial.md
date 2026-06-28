---
name: fishcast
type: adversarial-review
target: ARCHITECTURE-SPINE.md
created: '2026-06-28'
---

# Adversarial Review — fishcast Architecture Spine

## Verdict

The spine correctly fences *who computes what* (AD-1/AD-3/AD-4) and *how writes happen* (AD-5/AD-6/AD-9), but it leaves six concrete cross-cutting concerns — cross-source entity keying, multi-row transaction/read consistency, the exact contents of a "frozen" snapshot, R-subprocess failure blast radius, the home of the accuracy comparison, and season-lifecycle ownership — unconstrained, so two contributors can each follow every AD to the letter and still ship incompatible, behavior-breaking designs.

## Divergent Pair 1 — `ingest_creel` vs. `ingest_environmental`: two keying schemes for the same `Fishery`

**The pair:** `ingest_creel` matches incoming data.wa.gov records to a `Fishery` via `wdfw_code`, exactly as the Naming convention mandates. `ingest_environmental` (FR-2) must associate a NOAA tide/weather reading or USGS flow reading to "the correct fishery by location (river reach or marine area)" — but a USGS gauge ID and a NOAA station ID are not `wdfw_code` values; there is no WDFW location code on either upstream API. A contributor implementing `ingest_environmental` therefore adds *some* second join path (e.g., a manually maintained `Fishery.usgs_site_id` / `Fishery.noaa_station_id` field, or a separate crosswalk table, or worse, a brittle lat/lon proximity match) — entirely reasonably, since the Naming convention only constrains how *creel* records resolve to `Fishery`.

**The gap:** AD-5/AD-6 and the Naming convention assume one natural key resolves all ingestion into `Fishery`. They don't, because FR-2's own consequence ("associated by location") admits a second, environmental-specific identity scheme that the spine never names. Two contributors building `ingest_environmental` independently can each invent a different crosswalk shape (new column on `Fishery` vs. new join table vs. geometry match) — all compliant with every literal AD, all incompatible with each other, and a future contributor adding a new fishery type (e.g., the still-open marine/Puget Sound question, PRD §8 Q1) has no spine-mandated place to put the USGS/NOAA identifiers a marine fishery would need.

**Suggested fix:** Tighten the Naming convention (or add an AD) to require `Fishery` carry the upstream environmental identifiers explicitly as first-class columns (e.g., `usgs_site_id`, `noaa_station_id`, both nullable per fishery type), and state that `ingest_environmental` resolves to `Fishery` via these columns, never by name/geometry inference — mirroring the existing `wdfw_code` rule for creel.

## Divergent Pair 2 — Multi-species pipeline upsert vs. Manager View's "as of" read: torn reads across rows

**The pair:** AD-5 requires every row's upsert to be idempotent and keyed by `(fishery, date, source)`/`(fishery, date, species)` — a *per-row* guarantee. Nothing constrains the transaction boundary across the *set* of rows one pipeline run touches. Implementation A upserts each species' `HarvestEstimate` in its own transaction/statement, sequentially, as it finishes each R/Stan run — fully AD-5-compliant per row. Implementation B wraps the whole day's multi-species batch in one transaction before committing — also fully AD-5-compliant per row, and also compliant with AD-1 (pipeline is still the sole writer).

**The gap:** The Manager View's status badge and harvest-number trio (UX: "Status badge ... recalculates whenever new creel data lands") must show Chinook, Coho, and Sockeye all "as of" one consistent date in a single page render (UJ-1: "scans the same view for Coho and Sockeye"). Under Implementation A, a web request landing mid-pipeline-run can read Chinook's freshly-upserted estimate alongside Coho's stale one-day-old estimate in the same page, with one "as of" timestamp claiming to cover both — an internally inconsistent read that no AD prevents, because AD-1/AD-5 only police *write ownership* and *per-row* idempotency, not cross-row read consistency during a concurrent write window.

**Suggested fix:** Add an AD (or tighten AD-5) requiring the orchestrator to commit each pipeline run's full set of `HarvestEstimate`/`ForecastProjection` rows for a given `(fishery, date)` in a single transaction, so the web app never observes a partially-updated date across species.

## Divergent Pair 3 — Two compliant Share Summary handlers with different snapshot scopes

**The pair:** AD-9 says the Share handler writes "a frozen copy of the numbers at generation time" to `ShareSummarySnapshot`. Implementation A scopes "the numbers" narrowly: just the currently-displayed species' status badge + harvest trio (literally satisfies "a frozen copy of the numbers"). Implementation B scopes it broadly: all tracked species' statuses, CPUE, effort, encounters, and the projection band series for the fishery (also literally "a frozen copy of the numbers" — just more of them).

**The gap:** AD-9 never specifies the snapshot's required field set or scope, only that it must be frozen and keyed by a UUID4 token. But UJ-1's actual resolution step — "he scans the same view for Coho and Sockeye... taps Share Summary... sends it to the tribal co-managers" — depends on the briefing artifact containing *all three species'* statuses, since that's what the manager just reviewed and is sharing. Implementation A is AD-9-compliant and silently breaks the journey (co-managers get one species, not the full picture Mark saw). There's also no rule on schema evolution: if Manager View later adds a field, do old snapshots (frozen at generation time, by design) coexist with a new snapshot shape in the same table — and does a contributor need a versioned schema or a JSON blob? Unaddressed.

**Suggested fix:** Tighten AD-9 to enumerate the snapshot's required contents (mirror FR-5's full consequence list: all tracked species' status + estimated harvest + released + encounters + CPUE + effort, not just the in-view species), and require the snapshot payload be stored as a versioned/self-describing blob (e.g., JSON with a schema-version field) precisely because it's frozen and must outlive future Manager View field changes.

## Divergent Pair 4 — R/Stan subprocess failure: per-source skip convention doesn't reach the estimation step

**The pair:** The Consistency Conventions table's failure-isolation rule is scoped explicitly to "each of the three external sources" (data.wa.gov/NOAA/USGS) during *ingestion*. AD-3/AD-4 govern only *how* R is invoked (subprocess, flat files), not what happens when it fails. Implementation A treats any single fishery's `Rscript` non-zero exit (bad input data, Stan convergence failure, OOM in CI) as fatal to the whole orchestrator run — crashes the GitHub Actions job before any fishery's estimate is upserted that day. Implementation B catches the R subprocess error per-fishery inside the orchestrator's loop and continues to the next fishery, leaving that one fishery's estimate stale but landing all others. Both are fully compliant with AD-2, AD-3, AD-4, and AD-5 — none of those rules say which fishery-failure blast radius is correct.

**The gap:** This collides directly with the UX's staleness contract ("Data pending... Last successful 'as of {date}' timestamp shown plainly") and AD-2's own stated purpose (preventing silent truncation). Under Implementation A, one fishery's flaky Stan model silently regresses *every* fishery's "as of" date that day — a far larger blast radius than the source-outage handling the spine explicitly designed for ingestion, but with zero equivalent rule for the estimation step.

**Suggested fix:** Add an AD extending the existing source-isolation convention to the estimation step: one fishery's R/Stan failure is caught, logged, and skipped per-fishery; the orchestrator continues to other fisheries and the GitHub Actions job is marked failed/warned only in aggregate, never aborted mid-loop before other fisheries' upserts complete.

## Divergent Pair 5 — AccuracyRecord as a live join vs. a pipeline-materialized table

**The pair:** The spine's ERD note asserts AccuracyRecord is "a query joining `ForecastProjection.target_date` against the `HarvestEstimate` that later lands... not a stored table," governed by AD-1/AD-7/AD-10 — but this is a descriptive aside, not a rule with a "Prevents"/"Rule" pair like the ADs. Implementation A computes the join live on every Forecast Accuracy page request — arguably AD-1-compliant since AD-1 only forbids the web app deriving a *harvest number*, and an accuracy delta isn't literally a harvest number. Implementation B, anticipating per-request join cost growing with season length and worried about AD-7's "no separate API/JSON layer" pressure to keep pages fast, has the *pipeline* materialize the comparison into a real `AccuracyRecord` row on each run (also AD-1-compliant: the pipeline is still the one computing/writing it).

**The gap:** Both readings are individually defensible under AD-1's literal text ("never derives a harvest number inline" — silent on derived *statistics*). They produce incompatible schemas: one has no such table at all, the other adds a pipeline-written table the ERD explicitly says shouldn't exist. FR-8's consequence — "the comparison becomes visible once actual data 'catches up' to a past projection" — needs *some* trigger to know when that catch-up has happened; nothing says whether that's a read-time `WHERE target_date <= latest_estimate_date` filter (cheap, A) or a pipeline-side recompute-and-flag step (B, tempting toward AD-8's banned background-job territory if it's framed as "recompute on new data").

**Suggested fix:** Promote the ERD aside to an actual AD: "AccuracyRecord is computed at read-time only, via a query against `ForecastProjection` and `HarvestEstimate`; the pipeline never writes a materialized accuracy table." This forecloses Implementation B explicitly and gives AD-7's "no separate read path" principle a partner rule for the one read computation in the app that isn't a literal stored row.

## Divergent Pair 6 — Season lifecycle ownership: pipeline-written status vs. dashboard-inferred status

**The pair:** The UX state table requires distinct "Preseason" / "Season closed (Final)" treatments, but no AD says who decides and records that transition. Implementation A: the pipeline writes an explicit `Season.status` column (`preseason`/`open`/`closed`) each run — consistent with AD-1's spirit that the pipeline owns derived state. Implementation B: the dashboard infers "closed" at read-time purely from `Season.end_date < today()`, writing nothing — also AD-1-compliant by its literal text, since inferring a lifecycle label isn't computing a *harvest number*.

**The gap:** The two implementations diverge on where "closed" lives, but worse, neither AD-5 ("never deletes") nor any other AD says the pipeline must *stop* generating `ForecastProjection` rows once a season is closed. Implementation A's nightly orchestrator, written before this edge case was considered, can keep happily upserting forward-projection rows with `target_date`s past a season's now-known close date — fully AD-5-compliant (idempotent upsert, no deletes) — while the UX explicitly requires "no forward projection" once a season is closed. The dashboard (Implementation B) would have to filter these out at read-time to honor the UX contract, silently compensating for a pipeline that has no rule telling it to stop.

**Suggested fix:** Add an AD naming `Season.status` as a pipeline-owned, explicitly written field (not inferred), and require the orchestrator to check it before generating new `ForecastProjection` rows for a season — skipping projection generation (not estimation-to-date, which still applies post-closure per the "Final" numbers) once a season transitions to closed.

## Summary Table

| # | Pair | AD(s) nominally satisfied | Gap |
|---|---|---|---|
| 1 | `ingest_creel` vs `ingest_environmental` keying | Naming convention, AD-5 | No spine-mandated environmental identity columns on `Fishery` |
| 2 | Per-row upsert vs multi-species page read | AD-1, AD-5 | No transaction boundary across a pipeline run's row set |
| 3 | Narrow vs broad Share Summary handler | AD-9 | Snapshot content/schema unscoped |
| 4 | R crash = job-fatal vs per-fishery skip | AD-2, AD-3, AD-4 | No failure-isolation rule for the estimation step (only ingestion has one) |
| 5 | Live-joined vs pipeline-materialized AccuracyRecord | AD-1, AD-7 | ERD note is descriptive, not a binding AD |
| 6 | Pipeline-written vs dashboard-inferred season status | AD-1, AD-5 | No rule halting projection generation post-closure |

# Architecture Spine Review — fishcast (rubric walker)

Reviewed: `ARCHITECTURE-SPINE.md` (2026-06-28) against PRD (`prd-fishcast-2026-06-27/prd.md`) and UX spine (`DESIGN.md` / `EXPERIENCE.md`, 2026-06-27).

## Verdict

The spine is structurally sound and free of UX contradictions, but it silently skips operational observability/recovery (no monitoring, backup, or migration story even in Deferred) and leaves one real divergence risk — estimation method (PE/BSS) isn't part of the natural key that AD-5's idempotent upsert relies on.

## Critical

None. No finding here rises to "will produce incompatible builds with no recourse" — the closest candidate (method-key gap, below) is rated High because it's a schema-design oversight rather than an unenforceable/contradictory rule.

## High

- **AD-5 / Stack (natural key for estimates) — `(fishery, date, species)` doesn't include `method` (PE vs. BSS), but PRD §8 OQ2 and the spine's own Deferred section leave PE-vs-BSS open, and AD-3 explicitly says the R step computes "PE + BSS."** If a future builder runs both methods (e.g., to compare them) or switches methods mid-season, the second method's row collides with the first under the same upsert key per AD-5, silently overwriting it rather than producing two comparable rows. Two independent estimation-step builders (one targeting PE, one targeting BSS) would diverge on whether `method` is part of identity — exactly the kind of incompatible-build risk the Deferred-section check asks about, except it isn't called out in Deferred at all.
  *Fix:* Add `method` to the natural key for `HarvestEstimate`/`ForecastProjection` now (even if v1 ships only one method), or add an explicit Deferred line stating "estimates are single-method per date; adding a second concurrent method requires revisiting AD-5's key."

- **Operational envelope — no monitoring/alerting, backup/DR, or migration-process decision anywhere, not even in Deferred.** The checklist's "every structural dimension this altitude owns is decided, deferred, or an open question" calls out operations explicitly. The spine covers deployment topology (AD-2, Stack) and environments (Deferred: staging) well, but pipeline-run failure handling stops at "logged and skipped" for a *single source* (Consistency Conventions row 3) — there's no statement of what happens when the whole pipeline run fails (R/Stan crash, GitHub Actions job failure) or how an operator (the hobby-project builder) finds out. Similarly, Postgres backup/restore and Django migration rollout process are never mentioned. This isn't a wrong answer — it's a silently absent dimension, which the checklist treats as a failure mode distinct from "deferred."
  *Fix:* Add a one-line AD or Deferred entry: e.g. "Pipeline failure surfaces via GitHub Actions' own run-failure notification (email/UI) — no separate alerting system for v1" and "DB backup relies on the managed provider's (Neon/Supabase) default backup/PITR — no custom backup tooling," so these are explicit decisions rather than silent gaps.

## Medium

- **AD-2's quoted Vercel limits are asserted, not sourced, and drive a load-bearing decision.** "10s Hobby / 300s Pro execution cap, daily-only Hobby cron" is the entire justification for AD-2 (pipeline on GitHub Actions, not Vercel). These numbers are directionally consistent with Vercel's known serverless constraints but are stated as bare facts with no citation, and Vercel's limits have changed release-to-release historically. If a future builder checks Vercel's current docs and finds the numbers shifted, the *rule* ("pipeline never runs inside a Vercel Function") still holds, but the stated rationale would look stale/wrong, undermining confidence in the AD.
  *Fix:* Soften to "Vercel's free/Hobby tier historically caps serverless execution time well below what a multi-source ingestion + R/Stan estimation run needs" or cite a source/date, so the Rule's durability doesn't depend on exact numbers staying current.

- **No CI/test strategy decided or deferred.** `ci.yml` appears in the Structural Seed's source tree but nothing says what it runs (lint only? pytest? a check that pipeline output schema matches Django models?). For a "future builder adding ingestion modules" — exactly the persona this spine is supposed to protect — the absence of a stated contract for what CI enforces is a real silent gap, not just a nice-to-have.
  *Fix:* One line in Deferred or Consistency Conventions: e.g. "`ci.yml` runs Django test suite + lint; pipeline correctness is verified by rerunning against fixture data, not asserted here — implementation detail," which at least converts silence into an explicit (if deferred) decision.

## Low

- **AD-8's name ("No background task queue") is slightly broader than its actual scope.** The Rule only constrains Share Summary (FR-6), but the heading reads as a global ban on background processing, which could be misread by a future builder as also forbidding e.g. a deferred future async export feature outside FR-6's scope (the AD does gesture at this with "any future write that can't complete... requires revisiting this AD," which mitigates it).
  *Fix:* Retitle to "No background task queue for the Share Summary write" or add a parenthetical scope note in the heading itself, not just the Rule text.

- **Stack table's R/Stan versions (R 4.6, cmdstanr 0.9) aren't spot-checked for mid-2026 plausibility the way Django/Python/Postgres are, despite being just as load-bearing for AD-3/AD-4.** Not necessarily wrong, but the spine invites scrutiny of its named-tech currency and these two entries got less rigor than the others.
  *Fix:* No action required for this review pass, but worth a deliberate spot-check before the spine is finalized, given AD-3 is one of the higher-stakes ADs (methodology fidelity to WDFW's validated approach).

## Items checked and passing (no finding)

- **FR coverage:** FR-1 through FR-8 all appear in the Capability → Architecture Map; none silently missing.
- **UX-spine consistency:** AD-9's frozen-snapshot rule for `ShareSummarySnapshot` matches `EXPERIENCE.md`'s "frozen snapshot as of generation time... never silently refreshed" requirement exactly. AD-10 (no auth) matches `EXPERIENCE.md`'s "no login... all views open to anyone." AD-7 (no API layer, chart backed by embedded page data) is consistent with the UX spine's hover/tap-reveals-text requirement for the projection band chart and accuracy strip — both are described as page-rendered, not fetch-driven, and AD-7 explicitly binds FR-8 so the Forecast Accuracy chart is covered too.
- **Tech currency spot-check:** Python 3.14, Django 6.0, and PostgreSQL 18 are all plausible as current/stable releases by mid-2026 given each project's known release cadence (Python's annual October cycle, Django's post-5.2-LTS progression to 6.0, Postgres's annual September major-version cycle). No red flags.
- **Deferred section:** none of the six deferred items (BSS-vs-PE method choice, marine data source, Postgres provider, cron cadence, staging environment, fishery list) let two independent builders construct incompatible *interfaces* — they're either provisioning-time choices that don't affect schema/contract, or genuinely still-open PRD questions correctly not pre-answered by the architecture. (The one method-related risk that *does* qualify is captured above under High, because it's a missing key-design decision rather than the deferred choice itself.)
- **AD enforceability:** AD-1 through AD-10 each state a concrete, checkable rule (specific tables, specific file, specific subprocess boundary, specific natural key) rather than a vague principle — each is independently enforceable via code review (e.g., "does this PR add a Vercel Function that touches HarvestEstimate?").

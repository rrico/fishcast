# Tech Currency Review — ARCHITECTURE-SPINE.md

**Reviewed:** 2026-06-28
**Scope:** Stack table (Python, Django, R, cmdstanr, PostgreSQL, Vercel, GitHub Actions) and AD-2's Vercel-capability claims.

## Verdict

Most named versions are real and current, but AD-2's specific numbers for Vercel's Hobby-plan function timeout (10s) and the "at-least-once" cron delivery characterization are stale/wrong as of mid-2026 — Vercel raised Hobby's default function duration to 300s via Fluid Compute over a year ago, and Vercel's own docs describe cron delivery as "best effort" (can skip *or* duplicate), not "at-least-once" — though this doesn't undermine AD-2's conclusion (GitHub Actions is still the right call) since the daily-cron-only and best-effort/duplicate-risk problems are real and independently sufficient.

## Findings

### Stack table

1. **Python 3.14 — confirmed.** Released 2025-10-07; latest patch as of June 2026 is 3.14.6 (2026-06-10). A real, current version. Source: WebSearch (python.org release pages, Real Python Nov 2025 roundup).

2. **Django 6.0 — confirmed.** Released 2025-12-03; supports Python 3.12/3.13/3.14, consistent with pairing it with Python 3.14. Patch releases 6.0.1/6.0.2 exist as of mid-2026. Source: WebSearch (docs.djangoproject.com/en/6.0/releases/6.0/, djangoproject.com weblog).

3. **R 4.6 — confirmed, and notably fresh.** R 4.6.0 ("Because it was There") released 2026-04-24 — under 10 weeks before this document's stated date. This is a real, current release, not a hallucinated future version. Source: WebSearch (R-bloggers "What's new in R 4.6.0", CRAN NEWS).

4. **cmdstanr 0.9 — confirmed but slightly stale-pinned.** v0.9.0 is a real tagged release (stan-dev/cmdstanr), released 2025-03-30. It is over a year old as of mid-2026; CmdStan itself is already at 2.39 (released 2026-05-19), and cmdstanr has likely had newer releases since v0.9.0 that the search did not surface a version number for. Pinning "0.9" isn't wrong, but the spine doesn't show evidence of checking whether a newer cmdstanr exists — worth a footnote that this is a floor, not necessarily the latest. Source: WebSearch (Stan Forums "CmdStanR v0.9.0 Release", GitHub stan-dev/cmdstanr tags, Stan blog CmdStan 2.39).

5. **PostgreSQL 18 — confirmed.** Released 2025-09-25. Real, current major version; matches the "managed — Neon or Supabase" framing since both providers support PG18. Source: WebSearch (postgresql.org official release announcement).

6. **Vercel "zero-config Django/Python runtime" — confirmed as a real, very recent capability, with one caveat (uncertain).** Vercel shipped zero-configuration Django support around 2026-04-09 (auto-detects `manage.py`, infers `DJANGO_SETTINGS_MODULE`/WSGI/ASGI entrypoint). This is genuinely current — not asserted from stale training-data memory of Vercel-as-Next.js-only. However, search results disagreed on whether the underlying Python runtime itself is still labeled "Beta" or has graduated to a standard supported runtime as of mid-2026 — one source says Beta-on-all-plans, another (citing May 2026 docs) treats it as standard. **Uncertain** — recommend the architecture flag this as "confirm beta status at provisioning time" rather than treating it as fully GA. Sources: WebSearch (vercel.com/changelog/zero-configuration-django-support, vercel.com/docs/functions/runtimes/python).

7. **GitHub Actions — confirmed, not version-pinned (no currency risk).** No specific version claim to check; `schedule:` cron trigger is a stable, long-standing GHA feature.

### AD-2 claims about Vercel limits

8. **"10s Hobby / 300s Pro" execution cap — wrong / stale.** This describes Vercel's *pre-Fluid-Compute* legacy limits. As of the **2025-06-25** changelog "Higher defaults and limits for Vercel Functions running Fluid compute," Vercel made Fluid Compute the default for projects and raised the **default execution time to 300 seconds on all plans, including Hobby** (confirmed separately: Fluid Compute became default for new projects 2025-04-23). Current Hobby behavior: up to 300s by default with Fluid Compute; without Fluid Compute, the old 10s default/60s max legacy figures apply. Since Fluid Compute is now the default for new projects, a project created in 2026 would NOT hit a 10-second cap by default — it would need Fluid Compute explicitly disabled to hit the old 10s number. **The "10s Hobby" framing in AD-2 is out of date by about a year.** Sources: WebSearch citing vercel.com/changelog/higher-defaults-and-limits-for-vercel-functions-running-fluid-compute (2025-06-25) and a Vercel engineer's public confirmation (X/Twitter, @cramforce: "Hobby users can now run up to 300 seconds of compute at a time on Vercel. For Pro and Enterprise 300s is the new default timeout (can be increased to 800 seconds)"); corroborated by vercel.com/docs/functions/limitations via search summary (current doc states 300s default/max for Hobby with Fluid Compute, vs. 10s/60s without).

9. **"Daily-only Hobby cron" — confirmed, still accurate.** Vercel's Hobby plan still restricts cron jobs to once-per-day-or-less-frequent; any expression that would fire more than once per 24h fails deployment. This part of AD-2 holds up. Source: WebSearch (vercel.com/docs/cron-jobs/manage-cron-jobs, runhooks.app and crontap.com blog posts describing the same restriction as of 2026).

10. **"Best-effort at-least-once delivery" — wrong characterization (uncertain→wrong).** Vercel's own documentation describes cron delivery as **best-effort**, explicitly warning that invocations can be **skipped** (transient network errors → function never executes, no log) **or duplicated** (same scheduled run invoked more than once) — and tells users to design for both missed and duplicate runs. "At-least-once" is a specific, stronger delivery semantic (guarantees the message/trigger always arrives, possibly more than once, but never zero times). Vercel's actual guarantee is weaker than that: it can also simply not fire. So AD-2's phrase "best-effort at-least-once delivery" conflates two different things — the "best-effort" qualifier is right, but pairing it with "at-least-once" overstates the guarantee (it should read something like "best-effort delivery — invocations can be skipped or duplicated"). This doesn't change AD-2's conclusion (still a good reason to avoid Vercel cron for a pipeline with retry/backoff needs) but the stated justification is technically imprecise. Source: WebSearch summarizing vercel.com/docs/cron-jobs/manage-cron-jobs language ("cron job delivery is best effort... can also occasionally invoke the same scheduled run more than once... should be resilient to both missed runs and duplicate runs").

11. **Net effect on AD-2's argument — conclusion still holds, but for different/weaker stated reasons.** Even with the corrected 300s Hobby default (not 10s), GitHub Actions remains the right call for the pipeline: (a) daily-only Hobby cron (confirmed, finding 9) is still a hard blocker for any sub-daily retry/backoff need; (b) best-effort delivery with possible **skipped** runs (not just duplicates) is arguably a stronger argument for moving off Vercel than the AD currently states, since a silently-skipped daily ingestion run is worse than a double-invocation that idempotent upserts (AD-5) already handle. Recommend the architecture spine correct the 10s/300s figures and the delivery-semantics phrase, since an outside reviewer fact-checking those exact numbers would currently flag the document as relying on stale (pre-mid-2025) Vercel limits rather than mid-2026 ones — even though the inability-to-retry-frequently argument for GitHub Actions is sound either way.

## Sources checked (via WebSearch / WebFetch)

- python.org release pages (3.14.0, 3.14.6); Real Python Nov 2025 roundup
- docs.djangoproject.com/en/6.0/releases/6.0/; djangoproject.com weblog Dec 2025
- R-bloggers "What's new in R 4.6.0"; CRAN NEWS for R 4.6.0
- Stan Forums "CmdStanR v0.9.0 Release" announcement; stan-dev/cmdstanr GitHub tags; Stan blog "Release of CmdStan 2.39" (2026-05-19)
- postgresql.org "PostgreSQL 18 Released!" announcement
- vercel.com/changelog/zero-configuration-django-support; vercel.com/docs/functions/runtimes/python
- vercel.com/changelog/higher-defaults-and-limits-for-vercel-functions-running-fluid-compute (2025-06-25); vercel.com/changelog/fluid-compute-is-now-the-default-for-new-projects (2025-04-23); public statement from Vercel engineer (@cramforce) corroborating the 300s Hobby/300s-800s Pro figures
- vercel.com/docs/cron-jobs/manage-cron-jobs (daily-only Hobby restriction; best-effort/duplicate-or-missed delivery language)

Note: direct WebFetch to vercel.com/docs/* returned HTTP 403 (blocked) for this session; all Vercel-specific findings above are sourced from WebSearch result summaries that quote or paraphrase the relevant Vercel docs/changelog pages, not from a direct fetch of the live page. Treat findings 6, 8, 9, 10 as corroborated-via-search-summary rather than directly-read-primary-source, and re-verify by visiting the linked URLs directly if a higher confidence bar is needed before committing AD-2's exact wording.

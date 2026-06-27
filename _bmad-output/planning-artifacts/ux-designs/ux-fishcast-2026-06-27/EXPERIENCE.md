---
name: fishcast
status: draft
sources:
  - "{planning_artifacts}/prds/prd-fishcast-2026-06-27/prd.md"
updated: 2026-06-27
---

# fishcast — Experience Spine

> Web-only, responsive (desktop-primary, mobile-usable). One app, two modes (Manager / Angler) via an explicit header toggle — no login. Paired with `DESIGN.md` (fishcast visual identity, WDFW-inspired). Public data, no auth: every fishcast number is sourced from data.wa.gov/NOAA/USGS, which are themselves public, and the PRD doesn't scope an authentication system — so all views are open to anyone.

## Foundation

Single responsive web app, no native apps (PRD §5 Non-Goals). No UI system named in the PRD — components are bespoke, specified visually in `DESIGN.md`. Desktop is the primary surface for the Manager View (TAC tracking is a dense, multi-number job); the Angler View must work well on a phone, since "should I go fishing this weekend" is a question asked on the couch or in the truck, not at a desk. `DESIGN.md` is the visual identity reference; this spine is the experience.

Two modes, one app: **Manager** and **Angler** share navigation, the fishery/season picker, and the underlying data — they differ in which numbers are surfaced and how dense the layout is, not in brand or infrastructure. The mode toggle (`DESIGN.md.components.mode-toggle`) lives in the header and persists the selected fishery when switching modes — a user who picks Skagit Chinook in Angler mode and flips to Manager mode should land on Skagit Chinook's manager status, not a blank picker.

## Information Architecture

| Surface | Reached from | Purpose |
|---|---|---|
| Fishery Picker | App open (cold) | Choose a fishery + season; entry point for both modes |
| Angler View | Mode toggle → Angler (default) | Reported / estimated / projected harvest snapshot for the chosen fishery (FR-7) |
| Manager View | Mode toggle → Manager | Harvest vs. TAC/control-rule status, encounters, CPUE, effort (FR-5) |
| Share Summary | "Share" action on Manager View | Exportable/printable view for briefing co-managers (FR-6) |
| Forecast Accuracy | Linked from both Angler and Manager views | Past projections vs. later-confirmed actuals (FR-8) |

No drawer, no nested settings menu — v1 has no accounts, so there is no Settings surface. Modal/sheet stacking is one level deep at most (e.g., Share Summary opens as an overlay on Manager View, nothing opens on top of it).

→ Composition reference: `mockups/` (key screens added at Finalize). Spine wins on conflict.

## Voice and Tone

Microcopy. Brand posture lives in `DESIGN.md.Brand & Style`.

| Do | Don't |
|---|---|
| "Estimated harvest: 412 fish (as of June 26)." | "Whoa, 412 fish already! 🎣" |
| "Approaching the TAC threshold." | "Watch out — almost there!" |
| "Last week's projection was within 18 fish of the confirmed estimate." | "98% accurate!" (implies false precision) |
| Same plain, numbers-forward voice for both Manager and Angler — the audience differs, the tone doesn't. | A playful voice for anglers and a clinical voice for managers — fishcast speaks once. |
| Name uncertainty directly: "projected range: 380–460." | Hide the range behind a single confident-sounding number. |

## Component Patterns

Behavioral. Visual specs live in `DESIGN.md.Components`.

| Component | Use | Behavioral rules |
|---|---|---|
| Status badge | Manager View, Fishery Picker row | Three states only: under / approaching / exceeded threshold (FR-5). Always text + color. Recalculates whenever new creel data lands — never a static label. |
| Mode toggle | Header, global | Two-way switch, Manager/Angler. Switching preserves the active fishery and season; never resets to the picker. |
| Harvest number trio | Angler View, Manager View | Reported, estimated, and projected always render together, in that left-to-right or top-to-bottom order, using `DESIGN.md.components.harvest-number` styling. Never show one without the others in context — the comparison *is* the feature. |
| Projection band chart | Angler View, Manager View | Line + shaded uncertainty band, 1–7 days out (FR-4). Tapping/hovering a day reveals the point estimate and range as text (accessibility + precision). Recomputes and redraws when new data arrives — no stale silent chart. |
| Fishery/season picker | Fishery Picker, header (persistent) | Type-to-filter combobox, not a long unfiltered list — v1 covers a defined but non-trivial set of WA rivers + marine areas (PRD open question §8.4). Recently viewed fisheries surface first. |
| Share Summary action | Manager View | Generates a static, shareable/printable view (link or export) — no live-editing, no commenting. One click, one artifact (FR-6). |
| Accuracy strip | Forecast Accuracy view | Past projection (band) overlaid with the confirmed actual (point), per elapsed forecast. Framed in plain language first ("Last week's projection was within 18 fish"), raw error stats available but secondary (FR-8). |

## State Patterns

| State | Surface | Treatment |
|---|---|---|
| Preseason (fishery not yet open) | Fishery Picker, Angler/Manager View | "{Fishery} opens {date}. No data yet." No harvest numbers rendered as zero — zero implies a season in progress with no catch, which is a different fact. |
| Season closed | Angler/Manager View | Final harvest numbers shown, clearly labeled "Final" — no forward projection (FR-4 doesn't apply post-closure), no stale "as of" implying it's still live. |
| Data pending (today's creel not yet processed) | Angler/Manager View | Last successful "as of {date}" timestamp shown plainly. Never silently show yesterday's number as if it were today's — staleness is always labeled (PRD FR-1 assumption: ingestion is scheduled, not real-time). |
| Upstream source outage (data.wa.gov / NOAA / USGS) | Angler/Manager View | Show last-known good data with its timestamp; a small inline notice, not a blocking error page. Forecasting degrades gracefully (e.g., projection omits a missing environmental predictor) rather than disappearing entirely. |
| Threshold exceeded | Manager View, Fishery Picker row | Badge switches to "exceeded" (red); this is the single most important state in the app and must be visible from the picker, not just inside the fishery's own view. |
| Accuracy not yet available | Forecast Accuracy view | "Too recent to score — check back once {date}'s estimate is confirmed." Never show a comparison against unconfirmed data. |
| Empty fishery list / no match in picker | Fishery Picker | "No fishery matches '{query}'. fishcast currently covers {N} WA fisheries — see the full list." |

## Interaction Primitives

fishcast is a glance-and-decide tool, not a power-user workspace — interactions are simple and tap/click-first, not keyboard-shortcut-driven.

- Tap/click to select a fishery, switch modes, or open the Share Summary.
- Hover (desktop) / tap (mobile) on any chart point reveals its exact value and range as text — charts are never the *only* way to read a number.
- Filter-as-you-type in the fishery picker; no infinite scroll over the fishery list.
- **Banned:** auto-refreshing numbers without a visible "as of" timestamp, push notifications or alert banners on threshold breach (explicit PRD non-goal), drag/reorder anywhere, modal stacks more than one level deep.

## Accessibility Floor

Behavioral. Visual contrast lives in `DESIGN.md` (palette chosen to maintain WCAG AA contrast for text and status colors against `{colors.surface}`).

- WCAG 2.2 AA across the responsive web surface — this serves a public agency's data to the general public, including non-technical anglers and color-vision-deficient users.
- Threshold status and the reported/estimated/projected distinction are never color-only: every color-coded state ships with a text label or pattern (dashed vs. solid vs. hollow) as the primary signal, color as reinforcement.
- Every chart (projection band, accuracy strip) has a text/table equivalent reachable without relying on hover — screen reader and low-vision users get the same numbers sighted mouse users get from the chart.
- Tap targets ≥ 44×44px on the mode toggle, picker rows, and badge — these are the controls a user on a boat-ramp phone screen actually has to hit.
- Focus order follows visual reading order: picker → mode toggle → harvest numbers → chart → share/accuracy links.

## Key Flows

### Flow 1 — Mark checks whether the Skagit fishery needs to close (Mark Smith, WDFW statewide salmon & steelhead manager)

1. Mark opens fishcast, lands on the Fishery Picker, types "Skagit" and selects the current Skagit season.
2. Mode toggle is already on Manager (his last-used mode persists for the session). Manager View loads: status badge, total estimated harvest, released fish, total encounters, CPUE, total angler effort.
3. He checks the status badge for Puget Sound Chinook — "Approaching" (yellow) — then scans the same view for Coho and Sockeye, both "Under threshold" (green).
4. **Climax:** The Chinook badge's "Approaching" state, paired with the projection band showing the threshold being crossed within 3 days at current pace, gives Mark an unambiguous read: this fishery needs a decision now, not next week.
5. He taps Share Summary, gets a printable/link view of the current status, and sends it to the tribal co-managers ahead of a call to discuss closing or extending.

Failure: today's creel data hasn't landed yet → Manager View shows yesterday's numbers with a visible "as of {yesterday's date}" timestamp; Mark knows to treat the read as slightly stale, not wrong.

### Flow 2 — TrollRay decides whether to fish the Skagit sockeye run this weekend (TrollRay, weekend salmon angler)

1. TrollRay opens fishcast on his phone Thursday night, picks "Skagit Sockeye 2026" from the Fishery Picker. Mode toggle defaults to Angler.
2. Angler View loads: reported catch-to-date (hollow numeral), estimated harvest-to-date (solid navy), and the projection band chart running from today through the next 7 days.
3. He taps Saturday on the projection band — it shows a point estimate and a range, both clearly above where the run has been tracking.
4. **Climax:** The combination of "estimated harvest already healthy" and "Saturday's projected range stays strong" gives TrollRay a real answer, not just a vibe — the trip looks worth it.
5. He checks the Forecast Accuracy link out of curiosity — last week's projection was within a stated margin of the confirmed estimate — and decides he trusts the read enough to commit to Saturday.

Failure: the run is projected to taper sharply by Saturday → the projection band visibly narrows and drops; TrollRay decides to go Thursday or Friday instead, which is exactly the decision the projection exists to support.

## Inspiration & Anti-patterns

- **Lifted from weather forecast apps:** the explicit uncertainty band on the forward projection, and the "is it worth it" framing for a leisure decision (Angler View) rather than a raw data dump.
- **Lifted from financial/ops dashboards:** the threshold status badge pattern (under / approaching / exceeded) for the Manager View — the same shape as a budget-vs-limit indicator.
- **Rejected — alerts/push notifications on threshold breach:** explicit PRD non-goal; a manager or angler must visit fishcast to get a read, fishcast never reaches out to them.
- **Rejected — gamified accuracy scoring (streaks, leaderboards, badges for the model):** Forecast Accuracy exists to build calibrated trust, not to make the model's track record feel like a game.
- **Rejected — single blended "harvest" number:** collapsing reported/estimated/projected into one figure would be easier to design but would misrepresent exactly the distinction (raw report vs. statistical expansion vs. forecast) the PRD requires to stay visible (FR-3, FR-7).

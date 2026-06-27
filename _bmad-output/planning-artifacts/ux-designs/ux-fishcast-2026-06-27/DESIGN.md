---
name: fishcast
description: Probabilistic salmon/steelhead harvest forecasts for WA sport fisheries. WDFW-inspired but standalone — a hobby project, not an official agency product.
status: draft
sources:
  - "{planning_artifacts}/prd-fishcast-2026-06-27/prd.md"
  - "https://github.com/wdfw-fp/CreelEstimates/blob/main/template_scripts/styleRmd_WDFW.css"
updated: 2026-06-27
colors:
  primary: '#0F426B'
  primary-foreground: '#FFFFFF'
  secondary: '#007B55'
  secondary-foreground: '#FFFFFF'
  accent-water: '#92D2D8'
  accent-water-foreground: '#0F426B'
  status-safe: '#007B55'
  status-watch: '#FFD12E'
  status-watch-foreground: '#222222'
  status-exceeded: '#AA1F2E'
  data-reported: '#5A594D'
  data-estimated: '#0F426B'
  data-projected: '#92D2D8'
  surface: '#FFFFFF'
  surface-muted: '#F7F7F5'
  surface-sunken: '#EFEEEA'
  border: '#D1D3D4'
  text: '#222222'
  text-heading: '#59594A'
  text-muted: '#5A594D'
  link: '#18459A'
  link-visited: '#551A8B'
typography:
  display:
    fontFamily: 'Roboto Slab'
    fontSize: 34px
    fontWeight: '400'
    lineHeight: '1.2'
  heading-lg:
    fontFamily: 'Roboto Slab'
    fontSize: 26px
    fontWeight: '400'
    lineHeight: '1.25'
  heading-md:
    fontFamily: 'Roboto Slab'
    fontSize: 20px
    fontWeight: '400'
    lineHeight: '1.3'
  heading-sm:
    fontFamily: 'Roboto Slab'
    fontSize: 16px
    fontWeight: '500'
    lineHeight: '1.3'
  body:
    fontFamily: 'Roboto'
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.5'
  body-sm:
    fontFamily: 'Roboto'
    fontSize: 14px
    fontWeight: '400'
    lineHeight: '1.45'
  label:
    fontFamily: 'Roboto'
    fontSize: 13px
    fontWeight: '500'
    lineHeight: '1.3'
    letterSpacing: 0.02em
  data-numeric:
    fontFamily: 'Roboto'
    fontSize: 28px
    fontWeight: '500'
    lineHeight: '1.1'
  caption:
    fontFamily: 'Roboto'
    fontSize: 12px
    fontWeight: '400'
    lineHeight: '1.4'
rounded:
  sm: 4px
  md: 8px
  lg: 12px
  full: 9999px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '5': 24px
  '6': 32px
  '8': 48px
components:
  status-badge:
    safe:
      background: '{colors.status-safe}'
      foreground: '{colors.surface}'
      radius: '{rounded.full}'
    watch:
      background: '{colors.status-watch}'
      foreground: '{colors.status-watch-foreground}'
      radius: '{rounded.full}'
    exceeded:
      background: '{colors.status-exceeded}'
      foreground: '{colors.surface}'
      radius: '{rounded.full}'
  mode-toggle:
    background: '{colors.surface-muted}'
    activeBackground: '{colors.primary}'
    activeForeground: '{colors.primary-foreground}'
    radius: '{rounded.full}'
  harvest-number-trio:
    reported:
      color: '{colors.data-reported}'
      style: 'outline / hollow marker'
    estimated:
      color: '{colors.data-estimated}'
      style: 'solid fill'
    projected:
      color: '{colors.data-projected}'
      style: 'dashed line + shaded band'
  card:
    background: '{colors.surface}'
    border: '{colors.border}'
    radius: '{rounded.md}'
  projection-band-chart:
    line: '{colors.data-projected}'
    band-fill: '{colors.data-projected}'
    band-opacity: 0.18
    axis: '{colors.text-muted}'
    gridline: '{colors.surface-sunken}'
    hover-marker: '{colors.primary}'
  fishery-picker:
    input-background: '{colors.surface}'
    input-border: '{colors.border}'
    input-radius: '{rounded.sm}'
    row-radius: '{rounded.sm}'
    row-recent-background: '{colors.surface-muted}'
  share-summary-view:
    background: '{colors.surface}'
    chrome: '{colors.primary}'
    print-mode: 'monochrome — chrome and status colors collapse to {colors.text} on white for the printed/exported artifact; on-screen view keeps full color'
---

## Brand & Style

fishcast turns open WDFW creel data into forward-looking harvest forecasts for two audiences who need to trust the same numbers under very different stakes: a manager deciding whether to close a fishery, and an angler deciding whether a Saturday trip is worth the gas money. The brand has to read as **credible and calm** — closer to a weather forecast or a financial dashboard than a consumer app. No gamification, no urgency theater, no marketing gloss.

Visually, fishcast borrows WDFW's real palette and type system (sourced from `wdfw-fp/CreelEstimates`'s `styleRmd_WDFW.css`, itself citing "WDFW Style Guide v2.1, March 2025") so the numbers feel like they belong to the same lineage as the agency's own creel data. But fishcast is explicitly **not** an official WDFW product — it drops the dense government-report chrome in favor of a cleaner, more modern dashboard surface: more whitespace, rounder corners, card-based layout instead of dense tables-with-borders-everywhere.

## Colors

- **Navy (`{colors.primary}`, `#0F426B`)** — WDFW's agency navy. Primary brand color: header, primary buttons, the mode toggle's active state, and the **estimated harvest** data series (the model's confident, central number).
- **Green (`{colors.secondary}`, `#007B55`)** — WDFW's agency green. Reserved for "safe" states: a fishery tracking comfortably under its TAC/control-rule threshold (`{colors.status-safe}`). Never used decoratively — green always means "good news against a threshold."
- **Light Blue (`{colors.accent-water}`, `#92D2D8`)** — a water/tide motif pulled from the WDFW palette. Used for the **forward projection** data series (the speculative, uncertain number) and its uncertainty band — light, airy, "not yet real."
- **Yellow (`{colors.status-watch}`) / Red (`{colors.status-exceeded}`)** — threshold states only: approaching TAC (yellow) and projected-to-exceed or exceeded (red), both straight from the WDFW palette. Never used for anything else — a user should be able to learn "yellow = watch, red = stop" once and trust it everywhere in the app.
- **Dark Gray (`{colors.data-reported}`, `#5A594D`)** — the **reported catch** data series (raw, unprocessed angler reports). Deliberately the least visually assertive of the three harvest numbers — it's a count, not a conclusion.
- **Warm neutrals** (`{colors.surface-muted}`, `{colors.surface-sunken}`, `{colors.border}`) replace WDFW's print-oriented grays with a softer dashboard palette.
- **Link blue** (`{colors.link}` / `{colors.link-visited}`) — reserved for true navigational hyperlinks only: the Share Summary's printable/shareable link and the picker's "see the full list" empty-state link. Everything else that looks clickable (mode toggle, picker rows, badges) is a button or tap target, not a link, and should never use this color.

Avoid: introducing any new chromatic color beyond this set, using status colors (green/yellow/red) for anything other than threshold state, and ever rendering reported/estimated/projected harvest numbers in the same color — that collapse is the one mistake this product cannot make (see PRD FR-3, FR-7).

## Typography

Roboto (body) and Roboto Slab (headings) — straight from the WDFW style guide, carried forward as the typographic spine. Roboto Slab's slight warmth softens what would otherwise read as a sterile data tool; Roboto stays completely plain everywhere numbers and tables do the talking.

`{typography.data-numeric}` is fishcast's one invented role beyond the WDFW system: a larger, medium-weight numeric style reserved for the headline harvest figures (reported / estimated / projected) on the Angler and Manager views — these numbers are the product, they should be the most visually prominent thing on the screen.

## Layout & Spacing

4px-based scale (`{spacing.1}`–`{spacing.8}`). Dashboard-style card grid rather than WDFW's dense report-table layout: each fishery's status, harvest snapshot, and projection live in their own `{components.card}`, stacked on mobile and arranged in a responsive grid on wider viewports. Generous spacing (`{spacing.5}`–`{spacing.6}`) between cards; tighter spacing (`{spacing.2}`–`{spacing.3}`) inside a card so related numbers stay visually grouped.

## Elevation & Depth

Minimal. Cards are distinguished by a 1px `{colors.border}` outline and a flat `{colors.surface}` fill, not shadows — this is a data tool, not a marketing surface. The single exception: the active threshold badge may carry a very subtle shadow on the Manager View to pull it forward as the page's focal point.

## Shapes

`{rounded.sm}` for inputs and small chips. `{rounded.md}` for cards — soft enough to feel like a modern app, not so round it feels playful. `{rounded.full}` for the status badge and the mode toggle, since both are binary/categorical state indicators and a pill shape reads as "state," not "container."

## Components

- **Status badge** (`{components.status-badge}`) — pill, three variants (safe / watch / exceeded). Always paired with a text label ("Under threshold" / "Approaching" / "Exceeded"), never color alone.
- **Mode toggle** (`{components.mode-toggle}`) — segmented control, two options (Manager / Angler), lives in the header, persists across fishery navigation.
- **Harvest number trio** (`{components.harvest-number-trio}`) — the visual contract for the three harvest figures: reported is a hollow/outline numeral, estimated is solid, projected is set with a dashed underline and shown with its uncertainty range. The three must always appear together with this same treatment, never re-skinned per-screen.
- **Projection band chart** (`{components.projection-band-chart}`) — the highest-stakes visual in the product: a line in `{colors.data-projected}` with a low-opacity (`band-opacity: 0.18`) fill of the same color marking the uncertainty range, plotted against a muted axis (`{colors.text-muted}`) and faint gridlines (`{colors.surface-sunken}`). Hovering/tapping a point shows a `{colors.primary}` marker and reveals the exact value + range as text — the chart is never the only way to read a number.
- **Fishery/season picker** (`{components.fishery-picker}`) — a combobox: `{rounded.sm}` input, `{rounded.sm}` dropdown rows. Recently-viewed rows get a `{colors.surface-muted}` background to visually separate them from the rest of the filtered list, no other treatment difference.
- **Share Summary view** (`{components.share-summary-view}`) — reuses the standard card grid on-screen with full `{colors.primary}` chrome; the printed/exported artifact strips to monochrome (`{colors.text}` on white) so it photocopies and prints cleanly for a co-manager briefing.
- **Accuracy strip** — no new tokens; it reuses `{components.harvest-number-trio}`'s `estimated` treatment for the confirmed-actual point and its `projected` treatment for the past projection's band, so a past-projection-vs-actual comparison reads with the same visual grammar a user already learned from the Angler/Manager views.
- **Card** (`{components.card}`) — the base container for every status, snapshot, and projection block.

## Do's and Don'ts

| Do | Don't |
|---|---|
| Use navy for estimated, light-blue for projected, dark-gray for reported — every time | Let any two of the three harvest numbers share a color or visual weight |
| Reserve green/yellow/red strictly for threshold state | Use status colors decoratively or for unrelated UI |
| Pair every status badge with a text label | Rely on color alone to convey threshold state |
| Keep the WDFW palette and Roboto/Roboto Slab type | Recreate WDFW's dense government-report table chrome |
| Flat cards, 1px border, minimal shadow | Add gradients, glossy buttons, or marketing-style hero imagery |
| Strip Share Summary exports to monochrome for printing | Print full-color chrome that wastes a co-manager's printer ink or misreads on a black-and-white fax/scan |

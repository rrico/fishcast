# Spine Pair Review — fishcast

## Overall verdict

This is a strong, well-disciplined first draft. Flow and token coverage are essentially complete, the WDFW-grounded brand identity is coherent and well-justified, and the reported/estimated/projected three-color discipline is enforced consistently across both files. The main gap is structural: four of the seven behaviorally-specified components in EXPERIENCE.md (Projection band chart, Fishery/season picker, Share Summary action, Accuracy strip) have no matching visual-spec row in DESIGN.md's Components section, which will force a downstream architecture/dev-story phase to invent visual treatment with no spine backing it. State coverage on two of the five IA surfaces (Share Summary, Forecast Accuracy) is thin. No mockups exist yet, which is expected at this stage and explicitly flagged as pending in the spine itself.

## 1. Flow coverage — strong

Extracted from PRD: UJ-1 (Mark, Manager View / fishery closure decision), UJ-2 (TrollRay, Angler View / go-fishing decision). FRs: FR-1 (creel ingestion), FR-2 (environmental ingestion), FR-3 (estimated harvest-to-date), FR-4 (forward harvest projection), FR-5 (harvest status vs. control rule), FR-6 (shareable management summary), FR-7 (fishery harvest snapshot), FR-8 (forecast accuracy view).

EXPERIENCE.md Key Flows has Flow 1 (Mark/UJ-1) and Flow 2 (TrollRay/UJ-2), each with a named protagonist, numbered steps, an explicit "Climax" beat, and a failure path. Both UJs from the PRD are covered 1:1, using the PRD's own protagonist names (Mark, TrollRay) and journey shape (entry state → path → climax → resolution).

FR coverage by behavioral surface:
- FR-3, FR-4, FR-5, FR-6, FR-7, FR-8 — all have explicit behavioral coverage (Component Patterns rows, State Patterns rows, or Key Flow steps cite the FR number directly).
- FR-1 (creel ingestion) — only indirectly covered via the "Data pending" state row, which cites the FR-1 scheduling assumption. No flow or component addresses ingestion failure/retry behavior surfacing to a user.
- FR-2 (environmental data ingestion) — no dedicated behavioral coverage. The "Upstream source outage" state row generically covers "data.wa.gov / NOAA / USGS" together, but FR-2's specific consequence ("river-system fisheries pull weather+flow; marine fisheries pull weather+tides") has no UX-visible treatment — e.g., what an angler sees if a fishery's projection is missing one of its predictors isn't spelled out beyond "degrades gracefully."

### Findings
- **medium** FR-2's per-predictor degradation (the "river vs. marine pulls different environmental signals" consequence) has no explicit behavioral surface — only a generic outage state covers it (EXPERIENCE.md, State Patterns, "Upstream source outage" row). *Fix:* add a sentence or state row clarifying what a user sees when one predictor (e.g., tide data for a marine fishery) is missing but others are present.
- **low** FR-1's ingestion retry/backoff behavior (PRD FR-1 consequence: "backs off and retries... without failing the overall job") is backend-only and arguably doesn't need user-facing coverage — flagged for completeness, not a real gap.

## 2. Token completeness — strong

Frontmatter defines: 19 color tokens (primary, primary-foreground, secondary, secondary-foreground, accent-water, accent-water-foreground, status-safe, status-watch, status-watch-foreground, status-exceeded, data-reported, data-estimated, data-projected, surface, surface-muted, surface-sunken, border, text, text-heading, text-muted, link, link-visited — 22 actually, recount below), 9 typography roles (display, heading-lg, heading-md, heading-sm, body, body-sm, label, data-numeric, caption), 4 rounded scale values (sm, md, lg, full), 7 spacing levels (1,2,3,4,5,6,8), and 4 components (status-badge with 3 sub-variants, mode-toggle, harvest-number with 3 sub-variants, card).

Every `{path.to.token}` reference found in prose/components resolves to a token actually defined in frontmatter — checked `{colors.primary}`, `{colors.secondary}`, `{colors.status-safe}`, `{colors.accent-water}`, `{colors.status-watch}`, `{colors.status-exceeded}`, `{colors.data-reported}`, `{colors.surface-muted}`, `{colors.surface-sunken}`, `{colors.border}`, `{colors.surface}`, `{typography.data-numeric}`, `{spacing.1}`–`{spacing.8}`, `{rounded.sm}`, `{rounded.md}`, `{rounded.full}`, `{components.card}`, `{components.status-badge}`, `{components.mode-toggle}`, `{components.harvest-number}` — all defined. No broken references found. All color tokens carry hex values; no critical gaps.

Several frontmatter tokens are defined but never cross-referenced in prose: `secondary-foreground`, `accent-water-foreground`, `text`, `text-heading`, `text-muted`, `link`, `link-visited`, `heading-lg`, `heading-md`, `heading-sm`, `body-sm`, `caption`. This is normal for a token catalog (not every token needs a prose citation — implementers will use `text`, `link`, and the heading ramp directly) and is not counted as a defect, but is worth a mechanical note.

### Findings
- **low** `link` / `link-visited` tokens are defined (inherited from WDFW conventions) but never explained in prose — no Components or Do's/Don'ts guidance on when a true hyperlink (vs. a tap target/button) should appear in a dashboard-style app. (DESIGN.md frontmatter lines 30-31). *Fix:* either cite link usage in Components (e.g., the Share Summary's printable link, the "see the full list" picker-empty-state link) or drop if unused by any flow.

## 3. Component coverage — thin

Components named in DESIGN.md Components section: Status badge, Mode toggle, Harvest number, Card — 4 components, each with a real visual spec (anatomy/usage/state appearance), not one-word descriptions.

Components named in EXPERIENCE.md Component Patterns section: Status badge, Mode toggle, Harvest number trio, Projection band chart, Fishery/season picker, Share Summary action, Accuracy strip — 7 components, each with real behavioral rules.

Cross-check:
- Status badge — present both sides. Strong.
- Mode toggle — present both sides. Strong.
- Harvest number / Harvest number trio — present both sides (name varies slightly: "Harvest number" in DESIGN.md vs. "Harvest number trio" in EXPERIENCE.md — same referent, see Inheritance Discipline). Strong.
- Card — DESIGN.md only. No EXPERIENCE.md behavioral row. Likely acceptable (a card is a passive container, not an interactive component), but technically a one-sided entry.
- Projection band chart — EXPERIENCE.md only. No DESIGN.md visual spec (no row describing chart line weight, band fill/opacity, axis treatment, color application of the data-projected/data-estimated tokens to the actual chart geometry).
- Fishery/season picker — EXPERIENCE.md only. No DESIGN.md visual spec (no row describing the combobox's visual anatomy, input styling, dropdown row treatment).
- Share Summary action — EXPERIENCE.md only. No DESIGN.md visual spec (no row describing what the generated summary/printable view actually looks like).
- Accuracy strip — EXPERIENCE.md only. No DESIGN.md visual spec (no row describing the band-vs-point overlay's visual treatment beyond what's implied by harvest-number tokens).

### Findings
- **high** Projection band chart has detailed behavioral rules (EXPERIENCE.md, Component Patterns, row 4) but zero visual specification in DESIGN.md — no chart-specific row in Components, and the only token hint is the harvest-number component's "dashed line + shaded band" for projected (DESIGN.md, components.harvest-number.projected). This is the single most data-dense, highest-stakes visual in the whole product (it carries FR-4's uncertainty range for both UJ-1's climax and UJ-2's climax) and it lacks a dedicated visual contract. *Fix:* add a "Projection band chart" row to DESIGN.md Components specifying axis treatment, band opacity/fill color reference, line weight, and how hover/tap target affordance is rendered.
- **medium** Fishery/season picker has behavioral rules (combobox, type-to-filter, recently-viewed-first) but no DESIGN.md visual row — no token guidance on input styling, dropdown row height/spacing, or how "recently viewed" is visually distinguished from the rest of the list. *Fix:* add a Components row referencing `{rounded.sm}` (inputs), spacing tokens, and how active/recent rows are styled.
- **medium** Share Summary action has behavioral rules but no visual spec — what does the generated printable/link view actually look like (same card grid? a stripped-down print stylesheet? does it use `{colors.primary}` chrome or go monochrome for printing)? *Fix:* add a brief Components row or a note in Layout & Spacing about print/export treatment.
- **low** Accuracy strip has behavioral rules but no dedicated visual row — partially inheritable from harvest-number tokens (band = projected styling, point = estimated styling) but that inheritance isn't stated explicitly anywhere. *Fix:* one sentence in DESIGN.md Components confirming Accuracy strip reuses harvest-number's projected/estimated treatment.
- **low** Card (DESIGN.md) has no corresponding EXPERIENCE.md behavioral row — likely fine since it's a passive container with no states of its own, but flagged for completeness since the rubric calls for two-sided coverage.

## 4. State coverage — adequate

Walking all 5 IA surfaces:

- **Fishery Picker** — states present: empty/no-match ("Empty fishery list / no match in picker"), preseason (shared row). Plausible missing: cold-load (initial fetch of the fishery list itself failing or being slow) — not covered; the picker is assumed to always load instantly.
- **Angler View** — states present: preseason, season closed, data pending (stale), upstream outage, accuracy-not-yet-available (via linked surface). Solid coverage for a data-ingestion-driven tool.
- **Manager View** — states present: same as Angler plus threshold-exceeded (its signature state). Strong — this is the highest-stakes surface and it shows.
- **Share Summary** — no dedicated state row. Plausible states needed: export/share generation in progress, export failure, link expiration/staleness (a shared link viewed days later — does it show live data or a frozen snapshot? Component Patterns says "static" but State Patterns never covers what "static" looks like when viewed later against now-different live numbers).
- **Forecast Accuracy** — only "not yet available" covered. Plausible missing: empty-history state (a brand-new fishery added to fishcast with zero elapsed forecasts yet — distinct from "too recent to score" which implies at least one projection exists) and load/error state for the accuracy computation itself.

### Findings
- **high** Share Summary has zero rows in State Patterns despite being a named IA surface with its own behavioral row — no coverage for generation failure, or for what a stale/already-shared link shows when opened after the underlying fishery data has moved on (Component Patterns says "static, shareable" but doesn't say whether "static" means frozen-at-generation-time or always-fresh-at-open-time — a real ambiguity a manager will hit when re-sharing a link days later). *Fix:* add a State Patterns row, and resolve the static-snapshot-vs-live ambiguity in either Component Patterns or here.
- **medium** Forecast Accuracy's only state is "too recent to score," which presumes a projection already exists. A genuinely new fishery with zero forecast history yet is a distinct state (no projections to score at all, vs. "this specific projection hasn't matured"). (EXPERIENCE.md, State Patterns, row 6.) *Fix:* add or clarify a state for "no forecast history yet" distinct from "this projection hasn't matured yet."
- **low** Fishery Picker has no explicit cold-load/loading state for the picker's own list fetch (distinct from "no match," which presumes the list loaded fine). *Fix:* add a one-line state or fold into existing rows if the list is assumed to ship statically/bundled rather than fetched.

## 5. Visual reference coverage — pending (not a defect)

`ls` of the workspace directory shows `imports/` and `.working/` subdirectories, both empty — no mockups, wireframes, or rendered visuals exist yet. EXPERIENCE.md's Information Architecture section does include a forward-looking composition reference line ("→ Composition reference: `mockups/` (key screens added at Finalize). Spine wins on conflict.") which correctly anticipates this and defers it to a later Finalize step, consistent with the example spines' pattern (e.g. Quill/Drift examples link to `mockups/today-cold.html` etc. once those exist). This is expected at this stage of the workflow, not a defect.

### Findings
- *(none — flagged as pending per task instructions, not a defect)*

## 6. Bloat & overspecification — strong

No pixel-level overrides found that duplicate token coverage (e.g., no literal `34px` typed into prose where `{typography.display}` already covers it — checked Brand & Style, Colors, Typography sections; all numeric values appear only inside frontmatter or as the token's own definition, not restated redundantly in body prose). PRD content is cited by FR number rather than restated verbatim (e.g., "Realizes UJ-1" style PRD phrasing is not copy-pasted; EXPERIENCE.md paraphrases and cites). Tables are used appropriately for IA, Voice and Tone, Component Patterns, State Patterns, Do's/Don'ts — no prose-where-a-table-would-work issues found. No decorative/untied narrative — the WDFW-lineage justification in Brand & Style and the Inspiration & Anti-patterns entries each tie to a specific, real design decision (e.g., "Lifted from weather forecast apps: the explicit uncertainty band" ties directly to FR-4's uncertainty-range requirement).

### Findings
- *(none — clean)*

## 7. Inheritance discipline — adequate

`sources:` frontmatter in both files points to `{planning_artifacts}/prds/prd-fishcast-2026-06-27/prd.md`, which resolves correctly to the real file at `/home/user/fishcast/_bmad-output/planning-artifacts/prd-fishcast-2026-06-27/prd.md` (directory name matches once the `prds/` path-alias segment is resolved per the established pattern from the example spines). DESIGN.md additionally sources the real WDFW `styleRmd_WDFW.css` file via a live GitHub URL — a legitimate and traceable grounding for the color/type tokens, confirmed against the `.memlog.md` decision log in the same directory.

UJ names (Mark, TrollRay) and FR numbers (FR-1 through FR-8) are used verbatim from the PRD throughout EXPERIENCE.md. Component names are consistent within each file, and mostly consistent across files, with one inconsistency noted below. All `{path.to.token}` references in EXPERIENCE.md prose (e.g., `{colors.surface}`, `DESIGN.md.components.harvest-number`, `DESIGN.md.components.mode-toggle`) resolve to real DESIGN.md frontmatter tokens.

### Findings
- **medium** Component name drift: DESIGN.md names it "**Harvest number**" (singular, DESIGN.md Components section and frontmatter key `harvest-number`); EXPERIENCE.md names the same thing "**Harvest number trio**" (Component Patterns, row 3). Functionally the same component, but a downstream consumer doing exact-string lookups across both files (as instructed by this very task) will not get an automatic match. *Fix:* align to one name — "Harvest number" is the frontmatter-canonical key, so EXPERIENCE.md's row should either say "Harvest number" or DESIGN.md should adopt "trio" as part of its own name.
- **low** EXPERIENCE.md's Fishery/season picker row cites "PRD open question §8.4" (EXPERIENCE.md, Component Patterns, row 5) — the PRD's §8 Open Questions is a flat 4-item numbered list, not subsectioned into §8.1–§8.4. The citation is directionally correct (item 4 is indeed about the fishery list scope) but the §8.4 notation doesn't exist in the source document as written. *Fix:* cite as "PRD §8, item 4" or just "§8" to match the PRD's actual structure.

## 8. Shape fit — strong

**DESIGN.md** body section order: Brand & Style → Colors → Typography → Layout & Spacing → Elevation & Depth → Shapes → Components → Do's and Don'ts. This matches the canonical order exactly, with no triggered/optional sections inserted out of place. All 8 canonical sections present (none omitted).

**EXPERIENCE.md** section order: Foundation → Information Architecture → Voice and Tone → Component Patterns → State Patterns → Interaction Primitives → Accessibility Floor → Key Flows, with Inspiration & Anti-patterns inserted between Accessibility Floor and Key Flows. All required default sections are present. The triggered section present is Inspiration & Anti-patterns (4 entries: 2 "lifted from," 2 "rejected") — each entry ties to a concrete decision (uncertainty-band framing from weather apps, threshold-badge pattern from financial dashboards, rejecting alerts per explicit PRD non-goal, rejecting gamified accuracy scoring) — it earns its place rather than being generic filler. Responsive & Platform is omitted; given the spine states "desktop is the primary surface... Angler View must work well on a phone" but never gives breakpoint-specific behavior (unlike the Drift example's explicit breakpoint table), this omission is a borderline call — see finding below.

### Findings
- **medium** Responsive & Platform section is omitted, but Foundation explicitly raises a real responsive tension ("Desktop is the primary surface for the Manager View... the Angler View must work well on a phone") without ever specifying breakpoint behavior, card-grid-to-single-column collapse rules, or how the Manager View's denser numbers (CPUE, effort, encounters alongside the badge) degrade on a phone screen if a manager does check from mobile. The Drift example shows this section's expected shape (explicit breakpoint table) when a spine has real cross-device behavior to specify — fishcast's Layout & Spacing in DESIGN.md gestures at "stacked on mobile and arranged in a grid on wider viewports" but that's a DESIGN.md visual note, not an EXPERIENCE.md behavioral one (e.g., does the harvest-number trio reflow, does the chart resize, is anything hidden on mobile?). *Fix:* either add a minimal Responsive & Platform section with a breakpoint table, or fold an explicit mobile-behavior sentence into Foundation/Component Patterns so the omission is clearly a non-trigger rather than a missed section.

## Mechanical notes

- `sources:` paths in both files resolve correctly to the real PRD file once the `{planning_artifacts}/prds/...}` alias is expanded against the actual directory `prd-fishcast-2026-06-27/prd.md`. DESIGN.md's secondary source (WDFW CSS GitHub URL) is a real, fetchable file per the `.memlog.md` decision log.
- Component name inconsistency: "Harvest number" (DESIGN.md) vs. "Harvest number trio" (EXPERIENCE.md) — same referent, different string. See Finding 7.
- Broken/invented cross-reference: "PRD open question §8.4" (EXPERIENCE.md, Fishery/season picker row) — PRD §8 has no subsections; it's a flat numbered list of 4 items. See Finding 7.
- No mockups/wireframes/imports exist yet (`imports/` and `.working/` are both empty directories) — both spines correctly anticipate this and defer to a future `mockups/` reference rather than inventing fake links, consistent with the example spines' pattern.
- Frontmatter color token count: 22 color tokens defined (primary, primary-foreground, secondary, secondary-foreground, accent-water, accent-water-foreground, status-safe, status-watch, status-watch-foreground, status-exceeded, data-reported, data-estimated, data-projected, surface, surface-muted, surface-sunken, border, text, text-heading, text-muted, link, link-visited) — all carry hex values, no missing-hex critical defects found.
- No name collisions or duplicate component definitions found within either file.

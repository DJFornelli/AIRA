# ProHosts Revenue Management Decision Engine — v2.1 (Canonical)

**Status:** Single source of truth. Supersedes: Decision Engine v1.0/v1.1 (PDF), START HERE tab rules, Reference Guide v2 "Pricing Decision Rules" (8-rule set), and the v2.0 Rule Expansion. Spreadsheet rule tabs and the Reference Guide remain as human training views; where they conflict with this file, this file wins.

**Changelog**
- v2.1 — Reconciled all rule sources into one canonical numbering (RM-001–RM-051). Merged Booking Freeze into RM-002 as its Critically Behind branch. Parametrized pacing targets (no hardcoded universal targets). Added weekly cadence rule (RM-051), precedence hierarchy, confidence tiers, metric definitions, property parameter table, legacy mapping table, and machine-readable YAML companion.
- v2.0 — Added Rules 16–50 derived from Table 5 decision logs and weekly review history.
- v1.1 — Original Decision Engine, Rules 1–15.

---

## 1. Operating Principles

1. Maximize RevPAR, not occupancy — **except** during documented low season, when occupancy takes priority over ADR within margin-floor limits (see RM-032).
2. Protect ADR until demand proves discounting is necessary.
3. Always fix the closest broken booking window first (30 → 60 → 90 → 120).
4. Price for future demand, not current occupancy.
5. Diagnose before discounting.
6. Change as few variables as possible at one time (see RM-017, RM-051).
7. Every pricing change must have a measurable trigger, a cited rule, and an expected outcome.
8. Manual overrides are for exceptions only.
9. Data integrity precedes decisions: exceptions, blocks, and legacy distortions are resolved before any rule fires.

## 2. Precedence Hierarchy

When rules conflict, apply in this order (higher blocks lower):

1. **Data-integrity gates** — RM-010 Exception Handling, RM-042 Blocked Calendar, RM-039 Legacy Distortion, RM-034 Post-Event Inflation. If the data is unreadable or distorted, no pricing rule fires on it.
2. **Absolute guardrails (Policy)** — RM-044 Margin Floor, RM-047 Escalation Triggers. Never overridden by any occupancy or velocity rule.
3. **Hold rules** — RM-016 Hold After Positive Response, RM-018 Do-Nothing, RM-019 Settling Period, RM-020 Velocity Silence. A hold rule firing blocks action rules on the same variable.
4. **Diagnostic-first rules** — RM-009/RM-043 Listing Conversion, RM-021 Weekday/Weekend Split, RM-030 Booking-Window Alignment. Diagnosis must complete before price action.
5. **Action rules** — all remaining rules.
6. **Cadence cap** — RM-051 limits total decisions per property per weekly cycle to 2–3, regardless of how many action rules trigger. Highest-priority broken window wins the available decision slots.

## 3. Cadence (Canonical Ruling)

The specific weekday of the review is a parameter, not a rule. What is mandatory:

- **One review per property per week.** Complete Tables 1 and 4 (weekly), Tables 2 and 3 (first review of the month), Table 5 (every review).
- **Maximum 2–3 decisions per property per cycle.** A "decision" is one variable change (base, min, max, LMD structure, FOP structure, weekday discount, LOS rule, Booking Recency toggle, channel action).
- **Wait one full weekly cycle before evaluating any change.** Booking velocity (7-day and 15-day bookings) at the next review is the evaluation signal. Do not stack or reverse a change mid-cycle unless RM-047 escalation fires or the affected dates are inside 14 days (perishable inventory exception).

## 4. Metric Definitions

Suffix **(P)** = Past — trailing 30 days. Suffix **(N)** = Next — forward-looking booked nights.

| Term | Definition |
|---|---|
| ADR (P) | Total revenue ÷ nights sold, trailing 30 days |
| Occupancy (P) | Nights sold ÷ available nights, trailing 30 days |
| RevPAR (P) | Revenue per available night, trailing 30 days (Occ × ADR) |
| Revenue YTD | Total revenue year-to-date vs. annual goal |
| 30/60/90/120-Day Occ (N) | Percentage of nights already booked within each forward window |
| Avg Lead Time (P) | Average days between booking and arrival, trailing 30 days |
| Avg LOS (P) | Average nights per reservation, trailing 30 days |
| Variance | Current Occ (N) − Target Occ for the window; negative = behind |
| Status | Variance interpretation: Very Far Behind / Behind / Slightly Behind / On Track / Ahead / Very Far Ahead |
| Booking Velocity | Nights booked in the trailing 7 and 15 days; the primary feedback signal for every decision |
| FOP | Far-Out Premium — % increase on reservations made beyond a set lead time |
| LMD | Last-Minute Discount — graduated % decrease as arrival approaches; depth must increase toward arrival, never decrease |
| Booking Recency Factor | PriceLabs setting weighting recent booking activity into recommendations; graduated discount when bookingless days accumulate |
| Floor / Safety Minimum | Absolute minimum nightly price; no discount stack may breach it |
| Orphan Day Price | Pricing for single open nights between reservations |
| Severity grades | Slightly Behind ≤10 pts off pace · Moderately Behind 10–25 pts · Critically Behind >25 pts or 10+ bookingless days while behind pace |

**Available nights** exclude owner blocks, maintenance, and remediation blocks (see RM-042). Occupancy computed against blocked inventory is invalid input.

## 5. Property Parameters (Targets Are Parameters, Not Constants)

No rule contains a hardcoded universal target. Rules read the property's parameter set:

| Parameter | Standard Default | Notes |
|---|---|---|
| 30-day Occ (N) target | 65–75% | Lulu (seasonal low): 35–50% · Fat City: 55–70% |
| 60-day Occ (N) target | 45–55% | Lulu: 20–35% · Fat City: 40–55% |
| 90-day Occ (N) target | 30–40% | Lulu: 10–20% · Fat City: 25–40% |
| 120-day Occ (N) target | 20% | Lulu: 10% · Fat City: 15% |
| ADR / RevPAR / Revenue targets | per property tab | Season-adjusted per RM-033 |
| Margin floor | per property underwrite | Cleaning + fees (~15%) + utilities/supplies + wear |
| Booking-window class | per RM-030 | Late-booking / Balanced / Early-booking |
| Season calendar | per property tab | Peak / Shoulder / Soft by month; targets shift per RM-033 |
| 1-night premium multiplier | 1.5× base | Canonical value; 1.4× entries in legacy tabs are superseded |
| Review day | Tuesday (current) | Parameter only; cadence rules in §3 are what bind |

## 6. Legacy Rule Mapping

| Legacy source | Legacy rule | Canonical |
|---|---|---|
| Reference Guide v2 / START HERE | Rule 2 No Traction + Rule 8 Booking Freeze | **RM-002** (merged; Booking Freeze = Critically Behind branch) |
| Reference Guide v2 | Rule 3 Weekend Compression | RM-004 |
| Reference Guide v2 | Rule 4 Pace Protection | RM-003 (targets now parametrized) |
| Reference Guide v2 | Rule 5 Last-Minute Fill | RM-005 |
| Reference Guide v2 | Rule 6 Window Priority | RM-006 + RM-049 |
| Reference Guide v2 | Rule 7 Far-Out Guardrails | RM-006 (canonical Far-Out Pricing) |
| Decision Engine v1.1 | Rules 1–15 | RM-001–RM-015 (same order) |
| v2.0 Expansion | Rules 16–50 | RM-016–RM-050 |
| Old Desert Dog tab LMD (7d −30% / 3d −20% / 1d −10%) | — | **Deprecated — data-entry error.** Inverted curve violates RM-005. Do not learn from this entry. |

External marketing references to "nine confidential rules" describe the public-facing framing of this system and are not a rule source.

## 7. Confidence Tiers

- **Policy** — absolute; not subject to evidence revision: RM-010, RM-044, RM-047.
- **Proven** — multiple portfolio observations with logged velocity outcomes.
- **Provisional** — single observation or strong pattern; apply, but flag in output.
- **Hypothesis** — design-stage or outcome pending; apply only with Medium/Low confidence noted and escalate if stakes are high.

Aira must state the tier of every rule cited. Tiers are promoted or demoted as Table 5 evidence accumulates.

---

## 8. Canonical Rules

### Core Engine (RM-001 – RM-015)

**RM-001 — Early Traction** *(Proven)*
Trigger: 5+ nights book within 14 days of dates opening, OR bookings occur 60+ days before arrival. Action: raise base 5–8%; leave min unchanged. Why: early bookings prove willingness to pay. Exceptions: major holidays, new listings, unique events.

**RM-002 — Booking Velocity / No Traction** *(Proven — merged Booking Freeze)*
Trigger: no bookings for 7–10 days OR occupancy pace below target. Action by severity:
- Slightly Behind (≤10 pts): lower base 3–5%.
- Moderately Behind (10–25 pts): lower base 5–8%; reduce weekday pricing 5–10%; introduce 21–30-day discounts; trim weekend premium ~5% (only where a static premium exists).
- Critically Behind (>25 pts OR 10+ bookingless days while behind pace) — full correction set: lower base 8–15%; temporarily lower min 5–10% (never below margin floor); activate full discount curve (30 → 21 → 14 → 7 days); targeted overrides on unbooked nights inside 14–21 days; enable Booking Recency (RM-046).
Fix the 30-day window first. Escalate conservatively — never open at the critical response. Exceptions: RM-020 (long-lead silence), RM-042 (blocks), RM-035 (dead market), owner blocks, seasonal curve shifts.

**RM-003 — Occupancy Pace Protection** *(Proven)*
Trigger: any forward window below its property-parameter target. Action: 30-day behind → weekday pricing only (−5 to −15%); 60–90-day behind → base −3–5% plus far-out structure adjustments; never blanket-discount all nights. Exceptions: data-integrity gates.

**RM-004 — Weekend Compression** *(Proven — clarified)*
Trigger: weekends filling 30+ days before arrival. Action: raise Fri/Sat pricing 10–15% on **remaining open weekend nights**; enforce weekend LOS minimums.
Clarification (canonical ruling): **do not configure or stack static weekend premiums on weekends that are already selling out or already sold.** A premium on sold inventory does nothing, and weekends that reliably sell at current pricing need compression raises on remaining open nights (this rule, RM-023), not a standing premium. Static weekend premiums are for properties where weekend demand needs shaping (e.g., Thriveway +8–12%). "Don't do" notes on property tabs encode this ruling.

**RM-005 — Last-Minute Fill** *(Proven)*
Trigger: unbooked nights approaching arrival. Action: graduated curve — 30d −5% · 21d −10% · 14d −12–15% · 7d −18–20% · 3d −22–25% · 1d −25–30%; allow shorter stays inside 5–10 days per RM-024; remove weekend premiums inside 7–14 days if unbooked; targeted gap overrides. Depth must increase toward arrival — an inverted curve is invalid configuration. Influence bookings before the last-minute window, not only within it. Floor per RM-044 always binds.

**RM-006 — Far-Out Pricing** *(Proven)*
Trigger: strong demand 45+ days out. Action: 45–59d +5–10% · 60–89d +8–15% · 90+d +10–20%, scaled by booking-window class (RM-030). Apply only with demand support. Future inventory is the most valuable inventory.

**RM-007 — Length-of-Stay Optimization** *(Proven)*
Trigger: LOS restrictions limiting bookings, or gap/orphan patterns. Action: raise minimum stays in high demand, relax in low demand, allow premium 1-night gap fills per RM-024. LOS moves occupancy as much as price.

**RM-008 — Market Compression** *(Proven)*
Trigger: market occupancy, competitors, or events indicate compression (quantified thresholds in RM-036). Action: raise pricing, protect ADR, delay discounts. Compression is being ahead of expected pace relative to history and market — not "feeling busy."

**RM-009 — Listing Conversion** *(Proven)*
Trigger: high views, low bookings (check 30/90-day views, conversion rate, wishlist adds). Action: audit hero photo, photo order, title, amenities, reviews, fees, rules BEFORE any price change. Post-fix protocol in RM-043.

**RM-010 — Exception Handling** *(Policy)*
Trigger: holidays, festivals, events, owner blocks, maintenance, new listings, disasters, severe weather. Action: pause automated rules; review manually; log the exception. This gate runs first in every review.

**RM-011 — ADR Protection** *(Provisional)*
Trigger: occupancy on pace but ADR below target. Action: raise pricing. Occupancy without rate is not success.

**RM-012 — Market Outperformance** *(Provisional)*
Trigger: property materially outperforms market. Action: raise pricing more aggressively than standard increments.

**RM-013 — Market Underperformance** *(Provisional)*
Trigger: property underperforms market. Action: diagnose cause (listing, comp set, structure, seasonality) before discounting.

**RM-014 — Booking Curve Shift** *(Provisional)*
Trigger: guests booking materially earlier or later than historical norms. Action: reclassify booking-window class (RM-030) and restructure FOP/LMD accordingly.

**RM-015 — Revenue Opportunity** *(Provisional)*
Trigger: orphan nights and adjacent-reservation opportunities. Action: optimize per RM-024 to maximize total stay revenue.

### Response Discipline (RM-016 – RM-020)

**RM-016 — Hold After Positive Response** *(Proven)*
Trigger: change made in the last cycle AND velocity or 30-day pace improved. Action: no further reductions; hold one full cycle; confirm 60-day pace follows 30-day upward. Evidence: Thriveway 315→295 → 30-day occ 30%→57%, held; Fat City held 3 cycles post-recovery. Exceptions: a window the change could not affect may be adjusted under RM-028 within the cadence cap.

**RM-017 — Decision Cap Discipline** *(Proven — updated per canonical ruling)*
Trigger: every review. Action: maximum 2–3 decisions per property per weekly cycle (see §3); never change base, min, weekday discounts, and LMD simultaneously outside the RM-002 Critically Behind reset. Preserve attribution. Evidence: Trixie's two-move adjustment (base 190→180, min 125→115) with all other variables frozen.

**RM-018 — Do-Nothing Rule** *(Proven)*
Trigger: all windows within ±5 pts of target AND ≥1 booking in last 7 days. Action: no changes; log "no action — on pace." Evidence: Azure hold weeks followed by unprompted 9–11 booking resumption.

**RM-019 — Change Settling Period** *(Proven)*
Trigger: structural change (base/min/FOP/LOS) within the last cycle. Action: do not reverse or deepen that variable until one full cycle of velocity data exists; expect a 7–10 day lag after structural resets. Evidence: Thriveway rebuild lag; Trixie's whipsaw counter-example (raise to $215 mid-slump, reversed weeks later at cost). Exceptions: dates inside 14 days (perishable).

**RM-020 — Velocity Silence on Long-Lead Properties** *(Proven)*
Trigger: avg lead time ≥60 days AND 0 bookings in 7–15 days AND all windows on-track-or-ahead. Action: hold; do not fire RM-002; log "velocity silence — normal for lead-time profile." Evidence: Azure (lead 84–133 days) repeatedly quiet while 22–45 pts ahead; bookings resumed unprompted. Exceptions: 30-day window behind → RM-005 on those dates only.

### Calendar Structure (RM-021 – RM-027)

**RM-021 — Weekday/Weekend Split Diagnosis** *(Proven)*
Trigger: occupancy below target AND weekend occupancy ≥85%. Action: diagnose as weekday problem; Sun–Thu discounts only; never lower base, weekend pricing, or min for a weekday problem; at 100% weekend occ consider RM-004 raises on remaining weekends. Evidence: Thriveway (88% weekend), Desert Dog (100% weekend — "the market is buying weekends, ignoring weekdays").

**RM-022 — Month-Targeted Discount Laddering** *(Proven)*
Trigger: pace deficit concentrated in identifiable future months. Action: date-range weekday discounts scaled by distance — deepest nearest (e.g., −15% / −8% / −5% across consecutive months). Evidence: Thriveway July/Aug/Sept ladder; Trixie's July–Aug 5%. Exceptions: never ladder into peak months.

**RM-023 — Paired-Night Weekend Compression** *(Provisional)*
Trigger: Fri or Sat books; paired night open. Action: raise the open paired night +15–25% immediately (manual override acceptable). Inside 7 days unbooked pairs convert to RM-005.

**RM-024 — Orphan & Gap Night Monetization** *(Proven)*
Trigger: 1-night gaps or repeated Sun–Thu holes. Action: 1-night stays only inside 5–7 days at ≥1.5× base (canonical multiplier), or +20–25% gap premium, or require adjacent nights book together. Never default-priced 1-nights. LMD abuse guard: if any booking inside 3 days lands below the quality floor or >2 one-night stays occur in 30 days, cap LMD at −10%. Exceptions: MTR gap smoothing per RM-050.

**RM-025 — Floor-Flattening Detection** *(Proven)*
Trigger: significant share of upcoming weekdays priced at/near min. Action: diagnose as collapsed pricing curve, not low demand; either lift weekdays manually above floor OR lower base to reopen spread — never both. Evidence: Desert Dog June floor-pinning; Trixie's and Moonlight manual weekday lifts.

**RM-026 — Minimum-to-Base Spread Protection** *(Proven)*
Trigger: any base or min change. Action: keep min ≈60–70% of base; if lowering min for occupancy, do not collapse base toward it. Evidence: Trixie's $180/$115 spread rationale. Exceptions: MTR (RM-050).

**RM-027 — Wide-Band Strategy** *(Provisional)*
Trigger: behind near-term with compression on specific dates. Action: lower min modestly AND raise max in the same review (counts as one structural move within the decision cap). Evidence: Desert Dog 131→125 min with 400→450 max; far windows subsequently ahead.

### Two-Speed & Window Pricing (RM-028 – RM-031)

**RM-028 — Two-Speed Calendar Split** *(Proven)*
Trigger: near window behind while far windows ahead (or inverse). Action: discount tactically inside the broken near window while simultaneously raising FOP/base/max on ahead windows; treat as separate problems within the decision cap. Evidence: Azure −10% next-2-weeks + FOP 35% after 60 days in one review; Lulu near-term discounts + 20% FOP after 75 days.

**RM-029 — FOP Window Tuning** *(Proven)*
Trigger: one mid-range window lags while adjacent windows are fine and FOP begins inside the lagging window. Action: move FOP start beyond the lagging window instead of cutting base; when far windows run ahead, raise FOP % or extend range. Evidence: Thriveway FOP 45→60 days for the 60-day window; Desert Dog FOP 10%/60d with 90/120 staying ahead.

**RM-030 — Booking-Window Alignment** *(Proven)*
Trigger: onboarding, quarterly review, or distribution update. Action: classify and structure accordingly — Late-booking (≥30% inside 7 days, lead <~25d): FOP 0–8%, strong LMD, tolerate thin far calendars. Early-booking (majority 60+ days, lead ≥60d): FOP 10–35%, minimal early discounts, judge far windows seriously. Evidence: Desert Dog (33% inside 7d, median 17d) vs. Moonlight (71% at 60+d, FOP to 45%). Reclassify per RM-014.

**RM-031 — Calendar Opening Protocol** *(Provisional)*
Trigger: booking window <6 months, or dead 60–90-day pipeline. Action: open 6–9 months in one move; layer FOP across the opened range same day; add 3-night weekend minimums 30+ days out; if early bookings don't appear, check comps and trim FOP — do not lower base. Evidence: Lulu 9-month opening; Desert Dog opening ladder.

### Seasonality & Market (RM-032 – RM-038)

**RM-032 — Low-Season Protocol** *(Proven)*
Trigger: documented soft season AND market-level demand down (not property-specific). Action: occupancy priority within limits — remove/zero FOP; deepen LMD to 30–40% over 14 days; enable Booking Recency; lower min toward (never below) margin floor; keep weekend structure; frame Table 5 Key Issue as season-driven. Evidence: Trixie's recovery to 12/7d–20/15d; Desert Dog summer at 100% weekend occ. Floor (RM-044) always binds.

**RM-033 — Season-Relative Targets** *(Proven)*
Trigger: setting/reviewing targets. Action: monthly bands by season — Peak 80–90%, Shoulder 70–80%, Soft 55–70%; Table 4 status computed against the season-adjusted band (low end in soft season). Evidence: Lulu's low-end status labels.

**RM-034 — Post-Event Inflation Adjustment** *(Provisional)*
Trigger: trailing metrics include a major event period. Action: flag trailing metrics inflated; decide on forward pacing and new-booking ADR; never raise base off event-inflated trailing ADR. Evidence: Lulu post-Coachella (ADR (P) $614 vs. $230–260 target).

**RM-035 — Distribution Before Desperation** *(Hypothesis — outcome pending)*
Trigger: market occupancy near zero; multiple cuts failed; forward occ near zero. Action: stop cutting; expand channels (VRBO, Booking.com, direct); rework listing (RM-009/043); log market condition as key issue. Evidence: Lulu — cuts produced nothing; VRBO/BDC launched; results pending.

**RM-036 — Compression Thresholds** *(Provisional)*
Triggers/Actions: 30-day (N) >80–85% → raise base 5–10% + raise min · 60-day (N) >70–75% → raise FOP · any month 90+ days out at 30–40% occ → raise that range 8–12% · rolling 60-day >85% or 3 months projecting >85% → raise base 5–10%, min +5%, halve LMD. Validate against historical pace and market norms — compression is market-relative. Evidence: Moonlight Rules A/C; Azure serial raises to $625 while 28–45 pts ahead.

**RM-037 — Overperformance / "Too Easy" Signal** *(Proven)*
Trigger: pace +10 pts (30-day) or +5 pts (60-day) over target; or trailing occ ≥85–90%; or effortless bookings across windows. Action: raise base 5–10%, weekend pricing 10–15% on open nights, consider raising min. Trailing 85–90%+ occupancy = underpricing, not success. Exceptions: legacy-inflated occupancy (RM-039).

**RM-038 — Anchor Recovery on Peak Dates** *(Provisional)*
Trigger: holiday/peak weekend discounted before ~10 days out, now below market. Action: raise back toward market; cheap pricing on premium dates signals low quality. Evidence: Moonlight Memorial Day. Exceptions: inside 7 days → RM-005.

### Takeovers & Data Integrity (RM-039 – RM-043)

**RM-039 — Legacy Reservation Distortion** *(Proven)*
Trigger: takeover with inherited reservations in forward windows. Action: evaluate ADR on new bookings only; treat legacy-inflated forward occupancy as a false "ahead" signal; keep raising until new bookings land in target band; log the distortion. Evidence: Azure Table 5 entries; response was continued raises, not celebration.

**RM-040 — Takeover Reset Protocol** *(Proven)*
Trigger: takeover with no usable history or unsalvageable structure. Action, in order: PriceLabs staged not live → base/min/max from comp tier + margin floor → comp set → seasonal rules → weekday/weekend structure → lead-time (FOP/LMD) per market booking window → LOS discounts → minimum stays → gap rules → pacing strategy. Expect 7–10 day traction lag (RM-019). Evidence: Fat City 0 → 9 bookings/7d after rebuild + listing fix.

**RM-041 — Perception Re-Anchoring** *(Provisional)*
Trigger: extended 0 bookings AND base at/below market mid-tier AND no far-out structure. Action: raise base to re-anchor, raise min, replace early deep discounts with a later steeper curve, add FOP ladder. Structural reset, not a discount play. Exceptions: dead market → RM-035 governs.

**RM-042 — Blocked-Calendar Exception Handling** *(Proven)*
Trigger: significant blocked nights (owner, remediation, maintenance) in any window. Action: exclude blocked nights from denominators; mark affected windows unreadable; no pricing action on distorted pace; log block as Key Issue. Evidence: Moonlight fire remediation weeks; Desert Dog owner-block weeks.

**RM-043 — Listing Fix Verification** *(Proven)*
Trigger: high views + low conversion, or suspected merchandising problem. Action: fix listing first (RM-009); then hold pricing 1–2 cycles and attribute velocity change to the fix. Evidence: Fat City — pricing held, 0 → 9 bookings.

### Guardrails (RM-044 – RM-047)

**RM-044 — Margin Floor** *(Policy)*
Floor = cleaning per turn + platform fees (~15%) + utilities/supplies + wear. Min ≥ floor; no discount stack (LMD, Recency, overrides) may breach it; vacancy > below-floor bookings. Documented floors: Trixie's $115, Leaning Joshua $190, Moonlight $300, Azure $275, Ellis $120+ (≤3-night). Overrides all occupancy rules.

**RM-045 — Discount Timing Guard** *(Proven)*
Trigger: proposed discounts starting earlier than the property's average lead time. Action: reject/restructure; light influence 21–30d (−5–10%), aggressive only inside 7d, tactical inside 3d; depth grows toward arrival. Early deep discounts train guests to wait. Exceptions: RM-032 permits deeper 14-day discounts in low season.

**RM-046 — Booking Recency Deployment** *(Proven)*
Trigger: Moderately/Critically Behind needing continuous pressure between reviews. Action: enable with graduated cap (~5% after 15 bookingless days scaling to max 10–15%); disable/reduce on recovery; never stacks below floor. Not used on properties in compression or ahead of pace. Evidence: Azure, Trixie's, Fat City activations each followed by pickup.

**RM-047 — Escalation Triggers** *(Policy)*
Escalate to DJ, do not act, when: revenue −15%+ WoW; 14+ bookingless days while behind pace; major event dates unbooked inside 45 days; recommendation would breach min/max guardrails or the floor; conflicting rules with low confidence. Output diagnosis + evidence, mark confidence low, route for manual decision.

### Governance (RM-048 – RM-051)

**RM-048 — Weekly Decision Log Discipline** *(Proven)*
Every review logs: date, 30-day occ, Key Issue (one sentence, root-cause framed, season-accurate), Action Taken (old → new values), 7-day and 15-day bookings, and — added in v2.1 — Result at the following review. "No changes" is a logged action.

**RM-049 — Window Priority With Severity Grading** *(Proven)*
Fix closest broken window first (30 → 60 → 90 → 120). Severity grades per §4 definitions. One lever per window: 30-day → weekday pricing/LMD · 60-day → base + month-targeted discounts · 90-day → FOP only · 120-day → usually nothing.

**RM-050 — MTR Rulebook Separation** *(Provisional)*
MTR units switch rule sets entirely: occupancy in months occupied (target 90–95%; <85% = pricing/marketing/vacancy-response investigation, not a discount trigger); hold rent firm in peak; soft periods use incentives (flexible terms, move-in credits) — never rent cuts; response speed > price; vacant + <30% booked at 30 days → marketing investigation first; healthy demand appears 15–45 days out; STR backfill only when economics beat monthly rate, 5–7-night minimum. Open item: reconcile MTR season calendars (2420 11th Ave lists Peak Aug–Oct/Jan–Mar; CR 2.0 tabs mark Jan Soft, Mar–May Peak) — pending DJ ruling; until then use each unit's own tab.

**RM-051 — Weekly Review Cadence** *(Policy — canonical ruling)*
One review per property per week; maximum 2–3 decisions per property per cycle; one full cycle of velocity data before evaluating or modifying any change. The review weekday is a parameter. Perishable-inventory exception: dates inside 14 days may receive tactical LMD/override action mid-cycle without counting against the next cycle's evaluation.

---

## 9. Outcome Evidence Table

| Property | Action | Velocity Before | Velocity After | Rules |
|---|---|---|---|---|
| Thriveway | Base 315→295, 25% LMD/14d, −10% weekdays | 0/7d, 30-day occ 30% | 6/7d, 30-day occ 57% | RM-016, 021, 049 |
| Thriveway | FOP 45→60 days | 60-day occ lagging at 32% | window unblocked, held | RM-029 |
| Trixie's | Min→125, Base→190, 40% LMD, FOP removed, Recency on | 0/7d, 0/15d | 12/7d, 20/15d | RM-032, 046 |
| Trixie's | Raise min $165/base $215 mid-slump (reversed) | slowing | worsened — whipsaw | RM-019, 045 (counter-example) |
| Fat City | Listing fixed, pricing held $95/$125/$400 | 0 bookings | 9/7d | RM-043, 016 |
| Desert Dog | Min 131→125 + Max 400→450 + FOP 10%/60d | very far behind, weekends 100% | 90/120-day ahead; 11–15/15d | RM-027, 029, 021 |
| Azure | Serial raises 415→475→625, FOP 30–35%, min 425 | far windows 28–45 pts "ahead" (legacy-inflated) | new bookings at target ADR ($473) | RM-036, 039, 028 |
| Azure | 0 bookings/15d while ahead → held | quiet | 9–11 bookings resumed | RM-020, 018 |
| Lulu | Repeated cuts in dead summer market | 0 | 0 | RM-035 (price not the constraint) |
| Lulu | VRBO + BDC launch, listing rework, min trim | 0 | pending | RM-035, 043 |
| Moonlight | June–July remediation block, no actions | n/a | n/a | RM-042 |

---

## 10. Output Contract (What Aira Must Produce Per Review)

For each property, in order: (1) Exception check (RM-010/042/039/034); (2) window-by-window status vs. property parameters; (3) velocity read; (4) diagnosis with root cause; (5) recommendation(s) — max 2–3, each citing rule ID + tier + evidence + expected outcome + confidence (High/Medium/Low); (6) Table 5 log entry text; (7) escalation flag if RM-047 fires. Hold recommendations ("no action") are first-class outputs, not defaults to avoid.

*ProHosts Revenue Management Decision Engine v2.1 — Maintained by DJ Fornelli. Rule additions require a documented trigger, action, evidence citation, and assigned tier. This file and its YAML companion are the sole canonical sources.*

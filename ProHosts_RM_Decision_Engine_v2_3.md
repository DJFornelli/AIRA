# ProHosts Revenue Management Decision Engine — v2.3

**Powers AIRA (Artificial Intelligence Revenue Agent).**
Changes from v2.2: adds RM-000 (Prime Directive), RM-052 (New-Listing Ramp), RM-053 (Seasonality), RM-054 (Event Windows), RM-055 (Booking-Curve Window Validity), RM-056 (Far-Out Premium Calibration), RM-057 (RM-009 exit).

All new rules are derived from 1,409 reservations / 188 property-months across 12 properties (Jan 2023 – Jul 2026). Evidence cited inline.

---

## RM-000 — PRIME DIRECTIVE (supersedes all other rules)

**Maximize annual revenue (ADR × nights sold) per property, subject to the margin floor.**

- Occupancy is **not** a goal. ADR is **not** a goal. Both are *levers*.
- The target is the point where the next dollar of rate gained costs less than the occupancy it sheds — and vice versa.
- Horizon is **annual**, not weekly. A move that lifts this week's occupancy but trains the market to wait for discounts is a **failure** under RM-000.
- When any rule below conflicts with RM-000, RM-000 wins.

**Evidence — high occupancy is routinely a pricing failure:**

| Property | Month | Occ | ADR | RevPAR |
|---|---|---|---|---|
| Koya | Sep 2024 | **97%** | $93 | **$90** |
| Koya | Mar 2024 | 87% | $176 | **$153** |
| Leaning Joshua | Jan 2025 | **94%** | $116 | **$108** |
| Leaning Joshua | Apr 2025 | **50%** | $220 | **$110** |
| Moonlight | Mar 2025 (old regime) | **100%** | $316 | $316 |
| Moonlight | Mar 2026 (repriced) | 84% | $356 | $298 |

Koya at 97% made **less per available night** than Koya at 87%. Leaning Joshua at 50% occupancy out-earned itself at 94%. **Never celebrate occupancy.**

### Margin floor — PROVISIONAL

Two floors, both respected:
- **Economic floor** = variable cost per night (cleaning + platform fee + supplies + utilities). **NOT YET KNOWN for any property.**
- **Strategic floor** = the current min-price setting. A *judgment* number, not cost-derived.

Until true cost per night is supplied, **the strategic floor is treated as hard** and AIRA must:
- Never recommend below it.
- Flag when a property repeatedly presses against it (the floor may be set wrong in either direction).
- Never claim a floor is "profitable" — that is unknown.

---

## RM-053 — SEASONALITY (market-derived, replaces per-property season labels)

Season is read from the **market profile**, not property guesswork. Each property inherits its market's seasonal shape and learns its own offset.

### Two market *types* — this distinction is mandatory

| | Joshua Tree | Palm Springs |
|---|---|---|
| Occupancy range | **26%–100%** (swings) | **72%–91%** (stable) |
| ADR range | ~$159–$225 (stable) | **$200–$433 (2.2×)** |
| Season mechanic | **OCCUPANCY-season** | **ADR-season** |
| Therefore | Season moves *how many* nights sell | Season moves *what a night is worth* |

**Consequence:** a Palm Springs property at 80% occupancy in July is *normal*. A Palm Springs property whose **ADR** is flat across the year is **leaving the entire seasonal premium on the table**. In Joshua Tree the reverse: judge occupancy, hold rate.

### Joshua Tree seasonal profile
*8 properties, 146 property-months, 2023–2026. Occupancy index = month ÷ that property's annual mean.*

| Month | Occ idx | Occ mean | ADR | RevPAR | Season |
|---|---|---|---|---|---|
| Jan | 1.07 | 65% | $189 | $124 | SHOULDER+ |
| Feb | 1.09 | 68% | $204 | $144 | SHOULDER+ |
| Mar | 1.08 | 70% | $211 | $154 | SHOULDER+ |
| **Apr** | **1.20** | **73%** | **$225** | **$170** | **PEAK** |
| May | 0.96 | 61% | $193 | $122 | SHOULDER |
| Jun | 0.92 | 57% | $171 | $100 | SHOULDER |
| **Jul** | **0.86** | **57%** | **$159** | **$95** | **SOFT** |
| Aug | 0.90 | 56% | $176 | $98 | SHOULDER |
| Sep | 0.89 | 59% | $171 | $101 | SHOULDER |
| Oct | 0.91 | 58% | $183 | $106 | SHOULDER |
| Nov | 1.09 | 66% | $221 | $146 | SHOULDER+ |
| Dec | 0.98 | 61% | $211 | $135 | SHOULDER+ |

### Palm Springs seasonal profile
*Yara Hotel, 10 rooms, 24 months. ADR index = month ÷ annual mean ADR.*

| Month | Occ | ADR | ADR idx | RevPAR | Season |
|---|---|---|---|---|---|
| Jan | 80% | $281 | 0.98 | $224 | SHOULDER |
| **Feb** | 83% | **$433** | **1.51** | **$357** | **PEAK** |
| **Mar** | 91% | **$381** | **1.33** | **$345** | **PEAK** |
| **Apr** | 89% | **$362** | **1.27** | **$322** | **PEAK** |
| May | 86% | $276 | 0.96 | $236 | SHOULDER |
| Jun | 77% | $204 | 0.71 | $156 | SOFT |
| Jul | 80% | $200 | 0.70 | $161 | SOFT |
| Aug | 86% | $200 | 0.70 | $172 | SOFT |
| Sep | 84% | $205 | 0.72 | $172 | SOFT |
| Oct | 85% | $282 | 0.98 | $239 | SHOULDER |
| **Nov** | 80% | **$349** | **1.22** | **$278** | **PEAK** |
| Dec | 72% | $262 | 0.92 | $190 | SHOULDER |

### RM-053a — Season-positioned target ranges
Table 1 target *ranges* already encode seasonality. Read them **positionally**:
- **PEAK** → judge against the **top** of the range.
- **SHOULDER+/SHOULDER** → judge against the **middle**.
- **SOFT** → judge against the **bottom**.

A property at the low end of its ADR range in April is **underperforming**. The same ADR in July is **on target**.

### RM-053b — Forward-window season matching
The 60d/90d/120d windows land in *future* months. Judge each window against **the season it lands in**, not today's season. In July, the 90d window is October (SHOULDER) — do not price it like July (SOFT).

### RM-053c — Season transition
Within 30 days of a SOFT→PEAK transition: **stop discounting, hold rate for the ramp.** Within 30 days of PEAK→SOFT: release inventory earlier.

### ⚠️ RM-053d — LABEL CORRECTION REQUIRED (needs DJ review)
The season labels currently in AIRA for Joshua Tree properties **contradict 4 years of data across 8 properties**:

| Month | App label | Data | Status |
|---|---|---|---|
| Mar | Peak | SHOULDER+ | **MISMATCH** |
| **Apr** | **Shoulder** | **PEAK** | **MISMATCH — April is JT's best month** |
| Jun | Soft | SHOULDER | overstated |
| Aug | Soft | SHOULDER | overstated |
| Sep | Soft | SHOULDER | overstated |
| **Nov** | **Peak** | SHOULDER+ | **MISMATCH** |
| **Dec** | **Peak** | SHOULDER+ | **MISMATCH** |

**Joshua Tree's true peak is April. Nov–Dec are good, not peak.** Do not deploy until DJ confirms — market knowledge may explain the divergence.

---

## RM-055 — BOOKING-CURVE WINDOW VALIDITY (blocks meaningless alarms)

**A pace window is only valid if the property actually books into it.**

Before flagging any window Behind/Very Far Behind, check the property's measured far-out share:

- **<10% of bookings arrive 60+ days out → the 60d/90d/120d windows are NOT diagnostic.** Report them as *structurally thin — expected*, never "Very Far Behind."
- **<5% at 60+ → suppress those windows from the priority-action logic entirely.**

**Evidence — measured booking curves:**

| Property | n | Median lead | ≤7d | ≤30d | **60d+** |
|---|---|---|---|---|---|
| Finca Nopali | 225 | 38d | 16% | 44% | **35%** |
| Jackalope | 194 | 23d | 18% | 57% | 20% |
| Cantina | 169 | 23d | 23% | 62% | 17% |
| Koya | 130 | 18d | 27% | 65% | 11% |
| Desert Dog | 114 | 17d | 28% | 69% | 16% |
| Trixie's Oasis | 139 | 16d | 21% | 69% | 11% |
| **Fat City Ranch** | 26 | **10d** | 42% | 88% | **0%** |
| Leaning Joshua | 54 | 8d | 50% | 81% | 6% |

**Fat City has never taken a single booking 60+ days out.** Flagging it "Very Far Behind at 60d/90d/120d" is a false alarm generated by the engine, not a business problem.

---

## RM-056 — FAR-OUT PREMIUM CALIBRATION

FOP must be sized to **measured** far-out demand:

- **60d+ share ≥30%** → FOP justified, up to +35%
- **60d+ share 15–29%** → FOP capped at +8%
- **60d+ share <15%** → **remove FOP entirely.** It taxes a segment that does not exist.

**Audit of current settings:**

| Property | Current FOP | Measured 60d+ | Verdict |
|---|---|---|---|
| Azure Fern Haus | +35% / +35% | 42% *(n=12, low conf)* | ✅ OK |
| Moonlight Sunrise | +15% / +15% | 62% | ✅ OK — consider raising |
| Desert Dog | +15% / +15% | 16% | ⚠️ Reduce to ≤+8% |
| **Trixie's Oasis** | **+18% / +18%** | **11%** | ❌ **REMOVE** |
| **Fat City Ranch** | +5% / +3% | **0%** | ❌ **REMOVE** |
| **Lulu at the Palms** | +8% / +8% | **0%** *(n=13)* | ❌ **REMOVE** |
| **Thriveway Stays** | +15% / +10% | **9%** *(tracker, n=68)* | ❌ **REMOVE** |

**Tier gradient (Joshua Tree):** premium 38d median / 35% far-out · mid 23d / 18% · value 15d / 11%. Higher tier books further ahead. Set FOP by tier, not by habit.

---

## RM-052 — NEW-LISTING RAMP PROTOCOL

**Trigger:** listing age **90–120 days or less**, OR <10 reviews, AND market in SHOULDER/SOFT.

When active, classify as **RAMP** — not underperformance:

1. **Suppress panic language.** "Very Far Behind" is invalid during ramp; a new listing has no booking curve yet.
2. **Hold base.** Protect the converting segment (usually weekends).
3. **Ramp-up permitted:** light weekday movement **for review acquisition**, not pace. Capped at a few nights, floor-protected. Reviews are the ranking asset; a handful of cheap weekday nights that generate reviews is an *investment*, not a discount.
4. **Objective = the property's next PEAK**, not the current trough.
5. RM-009 may fire **once**; see RM-057.
6. **Confidence is capped at MEDIUM** during ramp. There is no history to be confident from.

**Fat City Ranch is the live case:** launched Mar 2026 into JT's soft stretch, 5 months of data, 0% far-out bookings, weekends at 75%. It is not broken — it is new. Its first true PEAK (April) has not yet occurred in its data.

---

## RM-057 — RM-009 EXIT CONDITION (closes the infinite-hold loop)

RM-009 ("audit listing before changing price") previously had an entrance and **no exit**, so it re-fired weekly forever.

- RM-009 may fire **once** per property.
- Once DJ confirms listing health, mark **RESOLVED — cannot re-fire for 30 days.**
- If a property sits in hold-and-audit for **3+ consecutive cycles**, force a decision: either declare the listing healthy and act, or escalate as a genuine listing/distribution problem (RM-035).
- **"Hold and confirm listing health" is not a strategy. It is a question awaiting an answer.**

---

## RM-054 — EVENT-WINDOW OVERRIDE (Palm Springs; per-market, never cross-market)

Events are **discrete date windows**, not seasons. Their ADR must never be smeared into a monthly baseline.

**Two layers:**

**1. Known anchors (Palm Springs / low desert only):** Modernism Week (Feb), BNP Paribas Open / Indian Wells (Mar), Coachella (Apr, two weekends), Stagecoach (late Apr), Palm Springs Film Festival (Jan), American Express golf (Jan), Pride (Nov).
**Dates shift annually — verify each year. Do not hardcode.**

**2. Dynamic detection (catches everything else):** flag any future date cluster where ADR or market occupancy runs **≥2× the monthly median**. Surface as *suspected event* for DJ to confirm.

**Validation — detection found real events with no hardcoded list:**

| Date | ADR | × median | Occ | Likely event |
|---|---|---|---|---|
| Feb 21–22 2025 (Fri/Sat) | $800 | **3.2×** | 10/10 | Modernism Week |
| **Apr 10–11 2026** | $796 / **$932** | **3.4×** | 10/10 | **Coachella W1** |
| Apr 26 2025 | $604 | 2.3× | 10/10 | Stagecoach |
| Nov 7–8 2025 | $539 | 2.7× | 10/10 | Pride |
| Oct 3–4 2025 | $557 | 2.6× | 7/10 | Fall event |

**Rules:** event dates priced to event demand · event ADR **excluded** from baseline seasonal averages · **Joshua Tree and Palm Springs are separate markets — a Palm Springs event NEVER flags a JT property** (high desert vs low desert; distinct demand).

---

## RM-058 — MARKET BENCHMARK (property vs market)

Where a market profile exists, judge the property against **the market**, not its own history.

**Lulu at the Palms — worked example:** Lulu runs **23–46% occupancy**. The Palm Springs market (Yara, 24mo) runs **72–91% year-round**. Lulu is not suffering a soft season — **the season is not soft.** It is 40+ points below its market. That is a listing/distribution failure (RM-035), and the fix is channels and conversion, **not price cuts**. Confirmed: Lulu's Feb–Jul window shows the market at 77–91% occupancy in the very months Lulu sat near zero.

---

## Gates (v2.2 retained, v2.3 inserted)

**Gate 0 — Prime Directive (RM-000).** Every recommendation states its expected effect on *revenue*, not occupancy.
**Gate 1 — Data integrity.** Incl. RM-010 exceptions. Now also: RM-052 ramp check, RM-055 window validity.
**Gate 2 — Guardrails.** RM-044 floor (PROVISIONAL), RM-047 escalation.
**Gate 3 — Hold rules.** RM-016, RM-020, + RM-057 exit.
**Gate 4 — Diagnose first.** RM-021, RM-009 (once only), RM-030, + RM-058 market benchmark, RM-053 season.
**Gate 5 — Velocity / lead-time alignment.** v2.2 framework retained, now calibrated by measured booking curve (RM-055/056).
**Gate 6 — Cadence cap.** Max 2 decisions per property per cycle.

---

## Confidence tiers — by data depth

| Property | Months | Pace n | Confidence |
|---|---|---|---|
| Desert Dog | 25 | 114 | HIGH |
| Thriveway Stays | 21 | 68 | HIGH |
| Moonlight Sunrise | 17 | 53 | HIGH |
| Trixie's Oasis | 17 | 139 | HIGH |
| Azure Fern Haus | 8 | **12** | **MEDIUM — pace unreliable** |
| Fat City Ranch | 5 | 26 | **LOW — ramp** |
| Lulu at the Palms | 5 | 13 | **LOW — but market benchmark now available** |

AIRA may never report HIGH confidence on a LOW-confidence property.

---

## Open items

1. **Variable cost per night** — unknown for all properties. Floors remain PROVISIONAL until owners supply it.
2. **Azure Fern pace** — 12 reservations. Airbnb export pending; treat far-out reads as tentative.
3. **RM-053d label correction** — requires DJ sign-off before deployment.
4. **Fat City first PEAK** — no April in its data yet. Re-derive after April 2027.
5. **Learning loop** — RM-000's optimum is found by *experiment over time*, not from history. The Confirm Actions → Results loop is the mechanism; history only gets AIRA close.

# ProHosts Revenue Management Decision Engine v2.2
### Last Updated: July 14, 2026
### Changes from v2.1: Added Velocity-Lead Time-Window Alignment Framework

---

## Overview

The ProHosts RM Decision Engine is the rule-based framework executed by AIRA (AI Revenue Agent) during every weekly revenue review. It governs all pricing decisions across the ProHosts portfolio and any managed client portfolios.

All rules execute in gate order. Higher gates block lower gates on the same variable.

---

## Pace Status Thresholds

| Variance | Status |
|----------|--------|
| +13% or more | Very Far Ahead |
| +6% to +12% | Ahead |
| +3% to +5% | Slightly Ahead |
| ±2% | On Track |
| -3% to -5% | Slightly Behind |
| -6% to -12% | Behind |
| -13% or more | Very Far Behind |

---

## Gate 1 — Data Integrity (Policy — runs first always)

- **RM-010:** Check for exceptions before any pricing action — owner blocks, maintenance, remediation, holidays, new listings, disasters. If exception found: flag, do not price.
- **RM-042:** Blocked nights excluded from occ denominators. Mark window unreadable if majority of nights are blocked.
- **RM-039:** Legacy inherited reservations = false occupancy signal. Evaluate new bookings only.
- **RM-034:** Post-event trailing ADR distortion — use forward-pacing data only.

---

## Gate 2 — Absolute Guardrails (Policy — never overridden)

- **RM-044:** No price or discount stack may breach the property's margin floor.
- **RM-047:** Set escalate flag and route to DJ for manual review if any of the following apply:
  - Revenue -15%+ week-over-week
  - 14+ bookingless days while behind pace
  - Major event dates unbooked inside 45 days
  - Any recommendation would breach floor
  - 2+ consecutive failed strategies

---

## Gate 3 — Hold Rules (block action rules on same variable)

- **RM-016:** Last week result = WORKED → hold all reductions this cycle. Confirm 60d follows 30d upward.
- **RM-018:** All windows within ±5pts AND ≥1 booking last 7d → no changes. Log "no action — on pace."
- **RM-019:** Structural change made last cycle → do not reverse or deepen that variable. Allow 7-10 day lag.
- **RM-020:** Lead time ≥60d AND 0 bookings 7-15d AND all windows on/ahead → hold. Log velocity silence — normal for long-lead properties.

---

## Gate 4 — Diagnose First

- **RM-021:** Occ below target BUT weekend occ ≥85% → weekday problem only. Sun–Thu discounts only. Never lower base/min/weekend for a weekday problem.
- **RM-009:** Market occ healthy but property occ low → audit listing before changing price. Flag listing conversion issue.
- **RM-030:** Classify booking window:
  - Late-booking (≥30% inside 7d, lead <25d): FOP 0-8%, strong LMD, thin far calendar is normal
  - Early-booking (majority 60+d, lead ≥60d): FOP 10-35%, minimal early discounts, judge far windows seriously

---

## Gate 5 — Velocity-Lead Time-Window Alignment (NEW in v2.2)

**This gate must be passed before any pricing-INCREASE rule fires.**

### Step A — Velocity Gate
- **0-4 bookings in last 7 days = LOW VELOCITY**
  - HOLD all pricing increases regardless of occupancy status
  - High occ + low velocity = existing bookings filled calendar but demand has stalled
  - Do not raise base price, FOP, weekend premium, or any other variable
- **5+ bookings in last 7 days = HEALTHY VELOCITY**
  - Pricing increases permitted — proceed to Step B

### Step B — Window Match
Identify which occupancy window received the demand signal this week:
- 30d occ increased → **BASE PRICE** is the correct lever
- 60d occ increased → **FOP 45-60d** is the correct lever
- 90d+ occ increased → **FOP 90d+** is the correct lever
- Weekend occ increased → **Fri/Sat premium** is the correct lever

**Never raise a window lever that did not receive the velocity signal.**

### Step C — Lead Time Confirmation
Lead time confirms which booking window is active right now:
- Lead time <15 days → LMD curve is the active lever
- Lead time 15-30 days → Base price is the active lever
- Lead time 30-60 days → FOP 45-60d is the active lever
- Lead time 60-90 days → FOP 90d+ is the active lever

Lead time must confirm the same window identified in Step B. If they conflict → HOLD and diagnose.

**All three steps must pass before any pricing-up recommendation is made.**

---

## Gate 5B — Action Rules

Window priority order: 30d → 60d → 90d → 120d. One lever per window.

- **RM-002 Booking Velocity by severity:**
  - Slightly Behind (≤10pts): base -3-5% only
  - Moderately Behind (10-25pts): base -5-8% + weekday -5-10% + 21-30d discounts
  - Critically Behind (>25pts or 10+ bookingless days): base -8-15% + min -5-10% temp + full discount curve + overrides + Booking Recency

- **RM-003 Pace Protection:** 30d behind → weekday only. 60-90d → base -3-5% + FOP adjust. Never blanket discount.

- **RM-004 Weekend Compression:** Weekends filling 30+d early → raise Fri/Sat +10-15% on OPEN nights only. Requires velocity 5+ AND lead time <30d.

- **RM-005 LMD Curve:** Must deepen toward arrival: 30d -5% / 21d -10% / 14d -12-15% / 7d -18-20% / 3d -22-25% / 1d -25-30%. Inverted curve = invalid.

- **RM-006 Far-Out Pricing:** Strong demand 45+d out → 45-59d +5-10% / 60-89d +8-15% / 90+d +10-20%. Requires velocity 5+ AND lead time confirming far-out booking pattern.

- **RM-011 ADR Protection:** Occ on pace + ADR below target → raise pricing. Only if velocity 5+.

- **RM-016 Hold After Positive Response:** Last week WORKED → do not reduce any variable this cycle.

- **RM-025 Floor-Flattening:** Weekdays pinned to min → lift weekdays OR lower base to reopen spread. Never both.

- **RM-026 Min-to-Base Spread:** Keep min ~60-70% of base. Never collapse spread.

- **RM-028 Two-Speed Calendar:** Near window behind + far window ahead → discount near AND raise FOP simultaneously (counts as 1 structural move).

- **RM-032 Low-Season Protocol:** Soft season + market demand down → zero FOP, LMD 30-40%/14d, enable Recency, lower min toward floor, keep weekend structure.

- **RM-035 Distribution Before Desperation:** Market occ near zero + multiple cuts failed → STOP cutting. Expand channels. Rework listing.

- **RM-037 Overperformance:** Pace +10pts (30d) or +5pts (60d) or trailing occ ≥85% → raise base 5-10%, weekends +10-15% on open nights. Requires velocity 5+ AND lead time alignment.

---

## Gate 6 — Cadence Cap (RM-017, RM-051)

- Maximum **2 decisions** per property per cycle
- One variable change = one decision
- Never change base + min + weekday + LMD simultaneously outside a RM-002 Critically Behind full reset
- Prioritize highest-impact broken window for available decision slots

---

## Confidence Tiers

| Tier | Rules | Description |
|------|-------|-------------|
| Policy | RM-010, 044, 047, 051 | Absolute — never overridden |
| Proven | Most RM-001 through RM-049 | Multi-evidence, reliable |
| Provisional | RM-011-015, 023, 027, 031, 034, 036, 038, 041, 050 | Single observation |
| Hypothesis | RM-035 | Pending validation |

---

## Guiding Principles

1. Maximize RevPAR, not occupancy
2. Protect ADR until demand proves discounting is necessary
3. Always fix the closest broken booking window first
4. Diagnose before discounting — never lower price out of anxiety
5. Change as few variables as possible at one time
6. Every recommendation must have a measurable reason and expected outcome
7. Velocity must confirm demand before any price increase
8. Lead time alignment determines which lever is active
9. When in doubt — hold and monitor


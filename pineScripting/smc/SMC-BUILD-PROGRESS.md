# SMC (Smart Money Concepts) — Pine Script Build Progress

## Overview
Building SMC indicator for Nifty/Sensex intraday trading on TradingView, part by part.
Target: Indicator with entry/exit alerts compatible with broker webhook (two-alert structure).

---

## Build Roadmap

| Part | Topic                        | Status      | File |
|------|------------------------------|-------------|------|
| 1    | Swing Highs & Lows           | Done        | `smc/part1-swing-highs-lows/smc-part1-swings.pine` |
| 2    | Market Structure (BOS/CHoCH) | Done        | `smc/part2-market-structure/smc-part2-market-structure.pine` |
| 3    | Fair Value Gaps (FVG)        | Done        | `smc/part3-fvg/smc-part3-fvg.pine` |
| 4    | Order Blocks (OB)            | Not Started | —    |
| 5    | Liquidity Levels             | Not Started | —    |
| 6    | Premium / Discount Zones     | Not Started | —    |
| 7    | Signal Logic & Alerts        | Not Started | —    |

---

## Part 1 — Swing Highs & Lows

**Goal:** Detect confirmed swing highs and lows using the Golden Rule (3-candle confirmation). Foundation for all other parts.

**Sub-parts:**

| # | Task                                        | Status      |
|---|---------------------------------------------|-------------|
| 1.1 | Detect swing high (pivot high logic)      | Done        |
| 1.2 | Detect swing low (pivot low logic)        | Done        |
| 1.3 | Plot swing high/low labels on chart       | Done        |
| 1.4 | Store recent N swing highs/lows in arrays | Done        |

**Key concepts implemented:**
- `ta.pivotlow(low, 1, 1)` / `ta.pivothigh(high, 1, 1)` — middle of 3 candles is the extreme. Fires 1 bar late (at bar[0] for pivot at bar[1])
- **Golden Rule:** pivot is only confirmed after 3 candles in the opposite direction each close beyond the previous counted candle's high/low
- **Inside candles:** `high < high[1] and low > low[1]` — skipped entirely (not counted, do not reset count)
- **Pivot candle as candle 1:** if the pivot candle itself is bullish (for low) or bearish (for high), it counts as candle 1
- **Lock:** once a pendingLow/pendingHigh is set, new pivots at same level or less extreme are ignored
- **Replace:** if a new more extreme pivot forms before confirmation (higher high / lower low), it replaces the pending pivot and resets the count. Old counting labels are deleted from chart.
- **Invalidation:** if price closes below pendingLow or above pendingHigh, the pending state is cleared entirely
- **Arrays:** last 5 confirmed swing highs and lows stored in `swingHighPrices[]`, `swingHighBars[]`, `swingLowPrices[]`, `swingLowBars[]` (index 0 = most recent) for use in Part 2

**File:** `smc/part1-swing-highs-lows/smc-part1-swings.pine`

---

## Part 2 — Market Structure (BOS & CHoCH)

**Goal:** Track trend direction by detecting when price breaks previous swing highs/lows.

**Sub-parts:**

| # | Task                                              | Status      |
|---|---------------------------------------------------|-------------|
| 2.1 | Track last swing high and last swing low        | Done (via Part 1 arrays) |
| 2.2 | Detect BOS (Break of Structure) — trend continuation | Done (chart verified) |
| 2.3 | Detect CHoCH (Change of Character) — reversal   | Done (chart verified) |
| 2.4 | Maintain current trend state (bullish/bearish)  | Done (chart verified) |
| 2.5 | Draw BOS/CHoCH lines and labels on chart        | Done (chart verified) |

**Agreed approach (ready to code):**

1. **Initialize from history** — Part 1 populates swing arrays from all historical bars. Pick `swingHighPrices[0]` and `swingLowPrices[0]` as starting refHigh and refLow. Trend = "up" if H bar more recent, "down" if L bar more recent.

2. **Draw lines from confirmed H/L** — when Part 1 confirms a new H, draw horizontal line extending right. Same for L. These are the BOS/CHoCH detection levels.

3. **BOS/CHoCH detection** — `close > refHigh` triggers BOS (if trend up) or CHoCH (if trend down). `close < refLow` triggers BOS (if trend down) or CHoCH (if trend up). Line ends at event bar, label placed there.

4. **After BOS/CHoCH — new levels:**
   - One side is **immediate**: scan lowest/highest point from confirmed H/L bar to event bar (range lookup)
   - Other side **waits**: next Golden Rule confirmed H or L from Part 1
   - Once BOTH are set → completely frozen until next BOS/CHoCH. No updates.

5. **Key rule**: new High and Low determined after BOS/CHoCH do NOT change until the next BOS or CHoCH fires on those levels.

**Notes:**
- BOS: price closes beyond a swing point in the direction of current trend (continuation)
- CHoCH: price closes beyond a swing point against current trend (reversal)
- After CHoCH: if price breaks that level again → it is CHoCH again (not BOS)
- New Low after bullish BOS = lowest candle between confirmed H bar and BOS bar
- New High after bearish BOS = highest candle between confirmed L bar and BOS bar
- Depends on Part 1 arrays (swingHighPrices, swingHighBars, swingLowPrices, swingLowBars)

**File:** `smc/part2-market-structure/smc-part2-market-structure.pine`

---

## Part 3 — Fair Value Gaps (FVG)

**Goal:** Identify 3-candle imbalance zones where price moved too fast, leaving a gap.

**Sub-parts:**

| # | Task                                              | Status      |
|---|---------------------------------------------------|-------------|
| 3.1 | Detect bullish FVG (candle[2].high < candle[0].low) | Done (needs chart test) |
| 3.2 | Detect bearish FVG (candle[2].low > candle[0].high) | Done (needs chart test) |
| 3.3 | Draw FVG boxes on chart                          | Done (needs chart test) |
| 3.4 | Mark FVG as mitigated when price fills the gap   | Done (needs chart test) |
| 3.5 | Option to hide mitigated FVGs                    | Done (needs chart test) |

**Notes:**
- Standalone — does not depend on swing points
- Limit number of active FVG boxes to avoid clutter (max N setting)

**File:** `smc/part3-fvg/smc-part3-fvg.pine` *(pending)*

---

## Part 4 — Order Blocks (OB)

**Goal:** Identify the last opposing candle before an impulsive move that broke structure.

**Sub-parts:**

| # | Task                                              | Status      |
|---|---------------------------------------------------|-------------|
| 4.1 | Find last bearish candle before a bullish BOS (bullish OB) | Not Started |
| 4.2 | Find last bullish candle before a bearish BOS (bearish OB) | Not Started |
| 4.3 | Draw OB zones as boxes on chart                  | Not Started |
| 4.4 | Mark OB as mitigated when price returns to zone  | Not Started |
| 4.5 | Option to hide mitigated OBs                     | Not Started |

**Notes:**
- Depends on Part 2 (BOS events trigger OB identification)

**File:** `smc/part4-order-blocks/smc-part4-order-blocks.pine` *(pending)*

---

## Part 5 — Liquidity Levels

**Goal:** Identify resting liquidity above swing highs (buy-side) and below swing lows (sell-side).

**Sub-parts:**

| # | Task                                              | Status      |
|---|---------------------------------------------------|-------------|
| 5.1 | Track significant swing highs as buy-side liquidity | Not Started |
| 5.2 | Track significant swing lows as sell-side liquidity | Not Started |
| 5.3 | Draw liquidity lines on chart                    | Not Started |
| 5.4 | Mark liquidity as swept when price wicks through and rejects | Not Started |

**Notes:**
- Depends on Part 1

**File:** `smc/part5-liquidity/smc-part5-liquidity.pine` *(pending)*

---

## Part 6 — Premium / Discount Zones

**Goal:** Define the current price range and split into premium (sell) and discount (buy) zones.

**Sub-parts:**

| # | Task                                              | Status      |
|---|---------------------------------------------------|-------------|
| 6.1 | Define current range (last CHoCH low to last BOS high, or vice versa) | Not Started |
| 6.2 | Calculate equilibrium (50% of range)             | Not Started |
| 6.3 | Shade premium zone (above 50%)                   | Not Started |
| 6.4 | Shade discount zone (below 50%)                  | Not Started |
| 6.5 | Optional: mark 0.705 optimal entry zone          | Not Started |

**Notes:**
- Depends on Part 2

**File:** `smc/part6-premium-discount/smc-part6-premium-discount.pine` *(pending)*

---

## Part 7 — Signal Logic & Alerts

**Goal:** Combine all parts into entry/exit signals with broker-compatible webhook alerts.

**Sub-parts:**

| # | Task                                              | Status      |
|---|---------------------------------------------------|-------------|
| 7.1 | Long condition: bullish structure + discount + OB/FVG confluence | Not Started |
| 7.2 | Short condition: bearish structure + premium + OB/FVG confluence | Not Started |
| 7.3 | Exit condition: structure break against trade     | Not Started |
| 7.4 | Entry alert ("SMC: BUY" / "SMC: SELL")           | Not Started |
| 7.5 | Exit alert ("SMC: CLOSE LONG" / "SMC: CLOSE SHORT") | Not Started |

**Notes:**
- Two alerts only (entry + exit) — broker algo is built on this structure
- Fixed SL: 15 pts Nifty, 35 pts Sensex (handled on broker side, not in Pine Script)
- Alerts set to "Once Per Bar" (not bar close) for intrabar triggers

**File:** `smc/part7-signals/smc-part7-signals.pine` *(pending)*

---

## Final Combined Script

Once all parts are tested individually, merge into a single indicator script.

**File:** `smc/smc-indicator.pine` *(pending)*

---

## Change Log

| Date       | Part | Change |
|------------|------|--------|
| 2026-06-14 | —    | Document created, roadmap defined |
| 2026-06-14 | 1.1–1.3 | Swing H/L detection complete — 3-candle Golden Rule, lock + invalidation, label cleanup on pivot replacement |
| 2026-06-14 | 1.4     | Confirmed swing H/L stored in arrays (last 5, index 0 = most recent) for Part 2 consumption |
| 2026-06-15 | 2.2–2.5 | Market Structure implemented: init-from-history trend, BOS/CHoCH detection, immediate side via range scan (lowest/highest from ref bar → event bar), waiting side via next confirmed swing, ref-level lines that end at event bar. Pending chart test on Nifty/Sensex. |
| 2026-06-15 | 2.5     | Display polish: BOS/CHoCH + H/L as plain text (no boxes), BOS/CHoCH centered on broken line, dotted lines width=2, raw swing labels toggle (default off), structure H/L toggle (default on). |
| 2026-06-15 | 2.x bug | Fixed: superseded structure H/L were deleted on each new event, so only the latest survived (older BOS showed no H). Now old levels are frozen + kept as history instead of deleted. |
| 2026-06-15 | 2.5     | Final display model (chart-verified): only TWO line types — BOS (blue) and CHoCH (green up / red down). Lines drawn only at a break, origin bar → break bar (no extending reference lines). H/L plain text markers at structure levels. Trend background (green up / red down, transp 92). Part 2 DONE. |
| 2026-06-15 | 3.1–3.3 | FVG detection (bullish high[2]<low[0], bearish low[2]>high[0]) + box drawing with max-box cap and right-extend. Standalone. Pending chart test. Mitigation (3.4/3.5) next. |
| 2026-06-15 | 3.1–3.5 | After comparing with LuxAlgo SMC_refrence.pine: ported 3 ideas onto our simpler base — (1) middle-candle confirmation close[1]>high[2] / close[1]<low[2]; (2) mitigation via first-touch into gap (grey-out, or hide via toggle); (3) optional displacement-strength threshold (auto avg body%). Skipped reference's 2-box gradient (cosmetic) and HTF/lookahead (repaint risk). Uses `type fvg` to track each gap. Pending chart test. OB filter deferred to Part 4. |

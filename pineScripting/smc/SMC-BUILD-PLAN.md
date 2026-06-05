# Smart Money Concepts — Phased Build Plan

Reference implementation: `refrence.pine` (LuxAlgo SMC indicator)

---

## Phase 1 — Swing Detection

**Goal:** Identify swing highs and swing lows, determine leg direction, and label each pivot as HH / HL / LH / LL.

**Concepts:**
- A *leg* is the current direction of price movement (bullish or bearish).
- A *pivot high* forms when price makes a new highest high after a bullish leg.
- A *pivot low* forms when price makes a new lowest low after a bearish leg.
- Label each new pivot relative to the previous pivot of the same type:
  - Higher High (HH), Lower High (LH)
  - Higher Low (HL), Lower Low (LL)

**Inputs:**
- `Swing Length` (int) — lookback period for pivot detection (default 50)

**Outputs:**
- Labels on chart: HH, HL, LH, LL at each detected pivot

**Key functions to build:**
- `leg(size)` — returns current leg direction (0 = bearish, 1 = bullish)
- `startOfNewLeg(leg)` — true when leg direction changes
- `getCurrentStructure(size)` — updates pivot high/low state and draws labels

**Reference lines in `refrence.pine`:**
- `leg()` → lines 337–346
- `startOfNewLeg/startOfBullishLeg/startOfBearishLeg` → lines 351–361
- `getCurrentStructure()` → lines 409–457
- `drawLabel()` → lines 370–377

**File:** `phase1-swing-detection.pine`

---

## Phase 2 — Market Structure (BOS & CHoCH)

**Goal:** Detect when price breaks a prior swing level and classify it as BOS or CHoCH. Track overall trend bias.

**Concepts:**
- *BOS (Break of Structure)* — price breaks a swing high/low in the direction of the current trend (trend continuation).
- *CHoCH (Change of Character)* — price breaks a swing high/low against the current trend (potential reversal).
- Two levels of structure:
  - *Swing structure* — based on major pivot highs/lows (longer lookback)
  - *Internal structure* — based on shorter-term pivots (faster signals)

**Inputs:**
- All Phase 1 inputs
- `Show Swing Structure` (bool)
- `Show Internal Structure` (bool)
- `Bullish/Bearish Structure filter` — All / BOS only / CHoCH only

**Outputs:**
- Horizontal lines from pivot to breakout bar
- Labels: BOS or CHoCH at midpoint of each line

**Key functions to build:**
- `displayStructure(internal)` — detects crossover/crossunder of pivot level, tags BOS vs CHoCH, updates trend bias
- `drawStructure(pivot, tag, color, ...)` — draws line and label

**Reference lines in `refrence.pine`:**
- `displayStructure()` → lines 551–612
- `drawStructure()` → lines 467–476

**Depends on:** Phase 1

**File:** `phase2-market-structure.pine`

---

## Phase 3 — Order Blocks

**Goal:** Identify the last up/down candle before a structural break and mark it as an Order Block zone. Remove it when price mitigates through it.

**Concepts:**
- A *bullish OB* is the last bearish candle before a bullish BOS/CHoCH.
- A *bearish OB* is the last bullish candle before a bearish BOS/CHoCH.
- *Mitigation* — OB is invalidated when price closes through it (or crosses high/low depending on setting).
- High-volatility bars are filtered using ATR or cumulative mean range.

**Inputs:**
- `Show Internal OBs` (bool), count (int, max 20)
- `Show Swing OBs` (bool), count (int, max 20)
- `OB Filter` — ATR or Cumulative Mean Range
- `OB Mitigation` — Close or High/Low

**Outputs:**
- Rectangular boxes on chart for active OBs (extend to the right until mitigated)

**Key functions to build:**
- `storeOrderBlock(pivot, internal, bias)` — finds the correct candle and stores OB data
- `deleteOrderBlocks(internal)` — checks mitigation and removes invalidated OBs
- `drawOrderBlocks(internal)` — renders boxes from stored OB data

**Reference lines in `refrence.pine`:**
- `storeOrdeBlock()` → lines 507–525
- `deleteOrderBlocks()` → lines 481–500
- `drawOrderBlocks()` → lines 530–546

**Depends on:** Phase 2

**File:** `phase3-order-blocks.pine`

---

## Phase 4 — Fair Value Gaps

**Goal:** Detect three-candle imbalance patterns (FVG) and draw them as zones. Remove when price fills the gap.

**Concepts:**
- A *bullish FVG* exists when `low[0] > high[2]` (gap between candle 0 and candle 2, with candle 1 in between).
- A *bearish FVG* exists when `high[0] < low[2]`.
- Optional auto-threshold filters out insignificant gaps.
- Supports multi-timeframe FVG detection.

**Inputs:**
- `Show FVGs` (bool)
- `Auto Threshold` (bool)
- `FVG Timeframe` (timeframe)
- `Extend FVG` (int) — how many bars to extend the box

**Outputs:**
- Two stacked boxes per FVG (top half and bottom half), colored bullish/bearish

**Key functions to build:**
- `drawFairValueGaps()` — detects and stores FVG data, draws boxes
- `deleteFairValueGaps()` — removes FVGs that price has filled

**Reference lines in `refrence.pine`:**
- `drawFairValueGaps()` → lines 634–649
- `deleteFairValueGaps()` → lines 625–630
- `fairValueGapBox()` → line 621

**Depends on:** Phase 1 (swing context optional; can stand alone)

**File:** `phase4-fair-value-gaps.pine`

---

## Phase 5 — Equal Highs & Lows

**Goal:** Detect when two swing highs or lows are nearly equal in price and mark them as EQH / EQL.

**Concepts:**
- Two highs are *equal* if their price difference is less than `threshold * ATR`.
- Drawn as a dotted horizontal line between the two pivot times.
- Label placed at midpoint between the two pivots.

**Inputs:**
- `Show EQH/EQL` (bool)
- `Bars Confirmation` (int) — lookback to confirm pivot
- `Threshold` (float, 0–0.5) — sensitivity

**Outputs:**
- Dotted lines and EQH / EQL labels between matching pivots

**Key functions to build:**
- `drawEqualHighLow(pivot, level, size, isHigh)` — draws line and label
- Logic inside `getCurrentStructure()` that checks equality before updating pivot

**Reference lines in `refrence.pine`:**
- `drawEqualHighLow()` → lines 384–402
- EQH/EQL detection inside `getCurrentStructure()` → lines 419–421, 440–442

**Depends on:** Phase 1

**File:** `phase5-equal-highs-lows.pine`

---

## Phase 6 — Premium / Discount Zones & MTF Levels

**Goal:** Draw premium/discount/equilibrium zones based on current swing range. Optionally show prior Daily/Weekly/Monthly highs and lows.

**Concepts:**
- *Premium zone* — top 5% of the swing range (above 95th percentile).
- *Discount zone* — bottom 5% of the swing range.
- *Equilibrium* — midpoint ±2.5% band.
- MTF levels pull prior period high/low from a higher timeframe using `request.security`.

**Inputs:**
- `Show Premium/Discount Zones` (bool)
- `Show Daily/Weekly/Monthly Levels` (bool each)
- Line style per level (Solid / Dashed / Dotted)

**Outputs:**
- Three zone boxes (premium, equilibrium, discount) anchored to last swing
- Horizontal lines for prior D/W/M high and low with labels

**Key functions to build:**
- `drawZone(...)` — draws label + box for a zone
- `drawPremiumDiscountZones()` — calls drawZone for all three zones
- `drawLevels(timeframe, ...)` — fetches and draws MTF high/low lines

**Reference lines in `refrence.pine`:**
- `drawZone()` → lines 744–751
- `drawPremiumDiscountZones()` → lines 755–761
- `drawLevels()` → lines 666–700

**Depends on:** Phase 2 (needs trailing swing extremes)

**File:** `phase6-zones-mtf-levels.pine`

---

## Phase 7 — Alerts & Final Polish

**Goal:** Wire all alert conditions, unify styling inputs (Colored vs Monochrome), add candle coloring based on trend bias, and merge all phases into a single production script.

**Features:**
- 16 `alertcondition()` calls covering all BOS, CHoCH, OB, FVG, EQH/EQL events
- Color candles by internal trend bias
- Style toggle: Colored vs Monochrome
- Historical vs Present mode (delete old drawings vs keep all)
- Final merged file with all phases combined

**Reference lines in `refrence.pine`:**
- Alerts → lines 827–847
- Style/mode inputs → lines 76–78
- Candle coloring → lines 766–768

**Depends on:** All phases

**File:** `phase7-final-smc.pine`

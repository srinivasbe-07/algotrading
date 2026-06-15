# SMC Part 3 — Fair Value Gaps (FVG)

Detects **3-candle imbalances** (gaps left by fast moves), draws them as boxes, and
tracks when price comes back to fill ("mitigate") them. **Standalone** — does not depend
on swings or market structure. OB-based filtering is deferred to Part 4.

**File:** `smc-part3-fvg.pine`

---

## 1. What is an FVG?

A Fair Value Gap is a price band the market moved through too quickly, leaving untraded
space. Looking at three consecutive candles — `[2]` oldest, `[1]` middle (the displacement),
`[0]` current:

```
Bullish FVG                         Bearish FVG
        ┌─┐ c3                       ┌─┐ c1
   ░░gap░░ ← low[0]  (top)           │ │
   ┌─┐                               └─┘ ← low[2]  (top)
   │ │ ← high[2] (bottom)       ░░gap░░
   c1                                ┌─┐ ← high[0] (bottom)
                                     c3
```

Price often returns to "rebalance" the gap, so FVGs act as potential entry zones.

---

## 2. Detection

```pine
bullFVG = high[2] < low[0] and close[1] > high[2] and (not useThresh or  deltaPct > threshold)
bearFVG = low[2]  > high[0] and close[1] < low[2]  and (not useThresh or -deltaPct > threshold)
```

| Check | Why |
|-------|-----|
| `high[2] < low[0]` (bull) / `low[2] > high[0]` (bear) | The core 3-candle gap: candle 1 and candle 3 don't overlap. |
| `close[1] > high[2]` (bull) / `close[1] < low[2]` (bear) | **Middle-candle confirmation** — the displacement candle actually *closed through* the gap. Removes false micro-gaps. |
| `deltaPct ⋛ threshold` | **Displacement-strength filter** (optional, see §4). |

The **gap band** stored per FVG:
- Bullish → `top = low[0]`, `bottom = high[2]`
- Bearish → `top = low[2]`, `bottom = high[0]`

---

## 3. Mitigation (first touch)

Each FVG is tracked in a `type fvg { top, bottom, bias, box, mitigated }` kept in a
persistent `var` array. Every bar, active FVGs:

1. **Extend** their box right edge to follow price (`bar_index + extendBars`).
2. Check for **first touch** back into the zone:
   - Bullish → `low < top`
   - Bearish → `high > bottom`
3. On touch → marked **mitigated**:
   - If **Hide mitigated FVGs** is on → the box is deleted.
   - Else → the box is **greyed out** and **frozen** at the fill bar.

> Mitigation here = **first touch** (the common SMC meaning), not a full far-side breach.
> Easy to switch to far-side if preferred.

The chart is also capped at **`Max FVG boxes`** — when exceeded, the oldest FVG is removed
(`trim()`), so history never piles up.

---

## 4. Inputs

| Input | Default | Effect |
|-------|---------|--------|
| `Show bullish FVG` | on | Draw bullish gaps. |
| `Show bearish FVG` | on | Draw bearish gaps. |
| `Bullish FVG` | green | Bullish box color. |
| `Bearish FVG` | red | Bearish box color. |
| `Mitigated FVG` | grey | Color a gap turns to once filled (when not hidden). |
| `Max FVG boxes` | 20 | Cap on visible gaps; oldest deleted past this. |
| `Box extend (bars)` | 10 | How far the box extends right of the current bar. |
| `Hide mitigated FVGs` | off | Delete filled gaps instead of greying them. |
| `Filter by displacement strength` | off | Keep only gaps whose middle candle body exceeds an auto threshold (`cum avg body% × 2`). |

---

## 5. Comparison vs LuxAlgo reference (`SMC_refrence.pine`)

This part was built by comparing against the LuxAlgo SMC reference and porting the proven
ideas onto a simpler, owned base.

| Aspect | Reference | This file |
|--------|-----------|-----------|
| Core gap rule | `low[0] > high[2]` | **Same** |
| Middle-candle confirmation | yes | **Adopted** (`close[1]`) |
| Significance threshold | yes (auto) | **Adopted** as optional toggle |
| Mitigation | far-side breach | first-touch (greys or hides) |
| Box count cap | none | **Added** (`maxBoxes`) |
| 2-box gradient visual | yes | **Skipped** (cosmetic) |
| HTF via `request.security`/`lookahead` | yes | **Skipped** (repaint risk; single-TF use) |

Net: the reference's signal quality, with cleaner, alert-ready, extensible code.

---

## 6. Known notes / TODO

- **Repaint:** detection uses `low[0]`/`high[0]` (current bar), so a forming gap can
  appear/disappear intrabar; it finalizes on candle 3's close. Historical bars are fixed.
- **OB filter (Part 4):** the "FVG must sit next to an order block" rule will be added when
  Order Blocks are built, then wired into this detection.

---

## 7. Status

Parts 3.1–3.5 complete. Pending final chart confirmation on NIFTY/SENSEX intraday.
See `../SMC-BUILD-PROGRESS.md` for the roadmap and change log.

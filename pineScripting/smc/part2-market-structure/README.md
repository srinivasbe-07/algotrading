# SMC Part 2 — Market Structure (BOS / CHoCH)

Detects market structure shifts on top of Part 1's confirmed swing highs/lows and
labels them as **BOS** (Break of Structure) or **CHoCH** (Change of Character),
while maintaining the current trend state.

**File:** `smc-part2-market-structure.pine`
**Depends on:** Part 1 swing arrays (`swingHighPrices/Bars`, `swingLowPrices/Bars`,
`swingHighJustConfirmed`, `swingLowJustConfirmed`).

---

## 1. Concepts

| Term | Meaning |
|------|---------|
| **refHigh / refLow** | The two structure levels currently being watched. A close beyond one is an event. |
| **BOS** | Price closes beyond a level **in the direction of the trend** → trend *continuation*. |
| **CHoCH** | Price closes beyond a level **against the trend** → trend *reversal*. |
| **trend** | `"up"` or `"down"`. Flips on the events above. |

- In an **uptrend**: close **above refHigh** = **BOS** (up). Close **below refLow** = **CHoCH** (down, reversal).
- In a **downtrend**: close **below refLow** = **BOS** (down). Close **above refHigh** = **CHoCH** (up, reversal).

---

## 2. How levels are set

### Initialization (from history)
Once Part 1 has at least one confirmed high **and** one confirmed low:
- `refHigh / refLow` = the most recent confirmed swing high / low.
- `trend` = `"up"` if the latest high is more recent than the latest low, else `"down"`.

No price break is needed to start — the most recent swing tells us the current structure.

### After a BOS/CHoCH — the new range
When an event fires, a **new range** (the levels the *next* event will watch) is formed:

**Bullish break (close above refHigh):**
- **New LOW (immediate):** the *lowest low* scanned from the broken-high's bar → the break bar.
  This is the protected low of the leg that produced the break (set instantly).
- **New HIGH (waits):** the next swing high confirmed by Part 1's **3-candle Golden Rule**
  *after* the event bar. Until then `refHigh = na`.

**Bearish break (close below refLow):** mirror image —
- **New HIGH (immediate):** highest high scanned from the broken-low's bar → break bar.
- **New LOW (waits):** next 3-candle confirmed swing low after the event.

> Levels never "drift": between events there is no running min/max. The immediate side is
> fixed by a one-time range scan, the other side is fixed when the next swing confirms.

---

## 3. Visual model — only two line *types*

A dotted line is drawn **only at a break**, never as a persistent reference rail. There are
exactly two kinds of line:

| Line | Color | When |
|------|-------|------|
| **BOS** | blue (`colBOS`) | break continues the trend |
| **CHoCH** | green up (`colUp`) / red down (`colDown`) | break reverses the trend |

Each line runs from the **broken level's origin bar → the break bar**, with the `BOS`/`CHoCH`
text **centered** on it.

Other on-chart elements:
- **`H` / `L`** — plain-text markers at each structure high / low (no box, no line). Kept as
  history so every past structure point stays visible.
- **Trend background** — faint green while trend is up, faint red while down.

---

## 4. Inputs

| Input | Default | Effect |
|-------|---------|--------|
| `Show raw swing H/L labels` | off | Part 1's per-swing pivot markers (debug). |
| `BOS label` | blue | Color of BOS line + text. |
| `Bullish level / CHoCH` | green | Up color (CHoCH up, H markers, up background). |
| `Bearish level / CHoCH` | red | Down color (CHoCH down, L markers, down background). |
| `Show structure H/L labels` | on | The `H`/`L` structure markers. |
| `Trend background (green up / red down)` | on | Background tint by trend. |

---

## 5. Key design decisions & a fixed bug

- **Two line types only.** Earlier drafts drew extending reference lines for every level,
  which cluttered the chart. Final model: lines appear *only* at BOS/CHoCH breaks.
- **Plain text, no boxes.** BOS/CHoCH and H/L are `label.style_none` with transparent
  backgrounds; BOS/CHoCH is centered on its line.
- **Bug fixed — superseded levels were deleted.** Originally, each new event called
  `line.delete` / `label.delete` on the previous level before drawing the new one, so in a
  trend with several BOS in a row only the *latest* H (or L) survived — older BOS appeared to
  have no H. Fix: superseded levels are **kept as history**, never deleted.

---

## 6. Known edge cases / TODO

- **Long left-side legs:** the immediate-side scan loop reads `high[i]` / `low[i]` back to the
  level's origin bar. On a very long leg this can exceed Pine's historical buffer. If a
  *"historical offset beyond buffer"* error appears, add `max_bars_back = 5000` to the
  `indicator(...)` call.
- **Bullish BOS with no confirmed high yet:** if price reverses (CHoCH down) before the waiting
  high 3-candle-confirms, that BOS's high is provided by the reversal's immediate scan instead.
  This is correct per the "3-candle confirmed high" rule, by design.

---

## 7. Status

Part 2 complete and chart-verified on NIFTY 15m (2026-06-15). Next: **Part 3 — Fair Value Gaps**.
See `../SMC-BUILD-PROGRESS.md` for the full roadmap and change log.

Here it is:

---

# Universal Indicator Logic Flow Template

---

## BLOCK 0 — Identity & Category
```
Name:
Type: [ Structure | Momentum | Volume | Oscillator | Pattern | Session | MTF | ML/Adaptive | Hybrid ]
Platform built on:
Timeframe intended for:
Repaints: [ Yes | No | On certain conditions ]
Fires on: [ Bar close | Real-time tick | Both ]
```

---

## BLOCK 1 — Input Layer
```
[INPUT NODE]
├── Price fields used: [ O | H | L | C | HL2 | HLC3 | OHLC4 ]
├── Volume used: [ Yes | No ] → type: [ Tick | Real ]
├── External inputs: [ ATR | Other indicator | Session time | Custom ]
├── Timeframes referenced: [ Current | HTF: ___ | LTF: ___ ]
├── HTF data type: [ Confirmed closed bar | Live real-time ]
└── Lookback period: ___ bars
```

---

## BLOCK 2 — Pre-Processing / Normalization
```
[PRE-PROCESS NODE]
├── Smoothing applied: [ None | SMA | EMA | WMA | Custom: ___ ]
├── Smoothing length: ___
├── Normalization: [ None | 0-100 | -1 to 1 | Z-score | Custom: ___ ]
├── Gap/missing data handling: ___
└── Output of this block: ___
```

---

## BLOCK 3 — Core Calculation
```
[CALCULATION NODE]
├── Formula / math: ___
├── Variables used: ___
├── Numeric constants / hardcoded values: ___
├── Dynamic values (change per bar): ___
├── Rolling window or cumulative: ___
└── Output of this block: [ Numerical value | Boolean | Level/Price | Zone ]
```

---

## BLOCK 4 — State Memory
```
[MEMORY NODE]
├── Does it store previous values: [ Yes | No ]
├── What is stored: [ Last swing high | Last signal bar | Previous level | Custom: ___ ]
├── How many bars back: ___
├── Reset condition: ___
└── Output of this block: ___
```

---

## BLOCK 5 — Detection / Trigger Gate
```
[DETECTION NODE]
├── Primary condition: ___
├── Secondary condition (if any): ___
├── Confirmation required: [ Yes | No ] → what confirms: ___
├── Trigger type: [ AND | OR | Sequential ]
├── Sequence order (if sequential): Step 1 → Step 2 → Step 3
└── Output: [ Signal fired | No signal ]
```

---

## BLOCK 6 — Filter / Suppression Gate
```
[FILTER NODE]
├── ATR filter: [ Yes | No ] → rule: ___
├── Volume filter: [ Yes | No ] → rule: ___
├── Session filter: [ Yes | No ] → session: ___ timezone: ___
├── Minimum bar requirement: ___
├── Volatility condition: ___
├── Any other suppression rule: ___
└── Output: [ Signal passes | Signal blocked ]
```

---

## BLOCK 7 — Invalidation Logic
```
[INVALIDATION NODE]
├── What cancels a pending signal: ___
├── What cancels an active signal: ___
├── Price level that voids setup: ___
├── Bar count expiry: ___
├── Opposite signal override: [ Yes | No ]
└── Output: [ Signal alive | Signal invalidated ]
```

---

## BLOCK 8 — Retest Logic
```
[RETEST NODE]
├── Retest required before entry: [ Yes | No ]
├── What qualifies as a retest: ___
├── Tolerance / margin allowed: ___
├── Max bars to wait for retest: ___
├── Retest invalidation: ___
└── Output: [ Retest confirmed | Retest failed | Not applicable ]
```

---

## BLOCK 9 — Output Declaration
```
[OUTPUT NODE]
├── Signal type: [ Long | Short | Neutral | Level | Zone | Label ]
├── Output value: [ Boolean | Price value | Range | Score ]
├── Output passed to: [ Chart render | Another indicator | Alert | All ]
└── Output label/name: ___
```

---

## BLOCK 10 — Visual Render Layer
```
[RENDER NODE]
├── What is plotted: [ Line | Shape | Label | Zone/Box | Arrow | Bar color ]
├── Color (bullish): ___
├── Color (bearish): ___
├── Color (neutral): ___
├── Line style: [ Solid | Dashed | Dotted ]
├── Line width: ___
├── Label text: ___
├── Label position: [ Above bar | Below bar | On level ]
├── Zone boundary (if applicable): Upper = ___ Lower = ___
└── Render condition: [ Always | Only on signal | Only on retest ]
```

---

## BLOCK 11 — Merge Wiring (fill when combining indicators)
```
[MERGE CIRCUIT]

Indicator A: ___
└── Output type: ___

Indicator B: ___
└── Output type: ___

Indicator C (if any): ___
└── Output type: ___

Gate logic between them:
├── Condition: [ AND | OR | Sequential ]
├── Sequence (if sequential): A fires → B confirms → C filters
├── Priority/hierarchy: ___ dominates when conflict
├── Conflict resolution rule: ___
├── Timing sync: [ All bar close | Mixed — specify: ___ ]
└── Final combined output: ___
```

---

## BLOCK 12 — Edge Cases & Special Conditions
```
[EDGE CASE NODE]
├── Behavior on first N bars (no lookback yet): ___
├── Behavior on gaps / missing bars: ___
├── Behavior on low liquidity / flat market: ___
├── Behavior on news / high volatility spikes: ___
├── Holiday / weekend session handling: ___
└── Any known failure condition: ___
```

---

**How to use this**

- Fill one template per indicator
- Block 11 only filled when merging
- Every empty field = a gap that will break your rebuild
- If a field is truly not applicable write `N/A` — never leave blank by assumption

---

This is now a complete, universal, circuit-style framework that works for any indicator type and any merge scenario.


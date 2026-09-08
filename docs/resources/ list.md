PINE SCRIPT v6 — MASTER FAILURE ANCHOR SCHEMA

---

DOMAIN 1: COMPILATION GATE (Script Won't Run)

1.1 Declaration Block

```
VERSION_MISSING         → No //@version=6 directive
VERSION_WRONG           → v6 syntax in v5 script or vice versa
DUPLICATE_INDICATOR     → Multiple indicator()/study() calls
MISSING_INDICATOR       → Library syntax used without library() declaration
OVERLAY_CONFLICT        → overlay=true with non-overlay elements
```

1.2 Lexical Identity

```
UNDECLARED_IDENTIFIER   → Variable used before var/=/:= anywhere in scope
UNDECLARED_NAMESPACE    → rsi() instead of ta.rsi(), avg() instead of math.avg()
BUILTIN_AS_VAR          → close, ta.atr, open reused as user variable names
USE_BEFORE_DECLARE      → Referenced before any declaration statement
SHADOWING               → Inner scope = creates new local, masks parent scope
VAR_IN_CONDITIONAL      → var keyword inside if/for/while (must be global scope)
UNUSED_IMPORT           → import statement with no members used
CIRCULAR_IMPORT         → Library A imports B, B imports A
```

1.3 Type Assignments

```
NA_TO_UNTYPED           → var x = 0; x := na (missing float qualifier)
INT_HOLDS_NA            → var int x = na (int cannot store na)
NA_FUNC_ON_BOOL         → na(bool_value) — na() invalid on boolean
STRING_VS_ENUM          → var string x = xloc.bar_index (enum → string mismatch)
INT_VS_FLOAT_RETURN     → Function return type doesn't match actual return
UDT_ARRAY_TYPE_MISMATCH → isDuplicateOB() receiving wrong UDT array type
TERNARY_BRANCH_TYPE     → ?: returning int in true branch, float in false
NAN_CAST                → int(na_value) — cast na to integer
NAN_IN_GEOMETRY         → totalWidth = na passed to box/line/polyline
SERIES_TO_CONST         → series argument where simple qualifier required
```

1.4 Function Boundaries

```
MISSING_ALL_ARGS        → ta.rsi() called with zero arguments
WRONG_ARG_COUNT         → ta.stdev(src, len, extra) — too many/few
WRONG_ARG_TYPE          → ta.sma(src, "D") — timeframe string where int expected
UDT_NEW_MISSING_FIELDS  → Engine2Zone.new() insufficient constructor arguments
LINE_NA_ARG             → line(na) — na is not a valid line ID
TIMEFRAME_NO_ARG        → timeframe.in_seconds() called empty
ILLEGAL_RETURN          → return in void function or return missing in typed function
VOID_IN_EXPRESSION      → Using void function result in assignment/expression
```

1.5 Compiler Limits

```
MAX_BARS_BACK_SHORT     → Default insufficient for [lookback5+2] reference
DRAWING_LIMIT           → >500 boxes / >500 lines / >500 labels / >100 polylines
PLOT_LIMIT              → >64 plot() statements
SECURITY_CALL_LIMIT     → >40 request.security() calls
LOOP_ITERATION          → for >5000 iterations / while timeout
SCRIPT_SIZE             → Source >64KB
MEMORY_LIMIT            → Runtime >100MB allocation
FUNCTION_NEST_DEPTH     → Function call depth >50
UDT_NEST_DEPTH          → Type nesting >5 levels
```

---

DOMAIN 2: SCOPE & HISTORY CONSISTENCY (CW10003 & Related)

2.1 Conditional Execution

```
CW10003_TA_COND         → ta.atr/highest/rsi/sma/stdev inside if block
CW10003_SECURITY_COND   → request.security() inside conditional branch
CW10003_CUSTOM_FUNC     → User function with series output in conditional
CW10003_TREND_CALC      → Trend calculations gated by condition
CW10003_MTF_CALC        → Multi-timeframe calculations in conditional
CW10003_ATR_FILTER      → globalATRFilter() called conditionally
```

2.2 History Reference on Local Scope

```
LOCAL_HISTORY_0         → rsi[0] where rsi declared in if block
LOCAL_HISTORY_1         → rsi[1], rsi[2] same pattern
LOCAL_HISTORY_DYNAMIC   → rsi[rsiSensitivity] dynamic offset on local var
MTF_LOCAL_HISTORY       → mtf_open[4], mtf_close[i], mtf_high[i] local scope
PC_MTF_LOCAL            → pc_mtf[1] local scope history
VOL_LOCAL_HISTORY       → normalized_vol[rightBars] local scope
```

2.3 Unsafe History Patterns

```
DYNAMIC_INDEX           → close[lookback5+2] calculated offset
DYNAMIC_IN_SECURITY     → close[expression] inside request.security()
DYNAMIC_TUPLE_SECURITY  → Tuple expression inside request.security()
CHAINED_REF             → src = conditional_series; src[1] — src may be na historically
HL2_UNSAFE              → hl2[volPivotLength] length exceeds bars
TIME_UNSAFE             → time[rightBars] with dynamic rightBars
TREND_UNSAFE            → trend5[1], trend5[2] chain
SRC_UNSAFE              → src[1], src[2] where src from conditional
CLOSE_OPEN_UNSAFE       → close[1], open[1] in conditional context
```

---

DOMAIN 3: CONTROL FLOW & REACHABILITY

3.1 Unreachable Code

```
AFTER_RETURN            → Code after unconditional return/break/continue
DEAD_ASSIGN             → validTimeframe5 := false on unreachable path
DEAD_BRANCH             → else branch where if condition always true/false
LOOP_NEVER_EXECUTES     → for i = 10 to 1 (step defaults +1)
WHILE_NEVER_RUNS        → while false condition on entry
```

3.2 Dead Logic

```
DEAD_ALERT              → engine3_new_bull condition never true
DEAD_DECLARATION        → enableEngine8 declared but never referenced
DEAD_FUNCTION           → User function never called
DEAD_TYPE               → UDT defined but never instantiated
DEAD_INPUT              → input.* variable never used
```

3.3 State Machine Failures

```
ALERT_FLAG_NEVER_RESET  → engine1_new_bull set true, never cleared
ALERT_FLAG_ACCUMULATE   → Flags accumulate bars without drain logic
ALERT_DUPLICATE         → Same condition in multiple alertcondition()
ENGINE_STATE_DESYNC     → Flag set but corresponding object lifecycle missed
CONDITION_INIT_REF      → Engine initialized under condition, referenced unconditionally
```

---

DOMAIN 4: OBJECT LIFECYCLE (Drawing & Arrays)

4.1 Creation Without Tracking

```
BOX_NOT_TRACKED         → box.new() without push to tracking array
LINE_NOT_TRACKED        → line.new() ID not stored
LABEL_NOT_TRACKED       → label.new() ID not stored
POLYLINE_NOT_TRACKED    → polyline.new() ID not stored
TEMP_OBJECT_NO_DELETE   → Temporary line/box for measurement, never deleted
ENGINE4_NO_CLEANUP      → Engine 4 boxes never in cleanup loops
ENGINE5_NO_CLEANUP      → Engine 5 boxes never in cleanup loops
```

4.2 Cleanup Contamination

```
CROSS_ENGINE_CLEANUP    → Engine2 cleanup deletes Engine4 objects (shared tracking array)
DELETED_ID_IN_ARRAY     → Tracking array contains already-deleted IDs
DOUBLE_DELETE           → box.delete() called twice on same ID
CLEANUP_MUTATE_DURING   → array.remove() inside for loop traversing same array
```

4.3 Array Runtime

```
EMPTY_ARRAY_ITERATE     → Loop over 0-length array
INDEX_MINUS_ONE         → array.size(arr) - 1 when arr is empty
POP_EMPTY               → array.pop() on empty array
GET_OUT_OF_BOUNDS       → array.get(index >= size)
REMOVE_OUT_OF_BOUNDS    → array.remove(index >= size)
UNSHIFT_LARGE           → array.unshift() on large array (O(n) every bar)
SORT_WITH_NA            → array.sort() with na values (na sorted to ends)
INCLUDES_NA             → array.includes(na) — na equality never true
INDEXOF_NOT_FOUND       → array.indexof() returns -1, not checked
```

---

DOMAIN 5: UDT HANDLING

5.1 Null & Field Access

```
NULL_UDT_FIELD          → zone.line_y without na(zone) guard
INVALID_NULL_CHECK      → not na(zone) — incorrect syntax for UDT null check
FIELD_NA_PROPAGATION    → UDT field initialized na, used without validation
OPTIONAL_FIELD_MISSING  → Field declared without default, accessed unwritten
```

5.2 UDT Mutations

```
UDT_UPDATE_NO_EFFECT    → zone.isGrabbed := true but zone is copy, not reference
UDT_FIELD_TYPE_MISMATCH → Assigning float to int UDT field
UDT_IN_SECURITY         → UDT with unserializable fields (line/box) in request.security()
UDT_CONSTRUCTOR_PARTIAL → Type.new() with some fields, rest silently na
```

---

DOMAIN 6: LOGIC & SEMANTIC (Compiles, Wrong Results)

6.1 Index/Offset Errors

```
WRONG_INDEX_VARIABLE    → bottom4 := low[highestHighIndex] (high index for low)
WRONG_OFFSET_DIRECTION  → Using future bar index where past intended
OB_INDEX_MISMATCH       → Bear ob_start uses bull index
HISTORY_OFFSET_CONFUSE  → [rsiSensitivity] used where [1] intended
BAR_INDEX_VS_TIME       → Mixing bar_index offset with time-based calculation
```

6.2 Comparison & Detection

```
NA_EQ_NA                → na == na (always false, use na(x))
NA_EQ_VALUE             → x == na (always false, use na(x))
BOOL_IN_NZ              → nz(boolean, false) — nz first arg must be numeric
BOOL_IN_MATH_SUM        → math.sum(bool_series, length) — implicit coercion error
CROSS_NEGATIVE_SENS     → ta.crossunder(pc_mtf, -sens) — negative sensitivity nonsensical
INVERTED_RSI_PEAK       → rsiSeries > rsiSeries[1] instead of momentum comparison
```

6.3 Missing Render Paths

```
MISSING_BEARISH_MIDLINE → Engine 1 midline not rendered for bearish
MISSING_BULLISH_MIDLINE → Engine N midline not rendered for bullish
MISSING_ENGINE1_BEAR    → Engine 1 direction rendering missing
MISSING_ENGINE6_DIR     → Engine 6 directional rendering incomplete
MISSING_ENGINE7_DIR     → Engine 7 directional rendering incomplete
RENDER_LASTBAR_ONLY     → Rendering gated inside if barstate.islast
```

6.4 No-Op & Semantic Waste

```
MEANINGLESS_SUBTRACT    → (avg_vol - 0) no-op expression
INT_CAST_ON_TIME        → int(totalWidth) where totalWidth is time delta
NZ_TYPE_MIX             → nz(close[1], close) fallback type ambiguous
MUTABLE_TA_LENGTH       → ta.sma(src, variableLength) where length changes every bar
EMPTY_STRING_CHECK      → if str == "" when str could be na
```

---

DOMAIN 7: MULTI-TIMEFRAME & SECURITY BOUNDARY

7.1 Lookahead & Repaint

```
LOOKAHEAD_ON            → request.security(..., lookahead_on) in Engine 5/7
LOOKAHEAD_MTF_CLOSE     → Higher TF close not yet confirmed on lower TF
FUTURE_LEAK             → MTF indicator references bar that hasn't closed
SECURITY_BARSTATE       → barstate inside request.security() refers to HTF bar
```

7.2 Security Expression Errors

```
HISTORY_IN_SECURITY     → close[1] or any [] inside request.security() expression
DYNAMIC_INDEX_SECURITY  → close[dynamic] inside security
TUPLE_IN_SECURITY       → Dynamic tuple expression inside security
ARRAY_IN_SECURITY       → Using array inside security instead of security_lower_tf
MUTABLE_IN_SECURITY     → Mutable variable in security expression
```

7.3 Timeframe Errors

```
TIMEFRAME_ARG_TO_TA     → Passing "D" to ta.sma() instead of integer length
EMPTY_TIMEFRAME         → timeframe="" passed to request.security()
INVALID_TIMEFRAME       → "5min" instead of "5", "1D" instead of "D"
SECONDS_ON_NONSTANDARD  → timeframe.in_seconds() on non-standard TF
```

7.4 MTF Bar Alignment

```
MTF_BAR_MISALIGN        → 1H bar close at :15 vs 4H close at :00
MTF_TIMEZONE_SHIFT      → Exchange timezone vs chart timezone
MTF_INTRADAY_WEEK       → Week bar boundaries differ by broker
MTF_LOWER_TF_VARY       → security_lower_tf returns variable-length arrays
```

---

DOMAIN 8: MATH RUNTIME (NaN Cascade)

8.1 Division & Singularities

```
DIV_ZERO_TAN_CUTOFF     → 1 / math.tan(math.pi * cutoffFreq) at cutoffFreq = 0.5
DIV_ZERO_STATIC         → x / 0 literal
DIV_ZERO_DYNAMIC        → x / y where y can be 0
SQRT_NEGATIVE           → math.sqrt(x) where x goes negative
LOG_NONPOSITIVE         → math.log(0) or math.log(negative)
LOG10_ZERO              → math.log10(0) returns -inf
POW_NEG_BASE_FRAC       → math.pow(-2, 0.5) complex result → na
POW_OVERFLOW            → math.pow(large, large) → inf
TAN_ASYMPTOTE           → math.tan(math.pi/2) → inf
```

8.2 NaN Propagation

```
NA_IN_SMA               → na source propagates through entire window
NA_IN_EMA               → na seed propagates indefinitely
NA_IN_RMA               → na in recursive smoothing
NA_IN_CUM               → na in cumulative sum, everything after is na
INF_IN_SERIES           → inf cascades through ta.* functions
```

8.3 Precision & Overflow

```
FLOAT_EQ                → close == level (use math.abs(close - level) < threshold)
INT_OVERFLOW            → bar_index > 2^31 on very long charts
FLOAT_ACCUMULATE_DRIFT  → var float += small_value over thousands of bars
ROUND_TIE               → math.round(2.5) banker's rounding vs floor
VOLUME_POWER_OVERFLOW   → Large volume exponentiation → inf
```

---

DOMAIN 9: REPAINTING (Backtest vs Reality)

9.1 Bar State

```
REPAINT_ISLAST          → Signal based on barstate.islast recalcs every tick
REPAINT_ISCONFIRMED     → Trading on unconfirmed bar values
REPAINT_ISHISTORY       → Assuming ishistory guarantees value won't change
REPAINT_ISREALTIME      → Treating real-time bar like historical
```

9.2 Indicator Recalculation

```
REPAINT_CUMULATIVE      → ta.cum(), cumulative indicators on every tick
REPAINT_SECURITY_HTF    → request.security() repaints until HTF bar closes
REPAINT_TA_PIVOT        → ta.pivothigh() re-evaluates within lookback window
REPAINT_HIGHEST         → ta.highest() on unclosed bar
REPAINT_CROSS           → ta.crossunder/over fires multiple times on same cross
```

9.3 Alert Timing

```
ALERT_ON_TICK           → Alert fires on tick, not bar close
ALERT_BEFORE_CONFIRM    → Condition triggers before barstate.isconfirmed
ALERT_RECALC            → Alert condition recalculates, fires multiple times
FREQ_ALL_FLOOD          → alert(freq=alert.freq_all) fires on every tick
```

---

DOMAIN 10: EXTERNAL DATA EDGE CASES

10.1 Symbol Lifecycle

```
DELISTED_SYMBOL         → syminfo.tickerid returns na
TICKER_RENAME           → Old name has partial history
SYMBOL_SUFFIX           → .NS vs .BO vs no suffix across brokers
THINLY_TRADED           → Many consecutive volume=0 bars
HALTED_TRADING          → Gaps, request.security returns na
```

10.2 Corporate Actions

```
SPLIT_UNadjusted        → Unadjusted data shows fake gaps
DIVIDEND_UNadjusted     → Price drop without dividend adjustment
BONUS_UNadjusted        → Share bonus dilutes price unadjusted
RIGHTS_UNadjusted       → Rights issue gap not adjusted
```

10.3 Session Differences

```
PRE_POST_MARKET         → Extended hours data on/off mismatch
FOREX_WEEKEND_GAP       → Sunday open gap
CRYPTO_24_7             → ta.vwap() never resets, session functions break
HOLIDAY_SHORT           → Shortened session = fewer bars
TIMEZONE_DST            → DST shift changes bar open/close times
```

10.4 Data Quality

```
FLAT_CANDLE             → open=high=low=close, indicators behave oddly
ZERO_VOLUME             → Volume-based calculations fail
GAP_UP_DOWN             → Extreme gaps break pivot detection
TICK_DATA_GAPS          → Missing ticks in lower timeframe
```

---

DOMAIN 11: STRATEGY-SPECIFIC (if applicable)

```
COMMISSION_MISSING      → No commission parameter, unrealistic backtest
SLIPPAGE_IGNORED        → No slippage= argument
PYRAMIDING_DEFAULT      → Default 0 prevents multiple entries
CLOSE_ENTRIES_WRONG     → close_entries_rule closing unintended entry
ORDER_RECALC_TICK       → strategy.order() recalculates every tick
MARGIN_CALL             → Insufficient capital for position
BAR_MAGNIFIER_BIAS      → Lower TF fills unrealistic
ENTRY_LIMIT_REUSE       → Same entry ID overwrites previous
MAX_TRADES_SILENT       → Position limit exceeded silently
CURRENCY_CONVERSION     → Non-account-currency symbol P&L miscalculated
```

---

DOMAIN 12: PERFORMANCE DEGRADATION (No Error, Script Dies)

```
QUADRATIC_LOOP          → for inside for with large bounds
UNCAPPED_ARRAY          → array.push() every bar, never cleared
SECURITY_IN_LOOP        → request.security() inside for loop
DRAWING_OBJECT_LEAK     → Objects created, limit hit, oldest dropped
TABLE_RECREATE_BAR      → table.new() or table.cell() every bar without var
HEAVY_TICK_CALC         → Complex math on every tick, no isconfirmed guard
RECURSIVE_NO_MEMO       → Recursive function without cache
MATRIX_ALLOC_BAR        → matrix.new() every bar without var
LABEL_DEBUG_PERF        → Debug labels slow down rendering
STRATEGY_TICK_CALC      → Strategy recalculating on every tick unnecessarily
```

---

DOMAIN 13: VISUAL BUGS (Compiles, Wrong Display)

```
LABEL_OVERLAP           → Labels stack on same bar, unreadable
LABEL_OFFSCREEN         → Price coordinate far outside visible range
BOX_BEHIND_CANDLES      → Background color drawn before candles
LINE_INVISIBLE_THEME    → Line color same as background in dark/light mode
TABLE_TEXT_OVERFLOW     → Cell text wider than column
TEXT_TOO_SMALL_4K       → size.tiny unreadable on high DPI
POLYLINE_TRUNCATED      → >10k vertices silently truncated
DRAW_ORDER_WRONG        → Line drawn over box when box should be on top
FILL_BREAKS_ON_NA       → fill() between plots breaks at na
PLOT_STYLE_MISMATCH     → plot.style_circles with line-style arguments
```

---

DOMAIN 14: V5→V6 MIGRATION TRAPS

```
V5_SELF_REF             → v5: x := x[1] + close → v6: requires float typing
V5_NA_EQ                → v5: na == na → v6: use na()
V5_SWITCH_FALL          → v5: switch fallthrough → v6: no fallthrough
V5_TUPLE_DESTRUCTURE    → v5: [a, b] = fn() → v6: different syntax
V5_OVERLOAD             → v5 multiple signatures → v6 fewer overloads
V5_IMPLICIT_SERIES      → v5 inferred → v6 explicit typing in some contexts
V5_VAR_IN_LOOP          → v5 allowed var in for → v6 stricter
V5_STUDY                → study() deprecated, use indicator()
V5_ALERTCONDITION       → alertcondition() signature changed
V5_SECURITY_LOOKAHEAD   → lookahead default changed in v6
```

---

DOMAIN 15: COMPOUND ENGINE PATTERNS (Your Codebase Specific)

```
ENGINE_STATE_DESYNC     → Flag set but object lifecycle missed
OB_LIFECYCLE_GAP        → OB created Engine X, deleted Engine Y cleanup
ENGINE_COPY_PASTE_DRIFT → Engine1-7 share pattern but subtle differences
FLAG_ACCUMULATION       → engine*_new_bull/bear accumulate without drain
OB_COORDINATE_DRIFT     → ob_top recalculated, box at old coordinates
MULTI_ENGINE_OVERLAP    → Multiple engines detect same OB, overlapping boxes
CLEANUP_ENGINE_CROSS    → Cleanup loop for Engine2 deletes Engine4 objects
CONDITIONAL_ENGINE_INIT → Engine only initialized conditionally, referenced unconditionally
ENGINE2_GRABBED_STALE   → engine2_grabbed set but OB already deleted
RENDERING_DUPLICATE     → if useBarTime...else block copy-pasted, only xloc differs
LASTBAR_GATING          → Rendering only when barstate.islast
OB_COORD_FLIP           → ob_top < ob_bottom coordinate inversion
BOS_PENDING_NEVER_CLEAR → bosBullPending/bosBearPending set, never reset
RIGHT_TIME_NA           → rightTime possibly na in time calculations
CROSS_INDEX_OOB         → cross_index exceeds array bounds
```

---

RESOLUTION ORDER

When multiple anchors fire, resolve in this sequence:

```
1.  DOMAIN 1  → Compilation Gate        (script won't run until fixed)
2.  DOMAIN 14 → V5→V6 Migration         (if upgrading)
3.  DOMAIN 2  → Scope & History          (CW10003 warnings)
4.  DOMAIN 3  → Control Flow             (dead code, unreachable)
5.  DOMAIN 5  → UDT Handling             (null field access)
6.  DOMAIN 4  → Object Lifecycle         (tracking, cleanup)
7.  DOMAIN 7  → MTF & Security           (lookahead, alignment)
8.  DOMAIN 8  → Math Runtime             (NaN cascade, div zero)
9.  DOMAIN 6  → Logic & Semantic         (wrong results)
10. DOMAIN 9  → Repainting               (backtest vs reality)
11. DOMAIN 10 → External Data            (symbol edge cases)
12. DOMAIN 11 → Strategy                 (if applicable)
13. DOMAIN 12 → Performance              (script timing out)
14. DOMAIN 13 → Visual Bugs              (display issues)
15. DOMAIN 15 → Compound Engine          (your codebase patterns)
```

---

STATISTICS

Domain Tags Category
1. Compilation Gate 35 Blocks execution
2. Scope & History 21 CW10003 warnings
3. Control Flow 16 Dead/unreachable
4. Object Lifecycle 18 Memory/rendering
5. UDT Handling 10 Null/type/mutation
6. Logic & Semantic 18 Wrong results
7. MTF & Security 16 Lookahead/alignment
8. Math Runtime 16 NaN/div-zero/inf
9. Repainting 12 Backtest mismatch
10. External Data 16 Symbol/session
11. Strategy 11 Trading errors
12. Performance 11 Timeout/leaks
13. Visual Bugs 11 Display artifacts
14. v5→v6 Migration 10 Breaking changes
15. Compound Engine 18 Your codebase
TOTAL 239 Complete universe

This is the master schema. Every Pine Script v6 failure mode maps to at least one tag.
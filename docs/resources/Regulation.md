Pine Script v6 Complete Error Prevention Guide (Corrected Edition)

Source: Pine Script v6 User Manual (converted Markdown)
Total Rules: 176
Principle: NEVER patterns cause errors. ALWAYS patterns prevent errors.
Summary
Section Rules
1. Script Structure 3
2. Line Continuation 2
3. Identifiers 1
4. Operators 4
5. Variable Declarations 8
6. Conditional Structures 6
7. Loops 8
8. Type System 10
9. Built-in Functions 8
10. Arrays 12
11. Matrices 12
12. Maps 5
13. User-Defined Functions 5
14. Methods 3
15. Objects 5
16. Enums 3
17. Alerts 6
18. Requests 11
19. Strategies 15
20. Plots 7
21. Colors 3
22. Bar States 4
23. Time 3
24. Debugging 4
25. Limitations 10
26. Error Messages Reference 20
27. Migration-Specific (v6) 10

TOTAL: 176 rules
---

Section 1: Script Structure

Rule 1.1 - Version Annotation

Never omit the version compiler annotation.
Always place //@version=6 at the top of the script.

```pinescript
// WRONG - no version (defaults to v1)
indicator("My Script")

// CORRECT
//@version=6
indicator("My Script")
```

Source: Section 4.0

---

Rule 1.2 - Global Scope Position

Never start global scope lines with whitespace (space or tab).
Always start global scope statements at column 0.

```pinescript
// WRONG - leading space
  plot(close)

// CORRECT
plot(close)
```

Source: Section 4.2

---

Rule 1.3 - Local Block Indentation

Never use inconsistent indentation in local blocks.
Always use exactly 4 spaces or 1 tab for each indentation level.

```pinescript
// WRONG - inconsistent indentation
if condition
   value := 1    // 3 spaces
    result := 2  // 4 spaces

// CORRECT - consistent
if condition
    value := 1
    result := 2
```

Source: Section 4.2

---

Section 2: Line Continuation

Rule 2.1 - Wrapped Lines Indentation

Never indent wrapped lines with 4 spaces or 1 tab.
Always use indentation that is NOT a multiple of 4 (e.g., 2 spaces, 6 spaces).

```pinescript
// WRONG - 4 spaces creates local block
a = open +
    high +
    low +
    close

// CORRECT - 2 spaces
a = open +
  high +
  low +
  close
```

Source: Section 4.4

Error prevented: mismatched input 'plot' expecting 'end of line without line continuation'

---

Rule 2.2 - Function Call Wrapping

Never break a function call at a point that creates invalid syntax.
Always wrap long function calls after commas or before closing parentheses.

```pinescript
// WRONG - break in middle of argument
plot(ta.correlation(src, ovr,
length), color = color.purple)

// CORRECT - break after comma
plot(ta.correlation(src, ovr, length),
     color = color.purple)
```

Source: Section 4.4

---

Section 3: Identifiers

Rule 3.1 - Identifier First Character

Never start an identifier with a number.
Always start identifiers with A-Z, a-z, or underscore (_).

```pinescript
// WRONG - starts with number
1var = close
123value = high

// CORRECT
var1 = close
value123 = high
_var = close
```

Source: Section 5.0

---

Section 4: Operators

Rule 4.1 - na Comparison

Never test for na using the == operator.
Always use the na() function.

```pinescript
// WRONG - does not work correctly
if myVar == na
    // Never works as expected

// CORRECT
if na(myVar)
    // Handle na case
```

Source: Section 9.9

---

Rule 4.2 - History Reference Chaining

Never chain the [] operator multiple times.
Always reference past values directly.

```pinescript
// WRONG - chaining not allowed
close[1][2]

// CORRECT
close[3]
// or
temp = close[1]
value = temp[2]
```

Source: Section 6.4

---

Rule 4.3 - Series vs Simple Multiplication

Never assume multiplication of "input" and "series" yields "input".
Always understand that any operation with "series" yields "series".

```pinescript
// WRONG - expecting input result
lenInput = input.int(14, "Length")
factor = year > 2020 ? 3 : 1  // series int
adjustedLength = lenInput * factor  // series int, NOT input
ma = ta.ema(close, adjustedLength)  // ERROR - expects simple int
```

Source: Section 6.0

Error prevented: Cannot call 'ta.ema' with argument 'length'='adjustedLength'. An argument of 'series int' type was used but a 'simple int' is expected

---

Section 5: Variable Declarations

Rule 5.1 - na Type Declaration

Never declare a variable with na without specifying its type.
Always specify the type explicitly.

```pinescript
// WRONG - type unknown
myVar = na

// CORRECT
float myVar = na
myVar = float(na)  // also correct
```

Source: Section 6.1

---

Rule 5.2 - var for Persistence

Never omit var when a variable must retain its value across bars.
Always use var for counters, accumulators, and persistent state.

```pinescript
// WRONG - resets every bar
count = 0
if close > open
    count := count + 1

// CORRECT
var count = 0
if close > open
    count := count + 1
```

Source: Section 6.4.2

---

Rule 5.3 - var for Drawings

Never create and delete drawings on every bar when you can update them.
Always use var to declare drawing objects once, then update with setters.

```pinescript
// WRONG - inefficient, creates/deletes every bar
closeLine = line.new(bar_index - 1, close, bar_index, close)
line.delete(closeLine[1])

// CORRECT - create once, update
var closeLine = line.new(bar_index - 1, close, bar_index, close)
if barstate.islast
    line.set_xy1(closeLine, bar_index - 1, close)
    line.set_xy2(closeLine, bar_index, close)
```

Source: Section 6.4.2

---

Rule 5.4 - var Performance

Never use var for simple constants (minor performance penalty).
Always prefer plain assignment for constants unless initialization is expensive.

Source: Section 6.4.2

```pinescript
// PREFERRED for simple constants
const COLOR_BULL = color.green

// ACCEPTABLE but has minor penalty
var COLOR_BULL = color.green
```

---

Rule 5.5 - varip Realtime Behavior

Never use varip for logic that must backtest accurately.
Always understand that varip-based logic cannot reproduce realtime behavior in backtesting.

Source: Section 6.4.3

---

Rule 5.6 - varip Reset Pattern

Never forget to reset varip variables on new bars.
Always use barstate.isnew to reset varip counters.

```pinescript
// WRONG - never resets
varip int updateNo = na
updateNo := updateNo + 1

// CORRECT - resets on new bar
varip int updateNo = na
if barstate.isnew
    updateNo := 1
else
    updateNo := updateNo + 1
```

Source: Section 6.4.3

---

Rule 5.7 - Variable Reassignment Operator

Never use = to reassign an existing variable.
Always use := for reassignment.

```pinescript
// WRONG - creates new local variable
var count = 0
if condition
    count = count + 1  // Creates new local variable!

// CORRECT - reassigns existing
var count = 0
if condition
    count := count + 1
```

Source: Section 6.3

---

Rule 5.8 - Tuple Underscore for Unused Values

Never declare unused tuple values with named variables.
Always use underscore _ for unused tuple returns.

```pinescript
// WRONG - creates unnecessary variables
[bbMiddle, bbUpper, bbLower] = ta.bb(close, 5, 4)
// bbMiddle never used

// CORRECT - discard unused
[_, bbUpper, bbLower] = ta.bb(close, 5, 4)
```

Source: Section 6.2.1

---

Section 6: Conditional Structures

Rule 6.1 - plot() in Conditionals

Never place plot(), plotshape(), plotchar(), bgcolor(), barcolor(), fill(), hline(), alertcondition() inside conditional blocks.
Always place these functions in global scope and control them via conditional values or na.

```pinescript
// WRONG - plot inside if
if close > open
    plot(close)

// CORRECT - conditional value
plot(close > open ? close : na)
```

Source: Section 7.0

---

Rule 6.2 - strategy.*() in Conditionals

Never call strategy.*() functions inside conditional blocks without understanding execution.
Always ensure strategy functions execute on every bar when they need historical continuity.

Source: Section 7.0

---

Rule 6.3 - if Return Value by Type (Corrected)

Never assume an if without else returns na for all types.
Always know the return value depends on the type:

· bool → returns false (not na)
· string → returns "" (empty string)
· All other types → returns na

Source: Section 7.2, 9.5.3

```pinescript
// v6 behavior
result = if close > open
    true
// result is false when condition false (not na)

text = if close > open
    "Up"
// text is "" when condition false (not na)

number = if close > open
    1.0
// number is na when condition false
```

---

Rule 6.4 - Type Matching in Conditional Returns

Never have conditional branches return different types when assigned to a variable.
Always ensure all branches return the same type.

```pinescript
// WRONG - float and string
x = if close > open
    close      // float
else
    "open"     // string - ERROR

// CORRECT - all branches same type
x = if close > open
    close
else
    open       // float
```

Source: Section 7.4

---

Rule 6.5 - switch Default Block

Never omit the default => block when the switch is assigned to a variable.
Always include a default block that returns a compatible value.

Source: Section 7.3

```pinescript
// WRONG - no default
color myColor = switch x
    1 => color.red
    2 => color.green

// CORRECT
color myColor = switch x
    1 => color.red
    2 => color.green
    => color.blue
```

---

Rule 6.6 - switch No Fall-Through

Never expect switch to fall through to next case.
Always know that only one local block executes.

Source: Section 7.3

---

Section 7: Loops

Rule 7.1 - Unnecessary Loops

Never use a loop when a built-in function exists.
Always prefer built-ins like ta.sma(), ta.sum(), ta.highest().

```pinescript
// INEFFICIENT
float sum = 0
for i = 0 to length - 1
    sum := sum + close[i]
float avg = sum / length

// EFFICIENT
float avg = ta.sma(close, length)
```

Source: Section 8.1

---

Rule 7.2 - Loop Local Variable Access

Never access loop counter or loop-declared variables outside the loop.
Always use variables declared in outer scope if needed outside.

```pinescript
// WRONG - i not accessible outside
for i = 0 to 10
    label.new(bar_index, high, str.tostring(i))
label.new(bar_index, low, str.tostring(i))  // ERROR - i not defined

// CORRECT - capture value
var int lastI = na
for i = 0 to 10
    lastI := i
label.new(bar_index, low, str.tostring(lastI))
```

Source: Section 8.3

---

Rule 7.3 - Loop Return Value

Never forget that loops implicitly return a value.
Always assign loop result to a variable if needed.

```pinescript
// CORRECT - capturing loop return
string result = for number in array
    if number == 8
        break
    tempString += str.tostring(number)
    tempString
```

Source: Section 8.3

---

Rule 7.4 - for Loop Counter Step

Never assume step is always positive.
Always use negative step to count downward.

```pinescript
// Counting down
for i = 10 to 1
    // i goes 10, 9, 8, ... 1
```

Source: Section 8.4

---

Rule 7.5 - while Loop Infinite

Never forget to update the condition variable inside a while loop.
Always modify the condition variable to avoid infinite loops.

```pinescript
// WRONG - infinite loop
while j < 10
    // j never changes

// CORRECT
while j < 10
    j := j + 1
```

Source: Section 8.5

---

Rule 7.6 - for...in Map Modification

Never add or remove key-value pairs while iterating directly through a map.
Always copy the map or iterate through map.keys() array.

```pinescript
// WRONG - modifying during iteration
for [key, value] in myMap
    if condition
        myMap.remove(key)  // RUNTIME ERROR

// CORRECT - iterate through keys array
for key in myMap.keys()
    if condition
        myMap.remove(key)
```

Source: Section 8.8

---

Rule 7.7 - Loop 500ms Timeout

Never write a loop that may take more than 500ms on any single bar.
Always optimize loops or move work across bars.

Source: Section 21.0.3

Error prevented: Loop is too long (> 500 ms)

---

Rule 7.8 - Conditional Function Calls in Loops

Never call functions that need historical continuity inside conditional loop blocks.
Always call such functions in global scope.

Source: Section 2.4

---

Section 8: Type System

Rule 8.1 - Type Qualifier Hierarchy

Never use a weaker qualifier where a stronger one is required.
Always know that const < input < simple < series.

Source: Section 9.0

---

Rule 8.2 - Cannot Lower Qualifier

Never change a value's qualifier to one lower on the hierarchy.
Always understand that once a value becomes "series", it stays "series".

Source: Section 9.0

---

Rule 8.3 - const No Reassignment

Never reassign a const variable.
Always use var or plain assignment for mutable variables.

```pinescript
// WRONG
const float myVar = 0.0
myVar := 1.0  // ERROR

// CORRECT
float myVar = 0.0
myVar := 1.0
```

Source: Section 9.1

---

Rule 8.4 - const string for title

Never use non-const strings for title parameters.
Always use "const string" for plot titles, input titles, etc.

```pinescript
// WRONG - simple string
title = syminfo.ticker
plot(close, title)  // ERROR

// CORRECT - const string
plot(close, "My Plot")
```

Source: Section 9.1

---

Rule 8.5 - bool Never na (v6)

Never assign na to a bool variable in v6.
Always use true or false only.

Source: Section 9.5.3

---

Rule 8.6 - Auto-casting Direction

Never assume float to int auto-casting exists.
Always know auto-casting is int → float → bool only.

Source: Section 9.10

```pinescript
// WRONG - float to int not auto-cast
length = 10.0
sma = ta.sma(close, length)  // ERROR

// CORRECT - explicit cast
sma = ta.sma(close, int(length))
```

---

Rule 8.7 - na Type Required

Never use na without type context when declaring variables.
Always specify type or cast.

```pinescript
// WRONG
myVar = na

// CORRECT
float myVar = na
myVar = float(na)
```

Source: Section 9.9

---

Rule 8.8 - na() for na Test

Never use == to test for na.
Always use na() function.

Source: Section 9.9

---

Rule 8.9 - nz() for na Replacement

Never let na propagate through calculations.
Always use nz() to replace na with default values.

```pinescript
// WRONG - na propagates
allTimeHigh := math.max(allTimeHigh, high)  // allTimeHigh starts na

// CORRECT
allTimeHigh := math.max(nz(allTimeHigh), high)
```

Source: Section 9.9

---

Rule 8.10 - Explicit Cast for Division

Never assume integer division when you need float result.
Always cast at least one operand to float.

```pinescript
// v5 behavior - integer division for const ints
// v6 behavior - always returns fractional
result = 5 / 2  // v6: 2.5, v5: 2

// For integer division in v6
intResult = int(5 / 2)  // 2
```

Source: Migration to v6 guide

---

Section 9: Built-in Functions

Rule 9.1 - ta.ema() Length Type

Never pass "series int" to ta.ema() length parameter.
Always use "const", "input", or "simple" int.

```pinescript
// WRONG
lenInput = input.int(14)
factor = year > 2020 ? 3 : 1
adjusted = lenInput * factor  // series int
ta.ema(close, adjusted)  // ERROR

// CORRECT
ta.sma(close, adjusted)  // ta.sma accepts series int
```

Source: Section 6.0

---

Rule 9.2 - ta.cum() Returns Series

Never assume ta.cum() returns a single value.
Always understand it accumulates over all bars.

Source: Section 3.0

---

Rule 9.3 - Functions with Side Effects

Never call functions with side effects (like label.new()) conditionally without understanding execution.
Always ensure side-effect functions execute on every bar if they need historical continuity.

Source: Section 2.4.1

---

Rule 9.4 - fill() Requires Two Plots or Two Hlines

Never mix plot and hline IDs in a single fill() call.
Always use two plot IDs or two hline IDs.

```pinescript
// WRONG - mixing types
plotID = plot(close)
hlineID = hline(0)
fill(plotID, hlineID, color.blue)  // ERROR

// CORRECT - two plots
plot1 = plot(close)
plot2 = plot(open)
fill(plot1, plot2, color.blue)
```

Source: Section 10.0

---

Rule 9.5 - fill() Color Series

Never assume fill() color must be constant.
Always know it accepts "series color".

Source: Section 10.0

---

Rule 9.6 - hline() Price Constant

Never pass "series" values to hline() price parameter.
Always use "input" or "const" values.

```pinescript
// WRONG
hline(ta.sma(close, 20))  // ERROR

// CORRECT
hline(0)
hline(input.int(100, "Level"))
```

Source: Section 12.0

---

Rule 9.7 - hline() Color Constant

Never pass "series color" to hline() color parameter.
Always use "input color" or "const color".

Source: Section 12.0

---

Rule 9.8 - input.*() in Local Blocks

Never expect input.*() calls in local blocks to behave differently.
Always know they function same as in global scope.

Source: Section 7.0

---

Section 10: Arrays

Rule 10.1 - Array Index Bounds

Never access array index without checking bounds.
Always ensure index < array.size().

```pinescript
// WRONG
value = array.get(myArray, 10)

// CORRECT
if 10 < array.size(myArray)
    value = array.get(myArray, 10)
```

Source: Section 13.8

Error prevented: Index xx is out of bounds. Array size is yy

---

Rule 10.2 - Empty Array Shift/Pop

Never call array.shift() or array.pop() on empty array.
Always check array.size() > 0 first.

Source: Section 13.8

Error prevented: Cannot use shift() if array is empty / Cannot use pop() if array is empty

---

Rule 10.3 - Array na Check

Never call array methods on na array variable.
Always initialize arrays before use.

```pinescript
// WRONG
array<int> a = na
array.push(a, 111)  // ERROR

// CORRECT
array<int> a = array.new<int>(0)
array.push(a, 111)
```

Source: Section 13.8

Error prevented: Cannot call array methods when ID of array is 'na'

---

Rule 10.4 - Maximum Array Size

Never create arrays larger than 100,000 elements.
Always use maxval = 100000 in input for array sizes.

```pinescript
// WRONG
sizeInput = input.int(1000000, "Size")
a = array.new<float>(sizeInput)  // ERROR if >100000

// CORRECT
sizeInput = input.int(100000, "Size", maxval=100000)
```

Source: Section 13.8

Error prevented: Array is too large. Maximum size is 100000

---

Rule 10.5 - Negative Array Size

Never create arrays with negative size.
Always use minval = 0 or minval = 1 in input.

Source: Section 13.8

Error prevented: Cannot create an array with a negative size

---

Rule 10.6 - Array Slice Index Order

Never use index_from >= index_to in array.slice().
Always ensure index_from < index_to.

Source: Section 13.8

Error prevented: Index 'from' should be less than index 'to'

---

Rule 10.7 - Array Slice Parent Bounds

Never let parent array shrink below slice window.
Always maintain parent array size when using slices.

Source: Section 13.8

Error prevented: Slice is out of bounds of the parent array

---

Rule 10.8 - Negative Index (v6)

Never assume negative indices work in v5.
Always know v6 allows negative indices (e.g., -1 for last element).

Source: Section 13.0 - v6 feature

---

Rule 10.9 - array.from() Type Consistency

Never mix types in array.from() arguments.
Always ensure all arguments are same type.

Source: Section 11.0

---

Rule 10.10 - array.includes() vs Loop

Never write manual loop to check if value exists in array.
Always use array.includes() for existence check.

Source: Section 11.0

---

Rule 10.11 - array.binary_search() Requires Sort

Never call array.binary_search() on unsorted array.
Always call array.sort() first (ascending order).

Source: Section 11.0

---

Rule 10.12 - Loop Guard Pattern

Never loop to array.size() without protection for empty arrays.
Always use array.size() == 0 ? na : array.size() - 1.

```pinescript
// CORRECT pattern
for i = 0 to (array.size(a) == 0 ? na : array.size(a) - 1)
    // safe access
```

Source: Section 13.8

---

Section 11: Matrices

Rule 11.1 - Matrix Index Bounds

Never access row/column outside matrix dimensions.
Always ensure row < matrix.rows() and column < matrix.columns().

Source: Section 14.7

Error prevented: The row/column index (xx) is out of bounds, row/column size is (yy)

---

Rule 11.2 - Matrix Row/Column Insert Size

Never insert row/column with array size mismatching matrix dimensions.
Always ensure row array size equals matrix columns, column array size equals matrix rows.

Source: Section 14.7

Error prevented: The array size does not match the number of rows/columns in the matrix

---

Rule 11.3 - Matrix na Check

Never call matrix methods on na matrix variable.
Always initialize matrices before use.

```pinescript
// WRONG
matrix<float> m = na
mCopy = m.copy()  // ERROR

// CORRECT
matrix<float> m = matrix.new<float>()
```

Source: Section 14.7

---

Rule 11.4 - Maximum Matrix Elements

Never create matrices with more than 100,000 total elements.
Always ensure rows * columns <= 100000.

Source: Section 14.7

Error prevented: Matrix is too large. Maximum size of the matrix is 100000 elements

---

Rule 11.5 - Matrix From/To Index Order

Never use from_row >= to_row or from_column >= to_column.
Always ensure from_* < to_*.

Source: Section 14.7

---

Rule 11.6 - Matrix Sum/Diff Dimensions

Never add or subtract matrices with different dimensions.
Always ensure same number of rows and columns.

Source: Section 14.7

Error prevented: Matrices 'id1' and 'id2' must have an equal number of rows and columns to be added

---

Rule 11.7 - Matrix Multiplication Dimensions

Never multiply matrices where columns of first != rows of second.
Always ensure matrix.columns(id1) == matrix.rows(id2).

Source: Section 14.7

Error prevented: The number of columns in the 'id1' matrix must equal the number of rows in the matrix 'id2'

---

Rule 11.8 - Square Matrix Operations

Never call matrix.det(), matrix.inv(), matrix.eigenvalues(), matrix.eigenvectors() on non-square matrices.
Always check matrix.is_square() first.

Source: Section 14.7

Error prevented: Operation not available for non-square matrices

---

Rule 11.9 - Matrix Row/Column Retrieval

Never assume matrix.row() or matrix.col() modify the original.
Always know they return copies (shallow for reference types).

Source: Section 14.3

---

Rule 11.10 - Matrix Reshape Element Count

Never change total element count with matrix.reshape().
Always ensure new_rows * new_columns == original_rows * original_columns.

Source: Section 14.4

---

Rule 11.11 - Matrix Sorting Columns

Never assume matrix.sort() sorts columns.
Always know it sorts rows based on column values.

Source: Section 14.4

---

Rule 11.12 - Matrix Copy (Shallow)

Never assume matrix.copy() creates deep copy for reference types.
Always implement custom deep copy for matrices containing objects.

Source: Section 14.4

---

Section 12: Maps

Rule 12.1 - Map na Check

Never call map methods on na map variable.
Always initialize maps with map.new<keyType, valueType>().

Source: Section 15.0

---

Rule 12.2 - Map Key Uniqueness

Never assume duplicate keys are allowed.
Always know map.put() replaces existing keys.

Source: Section 15.2

---

Rule 12.3 - Map Modification During Iteration

Never add or remove key-value pairs while iterating through a map with for...in.
Always copy map or iterate through map.keys() array.

Source: Section 8.8

Error prevented: Runtime error from modifying map during iteration

---

Rule 12.4 - Map Keys Insertion Order

Never assume map keys are sorted.
Always know map.keys() returns keys in insertion order.

Source: Section 15.2

---

Rule 12.5 - Map Value Types

Never store collections directly as map values.
Always wrap collections in user-defined types.

```pinescript
// WRONG - cannot store array directly in map
map<string, array<float>> myMap = map.new<string, array<float>>()  // ERROR

// CORRECT - use wrapper type
type Wrapper
    array<float> data
map<string, Wrapper> myMap = map.new<string, Wrapper>()
```

Source: Section 15.0

---

Section 13: User-Defined Functions

Rule 13.1 - Function Return Type

Never forget that function returns last expression value.
Always ensure last line is the return value (or use explicit return variable).

Source: Section 10.2

---

Rule 13.2 - Function Local Scope

Never access function parameters or local variables from outside.
Always return values needed from outer scope.

Source: Section 10.5

---

Rule 13.3 - No Nested Functions

Never declare a function inside another function.
Always declare all user functions in global scope.

Source: Section 10.5

---

Rule 13.4 - No Recursion

Never call a function from itself.
Always use iterative approaches.

Source: Section 10.0

---

Rule 13.5 - UDF Function Restrictions (Corrected)

Never assume all plotting functions can be called inside user-defined functions.
Always verify each function's restrictions individually. The manual explicitly prohibits plot() inside conditional structures, but does NOT provide a complete list of UDF-prohibited functions.

Source: Section 7.0 (conditional structures), Section 10.6 (UDF limitations vague in source)

Note: The manual confirms UDFs cannot call functions that require global scope execution patterns, but does not enumerate which specific functions are banned. Use caution.

---

Section 14: Methods

Rule 14.1 - Method First Parameter

Never omit explicit type for first parameter in user-defined method.
Always specify the type (the object the method belongs to).

Source: Section 11.1

---

Rule 14.2 - Method vs Function Syntax

Never call method using namespace prefix.
Always use dot notation: object.method() not namespace.method(object).

Source: Section 11.0

```pinescript
// WRONG
array.get(myArray, index)

// CORRECT
myArray.get(index)
```

---

Rule 14.3 - Method Overloading Requirements

Never define overloads with same number of parameters and same types.
Always ensure unique parameter counts or type combinations.

Source: Section 11.2

---

Section 15: Objects (UDT)

Rule 15.1 - UDT Field Type Requirements

Never use na as default field value without type context.
Always specify default value or accept na.

Source: Section 9.7

---

Rule 15.2 - Object var Behavior

Never assume var applies to object fields automatically.
Always know var applies to object reference, not fields.

Source: Section 9.7

---

Rule 15.3 - varip with UDT Fields

Never assume varip on object applies to all fields.
Always apply varip to individual fields in type declaration.

Source: Section 9.7

---

Rule 15.4 - Object Copy (Shallow)

Never assume copy() creates deep copy for reference type fields.
Always implement custom deep copy for fields that are collections or drawings.

Source: Section 12.2

---

Rule 15.5 - UDT Name Shadowing

Never name UDT after primitive types (int, float, string, bool, color).
Always choose unique names.

Source: Section 12.4

---

Section 16: Enums

Rule 16.1 - Enum Type Compatibility

Never compare members from different enums.
Always use members from same enum type.

Source: Section 13.1

---

Rule 16.2 - Enum Field Titles

Never assume field name equals display title.
Always use str.tostring(enumField) to get title.

Source: Section 13.2

---

Rule 16.3 - Enum in Maps

Never use non-fundamental types as map keys.
Always know enums are allowed as map keys (they are "simple" qualifier).

Source: Section 13.3

---

Section 17: Alerts

Rule 17.1 - alertcondition() Global Scope

Never place alertcondition() inside conditional blocks.
Always place at column 0 in global scope.

Source: Section 14.3

---

Rule 17.2 - alertcondition() Message Constant

Never use dynamic strings (series) in alertcondition() message.
Always use "const string" with optional placeholders.

Source: Section 14.3

```pinescript
// WRONG
alertcondition(condition, "Alert", str.tostring(close))

// CORRECT - placeholder
alertcondition(condition, "Alert", 'Value: {{plot("myPlot")}}')
```

---

Rule 17.3 - alert() Frequency Default

Never assume alert() triggers on every execution.
Always know default is alert.freq_once_per_bar (first call only).

Source: Section 14.1

---

Rule 17.4 - Strategy Alert Frequency

Never expect alert.freq_all to work in strategies without calc_on_every_tick = true.
Always enable calc_on_every_tick for per-tick alerts in strategies.

Source: Section 14.1

---

Rule 17.5 - Order Fill Alert Messages

Never forget to include {{strategy.order.alert_message}} placeholder to use custom messages.
Always include placeholder in alert message field.

Source: Section 14.2

---

Rule 17.6 - Alert Repainting Prevention

Never trigger alerts on unconfirmed bar values without understanding repainting.
Always use barstate.isconfirmed or freq_once_per_bar_close for confirmed alerts.

Source: Section 14.4

---

Section 18: Requests (request.security)

Rule 18.1 - HTF Non-Repainting Pattern

Never use lookahead = barmerge.lookahead_on without [1] offset.
Always use expression[1] with lookahead = barmerge.lookahead_on.

```pinescript
// WRONG - lookahead bias
request.security(syminfo.tickerid, "1D", close, lookahead = barmerge.lookahead_on)

// CORRECT - non-repainting
request.security(syminfo.tickerid, "1D", close[1], lookahead = barmerge.lookahead_on)
```

Source: Section 15.18.1

---

Rule 18.2 - Maximum Security Calls

Never exceed 40 unique request.*() calls.
Always reuse identical calls when possible.

Source: Section 21.2.1

Error prevented: Script requesting too many securities

---

Rule 18.3 - LTF with request.security()

Never use request.security() for lower timeframe when you need all intrabars.
Always use request.security_lower_tf() for complete intrabar arrays.

Source: Section 15.2

---

Rule 18.4 - gaps Parameter Behavior

Never assume gaps are filled by default.
Always know gaps = barmerge.gaps_off (default) fills gaps; gaps = barmerge.gaps_on returns na.

Source: Section 15.0

---

Rule 18.5 - lookahead Default

Never assume lookahead behavior is consistent across versions.
Always explicitly set lookahead parameter.

Source: Section 15.0

---

Rule 18.6 - Invalid Symbol Handling

Never let invalid symbols crash your script.
Always use ignore_invalid_symbol = true to return na.

Source: Section 15.0

---

Rule 18.7 - Dynamic Requests for Series Arguments

Never use "series" symbol/timeframe arguments without dynamic_requests = true.
Always enable dynamic requests when arguments change bar-to-bar.

Source: Section 15.16

---

Rule 18.8 - Request Expression Dependencies

Never use loop variables in request.*() expression arguments.
Always ensure expression does not depend on loop-invariant variables.

Source: Section 15.16.2

---

Rule 18.9 - Nested Requests

Never expect nested requests to work without dynamic requests.
Always use dynamic_requests = true for nested request.*() calls.

Source: Section 15.16.4

---

Rule 18.10 - HTF Timeframe Validation

Never request higher timeframe data without validating.
Always check that requested timeframe > chart timeframe.

Source: Section 15.18.1

---

Rule 18.11 - Future Leak Prohibition

Never use lookahead = barmerge.lookahead_on without offset in published scripts.
Always offset by [1] to avoid leaking future data.

Source: Section 15.18.1

Note: This is prohibited in script publications (moderated).

---

Section 19: Strategies

Rule 19.1 - Pyramiding Default

Never assume multiple entries in same direction are allowed.
Always set pyramiding = n in strategy() to allow multiple entries.

```pinescript
// WRONG - only 1 entry allowed
strategy("My Strategy")

// CORRECT - allows 4 entries
strategy("My Strategy", pyramiding=4)
```

Source: Section 19.2

---

Rule 19.2 - strategy.exit() Must Do Something

Never call strategy.exit() without exit parameters.
Always include at least one of: profit, limit, loss, stop, or trail_offset with trail_price/trail_points.

Source: Migration to v5 guide

---

Rule 19.3 - strategy.exit() Limit vs Stop

Never confuse limit and stop parameters with entry command behavior.
Always know that strategy.exit() with both creates TWO orders (take-profit AND stop-loss).

Source: Section 19.2

---

Rule 19.4 - strategy.exit() from_entry

Never assume strategy.exit() without from_entry applies only to recent entries.
Always know it applies to ALL entries in the position.

Source: Section 19.2

---

Rule 19.5 - strategy.close() Market Order

Never use strategy.close() for price-based exits.
Always use strategy.exit() for limit/stop orders.

Source: Section 19.2

---

Rule 19.6 - FIFO Closing Default

Never assume you can close specific entry trades in any order.
Always know FIFO (first in, first out) is default; use close_entries_rule = "ANY" to change.

Source: Section 19.2

---

Rule 19.7 - Margin Default (v6)

Never assume 0 margin means no checking (v5 behavior).
Always know v6 default margin is 100 (checked).

Source: Migration to v6 guide

---

Rule 19.8 - Commission Type

Never omit commission in backtests expecting realistic results.
Always set commission_type and commission_value for realistic simulation.

Source: Section 19.2

---

Rule 19.9 - Slippage Simulation

Never assume perfect order fills in backtesting.
Always set slippage parameter for realistic fills.

Source: Section 19.2

---

Rule 19.10 - calc_on_every_tick Repainting

Never use calc_on_every_tick = true without understanding repainting.
Always know historical bars cannot reproduce realtime tick behavior.

Source: Section 19.2

---

Rule 19.11 - calc_on_order_fills Unrealistic Fills

Never assume intrabar fills on historical bars are realistic.
Always understand each historical bar has only OHLC ticks.

Source: Section 19.2

---

Rule 19.12 - Strategy on Non-Standard Charts

Never backtest strategies on non-standard charts (Heikin Ashi, Renko, etc.).
Always use standard chart types or enable fill_orders_on_standard_ohlc.

Source: Section 19.0

---

Rule 19.13 - Maximum Orders (v5 vs v6)

Never assume order limit behavior is same across versions.
Always know v5: errors at 9000; v6: trims oldest orders.

Source: Migration to v6 guide

---

Rule 19.14 - strategy.order() vs strategy.entry()

Never expect strategy.order() to automatically reverse positions.
Always use strategy.entry() for automatic reversal.

Source: Section 19.2

---

Rule 19.15 - OCA Group Types

Never mix OCA types in same group.
Always use same oca_type for all orders in an OCA group.

Source: Section 19.2

---

Section 20: Plots

Rule 20.1 - Plot Count Limit (Corrected)

Never exceed 64 total plot counts.

Always know that:

· plot() with "const color" → 1 plot count
· plot() with any other color type ("simple", "input", "series") → 2 plot counts
· Other plot*() functions may generate multiple plot counts

Source: Section 21.1.1

To determine exact counts: Comment out suspected calls and observe the error message.

---

Rule 20.2 - transp Deprecated

Never use transp parameter.
Always use color.new(color, transparency).

Source: Section 9.2.5.1

---

Rule 20.3 - force_overlay for Pane Scripts

Never assume separate pane scripts can't draw on main chart.
Always use force_overlay = true in plotting functions to overlay on main chart.

Source: Section 12.0

---

Rule 20.4 - display Parameter

Never pollute chart scale with debug plots.
Always use display = display.data_window or display = display.all - display.pane.

Source: Section 12.0

---

Rule 20.5 - plot.style_columns Width

Never use linewidth expecting to control column width.
Always use plot.style_histogram for width control.

Source: Section 12.0

---

Rule 20.6 - Conditional Plotting with na

Never plot na values if you need line continuity.
Always use plot.style_linebr to break lines on na.

Source: Section 12.0

---

Rule 20.7 - Scale Distortion

Never plot price values (e.g., 40000) alongside oscillator values (0-100).
Always use separate panes or adjust scale.

Source: Section 12.0

---

Section 21: Colors

Rule 21.1 - Color Transparency for Plots

Never expect automatic color picker in Style tab when using color.new() with series color.
Always use color.new() on individual colors before conditional selection.

```pinescript
// WRONG - series color, no picker
plotColor = close > open ? color.new(color.green, 50) : color.new(color.red, 50)

// CORRECT - const colors created first
bullColor = color.new(color.green, 50)
bearColor = color.new(color.red, 50)
plotColor = close > open ? bullColor : bearColor
// Color pickers appear in Style tab
```

Source: Section 9.2.5

---

Rule 21.2 - color.from_gradient() Always Series

Never expect color pickers when using color.from_gradient().
Always know it returns "series color" regardless of inputs.

Source: Section 9.2.5

---

Rule 21.3 - Z-Index Order

Never assume all drawing types have same z-index.
Always know order (bottom to top): backgrounds, fills, plots, hlines, linefills, lines, boxes, labels, tables.

Source: Section 9.2.5

---

Section 22: Bar States

Rule 22.1 - barstate.isconfirmed for Historical Consistency

Never rely on realtime-only behavior for backtest consistency.
Always use barstate.isconfirmed when you need confirmed values.

Source: Section 9.1.4

---

Rule 22.2 - barstate.isnew Historical Behavior

Never assume barstate.isnew means "new bar" differently in realtime.
Always know it's true on ALL historical bars (each is "new" during sequential execution).

Source: Section 9.1.4

---

Rule 22.3 - barstate.islastconfirmedhistory for Last Bar

Never use barstate.islast for one-time calculations on last historical bar.
Always use barstate.islastconfirmedhistory to detect last historical bar before realtime.

Source: Section 9.1.4

---

Rule 22.4 - Strategy Bar States

Never assume bar states work same in strategies without calc_on_every_tick.
Always know strategies default to close-only execution.

Source: Section 9.1.4

---

Section 23: Time

Rule 23.1 - time_close on Non-Time-Based Charts

Never use time_close on tick charts or price-based charts expecting closing time.
Always know it returns na on realtime bars of non-time-based charts.

Source: Section 23.0

---

Rule 23.2 - Timestamp Format

Never assume UNIX timestamps are seconds.
Always know Pine Script uses milliseconds since 1970-01-01.

Source: Section 23.0

---

Rule 23.3 - str.format_time() for Readable Dates

Never use str.format() for time zone conversion.
Always use str.format_time() for time zone-aware formatting.

Source: Section 23.0

---

Section 24: Debugging

Rule 24.1 - Plot Debug Without Scale Impact

Never let debug plots distort script scale.
Always use display = display.data_window or display = display.all - display.pane.

Source: Section 18.0

---

Rule 24.2 - Conditional Plot Shapes

Never put plotshape() inside if blocks.
Always control with conditional value in series parameter.

```pinescript
// WRONG
if condition
    plotshape(true)

// CORRECT
plotshape(condition)
```

Source: Section 18.0

---

Rule 24.3 - Label Tooltips for Hover Debug

Never overcrowd chart with label text.
Always use tooltip parameter for hover-only information.

```pinescript
// Cleaner debugging
label.new(bar_index, high, "", tooltip = debugString)
```

Source: Section 18.0

---

Rule 24.4 - Pine Logs for Historical Debug

Never rely only on chart visuals for complex debugging.
Always use log.info(), log.warning(), log.error() for detailed execution traces.

Source: Section 18.3

---

Section 25: Limitations

Rule 25.1 - Maximum Bars Back

Never reference more than 5000 bars in the past without setting max_bars_back.
Always use max_bars_back parameter or function for large offsets.

Source: Section 21.5.1

---

Rule 25.2 - Maximum Bars Forward

Never position drawings more than 500 bars into the future with xloc.bar_index.
Always use xloc.bar_time for larger future offsets.

Source: Section 21.5.2

---

Rule 25.3 - Plot Count Management

Never approach 64 plot counts without tracking them.
Always comment out functions to identify plot count usage.

Source: Section 21.1.1

---

Rule 25.4 - Drawing Limits

Never assume unlimited lines, boxes, labels.
Always know defaults: ~50 each; max 500 for lines/boxes/labels, 100 for polylines.

Source: Section 21.1.2

---

Rule 25.5 - Table Position Uniqueness

Never create two tables in same position.
Always use different positions for multiple tables (max 9, one per position).

Source: Section 21.1.3

---

Rule 25.6 - Compilation Time Limit

Never write extremely large scripts without optimization.
Always know compilation limit is 2 minutes.

Source: Section 21.0.1

---

Rule 25.7 - Execution Time Limit

Never assume scripts will run indefinitely.
Always know execution limits: 20-40 seconds (account dependent).

Source: Section 21.0.2

---

Rule 25.8 - Compiled Token Limit

Never exceed 80,000 compiled tokens.
Always reduce repetitive code, use functions, utilize libraries.

Source: Section 21.3.1

---

Rule 25.9 - Variables Per Scope Limit

Never exceed 1,000 variables in any scope.
Always consolidate or split across scopes.

Source: Section 21.3.2

---

Rule 25.10 - Scope Count Limit

Never exceed 550 total scopes.
Always encapsulate logic in functions to reduce scope count.

Source: Section 21.3.3

---

Section 26: Error Messages Reference

Rule 26.1 - "Index xx is out of bounds. Array size is yy"

Always check index < array.size() before access.

Source: Section 13.8

---

Rule 26.2 - "Cannot call array methods when ID of array is 'na'"

Always initialize arrays with array.new<type>() before use.

Source: Section 13.8

---

Rule 26.3 - "Array is too large. Maximum size is 100000"

Always cap array size inputs with maxval = 100000.

Source: Section 13.8

---

Rule 26.4 - "Cannot create an array with a negative size"

Always use minval = 0 or minval = 1 for size inputs.

Source: Section 13.8

---

Rule 26.5 - "Cannot use shift() if array is empty" / "Cannot use pop() if array is empty"

Always check array.size() > 0 before shift/pop.

Source: Section 13.8

---

Rule 26.6 - "Slice is out of bounds of the parent array"

Always ensure parent array size remains larger than slice window.

Source: Section 13.8

---

Rule 26.7 - "The row/column index (xx) is out of bounds"

Always verify row < matrix.rows() and column < matrix.columns().

Source: Section 14.7

---

Rule 26.8 - "The array size does not match the number of rows/columns in the matrix"

Always match row insert array size to matrix columns, column insert array size to matrix rows.

Source: Section 14.7

---

Rule 26.9 - "Cannot call matrix methods when the ID of matrix is 'na'"

Always initialize matrices with matrix.new<type>() before use.

Source: Section 14.7

---

Rule 26.10 - "Matrix is too large. Maximum size of the matrix is 100000 elements"

Always ensure rows * columns <= 100000.

Source: Section 14.7

---

Rule 26.11 - "Loop is too long (> 500 ms)"

Always optimize loops, use built-ins, or distribute work across bars.

Source: Section 21.0.3

---

Rule 26.12 - "Script has too many local variables"

Always reduce variable count, consolidate expressions.

Source: Section 21.3.2

---

Rule 26.13 - "Cannot determine referencing length of a series"

Always use max_bars_back function for problematic series.

Source: Section 21.5.1

---

Rule 26.14 - "Script requesting too many securities"

Always stay under 40 unique request.*() calls.

Source: Section 21.2.1

---

Rule 26.15 - "If statement is too long"

Always break large if statements into functions.

Source: Section 20.1.11

---

Rule 26.16 - "Mismatched input '<...>' expecting '<...>'"

Always check line continuation indentation (avoid 4 spaces).

Source: Section 4.4

---

Rule 26.17 - "No viable alternative at character '$'"

Always check for invalid characters, especially in strings.

Source: Section 20.1.14

---

Rule 26.18 - "Memory limits exceeded"

Always avoid returning collections from request.*() functions unnecessarily.

Source: Section 20.1.19

---

Rule 26.19 - "Cannot call 'ta.ema' with argument 'length'='adjustedLength'"

Always use "simple int" for EMA length, not "series int".

Source: Section 6.0

---

Rule 26.20 - "Undeclared identifier"

Always declare variables before use; check scope.

Source: Section 10.5

---

Section 27: Migration-Specific Rules (v6)

Rule 27.1 - bool Never na

Never assign na to bool variables in v6.
Always use only true or false.

Source: Migration to v6 guide

---

Rule 27.2 - Lazy and/or Evaluation

Never rely on second expression being evaluated when first determines result.
Always know v6 uses lazy evaluation (short-circuit).

Source: Migration to v6 guide

---

Rule 27.3 - Dynamic Requests Default (Corrected)

Never add dynamic_requests = true to v6 scripts expecting it to enable dynamic requests.
Always know v6 defaults to dynamic_requests = true automatically. The parameter is only needed to explicitly disable dynamic requests.

Source: Migration to v6 guide

```pinescript
// v5 required
//@version=5
indicator("", dynamic_requests=true)

// v6 - dynamic_requests is default, no parameter needed
//@version=6
indicator("")
```

---

Rule 27.4 - Division of Const Ints

Never assume integer division for const ints in v6.
Always use int() to cast result if integer division needed.

Source: Migration to v6 guide

---

Rule 27.5 - No History on Literals

Never use [] operator on literal values in v6.
Always assign to variable first if history needed.

Source: Migration to v6 guide

---

Rule 27.6 - No History on UDT Fields

Never use [] directly on UDT fields in v6.
Always wrap in parentheses: (myObject[10]).field.

Source: Migration to v6 guide

---

Rule 27.7 - Default Margin 100

Never assume 0 margin (no checking) in v6.
Always know v6 default margin is 100 (checked).

Source: Migration to v6 guide

---

Rule 27.8 - Orders Trimmed at 9000 (v6)

Never assume strategy errors at 9000 orders in v6.
Always know v6 trims oldest orders instead.

Source: Migration to v6 guide

---

Rule 27.9 - timeframe.period Always Has Multiplier

Never compare timeframe.period to "D" in v6.
Always use "1D" instead.

Source: Migration to v6 guide

---

Rule 27.10 - No transp Parameter

Never use transp parameter in any function in v6.
Always use color.new().

Source: Migration to v6 guide

---




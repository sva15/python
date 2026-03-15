# 🏛️ Level 1 – Exploring | Lesson 7: Use `if` Statements in Python

## *"The Decision Engine"*

> **O'Reilly Skill Objective:** *Control program flow with if statements.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, type conversion, `type()`, comparison operators (briefly), f-strings

---

## 📖 Table of Contents

1. [Opening Scene](#1--opening-scene)
2. [Concept Introduction – Harvey Specter](#2--concept-introduction--harvey-specter)
3. [Technical Breakdown – Mike Ross](#3--technical-breakdown--mike-ross)
4. [Edge Cases & Pitfalls – Louis Litt](#4--edge-cases--pitfalls--louis-litt)
5. [Mental Model – Donna Paulsen](#5--mental-model--donna-paulsen)
6. [Practice Exercises – Rachel Zane](#6--practice-exercises--rachel-zane)
7. [Debugging Scenario](#7--debugging-scenario)
8. [Real-World Application](#8--real-world-application)
9. [Knowledge Check](#9--knowledge-check)

---

## 1. 🎬 Opening Scene

*It's 7:45 AM. You're early for once. Harvey is already in his office — he never leaves. He's staring at a screen, frowning.*

**Harvey:** "We have a problem."

*He turns the screen toward you. It's the client profitability report from Lesson 1 — but now with 200+ clients.*

**Harvey:** "Jessica wants a system that automatically flags clients based on their profitability. Red flag for unprofitable clients. Yellow for borderline. Green for profitable. She also wants high-value clients routed to senior partners and low-value ones to associates."

**You:** "So the program needs to… make decisions?"

**Harvey:** "Exactly. Right now your code calculates numbers and stores data. But it doesn't **think**. It doesn't say 'if this client is unprofitable, do X; otherwise, do Y.' It just runs straight through, top to bottom, like a train on tracks."

*Mike walks in.*

**Mike:** "Today we give your programs a steering wheel. **`if` statements** let your code make decisions — take different paths based on different conditions. It's the difference between a script and an *intelligent* system."

**Harvey:** "By end of day, I want a system that takes in any client's data and automatically categorizes them. No human judgment needed."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Every decision in life — and in code — comes down to a question with a yes/no answer."

**Harvey:** "Should we take this case? **If** the client can pay and the case is winnable, **then** yes. **Otherwise**, no."

**Harvey:** "Programming is no different. Your code constantly needs to answer questions:"
- Is this number positive or negative?
- Is this client profitable?
- Did the user enter valid data?
- Is this case urgent?
- Has the deadline passed?

**Harvey:** "`if` statements are the **brain** of your program. Without them, code runs in a single straight line. With them, code can branch, adapt, and respond to any situation."

**Harvey:** "Three levels:"

| Structure | Real-World Analogy |
|-----------|-------------------|
| `if` | "If the light is green, go" |
| `if-else` | "If guilty, prison. Otherwise, freedom" |
| `if-elif-else` | "If revenue > $1M, platinum. Elif > $500K, gold. Elif > $100K, silver. Else, bronze" |

**Harvey:** "Master these, and your programs can handle any scenario."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Let's build from the simplest case to the most complex."

---

### 3.1 Comparison Operators — The Foundation

**Mike:** "Before you can write `if` statements, you need to know how to **ask questions** in Python. That's what comparison operators do — they return `True` or `False`."

```python
# Comparison operators
a = 10
b = 20

print(a == b)    # False  — equal to?
print(a != b)    # True   — not equal to?
print(a < b)     # True   — less than?
print(a > b)     # False  — greater than?
print(a <= b)    # True   — less than or equal to?
print(a >= b)    # False  — greater than or equal to?
```

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `==` | Equal to | `5 == 5` | `True` |
| `!=` | Not equal to | `5 != 3` | `True` |
| `<` | Less than | `3 < 5` | `True` |
| `>` | Greater than | `5 > 3` | `True` |
| `<=` | Less than or equal | `5 <= 5` | `True` |
| `>=` | Greater than or equal | `3 >= 5` | `False` |

> ⚠️ **Critical:** `==` checks equality. `=` is assignment. Mixing them up is one of the most common bugs.
```python
x = 5     # ASSIGNS 5 to x
x == 5    # CHECKS if x equals 5 → True
```

**Mike:** "Comparisons work on strings too — alphabetically:"
```python
print("apple" < "banana")    # True (a comes before b)
print("Harvey" == "harvey")  # False (case-sensitive!)
print("A" < "a")             # True (uppercase < lowercase in ASCII)
```

---

### 3.2 The `if` Statement

**Mike:** "The simplest decision — do something **only if** a condition is true."

```python
revenue = 500000

if revenue > 100000:
    print("This is a high-value client")
    print("Assign to a senior partner")
```

**Mike:** "Let me break down the anatomy:"

```python
if revenue > 100000:          # ← condition (must evaluate to True/False)
    print("High-value")       # ← indented block (runs ONLY if condition is True)
    print("Assign senior")    # ← still part of the indented block

print("This ALWAYS runs")     # ← NOT indented — runs regardless
```

**Key rules:**
1. The condition is followed by a **colon** `:`
2. The body is **indented** (4 spaces — Python standard)
3. All indented lines belong to the `if` block
4. The first un-indented line is **outside** the `if` — it always runs

```python
# If the condition is False, the body is SKIPPED entirely
revenue = 50000

if revenue > 100000:
    print("This will NOT print")  # Skipped!

print("This prints no matter what")
```

---

### 3.3 The `if-else` Statement

**Mike:** "Two branches — do one thing if true, another if false."

```python
profit = -15000

if profit >= 0:
    print("✅ Client is profitable")
else:
    print("🔴 Client is unprofitable")
```

```python
# More complete example
revenue = 180000
costs = 220000
profit = revenue - costs

if profit >= 0:
    print(f"Profit: ${profit:,}")
    print("Status: KEEP client")
else:
    print(f"Loss: ${abs(profit):,}")
    print("Status: REVIEW for termination")
```

**Mike:** "The `else` block is the catch-all — it runs when the `if` condition is `False`. No condition needed on `else`."

---

### 3.4 The `if-elif-else` Chain

**Mike:** "Multiple conditions, multiple paths. This is where things get powerful."

```python
revenue = 750000

if revenue >= 1000000:
    tier = "Platinum"
    icon = "💎"
elif revenue >= 500000:
    tier = "Gold"
    icon = "🥇"
elif revenue >= 100000:
    tier = "Silver"
    icon = "🥈"
else:
    tier = "Bronze"
    icon = "🥉"

print(f"{icon} Client Tier: {tier}")
# Output: 🥇 Client Tier: Gold
```

**Mike:** "How it works:"

1. Python checks the `if` condition first
2. If `False`, it checks the first `elif`
3. If still `False`, it checks the next `elif`
4. It keeps going until one is `True` — then runs that block and **skips the rest**
5. If **nothing** is `True`, the `else` block runs

> 💡 **Key Point:** Only **ONE** block in an `if-elif-else` chain will ever execute. The first match wins.

```python
# You can have many elif blocks
score = 85

if score >= 97:
    grade = "A+"
elif score >= 93:
    grade = "A"
elif score >= 90:
    grade = "A-"
elif score >= 87:
    grade = "B+"
elif score >= 83:
    grade = "B"
elif score >= 80:
    grade = "B-"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(f"Score: {score} → Grade: {grade}")  # Score: 85 → Grade: B
```

---

### 3.5 Nested `if` Statements

**Mike:** "You can put `if` statements inside other `if` statements."

```python
client_type = "corporate"
revenue = 800000

if client_type == "corporate":
    print("Corporate client detected")
    
    if revenue >= 1000000:
        print("  → Assign to Harvey Specter")
    elif revenue >= 500000:
        print("  → Assign to Mike Ross")
    else:
        print("  → Assign to associate pool")
        
elif client_type == "individual":
    print("Individual client detected")
    
    if revenue >= 100000:
        print("  → Assign to Louis Litt")
    else:
        print("  → Assign to Rachel Zane")
else:
    print("Unknown client type")
```

**Mike:** "Nesting is powerful but can get hard to read. More than 3 levels deep? Refactor into functions (Lesson 11)."

---

### 3.6 Conditional Expressions (Ternary Operator)

**Mike:** "Python has a one-line `if-else` for simple cases:"

```python
# Standard if-else
profit = 25000
if profit >= 0:
    status = "Profitable"
else:
    status = "Unprofitable"

# Same thing, one line — conditional expression
status = "Profitable" if profit >= 0 else "Unprofitable"
print(status)  # Profitable
```

**Syntax:** `value_if_true if condition else value_if_false`

```python
# More examples
age = 30
category = "Senior" if age >= 40 else "Junior"

score = 92
grade = "Pass" if score >= 60 else "Fail"

items = 0
message = "Cart is empty" if items == 0 else f"{items} items in cart"

# In f-strings (from Lesson 2!)
print(f"Client is {'active' if True else 'inactive'}")
```

**Mike:** "Use ternary for **simple** assignments. For anything complex, use full `if-else` blocks."

---

### 3.7 Truthiness and Falsiness

**Mike:** "Python doesn't just check `True` and `False`. Many values have an inherent 'truth' value."

#### Falsy Values (evaluated as `False`):
```python
if not False:       print("False is falsy")
if not None:        print("None is falsy")
if not 0:           print("0 is falsy")
if not 0.0:         print("0.0 is falsy")
if not "":          print("Empty string is falsy")
if not []:          print("Empty list is falsy")
if not {}:          print("Empty dict is falsy")
if not ():          print("Empty tuple is falsy")
```

#### Truthy Values (evaluated as `True`):
```python
if True:            print("True is truthy")
if 42:              print("Non-zero int is truthy")
if 3.14:            print("Non-zero float is truthy")
if "hello":         print("Non-empty string is truthy")
if [1, 2]:          print("Non-empty list is truthy")
if {"a": 1}:        print("Non-empty dict is truthy")
if (1, 2):          print("Non-empty tuple is truthy")
```

**Mike:** "This is incredibly useful for checking empties:"

```python
# Instead of this (works but verbose):
name = input("Name: ").strip()
if len(name) > 0:
    print(f"Hello, {name}")

# Write this (Pythonic):
name = input("Name: ").strip()
if name:
    print(f"Hello, {name}")
else:
    print("Name cannot be empty!")

# Check for empty collections
clients = []
if clients:
    print(f"Processing {len(clients)} clients")
else:
    print("No clients to process")

# Check for None
result = None
if result is not None:
    print(f"Result: {result}")
else:
    print("No result available")
```

> 💡 **Connection to Lesson 6:** Remember the default value pattern `input("Name [default]: ").strip() or "default"`? That works because empty strings are falsy — `or` returns the second value when the first is falsy.

---

### 3.8 The `is` vs `==` Distinction

**Mike:** "These look similar but are fundamentally different:"

```python
# == checks VALUE equality
print(5 == 5)               # True — same value
print([1, 2] == [1, 2])     # True — same contents

# 'is' checks IDENTITY — are they the SAME OBJECT in memory?
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)    # True  — same values
print(a is b)    # False — different objects!
print(a is c)    # True  — same object (alias)
```

**Mike:** "Rule: Use `is` ONLY for `None`, `True`, and `False`. Use `==` for everything else."

```python
# ✅ Correct
if result is None:
    print("No result")

if flag is True:
    print("Enabled")

# ❌ Wrong
# if name is "Harvey":  # Don't use 'is' for strings or numbers
```

---

### 3.9 Multiple Conditions with `and`, `or`, `not`

**Mike:** "You can combine conditions. We'll go much deeper in Lesson 8, but here's the preview:"

```python
age = 30
income = 85000

# AND — both must be True
if age >= 25 and income >= 50000:
    print("Eligible for premium services")

# OR — at least one must be True
if age < 18 or income < 20000:
    print("Reduced rates apply")

# NOT — reverses the condition
active = False
if not active:
    print("Account is inactive")

# Combining
if (age >= 25 and income >= 50000) or income >= 200000:
    print("Qualified client")
```

---

### 3.10 Solving Harvey's Problem — Client Classification Engine

```python
# ============================================
# Pearson Specter Litt — Client Classification
# ============================================

print("=" * 60)
print(f"{'CLIENT CLASSIFICATION ENGINE':^60}")
print("=" * 60)
print()

# --- Gather client data ---
client_name = input("  Client Name: ").strip().title()
revenue_str = input("  Annual Revenue ($): ").strip().replace(",", "")
costs_str = input("  Annual Costs ($): ").strip().replace(",", "")
case_type = input("  Case Type (Merger/Litigation/Patent/Corporate): ").strip().title()
is_urgent = input("  Urgent? (yes/no): ").strip().lower() == "yes"

# --- Validate and convert ---
if revenue_str.replace(".", "", 1).isdigit() and costs_str.replace(".", "", 1).isdigit():
    revenue = float(revenue_str)
    costs = float(costs_str)
else:
    print("❌ Invalid financial data. Aborting.")
    revenue = 0
    costs = 0

# --- Calculate profitability ---
profit = revenue - costs
margin = (profit / revenue * 100) if revenue > 0 else 0

# --- Determine profit status ---
if margin >= 30:
    profit_flag = "🟢 HIGHLY PROFITABLE"
elif margin >= 10:
    profit_flag = "🟡 MODERATELY PROFITABLE"
elif margin >= 0:
    profit_flag = "🟠 BREAK-EVEN"
else:
    profit_flag = "🔴 UNPROFITABLE"

# --- Determine billing tier ---
if revenue >= 1000000:
    tier = "💎 Platinum"
elif revenue >= 500000:
    tier = "🥇 Gold"
elif revenue >= 100000:
    tier = "🥈 Silver"
else:
    tier = "🥉 Bronze"

# --- Assign attorney ---
if case_type == "Merger" and revenue >= 500000:
    attorney = "Harvey Specter"
elif case_type == "Litigation":
    attorney = "Louis Litt"
elif case_type == "Patent":
    attorney = "Mike Ross"
else:
    if revenue >= 500000:
        attorney = "Harvey Specter"
    elif revenue >= 100000:
        attorney = "Mike Ross"
    else:
        attorney = "Associate Pool"

# --- Priority level ---
if is_urgent and revenue >= 500000:
    priority = "⚡ CRITICAL"
elif is_urgent:
    priority = "🔶 HIGH"
elif revenue >= 500000:
    priority = "🔷 MEDIUM"
else:
    priority = "⬜ STANDARD"

# --- Action needed ---
if margin < 0:
    action = "⚠️ REVIEW: Consider client termination"
elif margin < 10 and revenue < 100000:
    action = "⚠️ REVIEW: Low margin, low revenue — discuss with partner"
elif is_urgent:
    action = "📋 ACTION: Schedule immediate consultation"
else:
    action = "✅ No immediate action required"

# --- Display Results ---
print()
print("=" * 60)
print(f"{'CLASSIFICATION REPORT':^60}")
print("=" * 60)
print()
print(f"  Client:       {client_name}")
print(f"  Revenue:      ${revenue:>14,.2f}")
print(f"  Costs:        ${costs:>14,.2f}")
print(f"  Profit:       ${profit:>14,.2f}")
print(f"  Margin:       {margin:>13.1f}%")
print()
print(f"  Profit Status: {profit_flag}")
print(f"  Billing Tier:  {tier}")
print(f"  Assigned To:   {attorney}")
print(f"  Priority:      {priority}")
print(f"  Case Type:     {case_type}")
print()
print(f"  → {action}")
print()
print("=" * 60)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "Conditional logic is where bugs *thrive*. Let me show you."

---

### Pitfall 1: Indentation Errors — Python's Silent Killer

**Louis:** "In Python, indentation IS the syntax. Get it wrong, and either your code crashes or does something completely different."

```python
# ❌ IndentationError — missing indent
if True:
print("This crashes")     # IndentationError: expected an indented block

# ❌ Logic error — wrong block membership
x = 10
if x > 5:
    print("Greater than 5")
print("This looks like it's in the if-block, but it's NOT")
# The second print ALWAYS runs — it's not indented!

# ❌ Mixed indentation — tabs and spaces
if True:
    print("Four spaces")
	print("One tab")         # IndentationError!
# NEVER mix tabs and spaces. Use 4 spaces. ALWAYS.
```

---

### Pitfall 2: `=` vs `==` — The Classic Bug

**Louis:** "Assignment vs comparison — the most common beginner mistake:"

```python
x = 5

# ❌ This ASSIGNS, doesn't compare!
# if x = 10:     # SyntaxError in Python (thankfully!)
#     print("ten")

# ✅ This COMPARES
if x == 10:
    print("ten")
```

**Louis:** "Python catches this as a SyntaxError, unlike C/Java. Be grateful."

---

### Pitfall 3: Chained Comparisons — Powerful but Surprising

**Louis:** "Python supports chained comparisons, which is unique and elegant — but can surprise you:"

```python
x = 15

# ✅ Python allows this (most languages don't!)
if 10 < x < 20:
    print("x is between 10 and 20")

# This is equivalent to:
if 10 < x and x < 20:
    print("Same thing")

# More chains
if 1 < 2 < 3 < 4:
    print("All true")  # Prints!

# ⚠️ Watch out — this might not do what you think:
if 5 > 3 > 1 > 0:
    print("Descending chain — all true")  # Prints!
```

---

### Pitfall 4: `if-elif` Order Matters

**Louis:** "The order of your conditions can completely change the result:"

```python
score = 95

# ❌ WRONG ORDER — first match wins, and EVERY score > 60 matches first!
if score >= 60:
    grade = "D"
elif score >= 70:
    grade = "C"
elif score >= 80:
    grade = "B"
elif score >= 90:
    grade = "A"

print(grade)  # "D" ← WRONG! 95 matches (score >= 60) first!

# ✅ RIGHT ORDER — check from highest to lowest
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(grade)  # "A" ✅
```

---

### Pitfall 5: Floating-Point Comparison

**Louis:** "Remember Lesson 1? Floats are imprecise:"

```python
result = 0.1 + 0.2

# ❌ This is FALSE!
if result == 0.3:
    print("Equal")     # Never prints!

# ✅ Use tolerance
if abs(result - 0.3) < 1e-9:
    print("Close enough")  # Prints!
```

---

### Pitfall 6: Multiple `if` vs `if-elif`

**Louis:** "These are NOT the same:"

```python
x = 15

# Multiple IF — ALL conditions are checked independently
if x > 10:
    print("Greater than 10")     # Prints ✅
if x > 5:
    print("Greater than 5")      # Prints ✅
if x > 0:
    print("Greater than 0")      # Prints ✅
# ALL THREE print!

# IF-ELIF — only the FIRST match executes
if x > 10:
    print("Greater than 10")     # Prints ✅ (stops here)
elif x > 5:
    print("Greater than 5")      # Skipped!
elif x > 0:
    print("Greater than 0")      # Skipped!
# Only ONE prints!
```

**Louis:** "Multiple `if`s = independent checks. `if-elif` = mutually exclusive choices."

---

### Pitfall 7: The `or` Trap (Revisited from Lesson 6)

**Louis:** "This bug is so common it deserves its own section:"

```python
day = "Saturday"

# ❌ WRONG — "Sunday" is always truthy!
if day == "Saturday" or "Sunday":
    print("Weekend!")  # ALWAYS prints, even for "Monday"!

# ✅ RIGHT
if day == "Saturday" or day == "Sunday":
    print("Weekend!")

# ✅ BEST — use 'in' with a tuple/list
if day in ("Saturday", "Sunday"):
    print("Weekend!")
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Let me make this crystal clear."

---

### 🚦 Analogy: `if` Statements Are Traffic Signals

**Donna:** "Think of your program as a car driving down a straight road."

```
WITHOUT if statements:
  Start → Step 1 → Step 2 → Step 3 → End
  (Straight road — no turns, no choices)

WITH if statements:
  Start → Step 1 →  🚦 DECISION 
                         ↙       ↘
                   [if True]   [else]
                    Step 2A     Step 2B
                         ↘       ↙
                       Step 3 → End
```

- **`if`** = A green light that only turns on when a condition is met. No condition met? You skip that road entirely.
- **`if-else`** = A fork in the road. You MUST go one way or the other. No middle ground.
- **`if-elif-else`** = A traffic roundabout with multiple exits. You take the FIRST exit that matches your direction and ignore all others.

**Donna:** "And **truthiness**? Think of it this way:"

| Value | Analogy | Truthy? |
|-------|---------|---------|
| `True`, `1`, `"hello"`, `[1,2]` | A full filing cabinet | ✅ Truthy |
| `False`, `0`, `""`, `[]`, `None` | An empty filing cabinet | ❌ Falsy |

**Donna:** "If the cabinet has *anything* in it, it counts as 'yes.' If it's completely empty, it's 'no.' That's truthiness."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Decision-making. The bread and butter of programming."

---

### 🟢 Beginner Exercises

**Exercise 1: Legal Drinking Age**
```
Ask the user for their age.
- If 21 or older: "You can attend the firm's cocktail party."
- Otherwise: "Sorry, you must be 21 or older."
```

**Exercise 2: Profit/Loss Checker**
```
Ask for revenue and costs.
Calculate profit.
- If profit is positive: print "Profitable: $X"
- If profit is zero: print "Break-even"
- If profit is negative: print "Loss: $X"
```

**Exercise 3: Grade Calculator**
```
Ask for a numeric score (0-100).
Print the letter grade:
  A+: 97-100, A: 93-96, A-: 90-92
  B+: 87-89, B: 83-86, B-: 80-82
  C+: 77-79, C: 73-76, C-: 70-72
  D: 60-69, F: below 60
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Billing Rate Calculator**
```
Ask for:
- Client type: "individual" or "corporate"
- Case complexity: "simple", "moderate", or "complex"
- Is pro-bono? (yes/no)

If pro-bono: rate = $0
If corporate:
  simple = $300/hr, moderate = $450/hr, complex = $600/hr
If individual:
  simple = $150/hr, moderate = $250/hr, complex = $400/hr

Print the calculated rate with all details.
Handle invalid inputs gracefully.
```

**Exercise 5: FizzBuzz (Classic)**
```
Ask the user for a number.
- If divisible by 3 AND 5: print "FizzBuzz"
- If divisible by 3 only: print "Fizz"
- If divisible by 5 only: print "Buzz"
- Otherwise: print the number

Test with: 15, 9, 10, 7
```

**Exercise 6: Triangle Classifier**
```
Ask for three side lengths of a triangle.
Determine:
1. Is it a valid triangle? (sum of any two sides > third side)
2. If valid, classify it:
   - Equilateral (all sides equal)
   - Isosceles (two sides equal)
   - Scalene (no sides equal)
3. Also check if it's a right triangle (a² + b² = c²)
```

---

### 🔴 Advanced Exercises

**Exercise 7: Tax Calculator**
```
Build a progressive tax calculator:

Income Brackets:
  $0 - $10,000:       10%
  $10,001 - $40,000:  12%
  $40,001 - $85,000:  22%
  $85,001 - $160,000: 24%
  $160,001 - $210,000: 32%
  $210,001+:           35%

IMPORTANT: Tax is PROGRESSIVE — each bracket rate applies
only to the income WITHIN that bracket.

Example: Income of $50,000
  First $10,000 at 10%  = $1,000
  Next $30,000 at 12%   = $3,600
  Next $10,000 at 22%   = $2,200
  Total Tax              = $6,800
  Effective Rate         = 13.6%

Ask for income, calculate total tax, and show the breakdown.
```

**Exercise 8: Case Assignment System**
```
Build an intelligent case assignment system:

Inputs: case_type, estimated_value, complexity (1-5), urgent (yes/no)

Rules:
- Mergers over $1M → Harvey
- Litigation over $500K → Harvey, else → Louis
- Patent cases → Mike
- Pro-bono cases (value = 0) → Rachel
- Urgent complex cases (complexity >= 4 AND urgent) → Harvey regardless
- If assigned attorney is Harvey and case < $100K → reassign to Louis

Print the assignment with reasoning for each decision.
```

**Exercise 9: Leap Year + Day Calculator**
```
Ask for a year and month number (1-12).

1. Determine if the year is a leap year:
   - Divisible by 4? Yes
   - But divisible by 100? Not a leap year
   - But divisible by 400? IS a leap year

2. Print how many days are in the given month of that year
   (February has 28 or 29 depending on leap year)

3. Print the month name

Test with: Feb 2024 (29), Feb 1900 (28), Feb 2000 (29)
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Conditional logic bugs. The hardest to spot."

### Buggy Code 🐛

```python
# Bug Hunt: Client Risk Assessment
# Find and fix ALL the bugs!

revenue = 450000
risk_score = 7
active = True

# Bug 1: Assignment vs comparison
if revenue = 500000:
    print("Exact match")

# Bug 2: Wrong elif order
score = 85
if score >= 60:
    grade = "D"
elif score >= 70:
    grade = "C"
elif score >= 80:
    grade = "B"
elif score >= 90:
    grade = "A"
print(f"Grade: {grade}")

# Bug 3: The 'or' trap
status = "pending"
if status == "active" or "approved":
    print("Client is good to go")

# Bug 4: Missing else for undefined variable
temperature = 25
if temperature > 30:
    message = "Too hot"
elif temperature < 10:
    message = "Too cold"
print(message)             # What if temperature is 25?

# Bug 5: Indentation error
if risk_score > 5:
    print("High risk")
    if active:
    print("Needs review")  # Missing indent

# Bug 6: Comparing different types
user_input = input("Enter 5: ")
if user_input == 5:
    print("Match!")         # Will this ever print?
```

---

### Fixed Code ✅

```python
# Fixed: Client Risk Assessment

revenue = 450000
risk_score = 7
active = True

# Fix 1: Use == for comparison, not =
if revenue == 500000:
    print("Exact match")

# Fix 2: Check highest threshold first
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"
print(f"Grade: {grade}")  # B ✅

# Fix 3: Explicit comparisons on both sides of 'or'
status = "pending"
if status == "active" or status == "approved":
    print("Client is good to go")
# Better: if status in ("active", "approved"):

# Fix 4: Always define the variable in ALL branches
temperature = 25
if temperature > 30:
    message = "Too hot"
elif temperature < 10:
    message = "Too cold"
else:
    message = "Just right"      # ✅ Now 'message' is always defined
print(message)

# Fix 5: Proper indentation
if risk_score > 5:
    print("High risk")
    if active:
        print("Needs review")  # ✅ Indented inside the nested if

# Fix 6: Compare same types — convert input to int
user_input = input("Enter 5: ")
if int(user_input) == 5:       # ✅ Convert string to int
    print("Match!")
# Or compare as strings: if user_input == "5":
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `=` (assign) instead of `==` (compare) | Assignment vs comparison |
| 2 | Wrong order — least restrictive condition first | elif order matters |
| 3 | `or "approved"` always truthy | Boolean evaluation |
| 4 | Variable undefined if no branch matches | Always have else |
| 5 | Body of nested `if` not indented | Indentation syntax |
| 6 | Comparing `str` to `int` — always False | Type matching |

---

## 8. 🌍 Real-World Application

**Harvey:** "Every system you've ever used makes decisions."

### Where `if` Statements Are Used in Production:

| Domain | Decision Example |
|--------|-----------------|
| **Authentication** | If password matches, grant access. Else, deny. |
| **E-Commerce** | If cart > $50, free shipping. Elif member, discounted. Else, standard. |
| **Gaming** | If health <= 0, game over. Elif health < 20, show warning. |
| **APIs** | If status 200, parse response. Elif 404, show "not found." Elif 500, retry. |
| **DevOps** | If CPU > 90%, scale up. Elif < 30%, scale down. |
| **Data Science** | If value is outlier, remove. Elif missing, impute. Else, keep. |
| **Finance** | If balance < 0, overdraft fee. Elif < minimum, warning. |

### Real Production Example: HTTP Status Handler

```python
def handle_response(status_code, body):
    """Handle API response based on status code."""
    if status_code == 200:
        print(f"✅ Success: {body}")
    elif status_code == 201:
        print(f"✅ Created: {body}")
    elif status_code == 400:
        print(f"❌ Bad Request: {body}")
    elif status_code == 401:
        print("🔒 Unauthorized — check your API key")
    elif status_code == 403:
        print("🚫 Forbidden — insufficient permissions")
    elif status_code == 404:
        print("🔍 Not Found — resource doesn't exist")
    elif status_code == 429:
        print("⏳ Rate Limited — slow down requests")
    elif 500 <= status_code < 600:
        print(f"🔥 Server Error ({status_code}) — retry later")
    else:
        print(f"❓ Unknown status: {status_code}")

# Test
handle_response(200, '{"clients": 42}')
handle_response(404, "Client not found")
handle_response(503, "Service unavailable")
```

### Real Production Example: Feature Flags

```python
# Feature flags — controlling what code runs in production
features = {
    "new_dashboard": True,
    "dark_mode": True,
    "ai_assistant": False,
    "beta_reports": False
}

user_role = "admin"

# Show features based on flags AND user role
if features["new_dashboard"]:
    print("📊 Loading new dashboard...")
else:
    print("📊 Loading classic dashboard...")

if features["ai_assistant"] and user_role == "admin":
    print("🤖 AI Assistant enabled")

if features["beta_reports"] or user_role == "admin":
    print("📋 Beta reports available")
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Quick drill."

---

**Q1.** What is the difference between `=` and `==`?

**Q2.** What is the output?
```python
x = 10
if x > 5:
    print("A")
elif x > 8:
    print("B")
else:
    print("C")
```

**Q3.** Which values are "falsy" in Python? Name at least 5.

**Q4.** What's wrong with `if x == "yes" or "y":`? How do you fix it?

**Q5.** Why does the *order* of `elif` conditions matter?

**Q6.** What is a conditional expression (ternary)? Write one that assigns `"Even"` or `"Odd"` based on a number.

**Q7.** What happens if you use `if` without `else`, and the condition is `False`?

**Q8.** When should you use `is` instead of `==`?

**Q9.** What is the output?
```python
if "":
    print("A")
else:
    print("B")
```

**Q10.** Write an `if-elif-else` chain that classifies a temperature as "Freezing" (< 32°F), "Cold" (32-59), "Mild" (60-79), or "Hot" (80+).

---

### 📋 Answer Key

> **Q1.** `=` assigns a value. `==` compares for equality.

> **Q2.** `"A"` — the first condition `x > 5` is True (10 > 5), so it runs and skips all `elif`/`else`.

> **Q3.** `False`, `0`, `0.0`, `""`, `[]`, `{}`, `()`, `None`, `set()`.

> **Q4.** `"y"` is always truthy, so the condition is always True. Fix: `if x == "yes" or x == "y":` or `if x in ("yes", "y"):`.

> **Q5.** Python takes the **first** matching condition. If a less restrictive condition comes first, more specific ones never execute.

> **Q6.** `result = "Even" if num % 2 == 0 else "Odd"`

> **Q7.** Nothing happens — the body is skipped, and execution continues with the next unindented line.

> **Q8.** Only for `None`, `True`, and `False`. E.g., `if x is None:`.

> **Q9.** `"B"` — empty string `""` is falsy.

> **Q10.**
```python
if temp < 32:
    print("Freezing")
elif temp < 60:
    print("Cold")
elif temp < 80:
    print("Mild")
else:
    print("Hot")
```

---

## 🎬 End of Lesson Scene

*It's 5:00 PM. The classification engine is live. Harvey runs three test clients through it.*

**Harvey:** "Wayne Enterprises — Platinum, assigned to me. Correct. Small pro-bono case — Bronze, associate pool. Correct. The unprofitable one — flagged for review. Perfect."

*He turns to you.*

**Harvey:** "Your programs can think now. That's dangerous — in a good way."

**Mike:** "Tomorrow is **Boolean operators**. Today you learned `if`. Tomorrow you learn how to build *complex* conditions — `and`, `or`, `not`, short-circuit evaluation."

**Donna:** "The brain is working. Tomorrow we upgrade the wiring."

**Louis:** *(reviewing the code)* "The elif order was correct from the start. I'm… impressed."

*Day seven. Your programs don't just follow instructions anymore — they make decisions.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | Comparison operators — `==`, `!=`, `<`, `>`, `<=`, `>=` | ✅ |
| 2 | `if` statement — condition + indented block | ✅ |
| 3 | `if-else` — two branches | ✅ |
| 4 | `if-elif-else` — multiple branches | ✅ |
| 5 | Nested `if` statements | ✅ |
| 6 | Conditional expression (ternary) | ✅ |
| 7 | Truthiness and falsiness | ✅ |
| 8 | `is` vs `==` | ✅ |
| 9 | `and`, `or`, `not` (preview) | ✅ |
| 10 | Common pitfalls — order, indentation, `=` vs `==` | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| `input()` + type conversion (Lesson 6) | Interactive conditional programs |
| Truthiness of empty collections (Lessons 3–5) | `if clients:` checks |
| f-strings (Lesson 2) | Formatted decision output |
| `in` operator (Lessons 2–5) | `if x in (...)` membership checks |
| Float precision (Lesson 1) | Float comparison gotcha |
| `or` default pattern (Lesson 6) | Built on truthiness concept |

---

> **Next Lesson →** Level 1, Lesson 8: *Use Boolean Operators in Python*  
> *Harvey's classification engine needs complex multi-condition logic — time to wire up `and`, `or`, and `not`…*

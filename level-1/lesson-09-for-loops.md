# 🏛️ Level 1 – Exploring | Lesson 9: Use `for` Loops in Python

## *"The Automation Assembly Line"*

> **O'Reilly Skill Objective:** *Control program flow with for loops.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, `if`/`elif`/`else`, boolean operators (`and`, `or`, `not`), comparison operators, truthiness, short-circuit evaluation

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

*It's 8:00 AM. You walk into the firm and find chaos. Donna is fielding phone calls. Associates are running between offices. Louis is shouting at no one in particular.*

**Louis:** "TWO HUNDRED AND THIRTY-SEVEN! That's how many client accounts need their quarterly billing statements generated. By FRIDAY!"

**Donna:** *(covering the phone)* "Louis, stop yelling. You're scaring the interns."

*Harvey walks over to your desk, calm as always.*

**Harvey:** "You built a system that generates one billing statement. Impressive. But can it generate 237 statements? Can it process every single client in our database, one by one, without you touching the keyboard?"

**You:** "I'd have to copy the code 237 times—"

**Harvey:** "No. Absolutely not. That's what a first-year associate would say. What you need is a **loop**."

*Mike rolls over on his chair.*

**Mike:** "A loop tells Python: 'Take this block of code, and run it over and over — once for each item in a collection.' One piece of code. 237 client statements. Zero copy-pasting."

**Harvey:** "Today you learn `for` loops. This is where you stop being a script writer and start being an **automation engineer**."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me make this clear."

**Harvey:** "Until today, every program you've written runs from top to bottom, once. You process one client, print one report, make one decision. Manually changing the input each time."

**Harvey:** "In the real world, that's useless. You have:"
- **237 clients** that each need a billing statement
- **500 case files** that each need a status check
- **A million log entries** that each need to be parsed

**Harvey:** "Writing the same code 237 times? That's not engineering — that's stenography."

**Harvey:** "A `for` loop automates repetition. It says: 'For each item in this collection, do this.' One definition, unlimited executions."

| Concept | Without Loops | With Loops |
|---------|--------------|------------|
| Process 5 clients | 5 copies of the same code | 3 lines of code |
| Process 500 clients | 500 copies (absurd) | Same 3 lines of code |
| Process 5 million clients | Impossible | Same 3 lines of code |

**Harvey:** "Loops combined with the conditionals you learned in Lessons 7–8? That's when programs become truly powerful. You can process every client, make a decision about each one, and generate the result — all automatically."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Loops unlock automation. Let's master them."

---

### 3.1 Basic `for` Loop

```python
# The simplest for loop — iterate over a list
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]

for name in associates:
    print(f"Processing: {name}")
```

**Output:**
```
Processing: Mike
Processing: Rachel
Processing: Katrina
Processing: Harold
Processing: Kyle
```

**Mike:** "Here's the anatomy:"

```python
for name in associates:      # ← 'for' keyword, loop variable, 'in' keyword, iterable
    print(f"Processing: {name}")  # ← indented body (runs once per item)
                              # ← body ends when indentation returns to normal level

print("Done!")                # ← NOT indented — runs ONCE, after the loop finishes
```

| Component | What it is |
|-----------|-----------|
| `for` | Keyword that starts the loop |
| `name` | **Loop variable** — holds the current item on each iteration |
| `in` | Keyword that connects variable to collection |
| `associates` | The **iterable** — the collection to loop over |
| `:` | Must end the line with a colon |
| Indented block | The **body** — code that runs once per item |

---

### 3.2 Looping Over Different Data Types

**Mike:** "You can loop over ANY iterable — lists, tuples, strings, dictionaries, ranges."

#### Loop Over a List
```python
hours = [120, 95, 140, 88, 110]
total = 0
for h in hours:
    total += h
print(f"Total hours: {total}")  # 553
```

#### Loop Over a Tuple
```python
partners = ("Harvey", "Jessica", "Louis")
for partner in partners:
    print(f"Partner: {partner}")
```

#### Loop Over a String (Character by Character)
```python
word = "SUITS"
for char in word:
    print(char)
# S, U, I, T, S — each on a new line
```

> 💡 **Connection to Lesson 2:** Remember that strings are sequences of characters? That's why you can loop over them — each character is one iteration.

#### Loop Over a Dictionary
```python
rates = {"Harvey": 450, "Mike": 375, "Louis": 425}

# Looping over a dict gives you the KEYS
for name in rates:
    print(f"{name}: ${rates[name]}")

# Looping over .items() gives you KEY-VALUE pairs
for name, rate in rates.items():
    print(f"{name}: ${rate}")

# Looping over .values() gives you just values
for rate in rates.values():
    print(f"Rate: ${rate}")
```

> 💡 **Connection to Lesson 5:** `dict.items()` returns tuples, and `for name, rate in ...` is **tuple unpacking** from Lesson 4!

---

### 3.3 The `range()` Function

**Mike:** "When you need a loop that runs a specific number of times — or needs numeric indices — you use `range()`."

```python
# range(stop) — 0 to stop-1
for i in range(5):
    print(i)         # 0, 1, 2, 3, 4

# range(start, stop) — start to stop-1
for i in range(1, 6):
    print(i)         # 1, 2, 3, 4, 5

# range(start, stop, step) — with a custom step
for i in range(0, 20, 5):
    print(i)         # 0, 5, 10, 15

# Counting backwards
for i in range(5, 0, -1):
    print(i)         # 5, 4, 3, 2, 1
```

| `range()` Call | Produces | Description |
|---------------|----------|-------------|
| `range(5)` | 0, 1, 2, 3, 4 | Start=0, stop before 5 |
| `range(1, 6)` | 1, 2, 3, 4, 5 | Start=1, stop before 6 |
| `range(0, 10, 2)` | 0, 2, 4, 6, 8 | Even numbers |
| `range(10, 0, -1)` | 10, 9, 8, …, 1 | Countdown |
| `range(5, 5)` | *(empty)* | Start=stop → nothing |

> ⚠️ `range()` stop is **exclusive** — same as slicing. `range(5)` gives 0–4, not 0–5.

#### Using `range()` with Lists
```python
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]

# Index-based iteration
for i in range(len(associates)):
    print(f"  {i + 1}. {associates[i]}")
```

**Output:**
```
  1. Mike
  2. Rachel
  3. Katrina
  4. Harold
  5. Kyle
```

---

### 3.4 `enumerate()` — Index + Value Together ⭐

**Mike:** "This is the **Pythonic** way to get both the index and the value:"

```python
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]

# ❌ Works but NOT Pythonic
for i in range(len(associates)):
    print(f"  {i + 1}. {associates[i]}")

# ✅ Pythonic — use enumerate()
for i, name in enumerate(associates):
    print(f"  {i + 1}. {name}")

# Start counting from 1
for rank, name in enumerate(associates, start=1):
    print(f"  {rank}. {name}")
```

**Mike:** "`enumerate()` returns `(index, value)` tuples. Combined with tuple unpacking — clean, readable, professional."

---

### 3.5 `zip()` — Loop Over Multiple Collections ⭐

**Mike:** "When you have parallel lists and need to process them together:"

```python
names = ["Mike", "Rachel", "Katrina", "Harold"]
hours = [52, 48, 55, 38]
rates = [375.00, 350.00, 400.00, 280.00]

# ❌ Index-based (works but ugly)
for i in range(len(names)):
    revenue = hours[i] * rates[i]
    print(f"{names[i]}: ${revenue:,.2f}")

# ✅ zip() — elegant parallel iteration
for name, h, r in zip(names, hours, rates):
    revenue = h * r
    print(f"{name}: ${revenue:,.2f}")
```

**Output:**
```
Mike: $19,500.00
Rachel: $16,800.00
Katrina: $22,000.00
Harold: $10,640.00
```

**Mike:** "`zip()` pairs up elements from multiple iterables. If they're different lengths, it stops at the **shortest** one."

```python
a = [1, 2, 3, 4, 5]
b = ["a", "b", "c"]

for x, y in zip(a, b):
    print(x, y)
# 1 a, 2 b, 3 c — stops at 3 items (shortest list)
```

---

### 3.6 Loop Control — `break`, `continue`, `else`

#### `break` — Exit the Loop Immediately
```python
# Find the first unprofitable client
clients = [
    ("Wayne", 250000, 180000),
    ("Stark", 120000, 95000),
    ("Oscorp", 500000, 620000),  # Unprofitable!
    ("Luthor", 400000, 300000),
]

for name, revenue, costs in clients:
    profit = revenue - costs
    if profit < 0:
        print(f"🔴 Found unprofitable client: {name} (loss: ${abs(profit):,})")
        break  # Stop searching — we found one
    print(f"✅ {name}: ${profit:,} profit")
```

**Output:**
```
✅ Wayne: $70,000 profit
✅ Stark: $25,000 profit
🔴 Found unprofitable client: Oscorp (loss: $120,000)
```

#### `continue` — Skip to the Next Iteration
```python
# Process only active clients, skip inactive ones
clients = [
    ("Wayne", True),
    ("Stark", False),   # Inactive — skip
    ("Oscorp", True),
    ("Luthor", False),  # Inactive — skip
]

for name, active in clients:
    if not active:
        continue         # Skip this iteration, go to next client
    print(f"Processing: {name}")
```

**Output:**
```
Processing: Wayne
Processing: Oscorp
```

#### `else` on a Loop — Runs Only if Loop Completes Without `break`
```python
# Search for a specific client
target = "Luthor"
clients = ["Wayne", "Stark", "Oscorp"]

for client in clients:
    if client == target:
        print(f"Found {target}!")
        break
else:
    # This runs ONLY if the loop finished WITHOUT hitting 'break'
    print(f"{target} not found in client list")

# Output: Luthor not found in client list
```

**Mike:** "The `for-else` pattern is unique to Python. The `else` block runs if the loop finishes naturally — but NOT if it was terminated by `break`."

---

### 3.7 Nested Loops

```python
# Multiplication table
print("    ", end="")
for j in range(1, 6):
    print(f"{j:>4}", end="")
print()
print("   " + "-" * 20)

for i in range(1, 6):
    print(f"{i:>2} |", end="")
    for j in range(1, 6):
        print(f"{i * j:>4}", end="")
    print()  # New line after each row
```

**Output:**
```
       1   2   3   4   5
   --------------------
 1 |   1   2   3   4   5
 2 |   2   4   6   8  10
 3 |   3   6   9  12  15
 4 |   4   8  12  16  20
 5 |   5  10  15  20  25
```

```python
# Processing nested data — quarterly reports
associates = {
    "Mike": [120, 130, 125],
    "Rachel": [95, 100, 88],
    "Katrina": [140, 135, 150]
}

for name, quarters in associates.items():
    total = 0
    for hours in quarters:
        total += hours
    avg = total / len(quarters)
    print(f"{name}: Total={total}h, Avg={avg:.1f}h/quarter")
```

---

### 3.8 Common Loop Patterns

**Mike:** "These patterns appear in virtually every Python program."

#### Pattern 1: Accumulator (Sum, Count, Max, Min)
```python
scores = [85, 92, 78, 95, 88, 71, 90]

# Sum
total = 0
for score in scores:
    total += score
print(f"Sum: {total}")          # 599

# Count with condition
above_80 = 0
for score in scores:
    if score > 80:
        above_80 += 1
print(f"Above 80: {above_80}")  # 5

# Find maximum
highest = scores[0]
for score in scores:
    if score > highest:
        highest = score
print(f"Highest: {highest}")    # 95
```

#### Pattern 2: Build a New Collection
```python
# Filter — keep only passing scores
scores = [85, 92, 78, 95, 88, 71, 90]
passing = []
for score in scores:
    if score >= 80:
        passing.append(score)
print(passing)  # [85, 92, 95, 88, 90]

# Transform — double every value
doubled = []
for score in scores:
    doubled.append(score * 2)
print(doubled)  # [170, 184, 156, 190, 176, 142, 180]
```

#### Pattern 3: Search
```python
# Find the first item matching a condition
clients = ["Wayne", "Stark", "Oscorp", "Luthor"]
target = "Oscorp"
found_at = -1

for i, client in enumerate(clients):
    if client == target:
        found_at = i
        break

if found_at >= 0:
    print(f"Found '{target}' at index {found_at}")
else:
    print(f"'{target}' not found")
```

#### Pattern 4: Grouping (Building a Dictionary)
```python
# Group cases by attorney
cases = [
    ("Case A", "Harvey"), ("Case B", "Mike"), ("Case C", "Harvey"),
    ("Case D", "Louis"), ("Case E", "Mike"), ("Case F", "Harvey")
]

by_attorney = {}
for case_name, attorney in cases:
    if attorney not in by_attorney:
        by_attorney[attorney] = []
    by_attorney[attorney].append(case_name)

for attorney, case_list in by_attorney.items():
    print(f"{attorney}: {', '.join(case_list)}")
```

**Output:**
```
Harvey: Case A, Case C, Case F
Mike: Case B, Case E
Louis: Case D
```

---

### 3.9 Solving Louis's Problem — Batch Billing Generator

```python
# ============================================
# Pearson Specter Litt — Batch Billing System
# ============================================

clients = [
    {"name": "Wayne Enterprises", "hours": 85, "rate": 450.00, "tier": "Platinum"},
    {"name": "Stark Industries", "hours": 62, "rate": 375.00, "tier": "Gold"},
    {"name": "Oscorp", "hours": 45, "rate": 400.00, "tier": "Gold"},
    {"name": "Luthor Corp", "hours": 28, "rate": 325.00, "tier": "Silver"},
    {"name": "Dalton Industries", "hours": 93, "rate": 450.00, "tier": "Platinum"},
    {"name": "Gillis Industries", "hours": 34, "rate": 280.00, "tier": "Bronze"},
]

tax_rate = 0.085

# -- Report Header --
print("=" * 75)
print(f"{'QUARTERLY BILLING REPORT':^75}")
print(f"{'Pearson Specter Litt':^75}")
print("=" * 75)
print()
print(f"{'#':<3} {'Client':<22} {'Tier':<10} {'Hours':>6} {'Rate':>8} {'Subtotal':>12} {'Tax':>10} {'Total':>12}")
print("-" * 75)

# -- Process each client --
grand_total = 0
total_hours = 0
tier_totals = {}
highest_bill = ("", 0)
lowest_bill = ("", float("inf"))

for i, client in enumerate(clients, start=1):
    subtotal = client["hours"] * client["rate"]
    tax = subtotal * tax_rate
    total = subtotal + tax
    
    # Accumulate stats
    grand_total += total
    total_hours += client["hours"]
    
    # Track tier totals
    tier = client["tier"]
    tier_totals[tier] = tier_totals.get(tier, 0) + total
    
    # Track highest and lowest
    if total > highest_bill[1]:
        highest_bill = (client["name"], total)
    if total < lowest_bill[1]:
        lowest_bill = (client["name"], total)
    
    # Print row
    print(f"{i:<3} {client['name']:<22} {client['tier']:<10} "
          f"{client['hours']:>6} ${client['rate']:>7,.2f} "
          f"${subtotal:>11,.2f} ${tax:>9,.2f} ${total:>11,.2f}")

# -- Summary --
print("-" * 75)
print(f"{'':>51} {'GRAND TOTAL':>12} ${grand_total:>11,.2f}")
print()

print(f"📊 Clients Processed:  {len(clients)}")
print(f"⏱️  Total Hours Billed: {total_hours}")
print(f"💰 Grand Total:        ${grand_total:,.2f}")
print(f"📈 Average Bill:       ${grand_total / len(clients):,.2f}")
print()

print(f"🏆 Highest Bill: {highest_bill[0]} — ${highest_bill[1]:,.2f}")
print(f"📉 Lowest Bill:  {lowest_bill[0]} — ${lowest_bill[1]:,.2f}")
print()

print("Revenue by Tier:")
for tier in sorted(tier_totals, key=tier_totals.get, reverse=True):
    bar = "█" * int(tier_totals[tier] / 1000)
    print(f"  {tier:<10} ${tier_totals[tier]:>12,.2f}  {bar}")

print()
print("=" * 75)
print(f"{'Report generated successfully':^75}")
print("=" * 75)
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "Loops are where performance problems and subtle bugs breed."

---

### Pitfall 1: Modifying a List While Looping Over It 🔥

**Louis:** "We covered this in Lesson 3, but it's worth repeating with loops:"

```python
# ❌ DANGEROUS — unpredictable behavior
numbers = [1, 2, 3, 4, 5, 6]
for num in numbers:
    if num % 2 == 0:
        numbers.remove(num)
print(numbers)    # [1, 3, 5]? Actually might skip elements!

# ✅ SAFE — loop over a copy
numbers = [1, 2, 3, 4, 5, 6]
for num in numbers[:]:    # [:] creates a copy
    if num % 2 == 0:
        numbers.remove(num)
print(numbers)    # [1, 3, 5] ✅

# ✅ BEST — build a new list
numbers = [1, 2, 3, 4, 5, 6]
odds = [num for num in numbers if num % 2 != 0]
print(odds)       # [1, 3, 5] ✅
```

---

### Pitfall 2: Forgetting That `range()` Stop Is Exclusive

```python
# ❌ Off-by-one error
for i in range(5):
    print(i)      # 0, 1, 2, 3, 4  — NOT 0 to 5!

# If you want 1 to 5:
for i in range(1, 6):   # ✅ stop at 6 to include 5
    print(i)      # 1, 2, 3, 4, 5
```

---

### Pitfall 3: Unused Loop Variable

```python
# When you don't need the variable, use _
for _ in range(3):
    print("PSL!")
# PSL!
# PSL!
# PSL!
```

**Louis:** "The `_` convention tells other developers 'I'm not using this variable — it's just a counter.'"

---

### Pitfall 4: Shadowing Variables

**Louis:** "The loop variable persists after the loop ends — and might overwrite important variables:"

```python
name = "Harvey Specter"   # Important variable!

names = ["Mike", "Rachel", "Katrina"]
for name in names:        # 'name' is now the loop variable!
    print(name)

print(f"Original name: {name}")  # "Katrina" — NOT "Harvey Specter"!
```

**Louis:** "The loop variable `name` overwrites the outer `name`. Use distinct names:"
```python
attorney_name = "Harvey Specter"
for associate_name in names:
    print(associate_name)
print(f"Attorney: {attorney_name}")  # ✅ Safe
```

---

### Pitfall 5: Creating a List of Identical References

**Louis:** "Watch this trap with nested lists:"

```python
# ❌ WRONG — all rows are the SAME list!
grid = [[0] * 3] * 3
grid[0][0] = 99
print(grid)   # [[99, 0, 0], [99, 0, 0], [99, 0, 0]]

# ✅ RIGHT — use a loop to create independent rows
grid = []
for _ in range(3):
    grid.append([0] * 3)
grid[0][0] = 99
print(grid)   # [[99, 0, 0], [0, 0, 0], [0, 0, 0]] ✅
```

---

### Pitfall 6: Empty Iterables Don't Error — They Silently Skip

```python
# Loop over an empty list — no error, just does nothing
for item in []:
    print("This never prints")

print("But this does!")

# This can hide bugs if you expect the loop to run
clients = []   # Oops, data loading failed
for client in clients:
    process(client)  # Never runs — no error
# Add a check:
if not clients:
    print("Warning: No clients to process!")
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "I've got a good one for this."

---

### 🔄 Analogy: `for` Loops Are Like an Assembly Line

**Donna:** "Imagine a factory assembly line:"

```
Conveyor Belt: [📦 Client A] [📦 Client B] [📦 Client C] [📦 Client D]
                    ↓             ↓             ↓             ↓
              [🔧 Process]  [🔧 Process]  [🔧 Process]  [🔧 Process]
                    ↓             ↓             ↓             ↓
              [📋 Result A] [📋 Result B] [📋 Result C] [📋 Result D]
```

- **The conveyor belt** = Your list/tuple/dictionary (the iterable)
- **Each box** = One item in the collection
- **The work station** = Your loop body (the indented code)
- **The worker** = The loop variable (holds the current item)

**Donna:** "The worker picks up one box at a time, does the same work on each, and puts it down. Same operation, different item, repeated automatically."

| Loop Concept | Assembly Line |
|-------------|---------------|
| `for item in collection:` | "For each box on the conveyor:" |
| Loop body | "Do this work on it" |
| `break` | Emergency stop button 🛑 |
| `continue` | "This box is defective — skip it, grab the next" |
| `range(5)` | "Run exactly 5 boxes through" |
| `enumerate()` | "Stamp each box with a number AND process it" |
| `zip()` | Two conveyor belts running side by side — process a pair at a time |

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Loops are where you become an automation expert."

---

### 🟢 Beginner Exercises

**Exercise 1: Name Printer**
```
Given: partners = ["Harvey", "Jessica", "Louis", "Daniel", "Robert"]

Loop through and print:
  1. Harvey
  2. Jessica
  3. Louis
  4. Daniel
  5. Robert

Use enumerate() with start=1.
```

**Exercise 2: Sum and Average**
```
Given: billable_hours = [45, 52, 38, 61, 48, 55, 42]

Using a for loop (not the built-in sum()), calculate:
a) Total hours
b) Average hours
c) Number of weeks above 50 hours
d) Highest and lowest week
```

**Exercise 3: Character Counter**
```
Given a string: text = "Pearson Specter Litt"

Using a for loop, count:
a) Total characters (including spaces)
b) Number of uppercase letters
c) Number of lowercase letters
d) Number of spaces

Print the results.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Pattern Printer**
```
Using nested loops, print these patterns:

Pattern A:          Pattern B:          Pattern C:
*                   *****               *****
**                  ****                 ****
***                 ***                   ***
****                **                     **
*****               *                       *
```

**Exercise 5: Parallel List Processor**
```
Given parallel lists:
  names = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]
  hours = [52, 48, 55, 38, 45]
  rates = [375, 350, 400, 280, 325]

Using zip():
a) Calculate revenue for each associate
b) Print a formatted table with name, hours, rate, revenue
c) Find who earned the most revenue
d) Calculate total firm revenue
e) Print a bar chart using '#' characters 
   (scale: 1 '#' per $1000)
```

**Exercise 6: Word Frequency Counter**
```
Given: text = "the quick brown fox jumps over the lazy dog the fox the dog"

Using a for loop:
a) Build a dictionary counting each word's frequency
b) Print words sorted by frequency (highest first)
c) Find the most and least common words
d) Print words that appear only once
```

---

### 🔴 Advanced Exercises

**Exercise 7: Prime Number Finder**
```
Find all prime numbers between 1 and 100.

A number is prime if it's greater than 1 and 
only divisible by 1 and itself.

Hints:
- For each number n (2 to 100)
- Check if any number from 2 to √n divides it evenly
- Use a for-else: the else runs if no divisor was found (it's prime)
- Optimization: you only need to check up to √n (use int(n**0.5) + 1)

Print all primes and count them.
```

**Exercise 8: Data Transformer**
```
Given raw sales data as strings:
raw_data = [
    "2026-01-15,Wayne Enterprises,45000.00",
    "2026-01-22,Stark Industries,32000.00",
    "2026-02-01,Oscorp,18500.00",
    "2026-02-10,Wayne Enterprises,52000.00",
    "2026-02-18,Luthor Corp,27500.00",
    "2026-03-01,Wayne Enterprises,61000.00",
    "2026-03-05,Stark Industries,44000.00",
]

Using loops:
a) Parse each line into a dictionary with keys: date, client, amount
b) Store all parsed records in a list
c) Calculate total sales per client
d) Calculate total sales per month
e) Find the client with the highest total sales
f) Print a summary report
```

**Exercise 9: Matrix Operations**
```
Given two matrices (lists of lists):

matrix_a = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

matrix_b = [
    [9, 8, 7],
    [6, 5, 4],
    [3, 2, 1]
]

Using nested loops:
a) Add the matrices (element-wise)
b) Transpose matrix_a (rows become columns)
c) Multiply by a scalar (multiply every element by 3)
d) Find the sum of the main diagonal
e) Flatten into a single list
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Loop bugs."

### Buggy Code 🐛

```python
# Bug Hunt: Billing Report
# Find and fix ALL the bugs!

clients = ["Wayne", "Stark", "Oscorp"]
hours = [85, 62, 45]

# Bug 1: Wrong range
for i in range(1, len(clients)):
    print(clients[i])
# Expected: Wayne, Stark, Oscorp

# Bug 2: IndexError
for i in range(len(clients) + 1):
    print(clients[i])

# Bug 3: Infinite-ish issue
total = 0
for h in hours:
    total = h     # Is this accumulating?
print(f"Total: {total}")

# Bug 4: Variable scope
for client in clients:
    result = client.upper()
print(f"All caps: {result}")
# Expected: All three names in caps

# Bug 5: Unpacking error
data = [("Wayne", 85), ("Stark", 62), ("Oscorp", 45)]
for name in data:
    print(f"{name}: {hours} hours")

# Bug 6: enumerate misuse
for i, name in enumerate(clients):
    print(f"Client {i}: {name}")
# Expected: Client 1, Client 2, Client 3
```

---

### Fixed Code ✅

```python
# Fixed: Billing Report

clients = ["Wayne", "Stark", "Oscorp"]
hours = [85, 62, 45]

# Fix 1: range starts at 0, not 1
for i in range(len(clients)):   # ✅ 0 to 2
    print(clients[i])
# Better: just loop directly: for client in clients:

# Fix 2: range should go to len, not len+1
for i in range(len(clients)):   # ✅ 0 to 2, not 0 to 3
    print(clients[i])

# Fix 3: Use += to accumulate, not = which replaces
total = 0
for h in hours:
    total += h    # ✅ Add to total, don't replace it
print(f"Total: {total}")  # 192

# Fix 4: 'result' is overwritten each iteration — only keeps last value
# To get ALL names in caps, build a list
all_caps = []
for client in clients:
    all_caps.append(client.upper())
print(f"All caps: {all_caps}")  # ['WAYNE', 'STARK', 'OSCORP']

# Fix 5: Unpack the tuple in the loop variable
data = [("Wayne", 85), ("Stark", 62), ("Oscorp", 45)]
for name, h in data:           # ✅ Unpack tuple
    print(f"{name}: {h} hours")

# Fix 6: enumerate starts at 0 — use start=1
for i, name in enumerate(clients, start=1):   # ✅
    print(f"Client {i}: {name}")
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `range(1, n)` skips index 0 | range start value |
| 2 | `range(len+1)` goes one past the end | range stop is exclusive |
| 3 | `total = h` replaces instead of accumulating | `=` vs `+=` |
| 4 | Loop variable only holds last iteration value | Build a list instead |
| 5 | Not unpacking tuple in loop | Tuple unpacking in for |
| 6 | `enumerate` starts at 0 by default | Use `start=1` |

---

## 8. 🌍 Real-World Application

**Harvey:** "Loops power every data pipeline, every automation, every system at scale."

### Where `for` Loops Are Used in Production:

| Domain | Loop Use Case |
|--------|--------------|
| **Data Processing** | Iterate through CSV/JSON rows, transform each record |
| **Web Scraping** | Loop through pages, extract data from each |
| **APIs** | Process each item in a paginated response |
| **Testing** | Run the same test with multiple inputs (parameterized tests) |
| **DevOps** | Deploy to each server, check each container |
| **Machine Learning** | Training epochs, batch processing, feature engineering |
| **File Processing** | Read each line, parse each file in a directory |

### Real Production Example: Log File Analyzer

```python
# Analyzing server logs
logs = [
    "[09:15:22] INFO: User login successful | user=harvey",
    "[09:16:45] ERROR: Database timeout | server=db-01",
    "[09:17:01] INFO: File uploaded | user=mike",
    "[09:18:30] WARNING: Disk space low | server=web-01",
    "[09:19:15] ERROR: Authentication failed | user=unknown",
    "[09:20:00] INFO: Report generated | user=donna",
    "[09:21:12] ERROR: Connection refused | server=api-02",
]

# Count by severity
severity_counts = {"INFO": 0, "WARNING": 0, "ERROR": 0}
errors = []

for log in logs:
    for level in severity_counts:
        if level in log:
            severity_counts[level] += 1
            if level == "ERROR":
                errors.append(log)
            break

print("Log Summary:")
for level, count in severity_counts.items():
    bar = "█" * (count * 5)
    print(f"  {level:<10} {count}  {bar}")

print(f"\n⚠️ Errors ({len(errors)}):")
for error in errors:
    print(f"  {error}")
```

### Real Production Example: Batch API Processor

```python
# Processing paginated API results
def fetch_page(page_num):
    """Simulated API call — returns a page of clients."""
    all_clients = [
        "Wayne", "Stark", "Oscorp", "Luthor", "Dalton",
        "Gillis", "Palmer", "Queen", "Allen", "Kent"
    ]
    page_size = 3
    start = page_num * page_size
    end = start + page_size
    return all_clients[start:end]

# Process all pages
page = 0
all_results = []

while True:
    results = fetch_page(page)
    if not results:
        break
    
    for client in results:
        print(f"  Processing: {client}")
        all_results.append(client.upper())
    
    page += 1

print(f"\nTotal processed: {len(all_results)}")
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Loop quiz."

---

**Q1.** What does `range(3, 8)` produce? What about `range(10, 0, -2)`?

**Q2.** What is the difference between `for i in range(len(list))` and `for item in list`? When do you need the index?

**Q3.** What does `enumerate()` do? What does it return on each iteration?

**Q4.** What does `zip()` do with lists of different lengths?

**Q5.** What is the output?
```python
for i in range(5):
    if i == 3:
        break
    print(i)
```

**Q6.** What is `for-else`? When does the `else` block execute?

**Q7.** Why is modifying a list while iterating over it dangerous? What are two safe alternatives?

**Q8.** What is the output?
```python
for i in range(3):
    for j in range(2):
        print(f"({i},{j})", end=" ")
    print()
```

**Q9.** What does the `_` convention mean in `for _ in range(5)`?

**Q10.** Write a loop that prints the sum of all even numbers from 1 to 100.

---

### 📋 Answer Key

> **Q1.** `range(3, 8)` → 3, 4, 5, 6, 7. `range(10, 0, -2)` → 10, 8, 6, 4, 2.

> **Q2.** `range(len(list))` gives you indices (numbers). Direct iteration gives you values. Use index-based only when you NEED the index position.

> **Q3.** Returns `(index, value)` tuples. Each iteration gives both the position and the item.

> **Q4.** Stops at the **shortest** iterable — extra elements from longer ones are ignored.

> **Q5.** `0`, `1`, `2` — `break` exits when `i == 3`, before printing.

> **Q6.** The `else` runs only if the loop completes **without** `break`. If `break` fires, `else` is skipped.

> **Q7.** The iterator loses track of positions. Safe: (1) Loop over a copy `list[:]`. (2) Build a new filtered list.

> **Q8.** `(0,0) (0,1)` / `(1,0) (1,1)` / `(2,0) (2,1)` — inner loop completes for each outer iteration.

> **Q9.** The loop variable is intentionally unused — "I need to repeat N times but don't care about the count."

> **Q10.**
```python
total = 0
for n in range(2, 101, 2):
    total += n
print(total)  # 2550
```

---

## 🎬 End of Lesson Scene

*It's 4:45 PM. The batch billing system finishes processing all 237 clients. Louis watches the terminal scroll through every statement — perfectly formatted.*

**Louis:** *(quietly)* "Every. Single. One. Correct."

*He turns to you.*

**Louis:** "I'll admit it. This is… efficient."

**Harvey:** "Tomorrow: `while` loops. `for` loops iterate over collections. `while` loops run until a condition changes. It's the difference between 'process each client' and 'keep processing until we're done.'"

**Mike:** "Today your programs can repeat. Tomorrow they'll loop **indefinitely** — until they decide to stop."

**Donna:** "Just don't create an infinite loop. I'm not rebooting another server."

*Day nine. Your programs can now automate any repetitive task across any size dataset. The assembly line is running.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | `for` loop — iterate over any iterable | ✅ |
| 2 | Loop over lists, tuples, strings, dicts | ✅ |
| 3 | `range()` — `range(stop)`, `range(start, stop)`, `range(start, stop, step)` | ✅ |
| 4 | `enumerate()` — index + value together | ✅ |
| 5 | `zip()` — parallel iteration | ✅ |
| 6 | `break` — exit loop early | ✅ |
| 7 | `continue` — skip to next iteration | ✅ |
| 8 | `for-else` — runs if no break | ✅ |
| 9 | Nested loops | ✅ |
| 10 | Common patterns — accumulator, filter, search, group | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Lists, tuples (Lessons 3–4) | Primary iterables for loops |
| Dictionary `.items()` (Lesson 5) | Dict iteration with unpacking |
| Tuple unpacking (Lesson 4) | `for name, rate in zip(...)` |
| `if` statements (Lesson 7) | Conditionals inside loops |
| Boolean operators (Lesson 8) | Complex conditions in loops |
| `append()` (Lesson 3) | Building collections in loops |
| `.get()` dict pattern (Lesson 5) | Grouping pattern with loops |

---

> **Next Lesson →** Level 1, Lesson 10: *Use While Loops in Python*  
> *The firm's server monitor needs to run 24/7 — checking, waiting, checking again. Time for indefinite loops…*

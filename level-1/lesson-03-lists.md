# 🏛️ Level 1 – Exploring | Lesson 3: Work With Lists in Python

## *"The Client Database Crisis"*

> **O'Reilly Skill Objective:** *Create, modify, and retrieve data from lists.*

> **Previously Learned:** Variables (`int`, `float`, `str`), arithmetic operators, string methods, f-strings, `print()`, indexing, slicing, immutability

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

*It's 7:58 AM. You ride the elevator up to the 50th floor. Before the doors even open, you hear Louis's voice echoing down the hallway.*

**Louis:** "This is UNACCEPTABLE!"

*You step out and see Louis standing at Donna's desk, waving a stack of printouts.*

**Louis:** "Donna, I have been tracking every associate's billable hours for the last quarter — BY HAND. Forty-seven associates. Thirteen weeks. Do you know how many numbers that is?"

**Donna:** *(not looking up)* "Six hundred and eleven. You counted wrong on week seven."

**Louis:** "That's— wait, how did you—"

**Donna:** "I know everything that happens in this firm, Louis."

*Harvey walks out of his office.*

**Harvey:** "What's the noise?"

**Louis:** "I need a way to track all 47 associates and their hours. In a spreadsheet, one wrong sort and the entire thing is ruined."

**Harvey:** *(looking at you)* "You. Day three. You're going to solve Louis's problem."

**You:** "What does he need?"

**Harvey:** "A system that can store a *collection* of data — names, hours, figures — and let you add, remove, sort, search, and filter. Yesterday you handled individual strings and numbers. Today? You handle **groups** of them."

*He looks at Mike's desk.*

**Harvey:** "Mike! Lists. Teach them."

*Mike rolls his chair over.*

**Mike:** "Today's the day everything you've learned starts to multiply. Ready?"

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me frame this for you."

**Harvey:** "So far, you've been working with **individual** pieces of data — one number, one string, one variable at a time. That's fine for a single client. But this firm has hundreds of clients, thousands of cases, and millions of data points."

**Harvey:** "You need a way to store *collections* of related data. That's what a **list** is — an **ordered, changeable collection** that can hold any type of data."

**Harvey:** "In the real world:"
- A **contact list** on your phone — ordered collection of names
- A **to-do list** — ordered tasks you can add, remove, reorder
- A **playlist** — ordered songs you can shuffle, insert, delete
- A **database query result** — rows of data to process one by one

**Harvey:** "Without lists, you'd need a separate variable for every single item. Imagine creating `client_1`, `client_2`, `client_3`… up to `client_500`. That's not engineering — that's insanity."

**Harvey:** "Lists are the first **data structure** you'll learn. Master them, and you can handle any amount of data."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Alright. Lists are where Python goes from a calculator to a real programming language. Let's go deep."

---

### 3.1 Creating Lists

```python
# Empty list
empty = []
also_empty = list()

# List of strings — our associates
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]

# List of numbers — billable hours
hours = [120, 95, 140, 88, 110]

# List of floats — hourly rates
rates = [375.50, 350.00, 400.75, 280.00, 325.00]

# Mixed types — Python allows this!
case_info = ["PSL-2026-001", "Harvey Specter", 250000.00, True]

# Nested lists — lists inside lists
quarterly_hours = [
    [120, 130, 125],   # Q1, Q2, Q3 for associate 1
    [95, 100, 88],     # Q1, Q2, Q3 for associate 2
    [140, 135, 150]    # Q1, Q2, Q3 for associate 3
]
```

**Mike:** "Lists use **square brackets** `[]`. Items are separated by **commas**. Unlike strings, lists can hold **any type** of data — even other lists."

---

### 3.2 List Indexing (Same Rules as Strings!)

**Mike:** "Remember indexing from Lesson 2? Lists use the **exact same system** — zero-based, positive and negative."

```python
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]
#               0        1         2          3        4     ← positive
#              -5       -4        -3         -2       -1     ← negative

# Access by index
print(associates[0])     # Mike (first)
print(associates[2])     # Katrina (third)
print(associates[-1])    # Kyle (last)
print(associates[-2])    # Harold (second to last)

# Length
print(len(associates))   # 5
```

> 💡 **Connection to Lesson 2:** Strings and lists share indexing, slicing, `len()`, and `in` operator — because they're both **sequences**. The skills transfer directly!

---

### 3.3 List Slicing (Same Rules as Strings!)

```python
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]

# Slice: [start:stop] — stop is EXCLUSIVE
print(associates[1:3])     # ['Rachel', 'Katrina']
print(associates[:3])      # ['Mike', 'Rachel', 'Katrina']
print(associates[2:])      # ['Katrina', 'Harold', 'Kyle']
print(associates[-2:])     # ['Harold', 'Kyle']

# Step
print(associates[::2])     # ['Mike', 'Katrina', 'Kyle'] (every 2nd)

# Reverse
print(associates[::-1])    # ['Kyle', 'Harold', 'Katrina', 'Rachel', 'Mike']
```

**Mike:** "Slicing a list always returns a **new list**. The original is untouched."

---

### 3.4 Lists Are MUTABLE (The Big Difference from Strings!)

**Mike:** "This is the **critical** difference. Strings are **immutable** — you can't change them in-place. Lists are **mutable** — you CAN."

```python
# With strings (Lesson 2) — this CRASHED:
# name = "Harvey"
# name[0] = "h"  # TypeError!

# With lists — this WORKS:
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]
associates[3] = "Alex"   # Replace Harold with Alex
print(associates)         # ['Mike', 'Rachel', 'Katrina', 'Alex', 'Kyle']

# Replace a slice
associates[1:3] = ["Jessica", "Donna"]
print(associates)         # ['Mike', 'Jessica', 'Donna', 'Alex', 'Kyle']
```

**Mike:** "Mutability is powerful, but it comes with gotchas — Louis will tear into those later."

---

### 3.5 Adding Elements

```python
team = ["Harvey", "Mike"]

# append() — adds ONE item to the END
team.append("Donna")
print(team)              # ['Harvey', 'Mike', 'Donna']

# insert() — adds ONE item at a SPECIFIC position
team.insert(1, "Jessica")
print(team)              # ['Harvey', 'Jessica', 'Mike', 'Donna']

# extend() — adds ALL items from another iterable
new_members = ["Louis", "Rachel"]
team.extend(new_members)
print(team)              # ['Harvey', 'Jessica', 'Mike', 'Donna', 'Louis', 'Rachel']

# Shorthand for extend: +=
team += ["Katrina"]
print(team)              # ['Harvey', 'Jessica', 'Mike', 'Donna', 'Louis', 'Rachel', 'Katrina']
```

> ⚠️ **Common Mistake — `append()` vs `extend()`:**
```python
list_a = [1, 2, 3]

# append adds the LIST ITSELF as a single element
list_a.append([4, 5])
print(list_a)       # [1, 2, 3, [4, 5]]  ← nested list!

# extend adds each ELEMENT individually
list_b = [1, 2, 3]
list_b.extend([4, 5])
print(list_b)       # [1, 2, 3, 4, 5]    ← flat list!
```

---

### 3.6 Removing Elements

```python
team = ["Harvey", "Jessica", "Mike", "Donna", "Louis", "Rachel"]

# remove() — removes the FIRST occurrence by VALUE
team.remove("Jessica")
print(team)              # ['Harvey', 'Mike', 'Donna', 'Louis', 'Rachel']

# pop() — removes by INDEX and RETURNS the removed item
fired = team.pop(3)      # Remove index 3 (Louis)
print(fired)             # Louis
print(team)              # ['Harvey', 'Mike', 'Donna', 'Rachel']

# pop() with no argument — removes the LAST item
last = team.pop()
print(last)              # Rachel
print(team)              # ['Harvey', 'Mike', 'Donna']

# del — removes by index or slice (doesn't return anything)
del team[1]
print(team)              # ['Harvey', 'Donna']

# clear() — removes EVERYTHING
team.clear()
print(team)              # []
```

#### Removal Cheat Sheet

| Method | Removes by | Returns removed item? | Error if not found? |
|--------|-----------|----------------------|-------------------|
| `remove(value)` | Value | No | Yes — `ValueError` |
| `pop(index)` | Index | Yes | Yes — `IndexError` |
| `pop()` | Last item | Yes | Yes — `IndexError` (if empty) |
| `del list[i]` | Index | No | Yes — `IndexError` |
| `clear()` | Everything | No | No |

---

### 3.7 Searching and Checking

```python
associates = ["Mike", "Rachel", "Katrina", "Harold", "Kyle"]

# Check membership with 'in'
print("Mike" in associates)        # True
print("Harvey" in associates)      # False
print("Harvey" not in associates)  # True

# Find the index of an element
print(associates.index("Katrina"))  # 2
# associates.index("Harvey")       # ValueError! Not in list

# Count occurrences
hours = [8, 8, 10, 8, 12, 10, 8]
print(hours.count(8))              # 4
print(hours.count(12))             # 1
```

---

### 3.8 Sorting and Reversing

```python
# sort() — sorts IN-PLACE (modifies the original list)
hours = [120, 95, 140, 88, 110]
hours.sort()
print(hours)               # [88, 95, 110, 120, 140]

# Sort descending
hours.sort(reverse=True)
print(hours)               # [140, 120, 110, 95, 88]

# Sort strings alphabetically
names = ["Rachel", "Mike", "Harvey", "Donna"]
names.sort()
print(names)               # ['Donna', 'Harvey', 'Mike', 'Rachel']

# sorted() — returns a NEW sorted list (original unchanged)
original = [30, 10, 50, 20, 40]
ordered = sorted(original)
print(original)            # [30, 10, 50, 20, 40] ← unchanged!
print(ordered)             # [10, 20, 30, 40, 50] ← new list

# reverse() — reverses IN-PLACE
names = ["Harvey", "Mike", "Donna"]
names.reverse()
print(names)               # ['Donna', 'Mike', 'Harvey']

# reversed() — returns iterator (original unchanged)
original = [1, 2, 3]
rev = list(reversed(original))
print(rev)                 # [3, 2, 1]
print(original)            # [1, 2, 3] ← unchanged
```

| Operation | Modifies original? | Returns |
|-----------|-------------------|---------|
| `list.sort()` | ✅ Yes | `None` |
| `sorted(list)` | ❌ No | New sorted list |
| `list.reverse()` | ✅ Yes | `None` |
| `reversed(list)` | ❌ No | Iterator |

> ⚠️ **Critical trap:** `sort()` returns `None`, NOT the sorted list!
```python
# WRONG — this destroys your data
result = [3, 1, 2].sort()
print(result)   # None ← oops!

# RIGHT
data = [3, 1, 2]
data.sort()
print(data)     # [1, 2, 3]

# Or use sorted() for a one-liner
result = sorted([3, 1, 2])
print(result)   # [1, 2, 3]
```

---

### 3.9 List Operations and Built-in Functions

```python
# Concatenation
team_a = ["Harvey", "Mike"]
team_b = ["Louis", "Donna"]
all_team = team_a + team_b
print(all_team)            # ['Harvey', 'Mike', 'Louis', 'Donna']

# Repetition
template = [0] * 5
print(template)            # [0, 0, 0, 0, 0]

# Numeric built-ins
hours = [120, 95, 140, 88, 110]
print(len(hours))          # 5
print(sum(hours))          # 553
print(min(hours))          # 88
print(max(hours))          # 140
print(sum(hours) / len(hours))  # 110.6 (average)
```

---

### 3.10 Nested Lists (2D Lists)

**Mike:** "Remember those quarterly hours? Let's work with a 2D list — a list of lists."

```python
# Each inner list = one associate's quarterly hours
# Columns: Q1, Q2, Q3
quarterly_hours = [
    [120, 130, 125],   # Mike
    [95, 100, 88],     # Rachel
    [140, 135, 150],   # Katrina
    [88, 92, 78],      # Harold
    [110, 115, 120]    # Kyle
]

# Access: outer_list[row][column]
print(quarterly_hours[0])       # [120, 130, 125] — Mike's full row
print(quarterly_hours[0][1])    # 130 — Mike's Q2 hours
print(quarterly_hours[2][2])    # 150 — Katrina's Q3 hours

# Modify a specific cell
quarterly_hours[3][2] = 85      # Fix Harold's Q3
print(quarterly_hours[3])       # [88, 92, 85]

# Get all Q1 hours (first column)
q1_hours = [row[0] for row in quarterly_hours]
print(q1_hours)                 # [120, 95, 140, 88, 110]
```

**Mike:** "Don't worry about that `[row[0] for row in ...]` syntax yet — that's a **list comprehension**, which you'll master in Level 2. For now, just know nested lists let you represent **tables** of data."

---

### 3.11 Converting Between Lists and Strings

**Mike:** "Lists and strings talk to each other constantly."

```python
# String → List: split()
csv_line = "Harvey,Mike,Louis,Donna,Rachel"
names = csv_line.split(",")
print(names)          # ['Harvey', 'Mike', 'Louis', 'Donna', 'Rachel']

# List → String: join()
names = ["Harvey", "Mike", "Louis"]
result = " | ".join(names)
print(result)         # Harvey | Mike | Louis

# String → List of characters
word = "Python"
letters = list(word)
print(letters)        # ['P', 'y', 't', 'h', 'o', 'n']

# List of characters → String
letters = ['P', 'y', 't', 'h', 'o', 'n']
word = "".join(letters)
print(word)           # Python
```

> 💡 **Connection to Lesson 2:** `split()` and `join()` are your bridges between strings and lists. You learned about them in the string lesson — now you see them from the list side.

---

### 3.12 Copying Lists

**Mike:** "This is important. Let me show you the RIGHT and WRONG way to copy a list."

```python
original = [1, 2, 3, 4, 5]

# WRONG — this creates an ALIAS, not a copy!
alias = original
alias.append(6)
print(original)    # [1, 2, 3, 4, 5, 6] ← ORIGINAL CHANGED!

# RIGHT — make a true COPY
copy_1 = original.copy()        # Method 1: .copy()
copy_2 = original[:]            # Method 2: slice
copy_3 = list(original)         # Method 3: list() constructor

copy_1.append(99)
print(original)    # [1, 2, 3, 4, 5, 6] ← SAFE, unchanged
print(copy_1)      # [1, 2, 3, 4, 5, 6, 99]
```

**Mike:** "For nested lists, you need a **deep copy** — but we'll cover that in Level 2 when you study mutability."

---

### 3.13 Solving Louis's Problem

**Mike:** "Let's build the associate tracking system Louis needs."

```python
# ============================================
# Pearson Specter Litt — Associate Tracker
# ============================================

# Associate data
associates = ["Mike Ross", "Rachel Zane", "Katrina Bennett", "Harold Gunderson", "Kyle Durant"]
weekly_hours = [52, 48, 55, 38, 45]
hourly_rates = [375.00, 350.00, 400.00, 280.00, 325.00]

# --- Calculate revenue for each associate ---
revenues = []
for i in range(len(associates)):
    rev = weekly_hours[i] * hourly_rates[i]
    revenues.append(rev)

# --- Display the report ---
print("=" * 60)
print(f"{'ASSOCIATE PERFORMANCE REPORT':^60}")
print("=" * 60)
print()
print(f"{'Name':<22} {'Hours':>6} {'Rate':>10} {'Revenue':>14}")
print("-" * 60)

for i in range(len(associates)):
    print(f"{associates[i]:<22} {weekly_hours[i]:>6} {hourly_rates[i]:>10,.2f} {revenues[i]:>14,.2f}")

print("-" * 60)

# --- Summary statistics ---
total_hours = sum(weekly_hours)
total_revenue = sum(revenues)
avg_hours = total_hours / len(associates)
avg_revenue = total_revenue / len(associates)

print(f"{'TOTALS':<22} {total_hours:>6} {'':>10} {total_revenue:>14,.2f}")
print(f"{'AVERAGES':<22} {avg_hours:>6.1f} {'':>10} {avg_revenue:>14,.2f}")
print()

# --- Find top and bottom performers ---
max_revenue = max(revenues)
min_revenue = min(revenues)
top_index = revenues.index(max_revenue)
bottom_index = revenues.index(min_revenue)

print(f"🏆 Top Performer:    {associates[top_index]} — ${max_revenue:,.2f}")
print(f"⚠️  Needs Attention: {associates[bottom_index]} — ${min_revenue:,.2f}")

# --- Sort associates by revenue (descending) ---
# Create pairs, sort, and display
print()
print("Ranked by Revenue:")
paired = list(zip(associates, revenues))
paired.sort(key=lambda x: x[1], reverse=True)

rank = 1
for name, rev in paired:
    print(f"  {rank}. {name:<22} ${rev:>12,.2f}")
    rank += 1

print("=" * 60)
```

**Expected Output:**
```
============================================================
              ASSOCIATE PERFORMANCE REPORT                
============================================================

Name                    Hours       Rate        Revenue
------------------------------------------------------------
Mike Ross                   52     375.00      19,500.00
Rachel Zane                 48     350.00      16,800.00
Katrina Bennett             55     400.00      22,000.00
Harold Gunderson            38     280.00      10,640.00
Kyle Durant                 45     325.00      14,625.00
------------------------------------------------------------
TOTALS                     238                 83,565.00
AVERAGES                  47.6                 16,713.00

🏆 Top Performer:    Katrina Bennett — $22,000.00
⚠️  Needs Attention: Harold Gunderson — $10,640.00

Ranked by Revenue:
  1. Katrina Bennett        $   22,000.00
  2. Mike Ross              $   19,500.00
  3. Rachel Zane            $   16,800.00
  4. Kyle Durant            $   14,625.00
  5. Harold Gunderson       $   10,640.00
============================================================
```

**Mike:** "Don't worry about `for`, `range`, `zip`, or `lambda` right now — those are coming in later lessons. Focus on how the **list** operations work: `append`, `sum`, `max`, `min`, `index`, `sort`."

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

*Louis has been watching over your shoulder the entire time.*

**Louis:** "Not bad. But let me show you every way this can explode."

---

### Pitfall 1: IndexError — The #1 List Bug

**Louis:** "Access an index that doesn't exist? Boom."

```python
items = ["a", "b", "c"]

print(items[3])    # IndexError: list index out of range
# Valid indices: 0, 1, 2 (or -1, -2, -3)

# SAFE: Check first
if len(items) > 3:
    print(items[3])
else:
    print("Index out of range!")
```

**Louis:** "But here's the interesting thing — **slicing** never raises an error:"

```python
items = ["a", "b", "c"]
print(items[5:10])    # [] ← empty list, NO error!
print(items[1:100])   # ['b', 'c'] ← just gives what's available
```

---

### Pitfall 2: The Alias Trap (Reference vs Copy) 🔥

**Louis:** "This is the #1 source of impossible-to-find bugs:"

```python
# You THINK you have two separate lists...
original = [1, 2, 3]
copy = original       # This is NOT a copy! It's an ALIAS!

copy.append(4)
print(original)       # [1, 2, 3, 4] ← WHAT?! Original changed!

# WHY: Both variables point to the SAME list object in memory
print(id(original))   # 140234567890
print(id(copy))       # 140234567890 ← SAME memory address!
```

```python
# The FIX: Use .copy(), [:], or list()
original = [1, 2, 3]
safe_copy = original.copy()

safe_copy.append(4)
print(original)       # [1, 2, 3] ← SAFE!
print(safe_copy)      # [1, 2, 3, 4]
```

---

### Pitfall 3: Mutating a List While Iterating 🔥

**Louis:** "Never modify a list while you're looping through it:"

```python
# DANGEROUS — skips elements!
numbers = [1, 2, 3, 4, 5, 6]
for num in numbers:
    if num % 2 == 0:
        numbers.remove(num)
print(numbers)    # [1, 3, 5]? WRONG! It's [1, 3, 5] by luck,
                  # but this approach is UNRELIABLE

# SAFE — iterate over a COPY, modify the original
numbers = [1, 2, 3, 4, 5, 6]
for num in numbers.copy():  # or numbers[:]
    if num % 2 == 0:
        numbers.remove(num)
print(numbers)    # [1, 3, 5] ✅

# BEST — create a new list with only what you want
numbers = [1, 2, 3, 4, 5, 6]
odds = [num for num in numbers if num % 2 != 0]
print(odds)       # [1, 3, 5] ✅
```

---

### Pitfall 4: `sort()` Returns `None`

**Louis:** "I see this *every* day:"

```python
# WRONG — sort() modifies in-place and returns NOTHING
sorted_list = [3, 1, 2].sort()
print(sorted_list)    # None ← your data is GONE

# RIGHT — use sorted() if you want a new list
sorted_list = sorted([3, 1, 2])
print(sorted_list)    # [1, 2, 3]

# Or sort in place deliberately
data = [3, 1, 2]
data.sort()
print(data)           # [1, 2, 3]
```

---

### Pitfall 5: `[0] * n` Trap for Nested Lists

**Louis:** "This one is *insidious*:"

```python
# Creates 3 INDEPENDENT zeros — FINE
row = [0] * 3
print(row)          # [0, 0, 0]

# Creates 3 references to THE SAME inner list — DANGEROUS!
grid = [[0] * 3] * 3
print(grid)         # [[0, 0, 0], [0, 0, 0], [0, 0, 0]] — looks fine...

grid[0][0] = 99
print(grid)         # [[99, 0, 0], [99, 0, 0], [99, 0, 0]] — ALL rows changed!

# FIX — use a loop or comprehension
grid = [[0] * 3 for _ in range(3)]
grid[0][0] = 99
print(grid)         # [[99, 0, 0], [0, 0, 0], [0, 0, 0]] ✅
```

---

### Pitfall 6: Comparing Lists

**Louis:** "List comparison checks **element by element**, in order:"

```python
print([1, 2, 3] == [1, 2, 3])    # True  — same elements, same order
print([1, 2, 3] == [3, 2, 1])    # False — same elements, different order!
print([1, 2, 3] == [1, 2])       # False — different lengths

# To check same elements regardless of order:
print(sorted([1, 2, 3]) == sorted([3, 2, 1]))  # True
```

---

## 5. 🧠 Mental Model — Donna Paulsen

*Donna walks over, coffee in hand.*

**Donna:** "Let me make this simple."

---

### 🗄️ Analogy: Lists Are Like a Filing Cabinet Drawer

**Donna:** "Imagine a **filing cabinet drawer** with numbered slots."

```
Filing Drawer: associates
┌──────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ Slot │    0     │    1     │    2     │    3     │    4     │
├──────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ File │  Mike    │ Rachel   │ Katrina  │ Harold   │  Kyle    │
└──────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

- **Creating a list** = Getting a new drawer and filing folders into it
- **Indexing** (`drawer[2]`) = Pulling out the file from slot 2 → Katrina
- **Slicing** (`drawer[1:3]`) = Photocopying files from slots 1–2 into a new drawer
- **Appending** = Adding a new file at the end
- **Inserting** = Squeezing a new file between existing ones (everyone after shifts right)
- **Removing** = Pulling a file out (everyone after shifts left)
- **Sorting** = Rearranging all the files in alphabetical/numerical order

**Donna:** "Now here's the key difference from strings:"

| | Strings (Lesson 2) | Lists (Today) |
|---|---|---|
| Analogy | Charm bracelet | Filing cabinet |
| Mutable? | ❌ No — charms are welded | ✅ Yes — files slide in and out |
| Change items? | Can't replace a charm | Can swap files freely |
| `s[0] = 'x'` | ❌ TypeError | ✅ Works fine |

**Donna:** "And the **alias trap**? Think of it this way: when you write `copy = original`, you're not photocopying the drawer. You're writing the *same drawer's label* on two sticky notes. Both sticky notes point to the same physical drawer. Move a file? Both 'see' the change."

**Donna:** "To make a real copy, you need `original.copy()` — that's the actual photocopy machine."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Time to get your hands dirty. These build on what you already know."

---

### 🟢 Beginner Exercises

**Exercise 1: List Basics**
```
Create a list called 'partners' with these names: 
Harvey, Jessica, Louis, Daniel

a) Print the list
b) Print the length of the list
c) Print the first and last elements
d) Check if "Harvey" is in the list
e) Check if "Mike" is in the list
```

**Exercise 2: Modifying Lists**
```
Start with: associates = ["Mike", "Rachel", "Harold"]

a) Add "Katrina" to the end
b) Insert "Kyle" at index 1
c) Remove "Harold"
d) Replace "Rachel" with "Jessica"
e) Print the final list
```

**Exercise 3: List Math**
```
Given: scores = [85, 92, 78, 95, 88, 71, 90]

Calculate and print:
a) Total number of scores
b) Sum of all scores
c) Average score
d) Highest and lowest scores
e) Range (highest - lowest)
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Grade Tracker**
```
You have these test scores: [85, 92, 78, 95, 88, 71, 90, 83, 97, 65]

a) Sort the scores from highest to lowest (keep original unchanged)
b) Find the top 3 scores using slicing
c) Find the bottom 3 scores using slicing
d) Remove the lowest score (drop the worst grade)
e) Calculate the new average after dropping the lowest
f) Count how many scores are above 85
```

**Exercise 5: Merging Client Lists**
```
Two departments have separate client lists:

corporate = ["Acme Corp", "Wayne Enterprises", "Stark Industries", "Acme Corp"]
litigation = ["Luthor Corp", "Wayne Enterprises", "Oscorp", "Stark Industries"]

a) Combine both lists into one (may have duplicates)
b) Create a list with ONLY the unique clients (no duplicates)
   Hint: Think about how to check "if not already in list"
c) Find clients that appear in BOTH lists
d) Sort the unique list alphabetically
```

**Exercise 6: Matrix Operations**
```
Given this 3×3 grid representing case priority scores:

grid = [
    [8, 3, 5],
    [2, 9, 1],
    [7, 4, 6]
]

a) Print the element at row 1, column 2 (should be 1)
b) Print the entire second row
c) Print all elements in the first column (one per line)
d) Calculate the sum of each row
e) Find the maximum value in the entire grid
f) Calculate the sum of the main diagonal (8 + 9 + 6)
```

---

### 🔴 Advanced Exercises

**Exercise 7: Stack Implementation**
```
Implement a simple "undo" stack for document changes:

A stack is LIFO (Last In, First Out) — like a stack of papers.
- Push (add) = append to the end
- Pop (remove) = remove from the end
- Peek = look at the last item without removing

Start with an empty stack. Perform these operations in order:
1. Push "Draft v1"
2. Push "Added clause 3"
3. Push "Fixed typo in clause 2"
4. Peek at the top (print it)
5. Pop (undo last change — print what was undone)
6. Push "Added signature block"
7. Print the full stack
8. Pop all items one by one, printing each
```

**Exercise 8: Inventory Manager**
```
Build a simple inventory system using parallel lists:

items = ["Laptop", "Monitor", "Keyboard", "Mouse", "Headset"]
quantities = [15, 8, 25, 30, 12]
prices = [1200.00, 450.00, 75.00, 25.00, 85.00]

a) Calculate the total inventory value (sum of qty × price for each item)
b) Find the most and least expensive items
c) Find which items have quantity below 10 (low stock alert)
d) Add a new item: "Webcam", quantity 20, price $65.00
e) Remove "Mouse" and its corresponding quantity and price
f) Sort all three lists by price (descending), keeping them aligned
   Hint: Use zip() to pair them, sort, then unzip
```

**Exercise 9: Flatten and Reshape**
```
Given a nested list:
nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]

a) Flatten it into a single list: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
   (use a loop — no imports)
b) Reshape the flat list into a 3×4 grid (3 rows, 4 columns)
c) Reshape it into a 6×2 grid
d) Reverse each inner list of the original nested list
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Bug hunt, round 3. This one's tricky."

### Buggy Code 🐛

```python
# Bug Hunt: Team Manager
# Find and fix ALL the bugs!

# Bug 1: Create a team
team = ("Harvey", "Mike", "Donna")   # Hmm, are those the right brackets?

# Bug 2: Add a new member
team.append["Louis"]                  # Something's off with the syntax

# Bug 3: Remove a member
team.remove("Jessica")               # Will this work?

# Bug 4: Copy the team
backup = team
backup.append("Rachel")
print(f"Original team: {team}")       # Expected: no Rachel here...

# Bug 5: Sort the team
sorted_team = team.sort()
print(f"Sorted: {sorted_team}")       # Expected: alphabetical list

# Bug 6: Access last element
last = team[len(team)]               # Off by how much?

# Bug 7: Combine teams
team_a = ["Harvey", "Mike"]
team_b = ["Louis", "Donna"]
combined = team_a.extend(team_b)
print(f"Combined: {combined}")        # Expected: all four names
```

---

### Fixed Code ✅

```python
# Fixed: Team Manager

# Fix 1: Use SQUARE BRACKETS for lists, not parentheses (that's a tuple!)
team = ["Harvey", "Mike", "Donna"]

# Fix 2: append uses PARENTHESES, not square brackets
team.append("Louis")
print(f"After append: {team}")  # ['Harvey', 'Mike', 'Donna', 'Louis']

# Fix 3: Can't remove "Jessica" — she's not in the list!
# Check first, or remove someone who IS in the list
if "Jessica" in team:
    team.remove("Jessica")
else:
    print("Jessica is not in the team")

# Fix 4: Use .copy() to make a TRUE copy, not an alias
backup = team.copy()
backup.append("Rachel")
print(f"Original team: {team}")    # ['Harvey', 'Mike', 'Donna', 'Louis']
print(f"Backup team:   {backup}")  # ['Harvey', 'Mike', 'Donna', 'Louis', 'Rachel']

# Fix 5: sort() returns None — it sorts IN-PLACE
team.sort()
print(f"Sorted: {team}")          # ['Donna', 'Harvey', 'Louis', 'Mike']
# Or use sorted() to get a new list:
# sorted_team = sorted(team)

# Fix 6: Last valid index is len-1 (zero-based indexing!)
last = team[len(team) - 1]
print(f"Last: {last}")            # Mike
# Better: use negative indexing
last = team[-1]
print(f"Last: {last}")            # Mike

# Fix 7: extend() modifies in-place and returns None
team_a = ["Harvey", "Mike"]
team_b = ["Louis", "Donna"]
team_a.extend(team_b)             # Modifies team_a directly
print(f"Combined: {team_a}")      # ['Harvey', 'Mike', 'Louis', 'Donna']
# Or use + for a new list:
# combined = team_a + team_b
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `()` makes a tuple, not a list | `[]` for lists |
| 2 | `append[]` — wrong brackets for method call | `append()` with parentheses |
| 3 | Removing non-existent item → `ValueError` | Check with `in` first |
| 4 | `backup = team` is an alias, not a copy | Use `.copy()` |
| 5 | `sort()` returns `None` | In-place vs. return value |
| 6 | `team[len(team)]` is out of bounds | Zero-based indexing |
| 7 | `extend()` returns `None` | In-place modification |

---

## 8. 🌍 Real-World Application

**Harvey:** "Lists are the backbone of every system you'll ever build."

### Where Lists Are Used in Production Software:

| Domain | List Use Case |
|--------|--------------|
| **Web APIs** | JSON arrays → Python lists; paginated results |
| **Data Science** | DataFrames are built on lists; feature vectors |
| **Machine Learning** | Training data batches, prediction arrays, model layers |
| **Game Dev** | Inventory systems, enemy lists, tile maps |
| **DevOps** | Server lists, container IDs, log entries |
| **Backend** | Message queues, task queues, event streams |
| **Databases** | Query results, batch inserts, migration lists |

### Real Production Example: Task Queue

```python
# Simplified task queue — used in web backends everywhere
task_queue = []

# Add tasks
task_queue.append({"id": 1, "type": "email", "to": "client@acme.com"})
task_queue.append({"id": 2, "type": "report", "client": "Wayne Enterprises"})
task_queue.append({"id": 3, "type": "backup", "database": "cases_db"})

print(f"Tasks pending: {len(task_queue)}")  # 3

# Process tasks (FIFO — First In, First Out)
while len(task_queue) > 0:
    current_task = task_queue.pop(0)  # Remove from front
    print(f"Processing task {current_task['id']}: {current_task['type']}")

print(f"Tasks remaining: {len(task_queue)}")  # 0
```

### Real Production Example: CSV Data Processor

```python
# Processing CSV-like data
raw_data = """Mike Ross,52,375.00
Rachel Zane,48,350.00
Katrina Bennett,55,400.00
Harold Gunderson,38,280.00"""

# Split into rows, then split each row into fields
rows = raw_data.strip().split("\n")
for row in rows:
    fields = row.split(",")
    name = fields[0]
    hours = int(fields[1])
    rate = float(fields[2])
    revenue = hours * rate
    print(f"{name:<22} {hours}h × ${rate:.2f} = ${revenue:,.2f}")
```

**Harvey:** "Every web request returns a list. Every database query returns a list. Every file you read becomes a list of lines. Lists aren't just a data structure — they're **the** data structure."

---

## 9. ✅ Knowledge Check

**Harvey:** "Final quiz. Get these right."

---

**Q1.** What is the difference between `append()` and `extend()`? What happens if you `append` a list to a list?

**Q2.** What does `["a", "b", "c"][1:1]` return? Why?

**Q3.** Why does `copy = original` NOT create an independent copy of a list? How do you fix it?

**Q4.** What is the output of `[1, 2, 3] + [4, 5]`? What about `[0] * 4`?

**Q5.** What is the difference between `list.sort()` and `sorted(list)`?

**Q6.** How do you safely check if an element exists before removing it from a list?

**Q7.** What is the output?
```python
a = [1, 2, 3]
b = a
a.append(4)
print(b)
```

**Q8.** What is the difference between `del list[i]`, `list.pop(i)`, and `list.remove(x)`?

**Q9.** Given `grid = [[1, 2], [3, 4]]`, how do you access the number `4`?

**Q10.** Why is mutating a list while iterating over it dangerous? What's the safe alternative?

---

### 📋 Answer Key

> **Q1.** `append(x)` adds `x` as a single element. `extend(iterable)` adds each element individually. `my_list.append([4,5])` → `[..., [4, 5]]` (nested). `my_list.extend([4,5])` → `[..., 4, 5]` (flat).

> **Q2.** `[]` — empty list. The slice `[1:1]` is a zero-width range (start = stop).

> **Q3.** `copy = original` makes both variables point to the **same object in memory**. Fix: `copy = original.copy()` or `copy = original[:]`.

> **Q4.** `[1, 2, 3, 4, 5]` (concatenation). `[0, 0, 0, 0]` (repetition).

> **Q5.** `sort()` modifies the list in-place and returns `None`. `sorted()` returns a new sorted list and leaves the original unchanged.

> **Q6.** `if x in my_list: my_list.remove(x)` — check membership with `in` first.

> **Q7.** `[1, 2, 3, 4]` — `b` is an alias for `a`, so both see the change.

> **Q8.** `del list[i]`: removes by index, no return. `pop(i)`: removes by index, returns the removed value. `remove(x)`: removes first occurrence by value, no return.

> **Q9.** `grid[1][1]` — row 1, column 1 (zero-indexed).

> **Q10.** The iterator's internal index gets out of sync with the changing list, causing skipped elements. Safe alternatives: iterate over a copy (`for x in list.copy():`), or build a new filtered list.

---

## 🎬 End of Lesson Scene

*It's 5:30 PM. Louis reviews the associate tracker on his screen. His eyes widen.*

**Louis:** "This… this is actually organized. Every associate, ranked, with revenue calculated."

*He turns to you.*

**Louis:** "I'll admit — you're not terrible. But don't let it go to your head."

**Harvey:** *(from his office doorway)* "Tomorrow you're learning **tuples**. Sometimes you *don't* want data to change."

**Mike:** "Tuples are like lists that went to law school — strict, orderly, and immutable."

**Donna:** "I already feel an analogy coming. See you at 8."

*Day three. The filing cabinet is full, sorted, and searchable. You're starting to think like a developer.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | List creation — `[]`, `list()`, mixed types, nested | ✅ |
| 2 | Indexing — zero-based, positive/negative | ✅ |
| 3 | Slicing — `[start:stop:step]` | ✅ |
| 4 | Mutability — direct item assignment, slice assignment | ✅ |
| 5 | Adding — `append()`, `insert()`, `extend()`, `+=` | ✅ |
| 6 | Removing — `remove()`, `pop()`, `del`, `clear()` | ✅ |
| 7 | Searching — `in`, `index()`, `count()` | ✅ |
| 8 | Sorting — `sort()`, `sorted()`, `reverse()`, `reversed()` | ✅ |
| 9 | Built-ins — `len()`, `sum()`, `min()`, `max()` | ✅ |
| 10 | Copying — alias trap, `.copy()`, `[:]` | ✅ |
| 11 | Nested lists (2D) — access, modify | ✅ |
| 12 | String ↔ List conversion — `split()`, `join()`, `list()` | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Indexing & slicing (Lesson 2) | Same `[start:stop:step]` syntax for lists |
| `split()` & `join()` (Lesson 2) | Bridge between strings and lists |
| `str()`, `type()` (Lesson 1) | Type checking list elements |
| f-strings & formatting (Lesson 2) | Formatted report output |
| Arithmetic operators (Lesson 1) | `sum()`, average calculations |

---

> **Next Lesson →** Level 1, Lesson 4: *Work With Tuples in Python*  
> *Harvey needs immutable case records that no associate can accidentally modify…*

# 🏛️ Level 1 – Exploring | Lesson 4: Work With Tuples in Python

## *"The Sealed Record"*

> **O'Reilly Skill Objective:** *Create, modify, and retrieve data from tuples.*

> **Previously Learned:** Variables, strings (immutable), lists (mutable), indexing, slicing, `in`, `len()`, `append()`, `sort()`, f-strings

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

*It's 8:05 AM. You walk toward your desk, but Rachel intercepts you in the hallway, looking serious.*

**Rachel:** "We have a problem. The Hessington Oil trial — all the case records from discovery got entered into the system yesterday. Dates, exhibit numbers, witness depositions. This morning, someone on the associate team accidentally *changed* one of the records. A deposition date was overwritten."

**You:** "Can't we just fix it?"

**Rachel:** "That's the problem. We don't know which record was changed. We don't know what the original value was. The opposing counsel could argue we tampered with evidence."

*Harvey walks over.*

**Harvey:** "This is exactly what I was afraid of. Yesterday you built a system using lists. Lists are great — but they have a fatal flaw for certain types of data."

**You:** "They can be changed?"

**Harvey:** "Exactly. Lists are **mutable**. Anyone can modify them at any time. For case records, financial data, configuration settings, court dates — you need data that is **locked down**. Data that, once created, **cannot be changed**."

*He looks at you.*

**Harvey:** "That's a **tuple**."

**Mike:** *(walking in with coffee)* "Think of it this way — a list is a whiteboard. A tuple is a stone tablet."

**Harvey:** "Today, you learn when to use the whiteboard and when to carve in stone."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Let me tell you why immutability isn't a limitation — it's a **feature**."

**Harvey:** "In this firm, certain documents are **sealed**. Once a court record is filed, once a deposition is entered, once a contract is signed — it **cannot be altered**. Any modification would be tampering with evidence."

**Harvey:** "Tuples enforce this exact principle in your code. They give you three guarantees:"

1. **Data Integrity** — No one can accidentally modify critical data
2. **Performance** — Python can optimize immutable data (tuples are faster than lists)
3. **Safety** — Tuples can be used as dictionary keys and in sets (lists cannot)

**Harvey:** "Here's the rule of thumb:"

| Use a **List** when... | Use a **Tuple** when... |
|----------------------|----------------------|
| Data needs to change | Data should never change |
| You're building a collection dynamically | You're recording a fixed set of values |
| Order matters AND items may be added/removed | Order matters AND items are permanent |
| Shopping cart, task queue, user input buffer | Coordinates, RGB colors, database records |

**Harvey:** "Lists are verbs — they *do* things. Tuples are nouns — they *are* things."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Tuples are simpler than lists, and that's by design. Let's cover everything."

---

### 3.1 Creating Tuples

```python
# Parentheses (standard way)
case_record = ("PSL-2026-001", "March 13, 2026", "Harvey Specter")

# Without parentheses — the COMMA makes it a tuple, not the parentheses!
coordinates = 40.7128, -74.0060
print(type(coordinates))  # <class 'tuple'>

# Single-element tuple — MUST have a trailing comma!
single = ("Harvey",)       # ✅ This is a tuple
not_a_tuple = ("Harvey")   # ❌ This is just a STRING in parentheses!

print(type(single))        # <class 'tuple'>
print(type(not_a_tuple))   # <class 'str'>

# Empty tuple
empty = ()
also_empty = tuple()

# Using the tuple() constructor
from_list = tuple([1, 2, 3])
print(from_list)           # (1, 2, 3)

from_string = tuple("SUITS")
print(from_string)         # ('S', 'U', 'I', 'T', 'S')
```

> 🚨 **Critical:** The **comma** creates the tuple, NOT the parentheses. `(42)` is just the number `42`. `(42,)` is a tuple. This is the #1 tuple gotcha!

---

### 3.2 Accessing Tuple Elements (Same as Lists!)

**Mike:** "Good news — **indexing and slicing work exactly the same as lists and strings**. Your skills transfer directly."

```python
case = ("PSL-2026-001", "Harvey Specter", "Merger", 15750000.00, True)
#         0                  1               2          3          4

# Indexing
print(case[0])       # PSL-2026-001
print(case[1])       # Harvey Specter
print(case[-1])      # True

# Slicing
print(case[1:3])     # ('Harvey Specter', 'Merger')
print(case[:2])      # ('PSL-2026-001', 'Harvey Specter')
print(case[::2])     # ('PSL-2026-001', 'Merger', True)

# Length
print(len(case))     # 5

# Membership
print("Merger" in case)      # True
print("Litigation" in case)  # False
```

> 💡 **Connection to Lessons 2 & 3:** Strings, lists, and tuples all share indexing (`[0]`), slicing (`[1:3]`), `len()`, and `in` — because they're all **sequences**.

---

### 3.3 Tuples Are IMMUTABLE (Like Strings, Unlike Lists)

**Mike:** "Here's the defining characteristic. Once a tuple is created, it **cannot** be changed."

```python
case = ("PSL-2026-001", "Harvey Specter", "Merger")

# ❌ Cannot modify elements
case[1] = "Mike Ross"       # TypeError: 'tuple' object does not support item assignment

# ❌ Cannot add elements
case.append("New item")     # AttributeError: 'tuple' object has no attribute 'append'

# ❌ Cannot remove elements
del case[0]                 # TypeError: 'tuple' object doesn't support item deletion

# ❌ Cannot sort in-place
case.sort()                 # AttributeError: 'tuple' object has no attribute 'sort'
```

**Mike:** "If you need to 'change' a tuple, you create a **new** tuple:"

```python
original = ("Harvey", "Mike", "Louis")

# Create a new tuple with modifications
updated = original[:1] + ("Jessica",) + original[2:]
print(updated)     # ('Harvey', 'Jessica', 'Louis')

# Original is UNTOUCHED
print(original)    # ('Harvey', 'Mike', 'Louis')
```

---

### 3.4 Tuple Packing and Unpacking ⭐

**Mike:** "This is where tuples really shine — **unpacking**. It's one of Python's most elegant features."

#### Packing — Creating a tuple from values
```python
# Tuple packing (values are "packed" into a tuple)
case_info = "PSL-2026-001", "Harvey Specter", 250000.00
print(case_info)       # ('PSL-2026-001', 'Harvey Specter', 250000.0)
print(type(case_info)) # <class 'tuple'>
```

#### Unpacking — Extracting values from a tuple
```python
# Tuple unpacking (values are "unpacked" into variables)
case_id, attorney, amount = case_info
print(case_id)     # PSL-2026-001
print(attorney)    # Harvey Specter
print(amount)      # 250000.0
```

#### Swapping Variables (The Classic Tuple Trick!)
```python
# Without tuples (traditional way)
a = "Harvey"
b = "Mike"
temp = a
a = b
b = temp

# With tuple unpacking (Pythonic way) — ONE LINE!
a = "Harvey"
b = "Mike"
a, b = b, a         # Swap!
print(a)             # Mike
print(b)             # Harvey
```

**Mike:** "Under the hood, Python creates a temporary tuple `(b, a)`, then unpacks it into `a` and `b`. Beautiful."

#### Unpacking with `*` (Extended Unpacking)
```python
scores = (95, 88, 76, 91, 83, 70, 98)

# Get first, last, and everything in between
first, *middle, last = scores
print(first)     # 95
print(middle)    # [88, 76, 91, 83, 70] — always a LIST
print(last)      # 98

# Get just the first two, rest goes to a list
top1, top2, *rest = scores
print(top1)      # 95
print(top2)      # 88
print(rest)      # [76, 91, 83, 70, 98]

# Ignore the middle
first, *_, last = scores
print(first)     # 95
print(last)      # 98
# _ is Python convention for "I don't care about this value"
```

---

### 3.5 Tuple Methods (Only Two!)

**Mike:** "Since tuples are immutable, they have very few methods — just two:"

```python
records = ("Harvey", "Mike", "Harvey", "Louis", "Harvey", "Donna")

# count() — how many times a value appears
print(records.count("Harvey"))   # 3
print(records.count("Rachel"))   # 0

# index() — position of first occurrence
print(records.index("Louis"))    # 3
print(records.index("Harvey"))   # 0 (first occurrence)
# records.index("Jessica")      # ValueError — not found
```

**Mike:** "Compare that to lists, which have 11 methods. Tuples are intentionally minimal."

| List Methods | Tuple Methods |
|-------------|--------------|
| `append()`, `insert()`, `extend()` | ❌ None (can't add) |
| `remove()`, `pop()`, `clear()` | ❌ None (can't remove) |
| `sort()`, `reverse()` | ❌ None (can't reorder) |
| `copy()` | ❌ Not needed (immutable) |
| `count()`, `index()` | `count()`, `index()` ✅ |

---

### 3.6 Converting Between Tuples and Lists

```python
# Tuple → List (to modify)
original_tuple = ("Harvey", "Mike", "Louis")
temp_list = list(original_tuple)
temp_list.append("Donna")
updated_tuple = tuple(temp_list)
print(updated_tuple)   # ('Harvey', 'Mike', 'Louis', 'Donna')

# List → Tuple (to protect)
mutable_data = [95, 88, 76, 91]
frozen_data = tuple(mutable_data)
print(frozen_data)     # (95, 88, 76, 91)
```

**Mike:** "Pattern: convert to list → modify → convert back to tuple. You'll see this when you need a one-time modification of immutable data."

---

### 3.7 Tuples as Dictionary Keys

**Mike:** "This is a major practical advantage of tuples over lists."

```python
# ✅ Tuples CAN be dictionary keys (because they're immutable/hashable)
office_locations = {
    ("New York", "Floor 50"): "Harvey Specter",
    ("New York", "Floor 48"): "Louis Litt",
    ("Boston", "Floor 12"): "Dana Scott"
}

print(office_locations[("New York", "Floor 50")])  # Harvey Specter

# ❌ Lists CANNOT be dictionary keys
# offices = {["New York", "Floor 50"]: "Harvey"}  # TypeError: unhashable type: 'list'
```

**Mike:** "We'll dive deep into dictionaries in the next lesson. For now, just remember: **tuples can be keys, lists cannot**."

---

### 3.8 Tuples in Functions (Multiple Return Values)

**Mike:** "One of the most common uses of tuples — returning multiple values from a function."

```python
def analyze_case(hours, rate):
    """Calculate billing details for a case."""
    revenue = hours * rate
    tax = revenue * 0.085
    total = revenue + tax
    return revenue, tax, total    # Returns a TUPLE!

# The function returns a tuple
result = analyze_case(85, 375.00)
print(result)           # (31875.0, 2709.375, 34584.375)
print(type(result))     # <class 'tuple'>

# Unpack directly into variables
revenue, tax, total = analyze_case(85, 375.00)
print(f"Revenue: ${revenue:,.2f}")   # Revenue: $31,875.00
print(f"Tax:     ${tax:,.2f}")       # Tax:     $2,709.38
print(f"Total:   ${total:,.2f}")     # Total:   $34,584.38
```

**Mike:** "Almost every Python function that returns multiple values uses tuples. You've been using them without knowing it."

```python
# divmod() from Lesson 1 — returns a tuple!
result = divmod(17, 5)
print(result)           # (3, 2)
quotient, remainder = divmod(17, 5)
print(quotient)         # 3
print(remainder)        # 2
```

---

### 3.9 Named Tuples (Preview)

**Mike:** "Regular tuples use numeric indices, which can be confusing. **Named tuples** give each position a name:"

```python
from collections import namedtuple

# Define a named tuple type
CaseRecord = namedtuple("CaseRecord", ["case_id", "attorney", "client", "amount"])

# Create instances
case1 = CaseRecord("PSL-2026-001", "Harvey Specter", "Wayne Enterprises", 250000.00)
case2 = CaseRecord("PSL-2026-002", "Mike Ross", "Stark Industries", 175000.00)

# Access by NAME (readable) or INDEX (fast)
print(case1.attorney)    # Harvey Specter
print(case1[1])          # Harvey Specter (same thing)
print(case2.amount)      # 175000.0

# Still immutable!
# case1.amount = 300000  # AttributeError: can't set attribute

# Great for readable code
print(f"Case {case1.case_id}: {case1.client} — ${case1.amount:,.2f}")
# Case PSL-2026-001: Wayne Enterprises — $250,000.00
```

**Mike:** "Named tuples are like a lightweight class with no methods. Perfect for structured records."

---

### 3.10 Solving the Evidence Protection Problem

**Mike:** "Let's solve the case record problem that started this lesson."

```python
# ============================================
# Pearson Specter Litt — Sealed Case Records
# ============================================
from collections import namedtuple

# Define the record structure
CaseRecord = namedtuple("CaseRecord", [
    "case_id", "date_filed", "attorney", 
    "client", "case_type", "amount", "sealed"
])

# Create sealed records (tuples — cannot be modified!)
records = (
    CaseRecord("PSL-2026-001", "2026-01-15", "Harvey Specter", 
               "Wayne Enterprises", "Merger", 15750000.00, True),
    CaseRecord("PSL-2026-002", "2026-02-03", "Mike Ross", 
               "Stark Industries", "Patent", 4200000.00, True),
    CaseRecord("PSL-2026-003", "2026-02-28", "Louis Litt", 
               "Oscorp", "Litigation", 8900000.00, True),
    CaseRecord("PSL-2026-004", "2026-03-10", "Harvey Specter", 
               "Dalton Industries", "Merger", 22000000.00, False),
)

# --- Display all records ---
print("=" * 75)
print(f"{'SEALED CASE RECORDS':^75}")
print("=" * 75)
print()
print(f"{'ID':<16} {'Date':<12} {'Attorney':<18} {'Type':<12} {'Amount':>14} {'Sealed':>6}")
print("-" * 75)

for record in records:
    seal_icon = "🔒" if record.sealed else "📝"
    print(f"{record.case_id:<16} {record.date_filed:<12} "
          f"{record.attorney:<18} {record.case_type:<12} "
          f"${record.amount:>13,.2f} {seal_icon:>4}")

print("-" * 75)

# --- Summary ---
total_value = sum(r.amount for r in records)
sealed_count = sum(1 for r in records if r.sealed)
harvey_cases = tuple(r for r in records if r.attorney == "Harvey Specter")

print(f"\nTotal Records:  {len(records)}")
print(f"Sealed Records: {sealed_count}")
print(f"Total Value:    ${total_value:,.2f}")
print(f"Harvey's Cases: {len(harvey_cases)}")

# --- Attempt to tamper with a record ---
print("\n--- Attempting to modify sealed record ---")
try:
    records[0] = "TAMPERED"
except TypeError as e:
    print(f"❌ BLOCKED: {e}")
    print("✅ Records are protected. Data integrity maintained.")

print("\n" + "=" * 75)
```

**Expected Output:**
```
===========================================================================
                          SEALED CASE RECORDS                              
===========================================================================

ID               Date         Attorney           Type           Amount Sealed
---------------------------------------------------------------------------
PSL-2026-001     2026-01-15   Harvey Specter     Merger      $15,750,000.00   🔒
PSL-2026-002     2026-02-03   Mike Ross          Patent       $4,200,000.00   🔒
PSL-2026-003     2026-02-28   Louis Litt         Litigation   $8,900,000.00   🔒
PSL-2026-004     2026-03-10   Harvey Specter     Merger      $22,000,000.00   📝
---------------------------------------------------------------------------

Total Records:  4
Sealed Records: 3
Total Value:    $50,850,000.00
Harvey's Cases: 2

--- Attempting to modify sealed record ---
❌ BLOCKED: 'tuple' object does not support item assignment
✅ Records are protected. Data integrity maintained.
===========================================================================
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

*Louis adjusts his glasses and clears his throat.*

**Louis:** "Tuples seem simple. That's when they're most dangerous. Let me show you the traps."

---

### Pitfall 1: The Single-Element Tuple Trap 🔥

**Louis:** "This is the NUMBER ONE tuple mistake in all of Python:"

```python
# What you THINK is a tuple:
not_a_tuple = (42)
print(type(not_a_tuple))   # <class 'int'> ← NOT a tuple!

# What actually IS a tuple:
is_a_tuple = (42,)          # The trailing comma is ESSENTIAL
print(type(is_a_tuple))     # <class 'tuple'>

# This also works:
also_tuple = 42,
print(type(also_tuple))     # <class 'tuple'>
```

**Louis:** "The parentheses are just grouping. The **comma** creates the tuple. `(42)` is just the number 42 in parentheses — like `(2 + 3)` is just 5."

---

### Pitfall 2: Mutable Objects Inside Tuples 🔥

**Louis:** "Here's a mind-bender. A tuple is immutable, but what if it *contains* a mutable object?"

```python
# A tuple containing a list
case = ("PSL-001", ["Harvey", "Mike"], 250000)

# ❌ Cannot replace the list itself
# case[1] = ["Louis"]     # TypeError!

# ✅ BUT you CAN modify the list INSIDE the tuple!
case[1].append("Louis")
print(case)  # ('PSL-001', ['Harvey', 'Mike', 'Louis'], 250000)
```

**Louis:** "The **tuple** didn't change — it still holds a reference to the *same* list object. But the *contents* of that list changed. The tuple is immutable; the list inside it is not."

**Louis:** "This is like a sealed envelope that contains a sticky note. You can't swap out the sticky note for a different one, but you CAN write on it through the envelope."

```python
# Rule: Tuple immutability is SHALLOW, not deep
t = ([1, 2], [3, 4])
t[0].append(99)
print(t)          # ([1, 2, 99], [3, 4]) — the list inside changed!

# But you still can't reassign
# t[0] = [5, 6]  # TypeError!
```

---

### Pitfall 3: Unpacking Count Must Match

**Louis:** "The number of variables MUST match the number of tuple elements:"

```python
point = (10, 20, 30)

# ❌ Too few variables
# x, y = point  # ValueError: too many values to unpack

# ❌ Too many variables
# x, y, z, w = point  # ValueError: not enough values to unpack

# ✅ Exact match
x, y, z = point

# ✅ Or use * to absorb extras
x, *rest = point
print(x)      # 10
print(rest)   # [20, 30]
```

---

### Pitfall 4: Tuple Concatenation Creates New Objects

**Louis:** "Every time you 'modify' a tuple, you create a brand new one:"

```python
import sys

original = (1, 2, 3)
print(sys.getsizeof(original))  # ~72 bytes

# Each concatenation creates a completely new tuple
extended = original + (4, 5) + (6, 7) + (8, 9)
print(extended)   # (1, 2, 3, 4, 5, 6, 7, 8, 9)

# For large data with frequent modifications, use a LIST, not a tuple
```

**Louis:** "If you're constantly adding to a collection, use a list. Convert to tuple only when done."

---

### Pitfall 5: Accidental Tuple Creation

**Louis:** "A trailing comma can create a tuple when you don't expect it:"

```python
# You think this is a number...
x = 42,
print(type(x))    # <class 'tuple'> — SURPRISE!

# Be especially careful in return statements
def calculate():
    return 42,     # Returns a tuple (42,), not the integer 42!

result = calculate()
print(result)      # (42,)
print(type(result)) # <class 'tuple'>
```

---

### Pitfall 6: Performance — Tuples vs Lists

**Louis:** "Let me prove why tuples are faster:"

```python
import sys

# Memory comparison
my_list = [1, 2, 3, 4, 5]
my_tuple = (1, 2, 3, 4, 5)

print(f"List size:  {sys.getsizeof(my_list)} bytes")   # ~96 bytes
print(f"Tuple size: {sys.getsizeof(my_tuple)} bytes")   # ~80 bytes
# Tuples use LESS memory

# Speed comparison (tuples are faster to create and iterate)
# Creating a tuple is ~5-10x faster than creating a list
# Iterating is marginally faster
# Attribute lookup is faster (fewer methods to search through)
```

**Louis:** "When you have data that doesn't need to change, always use a tuple. It's faster AND safer."

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Okay. I have the perfect analogy for this one."

---

### 📜 Analogy: Tuples Are Like a Signed Contract

**Donna:** "A **list** is a draft document — you can edit, add pages, remove paragraphs. A **tuple** is a **signed, notarized contract** — once both parties sign, it's locked."

```
📝 List (Draft):
   - You CAN add clauses        → append()
   - You CAN remove clauses     → remove()
   - You CAN reorder clauses    → sort()
   - You CAN change wording     → list[i] = new_value

📜 Tuple (Signed Contract):
   - You CANNOT add clauses     → ❌
   - You CANNOT remove clauses  → ❌
   - You CANNOT reorder         → ❌
   - You CANNOT change wording  → ❌
   - You CAN READ it            → tuple[i]
   - You CAN COPY it            → slicing
   - You CAN REFERENCE it       → as a dict key
```

**Donna:** "And **unpacking**? That's like opening an envelope with multiple documents inside and sorting each one into a different folder:"

```python
# The envelope contains three documents
envelope = ("Complaint", "Exhibit A", "Witness List")

# Open it and sort each into its own folder
doc1, doc2, doc3 = envelope
```

**Donna:** "Simple. The envelope is the tuple. The documents are the values. Unpacking sorts them out."

---

### The Three Sequence Types — Side by Side

| Feature | String | List | Tuple |
|---------|--------|------|-------|
| Syntax | `"Hello"` | `[1, 2, 3]` | `(1, 2, 3)` |
| Mutable? | ❌ No | ✅ Yes | ❌ No |
| Indexing? | ✅ Yes | ✅ Yes | ✅ Yes |
| Slicing? | ✅ Yes | ✅ Yes | ✅ Yes |
| `in` check? | ✅ Yes | ✅ Yes | ✅ Yes |
| `len()`? | ✅ Yes | ✅ Yes | ✅ Yes |
| Dict key? | ✅ Yes | ❌ No | ✅ Yes |
| Use case | Text | Dynamic collections | Fixed records |
| Analogy | Charm bracelet | Filing cabinet | Signed contract |

**Donna:** "You now know all three sequence types. Welcome to the pattern."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Tuples seem simple, but the exercises will test your understanding of *when* and *why* to use them."

---

### 🟢 Beginner Exercises

**Exercise 1: Tuple Basics**
```
Create a tuple called 'partner' with these values:
("Harvey Specter", "Senior Partner", 45, True)

a) Print the entire tuple
b) Print each element with a label using f-strings
c) Print the length
d) Check if "Senior Partner" is in the tuple
e) Try to change the age to 46 — what happens?
```

**Exercise 2: Tuple Unpacking**
```
Given: coordinates = (40.7128, -74.0060, "New York City")

a) Unpack into latitude, longitude, and city variables
b) Print: "Location: New York City at 40.7128°N, 74.006°W"

Given: rgb = (65, 105, 225)
c) Unpack into red, green, blue
d) Print: "Royal Blue: R=65, G=105, B=225"
```

**Exercise 3: Swap and Return**
```
a) Create two variables: first = "Harvey", second = "Mike"
   Swap them using tuple unpacking (one line!)
   Print both to verify

b) Write a function called min_max that takes three numbers
   and returns BOTH the minimum and maximum as a tuple.
   Call it and unpack the result.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Grade Records**
```
Create a tuple of tuples representing student grades:

grades = (
    ("Mike Ross", 95, 88, 92),
    ("Rachel Zane", 88, 91, 85),
    ("Harold Gunderson", 72, 68, 75),
    ("Katrina Bennett", 90, 93, 97),
)

a) Print each student's name and average grade
b) Find the student with the highest average
c) Find the student with the lowest single grade
d) Count how many students have an average above 85
```

**Exercise 5: Coordinate Distance**
```
Given two points as tuples:
  point_a = (3, 4)
  point_b = (7, 1)

a) Unpack both points into x1, y1, x2, y2
b) Calculate the Euclidean distance:
   distance = ((x2-x1)**2 + (y2-y1)**2) ** 0.5
c) Calculate the Manhattan distance:
   manhattan = abs(x2-x1) + abs(y2-y1)
d) Find the midpoint as a tuple:
   midpoint = ((x1+x2)/2, (y1+y2)/2)
```

**Exercise 6: Tuple vs List Decision**
```
For each scenario, decide whether to use a tuple or list.
Create the appropriate data structure and explain WHY:

a) Days of the week
b) Items in a shopping cart
c) GPS coordinates of the firm's office
d) A queue of client appointments for today
e) RGB color values for the firm's logo
f) Associate names to be sorted by performance
g) A function returning (success, message, error_code)
h) A log of server events that keeps growing
```

---

### 🔴 Advanced Exercises

**Exercise 7: Immutable Matrix**
```
Create an immutable 3×3 matrix (tuple of tuples):

matrix = (
    (1, 2, 3),
    (4, 5, 6),
    (7, 8, 9)
)

a) Access element at row 1, column 2 (should be 6)
b) Print the main diagonal (1, 5, 9) as a tuple
c) Calculate the sum of each row
d) "Transpose" the matrix (rows become columns):
   Result: ((1,4,7), (2,5,8), (3,6,9))
   Hint: use zip(*matrix)
```

**Exercise 8: Named Tuple Database**
```
Using namedtuple, create a mini database of employees:

Define an Employee type with: name, department, salary, years

Create at least 5 employees as named tuples.

a) Print a formatted table of all employees
b) Find the highest-paid employee
c) Calculate average salary
d) Find all employees in a specific department
e) Find employees with more than 5 years experience
f) Sort employees by salary (descending) — 
   remember, you'll need to convert to a list to sort!
```

**Exercise 9: Function Results Processor**
```
Write a function called 'analyze_numbers' that takes a tuple 
of numbers and returns a tuple containing:

(count, total, average, minimum, maximum, range, is_sorted)

Example:
  analyze_numbers((5, 3, 8, 1, 9, 2, 7))
  → (7, 35, 5.0, 1, 9, 8, False)

Test with at least 3 different tuples, including:
- An already-sorted tuple
- A tuple with negative numbers
- A tuple with a single element
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Tuple debugging. These bugs are subtle."

### Buggy Code 🐛

```python
# Bug Hunt: Case Record System
# Find and fix ALL the bugs!

# Bug 1: Create a single-element tuple
firm = ("Pearson Specter Litt")
print(type(firm))    # Expected: <class 'tuple'>

# Bug 2: Create a case record
case = "PSL-001", "Harvey", 250000
case[2] = 300000     # Update the amount

# Bug 3: Unpack the record
case = ("PSL-001", "Harvey", 250000, True)
id, attorney, amount = case    # Unpack all fields

# Bug 4: Combine tuples
team_a = ("Harvey", "Mike")
team_b = ("Louis", "Donna")
all_team = team_a.extend(team_b)

# Bug 5: Sort a tuple
scores = (88, 95, 72, 91, 85)
scores.sort()
print(scores)

# Bug 6: Named tuple error
from collections import namedtuple
Case = namedtuple("Case", ["id", "client", "amount"])
c = Case("PSL-001", "Wayne Enterprises", 500000)
c.amount = 600000    # Update amount
```

---

### Fixed Code ✅

```python
# Fixed: Case Record System

# Fix 1: Add a trailing COMMA for single-element tuple
firm = ("Pearson Specter Litt",)
print(type(firm))    # <class 'tuple'> ✅

# Fix 2: Tuples are IMMUTABLE — cannot change in place
# Create a new tuple instead
case = ("PSL-001", "Harvey", 250000)
case = case[:2] + (300000,)   # Replace last element
print(case)          # ('PSL-001', 'Harvey', 300000) ✅

# Fix 3: Number of variables must match tuple length
case = ("PSL-001", "Harvey", 250000, True)
case_id, attorney, amount, active = case   # Need 4 variables for 4 values ✅
# OR use * to absorb extras:
# case_id, attorney, *rest = case

# Fix 4: Tuples don't have extend() — use concatenation (+)
team_a = ("Harvey", "Mike")
team_b = ("Louis", "Donna")
all_team = team_a + team_b    # Concatenation ✅
print(all_team)      # ('Harvey', 'Mike', 'Louis', 'Donna')

# Fix 5: Tuples don't have sort() — use sorted() which returns a list
scores = (88, 95, 72, 91, 85)
sorted_scores = tuple(sorted(scores))   # sorted() → list → convert to tuple
print(sorted_scores)  # (72, 85, 88, 91, 95) ✅

# Fix 6: Named tuples are IMMUTABLE — use _replace() to create a new one
from collections import namedtuple
Case = namedtuple("Case", ["id", "client", "amount"])
c = Case("PSL-001", "Wayne Enterprises", 500000)
c = c._replace(amount=600000)   # Creates a NEW namedtuple ✅
print(c)             # Case(id='PSL-001', client='Wayne Enterprises', amount=600000)
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | Missing trailing comma → string, not tuple | Single-element tuple syntax |
| 2 | Direct assignment on immutable tuple | Tuple immutability |
| 3 | Unpacking count mismatch (3 vars, 4 values) | Unpacking rules |
| 4 | `extend()` doesn't exist on tuples | Tuple has no mutating methods |
| 5 | `sort()` doesn't exist on tuples | Use `sorted()` + `tuple()` |
| 6 | Named tuple attributes can't be set | Use `_replace()` method |

---

## 8. 🌍 Real-World Application

**Harvey:** "Tuples aren't glamorous, but they're everywhere."

### Where Tuples Are Used in Production Software:

| Domain | Tuple Use Case |
|--------|---------------|
| **Databases** | Query results (each row is a tuple) |
| **APIs** | Fixed response structures, status code + message |
| **Graphics** | RGB colors `(255, 128, 0)`, coordinates `(x, y)` |
| **Configuration** | Server settings, version numbers `(3, 12, 2)` |
| **Data Science** | DataFrame shapes `(1000, 50)`, array dimensions |
| **Web Frameworks** | Django URL patterns, Flask route parameters |
| **Functional Programming** | Function return values, immutable data pipelines |

### Real Production Example: HTTP Response Handler

```python
# Simulating API response handling
def make_api_request(endpoint):
    """Returns (status_code, response_body, headers) as a tuple."""
    # Simulated response
    if endpoint == "/clients":
        return (200, '{"clients": ["Wayne", "Stark"]}', {"Content-Type": "application/json"})
    elif endpoint == "/cases":
        return (200, '{"cases": 42}', {"Content-Type": "application/json"})
    else:
        return (404, "Not Found", {"Content-Type": "text/plain"})

# Unpack the response
status, body, headers = make_api_request("/clients")

if status == 200:
    print(f"✅ Success: {body}")
    print(f"   Content-Type: {headers['Content-Type']}")
else:
    print(f"❌ Error {status}: {body}")
```

### Real Production Example: Version Comparison

```python
# Software version comparison using tuples
current_version = (3, 12, 2)
required_version = (3, 10, 0)

# Tuples compare element by element — perfect for versions!
if current_version >= required_version:
    print(f"✅ Python {'.'.join(str(v) for v in current_version)} meets requirements")
else:
    print(f"❌ Upgrade needed! Requires {'.'.join(str(v) for v in required_version)}")

# Output: ✅ Python 3.12.2 meets requirements
```

**Harvey:** "Every database query returns rows as tuples. Every function that gives you multiple results uses tuples. Every coordinate, every color, every configuration value — tuples."

---

## 9. ✅ Knowledge Check

**Harvey:** "Last quiz of the week — on tuples."

---

**Q1.** What is the output of `type((42))`? What about `type((42,))`? Why is the comma essential?

**Q2.** Can you modify a list that's inside a tuple? Why or why not?

**Q3.** What is tuple unpacking? Write an example that unpacks `(10, 20, 30)` into three variables.

**Q4.** How do you swap two variables in one line using tuple unpacking?

**Q5.** Why can tuples be used as dictionary keys but lists cannot?

**Q6.** What is the output?
```python
a, *b, c = (1, 2, 3, 4, 5)
print(a, b, c)
```

**Q7.** Name two advantages of tuples over lists.

**Q8.** How do you sort a tuple? (Tuples don't have a `sort()` method.)

**Q9.** When would you use a named tuple instead of a regular tuple?

**Q10.** A function returns `("success", 200, {"data": "test"})`. Write one line to unpack these into `status`, `code`, and `body`.

---

### 📋 Answer Key

> **Q1.** `type((42))` → `<class 'int'>` (parentheses just group). `type((42,))` → `<class 'tuple'>`. The **comma** creates the tuple, not the parentheses.

> **Q2.** Yes! The tuple prevents replacing the list reference, but the list itself is mutable. `t = ([1,2],); t[0].append(3)` works fine.

> **Q3.** `x, y, z = (10, 20, 30)` — assigns 10 to x, 20 to y, 30 to z.

> **Q4.** `a, b = b, a`

> **Q5.** Dictionary keys must be **hashable** (immutable). Tuples are immutable → hashable. Lists are mutable → not hashable.

> **Q6.** `1 [2, 3, 4] 5` — `a` gets first, `c` gets last, `*b` absorbs everything in between as a **list**.

> **Q7.** (Any two): Faster creation/iteration, less memory, can be dict keys, signal data shouldn't change, thread-safe.

> **Q8.** `sorted_tuple = tuple(sorted(original_tuple))` — `sorted()` returns a list, wrap in `tuple()`.

> **Q9.** When you want readable, self-documenting code with named fields instead of numeric indices. E.g., `record.name` instead of `record[0]`.

> **Q10.** `status, code, body = ("success", 200, {"data": "test"})`

---

## 🎬 End of Lesson Scene

*It's 4:45 PM. The sealed case record system is deployed. Rachel runs a test — tries to modify a record. The system blocks it.*

**Rachel:** "It works. No one can tamper with discovery records now."

**Harvey:** *(smiling)* "Good. Now Jessica won't have a reason to fire anyone this week."

**Louis:** *(examining the output)* "I notice you used named tuples. Clever. I would've done the same thing."

**Donna:** "No, you wouldn't have, Louis."

**Mike:** *(packing up)* "Next lesson is **dictionaries**. If lists are filing cabinets and tuples are sealed records, dictionaries are the firm's contact book — you look up information by *name*, not by slot number."

*You head for the elevator. Four days in, and you can handle variables, strings, lists, and tuples. The foundation is getting solid.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | Tuple creation — `()`, comma syntax, `tuple()` | ✅ |
| 2 | Single-element tuple — trailing comma required | ✅ |
| 3 | Tuple indexing and slicing | ✅ |
| 4 | Tuple immutability — cannot assign, append, or remove | ✅ |
| 5 | Tuple packing and unpacking | ✅ |
| 6 | Extended unpacking with `*` | ✅ |
| 7 | Variable swapping with tuples | ✅ |
| 8 | Tuple methods — `count()`, `index()` | ✅ |
| 9 | Tuple ↔ List conversion | ✅ |
| 10 | Tuples as dictionary keys (hashable) | ✅ |
| 11 | Multiple return values from functions | ✅ |
| 12 | Named tuples — `namedtuple` | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Indexing & slicing (Lessons 2–3) | Same syntax for tuples |
| String immutability (Lesson 2) | Tuples share this trait |
| List mutability (Lesson 3) | Contrast — tuples can't do this |
| `in` operator (Lessons 2–3) | Works identically with tuples |
| f-string formatting (Lesson 2) | Displaying tuple data |
| `sort()` vs `sorted()` (Lesson 3) | Only `sorted()` works on tuples |

---

> **Next Lesson →** Level 1, Lesson 5: *Work with Dictionaries in Python*  
> *The firm's contact book needs an upgrade — Harvey wants to look up clients by name, not by number…*

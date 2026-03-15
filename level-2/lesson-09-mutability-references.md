# 🏛️ Level 2 – Applying | Lesson 9: Handle Mutability and Object References in Python

## *"The Alibi Problem"*

> **O'Reilly Skill Objective:** *Handle mutable vs immutable objects of different types, copying objects, and shallow vs deep copy.*

> **Previously Learned:** Variables, data types, lists, tuples, dictionaries, sets, functions, classes, objects, instance/class attributes, `@property`, `@classmethod`, `@staticmethod`

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

*Tuesday morning. Mike is staring at his screen, frozen.*

**Mike:** "Harvey, we have a problem."

```python
# What Mike wrote:
original_case = {"client": "Wayne", "charges": ["fraud", "embezzlement"]}
backup_case = original_case    # "Made a backup"

# Later, someone modifies the "backup":
backup_case["charges"].append("conspiracy")

# Mike checks the original:
print(original_case["charges"])
# ['fraud', 'embezzlement', 'conspiracy']  ← MODIFIED!
```

**Mike:** "I made a backup of the case file, but when the backup was changed, the original changed too. It's like they're... the same file."

**Harvey:** "They ARE the same file. You didn't make a copy. You made an alias — two names pointing to the same folder on the shelf. Move one, the other moves with it."

**Mike:** "This is how Python works. Variables don't store values. They store **references** — pointers to objects. And when you assign `b = a`, you're not copying the object. You're copying the pointer."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "In this firm, there are two types of case files."

| Type | Example | Can You Change It? |
|------|---------|-------------------|
| **Immutable** | Signed contracts, notarized documents | ❌ No — create a new version |
| **Mutable** | Working drafts, living documents | ✅ Yes — modify in place |

**Python's classification:**

| Immutable (can't change) | Mutable (can change in place) |
|:---:|:---:|
| `int`, `float`, `bool` | `list` |
| `str` | `dict` |
| `tuple` | `set` |
| `frozenset` | Custom objects (by default) |
| `bytes` | `bytearray` |

**Harvey:** "The alibi problem: if two people share the same mutable document, when one person edits it, both see the changes. That's not a bug — that's how references work. The question is: when do you WANT shared edits, and when do you need an independent copy?"

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Variables Are References, Not Boxes ⭐⭐⭐

```python
# Most beginners think variables CONTAIN values:
#   a = [1, 2, 3]   →   a: [1, 2, 3]  (box model)
#
# Reality: variables are NAME TAGS (references) that point to objects:
#   a = [1, 2, 3]   →   a ──→ [1, 2, 3]  (reference model)
#
# Assignment copies the REFERENCE, not the object:
#   b = a            →   a ──→ [1, 2, 3] ←── b

a = [1, 2, 3]
b = a               # b points to the SAME list object

print(a is b)        # True — same object in memory!
print(id(a))         # e.g., 140234567890
print(id(b))         # Same number — same object!

b.append(4)          # Modifying through b...
print(a)             # [1, 2, 3, 4] — a sees the change!
```

**Mike:** "`id()` tells you the unique identity (memory address) of an object. `is` checks whether two variables point to the same object."

```python
# == checks VALUE equality
# is checks IDENTITY (same object in memory)

a = [1, 2, 3]
b = [1, 2, 3]      # Different object, same values

print(a == b)        # True  — same values
print(a is b)        # False — different objects

c = a
print(a is c)        # True  — same object
```

---

### 3.2 Immutable Objects — Safe from the Alibi Problem

```python
# Integers are immutable
a = 42
b = a
print(a is b)     # True — same object initially

b = b + 1          # Creates a NEW int object (43)
print(a)           # 42 — unchanged!
print(b)           # 43 — new object
print(a is b)      # False — different objects now

# Strings are immutable
name = "Harvey"
alias = name
alias = alias + " Specter"    # Creates a NEW string
print(name)       # "Harvey" — unchanged
print(alias)      # "Harvey Specter" — new object

# Tuples are immutable
coords = (1, 2, 3)
# coords[0] = 99   # TypeError: 'tuple' does not support item assignment
```

**Mike:** "With immutable objects, you CAN'T accidentally modify the original through a reference. Any 'modification' creates a new object."

---

### 3.3 Mutable Objects — The Alibi Problem in Action 🔥

```python
# Lists are mutable
original = [1, 2, 3]
alias = original

alias.append(4)            # Modifies the SAME object
original[0] = 99            # Same — both are the same list

print(original)             # [99, 2, 3, 4]
print(alias)                # [99, 2, 3, 4] — same list!

# Dicts are mutable
case = {"client": "Wayne", "status": "open"}
case_ref = case

case_ref["status"] = "closed"
print(case["status"])       # "closed" — modified through reference

# Sets are mutable
attorneys = {"Harvey", "Mike", "Louis"}
team = attorneys

team.add("Katrina")
print(attorneys)            # Now includes Katrina!

# Custom objects are mutable by default
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

c = Client("Wayne", 2500000)
ref = c
ref.revenue = 3000000
print(c.revenue)            # 3000000 — changed through ref!
```

---

### 3.4 Functions and Mutability — "Pass by Object Reference"

```python
# Python is NOT "pass by value" or "pass by reference"
# It's "pass by OBJECT REFERENCE"

# Immutable objects: function can't modify the original
def try_modify_int(x):
    x = x + 1        # Creates a new int, rebinds local x
    return x

num = 42
try_modify_int(num)
print(num)            # 42 — unchanged! (immutable)

# Mutable objects: function CAN modify the original
def add_case(case_list, new_case):
    case_list.append(new_case)    # Modifies the SAME list!

cases = ["Wayne v. Stark"]
add_case(cases, "Dalton Merger")
print(cases)           # ['Wayne v. Stark', 'Dalton Merger'] — modified!
```

```python
# Reassignment vs Mutation — the critical distinction
def reassign(lst):
    lst = [99, 99, 99]    # Rebinds LOCAL variable — original unaffected

def mutate(lst):
    lst.append(99)         # Modifies the object — original IS affected
    lst[0] = 99            # Also modifies the same object

original = [1, 2, 3]
reassign(original)
print(original)           # [1, 2, 3] — unchanged!

mutate(original)
print(original)           # [99, 2, 3, 99] — changed!
```

---

### 3.5 Shallow Copy — One Level Deep ⭐

```python
# Shallow copy creates a NEW outer container but shares inner objects

original = [1, 2, [3, 4, 5]]

# Three ways to shallow copy a list:
copy1 = original.copy()          # Method 1: .copy()
copy2 = list(original)           # Method 2: constructor
copy3 = original[:]              # Method 3: slice

# The outer list IS a new object
print(original is copy1)          # False — different list

# But inner objects are SHARED
print(original[2] is copy1[2])    # True — same inner list!

# Modifying top-level items is safe
copy1[0] = 99
print(original[0])                # 1 — unchanged ✅

# But modifying nested objects affects BOTH!
copy1[2].append(6)
print(original[2])                # [3, 4, 5, 6] — CHANGED! ❌
```

```python
# Shallow copy for dicts
original_dict = {"name": "Wayne", "details": {"revenue": 2500000}}

copy = original_dict.copy()       # Shallow copy

copy["name"] = "Stark"             # Top-level — safe
print(original_dict["name"])       # "Wayne" ✅

copy["details"]["revenue"] = 0     # Nested — shared!
print(original_dict["details"])    # {"revenue": 0} ❌
```

---

### 3.6 Deep Copy — Fully Independent ⭐⭐

```python
import copy

original = {
    "client": "Wayne Enterprises",
    "cases": ["Merger", "Patent"],
    "contacts": [
        {"name": "Bruce", "phone": "555-0001"},
        {"name": "Alfred", "phone": "555-0002"},
    ]
}

# Deep copy — recursively copies EVERYTHING
deep = copy.deepcopy(original)

# Completely independent — no shared references at ANY level
deep["cases"].append("Litigation")
deep["contacts"][0]["name"] = "Thomas"

print(original["cases"])           # ['Merger', 'Patent'] ✅ unchanged
print(original["contacts"][0])     # {'name': 'Bruce', ...} ✅ unchanged
```

**Mike:** "The full comparison:"

| Method | Creates New Outer? | Inner Objects Shared? | Speed | Memory |
|--------|:-:|:-:|:-:|:-:|
| **Assignment** (`b = a`) | ❌ Same object | ✅ Shared | Instant | None |
| **Shallow copy** (`.copy()`) | ✅ New container | ✅ Shared | Fast | Small |
| **Deep copy** (`deepcopy()`) | ✅ New container | ✅ All new copies | Slow | Large |

---

### 3.7 Copy Methods Summary

```python
import copy

# === LIST ===
lst = [[1, 2], [3, 4]]
a = lst                         # Assignment — same object
b = lst.copy()                  # Shallow copy
c = lst[:]                      # Shallow copy (slice)
d = list(lst)                   # Shallow copy (constructor)
e = copy.copy(lst)              # Shallow copy (generic)
f = copy.deepcopy(lst)          # Deep copy

# === DICT ===
dct = {"a": [1, 2]}
a = dct                         # Assignment
b = dct.copy()                  # Shallow copy
c = dict(dct)                   # Shallow copy
d = {**dct}                     # Shallow copy (unpacking)
e = copy.copy(dct)              # Shallow copy
f = copy.deepcopy(dct)          # Deep copy

# === SET ===
s = {1, 2, 3}
a = s                           # Assignment
b = s.copy()                    # Shallow copy — but sets can't nest,
c = set(s)                      #   so shallow = deep for sets of
d = copy.copy(s)                #   immutable items!
```

---

### 3.8 Mutability and Default Arguments (Callback to Lesson 1!)

```python
# ❌ Mutable default argument — the infamous Python gotcha
def add_client(name, client_list=[]):
    client_list.append(name)
    return client_list

result1 = add_client("Wayne")
result2 = add_client("Stark")
print(result1)    # ['Wayne', 'Stark'] — contaminated!
print(result2)    # ['Wayne', 'Stark'] — same list!

# ✅ Fix: use None sentinel
def add_client(name, client_list=None):
    if client_list is None:
        client_list = []     # New list each time
    client_list.append(name)
    return client_list
```

---

### 3.9 Tuple Immutability — It's Complicated

```python
# Tuples are immutable — can't change WHICH objects they contain
t = (1, [2, 3], "hello")

# ❌ Can't reassign elements
# t[0] = 99        # TypeError

# ✅ But mutable CONTENTS can still be modified!
t[1].append(4)
print(t)              # (1, [2, 3, 4], 'hello')

# The tuple itself didn't change — it still holds the SAME list object
# But that list object's CONTENTS changed
```

---

### 3.10 `is` vs `==` — Identity vs Equality ⭐

```python
# == : Do these have the same VALUE?
# is : Are these the SAME OBJECT?

a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)    # True  — same content
print(a is b)    # False — different objects
print(a == c)    # True  — same content (and same object)
print(a is c)    # True  — same object

# ⚠️ Special case: Python interns small integers and some strings
x = 256
y = 256
print(x is y)    # True — Python caches small ints (-5 to 256)

x = 257
y = 257
print(x is y)    # May be True or False! Implementation detail!

# ✅ Rule: ALWAYS use == for value comparison
# Use 'is' ONLY for: None, True, False, or intentional identity checks
if x is None:         # ✅ Correct use of 'is'
    pass
```

---

### 3.11 Solving Mike's Problem — Safe Case File System

```python
# ============================================
# Pearson Specter Litt — Safe Case File System
# Understanding references and copying
# ============================================

import copy
from datetime import datetime

class CaseFile:
    """Case file with proper copy semantics."""
    
    def __init__(self, case_id, client, charges, evidence=None, notes=None):
        self.case_id = case_id
        self.client = client
        self.charges = list(charges)           # Always copy on init!
        self.evidence = list(evidence or [])   # Never share defaults
        self.notes = list(notes or [])
        self.created = datetime.now()
        self.versions = []
    
    def add_charge(self, charge):
        self.charges.append(charge)
    
    def add_evidence(self, item):
        self.evidence.append(item)
    
    def add_note(self, note):
        self.notes.append(note)
    
    def snapshot(self):
        """Create a deep copy snapshot for version history."""
        snapshot = copy.deepcopy(self)
        self.versions.append({
            "timestamp": datetime.now().isoformat(),
            "charges": list(self.charges),
            "evidence": list(self.evidence),
            "notes": list(self.notes),
        })
        return snapshot
    
    def create_backup(self):
        """Create a fully independent backup (deep copy)."""
        backup = copy.deepcopy(self)
        backup.case_id = f"{self.case_id}-BACKUP"
        return backup
    
    def create_related(self):
        """Create a related case (shares nothing)."""
        return CaseFile(
            case_id=f"{self.case_id}-RELATED",
            client=self.client,             # String — immutable, safe to share
            charges=[],                     # New empty list
            evidence=[],
            notes=[f"Related to case {self.case_id}"]
        )
    
    def __repr__(self):
        return (f"CaseFile('{self.case_id}', '{self.client}', "
                f"charges={self.charges}, evidence={len(self.evidence)} items)")

# === Demonstrate ===
print("=" * 60)
print(f"{'SAFE CASE FILE SYSTEM':^60}")
print("=" * 60)

# Create original case
original = CaseFile("CASE-001", "Wayne Enterprises",
                     charges=["fraud", "embezzlement"],
                     evidence=["doc_001.pdf", "email_chain_42.pdf"])

print(f"\n📁 Original: {original}")

# ❌ BAD: Assignment (alias)
alias = original
alias.add_charge("conspiracy")
print(f"\n❌ After alias.add_charge('conspiracy'):")
print(f"   Original charges: {original.charges}")
print(f"   Same object? {original is alias}")   # True!

# ✅ GOOD: Deep copy (independent backup)
backup = original.create_backup()
backup.add_charge("money laundering")
print(f"\n✅ After backup.add_charge('money laundering'):")
print(f"   Original charges: {original.charges}")   # No money laundering!
print(f"   Backup charges: {backup.charges}")
print(f"   Same object? {original is backup}")   # False!

# Snapshot for history
original.snapshot()
original.add_note("Key witness identified")
original.snapshot()
print(f"\n📸 Versions saved: {len(original.versions)}")
for i, v in enumerate(original.versions, 1):
    print(f"   v{i}: {v['timestamp'][:19]} | {len(v['charges'])} charges")

# Related case (shares nothing mutable)
related = original.create_related()
related.add_charge("insider trading")
print(f"\n🔗 Related case: {related}")
print(f"   Original unchanged: {original.charges}")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Modifying a List Passed to a Function

```python
def process_clients(clients):
    clients.sort()                    # ⚠️ Modifies the ORIGINAL list!
    clients.append("New Client")      # ⚠️ Modifies the ORIGINAL!
    return clients

my_clients = ["Wayne", "Stark", "Oscorp"]
result = process_clients(my_clients)
print(my_clients)    # ['New Client', 'Oscorp', 'Stark', 'Wayne'] — modified!

# ✅ Fix: work on a copy
def process_clients(clients):
    working_copy = clients.copy()     # Shallow copy — safe for flat lists
    working_copy.sort()
    working_copy.append("New Client")
    return working_copy
```

---

### Pitfall 2: `+=` on Mutable vs Immutable

```python
# For immutables, += creates a new object
a = 10
b = a
a += 5           # a = a + 5 → new int object
print(b)          # 10 — b still points to old object

# For mutables, += modifies IN PLACE!
a = [1, 2]
b = a
a += [3, 4]       # a.__iadd__([3, 4]) → modifies same list!
print(b)           # [1, 2, 3, 4] — b sees the change!

# But a = a + [3, 4] creates a NEW list!
a = [1, 2]
b = a
a = a + [3, 4]    # Creates new list, rebinds a
print(b)           # [1, 2] — b unchanged!
```

---

### Pitfall 3: Dictionary Values Are References Too

```python
# Common mistake: building a dict with references to the same list
row = [0, 0, 0]
grid = {"row1": row, "row2": row, "row3": row}

grid["row1"][0] = 1
print(grid["row2"])    # [1, 0, 0] — all rows are the same list!

# ✅ Fix: create independent lists
grid = {f"row{i}": [0, 0, 0] for i in range(1, 4)}
grid["row1"][0] = 1
print(grid["row2"])    # [0, 0, 0] ✅
```

---

### Pitfall 4: `copy.deepcopy` with Circular References

```python
import copy

# Deep copy handles circular references!
a = [1, 2]
a.append(a)         # Circular: a contains itself!
print(a)             # [1, 2, [...]]

b = copy.deepcopy(a)   # Handles this correctly!
print(b)                # [1, 2, [...]]
print(b is a)            # False
print(b[2] is b)         # True — the copy's self-reference is correct
```

---

### Pitfall 5: Comprehensions Create New Objects (Usually Safe)

```python
original = [[1, 2], [3, 4]]

# List comprehension creates a new outer list
copied = [row for row in original]     # Shallow! Inner lists shared!
copied[0].append(99)
print(original[0])    # [1, 2, 99] — affected!

# ✅ Copy inner lists too
copied = [row.copy() for row in original]   # Independent!
# or: copied = [list(row) for row in original]
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏷️ Analogy: Variables Are Name Tags on Luggage

```
IMMUTABLE OBJECTS (int, str, tuple) = Stone tablets
  📝 "Harvey" → 🗿 Stone carved with "Harvey Specter"
  You can't erase the stone. You can only carve a NEW stone.
  
  📝 name = "Harvey"
  📝 alias = name      → Both tags on the SAME stone
  📝 alias = "Mike"    → alias tag moves to a NEW stone
  📝 name still on the old stone → "Harvey" unchanged

MUTABLE OBJECTS (list, dict, set) = Whiteboards
  📝 "Cases" → 🏷️ Whiteboard with items written on it
  Anyone with a name tag (reference) can erase and rewrite.
  
  📝 a = [1, 2]
  📝 b = a           → Two tags on the SAME whiteboard
  📝 b.append(3)     → Writing on the whiteboard
  📝 a sees [1, 2, 3] → Same whiteboard!

COPY = Photograph of the whiteboard
  📸 Shallow copy → Photo of the outer whiteboard only
     Inner sticky notes are still the originals
  
  📷 Deep copy → Photo of EVERYTHING, recursively
     Completely independent reproduction
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Reference vs Copy**
```
a) Create a list [1, 2, 3]. Assign it to two variables.
   Demonstrate that modifying through one affects the other.
b) Create a proper copy. Demonstrate independence.
c) Do the same with a dictionary.
```

**Exercise 2: is vs ==**
```
Create examples where:
a) a == b is True but a is b is False
b) a == b is True AND a is b is True
c) a == b is False but type(a) == type(b) is True
```

**Exercise 3: Mutable Default Fix**
```
Write a function with a mutable default argument.
a) Show the bug
b) Fix it using None sentinel
c) Verify the fix works across multiple calls
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Copy Depth Explorer**
```
Create a nested structure: list containing dicts containing lists.
Show the difference between:
a) Assignment (alias)
b) Shallow copy (one level)
c) Deep copy (all levels)

For each, modify at different levels and check the original.
```

**Exercise 5: Immutability Guard**
```
Write a function: freeze(obj) that returns an immutable version:
  - list → tuple
  - dict → frozenset of tuples (or a MappingProxyType)
  - set → frozenset
  - nested structures → recursively frozen

Write the reverse: thaw(obj) that makes it mutable again.
```

**Exercise 6: Safe Data Pipeline**
```
Build a data processing pipeline that:
  - Accepts input data (list of dicts)
  - Each stage receives a COPY (not original)
  - Returns results without modifying input
  - Compare with an "unsafe" version that mutates input

Process 100+ records through 4 stages.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Copy-on-Write List**
```
Build a COWList (Copy-on-Write) class that:
  - Initially shares data with the original
  - Only copies when a mutation is attempted
  - Tracks whether a copy has been made
  - Is memory-efficient for read-heavy workloads

cow = COWList(original_data)
print(cow[0])        # Reads from shared data — no copy
cow.append(99)       # NOW copies, then appends
# Original is unchanged
```

**Exercise 8: Object Graph Visualizer**
```
Build a function that visualizes the reference graph:

visualize(a, b, c)
Output:
  a (id: 140...) ──→ [1, 2, 3] (id: 140...)
  b (id: 140...) ──→ [1, 2, 3] (id: 140...)  ← SHARED with a!
  c (id: 140...) ──→ [1, 2, 3] (id: 140...)  ← independent copy

Show which variables share objects.
```

**Exercise 9: Version Control**
```
Build a VersionedDict that:
  - Tracks every change as a new version (snapshot)
  - Can rollback to any previous version
  - Uses deep copy for independence
  - Has .diff(v1, v2) to show changes between versions
  - Memory-efficient (only stores diffs, not full copies)
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: "Copying" with assignment
data = [1, 2, 3]
backup = data
data.append(4)
print(backup)          # Expected: [1, 2, 3]

# Bug 2: Shallow copy of nested data
matrix = [[1, 0], [0, 1]]
copy_m = matrix.copy()
copy_m[0][0] = 99
print(matrix[0])       # Expected: [1, 0]

# Bug 3: Function mutates input
def sort_and_filter(items, threshold=0):
    items.sort()
    return [x for x in items if x > threshold]

nums = [5, 2, 8, 1, 9]
result = sort_and_filter(nums)
print(nums)            # Expected: [5, 2, 8, 1, 9]

# Bug 4: Shared default mutable
def append_to(item, target=[]):
    target.append(item)
    return target

a = append_to(1)
b = append_to(2)
print(a)               # Expected: [1]
print(b)               # Expected: [2]

# Bug 5: Tuple with mutable contents
record = ("Wayne", [2500000, 800000])
# record[1] = [999]   # TypeError... but:
record[1].append(600000)
print(record)          # Expected: unchanged tuple
```

### Fixed Code ✅

```python
# Fix 1: Use .copy() for an independent copy
data = [1, 2, 3]
backup = data.copy()         # ✅ New list object
data.append(4)
print(backup)                 # [1, 2, 3] ✅

# Fix 2: Use deepcopy for nested data
import copy
matrix = [[1, 0], [0, 1]]
copy_m = copy.deepcopy(matrix)   # ✅ Deep copy
copy_m[0][0] = 99
print(matrix[0])                  # [1, 0] ✅

# Fix 3: Don't mutate input — work on a copy
def sort_and_filter(items, threshold=0):
    sorted_items = sorted(items)   # ✅ sorted() returns new list
    return [x for x in sorted_items if x > threshold]

# Fix 4: Use None sentinel for mutable defaults
def append_to(item, target=None):
    if target is None:
        target = []            # ✅ New list each call
    target.append(item)
    return target

a = append_to(1)              # [1] ✅
b = append_to(2)              # [2] ✅

# Fix 5: Tuples contain references — mutable contents CAN change
record = ("Wayne", [2500000, 800000])
record[1].append(600000)       # This WORKS — the list is mutable
print(record)                   # ('Wayne', [2500000, 800000, 600000])
# If you need true immutability, use tuple of tuples:
record = ("Wayne", (2500000, 800000))   # ✅ Fully immutable
```

---

## 8. 🌍 Real-World Application

### Where Mutability Matters in Production

| Scenario | Issue | Solution |
|----------|-------|---------|
| **API responses** | Don't mutate shared cache | `deepcopy` before returning |
| **Config objects** | Accidental mutation breaks other modules | Frozen dataclasses, immutable configs |
| **Function args** | Caller's data modified unexpectedly | Copy on entry or use immutables |
| **Thread safety** | Multiple threads sharing mutable state | Locks, or use immutable data |
| **Undo/redo** | Need independent snapshots | `deepcopy` for each state |
| **Testing** | Test data mutated between tests | Fresh copy per test |

### Production Example: Safe API Cache

```python
import copy
from datetime import datetime, timedelta

class APICache:
    """Thread-safe cache that returns copies, not references."""
    
    def __init__(self, ttl_seconds=300):
        self._cache = {}
        self._timestamps = {}
        self.ttl = ttl_seconds
        self.stats = {"hits": 0, "misses": 0}
    
    def get(self, key):
        """Return a DEEP COPY — callers can't corrupt the cache."""
        if key in self._cache:
            if not self._is_expired(key):
                self.stats["hits"] += 1
                return copy.deepcopy(self._cache[key])   # ← Safe!
            else:
                del self._cache[key]
                del self._timestamps[key]
        
        self.stats["misses"] += 1
        return None
    
    def set(self, key, value):
        """Store a DEEP COPY — callers' later modifications don't affect cache."""
        self._cache[key] = copy.deepcopy(value)    # ← Safe!
        self._timestamps[key] = datetime.now()
    
    def _is_expired(self, key):
        age = (datetime.now() - self._timestamps[key]).total_seconds()
        return age > self.ttl

# Usage
cache = APICache(ttl_seconds=60)

# Store data
client_data = {"name": "Wayne", "cases": ["Merger", "Patent"]}
cache.set("wayne", client_data)

# Retrieve — gets a COPY
retrieved = cache.get("wayne")
retrieved["cases"].append("Litigation")   # Modify the copy

# Original cache is unaffected!
cached = cache.get("wayne")
print(cached["cases"])    # ['Merger', 'Patent'] — safe!
```

---

## 9. ✅ Knowledge Check

---

**Q1.** Are Python variables boxes (containers) or name tags (references)?

**Q2.** What is the difference between `==` and `is`?

**Q3.** Name three mutable and three immutable Python types.

**Q4.** What happens when you do `b = a` where `a` is a list?

**Q5.** What is a shallow copy? What doesn't it copy?

**Q6.** When do you need `copy.deepcopy()`?

**Q7.** What is the output?
```python
a = [1, [2, 3]]
b = a.copy()
b[1].append(4)
print(a)
```

**Q8.** Why is `def f(x=[]):` a bug? What's the fix?

**Q9.** Can you modify a list inside a tuple? Why?

**Q10.** What does `id()` return and when would you use it?

---

### 📋 Answer Key

> **Q1.** Name tags (references). Variables point TO objects; they don't contain them.

> **Q2.** `==` checks value equality. `is` checks identity (same object in memory).

> **Q3.** Mutable: list, dict, set. Immutable: int, str, tuple.

> **Q4.** `b` becomes another reference to the **same** list. No copy is made.

> **Q5.** A new outer container, but inner objects are still shared. Doesn't copy nested mutable objects.

> **Q6.** When you have nested mutable objects and need a fully independent copy.

> **Q7.** `[1, [2, 3, 4]]` — shallow copy shares the inner list!

> **Q8.** The default list is created ONCE at function definition and shared across all calls. Fix: `def f(x=None): if x is None: x = []`.

> **Q9.** Yes! The tuple can't change which objects it holds, but the list object itself is still mutable.

> **Q10.** The unique identity (memory address) of an object. Use it to verify whether two variables point to the same object.

---

## 🎬 End of Lesson Scene

**Mike:** *(looking at the backup system)* "So `b = a` copies the reference, not the object. `.copy()` copies one level. `deepcopy()` copies everything."

**Harvey:** "And for case files, we always want independent copies. One client's changes can't contaminate another's."

**Mike:** "Next: storing and retrieving data with Python objects — serialization, JSON, pickle, and building a persistence layer."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Variables are references, not boxes | ✅ |
| 2 | `id()`, `is` vs `==` | ✅ |
| 3 | Mutable vs immutable types | ✅ |
| 4 | The alibi problem (shared references) | ✅ |
| 5 | Pass by object reference in functions | ✅ |
| 6 | Shallow copy (`.copy()`, `[:]`, constructor) | ✅ |
| 7 | Deep copy (`copy.deepcopy()`) | ✅ |
| 8 | Mutable default argument bug | ✅ |
| 9 | Tuple immutability nuance | ✅ |
| 10 | `+=` on mutable vs immutable | ✅ |

---

> **Next Lesson →** Level 2, Lesson 10: *Store and Retrieve Data with Python Objects*

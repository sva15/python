# 🏛️ Level 2 – Applying | Lesson 6: Choose Lists, Dicts, or Sets in Python

## *"The Right Tool for the Right Job"*

> **O'Reilly Skill Objective:** *Choose lists, dictionaries, and sets based on performance implications for common operations.*

> **Previously Learned:** Lists, tuples, dictionaries, loops, comprehensions, built-in functions, generator expressions, classes, objects

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

*Thursday afternoon. Louis storms into Harvey's office holding a printout.*

**Louis:** "Your billing system is taking 45 seconds to check if a client is in the active roster. FORTY-FIVE SECONDS."

**Harvey:** "That's impossible. We only have 237 clients."

**Louis:** "We have 237 clients checked against 850,000 historical billing records. Your associate stored everything in LISTS."

*Harvey turns to you.*

**Harvey:** "How are you looking up clients?"

```python
# What you wrote:
if client_name in active_clients_list:    # Checks every item one by one!
    process(client_name)
```

**Mike:** "That's an O(n) search — 850,000 comparisons per lookup. Switch to a set, and it's O(1) — instant, regardless of size."

```python
# What you should have written:
active_clients_set = set(active_clients_list)   # One-time conversion
if client_name in active_clients_set:            # Instant lookup!
    process(client_name)
```

**Harvey:** "Same result. 45 seconds down to milliseconds. The difference? Choosing the right data structure."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "There are three core data structures. Each has strengths. Each has weaknesses. Choose wrong, and your program crawls."

| Structure | Think of It As | Best For |
|-----------|---------------|----------|
| **List** `[]` | An ordered filing cabinet | Ordered sequences, iteration, indexed access |
| **Dict** `{}` | A labeled lookup table | Key-value mappings, fast lookup by name |
| **Set** `set()` | A membership ledger | Uniqueness, membership testing, set math |

**Harvey:** "Lists for order. Dicts for lookup. Sets for membership. That's the decision framework."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 The Big-O Performance Table ⭐⭐⭐

**Mike:** "This is the most important table in this entire course."

| Operation | List | Dict | Set |
|-----------|------|------|-----|
| **Lookup** (`x in collection`) | **O(n)** 🐌 | **O(1)** ⚡ | **O(1)** ⚡ |
| **Insert** (append/add) | **O(1)** ⚡ | **O(1)** ⚡ | **O(1)** ⚡ |
| **Insert at position** | **O(n)** 🐌 | — | — |
| **Delete by value** | **O(n)** 🐌 | **O(1)** ⚡ | **O(1)** ⚡ |
| **Delete by index** | **O(n)** 🐌 | — | — |
| **Access by index** | **O(1)** ⚡ | — | — |
| **Access by key** | — | **O(1)** ⚡ | — |
| **Iteration** | **O(n)** | **O(n)** | **O(n)** |
| **Sort** | **O(n log n)** | — | — |
| **Order preserved?** | ✅ Yes | ✅ Yes (3.7+) | ❌ No |
| **Duplicates?** | ✅ Allowed | Keys unique | ❌ No duplicates |

**Mike:** "O(1) means constant time — same speed whether you have 10 items or 10 million. O(n) means linear — scales with the size of the collection."

---

### 3.2 Lists — When Order Matters

**When to use a list:**
- Maintaining insertion order
- Indexed access (first item, last item, n-th item)
- Allowing duplicates
- Frequent iteration from start to end
- Sorting and reversing
- Stack operations (append/pop from end)

```python
# ✅ GOOD list uses
case_timeline = ["Filed", "Discovery", "Deposition", "Hearing", "Verdict"]
# Order matters! Each step follows the previous

billable_hours = [8.5, 7.0, 9.5, 6.0, 8.0, 8.5, 7.0]
# Duplicates allowed, order = days of the week

# Indexed access — O(1), lists excel here
print(case_timeline[0])     # First event
print(case_timeline[-1])    # Last event

# Stack (LIFO) — O(1) append/pop from end
undo_stack = []
undo_stack.append("edit_1")
undo_stack.append("edit_2")
last_action = undo_stack.pop()   # "edit_2"

# Queue-like (FIFO) — ⚠️ O(n) for pop(0)!
# Use collections.deque instead for queues
```

```python
# ❌ BAD list uses

# Membership testing — SLOW for large lists
if client_name in huge_client_list:    # O(n) — scans every item!
    ...

# Removing by value — SLOW
huge_list.remove("target")   # O(n) — must find it first, then shift items
```

---

### 3.3 Dictionaries — When You Need Key-Based Lookup

**When to use a dict:**
- Mapping keys to values (lookup table)
- Fast access by a unique identifier
- Counting occurrences
- Grouping data by category
- Caching/memoization
- Configuration and settings

```python
# ✅ GOOD dict uses

# Instant lookup by name — O(1)
client_registry = {
    "Wayne Enterprises": {"revenue": 2500000, "type": "Merger"},
    "Stark Industries": {"revenue": 800000, "type": "Patent"},
    "Oscorp": {"revenue": 600000, "type": "Litigation"},
}

wayne = client_registry["Wayne Enterprises"]   # O(1) — instant!
print(wayne["revenue"])

# Counting — O(1) per update
word_counts = {}
for word in document.split():
    word_counts[word] = word_counts.get(word, 0) + 1

# Or using collections.Counter (built on dict)
from collections import Counter
word_counts = Counter(document.split())

# Grouping
by_type = {}
for client in clients:
    ct = client["type"]
    by_type.setdefault(ct, []).append(client["name"])

# Caching / Memoization
cache = {}
def expensive_calculation(n):
    if n not in cache:
        cache[n] = sum(i ** 2 for i in range(n))
    return cache[n]
```

```python
# ❌ BAD dict uses

# When you just need to check membership (use a set instead)
exists = {"Wayne": True, "Stark": True, "Oscorp": True}
if "Wayne" in exists:   # Works, but a set is simpler

# When order is your primary concern and you don't need keys
# Use a list instead
```

---

### 3.4 Sets — When Uniqueness and Membership Matter

**When to use a set:**
- Eliminating duplicates
- Fast membership testing (is X in the collection?)
- Mathematical set operations (union, intersection, difference)
- Tracking "seen" items
- Validating against an allowed list

```python
# ✅ GOOD set uses

# Deduplication — instant
all_attorneys = ["Harvey", "Mike", "Louis", "Harvey", "Mike", "Katrina"]
unique_attorneys = set(all_attorneys)
print(unique_attorneys)   # {'Harvey', 'Mike', 'Louis', 'Katrina'}

# Membership testing — O(1)
vip_clients = {"Wayne", "Stark", "Queen", "Dalton"}
if "Wayne" in vip_clients:          # O(1) — instant!
    print("VIP treatment activated")

# Set operations — mathematical elegance
team_a = {"Harvey", "Mike", "Rachel"}
team_b = {"Mike", "Louis", "Katrina"}

# Who's on BOTH teams?
print(team_a & team_b)          # {'Mike'} — intersection
# or: team_a.intersection(team_b)

# Who's on at LEAST ONE team?
print(team_a | team_b)          # {'Harvey', 'Mike', 'Rachel', 'Louis', 'Katrina'}
# or: team_a.union(team_b)

# Who's on team A but NOT team B?
print(team_a - team_b)          # {'Harvey', 'Rachel'}
# or: team_a.difference(team_b)

# Who's on EXACTLY ONE team (not both)?
print(team_a ^ team_b)          # {'Harvey', 'Rachel', 'Louis', 'Katrina'}
# or: team_a.symmetric_difference(team_b)

# Subset / superset
junior = {"Mike", "Rachel"}
senior = {"Harvey", "Mike", "Rachel", "Louis"}
print(junior <= senior)          # True — junior is a subset of senior
print(senior >= junior)          # True — senior is a superset of junior
```

```python
# "Seen" tracking — dedup during iteration
def unique_clients(records):
    """Return unique client names from records, in order."""
    seen = set()
    result = []
    for record in records:
        name = record["name"]
        if name not in seen:         # O(1) check
            seen.add(name)           # O(1) add
            result.append(name)
    return result
```

```python
# ❌ BAD set uses

# When order matters (sets are unordered)
# When you need duplicates (sets auto-deduplicate)
# When items aren't hashable (lists, dicts can't be in sets)
```

---

### 3.5 Performance Benchmark — Seeing Is Believing ⭐

```python
import time

def benchmark(name, func, *args):
    start = time.time()
    result = func(*args)
    elapsed = time.time() - start
    print(f"  {name:<35} {elapsed:.6f}s")
    return result

N = 1_000_000

# Create test data
test_list = list(range(N))
test_set = set(range(N))
test_dict = {i: True for i in range(N)}

target = N - 1    # Worst case for list (last element)

print(f"\n{'='*55}")
print(f"{'MEMBERSHIP TEST (x in collection)':^55}")
print(f"{'='*55}")
print(f"  Collection size: {N:,}\n")

# Membership test: list vs set vs dict
benchmark("List  (x in list)",  lambda: target in test_list)
benchmark("Set   (x in set)",   lambda: target in test_set)
benchmark("Dict  (x in dict)",  lambda: target in test_dict)

print(f"\n{'='*55}")
print(f"{'RESULTS':^55}")
print(f"{'='*55}")
print(f"  List:  O(n) — must check up to {N:,} items")
print(f"  Set:   O(1) — hash lookup, instant")
print(f"  Dict:  O(1) — hash lookup, instant")

# Typical results:
# List:   0.015000s (slow — scans entire list)
# Set:    0.000001s (instant — hash lookup)
# Dict:   0.000001s (instant — hash lookup)
# Set/Dict are ~15,000x faster for membership!
```

---

### 3.6 The Decision Flowchart

```python
"""
Do you need to map keys to values?
├── YES → Use a DICT
│   ├── Need default values? → collections.defaultdict
│   ├── Need ordered insertion? → dict (Python 3.7+)
│   └── Need to count things? → collections.Counter
│
├── NO:
│   Do you need uniqueness (no duplicates)?
│   ├── YES → Use a SET
│   │   ├── Need it immutable? → frozenset
│   │   ├── Need set math (∪ ∩ −)? → set
│   │   └── Just need fast "in" checks? → set
│   │
│   └── NO:
│       Do you need order or indexed access?
│       ├── YES → Use a LIST
│       │   ├── Need it immutable? → tuple
│       │   ├── Need FIFO queue? → collections.deque
│       │   └── Need sorted insertion? → bisect.insort
│       │
│       └── NOT SURE → Start with a LIST (most versatile)
"""
```

---

### 3.7 `collections` Module — Specialized Structures

```python
from collections import defaultdict, Counter, OrderedDict, deque

# --- defaultdict: dict with automatic default values ---
by_department = defaultdict(list)
employees = [("Legal", "Harvey"), ("Legal", "Mike"), ("IT", "David"), ("Legal", "Louis")]

for dept, name in employees:
    by_department[dept].append(name)    # No KeyError — auto-creates []

print(dict(by_department))
# {'Legal': ['Harvey', 'Mike', 'Louis'], 'IT': ['David']}

# --- Counter: counting things ---
words = "the the cat sat on the mat the cat".split()
counts = Counter(words)
print(counts.most_common(3))    # [('the', 4), ('cat', 2), ('sat', 1)]
print(counts["the"])             # 4

# Counter arithmetic
c1 = Counter("aaabbc")
c2 = Counter("abbccc")
print(c1 + c2)    # Counter({'a': 4, 'b': 4, 'c': 4})
print(c1 - c2)    # Counter({'a': 2}) — only positive counts

# --- deque: efficient queue (insert/remove from both ends) ---
queue = deque(maxlen=5)
queue.append("task1")
queue.append("task2")
queue.appendleft("urgent!")    # O(1) — lists are O(n) for this!
print(queue.popleft())          # "urgent!" — O(1)

# --- namedtuple: lightweight object ---
from collections import namedtuple
Client = namedtuple("Client", ["name", "revenue", "type"])
c = Client("Wayne", 2500000, "Merger")
print(c.name)       # "Wayne" — like an object
print(c[0])          # "Wayne" — like a tuple
```

---

### 3.8 Common Conversion Patterns

```python
# List → Set (deduplicate)
names = ["Harvey", "Mike", "Harvey", "Louis", "Mike"]
unique = set(names)           # {'Harvey', 'Mike', 'Louis'}

# List → Dict (index lookup)
items = ["alpha", "bravo", "charlie"]
lookup = {item: i for i, item in enumerate(items)}
print(lookup["bravo"])         # 1

# Dict → Set (just the keys)
registry = {"Wayne": 2500000, "Stark": 800000}
client_names = set(registry.keys())   # or just set(registry)

# Set → Sorted List (ordered unique items)
unique_names = {"Mike", "Harvey", "Louis"}
sorted_names = sorted(unique_names)
print(sorted_names)            # ['Harvey', 'Louis', 'Mike']

# Multiple Lists → Dict of Lists
keys = ["name", "revenue", "type"]
vals = ["Wayne", 2500000, "Merger"]
record = dict(zip(keys, vals))

# Flat List → Grouped Dict
clients = [
    {"name": "Wayne", "type": "Merger"},
    {"name": "Stark", "type": "Patent"},
    {"name": "Dalton", "type": "Merger"},
]
grouped = defaultdict(list)
for c in clients:
    grouped[c["type"]].append(c["name"])
# {'Merger': ['Wayne', 'Dalton'], 'Patent': ['Stark']}
```

---

### 3.9 Solving Louis's Problem — Optimized Client System

```python
# ============================================
# Pearson Specter Litt — Optimized Client System
# Choosing the right structure for each job
# ============================================

from collections import defaultdict, Counter
import time

class FirmDatabase:
    """Uses the right data structure for each operation."""
    
    def __init__(self):
        # LIST: ordered attorney roster (position matters)
        self.attorney_roster = []
        
        # DICT: instant client lookup by name
        self.client_registry = {}
        
        # SET: fast VIP membership checks
        self.vip_clients = set()
        
        # DICT of LISTS: group clients by type
        self.by_case_type = defaultdict(list)
        
        # COUNTER: track case assignments
        self.case_counts = Counter()
        
        # SET: track processed invoice IDs (no duplicates)
        self.processed_invoices = set()
    
    def add_attorney(self, name, rate, position=None):
        """Add attorney to ordered roster (LIST — position matters)."""
        if position is not None:
            self.attorney_roster.insert(position, {"name": name, "rate": rate})
        else:
            self.attorney_roster.append({"name": name, "rate": rate})
    
    def add_client(self, name, revenue, case_type, attorney):
        """Add client to registry (DICT — instant lookup by name)."""
        client = {
            "name": name, "revenue": revenue,
            "case_type": case_type, "attorney": attorney
        }
        self.client_registry[name] = client
        self.by_case_type[case_type].append(name)
        self.case_counts[attorney] += 1
        
        if revenue >= 1000000:
            self.vip_clients.add(name)
    
    def is_vip(self, name):
        """Check VIP status (SET — O(1) membership)."""
        return name in self.vip_clients
    
    def get_client(self, name):
        """Get client details (DICT — O(1) lookup)."""
        return self.client_registry.get(name)
    
    def process_invoice(self, invoice_id):
        """Process an invoice, skip duplicates (SET — O(1) check)."""
        if invoice_id in self.processed_invoices:
            return False    # Already processed
        self.processed_invoices.add(invoice_id)
        return True
    
    def clients_by_type(self, case_type):
        """Get clients for a case type (DICT of LISTS)."""
        return self.by_case_type.get(case_type, [])
    
    def busiest_attorneys(self, n=3):
        """Top N attorneys by caseload (COUNTER)."""
        return self.case_counts.most_common(n)
    
    def shared_clients(self, attorney1_clients, attorney2_clients):
        """Find clients shared between attorneys (SET intersection)."""
        return set(attorney1_clients) & set(attorney2_clients)
    
    def dashboard(self):
        """Display the firm dashboard."""
        print(f"\n{'='*60}")
        print(f"{'🏛️  FIRM DATABASE DASHBOARD':^60}")
        print(f"{'='*60}")
        
        print(f"\n📋 Attorney Roster (ordered by seniority — LIST):")
        for i, atty in enumerate(self.attorney_roster, 1):
            cases = self.case_counts[atty["name"]]
            print(f"  {i}. {atty['name']:<15} ${atty['rate']}/hr  ({cases} cases)")
        
        print(f"\n📖 Client Registry (instant lookup — DICT):")
        for name, client in self.client_registry.items():
            vip = "💎" if self.is_vip(name) else "  "
            print(f"  {vip} {name:<22} ${client['revenue']:>12,}  {client['case_type']}")
        
        print(f"\n⭐ VIP Clients (instant membership — SET):")
        print(f"  {', '.join(sorted(self.vip_clients))}")
        
        print(f"\n💼 By Case Type (grouped lookup — DICT of LISTS):")
        for ct, names in sorted(self.by_case_type.items()):
            print(f"  {ct}: {', '.join(names)}")
        
        print(f"\n📊 Busiest Attorneys (counting — COUNTER):")
        for atty, count in self.busiest_attorneys():
            bar = "█" * count + "░" * (10 - count)
            print(f"  {atty:<15} [{bar}] {count}")
        
        print(f"\n🧾 Processed Invoices (dedup — SET): {len(self.processed_invoices)}")
        
        print(f"\n{'='*60}")
        print(f"  Data Structures Used:")
        print(f"  • LIST:    Attorney roster (ordered, indexed)")
        print(f"  • DICT:    Client registry (key → value lookup)")
        print(f"  • SET:     VIP membership, invoice dedup (O(1) checks)")
        print(f"  • COUNTER: Attorney caseload (automatic counting)")
        print(f"  • defaultdict(list): Case type grouping")
        print(f"{'='*60}")


# === Build and Test ===
db = FirmDatabase()

# Add attorneys (order = seniority)
db.add_attorney("Jessica", 500)
db.add_attorney("Harvey", 450)
db.add_attorney("Louis", 425)
db.add_attorney("Mike", 375)

# Add clients
clients_data = [
    ("Wayne Enterprises", 2500000, "Merger", "Harvey"),
    ("Stark Industries", 800000, "Patent", "Mike"),
    ("Oscorp", 600000, "Litigation", "Louis"),
    ("Dalton Industries", 1200000, "Merger", "Harvey"),
    ("Luthor Corp", 150000, "Corporate", "Louis"),
    ("Queen Industries", 3000000, "Merger", "Jessica"),
    ("Palmer Tech", 400000, "Patent", "Mike"),
]

for name, rev, ct, atty in clients_data:
    db.add_client(name, rev, ct, atty)

# Process some invoices (duplicates handled)
for inv_id in ["INV-001", "INV-002", "INV-003", "INV-001", "INV-002"]:
    result = db.process_invoice(inv_id)
    status = "✅ Processed" if result else "⏭️ Skipped (duplicate)"
    print(f"  {inv_id}: {status}")

# VIP check — instant!
print(f"\n  Wayne is VIP: {db.is_vip('Wayne Enterprises')}")
print(f"  Luthor is VIP: {db.is_vip('Luthor Corp')}")

db.dashboard()
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Using Lists for Membership Testing 🔥🔥🔥

```python
# ❌ O(n) — gets slower with more data
big_list = list(range(1_000_000))
if 999_999 in big_list:    # Scans up to 1,000,000 items!
    pass

# ✅ O(1) — constant speed regardless of size
big_set = set(range(1_000_000))
if 999_999 in big_set:     # Instant! Hash lookup
    pass

# Rule: If you check "in" more than once and the collection doesn't 
# change, convert to a set
```

---

### Pitfall 2: Unhashable Types Can't Be in Sets or Dict Keys

```python
# Sets and dict keys require HASHABLE items (immutable)

# ✅ Hashable — can be in sets
{1, 2, 3}                    # integers
{"a", "b", "c"}              # strings
{(1, 2), (3, 4)}             # tuples (if contents are hashable)
{frozenset([1, 2, 3])}       # frozensets

# ❌ Unhashable — CANNOT be in sets
# {[1, 2], [3, 4]}           # TypeError: unhashable type: 'list'
# {{"a": 1}}                 # TypeError: unhashable type: 'dict'
# {set()}                    # TypeError: unhashable type: 'set'

# ✅ Convert to hashable equivalent
# list → tuple
# dict → tuple of sorted items: tuple(sorted(d.items()))
# set → frozenset
```

---

### Pitfall 3: Sets Are Unordered

```python
# ❌ Don't rely on set order
names = {"Charlie", "Alice", "Bob"}
print(names)    # Could print in ANY order!
# for i, name in enumerate(names): ← index is meaningless

# ✅ Sort if you need order
for name in sorted(names):
    print(name)    # Alice, Bob, Charlie — guaranteed

# If you need unique AND ordered, use dict.fromkeys()
ordered_unique = list(dict.fromkeys(["c", "a", "b", "a", "c"]))
print(ordered_unique)    # ['c', 'a', 'b'] — insertion order preserved
```

---

### Pitfall 4: Mutating a Dict/Set During Iteration

```python
# ❌ RuntimeError: dictionary changed size during iteration
d = {"a": 1, "b": 2, "c": 3}
# for k in d:
#     if d[k] < 3:
#         del d[k]    # Crash!

# ✅ Iterate over a copy of the keys
for k in list(d.keys()):
    if d[k] < 3:
        del d[k]       # Safe!

# ✅ Or build a new dict with comprehension
d = {k: v for k, v in d.items() if v >= 3}
```

---

### Pitfall 5: `dict.fromkeys()` Shared Reference Trap

```python
# ❌ All values point to the SAME list!
d = dict.fromkeys(["a", "b", "c"], [])
d["a"].append(1)
print(d)    # {'a': [1], 'b': [1], 'c': [1]} — ALL changed!

# ✅ Use a comprehension for independent values
d = {k: [] for k in ["a", "b", "c"]}
d["a"].append(1)
print(d)    # {'a': [1], 'b': [], 'c': []} ✅
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🗂️ Analogy: Three Types of Filing Systems

```
LIST = Filing Cabinet (numbered drawers):
  📁 Drawer 0: Wayne
  📁 Drawer 1: Stark
  📁 Drawer 2: Oscorp
  ✅ "Give me drawer 3" — instant
  🐌 "Is Wayne in here?" — check every drawer

DICT = Labeled Filing System (name → contents):
  🏷️ "Wayne" → {revenue: 2.5M, type: Merger}
  🏷️ "Stark" → {revenue: 800K, type: Patent}
  ✅ "Give me Wayne's file" — instant
  ✅ "Is Wayne in here?" — instant

SET = Membership Roster (names only, no duplicates):
  ✅ Wayne, Stark, Oscorp (unique members)
  ✅ "Is Wayne a member?" — instant
  ✅ "Who's in BOTH clubs?" — instant (intersection)
  ❌ No associated data, no order
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Choose the Structure**
```
For each scenario, state which structure (list, dict, set) is best and why:

a) Track the order of tasks completed today
b) Count how many times each word appears in a document
c) Track unique visitors to a website
d) Map student names to their grades
e) Store a series of temperature readings
f) Find common friends between two users
g) Implement an undo history
```

**Exercise 2: Set Operations**
```
Given:
  cs_students = {"Alice", "Bob", "Charlie", "Diana"}
  math_students = {"Bob", "Diana", "Eve", "Frank"}

Find:
  a) Students in both (intersection)
  b) All students (union)
  c) CS only (difference)
  d) In exactly one class (symmetric diff)
  e) Is CS a subset of all students?
```

**Exercise 3: Performance Test**
```
Create a list and a set with 100,000 numbers.
Time the membership test for the LAST element.
Print the speed ratio.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Data Structure Migration**
```
Given a list of 10,000 random client names with duplicates:
  a) Deduplicate (list → set)
  b) Create a frequency count (list → Counter)
  c) Create a name → index lookup (list → dict)
  d) Group by first letter (list → defaultdict)

Benchmark each conversion.
```

**Exercise 5: Anagram Finder**
```
Given a list of words, find all groups of anagrams.

Use:
  - sorted() to normalize words
  - defaultdict(list) to group anagrams
  - set for deduplication

Input: ["eat", "tea", "tan", "ate", "nat", "bat"]
Output: [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
```

**Exercise 6: Inventory System**
```
Build an inventory system using the right structures:
  - LIST: transaction history (ordered)
  - DICT: product catalog (name → details)  
  - SET: out-of-stock items (fast membership)
  - COUNTER: sales counts
  - deque: recent transactions (limited size)

Implement: add_product, sell, restock, get_bestsellers, report
```

---

### 🔴 Advanced Exercises

**Exercise 7: Graph with Mixed Structures**
```
Represent a social network:
  - DICT: user → set of friends (adjacency list)
  - SET: mutual friends (intersection)
  - LIST: friend suggestions (friends-of-friends, ordered by count)

Implement: add_user, add_friendship, mutual_friends,
           suggest_friends, degrees_of_separation
```

**Exercise 8: Performance Optimizer**
```
Given poorly-performing code that uses lists for everything:
  a) Profile it to find bottlenecks
  b) Replace lists with dicts/sets where appropriate
  c) Benchmark before and after
  d) Document why each change helps

The code should process 100K+ records.
```

**Exercise 9: Multi-Index Database**
```
Build a database that maintains multiple indexes:
  - Primary: LIST of records (ordered)
  - Name index: DICT mapping name → record
  - Category index: defaultdict(list) mapping category → records
  - Active set: SET for instant membership check

All indexes stay in sync on insert/update/delete.
Benchmark lookups by index vs by linear scan.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Using list for frequent lookups
banned_users = ["spammer1", "spammer2", "bot99", ...]  # 50,000 items
for user in incoming_users:
    if user in banned_users:   # O(n) each time!
        block(user)

# Bug 2: Losing duplicates unexpectedly
scores = [88, 92, 88, 75, 92, 100]
unique_scores = set(scores)
print(sorted(unique_scores))    # [75, 88, 92, 100] — lost count of 88s!

# Bug 3: Mutable key in dict
record = {[1, 2]: "data"}    # TypeError!

# Bug 4: Dict.fromkeys trap
defaults = dict.fromkeys(["a", "b", "c"], [])
defaults["a"].append(1)
print(defaults["b"])    # Expected: []

# Bug 5: Modifying set during iteration
numbers = {1, 2, 3, 4, 5}
for n in numbers:
    if n % 2 == 0:
        numbers.remove(n)    # RuntimeError!
```

### Fixed Code ✅

```python
# Fix 1: Convert to set for O(1) lookups
banned_users = set(["spammer1", "spammer2", "bot99", ...])   # ✅
for user in incoming_users:
    if user in banned_users:    # O(1) each time!
        block(user)

# Fix 2: Use Counter to keep counts
from collections import Counter
scores = [88, 92, 88, 75, 92, 100]
score_counts = Counter(scores)   # ✅ Counter({88: 2, 92: 2, 75: 1, 100: 1})

# Fix 3: Use tuple (hashable) as key
record = {(1, 2): "data"}     # ✅ Tuples are hashable

# Fix 4: Use comprehension for independent defaults
defaults = {k: [] for k in ["a", "b", "c"]}   # ✅
defaults["a"].append(1)
print(defaults["b"])    # [] ✅

# Fix 5: Iterate over a frozen copy
numbers = {1, 2, 3, 4, 5}
for n in list(numbers):       # ✅ Iterate over a list copy
    if n % 2 == 0:
        numbers.remove(n)
print(numbers)             # {1, 3, 5} ✅
```

---

## 8. 🌍 Real-World Application

| Scenario | Best Structure | Why |
|----------|---------------|-----|
| User session store | **Dict** (session_id → data) | O(1) lookup by ID |
| Rate limiter | **Dict** (IP → Counter) + **Set** (blocked IPs) | Fast lookup + membership |
| URL shortener | **Dict** (short → long) | Key-value mapping |
| Tag system | **Set** per item | Unique tags, set operations |
| Message queue | **deque** | O(1) on both ends |
| Leaderboard | **sorted List** + **Dict** (player → score) | Order + lookup |
| Spell checker | **Set** of valid words | O(1) membership |
| Cache | **Dict** (key → value) | Fast get/set |
| Transaction log | **List** | Time-ordered, indexed |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the Big-O for `x in my_list` vs `x in my_set`?

**Q2.** When should you use a list over a dict or set?

**Q3.** Can you use a list as a dictionary key? Why or why not?

**Q4.** What is `collections.Counter`? Give an example.

**Q5.** What are the four main set operations? Give the operator for each.

**Q6.** What is `defaultdict`? How does it differ from a regular dict?

**Q7.** How do you efficiently deduplicate a list while preserving order?

**Q8.** Why is `dict.fromkeys(keys, [])` dangerous?

**Q9.** When should you use `deque` instead of a list?

**Q10.** You have 1M records and need to check if each is in a "blocked" list of 50K items. What structure for the blocked list?

---

### 📋 Answer Key

> **Q1.** List: O(n) — linear scan. Set: O(1) — hash lookup.

> **Q2.** When order matters, you need indexed access, duplicates are allowed, or you're doing stack/queue operations.

> **Q3.** No. Lists are mutable (unhashable). Use a tuple instead.

> **Q4.** A dict subclass for counting. `Counter(["a","b","a"])` → `Counter({'a': 2, 'b': 1})`.

> **Q5.** Union `|`, intersection `&`, difference `-`, symmetric difference `^`.

> **Q6.** A dict that auto-creates missing keys with a factory function. `defaultdict(list)` → missing key returns `[]`.

> **Q7.** `list(dict.fromkeys(items))` — dict preserves insertion order and deduplicates.

> **Q8.** All keys share the **same** list object. Append to one, all change.

> **Q9.** When you need fast append/pop from **both** ends (O(1) vs O(n) for list.insert(0)).

> **Q10.** **Set**. Convert the 50K blocked list to a set for O(1) membership testing.

---

## 🎬 End of Lesson Scene

**Louis:** *(checking the system)* "Client lookup: 0.002 seconds. Down from 45."

**Harvey:** "Same data. Same logic. Different structure."

**Mike:** "The right data structure isn't about style — it's about performance. Lists for order. Dicts for lookup. Sets for membership. Next: accessing and modifying object attributes."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Big-O performance comparison table | ✅ |
| 2 | Lists — when order and indexing matter | ✅ |
| 3 | Dicts — when key-based lookup matters | ✅ |
| 4 | Sets — when uniqueness/membership matter | ✅ |
| 5 | Set operations: union, intersection, difference | ✅ |
| 6 | Decision flowchart for choosing structures | ✅ |
| 7 | `collections` module (Counter, defaultdict, deque, namedtuple) | ✅ |
| 8 | Conversion patterns between structures | ✅ |
| 9 | Performance benchmarking | ✅ |
| 10 | Hashability requirements | ✅ |

---

> **Next Lesson →** Level 2, Lesson 7: *Access and Modify Python Object Attributes*

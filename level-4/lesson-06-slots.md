# 🏛️ Level 4 – Advancing | Lesson 6: Use `__slots__` to Create Memory-Efficient Python Classes

## *"Lean and Mean"*

> **O'Reilly Skill Objective:** *Use `__slots__` to create memory-efficient classes.*

> **Previously Learned:** Classes, `__init__`, `__dict__`, `@property`, inheritance, `super()`, multiple inheritance, mixins

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

*The firm's case management system loads 500,000 client records into memory. The server crashes.*

**Louis:** "We're using 4 GB of RAM for half a million Client objects. The server only has 8."

**Mike:** "Each Client object carries a `__dict__` — a full hash table — even though every client has the same three fields: `name`, `revenue`, `tier`. That's a LOT of per-instance overhead."

**Harvey:** "Slim them down. Fixed fields, no dict, no wasted memory."

```python
class Client:
    __slots__ = ('name', 'revenue', 'tier')
    
    def __init__(self, name, revenue, tier):
        self.name = name
        self.revenue = revenue
        self.tier = tier

# Result: 40-50% less memory per instance!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "By default, every Python object carries a dictionary. `__slots__` replaces it with a fixed-size array. Lean and mean."

| Feature | Without `__slots__` | With `__slots__` |
|---------|:-------------------:|:----------------:|
| Per-instance `__dict__` | ✅ Yes (flexible) | ❌ No (fixed) |
| Memory per instance | ~200+ bytes overhead | ~60-80 bytes |
| Dynamic attribute addition | ✅ `obj.anything = 1` | ❌ `AttributeError` |
| Attribute access speed | Normal | ~5-10% faster |
| Works with `@property` | ✅ | ✅ |
| Pickling | ✅ | ⚠️ Needs extra work |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 How Python Normally Stores Attributes ⭐

```python
class RegularClient:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

c = RegularClient("Wayne", 2500000)

# Every instance has its OWN __dict__
print(c.__dict__)    # {'name': 'Wayne', 'revenue': 2500000}

# __dict__ is a full hash table — flexible but heavy
c.tier = "Platinum"              # ✅ Add any attribute dynamically
c.random_field = "anything"      # ✅ No restriction!
print(c.__dict__)    
# {'name': 'Wayne', 'revenue': 2500000, 'tier': 'Platinum', 'random_field': 'anything'}

import sys
print(sys.getsizeof(c.__dict__))    # ~104 bytes (empty) + per-key overhead
```

---

### 3.2 `__slots__` Basics ⭐⭐

```python
class SlottedClient:
    __slots__ = ('name', 'revenue', 'tier')
    
    def __init__(self, name, revenue, tier="Bronze"):
        self.name = name
        self.revenue = revenue
        self.tier = tier

c = SlottedClient("Wayne", 2500000, "Platinum")

# No __dict__!
# print(c.__dict__)    # AttributeError: 'SlottedClient' has no attribute '__dict__'

# Fixed attributes only
print(c.name)       # Wayne
print(c.revenue)    # 2500000

# ❌ Cannot add dynamic attributes
# c.email = "wayne@example.com"    # AttributeError: 'SlottedClient' has no attribute 'email'

# ✅ Can still modify existing slot attributes
c.revenue = 3000000
c.tier = "Platinum"
```

---

### 3.3 Memory Comparison — The Real Savings ⭐⭐

```python
import sys

class Regular:
    def __init__(self, name, revenue, tier):
        self.name = name
        self.revenue = revenue
        self.tier = tier

class Slotted:
    __slots__ = ('name', 'revenue', 'tier')
    def __init__(self, name, revenue, tier):
        self.name = name
        self.revenue = revenue
        self.tier = tier

# Single instance comparison
r = Regular("Wayne", 2500000, "Platinum")
s = Slotted("Wayne", 2500000, "Platinum")

regular_size = sys.getsizeof(r) + sys.getsizeof(r.__dict__)
slotted_size = sys.getsizeof(s)

print(f"Regular: {regular_size} bytes")    # ~200+ bytes
print(f"Slotted: {slotted_size} bytes")    # ~56-64 bytes
print(f"Savings: {1 - slotted_size/regular_size:.0%}")

# At scale: 500,000 instances
n = 500_000
regular_total = regular_size * n
slotted_total = slotted_size * n

print(f"\n500K Regular: {regular_total / 1024 / 1024:.1f} MB")
print(f"500K Slotted: {slotted_total / 1024 / 1024:.1f} MB")
print(f"Saved: {(regular_total - slotted_total) / 1024 / 1024:.1f} MB")
```

---

### 3.4 `__slots__` with Methods and Properties ⭐

```python
class Client:
    __slots__ = ('_name', '_revenue', '_tier')
    
    def __init__(self, name, revenue, tier="Bronze"):
        self._name = name
        self._revenue = revenue
        self._tier = tier
    
    # Methods work normally!
    def display(self):
        return f"{self._name}: ${self._revenue:,.2f} [{self._tier}]"
    
    # @property works with slots — use underscore-prefixed slot names
    @property
    def name(self):
        return self._name
    
    @property
    def revenue(self):
        return self._revenue
    
    @revenue.setter
    def revenue(self, value):
        if value < 0:
            raise ValueError("Revenue cannot be negative")
        self._revenue = value
    
    @property
    def tier(self):
        return self._tier
    
    # Class methods and static methods work too
    @classmethod
    def from_dict(cls, data):
        return cls(data["name"], data["revenue"], data.get("tier", "Bronze"))
    
    @staticmethod
    def classify(revenue):
        if revenue >= 1_000_000: return "Platinum"
        if revenue >= 500_000: return "Gold"
        return "Bronze"
    
    # Dunder methods work!
    def __repr__(self):
        return f"Client('{self._name}', {self._revenue}, '{self._tier}')"
    
    def __eq__(self, other):
        if not isinstance(other, Client):
            return NotImplemented
        return self._name == other._name

wayne = Client("Wayne Enterprises", 2500000, "Platinum")
print(wayne.display())
print(wayne.revenue)
wayne.revenue = 3000000    # Property setter ✅
print(repr(wayne))
```

---

### 3.5 `__slots__` with Inheritance ⭐⭐

```python
class Employee:
    __slots__ = ('name', 'employee_id')
    
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id

class Partner(Employee):
    __slots__ = ('equity', 'hourly_rate')    # ONLY new attributes!
    
    def __init__(self, name, employee_id, equity, hourly_rate):
        super().__init__(name, employee_id)
        self.equity = equity
        self.hourly_rate = hourly_rate

harvey = Partner("Harvey", "P001", 0.15, 450)
print(harvey.name)           # ✅ From Employee's slots
print(harvey.equity)         # ✅ From Partner's slots

# ⚠️ DON'T redeclare parent slots in child!
class BadPartner(Employee):
    __slots__ = ('name', 'equity')    # ❌ 'name' already in Employee's slots!
    # Creates duplicate slot — wastes memory, may cause bugs!

# ✅ Only declare NEW attributes in child's __slots__
class GoodPartner(Employee):
    __slots__ = ('equity',)    # Only the new one!
```

---

### 3.6 Allowing `__dict__` Alongside `__slots__` ⭐

```python
# Add '__dict__' to __slots__ for hybrid approach
class FlexibleClient:
    __slots__ = ('name', 'revenue', '__dict__')
    
    def __init__(self, name, revenue):
        self.name = name          # Stored in slot (fast)
        self.revenue = revenue    # Stored in slot (fast)
    
c = FlexibleClient("Wayne", 2500000)
c.name = "Wayne Enterprises"     # Slot attribute ✅
c.tier = "Platinum"               # Dynamic attribute via __dict__ ✅
c.notes = "VIP client"            # Also dynamic ✅

print(c.name)                     # Slot
print(c.tier)                     # __dict__
print(c.__dict__)                 # {'tier': 'Platinum', 'notes': 'VIP client'}

# Benefits: common attributes are fast/lean, extras allowed
```

---

### 3.7 `__slots__` with `__weakref__`

```python
import weakref

# Regular classes support weak references automatically
class Regular:
    pass

r = Regular()
ref = weakref.ref(r)    # ✅ Works

# Slotted classes need '__weakref__' explicitly
class Slotted:
    __slots__ = ('name',)

s = Slotted()
# ref = weakref.ref(s)    # TypeError: cannot create weak reference!

# ✅ Fix: add __weakref__ to slots
class SlottedWithRef:
    __slots__ = ('name', '__weakref__')

s = SlottedWithRef()
ref = weakref.ref(s)    # ✅ Works now
```

---

### 3.8 `__slots__` with Dataclasses (Python 3.10+)

```python
from dataclasses import dataclass

# Python 3.10+: dataclass with slots=True
@dataclass(slots=True)
class Client:
    name: str
    revenue: float
    tier: str = "Bronze"

wayne = Client("Wayne", 2500000, "Platinum")
print(wayne)    # Client(name='Wayne', revenue=2500000, tier='Platinum')

# No __dict__!
# wayne.email = "test"    # AttributeError! ✅

# Pre-3.10: manual approach
@dataclass
class ClientManual:
    __slots__ = ('name', 'revenue', 'tier')
    name: str
    revenue: float
    tier: str = "Bronze"
# ⚠️ This can have quirks — prefer slots=True parameter on 3.10+
```

---

### 3.9 `__slots__` with NamedTuple and Frozen Dataclass

```python
from typing import NamedTuple
from dataclasses import dataclass

# NamedTuple: already memory-efficient (tuple-based)
class ClientTuple(NamedTuple):
    name: str
    revenue: float
    tier: str = "Bronze"

# Frozen dataclass with slots: immutable + efficient
@dataclass(frozen=True, slots=True)
class ClientFrozen:
    name: str
    revenue: float
    tier: str = "Bronze"

# Comparison of approaches:
import sys

class DictClient:
    def __init__(self, n, r, t): self.name, self.revenue, self.tier = n, r, t

class SlotClient:
    __slots__ = ('name', 'revenue', 'tier')
    def __init__(self, n, r, t): self.name, self.revenue, self.tier = n, r, t

dc = DictClient("W", 1, "B")
sc = SlotClient("W", 1, "B")
tc = ClientTuple("W", 1, "B")
fc = ClientFrozen("W", 1, "B")

print(f"Dict:        {sys.getsizeof(dc) + sys.getsizeof(dc.__dict__)} bytes")
print(f"Slotted:     {sys.getsizeof(sc)} bytes")
print(f"NamedTuple:  {sys.getsizeof(tc)} bytes")
print(f"Frozen+Slot: {sys.getsizeof(fc)} bytes")
```

---

### 3.10 Solving Louis's Problem — Memory-Optimized System

```python
# ============================================
# Pearson Specter Litt — Memory-Optimized Records
# __slots__ for 500K+ client records
# ============================================

import sys
from dataclasses import dataclass

class Client:
    """Memory-optimized client record."""
    __slots__ = ('name', 'revenue', 'tier', 'attorney_id', 'active')
    
    TIERS = {"Platinum": 1_000_000, "Gold": 500_000, "Silver": 100_000}
    
    def __init__(self, name, revenue, attorney_id, active=True):
        self.name = name
        self.revenue = revenue
        self.tier = self._classify(revenue)
        self.attorney_id = attorney_id
        self.active = active
    
    @staticmethod
    def _classify(revenue):
        if revenue >= 1_000_000: return "Platinum"
        if revenue >= 500_000: return "Gold"
        if revenue >= 100_000: return "Silver"
        return "Bronze"
    
    def __repr__(self):
        return f"Client('{self.name}', ${self.revenue:,}, '{self.tier}')"


class CaseRecord:
    """Memory-optimized case record."""
    __slots__ = ('case_id', 'client_name', 'attorney_id', 'hours', 'status')
    
    def __init__(self, case_id, client_name, attorney_id, hours=0, status="open"):
        self.case_id = case_id
        self.client_name = client_name
        self.attorney_id = attorney_id
        self.hours = hours
        self.status = status
    
    def __repr__(self):
        return f"Case({self.case_id}, '{self.client_name}', {self.hours}h)"


class RegularClient:
    """Non-optimized version for comparison."""
    def __init__(self, name, revenue, attorney_id, active=True):
        self.name = name
        self.revenue = revenue
        self.tier = Client._classify(revenue)
        self.attorney_id = attorney_id
        self.active = active


# === Benchmark ===
print("=" * 60)
print(f"{'🏛️  MEMORY OPTIMIZATION BENCHMARK':^60}")
print("=" * 60)

N = 10_000    # Scale for demo (use 500_000 in production)

# Create instances
import random
names = [f"Client_{i}" for i in range(N)]
revenues = [random.randint(10_000, 5_000_000) for _ in range(N)]

regular_clients = [
    RegularClient(names[i], revenues[i], f"A{i%10}")
    for i in range(N)
]

slotted_clients = [
    Client(names[i], revenues[i], f"A{i%10}")
    for i in range(N)
]

# Measure memory
def total_size_regular(objs):
    return sum(sys.getsizeof(o) + sys.getsizeof(o.__dict__) for o in objs)

def total_size_slotted(objs):
    return sum(sys.getsizeof(o) for o in objs)

reg_mem = total_size_regular(regular_clients)
slot_mem = total_size_slotted(slotted_clients)

print(f"\n📊 Results for {N:,} instances:")
print(f"  Regular (with __dict__): {reg_mem:>12,} bytes ({reg_mem/1024/1024:.2f} MB)")
print(f"  Slotted (__slots__):     {slot_mem:>12,} bytes ({slot_mem/1024/1024:.2f} MB)")
print(f"  Savings:                 {reg_mem - slot_mem:>12,} bytes ({(1-slot_mem/reg_mem)*100:.1f}%)")

# Per-instance
print(f"\n📏 Per instance:")
print(f"  Regular: {reg_mem // N} bytes")
print(f"  Slotted: {slot_mem // N} bytes")

# Extrapolate to 500K
factor = 500_000 / N
print(f"\n📈 Projected for 500,000 instances:")
print(f"  Regular: {reg_mem * factor / 1024 / 1024:.1f} MB")
print(f"  Slotted: {slot_mem * factor / 1024 / 1024:.1f} MB")
print(f"  Saved:   {(reg_mem - slot_mem) * factor / 1024 / 1024:.1f} MB")

# Tier distribution
from collections import Counter
tiers = Counter(c.tier for c in slotted_clients)
print(f"\n📊 Tier Distribution:")
for tier, count in tiers.most_common():
    print(f"  {tier:>10}: {count:>6,} ({count/N*100:.1f}%)")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Can't Add Dynamic Attributes 🔥

```python
class Slotted:
    __slots__ = ('name',)
    def __init__(self, name):
        self.name = name

s = Slotted("Harvey")
# s.title = "Partner"    # AttributeError!

# ✅ If you need flexibility, add '__dict__' to slots
class Flexible:
    __slots__ = ('name', '__dict__')
```

---

### Pitfall 2: Inheritance Without Slots Breaks Optimization

```python
class SlottedParent:
    __slots__ = ('x',)

class NoSlotChild(SlottedParent):
    pass    # No __slots__ defined!

c = NoSlotChild()
c.x = 10        # Slot from parent ✅
c.anything = 20  # ❌ Child has __dict__! Optimization lost!

# ✅ Define __slots__ in every level
class SlotChild(SlottedParent):
    __slots__ = ('y',)    # Continue the optimization
```

---

### Pitfall 3: Duplicate Slot Names in Inheritance

```python
class Parent:
    __slots__ = ('name',)

class Child(Parent):
    __slots__ = ('name', 'age')    # ⚠️ 'name' duplicated!
    # Warning in some Python versions, wastes memory

# ✅ Only new attributes in child
class Child(Parent):
    __slots__ = ('age',)    # ✅ 'name' inherited from Parent
```

---

### Pitfall 4: No Default Values in `__slots__`

```python
# ❌ __slots__ is just a sequence of attribute NAMES
class Bad:
    __slots__ = {'name': 'Unknown'}    # Dict? Nope — just uses keys as names!
    # This works syntactically but doesn't set defaults

# ✅ Set defaults in __init__
class Good:
    __slots__ = ('name', 'tier')
    def __init__(self, name="Unknown", tier="Bronze"):
        self.name = name
        self.tier = tier
```

---

### Pitfall 5: Pickling/Serialization Issues

```python
import pickle

class Slotted:
    __slots__ = ('name', 'value')
    def __init__(self, name, value):
        self.name = name
        self.value = value

s = Slotted("test", 42)

# Pickling works but needs __getstate__/__setstate__ for custom behavior
data = pickle.dumps(s)
s2 = pickle.loads(data)
print(s2.name, s2.value)    # test 42 — works by default in Python 3

# For JSON serialization, manually extract attributes:
def to_dict(slotted_obj):
    return {slot: getattr(slotted_obj, slot) 
            for slot in slotted_obj.__slots__
            if hasattr(slotted_obj, slot)}

print(to_dict(s))    # {'name': 'test', 'value': 42}
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📁 Analogy: __slots__ = Fixed Filing Cabinet

```
WITHOUT __slots__ (dictionary-based):
  📂 Every client gets a filing CABINET with:
     • Expandable folders (add any field anytime)
     • Label maker (dynamic keys)
     • Extra storage at the back (hash table overhead)
  
  500K cabinets × heavy cabinet = 🏢 office building full

WITH __slots__ (fixed slots):
  📋 Every client gets a CLIPBOARD with:
     • 3 fixed lines: Name, Revenue, Tier
     • No extras — no expansion slots
     • Lightweight — just a card
  
  500K clipboards × light clipboard = 📦 one filing room

DONNA'S RULE:
  "If every client has the same three fields,
   don't give them each a filing cabinet.
   Give them a clipboard.
   Save the cabinets for the clients who need them."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Basic Slots**
```
Create a Point class with __slots__ = ('x', 'y').
Prove: you can set x and y, but NOT z.
Compare memory with a regular Point class.
```

**Exercise 2: Slots with Methods**
```
Create an Employee with slots + methods:
  __slots__ = ('name', 'salary')
  display(), raise_salary(percent), __repr__
Verify all methods work normally.
```

**Exercise 3: Slots with Property**
```
Create a Temperature class:
  __slots__ = ('_celsius',)
  @property celsius, fahrenheit (computed)
  Validate: temperature can't go below absolute zero
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Inheritance Chain**
```
Build: Shape(color) → Circle(color, radius) → Sphere(color, radius)
All with __slots__. Each adds only NEW attributes.
Verify no __dict__ at any level.
```

**Exercise 5: Hybrid Slots**
```
Build a class with common slots + optional __dict__:
  __slots__ = ('id', 'name', '__dict__')
  Core fields in slots, metadata in dict.
  Benchmark vs pure-dict and pure-slot versions.
```

**Exercise 6: Memory Benchmark**
```
Create 100,000 instances of a 5-field class:
  a) Regular class
  b) __slots__ class
  c) NamedTuple
  d) dataclass(slots=True)
Measure and compare memory usage for all four.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Slotted ORM Model**
```
Build a lightweight ORM with __slots__:
  class Model: __slots__ auto-generated from field definitions
  class User(Model): name = StringField(), age = IntField()
  user = User(name="Harvey", age=40)
```

**Exercise 8: Slots + Weakref Cache**
```
Build an object cache using __slots__ + __weakref__:
  Cache holds weak references to slotted objects
  Auto-evicts when objects are garbage collected
```

**Exercise 9: Dynamic Slots**
```
Build a metaclass that auto-generates __slots__
from __init__ parameter inspection:

class Auto(metaclass=AutoSlotMeta):
    def __init__(self, name, age, email):
        ...
# __slots__ automatically set to ('name', 'age', 'email')
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Adding dynamic attribute to slotted class
class Record:
    __slots__ = ('name',)
    def __init__(self, name):
        self.name = name

r = Record("test")
r.data = []    # AttributeError!

# Bug 2: Child doesn't define __slots__
class Parent:
    __slots__ = ('x',)
class Child(Parent):
    pass
c = Child()
c.anything = 1    # Works! __dict__ is back! Optimization lost!

# Bug 3: Duplicate slot names
class Base:
    __slots__ = ('name',)
class Sub(Base):
    __slots__ = ('name', 'age')    # 'name' duplicated!

# Bug 4: Trying to use __dict__ on slotted object
class Slotted:
    __slots__ = ('x', 'y')
s = Slotted()
# vars(s)    # TypeError!

# Bug 5: Mixin breaks slots
class MyMixin:
    def helper(self):
        self.temp = True    # AttributeError if mixed with slotted class!
```

### Fixed Code ✅

```python
# Fix 1: Add the attribute to __slots__
class Record:
    __slots__ = ('name', 'data')

# Fix 2: Define __slots__ in child
class Child(Parent):
    __slots__ = ('y',)    # Continue optimization

# Fix 3: Only new attributes
class Sub(Base):
    __slots__ = ('age',)    # 'name' inherited

# Fix 4: Access slots by name
s = Slotted()
s.x = 1; s.y = 2
attrs = {slot: getattr(s, slot) for slot in s.__slots__}

# Fix 5: Mixin uses hasattr check
class MyMixin:
    def helper(self):
        if not hasattr(self, '_temp'):
            # Only set if slot exists or __dict__ available
            try:
                self._temp = True
            except AttributeError:
                pass
```

---

## 8. 🌍 Real-World Application

### When to Use `__slots__`

| Scenario | Use `__slots__`? | Why |
|----------|:----------------:|-----|
| 500K+ instances | ✅ | Memory savings add up |
| Data records (POD) | ✅ | Fixed fields, no dynamism needed |
| ORM models | ⚠️ | Frameworks may need `__dict__` |
| Configuration objects | ❌ | Few instances, flexibility matters |
| Base classes for APIs | ❌ | Users may need dynamic attrs |
| Internal data structures | ✅ | Performance-critical code |
| Dataclasses (3.10+) | ✅ | `@dataclass(slots=True)` |

---

## 9. ✅ Knowledge Check

---

**Q1.** What does `__slots__` do?

**Q2.** What is the main benefit of `__slots__`?

**Q3.** What happens if you try to add a dynamic attribute to a slotted class?

**Q4.** How do you allow both slots AND dynamic attributes?

**Q5.** How does `__slots__` work with inheritance?

**Q6.** Can you use `@property` with `__slots__`?

**Q7.** Why shouldn't you repeat slot names in a child class?

**Q8.** How do you serialize a slotted object to dict?

**Q9.** What is `__weakref__` and when do you need it in slots?

**Q10.** When should you NOT use `__slots__`?

---

### 📋 Answer Key

> **Q1.** Replaces the per-instance `__dict__` with fixed-size slot descriptors on the class, preventing dynamic attribute creation.

> **Q2.** Memory efficiency — each instance uses 40-60% less memory because there's no per-instance dictionary overhead.

> **Q3.** `AttributeError` — slotted classes only allow attributes listed in `__slots__`.

> **Q4.** Include `'__dict__'` in `__slots__`: `__slots__ = ('name', '__dict__')`. Slotted attrs are fast, extras go to dict.

> **Q5.** Each class in the chain should define `__slots__` with only its NEW attributes. If any class omits `__slots__`, its instances get `__dict__` back.

> **Q6.** Yes — `@property` works normally. Use underscore-prefixed slot names (`_name`) with property getters/setters.

> **Q7.** It wastes memory by creating duplicate slot descriptors and can cause confusing behavior.

> **Q8.** `{slot: getattr(obj, slot) for slot in obj.__slots__ if hasattr(obj, slot)}`.

> **Q9.** A slot needed for `weakref.ref()` to work. Regular classes include it automatically; slotted classes must add `'__weakref__'` explicitly.

> **Q10.** When you have few instances (savings are negligible), need dynamic attributes, or use frameworks that depend on `__dict__`.

---

## 🎬 End of Lesson Scene

**Louis:** "Memory usage?"

**Mike:** "Down 50%. Same 500,000 records, half the RAM. The server is stable."

**Louis:** "What changed?"

**Mike:** "`__slots__`. Five characters in the class definition. Each object went from a filing cabinet to a clipboard."

**Harvey:** "Lean and mean. Next: iterators — building custom sequences with `__iter__` and `__next__`."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `__dict__` overhead and how Python stores attributes | ✅ |
| 2 | `__slots__` syntax and basic usage | ✅ |
| 3 | Memory savings benchmark | ✅ |
| 4 | Slots with methods, properties, and dunders | ✅ |
| 5 | Slots with inheritance (only new attrs) | ✅ |
| 6 | Hybrid approach (`__dict__` in slots) | ✅ |
| 7 | `__weakref__` for weak reference support | ✅ |
| 8 | `@dataclass(slots=True)` (Python 3.10+) | ✅ |
| 9 | Comparison with NamedTuple and frozen dataclass | ✅ |
| 10 | When to use vs when to avoid `__slots__` | ✅ |

---

> **Next Lesson →** Level 4, Lesson 7: *Create Iterators with `__iter__` and `__next__` in Python*

# 🏛️ Level 3 – Building | Lesson 4: Use @property to Define Read-Only Attributes in Python

## *"The Firewall"*

> **O'Reilly Skill Objective:** *Use the `@property` decorator to create read-only attributes and controlled access to object data.*

> **Previously Learned:** Classes, `__init__`, `self`, instance methods, `@classmethod`, `@staticmethod`, dunder methods, getattr/setattr, data structures

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

*Thursday morning. A junior associate broke the billing system.*

**Louis:** "Some associate set a client's revenue to NEGATIVE. Now the reports show we owe Wayne Enterprises $2.5 million."

```python
wayne = Client("Wayne Enterprises", 2500000)
wayne.revenue = -2500000    # Anyone can set anything! No protection!
```

**Harvey:** "Public attributes are an open door. Anyone can walk in and change whatever they want."

**Mike:** "We need a firewall. Something that looks like a simple attribute from the outside — `client.revenue` — but runs validation code behind the scenes. And some attributes should be read-only: case IDs, creation dates, computed values."

**Harvey:** "`@property`. It lets you put business logic behind the dot."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Three levels of attribute access."

| Level | Syntax | Protection |
|-------|--------|-----------|
| **Public attribute** | `self.name = "Wayne"` | ❌ None — anyone can read/write anything |
| **Property (getter only)** | `@property` | ✅ Read-only — can't set from outside |
| **Property (getter + setter)** | `@property` + `@x.setter` | ✅ Controlled — validation on write |

**Harvey:** "The genius of `@property` is that the interface stays the same. `client.revenue` looks identical whether it's a plain attribute or a property. No parentheses, no `get_revenue()` method call. It's invisible validation."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 The Problem: Unprotected Attributes

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

wayne = Client("Wayne", 2500000)

# Anyone can set attributes to garbage:
wayne.revenue = -999         # ❌ Negative revenue?
wayne.revenue = "lots"       # ❌ String instead of number?
wayne.name = ""              # ❌ Empty name?
wayne.name = 42              # ❌ Integer instead of string?
```

**Mike:** "The old-school fix is getters and setters — but they're ugly."

```python
# Java-style getters/setters — NOT Pythonic!
class Client:
    def __init__(self, name, revenue):
        self._name = name
        self._revenue = revenue
    
    def get_revenue(self):
        return self._revenue
    
    def set_revenue(self, value):
        if value < 0:
            raise ValueError("Revenue cannot be negative")
        self._revenue = value

wayne = Client("Wayne", 2500000)
wayne.set_revenue(3000000)        # Ugly! Not Pythonic!
print(wayne.get_revenue())        # Ugly!
```

---

### 3.2 `@property` — Read-Only Attributes ⭐⭐

```python
class Client:
    def __init__(self, name, revenue):
        self._name = name
        self._revenue = revenue
        self._created = __import__('datetime').datetime.now()
    
    @property
    def name(self):
        """Read-only: client name cannot be changed."""
        return self._name
    
    @property
    def revenue(self):
        """Read-only: revenue exposed as property."""
        return self._revenue
    
    @property
    def created(self):
        """Read-only: creation timestamp."""
        return self._created
    
    @property
    def tier(self):
        """Computed property — calculated every time."""
        if self._revenue >= 1000000: return "Platinum"
        elif self._revenue >= 500000: return "Gold"
        return "Silver"

wayne = Client("Wayne Enterprises", 2500000)

# Access looks like a normal attribute — no parentheses!
print(wayne.name)        # Wayne Enterprises
print(wayne.revenue)     # 2500000
print(wayne.tier)        # Platinum
print(wayne.created)     # 2026-03-25 09:05:59.123456

# But you CAN'T set them!
# wayne.name = "Changed"    # AttributeError: property 'name' has no setter
# wayne.tier = "Bronze"     # AttributeError: property 'tier' has no setter
```

---

### 3.3 `@property` with Setter — Validated Access ⭐⭐

```python
class Client:
    def __init__(self, name, revenue):
        # Use the setters even in __init__ for validation!
        self.name = name           # Goes through @name.setter
        self.revenue = revenue     # Goes through @revenue.setter
    
    # --- Name property ---
    
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError(f"Name must be a string, got {type(value).__name__}")
        if not value.strip():
            raise ValueError("Name cannot be empty")
        self._name = value.strip()
    
    # --- Revenue property ---
    
    @property
    def revenue(self):
        return self._revenue
    
    @revenue.setter
    def revenue(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"Revenue must be numeric, got {type(value).__name__}")
        if value < 0:
            raise ValueError(f"Revenue cannot be negative: {value}")
        self._revenue = float(value)
    
    # --- Computed (read-only) ---
    
    @property
    def tier(self):
        if self._revenue >= 1000000: return "💎 Platinum"
        elif self._revenue >= 500000: return "🥇 Gold"
        return "🥈 Silver"

# Validation happens automatically!
wayne = Client("Wayne Enterprises", 2500000)

wayne.revenue = 3000000        # ✅ Valid — setter runs validation
print(wayne.revenue)            # 3000000.0

# wayne.revenue = -500          # ❌ ValueError: Revenue cannot be negative
# wayne.revenue = "lots"        # ❌ TypeError: Revenue must be numeric
# wayne.name = ""               # ❌ ValueError: Name cannot be empty
# wayne.name = 42               # ❌ TypeError: Name must be a string

# Even __init__ goes through validation:
# Client("", 2500000)           # ❌ ValueError: Name cannot be empty
# Client("Wayne", -100)         # ❌ ValueError: Revenue cannot be negative
```

---

### 3.4 `@property` with Deleter

```python
class Employee:
    def __init__(self, name, email):
        self.name = name
        self._email = email
    
    @property
    def email(self):
        return self._email
    
    @email.setter
    def email(self, value):
        if "@" not in value:
            raise ValueError("Invalid email address")
        self._email = value
    
    @email.deleter
    def email(self):
        """Allow clearing the email."""
        print(f"  Clearing email for {self.name}")
        self._email = None

emp = Employee("Mike", "mike@psl.com")
print(emp.email)          # mike@psl.com

emp.email = "mike@firm.com"   # Setter with validation
print(emp.email)              # mike@firm.com

del emp.email                  # Deleter runs
print(emp.email)              # None
```

---

### 3.5 Computed Properties — Derived Values ⭐

```python
class Invoice:
    def __init__(self, hours, rate, tax_rate=0.09):
        self.hours = hours
        self.rate = rate
        self.tax_rate = tax_rate
    
    @property
    def subtotal(self):
        """Computed: always up-to-date."""
        return self.hours * self.rate
    
    @property
    def tax(self):
        return self.subtotal * self.tax_rate
    
    @property
    def total(self):
        return self.subtotal + self.tax
    
    @property
    def summary(self):
        return (f"Invoice: {self.hours}h × ${self.rate}/hr\n"
                f"  Subtotal: ${self.subtotal:,.2f}\n"
                f"  Tax ({self.tax_rate:.0%}): ${self.tax:,.2f}\n"
                f"  Total: ${self.total:,.2f}")

inv = Invoice(85, 450)
print(inv.summary)

# Change the inputs — computed values update automatically!
inv.hours = 100
inv.rate = 500
print(f"\nUpdated total: ${inv.total:,.2f}")   # $54,500.00
```

**Mike:** "Computed properties are always consistent. If you stored `total` as a regular attribute, it could get out of sync with `hours` and `rate`."

---

### 3.6 Cached Properties — Best of Both Worlds

```python
class Report:
    """Property that is expensive to compute — cache the result."""
    
    def __init__(self, data):
        self.data = data
        self._analysis_cache = None
    
    @property
    def analysis(self):
        """Expensive computation — cached after first call."""
        if self._analysis_cache is None:
            print("  ⏳ Computing analysis (slow)...")
            import time; time.sleep(0.1)    # Simulate expensive work
            self._analysis_cache = {
                "count": len(self.data),
                "total": sum(self.data),
                "average": sum(self.data) / len(self.data),
                "max": max(self.data),
                "min": min(self.data),
            }
        return self._analysis_cache
    
    def invalidate_cache(self):
        """Clear cache when data changes."""
        self._analysis_cache = None

report = Report([100, 200, 300, 400, 500])
print(report.analysis)     # ⏳ Computing... (first call)
print(report.analysis)     # Instant! (cached)

# Python 3.8+ has functools.cached_property for this!
from functools import cached_property

class Report:
    def __init__(self, data):
        self.data = data
    
    @cached_property
    def analysis(self):
        """Computed once, then cached as a regular attribute."""
        print("  ⏳ Computing...")
        return {
            "count": len(self.data),
            "total": sum(self.data),
            "average": sum(self.data) / len(self.data),
        }

report = Report([100, 200, 300, 400, 500])
print(report.analysis)     # ⏳ Computing... (first)
print(report.analysis)     # Instant! (cached as attribute)
```

---

### 3.7 Properties vs Methods — When to Use Which

```python
class Attorney:
    def __init__(self, name, rate, cases=None):
        self._name = name
        self._rate = rate
        self.cases = cases or []
    
    # USE @property WHEN: it feels like data (noun)
    @property
    def name(self):
        """Attribute-like — no surprise side effects."""
        return self._name
    
    @property
    def case_count(self):
        """Quick computation, no side effects."""
        return len(self.cases)
    
    @property
    def total_billing(self):
        """Computed from existing data — feels like data."""
        return sum(c["hours"] * c["rate"] for c in self.cases)
    
    # USE A METHOD WHEN: it feels like an action (verb)
    def add_case(self, title, hours, rate=None):
        """Action with side effects — modifies state."""
        self.cases.append({"title": title, "hours": hours, "rate": rate or self._rate})
    
    def generate_report(self):
        """Expensive, has side effects, returns complex object."""
        return {"name": self._name, "cases": self.cases, "total": self.total_billing}
    
    def export_to_csv(self, filepath):
        """Side effect (writes file) — definitely a method."""
        ...
```

**Decision guide:**

| Use `@property` when... | Use a method when... |
|:-:|:-:|
| It's a **noun** (data-like) | It's a **verb** (action-like) |
| Fast (O(1) or cheap) | May be slow |
| No side effects | Has side effects |
| No required arguments | Needs arguments |
| Callers expect attribute access | Callers expect a function call |

---

### 3.8 Property Pattern: Clamping and Normalization

```python
class AudioMixer:
    """Properties for clamped and normalized values."""
    
    def __init__(self, volume=50, bass=0, treble=0):
        self.volume = volume    # Goes through setter
        self.bass = bass
        self.treble = treble
    
    @property
    def volume(self):
        return self._volume
    
    @volume.setter
    def volume(self, value):
        """Clamp volume between 0 and 100."""
        self._volume = max(0, min(100, value))
    
    @property
    def bass(self):
        return self._bass
    
    @bass.setter
    def bass(self, value):
        """Clamp bass between -10 and 10."""
        self._bass = max(-10, min(10, value))
    
    @property
    def treble(self):
        return self._treble
    
    @treble.setter
    def treble(self, value):
        self._treble = max(-10, min(10, value))

mixer = AudioMixer()
mixer.volume = 150    # Clamped to 100
mixer.volume = -20    # Clamped to 0
print(mixer.volume)   # 0

mixer.bass = 15       # Clamped to 10
print(mixer.bass)     # 10
```

---

### 3.9 Solving Louis's Problem — Bulletproof Client System

```python
# ============================================
# Pearson Specter Litt — Bulletproof Client System
# Properties for validation, computed values, and access control
# ============================================

from datetime import datetime
from functools import cached_property

class SecureClient:
    """Client with property-based validation and access control."""
    
    def __init__(self, name, revenue, case_type="General", attorney=None):
        # These all go through setters!
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
        self.attorney = attorney
        
        # Truly private — no setter, no external access
        self._id = f"CLT-{id(self) % 100000:05d}"
        self._created = datetime.now()
        self._cases = []
        self._audit_log = []
        self._log("Client created")
    
    # === Read-only properties ===
    
    @property
    def id(self):
        """Immutable client ID."""
        return self._id
    
    @property
    def created(self):
        """Immutable creation timestamp."""
        return self._created
    
    @property
    def audit_log(self):
        """Read-only copy of the audit log."""
        return list(self._audit_log)    # Return a COPY!
    
    # === Validated properties ===
    
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError(f"Name must be string, got {type(value).__name__}")
        value = value.strip()
        if len(value) < 2:
            raise ValueError("Name must be at least 2 characters")
        if hasattr(self, '_name'):
            self._log(f"Name changed: '{self._name}' → '{value}'")
        self._name = value
    
    @property
    def revenue(self):
        return self._revenue
    
    @revenue.setter
    def revenue(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"Revenue must be numeric, got {type(value).__name__}")
        if value < 0:
            raise ValueError(f"Revenue cannot be negative: {value}")
        if hasattr(self, '_revenue'):
            self._log(f"Revenue changed: ${self._revenue:,.2f} → ${value:,.2f}")
        self._revenue = float(value)
    
    @property
    def case_type(self):
        return self._case_type
    
    @case_type.setter
    def case_type(self, value):
        valid_types = {"General", "Merger", "Patent", "Litigation", "Corporate"}
        if value not in valid_types:
            raise ValueError(f"Invalid case type: '{value}'. Must be one of {valid_types}")
        self._case_type = value
    
    @property
    def attorney(self):
        return self._attorney
    
    @attorney.setter
    def attorney(self, value):
        if value is not None and not isinstance(value, str):
            raise TypeError("Attorney must be a string or None")
        if hasattr(self, '_attorney') and self._attorney != value:
            self._log(f"Attorney changed: {self._attorney} → {value}")
        self._attorney = value
    
    # === Computed properties ===
    
    @property
    def tier(self):
        if self._revenue >= 1_000_000: return "💎 Platinum"
        elif self._revenue >= 500_000: return "🥇 Gold"
        elif self._revenue >= 100_000: return "🥈 Silver"
        return "🥉 Bronze"
    
    @property
    def case_count(self):
        return len(self._cases)
    
    @property
    def total_billed(self):
        return sum(c["amount"] for c in self._cases)
    
    @property
    def profitability_ratio(self):
        if not self._cases:
            return float('inf')
        return self._revenue / self.total_billed
    
    @property
    def status_summary(self):
        return (
            f"  📋 {self.name} [{self.id}]\n"
            f"  {self.tier}\n"
            f"  Attorney: {self.attorney or 'Unassigned'}\n"
            f"  Revenue: ${self.revenue:>12,.2f}\n"
            f"  Billed:  ${self.total_billed:>12,.2f}\n"
            f"  Cases: {self.case_count}\n"
            f"  Created: {self.created:%Y-%m-%d %H:%M}"
        )
    
    # === Methods (actions) ===
    
    def add_case(self, title, hours, rate):
        case = {"title": title, "hours": hours, "rate": rate, "amount": hours * rate}
        self._cases.append(case)
        self._log(f"Case added: '{title}' (${case['amount']:,.2f})")
        return self
    
    def _log(self, message):
        self._audit_log.append({
            "timestamp": datetime.now().isoformat(),
            "message": message,
        })
    
    def __repr__(self):
        return f"SecureClient('{self.name}', ${self.revenue:,.0f})"


# === Demonstrate ===
print("=" * 55)
print(f"{'🏛️  BULLETPROOF CLIENT SYSTEM':^55}")
print("=" * 55)

# Create with validation
wayne = SecureClient("Wayne Enterprises", 2500000, "Merger", "Harvey")
wayne.add_case("Wayne v. Stark", 85, 450)
wayne.add_case("Dalton Merger", 70, 450)

print(wayne.status_summary)

# Properties enforce rules
print(f"\n🔒 Testing validation:")

try:
    wayne.revenue = -500
except ValueError as e:
    print(f"  ❌ {e}")

try:
    wayne.name = ""
except ValueError as e:
    print(f"  ❌ {e}")

try:
    wayne.case_type = "Murder"
except ValueError as e:
    print(f"  ❌ {e}")

# Read-only properties
try:
    wayne.id = "HACKED"
except AttributeError as e:
    print(f"  ❌ {e}")

# Valid changes are logged
wayne.revenue = 3000000
wayne.attorney = "Jessica"

# Audit trail
print(f"\n📋 Audit Log:")
for entry in wayne.audit_log:
    print(f"  [{entry['timestamp'][:19]}] {entry['message']}")

print(f"\n{'='*55}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting the Underscore — Infinite Recursion 🔥

```python
class Client:
    def __init__(self, name):
        self.name = name        # Calls the setter
    
    @property
    def name(self):
        return self.name        # ❌ Calls ITSELF → infinite recursion!
    
    @name.setter
    def name(self, value):
        self.name = value       # ❌ Calls ITSELF → infinite recursion!

# ✅ Store in a DIFFERENT attribute (underscore convention)
class Client:
    @property
    def name(self):
        return self._name       # ✅ Different attribute
    
    @name.setter
    def name(self, value):
        self._name = value      # ✅ Different attribute
```

---

### Pitfall 2: Property on a Method (Forgetting It's Not Callable)

```python
class Report:
    @property
    def total(self):
        return 42500

r = Report()
# print(r.total())    # ❌ TypeError: 'int' object is not callable!
print(r.total)        # ✅ 42500 — no parentheses with @property
```

---

### Pitfall 3: `@cached_property` Doesn't Work with `__slots__`

```python
from functools import cached_property

# ❌ cached_property needs a __dict__ to store the cache
class Data:
    __slots__ = ('values',)
    
    def __init__(self, values):
        self.values = values
    
    @cached_property
    def total(self):
        return sum(self.values)
    # TypeError on access!

# ✅ Either don't use __slots__ or use manual caching
```

---

### Pitfall 4: Setter Without Getter

```python
# ❌ This doesn't work — you need the @property getter first
class Item:
    @name.setter        # NameError: name 'name' is not defined!
    def name(self, value):
        self._name = value

# ✅ Define @property (getter) FIRST, then @name.setter
class Item:
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        self._name = value
```

---

### Pitfall 5: Mutable Property Return Values

```python
class Portfolio:
    def __init__(self):
        self._clients = ["Wayne", "Stark"]
    
    @property
    def clients(self):
        """Returns the list directly — can be mutated!"""
        return self._clients         # ❌ Caller can modify the internal list!

p = Portfolio()
p.clients.append("Hacker")           # Modified the internal list!
print(p.clients)                      # ['Wayne', 'Stark', 'Hacker'] — oops!

# ✅ Return a COPY to prevent external mutation
class Portfolio:
    @property
    def clients(self):
        return list(self._clients)   # ✅ Returns a copy
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🔐 Analogy: Properties = Security Checkpoint

```
PUBLIC ATTRIBUTE (open door):
  🚪 Anyone walks in, takes or leaves anything
  client.revenue = -9999     ← No guard, no check

@PROPERTY GETTER ONLY (one-way mirror):
  🪟 You can LOOK at the data but can't change it
  print(client.id)           ← Visible
  client.id = "HACKED"      ← 🚫 Blocked! No setter.

@PROPERTY WITH SETTER (security guard):
  👮 Guard checks your credentials before letting you in
  client.revenue = 3000000   ← Guard: "Valid. Proceed."
  client.revenue = -500      ← Guard: "DENIED. Negative not allowed."
  client.revenue = "lots"    ← Guard: "DENIED. Must be numeric."

COMPUTED PROPERTY (information board):
  📊 Always shows up-to-date info, computed on demand
  client.tier                ← "Platinum" (computed from revenue)
  No stored value — always recalculated from source data

DONNA'S RULE:
  "From the outside, it all looks the same: client.revenue
   But behind the dot, there can be a guard, a calculator,
   or a read-only display. That's the beauty of @property."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Temperature Converter**
```
Create a Temperature class where:
  - Setting .celsius automatically updates .fahrenheit
  - Setting .fahrenheit automatically updates .celsius
  - Both are validated (reject below absolute zero)

t = Temperature(celsius=100)
print(t.fahrenheit)    # 212.0
t.fahrenheit = 32
print(t.celsius)       # 0.0
```

**Exercise 2: Circle**
```
Create a Circle where:
  - .radius has a setter (must be positive)
  - .diameter is computed (2 * radius)
  - .area is computed (π * r²)
  - .circumference is computed (2 * π * r)
  - Setting .diameter updates .radius

c = Circle(5)
print(c.area)          # 78.54
c.diameter = 20
print(c.radius)        # 10.0
```

**Exercise 3: Password Field**
```
Create a User class where:
  - .password setter hashes the password (never store plain text)
  - .password getter returns "********" (never reveal)
  - .verify_password(plain) checks against hash
  - .username is read-only after creation
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Validated Config**
```
Build a Config class where every property has type + range validation:

config = Config()
config.port = 8080          # int, 1-65535
config.host = "0.0.0.0"    # string, valid IP
config.debug = True         # bool only
config.workers = 4          # int, 1-32
config.timeout = 30.0       # float, 0.1-300.0

All invalid assignments raise descriptive errors.
```

**Exercise 5: Reactive Properties**
```
Build a class where changing one property triggers callbacks:

class Reactive:
    @reactive_property
    def value(self):
        return self._value

obj.on_change("value", lambda old, new: print(f"{old} → {new}"))
obj.value = 42    # Prints: None → 42
```

**Exercise 6: Immutable After Init**
```
Build a FrozenClient where:
  - All properties can be set in __init__
  - After __init__, all setters raise FrozenError
  - Has .unfreeze() to allow modifications temporarily
  - Has .freeze() to lock again
```

---

### 🔴 Advanced Exercises

**Exercise 7: Descriptor Protocol**
```
Build a Validated descriptor that can be reused:

class ValidatedAttr:
    def __init__(self, type_, min_val=None, max_val=None): ...

class Attorney:
    name = ValidatedAttr(str, min_len=2)
    rate = ValidatedAttr(float, min_val=0, max_val=1000)
    age = ValidatedAttr(int, min_val=21, max_val=100)
```

**Exercise 8: Lazy Database Property**
```
Build a @db_property decorator that:
  - On first access, queries the database
  - Caches the result
  - Invalidates on .save()
  - Tracks access patterns for optimization
```

**Exercise 9: Property-Based State Machine**
```
Build a class where setting .status transitions through states:

order.status = "confirmed"    # OK: pending → confirmed
order.status = "shipped"      # OK: confirmed → shipped
order.status = "pending"      # Error! Can't go backwards!

Define valid transitions. Log all transitions.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Infinite recursion
class Person:
    @property
    def name(self):
        return self.name       # Calls itself!

# Bug 2: Calling property with parentheses
class Calc:
    @property
    def result(self):
        return 42
c = Calc()
print(c.result())              # TypeError!

# Bug 3: Property returns mutable internal state
class Team:
    def __init__(self):
        self._members = ["Harvey", "Mike"]
    @property
    def members(self):
        return self._members   # Returns actual list!

t = Team()
t.members.append("Hacker")    # Mutates internal state!

# Bug 4: Setter defined before getter
class Item:
    @price.setter              # NameError!
    def price(self, val):
        self._price = val

# Bug 5: Cached property never updates
from functools import cached_property
class Stats:
    def __init__(self, data):
        self.data = data
    @cached_property
    def total(self):
        return sum(self.data)

s = Stats([1, 2, 3])
print(s.total)       # 6
s.data = [10, 20, 30]
print(s.total)       # Still 6! Cached!
```

### Fixed Code ✅

```python
# Fix 1: Use different attribute name
class Person:
    @property
    def name(self):
        return self._name      # ✅ Different attribute

# Fix 2: No parentheses with property
print(c.result)                # ✅ 42

# Fix 3: Return a copy
@property
def members(self):
    return list(self._members)  # ✅ Copy

# Fix 4: Define getter first
class Item:
    @property
    def price(self):
        return self._price
    @price.setter
    def price(self, val):
        self._price = val

# Fix 5: Manually invalidate cache
s = Stats([1, 2, 3])
print(s.total)        # 6
s.data = [10, 20, 30]
del s.__dict__["total"]   # ✅ Clear the cache
print(s.total)            # 60
```

---

## 8. 🌍 Real-World Application

### Properties in Popular Frameworks

| Framework | Property | What It Does |
|-----------|----------|-------------|
| **Django** | `request.user` | Lazy-loads authenticated user |
| **SQLAlchemy** | `column.type` | Returns column type descriptor |
| **Pydantic** | `model.dict()` | Validated data access |
| **dataclasses** | `@dataclass` fields | Can combine with `@property` |
| **Flask** | `request.json` | Parses JSON body on access |
| **pathlib** | `path.stem` | Computed from path string |

---

## 9. ✅ Knowledge Check

---

**Q1.** What does `@property` do to a method?

**Q2.** How do you create a read-only property?

**Q3.** How do you add a setter to a property?

**Q4.** What causes infinite recursion in a property?

**Q5.** When should you use `@property` vs a regular method?

**Q6.** What is a computed property?

**Q7.** What is `@cached_property` and when would you use it?

**Q8.** Why should read-only properties return copies of mutable objects?

**Q9.** How do properties help with validation?

**Q10.** Can you have a setter without a getter?

---

### 📋 Answer Key

> **Q1.** It turns a method into an attribute-like access — `obj.method` without parentheses.

> **Q2.** Define only the `@property` getter — no `@x.setter`. Attempts to set will raise `AttributeError`.

> **Q3.** Define `@property_name.setter` below the getter: `@name.setter def name(self, value):`.

> **Q4.** When the property getter/setter references the same name as the property: `self.name` inside `name` property → calls itself.

> **Q5.** Use `@property` for noun-like, cheap, side-effect-free access. Use methods for verb-like actions, expensive operations, or when arguments are needed.

> **Q6.** A property that calculates its value from other attributes every time it's accessed (not stored).

> **Q7.** A property computed once on first access, then cached as a regular attribute. Use for expensive computations that won't change.

> **Q8.** Otherwise callers can mutate the internal state by modifying the returned object (lists, dicts, etc.).

> **Q9.** Setters run validation code automatically whenever the attribute is assigned — type checks, range checks, format validation.

> **Q10.** No — you must define the `@property` getter first. The setter is defined as `@property_name.setter`.

---

## 🎬 End of Lesson Scene

**Louis:** *(testing the system)* "I tried setting revenue to negative. It raised a `ValueError`. I tried changing the client ID. `AttributeError`. I tried passing a string for revenue. `TypeError`."

**Harvey:** "The firewall works."

**Mike:** "And from the outside, it's still just `client.revenue = 3000000`. The interface didn't change. The protection is invisible."

**Harvey:** "Next: exception handling. What happens when things go wrong — and how to handle it gracefully."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `@property` for read-only access | ✅ |
| 2 | `@property` with `@x.setter` for validation | ✅ |
| 3 | `@x.deleter` for controlled deletion | ✅ |
| 4 | Computed properties (derived values) | ✅ |
| 5 | `@cached_property` for expensive computations | ✅ |
| 6 | Property vs method decision guide | ✅ |
| 7 | Clamping and normalization patterns | ✅ |
| 8 | Audit logging through setters | ✅ |
| 9 | Returning copies from properties | ✅ |
| 10 | Infinite recursion pitfall and fix | ✅ |

---

> **Next Lesson →** Level 3, Lesson 5: *Handle Exceptions with Try-Except in Python*

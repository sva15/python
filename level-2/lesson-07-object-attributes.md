# 🏛️ Level 2 – Applying | Lesson 7: Access and Modify Python Object Attributes

## *"The Personnel File"*

> **O'Reilly Skill Objective:** *Access and modify instance variables via object attributes.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, loops, functions, `*args`/`**kwargs`, comprehensions, generators, data structure selection, classes (`__init__`, `self`), objects, file I/O

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

*Friday morning. Jessica walks to Harvey's office.*

**Jessica:** "HR just called. They need to update Louis's title — he's being promoted to Name Partner. Mike's billing rate needs to increase. And we need to pull Harvey's current case count for the quarterly review."

**Harvey:** "So we need to read attributes, change attributes, and sometimes add attributes that weren't there when the object was created?"

**Mike:** "Exactly. Lesson 12 taught you how to define classes and set initial attributes. Lesson 13 taught you how to create objects. This lesson goes deeper — the mechanics of how Python stores, accesses, modifies, removes, and dynamically adds attributes on any object."

**Harvey:** "Every personnel file in this firm — names, titles, rates, case counts — they're all object attributes. You need to know how to read them, update them, and protect the ones that shouldn't be changed."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "An object's attributes are its data. Like a personnel file that stores name, title, rate, and cases."

| Operation | Analogy | Python |
|-----------|---------|--------|
| **Read** an attribute | Pull the personnel file | `attorney.name` |
| **Modify** an attribute | Update a field in the file | `attorney.rate = 500` |
| **Add** an attribute | Staple a new page to the file | `attorney.bonus = 50000` |
| **Delete** an attribute | Remove a page from the file | `del attorney.bonus` |
| **Check** if attribute exists | "Does this file have a bonus page?" | `hasattr(attorney, 'bonus')` |
| **List** all attributes | "Show me the full file contents" | `vars(attorney)` |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Reading Attributes — Dot Notation

```python
class Attorney:
    def __init__(self, name, title, rate):
        self.name = name
        self.title = title
        self.rate = rate
        self.cases = []

harvey = Attorney("Harvey Specter", "Senior Partner", 450)

# Read attributes with dot notation
print(harvey.name)      # Harvey Specter
print(harvey.title)     # Senior Partner
print(harvey.rate)      # 450
print(harvey.cases)     # []

# Use in expressions
annual_target = harvey.rate * 2000    # $900,000
print(f"{harvey.name} ({harvey.title}): ${annual_target:,.0f}/year target")
```

---

### 3.2 Modifying Attributes — Direct Assignment

```python
# Modify existing attributes
harvey.rate = 500                          # Raise!
harvey.title = "Managing Partner"          # Promotion!
harvey.cases.append("Wayne v. Stark")      # Add a case

print(f"{harvey.name}: {harvey.title}, ${harvey.rate}/hr")
print(f"Cases: {harvey.cases}")

# Modify via calculation
harvey.rate *= 1.10    # 10% raise
print(f"New rate: ${harvey.rate:.2f}/hr")
```

---

### 3.3 Adding Attributes Dynamically

**Mike:** "In Python, you can add attributes to an object at ANY time — they don't have to be defined in `__init__`."

```python
class Attorney:
    def __init__(self, name):
        self.name = name

harvey = Attorney("Harvey")

# Add attributes that weren't in __init__
harvey.title = "Senior Partner"
harvey.rate = 450
harvey.specialty = "Corporate law"
harvey.office = "Corner office, 50th floor"

print(harvey.title)       # Senior Partner
print(harvey.specialty)   # Corporate law

# Other instances don't have these attributes!
mike = Attorney("Mike")
# print(mike.title)       # AttributeError! — only harvey has title
```

> ⚠️ **Warning:** Dynamic attributes are powerful but risky. If different instances have different attributes, code that expects a consistent interface will break.

---

### 3.4 `getattr()`, `setattr()`, `delattr()`, `hasattr()` — Programmatic Access ⭐

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

wayne = Client("Wayne Enterprises", 2500000)

# --- getattr(obj, name, default) ---
# Read an attribute by STRING name
print(getattr(wayne, "name"))           # "Wayne Enterprises"
print(getattr(wayne, "revenue"))        # 2500000
print(getattr(wayne, "tier", "N/A"))    # "N/A" — attribute doesn't exist, returns default

# Without default: raises AttributeError
# getattr(wayne, "tier")   # AttributeError!

# --- setattr(obj, name, value) ---
# Set an attribute by STRING name
setattr(wayne, "tier", "Platinum")
setattr(wayne, "revenue", 3000000)      # Can also update existing
print(wayne.tier)                        # "Platinum"
print(wayne.revenue)                     # 3000000

# --- hasattr(obj, name) ---
# Check if an attribute exists
print(hasattr(wayne, "name"))     # True
print(hasattr(wayne, "phone"))    # False

# --- delattr(obj, name) ---
# Remove an attribute
delattr(wayne, "tier")
print(hasattr(wayne, "tier"))     # False
```

**Mike:** "Why use these instead of dot notation?"

```python
# When the attribute name is a VARIABLE
field_name = "revenue"
value = getattr(wayne, field_name)    # Can't do wayne.field_name!

# When processing multiple fields dynamically
fields = ["name", "revenue", "tier"]
for field in fields:
    val = getattr(wayne, field, "missing")
    print(f"  {field}: {val}")

# When loading data from a file/dict into an object
data = {"name": "Stark", "revenue": 800000, "type": "Patent"}
client = Client.__new__(Client)   # Create without __init__
for key, value in data.items():
    setattr(client, key, value)
print(client.name)   # "Stark"
```

---

### 3.5 `vars()` and `__dict__` — Inspecting All Attributes

```python
class Attorney:
    firm = "Pearson Specter Litt"    # Class attribute
    
    def __init__(self, name, rate):
        self.name = name             # Instance attribute
        self.rate = rate             # Instance attribute

harvey = Attorney("Harvey", 450)

# vars() returns the instance's __dict__
print(vars(harvey))
# {'name': 'Harvey', 'rate': 450}

# __dict__ is the actual dict where attributes are stored
print(harvey.__dict__)
# {'name': 'Harvey', 'rate': 450}

# They're the same object!
print(vars(harvey) is harvey.__dict__)   # True

# Modify via __dict__
harvey.__dict__["bonus"] = 100000
print(harvey.bonus)   # 100000

# Iterate over all attributes
for attr, value in vars(harvey).items():
    print(f"  {attr}: {value}")
```

---

### 3.6 Instance Attributes vs Class Attributes ⭐

```python
class Attorney:
    # CLASS attributes — shared by ALL instances
    firm = "Pearson Specter Litt"
    headcount = 0
    
    def __init__(self, name, rate):
        # INSTANCE attributes — unique to each object
        self.name = name
        self.rate = rate
        Attorney.headcount += 1

harvey = Attorney("Harvey", 450)
mike = Attorney("Mike", 375)

# Instance attributes — different for each object
print(harvey.name)    # Harvey
print(mike.name)      # Mike

# Class attributes — same for all objects
print(harvey.firm)     # Pearson Specter Litt
print(mike.firm)       # Pearson Specter Litt
print(Attorney.firm)   # Pearson Specter Litt

# Headcount is shared
print(Attorney.headcount)   # 2
```

**The lookup chain:**
```python
# When you access obj.attr, Python looks in this order:
# 1. Instance __dict__ (obj.__dict__)
# 2. Class __dict__ (type(obj).__dict__)
# 3. Parent classes (via MRO)
# 4. If not found → AttributeError

# ⚠️ Assigning to an instance SHADOWS the class attribute:
harvey.firm = "Specter Litt"    # Creates an INSTANCE attribute
print(harvey.firm)    # "Specter Litt" — instance attribute
print(mike.firm)      # "Pearson Specter Litt" — still class attribute!
print(Attorney.firm)  # "Pearson Specter Litt" — class unchanged!
```

---

### 3.7 Attribute Access in Methods

```python
class Client:
    TIERS = {1000000: "Platinum", 500000: "Gold", 100000: "Silver", 0: "Bronze"}
    
    def __init__(self, name, revenue, case_type):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
        self._history = []           # "Internal" attribute (convention)
    
    def classify(self):
        """Read attributes to compute tier."""
        for threshold, tier in sorted(Client.TIERS.items(), reverse=True):
            if self.revenue >= threshold:
                return tier
        return "Bronze"
    
    def apply_raise(self, percentage):
        """Modify attributes via methods."""
        old = self.revenue
        self.revenue *= (1 + percentage)
        self._history.append(f"Revenue: ${old:,.0f} → ${self.revenue:,.0f}")
    
    def add_note(self, note):
        """Dynamically enriching the object."""
        if not hasattr(self, 'notes'):
            self.notes = []
        self.notes.append(note)
    
    def summary(self):
        """Read multiple attributes in formatted output."""
        lines = [
            f"Client: {self.name}",
            f"Tier: {self.classify()}",
            f"Revenue: ${self.revenue:,.2f}",
            f"Type: {self.case_type}",
        ]
        if hasattr(self, 'notes'):
            lines.append(f"Notes: {len(self.notes)}")
        if self._history:
            lines.append(f"Changes: {len(self._history)}")
        return "\n".join(lines)

wayne = Client("Wayne Enterprises", 2500000, "Merger")
wayne.apply_raise(0.10)
wayne.add_note("High priority account")
wayne.add_note("Prefers email communication")
print(wayne.summary())
```

---

### 3.8 Property Decorators — Controlled Access ⭐

**Mike:** "When you need control over HOW attributes are read or modified."

```python
class BillableClient:
    def __init__(self, name, hours, rate):
        self.name = name
        self.hours = hours
        self._rate = rate         # "Private" storage
        self._total = None        # Cache
    
    @property
    def rate(self):
        """GET the rate — like reading an attribute."""
        return self._rate
    
    @rate.setter
    def rate(self, value):
        """SET the rate — with validation!"""
        if value < 0:
            raise ValueError("Rate cannot be negative")
        if value > 1000:
            raise ValueError("Rate cannot exceed $1000/hr")
        self._rate = value
        self._total = None    # Invalidate cache
    
    @property
    def total(self):
        """Computed attribute — calculated on access."""
        if self._total is None:
            self._total = self.hours * self._rate
        return self._total
    
    @property
    def tier(self):
        """Read-only computed property — no setter defined."""
        if self.total >= 100000:
            return "Platinum"
        elif self.total >= 50000:
            return "Gold"
        return "Silver"

wayne = BillableClient("Wayne", 85, 450)

# Looks like normal attribute access, but runs code behind the scenes!
print(wayne.rate)     # 450 (calls the @property getter)
print(wayne.total)    # 38250 (computed and cached)
print(wayne.tier)     # Silver (computed from total)

wayne.rate = 500       # Calls the setter (validates!)
print(wayne.total)     # 42500 (recalculated — cache invalidated)

# wayne.rate = -100    # ValueError: Rate cannot be negative
# wayne.tier = "Gold"  # AttributeError: can't set (no setter defined)
```

---

### 3.9 The `__slots__` Preview

```python
# Normal class — attributes stored in __dict__ (flexible but uses more memory)
class NormalClient:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

# Slotted class — fixed attributes (less memory, slightly faster)
class SlottedClient:
    __slots__ = ['name', 'revenue']
    
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

normal = NormalClient("Wayne", 2500000)
slotted = SlottedClient("Wayne", 2500000)

# Normal: can add dynamic attributes
normal.tier = "Platinum"     # ✅ Works

# Slotted: CANNOT add attributes not in __slots__
# slotted.tier = "Platinum"  # ❌ AttributeError!
# slotted.__dict__           # ❌ AttributeError — no __dict__!

import sys
print(f"Normal:  {sys.getsizeof(normal.__dict__) + sys.getsizeof(normal)} bytes")
print(f"Slotted: {sys.getsizeof(slotted)} bytes")
# Slotted uses ~40% less memory per instance
```

---

### 3.10 Solving Jessica's Problem — Dynamic Personnel System

```python
# ============================================
# Pearson Specter Litt — Personnel Management
# Dynamic attribute access for HR operations
# ============================================

class Person:
    """Base class with dynamic attribute management."""
    
    _required_fields = {"name"}
    
    def __init__(self, **kwargs):
        missing = self._required_fields - set(kwargs.keys())
        if missing:
            raise ValueError(f"Missing required fields: {missing}")
        
        for key, value in kwargs.items():
            setattr(self, key, value)
    
    def update(self, **kwargs):
        """Update multiple attributes at once."""
        changes = []
        for key, value in kwargs.items():
            old = getattr(self, key, "<new>")
            setattr(self, key, value)
            changes.append(f"{key}: {old} → {value}")
        return changes
    
    def get(self, field, default=None):
        """Safe attribute access."""
        return getattr(self, field, default)
    
    def has(self, field):
        """Check if attribute exists."""
        return hasattr(self, field)
    
    def remove(self, field):
        """Remove an attribute."""
        if hasattr(self, field):
            delattr(self, field)
            return True
        return False
    
    def fields(self):
        """List all data fields."""
        return {k: v for k, v in vars(self).items() if not k.startswith("_")}
    
    def to_dict(self):
        """Export all attributes as a dictionary."""
        return dict(vars(self))
    
    def __str__(self):
        fields = ", ".join(f"{k}={v!r}" for k, v in self.fields().items())
        return f"{type(self).__name__}({fields})"


class Attorney(Person):
    _required_fields = {"name", "title", "rate"}
    
    def __init__(self, **kwargs):
        kwargs.setdefault("cases", [])
        kwargs.setdefault("total_billed", 0)
        super().__init__(**kwargs)
    
    @property
    def utilization(self):
        """Computed: cases / target."""
        target = getattr(self, "case_target", 10)
        return len(self.cases) / target * 100 if target > 0 else 0


class Client(Person):
    _required_fields = {"name", "revenue"}
    
    def __init__(self, **kwargs):
        kwargs.setdefault("active", True)
        kwargs.setdefault("case_type", "General")
        super().__init__(**kwargs)
    
    @property
    def tier(self):
        rev = getattr(self, "revenue", 0)
        if rev >= 1000000: return "💎 Platinum"
        elif rev >= 500000: return "🥇 Gold"
        elif rev >= 100000: return "🥈 Silver"
        return "🥉 Bronze"


# === Build the system ===
print("=" * 60)
print(f"{'PERSONNEL MANAGEMENT SYSTEM':^60}")
print("=" * 60)

# Create attorneys with flexible attributes
harvey = Attorney(name="Harvey Specter", title="Senior Partner", rate=450,
                  specialty="Corporate", office="Corner, 50th floor")

mike = Attorney(name="Mike Ross", title="Associate", rate=375,
                specialty="IP Law")

# Create clients
wayne = Client(name="Wayne Enterprises", revenue=2500000, case_type="Merger",
               contact="Bruce Wayne", priority="high")

stark = Client(name="Stark Industries", revenue=800000, case_type="Patent")

# -- Dynamic attribute operations --
print("\n📝 PROMOTIONS & UPDATES")
changes = mike.update(title="Senior Associate", rate=400, office="Window office")
for c in changes:
    print(f"  Mike: {c}")

print(f"\n📋 HARVEY'S PROFILE")
for field, value in harvey.fields().items():
    print(f"  {field}: {value}")

print(f"\n🔍 ATTRIBUTE CHECKS")
print(f"  Harvey has office: {harvey.has('office')}")
print(f"  Mike has office: {mike.has('office')}")
print(f"  Wayne priority: {wayne.get('priority', 'normal')}")
print(f"  Stark priority: {stark.get('priority', 'normal')}")

# Dynamic field reading
print(f"\n📊 DYNAMIC REPORT")
fields_to_show = ["name", "title", "rate", "specialty", "office"]
for atty in [harvey, mike]:
    print(f"\n  {atty.name}:")
    for field in fields_to_show:
        val = atty.get(field, "—")
        print(f"    {field}: {val}")

# Computed properties
harvey.cases = ["Wayne v. Stark", "Dalton Merger", "Queen IPO"]
print(f"\n  Harvey utilization: {harvey.utilization:.0f}%")

# Client tier (computed property)
for client in [wayne, stark]:
    print(f"\n  {client.name}: {client.tier}")

# Export to dict
print(f"\n📦 EXPORT (Harvey as dict):")
for k, v in harvey.to_dict().items():
    print(f"  {k}: {v}")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Shadowing Class Attributes with Instance Attributes

```python
class Counter:
    count = 0    # Class attribute
    
    def __init__(self):
        self.count = 0    # ⚠️ Creates an INSTANCE attribute that shadows!

c = Counter()
c.count += 1
print(c.count)         # 1 — instance attribute
print(Counter.count)   # 0 — class attribute unchanged!

# ✅ If you want shared state, use the class name:
class Counter:
    count = 0
    
    def increment(self):
        Counter.count += 1    # Modify the CLASS attribute directly
```

---

### Pitfall 2: `AttributeError` on Missing Attributes

```python
class Client:
    def __init__(self, name):
        self.name = name

c = Client("Wayne")
# print(c.revenue)   # AttributeError: 'Client' has no attribute 'revenue'

# ✅ Use getattr with default
revenue = getattr(c, 'revenue', 0)

# ✅ Or hasattr check
if hasattr(c, 'revenue'):
    print(c.revenue)
else:
    print("No revenue set")

# ✅ Or try/except
try:
    print(c.revenue)
except AttributeError:
    print("No revenue set")
```

---

### Pitfall 3: Mutable Class Attribute Shared by All Instances

```python
class Team:
    members = []    # ⚠️ Class attribute — SHARED!
    
    def add(self, name):
        self.members.append(name)   # Modifies the CLASS list!

team_a = Team()
team_b = Team()

team_a.add("Harvey")
print(team_b.members)    # ['Harvey'] ← team_b sees Harvey!

# ✅ Fix: initialize in __init__
class Team:
    def __init__(self):
        self.members = []    # Instance attribute — independent!
```

---

### Pitfall 4: Dynamic Attributes Lead to Inconsistent Objects

```python
class Client:
    def __init__(self, name):
        self.name = name

c1 = Client("Wayne")
c2 = Client("Stark")

c1.revenue = 2500000    # Only c1 has revenue!

# Later in code:
for client in [c1, c2]:
    print(client.revenue)   # ❌ Crashes on c2!

# ✅ Fix: define all attributes in __init__ with defaults
class Client:
    def __init__(self, name, revenue=0):
        self.name = name
        self.revenue = revenue
```

---

### Pitfall 5: `vars()` Doesn't Show Class Attributes or Properties

```python
class Attorney:
    firm = "PSL"    # Class attribute
    
    def __init__(self, name):
        self.name = name
    
    @property
    def greeting(self):
        return f"Hello, I'm {self.name}"

harvey = Attorney("Harvey")

print(vars(harvey))       # {'name': 'Harvey'} — no 'firm' or 'greeting'!
print(harvey.firm)        # "PSL" — exists but not in vars()
print(harvey.greeting)    # "Hello, I'm Harvey" — works but not in vars()

# ✅ To see everything available, use dir()
print(dir(harvey))        # Shows ALL attributes and methods
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📂 Analogy: Object Attributes = Personnel File Fields

```
OBJECT = A personnel file folder
  ┌──────────────────────────┐
  │  📄 name: "Harvey"       │  ← Instance attribute (unique to this file)
  │  📄 rate: 450            │  ← Instance attribute
  │  📄 cases: [...]         │  ← Instance attribute
  │                          │
  │  🏷️ firm: "PSL"          │  ← Class attribute (same on ALL files)
  │                          │
  │  🔧 classify()           │  ← Method (action the file can perform)
  │  🔧 bill()               │  ← Method
  └──────────────────────────┘

  dot notation = Opening a specific page
  hasattr()    = "Does this file have a bonus page?"
  getattr()    = "Pull whatever page I ask for by name"
  setattr()    = "Add or update a page"
  delattr()    = "Remove a page"
  vars()       = "Show me ALL the pages"
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Attribute Basics**
```
Create a Car class with: make, model, year, mileage (default 0)
a) Create two cars
b) Read and print each car's attributes
c) Update one car's mileage
d) Add a "color" attribute dynamically
e) Check if "color" exists on both cars
```

**Exercise 2: Attribute Inspector**
```
Write: inspect(obj) that prints:
  - Object type
  - All instance attributes and values
  - Total number of attributes

Test with 3 different object types.
```

**Exercise 3: Bulk Update**
```
Write: bulk_update(obj, **kwargs) that:
  - Updates all provided attributes
  - Returns a list of changes made
  - Handles new vs existing attributes differently

Test: bulk_update(car, color="red", year=2025, seats=4)
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Config Object**
```
Build a Config class that:
  - Accepts any keyword arguments in __init__
  - Supports dot notation access
  - Supports dict-style access: config["key"]
  - Has a .to_dict() method
  - Has a .from_dict(d) classmethod
  - Tracks which attributes have been modified

Implement __getitem__, __setitem__, __contains__.
```

**Exercise 5: Attribute History Tracker**
```
Build a TrackedObject class that:
  - Records every attribute change with timestamp
  - Can show the history of any attribute
  - Can rollback an attribute to a previous value
  - Has a .diff() method showing all changes since creation
```

**Exercise 6: Schema Validator**
```
Build a ValidatedModel class that:
  - Defines expected attributes and their types
  - Raises TypeError on wrong type assignment
  - Raises ValueError on invalid values (min/max, etc.)
  - Uses @property for all validated fields. 

class Employee(ValidatedModel):
    _schema = {
        "name": {"type": str, "required": True},
        "age": {"type": int, "min": 18, "max": 100},
        "salary": {"type": float, "min": 0},
    }
```

---

### 🔴 Advanced Exercises

**Exercise 7: Dynamic Form Builder**
```
Build a Form class where fields are defined as class attributes:

class RegistrationForm(Form):
    name = TextField(required=True)
    email = EmailField(required=True)
    age = IntField(min=18)

form = RegistrationForm(name="Harvey", email="h@psl.com", age=42)
form.validate()  # Uses the field descriptors
form.to_dict()   # Export data
```

**Exercise 8: Proxy Object**
```
Build a Proxy class that:
  - Wraps any object
  - Intercepts ALL attribute access (get/set/del)
  - Logs every operation
  - Can block access to certain attributes
  - Can transform values on read/write

proxy = Proxy(real_obj, blocked=["password"], log=True)
proxy.name            # Logs: "GET name → 'Harvey'"
proxy.password        # Raises: "Access denied: 'password'"
```

**Exercise 9: ORM-Style Model**
```
Build a simple ORM-style model:

class Client(Model):
    name = StringField()
    revenue = FloatField(default=0)
    active = BoolField(default=True)

c = Client(name="Wayne", revenue=2500000)
c.save()              # Writes to a JSON file
c2 = Client.load(1)   # Reads from JSON
Client.find(revenue_gt=1000000)    # Query by attribute
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Accessing non-existent attribute
class Client:
    def __init__(self, name):
        self.name = name

c = Client("Wayne")
print(c.revenue)              # AttributeError!

# Bug 2: Shadowing class attribute
class Config:
    debug = False
    
    def enable_debug(self):
        self.debug = True     # Creates instance attr, doesn't change class!

c1 = Config()
c2 = Config()
c1.enable_debug()
print(c2.debug)               # Expected: True

# Bug 3: Shared mutable class attribute
class Team:
    roster = []
    def add(self, name):
        self.roster.append(name)

t1 = Team()
t2 = Team()
t1.add("Harvey")
print(t2.roster)              # Expected: []

# Bug 4: vars() missing class attrs
class Firm:
    name = "PSL"
    def __init__(self, city):
        self.city = city

f = Firm("NYC")
print("name" in vars(f))     # Expected: True

# Bug 5: Modifying __dict__ directly
class Safe:
    def __init__(self):
        self.value = 100

s = Safe()
s.__dict__["_hidden"] = "secret"
print(hasattr(s, "_hidden"))   # True — bypassed any protections!
```

### Fixed Code ✅

```python
# Fix 1: Use getattr with default
c = Client("Wayne")
print(getattr(c, "revenue", 0))   # 0 ✅

# Fix 2: Modify the CLASS attribute directly
class Config:
    debug = False
    
    def enable_debug(self):
        Config.debug = True        # ✅ Changes the class attribute!

c1 = Config()
c2 = Config()
c1.enable_debug()
print(c2.debug)                    # True ✅

# Fix 3: Initialize mutable attributes in __init__
class Team:
    def __init__(self):
        self.roster = []           # ✅ Instance attribute
    def add(self, name):
        self.roster.append(name)

# Fix 4: Class attrs aren't in vars() — use dir() or check the class
f = Firm("NYC")
print("name" in dir(f))           # True ✅
print(hasattr(f, "name"))         # True ✅

# Fix 5: Use properties or __slots__ to prevent direct dict manipulation
class Safe:
    __slots__ = ['value']          # ✅ No __dict__, no bypass!
    def __init__(self):
        self.value = 100
```

---

## 8. 🌍 Real-World Application

### Where Dynamic Attribute Access Is Used

| Framework | How Attributes Are Used |
|-----------|----------------------|
| **Django ORM** | `user.email = "new@test.com"; user.save()` — modify then persist |
| **SQLAlchemy** | Model columns accessed as attributes |
| **Flask** | Config via attributes: `app.config["DEBUG"]` |
| **argparse** | `args = parser.parse_args(); args.verbose` |
| **dataclasses** | Auto-generated `__init__` based on class attributes |
| **pydantic** | Schema validation on attribute assignment |

### Production Example: Feature Flags

```python
class FeatureFlags:
    """Dynamic feature flag system using attributes."""
    
    _defaults = {
        "dark_mode": False,
        "beta_features": False,
        "max_upload_mb": 10,
        "log_level": "INFO",
    }
    
    def __init__(self, **overrides):
        for key, default in self._defaults.items():
            setattr(self, key, overrides.get(key, default))
    
    def enable(self, flag):
        if hasattr(self, flag):
            setattr(self, flag, True)
    
    def disable(self, flag):
        if hasattr(self, flag):
            setattr(self, flag, False)
    
    def is_enabled(self, flag):
        return bool(getattr(self, flag, False))
    
    def __str__(self):
        flags = vars(self)
        enabled = [k for k, v in flags.items() if v is True]
        disabled = [k for k, v in flags.items() if v is False]
        return (f"Enabled: {', '.join(enabled) or 'none'}\n"
                f"Disabled: {', '.join(disabled) or 'none'}")

flags = FeatureFlags(dark_mode=True)
print(flags.is_enabled("dark_mode"))    # True
flags.enable("beta_features")
print(flags)
```

---

## 9. ✅ Knowledge Check

---

**Q1.** What is the difference between reading `obj.attr` and `getattr(obj, "attr")`?

**Q2.** How do you add an attribute to an object after creation?

**Q3.** What does `vars(obj)` return? What does it NOT include?

**Q4.** What is the attribute lookup order in Python?

**Q5.** How can you check if an object has an attribute?

**Q6.** What is the difference between an instance attribute and a class attribute?

**Q7.** What happens if you assign `self.x = value` when `x` is already a class attribute?

**Q8.** What does `@property` do and why is it useful for attribute access?

**Q9.** What is `__slots__` and what does it prevent?

**Q10.** When should you use `getattr/setattr` instead of dot notation?

---

### 📋 Answer Key

> **Q1.** Same result, but `getattr` accepts the attribute name as a **string** (variable), and can provide a `default` value.

> **Q2.** `obj.new_attr = value` or `setattr(obj, "new_attr", value)`.

> **Q3.** Returns `obj.__dict__` — the instance's attribute dictionary. Does NOT include class attributes, properties, or methods.

> **Q4.** Instance `__dict__` → Class `__dict__` → Parent classes (MRO) → `AttributeError`.

> **Q5.** `hasattr(obj, "attr")` — returns `True`/`False`.

> **Q6.** Instance: unique to each object, stored in `obj.__dict__`. Class: shared by all instances, stored in `ClassName.__dict__`.

> **Q7.** Creates a new instance attribute that **shadows** (hides) the class attribute. The class attribute is unchanged.

> **Q8.** Makes a method look like an attribute. Useful for validation, computed values, caching, and read-only attributes.

> **Q9.** Restricts an object to fixed attributes. Prevents `__dict__` creation, saves memory, blocks dynamic attribute addition.

> **Q10.** When the attribute name is a **variable** (dynamic), when you need a default value, or when processing attributes programmatically.

---

## 🎬 End of Lesson Scene

**Jessica:** "Louis is now Name Partner. Mike's rate is updated. Harvey's case load is on the dashboard. All through attribute operations."

**Harvey:** "Next: class methods and static variables. The shared infrastructure that every instance relies on."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Dot notation for reading/modifying | ✅ |
| 2 | Dynamic attribute addition | ✅ |
| 3 | `getattr()`, `setattr()`, `delattr()`, `hasattr()` | ✅ |
| 4 | `vars()` and `__dict__` inspection | ✅ |
| 5 | Instance vs class attributes | ✅ |
| 6 | Attribute lookup chain (instance → class → parents) | ✅ |
| 7 | Shadowing class attributes | ✅ |
| 8 | `@property` for controlled access | ✅ |
| 9 | `__slots__` preview | ✅ |
| 10 | Dynamic personnel management system | ✅ |

---

> **Next Lesson →** Level 2, Lesson 8: *Use Static Variables and Class Methods in Python*

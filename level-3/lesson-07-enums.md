# 🏛️ Level 3 – Building | Lesson 7: Create Enums with the enum Module in Python

## *"The Statute Book"*

> **O'Reilly Skill Objective:** *Create enumerations using the `enum` module to define named constants.*

> **Previously Learned:** Classes, dunder methods, `@property`, exception handling, lambda functions, type validation, data structures

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

*A bug in the billing system. Someone typed "mergr" instead of "merger".*

```python
def set_case_type(case, case_type):
    case["type"] = case_type

set_case_type(wayne_case, "merger")     # ✅
set_case_type(stark_case, "Merger")     # ✅ or ❌? Case-sensitive?
set_case_type(oscorp_case, "mergr")     # ❌ Typo! No error raised!
set_case_type(dalton_case, "hostile_takeover")  # ❌ Not a valid type! No error!
```

**Louis:** "We have case types scattered across 50 files as magic strings — `'merger'`, `'Merger'`, `'MERGER'`, `'patent'`. Nobody knows which is correct."

**Harvey:** "Named constants. One canonical definition. If you misspell it, Python catches it immediately."

**Mike:** "The `enum` module. Enumerations — a fixed set of named constants. Case types, statuses, priorities, roles — anything that should be one of a defined set."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Enums replace magic strings and magic numbers with named, type-safe constants."

| Approach | Problem |
|----------|---------|
| Strings: `"merger"` | Typos, case inconsistency, no validation |
| Integers: `1, 2, 3` | Meaningless — what does `3` mean? |
| Constants: `MERGER = "merger"` | No type safety, no grouping |
| **Enum**: `CaseType.MERGER` | ✅ Named, grouped, type-safe, IDE-autocomplete |

```python
from enum import Enum

class CaseType(Enum):
    MERGER = "merger"
    PATENT = "patent"
    LITIGATION = "litigation"
    CORPORATE = "corporate"

# Now:
# CaseType.MERGR    → AttributeError! Typo caught immediately!
# CaseType.MERGER   → CaseType.MERGER ✅
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Enum Definition ⭐

```python
from enum import Enum

class CaseType(Enum):
    MERGER = "merger"
    PATENT = "patent"
    LITIGATION = "litigation"
    CORPORATE = "corporate"

# Accessing members
print(CaseType.MERGER)          # CaseType.MERGER
print(CaseType.MERGER.name)     # MERGER
print(CaseType.MERGER.value)    # merger
print(repr(CaseType.MERGER))    # <CaseType.MERGER: 'merger'>

# Comparison
print(CaseType.MERGER == CaseType.MERGER)     # True
print(CaseType.MERGER == CaseType.PATENT)     # False
print(CaseType.MERGER is CaseType.MERGER)     # True (singletons!)

# ❌ Can't compare with raw strings by default
print(CaseType.MERGER == "merger")             # False!

# Iteration
for ct in CaseType:
    print(f"  {ct.name}: {ct.value}")
# MERGER: merger
# PATENT: patent
# LITIGATION: litigation
# CORPORATE: corporate

# Count
print(f"Total case types: {len(CaseType)}")    # 4
```

---

### 3.2 Integer Enums — `IntEnum` ⭐

```python
from enum import IntEnum

class Priority(IntEnum):
    CRITICAL = 1
    HIGH = 2
    MEDIUM = 3
    LOW = 4

# IntEnum members can be compared with integers!
print(Priority.HIGH == 2)          # True
print(Priority.HIGH < Priority.LOW)  # True (2 < 4)
print(Priority.CRITICAL + 1)       # 2

# Useful for sorting
cases = [
    {"name": "Wayne", "priority": Priority.LOW},
    {"name": "Stark", "priority": Priority.CRITICAL},
    {"name": "Oscorp", "priority": Priority.MEDIUM},
]

sorted_cases = sorted(cases, key=lambda c: c["priority"])
for c in sorted_cases:
    print(f"  [{c['priority'].name}] {c['name']}")
# [CRITICAL] Stark
# [MEDIUM] Oscorp
# [LOW] Wayne
```

---

### 3.3 String Enums — `StrEnum` (Python 3.11+)

```python
# Python 3.11+
from enum import StrEnum

class Status(StrEnum):
    ACTIVE = "active"
    PENDING = "pending"
    CLOSED = "closed"
    ARCHIVED = "archived"

# StrEnum members compare with strings directly!
print(Status.ACTIVE == "active")    # True
print(f"Status is {Status.ACTIVE}")  # Status is active

# Great for APIs and JSON
import json
data = {"status": Status.ACTIVE}
print(json.dumps(data))    # {"status": "active"} — auto-converts!

# Pre-3.11 equivalent:
from enum import Enum

class Status(str, Enum):
    ACTIVE = "active"
    PENDING = "pending"
    CLOSED = "closed"
```

---

### 3.4 Auto Values and Aliases ⭐

```python
from enum import Enum, auto

class Department(Enum):
    """auto() generates values automatically."""
    LITIGATION = auto()    # 1
    CORPORATE = auto()     # 2
    MERGERS = auto()       # 3
    PATENTS = auto()       # 4
    PRO_BONO = auto()      # 5

print(Department.LITIGATION.value)    # 1
print(Department.PATENTS.value)       # 4

# Custom auto values
class CaseType(Enum):
    def _generate_next_value_(name, start, count, last_values):
        """Auto-generate lowercase names as values."""
        return name.lower()
    
    MERGER = auto()       # "merger"
    PATENT = auto()       # "patent"
    LITIGATION = auto()   # "litigation"

print(CaseType.MERGER.value)    # merger

# Aliases — multiple names for the same value
class Status(Enum):
    ACTIVE = 1
    OPEN = 1        # Alias for ACTIVE (same value!)
    CLOSED = 2
    DONE = 2        # Alias for CLOSED

print(Status.ACTIVE is Status.OPEN)    # True — same member!
print(Status(1))                        # Status.ACTIVE (canonical name)

# List only canonical names (no aliases)
print(list(Status))
# [<Status.ACTIVE: 1>, <Status.CLOSED: 2>]

# List all including aliases
print(Status.__members__)
# {'ACTIVE': ..., 'OPEN': ..., 'CLOSED': ..., 'DONE': ...}
```

---

### 3.5 Enum Lookup — From Name or Value ⭐

```python
class CaseType(Enum):
    MERGER = "merger"
    PATENT = "patent"
    LITIGATION = "litigation"

# Lookup by value
ct = CaseType("merger")
print(ct)    # CaseType.MERGER

# Lookup by name
ct = CaseType["MERGER"]
print(ct)    # CaseType.MERGER

# Safe lookup with validation
def get_case_type(value):
    """Convert a string to CaseType with error handling."""
    try:
        return CaseType(value.lower())
    except ValueError:
        valid = [ct.value for ct in CaseType]
        raise ValueError(f"Invalid case type: '{value}'. Must be one of: {valid}")

print(get_case_type("merger"))      # CaseType.MERGER
print(get_case_type("MERGER"))      # CaseType.MERGER
# get_case_type("mergr")           # ValueError with helpful message!

# Check membership
print("merger" in CaseType._value2member_map_)    # True
print("mergr" in CaseType._value2member_map_)     # False
```

---

### 3.6 Enums with Methods and Properties ⭐⭐

```python
from enum import Enum

class CaseType(Enum):
    MERGER = ("merger", "💼", 500)
    PATENT = ("patent", "⚙️", 425)
    LITIGATION = ("litigation", "⚖️", 450)
    CORPORATE = ("corporate", "🏢", 400)
    
    def __init__(self, value, icon, default_rate):
        self._value_ = value
        self.icon = icon
        self.default_rate = default_rate
    
    @property
    def display_name(self):
        return f"{self.icon} {self.value.title()}"
    
    def estimate_cost(self, hours):
        return hours * self.default_rate
    
    @classmethod
    def from_string(cls, s):
        """Case-insensitive lookup."""
        s = s.strip().lower()
        for member in cls:
            if member.value == s:
                return member
        raise ValueError(f"Unknown case type: '{s}'")

# Rich enum members:
ct = CaseType.MERGER
print(ct.display_name)              # 💼 Merger
print(ct.default_rate)              # 500
print(ct.estimate_cost(85))         # 42500

# All types with rates:
for ct in CaseType:
    print(f"  {ct.display_name:<20} ${ct.default_rate}/hr")

# Safe lookup
print(CaseType.from_string("patent"))     # CaseType.PATENT
print(CaseType.from_string("LITIGATION")) # CaseType.LITIGATION
```

---

### 3.7 Flag Enums — Bitwise Combinations

```python
from enum import Flag, auto

class Permission(Flag):
    READ = auto()       # 1
    WRITE = auto()      # 2
    EXECUTE = auto()    # 4
    DELETE = auto()     # 8
    
    # Combinations
    READ_WRITE = READ | WRITE
    ADMIN = READ | WRITE | EXECUTE | DELETE

# Combine permissions
partner_perms = Permission.READ | Permission.WRITE | Permission.DELETE
associate_perms = Permission.READ | Permission.WRITE
paralegal_perms = Permission.READ

# Check permissions
print(Permission.READ in partner_perms)       # True
print(Permission.DELETE in associate_perms)    # False
print(Permission.DELETE in partner_perms)      # True

# Display
print(partner_perms)       # Permission.READ|WRITE|DELETE
print(f"Admin: {Permission.ADMIN}")  # Permission.READ|WRITE|EXECUTE|DELETE

# Practical check
def can_delete(user_perms):
    return Permission.DELETE in user_perms

print(can_delete(partner_perms))     # True
print(can_delete(associate_perms))   # False
```

---

### 3.8 Enum Guards — Preventing Invalid Values

```python
from enum import Enum, unique

@unique    # Ensures no duplicate values (no aliases allowed)
class Status(Enum):
    ACTIVE = "active"
    PENDING = "pending"
    CLOSED = "closed"
    # DONE = "closed"    # ValueError! @unique prevents aliases!

# Using enums as type guards in functions
class CaseType(Enum):
    MERGER = "merger"
    PATENT = "patent"
    LITIGATION = "litigation"

def create_case(client, case_type, hours):
    """Type-safe case creation."""
    if not isinstance(case_type, CaseType):
        raise TypeError(f"Expected CaseType enum, got {type(case_type).__name__}")
    
    return {
        "client": client,
        "type": case_type,
        "type_value": case_type.value,
        "hours": hours,
    }

# ✅ Correct usage
case = create_case("Wayne", CaseType.MERGER, 85)

# ❌ Caught immediately
# create_case("Wayne", "merger", 85)    # TypeError!
# create_case("Wayne", "mergr", 85)     # TypeError!
```

---

### 3.9 Enums with JSON Serialization

```python
import json
from enum import Enum

class CaseType(Enum):
    MERGER = "merger"
    PATENT = "patent"

class Status(Enum):
    ACTIVE = "active"
    CLOSED = "closed"

# Custom JSON encoder for enums
class EnumEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Enum):
            return obj.value
        return super().default(obj)

case = {
    "client": "Wayne",
    "type": CaseType.MERGER,
    "status": Status.ACTIVE,
}

# Serialize
json_str = json.dumps(case, cls=EnumEncoder, indent=2)
print(json_str)
# {
#   "client": "Wayne",
#   "type": "merger",
#   "status": "active"
# }

# Deserialize
def decode_case(data):
    """Convert JSON strings back to enums."""
    return {
        "client": data["client"],
        "type": CaseType(data["type"]),
        "status": Status(data["status"]),
    }

restored = decode_case(json.loads(json_str))
print(restored["type"])      # CaseType.MERGER
print(restored["status"])    # Status.ACTIVE
```

---

### 3.10 Solving Louis's Problem — Type-Safe Case System

```python
# ============================================
# Pearson Specter Litt — Enum-Driven Case System
# Named constants for type safety across the firm
# ============================================

from enum import Enum, IntEnum, Flag, auto, unique
from datetime import datetime

@unique
class CaseType(Enum):
    MERGER = ("merger", "💼", 500)
    PATENT = ("patent", "⚙️", 425)
    LITIGATION = ("litigation", "⚖️", 450)
    CORPORATE = ("corporate", "🏢", 400)
    PRO_BONO = ("pro_bono", "🤝", 0)
    
    def __init__(self, val, icon, rate):
        self._value_ = val
        self.icon = icon
        self.default_rate = rate
    
    @property
    def label(self):
        return f"{self.icon} {self.value.replace('_', ' ').title()}"

class Priority(IntEnum):
    CRITICAL = 1
    HIGH = 2
    MEDIUM = 3
    LOW = 4

class Status(Enum):
    OPEN = "open"
    IN_PROGRESS = "in_progress"
    REVIEW = "review"
    CLOSED = "closed"
    ARCHIVED = "archived"
    
    @property
    def is_active(self):
        return self in (Status.OPEN, Status.IN_PROGRESS, Status.REVIEW)

class Permission(Flag):
    VIEW = auto()
    EDIT = auto()
    BILL = auto()
    CLOSE = auto()
    ADMIN = VIEW | EDIT | BILL | CLOSE

# Role-to-permission mapping
ROLE_PERMISSIONS = {
    "Partner": Permission.ADMIN,
    "Associate": Permission.VIEW | Permission.EDIT | Permission.BILL,
    "Paralegal": Permission.VIEW | Permission.EDIT,
    "Intern": Permission.VIEW,
}


class Case:
    """Case with enum-driven type safety."""
    
    def __init__(self, title, client, case_type, priority=Priority.MEDIUM):
        if not isinstance(case_type, CaseType):
            raise TypeError(f"Expected CaseType, got {type(case_type).__name__}")
        if not isinstance(priority, Priority):
            raise TypeError(f"Expected Priority, got {type(priority).__name__}")
        
        self.title = title
        self.client = client
        self.case_type = case_type
        self.priority = priority
        self.status = Status.OPEN
        self.created = datetime.now()
    
    def transition(self, new_status):
        """Validate status transitions."""
        valid = {
            Status.OPEN: {Status.IN_PROGRESS},
            Status.IN_PROGRESS: {Status.REVIEW, Status.OPEN},
            Status.REVIEW: {Status.CLOSED, Status.IN_PROGRESS},
            Status.CLOSED: {Status.ARCHIVED},
            Status.ARCHIVED: set(),
        }
        
        if new_status not in valid[self.status]:
            allowed = [s.value for s in valid[self.status]] or ["none"]
            raise ValueError(
                f"Cannot go from {self.status.value} → {new_status.value}. "
                f"Allowed: {', '.join(allowed)}"
            )
        self.status = new_status
    
    def __repr__(self):
        return (f"Case('{self.title}', type={self.case_type.label}, "
                f"priority={self.priority.name}, status={self.status.value})")


# === Demonstrate ===
print("=" * 60)
print(f"{'🏛️  ENUM-DRIVEN CASE MANAGEMENT':^60}")
print("=" * 60)

# Create cases with type safety
cases = [
    Case("Wayne v. Stark", "Wayne Enterprises", CaseType.MERGER, Priority.CRITICAL),
    Case("Arc Reactor IP", "Stark Industries", CaseType.PATENT, Priority.HIGH),
    Case("Oscorp Liability", "Oscorp", CaseType.LITIGATION, Priority.MEDIUM),
    Case("Rand Foundation", "Rand Corp", CaseType.PRO_BONO, Priority.LOW),
]

# Sorted by priority (IntEnum allows direct comparison)
print("\n📋 Cases by Priority:")
for case in sorted(cases, key=lambda c: c.priority):
    print(f"  [{case.priority.name:>8}] {case.case_type.label:<20} {case.title}")

# Status transitions
print("\n🔄 Status Transitions:")
wayne = cases[0]
print(f"  Current: {wayne.status.value}")

wayne.transition(Status.IN_PROGRESS)
print(f"  → {wayne.status.value}")

wayne.transition(Status.REVIEW)
print(f"  → {wayne.status.value}")

try:
    wayne.transition(Status.OPEN)    # Invalid!
except ValueError as e:
    print(f"  ❌ {e}")

# Permission checks
print("\n🔐 Permissions:")
for role, perms in ROLE_PERMISSIONS.items():
    can_bill = "✅" if Permission.BILL in perms else "❌"
    can_close = "✅" if Permission.CLOSE in perms else "❌"
    print(f"  {role:<12} Bill: {can_bill}  Close: {can_close}")

# Catch typos at creation!
print("\n🛡️ Type Safety:")
try:
    bad = Case("Test", "Client", "mergr")    # ❌ String, not enum!
except TypeError as e:
    print(f"  ❌ {e}")

# All case types
print(f"\n📚 Available Case Types:")
for ct in CaseType:
    print(f"  {ct.label:<25} ${ct.default_rate}/hr")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Comparing Enum with Raw Values 🔥

```python
class Color(Enum):
    RED = 1
    GREEN = 2

# ❌ Enum != raw value (by default)
print(Color.RED == 1)       # False!
print(Color.RED == "RED")   # False!

# ✅ Compare enum to enum
print(Color.RED == Color.RED)   # True

# ✅ Use .value for comparison
print(Color.RED.value == 1)     # True

# ✅ Or use IntEnum / StrEnum for automatic comparison
class Color(IntEnum):
    RED = 1
print(Color.RED == 1)    # True — IntEnum compares with ints
```

---

### Pitfall 2: Enums Are Immutable — Can't Change Values

```python
class Status(Enum):
    ACTIVE = 1
    CLOSED = 2

# ❌ Can't modify enum members!
# Status.ACTIVE.value = 99     # AttributeError!
# Status.ACTIVE = "something"  # AttributeError!
# Status.NEW = 3               # AttributeError!

# This is BY DESIGN — enums are constants!
```

---

### Pitfall 3: Enum Members Are Not the Value

```python
class Size(Enum):
    SMALL = 1
    MEDIUM = 2
    LARGE = 3

# ❌ Can't do math with Enum (use IntEnum for that)
# Size.SMALL + Size.MEDIUM    # TypeError!

# ✅ Use .value or IntEnum
print(Size.SMALL.value + Size.MEDIUM.value)    # 3
```

---

### Pitfall 4: Accidentally Creating Aliases

```python
class Color(Enum):
    RED = 1
    CRIMSON = 1    # This is an ALIAS, not a new member!

print(Color.RED is Color.CRIMSON)     # True!
print(list(Color))    # [<Color.RED: 1>] — CRIMSON not listed!

# ✅ Use @unique to prevent accidental aliases
from enum import unique

@unique
class Color(Enum):
    RED = 1
    # CRIMSON = 1    # ValueError: duplicate values found!
```

---

### Pitfall 5: JSON Serialization Fails by Default

```python
import json
from enum import Enum

class Status(Enum):
    ACTIVE = "active"

# ❌ Default JSON encoder doesn't know about enums
# json.dumps({"status": Status.ACTIVE})   # TypeError!

# ✅ Use .value
json.dumps({"status": Status.ACTIVE.value})    # Works!

# ✅ Or use a custom encoder (see section 3.9)
# ✅ Or use StrEnum (Python 3.11+) — auto-converts
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📖 Analogy: Enums = The Firm's Official Statute Book

```
WITHOUT ENUMS (anarchy):
  Anyone writes anything → "merger", "Merger", "MERGER", "mergr"
  No one knows which is canonical
  Typos go unnoticed for months

WITH ENUMS (statute book):
  📖 The Official Case Type Statute:
    Article 1: MERGER     = "merger"
    Article 2: PATENT     = "patent"  
    Article 3: LITIGATION = "litigation"
    Article 4: CORPORATE  = "corporate"
  
  ✅ "CaseType.MERGER"  → Valid, exists in the statute
  ❌ "CaseType.MERGR"   → AttributeError! Not in the statute!
  
  If it's not in the book, it doesn't exist.

DONNA'S RULE:
  "Magic strings are verbal agreements.
   Enums are signed contracts.
   Which one holds up in court?"
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Day of Week**
```
Create a DayOfWeek enum:
  - MONDAY through SUNDAY (values 1-7)
  - is_weekend property
  - is_weekday property
  - next_day() method
  - classmethod from_string("Monday") → DayOfWeek.MONDAY
```

**Exercise 2: HTTP Status Codes**
```
Create HTTPStatus enum:
  OK=200, NOT_FOUND=404, SERVER_ERROR=500, etc.
  - is_success property (200-299)
  - is_error property (400+)
  - description property (human-readable)
```

**Exercise 3: Card Suits and Ranks**
```
Create Suit and Rank enums for a deck of cards.
  - Suit: HEARTS, DIAMONDS, CLUBS, SPADES
  - Rank: TWO through ACE
  - Generate all 52 cards using both enums
```

---

### 🟡 Intermediate Exercises

**Exercise 4: State Machine**
```
Build an OrderStatus enum with valid transitions:
  PLACED → CONFIRMED → PREPARING → SHIPPED → DELIVERED
  
With methods:
  - can_transition_to(new_status) → bool
  - transition_to(new_status) → raises if invalid
  - all_reachable() → set of states reachable from current
```

**Exercise 5: Configuration Enum**
```
Build an Environment enum with associated configs:

class Environment(Enum):
    DEV = ("localhost", 8080, True)
    STAGING = ("staging.firm.com", 443, True)
    PRODUCTION = ("firm.com", 443, False)

env = Environment.DEV
print(env.host)       # localhost
print(env.port)       # 8080
print(env.debug)      # True
```

**Exercise 6: Permission System**
```
Build a Flag-based permission system:
  - Define permissions: CREATE, READ, UPDATE, DELETE
  - Define roles: VIEWER, EDITOR, ADMIN
  - check_permission(user_role, required_permission) → bool
  - Demonstrate combining and checking permissions
```

---

### 🔴 Advanced Exercises

**Exercise 7: Enum Registry**
```
Build an auto-discovering enum registry:

@register
class CaseType(Enum): ...

@register  
class Status(Enum): ...

registry.list_all()          # All registered enums
registry.get("CaseType")    # Returns CaseType class
registry.validate("CaseType", "merger")   # True
```

**Exercise 8: Protocol Enum**
```
Build a protocol message type system:

class MessageType(Enum):
    HANDSHAKE = (0x01, HandshakeHandler)
    DATA = (0x02, DataHandler)
    ACK = (0x03, AckHandler)
    ERROR = (0xFF, ErrorHandler)
    
    def handle(self, payload):
        return self.handler_class(payload).execute()
```

**Exercise 9: Enum-Based Parser**
```
Build a token type enum for a simple expression parser:
  NUMBER, PLUS, MINUS, MULTIPLY, DIVIDE, LPAREN, RPAREN
  
Each with: regex pattern, precedence, associativity.
Use the enum to tokenize and evaluate: "3 + 4 * (2 - 1)"
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Comparing enum to string
class Color(Enum):
    RED = "red"

if Color.RED == "red":       # False! Enum != string
    print("Match!")          # Never prints!

# Bug 2: Duplicate values without @unique
class Priority(Enum):
    HIGH = 1
    URGENT = 1    # ALIAS, not separate!

print(list(Priority))    # Only [HIGH] — URGENT missing!

# Bug 3: Trying to extend an enum
class Base(Enum):
    A = 1
# class Extended(Base):    # TypeError: Cannot extend enumerations!
#     B = 2

# Bug 4: Forgetting .value for JSON
import json
class Status(Enum):
    ACTIVE = "active"
# json.dumps({"s": Status.ACTIVE})    # TypeError!

# Bug 5: Enum in set/dict without proper __hash__
# (Enums are hashable by default, but custom __eq__ can break it)
```

### Fixed Code ✅

```python
# Fix 1: Compare with .value or use StrEnum
if Color.RED.value == "red":    # ✅
    print("Match!")

# Or use StrEnum:
class Color(str, Enum):
    RED = "red"
if Color.RED == "red":    # ✅ True with StrEnum!
    print("Match!")

# Fix 2: Use @unique
@unique
class Priority(Enum):
    HIGH = 1
    URGENT = 2    # ✅ Must be unique now

# Fix 3: Use composition instead of inheritance
class Base(Enum):
    A = 1
class Extended(Enum):
    A = Base.A.value
    B = 2

# Fix 4: Use .value or custom encoder
json.dumps({"s": Status.ACTIVE.value})    # ✅
```

---

## 8. 🌍 Real-World Application

### Enums in Popular Frameworks

| Framework | Usage |
|-----------|-------|
| **Django** | `models.TextChoices`, `models.IntegerChoices` |
| **FastAPI** | Enum types in path/query params auto-validate |
| **SQLAlchemy** | `Enum` column type for constrained values |
| **HTTP** | Status codes (200, 404, 500) |
| **Logging** | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |
| **pathlib** | Internal use for path types |

---

## 9. ✅ Knowledge Check

---

**Q1.** What problem do enums solve?

**Q2.** What is the difference between `Enum`, `IntEnum`, and `StrEnum`?

**Q3.** How do you access an enum member's name and value?

**Q4.** How do you look up an enum member by value?

**Q5.** Can you compare an `Enum` member with a string? What about `StrEnum`?

**Q6.** What is an enum alias?

**Q7.** What does `@unique` do?

**Q8.** What is `Flag` and how does it differ from `Enum`?

**Q9.** Can enum members have methods and properties?

**Q10.** How do you serialize enums to JSON?

---

### 📋 Answer Key

> **Q1.** Magic strings/numbers — replacing `"merger"` with `CaseType.MERGER` prevents typos, provides autocomplete, and enforces a fixed set of valid values.

> **Q2.** `Enum`: no comparison with values. `IntEnum`: compares with `int`. `StrEnum` (3.11+): compares with `str`.

> **Q3.** `member.name` → `"MERGER"` (string name). `member.value` → `"merger"` (the assigned value).

> **Q4.** `CaseType("merger")` — call the enum class with the value.

> **Q5.** `Enum` members do NOT equal raw strings. `StrEnum` members DO.

> **Q6.** Two names with the same value. `RED = 1; CRIMSON = 1` — `CRIMSON` is an alias for `RED`.

> **Q7.** Prevents accidental aliases — raises `ValueError` if two members share a value.

> **Q8.** `Flag` supports bitwise operations (`|`, `&`). Members can be combined: `READ | WRITE`. Regular `Enum` cannot.

> **Q9.** Yes. Enums are classes — they can have methods, properties, `@classmethod`, and custom `__init__`.

> **Q10.** Use `.value`, a custom `JSONEncoder`, or `StrEnum` (which auto-converts).

---

## 🎬 End of Lesson Scene

**Louis:** "I searched the codebase for the string `'merger'`. It appeared 47 times in 12 different files. Different capitalizations, two typos. Now there's ONE definition: `CaseType.MERGER`."

**Mike:** "And if someone types `CaseType.MERGR`, Python raises an `AttributeError` immediately. Typos caught at write-time, not in production."

**Harvey:** "Next: context managers. The `with` statement — and why every resource should have a cleanup plan."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Basic `Enum` definition and access | ✅ |
| 2 | `IntEnum` and `StrEnum` for value comparison | ✅ |
| 3 | `auto()` for automatic values | ✅ |
| 4 | Enum aliases and `@unique` | ✅ |
| 5 | Lookup by name and value | ✅ |
| 6 | Enums with methods, properties, `__init__` | ✅ |
| 7 | `Flag` for bitwise permission combinations | ✅ |
| 8 | JSON serialization of enums | ✅ |
| 9 | Type safety guards with enums | ✅ |
| 10 | Status transitions and state machines | ✅ |

---

> **Next Lesson →** Level 3, Lesson 8: *Use Context Managers in Python*

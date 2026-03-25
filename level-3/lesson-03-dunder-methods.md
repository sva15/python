# 🏛️ Level 3 – Building | Lesson 3: Override Methods and Dunder Methods in Python

## *"The Rules of the Court"*

> **O'Reilly Skill Objective:** *Customize class behavior by overriding built-in behavior with dunder methods (e.g., `__str__`, `__repr__`), and override methods in a child class to customize behavior.*

> **Previously Learned:** Classes, `__init__`, `self`, instance methods, `@classmethod`, `@staticmethod`, attributes, data structures, generators

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

*Wednesday. Mike prints a client object to the screen.*

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

wayne = Client("Wayne Enterprises", 2500000)
print(wayne)
# <__main__.Client object at 0x7f3b2c1d5a90>
```

**Mike:** "That's useless. I need it to say 'Wayne Enterprises — $2,500,000'. And I need to compare two clients with `>`, sort them, add their revenues with `+`, and use them in f-strings."

**Harvey:** "Python has rules for how objects behave — when you print them, compare them, add them, or iterate over them. Those rules are defined by special methods called **dunder methods** — double underscore methods. `__str__`, `__repr__`, `__eq__`, `__lt__`, `__add__`, and dozens more."

**Mike:** "You're saying I can teach Python how MY objects should behave in built-in operations?"

**Harvey:** "Exactly. You're overriding the rules of the court."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "When you call `print(obj)`, Python doesn't just magically know what to display. It calls `obj.__str__()`. When you write `a + b`, Python calls `a.__add__(b)`. Every Python operation maps to a dunder method."

| Operation | What Python Actually Calls | Dunder Method |
|-----------|---------------------------|:---:|
| `print(obj)` | `obj.__str__()` | `__str__` |
| `repr(obj)` | `obj.__repr__()` | `__repr__` |
| `len(obj)` | `obj.__len__()` | `__len__` |
| `a + b` | `a.__add__(b)` | `__add__` |
| `a == b` | `a.__eq__(b)` | `__eq__` |
| `a < b` | `a.__lt__(b)` | `__lt__` |
| `a[key]` | `a.__getitem__(key)` | `__getitem__` |
| `for x in obj` | `obj.__iter__()` | `__iter__` |
| `bool(obj)` | `obj.__bool__()` | `__bool__` |
| `obj()` | `obj.__call__()` | `__call__` |

**Harvey:** "Override these methods around 'dunder' is short for **d**ouble **under**score. `__init__` is the one you already know."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `__str__` vs `__repr__` — How Objects Display ⭐⭐⭐

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    
    def __str__(self):
        """For users: readable, pretty output."""
        return f"{self.name} — ${self.revenue:,.2f}"
    
    def __repr__(self):
        """For developers: unambiguous, recreatable."""
        return f"Client('{self.name}', {self.revenue})"

wayne = Client("Wayne Enterprises", 2500000)

# __str__ is used by: print(), str(), f-strings
print(wayne)                    # Wayne Enterprises — $2,500,000.00
print(f"Client: {wayne}")       # Client: Wayne Enterprises — $2,500,000.00

# __repr__ is used by: REPL, debug, repr(), containers
print(repr(wayne))              # Client('Wayne Enterprises', 2500000)
print([wayne])                  # [Client('Wayne Enterprises', 2500000)]

# If __str__ is not defined, Python falls back to __repr__!
```

**Rule of thumb:**

| Method | Audience | Goal | Example |
|:------:|:--------:|------|---------|
| `__str__` | End user | Readable, pretty | `Wayne Enterprises — $2.5M` |
| `__repr__` | Developer | Unambiguous, copy-pasteable | `Client('Wayne Enterprises', 2500000)` |

---

### 3.2 Comparison Methods — `__eq__`, `__lt__`, `__gt__` ⭐⭐

```python
from functools import total_ordering

@total_ordering    # Auto-generates __gt__, __ge__, __le__ from __eq__ + __lt__
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    
    def __eq__(self, other):
        """Equal if same name and revenue."""
        if not isinstance(other, Client):
            return NotImplemented      # Let Python try the other side
        return self.name == other.name and self.revenue == other.revenue
    
    def __lt__(self, other):
        """Less than: compared by revenue."""
        if not isinstance(other, Client):
            return NotImplemented
        return self.revenue < other.revenue
    
    def __hash__(self):
        """Required if __eq__ is defined (for sets and dict keys)."""
        return hash((self.name, self.revenue))
    
    def __repr__(self):
        return f"Client('{self.name}', {self.revenue})"

clients = [
    Client("Wayne", 2500000),
    Client("Stark", 800000),
    Client("Oscorp", 600000),
    Client("Dalton", 1200000),
]

# Comparison operators now work!
print(clients[0] > clients[1])          # True (Wayne > Stark)
print(clients[0] == Client("Wayne", 2500000))  # True

# sorted() uses __lt__ automatically!
for c in sorted(clients):
    print(f"  {c}")
# Client('Oscorp', 600000)
# Client('Stark', 800000)
# Client('Dalton', 1200000)
# Client('Wayne', 2500000)

# max() and min() work too!
print(f"Top client: {max(clients)}")     # Client('Wayne', 2500000)

# Can use in sets (because __hash__ is defined)
unique = {Client("Wayne", 2500000), Client("Wayne", 2500000)}
print(len(unique))   # 1 — duplicates removed!
```

**All comparison dunder methods:**

| Method | Operator | Note |
|:------:|:--------:|------|
| `__eq__` | `==` | Also affects `__hash__` |
| `__ne__` | `!=` | Auto-generated from `__eq__` |
| `__lt__` | `<` | |
| `__le__` | `<=` | |
| `__gt__` | `>` | |
| `__ge__` | `>=` | |

> 💡 Use `@functools.total_ordering`: define just `__eq__` + `__lt__` and get all 6!

---

### 3.3 Arithmetic Methods — `__add__`, `__sub__`, `__mul__` ⭐

```python
class Revenue:
    """Revenue value that supports arithmetic operations."""
    
    def __init__(self, amount, currency="USD"):
        self.amount = amount
        self.currency = currency
    
    def __add__(self, other):
        """Revenue + Revenue → combined Revenue."""
        if isinstance(other, Revenue):
            if self.currency != other.currency:
                raise ValueError(f"Cannot add {self.currency} and {other.currency}")
            return Revenue(self.amount + other.amount, self.currency)
        if isinstance(other, (int, float)):
            return Revenue(self.amount + other, self.currency)
        return NotImplemented
    
    def __radd__(self, other):
        """Reverse add: int + Revenue (needed for sum())."""
        if isinstance(other, (int, float)):
            return Revenue(self.amount + other, self.currency)
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, Revenue):
            return Revenue(self.amount - other.amount, self.currency)
        return NotImplemented
    
    def __mul__(self, factor):
        """Revenue * number."""
        if isinstance(factor, (int, float)):
            return Revenue(self.amount * factor, self.currency)
        return NotImplemented
    
    def __rmul__(self, factor):
        """number * Revenue."""
        return self.__mul__(factor)
    
    def __neg__(self):
        """-Revenue."""
        return Revenue(-self.amount, self.currency)
    
    def __abs__(self):
        """abs(Revenue)."""
        return Revenue(abs(self.amount), self.currency)
    
    def __float__(self):
        """float(Revenue) → the amount."""
        return float(self.amount)
    
    def __str__(self):
        return f"${self.amount:,.2f} {self.currency}"
    
    def __repr__(self):
        return f"Revenue({self.amount}, '{self.currency}')"

# Natural arithmetic!
a = Revenue(2500000)
b = Revenue(800000)
c = Revenue(600000)

print(a + b)           # $3,300,000.00 USD
print(a - b)           # $1,700,000.00 USD
print(a * 1.1)         # $2,750,000.00 USD
print(2 * b)           # $1,600,000.00 USD

# sum() works because we defined __radd__!
total = sum([a, b, c])
print(f"Total: {total}")   # Total: $3,900,000.00 USD
```

**Key arithmetic dunders:**

| Method | Operator | Reverse |
|:------:|:--------:|:-------:|
| `__add__` | `a + b` | `__radd__` |
| `__sub__` | `a - b` | `__rsub__` |
| `__mul__` | `a * b` | `__rmul__` |
| `__truediv__` | `a / b` | `__rtruediv__` |
| `__floordiv__` | `a // b` | `__rfloordiv__` |
| `__mod__` | `a % b` | `__rmod__` |
| `__pow__` | `a ** b` | `__rpow__` |

> 💡 `__radd__` (reverse add) is called when the LEFT operand doesn't support the operation. `5 + Revenue(100)` → `Revenue.__radd__(5)`

---

### 3.4 Container Methods — `__len__`, `__getitem__`, `__contains__` ⭐⭐

```python
class Portfolio:
    """A collection of clients that behaves like a built-in container."""
    
    def __init__(self, name):
        self.name = name
        self._clients = []
    
    def add(self, client):
        self._clients.append(client)
        return self
    
    def __len__(self):
        """len(portfolio)."""
        return len(self._clients)
    
    def __getitem__(self, index):
        """portfolio[0] or portfolio['Wayne']."""
        if isinstance(index, int):
            return self._clients[index]
        if isinstance(index, str):
            for c in self._clients:
                if c.name == index:
                    return c
            raise KeyError(f"Client '{index}' not found")
        if isinstance(index, slice):
            return self._clients[index]
        raise TypeError(f"Invalid index type: {type(index)}")
    
    def __setitem__(self, index, value):
        """portfolio[0] = new_client."""
        self._clients[index] = value
    
    def __delitem__(self, index):
        """del portfolio[0]."""
        del self._clients[index]
    
    def __contains__(self, item):
        """'Wayne' in portfolio."""
        if isinstance(item, str):
            return any(c.name == item for c in self._clients)
        return item in self._clients
    
    def __iter__(self):
        """for client in portfolio."""
        return iter(self._clients)
    
    def __reversed__(self):
        """reversed(portfolio)."""
        return reversed(self._clients)
    
    def __bool__(self):
        """bool(portfolio) — True if non-empty."""
        return len(self._clients) > 0
    
    def __repr__(self):
        return f"Portfolio('{self.name}', {len(self)} clients)"

# Use it like a built-in!
portfolio = Portfolio("Harvey's Clients")
portfolio.add(Client("Wayne", 2500000))
portfolio.add(Client("Stark", 800000))
portfolio.add(Client("Oscorp", 600000))

print(len(portfolio))              # 3
print(portfolio[0])                # Client('Wayne', 2500000)
print(portfolio["Stark"])          # Client('Stark', 800000)
print("Wayne" in portfolio)        # True
print(portfolio[1:])               # Slice works!

for client in portfolio:           # Iteration works!
    print(f"  {client}")

if portfolio:                      # Truthiness works!
    print("Portfolio has clients")
```

---

### 3.5 `__call__` — Making Objects Callable ⭐

```python
class BillingCalculator:
    """An object you can call like a function."""
    
    def __init__(self, default_rate=400, tax_rate=0.09):
        self.default_rate = default_rate
        self.tax_rate = tax_rate
        self.history = []
    
    def __call__(self, hours, rate=None):
        """Calculate billing — call the object like a function!"""
        actual_rate = rate or self.default_rate
        subtotal = hours * actual_rate
        tax = subtotal * self.tax_rate
        total = subtotal + tax
        
        self.history.append(total)
        return total
    
    def __repr__(self):
        return f"BillingCalculator(rate={self.default_rate}, calls={len(self.history)})"

# Use it like a function!
calculate = BillingCalculator(rate=450)

print(calculate(85))          # 41647.5 (85 * 450 * 1.09)
print(calculate(62, 400))     # 26972.0 (62 * 400 * 1.09)
print(calculate(40))          # 19620.0

print(f"Total billed: ${sum(calculate.history):,.2f}")
print(callable(calculate))    # True

# Common use cases for __call__:
# - Decorators (classes that wrap functions)
# - Stateful functions (function with memory)
# - Strategy pattern (interchangeable algorithms)
```

---

### 3.6 `__format__` — Custom Formatting

```python
class Revenue:
    def __init__(self, amount):
        self.amount = amount
    
    def __format__(self, spec):
        """Custom format specifications."""
        if spec == "short":
            if self.amount >= 1_000_000:
                return f"${self.amount / 1_000_000:.1f}M"
            elif self.amount >= 1_000:
                return f"${self.amount / 1_000:.0f}K"
            return f"${self.amount:.0f}"
        elif spec == "full":
            return f"${self.amount:,.2f} USD"
        elif spec == "":
            return f"${self.amount:,.2f}"
        return format(self.amount, spec)

r = Revenue(2500000)
print(f"{r}")            # $2,500,000.00 (default)
print(f"{r:short}")      # $2.5M
print(f"{r:full}")       # $2,500,000.00 USD
print(f"{r:.0f}")        # 2500000 (falls through to number format)
```

---

### 3.7 Context Manager Dunders (Preview)

```python
class DatabaseConnection:
    """Preview: __enter__ and __exit__ (covered in Lesson 8)."""
    
    def __init__(self, db_name):
        self.db_name = db_name
        self.connected = False
    
    def __enter__(self):
        """Called when entering 'with' block."""
        print(f"  Connecting to {self.db_name}...")
        self.connected = True
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Called when leaving 'with' block (even on error)."""
        print(f"  Closing connection to {self.db_name}...")
        self.connected = False
        return False    # Don't suppress exceptions

# with DatabaseConnection("firm_db") as db:
#     print(f"  Connected: {db.connected}")
# print(f"Connected after with: {db.connected}")
```

---

### 3.8 Method Overriding in Child Classes

```python
class Attorney:
    """Base attorney class."""
    
    def __init__(self, name, rate):
        self.name = name
        self.rate = rate
    
    def bill(self, hours):
        """Standard billing."""
        return hours * self.rate
    
    def introduce(self):
        return f"I'm {self.name}, an attorney."
    
    def __repr__(self):
        return f"Attorney('{self.name}', ${self.rate}/hr)"

class Partner(Attorney):
    """Partner overrides some behaviors."""
    
    def __init__(self, name, rate, equity_share):
        super().__init__(name, rate)     # Call parent's __init__
        self.equity_share = equity_share
    
    def bill(self, hours):
        """Partners add a premium — OVERRIDES parent method."""
        base = super().bill(hours)       # Call parent's bill()
        premium = base * 0.15
        return base + premium
    
    def introduce(self):
        """Override with partner-specific intro."""
        return f"I'm {self.name}, a named partner."
    
    def __repr__(self):
        return f"Partner('{self.name}', ${self.rate}/hr, equity={self.equity_share}%)"

class Associate(Attorney):
    """Associate with different billing rules."""
    
    def bill(self, hours):
        """Associates have a cap — OVERRIDES parent method."""
        base = super().bill(hours)
        cap = 40000                      # Monthly cap
        return min(base, cap)
    
    def introduce(self):
        return f"I'm {self.name}, an associate."

# Same interface, different behavior!
harvey = Partner("Harvey", 450, 15)
mike = Associate("Mike", 375)

print(harvey.introduce())     # I'm Harvey, a named partner.
print(mike.introduce())       # I'm Mike, an associate.

print(harvey.bill(100))        # 51750.0 (100 * 450 * 1.15 = premium)
print(mike.bill(100))          # 37500   (100 * 375, under cap)
print(mike.bill(200))          # 40000   (capped!)
```

---

### 3.9 Complete Reference: Essential Dunder Methods

```python
"""
ESSENTIAL DUNDER METHODS REFERENCE
Organized by category for quick lookup.
"""

# === Representation ===
# __repr__(self)       → repr(obj), debugging, containers
# __str__(self)        → str(obj), print(obj), f"{obj}"
# __format__(self, s)  → f"{obj:spec}", format(obj, spec)

# === Comparison ===
# __eq__(self, other)  → obj == other
# __ne__(self, other)  → obj != other
# __lt__(self, other)  → obj < other
# __le__(self, other)  → obj <= other
# __gt__(self, other)  → obj > other
# __ge__(self, other)  → obj >= other
# __hash__(self)       → hash(obj), sets, dict keys

# === Arithmetic ===
# __add__(self, other) → obj + other      __radd__ for reverse
# __sub__(self, other) → obj - other      __rsub__
# __mul__(self, other) → obj * other      __rmul__
# __truediv__(self, o) → obj / other      __rtruediv__
# __floordiv__(self,o) → obj // other     __rfloordiv__
# __mod__(self, other) → obj % other      __rmod__
# __pow__(self, other) → obj ** other     __rpow__
# __neg__(self)        → -obj
# __abs__(self)        → abs(obj)
# __round__(self, n)   → round(obj, n)

# === Container ===
# __len__(self)        → len(obj)
# __getitem__(self, k) → obj[key]
# __setitem__(self,k,v)→ obj[key] = value
# __delitem__(self, k) → del obj[key]
# __contains__(self,i) → item in obj
# __iter__(self)       → for x in obj
# __reversed__(self)   → reversed(obj)

# === Type Conversion ===
# __bool__(self)       → bool(obj), if obj:
# __int__(self)        → int(obj)
# __float__(self)      → float(obj)
# __complex__(self)    → complex(obj)
# __bytes__(self)      → bytes(obj)

# === Callable & Context ===
# __call__(self, ...)  → obj(args)
# __enter__(self)      → with obj as x:
# __exit__(self,...)   → end of with block

# === Lifecycle ===
# __init__(self, ...)  → obj = Class(args)
# __new__(cls, ...)    → object creation (before __init__)
# __del__(self)        → cleanup (unreliable, use __exit__)
```

---

### 3.10 Solving Mike's Problem — Smart Objects

```python
# ============================================
# Pearson Specter Litt — Smart Client Objects
# Full dunder method implementation
# ============================================

from functools import total_ordering
from datetime import datetime

@total_ordering
class SmartClient:
    """Client with rich dunder methods — behaves like a built-in type."""
    
    def __init__(self, name, revenue, case_type="General"):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
        self.cases = []
        self.created = datetime.now()
    
    # === Representation ===
    
    def __str__(self):
        return f"{self.name} — {self._format_revenue()}"
    
    def __repr__(self):
        return f"SmartClient('{self.name}', {self.revenue})"
    
    def __format__(self, spec):
        if spec == "brief":
            return f"{self.name}: {self._format_revenue()}"
        if spec == "full":
            return (f"{self.name}\n"
                    f"  Revenue: {self._format_revenue()}\n"
                    f"  Type: {self.case_type}\n"
                    f"  Cases: {len(self.cases)}")
        return str(self)
    
    # === Comparison (by revenue) ===
    
    def __eq__(self, other):
        if not isinstance(other, SmartClient):
            return NotImplemented
        return self.name == other.name
    
    def __lt__(self, other):
        if not isinstance(other, SmartClient):
            return NotImplemented
        return self.revenue < other.revenue
    
    def __hash__(self):
        return hash(self.name)
    
    # === Arithmetic ===
    
    def __add__(self, other):
        """Merge two clients into a combined entity."""
        if isinstance(other, SmartClient):
            return SmartClient(
                f"{self.name} + {other.name}",
                self.revenue + other.revenue,
                "Merged"
            )
        return NotImplemented
    
    # === Container-like ===
    
    def __len__(self):
        """Number of cases."""
        return len(self.cases)
    
    def __bool__(self):
        """True if the client has revenue."""
        return self.revenue > 0
    
    def __contains__(self, case_name):
        """'Merger' in client."""
        return any(c["name"] == case_name for c in self.cases)
    
    # === Type conversion ===
    
    def __float__(self):
        return float(self.revenue)
    
    def __int__(self):
        return int(self.revenue)
    
    # === Helper ===
    
    def _format_revenue(self):
        if self.revenue >= 1_000_000:
            return f"${self.revenue / 1_000_000:.1f}M"
        elif self.revenue >= 1_000:
            return f"${self.revenue / 1_000:.0f}K"
        return f"${self.revenue:.0f}"
    
    def add_case(self, name, hours, rate):
        self.cases.append({"name": name, "hours": hours, "rate": rate})
        return self


# === Demonstrate ===
print("=" * 55)
print(f"{'🏛️  SMART CLIENT OBJECTS':^55}")
print("=" * 55)

wayne = SmartClient("Wayne Enterprises", 2500000, "Merger")
wayne.add_case("Merger", 85, 450).add_case("Patent", 40, 400)

stark = SmartClient("Stark Industries", 800000, "Patent")
oscorp = SmartClient("Oscorp", 600000, "Litigation")
dalton = SmartClient("Dalton Industries", 1200000, "Merger")

clients = [wayne, stark, oscorp, dalton]

# __str__ via print
print(f"\n📋 Clients:")
for c in clients:
    print(f"  {c}")

# __format__ with spec
print(f"\n📄 Full view:\n{wayne:full}")

# __lt__ via sorted
print(f"\n📊 Sorted by revenue:")
for c in sorted(clients, reverse=True):
    print(f"  {c}")

# __eq__
print(f"\n🔍 Wayne == Wayne? {wayne == SmartClient('Wayne Enterprises', 0)}")

# __gt__
print(f"  Wayne > Stark? {wayne > stark}")

# __add__ (merge)
merged = wayne + stark
print(f"\n🤝 Merged: {merged}")

# __contains__
print(f"\n📂 'Merger' in Wayne? {'Merger' in wayne}")

# max / min (uses __lt__)
print(f"\n🏆 Top client: {max(clients)}")
print(f"📉 Smallest: {min(clients)}")

# sum with float conversion
total_rev = sum(float(c) for c in clients)
print(f"\n💰 Total revenue: ${total_rev:,.2f}")

print(f"\n{'='*55}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting `__hash__` When Defining `__eq__`

```python
class Client:
    def __init__(self, name):
        self.name = name
    
    def __eq__(self, other):
        return self.name == other.name

# ❌ Python automatically sets __hash__ = None when you define __eq__
# client_set = {Client("Wayne")}   # TypeError: unhashable type!

# ✅ Always define __hash__ alongside __eq__
class Client:
    def __init__(self, name):
        self.name = name
    
    def __eq__(self, other):
        return isinstance(other, Client) and self.name == other.name
    
    def __hash__(self):
        return hash(self.name)
```

---

### Pitfall 2: Returning `NotImplemented` vs Raising `NotImplementedError`

```python
class Revenue:
    def __add__(self, other):
        if isinstance(other, Revenue):
            return Revenue(self.amount + other.amount)
        
        # ❌ DON'T raise NotImplementedError
        # raise NotImplementedError    # Crashes immediately!
        
        # ✅ Return NotImplemented (lets Python try __radd__ on the other side)
        return NotImplemented
```

---

### Pitfall 3: `__str__` Not Returning a String

```python
class Price:
    def __init__(self, amount):
        self.amount = amount
    
    # ❌ Returns an int!
    def __str__(self):
        return self.amount      # TypeError: __str__ returned non-string!
    
    # ✅ Must return a string
    def __str__(self):
        return f"${self.amount:,.2f}"
```

---

### Pitfall 4: Infinite Recursion in `__repr__`

```python
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = children or []
    
    # ❌ If children reference parent → infinite recursion!
    def __repr__(self):
        return f"Node({self.value}, children={self.children})"
    
    # ✅ Limit depth
    def __repr__(self):
        child_count = len(self.children)
        return f"Node({self.value}, {child_count} children)"
```

---

### Pitfall 5: `__bool__` vs `__len__` Confusion

```python
class Team:
    def __init__(self):
        self.members = []
    
    def __len__(self):
        return len(self.members)
    
    # If __bool__ is not defined, Python uses __len__:
    # bool(team) → True if len(team) > 0

t = Team()
print(bool(t))    # False (len is 0)

# ⚠️ This can be surprising!
# If you want an empty container to be "truthy", define __bool__:
    def __bool__(self):
        return True    # Always truthy, regardless of size
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### ⚖️ Analogy: Dunder Methods = Rules of the Court

```
PYTHON OPERATIONS = COURTROOM PROCEDURES
  
  print(obj)     = "Witness, state your name for the record."
                   → __str__: "Wayne Enterprises — $2.5M"
  
  repr(obj)      = "Witness, provide your full legal identification."
                   → __repr__: "Client('Wayne Enterprises', 2500000)"
  
  a == b         = "Are these two contracts equivalent?"
                   → __eq__: compare terms and parties
  
  a > b          = "Which party has the larger claim?"
                   → __gt__: compare by value
  
  a + b          = "Merge these two agreements."
                   → __add__: combine into one
  
  len(portfolio) = "How many active cases?"
                   → __len__: count them
  
  if portfolio:  = "Is this portfolio worth reviewing?"
                   → __bool__: is it non-empty?

DONNA'S RULE:
  "Every operation Python performs on your object follows a protocol.
   Override the dunder method = change how the court handles your case."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Temperature Class**
```
Create a Temperature class with:
  - __str__: "72.5°F" or "22.5°C"
  - __repr__: "Temperature(72.5, 'F')"
  - __eq__: equal if same value after conversion
  - __lt__: compare values after conversion
  - to_celsius() / to_fahrenheit()
```

**Exercise 2: Fraction**
```
Create a Fraction class (e.g., 3/4):
  - __str__: "3/4"
  - __add__: add fractions (find LCD)
  - __mul__: multiply fractions
  - __eq__: equal if same after simplification
  - __float__: convert to decimal
```

**Exercise 3: Make It Printable**
```
Take any class from a previous lesson.
Add __str__, __repr__, __eq__, and __hash__.
Demonstrate that it works in print(), sets, and dicts.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Vector Class**
```
Create a Vector class supporting:
  - __add__: v1 + v2 (element-wise)
  - __sub__: v1 - v2
  - __mul__: v * scalar (dot product with scalar)
  - __abs__: magnitude (Euclidean norm)
  - __eq__: component-wise equality
  - __len__: number of dimensions
  - __getitem__: v[0], v[1]
  - __repr__: "Vector(1, 2, 3)"
```

**Exercise 5: Smart Dictionary**
```
Build a SmartDict that:
  - __getitem__: case-insensitive keys
  - __contains__: case-insensitive lookup
  - __add__: merge two dicts
  - __len__: key count
  - __iter__: iterate keys
  - __or__: d1 | d2 (like Python 3.9+ dict merge)
```

**Exercise 6: Matrix**
```
Build a Matrix class:
  - __add__: matrix addition
  - __mul__: matrix multiplication
  - __getitem__: m[row][col] or m[row, col]
  - __str__: pretty-printed grid
  - __eq__: element-wise comparison
  - transpose() method
```

---

### 🔴 Advanced Exercises

**Exercise 7: Money with Currency**
```
Build Money with automatic currency conversion:
  Money(100, "USD") + Money(85, "EUR") → Money(193.50, "USD")
  - Support +, -, *, /, //, ==, <
  - Exchange rates configurable
  - __format__ with locale-aware formatting
```

**Exercise 8: Lazy List**
```
Build a LazyList that:
  - Stores a generator, not a list
  - __getitem__: materializes only needed elements
  - __len__: materializes entire generator (with warning)
  - __iter__: yields lazily
  - Caches computed elements for re-access
  - __repr__: shows computed portion only
```

**Exercise 9: ORM-Style Model**
```
Build a Model base class where dunders enable:

class User(Model):
    name = StringField()
    age = IntField()

user = User(name="Harvey", age=45)
print(user)              # User(name='Harvey', age=45)
print(user == User(name="Harvey", age=45))   # True
users = [User(...), User(...)]
sorted(users)            # Sort by primary key
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: __str__ returns non-string
class Item:
    def __init__(self, price):
        self.price = price
    def __str__(self):
        return self.price    # Returns int! TypeError!

# Bug 2: __eq__ without isinstance check
class Box:
    def __init__(self, size):
        self.size = size
    def __eq__(self, other):
        return self.size == other.size   # AttributeError if other is not Box!

# Bug 3: __add__ doesn't return new object
class Score:
    def __init__(self, value):
        self.value = value
    def __add__(self, other):
        self.value += other.value   # Mutates self!
        return self                 # Returns same object!

# Bug 4: Missing __hash__ with __eq__
class Tag:
    def __init__(self, name):
        self.name = name
    def __eq__(self, other):
        return self.name == other.name
# tags = {Tag("python")}    # TypeError: unhashable!

# Bug 5: __repr__ with f-string missing quotes
class Person:
    def __init__(self, name):
        self.name = name
    def __repr__(self):
        return f"Person({self.name})"    # Missing quotes around name!
```

### Fixed Code ✅

```python
# Fix 1: Return a string
def __str__(self):
    return f"${self.price:,.2f}"

# Fix 2: Type check first
def __eq__(self, other):
    if not isinstance(other, Box):
        return NotImplemented
    return self.size == other.size

# Fix 3: Return a NEW object
def __add__(self, other):
    return Score(self.value + other.value)   # ✅ New object!

# Fix 4: Add __hash__
def __hash__(self):
    return hash(self.name)

# Fix 5: Include quotes in repr
def __repr__(self):
    return f"Person('{self.name}')"   # ✅ Copy-pasteable!
```

---

## 8. 🌍 Real-World Application

### Dunder Methods in Popular Libraries

| Library | Dunders Used | What They Enable |
|---------|-------------|-----------------|
| **pathlib.Path** | `__truediv__`, `__str__` | `Path("dir") / "file.txt"` |
| **datetime** | `__sub__`, `__lt__`, `__str__` | `date2 - date1 → timedelta` |
| **pandas** | `__getitem__`, `__len__`, `__repr__` | `df["column"]`, `len(df)` |
| **SQLAlchemy** | `__eq__`, `__and__` | `User.name == "Harvey"` |
| **NumPy** | `__add__`, `__mul__`, `__getitem__` | Vectorized math: `a + b` |
| **Django** | `__str__`, `__eq__` | `print(user)` → "Harvey Specter" |

---

## 9. ✅ Knowledge Check

---

**Q1.** What does "dunder" stand for?

**Q2.** What is the difference between `__str__` and `__repr__`?

**Q3.** What happens if you define `__eq__` but not `__hash__`?

**Q4.** What should you return when a dunder method doesn't know how to handle an argument type?

**Q5.** What is `@functools.total_ordering` and why is it useful?

**Q6.** What dunder method makes `len(obj)` work?

**Q7.** What is `__radd__` and when is it called?

**Q8.** How does `__call__` change an object's behavior?

**Q9.** What's the difference between overriding a method and defining a dunder?

**Q10.** If `__str__` is not defined, what does Python fall back to?

---

### 📋 Answer Key

> **Q1.** **D**ouble **under**score — methods like `__str__`, `__eq__`, `__add__`.

> **Q2.** `__str__` = user-friendly display. `__repr__` = developer-friendly, unambiguous representation.

> **Q3.** Python sets `__hash__ = None`, making instances unhashable (can't use in sets or as dict keys).

> **Q4.** Return `NotImplemented` (the singleton, not the exception). Python will try the reverse method on the other operand.

> **Q5.** A decorator that auto-generates `__gt__`, `__ge__`, `__le__` from just `__eq__` and `__lt__`.

> **Q6.** `__len__(self)` — must return a non-negative integer.

> **Q7.** Reverse add: called when the left operand's `__add__` returns `NotImplemented`. E.g., `5 + obj` → `obj.__radd__(5)`.

> **Q8.** Makes the object callable like a function: `obj(args)` → `obj.__call__(args)`.

> **Q9.** Overriding replaces an inherited method in a subclass. Dunders are special hooks Python calls for built-in operations.

> **Q10.** `__repr__`. If neither is defined, Python uses the default: `<ClassName object at 0x...>`.

---

## 🎬 End of Lesson Scene

**Mike:** *(demonstrating)* "Now our objects speak Python fluently. Print them, compare them, sort them, add them — all with natural syntax."

**Harvey:** "You've taught the court how to handle your evidence. Next: `@property` — controlling how attributes are accessed and validated without changing the interface."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `__str__` and `__repr__` | ✅ |
| 2 | Comparison dunders (`__eq__`, `__lt__`) + `@total_ordering` | ✅ |
| 3 | `__hash__` and its relationship to `__eq__` | ✅ |
| 4 | Arithmetic dunders (`__add__`, `__radd__`, etc.) | ✅ |
| 5 | Container dunders (`__len__`, `__getitem__`, `__contains__`, `__iter__`) | ✅ |
| 6 | `__call__` — callable objects | ✅ |
| 7 | `__format__` — custom format specs | ✅ |
| 8 | `__bool__` — truthiness | ✅ |
| 9 | Method overriding in child classes with `super()` | ✅ |
| 10 | `NotImplemented` return protocol | ✅ |

---

> **Next Lesson →** Level 3, Lesson 4: *Use @property to Define Read-Only Attributes in Python*

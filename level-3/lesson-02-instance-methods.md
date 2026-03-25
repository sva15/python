# 🏛️ Level 3 – Building | Lesson 2: Define Instance Methods in Python

## *"The Operator's Manual"*

> **O'Reilly Skill Objective:** *Define instance methods inside a class to operate on object data.*

> **Previously Learned:** Functions, `*args`/`**kwargs`, classes, `__init__`, `self`, attributes, `getattr`/`setattr`, `@classmethod`, `@staticmethod`, generators

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

*Tuesday. Harvey watches Mike create another dictionary to hold client data.*

**Harvey:** "You've been storing data in dictionaries and writing standalone functions to process them. That works when you have three clients. What happens with 300?"

```python
# What Mike has been writing:
client = {"name": "Wayne", "revenue": 2500000, "cases": []}
def add_case(client, case_name, hours, rate):
    client["cases"].append({"name": case_name, "hours": hours, "rate": rate})
def total_billing(client):
    return sum(c["hours"] * c["rate"] for c in client["cases"])
```

**Harvey:** "Now the data and the operations are separate. Any function can corrupt the data. The data has no idea what actions are valid on it. It's like a case file with no filing system."

**Mike:** "Instance methods. Functions that live INSIDE the class — bound to the object. The data and the operations that work on that data travel together. `client.add_case(...)` instead of `add_case(client, ...)`."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Three types of methods. We covered class methods and static methods in Level 2. Instance methods are the most important — and the most common."

| Method Type | First Param | Knows About | Purpose |
|-------------|:-----------:|-------------|---------|
| **Instance method** | `self` | THIS specific object | Operate on object data |
| `@classmethod` | `cls` | The class itself | Alternate constructors, class-level ops |
| `@staticmethod` | *(none)* | Nothing (only args) | Utility functions |

**Harvey:** "Instance methods are the default. No decorator needed. They receive `self` — a reference to the specific object calling the method."

```python
class Attorney:
    def __init__(self, name, rate):
        self.name = name
        self.rate = rate
    
    def bill(self, hours):             # Instance method — MOST COMMON
        return self.rate * hours       # Uses self.rate (THIS attorney's rate)

harvey = Attorney("Harvey", 450)
mike = Attorney("Mike", 375)

print(harvey.bill(85))    # 38250 — Harvey's rate × 85
print(mike.bill(85))      # 31875 — Mike's rate × 85
# Same method, different data — that's the power of instance methods!
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Defining Instance Methods ⭐

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
        self.cases = []
        self.active = True
    
    # Instance method — operates on THIS client's data
    def add_case(self, case_name, hours, rate):
        """Add a case to this client."""
        self.cases.append({
            "name": case_name,
            "hours": hours,
            "rate": rate,
            "amount": hours * rate
        })
    
    def total_billing(self):
        """Calculate total billing for this client."""
        return sum(c["amount"] for c in self.cases)
    
    def deactivate(self):
        """Mark this client as inactive."""
        self.active = False
    
    def summary(self):
        """Return a formatted summary."""
        status = "Active" if self.active else "Inactive"
        return (f"{self.name} ({status})\n"
                f"  Revenue: ${self.revenue:,.2f}\n"
                f"  Cases: {len(self.cases)}\n"
                f"  Total Billed: ${self.total_billing():,.2f}")

wayne = Client("Wayne Enterprises", 2500000)
wayne.add_case("Merger", 85, 450)
wayne.add_case("Patent", 40, 400)
print(wayne.summary())
# Wayne Enterprises (Active)
#   Revenue: $2,500,000.00
#   Cases: 2
#   Total Billed: $54,250.00
```

---

### 3.2 `self` — What It Actually Is ⭐⭐

```python
class Attorney:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        print(f"I'm {self.name}")

harvey = Attorney("Harvey")
mike = Attorney("Mike")

# When you call: harvey.greet()
# Python translates it to: Attorney.greet(harvey)
# self = harvey (the specific object calling the method)

harvey.greet()              # I'm Harvey
Attorney.greet(harvey)      # I'm Harvey  ← SAME THING!

mike.greet()                # I'm Mike
Attorney.greet(mike)        # I'm Mike    ← SAME THING!

# self is just a convention — you COULD use any name:
class Demo:
    def method(this):       # Works, but DON'T DO THIS!
        print(this)

# Always use self — it's a universal Python convention.
```

---

### 3.3 Methods Calling Other Methods

```python
class BillingSystem:
    def __init__(self):
        self.invoices = []
        self.total = 0
    
    def create_invoice(self, client, hours, rate):
        """Create an invoice (calls other methods internally)."""
        amount = self._calculate_amount(hours, rate)    # Call internal method
        tax = self._calculate_tax(amount)                # Call another
        
        invoice = {
            "client": client,
            "hours": hours,
            "rate": rate,
            "subtotal": amount,
            "tax": tax,
            "total": amount + tax,
        }
        
        self.invoices.append(invoice)
        self._update_total(invoice["total"])              # And another
        return invoice
    
    def _calculate_amount(self, hours, rate):
        """Private helper method (convention: leading underscore)."""
        return hours * rate
    
    def _calculate_tax(self, amount, tax_rate=0.09):
        """Private helper: calculate tax."""
        return amount * tax_rate
    
    def _update_total(self, amount):
        """Private helper: update running total."""
        self.total += amount
    
    def get_summary(self):
        """Public method: return summary."""
        return {
            "invoice_count": len(self.invoices),
            "total_billed": self.total,
            "avg_invoice": self.total / len(self.invoices) if self.invoices else 0,
        }

billing = BillingSystem()
billing.create_invoice("Wayne", 85, 450)
billing.create_invoice("Stark", 62, 400)
print(billing.get_summary())
# {'invoice_count': 2, 'total_billed': 68632.0, 'avg_invoice': 34316.0}
```

**Mike:** "By convention, methods starting with `_` are 'private' — meant for internal use only. Python doesn't enforce this, but other developers know not to call them directly."

---

### 3.4 Methods That Return `self` — Method Chaining ⭐

```python
class QueryBuilder:
    """SQL-like query builder with method chaining."""
    
    def __init__(self, table):
        self.table = table
        self._columns = ["*"]
        self._conditions = []
        self._order = None
        self._limit = None
    
    def select(self, *columns):
        self._columns = list(columns)
        return self                    # ← Return self for chaining!
    
    def where(self, condition):
        self._conditions.append(condition)
        return self
    
    def order_by(self, column, desc=False):
        direction = "DESC" if desc else "ASC"
        self._order = f"{column} {direction}"
        return self
    
    def limit(self, n):
        self._limit = n
        return self
    
    def build(self):
        """Build the query string."""
        query = f"SELECT {', '.join(self._columns)} FROM {self.table}"
        if self._conditions:
            query += f" WHERE {' AND '.join(self._conditions)}"
        if self._order:
            query += f" ORDER BY {self._order}"
        if self._limit:
            query += f" LIMIT {self._limit}"
        return query

# Method chaining — fluent interface!
query = (QueryBuilder("clients")
         .select("name", "revenue", "type")
         .where("revenue > 500000")
         .where("active = true")
         .order_by("revenue", desc=True)
         .limit(10)
         .build())

print(query)
# SELECT name, revenue, type FROM clients 
# WHERE revenue > 500000 AND active = true 
# ORDER BY revenue DESC LIMIT 10
```

---

### 3.5 Methods That Modify State vs Return New Objects

```python
class CaseFile:
    def __init__(self, case_id, client, charges=None):
        self.case_id = case_id
        self.client = client
        self.charges = list(charges or [])
        self.status = "open"
    
    # --- Mutating methods (modify self) ---
    
    def add_charge(self, charge):
        """Modifies THIS object."""
        self.charges.append(charge)
        return self    # Return self for chaining
    
    def close(self):
        """Modifies THIS object."""
        self.status = "closed"
    
    # --- Non-mutating methods (return new objects) ---
    
    def with_charge(self, charge):
        """Returns a NEW CaseFile with the added charge."""
        new_charges = self.charges + [charge]
        new_case = CaseFile(self.case_id, self.client, new_charges)
        new_case.status = self.status
        return new_case
    
    def merged_with(self, other):
        """Returns a NEW CaseFile merging two case files."""
        combined_charges = self.charges + other.charges
        return CaseFile(
            f"{self.case_id}+{other.case_id}",
            self.client,
            combined_charges
        )
    
    def __repr__(self):
        return f"CaseFile('{self.case_id}', charges={self.charges})"

case = CaseFile("C001", "Wayne", ["fraud"])

# Mutating: modifies the original
case.add_charge("embezzlement")
print(case)    # CaseFile('C001', charges=['fraud', 'embezzlement'])

# Non-mutating: original unchanged, new object returned
case2 = case.with_charge("conspiracy")
print(case)    # CaseFile('C001', charges=['fraud', 'embezzlement']) — unchanged!
print(case2)   # CaseFile('C001', charges=['fraud', 'embezzlement', 'conspiracy'])
```

---

### 3.6 Public, Private, and Name-Mangled Methods

```python
class Account:
    def __init__(self, owner, balance):
        self.owner = owner           # Public attribute
        self._balance = balance      # Convention: "private" (single _)
        self.__pin = 1234            # Name-mangled (double __)
    
    # Public method — part of the API
    def deposit(self, amount):
        """Anyone can call this."""
        self._validate_amount(amount)
        self._balance += amount
    
    def get_balance(self):
        """Public getter."""
        return self._balance
    
    # "Private" method — internal use (convention)
    def _validate_amount(self, amount):
        """Not meant to be called from outside."""
        if amount <= 0:
            raise ValueError("Amount must be positive")
    
    # Name-mangled method — harder to access from outside
    def __verify_pin(self, pin):
        """Truly internal — name gets mangled."""
        return pin == self.__pin

acc = Account("Harvey", 100000)

# Public — accessible
acc.deposit(5000)
print(acc.get_balance())       # 105000

# "Private" — accessible but shouldn't be used externally
acc._validate_amount(100)       # Works, but bad practice

# Name-mangled — accessible via mangled name (but DON'T)
# acc.__verify_pin(1234)        # AttributeError!
# acc._Account__verify_pin(1234)  # Works (mangled name) — but NEVER do this!
```

| Convention | Syntax | Accessibility | Usage |
|:----------:|:------:|:---:|-------|
| Public | `method()` | ✅ Anyone | Part of the API |
| Protected | `_method()` | ✅ But don't | Internal helpers |
| Name-mangled | `__method()` | Via `_Class__method` | Avoid name clashes in inheritance |

---

### 3.7 Computed Methods vs Stored Attributes

```python
class Attorney:
    def __init__(self, name, rate, cases=None):
        self.name = name
        self.rate = rate
        self.cases = cases or []
    
    # Stored: directly reads self.name
    def get_name(self):
        return self.name
    
    # Computed: calculates on each call
    def total_billing(self):
        return sum(c["hours"] * c["rate"] for c in self.cases)
    
    # Computed with logic
    def tier(self):
        total = self.total_billing()
        if total >= 100000: return "Platinum"
        elif total >= 50000: return "Gold"
        return "Silver"
    
    # Computed with external data
    def utilization(self, target_hours=2000):
        actual = sum(c["hours"] for c in self.cases)
        return (actual / target_hours) * 100 if target_hours else 0

harvey = Attorney("Harvey", 450, [
    {"hours": 85, "rate": 450, "client": "Wayne"},
    {"hours": 70, "rate": 450, "client": "Dalton"},
])

print(f"{harvey.name}: ${harvey.total_billing():,.2f}")   # $69,750.00
print(f"Tier: {harvey.tier()}")                            # Gold
print(f"Utilization: {harvey.utilization():.1f}%")         # 7.8%
```

---

### 3.8 Methods with Complex Logic

```python
class TimeTracker:
    """Track billable hours with validation and reporting."""
    
    def __init__(self, attorney_name, max_daily_hours=12):
        self.attorney = attorney_name
        self.max_daily = max_daily_hours
        self.entries = []
    
    def log_hours(self, client, hours, date=None, billable=True):
        """Log billable hours with validation."""
        from datetime import datetime
        
        if hours <= 0:
            raise ValueError("Hours must be positive")
        if hours > self.max_daily:
            raise ValueError(f"Cannot log more than {self.max_daily}h per day")
        
        entry = {
            "client": client,
            "hours": hours,
            "date": date or datetime.now().strftime("%Y-%m-%d"),
            "billable": billable,
        }
        self.entries.append(entry)
        return entry
    
    def hours_for_client(self, client):
        """Total hours for a specific client."""
        return sum(e["hours"] for e in self.entries if e["client"] == client)
    
    def hours_for_date(self, date):
        """Total hours for a specific date."""
        return sum(e["hours"] for e in self.entries if e["date"] == date)
    
    def billable_hours(self):
        """Total billable hours."""
        return sum(e["hours"] for e in self.entries if e["billable"])
    
    def non_billable_hours(self):
        """Total non-billable hours."""
        return sum(e["hours"] for e in self.entries if not e["billable"])
    
    def billable_ratio(self):
        """Percentage of hours that are billable."""
        total = self.billable_hours() + self.non_billable_hours()
        return (self.billable_hours() / total * 100) if total else 0
    
    def top_clients(self, n=5):
        """Top N clients by hours."""
        from collections import Counter
        counts = Counter()
        for e in self.entries:
            counts[e["client"]] += e["hours"]
        return counts.most_common(n)
    
    def weekly_report(self):
        """Generate a formatted weekly report."""
        lines = [
            f"{'='*50}",
            f"{'⏰ WEEKLY TIME REPORT':^50}",
            f"{'='*50}",
            f"  Attorney: {self.attorney}",
            f"  Total Entries: {len(self.entries)}",
            f"  Billable: {self.billable_hours():.1f}h",
            f"  Non-Billable: {self.non_billable_hours():.1f}h",
            f"  Ratio: {self.billable_ratio():.0f}% billable",
            f"",
            f"  Top Clients:",
        ]
        for client, hours in self.top_clients():
            bar = "█" * int(hours / 2)
            lines.append(f"    {client:<20} [{bar:<20}] {hours:.1f}h")
        lines.append(f"{'='*50}")
        return "\n".join(lines)

# Use it
tracker = TimeTracker("Harvey")
tracker.log_hours("Wayne Enterprises", 8.5, "2026-03-25")
tracker.log_hours("Dalton Industries", 6.0, "2026-03-25")
tracker.log_hours("Wayne Enterprises", 7.5, "2026-03-26")
tracker.log_hours("Stark Industries", 5.0, "2026-03-26")
tracker.log_hours("Internal", 2.0, "2026-03-26", billable=False)

print(tracker.weekly_report())
```

---

### 3.9 Solving the Problem — Full Client Management System

```python
# ============================================
# Pearson Specter Litt — Client Management via Instance Methods
# ============================================

from datetime import datetime
from collections import Counter

class Client:
    """Client with rich instance methods for all operations."""
    
    _registry = {}
    
    def __init__(self, name, revenue, case_type="General", attorney=None):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
        self.attorney = attorney
        self.cases = []
        self.notes = []
        self.active = True
        self.created = datetime.now()
        Client._registry[name] = self
    
    # --- Core operations ---
    
    def add_case(self, title, hours, rate=None):
        """Add a case to this client."""
        case_rate = rate or self._default_rate()
        case = {
            "title": title,
            "hours": hours,
            "rate": case_rate,
            "amount": hours * case_rate,
            "date": datetime.now().isoformat(),
        }
        self.cases.append(case)
        return self
    
    def add_note(self, text):
        """Add a note with timestamp."""
        self.notes.append({
            "text": text,
            "timestamp": datetime.now().isoformat(),
        })
        return self
    
    def deactivate(self, reason=""):
        """Mark client as inactive."""
        self.active = False
        self.add_note(f"Deactivated: {reason}" if reason else "Deactivated")
        return self
    
    def reactivate(self):
        """Reactivate the client."""
        self.active = True
        self.add_note("Reactivated")
        return self
    
    # --- Computations ---
    
    def total_billed(self):
        """Total billing across all cases."""
        return sum(c["amount"] for c in self.cases)
    
    def total_hours(self):
        """Total hours across all cases."""
        return sum(c["hours"] for c in self.cases)
    
    def average_rate(self):
        """Weighted average billing rate."""
        if not self.cases:
            return 0
        return self.total_billed() / self.total_hours()
    
    def tier(self):
        """Classify client by revenue."""
        if self.revenue >= 1000000: return "💎 Platinum"
        elif self.revenue >= 500000: return "🥇 Gold"
        elif self.revenue >= 100000: return "🥈 Silver"
        return "🥉 Bronze"
    
    def profitability(self):
        """Revenue-to-billing ratio."""
        billed = self.total_billed()
        return (self.revenue / billed * 100) if billed else float('inf')
    
    def _default_rate(self):
        """Internal: determine default rate based on tier."""
        if self.revenue >= 1000000: return 500
        elif self.revenue >= 500000: return 425
        return 350
    
    # --- Comparison and interaction ---
    
    def is_higher_value_than(self, other):
        """Compare revenue with another client."""
        return self.revenue > other.revenue
    
    def transfer_to(self, new_attorney):
        """Transfer this client to a new attorney."""
        old = self.attorney
        self.attorney = new_attorney
        self.add_note(f"Transferred from {old} to {new_attorney}")
        return self
    
    # --- Output ---
    
    def summary(self):
        """Comprehensive summary."""
        lines = [
            f"\n{'─'*50}",
            f"  {self.tier()} {self.name}",
            f"{'─'*50}",
            f"  Attorney: {self.attorney or 'Unassigned'}",
            f"  Status: {'✅ Active' if self.active else '❌ Inactive'}",
            f"  Revenue: ${self.revenue:>12,.2f}",
            f"  Billed:  ${self.total_billed():>12,.2f}",
            f"  Cases: {len(self.cases)}",
            f"  Avg Rate: ${self.average_rate():,.2f}/hr" if self.cases else "",
        ]
        if self.notes:
            lines.append(f"  Notes: {len(self.notes)}")
        lines.append(f"{'─'*50}")
        return "\n".join(line for line in lines if line)
    
    @classmethod
    def find(cls, name):
        return cls._registry.get(name)
    
    def __repr__(self):
        return f"Client('{self.name}', ${self.revenue:,.0f})"


# === Build and demonstrate ===
print("=" * 50)
print(f"{'🏛️  CLIENT MANAGEMENT SYSTEM':^50}")
print("=" * 50)

# Create clients with method chaining
wayne = (Client("Wayne Enterprises", 2500000, "Merger", "Harvey")
         .add_case("Wayne v. Stark", 85, 450)
         .add_case("Dalton Merger", 70, 450)
         .add_note("VIP — priority service"))

stark = (Client("Stark Industries", 800000, "Patent", "Mike")
         .add_case("Arc Reactor Patent", 62, 400))

oscorp = (Client("Oscorp", 600000, "Litigation", "Louis")
          .add_case("Product Liability", 55, 425))

# Instance methods in action
for client in [wayne, stark, oscorp]:
    print(client.summary())

# Comparison
print(f"\n🔍 Wayne > Stark? {wayne.is_higher_value_than(stark)}")

# Transfer
stark.transfer_to("Rachel")
print(f"\n📋 Stark now with: {stark.attorney}")

# Find by name
found = Client.find("Wayne Enterprises")
print(f"\n🔎 Found: {found}")

print(f"\n{'='*50}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting `self` as the First Parameter

```python
# ❌ Missing self
class Calculator:
    def add(a, b):          # Missing self!
        return a + b

c = Calculator()
# c.add(2, 3)   # TypeError: add() takes 2 positional arguments but 3 were given
# Python passes c as first arg → add(c, 2, 3) — too many args!

# ✅ Include self
class Calculator:
    def add(self, a, b):    # self receives the instance
        return a + b
```

---

### Pitfall 2: Forgetting `self.` When Accessing Attributes

```python
class Client:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        # ❌ Missing self. — creates a local variable!
        return f"Hello, {name}"    # NameError: name 'name' is not defined
    
    def greet_fixed(self):
        # ✅ Access via self
        return f"Hello, {self.name}"
```

---

### Pitfall 3: Calling Methods Without Parentheses

```python
class Timer:
    def elapsed(self):
        return 42.5

t = Timer()
print(t.elapsed)      # <bound method Timer.elapsed of ...> — NOT the value!
print(t.elapsed())     # 42.5 ✅ — parentheses call the method!
```

---

### Pitfall 4: Mutating Arguments Inside Methods

```python
class Processor:
    def process(self, items):
        items.sort()         # ⚠️ Mutates the caller's list!
        return items

data = [3, 1, 2]
p = Processor()
p.process(data)
print(data)   # [1, 2, 3] — original modified!

# ✅ Work on a copy
class Processor:
    def process(self, items):
        sorted_items = sorted(items)    # Returns new list
        return sorted_items
```

---

### Pitfall 5: Method vs Function — Unbound Calls

```python
class Dog:
    def bark(self):
        return "Woof!"

d = Dog()

# Bound method call — Python passes d as self
d.bark()           # "Woof!" ✅

# Unbound call — YOU must pass the instance manually
Dog.bark(d)        # "Woof!" ✅
Dog.bark()         # TypeError: bark() missing 1 required positional argument: 'self'

# Assigning method to variable
bark = d.bark      # Bound method — remembers d
bark()              # "Woof!" ✅

bark_unbound = Dog.bark   # Unbound — needs self
bark_unbound(d)            # "Woof!" ✅
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📋 Analogy: Instance Methods = Things a Person Can DO

```
OBJECT = A person (attorney Harvey)
  📄 Attributes = What he KNOWS (name, rate, cases)
  🔧 Methods = What he CAN DO (bill, negotiate, advise)

  harvey.name             → "Harvey Specter" (data)
  harvey.bill(hours=85)   → $38,250 (action on data)
  harvey.negotiate(case)  → "Settlement reached" (action)

KEY INSIGHT:
  Data + Operations = Object
  
  Dict + standalone functions   = data and code are separate
  Class + instance methods      = data and code travel TOGETHER
  
  When Harvey moves offices, his methods move with him.
  You don't need separate "harvey_bill" and "mike_bill" functions.
  You just call: attorney.bill(hours)
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Bank Account**
```
Create a BankAccount class with methods:
  - deposit(amount)
  - withdraw(amount) — with insufficient funds check
  - get_balance()
  - transfer_to(other_account, amount)
  - statement() — print all transactions
```

**Exercise 2: Shopping Cart**
```
Create a ShoppingCart with methods:
  - add_item(name, price, quantity=1)
  - remove_item(name)
  - update_quantity(name, new_qty)
  - subtotal()
  - apply_discount(percentage) → returns new total
  - checkout() → returns receipt string
```

**Exercise 3: Contact Book**
```
Create a Contact class with methods:
  - add_phone(label, number)
  - add_email(label, address)
  - primary_phone()
  - primary_email()
  - vcard() — returns vCard format string
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Playlist Manager**
```
Build a Playlist class with:
  - add_song(title, artist, duration)
  - remove_song(title)
  - shuffle() → randomize order
  - sort_by(field) → sort by title/artist/duration
  - total_duration() → formatted as HH:MM:SS
  - search(query) → find matching songs
  - next_song() / prev_song() → navigation with wraparound
```

**Exercise 5: Method Chaining Builder**
```
Build an HTMLBuilder with method chaining:

html = (HTMLBuilder()
        .tag("div", class_="container")
        .tag("h1").text("Title").close()
        .tag("p").text("Paragraph").close()
        .close()
        .build())
```

**Exercise 6: State Machine**
```
Build a TrafficLight class where:
  - Methods transition between states (red→green→yellow→red)
  - Invalid transitions raise errors
  - History of state changes is tracked
  - Each state has a duration
  - .run(cycles=3) simulates the traffic light
```

---

### 🔴 Advanced Exercises

**Exercise 7: Fluent Validator**
```
Build a fluent validation system:

result = (Validator(data)
          .field("name").required().min_length(2).max_length(50)
          .field("age").required().is_int().between(0, 150)
          .field("email").required().matches(r".*@.*\\..*")
          .validate())
```

**Exercise 8: Event Emitter**
```
Build an EventEmitter class:
  - on(event, callback)
  - off(event, callback)
  - emit(event, *args)
  - once(event, callback) — fires only once
  
emitter.on("data", lambda d: print(f"Got: {d}"))
emitter.emit("data", "hello")  # Got: hello
```

**Exercise 9: Data Frame Basics**
```
Build a simple DataFrame class:
  - select(*columns) → new DataFrame with subset
  - where(condition_func) → filtered
  - sort_by(column)
  - group_by(column).agg(func)
  - head(n) / tail(n)
  - to_csv(path)
All methods return new DataFrames (immutable operations).
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Missing self
class Greeter:
    def greet(name):              # Missing self!
        return f"Hello, {name}"

# Bug 2: Missing self. prefix
class Counter:
    def __init__(self):
        self.count = 0
    def increment(self):
        count += 1                 # NameError — should be self.count!

# Bug 3: Not calling the method
class Clock:
    def current_time(self):
        from datetime import datetime
        return datetime.now().strftime("%H:%M")

c = Clock()
print(f"Time: {c.current_time}")   # Prints method object, not time!

# Bug 4: Returning None implicitly
class Calculator:
    def add(self, a, b):
        result = a + b             # Forgot to return!

print(Calculator().add(2, 3))      # None!

# Bug 5: Method modifies shared mutable default
class Logger:
    def __init__(self, entries=[]):   # Mutable default!
        self.entries = entries
```

### Fixed Code ✅

```python
# Fix 1: Add self
class Greeter:
    def greet(self, name):
        return f"Hello, {name}"

# Fix 2: Use self.count
class Counter:
    def __init__(self):
        self.count = 0
    def increment(self):
        self.count += 1

# Fix 3: Add parentheses to call the method
c = Clock()
print(f"Time: {c.current_time()}")   # ✅

# Fix 4: Return the result
class Calculator:
    def add(self, a, b):
        return a + b                   # ✅

# Fix 5: Use None sentinel
class Logger:
    def __init__(self, entries=None):
        self.entries = entries or []    # ✅
```

---

## 8. 🌍 Real-World Application

### Instance Methods Everywhere

| Framework | Instance Methods |
|-----------|-----------------|
| **Django** | `user.save()`, `queryset.filter()`, `form.is_valid()` |
| **Flask** | `app.route()`, `response.set_cookie()` |
| **SQLAlchemy** | `session.commit()`, `query.all()` |
| **Pandas** | `df.describe()`, `df.groupby()`, `df.to_csv()` |
| **Requests** | `response.json()`, `session.get()` |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is `self` in an instance method?

**Q2.** Do you need a decorator for instance methods?

**Q3.** What happens when you call `obj.method(arg)`?

**Q4.** What is the naming convention for "private" methods?

**Q5.** How do you enable method chaining?

**Q6.** What is the difference between mutating and non-mutating methods?

**Q7.** Why is forgetting `self` as the first parameter a common error?

**Q8.** Can an instance method call other methods on the same object?

**Q9.** What happens if you reference `name` instead of `self.name` inside a method?

**Q10.** What is the difference between a bound and unbound method call?

---

### 📋 Answer Key

> **Q1.** A reference to the specific instance calling the method. It gives the method access to that object's attributes and other methods.

> **Q2.** No. Instance methods are the default — no decorator needed.

> **Q3.** Python translates it to `ClassName.method(obj, arg)` — passing `obj` as `self`.

> **Q4.** Single underscore prefix: `_helper_method()`. It signals "internal use only."

> **Q5.** Return `self` from methods: `return self`.

> **Q6.** Mutating methods change `self`'s state in place. Non-mutating methods return new objects, leaving `self` unchanged.

> **Q7.** Python passes the instance automatically, so `def add(a, b)` receives `(self, a)` — `b` is missing.

> **Q8.** Yes! Use `self.other_method()` to call other methods on the same object.

> **Q9.** Python looks for a local variable `name`. If it doesn't exist, you get a `NameError`.

> **Q10.** Bound: `obj.method()` — `self` is automatically `obj`. Unbound: `Class.method(obj)` — you pass the instance manually.

---

## 🎬 End of Lesson Scene

**Harvey:** "Data and operations, traveling together. No more scattered functions. No more 'which client dict does this function expect?' The object knows what it can do."

**Mike:** "Next: overriding methods and dunder methods. Teaching Python how YOUR objects should behave when printed, compared, added, or converted."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Instance methods and `self` | ✅ |
| 2 | How `obj.method()` translates to `Class.method(obj)` | ✅ |
| 3 | Methods calling other methods | ✅ |
| 4 | Method chaining (returning `self`) | ✅ |
| 5 | Mutating vs non-mutating methods | ✅ |
| 6 | Public, private (`_`), name-mangled (`__`) | ✅ |
| 7 | Computed methods vs stored attributes | ✅ |
| 8 | Complex methods with validation and logic | ✅ |
| 9 | Bound vs unbound method calls | ✅ |
| 10 | Client Management System with rich methods | ✅ |

---

> **Next Lesson →** Level 3, Lesson 3: *Override Methods and Dunder Methods in Python*

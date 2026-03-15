# 🏛️ Level 1 – Exploring | Lesson 13: Instantiate Objects in Python

## *"The Roster"*

> **O'Reilly Skill Objective:** *Instantiate objects in Python.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, conditionals, boolean operators, loops, functions, classes, `__init__`, `self`, attributes, methods, `__str__`/`__repr__`, encapsulation, `@property`

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

*It's 8:30 AM. Jessica has called an all-hands meeting. She's standing at the head of the conference table with a spreadsheet on the projector.*

**Jessica:** "We've grown. Twelve partners. Forty-seven associates. Two hundred and thirty-seven active clients. And not a single unified system tracking all of them."

*She pulls up a screen showing disconnected files, spreadsheets, and legacy tools.*

**Jessica:** "Harvey, you had your new associate build that `Client` class. Impressive. But one blueprint is useless if you can't build an army from it."

**Harvey:** *(to you)* "Yesterday you learned to define a class — the blueprint. Today you learn to **instantiate** — to build actual objects from that blueprint. Dozens of them. Hundreds. And then manage them: store them in collections, compare them, pass them between systems, and make them interact with each other."

**Mike:** "Instantiation is where classes become useful. A blueprint on a shelf does nothing. But 237 `Client` objects created from that blueprint, stored in a list, filtered, sorted, and processed? That's a management system."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Lesson 12 was about designing the blueprint. This lesson is about building from it."

| Lesson 12 | Lesson 13 |
|-----------|-----------|
| **Define** a class | **Create** objects from it |
| The blueprint | The buildings |
| `class Client:` | `client = Client("Wayne", ...)` |
| Attributes & methods defined | Attributes populated, methods called |
| One definition | Unlimited instances |

**Harvey:** "Today's agenda:"
1. **Create** objects (instantiation)
2. **Store** objects in collections (lists, dicts)
3. **Compare** objects (equality, ordering)
4. **Copy** objects (shallow vs deep)
5. **Pass** objects between functions
6. **Make** objects interact with each other

**Harvey:** "By end of day, I want a full firm roster — partners, associates, clients — all as objects that know how to manage themselves."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "From blueprint to reality."

---

### 3.1 Creating Objects — Instantiation Basics

```python
class Attorney:
    def __init__(self, name, title, rate):
        self.name = name
        self.title = title
        self.rate = rate
    
    def __str__(self):
        return f"{self.title} {self.name} (${self.rate}/hr)"

# INSTANTIATION — creating objects from the class
a1 = Attorney("Harvey Specter", "Senior Partner", 450)
a2 = Attorney("Mike Ross", "Associate", 375)
a3 = Attorney("Louis Litt", "Name Partner", 425)

print(a1)  # Senior Partner Harvey Specter ($450/hr)
print(a2)  # Associate Mike Ross ($375/hr)
print(a3)  # Name Partner Louis Litt ($425/hr)

# Each object is independent
print(a1.name)   # Harvey Specter
print(a2.name)   # Mike Ross — different object, different data
print(a1 is a2)  # False — different objects in memory
```

**Mike:** "What happens during `Attorney("Harvey Specter", "Senior Partner", 450)`:"

1. Python creates a new, empty `Attorney` object in memory
2. Python calls `__init__(self, "Harvey Specter", "Senior Partner", 450)`
3. `self` refers to the brand-new object
4. `__init__` attaches attributes via `self.name = name`, etc.
5. The fully initialized object is returned and assigned to `a1`

---

### 3.2 Storing Objects in Collections ⭐

**Mike:** "Individual objects are useful. Collections of objects are powerful."

#### Lists of Objects
```python
class Client:
    def __init__(self, name, revenue, case_type):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
        self.active = True
    
    def classify(self):
        if self.revenue >= 1000000: return "Platinum"
        elif self.revenue >= 500000: return "Gold"
        elif self.revenue >= 100000: return "Silver"
        else: return "Bronze"
    
    def __str__(self):
        return f"{self.name} — {self.classify()} — ${self.revenue:,.2f}"

# Create a list of client objects
clients = [
    Client("Wayne Enterprises", 2500000, "Merger"),
    Client("Stark Industries", 800000, "Patent"),
    Client("Oscorp", 600000, "Litigation"),
    Client("Luthor Corp", 150000, "Corporate"),
    Client("Dalton Industries", 1200000, "Merger"),
    Client("Gillis Industries", 75000, "Patent"),
]

# Loop through objects
for client in clients:
    print(client)

# Filter objects
platinum_clients = [c for c in clients if c.classify() == "Platinum"]
print(f"\nPlatinum clients: {len(platinum_clients)}")
for c in platinum_clients:
    print(f"  {c.name}")

# Aggregate data from objects
total_revenue = sum(c.revenue for c in clients)
print(f"\nTotal firm revenue: ${total_revenue:,.2f}")
```

#### Dictionaries of Objects
```python
# Look up clients by name
client_registry = {}
for client in clients:
    client_registry[client.name] = client

# Quick access
wayne = client_registry["Wayne Enterprises"]
print(f"Wayne tier: {wayne.classify()}")
print(f"Wayne revenue: ${wayne.revenue:,.2f}")

# Group clients by tier
from collections import defaultdict

by_tier = defaultdict(list)
for client in clients:
    by_tier[client.classify()].append(client)

for tier, tier_clients in sorted(by_tier.items()):
    print(f"\n{tier}:")
    for c in tier_clients:
        print(f"  {c.name}: ${c.revenue:,.2f}")
```

---

### 3.3 Sorting Objects

```python
# Sort by attribute using sorted() with key
clients_by_revenue = sorted(clients, key=lambda c: c.revenue, reverse=True)

print("Clients by Revenue (descending):")
for i, c in enumerate(clients_by_revenue, 1):
    print(f"  {i}. {c.name}: ${c.revenue:,.2f}")

# Sort by name (alphabetical)
clients_by_name = sorted(clients, key=lambda c: c.name)

# Sort by multiple criteria — tier first, then revenue within tier
clients_sorted = sorted(clients, key=lambda c: (-c.revenue, c.name))
```

#### Making Objects Sortable with `__lt__`
```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    
    def __lt__(self, other):
        """Less than — enables sorting."""
        return self.revenue < other.revenue
    
    def __str__(self):
        return f"{self.name}: ${self.revenue:,.2f}"

clients = [Client("Wayne", 2500000), Client("Stark", 800000), Client("Oscorp", 600000)]

# Now you can sort directly!
for c in sorted(clients):
    print(c)
# Oscorp: $600,000.00
# Stark: $800,000.00
# Wayne: $2,500,000.00

for c in sorted(clients, reverse=True):
    print(c)
# Wayne first (highest)
```

---

### 3.4 Comparing Objects

**Mike:** "By default, `==` checks identity (same object in memory), not equality (same values)."

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

c1 = Client("Wayne", 2500000)
c2 = Client("Wayne", 2500000)

print(c1 == c2)    # False — different objects!
print(c1 is c2)    # False — confirms different objects
```

#### Defining Equality with `__eq__`
```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    
    def __eq__(self, other):
        """Two clients are equal if they have the same name."""
        if not isinstance(other, Client):
            return NotImplemented
        return self.name == other.name
    
    def __hash__(self):
        """Required when you define __eq__ (for use in sets/dicts)."""
        return hash(self.name)

c1 = Client("Wayne", 2500000)
c2 = Client("Wayne", 2500000)
c3 = Client("Stark", 800000)

print(c1 == c2)   # True  — same name
print(c1 == c3)   # False — different name
print(c1 is c2)   # False — still different objects in memory

# Can now use in sets
unique_clients = {c1, c2, c3}
print(len(unique_clients))   # 2 (Wayne counted once)
```

---

### 3.5 Copying Objects — Shallow vs Deep

```python
import copy

class Team:
    def __init__(self, name, members):
        self.name = name
        self.members = members   # This is a list (mutable!)
    
    def __str__(self):
        return f"Team {self.name}: {self.members}"

original = Team("Legal", ["Harvey", "Mike", "Katrina"])

# === Assignment (NOT a copy — just another reference!) ===
alias = original
alias.members.append("Rachel")
print(original)   # Team Legal: ['Harvey', 'Mike', 'Katrina', 'Rachel']
# Original changed! 'alias' and 'original' are the SAME object

# === Shallow Copy ===
shallow = copy.copy(original)
shallow.name = "Legal-Copy"       # This is fine — strings are immutable
shallow.members.append("Louis")    # This ALSO changes original!
print(original.members)   # ['Harvey', 'Mike', 'Katrina', 'Rachel', 'Louis']
# Shallow copy creates a new Team, but members list is SHARED!

# === Deep Copy ===
deep = copy.deepcopy(original)
deep.name = "Legal-Deep"
deep.members.append("Jessica")   # Only affects the deep copy
print(original.members)  # ['Harvey', 'Mike', 'Katrina', 'Rachel', 'Louis']
print(deep.members)      # ['Harvey', 'Mike', 'Katrina', 'Rachel', 'Louis', 'Jessica']
# Deep copy = completely independent — everything is duplicated
```

| Type | Syntax | New Object? | Nested Data? |
|------|--------|------------|-------------|
| Assignment | `b = a` | ❌ Same object | ❌ Same |
| Shallow copy | `copy.copy(a)` | ✅ New object | ❌ Shared |
| Deep copy | `copy.deepcopy(a)` | ✅ New object | ✅ Independent |

---

### 3.6 Objects as Function Arguments

```python
class Client:
    def __init__(self, name, revenue, rate):
        self.name = name
        self.revenue = revenue
        self.rate = rate
    
    def classify(self):
        if self.revenue >= 1000000: return "Platinum"
        elif self.revenue >= 500000: return "Gold"
        elif self.revenue >= 100000: return "Silver"
        else: return "Bronze"

# Functions that accept objects
def generate_invoice(client, hours):
    """Generate an invoice for a client."""
    subtotal = hours * client.rate
    tax = subtotal * 0.09
    total = subtotal + tax
    print(f"\nInvoice for {client.name}:")
    print(f"  Hours: {hours} × ${client.rate:.2f}")
    print(f"  Subtotal: ${subtotal:,.2f}")
    print(f"  Tax: ${tax:,.2f}")
    print(f"  Total: ${total:,.2f}")
    return total

# Functions that return objects
def create_vip_client(name, revenue):
    """Create a pre-configured VIP client."""
    client = Client(name, revenue, 500.00)   # Premium rate
    return client

# Functions that modify objects
def apply_rate_increase(client, percentage):
    """Increase a client's billing rate."""
    old_rate = client.rate
    client.rate *= (1 + percentage)
    print(f"{client.name}: rate ${old_rate:.2f} → ${client.rate:.2f}")

# Usage
wayne = Client("Wayne Enterprises", 2500000, 450.00)
generate_invoice(wayne, 85)
apply_rate_increase(wayne, 0.10)    # 10% increase
generate_invoice(wayne, 85)          # Now uses the new rate

vip = create_vip_client("Queen Industries", 3000000)
print(f"VIP: {vip.name} at ${vip.rate}/hr")
```

> ⚠️ **Key Insight:** Objects passed to functions are passed **by reference** — modifying the object inside the function changes the original object.

---

### 3.7 Objects Interacting with Each Other

```python
class Attorney:
    def __init__(self, name, rate):
        self.name = name
        self.rate = rate
        self.clients = []
        self.total_billed = 0
    
    def take_client(self, client):
        """Assign a client to this attorney."""
        self.clients.append(client)
        client.attorney = self     # The client now knows its attorney too!
        print(f"  {self.name} assigned to {client.name}")
    
    def bill_client(self, client, hours):
        """Bill a specific client."""
        if client not in self.clients:
            print(f"  ❌ {client.name} is not assigned to {self.name}")
            return 0
        total = hours * self.rate
        self.total_billed += total
        return total
    
    def __str__(self):
        return f"{self.name} — {len(self.clients)} clients — ${self.total_billed:,.2f} billed"


class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
        self.attorney = None      # Will be set when assigned
    
    def __str__(self):
        atty = self.attorney.name if self.attorney else "Unassigned"
        return f"{self.name} (${self.revenue:,.2f}) — Attorney: {atty}"


# Create objects
harvey = Attorney("Harvey Specter", 450)
mike = Attorney("Mike Ross", 375)

wayne = Client("Wayne Enterprises", 2500000)
stark = Client("Stark Industries", 800000)
oscorp = Client("Oscorp", 600000)

# Objects interacting
harvey.take_client(wayne)
harvey.take_client(oscorp)
mike.take_client(stark)

# Bill through the attorney
harvey.bill_client(wayne, 85)
mike.bill_client(stark, 62)

# Each object knows about the other
print(f"\n{wayne}")           # Wayne → Attorney: Harvey Specter
print(f"{harvey}")             # Harvey → 2 clients, $ billed
print(f"{wayne.attorney}")     # Harvey Specter (the actual Attorney object!)
```

---

### 3.8 Factory Pattern — Alternative Ways to Create Objects

```python
class Client:
    def __init__(self, name, revenue, rate, case_type):
        self.name = name
        self.revenue = revenue
        self.rate = rate
        self.case_type = case_type
    
    @classmethod
    def platinum(cls, name, revenue):
        """Factory: create a Platinum client with premium rate."""
        return cls(name, revenue, 500.00, "Merger")
    
    @classmethod
    def probono(cls, name, case_type="Litigation"):
        """Factory: create a pro-bono client with zero rate."""
        return cls(name, 0, 0.00, case_type)
    
    @classmethod
    def from_dict(cls, data):
        """Factory: create a client from a dictionary."""
        return cls(
            data["name"],
            data.get("revenue", 0),
            data.get("rate", 300.00),
            data.get("case_type", "General")
        )
    
    def __str__(self):
        return f"{self.name} — ${self.revenue:,.2f} — ${self.rate}/hr — {self.case_type}"

# Standard instantiation
c1 = Client("Wayne", 2500000, 450, "Merger")

# Factory methods — alternative constructors
c2 = Client.platinum("Queen Industries", 3000000)
c3 = Client.probono("Legal Aid Foundation")
c4 = Client.from_dict({"name": "Stark", "revenue": 800000, "rate": 375, "case_type": "Patent"})

for c in [c1, c2, c3, c4]:
    print(c)
```

---

### 3.9 `isinstance()` and Type Checking

```python
class Attorney:
    def __init__(self, name):
        self.name = name

class Client:
    def __init__(self, name):
        self.name = name

class Partner(Attorney):
    pass

harvey = Attorney("Harvey")
wayne = Client("Wayne")
jessica = Partner("Jessica")

# Type checking
print(isinstance(harvey, Attorney))   # True
print(isinstance(wayne, Client))     # True
print(isinstance(wayne, Attorney))   # False — Wayne is not an Attorney!

# Works with inheritance
print(isinstance(jessica, Partner))   # True
print(isinstance(jessica, Attorney))  # True — Partner IS an Attorney

# Use in functions
def process_entity(entity):
    if isinstance(entity, Attorney):
        print(f"Attorney: {entity.name}")
    elif isinstance(entity, Client):
        print(f"Client: {entity.name}")
    else:
        print(f"Unknown: {entity}")

process_entity(harvey)   # Attorney: Harvey
process_entity(wayne)    # Client: Wayne
```

---

### 3.10 Solving Jessica's Problem — The Firm Roster System

```python
# ============================================
# Pearson Specter Litt — Firm Roster System
# ============================================

class Person:
    """Base class for all firm personnel and contacts."""
    _id_counter = 0
    
    def __init__(self, name, email):
        Person._id_counter += 1
        self.id = Person._id_counter
        self.name = name
        self.email = email


class Attorney(Person):
    """An attorney at the firm."""
    
    def __init__(self, name, email, title, rate, specialization):
        super().__init__(name, email)
        self.title = title
        self.rate = rate
        self.specialization = specialization
        self.clients = []
        self.total_hours = 0
        self.total_billed = 0.0
    
    def assign_client(self, client):
        if client not in self.clients:
            self.clients.append(client)
            client.attorney = self
    
    def bill(self, client, hours):
        amount = hours * self.rate
        self.total_hours += hours
        self.total_billed += amount
        client.total_billed += amount
        return amount
    
    def utilization(self, target_hours=160):
        return (self.total_hours / target_hours * 100) if target_hours > 0 else 0
    
    def __str__(self):
        return (f"[{self.id:03d}] {self.title} {self.name} | "
                f"{self.specialization} | ${self.rate}/hr | "
                f"{len(self.clients)} clients | ${self.total_billed:,.2f} billed")


class Client(Person):
    """A client of the firm."""
    
    TIERS = [
        (1000000, "Platinum", "💎"),
        (500000, "Gold", "🥇"),
        (100000, "Silver", "🥈"),
        (0, "Bronze", "🥉"),
    ]
    
    def __init__(self, name, email, revenue, case_type):
        super().__init__(name, email)
        self.revenue = revenue
        self.case_type = case_type
        self.attorney = None
        self.total_billed = 0.0
        self.active = True
    
    def classify(self):
        for threshold, tier, icon in Client.TIERS:
            if self.revenue >= threshold:
                return tier, icon
        return "Bronze", "🥉"
    
    def __str__(self):
        tier, icon = self.classify()
        atty = self.attorney.name if self.attorney else "Unassigned"
        return (f"[{self.id:03d}] {icon} {self.name} | {tier} | "
                f"${self.revenue:,.2f} | {self.case_type} | {atty}")
    
    def __eq__(self, other):
        if not isinstance(other, Client):
            return NotImplemented
        return self.name == other.name
    
    def __hash__(self):
        return hash(self.name)
    
    def __lt__(self, other):
        return self.revenue < other.revenue


class Firm:
    """The firm itself — manages attorneys and clients."""
    
    def __init__(self, name):
        self.name = name
        self.attorneys = []
        self.clients = []
    
    def hire_attorney(self, attorney):
        self.attorneys.append(attorney)
        return attorney
    
    def onboard_client(self, client):
        self.clients.append(client)
        self._auto_assign(client)
        return client
    
    def _auto_assign(self, client):
        """Automatically assign an attorney based on case type and value."""
        rules = {
            "Merger": lambda a: a.specialization == "Corporate" and client.revenue >= 500000,
            "Litigation": lambda a: a.specialization == "Litigation",
            "Patent": lambda a: a.specialization == "IP",
        }
        
        rule = rules.get(client.case_type)
        if rule:
            for attorney in self.attorneys:
                if rule(attorney):
                    attorney.assign_client(client)
                    return
        
        # Default: assign to attorney with fewest clients
        if self.attorneys:
            least_busy = min(self.attorneys, key=lambda a: len(a.clients))
            least_busy.assign_client(client)
    
    def total_revenue(self):
        return sum(c.revenue for c in self.clients if c.active)
    
    def total_billed(self):
        return sum(a.total_billed for a in self.attorneys)
    
    def dashboard(self):
        """Print the firm dashboard."""
        print(f"\n{'='*70}")
        print(f"{'🏛️  ' + self.name + '  🏛️':^70}")
        print(f"{'='*70}")
        
        # Attorney roster
        print(f"\n📋 ATTORNEY ROSTER ({len(self.attorneys)})")
        print(f"{'─'*70}")
        for atty in sorted(self.attorneys, key=lambda a: a.total_billed, reverse=True):
            print(f"  {atty}")
        
        # Client roster
        print(f"\n📋 CLIENT ROSTER ({len(self.clients)})")
        print(f"{'─'*70}")
        for client in sorted(self.clients, reverse=True):   # Uses __lt__
            print(f"  {client}")
        
        # Summary stats
        active = [c for c in self.clients if c.active]
        tiers = {}
        for c in active:
            tier, _ = c.classify()
            tiers[tier] = tiers.get(tier, 0) + 1
        
        print(f"\n📊 FIRM SUMMARY")
        print(f"{'─'*70}")
        print(f"  Attorneys:    {len(self.attorneys)}")
        print(f"  Clients:      {len(active)} active / {len(self.clients)} total")
        print(f"  Revenue:      ${self.total_revenue():>14,.2f}")
        print(f"  Billed:       ${self.total_billed():>14,.2f}")
        print(f"\n  Client Tiers:")
        for tier in ["Platinum", "Gold", "Silver", "Bronze"]:
            count = tiers.get(tier, 0)
            bar = "█" * count + "░" * (10 - count)
            print(f"    {tier:<10} [{bar}] {count}")
        
        print(f"\n{'='*70}")


# === BUILD THE FIRM ===
firm = Firm("Pearson Specter Litt")

# Hire attorneys
harvey = firm.hire_attorney(
    Attorney("Harvey Specter", "harvey@psl.com", "Senior Partner", 450, "Corporate")
)
louis = firm.hire_attorney(
    Attorney("Louis Litt", "louis@psl.com", "Name Partner", 425, "Litigation")
)
mike = firm.hire_attorney(
    Attorney("Mike Ross", "mike@psl.com", "Associate", 375, "IP")
)

# Onboard clients (auto-assigned)
clients_data = [
    ("Wayne Enterprises", "wayne@we.com", 2500000, "Merger"),
    ("Stark Industries", "contact@stark.com", 800000, "Patent"),
    ("Oscorp", "info@oscorp.com", 600000, "Litigation"),
    ("Dalton Industries", "dalton@di.com", 1200000, "Merger"),
    ("Luthor Corp", "lex@luthor.com", 150000, "Corporate"),
    ("Queen Industries", "oliver@qi.com", 3000000, "Merger"),
    ("Palmer Tech", "ray@palmer.com", 400000, "Patent"),
]

for name, email, rev, case_type in clients_data:
    firm.onboard_client(Client(name, email, rev, case_type))

# Simulate billing
harvey.bill(harvey.clients[0], 85) if harvey.clients else None
harvey.bill(harvey.clients[1], 42) if len(harvey.clients) > 1 else None
louis.bill(louis.clients[0], 65) if louis.clients else None
mike.bill(mike.clients[0], 52) if mike.clients else None

# Display the complete dashboard
firm.dashboard()
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "Object management is where professional code separates from amateur code."

---

### Pitfall 1: Identity vs Equality (Revisited at Object Level) 🔥

```python
class Client:
    def __init__(self, name):
        self.name = name

c1 = Client("Wayne")
c2 = Client("Wayne")

# Without __eq__, Python compares IDENTITY (memory address)
print(c1 == c2)     # False — same data, different objects!
print(c1 is c2)     # False — definitely different objects

# After defining __eq__:
# class Client:
#     def __eq__(self, other):
#         return self.name == other.name
# Now c1 == c2 → True
```

**Louis:** "If you need to check equality by value, you MUST define `__eq__`. And if you define `__eq__`, you SHOULD also define `__hash__` — or your objects can't be used in sets or as dict keys."

---

### Pitfall 2: Shallow Copy Surprises

```python
import copy

class Team:
    def __init__(self, name):
        self.name = name
        self.members = []

team1 = Team("Alpha")
team1.members.append("Harvey")

team2 = copy.copy(team1)        # Shallow copy!
team2.members.append("Mike")

print(team1.members)   # ['Harvey', 'Mike'] ← SURPRISE!
# team1 and team2 share the SAME members list

# ✅ Fix: use deepcopy
team3 = copy.deepcopy(team1)
team3.members.append("Louis")
print(team1.members)   # ['Harvey', 'Mike'] — unchanged ✅
```

---

### Pitfall 3: Modifying Objects Passed to Functions

```python
def reset_billing(client):
    client.revenue = 0    # ⚠️ This changes the ORIGINAL object!

wayne = Client("Wayne")
wayne.revenue = 2500000
reset_billing(wayne)
print(wayne.revenue)     # 0 — changed!

# If you DON'T want to modify the original, work on a copy:
import copy

def reset_billing_safe(client):
    new_client = copy.deepcopy(client)
    new_client.revenue = 0
    return new_client     # Return the copy, leave original alone
```

---

### Pitfall 4: Circular References

```python
class Attorney:
    def __init__(self, name):
        self.name = name
        self.clients = []

class Client:
    def __init__(self, name):
        self.name = name
        self.attorney = None

# Circular reference: attorney → client → attorney → client → ...
harvey = Attorney("Harvey")
wayne = Client("Wayne")

harvey.clients.append(wayne)
wayne.attorney = harvey

# This is fine in Python (garbage collector handles it)
# But be careful with __repr__ — can cause infinite recursion!

# ❌ BAD
# class Client:
#     def __repr__(self):
#         return f"Client({self.name}, attorney={self.attorney})"
# print(wayne) → prints attorney → which prints clients → which prints attorney → INFINITE!

# ✅ GOOD — only show the attorney's name, not the full object
class Client:
    def __repr__(self):
        atty_name = self.attorney.name if self.attorney else "None"
        return f"Client({self.name}, attorney={atty_name})"
```

---

### Pitfall 5: Forgetting `super().__init__()` in Inheritance

```python
class Person:
    def __init__(self, name):
        self.name = name

class Attorney(Person):
    def __init__(self, name, rate):
        # ❌ Forgot to call parent's __init__!
        self.rate = rate

a = Attorney("Harvey", 450)
# print(a.name)   # AttributeError: 'Attorney' has no attribute 'name'

# ✅ Fix: call super().__init__()
class Attorney(Person):
    def __init__(self, name, rate):
        super().__init__(name)    # ✅ Initialize parent class
        self.rate = rate
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "Building from blueprints."

---

### 🏭 Analogy: Instantiation Is Like a Production Line

**Donna:** "You designed the blueprint in Lesson 12. Now you're running the factory."

```
Blueprint (Class):
  ┌──────────────┐
  │  Client      │
  │  - name      │
  │  - revenue   │
  │  - classify()│
  └──────────────┘

Factory (Instantiation):
  Blueprint → 🏭 → 📦 Wayne ($2.5M, Platinum)
  Blueprint → 🏭 → 📦 Stark ($800K, Gold)
  Blueprint → 🏭 → 📦 Oscorp ($600K, Gold)

Warehouse (Collection):
  📦📦📦📦📦📦📦 → clients = [wayne, stark, oscorp, ...]
  
Sorting & Filtering:
  📦📦📦 → sorted(clients, key=lambda c: c.revenue)
  📦📦   → [c for c in clients if c.classify() == "Gold"]
```

**Donna:** "And when two objects interact?"

```
Attorney Harvey ←──→ Client Wayne
    │                      │
    │  harvey.take(wayne)  │
    │  wayne.attorney = harvey
    │                      │
    └── Both objects now know about each other
```

**Donna:** "Objects aren't isolated. They connect, reference each other, and form networks — just like people in a law firm."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "From blueprints to production."

---

### 🟢 Beginner Exercises

**Exercise 1: Student Roster**
```
Create a Student class with: name, grade, gpa
- Create 5 students
- Store in a list
- Find the student with the highest GPA
- Print all students sorted by GPA (descending)
```

**Exercise 2: Shopping Cart**
```
Create Product(name, price) and Cart() classes.
Cart has:
  - add_item(product, quantity)
  - remove_item(product_name)
  - total(): sum of price × quantity
  - display(): formatted list

Create 5 products, add them to a cart, and display the total.
```

**Exercise 3: Contact Book**
```
Create Contact(name, phone, email) class.
Create a ContactBook class that can:
  - add_contact(contact)
  - find_by_name(name) → returns Contact or None
  - delete_contact(name)
  - list_all() → prints sorted alphabetically
  
Add 5 contacts, search for one, delete one, list remaining.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Playlist Manager**
```
Create Song(title, artist, duration_seconds) class.
Create Playlist(name) class:
  - add_song(song)
  - remove_song(title)
  - total_duration() → formatted as "X mins Y secs"
  - shuffle() → randomize song order
  - sort_by(field) → sort by title, artist, or duration
  - display() → formatted with track numbers

Create 2 playlists with 5+ songs each. Transfer songs between them.
```

**Exercise 5: Object Comparison Exercise**
```
Create a Movie(title, year, rating) class.
Implement:
  - __eq__: equal if same title AND year
  - __lt__: compare by rating
  - __str__: "Title (Year) - ★ Rating"
  - __hash__: based on title and year

Create 10 movies, put them in a list:
  - Sort by rating
  - Find duplicates using a set
  - Create a dict mapping title to Movie object
  - Find the top 3 rated movies
```

**Exercise 6: Factory Pattern Practice**
```
Create a Vehicle class with: make, model, year, price, category
Add factory classmethods:
  - Vehicle.sedan(make, model, year) → price=25000
  - Vehicle.suv(make, model, year) → price=35000
  - Vehicle.sports(make, model, year) → price=50000
  - Vehicle.from_string("Toyota,Camry,2024") → parsed

Create 5 vehicles using different factory methods.
Sort by price and display.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Social Network**
```
Create User(username, name) class with:
  - follow(other_user)
  - unfollow(other_user)
  - followers: set of users who follow this user
  - following: set of users this user follows
  - mutual_friends(other_user): users both follow
  - suggest_follows(): friends of friends not yet followed

Create 6+ users with various follow relationships.
Print each user's followers, following, and mutual friends.
```

**Exercise 8: Event System**
```
Create Event(name, date, capacity) class.
Create Attendee(name, email) class.
Create EventManager class with:
  - create_event(name, date, capacity)
  - register(attendee, event_name) → success or full
  - cancel_registration(attendee_name, event_name)
  - waitlist(attendee, event_name) → add to waitlist if full
  - event_report(event_name) → attendees, waitlist, spots left
  - attendee_events(name) → list of events for an attendee

Handle: full events, duplicate registrations, promoting from waitlist.
```

**Exercise 9: Full Firm Simulation**
```
Build on the firm roster system to add:
  - Monthly billing cycles (iterate months, bill random hours)
  - Performance reviews (utilization rates per attorney)
  - Client retention (deactivate clients below a threshold)
  - New client pipeline (add random clients each month)
  - Quarterly report (aggregate 3 months of data)

Run a 12-month simulation and print quarterly + annual reports.
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Instantiation bugs."

### Buggy Code 🐛

```python
# Bug Hunt: Object Management
# Find and fix ALL the bugs!

class Employee:
    all_employees = []
    
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
        self.all_employees.append(self)

# Bug 1: Shared class attribute causes unexpected behavior
e1 = Employee("Harvey", 500000)
e2 = Employee("Mike", 150000)
print(f"Employee 1 list: {len(e1.all_employees)}")
print(f"Employee 2 list: {len(e2.all_employees)}")
# Are these the same list?

# Bug 2: Comparing objects without __eq__
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p1 = Point(3, 4)
p2 = Point(3, 4)
print(p1 == p2)    # Expected: True

# Bug 3: Shallow copy issue
import copy
class Inventory:
    def __init__(self):
        self.items = ["pen", "paper"]

inv1 = Inventory()
inv2 = copy.copy(inv1)
inv2.items.append("stapler")
print(f"inv1: {inv1.items}")   # Expected: ['pen', 'paper']

# Bug 4: Object passed by reference
class Counter:
    def __init__(self):
        self.value = 0

def reset(counter):
    counter = Counter()    # Does this reset the original?

c = Counter()
c.value = 42
reset(c)
print(c.value)             # Expected: 0

# Bug 5: Missing super().__init__
class Animal:
    def __init__(self, species):
        self.species = species

class Dog(Animal):
    def __init__(self, name, breed):
        self.name = name
        self.breed = breed

d = Dog("Rex", "Labrador")
print(d.species)           # Expected: "Dog"
```

---

### Fixed Code ✅

```python
# Fixed: Object Management

# Fix 1: This is intentional behavior — class attrs are shared
# If you want independent tracking, use an instance attribute or be aware
class Employee:
    all_employees = []   # This IS shared — it's a class-level registry
    
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
        Employee.all_employees.append(self)  # ✅ Use Employee., not self.

e1 = Employee("Harvey", 500000)
e2 = Employee("Mike", 150000)
print(f"Total employees: {len(Employee.all_employees)}")  # 2 ✅
# If you DON'T want shared state, use instance attributes instead

# Fix 2: Define __eq__ for value comparison
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __eq__(self, other):                    # ✅
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

p1 = Point(3, 4)
p2 = Point(3, 4)
print(p1 == p2)    # True ✅

# Fix 3: Use deepcopy for objects with mutable data
import copy
class Inventory:
    def __init__(self):
        self.items = ["pen", "paper"]

inv1 = Inventory()
inv2 = copy.deepcopy(inv1)                    # ✅ Deep copy
inv2.items.append("stapler")
print(f"inv1: {inv1.items}")   # ['pen', 'paper'] ✅

# Fix 4: Reassigning the local variable doesn't change the original
class Counter:
    def __init__(self):
        self.value = 0

def reset(counter):
    counter.value = 0     # ✅ Modify the attribute, don't reassign the variable!

c = Counter()
c.value = 42
reset(c)
print(c.value)             # 0 ✅

# Fix 5: Call super().__init__() to initialize parent class
class Animal:
    def __init__(self, species):
        self.species = species

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__("Dog")    # ✅ Initialize parent
        self.name = name
        self.breed = breed

d = Dog("Rex", "Labrador")
print(d.species)           # "Dog" ✅
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | Class attribute is shared — not a bug, but a gotcha | Class vs instance attributes |
| 2 | Objects compared by identity without `__eq__` | Define `__eq__` for value equality |
| 3 | `copy.copy` shares nested mutable data | Use `copy.deepcopy` |
| 4 | Reassigning parameter doesn't change original object | Modify attributes, don't reassign |
| 5 | Child class doesn't init parent class | Use `super().__init__()` |

---

## 8. 🌍 Real-World Application

**Harvey:** "Object management powers every enterprise system."

### Where Object Instantiation Is Used:

| Domain | Objects In Action |
|--------|------------------|
| **ORMs** | Each database row becomes an object (`user = User.get(1)`) |
| **Web Servers** | Each HTTP request is a `Request` object |
| **Game Engines** | Hundreds of `Entity` objects instantiated per frame |
| **Data Pipelines** | Records transformed into objects for processing |
| **Testing** | Mock objects created to simulate dependencies |
| **GUIs** | `Button`, `Window`, `Dialog` objects created and managed |

### Real Production Example: Object Pool Pattern

```python
class Connection:
    """Simulates a database connection."""
    _id_counter = 0
    
    def __init__(self):
        Connection._id_counter += 1
        self.id = Connection._id_counter
        self.in_use = False
    
    def execute(self, query):
        print(f"  [Conn-{self.id}] Executing: {query}")
    
    def __str__(self):
        status = "BUSY" if self.in_use else "FREE"
        return f"Connection-{self.id} [{status}]"


class ConnectionPool:
    """Manages a pool of reusable connection objects."""
    
    def __init__(self, size):
        self.pool = [Connection() for _ in range(size)]
    
    def acquire(self):
        """Get a free connection from the pool."""
        for conn in self.pool:
            if not conn.in_use:
                conn.in_use = True
                print(f"  ✅ Acquired {conn}")
                return conn
        print("  ❌ No connections available!")
        return None
    
    def release(self, conn):
        """Return a connection to the pool."""
        conn.in_use = False
        print(f"  🔄 Released {conn}")
    
    def status(self):
        free = sum(1 for c in self.pool if not c.in_use)
        busy = len(self.pool) - free
        print(f"  Pool: {free} free / {busy} busy / {len(self.pool)} total")

# Usage
pool = ConnectionPool(3)
pool.status()

c1 = pool.acquire()
c2 = pool.acquire()
pool.status()

c1.execute("SELECT * FROM clients")
pool.release(c1)
pool.status()
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Object quiz."

---

**Q1.** What happens when you call `Client("Wayne", 2500000)`? Describe the steps.

**Q2.** How do you store objects in a collection? Give an example with a list and a dictionary.

**Q3.** What is the difference between `==` and `is` for objects? How do you make `==` check values?

**Q4.** What is the difference between `copy.copy()` and `copy.deepcopy()`?

**Q5.** When you pass an object to a function, does the function get a copy or a reference?

**Q6.** What is `isinstance()`? When should you use it?

**Q7.** What is a factory method? What does `@classmethod` do?

**Q8.** What happens if you define `__eq__` but not `__hash__`?

**Q9.** Write code that creates 5 objects, stores them in a list, sorts by an attribute, and prints the top 3.

**Q10.** What is `super().__init__()` and why is it important?

---

### 📋 Answer Key

> **Q1.** Python creates a new empty object → calls `__init__(self, "Wayne", 2500000)` → `self` is the new object → attributes are set → initialized object is returned.

> **Q2.** List: `clients = [Client("A"), Client("B")]`. Dict: `registry = {"A": Client("A")}`.

> **Q3.** `==` checks identity by default (or value if `__eq__` is defined). `is` ALWAYS checks identity (same memory address). Define `__eq__` for value comparison.

> **Q4.** `copy.copy()` = new object but shared nested data. `copy.deepcopy()` = new object with ALL nested data copied independently.

> **Q5.** A reference. Changes to the object's attributes inside the function affect the original.

> **Q6.** `isinstance(obj, Class)` checks if an object is an instance of a class (including subclasses). Use it for type checking in functions.

> **Q7.** A factory method is an alternative constructor defined with `@classmethod`. It returns a new instance with preset configurations.

> **Q8.** The object becomes unhashable — you can't use it in sets or as dict keys. Python sets `__hash__` to `None` when `__eq__` is defined.

> **Q9.**
```python
class Student:
    def __init__(self, name, gpa):
        self.name = name
        self.gpa = gpa
students = [Student("A",3.8), Student("B",3.5), Student("C",3.9), Student("D",3.2), Student("E",3.7)]
top3 = sorted(students, key=lambda s: s.gpa, reverse=True)[:3]
for s in top3:
    print(f"{s.name}: {s.gpa}")
```

> **Q10.** `super().__init__()` calls the parent class's constructor. Without it, parent attributes won't be initialized, causing `AttributeError`.

---

## 🎬 End of Lesson Scene

*It's 6:30 PM. The firm roster system is live. Jessica watches as 12 partners, 47 associates, and 237 clients populate the dashboard — all as objects, all interconnected.*

**Jessica:** "Every person in this firm is in the system. Every client classified. Every attorney's workload visible."

**Harvey:** "One lesson left. File I/O. Once you can read and write data to files, your programs become permanent. They survive reboots. They share data with other systems."

**Mike:** "We've gone from variables to full object-oriented systems. Tomorrow we make them persistent."

**Donna:** "The system finally has memory. Tomorrow it gets a filing cabinet."

*Day thirteen. You've built an entire firm management system — 237 clients, 59 firm members, all as objects that interact, compare, sort, and display. Object-oriented programming isn't just a concept anymore. It's how you build software.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | Instantiation — creating objects from classes | ✅ |
| 2 | Storing objects in lists and dictionaries | ✅ |
| 3 | Sorting objects — `sorted()` with `key`, `__lt__` | ✅ |
| 4 | Comparing objects — `__eq__`, `__hash__` | ✅ |
| 5 | Copying objects — shallow vs deep | ✅ |
| 6 | Passing objects to/from functions | ✅ |
| 7 | Objects interacting with each other | ✅ |
| 8 | Factory methods with `@classmethod` | ✅ |
| 9 | `isinstance()` type checking | ✅ |
| 10 | Inheritance basics — `super().__init__()` | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Classes (Lesson 12) | Instantiating defined classes |
| Lists (Lesson 3) | Storing objects in lists |
| Dictionaries (Lesson 5) | Object registries, grouping |
| List comprehensions (Lesson 9) | Filtering objects |
| `sorted()` + `lambda` (Lesson 9) | Sorting objects by attribute |
| Functions (Lesson 11) | Passing/returning objects |
| `for` loops (Lesson 9) | Iterating over object collections |
| `is` vs `==` (Lesson 7) | Object identity vs equality |

---

> **Next Lesson →** Level 1, Lesson 14: *Read and Write Text Files in Python*  
> *The firm's data lives in memory — when the program stops, everything vanishes. Time to make it permanent…*

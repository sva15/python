# 🏛️ Level 1 – Exploring | Lesson 12: Define a Python Class

## *"The Blueprint"*

> **O'Reilly Skill Objective:** *Define a Python class.*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, `input()`, conditionals, boolean operators, `for`/`while` loops, functions, scope, `*args`/`**kwargs`, docstrings

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

*It's 8:00 AM, Monday. A new week. You walk in and Harvey is already at the whiteboard, drawing boxes and arrows.*

**Harvey:** "We have a problem."

**You:** "We always have a problem."

**Harvey:** *(half-smiling)* "Your billing functions from last week work great. But they float around disconnected — `calculate_billing()` here, `classify_client()` there, `assign_attorney()` over there. Nothing ties them together."

*He draws a box labeled "CLIENT" on the whiteboard.*

**Harvey:** "When I think of a client, I don't think of separate pieces. I think of ONE entity — a **Client**. It has a name, a revenue number, a case type, an assigned attorney. It can be billed, classified, and invoiced. Everything about that client belongs TOGETHER."

**Mike:** *(walking in)* "That's **object-oriented programming**. Instead of free-floating functions and loose dictionaries, you define a **class** — a **blueprint** — that packages data AND behavior into a single, organized unit."

**Harvey:** "A class called `Client` that knows its name, tracks its revenue, calculates its own billing, and classifies itself. One blueprint. Unlimited clients."

**Mike:** "Today you learn to build blueprints."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Here's the evolution you've been through:"

| Level | Approach | Problem |
|-------|----------|---------|
| **Lesson 1** | Variables | Data scattered everywhere |
| **Lesson 5** | Dictionaries | Data grouped, but no behavior |
| **Lesson 11** | Functions | Behavior defined, but data and behavior separate |
| **Lesson 12** | **Classes** | Data AND behavior bundled together |

**Harvey:** "Think about our `Client` dictionary:"

```python
# Dictionary approach — data only, no behavior
client = {
    "name": "Wayne Enterprises",
    "revenue": 2500000,
    "rate": 450.00,
    "case_type": "Merger"
}

# Functions floating separately
billing = calculate_billing(client["hours"], client["rate"])
tier = classify_client(client["revenue"])
```

**Harvey:** "Now imagine instead:"

```python
# Class approach — data AND behavior, together
client = Client("Wayne Enterprises", revenue=2500000, rate=450.00)
client.calculate_billing(hours=85)    # The client knows its own rate
client.classify()                      # The client classifies itself
client.generate_invoice()              # The client generates its own invoice
```

**Harvey:** "The data knows what it can DO. That's the power of classes."

---

## 3. 🔧 Technical Breakdown — Mike Ross

**Mike:** "Classes and objects. The foundation of modern software design."

---

### 3.1 Your First Class

```python
class Client:
    pass       # Empty class — a placeholder

# Create an instance (an object from the blueprint)
c = Client()
print(type(c))   # <class '__main__.Client'>
print(c)         # <__main__.Client object at 0x...>
```

**Mike:** "Two key concepts:"
- **Class** = The blueprint (defines WHAT a client is)
- **Object / Instance** = An actual client created FROM the blueprint

```
Class (Blueprint):          Object (Instance):
┌─────────────────┐        ┌─────────────────────────────┐
│     Client      │   →    │  Wayne Enterprises          │
│  - name         │   →    │  name = "Wayne Enterprises" │
│  - revenue      │   →    │  revenue = 2500000          │
│  - classify()   │   →    │  classify() → "Gold"        │
└─────────────────┘        └─────────────────────────────┘
                           ┌─────────────────────────────┐
                      →    │  Stark Industries           │
                      →    │  name = "Stark Industries"  │
                      →    │  revenue = 800000           │
                      →    │  classify() → "Gold"        │
                           └─────────────────────────────┘
```

---

### 3.2 The `__init__` Method (Constructor)

**Mike:** "The `__init__` method runs automatically when you create an object. It sets up the initial data."

```python
class Client:
    def __init__(self, name, revenue, case_type):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type

# Create objects — __init__ runs automatically
client1 = Client("Wayne Enterprises", 2500000, "Merger")
client2 = Client("Stark Industries", 800000, "Patent")

# Access data via dot notation
print(client1.name)        # Wayne Enterprises
print(client2.revenue)     # 800000
print(client1.case_type)   # Merger
```

**Mike:** "Let me break down the magic:"

```python
class Client:
    def __init__(self, name, revenue, case_type):
    #           ^^^^  ^^^^  ^^^^^^^  ^^^^^^^^^
    #           |     |     |        |
    #           |     |     └────────┴── Regular parameters (you pass these)
    #           |     └── The instance being created (Python passes this automatically)
    #           └── Always the first parameter of every method
        
        self.name = name           # Attach 'name' to THIS object
        self.revenue = revenue     # Attach 'revenue' to THIS object
        self.case_type = case_type # Attach 'case_type' to THIS object
```

| Term | Meaning |
|------|---------|
| `self` | Reference to the current object being created/used |
| `self.name` | An **instance attribute** — data that belongs to THIS specific object |
| `__init__` | The **constructor** — called automatically when creating an object |

> ⚠️ `self` is NOT optional. Every method in a class must have `self` as its first parameter. Python passes it automatically — you never pass it when calling.

---

### 3.3 Adding Methods (Behavior)

**Mike:** "Methods are functions that belong to a class. They define what objects can DO."

```python
class Client:
    def __init__(self, name, revenue, rate, case_type):
        self.name = name
        self.revenue = revenue
        self.rate = rate
        self.case_type = case_type
    
    def classify(self):
        """Classify the client into a billing tier."""
        if self.revenue >= 1000000:
            return "💎 Platinum"
        elif self.revenue >= 500000:
            return "🥇 Gold"
        elif self.revenue >= 100000:
            return "🥈 Silver"
        else:
            return "🥉 Bronze"
    
    def calculate_billing(self, hours, tax_rate=0.09):
        """Calculate billing for given hours."""
        subtotal = hours * self.rate
        tax = subtotal * tax_rate
        return subtotal + tax
    
    def summary(self):
        """Return a formatted summary string."""
        tier = self.classify()   # Methods can call other methods!
        return f"{self.name} | {tier} | ${self.revenue:,.2f} | {self.case_type}"

# Create and use
client = Client("Wayne Enterprises", 2500000, 450.00, "Merger")

print(client.classify())                    # 💎 Platinum
print(client.calculate_billing(85))         # 41647.5
print(client.summary())                     # Wayne Enterprises | 💎 Platinum | $2,500,000.00 | Merger
```

**Mike:** "Notice: `self.rate` lets the `calculate_billing` method access the rate that was set in `__init__`. The object **remembers** its own data."

---

### 3.4 Instance Attributes vs Class Attributes

```python
class Client:
    # CLASS ATTRIBUTE — shared by ALL instances
    firm_name = "Pearson Specter Litt"
    tax_rate = 0.09
    client_count = 0
    
    def __init__(self, name, revenue):
        # INSTANCE ATTRIBUTES — unique to each object
        self.name = name
        self.revenue = revenue
        Client.client_count += 1    # Modify the class attribute
    
    def display(self):
        # Can access both instance AND class attributes
        print(f"{self.name} — {Client.firm_name}")

# Usage
c1 = Client("Wayne", 2500000)
c2 = Client("Stark", 800000)

print(Client.firm_name)     # Pearson Specter Litt (accessed via class)
print(c1.firm_name)         # Pearson Specter Litt (also accessible via instance)
print(Client.client_count)  # 2 (shared across all instances)
print(c1.name)              # Wayne (unique to c1)
print(c2.name)              # Stark (unique to c2)
```

| Feature | Class Attribute | Instance Attribute |
|---------|----------------|-------------------|
| Defined | In the class body (outside `__init__`) | Inside `__init__` with `self.` |
| Shared | Yes — all instances share it | No — unique to each instance |
| Access | `ClassName.attr` or `instance.attr` | `instance.attr` only |
| Use case | Constants, counters, defaults | Object-specific data |

---

### 3.5 The `__str__` and `__repr__` Methods

**Mike:** "Control how your objects display as strings."

```python
class Client:
    def __init__(self, name, revenue, case_type):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
    
    def __str__(self):
        """Human-readable string — used by print() and str()."""
        return f"{self.name} (${self.revenue:,.2f} — {self.case_type})"
    
    def __repr__(self):
        """Developer-readable string — used in the REPL and debugging."""
        return f"Client('{self.name}', {self.revenue}, '{self.case_type}')"

client = Client("Wayne Enterprises", 2500000, "Merger")

print(client)          # Wayne Enterprises ($2,500,000.00 — Merger)  ← __str__
print(repr(client))    # Client('Wayne Enterprises', 2500000, 'Merger')  ← __repr__

# Without __str__, print shows: <__main__.Client object at 0x...>
# Much less useful!
```

---

### 3.6 Encapsulation — Controlling Access

**Mike:** "Python uses naming conventions (not strict enforcement) to signal access levels:"

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner          # Public — anyone can access
        self._account_id = "ACC-001"  # Protected — convention: "internal use"
        self.__balance = balance      # Private — name mangled by Python
    
    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount
            return True
        return False
    
    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            return True
        return False
    
    def get_balance(self):
        """Public method to access private data."""
        return self.__balance

# Usage
account = BankAccount("Harvey", 50000)

print(account.owner)           # Harvey ✅ Public
print(account._account_id)    # ACC-001 ✅ Works, but convention says don't
# print(account.__balance)    # ❌ AttributeError — name mangled!
print(account.get_balance())  # 50000 ✅ Access through method
```

| Prefix | Convention | Access |
|--------|-----------|--------|
| `name` | Public | Anyone can read/write |
| `_name` | Protected | "Internal use" — can access, but shouldn't |
| `__name` | Private | Name-mangled — hard to access from outside |

---

### 3.7 Properties — Pythonic Getters/Setters

```python
class Attorney:
    def __init__(self, name, hourly_rate):
        self.name = name
        self._hourly_rate = hourly_rate
    
    @property
    def hourly_rate(self):
        """Get the hourly rate."""
        return self._hourly_rate
    
    @hourly_rate.setter
    def hourly_rate(self, value):
        """Set hourly rate with validation."""
        if value < 0:
            raise ValueError("Rate cannot be negative!")
        if value > 1000:
            raise ValueError("Rate exceeds maximum ($1000)!")
        self._hourly_rate = value
    
    @property
    def daily_rate(self):
        """Calculated property — no setter needed."""
        return self._hourly_rate * 8

# Usage — looks like attribute access, but runs validation!
attorney = Attorney("Harvey", 450)

print(attorney.hourly_rate)   # 450 (calls the getter)
print(attorney.daily_rate)    # 3600 (calculated property)

attorney.hourly_rate = 500    # Calls the setter (with validation)
print(attorney.hourly_rate)   # 500

# attorney.hourly_rate = -50  # ValueError: Rate cannot be negative!
```

---

### 3.8 Methods Calling Other Methods

```python
class CaseManager:
    def __init__(self):
        self.cases = []
    
    def add_case(self, case_id, client, case_type, value):
        """Add a new case."""
        case = {
            "id": case_id,
            "client": client,
            "type": case_type,
            "value": value,
            "status": "Active"
        }
        self.cases.append(case)
        return case
    
    def find_case(self, case_id):
        """Find a case by ID."""
        for case in self.cases:
            if case["id"] == case_id:
                return case
        return None
    
    def close_case(self, case_id):
        """Close a case by ID — uses find_case internally."""
        case = self.find_case(case_id)   # Calling another method!
        if case:
            case["status"] = "Closed"
            return True
        return False
    
    def get_active_cases(self):
        """Get all active cases — uses a loop with a conditional."""
        return [c for c in self.cases if c["status"] == "Active"]
    
    def total_value(self):
        """Calculate total value of all active cases."""
        return sum(c["value"] for c in self.get_active_cases())

# Usage
manager = CaseManager()
manager.add_case("PSL-001", "Wayne", "Merger", 2500000)
manager.add_case("PSL-002", "Stark", "Patent", 800000)
manager.add_case("PSL-003", "Oscorp", "Litigation", 600000)

print(f"Active cases: {len(manager.get_active_cases())}")
print(f"Total value: ${manager.total_value():,.2f}")

manager.close_case("PSL-002")
print(f"Active after closing: {len(manager.get_active_cases())}")
print(f"Total value: ${manager.total_value():,.2f}")
```

---

### 3.9 Solving Harvey's Problem — The Client Object

```python
# ============================================
# Pearson Specter Litt — Client Class
# ============================================

class Client:
    """Represents a client of the firm."""
    
    firm_name = "Pearson Specter Litt"
    tax_rate = 0.09
    _next_id = 1
    
    def __init__(self, name, revenue, rate, case_type, is_referral=False):
        self.client_id = f"PSL-{Client._next_id:04d}"
        Client._next_id += 1
        
        self.name = name
        self.revenue = revenue
        self.rate = rate
        self.case_type = case_type
        self.is_referral = is_referral
        self.attorney = None
        self.billing_history = []
    
    def classify(self):
        """Returns the billing tier."""
        if self.revenue >= 1000000:
            return "Platinum", "💎"
        elif self.revenue >= 500000:
            return "Gold", "🥇"
        elif self.revenue >= 100000:
            return "Silver", "🥈"
        else:
            return "Bronze", "🥉"
    
    def assign_attorney(self):
        """Assign the appropriate attorney."""
        if self.case_type == "Merger" and self.revenue >= 500000:
            self.attorney = "Harvey Specter"
        elif self.case_type == "Litigation":
            self.attorney = "Louis Litt"
        elif self.case_type == "Patent":
            self.attorney = "Mike Ross"
        else:
            self.attorney = "Associate Pool"
        return self.attorney
    
    def bill(self, hours):
        """Calculate and record billing."""
        subtotal = hours * self.rate
        tax = subtotal * Client.tax_rate
        total = subtotal + tax
        
        record = {
            "hours": hours,
            "subtotal": subtotal,
            "tax": tax,
            "total": total
        }
        self.billing_history.append(record)
        return record
    
    def total_billed(self):
        """Sum of all billing history."""
        return sum(b["total"] for b in self.billing_history)
    
    def profile_card(self):
        """Print a formatted profile card."""
        tier, icon = self.classify()
        if not self.attorney:
            self.assign_attorney()
        
        print(f"\n  ┌{'─'*44}┐")
        print(f"  │{self.client_id:^44}│")
        print(f"  ├{'─'*44}┤")
        print(f"  │  Name:      {self.name:<30}│")
        print(f"  │  Revenue:   ${self.revenue:<29,.2f}│")
        print(f"  │  Rate:      ${self.rate:<29,.2f}│")
        print(f"  │  Type:      {self.case_type:<30}│")
        print(f"  │  Tier:      {icon} {tier:<27}│")
        print(f"  │  Attorney:  {self.attorney:<30}│")
        print(f"  │  Billed:    ${self.total_billed():<29,.2f}│")
        print(f"  │  Referral:  {'Yes' if self.is_referral else 'No':<30}│")
        print(f"  └{'─'*44}┘")
    
    def __str__(self):
        tier, icon = self.classify()
        return f"{icon} {self.name} ({self.client_id}) — {tier} — ${self.revenue:,.2f}"
    
    def __repr__(self):
        return (f"Client('{self.name}', {self.revenue}, {self.rate}, "
                f"'{self.case_type}', is_referral={self.is_referral})")


# === Demonstrate the Client class ===
print("=" * 50)
print(f"{'CLIENT MANAGEMENT SYSTEM':^50}")
print(f"{Client.firm_name:^50}")
print("=" * 50)

# Create clients
clients = [
    Client("Wayne Enterprises", 2500000, 450.00, "Merger"),
    Client("Stark Industries", 800000, 375.00, "Patent"),
    Client("Oscorp", 600000, 400.00, "Litigation"),
    Client("Dalton Industries", 150000, 325.00, "Corporate"),
    Client("Gillis Industries", 75000, 280.00, "Merger", is_referral=True),
]

# Process all clients
for client in clients:
    client.assign_attorney()
    client.bill(85)   # Bill each for 85 hours

# Display
for client in clients:
    print(client)     # Uses __str__

# Detailed profile
clients[0].profile_card()

# Add more billing
clients[0].bill(42)
clients[0].bill(63)

print(f"\n{clients[0].name} — Total Billed: ${clients[0].total_billed():,.2f}")
print(f"Billing records: {len(clients[0].billing_history)}")

# Class-level data
print(f"\nTotal clients created: {Client._next_id - 1}")
print(f"Current tax rate: {Client.tax_rate * 100}%")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

**Louis:** "Classes introduce a whole new category of bugs."

---

### Pitfall 1: Forgetting `self` 🔥

```python
class Client:
    def __init__(self, name):
        name = name     # ❌ This is a LOCAL variable — not attached to the object!

c = Client("Harvey")
# print(c.name)       # ❌ AttributeError: 'Client' has no attribute 'name'

# ✅ Fix: use self.name
class Client:
    def __init__(self, name):
        self.name = name  # ✅ Attached to the object
```

---

### Pitfall 2: Forgetting `self` in Method Parameters

```python
class Client:
    def __init__(self, name):
        self.name = name
    
    # ❌ Missing 'self' parameter
    def greet():
        print(f"Hello, {self.name}")

c = Client("Harvey")
# c.greet()  # TypeError: greet() takes 0 positional arguments but 1 was given

# ✅ Fix: add self
class Client:
    def __init__(self, name):
        self.name = name
    
    def greet(self):          # ✅
        print(f"Hello, {self.name}")
```

---

### Pitfall 3: Mutable Class Attributes (Shared State!) 🔥

```python
class Client:
    # ❌ DANGER — mutable class attribute shared by ALL instances!
    cases = []
    
    def __init__(self, name):
        self.name = name
    
    def add_case(self, case):
        self.cases.append(case)

c1 = Client("Harvey")
c2 = Client("Mike")

c1.add_case("Case A")
print(c2.cases)    # ['Case A'] ← WHAT? Mike has Harvey's case!
# Both share the SAME list!

# ✅ Fix: initialize mutable data in __init__
class Client:
    def __init__(self, name):
        self.name = name
        self.cases = []    # ✅ Each instance gets its own list

c1 = Client("Harvey")
c2 = Client("Mike")
c1.add_case("Case A")
print(c2.cases)    # [] ✅ Independent
```

---

### Pitfall 4: `__init__` Should NOT Return a Value

```python
class Client:
    def __init__(self, name):
        self.name = name
        return self.name     # ❌ TypeError: __init__() should return None

# __init__ always returns None. It modifies 'self' in place.
```

---

### Pitfall 5: Confusing Class and Instance Attributes

```python
class Counter:
    count = 0    # Class attribute
    
    def __init__(self):
        self.count = 0    # Instance attribute — SHADOWS the class attribute!
    
    def increment(self):
        self.count += 1    # Modifies the INSTANCE attribute

c = Counter()
c.increment()
print(c.count)         # 1 (instance attribute)
print(Counter.count)   # 0 (class attribute — untouched!)
```

---

### Pitfall 6: Printing Objects Without `__str__`

```python
class Client:
    def __init__(self, name):
        self.name = name

c = Client("Harvey")
print(c)   # <__main__.Client object at 0x7f...> ← Not useful!

# ✅ Always define __str__ for your classes
```

---

## 5. 🧠 Mental Model — Donna Paulsen

**Donna:** "I have the perfect analogy."

---

### 🏗️ Analogy: Classes Are Architectural Blueprints

**Donna:** "Think of building a house."

```
BLUEPRINT (Class):
  ┌────────────────────────┐
  │  House Blueprint       │
  │                        │
  │  Has: bedrooms, color  │  ← Attributes (data)
  │  Can: lock, heat, cool │  ← Methods (behavior)
  └────────────────────────┘
         │
         ├──→ 🏠 House 1: 3 bedrooms, blue     ← Instance (object)
         ├──→ 🏠 House 2: 5 bedrooms, white    ← Instance (object)
         └──→ 🏠 House 3: 2 bedrooms, red      ← Instance (object)
```

- **Class** = The blueprint drawn by the architect (defines the structure)
- **Object** = An actual house built from that blueprint (has specific values)
- **`__init__`** = The construction process (builds EACH house with its specific details)
- **`self`** = "This particular house" (not the blueprint, not another house — THIS one)
- **Methods** = Things the house can DO (lock the doors, turn on the heat)
- **Attributes** = Things the house HAS (number of bedrooms, color)

**Donna:** "You draw the blueprint once. You build as many houses as you want. Each house has its own bedrooms and color, but they all follow the same structure."

---

## 6. 💪 Practice Exercises — Rachel Zane

**Rachel:** "Classes — where you design real systems."

---

### 🟢 Beginner Exercises

**Exercise 1: Person Class**
```
Create a class 'Person' with:
  - __init__: name, age
  - greet(): prints "Hi, I'm {name} and I'm {age} years old"
  - is_adult(): returns True if age >= 18
  - __str__: returns "Person(name, age)"

Create at least 3 instances and test all methods.
```

**Exercise 2: Rectangle Class**
```
Create a class 'Rectangle' with:
  - __init__: width, height
  - area(): returns width × height
  - perimeter(): returns 2 × (width + height)
  - is_square(): returns True if width == height
  - __str__: returns "Rectangle(w×h)"

Test with several rectangles, including a square.
```

**Exercise 3: Temperature Class**
```
Create a class 'Temperature' with:
  - __init__: value, scale ("C" or "F")
  - to_celsius(): returns value in Celsius
  - to_fahrenheit(): returns value in Fahrenheit
  - __str__: returns "72°F" or "22°C"

Test conversions between Celsius and Fahrenheit.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: BankAccount Class**
```
Create a class 'BankAccount' with:
  - __init__: owner, balance (default 0)
  - deposit(amount): add money (validate > 0)
  - withdraw(amount): remove money (validate sufficient funds)
  - transfer(other_account, amount): transfer to another account
  - get_balance(): return current balance
  - __str__: shows owner and balance

Track transaction history as a list of tuples:
  ("deposit", 500, "2026-03-13")
  ("withdraw", 200, "2026-03-13")
```

**Exercise 5: TodoList Class**
```
Create a class 'TodoList' with:
  - __init__: name (list name), tasks=[]
  - add_task(description, priority="medium")
  - complete_task(index): mark as done
  - remove_task(index)
  - get_pending(): return uncompleted tasks
  - get_completed(): return completed tasks
  - display(): formatted display of all tasks

Each task is a dict: {"desc": ..., "priority": ..., "done": False}
```

**Exercise 6: Deck of Cards**
```
Create two classes:

Card:
  - __init__: rank, suit
  - __str__: "Ace of Spades"

Deck:
  - __init__: creates all 52 cards
  - shuffle()
  - deal(): removes and returns top card
  - deal_hand(n): deals n cards
  - cards_remaining(): count of remaining cards
  - __str__: "Deck: 52 cards remaining"
```

---

### 🔴 Advanced Exercises

**Exercise 7: Employee Hierarchy**
```
Create classes:

Employee:
  - __init__: name, employee_id, salary, department
  - calculate_bonus(percentage): returns bonus amount
  - promote(new_salary): updates salary, tracks history
  - __str__: formatted employee info

Manager(Employee):  ← inherits from Employee
  - __init__: adds team=[] and budget
  - add_team_member(employee)
  - remove_team_member(employee_id)
  - team_cost(): total salary of team
  - __str__: includes team size

Create at least 1 manager with 3 employees.
```

**Exercise 8: Library System**
```
Create classes:

Book:
  - title, author, isbn, available=True
  - __str__

Member:
  - name, member_id, borrowed_books=[]
  - __str__

Library:
  - books=[], members=[]
  - add_book(book)
  - register_member(member)
  - borrow_book(member_id, isbn): move book from library to member
  - return_book(member_id, isbn): move book back
  - search(query): search by title or author
  - available_books(): list all available
  - member_books(member_id): list member's books

Include proper validation (book exists, available, member registered).
```

**Exercise 9: Complete Attorney Management System**
```
Build the full firm management system using classes:

Attorney:
  - name, title, hourly_rate, specialization
  - cases=[] (list of Case objects)
  - total_billing()
  - utilization_rate(target_hours)
  - __str__

Case:
  - case_id, client_name, case_type, value, status
  - hours_logged=0
  - billing()
  - close()
  - __str__

Firm:
  - name, attorneys=[], cases=[]
  - hire_attorney(attorney)
  - new_case(case) → auto-assigns based on rules
  - total_revenue()
  - utilization_report()
  - generate_dashboard()
```

---

## 7. 🐛 Debugging Scenario

**Mike:** "Class bugs."

### Buggy Code 🐛

```python
# Bug Hunt: Client Management Class
# Find and fix ALL the bugs!

# Bug 1: Missing self
class Client:
    def __init__(self, name, revenue):
        name = name
        revenue = revenue
    
    def display(self):
        print(f"Client: {self.name}")

c = Client("Wayne", 2500000)
c.display()

# Bug 2: Missing self in method
class Calculator:
    def __init__(self, value):
        self.value = value
    
    def double():
        return self.value * 2

calc = Calculator(21)
print(calc.double())

# Bug 3: Mutable class attribute
class Team:
    members = []
    
    def __init__(self, name):
        self.name = name
    
    def add_member(self, member):
        self.members.append(member)

team_a = Team("Legal")
team_b = Team("Finance")
team_a.add_member("Harvey")
print(f"Finance team: {team_b.members}")

# Bug 4: __init__ return
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
        return self.price

p = Product("Widget", 29.99)

# Bug 5: Calling class instead of creating instance
class Greeter:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        return f"Hello, {self.name}!"

# Trying to use without creating an instance
print(Greeter.greet())
```

---

### Fixed Code ✅

```python
# Fixed: Client Management Class

# Fix 1: Use self. to attach attributes to the object
class Client:
    def __init__(self, name, revenue):
        self.name = name           # ✅ self.name
        self.revenue = revenue     # ✅ self.revenue
    
    def display(self):
        print(f"Client: {self.name}")

c = Client("Wayne", 2500000)
c.display()                        # Client: Wayne ✅

# Fix 2: Add 'self' parameter to method
class Calculator:
    def __init__(self, value):
        self.value = value
    
    def double(self):              # ✅ Added self
        return self.value * 2

calc = Calculator(21)
print(calc.double())               # 42 ✅

# Fix 3: Move mutable attribute to __init__
class Team:
    def __init__(self, name):
        self.name = name
        self.members = []          # ✅ Instance attribute

    def add_member(self, member):
        self.members.append(member)

team_a = Team("Legal")
team_b = Team("Finance")
team_a.add_member("Harvey")
print(f"Finance team: {team_b.members}")  # [] ✅ Independent

# Fix 4: __init__ should NOT return a value
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
        # Don't return anything!     # ✅ Removed return

p = Product("Widget", 29.99)

# Fix 5: Create an INSTANCE first, then call methods on it
class Greeter:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        return f"Hello, {self.name}!"

g = Greeter("Harvey")              # ✅ Create instance
print(g.greet())                    # Hello, Harvey! ✅
```

### Bug Summary

| Bug # | Issue | Concept |
|-------|-------|---------|
| 1 | `name = name` instead of `self.name = name` | `self` for attributes |
| 2 | Method missing `self` parameter | Every method needs `self` |
| 3 | Mutable class attr shared across instances | Init mutable data in `__init__` |
| 4 | `__init__` should not return a value | Constructor returns None |
| 5 | Calling method on the class, not an instance | Must create an object first |

---

## 8. 🌍 Real-World Application

**Harvey:** "Everything in modern software is an object."

### Where Classes Are Used:

| Domain | Class Examples |
|--------|---------------|
| **Web Frameworks** | `Request`, `Response`, `User`, `Session` (Django, Flask) |
| **Databases** | `Model`, `QuerySet`, `Connection` (ORMs like SQLAlchemy) |
| **APIs** | `Client`, `Resource`, `Error` (API wrappers) |
| **Games** | `Player`, `Enemy`, `Weapon`, `Level` (game entities) |
| **Data Science** | `DataFrame`, `Model`, `Pipeline` (pandas, scikit-learn) |
| **GUI Apps** | `Window`, `Button`, `TextField` (Tkinter, PyQt) |
| **DevOps** | `Server`, `Container`, `Deployment` (infrastructure) |

### Real Production Example: API Client

```python
class APIClient:
    """A reusable API client class."""
    
    def __init__(self, base_url, api_key, timeout=30):
        self.base_url = base_url
        self.api_key = api_key
        self.timeout = timeout
        self._request_count = 0
    
    def _build_url(self, endpoint):
        return f"{self.base_url}/{endpoint.lstrip('/')}"
    
    def get(self, endpoint):
        """Simulate a GET request."""
        url = self._build_url(endpoint)
        self._request_count += 1
        print(f"  GET {url} (key={self.api_key[:8]}...)")
        return {"status": 200, "data": f"Response from {endpoint}"}
    
    def post(self, endpoint, data):
        """Simulate a POST request."""
        url = self._build_url(endpoint)
        self._request_count += 1
        print(f"  POST {url} → {data}")
        return {"status": 201, "data": "Created"}
    
    @property
    def request_count(self):
        return self._request_count

# Usage
client = APIClient("https://api.psl.com/v1", "sk_live_abc123xyz")
client.get("/clients")
client.post("/clients", {"name": "Wayne"})
print(f"Requests made: {client.request_count}")
```

---

## 9. ✅ Knowledge Check

**Harvey:** "Class quiz."

---

**Q1.** What is the difference between a class and an object?

**Q2.** What is `self`? Why is it needed?

**Q3.** What does `__init__` do? When is it called?

**Q4.** What is the difference between an instance attribute and a class attribute?

**Q5.** Why is this dangerous?
```python
class Foo:
    items = []
```

**Q6.** What do `__str__` and `__repr__` do? When is each called?

**Q7.** What is encapsulation? What do `_name` and `__name` mean in Python?

**Q8.** What is the output?
```python
class Dog:
    count = 0
    def __init__(self, name):
        self.name = name
        Dog.count += 1

d1 = Dog("Rex")
d2 = Dog("Buddy")
print(Dog.count)
```

**Q9.** What is a property? Why use `@property` instead of direct attribute access?

**Q10.** Write a simple class `Counter` with `increment()`, `decrement()`, `reset()`, and `get_value()` methods.

---

### 📋 Answer Key

> **Q1.** A class is a blueprint/template. An object is a specific instance created from that blueprint.

> **Q2.** `self` is a reference to the current instance. It's needed so methods can access the instance's own attributes and other methods.

> **Q3.** `__init__` is the constructor — it initializes a new object's attributes. Called automatically when you create an instance with `ClassName()`.

> **Q4.** Class attributes are shared by ALL instances (defined in the class body). Instance attributes are unique to each instance (defined with `self.` in `__init__`).

> **Q5.** `items = []` is a class attribute — ALL instances share the SAME list. Appending in one instance affects all.

> **Q6.** `__str__` = human-readable string, used by `print()` and `str()`. `__repr__` = developer string, used in REPL and `repr()`.

> **Q7.** Encapsulation = bundling data and methods, controlling access. `_name` = "protected" (convention). `__name` = "private" (name-mangled).

> **Q8.** `2` — `count` is a class attribute, incremented each time a Dog is created.

> **Q9.** A property looks like an attribute but runs a method. Use `@property` to add validation, computation, or access control without changing the interface.

> **Q10.**
```python
class Counter:
    def __init__(self):
        self._value = 0
    def increment(self):
        self._value += 1
    def decrement(self):
        self._value -= 1
    def reset(self):
        self._value = 0
    def get_value(self):
        return self._value
```

---

## 🎬 End of Lesson Scene

*It's 5:30 PM. The Client class is deployed. Every system now uses `Client` objects instead of loose dictionaries.*

**Harvey:** "This is organized. Professional. I like it."

**Mike:** "Tomorrow we cover **instantiation** in depth — creating, comparing, and managing collections of objects."

**Louis:** *(examining the code)* "The encapsulation is acceptable. The properties validate input. I would have added more edge case handling."

**Donna:** "You always would, Louis."

*Day twelve. You're not just writing scripts or functions anymore — you're designing blueprints. The shift from procedural to object-oriented thinking has begun.*

---

## 📌 Concepts Mastered in This Lesson

| # | Concept | Status |
|---|---------|--------|
| 1 | Class definition with `class` | ✅ |
| 2 | `__init__` constructor | ✅ |
| 3 | `self` — reference to the current instance | ✅ |
| 4 | Instance attributes vs class attributes | ✅ |
| 5 | Methods — functions that belong to a class | ✅ |
| 6 | `__str__` and `__repr__` | ✅ |
| 7 | Encapsulation — `_protected` and `__private` | ✅ |
| 8 | Properties with `@property` | ✅ |
| 9 | Methods calling other methods | ✅ |
| 10 | Mutable class attribute trap | ✅ |

### 🔗 Connections to Previous Lessons

| From Earlier | Used in This Lesson |
|-------------|---------------------|
| Functions (Lesson 11) | Methods ARE functions (with `self`) |
| Dictionaries (Lesson 5) | Classes replace loose dicts |
| Lists (Lesson 3) | Storing collections in attributes |
| Conditionals (Lesson 7) | Logic inside methods |
| Loops (Lessons 9–10) | Loops inside methods |
| f-strings (Lesson 2) | `__str__` formatting |
| Mutable default trap (Lesson 11) | Same trap with class attributes |

---

> **Next Lesson →** Level 1, Lesson 13: *Instantiate Objects in Python*  
> *Building a roster of attorneys, managing client portfolios, comparing objects — time to put these blueprints to real use…*

# 🏛️ Level 4 – Advancing | Lesson 4: Use super() to Call Parent Methods in Python

## *"Calling Upstairs"*

> **O'Reilly Skill Objective:** *Use `super()` to call methods in a parent class.*

> **Previously Learned:** Inheritance, `__init__`, method overriding, `isinstance()`, MRO, abstract base classes

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

*Mike overrides `display()` in Associate — but now the employee ID and email are missing from the output.*

```python
class Employee:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    def display(self):
        return f"[{self.employee_id}] {self.name}"

class Associate(Employee):
    def __init__(self, name, employee_id, hourly_rate):
        self.name = name            # ❌ Duplicated!
        self.employee_id = employee_id  # ❌ Duplicated!
        self.hourly_rate = hourly_rate
    
    def display(self):
        return f"Associate: {self.name} (${self.hourly_rate}/hr)"
        # ❌ Lost the employee_id formatting from parent!
```

**Harvey:** "You're rewriting what the parent already does. Use `super()` — call upstairs, get what they have, then add your own."

```python
class Associate(Employee):
    def __init__(self, name, employee_id, hourly_rate):
        super().__init__(name, employee_id)    # ✅ Parent handles name + id
        self.hourly_rate = hourly_rate
    
    def display(self):
        base = super().display()               # ✅ Parent handles [id] name
        return f"{base} — Associate (${self.hourly_rate}/hr)"
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "`super()` delegates to the parent. Don't rewrite — extend."

| Pattern | What It Does |
|---------|-------------|
| `super().__init__(...)` | Call parent's constructor |
| `super().method(...)` | Call parent's version of an overridden method |
| `super()` in MRO | Follows the Method Resolution Order, not just the direct parent |

```python
# The core pattern:
class Child(Parent):
    def method(self, *args):
        result = super().method(*args)    # Parent does its thing
        # Child adds its own logic          # You add yours
        return enhanced_result
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `super()` in `__init__` — The Foundation ⭐

```python
class Employee:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
        print(f"  Employee.__init__: {name}")

class Partner(Employee):
    def __init__(self, name, employee_id, equity_share):
        super().__init__(name, employee_id)    # Calls Employee.__init__
        self.equity_share = equity_share
        print(f"  Partner.__init__: equity={equity_share}")

class ManagingPartner(Partner):
    def __init__(self, name, employee_id, equity_share, department):
        super().__init__(name, employee_id, equity_share)    # Calls Partner.__init__
        self.department = department
        print(f"  ManagingPartner.__init__: dept={department}")

jessica = ManagingPartner("Jessica", "MP001", 0.25, "Litigation")
# Employee.__init__: Jessica
# Partner.__init__: equity=0.25
# ManagingPartner.__init__: dept=Litigation

# All attributes set correctly through the chain:
print(jessica.name)           # Jessica
print(jessica.equity_share)   # 0.25
print(jessica.department)     # Litigation
```

---

### 3.2 `super()` in Regular Methods — Extend, Don't Replace ⭐⭐

```python
class Employee:
    def __init__(self, name):
        self.name = name
        self.permissions = {"view_own_profile"}
    
    def display(self):
        return f"{self.name}"
    
    def get_permissions(self):
        return self.permissions.copy()

class Partner(Employee):
    def __init__(self, name, equity):
        super().__init__(name)
        self.equity = equity
        self.permissions.update({"manage_clients", "view_financials"})
    
    def display(self):
        base = super().display()
        return f"⚖️ Partner: {base} ({self.equity:.0%} equity)"
    
    def get_permissions(self):
        perms = super().get_permissions()        # Get parent's permissions
        perms.add("sign_contracts")              # Add partner-specific
        return perms

class ManagingPartner(Partner):
    def __init__(self, name, equity, department):
        super().__init__(name, equity)
        self.department = department
    
    def display(self):
        base = super().display()
        return f"👑 {base} — Managing: {self.department}"
    
    def get_permissions(self):
        perms = super().get_permissions()        # Gets Partner's (which includes Employee's!)
        perms.add("hire_fire")
        perms.add("set_budgets")
        return perms

jessica = ManagingPartner("Jessica Pearson", 0.25, "Litigation")
print(jessica.display())
# 👑 ⚖️ Partner: Jessica Pearson (25% equity) — Managing: Litigation

print(jessica.get_permissions())
# {'view_own_profile', 'manage_clients', 'view_financials', 
#  'sign_contracts', 'hire_fire', 'set_budgets'}
```

---

### 3.3 How `super()` Actually Works — MRO ⭐⭐

```python
# super() does NOT always call the direct parent!
# It follows the MRO (Method Resolution Order).

class A:
    def greet(self):
        print("  A.greet")

class B(A):
    def greet(self):
        print("  B.greet")
        super().greet()    # Next in MRO, not necessarily A!

class C(A):
    def greet(self):
        print("  C.greet")
        super().greet()

class D(B, C):    # Multiple inheritance
    def greet(self):
        print("  D.greet")
        super().greet()

# MRO for D:
print(D.__mro__)
# D → B → C → A → object

d = D()
d.greet()
# D.greet
# B.greet       ← super() in D calls B (next in MRO)
# C.greet       ← super() in B calls C (next in MRO, NOT A!)
# A.greet       ← super() in C calls A

# KEY INSIGHT:
# super() means "next in the MRO" — not "my parent class"
```

---

### 3.4 `super()` with `*args` and `**kwargs` — Cooperative Inheritance ⭐⭐

```python
# When multiple classes need to cooperate with super(),
# use **kwargs to pass through unknown arguments.

class Employee:
    def __init__(self, name, employee_id, **kwargs):
        super().__init__(**kwargs)    # Pass remaining kwargs up
        self.name = name
        self.employee_id = employee_id

class Billable:
    def __init__(self, hourly_rate, **kwargs):
        super().__init__(**kwargs)
        self.hourly_rate = hourly_rate

class Trackable:
    def __init__(self, department, **kwargs):
        super().__init__(**kwargs)
        self.department = department

class Associate(Employee, Billable, Trackable):
    def __init__(self, name, employee_id, hourly_rate, department):
        # All kwargs flow through the MRO chain
        super().__init__(
            name=name,
            employee_id=employee_id,
            hourly_rate=hourly_rate,
            department=department
        )

mike = Associate("Mike Ross", "A001", 300, "Corporate")
print(mike.name)           # Mike Ross
print(mike.hourly_rate)    # 300
print(mike.department)     # Corporate

# MRO: Associate → Employee → Billable → Trackable → object
# Each class picks its kwargs and passes the rest via super()
```

---

### 3.5 Old-Style vs New-Style `super()` ⭐

```python
class Parent:
    def greet(self):
        return "Hello from Parent"

class Child(Parent):
    def greet(self):
        # ✅ Modern (Python 3): no arguments needed
        return super().greet() + " and Child"
    
    def greet_old_style(self):
        # Old (Python 2 / explicit): specify class and instance
        return super(Child, self).greet() + " and Child"
    
    def greet_explicit_skip(self):
        # Advanced: skip to grandparent (almost never needed)
        return super(Parent, self).greet()  # Calls object.greet — usually wrong!

# ✅ Always use super() without arguments in Python 3
# The compiler automatically inserts the right class and instance
```

---

### 3.6 `super()` in Class Methods and Static Methods

```python
class Employee:
    _registry = []
    
    def __init__(self, name):
        self.name = name
        self._registry.append(self)
    
    @classmethod
    def count(cls):
        return len([e for e in cls._registry if isinstance(e, cls)])
    
    @classmethod
    def from_string(cls, info_string):
        """Factory: 'name,id' → Employee."""
        name, emp_id = info_string.split(",")
        return cls(name.strip(), emp_id.strip())

class Partner(Employee):
    def __init__(self, name, emp_id, equity=0.10):
        super().__init__(name)
        self.emp_id = emp_id
        self.equity = equity
    
    @classmethod
    def from_string(cls, info_string):
        """Factory: 'name,id,equity' → Partner."""
        parts = info_string.split(",")
        name, emp_id = parts[0].strip(), parts[1].strip()
        equity = float(parts[2].strip()) if len(parts) > 2 else 0.10
        return cls(name, emp_id, equity)
    
    @classmethod
    def count(cls):
        base_count = super().count()    # super() works in classmethods too!
        return f"{base_count} partners"

harvey = Partner.from_string("Harvey Specter, P001, 0.15")
print(harvey.name)       # Harvey Specter
print(harvey.equity)     # 0.15
print(Partner.count())   # 1 partners
```

---

### 3.7 `super()` in `__str__`, `__repr__`, and Dunder Methods

```python
class Employee:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    def __str__(self):
        return f"{self.name} [{self.employee_id}]"
    
    def __repr__(self):
        return f"{type(self).__name__}('{self.name}', '{self.employee_id}')"
    
    def __eq__(self, other):
        if not isinstance(other, Employee):
            return NotImplemented
        return self.employee_id == other.employee_id

class Partner(Employee):
    def __init__(self, name, employee_id, equity):
        super().__init__(name, employee_id)
        self.equity = equity
    
    def __str__(self):
        base = super().__str__()
        return f"⚖️ {base} (equity: {self.equity:.0%})"
    
    def __repr__(self):
        return f"Partner('{self.name}', '{self.employee_id}', {self.equity})"
    
    def __eq__(self, other):
        if not isinstance(other, Partner):
            return super().__eq__(other)    # Fall back to Employee's eq
        return (self.employee_id == other.employee_id and 
                self.equity == other.equity)

harvey = Partner("Harvey", "P001", 0.15)
print(str(harvey))     # ⚖️ Harvey [P001] (equity: 15%)
print(repr(harvey))    # Partner('Harvey', 'P001', 0.15)
```

---

### 3.8 Abstract Methods and `super()`

```python
from abc import ABC, abstractmethod

class Employee(ABC):
    def __init__(self, name):
        self.name = name
    
    @abstractmethod
    def compensation(self):
        """Must be implemented. Can still provide a base calculation."""
        pass
    
    def annual_report(self):
        return f"{self.name}: ${self.compensation():,.2f}/yr"

class Partner(Employee):
    def __init__(self, name, equity):
        super().__init__(name)
        self.equity = equity
    
    def compensation(self):
        return self.equity * 5_000_000

class Associate(Employee):
    def __init__(self, name, salary, bonus_rate=0.1):
        super().__init__(name)
        self.salary = salary
        self.bonus_rate = bonus_rate
    
    def compensation(self):
        return self.salary * (1 + self.bonus_rate)

# Abstract method with a base implementation:
class Employee2(ABC):
    def __init__(self, name, base_salary):
        self.name = name
        self.base_salary = base_salary
    
    @abstractmethod
    def compensation(self):
        """Base provides a starting calculation."""
        return self.base_salary    # Can provide default logic!

class Partner2(Employee2):
    def __init__(self, name, base_salary, equity):
        super().__init__(name, base_salary)
        self.equity = equity
    
    def compensation(self):
        base = super().compensation()    # Get base salary from abstract method!
        return base + (self.equity * 3_000_000)

p = Partner2("Harvey", 200_000, 0.15)
print(p.compensation())    # 650000.0
```

---

### 3.9 When NOT to Use `super()`

```python
# 1. When you intentionally want to REPLACE, not extend
class CSVExporter:
    def export(self, data):
        return ",".join(str(x) for x in data)

class JSONExporter(CSVExporter):
    def export(self, data):
        import json
        return json.dumps(data)    # Complete replacement — super() not needed

# 2. When explicitly calling a SPECIFIC parent (rare)
class A:
    def method(self): return "A"
class B(A):
    def method(self): return "B"
class C(A):
    def method(self): return "C"

class D(B, C):
    def method(self):
        # Skip MRO, call a specific parent directly
        return C.method(self)    # ← Explicit, not super()
        # ⚠️ Use sparingly! Breaks cooperative inheritance

# 3. When the parent method signature is completely different
class Base:
    def process(self, x):
        return x * 2

class Child(Base):
    def process(self, x, y, z):    # Different signature!
        # super().process(x, y, z) would fail — Base only takes x!
        return x + y + z
```

---

### 3.10 Solving Mike's Problem — Cooperative Hierarchy

```python
# ============================================
# Pearson Specter Litt — Cooperative super() Chain
# Each level adds its own logic without duplicating
# ============================================

from abc import ABC, abstractmethod
from datetime import datetime

class Employee(ABC):
    """Level 1: base employee — name, id, email."""
    
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
        self.hire_date = datetime.now()
    
    @abstractmethod
    def role(self):
        pass
    
    @abstractmethod
    def compensation(self):
        pass
    
    def display(self):
        return f"[{self.employee_id}] {self.name}"
    
    def full_report(self):
        return (f"  {self.display()}\n"
                f"    Role: {self.role()}\n"
                f"    Comp: ${self.compensation():,.2f}/yr")
    
    def __repr__(self):
        return f"{type(self).__name__}('{self.name}')"


class Attorney(Employee):
    """Level 2: adds billable rate and case tracking."""
    
    def __init__(self, name, employee_id, hourly_rate):
        super().__init__(name, employee_id)
        self.hourly_rate = hourly_rate
        self.cases = []
        self.hours_billed = 0
    
    def bill(self, client, hours):
        self.hours_billed += hours
        self.cases.append({"client": client, "hours": hours})
        return hours * self.hourly_rate
    
    def display(self):
        base = super().display()
        return f"{base} (${self.hourly_rate}/hr, {self.hours_billed}h billed)"
    
    def full_report(self):
        base = super().full_report()
        billing = self.hours_billed * self.hourly_rate
        return f"{base}\n    Billing: ${billing:,.2f} ({len(self.cases)} cases)"


class Partner(Attorney):
    """Level 3: adds equity and client portfolio."""
    
    def __init__(self, name, employee_id, hourly_rate, equity_share):
        super().__init__(name, employee_id, hourly_rate)
        self.equity_share = equity_share
        self.portfolio = []
    
    def role(self):
        return "Partner"
    
    def compensation(self):
        return int(self.equity_share * 5_000_000)
    
    def display(self):
        base = super().display()
        return f"⚖️ {base} [{self.equity_share:.0%} equity]"
    
    def full_report(self):
        base = super().full_report()
        return f"{base}\n    Portfolio: {len(self.portfolio)} clients"


class Associate(Attorney):
    """Level 3: adds salary and mentoring."""
    
    def __init__(self, name, employee_id, hourly_rate, base_salary):
        super().__init__(name, employee_id, hourly_rate)
        self.base_salary = base_salary
    
    def role(self):
        return "Senior Associate" if self.hours_billed > 2000 else "Associate"
    
    def compensation(self):
        bonus = self.hours_billed * self.hourly_rate * 0.05
        return int(self.base_salary + bonus)
    
    def display(self):
        base = super().display()
        return f"📋 {base}"


class ManagingPartner(Partner):
    """Level 4: adds department management."""
    
    def __init__(self, name, employee_id, hourly_rate, equity_share, department):
        super().__init__(name, employee_id, hourly_rate, equity_share)
        self.department = department
    
    def role(self):
        return f"Managing Partner — {self.department}"
    
    def compensation(self):
        base = super().compensation()
        return int(base * 1.2)    # 20% management premium
    
    def display(self):
        base = super().display()
        return f"👑 {base}"
    
    def full_report(self):
        base = super().full_report()
        return f"{base}\n    Department: {self.department}"


# === Demonstrate ===
print("=" * 70)
print(f"{'🏛️  COOPERATIVE super() CHAIN':^70}")
print("=" * 70)

staff = [
    ManagingPartner("Jessica Pearson", "MP001", 500, 0.25, "Litigation"),
    Partner("Harvey Specter", "P001", 450, 0.15),
    Partner("Louis Litt", "P002", 400, 0.12),
    Associate("Mike Ross", "A001", 300, 200_000),
    Associate("Katrina Bennett", "A002", 280, 180_000),
]

# Simulate billing
staff[1].bill("Wayne Enterprises", 85)
staff[1].bill("Stark Industries", 40)
staff[3].bill("Wayne Enterprises", 62)
staff[3].bill("Oscorp", 2000)    # Mike works hard

# Display chain: ManagingPartner → Partner → Attorney → Employee
print("\n📋 ROSTER (display chain):")
for emp in staff:
    print(f"  {emp.display()}")

# Full reports — each level adds its own section
print("\n📊 FULL REPORTS:")
for emp in staff:
    print(emp.full_report())
    print()

# MRO
print("🔗 MRO for ManagingPartner:")
for cls in ManagingPartner.__mro__:
    print(f"  → {cls.__name__}")

print(f"\n{'='*70}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting to Call `super().__init__()` 🔥

```python
class Parent:
    def __init__(self):
        self.important = True

class Child(Parent):
    def __init__(self):
        self.extra = True
        # Forgot super().__init__()!

c = Child()
# c.important    # AttributeError!

# ✅ Always call super().__init__()
class Child(Parent):
    def __init__(self):
        super().__init__()
        self.extra = True
```

---

### Pitfall 2: Wrong Argument Passing to `super()`

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        # ❌ Passes breed to Animal — Animal doesn't accept it!
        # super().__init__(name, breed)    # TypeError!
        
        # ✅ Only pass what the parent expects
        super().__init__(name)
        self.breed = breed
```

---

### Pitfall 3: `super()` in Multiple Inheritance — MRO Surprise

```python
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        print("B")
        super().method()

class C(A):
    def method(self):
        print("C")
        super().method()

class D(B, C):
    def method(self):
        print("D")
        super().method()

D().method()
# D → B → C → A (NOT D → B → A!)
# super() in B calls C, not A!
```

---

### Pitfall 4: Mixing `super()` and Direct Parent Calls

```python
# ❌ Inconsistent — some use super(), some don't
class Base:
    def __init__(self):
        self.x = 1

class Middle(Base):
    def __init__(self):
        Base.__init__(self)    # Direct call
        self.y = 2

class Child(Middle):
    def __init__(self):
        super().__init__()     # super() call
        self.z = 3

# ✅ Be consistent — always use super()
class Middle(Base):
    def __init__(self):
        super().__init__()     # ✅ Consistent
        self.y = 2
```

---

### Pitfall 5: Calling `super()` Outside a Class

```python
# ❌ super() without arguments only works inside a class
# result = super().some_method()    # RuntimeError: super(): no current class

# ✅ Outside a class, use explicit form (rare)
# super(SpecificClass, instance).method()
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📞 Analogy: super() = Calling Upstairs

```
THE FIRM HIERARCHY:
  4th Floor: Jessica (ManagingPartner)
  3rd Floor: Harvey (Partner)
  2nd Floor: Attorney (Attorney)
  1st Floor: HR (Employee)

WITHOUT super():
  Jessica writes her own report from scratch.
  Duplicates Harvey's equity info, the billing info,
  the HR info — everything.

WITH super():
  Jessica calls downstairs: "Send me your section."
  Harvey adds equity info, calls downstairs.
  Attorney adds billing info, calls downstairs.
  HR adds name and ID.
  
  Each floor adds ONE section. No duplication.
  Change HR's format → all floors update automatically.

DONNA'S RULE:
  "super() is the intercom.
   Don't rewrite what downstairs already handles.
   Call them, get their part, add yours."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Shape with Perimeter**
```
Shape.__init__(color) → Circle.__init__(color, radius)
Shape.describe() → "A red shape"
Circle.describe() → "A red shape — Circle with radius 5"
Use super() in both __init__ and describe().
```

**Exercise 2: Logger Chain**
```
BaseLogger.log(msg) → prints "[LOG] msg"
TimedLogger.log(msg) → adds timestamp, then super().log()
LevelLogger.log(msg, level) → adds level, then super().log()
```

**Exercise 3: Menu Items**
```
MenuItem(name, price)
DrinkItem(name, price, size) — adds size surcharge
ComboItem(name, items) — sums item prices
All override display() using super()
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Validation Chain**
```
BaseValidator.validate(data) → basic checks
StringValidator.validate(data) → super() + length, pattern checks
EmailValidator.validate(data) → super() + email format check
Each level calls super() before adding its own rules.
```

**Exercise 5: Event Pipeline**
```
BaseHandler.handle(event) → logs event
AuthHandler.handle(event) → checks auth, then super()
RateLimitHandler.handle(event) → checks rate, then super()
Stack handlers with super() — order determines processing.
```

**Exercise 6: Cooperative `__init__` with `**kwargs`**
```
Build 4 mixin classes that all cooperate:
  NameMixin, TimestampMixin, TagMixin, PriorityMixin
  
class Task(NameMixin, TimestampMixin, TagMixin, PriorityMixin):
    pass

All use **kwargs to pass through unknown arguments.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Django-Style Model**
```
Build a Model base class where:
  - __init_subclass__ auto-registers models
  - save() chains through super() for validation
  - Each level adds its own validation rules
  - Full audit trail through the super() chain
```

**Exercise 8: Middleware Stack**
```
Build a web-style middleware stack:
  Each middleware calls super().process(request)
  before/after its own logic.

class AuthMiddleware(BaseMiddleware):
    def process(self, request):
        self.check_auth(request)
        response = super().process(request)
        self.log_access(request)
        return response
```

**Exercise 9: MRO Visualizer**
```
Build visualize_mro(cls) that:
  - Prints the full MRO chain
  - Shows which class defines each method
  - Shows which super() call leads where
  - Highlights diamond inheritance patterns
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Forgot super().__init__
class Base:
    def __init__(self):
        self.ready = True

class Child(Base):
    def __init__(self):
        self.value = 42
# Child().ready    # AttributeError!

# Bug 2: Wrong args to super
class Animal:
    def __init__(self, name):
        self.name = name
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name, breed)  # TypeError! Animal takes 1 arg!

# Bug 3: super() calls A, not expected B
class A:
    def m(self): return "A"
class B(A):
    def m(self): return "B"
class C(A):
    def m(self): return "C"
class D(B, C):
    def m(self): return super().m()  # Returns "B", user expected "A"

# Bug 4: Overriding without super — losing parent behavior
class Logger:
    def log(self, msg):
        print(f"[LOG] {msg}")
class FileLogger(Logger):
    def log(self, msg):
        with open("log.txt", "a") as f:
            f.write(msg)
        # ❌ No longer prints to console!
```

### Fixed Code ✅

```python
# Fix 1: Add super().__init__()
class Child(Base):
    def __init__(self):
        super().__init__()    # ✅
        self.value = 42

# Fix 2: Pass correct args
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)    # ✅ Only name
        self.breed = breed

# Fix 3: Understand MRO — D → B → C → A
# super() in D calls B, which is correct per MRO
# If you need A specifically: A.m(self)

# Fix 4: Call super() to keep console logging
class FileLogger(Logger):
    def log(self, msg):
        super().log(msg)    # ✅ Console output preserved
        with open("log.txt", "a") as f:
            f.write(msg + "\n")
```

---

## 8. 🌍 Real-World Application

### `super()` in Popular Frameworks

| Framework | `super()` Usage |
|-----------|----------------|
| **Django** | `super().save()` in model overrides |
| **Flask** | `super().__init__()` in custom views |
| **SQLAlchemy** | `super().__init_subclass__()` for model registration |
| **unittest** | `super().setUp()` in test fixtures |
| **logging** | `super().__init__()` in custom handlers |
| **tkinter** | `super().__init__(master)` in custom widgets |
| **dataclasses** | `super().__init_subclass__()` for field inheritance |
| **Exception classes** | `super().__init__(message)` in custom exceptions |

---

## 9. ✅ Knowledge Check

---

**Q1.** What does `super()` return?

**Q2.** What is the most common use of `super()` in `__init__`?

**Q3.** Does `super()` always call the direct parent class?

**Q4.** What is cooperative multiple inheritance?

**Q5.** How does `super()` decide which class to call?

**Q6.** Can you use `super()` in classmethods?

**Q7.** What happens if you forget `super().__init__()`?

**Q8.** How do you pass arguments through a cooperative `super()` chain?

**Q9.** When is it okay NOT to call `super()`?

**Q10.** Why is `super()` preferred over `ParentClass.method(self)`?

---

### 📋 Answer Key

> **Q1.** A proxy object that delegates method calls to the next class in the MRO.

> **Q2.** To call the parent's `__init__` and set up inherited attributes without duplicating code.

> **Q3.** No — it calls the next class in the MRO, which may not be the direct parent (especially with multiple inheritance).

> **Q4.** All classes in a hierarchy use `super()` and `**kwargs` to pass arguments through the MRO chain, ensuring every class gets initialized.

> **Q5.** It uses the Method Resolution Order (MRO), computed by the C3 linearization algorithm. Viewable via `ClassName.__mro__`.

> **Q6.** Yes — `super()` works in `@classmethod` and `@staticmethod` (though less common in static methods).

> **Q7.** The parent's `__init__` doesn't run, so attributes set there won't exist. You'll get `AttributeError` when accessing them.

> **Q8.** Use `**kwargs` — each class extracts the arguments it needs and passes the rest via `super().__init__(**kwargs)`.

> **Q9.** When you intentionally want to completely replace (not extend) the parent's behavior, or when calling a specific parent directly.

> **Q10.** `super()` follows the MRO correctly, supporting cooperative multiple inheritance. Direct calls bypass the MRO and can cause methods to be called multiple times or skipped.

---

## 🎬 End of Lesson Scene

**Mike:** "Each level in the hierarchy adds one thing. `super()` chains them together. ManagingPartner calls Partner, Partner calls Attorney, Attorney calls Employee. No duplication."

**Harvey:** "Change the email format in Employee?"

**Mike:** "Every subclass gets it automatically. `super()` is the intercom — each floor handles its own section."

**Harvey:** "Next: multiple inheritance. When a class has MORE than one parent."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `super().__init__()` for parent construction | ✅ |
| 2 | `super().method()` to extend overrides | ✅ |
| 3 | `super()` follows MRO, not just direct parent | ✅ |
| 4 | Cooperative inheritance with `**kwargs` | ✅ |
| 5 | `super()` in classmethods | ✅ |
| 6 | `super()` in dunder methods (`__str__`, `__eq__`) | ✅ |
| 7 | Abstract methods with `super()` base logic | ✅ |
| 8 | Old-style vs new-style `super()` | ✅ |
| 9 | When NOT to use `super()` | ✅ |
| 10 | Full cooperative hierarchy chain | ✅ |

---

> **Next Lesson →** Level 4, Lesson 5: *Implement Multiple Inheritance in Python*

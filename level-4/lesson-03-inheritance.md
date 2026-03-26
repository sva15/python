# 🏛️ Level 4 – Advancing | Lesson 3: Use Inheritance to Extend Python Classes

## *"The Family Name"*

> **O'Reilly Skill Objective:** *Implement inheritance in a class to create specialized classes.*

> **Previously Learned:** Classes, `__init__`, instance methods, dunder methods, `@property`, duck typing, decorators, closures

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

*Three employee types — Partners, Associates, Paralegals — all share common attributes but behave differently.*

```python
# ❌ Without inheritance — massive duplication!
class Partner:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
        self.role = "Partner"
    def get_email(self):
        return f"{self.name.lower().replace(' ', '.')}@psl.com"
    def annual_salary(self):
        return 500_000   # Partner-specific

class Associate:
    def __init__(self, name, employee_id):
        self.name = name                    # Duplicated!
        self.employee_id = employee_id      # Duplicated!
        self.role = "Associate"
    def get_email(self):                     # Duplicated!
        return f"{self.name.lower().replace(' ', '.')}@psl.com"
    def annual_salary(self):
        return 200_000   # Associate-specific
```

**Harvey:** "Three classes, 80% identical code. Change the email format and you update three places."

**Mike:** "Inheritance. Define the common parts once in a **base class**. Each role **inherits** and only adds what's different."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Inheritance is the family name. The child gets everything the parent has — and can add their own."

| Term | Meaning |
|------|---------|
| **Base class** (parent/superclass) | The general class being inherited from |
| **Derived class** (child/subclass) | The specialized class that inherits |
| **Inherit** | Child gets all parent's methods and attributes |
| **Override** | Child replaces a parent's method with its own |
| **Extend** | Child adds new methods the parent doesn't have |
| **`isinstance()`** | Check if object is an instance of a class (or its parents) |
| **`issubclass()`** | Check if a class inherits from another |

```python
class Employee:              # Base class
    ...

class Partner(Employee):     # Inherits from Employee
    ...

class Associate(Employee):   # Also inherits from Employee
    ...
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Inheritance ⭐

```python
class Employee:
    """Base class for all firm employees."""
    
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    def get_email(self):
        return f"{self.name.lower().replace(' ', '.')}@psl.com"
    
    def display(self):
        return f"[{self.employee_id}] {self.name} ({self.get_email()})"
    
    def __repr__(self):
        return f"{type(self).__name__}('{self.name}', '{self.employee_id}')"

# Child class — inherits EVERYTHING from Employee
class Partner(Employee):
    pass    # No new code — but has all Employee methods!

# Usage
harvey = Partner("Harvey Specter", "P001")
print(harvey.name)          # Harvey Specter       ← Inherited
print(harvey.get_email())   # harvey.specter@psl.com ← Inherited
print(harvey.display())     # [P001] Harvey Specter (harvey.specter@psl.com) ← Inherited
print(repr(harvey))         # Partner('Harvey Specter', 'P001')
```

---

### 3.2 Extending the Parent — Adding New Features ⭐

```python
class Employee:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    def get_email(self):
        return f"{self.name.lower().replace(' ', '.')}@psl.com"

class Partner(Employee):
    """Partner adds equity share and client portfolio."""
    
    def __init__(self, name, employee_id, equity_share):
        # Call parent's __init__ to set name and employee_id
        super().__init__(name, employee_id)
        # Add partner-specific attributes
        self.equity_share = equity_share
        self.clients = []
    
    def add_client(self, client_name):
        """Partner-specific method — Employee doesn't have this."""
        self.clients.append(client_name)
    
    def annual_bonus(self):
        """Calculate equity-based bonus."""
        return self.equity_share * 2_000_000

class Associate(Employee):
    """Associate adds billable hour tracking."""
    
    def __init__(self, name, employee_id, hourly_rate):
        super().__init__(name, employee_id)
        self.hourly_rate = hourly_rate
        self.hours_billed = 0
    
    def log_hours(self, hours):
        """Associate-specific method."""
        self.hours_billed += hours
    
    def total_billing(self):
        return self.hours_billed * self.hourly_rate

# Usage
harvey = Partner("Harvey Specter", "P001", equity_share=0.15)
harvey.add_client("Wayne Enterprises")
print(harvey.get_email())          # harvey.specter@psl.com (inherited)
print(harvey.annual_bonus())       # 300000.0 (partner-specific)

mike = Associate("Mike Ross", "A001", hourly_rate=300)
mike.log_hours(85)
print(mike.get_email())            # mike.ross@psl.com (inherited)
print(mike.total_billing())        # 25500 (associate-specific)
```

---

### 3.3 Overriding Methods ⭐⭐

```python
class Employee:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    def annual_salary(self):
        """Default salary — subclasses should override."""
        return 50_000
    
    def display(self):
        return f"{self.name} (${self.annual_salary():,}/yr)"

class Partner(Employee):
    def __init__(self, name, employee_id, equity_share):
        super().__init__(name, employee_id)
        self.equity_share = equity_share
    
    def annual_salary(self):
        """Override: partners earn based on equity."""
        return int(self.equity_share * 5_000_000)

class Associate(Employee):
    def __init__(self, name, employee_id, base_salary):
        super().__init__(name, employee_id)
        self.base_salary = base_salary
    
    def annual_salary(self):
        """Override: associates have a fixed salary."""
        return self.base_salary

class Paralegal(Employee):
    # No override — uses Employee's default: $50,000
    pass

# Polymorphism: same method name, different behavior!
staff = [
    Partner("Harvey", "P001", 0.15),       # $750,000
    Associate("Mike", "A001", 200_000),     # $200,000
    Paralegal("Rachel", "PL001"),           # $50,000
]

for person in staff:
    print(f"  {person.display()}")
# Harvey ($750,000/yr)
# Mike ($200,000/yr)
# Rachel ($50,000/yr)
```

---

### 3.4 `isinstance()` and `issubclass()` ⭐

```python
class Employee: pass
class Partner(Employee): pass
class Associate(Employee): pass

harvey = Partner("Harvey", "P001")

# isinstance checks the entire inheritance chain
print(isinstance(harvey, Partner))     # True
print(isinstance(harvey, Employee))    # True (Partner IS an Employee!)
print(isinstance(harvey, Associate))   # False

# issubclass checks class hierarchy
print(issubclass(Partner, Employee))     # True
print(issubclass(Associate, Employee))   # True
print(issubclass(Partner, Associate))    # False
print(issubclass(Employee, object))      # True (everything inherits from object!)

# Practical use: type-based logic
def process_employee(emp):
    if isinstance(emp, Partner):
        print(f"  Partner: {emp.name} — equity distributions")
    elif isinstance(emp, Associate):
        print(f"  Associate: {emp.name} — performance review")
    elif isinstance(emp, Employee):
        print(f"  Employee: {emp.name} — standard processing")

# ⚠️ But prefer duck typing when possible:
def process_employee_duck(emp):
    emp.process()    # Let each class define its own process()
```

---

### 3.5 The `object` Base Class — Everything Inherits From `object`

```python
# ALL Python classes implicitly inherit from object
class MyClass:
    pass

print(issubclass(MyClass, object))    # True
print(isinstance(42, object))         # True
print(isinstance("hello", object))    # True
print(isinstance(None, object))       # True

# object provides default dunder methods:
print(dir(object))
# ['__class__', '__delattr__', '__dir__', '__doc__', '__eq__',
#  '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__',
#  '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__',
#  '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__',
#  '__sizeof__', '__str__', '__subclasshook__']

# That's why every object has __repr__, __str__, __eq__, etc.!
```

---

### 3.6 Method Resolution Order (MRO) ⭐⭐

```python
class Employee:
    def identify(self):
        return "I'm an Employee"

class Partner(Employee):
    def identify(self):
        return "I'm a Partner"

class SeniorPartner(Partner):
    pass    # No override — inherits from Partner

# MRO: the order Python searches for methods
print(SeniorPartner.__mro__)
# (<class 'SeniorPartner'>, <class 'Partner'>, <class 'Employee'>, <class 'object'>)

sp = SeniorPartner("Jessica", "SP001")
print(sp.identify())    # "I'm a Partner"
# Search: SeniorPartner → not found → Partner → FOUND ✅

# Visualize MRO
for cls in SeniorPartner.__mro__:
    print(f"  {cls.__name__}", end="")
    if hasattr(cls, 'identify') and 'identify' in cls.__dict__:
        print(f" ← has identify()")
    else:
        print()
# SeniorPartner
# Partner ← has identify()
# Employee ← has identify()
# object
```

---

### 3.7 Calling Parent Methods with `super()` ⭐⭐

```python
class Employee:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    def display(self):
        return f"{self.name} [{self.employee_id}]"

class Partner(Employee):
    def __init__(self, name, employee_id, equity_share):
        super().__init__(name, employee_id)    # Call parent's __init__
        self.equity_share = equity_share
    
    def display(self):
        # Extend parent's display — don't duplicate!
        base = super().display()
        return f"{base} — Partner ({self.equity_share:.0%} equity)"

class ManagingPartner(Partner):
    def __init__(self, name, employee_id, equity_share, department):
        super().__init__(name, employee_id, equity_share)
        self.department = department
    
    def display(self):
        base = super().display()
        return f"{base} — Managing: {self.department}"

# Chain of super() calls
jessica = ManagingPartner("Jessica Pearson", "MP001", 0.25, "Litigation")
print(jessica.display())
# Jessica Pearson [MP001] — Partner (25% equity) — Managing: Litigation
# ManagingPartner.display → Partner.display → Employee.display
```

---

### 3.8 Abstract Base Classes — Enforcing Override ⭐

```python
from abc import ABC, abstractmethod

class Employee(ABC):
    """Abstract base class — cannot be instantiated directly."""
    
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    @abstractmethod
    def annual_salary(self):
        """Subclasses MUST implement this."""
        pass
    
    @abstractmethod
    def role_title(self):
        """Subclasses MUST implement this."""
        pass
    
    # Concrete method — inherited as-is
    def get_email(self):
        return f"{self.name.lower().replace(' ', '.')}@psl.com"
    
    def display(self):
        return f"{self.role_title()}: {self.name} — ${self.annual_salary():,}/yr"

# ❌ Can't instantiate abstract class!
# emp = Employee("Test", "E001")    # TypeError: Can't instantiate abstract class!

class Partner(Employee):
    def __init__(self, name, employee_id, equity):
        super().__init__(name, employee_id)
        self.equity = equity
    
    def annual_salary(self):
        return int(self.equity * 5_000_000)
    
    def role_title(self):
        return "Partner"

class Associate(Employee):
    def __init__(self, name, employee_id, salary):
        super().__init__(name, employee_id)
        self._salary = salary
    
    def annual_salary(self):
        return self._salary
    
    def role_title(self):
        return "Associate"

# ❌ Forgot to implement role_title?
# class Incomplete(Employee):
#     def annual_salary(self): return 0
#     # Missing role_title! → TypeError on instantiation!

# ✅ Concrete classes work
harvey = Partner("Harvey", "P001", 0.15)
mike = Associate("Mike", "A001", 200_000)

for emp in [harvey, mike]:
    print(f"  {emp.display()}")
# Partner: Harvey — $750,000/yr
# Associate: Mike — $200,000/yr
```

---

### 3.9 Inheritance vs Composition — When to Use Which

```python
# === INHERITANCE: "is-a" relationship ===
# A Partner IS AN Employee
class Employee:
    def __init__(self, name):
        self.name = name

class Partner(Employee):    # Partner IS-A Employee ✅
    pass

# === COMPOSITION: "has-a" relationship ===
# A Case HAS A Client (not a case IS a client!)
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

class Case:
    def __init__(self, title, client):
        self.title = title
        self.client = client    # HAS-A Client ✅

# === Decision guide ===
"""
USE INHERITANCE when:
  ✅ True "is-a" relationship (Dog IS Animal)
  ✅ Subclass extends/specializes parent behavior
  ✅ Polymorphism needed (treat all subtypes uniformly)
  ✅ Abstract base class defines a contract

USE COMPOSITION when:
  ✅ "Has-a" relationship (Car HAS Engine)
  ✅ Mixing behaviors from unrelated classes
  ✅ More flexibility needed (swap components)
  ✅ Avoid deep inheritance hierarchies
  
GENERAL RULE: Favor composition over inheritance.
Inheritance is for specialization, not code reuse.
"""
```

---

### 3.10 Solving Harvey's Problem — Firm Employee Hierarchy

```python
# ============================================
# Pearson Specter Litt — Employee Hierarchy
# Inheritance for specialization, polymorphism
# ============================================

from abc import ABC, abstractmethod
from datetime import datetime

class Employee(ABC):
    """Abstract base: all employees share these features."""
    
    _employee_count = 0
    
    def __init__(self, name, employee_id, start_date=None):
        self.name = name
        self.employee_id = employee_id
        self.start_date = start_date or datetime.now()
        Employee._employee_count += 1
    
    @abstractmethod
    def annual_salary(self):
        """Each role calculates salary differently."""
        pass
    
    @abstractmethod
    def role_title(self):
        pass
    
    def get_email(self):
        return f"{self.name.lower().replace(' ', '.')}@psl.com"
    
    @property
    def years_at_firm(self):
        return (datetime.now() - self.start_date).days / 365.25
    
    def display(self):
        return (f"  {self.role_title():<20} {self.name:<20} "
                f"${self.annual_salary():>12,}/yr  {self.get_email()}")
    
    def __repr__(self):
        return f"{type(self).__name__}('{self.name}')"


class Partner(Employee):
    """Partners earn equity-based compensation."""
    
    def __init__(self, name, employee_id, equity_share, managing=False):
        super().__init__(name, employee_id)
        self.equity_share = equity_share
        self.managing = managing
        self.clients = []
    
    def annual_salary(self):
        base = int(self.equity_share * 5_000_000)
        return int(base * 1.2) if self.managing else base
    
    def role_title(self):
        return "Managing Partner" if self.managing else "Partner"
    
    def add_client(self, client_name):
        self.clients.append(client_name)
        return f"  ✅ {client_name} added to {self.name}'s portfolio"


class Associate(Employee):
    """Associates earn salary + billable bonus."""
    
    def __init__(self, name, employee_id, base_salary, hourly_rate):
        super().__init__(name, employee_id)
        self.base_salary = base_salary
        self.hourly_rate = hourly_rate
        self.hours_billed = 0
    
    def annual_salary(self):
        bonus = self.hours_billed * self.hourly_rate * 0.1
        return int(self.base_salary + bonus)
    
    def role_title(self):
        return "Senior Associate" if self.hours_billed > 2000 else "Associate"
    
    def log_hours(self, hours):
        self.hours_billed += hours


class Paralegal(Employee):
    """Paralegals earn a fixed salary."""
    
    def __init__(self, name, employee_id, salary):
        super().__init__(name, employee_id)
        self._salary = salary
    
    def annual_salary(self):
        return self._salary
    
    def role_title(self):
        return "Paralegal"


# === Demonstrate ===
print("=" * 80)
print(f"{'🏛️  PEARSON SPECTER LITT — EMPLOYEE ROSTER':^80}")
print("=" * 80)

staff = [
    Partner("Jessica Pearson", "MP001", 0.25, managing=True),
    Partner("Harvey Specter", "P001", 0.15),
    Partner("Louis Litt", "P002", 0.12),
    Associate("Mike Ross", "A001", 200_000, 300),
    Associate("Katrina Bennett", "A002", 180_000, 280),
    Paralegal("Rachel Zane", "PL001", 85_000),
    Paralegal("Donna Paulsen", "PL002", 90_000),
]

# Mike bills 2500 hours
staff[3].log_hours(2500)

# Harvey adds clients
staff[1].add_client("Wayne Enterprises")
staff[1].add_client("Stark Industries")

# Polymorphic display — same method, different behavior
print(f"\n  {'Role':<20} {'Name':<20} {'Salary':>14}  {'Email'}")
print(f"  {'─'*20} {'─'*20} {'─'*14}  {'─'*30}")
for emp in staff:
    print(emp.display())

# Totals
total_payroll = sum(e.annual_salary() for e in staff)
print(f"\n  {'Total Payroll:':<42} ${total_payroll:>12,}")

# Type checking
print(f"\n📊 Breakdown:")
partners = [e for e in staff if isinstance(e, Partner)]
associates = [e for e in staff if isinstance(e, Associate)]
paralegals = [e for e in staff if isinstance(e, Paralegal)]
print(f"  Partners: {len(partners)}")
print(f"  Associates: {len(associates)}")
print(f"  Paralegals: {len(paralegals)}")

# MRO
print(f"\n🔗 MRO for Partner:")
for cls in Partner.__mro__:
    print(f"  → {cls.__name__}")

print(f"\n{'='*80}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting `super().__init__()` 🔥

```python
class Employee:
    def __init__(self, name):
        self.name = name

class Partner(Employee):
    def __init__(self, name, equity):
        # ❌ Forgot super().__init__!
        self.equity = equity

harvey = Partner("Harvey", 0.15)
# print(harvey.name)    # AttributeError: 'Partner' has no attribute 'name'!

# ✅ Always call super().__init__
class Partner(Employee):
    def __init__(self, name, equity):
        super().__init__(name)    # ✅ Sets self.name
        self.equity = equity
```

---

### Pitfall 2: Deep Inheritance Hierarchies

```python
# ❌ Too deep — hard to understand and maintain!
class A: pass
class B(A): pass
class C(B): pass
class D(C): pass
class E(D): pass    # 5 levels deep!

# ✅ Prefer flat hierarchies (2-3 levels max)
# Use composition for shared behavior instead of deep chains
```

---

### Pitfall 3: Overriding Without Calling `super()`

```python
class Employee:
    def display(self):
        return f"Employee: {self.name}"

class Partner(Employee):
    def display(self):
        # ❌ Completely replaces parent — duplicates name logic!
        return f"Partner: {self.name} (equity: {self.equity})"

# ✅ Extend instead of replace
class Partner(Employee):
    def display(self):
        base = super().display()    # Get parent's output
        return f"{base} (equity: {self.equity})"    # Add to it
```

---

### Pitfall 4: Mutable Class Attributes Shared Across Instances

```python
class Employee:
    skills = []    # ⚠️ Shared across ALL instances!

class Partner(Employee): pass
class Associate(Employee): pass

p = Partner()
a = Associate()
p.skills.append("negotiation")
print(a.skills)    # ['negotiation'] ← Shared!

# ✅ Initialize mutable attributes in __init__
class Employee:
    def __init__(self):
        self.skills = []    # ✅ Each instance gets its own list
```

---

### Pitfall 5: Using Inheritance for Code Reuse (Not Specialization)

```python
# ❌ Dog is not a "kind of" JSONSerializer!
class JSONSerializer:
    def to_json(self): ...

class Dog(JSONSerializer):    # Wrong! Dog is not a serializer!
    pass

# ✅ Use composition or mixins
class Dog:
    def __init__(self):
        self.serializer = JSONSerializer()
    
    def to_json(self):
        return self.serializer.serialize(self)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 👨‍👩‍👧‍👦 Analogy: Inheritance = The Family Name

```
EMPLOYEE (Parent):
  📋 name, employee_id, get_email()
  → The family name: everyone in the family has these

PARTNER (Child):
  📋 Inherits: name, employee_id, get_email()
  📋+ Adds: equity_share, add_client(), annual_bonus()
  → Eldest child: has family traits PLUS leadership responsibilities

ASSOCIATE (Child):
  📋 Inherits: name, employee_id, get_email()
  📋+ Adds: hourly_rate, log_hours(), total_billing()
  → Second child: has family traits PLUS their own skills

DONNA'S RULE:
  "Inheritance is the family name.
   The child inherits everything from the parent.
   But the child is free to forge their own path.
   Just don't forget where you came from (super().__init__)."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Shape Hierarchy**
```
Build: Shape → Circle, Rectangle, Triangle
  - Shape: abstract with area() and perimeter()
  - Each subclass implements its own formulas
  - Polymorphic: loop over all shapes, print area
```

**Exercise 2: Vehicle Hierarchy**
```
Build: Vehicle → Car, Truck, Motorcycle
  - Common: make, model, year, fuel_type
  - Car: num_doors, trunk_size
  - Truck: payload_capacity, num_axles
  - Motorcycle: engine_cc
```

**Exercise 3: Bank Account**
```
Build: Account → CheckingAccount, SavingsAccount
  - Account: balance, deposit(), withdraw()
  - Checking: overdraft_limit, monthly_fee
  - Savings: interest_rate, apply_interest()
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Game Character System**
```
Build: Character → Warrior, Mage, Healer
  - Abstract: attack(), defend(), special_ability()
  - Different stats per class
  - Battle simulation using polymorphism
```

**Exercise 5: Document System**
```
Build: Document → PDF, Word, Spreadsheet
  - Abstract: render(), export(), word_count()
  - Each subclass renders differently
  - DocumentManager processes any Document
```

**Exercise 6: Notification System**
```
Build: Notification → EmailNotification, SMSNotification, PushNotification
  - Abstract: send(recipient, message)
  - Each subclass uses different delivery mechanism
  - NotificationService routes to correct type
```

---

### 🔴 Advanced Exercises

**Exercise 7: Plugin Architecture**
```
Build: BasePlugin → LogPlugin, AuthPlugin, CachePlugin
  - Abstract: initialize(), execute(), cleanup()
  - PluginManager: register, enable, disable, execute_all
  - Lifecycle management via inheritance
```

**Exercise 8: ORM-Style Model**
```
Build: Model → User, Post, Comment
  - Model: save(), delete(), find(), all()
  - Each subclass defines its own fields
  - Polymorphic query: Model.find(id=1)
```

**Exercise 9: Inheritance vs Composition Refactor**
```
Take a deep inheritance hierarchy (4+ levels).
Refactor it to use composition instead.
Compare: flexibility, testability, readability.
Document the trade-offs.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Missing super().__init__
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        self.breed = breed
# Dog("Rex", "Lab").name    # AttributeError!

# Bug 2: Overriding without calling super
class Base:
    def setup(self):
        self.ready = True

class Child(Base):
    def setup(self):
        self.extra = True
# c = Child(); c.setup(); c.ready    # AttributeError!

# Bug 3: Mutable class attribute
class Team:
    members = []

class A(Team): pass
class B(Team): pass
A().members.append("x")
print(B().members)    # ['x'] — shared!

# Bug 4: isinstance with wrong type
class Animal: pass
class Dog(Animal): pass
d = Dog()
print(type(d) == Animal)       # False!
print(isinstance(d, Animal))   # True!
```

### Fixed Code ✅

```python
# Fix 1: Call super().__init__
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)    # ✅
        self.breed = breed

# Fix 2: Call super().setup()
class Child(Base):
    def setup(self):
        super().setup()    # ✅
        self.extra = True

# Fix 3: Instance attribute in __init__
class Team:
    def __init__(self):
        self.members = []    # ✅

# Fix 4: Use isinstance, not type()
print(isinstance(d, Animal))    # True ✅
```

---

## 8. 🌍 Real-World Application

### Inheritance in Python Ecosystem

| Library | Inheritance Usage |
|---------|------------------|
| **Django** | `models.Model` → `MyModel` |
| **Flask** | `flask.views.View` → custom views |
| **SQLAlchemy** | `Base` → ORM models |
| **unittest** | `TestCase` → custom test classes |
| **Exception hierarchy** | `Exception` → `ValueError` → custom |
| **collections.abc** | Abstract base classes for containers |
| **tkinter** | `Widget` → `Button`, `Label`, etc. |
| **logging** | `Handler` → `FileHandler`, `StreamHandler` |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is inheritance?

**Q2.** What is the syntax for creating a subclass?

**Q3.** What does `super()` do?

**Q4.** What is the difference between overriding and extending?

**Q5.** What does `isinstance()` check?

**Q6.** What is an abstract base class?

**Q7.** What is the MRO?

**Q8.** What happens if you forget `super().__init__()`?

**Q9.** When should you prefer composition over inheritance?

**Q10.** What does every Python class inherit from?

---

### 📋 Answer Key

> **Q1.** A mechanism where a child class gets all attributes and methods from a parent class, and can add or override them.

> **Q2.** `class Child(Parent):` — the parent class goes in parentheses.

> **Q3.** Returns a proxy object that delegates method calls to a parent class. Used to call parent methods from within an overridden method.

> **Q4.** Overriding: replacing a parent method entirely. Extending: calling `super().method()` first, then adding new behavior.

> **Q5.** Whether an object is an instance of a class OR any of its parent classes. `isinstance(dog, Animal)` returns `True` if Dog inherits from Animal.

> **Q6.** A class with `@abstractmethod` methods that cannot be instantiated directly. Forces subclasses to implement specific methods.

> **Q7.** Method Resolution Order — the order Python searches for methods: current class → parent → grandparent → ... → object.

> **Q8.** The parent's `__init__` doesn't run, so attributes set there (like `self.name`) won't exist on the child instance.

> **Q9.** When the relationship is "has-a" (not "is-a"), when you need flexibility to swap components, or when avoiding deep hierarchies.

> **Q10.** `object` — the ultimate base class of all Python classes.

---

## 🎬 End of Lesson Scene

**Harvey:** "One Employee base class. Three specialized roles. Change the email format once — all three update."

**Mike:** "And with abstract methods, you can't create an Employee directly — you MUST choose a role. The contract is enforced."

**Harvey:** "Next: `super()` in depth. How to call parent methods properly — especially when the hierarchy gets complex."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Base class and subclass syntax | ✅ |
| 2 | Inheriting methods and attributes | ✅ |
| 3 | Extending with new methods/attributes | ✅ |
| 4 | Overriding parent methods | ✅ |
| 5 | `super().__init__()` for parent construction | ✅ |
| 6 | `isinstance()` and `issubclass()` | ✅ |
| 7 | `object` — the universal base class | ✅ |
| 8 | Method Resolution Order (MRO) | ✅ |
| 9 | Abstract Base Classes (ABC) | ✅ |
| 10 | Inheritance vs Composition | ✅ |

---

> **Next Lesson →** Level 4, Lesson 4: *Use super() to Call Parent Methods in Python*

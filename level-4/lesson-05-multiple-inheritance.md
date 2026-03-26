# 🏛️ Level 4 – Advancing | Lesson 5: Implement Multiple Inheritance in Python

## *"The Merger"*

> **O'Reilly Skill Objective:** *Implement multiple inheritance to combine functionality from multiple classes.*

> **Previously Learned:** Inheritance, `super()`, MRO, ABC, `isinstance()`, cooperative `**kwargs`, decorators

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

*Harvey needs employees who are both billable AND manageable — two separate capability sets.*

```python
class Billable:
    def calculate_billing(self): ...
    
class Manageable:
    def assign_department(self): ...
```

**Louis:** "A Partner needs to be billable AND manageable. Which class do they inherit from?"

**Harvey:** "Both."

```python
class Partner(Employee, Billable, Manageable):
    pass    # Inherits from THREE classes!
```

**Mike:** "It's a merger. Two firms combining their strengths into one entity. But mergers can get messy — that's why Python has the MRO."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Multiple inheritance means one class inherits from MORE than one parent. Powerful — but dangerous if misused."

| Pattern | Description | Risk Level |
|---------|-------------|:----------:|
| **Mixins** | Small, focused behavior classes | 🟢 Low |
| **Interface-style** | Abstract base classes defining contracts | 🟢 Low |
| **Diamond inheritance** | Two parents share a common grandparent | 🟡 Medium |
| **Deep multiple** | Complex hierarchies with multiple paths | 🔴 High |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Multiple Inheritance ⭐

```python
class Billable:
    """Tracks billable hours and revenue."""
    
    def __init__(self):
        self.hours_billed = 0
        self.hourly_rate = 0
    
    def bill(self, hours):
        self.hours_billed += hours
        return hours * self.hourly_rate
    
    def total_revenue(self):
        return self.hours_billed * self.hourly_rate

class Manageable:
    """Tracks department and reports."""
    
    def __init__(self):
        self.department = None
        self.reports = []
    
    def assign_department(self, dept):
        self.department = dept
    
    def add_report(self, employee):
        self.reports.append(employee)

# Multiple inheritance: combines both
class Partner(Billable, Manageable):
    def __init__(self, name, rate):
        Billable.__init__(self)        # Initialize billing
        Manageable.__init__(self)      # Initialize management
        self.name = name
        self.hourly_rate = rate

harvey = Partner("Harvey Specter", 450)
harvey.bill(85)
harvey.assign_department("Corporate")
harvey.add_report("Mike Ross")

print(harvey.total_revenue())    # 38250
print(harvey.department)          # Corporate
print(harvey.reports)             # ['Mike Ross']
```

---

### 3.2 Mixins — The Recommended Pattern ⭐⭐

```python
# Mixins are small, focused classes that ADD behavior.
# Convention: name ends with "Mixin"
# Rules: no __init__, no state of their own, single responsibility

class SerializableMixin:
    """Adds JSON serialization to any class."""
    
    def to_dict(self):
        return {k: v for k, v in self.__dict__.items() 
                if not k.startswith('_')}
    
    def to_json(self):
        import json
        return json.dumps(self.to_dict(), default=str, indent=2)

class DisplayMixin:
    """Adds formatted display to any class."""
    
    def display(self):
        cls_name = type(self).__name__
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items()
                         if not k.startswith('_'))
        return f"{cls_name}({attrs})"
    
    def summary(self):
        return f"{type(self).__name__}: {getattr(self, 'name', 'Unknown')}"

class AuditMixin:
    """Adds audit tracking to any class."""
    
    def audit_log(self):
        if not hasattr(self, '_audit_trail'):
            self._audit_trail = []
        return list(self._audit_trail)
    
    def log_action(self, action):
        if not hasattr(self, '_audit_trail'):
            self._audit_trail = []
        from datetime import datetime
        self._audit_trail.append({
            "action": action,
            "timestamp": datetime.now().isoformat(),
        })

# Compose classes from mixins!
class Employee:
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id

class Partner(Employee, SerializableMixin, DisplayMixin, AuditMixin):
    def __init__(self, name, employee_id, equity):
        super().__init__(name, employee_id)
        self.equity = equity

harvey = Partner("Harvey Specter", "P001", 0.15)
harvey.log_action("Created partner record")
harvey.log_action("Assigned to Corporate")

print(harvey.display())
# Partner(name='Harvey Specter', employee_id='P001', equity=0.15)

print(harvey.to_json())
# {
#   "name": "Harvey Specter",
#   "employee_id": "P001",
#   "equity": 0.15
# }

print(harvey.audit_log())
# [{'action': 'Created partner record', 'timestamp': '...'}, ...]
```

---

### 3.3 The Diamond Problem — Python's Solution ⭐⭐

```python
# The "Diamond Problem": two parents share a common ancestor

class Employee:
    def __init__(self, name):
        self.name = name
        print(f"  Employee.__init__({name})")
    
    def role(self):
        return "Employee"

class Attorney(Employee):
    def __init__(self, name, bar_id):
        super().__init__(name)
        self.bar_id = bar_id
        print(f"  Attorney.__init__({name})")
    
    def role(self):
        return "Attorney"

class Manager(Employee):
    def __init__(self, name, department):
        super().__init__(name)
        self.department = department
        print(f"  Manager.__init__({name})")
    
    def role(self):
        return "Manager"

class Partner(Attorney, Manager):
    def __init__(self, name, bar_id, department):
        # ⚠️ Direct parent calls — Employee.__init__ called TWICE!
        # Attorney.__init__(self, name, bar_id)
        # Manager.__init__(self, name, department)
        
        # ✅ Use super() — Employee.__init__ called ONCE!
        super().__init__(name, bar_id)
        # But wait — Manager's department isn't set!
        self.department = department
        print(f"  Partner.__init__({name})")

#       Employee        ← The diamond top
#       /      \
#   Attorney   Manager  ← Two paths
#       \      /
#       Partner          ← The diamond bottom

# MRO resolves the diamond:
print(Partner.__mro__)
# Partner → Attorney → Manager → Employee → object
```

---

### 3.4 Cooperative Multiple Inheritance with `**kwargs` ⭐⭐⭐

```python
# The CORRECT way to handle the diamond problem:
# All classes use super() + **kwargs

class Employee:
    def __init__(self, name, **kwargs):
        super().__init__(**kwargs)    # Pass remaining to object
        self.name = name

class Attorney(Employee):
    def __init__(self, bar_id, **kwargs):
        super().__init__(**kwargs)    # Passes name to Employee
        self.bar_id = bar_id

class Manager(Employee):
    def __init__(self, department, **kwargs):
        super().__init__(**kwargs)    # Passes name to Employee
        self.department = department

class Partner(Attorney, Manager):
    def __init__(self, name, bar_id, department, equity):
        # All kwargs flow through the MRO chain
        super().__init__(
            name=name,
            bar_id=bar_id,
            department=department,
        )
        self.equity = equity

# MRO: Partner → Attorney → Manager → Employee → object
# Flow: Partner calls Attorney.__init__(bar_id=..., name=..., department=...)
#       Attorney takes bar_id, passes name + department to Manager
#       Manager takes department, passes name to Employee
#       Employee takes name, passes nothing to object

harvey = Partner("Harvey", bar_id="NY-12345", department="Corporate", equity=0.15)
print(harvey.name)         # Harvey
print(harvey.bar_id)       # NY-12345
print(harvey.department)   # Corporate
print(harvey.equity)       # 0.15

# Employee.__init__ is called exactly ONCE! ✅
```

---

### 3.5 Method Resolution Order (MRO) Deep Dive ⭐⭐

```python
# MRO uses the C3 Linearization algorithm
# Rules:
# 1. Children come before parents
# 2. Parents maintain their listed order
# 3. Each class appears only once

class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.__mro__)
# D → B → C → A → object

# More complex example:
class Base: 
    def who(self): return "Base"
    
class Left(Base): 
    def who(self): return "Left"
    
class Right(Base): 
    def who(self): return "Right"
    
class Bottom(Left, Right):
    pass    # Inherits Left.who because Left comes first in MRO

print(Bottom().who())    # "Left"
print(Bottom.__mro__)    # Bottom → Left → Right → Base → object

# Change parent order → changes MRO!
class Bottom2(Right, Left):
    pass

print(Bottom2().who())    # "Right"
print(Bottom2.__mro__)    # Bottom2 → Right → Left → Base → object

# Invalid MRO — Python raises TypeError!
# class X(A, B): pass    # If B inherits from A, can't put A before B
# TypeError: Cannot create a consistent MRO
```

---

### 3.6 Interface-Style ABCs (Abstract Mixins) ⭐

```python
from abc import ABC, abstractmethod

# Define capabilities as abstract interfaces
class Billable(ABC):
    @abstractmethod
    def calculate_billing(self):
        pass
    
    @abstractmethod
    def billing_summary(self):
        pass

class Schedulable(ABC):
    @abstractmethod
    def schedule(self, date, task):
        pass
    
    @abstractmethod
    def available_slots(self):
        pass

class Reportable(ABC):
    @abstractmethod
    def generate_report(self):
        pass

# Concrete class implements ALL interfaces
class Partner(Billable, Schedulable, Reportable):
    def __init__(self, name, rate):
        self.name = name
        self.rate = rate
        self._hours = 0
        self._schedule = []
    
    def calculate_billing(self):
        return self._hours * self.rate
    
    def billing_summary(self):
        return f"{self.name}: ${self.calculate_billing():,.2f}"
    
    def schedule(self, date, task):
        self._schedule.append({"date": date, "task": task})
    
    def available_slots(self):
        return 40 - len(self._schedule)
    
    def generate_report(self):
        return {
            "name": self.name,
            "billing": self.calculate_billing(),
            "scheduled_tasks": len(self._schedule),
        }

# Type checking works!
harvey = Partner("Harvey", 450)
print(isinstance(harvey, Billable))      # True
print(isinstance(harvey, Schedulable))   # True
print(isinstance(harvey, Reportable))    # True
```

---

### 3.7 Practical Mixin Examples

```python
import json
from datetime import datetime
from functools import total_ordering

# === Timestamp Mixin ===
class TimestampMixin:
    """Auto-track creation and modification times."""
    
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        original_init = cls.__init__
        
        def new_init(self, *args, **kwargs):
            original_init(self, *args, **kwargs)
            if not hasattr(self, '_created_at'):
                self._created_at = datetime.now()
            self._updated_at = datetime.now()
        
        cls.__init__ = new_init
    
    @property
    def created_at(self):
        return self._created_at
    
    @property
    def updated_at(self):
        return self._updated_at

# === Comparison Mixin ===
class ComparisonMixin:
    """Add all comparisons from a single 'comparison_key' property."""
    
    @property
    def comparison_key(self):
        raise NotImplementedError("Subclass must define comparison_key")
    
    def __eq__(self, other):
        if not isinstance(other, type(self)):
            return NotImplemented
        return self.comparison_key == other.comparison_key
    
    def __lt__(self, other):
        if not isinstance(other, type(self)):
            return NotImplemented
        return self.comparison_key < other.comparison_key
    
    def __le__(self, other):
        return self == other or self < other
    
    def __gt__(self, other):
        return not self <= other
    
    def __ge__(self, other):
        return not self < other

# === Validation Mixin ===
class ValidationMixin:
    """Add input validation to any class."""
    
    _validators = {}
    
    @classmethod
    def add_validator(cls, field, validator_func):
        if cls not in cls._validators:
            cls._validators[cls] = {}
        cls._validators[cls][field] = validator_func
    
    def validate(self):
        errors = []
        validators = self._validators.get(type(self), {})
        for field, validator in validators.items():
            value = getattr(self, field, None)
            if not validator(value):
                errors.append(f"Validation failed for '{field}': {value!r}")
        if errors:
            raise ValueError("\n".join(errors))
        return True

# === Compose! ===
class Client(ComparisonMixin, TimestampMixin):
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    
    @property
    def comparison_key(self):
        return self.revenue    # Sort by revenue

clients = [
    Client("Wayne", 2500000),
    Client("Stark", 800000),
    Client("Oscorp", 600000),
]

# Sorting works via ComparisonMixin!
for c in sorted(clients, reverse=True):
    print(f"  {c.name}: ${c.revenue:,} (created: {c.created_at:%H:%M:%S})")
```

---

### 3.8 `__init_subclass__` — Hook for Subclass Creation

```python
class RegisteredModel:
    """Auto-register all subclasses."""
    _registry = {}
    
    def __init_subclass__(cls, model_type=None, **kwargs):
        super().__init_subclass__(**kwargs)
        name = model_type or cls.__name__
        RegisteredModel._registry[name] = cls
        print(f"  📋 Registered model: {name}")
    
    @classmethod
    def get_model(cls, name):
        return cls._registry.get(name)
    
    @classmethod
    def all_models(cls):
        return dict(cls._registry)

class Client(RegisteredModel, model_type="client"):
    def __init__(self, name):
        self.name = name

class Case(RegisteredModel, model_type="case"):
    def __init__(self, title):
        self.title = title

class Invoice(RegisteredModel):    # Uses class name
    def __init__(self, amount):
        self.amount = amount

# 📋 Registered model: client
# 📋 Registered model: case
# 📋 Registered model: Invoice

print(RegisteredModel.all_models())
# {'client': <class 'Client'>, 'case': <class 'Case'>, 'Invoice': <class 'Invoice'>}

# Factory pattern:
ModelClass = RegisteredModel.get_model("client")
wayne = ModelClass("Wayne Enterprises")
```

---

### 3.9 When to Use (and Avoid) Multiple Inheritance

```python
"""
✅ USE MULTIPLE INHERITANCE FOR:

1. Mixins — small, focused behavior additions
   class MyModel(Model, SerializableMixin, AuditMixin): ...

2. Interface composition — combining ABCs
   class Partner(Billable, Schedulable, Employee): ...

3. Cooperative classes designed for MI
   Django class-based views, unittest.TestCase, etc.


❌ AVOID MULTIPLE INHERITANCE WHEN:

1. It creates confusion about which method is called
2. Two parents have overlapping, conflicting methods
3. Deep hierarchies with multiple paths
4. You can use composition instead

ALTERNATIVES:
  - Composition: self.serializer = JSONSerializer()
  - Protocols: typing.Protocol for structural typing
  - Decorators: @serializable, @auditable
"""
```

---

### 3.10 Solving Harvey's Problem — Mixin-Based Firm System

```python
# ============================================
# Pearson Specter Litt — Mixin Architecture
# Compose capabilities with multiple inheritance
# ============================================

from abc import ABC, abstractmethod
from datetime import datetime
import json

# === Core Mixins ===

class SerializableMixin:
    def to_dict(self):
        return {k: v for k, v in self.__dict__.items() 
                if not k.startswith('_')}
    
    def to_json(self):
        return json.dumps(self.to_dict(), default=str, indent=2)

class AuditMixin:
    def _ensure_audit(self):
        if not hasattr(self, '_audit'):
            self._audit = []
    
    def log_action(self, action):
        self._ensure_audit()
        self._audit.append({"action": action, "time": datetime.now().isoformat()})
    
    def audit_trail(self):
        self._ensure_audit()
        return list(self._audit)

class BillableMixin:
    def _ensure_billing(self):
        if not hasattr(self, '_billing_hours'):
            self._billing_hours = 0
    
    def bill_hours(self, hours):
        self._ensure_billing()
        self._billing_hours += hours
        rate = getattr(self, 'hourly_rate', 0)
        return hours * rate
    
    def total_billed(self):
        self._ensure_billing()
        rate = getattr(self, 'hourly_rate', 0)
        return self._billing_hours * rate

class NotifiableMixin:
    def _ensure_notifications(self):
        if not hasattr(self, '_notifications'):
            self._notifications = []
    
    def notify(self, message, priority="normal"):
        self._ensure_notifications()
        self._notifications.append({
            "message": message, "priority": priority,
            "timestamp": datetime.now().isoformat()
        })
    
    def unread_notifications(self):
        self._ensure_notifications()
        return [n for n in self._notifications if not n.get("read")]


# === Base Class ===

class Employee(ABC):
    def __init__(self, name, employee_id):
        self.name = name
        self.employee_id = employee_id
    
    @abstractmethod
    def role(self):
        pass
    
    def __repr__(self):
        return f"{type(self).__name__}('{self.name}')"


# === Composed Classes ===

class Partner(Employee, BillableMixin, AuditMixin, SerializableMixin, NotifiableMixin):
    def __init__(self, name, employee_id, hourly_rate, equity):
        super().__init__(name, employee_id)
        self.hourly_rate = hourly_rate
        self.equity = equity
    
    def role(self):
        return "Partner"

class Associate(Employee, BillableMixin, AuditMixin, SerializableMixin):
    def __init__(self, name, employee_id, hourly_rate, salary):
        super().__init__(name, employee_id)
        self.hourly_rate = hourly_rate
        self.salary = salary
    
    def role(self):
        return "Associate"

class Paralegal(Employee, AuditMixin, SerializableMixin):
    """Paralegals don't bill — no BillableMixin!"""
    def __init__(self, name, employee_id, salary):
        super().__init__(name, employee_id)
        self.salary = salary
    
    def role(self):
        return "Paralegal"


# === Demonstrate ===
print("=" * 65)
print(f"{'🏛️  MIXIN ARCHITECTURE':^65}")
print("=" * 65)

harvey = Partner("Harvey Specter", "P001", 450, 0.15)
mike = Associate("Mike Ross", "A001", 300, 200_000)
rachel = Paralegal("Rachel Zane", "PL001", 85_000)

# Billing (only Partner and Associate have BillableMixin)
harvey.bill_hours(85)
mike.bill_hours(62)
# rachel.bill_hours(40)    # ❌ AttributeError — Paralegal isn't Billable!

# Audit (everyone has AuditMixin)
harvey.log_action("Billed Wayne Enterprises 85 hours")
mike.log_action("Completed due diligence")
rachel.log_action("Filed discovery documents")

# Notifications (only Partner has NotifiableMixin)
harvey.notify("New case: Stark Industries merger", "high")
harvey.notify("Board meeting at 3pm")

# Serialization (everyone has SerializableMixin)
print(f"\n📋 HARVEY (Partner):")
print(f"  Capabilities: Billable ✅  Auditable ✅  Serializable ✅  Notifiable ✅")
print(f"  Total billed: ${harvey.total_billed():,.2f}")
print(f"  Audit trail: {len(harvey.audit_trail())} entries")
print(f"  Notifications: {len(harvey.unread_notifications())} unread")

print(f"\n📋 MIKE (Associate):")
print(f"  Capabilities: Billable ✅  Auditable ✅  Serializable ✅  Notifiable ❌")
print(f"  Total billed: ${mike.total_billed():,.2f}")

print(f"\n📋 RACHEL (Paralegal):")
print(f"  Capabilities: Billable ❌  Auditable ✅  Serializable ✅  Notifiable ❌")

# MRO
print(f"\n🔗 MRO for Partner:")
for cls in Partner.__mro__:
    print(f"  → {cls.__name__}")

# isinstance checks
print(f"\n🔍 Type checks:")
print(f"  Harvey is BillableMixin: {isinstance(harvey, BillableMixin)}")
print(f"  Rachel is BillableMixin: {isinstance(rachel, BillableMixin)}")
print(f"  Harvey is Employee: {isinstance(harvey, Employee)}")

# JSON output
print(f"\n📄 Harvey as JSON:")
print(harvey.to_json())

print(f"\n{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: The Diamond `__init__` Problem 🔥

```python
class A:
    def __init__(self):
        print("A.__init__")

class B(A):
    def __init__(self):
        A.__init__(self)    # Direct call
        print("B.__init__")

class C(A):
    def __init__(self):
        A.__init__(self)    # Direct call
        print("C.__init__")

class D(B, C):
    def __init__(self):
        B.__init__(self)
        C.__init__(self)

D()
# A.__init__    ← Called twice!
# B.__init__
# A.__init__    ← Called AGAIN!
# C.__init__

# ✅ Fix: use super() everywhere
class B(A):
    def __init__(self):
        super().__init__()    # Follows MRO
        print("B.__init__")

class C(A):
    def __init__(self):
        super().__init__()
        print("C.__init__")

class D(B, C):
    def __init__(self):
        super().__init__()

D()
# A.__init__    ← Called exactly ONCE ✅
# C.__init__
# B.__init__
```

---

### Pitfall 2: Method Name Conflicts

```python
class FileLogger:
    def log(self, message):
        print(f"[FILE] {message}")

class ConsoleLogger:
    def log(self, message):
        print(f"[CONSOLE] {message}")

class App(FileLogger, ConsoleLogger):
    pass

App().log("test")    # [FILE] test — ConsoleLogger.log is shadowed!

# ✅ Fix: explicit naming or composition
class App:
    def __init__(self):
        self.file_logger = FileLogger()
        self.console_logger = ConsoleLogger()
    
    def log(self, message):
        self.file_logger.log(message)
        self.console_logger.log(message)
```

---

### Pitfall 3: Mixin With `__init__`

```python
# ❌ Mixins with __init__ create dependency issues
class BadMixin:
    def __init__(self):
        self.mixin_data = []

class MyClass(Employee, BadMixin):
    def __init__(self, name):
        super().__init__(name)    # Might not call BadMixin.__init__!
        # self.mixin_data → AttributeError!

# ✅ Mixins should use lazy initialization
class GoodMixin:
    def _ensure_data(self):
        if not hasattr(self, '_mixin_data'):
            self._mixin_data = []
    
    def add_data(self, item):
        self._ensure_data()
        self._mixin_data.append(item)
```

---

### Pitfall 4: Too Many Parents

```python
# ❌ This is a code smell
class Monster(A, B, C, D, E, F, G):    # 7 parents!
    pass

# ✅ Limit to 2-4 parents maximum
# Use composition for the rest
class Clean(Employee, SerializableMixin):
    def __init__(self):
        super().__init__()
        self.notifier = NotificationService()    # Composition
        self.billing = BillingEngine()            # Composition
```

---

### Pitfall 5: Invalid MRO

```python
# Some inheritance patterns are impossible!
class A: pass
class B(A): pass

# ❌ C inherits A before B, but B also inherits A
# class C(A, B): pass    # TypeError: Cannot create consistent MRO!

# ✅ Put more specific classes first
class C(B, A): pass    # B before A ✅
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🤝 Analogy: Multiple Inheritance = Corporate Merger

```
SINGLE INHERITANCE:
  Pearson Hardman → Pearson Specter
  One parent, one child. Simple lineage.

MULTIPLE INHERITANCE (Mixins):
  Pearson Specter + Billing Division + Compliance Division
  = Pearson Specter Litt (Full Service Firm)
  Each division adds specific capabilities.

THE DIAMOND:
  Law School trained both Harvey and Louis.
  Both Harvey and Louis trained Mike.
  Mike has ONE legal education, not two!
  Python's MRO ensures: Law School.__init__ runs once.

DONNA'S RULE:
  "Multiple inheritance is a merger.
   Mixins are small acquisitions — manageable.
   Full mergers are complex — use sparingly.
   And never merge two firms that do the same thing."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Logging Mixins**
```
Create: ConsoleMixin, FileMixin, JSONMixin
Each adds a different logging method.
Compose: class FullLogger(ConsoleMixin, FileMixin, JSONMixin)
```

**Exercise 2: Animal Traits**
```
Create trait mixins: Swimmable, Flyable, Walkable
Compose: Duck(Swimmable, Flyable, Walkable)
         Penguin(Swimmable, Walkable)
         Eagle(Flyable, Walkable)
```

**Exercise 3: Data Class Mixins**
```
Create: EqualityMixin, HashableMixin, PrintableMixin
Apply to a Product class that gets __eq__, __hash__, __repr__
for free from the mixins.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Plugin System**
```
BasePlugin (ABC) + LifecycleMixin + ConfigMixin
  - Plugins: AuthPlugin, CachePlugin, LogPlugin
  - Each has initialize(), process(), cleanup()
  - Mixins add lifecycle hooks and config loading
```

**Exercise 5: Django-Style Views**
```
Build class-based views using mixins:
  LoginRequiredMixin + ListView → ProtectedListView
  PaginationMixin + FilterMixin + ListView → FilteredPaginatedList
```

**Exercise 6: Cooperative Diamond**
```
Build a full cooperative diamond with **kwargs:
  Entity → Character, NPC → QuestGiver
  All use super().__init__(**kwargs)
  Verify each __init__ runs exactly once.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Mixin Dependency Checker**
```
Build @requires_mixin(MixinClass):
  Raises TypeError at class creation if the required
  mixin is not in the MRO.

@requires_mixin(SerializableMixin)
class MyModel(Employee):    # TypeError: missing SerializableMixin!
```

**Exercise 8: Auto-Register Subclasses**
```
Build a metaclass or __init_subclass__ that:
  - Auto-registers all subclasses
  - Supports factory pattern: Model.create("client", data)
  - Tracks MRO for each registered class
```

**Exercise 9: Composition-to-MI Refactor**
```
Take a class using composition for 3 behaviors.
Refactor to use mixins instead.
Compare: code length, flexibility, testability.
Document which approach is better and why.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Diamond — parent init called twice
class Base:
    def __init__(self):
        print("Base init")
        self.count = 0
    
class Left(Base):
    def __init__(self):
        Base.__init__(self)    # Direct call
        self.count += 1

class Right(Base):
    def __init__(self):
        Base.__init__(self)    # Direct call — resets count!
        self.count += 10

class Bottom(Left, Right):
    def __init__(self):
        Left.__init__(self)
        Right.__init__(self)

b = Bottom()
print(b.count)    # 10 (not 11!) — Base reset count to 0 on second call!

# Bug 2: Method conflict
class A:
    def process(self): return "A"
class B:
    def process(self): return "B"
class C(A, B):
    pass
C().process()    # "A" — B.process is silently hidden!

# Bug 3: Invalid MRO
class X: pass
class Y(X): pass
# class Z(X, Y): pass    # TypeError!
```

### Fixed Code ✅

```python
# Fix 1: Use super() everywhere
class Left(Base):
    def __init__(self):
        super().__init__()
        self.count += 1

class Right(Base):
    def __init__(self):
        super().__init__()
        self.count += 10

class Bottom(Left, Right):
    def __init__(self):
        super().__init__()

b = Bottom()
print(b.count)    # 11 ✅ Base init runs once, both increments apply

# Fix 2: Explicit method resolution
class C(A, B):
    def process(self):
        return f"{A.process(self)} + {B.process(self)}"

# Fix 3: Correct order
class Z(Y, X): pass    # ✅ More specific first
```

---

## 8. 🌍 Real-World Application

### Multiple Inheritance in Python Ecosystem

| Framework | MI Usage |
|-----------|----------|
| **Django** | `LoginRequiredMixin + CreateView` for class-based views |
| **DRF** | `ListModelMixin + GenericAPIView` for API views |
| **SQLAlchemy** | Declarative base + mixins for models |
| **unittest** | `TestCase` combines multiple testing capabilities |
| **collections.abc** | `MutableSequence(Sequence)` chains |
| **socketserver** | `ThreadingMixIn + TCPServer` for threaded servers |
| **tkinter** | Widget mixins for UI components |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is multiple inheritance?

**Q2.** What is a mixin?

**Q3.** What is the diamond problem?

**Q4.** How does Python solve the diamond problem?

**Q5.** What is the MRO?

**Q6.** Why should mixins avoid `__init__`?

**Q7.** What is cooperative multiple inheritance?

**Q8.** How do you check a class's MRO?

**Q9.** When should you use composition instead of multiple inheritance?

**Q10.** What does `__init_subclass__` do?

---

### 📋 Answer Key

> **Q1.** A class inheriting from more than one parent class, combining their attributes and methods.

> **Q2.** A small, focused class that adds specific behavior without `__init__`. Designed to be combined with other classes via multiple inheritance. Convention: name ends with "Mixin".

> **Q3.** When two parent classes share a common ancestor, the ancestor's `__init__` could be called multiple times.

> **Q4.** The MRO (C3 linearization) ensures each class appears only once. `super()` follows the MRO, calling each `__init__` exactly once.

> **Q5.** Method Resolution Order — the order Python searches for methods across the inheritance chain. Computed by C3 linearization.

> **Q6.** Mixins with `__init__` create fragile dependency chains with `super()`. Use lazy initialization (`if not hasattr`) instead.

> **Q7.** All classes use `super()` and `**kwargs` to pass arguments through the MRO, ensuring every class gets properly initialized.

> **Q8.** `ClassName.__mro__` or `ClassName.mro()`.

> **Q9.** When parents have conflicting methods, when the relationship is "has-a" not "is-a", or when you need more than 3-4 parents.

> **Q10.** A hook called when a class is subclassed. Allows the parent to customize or register subclasses automatically.

---

## 🎬 End of Lesson Scene

**Harvey:** "So now an employee can be billable, auditable, serializable, and notifiable — all without code duplication?"

**Mike:** "Exactly. Each mixin adds one capability. Compose them like building blocks. Partners get billing + audit + notifications. Paralegals get just audit — no billing access."

**Harvey:** "The merger is complete. Next: `__slots__` — making these objects memory-efficient."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Basic multiple inheritance syntax | ✅ |
| 2 | Mixin pattern (small, focused, no `__init__`) | ✅ |
| 3 | The diamond problem and Python's solution | ✅ |
| 4 | Cooperative MI with `super()` + `**kwargs` | ✅ |
| 5 | MRO deep dive (C3 linearization) | ✅ |
| 6 | Interface-style abstract base classes | ✅ |
| 7 | Practical mixin examples (Serializable, Audit, etc.) | ✅ |
| 8 | `__init_subclass__` for auto-registration | ✅ |
| 9 | When to use MI vs composition | ✅ |
| 10 | Real-world MI in Django, DRF, stdlib | ✅ |

---

> **Next Lesson →** Level 4, Lesson 6: *Use `__slots__` to Create Memory-Efficient Python Classes*

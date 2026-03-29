# 🏛️ Level 5 – Innovating | Lesson 2: Use Metaclasses to Customize Class Creation in Python

## *"The Architects"*

> **O'Reilly Skill Objective:** *Use metaclasses to customize class creation and behavior.*

> **Previously Learned:** Classes, `__init__`, inheritance, `super()`, `__init_subclass__`, decorators, `type()`, `isinstance()`, ABC

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

*The firm needs every model class to automatically register itself, validate its fields, and enforce naming conventions — without any extra code in each class.*

```python
# ❌ Manual registration in every class
class Client:
    _registry = {}
    def __init_subclass__(cls): ...    # Repetitive!

class Case:
    _registry = {}
    def __init_subclass__(cls): ...    # Duplicated!
```

**Mike:** "What if we could control HOW classes are created? Not what instances do — what the classes themselves look like the moment they're defined?"

**Harvey:** "That's not writing classes. That's writing the blueprint for blueprints."

```python
# ✅ Metaclass: ONE place controls ALL class creation
class ModelMeta(type):
    registry = {}
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if name != 'Model':
            ModelMeta.registry[name] = cls
        return cls

class Model(metaclass=ModelMeta):
    pass

class Client(Model): pass    # Auto-registered!
class Case(Model): pass      # Auto-registered!
print(ModelMeta.registry)     # {'Client': <class 'Client'>, 'Case': <class 'Case'>}
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "A class creates instances. A metaclass creates CLASSES. It's the architect of architects."

```
metaclass (type)         creates →    class (Client)
class (Client)           creates →    instance (wayne)

type is to class   as   class is to instance
```

| Concept | What It Is |
|---------|-----------|
| **`type`** | The default metaclass — creates all classes |
| **Metaclass** | A class whose instances are themselves classes |
| **`__new__`** | Controls class CREATION (before `__init__`) |
| **`__init__`** | Initializes the class after creation |
| **`__call__`** | Controls what happens when the class is called (instance creation) |
| **`metaclass=`** | Keyword to specify a custom metaclass |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Everything Is an Object — Even Classes ⭐

```python
# In Python, classes are objects too!
# They are instances of 'type'

class Client:
    pass

# Client is an OBJECT — an instance of type
print(type(Client))       # <class 'type'>
print(type(42))           # <class 'int'>
print(type(int))          # <class 'type'>

# type is its own metaclass!
print(type(type))         # <class 'type'>

# ALL classes are instances of type:
print(isinstance(Client, type))    # True
print(isinstance(int, type))      # True
print(isinstance(str, type))      # True

# Hierarchy:
# type → creates → Client → creates → wayne (instance)
# type → creates → int    → creates → 42
```

---

### 3.2 Creating Classes Dynamically with `type()` ⭐⭐

```python
# type() has TWO uses:
# 1. type(obj)          → returns the type of an object
# 2. type(name, bases, dict) → CREATES a new class!

# Normal class definition:
class Attorney:
    role = "Attorney"
    def display(self):
        return f"{self.name} ({self.role})"

# IDENTICAL class created with type():
Attorney2 = type(
    'Attorney',                      # Class name
    (),                              # Base classes (tuple)
    {                                # Class dictionary (namespace)
        'role': 'Attorney',
        'display': lambda self: f"{self.name} ({self.role})",
    }
)

# They work exactly the same!
a = Attorney2()
a.name = "Harvey"
print(a.display())    # Harvey (Attorney)

# This is what Python does behind the scenes for every 'class' statement!
# class Foo: ...  ≡  Foo = type('Foo', (), {...})
```

---

### 3.3 Your First Metaclass ⭐⭐

```python
class MyMeta(type):
    """A metaclass that prints when a class is created."""
    
    def __new__(mcs, name, bases, namespace):
        print(f"🔨 Creating class: '{name}'")
        print(f"   Bases: {bases}")
        print(f"   Attributes: {list(namespace.keys())}")
        
        # Call type.__new__ to actually create the class
        cls = super().__new__(mcs, name, bases, namespace)
        return cls

# Use the metaclass
class Attorney(metaclass=MyMeta):
    role = "Attorney"
    def greet(self):
        return "Hello"

# Output (printed at CLASS CREATION time, not instantiation!):
# 🔨 Creating class: 'Attorney'
#    Bases: ()
#    Attributes: ['__module__', '__qualname__', 'role', 'greet']

# The metaclass runs when the CLASS is DEFINED — not when instances are created
a = Attorney()    # No metaclass output here!
```

---

### 3.4 `__new__` vs `__init__` vs `__call__` in Metaclasses ⭐⭐

```python
class VerboseMeta(type):
    
    def __new__(mcs, name, bases, namespace):
        """Called to CREATE the class object."""
        print(f"1. __new__: Creating class '{name}'")
        cls = super().__new__(mcs, name, bases, namespace)
        return cls
    
    def __init__(cls, name, bases, namespace):
        """Called to INITIALIZE the class object (after __new__)."""
        print(f"2. __init__: Initializing class '{name}'")
        super().__init__(name, bases, namespace)
    
    def __call__(cls, *args, **kwargs):
        """Called when the class is CALLED (to create an instance)."""
        print(f"3. __call__: Class '{cls.__name__}' called to create instance")
        instance = super().__call__(*args, **kwargs)
        return instance

class Person(metaclass=VerboseMeta):
    def __init__(self, name):
        self.name = name

# Class definition triggers __new__ and __init__:
# 1. __new__: Creating class 'Person'
# 2. __init__: Initializing class 'Person'

print("--- Now creating instance ---")
p = Person("Harvey")
# 3. __call__: Class 'Person' called to create instance

# Timeline:
# ┌─────────────────────────────────────────┐
# │ class Person(metaclass=VerboseMeta):    │
# │   → VerboseMeta.__new__()              │ ← Creates the class
# │   → VerboseMeta.__init__()             │ ← Initializes the class
# │                                         │
# │ p = Person("Harvey")                   │
# │   → VerboseMeta.__call__()             │ ← Class is called
# │     → Person.__new__()                 │ ← Creates the instance
# │     → Person.__init__()               │ ← Initializes the instance
# └─────────────────────────────────────────┘
```

---

### 3.5 Auto-Registration Metaclass ⭐⭐

```python
class RegisteredMeta(type):
    """Auto-register all classes created with this metaclass."""
    _registry = {}
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        
        # Don't register the base class itself
        if bases:    # Only register subclasses
            RegisteredMeta._registry[name] = cls
        
        return cls
    
    @classmethod
    def get(mcs, name):
        return mcs._registry.get(name)
    
    @classmethod
    def all(mcs):
        return dict(mcs._registry)

class Model(metaclass=RegisteredMeta):
    """Base model — not registered itself."""
    pass

class Client(Model):
    """Auto-registered!"""
    def __init__(self, name):
        self.name = name

class Case(Model):
    """Also auto-registered!"""
    def __init__(self, title):
        self.title = title

class Invoice(Model):
    """Also auto-registered!"""
    def __init__(self, amount):
        self.amount = amount

# All registered automatically!
print(RegisteredMeta.all())
# {'Client': <class 'Client'>, 'Case': <class 'Case'>, 'Invoice': <class 'Invoice'>}

# Factory pattern:
ClientClass = RegisteredMeta.get("Client")
wayne = ClientClass("Wayne Enterprises")
print(wayne.name)    # Wayne Enterprises
```

---

### 3.6 Validation Metaclass — Enforce Rules at Class Creation ⭐⭐

```python
class ValidatedMeta(type):
    """Enforce rules on class definitions."""
    
    def __new__(mcs, name, bases, namespace):
        # Skip validation for the base class
        if bases:
            # Rule 1: Class name must be CamelCase
            if not name[0].isupper():
                raise TypeError(f"Class name '{name}' must start with uppercase")
            
            # Rule 2: Must have a docstring
            if not namespace.get('__doc__'):
                raise TypeError(f"Class '{name}' must have a docstring")
            
            # Rule 3: Must define __repr__
            if '__repr__' not in namespace:
                # Auto-generate one!
                def auto_repr(self):
                    attrs = ', '.join(f'{k}={v!r}' for k, v in self.__dict__.items())
                    return f"{type(self).__name__}({attrs})"
                namespace['__repr__'] = auto_repr
            
            # Rule 4: All methods must have type hints (simplified check)
            import inspect
            for attr_name, attr_value in namespace.items():
                if callable(attr_value) and not attr_name.startswith('_'):
                    sig = inspect.signature(attr_value)
                    if sig.return_annotation is inspect.Parameter.empty:
                        print(f"  ⚠️ Warning: {name}.{attr_name}() missing return type hint")
        
        return super().__new__(mcs, name, bases, namespace)

class Model(metaclass=ValidatedMeta):
    """Base model."""
    pass

class Client(Model):
    """A firm client."""                    # ✅ Has docstring
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue

# class bad_client(Model):                 # ❌ TypeError: must start uppercase!
#     """Bad."""

# class NoDoc(Model):                      # ❌ TypeError: must have docstring!
#     pass

c = Client("Wayne", 2500000)
print(repr(c))    # Client(name='Wayne', revenue=2500000)  ← Auto-generated!
```

---

### 3.7 Singleton Metaclass ⭐

```python
class SingletonMeta(type):
    """Only ONE instance of each class can exist."""
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self, connection_string="sqlite:///firm.db"):
        self.connection_string = connection_string
        print(f"  📦 Database initialized: {connection_string}")

# Only creates ONE instance!
db1 = Database("sqlite:///firm.db")     # 📦 Database initialized
db2 = Database("sqlite:///other.db")    # NOT printed — returns existing!
db3 = Database()                         # Same instance again

print(db1 is db2)    # True
print(db1 is db3)    # True
print(id(db1) == id(db2) == id(db3))    # True
```

---

### 3.8 `__init_subclass__` — The Simpler Alternative ⭐⭐

```python
# For MOST use cases, __init_subclass__ is simpler than a metaclass!
# Added in Python 3.6 — specifically to avoid metaclass overuse

class Model:
    _registry = {}
    
    def __init_subclass__(cls, table_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        cls._table_name = table_name or cls.__name__.lower()
        Model._registry[cls._table_name] = cls
        print(f"  📋 Registered: {cls.__name__} → table '{cls._table_name}'")
    
    @classmethod
    def get_model(cls, table_name):
        return cls._registry.get(table_name)

class Client(Model, table_name="clients"):
    def __init__(self, name):
        self.name = name

class Case(Model, table_name="cases"):
    def __init__(self, title):
        self.title = title

class Invoice(Model):    # Uses default: 'invoice'
    def __init__(self, amount):
        self.amount = amount

# 📋 Registered: Client → table 'clients'
# 📋 Registered: Case → table 'cases'
# 📋 Registered: Invoice → table 'invoice'

print(Model._registry)

# WHEN TO USE WHICH:
"""
__init_subclass__   →  Registration, simple validation, adding attributes
                        Works for 90% of cases. Prefer this!

metaclass           →  Modify the class namespace before creation
                        Control instance creation (__call__)
                        Complex class transformations
                        Framework-level magic (Django, SQLAlchemy)
"""
```

---

### 3.9 `__prepare__` — Controlling the Class Namespace

```python
from collections import OrderedDict

class OrderedMeta(type):
    """Metaclass that remembers the order attributes were defined."""
    
    @classmethod
    def __prepare__(mcs, name, bases):
        """Called BEFORE the class body executes.
        Returns the object used as the class namespace."""
        return OrderedDict()    # Track definition order!
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, dict(namespace))
        cls._field_order = [
            key for key in namespace
            if not key.startswith('_') and not callable(namespace[key])
        ]
        return cls

class Form(metaclass=OrderedMeta):
    """Fields rendered in definition order."""
    pass

class ClientForm(Form):
    name = "text"
    revenue = "number"
    tier = "select"
    active = "checkbox"

print(ClientForm._field_order)
# ['name', 'revenue', 'tier', 'active'] — definition order preserved!
```

---

### 3.10 Solving the Firm's Problem — ORM-Style Metaclass

```python
# ============================================
# Pearson Specter Litt — ORM-Style Metaclass
# Auto-register, validate, and configure models
# ============================================

class Field:
    """Descriptor for model fields."""
    def __init__(self, field_type, required=True, default=None):
        self.field_type = field_type
        self.required = required
        self.default = default
        self.name = None    # Set by metaclass
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, self.default)
    
    def __set__(self, obj, value):
        if value is not None and not isinstance(value, self.field_type):
            raise TypeError(
                f"{self.name} must be {self.field_type.__name__}, "
                f"got {type(value).__name__}"
            )
        obj.__dict__[self.name] = value

class StringField(Field):
    def __init__(self, max_length=255, **kwargs):
        super().__init__(str, **kwargs)
        self.max_length = max_length

class IntField(Field):
    def __init__(self, **kwargs):
        super().__init__(int, **kwargs)

class FloatField(Field):
    def __init__(self, **kwargs):
        super().__init__(float, **kwargs)

class BoolField(Field):
    def __init__(self, **kwargs):
        super().__init__(bool, **kwargs)


class ModelMeta(type):
    """Metaclass that auto-configures model classes."""
    _registry = {}
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        
        # Skip the base Model class
        if name == 'Model':
            return cls
        
        # Collect field definitions
        fields = {}
        for key, value in namespace.items():
            if isinstance(value, Field):
                fields[key] = value
        
        cls._fields = fields
        cls._table_name = namespace.get('__tablename__', name.lower() + 's')
        
        # Register
        ModelMeta._registry[name] = cls
        
        # Auto-generate __repr__
        if '__repr__' not in namespace:
            def __repr__(self):
                field_strs = ', '.join(
                    f'{name}={getattr(self, name)!r}'
                    for name in self._fields
                )
                return f"{type(self).__name__}({field_strs})"
            cls.__repr__ = __repr__
        
        return cls


class Model(metaclass=ModelMeta):
    """Base model with ORM-like field definitions."""
    
    def __init__(self, **kwargs):
        for name, field in self._fields.items():
            if name in kwargs:
                setattr(self, name, kwargs[name])
            elif field.required and field.default is None:
                raise ValueError(f"Missing required field: {name}")
            else:
                setattr(self, name, field.default)
    
    def validate(self):
        errors = []
        for name, field in self._fields.items():
            value = getattr(self, name)
            if field.required and value is None:
                errors.append(f"{name} is required")
        if errors:
            raise ValueError(f"Validation errors: {', '.join(errors)}")
        return True
    
    def to_dict(self):
        return {name: getattr(self, name) for name in self._fields}
    
    @classmethod
    def from_dict(cls, data):
        return cls(**{k: v for k, v in data.items() if k in cls._fields})


# === Define models — metaclass handles everything! ===

class Client(Model):
    __tablename__ = 'clients'
    name = StringField(max_length=100)
    revenue = FloatField(required=False, default=0.0)
    tier = StringField(required=False, default="Bronze")
    active = BoolField(required=False, default=True)

class Attorney(Model):
    __tablename__ = 'attorneys'
    name = StringField(max_length=100)
    role = StringField(required=False, default="Associate")
    hourly_rate = FloatField()

class Case(Model):
    __tablename__ = 'cases'
    title = StringField(max_length=200)
    status = StringField(required=False, default="open")
    hours = FloatField(required=False, default=0.0)


# === Demonstrate ===
print("=" * 65)
print(f"{'🏛️  ORM-STYLE METACLASS DEMO':^65}")
print("=" * 65)

# Auto-registration
print(f"\n📋 Registered models: {list(ModelMeta._registry.keys())}")

# Field inspection
print(f"\n📊 Client fields:")
for name, field in Client._fields.items():
    req = "required" if field.required else "optional"
    print(f"  {name}: {field.field_type.__name__} ({req}, default={field.default})")

# Create instances
wayne = Client(name="Wayne Enterprises", revenue=2500000.0, tier="Platinum")
harvey = Attorney(name="Harvey Specter", hourly_rate=450.0, role="Partner")
merger = Case(title="Wayne Merger", status="open", hours=85.0)

print(f"\n🏢 Instances:")
print(f"  {repr(wayne)}")
print(f"  {repr(harvey)}")
print(f"  {repr(merger)}")

# Type validation
try:
    bad = Client(name=123)    # TypeError: name must be str!
except TypeError as e:
    print(f"\n❌ Type error caught: {e}")

# Serialization
print(f"\n📄 Wayne as dict: {wayne.to_dict()}")

# Factory
data = {"name": "Stark Industries", "revenue": 1800000.0, "tier": "Gold"}
stark = Client.from_dict(data)
print(f"📦 From dict: {repr(stark)}")

# Table names
print(f"\n🗄️ Table names:")
for name, cls in ModelMeta._registry.items():
    print(f"  {name} → '{cls._table_name}' ({len(cls._fields)} fields)")

print(f"\n{'='*65}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Metaclass Conflicts 🔥

```python
class MetaA(type): pass
class MetaB(type): pass

class A(metaclass=MetaA): pass
class B(metaclass=MetaB): pass

# ❌ Can't inherit from both — metaclass conflict!
# class C(A, B): pass
# TypeError: metaclass conflict

# ✅ Fix: create a combined metaclass
class MetaC(MetaA, MetaB): pass
class C(A, B, metaclass=MetaC): pass    # ✅
```

---

### Pitfall 2: Overusing Metaclasses

```python
# ❌ Using a metaclass for simple registration
class RegistryMeta(type):
    registry = {}
    def __new__(mcs, name, bases, ns):
        cls = super().__new__(mcs, name, bases, ns)
        RegistryMeta.registry[name] = cls
        return cls

# ✅ __init_subclass__ is simpler!
class Base:
    _registry = {}
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        Base._registry[cls.__name__] = cls

# ✅ Or even a class decorator!
registry = {}
def register(cls):
    registry[cls.__name__] = cls
    return cls

@register
class Client: pass
```

---

### Pitfall 3: Forgetting to Call `super().__new__()`

```python
class BrokenMeta(type):
    def __new__(mcs, name, bases, namespace):
        # ❌ Forgot to create the class!
        print(f"Creating {name}")
        # Missing: return super().__new__(mcs, name, bases, namespace)
        # Returns None!

# ✅ Always call super().__new__() and return the result
class FixedMeta(type):
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        return cls    # ✅
```

---

### Pitfall 4: Metaclass + Dataclass Interaction

```python
from dataclasses import dataclass

# ⚠️ Combining metaclasses with dataclasses can be tricky
class MyMeta(type): pass

# This works:
@dataclass
class Simple(metaclass=MyMeta):
    name: str
    value: int

# But complex metaclasses that modify __init__ may conflict
# with @dataclass's auto-generated __init__
```

---

### Pitfall 5: Debugging Metaclass Magic

```python
# ❌ Metaclass issues are HARD to debug because they happen at import time!
# If your metaclass raises an error, the CLASS DEFINITION fails

# ✅ Add clear error messages
class StrictMeta(type):
    def __new__(mcs, name, bases, namespace):
        if bases and '__doc__' not in namespace:
            raise TypeError(
                f"Class '{name}' must have a docstring. "
                f"Add a docstring to fix this error."    # Clear message!
            )
        return super().__new__(mcs, name, bases, namespace)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏗️ Analogy: Metaclass = The Firm's Founding Charter

```
INSTANCE = An individual attorney (Harvey, Mike)
CLASS = The role template (Partner, Associate)
METACLASS = The founding charter that defines how roles are created

Without metaclass:
  Each new role is defined ad-hoc.
  No consistency, no rules, no auto-registration.

With metaclass:
  The charter says: "Every new role MUST have a description,
  MUST be registered in the firm directory, and MUST follow
  naming conventions."
  
  When someone defines 'class Partner', the charter
  automatically enforces rules and registers it.

DONNA'S RULE:
  "A metaclass is the founding charter.
   It doesn't do the work — it defines
   HOW work gets organized.
   Use it sparingly. Most firms don't need one.
   But when you do, it's powerful."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Logging Metaclass**
```
Build a metaclass that prints every class creation:
  "Class 'Client' created with 3 methods and 2 attributes"
```

**Exercise 2: Dynamic Class Creation**
```
Use type() to create a class dynamically:
  MyClass = type('MyClass', (Base,), {'x': 10, 'greet': lambda self: 'hi'})
Verify it works like a normal class.
```

**Exercise 3: Singleton with `__init_subclass__`**
```
Implement the Singleton pattern using __init_subclass__
instead of a metaclass. Compare the two approaches.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Frozen Class Metaclass**
```
Build a metaclass that makes class attributes immutable
after class creation. (Like a frozen dataclass but for
the class itself, not instances.)
```

**Exercise 5: Interface Enforcer**
```
Build a metaclass that enforces interface contracts:
  class ISerializable(metaclass=InterfaceMeta):
      def to_json(self): ...
      def from_json(cls, data): ...
  
  class Client(ISerializable):
      pass    # TypeError: must implement to_json, from_json!
```

**Exercise 6: Auto-Properties**
```
Metaclass that auto-converts _name attributes into
@property with getter/setter:
  class Person(metaclass=AutoPropMeta):
      _name: str
      _age: int
  p = Person(); p.name = "Harvey"    # Uses auto-generated property
```

---

### 🔴 Advanced Exercises

**Exercise 7: Django-Style ORM**
```
Build a mini ORM with metaclass:
  class User(Model):
      name = CharField(max_length=100)
      email = CharField(max_length=200)
      age = IntegerField()
  
  User.objects.create(name="Harvey", email="h@psl.com", age=40)
  User.objects.filter(age__gt=30)
```

**Exercise 8: Abstract Method Enforcer**
```
Build your own version of abc.ABC without using abc:
  class MyABC(metaclass=AbstractMeta):
      @abstract
      def method(self): ...
  
  class Concrete(MyABC):
      pass    # TypeError: must implement method!
```

**Exercise 9: Metaclass vs Decorator Benchmark**
```
Implement the same functionality 3 ways:
  a) Metaclass
  b) __init_subclass__
  c) Class decorator
Compare: readability, flexibility, performance.
Document which to use when.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Metaclass __new__ doesn't return anything
class Meta(type):
    def __new__(mcs, name, bases, ns):
        super().__new__(mcs, name, bases, ns)    # Missing return!

# class Foo(metaclass=Meta): pass    # Foo is None!

# Bug 2: Metaclass conflict
class M1(type): pass
class M2(type): pass
class A(metaclass=M1): pass
class B(metaclass=M2): pass
# class C(A, B): pass    # TypeError!

# Bug 3: Infinite recursion in __call__
class BadMeta(type):
    def __call__(cls, *args, **kwargs):
        return cls(*args, **kwargs)    # Calls __call__ again! Infinite!

# Bug 4: Modifying namespace after class creation
class LateMeta(type):
    def __new__(mcs, name, bases, ns):
        cls = super().__new__(mcs, name, bases, ns)
        ns['extra'] = True    # ❌ Modifying ns after creation — no effect!
        return cls
```

### Fixed Code ✅

```python
# Fix 1: Return the class!
class Meta(type):
    def __new__(mcs, name, bases, ns):
        return super().__new__(mcs, name, bases, ns)    # ✅

# Fix 2: Combined metaclass
class M3(M1, M2): pass
class C(A, B, metaclass=M3): pass    # ✅

# Fix 3: Call super().__call__
class GoodMeta(type):
    def __call__(cls, *args, **kwargs):
        return super().__call__(*args, **kwargs)    # ✅

# Fix 4: Modify cls, not ns
class FixedMeta(type):
    def __new__(mcs, name, bases, ns):
        cls = super().__new__(mcs, name, bases, ns)
        cls.extra = True    # ✅ Set on the class object
        return cls
```

---

## 8. 🌍 Real-World Application

### Metaclasses in the Wild

| Framework | Metaclass Usage |
|-----------|----------------|
| **Django ORM** | `ModelBase` — auto-registers models, configures fields |
| **SQLAlchemy** | `DeclarativeMeta` — maps classes to tables |
| **abc.ABC** | `ABCMeta` — enforces abstract methods |
| **Enum** | `EnumMeta` — creates enum members |
| **Pydantic v1** | `ModelMetaclass` — field validation |
| **attrs** | Uses `__init_subclass__` (simpler approach) |
| **WTForms** | Form metaclass for field registration |
| **Protocol Buffers** | Generated metaclasses for message types |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is a metaclass?

**Q2.** What is the default metaclass for all Python classes?

**Q3.** How do you specify a custom metaclass?

**Q4.** What does `type(name, bases, dict)` do?

**Q5.** What is `__new__` in a metaclass?

**Q6.** What is `__call__` in a metaclass?

**Q7.** What is `__prepare__`?

**Q8.** When should you use `__init_subclass__` instead?

**Q9.** What is a metaclass conflict?

**Q10.** Name two real-world frameworks that use metaclasses.

---

### 📋 Answer Key

> **Q1.** A class whose instances are classes. It controls how classes are created, similar to how a class controls how instances are created.

> **Q2.** `type` — every class is an instance of `type` by default.

> **Q3.** `class MyClass(metaclass=MyMeta):` — the `metaclass=` keyword argument.

> **Q4.** Dynamically creates a new class with the given name, base classes, and namespace dictionary. This is what Python does behind every `class` statement.

> **Q5.** Called to CREATE the class object. Receives the metaclass, class name, bases, and namespace. Must return the new class.

> **Q6.** Called when the class is CALLED to create an instance. Controls instance creation (before `__new__` and `__init__` of the class itself).

> **Q7.** Called before the class body executes. Returns the object used as the class namespace (e.g., `OrderedDict` to track definition order).

> **Q8.** For simple tasks: registration, adding class attributes, basic validation. `__init_subclass__` covers ~90% of use cases without metaclass complexity.

> **Q9.** When inheriting from two classes with different metaclasses. Fix by creating a combined metaclass that inherits from both.

> **Q10.** Django ORM (`ModelBase`), SQLAlchemy (`DeclarativeMeta`), `abc.ABC` (`ABCMeta`), Python `Enum` (`EnumMeta`).

---

## 🎬 End of Lesson Scene

**Mike:** "So Django's `Model`, SQLAlchemy's `Base`, Python's `Enum` — they all use metaclasses under the hood?"

**Harvey:** "The blueprints that create blueprints. Most developers never need to write one — but understanding them means you understand how Python's class system actually works."

**Mike:** "And for 90% of cases, `__init_subclass__` or a class decorator is enough."

**Harvey:** "Next: CSV and JSON — structured data that every real application handles."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Classes as objects — instances of `type` | ✅ |
| 2 | Dynamic class creation with `type()` | ✅ |
| 3 | Custom metaclass basics | ✅ |
| 4 | `__new__` vs `__init__` vs `__call__` in metaclasses | ✅ |
| 5 | Auto-registration metaclass | ✅ |
| 6 | Validation metaclass | ✅ |
| 7 | Singleton metaclass | ✅ |
| 8 | `__init_subclass__` as simpler alternative | ✅ |
| 9 | `__prepare__` for namespace control | ✅ |
| 10 | ORM-style metaclass with field descriptors | ✅ |

---

> **Next Lesson →** Level 5, Lesson 3: *Handle CSV or JSON Files with Python Modules*

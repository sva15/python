# 🏛️ Level 2 – Applying | Lesson 8: Use Static Variables and Class Methods in Python

## *"The Firm's Playbook"*

> **O'Reilly Skill Objective:** *Know the purpose of static variables, class methods, and static methods.*

> **Previously Learned:** Variables, data types, functions, classes (`__init__`, `self`), objects, instance vs class attributes, `@property`, `getattr`/`setattr`, `__dict__`/`vars()`

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

*Monday morning. You're reviewing the firm's codebase when Jessica calls a meeting.*

**Jessica:** "Three problems. First, I need to know how many active attorneys we have — without querying every attorney object. Second, I need alternative ways to create client records — from CSV rows, from database rows, from API payloads. Third, I need utility functions that logically belong to a class but don't need instance data."

**Harvey:** "The first is a class variable. Shared state across all instances."

**Mike:** "The second is a class method. Alternate constructors that build objects from different sources."

**Harvey:** "The third is a static method. A plain function that lives inside a class for organizational purposes."

**Mike:** "Three decorators. Three purposes. One lesson."

```python
class Attorney:
    headcount = 0                       # Class variable — shared state
    
    def __init__(self, name, rate):      # Regular method — uses self
        self.name = name
        self.rate = rate
        Attorney.headcount += 1
    
    @classmethod                        # Class method — uses cls
    def from_csv(cls, csv_line):
        name, rate = csv_line.split(",")
        return cls(name.strip(), float(rate.strip()))
    
    @staticmethod                       # Static method — no self or cls
    def validate_rate(rate):
        return 100 <= rate <= 1000
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Think of it as three layers of the firm."

| Layer | Keyword | What It Knows | Example |
|-------|---------|--------------|---------|
| **Instance method** | `self` | Everything about ONE attorney | `self.name`, `self.rate` |
| **Class method** | `cls` | Everything about the Attorney CLASS | `cls.headcount`, creates new instances |
| **Static method** | *(nothing)* | Only what you pass in | Utility/validation functions |

**Harvey:** "An instance method is like your personal assistant — knows everything about you. A class method is like HR — knows about the firm and can hire new people. A static method is like a calculator on your desk — doesn't know anything about the firm, just does math."

| Feature | Instance Method | Class Method | Static Method |
|---------|----------------|-------------|---------------|
| Decorator | *(none)* | `@classmethod` | `@staticmethod` |
| First param | `self` (instance) | `cls` (class) | *(none)* |
| Can access instance data? | ✅ | ❌ | ❌ |
| Can access class data? | ✅ (via `self.__class__` or `ClassName`) | ✅ (via `cls`) | ❌ (unless passed in) |
| Can create new instances? | ✅ | ✅ (via `cls(...)`) | ❌ (unless class passed in) |
| Callable on class? | ❌ (needs an instance) | ✅ | ✅ |
| Callable on instance? | ✅ | ✅ | ✅ |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Class Variables (Static Variables) ⭐

**Mike:** "Class variables are attributes that belong to the CLASS itself, not to any individual instance. Every instance shares the same data."

```python
class Attorney:
    # Class variables — shared by ALL instances
    firm_name = "Pearson Specter Litt"
    headcount = 0
    all_attorneys = []
    max_rate = 1000
    
    def __init__(self, name, rate):
        # Instance variables — unique to each object
        self.name = name
        self.rate = rate
        
        # Update class-level state
        Attorney.headcount += 1
        Attorney.all_attorneys.append(self)

# Create instances
harvey = Attorney("Harvey", 450)
mike = Attorney("Mike", 375)
louis = Attorney("Louis", 425)

# Class variables are shared
print(Attorney.headcount)      # 3
print(Attorney.firm_name)      # Pearson Specter Litt
print(len(Attorney.all_attorneys))  # 3

# Accessible from instances too (via lookup chain)
print(harvey.firm_name)        # Pearson Specter Litt
print(mike.headcount)          # 3

# But instance variables are UNIQUE
print(harvey.name)             # Harvey
print(mike.name)               # Mike
```

**Common uses for class variables:**

| Use Case | Example |
|----------|---------|
| **Tracking instances** | `headcount`, `all_instances` list |
| **Shared constants** | `MAX_RATE`, `VALID_TITLES` |
| **Default configuration** | `default_rate = 350` |
| **Class-level settings** | `verbose = False`, `log_enabled = True` |
| **Registry patterns** | Mapping subclasses or instances |

---

### 3.2 Accessing Class Variables Correctly

```python
class Config:
    debug = False
    version = "2.0"
    
    def __init__(self, name):
        self.name = name

# ✅ Access via class name (preferred for clarity)
print(Config.debug)     # False

# ✅ Access via instance (Python looks up the chain)
c = Config("Production")
print(c.debug)           # False — found on the class

# ⚠️ Assignment via instance CREATES an instance variable!
c.debug = True           # Creates c.__dict__["debug"] = True
print(c.debug)           # True — instance variable (shadows class)
print(Config.debug)      # False — class variable unchanged!

# ✅ To modify the class variable, use the class name:
Config.debug = True
print(Config.debug)      # True — class variable changed
print(c.debug)           # True — but this is still the instance variable!
```

**The lookup diagram:**
```
obj.attr
  │
  ├─ 1. Check obj.__dict__ (instance attributes)
  │     Found? → Return it
  │     Not found? ↓
  │
  ├─ 2. Check type(obj).__dict__ (class attributes)
  │     Found? → Return it
  │     Not found? ↓
  │
  ├─ 3. Check parent classes (MRO)
  │     Found? → Return it
  │     Not found? ↓
  │
  └─ 4. AttributeError
```

---

### 3.3 Class Methods — `@classmethod` ⭐⭐

**Mike:** "Class methods receive the CLASS as their first argument (`cls`) instead of an instance (`self`). Their primary use: **alternate constructors**."

```python
class Client:
    def __init__(self, name, revenue, case_type):
        self.name = name
        self.revenue = revenue
        self.case_type = case_type
    
    # --- Alternate constructors ---
    
    @classmethod
    def from_csv(cls, csv_line):
        """Create a Client from a CSV string."""
        parts = csv_line.split(",")
        return cls(
            name=parts[0].strip(),
            revenue=float(parts[1].strip()),
            case_type=parts[2].strip()
        )
    
    @classmethod
    def from_dict(cls, data):
        """Create a Client from a dictionary."""
        return cls(
            name=data["name"],
            revenue=data.get("revenue", 0),
            case_type=data.get("case_type", "General")
        )
    
    @classmethod
    def from_database_row(cls, row):
        """Create a Client from a database tuple."""
        # row = (id, name, revenue, case_type, created_at)
        return cls(name=row[1], revenue=row[2], case_type=row[3])
    
    def __repr__(self):
        return f"Client('{self.name}', {self.revenue}, '{self.case_type}')"

# Different ways to create the SAME type of object
c1 = Client("Wayne Enterprises", 2500000, "Merger")             # Regular
c2 = Client.from_csv("Stark Industries, 800000, Patent")        # From CSV
c3 = Client.from_dict({"name": "Oscorp", "revenue": 600000})    # From dict
c4 = Client.from_database_row((1, "Dalton", 1200000, "Merger", "2025-01-01"))

print(c1)   # Client('Wayne Enterprises', 2500000, 'Merger')
print(c2)   # Client('Stark Industries', 800000.0, 'Patent')
print(c3)   # Client('Oscorp', 600000, 'General')
print(c4)   # Client('Dalton', 1200000, 'Merger')
```

---

### 3.4 Why `cls` Instead of the Class Name? ⭐

**Mike:** "Because of **inheritance**. Using `cls` ensures subclasses create the RIGHT type."

```python
class Client:
    def __init__(self, name, revenue):
        self.name = name
        self.revenue = revenue
    
    @classmethod
    def high_value(cls, name):
        """Create a high-value client."""
        return cls(name, 1000000)    # Uses cls, not Client!
    
    def __repr__(self):
        return f"{type(self).__name__}('{self.name}', {self.revenue})"

class VIPClient(Client):
    def __init__(self, name, revenue):
        super().__init__(name, revenue)
        self.concierge = True

# cls ensures the right type is created
regular = Client.high_value("Wayne")
vip = VIPClient.high_value("Stark")

print(regular)              # Client('Wayne', 1000000)
print(vip)                   # VIPClient('Stark', 1000000)
print(type(vip))             # <class 'VIPClient'> ← correct!
print(vip.concierge)         # True ← VIPClient.__init__ ran!

# If we had used Client(name, 1000000) instead of cls(name, 1000000):
# vip would have been a Client, not a VIPClient!
```

---

### 3.5 Class Methods for Registry / Factory Patterns

```python
class CaseHandler:
    """Registry of case handlers using class methods."""
    _registry = {}
    
    def __init_subclass__(cls, case_type=None, **kwargs):
        """Auto-register subclasses."""
        super().__init_subclass__(**kwargs)
        if case_type:
            CaseHandler._registry[case_type] = cls
    
    @classmethod
    def create(cls, case_type, *args, **kwargs):
        """Factory: create the right handler for a case type."""
        handler_class = cls._registry.get(case_type)
        if not handler_class:
            raise ValueError(f"Unknown case type: {case_type}")
        return handler_class(*args, **kwargs)
    
    @classmethod
    def available_types(cls):
        """List all registered case types."""
        return list(cls._registry.keys())

class MergerHandler(CaseHandler, case_type="Merger"):
    def __init__(self, client_name):
        self.client = client_name
    def process(self):
        return f"Processing merger for {self.client}"

class PatentHandler(CaseHandler, case_type="Patent"):
    def __init__(self, client_name):
        self.client = client_name
    def process(self):
        return f"Filing patent for {self.client}"

class LitigationHandler(CaseHandler, case_type="Litigation"):
    def __init__(self, client_name):
        self.client = client_name
    def process(self):
        return f"Litigation defense for {self.client}"

# Factory creates the right type automatically
print(CaseHandler.available_types())
# ['Merger', 'Patent', 'Litigation']

handler = CaseHandler.create("Merger", "Wayne Enterprises")
print(handler.process())        # Processing merger for Wayne Enterprises
print(type(handler).__name__)    # MergerHandler
```

---

### 3.6 Static Methods — `@staticmethod`

**Mike:** "Static methods don't receive `self` or `cls`. They're regular functions that live inside a class for organizational purposes."

```python
class BillingUtils:
    """Utility functions related to billing — organized in a class."""
    
    TAX_RATE = 0.09    # Class variable
    
    @staticmethod
    def calculate_total(hours, rate, tax_rate=None):
        """Calculate billing total with tax."""
        if tax_rate is None:
            tax_rate = BillingUtils.TAX_RATE
        subtotal = hours * rate
        return subtotal * (1 + tax_rate)
    
    @staticmethod
    def format_currency(amount, currency="USD"):
        """Format a number as currency."""
        symbols = {"USD": "$", "EUR": "€", "GBP": "£"}
        symbol = symbols.get(currency, currency + " ")
        return f"{symbol}{amount:,.2f}"
    
    @staticmethod
    def validate_hours(hours):
        """Validate billable hours."""
        if not isinstance(hours, (int, float)):
            return False, "Hours must be a number"
        if hours < 0:
            return False, "Hours cannot be negative"
        if hours > 24:
            return False, "Hours cannot exceed 24 per day"
        return True, "Valid"
    
    @staticmethod
    def parse_rate(rate_string):
        """Parse a rate string like '$450/hr' into a float."""
        cleaned = rate_string.replace("$", "").replace("/hr", "").replace(",", "")
        return float(cleaned)

# Call on the class (no instance needed!)
total = BillingUtils.calculate_total(85, 450)
print(BillingUtils.format_currency(total))    # $41,602.50

valid, msg = BillingUtils.validate_hours(25)
print(f"Valid: {valid}, Message: {msg}")       # Valid: False, Hours cannot exceed 24

rate = BillingUtils.parse_rate("$450/hr")
print(rate)   # 450.0
```

---

### 3.7 When to Use Each ⭐

| Question | Answer |
|----------|--------|
| Does it need instance data (`self`)? | → **Instance method** |
| Does it need the class itself (`cls`)? | → **`@classmethod`** |
| Does it need neither? | → **`@staticmethod`** |
| Is it an alternate constructor? | → **`@classmethod`** |
| Is it a utility/helper function? | → **`@staticmethod`** |
| Could it be a standalone function? | → **`@staticmethod`** (if it logically belongs to the class) |

```python
class Attorney:
    firm = "PSL"
    headcount = 0
    
    def __init__(self, name, rate):
        self.name = name                    # Needs self → instance method
        self.rate = rate
        Attorney.headcount += 1
    
    def bill(self, hours):                  # Needs self.rate → instance method
        return hours * self.rate
    
    @classmethod
    def from_csv(cls, line):                # Creates new instance → classmethod
        name, rate = line.split(",")
        return cls(name.strip(), float(rate))
    
    @classmethod
    def get_headcount(cls):                 # Accesses cls.headcount → classmethod
        return cls.headcount
    
    @staticmethod
    def validate_rate(rate):                # Just math, no self/cls → staticmethod
        return 100 <= rate <= 1000
    
    @staticmethod
    def format_name(first, last):           # Just string ops → staticmethod
        return f"{first} {last}"
```

---

### 3.8 Class Variables as Configuration

```python
class APIClient:
    """Configurable API client using class variables."""
    
    # Class-level configuration
    base_url = "https://api.pearsonspecter.com"
    timeout = 30
    max_retries = 3
    api_version = "v2"
    debug = False
    
    def __init__(self, api_key):
        self.api_key = api_key
        self.request_count = 0
    
    @classmethod
    def configure(cls, **settings):
        """Update class-level configuration."""
        for key, value in settings.items():
            if hasattr(cls, key):
                setattr(cls, key, value)
            else:
                raise ValueError(f"Unknown setting: {key}")
    
    @classmethod
    def get_config(cls):
        """Get current configuration."""
        return {
            "base_url": cls.base_url,
            "timeout": cls.timeout,
            "max_retries": cls.max_retries,
            "api_version": cls.api_version,
            "debug": cls.debug,
        }
    
    def request(self, endpoint, method="GET"):
        """Make an API request."""
        url = f"{self.base_url}/{self.api_version}/{endpoint}"
        self.request_count += 1
        if self.debug:
            print(f"  [{method}] {url} (timeout: {self.timeout}s)")
        return {"url": url, "status": 200}

# Configure at the class level — affects ALL instances
APIClient.configure(debug=True, timeout=60)

client1 = APIClient("key_123")
client2 = APIClient("key_456")

# Both clients share the same configuration
client1.request("clients")     # Uses debug=True, timeout=60
client2.request("billing")     # Same config!

print(APIClient.get_config())
```

---

### 3.9 Solving Jessica's Problem — Full Firm System

```python
# ============================================
# Pearson Specter Litt — Complete Class System
# Class variables + @classmethod + @staticmethod
# ============================================

class FirmMember:
    """Base class for all firm members."""
    
    # Class variables — shared state
    _all_members = []
    _id_counter = 0
    firm_name = "Pearson Specter Litt"
    
    def __init__(self, name, title, department):
        FirmMember._id_counter += 1
        self.id = FirmMember._id_counter
        self.name = name
        self.title = title
        self.department = department
        FirmMember._all_members.append(self)
    
    @classmethod
    def total_count(cls):
        return len(cls._all_members)
    
    @classmethod
    def find_by_department(cls, dept):
        return [m for m in cls._all_members if m.department == dept]
    
    @classmethod
    def find_by_id(cls, member_id):
        return next((m for m in cls._all_members if m.id == member_id), None)
    
    @staticmethod
    def format_title(name, title):
        return f"{name}, {title}"
    
    def __repr__(self):
        return f"{type(self).__name__}('{self.name}', '{self.title}')"


class Attorney(FirmMember):
    """Attorney with billing capabilities."""
    
    _attorneys = []
    VALID_TITLES = {"Associate", "Senior Associate", "Junior Partner", 
                    "Partner", "Senior Partner", "Managing Partner", "Name Partner"}
    MIN_RATE = 200
    MAX_RATE = 800
    
    def __init__(self, name, title, rate, specialty="General"):
        super().__init__(name, title, "Legal")
        self.rate = rate
        self.specialty = specialty
        self.cases = []
        self.total_billed = 0
        Attorney._attorneys.append(self)
    
    @classmethod
    def from_csv(cls, csv_line):
        """Alternate constructor: from CSV."""
        parts = [p.strip() for p in csv_line.split(",")]
        return cls(name=parts[0], title=parts[1], rate=float(parts[2]),
                   specialty=parts[3] if len(parts) > 3 else "General")
    
    @classmethod
    def from_dict(cls, data):
        """Alternate constructor: from dictionary."""
        return cls(**data)
    
    @classmethod
    def get_attorneys(cls):
        """Get all attorneys."""
        return list(cls._attorneys)
    
    @classmethod
    def by_specialty(cls, specialty):
        """Find attorneys by specialty."""
        return [a for a in cls._attorneys if a.specialty == specialty]
    
    @classmethod
    def top_billers(cls, n=3):
        """Get top N billing attorneys."""
        return sorted(cls._attorneys, key=lambda a: a.total_billed, reverse=True)[:n]
    
    @staticmethod
    def validate_rate(rate):
        """Validate a billing rate."""
        if not isinstance(rate, (int, float)):
            return False, "Rate must be a number"
        if rate < Attorney.MIN_RATE:
            return False, f"Rate below minimum (${Attorney.MIN_RATE})"
        if rate > Attorney.MAX_RATE:
            return False, f"Rate exceeds maximum (${Attorney.MAX_RATE})"
        return True, "Valid"
    
    @staticmethod
    def calculate_billing(hours, rate, discount=0):
        """Calculate billing amount."""
        return hours * rate * (1 - discount)
    
    def bill(self, client_name, hours):
        """Bill a client (instance method — needs self)."""
        amount = self.calculate_billing(hours, self.rate)
        self.total_billed += amount
        self.cases.append({"client": client_name, "hours": hours, "amount": amount})
        return amount


class Client(FirmMember):
    """Client with revenue tracking."""
    
    _clients = []
    TIER_THRESHOLDS = [(1000000, "💎 Platinum"), (500000, "🥇 Gold"), 
                       (100000, "🥈 Silver"), (0, "🥉 Bronze")]
    
    def __init__(self, name, revenue, case_type="General"):
        super().__init__(name, "Client", "External")
        self.revenue = revenue
        self.case_type = case_type
        Client._clients.append(self)
    
    @classmethod
    def from_csv(cls, csv_line):
        parts = [p.strip() for p in csv_line.split(",")]
        return cls(name=parts[0], revenue=float(parts[1]),
                   case_type=parts[2] if len(parts) > 2 else "General")
    
    @classmethod
    def total_revenue(cls):
        return sum(c.revenue for c in cls._clients)
    
    @classmethod
    def by_tier(cls, tier_name):
        return [c for c in cls._clients if tier_name in c.tier]
    
    @property
    def tier(self):
        for threshold, label in Client.TIER_THRESHOLDS:
            if self.revenue >= threshold:
                return label
        return "🥉 Bronze"


# ============================================
# BUILD THE FIRM
# ============================================
print("=" * 60)
print(f"{'🏛️  BUILDING PEARSON SPECTER LITT':^60}")
print("=" * 60)

# Regular constructors
harvey = Attorney("Harvey Specter", "Senior Partner", 450, "Corporate")
louis = Attorney("Louis Litt", "Name Partner", 425, "Financial")

# Alternate constructor: from CSV
mike = Attorney.from_csv("Mike Ross, Senior Associate, 400, IP Law")
katrina = Attorney.from_csv("Katrina Bennett, Associate, 350, Corporate")

# Alternate constructor: from dict
rachel = Attorney.from_dict({
    "name": "Rachel Zane", "title": "Associate", 
    "rate": 375, "specialty": "IP Law"
})

# Create clients (various methods)
wayne = Client("Wayne Enterprises", 2500000, "Merger")
stark = Client.from_csv("Stark Industries, 800000, Patent")
oscorp = Client.from_csv("Oscorp, 600000, Litigation")
dalton = Client("Dalton Industries", 1200000, "Merger")
luthor = Client("Luthor Corp", 150000, "Corporate")

# Simulate billing
harvey.bill("Wayne Enterprises", 85)
harvey.bill("Dalton Industries", 70)
mike.bill("Stark Industries", 62)
louis.bill("Oscorp", 55)
katrina.bill("Luthor Corp", 28)

# ============================================
# DASHBOARD
# ============================================
print(f"\n📊 FIRM OVERVIEW")
print(f"  Total Members: {FirmMember.total_count()}")
print(f"  Attorneys: {len(Attorney.get_attorneys())}")
print(f"  Clients: {len(Client._clients)}")
print(f"  Total Client Revenue: ${Client.total_revenue():,.2f}")

print(f"\n👔 ATTORNEY ROSTER")
for atty in Attorney.get_attorneys():
    valid, _ = Attorney.validate_rate(atty.rate)
    status = "✅" if valid else "⚠️"
    print(f"  {status} {atty.name:<20} {atty.title:<18} ${atty.rate}/hr  "
          f"{atty.specialty}  Billed: ${atty.total_billed:,.2f}")

print(f"\n🏆 TOP BILLERS")
for i, atty in enumerate(Attorney.top_billers(), 1):
    print(f"  {i}. {atty.name}: ${atty.total_billed:,.2f}")

print(f"\n💼 CLIENTS BY TIER")
for threshold, tier_name in Client.TIER_THRESHOLDS:
    tier_clients = Client.by_tier(tier_name.split()[-1])
    if tier_clients:
        names = ", ".join(c.name for c in tier_clients)
        print(f"  {tier_name}: {names}")

print(f"\n🔍 BY SPECIALTY")
for spec in sorted({a.specialty for a in Attorney.get_attorneys()}):
    attorneys = Attorney.by_specialty(spec)
    print(f"  {spec}: {', '.join(a.name for a in attorneys)}")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Modifying Class Variables via Instances

```python
class Settings:
    theme = "light"

s1 = Settings()
s2 = Settings()

# ❌ This creates an INSTANCE variable, doesn't change class!
s1.theme = "dark"
print(s1.theme)        # "dark" — instance attribute
print(s2.theme)        # "light" — class attribute unchanged!
print(Settings.theme)  # "light" — class attribute unchanged!

# ✅ Modify via class name
Settings.theme = "dark"
print(s2.theme)        # "dark" — now ALL instances see the change
```

---

### Pitfall 2: Mutable Class Variables Are Shared! 🔥

```python
class Team:
    members = []    # ⚠️ Shared mutable class variable!
    
    def add(self, name):
        self.members.append(name)

t1 = Team()
t2 = Team()
t1.add("Harvey")
print(t2.members)    # ['Harvey'] ← t2 sees Harvey!

# ✅ Fix Option A: Initialize in __init__
class Team:
    def __init__(self):
        self.members = []    # Instance variable — independent!

# ✅ Fix Option B: Keep as class variable if intended to be shared
class Team:
    _all_members = []    # Intentionally shared registry
    
    @classmethod
    def add_member(cls, name):
        cls._all_members.append(name)
```

---

### Pitfall 3: `@staticmethod` Can't Access Class or Instance Data

```python
class Calculator:
    precision = 2    # Class variable
    
    @staticmethod
    def add(a, b):
        # ❌ Can't access self.anything or cls.precision!
        return round(a + b, 2)    # Must hardcode 2
    
    @classmethod
    def add_precise(cls, a, b):
        # ✅ Can access cls.precision!
        return round(a + b, cls.precision)

# If you need class data, use @classmethod instead!
```

---

### Pitfall 4: Forgetting `cls()` in Class Methods

```python
class Client:
    def __init__(self, name):
        self.name = name
    
    @classmethod
    def default(cls):
        # ❌ Using the class name directly — breaks inheritance!
        return Client("Default Client")   # Always creates Client, never subclass
    
    @classmethod
    def default_correct(cls):
        # ✅ Using cls — works with inheritance!
        return cls("Default Client")

class VIPClient(Client):
    pass

c = VIPClient.default()           # Returns Client (wrong!)
c = VIPClient.default_correct()   # Returns VIPClient (correct!)
```

---

### Pitfall 5: Testing Difficulties with Class State

```python
class Counter:
    count = 0
    
    def __init__(self):
        Counter.count += 1

# ⚠️ Class state persists between tests!
# Test 1
c1 = Counter()
assert Counter.count == 1    # ✅ Passes

# Test 2 — FAILS because count is still 1 from Test 1
c2 = Counter()
assert Counter.count == 1    # ❌ It's 2!

# ✅ Fix: Reset class state in test setup
Counter.count = 0
c2 = Counter()
assert Counter.count == 1    # ✅ Passes
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🏢 Analogy: The Firm Building

```
THE FIRM BUILDING:
┌────────────────────────────────────┐
│  🏛️ FIRM-LEVEL (Class Variables)  │
│  firm_name = "PSL"                │
│  headcount = 5                    │
│  ALL attorneys share this floor   │
├────────────────────────────────────┤
│  📋 HR OFFICE (@classmethod)      │
│  Can hire new people (cls(...))   │
│  Knows firm stats (cls.headcount) │
│  from_csv(), from_dict()          │
├────────────────────────────────────┤
│  🔧 UTILITY ROOM (@staticmethod)  │
│  Calculators, validators          │
│  No access to firm data           │
│  validate_rate(), format_currency │
├────────────────────────────────────┤
│  🚪 INDIVIDUAL OFFICES (instance) │
│  harvey.name = "Harvey"           │
│  mike.rate = 375                  │
│  Each person has their own data   │
└────────────────────────────────────┘
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Instance Counter**
```
Create a class Product with:
  - Class variable: count (tracks total products)
  - Class variable: all_products (list of all)
  - Instance variables: name, price

Create 5 products, then:
  a) Print total count
  b) List all product names
  c) Calculate average price (class method)
```

**Exercise 2: Static Utilities**
```
Create a class MathUtils with static methods:
  a) is_prime(n)
  b) factorial(n)
  c) gcd(a, b)
  d) lcm(a, b)

Call each WITHOUT creating an instance.
```

**Exercise 3: Alternate Constructors**
```
Create a class Date with: year, month, day

Add class methods:
  a) from_string("2025-03-14")
  b) from_timestamp(1710403200)
  c) today() — returns today's date
  d) from_dict({"year": 2025, "month": 3, "day": 14})
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Configuration System**
```
Build a class AppConfig that:
  - Has class variables for all settings
  - @classmethod configure(**settings) to update
  - @classmethod reset() to restore defaults
  - @classmethod load_from_file(path)
  - @staticmethod validate_setting(key, value)
  - Tracks all changes with timestamps
```

**Exercise 5: Object Registry**
```
Build a RegisteredModel base class that:
  - Auto-registers every instance in a class-level registry
  - @classmethod find_by_id(id) → instance
  - @classmethod find_all(**criteria) → filtered list
  - @classmethod count() → total instances
  - @classmethod clear_registry() → for testing

Subclass it with User and Product.
```

**Exercise 6: Unit Converter**
```
Build a class Temperature with:
  - Instance: value, unit ("C", "F", "K")
  - @classmethod from_celsius(value)
  - @classmethod from_fahrenheit(value)
  - @classmethod from_kelvin(value)
  - @staticmethod c_to_f(celsius), f_to_c(f), etc.
  - Instance method: to(unit) → converted Temperature

t = Temperature.from_celsius(100)
print(t.to("F"))    # Temperature(212.0, 'F')
```

---

### 🔴 Advanced Exercises

**Exercise 7: Singleton Pattern**
```
Implement a Singleton using @classmethod:

class Database:
    _instance = None
    
    @classmethod
    def get_instance(cls):
        ...  # Returns the single instance (creates if needed)
    
    @classmethod
    def reset(cls):
        ...  # For testing

db1 = Database.get_instance()
db2 = Database.get_instance()
assert db1 is db2    # Same instance!
```

**Exercise 8: Plugin System**
```
Build a plugin framework:

class Plugin:
    _plugins = {}
    
    @classmethod
    def register(cls, name):  → decorator
    @classmethod
    def get(cls, name): → plugin class
    @classmethod
    def list_all(cls): → all registered names
    @staticmethod
    def validate_plugin(plugin_class): → bool

@Plugin.register("formatter")
class FormatterPlugin(Plugin):
    ...
```

**Exercise 9: Metaclass-Light**
```
Build a class that tracks ALL subclasses:

class Tracked:
    _subclasses = {}
    
    def __init_subclass__(cls, **kwargs):
        Tracked._subclasses[cls.__name__] = cls
    
    @classmethod
    def get_subclass(cls, name): → class
    @classmethod
    def create(cls, class_name, *args, **kwargs): → instance of subclass
    @classmethod
    def hierarchy(): → dict showing class tree
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Instance variable shadows class variable
class Config:
    verbose = False
    def toggle(self):
        self.verbose = not self.verbose    # Creates instance attr!

c1 = Config()
c2 = Config()
c1.toggle()
print(c2.verbose)              # Expected: True

# Bug 2: @classmethod without cls
class Builder:
    @classmethod
    def create(name):           # Missing cls!
        return Builder(name)

# Bug 3: @staticmethod trying to use self
class Validator:
    min_value = 0
    @staticmethod
    def check(value):
        return value >= self.min_value   # Can't use self!

# Bug 4: Class variable list shared between instances
class Logger:
    entries = []
    def log(self, msg):
        self.entries.append(msg)

# Bug 5: Using class name instead of cls
class Base:
    @classmethod
    def create(cls):
        return Base()    # Ignores subclasses!
```

### Fixed Code ✅

```python
# Fix 1: Use the class name to modify class variables
class Config:
    verbose = False
    def toggle(self):
        Config.verbose = not Config.verbose  # ✅ Modifies class

# Fix 2: @classmethod MUST have cls as first parameter
class Builder:
    @classmethod
    def create(cls, name):      # ✅ cls is first
        return cls(name)

# Fix 3: @staticmethod can't use self — access via class or parameter
class Validator:
    min_value = 0
    @staticmethod
    def check(value, min_value=0):       # ✅ Pass as parameter
        return value >= min_value
    # OR use @classmethod:
    @classmethod
    def check_v2(cls, value):            # ✅ Access via cls
        return value >= cls.min_value

# Fix 4: Initialize mutable variables in __init__
class Logger:
    def __init__(self):
        self.entries = []    # ✅ Instance variable
    def log(self, msg):
        self.entries.append(msg)

# Fix 5: Use cls() to respect inheritance
class Base:
    @classmethod
    def create(cls):
        return cls()          # ✅ Correct type for subclasses
```

---

## 8. 🌍 Real-World Application

### Where These Patterns Are Used

| Pattern | Example |
|---------|---------|
| **Class variable** | Django `Model.objects` (shared manager for all instances) |
| **@classmethod alt constructor** | `datetime.now()`, `dict.fromkeys()`, `int.from_bytes()` |
| **@classmethod factory** | `json.JSONDecoder` subclasses |
| **@staticmethod** | Validators, formatters, converters in utility classes |
| **Registry** | Flask route registration, pytest fixtures |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is a class variable? How is it different from an instance variable?

**Q2.** What happens when you do `self.class_var = value` instead of `ClassName.class_var = value`?

**Q3.** What does `@classmethod` do? What is `cls`?

**Q4.** What is the primary use case for `@classmethod`?

**Q5.** Why should you use `cls(...)` instead of `ClassName(...)` inside a classmethod?

**Q6.** What does `@staticmethod` do? When should you use it?

**Q7.** Can a static method access class variables? How?

**Q8.** Write a classmethod `from_string` for a `Color` class that parses `"rgb(255, 0, 128)"`.

**Q9.** Why are mutable class variables dangerous?

**Q10.** In what order should you try: instance method → classmethod → staticmethod?

---

### 📋 Answer Key

> **Q1.** A class variable belongs to the class and is shared by all instances. An instance variable belongs to a specific object.

> **Q2.** It creates a new instance variable that **shadows** the class variable. The class variable is unchanged.

> **Q3.** It makes a method receive the class (`cls`) as its first argument instead of an instance (`self`).

> **Q4.** **Alternate constructors** — creating objects from different data sources.

> **Q5.** Because `cls` might be a subclass. `cls(...)` creates the correct type; `ClassName(...)` always creates the parent.

> **Q6.** It creates a method that receives neither `self` nor `cls`. Use for utility functions that logically belong to the class.

> **Q7.** Not directly (no `cls` or `self`). It must reference the class by name: `ClassName.var`.

> **Q8.**
```python
@classmethod
def from_string(cls, s):
    # "rgb(255, 0, 128)"
    values = s.replace("rgb(", "").replace(")", "").split(",")
    r, g, b = (int(v.strip()) for v in values)
    return cls(r, g, b)
```

> **Q9.** All instances share the same object. If one instance modifies it (e.g., `.append()`), all instances see the change.

> **Q10.** Default to instance method. Use `@classmethod` when you need the class (alt constructors, class-level operations). Use `@staticmethod` when you need neither.

---

## 🎬 End of Lesson Scene

**Jessica:** "Attorney headcount: real-time. Client ingestion: from CSV, API, and database. Utility functions: organized and accessible without instantiation."

**Harvey:** "Three tools, three jobs. Class variables for shared state. Class methods for alternate construction. Static methods for utilities."

**Mike:** "Next: mutability and object references. Understanding why `a = b` doesn't always mean what you think it means."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Class variables — shared state | ✅ |
| 2 | Instance vs class variable access rules | ✅ |
| 3 | `@classmethod` — alternate constructors | ✅ |
| 4 | `cls` vs class name (inheritance support) | ✅ |
| 5 | Factory/registry patterns with classmethods | ✅ |
| 6 | `@staticmethod` — utility functions | ✅ |
| 7 | When to use each method type | ✅ |
| 8 | Class variables as configuration | ✅ |
| 9 | Mutable class variable dangers | ✅ |
| 10 | Testing challenges with class state | ✅ |

---

> **Next Lesson →** Level 2, Lesson 9: *Handle Mutability and Object References in Python*

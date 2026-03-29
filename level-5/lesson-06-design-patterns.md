# 🏛️ Level 5 – Innovating | Lesson 6: Apply Common Design Patterns in Python

## *"The Playbook"*

> **O'Reilly Skill Objective:** *Apply common design patterns in Python (e.g., Singleton, Factory) to solve recurring problems.*

> **Previously Learned:** Classes, inheritance, decorators, metaclasses, `__init_subclass__`, closures, context managers, ABC, `__call__`

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

*The firm's codebase has grown: duplicate database connections, tangled notification logic, incompatible data sources, and objects that nobody knows how to extend.*

**Harvey:** "We need a playbook. Proven strategies for recurring problems."

**Mike:** "Design patterns. They're not frameworks or libraries — they're battle-tested blueprints that experienced developers have refined over decades. The same problems show up in every project. These are the solutions."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "A design pattern is what a precedent is to a lawyer. Someone already solved this problem. Learn the solution."

| Category | Pattern | Solves |
|----------|---------|--------|
| **Creational** | Singleton, Factory, Builder | How objects are created |
| **Structural** | Adapter, Decorator, Facade | How objects are composed |
| **Behavioral** | Observer, Strategy, Command | How objects communicate |

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Singleton — One Instance Only ⭐⭐

```python
# PROBLEM: Multiple database connections waste resources
# SOLUTION: Singleton — guarantee only ONE instance exists

# === Approach 1: Module-level (Pythonic!) ===
# db.py
# _connection = None
# def get_connection():
#     global _connection
#     if _connection is None:
#         _connection = create_connection()
#     return _connection

# === Approach 2: Decorator ===
def singleton(cls):
    instances = {}
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self, connection_string="sqlite:///firm.db"):
        self.connection_string = connection_string
        print(f"  🔌 Connected to {connection_string}")
    
    def query(self, sql):
        return f"Results for: {sql}"

db1 = DatabaseConnection()    # 🔌 Connected
db2 = DatabaseConnection()    # No output — returns same instance!
print(db1 is db2)             # True ✅

# === Approach 3: __new__ ===
class Logger:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self, name="app"):
        self.name = name
    
    def log(self, message):
        print(f"[{self.name}] {message}")

log1 = Logger("firm")
log2 = Logger("other")
print(log1 is log2)    # True — same instance
print(log2.name)       # "other" — ⚠️ __init__ reruns!
```

---

### 3.2 Factory — Delegated Object Creation ⭐⭐

```python
from abc import ABC, abstractmethod

# PROBLEM: Code is tightly coupled to specific classes
# SOLUTION: Factory — delegate creation to a function/class

# === Simple Factory Function ===
class PDFReport:
    def generate(self, data):
        return f"PDF: {len(data)} records"

class HTMLReport:
    def generate(self, data):
        return f"<html>{len(data)} records</html>"

class CSVReport:
    def generate(self, data):
        return f"CSV: {','.join(str(d) for d in data)}"

def create_report(format_type):
    """Factory function — creates the right report type."""
    factories = {
        "pdf": PDFReport,
        "html": HTMLReport,
        "csv": CSVReport,
    }
    cls = factories.get(format_type.lower())
    if cls is None:
        raise ValueError(f"Unknown format: {format_type}")
    return cls()

# Usage — caller doesn't know or care which class is used
report = create_report("pdf")
print(report.generate([1, 2, 3]))    # PDF: 3 records

# === Factory with Registration ===
class ReportFactory:
    _creators = {}
    
    @classmethod
    def register(cls, format_type):
        def decorator(report_cls):
            cls._creators[format_type] = report_cls
            return report_cls
        return decorator
    
    @classmethod
    def create(cls, format_type, **kwargs):
        creator = cls._creators.get(format_type)
        if not creator:
            raise ValueError(f"Unknown: {format_type}. Available: {list(cls._creators)}")
        return creator(**kwargs)

@ReportFactory.register("pdf")
class PDFReport:
    def generate(self, data): return f"PDF: {len(data)} records"

@ReportFactory.register("html")
class HTMLReport:
    def generate(self, data): return f"<html>{len(data)} records</html>"

# Adding new formats requires NO changes to factory code!
@ReportFactory.register("markdown")
class MarkdownReport:
    def generate(self, data): return f"# Report\n{len(data)} records"

report = ReportFactory.create("markdown")
print(report.generate([1, 2, 3]))    # # Report\n3 records
```

---

### 3.3 Strategy — Swappable Algorithms ⭐⭐

```python
from abc import ABC, abstractmethod

# PROBLEM: Different billing rules need different calculation logic
# SOLUTION: Strategy — define a family of algorithms, make them interchangeable

# === Strategy with classes ===
class BillingStrategy(ABC):
    @abstractmethod
    def calculate(self, hours, rate):
        pass

class StandardBilling(BillingStrategy):
    def calculate(self, hours, rate):
        return hours * rate

class DiscountBilling(BillingStrategy):
    def __init__(self, discount=0.1):
        self.discount = discount
    
    def calculate(self, hours, rate):
        return hours * rate * (1 - self.discount)

class ProBonoBilling(BillingStrategy):
    def calculate(self, hours, rate):
        return 0.0

class CappedBilling(BillingStrategy):
    def __init__(self, cap=50000):
        self.cap = cap
    
    def calculate(self, hours, rate):
        return min(hours * rate, self.cap)

class Invoice:
    def __init__(self, client, strategy: BillingStrategy):
        self.client = client
        self.strategy = strategy
        self.entries = []
    
    def add_entry(self, hours, rate):
        self.entries.append((hours, rate))
    
    def total(self):
        return sum(self.strategy.calculate(h, r) for h, r in self.entries)

# Same invoice, different strategies!
inv1 = Invoice("Wayne", StandardBilling())
inv1.add_entry(10, 450)
print(f"Standard: ${inv1.total():,.2f}")    # $4,500.00

inv2 = Invoice("Oscorp", DiscountBilling(0.15))
inv2.add_entry(10, 450)
print(f"Discount: ${inv2.total():,.2f}")    # $3,825.00

# === Pythonic: Strategy with functions ===
# In Python, you don't always need a class for strategy!
def standard_billing(hours, rate):
    return hours * rate

def discount_billing(hours, rate, discount=0.1):
    return hours * rate * (1 - discount)

class Invoice:
    def __init__(self, client, billing_func=standard_billing):
        self.client = client
        self.billing_func = billing_func
        self.entries = []
    
    def add_entry(self, hours, rate):
        self.entries.append((hours, rate))
    
    def total(self):
        return sum(self.billing_func(h, r) for h, r in self.entries)
```

---

### 3.4 Observer — Event Notification ⭐⭐

```python
# PROBLEM: When a case status changes, multiple systems need to know
# SOLUTION: Observer — one-to-many notification

class EventEmitter:
    """Generic observer/event system."""
    
    def __init__(self):
        self._listeners = {}
    
    def on(self, event, callback):
        """Register a listener for an event."""
        self._listeners.setdefault(event, []).append(callback)
        return self    # For chaining
    
    def off(self, event, callback):
        """Remove a listener."""
        if event in self._listeners:
            self._listeners[event].remove(callback)
    
    def emit(self, event, *args, **kwargs):
        """Notify all listeners of an event."""
        for callback in self._listeners.get(event, []):
            callback(*args, **kwargs)

class CaseManager(EventEmitter):
    def __init__(self):
        super().__init__()
        self.cases = {}
    
    def update_status(self, case_id, new_status):
        old_status = self.cases.get(case_id, "unknown")
        self.cases[case_id] = new_status
        self.emit("status_changed", case_id, old_status, new_status)
    
    def add_document(self, case_id, document):
        self.emit("document_added", case_id, document)

# Observers register for events
manager = CaseManager()

# Email notification
manager.on("status_changed", 
    lambda cid, old, new: print(f"  📧 Email: {cid} changed {old} → {new}"))

# Audit log
manager.on("status_changed",
    lambda cid, old, new: print(f"  📋 Audit: {cid} status change logged"))

# Dashboard update
manager.on("status_changed",
    lambda cid, old, new: print(f"  📊 Dashboard: {cid} now shows '{new}'"))

# Trigger — all observers notified automatically!
manager.update_status("WAYNE-001", "Discovery")
# 📧 Email: WAYNE-001 changed unknown → Discovery
# 📋 Audit: WAYNE-001 status change logged
# 📊 Dashboard: WAYNE-001 now shows 'Discovery'
```

---

### 3.5 Decorator Pattern — Dynamic Behavior ⭐

```python
# PROBLEM: Add features to objects without changing their class
# SOLUTION: Decorator pattern — wrap objects with new behavior
# (Not to be confused with Python @decorators, though related!)

from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def send(self, message):
        pass

class EmailNotifier(Notifier):
    def send(self, message):
        return f"📧 Email: {message}"

class SMSNotifier(Notifier):
    """Wraps another notifier, adding SMS."""
    def __init__(self, wrapped: Notifier):
        self.wrapped = wrapped
    
    def send(self, message):
        result = self.wrapped.send(message)
        return f"{result} | 📱 SMS: {message}"

class SlackNotifier(Notifier):
    """Wraps another notifier, adding Slack."""
    def __init__(self, wrapped: Notifier):
        self.wrapped = wrapped
    
    def send(self, message):
        result = self.wrapped.send(message)
        return f"{result} | 💬 Slack: {message}"

# Compose behaviors dynamically!
notifier = EmailNotifier()
print(notifier.send("Case update"))
# 📧 Email: Case update

notifier = SMSNotifier(EmailNotifier())
print(notifier.send("Case update"))
# 📧 Email: Case update | 📱 SMS: Case update

notifier = SlackNotifier(SMSNotifier(EmailNotifier()))
print(notifier.send("Case update"))
# 📧 Email: Case update | 📱 SMS: Case update | 💬 Slack: Case update
```

---

### 3.6 Adapter — Making Incompatible Interfaces Work ⭐

```python
# PROBLEM: New court API returns JSON, old system expects XML
# SOLUTION: Adapter — translate between interfaces

class OldCourtAPI:
    """Legacy API — returns XML strings."""
    def get_cases_xml(self, court_id):
        return f"<cases><case id='{court_id}-001'/></cases>"

class NewCourtAPI:
    """Modern API — returns dicts."""
    def fetch_cases(self, court_id):
        return [{"id": f"{court_id}-001", "status": "open"}]

# Adapter: makes NewCourtAPI look like OldCourtAPI
class CourtAPIAdapter:
    """Adapts NewCourtAPI to the OldCourtAPI interface."""
    
    def __init__(self, new_api: NewCourtAPI):
        self.new_api = new_api
    
    def get_cases_xml(self, court_id):
        """Calls new API, returns XML format."""
        cases = self.new_api.fetch_cases(court_id)
        xml_parts = [f"<case id='{c['id']}'/>" for c in cases]
        return f"<cases>{''.join(xml_parts)}</cases>"

# Old code works unchanged!
def process_court_data(api):
    xml = api.get_cases_xml("SUPREME")
    print(f"  Processing: {xml}")

process_court_data(OldCourtAPI())                      # Works ✅
process_court_data(CourtAPIAdapter(NewCourtAPI()))      # Also works ✅
```

---

### 3.7 Builder — Step-by-Step Construction ⭐

```python
# PROBLEM: Complex objects with many optional parameters
# SOLUTION: Builder — construct step by step

class Contract:
    def __init__(self):
        self.parties = []
        self.clauses = []
        self.term_months = 12
        self.value = 0
        self.jurisdiction = "New York"
        self.arbitration = False
        self.confidential = False
    
    def __repr__(self):
        return (f"Contract(parties={self.parties}, clauses={len(self.clauses)}, "
                f"term={self.term_months}mo, value=${self.value:,})")

class ContractBuilder:
    def __init__(self):
        self._contract = Contract()
    
    def add_party(self, name):
        self._contract.parties.append(name)
        return self    # Fluent interface — enables chaining!
    
    def add_clause(self, clause):
        self._contract.clauses.append(clause)
        return self
    
    def set_term(self, months):
        self._contract.term_months = months
        return self
    
    def set_value(self, amount):
        self._contract.value = amount
        return self
    
    def set_jurisdiction(self, jurisdiction):
        self._contract.jurisdiction = jurisdiction
        return self
    
    def with_arbitration(self):
        self._contract.arbitration = True
        return self
    
    def with_confidentiality(self):
        self._contract.confidential = True
        return self
    
    def build(self):
        if not self._contract.parties:
            raise ValueError("Contract must have at least one party")
        contract = self._contract
        self._contract = Contract()    # Reset for reuse
        return contract

# Fluent API — reads like English!
contract = (ContractBuilder()
    .add_party("Wayne Enterprises")
    .add_party("Stark Industries")
    .set_value(2_500_000)
    .set_term(24)
    .add_clause("Non-compete within 50 miles")
    .add_clause("Quarterly revenue sharing")
    .with_arbitration()
    .with_confidentiality()
    .build()
)

print(contract)
# Contract(parties=['Wayne Enterprises', 'Stark Industries'], clauses=2, term=24mo, value=$2,500,000)
```

---

### 3.8 Command — Encapsulate Actions ⭐

```python
from abc import ABC, abstractmethod
from datetime import datetime

# PROBLEM: Need undo/redo, action logging, or queued operations
# SOLUTION: Command — encapsulate operations as objects

class Command(ABC):
    @abstractmethod
    def execute(self): pass
    
    @abstractmethod
    def undo(self): pass

class Case:
    def __init__(self, case_id, status="open"):
        self.case_id = case_id
        self.status = status
        self.notes = []
    
    def __repr__(self):
        return f"Case({self.case_id}, status='{self.status}', notes={len(self.notes)})"

class ChangeStatusCommand(Command):
    def __init__(self, case, new_status):
        self.case = case
        self.new_status = new_status
        self.old_status = None
    
    def execute(self):
        self.old_status = self.case.status
        self.case.status = self.new_status
        print(f"  ✅ {self.case.case_id}: {self.old_status} → {self.new_status}")
    
    def undo(self):
        self.case.status = self.old_status
        print(f"  ↩️ {self.case.case_id}: reverted to {self.old_status}")

class AddNoteCommand(Command):
    def __init__(self, case, note):
        self.case = case
        self.note = note
    
    def execute(self):
        self.case.notes.append(self.note)
        print(f"  📝 Note added to {self.case.case_id}")
    
    def undo(self):
        self.case.notes.remove(self.note)
        print(f"  ↩️ Note removed from {self.case.case_id}")

class CommandHistory:
    def __init__(self):
        self.history = []
    
    def execute(self, command):
        command.execute()
        self.history.append(command)
    
    def undo(self):
        if self.history:
            command = self.history.pop()
            command.undo()
    
    def undo_all(self):
        while self.history:
            self.undo()

# Usage
case = Case("WAYNE-001")
history = CommandHistory()

history.execute(ChangeStatusCommand(case, "In Discovery"))
history.execute(AddNoteCommand(case, "Subpoena filed"))
history.execute(ChangeStatusCommand(case, "Trial Prep"))

print(f"\n  Current: {case}")
# Case(WAYNE-001, status='Trial Prep', notes=1)

history.undo()       # Revert to "In Discovery"
history.undo()       # Remove note
print(f"  After 2 undos: {case}")
# Case(WAYNE-001, status='In Discovery', notes=0)
```

---

### 3.9 Facade — Simplified Interface ⭐

```python
# PROBLEM: Multiple complex subsystems that clients must coordinate
# SOLUTION: Facade — one simple interface hides the complexity

class CaseDatabase:
    def get_case(self, case_id):
        return {"id": case_id, "status": "open", "client": "Wayne"}

class BillingSystem:
    def calculate_fees(self, case_id, hours):
        return hours * 450

class NotificationService:
    def send_email(self, to, subject, body):
        return f"Email sent to {to}: {subject}"

class DocumentManager:
    def generate_report(self, case_id, data):
        return f"Report for {case_id}: {len(data)} items"

# WITHOUT Facade — client must know all subsystems:
# db = CaseDatabase()
# billing = BillingSystem()
# notifications = NotificationService()
# docs = DocumentManager()
# case = db.get_case("W-001")
# fee = billing.calculate_fees("W-001", 10)
# report = docs.generate_report("W-001", {...})
# notifications.send_email("harvey", "Report", report)

# WITH Facade — one simple call:
class CaseFacade:
    """Simple interface to the firm's case management systems."""
    
    def __init__(self):
        self._db = CaseDatabase()
        self._billing = BillingSystem()
        self._notifications = NotificationService()
        self._docs = DocumentManager()
    
    def close_case(self, case_id, hours, attorney_email):
        """One call does everything."""
        case = self._db.get_case(case_id)
        fee = self._billing.calculate_fees(case_id, hours)
        report = self._docs.generate_report(case_id, case)
        self._notifications.send_email(
            attorney_email, f"Case {case_id} Closed", f"Fee: ${fee:,}\n{report}"
        )
        return {"case": case_id, "fee": fee, "status": "closed"}

# Client code is simple:
facade = CaseFacade()
result = facade.close_case("WAYNE-001", 85, "harvey@psl.com")
print(f"  {result}")
```

---

### 3.10 Choosing the Right Pattern

```python
"""
DESIGN PATTERN DECISION GUIDE:

┌─────────────────────────────────────────────────────────────┐
│ PROBLEM                          │ PATTERN                  │
├──────────────────────────────────┼──────────────────────────┤
│ Need only one instance?          │ Singleton (or module)    │
│ Don't know which class to create?│ Factory                  │
│ Complex object construction?     │ Builder                  │
│ Different algorithms at runtime? │ Strategy                 │
│ Notify many when one changes?    │ Observer                 │
│ Incompatible interfaces?         │ Adapter                  │
│ Add behavior without subclassing?│ Decorator pattern        │
│ Need undo/redo or action queuing?│ Command                  │
│ Simplify complex subsystems?     │ Facade                   │
│ Share expensive objects?         │ Flyweight (__slots__)    │
│ Access collection sequentially?  │ Iterator (__iter__)      │
│ Define skeleton, let subs fill?  │ Template Method          │
└──────────────────────────────────┴──────────────────────────┘

PYTHON-SPECIFIC ALTERNATIVES:
  Singleton    → module-level variables
  Strategy     → first-class functions
  Observer     → signals/events, callbacks
  Decorator    → Python @decorators
  Iterator     → generators (yield)
  Factory      → __init_subclass__ + registry
  Command      → closures, functools.partial
  Template     → ABC with default implementations
"""
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Over-Engineering with Patterns 🔥

```python
# ❌ Don't use a pattern just because you can!
# A Factory for two types is overkill:
class AnimalFactory:
    def create(self, type):
        if type == "dog": return Dog()
        if type == "cat": return Cat()

# ✅ Just use a dict or if/else for simple cases
creators = {"dog": Dog, "cat": Cat}
animal = creators[type_name]()
```

---

### Pitfall 2: Singleton With Mutable State

```python
# ❌ Singleton with shared mutable state = hidden coupling!
@singleton
class Config:
    def __init__(self):
        self.settings = {}

# Test A modifies Config → Test B sees those changes → test pollution!

# ✅ Use dependency injection instead
class Service:
    def __init__(self, config):    # Explicit dependency
        self.config = config
```

---

### Pitfall 3: Strategy as Classes When Functions Suffice

```python
# ❌ Java-style: a whole class for each strategy
class UpperStrategy:
    def transform(self, text): return text.upper()

class LowerStrategy:
    def transform(self, text): return text.lower()

# ✅ Pythonic: just pass functions!
processor = TextProcessor(transform=str.upper)
processor = TextProcessor(transform=str.lower)
processor = TextProcessor(transform=lambda s: s.title())
```

---

### Pitfall 4: Observer Memory Leaks

```python
# ❌ Listeners hold references → objects can't be garbage collected!
class Button:
    def __init__(self):
        self.listeners = []
    
    def on_click(self, callback):
        self.listeners.append(callback)    # Strong reference!

# ✅ Use weak references
import weakref

class Button:
    def __init__(self):
        self.listeners = []
    
    def on_click(self, callback):
        self.listeners.append(weakref.ref(callback))
    
    def click(self):
        self.listeners = [ref for ref in self.listeners if ref() is not None]
        for ref in self.listeners:
            ref()()
```

---

### Pitfall 5: Mixing Patterns Unnecessarily

```python
# ❌ Using Observer + Command + Factory + Strategy for a simple app
# Complexity must be JUSTIFIED by actual requirements

# ✅ Start simple, add patterns when the pain is real
# "You Ain't Gonna Need It" (YAGNI)
# Add a Factory when you have 5+ types, not 2
# Add Observer when 3+ things need to react, not 1
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 🗂️ Analogy: Patterns = The Firm's Operational Playbook

```
SINGLETON = ONE managing partner
  Jessica is THE managing partner. There's only one.

FACTORY = HR department
  "I need a tax attorney" → HR finds the right one.
  You don't recruit directly.

STRATEGY = Different litigation approaches
  Same case, different strategy — settle, litigate, arbitrate.
  Swap the approach without changing the case.

OBSERVER = The firm's notification system
  Case status changes → email, dashboard, audit log all update.

ADAPTER = The translator
  Old client speaks Latin. New system speaks English.
  The adapter translates between them.

BUILDER = Contract drafting
  Clause by clause, party by party — build the contract
  step by step, then finalize.

COMMAND = The to-do list with undo
  Each action is written down. Can replay or undo any step.

FACADE = Donna herself
  "Donna, close the Wayne case." She coordinates everything
  behind the scenes. You just make one request.

DONNA'S RULE:
  "Use a pattern when the pain is real.
   Don't build a cathedral when you need a shed."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Simple Factory**
```
Build a NotificationFactory that creates:
  EmailNotification, SMSNotification, PushNotification
Based on a string parameter.
```

**Exercise 2: Strategy Calculator**
```
Build a calculator that swaps operations:
  calc = Calculator(strategy=add)
  calc.execute(5, 3)  → 8
  calc.strategy = multiply
  calc.execute(5, 3)  → 15
Use functions as strategies.
```

**Exercise 3: Observer Event System**
```
Build an event system:
  events = EventSystem()
  events.on("login", log_login)
  events.on("login", send_welcome)
  events.emit("login", user="Harvey")
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Builder for SQL Queries**
```
query = (QueryBuilder()
    .select("name", "revenue")
    .from_table("clients")
    .where("tier = 'Platinum'")
    .order_by("revenue DESC")
    .limit(10)
    .build()
)
# → "SELECT name, revenue FROM clients WHERE tier = 'Platinum' ORDER BY revenue DESC LIMIT 10"
```

**Exercise 5: Command with Undo/Redo**
```
Extend the Command pattern with redo:
  history.execute(cmd)
  history.undo()
  history.redo()    # Re-executes the undone command
```

**Exercise 6: Adapter for Data Sources**
```
Adapt 3 different data sources to one interface:
  - CSVDataSource → read(filepath)
  - JSONDataSource → fetch(url)
  - SQLDataSource → query(sql)
All adapted to: DataAdapter.get_records() → list[dict]
```

---

### 🔴 Advanced Exercises

**Exercise 7: Plugin System**
```
Build a plugin system using Factory + Observer:
  pm = PluginManager()
  pm.register(MyPlugin)              # Factory registers
  pm.emit("on_startup")              # Observer notifies
  pm.emit("on_document_saved", doc)
```

**Exercise 8: Middleware Pattern**
```
Build a middleware pipeline (like Express.js or Django):
  app = App()
  app.use(logging_middleware)
  app.use(auth_middleware)
  app.use(rate_limit_middleware)
  app.handle(request)    # Request flows through all middleware
```

**Exercise 9: State Machine**
```
Build a case lifecycle using State pattern:
  case = Case()                # state: Draft
  case.submit()                # state: Filed
  case.assign("Harvey")        # state: Active
  case.close("Settled")        # state: Closed
  case.submit()                # Error: can't submit a closed case!
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Singleton __init__ runs on every call
class DB:
    _instance = None
    def __new__(cls, url):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    def __init__(self, url):
        self.url = url              # Resets every time!
        self.connect()              # Reconnects every time!

# Bug 2: Observer modifies list during iteration
class EventBus:
    def emit(self, event):
        for listener in self.listeners:    # Modifying during iteration!
            if listener(event) == "remove":
                self.listeners.remove(listener)

# Bug 3: Factory with hardcoded types
def create(type):
    if type == "a": return A()
    elif type == "b": return B()
    # Adding C requires modifying this function!
```

### Fixed Code ✅

```python
# Fix 1: Guard __init__ with a flag
class DB:
    _instance = None
    _initialized = False
    def __new__(cls, url=""):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    def __init__(self, url=""):
        if not DB._initialized:
            self.url = url
            self.connect()
            DB._initialized = True

# Fix 2: Iterate over a copy
class EventBus:
    def emit(self, event):
        for listener in self.listeners[:]:    # Copy!
            listener(event)

# Fix 3: Registry-based factory (open/closed principle)
class Factory:
    _registry = {}
    @classmethod
    def register(cls, key):
        def decorator(klass):
            cls._registry[key] = klass
            return klass
        return decorator
```

---

## 8. 🌍 Real-World Application

### Design Patterns in Python Frameworks

| Framework | Pattern | Where |
|-----------|---------|-------|
| **Django** | Observer | Signals (post_save, pre_delete) |
| **Django** | Template Method | Class-based views |
| **Flask** | Decorator | Route registration |
| **SQLAlchemy** | Unit of Work | Session management |
| **pytest** | Factory | Fixtures |
| **logging** | Singleton | Logger instances |
| **ABC** | Template Method | Abstract base classes |
| **functools** | Decorator | @lru_cache, @wraps |
| **contextlib** | Facade | @contextmanager |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is a design pattern?

**Q2.** What problem does the Singleton pattern solve?

**Q3.** What is the Factory pattern?

**Q4.** How does Strategy differ from Template Method?

**Q5.** What is the Observer pattern?

**Q6.** What does the Adapter pattern do?

**Q7.** What is the Builder pattern's key feature?

**Q8.** What does the Command pattern enable?

**Q9.** What is a Facade?

**Q10.** When should you NOT use a design pattern?

---

### 📋 Answer Key

> **Q1.** A reusable solution to a commonly occurring problem in software design. Not code — a blueprint describing how to solve a recurring problem.

> **Q2.** Ensuring only one instance of a class exists. Used for shared resources (database connections, loggers, config).

> **Q3.** Delegates object creation to a function or class, decoupling the client from specific implementations.

> **Q4.** Strategy swaps the ENTIRE algorithm from outside. Template Method defines a skeleton with SOME steps overridden by subclasses.

> **Q5.** A one-to-many notification system. When one object changes, all dependents are notified automatically.

> **Q6.** Makes incompatible interfaces work together by wrapping one interface to look like another.

> **Q7.** Step-by-step construction with a fluent interface. Each method returns `self`, enabling chaining: `builder.add_x().set_y().build()`.

> **Q8.** Encapsulates actions as objects, enabling undo/redo, queuing, logging, and replay of operations.

> **Q9.** A simplified interface to a complex subsystem. Hides internal complexity behind one easy-to-use API.

> **Q10.** When the problem is simple enough that a pattern adds unnecessary complexity. Follow YAGNI — add patterns when the pain is real, not preemptively.

---

## 🎬 End of Lesson Scene

**Harvey:** "So these are the plays."

**Mike:** "Nine proven solutions: Singleton for shared resources, Factory for flexible creation, Strategy for swappable logic, Observer for event notification, Decorator for dynamic features, Adapter for compatibility, Builder for complex construction, Command for undo/redo, and Facade for simplicity."

**Harvey:** "And the most important rule?"

**Mike:** "Don't use a pattern until you feel the pain it solves. Premature abstraction is worse than no abstraction."

**Harvey:** "Next: making it fast. timeit and cProfile."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Singleton (decorator, `__new__`, module) | ✅ |
| 2 | Factory (function, registry, `__init_subclass__`) | ✅ |
| 3 | Strategy (class-based and function-based) | ✅ |
| 4 | Observer (event emitter) | ✅ |
| 5 | Decorator pattern (wrapping objects) | ✅ |
| 6 | Adapter (interface translation) | ✅ |
| 7 | Builder (fluent step-by-step construction) | ✅ |
| 8 | Command (undo/redo, action history) | ✅ |
| 9 | Facade (simplified interface) | ✅ |
| 10 | Pattern selection decision guide | ✅ |

---

> **Next Lesson →** Level 5, Lesson 7: *Optimize Python Code with timeit and cProfile*

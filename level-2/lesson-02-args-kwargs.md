# 🏛️ Level 2 – Applying | Lesson 2: Use *args and **kwargs in Python Functions

## *"The Open Brief"*

> **O'Reilly Skill Objective:** *Define a function with variable positional arguments (`*args`) and variable keyword arguments (`**kwargs`).*

> **Previously Learned:** Variables, strings, lists, tuples, dictionaries, conditionals, loops, functions, default arguments, keyword-only parameters, classes, objects, file I/O

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

*Harvey's office. He's on the phone.*

**Harvey:** *(into phone)* "Yes. We'll handle Wayne, Stark, AND Oscorp. All three. Send the documents."

*He hangs up.*

**Harvey:** "We just landed three new mergers, and they all need intake processing. Problem is, sometimes we onboard one client, sometimes three, sometimes twelve. Our intake function needs to handle ANY number of clients in a single call."

**Mike:** "That's `*args`. But it gets better — sometimes each client comes with different metadata. Wayne sends revenue and case type. Stark sends revenue, case type, AND a preferred attorney. Oscorp sends a completely different set of fields."

**Harvey:** "So the function needs to accept whatever they throw at us?"

**Mike:** "`**kwargs`. Variable keyword arguments. Together, `*args` and `**kwargs` make functions that can accept **anything** — any number of positional arguments, any number of named arguments, in any combination."

**Harvey:** "Build me a function that never says 'I don't accept that parameter.'"

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Think of it this way."

| Approach | What You Know | What You Don't Know |
|----------|--------------|-------------------|
| Regular params | Exactly which arguments | — |
| Default params | Which arguments + their usual values | Whether caller will override |
| `*args` | The function exists | How many positional args will be passed |
| `**kwargs` | The function exists | What keyword args will be passed — or how many |

**Harvey:** "Regular parameters are a fixed contract. `*args` and `**kwargs` are an **open brief** — handle whatever comes in."

```python
# Fixed — only accepts exactly 3 clients
def onboard(client1, client2, client3): ...

# Flexible — accepts ANY number of clients
def onboard(*clients): ...

# Fully flexible — accepts anything
def onboard(*clients, **options): ...
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `*args` — Variable Positional Arguments

```python
def onboard_clients(*names):
    """Accept any number of client names."""
    print(f"Onboarding {len(names)} clients:")
    for name in names:
        print(f"  → {name}")

onboard_clients("Wayne")
# Onboarding 1 clients:
#   → Wayne

onboard_clients("Wayne", "Stark", "Oscorp")
# Onboarding 3 clients:
#   → Wayne  → Stark  → Oscorp

onboard_clients()
# Onboarding 0 clients:
```

**Mike:** "Key facts about `*args`:"

| Fact | Detail |
|------|--------|
| `*args` collects extra positional arguments | Into a **tuple** (not a list!) |
| The name `args` is convention | You can use `*clients`, `*values`, `*items` — anything |
| `*args` can be empty | If no extra args passed, it's `()` |
| Position in signature | After regular positional parameters |

```python
# Examining the type
def show(*args):
    print(type(args))    # <class 'tuple'>
    print(args)

show(1, 2, 3)    # (1, 2, 3)
show()            # ()
```

---

### 3.2 `*args` with Regular Parameters

```python
def calculate_billing(rate, *hours_per_case):
    """First arg is rate, rest are hours for each case."""
    total_hours = sum(hours_per_case)
    total = total_hours * rate
    print(f"Rate: ${rate}/hr | Cases: {len(hours_per_case)}")
    print(f"Hours per case: {hours_per_case}")
    print(f"Total: {total_hours}h × ${rate} = ${total:,.2f}")
    return total

calculate_billing(450, 85)                    # 1 case
calculate_billing(450, 85, 42, 63)            # 3 cases
calculate_billing(450, 85, 42, 63, 28, 95)    # 5 cases
```

**Mike:** "Regular params are filled first, then `*args` catches everything else:"

```python
def func(a, b, *args):
    print(f"a={a}, b={b}, args={args}")

func(1, 2)           # a=1, b=2, args=()
func(1, 2, 3)        # a=1, b=2, args=(3,)
func(1, 2, 3, 4, 5)  # a=1, b=2, args=(3, 4, 5)
```

---

### 3.3 `**kwargs` — Variable Keyword Arguments

```python
def create_client(name, **details):
    """Accept a name plus any number of keyword details."""
    client = {"name": name}
    client.update(details)    # Merge all kwargs into the dict
    return client

c1 = create_client("Wayne Enterprises",
                    revenue=2500000, 
                    case_type="Merger")
print(c1)
# {'name': 'Wayne Enterprises', 'revenue': 2500000, 'case_type': 'Merger'}

c2 = create_client("Stark Industries",
                    revenue=800000,
                    case_type="Patent",
                    attorney="Mike Ross",
                    priority="high",
                    referral_source="Jessica")
print(c2)
# {'name': 'Stark Industries', 'revenue': 800000, 'case_type': 'Patent', 
#  'attorney': 'Mike Ross', 'priority': 'high', 'referral_source': 'Jessica'}
```

**Mike:** "Key facts about `**kwargs`:"

| Fact | Detail |
|------|--------|
| `**kwargs` collects extra keyword arguments | Into a **dictionary** |
| The name `kwargs` is convention | You can use `**options`, `**config`, `**fields` |
| `**kwargs` can be empty | If no extra kwargs, it's `{}` |
| Position in signature | Must be LAST |
| Keys are strings | Keyword argument names become string keys |

```python
def show(**kwargs):
    print(type(kwargs))    # <class 'dict'>
    print(kwargs)

show(a=1, b=2, c=3)    # {'a': 1, 'b': 2, 'c': 3}
show()                  # {}
```

---

### 3.4 Combining `*args` and `**kwargs`

```python
def universal_function(*args, **kwargs):
    """Accepts literally anything."""
    print(f"  Positional args ({len(args)}): {args}")
    print(f"  Keyword args ({len(kwargs)}): {kwargs}")

universal_function(1, 2, 3, name="Harvey", rate=450)
# Positional args (3): (1, 2, 3)
# Keyword args (2): {'name': 'Harvey', 'rate': 450}

universal_function()
# Positional args (0): ()
# Keyword args (0): {}

universal_function("only", "positional", "args")
# Positional args (3): ('only', 'positional', 'args')
# Keyword args (0): {}
```

#### The Complete Parameter Order

```python
def full_example(
    pos_only1, pos_only2,         # 1. Positional-only (before /)
    /,
    regular1, regular2,           # 2. Regular positional-or-keyword
    *args,                        # 3. Variable positional
    kw_only1, kw_only2="default", # 4. Keyword-only (after *args)
    **kwargs                      # 5. Variable keyword (always last)
):
    pass

# Order must be: positional-only → regular → *args → keyword-only → **kwargs
```

**Mike:** "Memorize the order:"
```
def f(pos_only, /, regular, *args, kw_only, **kwargs):
     ─────────  ─  ───────  ────  ───────  ────────
        1.       2.    3.     4.      5.       6.
     pos-only  sep  regular  var   kw-only   var-kw
```

---

### 3.5 Unpacking Arguments with `*` and `**` ⭐

**Mike:** "The reverse — unpacking collections INTO function calls."

#### Unpacking Lists/Tuples with `*`
```python
def add(a, b, c):
    return a + b + c

numbers = [10, 20, 30]

# ❌ Can't pass a list as three arguments
# add(numbers)    # TypeError: missing 2 required positional arguments

# ✅ Unpack the list
add(*numbers)     # Same as add(10, 20, 30) → 60

# Works with tuples too
coords = (3, 4, 5)
add(*coords)      # 12
```

#### Unpacking Dictionaries with `**`
```python
def create_invoice(client, hours, rate, tax_rate=0.09):
    total = hours * rate * (1 + tax_rate)
    print(f"  {client}: {hours}h × ${rate} + tax = ${total:,.2f}")

# Invoice data as a dictionary
invoice_data = {
    "client": "Wayne Enterprises",
    "hours": 85,
    "rate": 450,
    "tax_rate": 0.12
}

# Unpack the dict into keyword arguments
create_invoice(**invoice_data)
# Same as: create_invoice(client="Wayne...", hours=85, rate=450, tax_rate=0.12)
```

#### Combining `*` and `**` Unpacking
```python
def process(action, *items, verbose=False, **metadata):
    print(f"  Action: {action}")
    print(f"  Items: {items}")
    print(f"  Verbose: {verbose}")
    print(f"  Metadata: {metadata}")

args = ["process"]
more_items = ["file1.txt", "file2.txt"]
options = {"verbose": True, "user": "harvey", "priority": "high"}

process(*args, *more_items, **options)
# Action: process
# Items: ('file1.txt', 'file2.txt')
# Verbose: True
# Metadata: {'user': 'harvey', 'priority': 'high'}
```

---

### 3.6 Forwarding Arguments (Wrapper/Decorator Pattern) ⭐

**Mike:** "The #1 use case for `*args, **kwargs` — wrapping another function."

```python
def log_call(func):
    """Wrapper that logs function calls — forwards ALL arguments."""
    def wrapper(*args, **kwargs):
        print(f"  📞 Calling {func.__name__}(args={args}, kwargs={kwargs})")
        result = func(*args, **kwargs)    # Forward everything!
        print(f"  📤 {func.__name__} returned: {result}")
        return result
    return wrapper

# Original function — takes specific parameters
def calculate_billing(hours, rate, tax_rate=0.09):
    return hours * rate * (1 + tax_rate)

# Wrapped version — works without knowing the original's signature
logged_billing = log_call(calculate_billing)
result = logged_billing(85, 450, tax_rate=0.12)
# 📞 Calling calculate_billing(args=(85, 450), kwargs={'tax_rate': 0.12})
# 📤 calculate_billing returned: 42840.0
```

**Mike:** "The wrapper doesn't need to know what arguments `calculate_billing` accepts. `*args` catches all positional, `**kwargs` catches all keyword, and `*args, **kwargs` forwards them all."

---

### 3.7 Merging Dictionaries with `**`

```python
# Merging defaults with overrides
defaults = {"tax_rate": 0.09, "currency": "USD", "format": "text"}
user_config = {"tax_rate": 0.12, "verbose": True}

# Method 1: dict unpacking (Python 3.5+)
final_config = {**defaults, **user_config}
print(final_config)
# {'tax_rate': 0.12, 'currency': 'USD', 'format': 'text', 'verbose': True}
# Later values override earlier ones!

# Method 2: | operator (Python 3.9+)
final_config = defaults | user_config
```

---

### 3.8 Real-World Patterns

#### Pattern 1: Flexible Constructor
```python
class Client:
    REQUIRED = {"name", "revenue"}
    
    def __init__(self, **fields):
        # Validate required fields
        missing = Client.REQUIRED - set(fields.keys())
        if missing:
            raise ValueError(f"Missing required fields: {missing}")
        
        # Set all fields as attributes
        for key, value in fields.items():
            setattr(self, key, value)
    
    def __str__(self):
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"Client({attrs})"

# Flexible instantiation
c1 = Client(name="Wayne", revenue=2500000)
c2 = Client(name="Stark", revenue=800000, attorney="Mike", priority="high")
c3 = Client(name="Oscorp", revenue=600000, case_type="Litigation", 
            referred_by="Jessica", notes="Handle with care")

print(c1)
print(c2)
print(c3.notes)   # "Handle with care"
```

#### Pattern 2: Function Dispatch
```python
def execute_command(command, *args, **kwargs):
    """Execute different commands with variable arguments."""
    commands = {
        "add": lambda *a: sum(a),
        "multiply": lambda *a: eval("*".join(str(x) for x in a)) if a else 0,
        "greet": lambda *a, **kw: f"{kw.get('greeting', 'Hello')}, {a[0]}!",
    }
    
    if command not in commands:
        return f"Unknown command: {command}"
    
    return commands[command](*args, **kwargs)

print(execute_command("add", 1, 2, 3, 4, 5))           # 15
print(execute_command("multiply", 2, 3, 4))             # 24
print(execute_command("greet", "Harvey", greeting="Good morning"))  # Good morning, Harvey!
```

#### Pattern 3: Configuration Override Chain
```python
def configure(*config_dicts, **overrides):
    """Merge multiple config sources, with later values winning."""
    result = {}
    for d in config_dicts:
        result.update(d)
    result.update(overrides)
    return result

defaults = {"debug": False, "port": 8080, "host": "0.0.0.0"}
env_config = {"port": 3000, "database": "psl_prod"}
cli_config = {"debug": True}

final = configure(defaults, env_config, cli_config, host="localhost")
print(final)
# {'debug': True, 'port': 3000, 'host': 'localhost', 'database': 'psl_prod'}
```

---

### 3.9 Solving Harvey's Problem — Universal Intake System

```python
# ============================================
# Pearson Specter Litt — Universal Intake System
# ============================================

class IntakeSystem:
    """Handles client intake with variable data fields."""
    
    def __init__(self):
        self.clients = []
        self.log = []
    
    def onboard(self, *names, **shared_options):
        """Onboard one or more clients with shared options."""
        results = []
        for name in names:
            client = {"name": name, **shared_options}
            client["id"] = f"PSL-{len(self.clients) + 1:04d}"
            self.clients.append(client)
            results.append(client)
            self._log(f"Onboarded: {name}")
        
        print(f"  ✅ Onboarded {len(names)} client(s)")
        return results
    
    def enrich(self, client_id, **updates):
        """Add or update fields for an existing client."""
        for client in self.clients:
            if client["id"] == client_id:
                client.update(updates)
                self._log(f"Enriched {client_id}: {list(updates.keys())}")
                print(f"  ✅ Updated {client_id}: +{len(updates)} fields")
                return client
        print(f"  ❌ Client {client_id} not found")
        return None
    
    def search(self, **criteria):
        """Search clients by any field(s)."""
        results = self.clients
        for key, value in criteria.items():
            results = [c for c in results if c.get(key) == value]
        return results
    
    def report(self, *fields, title="Client Report", separator="─"):
        """Generate a report with variable columns."""
        if not fields:
            fields = ("id", "name")
        
        print(f"\n  {title}")
        print(f"  {separator * 60}")
        
        # Header
        header = "  ".join(f"{f:<20}" for f in fields)
        print(f"  {header}")
        print(f"  {separator * 60}")
        
        # Rows
        for client in self.clients:
            row = "  ".join(f"{str(client.get(f, '—')):<20}" for f in fields)
            print(f"  {row}")
        
        print(f"  {separator * 60}")
        print(f"  Total: {len(self.clients)} clients")
    
    def _log(self, message):
        from datetime import datetime
        self.log.append(f"[{datetime.now():%H:%M:%S}] {message}")
    
    def export(self, *extra_fields, **format_options):
        """Export client data with variable fields and format options."""
        base_fields = ["id", "name"]
        all_fields = base_fields + list(extra_fields)
        fmt = format_options.get("format", "text")
        
        if fmt == "csv":
            lines = [",".join(all_fields)]
            for c in self.clients:
                row = [str(c.get(f, "")) for f in all_fields]
                lines.append(",".join(row))
            return "\n".join(lines)
        else:
            return [
                {f: c.get(f) for f in all_fields}
                for c in self.clients
            ]


# === Demonstrate the system ===
print("=" * 60)
print(f"{'UNIVERSAL INTAKE SYSTEM':^60}")
print("=" * 60)

intake = IntakeSystem()

# Onboard multiple clients at once with shared options
intake.onboard("Wayne Enterprises", "Dalton Industries",
               case_type="Merger", priority="high")

intake.onboard("Stark Industries",
               case_type="Patent", attorney="Mike Ross")

intake.onboard("Oscorp", "Luthor Corp", "Queen Industries",
               case_type="Litigation", region="East Coast")

# Enrich with additional data
intake.enrich("PSL-0001", revenue=2500000, tier="Platinum", contact="Bruce Wayne")
intake.enrich("PSL-0003", revenue=800000, tier="Gold")

# Search with variable criteria
print("\nMerger cases:")
mergers = intake.search(case_type="Merger")
for c in mergers:
    print(f"  {c['id']}: {c['name']}")

print("\nHigh priority:")
high_pri = intake.search(priority="high")
for c in high_pri:
    print(f"  {c['id']}: {c['name']}")

# Flexible report
intake.report("id", "name", "case_type", "priority",
              title="PSL Intake Report — All Clients")

# Export with variable fields
csv_data = intake.export("case_type", "revenue", "priority", format="csv")
print(f"\nCSV Export:\n{csv_data}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: `*args` and `**kwargs` Position Rules

```python
# ❌ **kwargs must be LAST
# def f(**kwargs, *args): ...   # SyntaxError!
# def f(**kwargs, x): ...       # SyntaxError!

# ❌ Can't have two *args or two **kwargs
# def f(*a, *b): ...            # SyntaxError!
# def f(**a, **b): ...          # SyntaxError!

# ✅ Correct order
def f(regular, *args, keyword_only=True, **kwargs):
    pass
```

---

### Pitfall 2: `*args` Blocks Keyword Defaults

```python
# ⚠️ After *args, parameters are keyword-only!
def func(a, *args, b=10):
    print(f"a={a}, args={args}, b={b}")

func(1, 2, 3)         # a=1, args=(2, 3), b=10
func(1, 2, 3, b=20)   # a=1, args=(2, 3), b=20
# func(1, 2, 3, 20)   # b is NOT 20! args=(2, 3, 20), b=10
```

**Louis:** "`2, 3, 20` — all three go into `*args`. To set `b`, you MUST use `b=20`."

---

### Pitfall 3: Modifying `*args` (It's a Tuple!)

```python
def modify(*args):
    args[0] = 999       # ❌ TypeError: 'tuple' does not support item assignment

# ✅ Convert to list first
def modify(*args):
    args_list = list(args)
    args_list[0] = 999
    return tuple(args_list)
```

---

### Pitfall 4: `**kwargs` Key Collisions

```python
def func(a, **kwargs):
    print(f"a={a}, kwargs={kwargs}")

# ❌ 'a' is both a regular param AND a keyword arg
# func(1, a=2)   # TypeError: got multiple values for argument 'a'

# ❌ Same problem with unpacking
data = {"a": 1, "b": 2}
# func(1, **data)   # TypeError: got multiple values for argument 'a'

# ✅ Fix: remove conflicting keys
data = {"a": 1, "b": 2}
func(**data)          # a=1, kwargs={'b': 2}
# or
func(1, **{k: v for k, v in data.items() if k != "a"})
```

---

### Pitfall 5: Confusing `*` in Different Contexts

```python
# In function DEFINITION — collects args into tuple
def f(*args):        # args = (1, 2, 3)
    pass
f(1, 2, 3)

# In function CALL — unpacks iterable into args
nums = [1, 2, 3]
f(*nums)             # Same as f(1, 2, 3)

# In assignment — extended unpacking
first, *rest = [1, 2, 3, 4, 5]
print(first)          # 1
print(rest)           # [2, 3, 4, 5]  ← list, NOT tuple!
```

---

### Pitfall 6: Loss of Type Safety

```python
# With *args/**kwargs, you lose compile-time checks
def process(**kwargs):
    # Typos in keyword names go undetected!
    name = kwargs.get("name")
    revenue = kwargs.get("revnue")    # Typo! Should be "revenue"
    # No error — just returns None

process(name="Wayne", revenue=2500000)
# 'revenue' is in kwargs, but code looks for 'revnue' → silent None

# ✅ Mitigation: validate early
def process(**kwargs):
    VALID_KEYS = {"name", "revenue", "case_type"}
    invalid = set(kwargs.keys()) - VALID_KEYS
    if invalid:
        raise ValueError(f"Unknown fields: {invalid}")
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📦 Analogy: `*args` and `**kwargs` = Shipping Containers

```
Regular params → Labeled boxes of fixed sizes
  📦 Name    📦 Revenue    📦 Type
  
*args → An open container for extra unlabeled boxes
  🚛 [...any number of boxes...]
  You know they're boxes, but you don't know how many
  
**kwargs → An open container for extra LABELED boxes
  🚛 [...any number of labeled boxes...]
  Each box has a tag: priority="high", region="east"
```

**Donna:** "Regular parameters are reserved parking spots — each one has a name. `*args` is overflow parking. `**kwargs` is valet parking with custom tags."

```
Unpacking (* and **) = Opening the containers

*list → Take items OUT of a container and spread them as separate args
  🚛[1, 2, 3] → 📦1  📦2  📦3

**dict → Take labeled items OUT and spread as keyword args
  🚛{a:1, b:2} → a=📦1  b=📦2
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Flexible Sum**
```
Write: total(*numbers)
Returns the sum of any number of arguments.

total(1, 2, 3)       → 6
total(10)             → 10
total()               → 0
total(1, 2, 3, 4, 5)  → 15
```

**Exercise 2: Profile Builder**
```
Write: build_profile(name, age, **details)
Returns a dict with name, age, and all extra details.

build_profile("Harvey", 42, firm="PSL", title="Partner")
→ {"name": "Harvey", "age": 42, "firm": "PSL", "title": "Partner"}
```

**Exercise 3: Argument Inspector**
```
Write: inspect_args(*args, **kwargs)
Prints detailed info about what was received:
  - Number of positional args and their types
  - Number of keyword args and their key-value pairs
  - Total argument count

Test with various combinations.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Multi-Print**
```
Write: multi_print(*items, sep=" | ", end="\n", prefix="", uppercase=False)

Customizable print function.
  multi_print("a", "b", "c")                    → "a | b | c"
  multi_print("a", "b", sep=", ", prefix="> ")   → "> a, b"
  multi_print("hello", uppercase=True)           → "HELLO"
```

**Exercise 5: Dict Merger**
```
Write: merge(*dicts, **overrides)
Merges any number of dictionaries, with later values winning.
Final overrides apply last.

d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}
merge(d1, d2, c=10)  → {"a": 1, "b": 3, "c": 10}
```

**Exercise 6: Function Logger Wrapper**
```
Write: logged(func)
Returns a wrapper that:
- Prints the function name, args, and kwargs before calling
- Prints the return value after
- Forwards ALL arguments transparently

Works with ANY function regardless of its signature.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Event System**
```
Build an event system using *args/**kwargs:

class EventBus:
    def subscribe(self, event_name, callback)
    def emit(self, event_name, *args, **kwargs)
    def unsubscribe(self, event_name, callback)

Callbacks receive whatever args/kwargs are emitted.
Support multiple subscribers per event.
```

**Exercise 8: Flexible ORM-Style Query**
```
Build: query(table, *columns, **conditions)

query("clients", "name", "revenue", tier="Gold", active=True)
→ "SELECT name, revenue FROM clients WHERE tier='Gold' AND active=True"

query("cases")
→ "SELECT * FROM cases"

query("attorneys", "name", min_rate=400)
→ "SELECT name FROM attorneys WHERE rate >= 400"

Handle: gt, lt, gte, lte suffixes (e.g., min_rate → rate >= 400)
```

**Exercise 9: Plugin Architecture**
```
Build a plugin system:

class Application:
    def register_plugin(self, name, init_func, **config)
    def call_plugin(self, name, *args, **kwargs)
    def list_plugins()

Each plugin is initialized with its config.
When called, receives *args/**kwargs plus its config.
Handle missing plugins, initialization errors, etc.
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: Args/kwargs order wrong
def func(**kwargs, *args):
    pass

# Bug 2: Duplicate argument
def create(name, **kwargs):
    pass

create("Harvey", name="Mike")

# Bug 3: Modifying args tuple
def double_all(*args):
    for i in range(len(args)):
        args[i] *= 2
    return args

# Bug 4: Unpacking into wrong type
def add(a, b, c):
    return a + b + c

data = {"a": 1, "b": 2, "c": 3}
result = add(*data)    # What does this unpack?

# Bug 5: kwargs.get typo
def process(**kwargs):
    name = kwargs.get("client_name")
    return name

result = process(clientname="Wayne")
print(result)   # Expected: "Wayne"
```

### Fixed Code ✅

```python
# Fix 1: *args before **kwargs
def func(*args, **kwargs):
    pass

# Fix 2: Don't duplicate — use either regular param or kwargs
def create(name, **kwargs):
    pass
create("Harvey")                  # ✅ name as positional
# or
create(name="Harvey", title="Partner")  # ✅ but not both!

# Fix 3: Convert tuple to list
def double_all(*args):
    result = [x * 2 for x in args]    # ✅ Create new list
    return tuple(result)

# Fix 4: Use ** to unpack dict as keyword args
data = {"a": 1, "b": 2, "c": 3}
result = add(**data)    # ✅ Unpacks as a=1, b=2, c=3

# Fix 5: Match the KEY used by the caller
def process(**kwargs):
    name = kwargs.get("clientname")    # ✅ Match caller's key
    return name

result = process(clientname="Wayne")
print(result)   # "Wayne" ✅
```

---

## 8. 🌍 Real-World Application

### Production Example: Logging Wrapper

```python
import functools
from datetime import datetime

def timed_log(func):
    """Universal wrapper — works with ANY function signature."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = datetime.now()
        print(f"  ⏱️ {func.__name__} called at {start:%H:%M:%S}")
        
        try:
            result = func(*args, **kwargs)
            elapsed = (datetime.now() - start).total_seconds()
            print(f"  ✅ {func.__name__} → {result!r} ({elapsed:.3f}s)")
            return result
        except Exception as e:
            elapsed = (datetime.now() - start).total_seconds()
            print(f"  ❌ {func.__name__} raised {type(e).__name__}: {e} ({elapsed:.3f}s)")
            raise
    
    return wrapper

@timed_log
def calculate_billing(hours, rate, tax_rate=0.09):
    return hours * rate * (1 + tax_rate)

@timed_log
def classify(name, revenue, **extra):
    tier = "Platinum" if revenue >= 1000000 else "Gold" if revenue >= 500000 else "Silver"
    return {"name": name, "tier": tier, **extra}

calculate_billing(85, 450)
classify("Wayne", 2500000, region="East", priority="high")
```

---

## 9. ✅ Knowledge Check

---

**Q1.** What type does `*args` collect into? What about `**kwargs`?

**Q2.** What is the correct order of parameters: regular, `*args`, keyword-only, `**kwargs`?

**Q3.** What is the output?
```python
def f(a, *args, b=10, **kwargs):
    print(a, args, b, kwargs)

f(1, 2, 3, b=20, x=30)
```

**Q4.** What is the difference between `*` in a function definition vs a function call?

**Q5.** How do you forward all arguments from a wrapper to the wrapped function?

**Q6.** What happens if you do `create(name="A", **{"name": "B"})`?

**Q7.** Write a function that takes any number of strings and returns them joined with a configurable separator (default `", "`).

**Q8.** What is `functools.wraps` and why should you use it with `*args, **kwargs` wrappers?

**Q9.** Why can't you modify `args[0]` inside a function that uses `*args`?

**Q10.** When should you prefer explicit parameters over `**kwargs`?

---

### 📋 Answer Key

> **Q1.** `*args` → tuple. `**kwargs` → dictionary.

> **Q2.** `def f(regular, *args, kw_only, **kwargs)`

> **Q3.** `1 (2, 3) 20 {'x': 30}`

> **Q4.** Definition: **collects** arguments into a tuple/dict. Call: **unpacks** an iterable/dict into separate arguments.

> **Q5.** `def wrapper(*args, **kwargs): return func(*args, **kwargs)`

> **Q6.** `TypeError: got multiple values for argument 'name'`

> **Q7.** 
```python
def join_strings(*strings, sep=", "):
    return sep.join(strings)
```

> **Q8.** `functools.wraps` copies the original function's name, docstring, and metadata to the wrapper, making debugging easier.

> **Q9.** `args` is a tuple — tuples are immutable.

> **Q10.** When you know the exact parameters (better IDE support, type checking, documentation, and error messages). Use `**kwargs` only when flexibility is truly needed.

---

## 🎬 End of Lesson Scene

**Harvey:** "Universal intake system is live. One function handles any client, any data."

**Mike:** "And the argument forwarding pattern — wrapping functions without knowing their signatures — that's how every framework works. Flask, Django, pytest — they all use `*args, **kwargs`."

**Harvey:** "Next: list comprehensions. Making data processing one-liners."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `*args` — variable positional arguments (tuple) | ✅ |
| 2 | `**kwargs` — variable keyword arguments (dict) | ✅ |
| 3 | Combining regular, `*args`, keyword-only, `**kwargs` | ✅ |
| 4 | The complete parameter order | ✅ |
| 5 | Unpacking with `*` and `**` in function calls | ✅ |
| 6 | Argument forwarding / wrapper pattern | ✅ |
| 7 | Dictionary merging with `**` | ✅ |
| 8 | Flexible constructors and dispatch patterns | ✅ |

---

> **Next Lesson →** Level 2, Lesson 3: *Use List Comprehension in Python*

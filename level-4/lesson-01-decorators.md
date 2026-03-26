# 🏛️ Level 4 – Advancing | Lesson 1: Create and Use Decorators in Python

## *"The Power of Attorney"*

> **O'Reilly Skill Objective:** *Create and apply custom decorators to modify function behavior.*

> **Previously Learned:** Functions as first-class objects, closures (implicit), `*args`/`**kwargs`, `functools`, lambda, context managers, duck typing

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

*Every function that touches client data needs authorization, logging, and timing. Same boilerplate in 200 functions.*

```python
def get_client_revenue(client_id, user):
    # Authorization boilerplate
    if not user.has_permission("read_billing"):
        raise PermissionError("Unauthorized")
    
    # Logging boilerplate
    log.info(f"{user.name} called get_client_revenue({client_id})")
    
    # Timing boilerplate
    start = time.perf_counter()
    
    # The ACTUAL logic — 2 lines buried under boilerplate
    client = db.get(client_id)
    revenue = client.total_revenue()
    
    # More boilerplate
    log.info(f"Completed in {time.perf_counter() - start:.4f}s")
    return revenue
```

**Harvey:** "200 functions. Same auth check. Same logging. Same timing. One missed check is a security breach."

**Mike:** "Decorators. Wrap the behavior AROUND the function. Write auth/logging/timing once. Apply with one line."

```python
@require_permission("read_billing")
@log_call
@timer
def get_client_revenue(client_id):
    client = db.get(client_id)
    return client.total_revenue()
```

**Harvey:** "A decorator is a power of attorney — it acts on behalf of the function, adding capabilities the function doesn't need to know about."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "A decorator is a function that takes a function and returns an enhanced version."

```
┌──────────────────────────────────────┐
│          BEFORE DECORATOR            │
│                                      │
│  get_revenue() → result              │
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│          AFTER @timer                │
│                                      │
│  ⏱️ start timer                      │
│  │   get_revenue() → result          │
│  ⏱️ stop timer, log elapsed          │
│  return result                       │
└──────────────────────────────────────┘
```

**The fundamental pattern:**

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        # Before the original function
        result = func(*args, **kwargs)
        # After the original function
        return result
    return wrapper
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Functions Are Objects — The Foundation ⭐

```python
# Functions are first-class objects in Python
def greet(name):
    return f"Hello, {name}!"

# 1. Assign to a variable
say_hello = greet
print(say_hello("Harvey"))    # Hello, Harvey!

# 2. Pass as an argument
def call_twice(func, arg):
    return func(arg) + " " + func(arg)

print(call_twice(greet, "Mike"))    # Hello, Mike! Hello, Mike!

# 3. Return from a function
def make_greeter(greeting):
    def greeter(name):
        return f"{greeting}, {name}!"
    return greeter

formal = make_greeter("Good evening")
print(formal("Jessica"))    # Good evening, Jessica!

# 4. Store in data structures
handlers = {"greet": greet, "formal": formal}
print(handlers["greet"]("Louis"))    # Hello, Louis!
```

---

### 3.2 Your First Decorator ⭐⭐

```python
import functools

def timer(func):
    """Measure execution time of any function."""
    @functools.wraps(func)    # Preserves original function's identity
    def wrapper(*args, **kwargs):
        import time
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"  ⏱️ {func.__name__}: {elapsed:.4f}s")
        return result
    return wrapper

# Apply with @syntax
@timer
def process_clients(n):
    """Process n clients."""
    return sum(range(n))

# @timer is syntactic sugar for:
# process_clients = timer(process_clients)

result = process_clients(1_000_000)
# ⏱️ process_clients: 0.0312s

# The function's identity is preserved thanks to @functools.wraps
print(process_clients.__name__)    # process_clients (not "wrapper")
print(process_clients.__doc__)     # Process n clients.
```

---

### 3.3 `@functools.wraps` — Why It Matters 🔥

```python
import functools

# ❌ WITHOUT @functools.wraps
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def my_func():
    """My docstring."""
    pass

print(my_func.__name__)   # "wrapper" ← WRONG!
print(my_func.__doc__)    # None ← LOST!
help(my_func)             # Shows wrapper, not my_func!

# ✅ WITH @functools.wraps
def good_decorator(func):
    @functools.wraps(func)    # Copies __name__, __doc__, __module__, etc.
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@good_decorator
def my_func():
    """My docstring."""
    pass

print(my_func.__name__)   # "my_func" ✅
print(my_func.__doc__)    # "My docstring." ✅

# RULE: Always use @functools.wraps in every decorator!
```

---

### 3.4 Common Decorator Patterns ⭐⭐

```python
import functools
import time
import logging

logger = logging.getLogger(__name__)

# === 1. Logging decorator ===
def log_call(func):
    """Log function entry, exit, and errors."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logger.info(f"→ {func.__name__}({args}, {kwargs})")
        try:
            result = func(*args, **kwargs)
            logger.info(f"← {func.__name__} returned {result!r}")
            return result
        except Exception as e:
            logger.error(f"✗ {func.__name__} raised {e!r}")
            raise
    return wrapper

# === 2. Retry decorator ===
def retry(max_attempts=3, delay=1):
    """Retry on failure with configurable attempts."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts:
                        raise
                    print(f"  ⚠️ Attempt {attempt} failed: {e}. Retrying in {delay}s...")
                    time.sleep(delay)
        return wrapper
    return decorator

# === 3. Cache decorator ===
def cache(func):
    """Simple memoization cache."""
    _cache = {}
    @functools.wraps(func)
    def wrapper(*args):
        if args not in _cache:
            _cache[args] = func(*args)
        return _cache[args]
    wrapper.cache = _cache           # Expose cache for inspection
    wrapper.clear_cache = _cache.clear
    return wrapper

# === 4. Validation decorator ===
def validate_positive(*param_names):
    """Ensure specified parameters are positive."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            import inspect
            sig = inspect.signature(func)
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            
            for name in param_names:
                value = bound.arguments.get(name)
                if value is not None and value <= 0:
                    raise ValueError(f"{name} must be positive, got {value}")
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

# === Usage ===
@log_call
@retry(max_attempts=3, delay=0.5)
def fetch_data(url):
    """Fetch data from a URL."""
    return {"status": "ok"}

@cache
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

@validate_positive("hours", "rate")
def calculate_billing(client, hours, rate):
    return hours * rate
```

---

### 3.5 Decorators with Arguments ⭐⭐⭐

```python
import functools

# A decorator WITH arguments needs THREE levels of nesting:
# 1. Outer function receives the ARGUMENTS
# 2. Middle function receives the FUNCTION
# 3. Inner function IS the wrapper

def require_permission(permission):
    """Only allow users with the specified permission."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Assume first arg or kwarg 'user' has permissions
            user = kwargs.get("user") or (args[0] if args else None)
            
            if user and hasattr(user, 'permissions'):
                if permission not in user.permissions:
                    raise PermissionError(
                        f"User '{user.name}' lacks '{permission}' permission"
                    )
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@require_permission("billing_read")
def get_revenue(user, client_id):
    return f"Revenue for {client_id}: $2,500,000"

# How the three layers work:
# require_permission("billing_read")  → returns decorator
# decorator(get_revenue)              → returns wrapper
# wrapper(user, "WAYNE")             → checks permission, calls get_revenue

# Another example: rate limiter
import time

def rate_limit(calls_per_second=1):
    """Limit how often a function can be called."""
    min_interval = 1.0 / calls_per_second
    
    def decorator(func):
        last_called = [0.0]    # Mutable container for closure
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.perf_counter() - last_called[0]
            if elapsed < min_interval:
                wait = min_interval - elapsed
                time.sleep(wait)
            
            last_called[0] = time.perf_counter()
            return func(*args, **kwargs)
        
        return wrapper
    return decorator

@rate_limit(calls_per_second=2)
def api_call(endpoint):
    return f"Response from {endpoint}"
```

---

### 3.6 Stacking Multiple Decorators ⭐

```python
import functools
import time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"  ⏱️ {func.__name__}: {time.perf_counter() - start:.4f}s")
        return result
    return wrapper

def log_call(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"  📝 Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"  📝 {func.__name__} returned {result!r}")
        return result
    return wrapper

def uppercase_result(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper() if isinstance(result, str) else result
    return wrapper

# Stacking order matters! Bottom-up application:
@timer              # 3rd: outermost — measures total time
@log_call           # 2nd: middle — logs the call
@uppercase_result   # 1st: innermost — modifies result
def get_greeting(name):
    return f"Hello, {name}!"

# Equivalent to:
# get_greeting = timer(log_call(uppercase_result(get_greeting)))

print(get_greeting("Harvey"))
# 📝 Calling get_greeting
# 📝 get_greeting returned 'HELLO, HARVEY!'
# ⏱️ get_greeting: 0.0001s
# HELLO, HARVEY!
```

**Execution order:**

```
timer.wrapper() called
  → log_call.wrapper() called
    → uppercase_result.wrapper() called
      → get_greeting("Harvey") called → "Hello, Harvey!"
    → uppercase_result converts → "HELLO, HARVEY!"
  → log_call logs result
→ timer logs elapsed time
```

---

### 3.7 Class-Based Decorators

```python
import functools
import time

class Timer:
    """Decorator as a class — reusable and stateful."""
    
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.call_count = 0
        self.total_time = 0
    
    def __call__(self, *args, **kwargs):
        self.call_count += 1
        start = time.perf_counter()
        result = self.func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        self.total_time += elapsed
        print(f"  ⏱️ {self.func.__name__} call #{self.call_count}: {elapsed:.4f}s")
        return result
    
    @property
    def average_time(self):
        return self.total_time / self.call_count if self.call_count else 0
    
    def reset(self):
        self.call_count = 0
        self.total_time = 0

@Timer
def sort_data(n):
    return sorted(range(n, 0, -1))

sort_data(100_000)
sort_data(200_000)
sort_data(300_000)

print(f"  Calls: {sort_data.call_count}")
print(f"  Avg: {sort_data.average_time:.4f}s")
print(f"  Total: {sort_data.total_time:.4f}s")
```

---

### 3.8 Class-Based Decorators WITH Arguments

```python
import functools

class Retry:
    """Retry decorator as a class — with arguments."""
    
    def __init__(self, max_attempts=3, exceptions=(Exception,)):
        # When used with arguments, __init__ receives the ARGUMENTS
        self.max_attempts = max_attempts
        self.exceptions = exceptions
    
    def __call__(self, func):
        # __call__ receives the FUNCTION
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, self.max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except self.exceptions as e:
                    if attempt == self.max_attempts:
                        raise
                    print(f"  ⚠️ Attempt {attempt}/{self.max_attempts}: {e}")
        return wrapper

@Retry(max_attempts=3, exceptions=(ConnectionError, TimeoutError))
def connect_to_server(host):
    import random
    if random.random() < 0.7:
        raise ConnectionError(f"Cannot reach {host}")
    return f"Connected to {host}"

# Class decorator vs function decorator:
# Without args: __init__(func), __call__(*args) is the wrapper
# With args: __init__(**decorator_args), __call__(func) returns wrapper
```

---

### 3.9 Built-in and Standard Library Decorators

```python
# Built-in decorators you already know:
class Example:
    @staticmethod               # No self parameter
    def utility(): ...
    
    @classmethod                # First arg is cls
    def from_string(cls, s): ...
    
    @property                   # Getter
    def name(self): ...
    
    @name.setter                # Setter
    def name(self, value): ...

# functools decorators:
from functools import lru_cache, cache, cached_property, wraps, total_ordering

# lru_cache — automatic memoization with LRU eviction
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2: return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(100))     # Instant! (cached)
print(fibonacci.cache_info())
# CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)

# cache — unbounded cache (Python 3.9+)
@cache
def expensive(n):
    return sum(range(n))

# total_ordering — generate all comparisons from __eq__ and one of __lt__, etc.
@total_ordering
class Priority:
    def __init__(self, level):
        self.level = level
    def __eq__(self, other):
        return self.level == other.level
    def __lt__(self, other):
        return self.level < other.level
    # __le__, __gt__, __ge__ are auto-generated!

# dataclasses decorator (Python 3.7+)
from dataclasses import dataclass

@dataclass
class Client:
    name: str
    revenue: float
    tier: str = "Bronze"
    
    # Auto-generates: __init__, __repr__, __eq__
```

---

### 3.10 Solving Harvey's Problem — Decorator Toolkit

```python
# ============================================
# Pearson Specter Litt — Decorator Toolkit
# Authorization, logging, timing, caching — all as decorators
# ============================================

import functools
import time
import logging
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format="  %(asctime)s [%(levelname)s] %(message)s",
    datefmt="%H:%M:%S"
)
logger = logging.getLogger("firm")

# === Decorators ===

def timer(func):
    """Log execution time."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        logger.debug(f"⏱️ {func.__name__}: {elapsed:.4f}s")
        return result
    return wrapper

def log_call(func):
    """Log function calls and results."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logger.info(f"→ {func.__name__} called")
        try:
            result = func(*args, **kwargs)
            logger.info(f"← {func.__name__} completed")
            return result
        except Exception as e:
            logger.error(f"✗ {func.__name__} raised {type(e).__name__}: {e}")
            raise
    return wrapper

def require_role(*roles):
    """Restrict function access to specified roles."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            user_role = kwargs.get("role", "Intern")
            if user_role not in roles:
                raise PermissionError(
                    f"Role '{user_role}' not authorized. Required: {', '.join(roles)}"
                )
            return func(*args, **kwargs)
        return wrapper
    return decorator

def validate_args(**validators):
    """Validate function arguments with custom validators."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            import inspect
            sig = inspect.signature(func)
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            
            for param_name, validator in validators.items():
                if param_name in bound.arguments:
                    value = bound.arguments[param_name]
                    if not validator(value):
                        raise ValueError(
                            f"Validation failed for '{param_name}': {value!r}"
                        )
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

def cache_result(func):
    """Cache results with hit/miss tracking."""
    _cache = {}
    _stats = {"hits": 0, "misses": 0}
    
    @functools.wraps(func)
    def wrapper(*args):
        if args in _cache:
            _stats["hits"] += 1
            return _cache[args]
        _stats["misses"] += 1
        result = func(*args)
        _cache[args] = result
        return result
    
    wrapper.cache_stats = lambda: dict(_stats)
    wrapper.clear_cache = lambda: (_cache.clear(), _stats.update(hits=0, misses=0))
    return wrapper


# === Business functions with decorators applied ===

@log_call
@timer
@require_role("Partner", "Associate")
@validate_args(hours=lambda h: h >= 0, rate=lambda r: r > 0)
def calculate_billing(client, hours, rate, role="Intern"):
    """Calculate billing for a client."""
    return round(hours * rate * 1.09, 2)

@cache_result
def get_client_tier(revenue):
    """Classify client tier — cached for repeated lookups."""
    if revenue >= 1_000_000: return "Platinum"
    elif revenue >= 500_000: return "Gold"
    elif revenue >= 100_000: return "Silver"
    return "Bronze"


# === Demonstrate ===
print("=" * 60)
print(f"{'🏛️  DECORATOR TOOLKIT':^60}")
print("=" * 60)

# Authorized call
print("\n✅ AUTHORIZED CALL:")
result = calculate_billing("Wayne", hours=85, rate=450, role="Partner")
print(f"  Result: ${result:,.2f}")

# Unauthorized call
print("\n❌ UNAUTHORIZED CALL:")
try:
    calculate_billing("Wayne", hours=85, rate=450, role="Intern")
except PermissionError as e:
    print(f"  {e}")

# Validation failure
print("\n❌ VALIDATION FAILURE:")
try:
    calculate_billing("Wayne", hours=-5, rate=450, role="Partner")
except ValueError as e:
    print(f"  {e}")

# Caching
print("\n📦 CACHING:")
revenues = [2500000, 800000, 2500000, 200000, 800000, 2500000]
for rev in revenues:
    tier = get_client_tier(rev)
    print(f"  ${rev:>10,} → {tier}")
print(f"  Cache stats: {get_client_tier.cache_stats()}")

# Identity preserved
print(f"\n🔍 FUNCTION IDENTITY:")
print(f"  Name: {calculate_billing.__name__}")
print(f"  Doc: {calculate_billing.__doc__}")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: Forgetting `@functools.wraps` 🔥

```python
# ❌ Without wraps — function identity is lost!
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def my_func():
    """Important docstring."""
    pass

print(my_func.__name__)    # "wrapper" ← WRONG!
print(my_func.__doc__)     # None ← LOST!

# ✅ ALWAYS use @functools.wraps
def good_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

---

### Pitfall 2: Forgetting to Return the Result

```python
# ❌ Missing return — function always returns None!
def bad_log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        func(*args, **kwargs)    # Result is LOST!
    return wrapper

@bad_log
def add(a, b):
    return a + b

print(add(1, 2))    # None! Not 3!

# ✅ Return the result!
def good_log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)    # Capture result
        return result                      # Return it!
    return wrapper
```

---

### Pitfall 3: Stacking Order Confusion

```python
# Decorators are applied BOTTOM-UP but execute TOP-DOWN

@A    # Applied 3rd, executes 1st (outermost)
@B    # Applied 2nd, executes 2nd
@C    # Applied 1st, executes 3rd (innermost)
def func(): ...

# Equivalent to: func = A(B(C(func)))
# Execution: A.wrapper → B.wrapper → C.wrapper → func
```

---

### Pitfall 4: Mutable Default in Closure

```python
# ❌ Shared mutable state between decorated calls
def count_calls(func):
    count = 0    # This lives in the closure
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        nonlocal count    # Must use nonlocal to modify!
        count += 1
        print(f"  Call #{count}")
        return func(*args, **kwargs)
    wrapper.get_count = lambda: count
    return wrapper

# Without 'nonlocal': UnboundLocalError!
```

---

### Pitfall 5: Decorating Methods in Classes

```python
import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"  Decorated: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

class Firm:
    @my_decorator
    def greet(self):    # self is passed as first arg via *args — works!
        return "Hello from the firm"
    
    @staticmethod
    @my_decorator       # ⚠️ @staticmethod must be outermost!
    def utility():
        return "Utility function"

    # ❌ Wrong order:
    # @my_decorator     # This would wrap staticmethod, not the function!
    # @staticmethod
    # def utility(): ...

firm = Firm()
print(firm.greet())      # ✅ Works
print(Firm.utility())    # ✅ Works
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📜 Analogy: Decorators = Power of Attorney

```
WITHOUT DECORATORS:
  Every time Harvey makes a deal, he must:
    1. Check client authorization     (auth boilerplate)
    2. Log the meeting                (logging boilerplate)
    3. Time the billable hours        (timing boilerplate)
    4. Actually negotiate the deal    (THE REAL WORK)
    5. File the record                (more boilerplate)

WITH DECORATORS:
  Donna handles steps 1, 2, 3, and 5.
  Harvey ONLY does step 4 — the actual work.

  @donna_handles_auth
  @donna_handles_logging
  @donna_handles_timing
  def negotiate_deal(client):
      # Harvey just does his thing
      return close_the_deal(client)

DONNA'S RULE:
  "A decorator is a power of attorney.
   It acts ON BEHALF of the function.
   The function does its job.
   The decorator handles everything around it."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Simple Logger**
```
Write @log_call that prints:
  "Calling function_name(args)"
  "function_name returned result"
Apply to 3 different functions.
```

**Exercise 2: Timer**
```
Write @timer that measures and prints execution time.
Test with functions of varying complexity.
```

**Exercise 3: Type Checker**
```
Write @enforce_types that checks argument types:

@enforce_types(name=str, hours=(int, float), rate=(int, float))
def bill(name, hours, rate):
    return hours * rate
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Retry with Backoff**
```
Write @retry(max_attempts, backoff_factor):
  - Retries on exception
  - Exponential backoff: delay = backoff_factor ** attempt
  - Configurable exception types
```

**Exercise 5: Access Control**
```
Write @require_auth(min_level):
  - Checks user.auth_level >= min_level
  - Raises PermissionError with clear message
  - Logs all access attempts
```

**Exercise 6: Result Transformer**
```
Write @transform(func) that applies func to the result:

@transform(str.upper)
def get_name():
    return "harvey specter"    # Returns "HARVEY SPECTER"

@transform(sorted)
def get_items():
    return [3, 1, 2]          # Returns [1, 2, 3]
```

---

### 🔴 Advanced Exercises

**Exercise 7: Decorator Registry**
```
Build a decorator that registers functions:

@registry.register("billing")
def calc_billing(): ...

@registry.register("client")
def get_client(): ...

registry.list_all()    # {"billing": calc_billing, "client": get_client}
registry.call("billing", args)
```

**Exercise 8: Circuit Breaker**
```
Build @circuit_breaker(threshold=5, reset_timeout=60):
  - After N failures, "open" the circuit (reject calls)
  - After timeout, allow one test call ("half-open")
  - On success, "close" the circuit
  - Track state: CLOSED → OPEN → HALF_OPEN → CLOSED
```

**Exercise 9: Decorator Composition**
```
Build compose_decorators(*decorators) that combines:

secure_endpoint = compose_decorators(
    require_auth("admin"),
    rate_limit(10),
    log_call,
    timer,
)

@secure_endpoint
def admin_action(): ...
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: No @functools.wraps
def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
# __name__ and __doc__ are lost!

# Bug 2: Not returning the result
def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)    # Result dropped!
    return wrapper

# Bug 3: Decorator with args — wrong nesting
def repeat(n):
    def wrapper(*args, **kwargs):    # ❌ Missing middle layer!
        for _ in range(n):
            func(*args, **kwargs)   # NameError: func not defined!
    return wrapper

# Bug 4: Stacking order
@staticmethod
@my_decorator      # ❌ Decorates staticmethod descriptor, not the function!
def utility(): ...
```

### Fixed Code ✅

```python
# Fix 1: Add @functools.wraps
def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# Fix 2: Return result
def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result    # ✅

# Fix 3: Three layers for decorator with args
def repeat(n):
    def decorator(func):    # ← Middle layer receives func
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                func(*args, **kwargs)
        return wrapper
    return decorator

# Fix 4: @staticmethod outermost
@staticmethod
@my_decorator    # ❌
# Should be:
@my_decorator
@staticmethod    # ❌ — actually neither works perfectly
# Best: decorate before assigning as staticmethod
```

---

## 8. 🌍 Real-World Application

### Decorators in Popular Frameworks

| Framework | Decorator | Purpose |
|-----------|-----------|---------|
| **Flask** | `@app.route("/")` | URL routing |
| **Django** | `@login_required` | Auth protection |
| **FastAPI** | `@app.get("/items")` | API endpoints |
| **pytest** | `@pytest.fixture` | Test fixtures |
| **functools** | `@lru_cache` | Memoization |
| **dataclasses** | `@dataclass` | Auto-generate methods |
| **abc** | `@abstractmethod` | Abstract methods |
| **contextlib** | `@contextmanager` | Context managers |
| **typing** | `@overload` | Type overloads |
| **click** | `@click.command()` | CLI commands |

---

## 9. ✅ Knowledge Check

---

**Q1.** What is a decorator?

**Q2.** What does `@decorator` syntax actually do?

**Q3.** Why must you always use `@functools.wraps`?

**Q4.** How many nesting levels does a decorator WITH arguments need?

**Q5.** In what order are stacked decorators applied vs executed?

**Q6.** What is the difference between a function decorator and a class decorator?

**Q7.** How do you write a decorator that works on class methods?

**Q8.** What does `@lru_cache` do?

**Q9.** Why must the wrapper use `*args, **kwargs`?

**Q10.** What happens if you forget to return the result inside the wrapper?

---

### 📋 Answer Key

> **Q1.** A function (or class) that takes a function as input and returns an enhanced version of it, adding behavior without modifying the original code.

> **Q2.** `@decorator` above `def func` is syntactic sugar for `func = decorator(func)`.

> **Q3.** Without it, the wrapper replaces the function's `__name__`, `__doc__`, and `__module__`. `@wraps` copies these attributes to preserve identity.

> **Q4.** Three: outer (receives arguments) → middle (receives function) → inner (the wrapper that runs).

> **Q5.** Applied bottom-up (innermost first), executed top-down (outermost first). `@A @B @C def f` = `f = A(B(C(f)))`.

> **Q6.** Function decorator: `def decorator(func): def wrapper: ... return wrapper`. Class decorator: `class Decorator: def __init__(self, func): ... def __call__: ...` — can store state.

> **Q7.** Use `*args, **kwargs` — `self` is automatically passed as the first positional argument.

> **Q8.** Least Recently Used cache — memoizes function results. `@lru_cache(maxsize=128)` caches up to 128 results.

> **Q9.** To accept any function signature — the decorator doesn't know what arguments the decorated function takes.

> **Q10.** The decorated function always returns `None`, regardless of what the original function returns.

---

## 🎬 End of Lesson Scene

**Harvey:** "How many functions did you update?"

**Mike:** "Zero. I wrote four decorators — `@timer`, `@log_call`, `@require_role`, `@validate_args`. Then I added one line above each of the 200 functions. The functions themselves are untouched."

**Harvey:** "And if we need to change the auth logic?"

**Mike:** "One decorator. One change. 200 functions updated instantly."

**Harvey:** "That's leverage. Next: closures and scoping — the engine that makes decorators possible."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Functions as first-class objects | ✅ |
| 2 | Basic decorator pattern | ✅ |
| 3 | `@functools.wraps` — preserving identity | ✅ |
| 4 | Common patterns (logging, timing, retry, cache, validation) | ✅ |
| 5 | Decorators with arguments (three-level nesting) | ✅ |
| 6 | Stacking decorators (order of application vs execution) | ✅ |
| 7 | Class-based decorators (stateful) | ✅ |
| 8 | Built-in decorators (`@lru_cache`, `@dataclass`, etc.) | ✅ |
| 9 | Decorating class methods (`@staticmethod` ordering) | ✅ |
| 10 | Real-world frameworks (Flask `@route`, pytest `@fixture`) | ✅ |

---

> **Next Lesson →** Level 4, Lesson 2: *Use Closures and Understand Scoping Rules in Python*

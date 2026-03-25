# 🏛️ Level 3 – Building | Lesson 10: Use Python's logging Module

## *"The Official Record"*

> **O'Reilly Skill Objective:** *Use Python's `logging` module to add structured, configurable logging to applications.*

> **Previously Learned:** `print()`, f-strings, exception handling, context managers, file I/O, decorators, classes

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

*3 AM. Production billing is down. No one knows why.*

**Louis:** "When did it start failing? What was the last operation? What data was it processing? What error occurred?"

**Mike:** "I don't know. We use `print()` statements for debugging, and print goes to the console. There IS no console in production. The output vanished."

**Jessica:** "In a courtroom, every word is transcribed. Every exhibit is logged. If it's not on the record, it didn't happen. Your code needs the same — official, permanent, structured records."

**Harvey:** "`print()` is talking to yourself. `logging` is filing court records."

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "Five levels of severity. One system. Total control."

| Level | Numeric | When to Use |
|:-----:|:-------:|-------------|
| `DEBUG` | 10 | Detailed diagnostic info — only during development |
| `INFO` | 20 | Confirmation things are working as expected |
| `WARNING` | 30 | Something unexpected but not broken (yet) |
| `ERROR` | 40 | Something failed — the operation couldn't complete |
| `CRITICAL` | 50 | System-level failure — the application may crash |

```python
import logging

logging.debug("Checking client data...")             # 🔍 Developer only
logging.info("Loaded 42 clients")                     # ✅ Normal operation
logging.warning("Client 'Wayne' has no billing rate")  # ⚠️ Unusual
logging.error("Failed to process invoice #1234")       # ❌ Operation failed
logging.critical("Database connection lost!")           # 🔥 System failure
```

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Basic Logging — Beyond `print()` ⭐

```python
import logging

# Default: only WARNING and above are shown
logging.warning("This will show")
logging.info("This will NOT show (below WARNING)")

# Configure to show all levels
logging.basicConfig(level=logging.DEBUG)

logging.debug("Now this shows too!")
logging.info("And this!")
logging.warning("And this!")

# Output:
# WARNING:root:This will show
# DEBUG:root:Now this shows too!
# INFO:root:And this!
# WARNING:root:And this!
```

**Why not `print()`?**

| Feature | `print()` | `logging` |
|---------|:---------:|:---------:|
| Severity levels | ❌ | ✅ DEBUG/INFO/WARNING/ERROR/CRITICAL |
| Timestamps | ❌ | ✅ Configurable |
| Source file/line | ❌ | ✅ Automatic |
| Output to file | Manual | ✅ Built-in |
| On/off by level | ❌ | ✅ Per-module control |
| Multiple outputs | ❌ | ✅ Console + file + email |
| Thread-safe | ❌ | ✅ |
| Production-ready | ❌ | ✅ |

---

### 3.2 `basicConfig` — Quick Setup ⭐

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)8s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logging.info("Application started")
logging.debug("Loading configuration...")
logging.warning("Using default database")
logging.error("Connection timeout after 30s")

# Output:
# 2026-03-25 09:38:19 [    INFO] root: Application started
# 2026-03-25 09:38:19 [   DEBUG] root: Loading configuration...
# 2026-03-25 09:38:19 [ WARNING] root: Using default database
# 2026-03-25 09:38:19 [   ERROR] root: Connection timeout after 30s
```

**Common format codes:**

| Code | Output | Example |
|------|--------|---------|
| `%(asctime)s` | Timestamp | `2026-03-25 09:38:19` |
| `%(levelname)s` | Level name | `WARNING` |
| `%(name)s` | Logger name | `billing.processor` |
| `%(module)s` | Module name | `processor` |
| `%(funcName)s` | Function name | `process_invoice` |
| `%(lineno)d` | Line number | `42` |
| `%(message)s` | The log message | `Client loaded` |
| `%(filename)s` | File name | `processor.py` |
| `%(pathname)s` | Full path | `/app/billing/processor.py` |

---

### 3.3 Named Loggers — Per-Module Logging ⭐⭐

```python
import logging

# Create named loggers for each module
billing_logger = logging.getLogger("billing")
client_logger = logging.getLogger("client")
auth_logger = logging.getLogger("auth")

# Configure
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%H:%M:%S",
)

# Each logger identifies its source
billing_logger.info("Invoice #1234 created")
client_logger.info("Loaded 42 clients")
auth_logger.warning("Failed login attempt from 192.168.1.1")

# Output:
# 09:38:19 [INFO] billing: Invoice #1234 created
# 09:38:19 [INFO] client: Loaded 42 clients
# 09:38:19 [WARNING] auth: Failed login attempt from 192.168.1.1

# Best practice: use __name__
logger = logging.getLogger(__name__)
# In billing/processor.py → logger name = "billing.processor"
```

**Logger hierarchy:**

```
root                    ← logging.getLogger()
 ├── billing            ← logging.getLogger("billing")
 │    ├── billing.processor    ← logging.getLogger("billing.processor")
 │    └── billing.invoice      ← logging.getLogger("billing.invoice")
 ├── client             ← logging.getLogger("client")
 └── auth               ← logging.getLogger("auth")
```

> Loggers inherit settings from parents. Setting `billing` to `DEBUG` affects `billing.processor` too.

---

### 3.4 Handlers — Where Logs Go ⭐⭐

```python
import logging
from logging import FileHandler, StreamHandler
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

logger = logging.getLogger("firm")
logger.setLevel(logging.DEBUG)

# === Console handler (INFO and above) ===
console_handler = StreamHandler()
console_handler.setLevel(logging.INFO)
console_format = logging.Formatter("%(asctime)s [%(levelname)s] %(message)s", "%H:%M:%S")
console_handler.setFormatter(console_format)
logger.addHandler(console_handler)

# === File handler (DEBUG and above) ===
file_handler = FileHandler("firm.log", mode="a")
file_handler.setLevel(logging.DEBUG)
file_format = logging.Formatter(
    "%(asctime)s [%(levelname)8s] %(name)s.%(funcName)s:%(lineno)d — %(message)s"
)
file_handler.setFormatter(file_format)
logger.addHandler(file_handler)

# === Rotating file handler (prevents huge log files) ===
rotating_handler = RotatingFileHandler(
    "firm_rotating.log",
    maxBytes=5_000_000,    # 5 MB per file
    backupCount=3,          # Keep 3 backup files
)
rotating_handler.setLevel(logging.WARNING)
rotating_handler.setFormatter(file_format)
logger.addHandler(rotating_handler)

# Now: DEBUG → file only, INFO → file + console, WARNING → all three!
logger.debug("Detailed trace info")        # File only
logger.info("Client Wayne loaded")         # Console + file
logger.warning("Rate not set!")            # Console + file + rotating
logger.error("Invoice processing failed")  # Console + file + rotating

# Clean up
import os
for f in ["firm.log", "firm_rotating.log"]:
    if os.path.exists(f): os.remove(f)
```

---

### 3.5 Formatters — Customizing Output ⭐

```python
import logging

# Simple format
simple = logging.Formatter("%(levelname)s: %(message)s")
# WARNING: Something happened

# Detailed format
detailed = logging.Formatter(
    "%(asctime)s | %(levelname)-8s | %(name)-15s | %(funcName)-20s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)
# 2026-03-25 09:38:19 | WARNING  | billing         | process_invoice      | Rate missing

# JSON format (for log aggregation tools)
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": datetime.fromtimestamp(record.created).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
            "message": record.getMessage(),
        }
        if record.exc_info:
            log_data["exception"] = self.formatException(record.exc_info)
        return json.dumps(log_data)

# Usage:
logger = logging.getLogger("json_demo")
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.info("Client loaded")
# {"timestamp": "2026-03-25T09:38:19", "level": "INFO", ...}
```

---

### 3.6 Logging Exceptions ⭐

```python
import logging

logger = logging.getLogger("billing")
logging.basicConfig(level=logging.DEBUG)

# Log exceptions with full traceback
def process_invoice(client, hours, rate):
    try:
        total = hours * rate
        if total > 1_000_000:
            raise ValueError(f"Suspiciously high invoice: ${total:,.2f}")
        return total
    except ValueError:
        # exc_info=True adds the full traceback!
        logger.error("Invoice processing failed", exc_info=True)
        return None
    except Exception:
        # Or use logger.exception() — shorthand for error + exc_info
        logger.exception("Unexpected error in process_invoice")
        return None

process_invoice("Wayne", 10000, 500)

# Output:
# ERROR:billing:Invoice processing failed
# Traceback (most recent call last):
#   File "script.py", line 8, in process_invoice
#     raise ValueError(f"Suspiciously high invoice: ${total:,.2f}")
# ValueError: Suspiciously high invoice: $5,000,000.00
```

---

### 3.7 Structured Logging — Extra Fields

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(message)s | client=%(client)s amount=%(amount)s",
    datefmt="%H:%M:%S",
)

logger = logging.getLogger("billing")

# Pass extra fields
logger.info(
    "Invoice created",
    extra={"client": "Wayne", "amount": "$38,250"}
)
# 09:38:19 [INFO] Invoice created | client=Wayne amount=$38,250

# Using LoggerAdapter for automatic context
class ClientAdapter(logging.LoggerAdapter):
    def process(self, msg, kwargs):
        return f"[{self.extra['client']}] {msg}", kwargs

adapter = ClientAdapter(logger, {"client": "Wayne Enterprises"})
adapter.info("Invoice generated")
adapter.warning("Payment overdue")
# [Wayne Enterprises] Invoice generated
# [Wayne Enterprises] Payment overdue
```

---

### 3.8 Configuration via Dictionary ⭐

```python
import logging
import logging.config

LOGGING_CONFIG = {
    "version": 1,
    "disable_existing_loggers": False,
    
    "formatters": {
        "standard": {
            "format": "%(asctime)s [%(levelname)8s] %(name)s: %(message)s",
            "datefmt": "%Y-%m-%d %H:%M:%S",
        },
        "detailed": {
            "format": "%(asctime)s [%(levelname)s] %(name)s.%(funcName)s:%(lineno)d — %(message)s",
        },
    },
    
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "INFO",
            "formatter": "standard",
            "stream": "ext://sys.stdout",
        },
        "file": {
            "class": "logging.FileHandler",
            "level": "DEBUG",
            "formatter": "detailed",
            "filename": "app.log",
            "mode": "a",
        },
    },
    
    "loggers": {
        "billing": {
            "level": "DEBUG",
            "handlers": ["console", "file"],
            "propagate": False,
        },
        "auth": {
            "level": "WARNING",
            "handlers": ["console"],
            "propagate": False,
        },
    },
    
    "root": {
        "level": "WARNING",
        "handlers": ["console"],
    },
}

logging.config.dictConfig(LOGGING_CONFIG)

# Now each module uses its configured logger
billing = logging.getLogger("billing")
auth = logging.getLogger("auth")

billing.debug("This goes to file only")
billing.info("This goes to console AND file")
auth.info("This is suppressed (auth level is WARNING)")
auth.warning("This shows on console")
```

---

### 3.9 Logging Decorator

```python
import logging
import functools
import time

def log_call(logger=None, level=logging.INFO):
    """Decorator that logs function calls, returns, and errors."""
    def decorator(func):
        _logger = logger or logging.getLogger(func.__module__)
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            func_name = func.__qualname__
            _logger.log(level, f"→ {func_name}({args}, {kwargs})")
            start = time.perf_counter()
            
            try:
                result = func(*args, **kwargs)
                elapsed = time.perf_counter() - start
                _logger.log(level, f"← {func_name} returned {result!r} [{elapsed:.4f}s]")
                return result
            except Exception as e:
                elapsed = time.perf_counter() - start
                _logger.error(f"✗ {func_name} raised {type(e).__name__}: {e} [{elapsed:.4f}s]")
                raise
        
        return wrapper
    return decorator

# Usage:
logging.basicConfig(level=logging.DEBUG, format="%(asctime)s %(message)s", datefmt="%H:%M:%S")

@log_call()
def calculate_billing(client, hours, rate):
    if hours < 0:
        raise ValueError("Negative hours!")
    return hours * rate

calculate_billing("Wayne", 85, 450)
# 09:38:19 → calculate_billing(('Wayne', 85, 450), {})
# 09:38:19 ← calculate_billing returned 38250 [0.0001s]
```

---

### 3.10 Solving the Production Problem — Full Logging System

```python
# ============================================
# Pearson Specter Litt — Production Logging System
# Structured, multi-handler, level-filtered logging
# ============================================

import logging
import logging.config
import json
import os
from datetime import datetime
from contextlib import contextmanager

# === JSON Formatter ===
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            "timestamp": datetime.fromtimestamp(record.created).strftime("%Y-%m-%dT%H:%M:%S"),
            "level": record.levelname,
            "logger": record.name,
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
            "message": record.getMessage(),
        }
        if record.exc_info and record.exc_info[0]:
            log_entry["exception"] = self.formatException(record.exc_info)
        if hasattr(record, "client"):
            log_entry["client"] = record.client
        if hasattr(record, "amount"):
            log_entry["amount"] = record.amount
        return json.dumps(log_entry)


# === Setup ===
def setup_logging(log_dir="logs"):
    os.makedirs(log_dir, exist_ok=True)
    
    config = {
        "version": 1,
        "disable_existing_loggers": False,
        "formatters": {
            "console": {
                "format": "  %(asctime)s [%(levelname)8s] %(name)-12s: %(message)s",
                "datefmt": "%H:%M:%S",
            },
            "file": {
                "format": "%(asctime)s [%(levelname)8s] %(name)s.%(funcName)s:%(lineno)d — %(message)s",
                "datefmt": "%Y-%m-%d %H:%M:%S",
            },
        },
        "handlers": {
            "console": {
                "class": "logging.StreamHandler",
                "level": "INFO",
                "formatter": "console",
            },
            "file_debug": {
                "class": "logging.FileHandler",
                "level": "DEBUG",
                "formatter": "file",
                "filename": os.path.join(log_dir, "debug.log"),
                "mode": "w",
            },
            "file_errors": {
                "class": "logging.FileHandler",
                "level": "ERROR",
                "formatter": "file",
                "filename": os.path.join(log_dir, "errors.log"),
                "mode": "w",
            },
        },
        "loggers": {
            "billing": {"level": "DEBUG", "handlers": ["console", "file_debug", "file_errors"]},
            "client": {"level": "DEBUG", "handlers": ["console", "file_debug"]},
            "auth": {"level": "WARNING", "handlers": ["console", "file_errors"]},
        },
        "root": {"level": "WARNING", "handlers": ["console"]},
    }
    logging.config.dictConfig(config)


# === Application Code ===
@contextmanager
def log_operation(logger, operation_name):
    """Context manager that logs operation start/end."""
    logger.info(f"▶ Starting: {operation_name}")
    import time
    start = time.perf_counter()
    try:
        yield
    except Exception as e:
        elapsed = time.perf_counter() - start
        logger.error(f"✗ Failed: {operation_name} ({elapsed:.2f}s)", exc_info=True)
        raise
    else:
        elapsed = time.perf_counter() - start
        logger.info(f"✓ Completed: {operation_name} ({elapsed:.2f}s)")


def process_billing(records):
    """Process billing records with full logging."""
    logger = logging.getLogger("billing")
    
    with log_operation(logger, "Billing Pipeline"):
        total = 0
        successes = 0
        failures = 0
        
        for i, record in enumerate(records, 1):
            try:
                logger.debug(f"Processing record {i}: {record}")
                
                if "client" not in record:
                    raise ValueError("Missing 'client' field")
                if "hours" not in record or "rate" not in record:
                    raise ValueError("Missing billing fields")
                
                amount = record["hours"] * record["rate"]
                
                if amount <= 0:
                    logger.warning(f"Zero/negative amount for {record['client']}: ${amount}")
                
                total += amount
                successes += 1
                logger.info(
                    f"✅ {record['client']}: ${amount:,.2f}",
                    extra={"client": record["client"], "amount": amount}
                )
                
            except (ValueError, TypeError, KeyError) as e:
                failures += 1
                logger.error(f"❌ Record {i}: {e}")
        
        logger.info(f"📊 Results: {successes} ok, {failures} failed, total: ${total:,.2f}")
        return {"total": total, "successes": successes, "failures": failures}


# === Run ===
setup_logging()

print("=" * 60)
print(f"{'🏛️  PRODUCTION LOGGING SYSTEM':^60}")
print("=" * 60)
print()

# Simulate authentication
auth_logger = logging.getLogger("auth")
auth_logger.info("Login attempt (suppressed — auth level is WARNING)")
auth_logger.warning("Failed login attempt from 10.0.0.42")

# Process billing
records = [
    {"client": "Wayne Enterprises", "hours": 85, "rate": 450},
    {"client": "Stark Industries", "hours": 62, "rate": 400},
    {"hours": 55, "rate": 425},                                   # Missing client
    {"client": "Oscorp", "hours": 0, "rate": 450},               # Zero hours
    {"client": "Dalton Industries", "hours": 70, "rate": 450},
]

result = process_billing(records)

# Show log files
print(f"\n📂 Log Files Created:")
for log_file in ["logs/debug.log", "logs/errors.log"]:
    if os.path.exists(log_file):
        with open(log_file) as f:
            lines = f.readlines()
        print(f"  {log_file}: {len(lines)} lines")

# Cleanup
import shutil
if os.path.exists("logs"):
    shutil.rmtree("logs")

print(f"\n{'='*60}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

---

### Pitfall 1: `basicConfig` Only Works Once 🔥

```python
import logging

logging.basicConfig(level=logging.DEBUG)      # ✅ Configures
logging.basicConfig(level=logging.WARNING)    # ❌ Ignored! Already configured!

# Fix: force reconfigure (Python 3.8+)
logging.basicConfig(level=logging.WARNING, force=True)

# Or: use dictConfig / manual handler setup instead
```

---

### Pitfall 2: Using f-strings in Log Messages

```python
import logging
logger = logging.getLogger("app")

client = "Wayne"

# ❌ f-string — string is ALWAYS formatted, even if level is filtered
logger.debug(f"Loading client {client}")   # Formats even if DEBUG is off!

# ✅ %-style — string is only formatted if the message is actually emitted
logger.debug("Loading client %s", client)  # Lazy: only formats if DEBUG is on

# Performance difference with expensive operations:
# ❌ Always evaluates expensive_call()
logger.debug(f"Result: {expensive_call()}")

# ✅ Only evaluates if DEBUG is enabled
logger.debug("Result: %s", expensive_call())

# Even better for expensive computations:
if logger.isEnabledFor(logging.DEBUG):
    logger.debug("Result: %s", expensive_call())
```

---

### Pitfall 3: Propagation Confusion

```python
import logging

# Child logger with its own handler
child = logging.getLogger("app.billing")
child.addHandler(logging.StreamHandler())
child.setLevel(logging.DEBUG)

child.info("Hello!")
# ⚠️ "Hello!" appears TWICE if root also has a handler!
# Once from child's handler, once propagated to root!

# Fix: disable propagation
child.propagate = False
```

---

### Pitfall 4: Missing `exc_info`

```python
import logging
logger = logging.getLogger("app")

try:
    1 / 0
except ZeroDivisionError:
    # ❌ Loses the traceback!
    logger.error("Something went wrong")
    
    # ✅ Includes full traceback
    logger.error("Something went wrong", exc_info=True)
    
    # ✅ Shorthand (error level + exc_info)
    logger.exception("Something went wrong")
```

---

### Pitfall 5: Logging in Libraries vs Applications

```python
# === LIBRARY code (don't configure logging!) ===
# my_library/__init__.py
import logging
logger = logging.getLogger(__name__)
logger.addHandler(logging.NullHandler())  # ← Silence by default!

# Let the APPLICATION configure logging.
# Libraries should NEVER call basicConfig()!

# === APPLICATION code (configure logging here) ===
import logging
logging.basicConfig(level=logging.INFO)
# Now library logs will appear if the app sets the level
```

---

## 5. 🧠 Mental Model — Donna Paulsen

### 📝 Analogy: Logging = The Firm's Official Record System

```
print() = Talking in the hallway
  → No record. No timestamp. No audience control.
  → "Hey Mike, the case is going well" — and it's gone.

logging = The official court record
  → Every statement timestamped.
  → Every speaker identified.
  → Severity classified: INFO, WARNING, ERROR.
  → Permanent record. Filed and searchable.

LEVELS:
  DEBUG    = Internal notes (attorney eyes only)
  INFO     = Case progress updates (team-wide)
  WARNING  = Yellow flags (potential issues)
  ERROR    = Failed motions (something broke)
  CRITICAL = Contempt of court (system failure)

HANDLERS:
  Console   = The courtroom display (live)
  File      = The official transcript (permanent)
  Rotating  = Annual archives (managed size)

DONNA'S RULE:
  "If it's important enough to know later,
   it's important enough to log properly."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

---

### 🟢 Beginner Exercises

**Exercise 1: Basic Application Logger**
```
Set up logging for a simple app:
  - Console: INFO and above
  - File (app.log): DEBUG and above
  - Format: timestamp, level, module, message
  - Log 5 messages at different levels
```

**Exercise 2: Function Logger**
```
Write a function that logs:
  - Entry: function name + arguments
  - Exit: return value + elapsed time
  - Error: exception type + message + traceback
```

**Exercise 3: Level Filter Demo**
```
Create a script that demonstrates all 5 levels.
Run it with different basicConfig levels:
  DEBUG, INFO, WARNING, ERROR, CRITICAL
Show which messages appear at each setting.
```

---

### 🟡 Intermediate Exercises

**Exercise 4: Multi-Module Logger**
```
Create 3 modules: auth.py, billing.py, reports.py
Each with its own logger (logging.getLogger(__name__))
Set different levels per module.
Show logs flowing from children → parent → handlers.
```

**Exercise 5: Log Decorator**
```
Build @log_call that:
  - Logs function entry/exit
  - Logs exceptions with traceback
  - Measures elapsed time
  - Configurable level
  - Works with any function signature
```

**Exercise 6: JSON Log Aggregator**
```
Build a system that:
  - Writes logs as JSON lines to a file
  - Reads the file and parses entries
  - Filters by level, time range, logger name
  - Generates summary: error count, warning count, etc.
```

---

### 🔴 Advanced Exercises

**Exercise 7: Custom Handler**
```
Build a handler that sends ERROR+ logs via email:
  class EmailHandler(logging.Handler):
      def emit(self, record): ...

Include: batching (collect for 5 min), template, rate limiting.
```

**Exercise 8: Structured Logging System**
```
Build a structured logging system with:
  - Context variables (request_id, user_id)
  - Automatic correlation across functions
  - JSON output for log aggregation
  - Sensitive data redaction (passwords, tokens)
```

**Exercise 9: Log Analysis Dashboard**
```
Build a tool that reads log files and generates:
  - Error frequency over time
  - Top 10 error messages
  - Slowest operations (from timing logs)
  - Warning trends
  - Output as formatted table or HTML report
```

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛

```python
# Bug 1: basicConfig called twice
logging.basicConfig(level=logging.DEBUG)
logging.basicConfig(level=logging.WARNING)   # Ignored!
logging.debug("Still shows!")                 # Unexpected!

# Bug 2: f-string performance waste
logger.debug(f"Data: {expensive_serialize(huge_object)}")
# expensive_serialize runs even if DEBUG is disabled!

# Bug 3: No handlers
logger = logging.getLogger("myapp")
logger.warning("Hello?")
# "No handlers could be found for logger 'myapp'" — no output!

# Bug 4: Double logging
child = logging.getLogger("app.module")
child.addHandler(logging.StreamHandler())
logging.basicConfig(level=logging.INFO)
child.info("Appears twice!")   # Once from child, once from root!

# Bug 5: Lost exception info
try:
    1/0
except:
    logging.error("Failed")   # Where's the traceback?!
```

### Fixed Code ✅

```python
# Fix 1: Use force=True or dictConfig
logging.basicConfig(level=logging.WARNING, force=True)

# Fix 2: Use %-formatting (lazy)
logger.debug("Data: %s", expensive_serialize(huge_object))

# Fix 3: Add a handler (or use basicConfig)
logger.addHandler(logging.StreamHandler())

# Fix 4: Disable propagation
child.propagate = False

# Fix 5: Include exc_info
logging.error("Failed", exc_info=True)
# Or: logging.exception("Failed")
```

---

## 8. 🌍 Real-World Application

### Logging in Production

| Tool | What It Does |
|------|-------------|
| **ELK Stack** | Elasticsearch + Logstash + Kibana — log aggregation |
| **Datadog** | Cloud monitoring with log integration |
| **Sentry** | Error tracking with full context |
| **structlog** | Structured logging library for Python |
| **loguru** | Drop-in replacement with better defaults |
| **CloudWatch** | AWS log management |
| **Fluentd** | Log collector and forwarder |

---

## 9. ✅ Knowledge Check

---

**Q1.** What are the 5 standard logging levels, from lowest to highest?

**Q2.** Why is `logging` better than `print()` for production code?

**Q3.** What does `logging.basicConfig()` do?

**Q4.** What is a handler and how do you use one?

**Q5.** Why should you use `logger.error("msg %s", var)` instead of f-strings?

**Q6.** How do you include a full traceback in a log message?

**Q7.** What is `logging.getLogger(__name__)` and why is it the best practice?

**Q8.** What is log propagation?

**Q9.** Why should libraries use `NullHandler`?

**Q10.** What does `dictConfig` provide over `basicConfig`?

---

### 📋 Answer Key

> **Q1.** DEBUG (10) → INFO (20) → WARNING (30) → ERROR (40) → CRITICAL (50).

> **Q2.** Levels, timestamps, source tracking, file output, thread safety, configurable per-module, can send to multiple destinations.

> **Q3.** Quick one-time setup of the root logger — sets level, format, and output destination. Only works once unless `force=True`.

> **Q4.** A handler determines WHERE log messages go — `StreamHandler` (console), `FileHandler` (file), `RotatingFileHandler` (size-limited files). Multiple handlers = multiple destinations.

> **Q5.** %-formatting is lazy — the string is only formatted if the message passes the level filter. f-strings are always evaluated, wasting CPU on filtered messages.

> **Q6.** `logger.error("msg", exc_info=True)` or the shorthand `logger.exception("msg")`.

> **Q7.** Creates a logger named after the current module (e.g., `billing.processor`). Enables per-module level control and clear log source identification.

> **Q8.** Child loggers pass messages UP to parent loggers. `billing.processor` → `billing` → root. Set `propagate = False` to stop this.

> **Q9.** Libraries should never configure logging — that's the application's job. `NullHandler` silences the "no handlers" warning without producing output.

> **Q10.** Full programmatic control: multiple handlers, formatters, loggers, levels — all in one declarative dictionary. Supports complex multi-module configurations.

---

## 🎬 End of Lesson Scene

**Jessica:** "I need to know what happened at 3 AM."

**Mike:** *(opens logs)* "Here — timestamped, leveled, sourced. `09:03:42 [ERROR] billing.processor:87 — Failed to process invoice #1234: missing rate field`. The debug log has every step leading up to it."

**Harvey:** "And next time it happens?"

**Mike:** "Errors go to a separate file. Warning trends are tracked. We can filter by module, level, time range. It's a full court record."

**Harvey:** "Next: unit testing. Proving the code works — with evidence."

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | 5 logging levels (DEBUG → CRITICAL) | ✅ |
| 2 | `basicConfig` quick setup | ✅ |
| 3 | Named loggers (`getLogger(__name__)`) | ✅ |
| 4 | Handlers (Stream, File, Rotating) | ✅ |
| 5 | Formatters (standard + JSON) | ✅ |
| 6 | Exception logging (`exc_info`, `.exception()`) | ✅ |
| 7 | Structured logging with `extra` and `LoggerAdapter` | ✅ |
| 8 | Dictionary-based configuration | ✅ |
| 9 | Logger hierarchy and propagation | ✅ |
| 10 | Library vs application logging best practices | ✅ |

---

> **Next Lesson →** Level 3, Lesson 11: *Write Unit Tests and Validate Code Behavior in Python*

# Phase 2 · Lesson 2 — The `logging` Module
### *"The Official Record"*

> **O'Reilly Skill:** Use Python's logging module to add structured, configurable logging to applications.
> **DevOps connection:** At 3 AM when production is down, `print()` is useless — it went to a console that no longer exists. Logs in files are your crime scene evidence.
> **10-min sprint:** Replace every `print()` in a script you wrote with a `logger.info()` or `logger.debug()`. Run it and see the difference in output.

---

## Why This Matters (Harvey's take)

`print()` is talking to yourself. `logging` is filing court records. When production goes down at 3 AM, you need a permanent, structured, timestamped trail of everything your script did. `logging` gives you that. `print()` gives you nothing.

---

## The Core Concept (Mike's breakdown)

### The 5 Levels

```python
import logging

logging.debug("Detailed trace — developer only")   # level 10
logging.info("Normal operation confirmed")          # level 20
logging.warning("Unexpected but not broken")        # level 30
logging.error("Operation failed")                   # level 40
logging.critical("System is going down!")           # level 50
```

Default: only `WARNING` and above are shown. You need to configure to see `DEBUG`/`INFO`.

### Quick Setup (`basicConfig`)

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logging.info("Script started")
logging.debug("Loading config file...")
logging.warning("No rate found for client — using default")
logging.error("Failed to connect to database")

# 2026-03-25 09:38:19 [INFO] root: Script started
# 2026-03-25 09:38:19 [DEBUG] root: Loading config file...
# 2026-03-25 09:38:19 [WARNING] root: No rate found for client — using default
# 2026-03-25 09:38:19 [ERROR] root: Failed to connect to database
```

### Named Loggers (the right way)

```python
import logging

# Create a named logger — best practice in every file
logger = logging.getLogger(__name__)   # name = module name
# In deploy.py → name is "deploy"
# In billing/processor.py → name is "billing.processor"

logger.info("Deploying billing service")
logger.debug(f"Config: {config}")
logger.warning(f"High CPU: {cpu}%")
logger.error(f"Deploy failed: {error}")
```

> **Always use `logging.getLogger(__name__)`** — never use the root logger directly in modules.

### Log to File AND Console

```python
import logging

logger = logging.getLogger("deploy")
logger.setLevel(logging.DEBUG)

# Console — show INFO and above
console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))

# File — capture everything including DEBUG
file_handler = logging.FileHandler("deploy.log", mode="a")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s [%(levelname)8s] %(name)s:%(lineno)d — %(message)s"
))

logger.addHandler(console)
logger.addHandler(file_handler)

# Now: DEBUG goes to file only, INFO goes to both
logger.debug("Checking pod readiness...")    # file only
logger.info("Pods ready")                    # console + file
logger.error("Rollout failed!")              # console + file
```

### Rotating Log Files (prevents disk fill-up)

```python
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    "app.log",
    maxBytes=5_000_000,   # 5 MB per file
    backupCount=5,         # keep 5 old files: app.log.1 ... app.log.5
)
```

### Log Exceptions with Full Traceback

```python
try:
    result = process(data)
except Exception as e:
    logger.error("Processing failed", exc_info=True)   # includes full traceback
    # OR — shorthand for the same thing:
    logger.exception("Processing failed")
```

### JSON Logging (for Elasticsearch/Splunk/Datadog)

```python
import logging, json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "function": record.funcName,
            "line": record.lineno,
            "message": record.getMessage(),
        })

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

logger.info("Server started")
# {"timestamp": "2026-03-25T09:38:19", "level": "INFO", "logger": "deploy", ...}
```

---

## DevOps Example — Production Deploy Logger

```python
import logging
import sys
from logging.handlers import RotatingFileHandler

def setup_logger(name: str, log_file: str, level=logging.DEBUG) -> logging.Logger:
    """Standard logger setup for any DevOps script."""
    logger = logging.getLogger(name)
    logger.setLevel(level)

    fmt = logging.Formatter(
        "%(asctime)s [%(levelname)8s] %(name)s: %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )

    # Console — INFO+
    sh = logging.StreamHandler(sys.stdout)
    sh.setLevel(logging.INFO)
    sh.setFormatter(fmt)
    logger.addHandler(sh)

    # Rotating file — DEBUG+
    fh = RotatingFileHandler(log_file, maxBytes=10_000_000, backupCount=3)
    fh.setLevel(logging.DEBUG)
    fh.setFormatter(fmt)
    logger.addHandler(fh)

    return logger

# Usage in any script:
logger = setup_logger("deploy", "deploy.log")

def deploy(version, env):
    logger.info(f"Starting deploy: v{version} → {env}")
    logger.debug(f"Full config: {get_config()}")
    try:
        run_deploy(version, env)
        logger.info(f"Deploy complete ✅")
    except Exception as e:
        logger.error(f"Deploy failed ❌", exc_info=True)
        raise
```

---

## `print()` vs `logging` (the definitive comparison)

| Feature | `print()` | `logging` |
|---------|:---------:|:---------:|
| Turn off without deleting | ❌ | ✅ Change level |
| Severity levels | ❌ | ✅ 5 levels |
| Write to file | ❌ | ✅ FileHandler |
| Timestamps | ❌ | ✅ Automatic |
| Source file + line number | ❌ | ✅ Automatic |
| Works in production | ❌ | ✅ |
| Multiple outputs at once | ❌ | ✅ |
| Include exception traceback | ❌ | ✅ `exc_info=True` |

**Rule:** Use `print()` only in quick throwaway scripts. Use `logging` in anything you'll run more than once.

---

## Format Codes Cheat Sheet

| Code | Example Output |
|------|---------------|
| `%(asctime)s` | `2026-03-25 09:38:19` |
| `%(levelname)s` | `WARNING` |
| `%(name)s` | `billing.processor` |
| `%(funcName)s` | `process_invoice` |
| `%(lineno)d` | `42` |
| `%(filename)s` | `processor.py` |
| `%(message)s` | your message |

---

## Gotchas (Louis's warnings)

**1. Never configure logging in a library/module — only in scripts:**
```python
# ❌ In a library file — forces your settings on callers
logging.basicConfig(level=logging.DEBUG)

# ✅ In a library — just get a logger, don't configure it
logger = logging.getLogger(__name__)
# The calling script configures logging
```

**2. `basicConfig` only works ONCE — on first call:**
```python
# This second call does NOTHING if logging was already configured
logging.basicConfig(level=logging.DEBUG)
logging.basicConfig(level=logging.INFO)   # Ignored!
```

**3. `logging.info()` vs `logger.info()` — use named loggers:**
```python
logging.info("This uses the root logger")      # avoid in modules
logger.info("This uses the named logger")      # correct
```

**4. f-strings are evaluated even if level is too low:**
```python
# ❌ This builds the string even if DEBUG is off
logger.debug(f"Huge dict: {expensive_to_compute()}")

# ✅ Use % style — only evaluated if the message is actually logged
logger.debug("Huge dict: %s", expensive_to_compute())
```

---

## Practice (pick one, 10 minutes)

**Beginner:** Set up a logger that writes `DEBUG+` to a file called `output.log` and `WARNING+` to the console. Log one message at each of the 5 levels. Then open the file and verify what's there.

**Intermediate:** Refactor the batch processor from Lesson 1 to use proper logging instead of print statements. Include timestamps, levels, and the module name.

**DevOps-specific:** Write a `setup_deploy_logger(script_name)` function that returns a configured logger: colored INFO/WARNING/ERROR to console, full DEBUG to a rotating file, and JSON format to a second file for ingestion by a log aggregator.

---

## Knowledge Check

1. What are the 5 logging levels in order from lowest to highest?
2. What does `logging.getLogger(__name__)` do and why use it?
3. What's the difference between `logger.error("msg")` and `logger.error("msg", exc_info=True)`?
4. Why does `basicConfig` only work once?
5. What is a `RotatingFileHandler` and why would you use it?

**Answers:** (1) DEBUG, INFO, WARNING, ERROR, CRITICAL. (2) Creates a logger named after the current module — enables fine-grained level control. (3) `exc_info=True` adds the full exception traceback to the log entry. (4) It only applies settings if the root logger hasn't been configured yet. (5) It splits logs across multiple files of a max size, preventing disk fill-up.

---

> **Next:** Lesson 3 — subprocess → Run `kubectl`, `terraform`, `docker`, and `aws` directly from Python.

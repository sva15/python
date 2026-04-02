# Phase 2 · Lesson 1 — Exception Handling
### *"The Contingency Plan"*

> **O'Reilly Skill:** Use try-except to handle exceptions gracefully. Handle specific types. Use else and finally for resource cleanup.
> **DevOps connection:** Scripts that crash at 3 AM because one bad config file take down your entire pipeline. Exception handling is what makes automation reliable.
> **10-min sprint:** Take any script you wrote in Phase 1. Wrap the risky parts in try-except. Make it so a bad input can't crash the whole thing.

---

## Why This Matters (Harvey's take)

One bad file shouldn't kill 237 others. One unreachable server shouldn't stop the health check for all the rest. Exception handling is your contingency plan — attempt the operation, and if it fails, handle the failure gracefully and move on.

---

## The Core Concept (Mike's breakdown)

### Basic try-except

```python
# Without handling — crashes on bad input
number = int("abc")   # ValueError: invalid literal

# With handling — recovers gracefully
try:
    number = int("abc")
except ValueError:
    print("That's not a valid number!")
    number = 0

print(f"Number: {number}")   # Number: 0
```

### The Full Structure

```python
try:
    result = process(data)          # attempt the risky operation
except FileNotFoundError:
    print("File missing — skipping")
    result = None
except ValueError as e:
    print(f"Bad data: {e}")         # 'e' holds the error message
    result = None
else:
    save(result)                    # runs ONLY if try succeeded
finally:
    cleanup()                       # ALWAYS runs — success or failure
```

### Catching the Exception Object

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Type: {type(e).__name__}")   # ZeroDivisionError
    print(f"Message: {e}")               # division by zero
```

### Catching Multiple Types

```python
try:
    value = process(data)
except (ValueError, TypeError, KeyError) as e:
    print(f"Data error: {e}")

# Order: most specific first
try:
    value = risky()
except FileNotFoundError:    # specific
    print("File missing")
except OSError:               # broader parent class
    print("OS error")
except Exception:             # catch-all (last resort only)
    print("Something went wrong")
```

### `else` — the success handler

```python
def load_config(filepath):
    try:
        with open(filepath) as f:
            data = f.read()
    except FileNotFoundError:
        return None
    else:
        # Only runs if no exception — keeps success logic separate
        return parse(data)
```

### `finally` — always runs

```python
def query_database(sql):
    conn = connect()
    try:
        return conn.execute(sql)
    except DatabaseError as e:
        log(e)
        return None
    finally:
        conn.close()    # ALWAYS closes, even if exception or return
```

### Raising Exceptions

```python
def validate_hours(hours):
    if not isinstance(hours, (int, float)):
        raise TypeError(f"Hours must be numeric, got {type(hours).__name__}")
    if hours < 0:
        raise ValueError(f"Hours cannot be negative: {hours}")
    if hours > 24:
        raise ValueError(f"Cannot bill more than 24h/day: {hours}")
    return hours

try:
    validate_hours("85")
except (TypeError, ValueError) as e:
    print(f"Validation failed: {e}")

# Re-raise the same exception (preserves traceback)
try:
    result = risky()
except ValueError as e:
    log(e)
    raise    # re-raises the same exception
```

### Custom Exceptions

```python
class DevOpsError(Exception):
    """Base for all deployment errors."""
    pass

class DeploymentFailed(DevOpsError):
    def __init__(self, service, reason):
        self.service = service
        self.reason = reason
        super().__init__(f"Deploy of {service} failed: {reason}")

class HealthCheckFailed(DevOpsError):
    pass

# Usage:
try:
    deploy_service("billing-api", "v1.2.3")
except DeploymentFailed as e:
    print(f"❌ {e}")
    rollback(e.service)
except DevOpsError as e:
    print(f"DevOps error: {e}")
```

---

## DevOps Example — Resilient Batch Processor

```python
import logging

logger = logging.getLogger(__name__)

def process_all_servers(server_list):
    """Process all servers — one bad one doesn't stop the rest."""
    results = {"ok": [], "failed": [], "skipped": []}

    for server in server_list:
        try:
            status = check_server_health(server)
            results["ok"].append(server)
            logger.info(f"✅ {server}: {status}")

        except ConnectionError:
            results["failed"].append(server)
            logger.error(f"❌ {server}: unreachable")

        except TimeoutError:
            results["skipped"].append(server)
            logger.warning(f"⏰ {server}: timed out — skipping")

        except Exception as e:
            results["failed"].append(server)
            logger.error(f"💥 {server}: unexpected error: {e}", exc_info=True)

    print(f"\nResults: {len(results['ok'])} ok, "
          f"{len(results['failed'])} failed, "
          f"{len(results['skipped'])} skipped")
    return results
```

---

## Gotchas (Louis's warnings)

**1. Never use bare `except:` or `except Exception: pass`:**
```python
# ❌ Swallows ALL errors — bugs become invisible
try:
    result = process(data)
except:
    pass

# ✅ At minimum, log the error
try:
    result = process(data)
except Exception as e:
    logger.error(f"process() failed: {e}", exc_info=True)
    result = default_value
```

**2. Most specific exceptions first:**
```python
# ❌ FileNotFoundError is an OSError — this catches it too early
try: open("file")
except OSError: print("OS error")
except FileNotFoundError: print("never reached")

# ✅ Most specific first
try: open("file")
except FileNotFoundError: print("file missing")
except OSError: print("other OS error")
```

**3. Never catch `KeyboardInterrupt` or `SystemExit` with `except Exception`:**
```python
# These are not subclasses of Exception — they won't be caught accidentally
# But if you use bare except:, you WILL catch them and break Ctrl+C!
```

**4. `assert` is not exception handling:**
```python
# assert can be disabled with python -O
# Use for development sanity checks only
# Use raise ValueError() for real validation
```

---

## Common Error Types (Cheat Sheet)

| Error | Means | Common Fix |
|-------|-------|------------|
| `ValueError` | Right type, wrong value | Validate before use |
| `TypeError` | Wrong type for operation | Check/convert types |
| `KeyError` | Dict key doesn't exist | Use `.get()` |
| `IndexError` | List index out of range | Check `len()` |
| `FileNotFoundError` | File doesn't exist | Check path |
| `PermissionError` | No access rights | Check permissions |
| `ConnectionError` | Network issue | Retry with backoff |
| `TimeoutError` | Operation too slow | Add timeout |
| `AttributeError` | Object lacks attribute | Check type |
| `ImportError` | Module not found | Install package |

---

## Retry Pattern (DevOps essential)

```python
import time

def with_retry(func, max_attempts=3, delay=2):
    """Retry a function on failure — essential for flaky infrastructure."""
    for attempt in range(1, max_attempts + 1):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts:
                raise
            print(f"Attempt {attempt}/{max_attempts} failed: {e}. Retrying in {delay}s...")
            time.sleep(delay * attempt)   # exponential backoff

# Usage:
result = with_retry(lambda: call_flaky_api())
```

---

## Practice (pick one, 10 minutes)

**Beginner:** Write `safe_divide(a, b)` that handles ZeroDivisionError, TypeError, and ValueError. Return None on any error, and print what went wrong.

**Intermediate:** Build a batch file reader. Given a list of file paths, read each one. If it doesn't exist, skip it. If it can't be parsed as JSON, log the error and skip. Return a list of successfully parsed dicts.

**DevOps-specific:** Write a `run_command(cmd)` wrapper around `subprocess.run()`. Handle `FileNotFoundError` (command not found), `subprocess.TimeoutExpired`, and non-zero exit codes. Return `(success, output, error)`.

---

## Knowledge Check

1. What is the difference between `except ValueError` and `except Exception`?
2. When does the `else` block run?
3. When does `finally` run?
4. What does `raise` (without arguments) do inside an except block?
5. Why is `except Exception: pass` dangerous?

**Answers:** (1) `ValueError` catches only that type; `Exception` catches almost everything — prefer specific. (2) `else` runs only if no exception was raised in `try`. (3) `finally` always runs — after try, after except, even after a return. (4) Re-raises the current exception with original traceback. (5) It silently hides bugs — you'll never know they happened.

---

> **Next:** Lesson 2 — Logging → `print()` is talking to yourself. `logging` is filing a court record.

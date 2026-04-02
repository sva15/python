# Phase 2 · Lesson 4 — os / pathlib / shutil
### *"The Filing System"*

> **DevOps Objective:** Work with environment variables, file paths, directories, and file operations using Python's standard library.
> **DevOps connection:** Reading secrets from env vars, finding config files, copying build artifacts, cleaning temp dirs — these are daily tasks for every DevOps script.
> **10-min sprint:** Write a script that reads `DB_HOST` and `DB_PORT` from environment variables (with defaults), checks if a config file exists, and prints whether the setup looks correct.

---

## Why This Matters

Your deploy script needs to: read credentials from env vars (never hardcode secrets!), find config files, create output directories, copy build artifacts, and clean up temp files. These three modules handle all of that.

---

## The Core Concept

### `os` — Environment Variables & Basic Paths

```python
import os

# === Environment variables — the #1 way to configure scripts ===

# Get a variable (returns None if not set)
db_host = os.environ.get("DB_HOST")

# Get with default
db_host = os.environ.get("DB_HOST", "localhost")
db_port = int(os.environ.get("DB_PORT", "5432"))

# Require a variable — crash if missing (appropriate for secrets)
api_key = os.environ["AWS_SECRET_ACCESS_KEY"]   # KeyError if not set

# Set for current process and child processes
os.environ["DEPLOY_ENV"] = "staging"

# All env vars as a dict
all_env = dict(os.environ)
```

```python
# === Basic file system checks ===
os.path.exists("/etc/nginx/nginx.conf")    # True or False
os.path.isfile("config.yaml")             # True if it's a file
os.path.isdir("/var/log/app")             # True if it's a directory

# List directory contents
for filename in os.listdir("/var/log"):
    print(filename)

# Walk directory tree recursively
for root, dirs, files in os.walk("/app/configs"):
    for file in files:
        if file.endswith(".yaml"):
            full_path = os.path.join(root, file)
            print(full_path)
```

### `pathlib` — The Modern Way (use this)

```python
from pathlib import Path

# Create path objects
config_dir = Path("/app/configs")
log_dir = Path("/var/log/app")
home = Path.home()           # /home/username
cwd = Path.cwd()             # current working directory

# Build paths safely (works on Windows AND Linux)
config_file = config_dir / "database.yaml"   # /app/configs/database.yaml
log_file = log_dir / "2026-03" / "app.log"   # nested

# Check existence
print(config_file.exists())     # True or False
print(config_file.is_file())    # True if file
print(config_dir.is_dir())      # True if directory

# Create directories
log_dir.mkdir(parents=True, exist_ok=True)   # mkdir -p equivalent

# Read and write files
content = Path("config.yaml").read_text()
Path("output.json").write_text('{"status": "ok"}')

# Find files (glob)
yaml_files = list(config_dir.glob("*.yaml"))        # non-recursive
all_yaml = list(config_dir.glob("**/*.yaml"))        # recursive

# Path components
p = Path("/var/log/app/error.log")
print(p.name)      # error.log
print(p.stem)      # error
print(p.suffix)    # .log
print(p.parent)    # /var/log/app
print(p.parts)     # ('/', 'var', 'log', 'app', 'error.log')

# List directory contents
for item in config_dir.iterdir():
    if item.is_file():
        print(f"File: {item.name} ({item.stat().st_size} bytes)")
```

### `shutil` — Copy, Move, Archive

```python
import shutil

# Copy a file
shutil.copy("app.conf", "/etc/nginx/app.conf")         # copy content + permissions
shutil.copy2("app.conf", "/etc/nginx/app.conf")        # copy2 also copies metadata

# Copy a directory tree
shutil.copytree("configs/", "/deploy/configs/")

# Move (rename) a file or directory
shutil.move("build/output.zip", "/releases/output.zip")

# Delete a directory and all contents
shutil.rmtree("/tmp/build")     # ⚠️ no undo — use with care!

# Create archives
shutil.make_archive("backup", "tar", "/var/data")       # → backup.tar
shutil.make_archive("release", "zip", "/app/dist")      # → release.zip

# Extract archives
shutil.unpack_archive("backup.tar", "/restore/")

# Disk usage
total, used, free = shutil.disk_usage("/")
print(f"Disk: {free // (1024**3)} GB free of {total // (1024**3)} GB")
```

---

## DevOps Example — Build Artifact Manager

```python
import os
import shutil
import logging
from pathlib import Path

logger = logging.getLogger(__name__)

def prepare_release(version: str, env: str):
    """
    Build, package, and stage a release artifact.
    Reads config from env vars. Works without hardcoded paths.
    """
    # --- Config from environment ---
    build_dir = Path(os.environ.get("BUILD_DIR", "/tmp/build"))
    artifacts_dir = Path(os.environ.get("ARTIFACTS_DIR", "/releases"))
    max_releases = int(os.environ.get("MAX_RELEASES", "10"))

    logger.info(f"Preparing release v{version} for {env}")
    logger.debug(f"Build dir: {build_dir}, Artifacts: {artifacts_dir}")

    # --- Create directories ---
    release_dir = build_dir / version
    release_dir.mkdir(parents=True, exist_ok=True)
    logger.info(f"Created release dir: {release_dir}")

    # --- Copy config files for this environment ---
    config_src = Path("configs") / env
    if not config_src.exists():
        raise FileNotFoundError(f"No config directory for environment: {env}")

    config_dst = release_dir / "configs"
    shutil.copytree(str(config_src), str(config_dst))
    logger.info(f"Copied {len(list(config_dst.iterdir()))} config files")

    # --- Archive the release ---
    artifacts_dir.mkdir(parents=True, exist_ok=True)
    archive_name = str(artifacts_dir / f"release-{version}-{env}")
    shutil.make_archive(archive_name, "tar", str(release_dir))
    archive_path = Path(archive_name + ".tar")
    size_mb = archive_path.stat().st_size / (1024 * 1024)
    logger.info(f"Created archive: {archive_path.name} ({size_mb:.1f} MB)")

    # --- Rotate old releases (keep only last N) ---
    all_releases = sorted(
        artifacts_dir.glob(f"release-*-{env}.tar"),
        key=lambda p: p.stat().st_mtime,
    )
    to_delete = all_releases[:-max_releases] if len(all_releases) > max_releases else []
    for old in to_delete:
        old.unlink()
        logger.info(f"Removed old release: {old.name}")

    # --- Clean up build directory ---
    shutil.rmtree(str(release_dir))
    logger.debug(f"Cleaned up: {release_dir}")

    return str(archive_path)
```

---

## Common Patterns Cheat Sheet

```python
from pathlib import Path
import os

# Read a config file next to the current script
script_dir = Path(__file__).parent
config = (script_dir / "config.yaml").read_text()

# Create a temp working dir and clean it up
import tempfile
with tempfile.TemporaryDirectory() as tmpdir:
    work_dir = Path(tmpdir)
    # ... do work ...
    # auto-deleted when block exits

# Find all log files older than 7 days
import time
log_dir = Path("/var/log/app")
cutoff = time.time() - (7 * 86400)
old_logs = [p for p in log_dir.glob("*.log") if p.stat().st_mtime < cutoff]

# Check disk space before writing
total, used, free = shutil.disk_usage("/")
if free < 1024**3:   # less than 1 GB free
    raise RuntimeError(f"Insufficient disk space: {free // 1024**2} MB free")

# Get config from env with type safety
def get_env(key: str, default=None, required=False, cast=str):
    value = os.environ.get(key, default)
    if required and value is None:
        raise EnvironmentError(f"Required env var not set: {key}")
    return cast(value) if value is not None else None

port = get_env("APP_PORT", default="8080", cast=int)
debug = get_env("DEBUG", default="false") == "true"
```

---

## Gotchas (Louis's warnings)

**1. Never hardcode paths — use env vars or relative paths:**
```python
# ❌ Breaks on every other machine
config = open("/home/john/projects/myapp/config.yaml").read()

# ✅ Works everywhere
config_path = Path(os.environ.get("CONFIG_PATH", "config.yaml"))
config = config_path.read_text()
```

**2. Never hardcode secrets — use env vars:**
```python
# ❌ Secret in code = secret in git = compromised
DB_PASSWORD = "SuperSecret123"

# ✅ Read from environment at runtime
DB_PASSWORD = os.environ["DB_PASSWORD"]   # fail fast if not set
```

**3. `shutil.rmtree()` has no undo:**
```python
# ❌ Catastrophic if variable is empty
shutil.rmtree(f"/releases/{version}")   # if version="", deletes /releases/!

# ✅ Always validate before deleting
if version and len(version) > 3:
    target = Path("/releases") / version
    if target.exists():
        shutil.rmtree(str(target))
```

**4. `Path` vs `str` — some libraries still need strings:**
```python
path = Path("/var/log/app")
# Most modern Python accepts both Path and str
open(path)                # ✅ works
shutil.rmtree(path)       # ✅ works
# Old libraries may need: str(path)
```

**5. `glob("**/*.yaml")` needs `recursive=True` in `os.glob` but NOT in `Path.glob`:**
```python
from pathlib import Path
import glob

# pathlib — ** works natively
list(Path(".").glob("**/*.yaml"))   # ✅

# os.glob — needs recursive=True
glob.glob("**/*.yaml", recursive=True)   # ✅
glob.glob("**/*.yaml")                   # ❌ won't recurse
```

---

## Practice (pick one, 10 minutes)

**Beginner:** Write a script that reads `APP_ENV`, `LOG_LEVEL`, and `PORT` from environment variables (with sensible defaults). Print a config summary. Check if a `configs/{APP_ENV}.yaml` file exists and warn if it's missing.

**Intermediate:** Write a `clean_old_logs(log_dir, days=7)` function. Find all `.log` files older than `days` days. Print what would be deleted (dry run), then actually delete them. Print a summary.

**DevOps-specific:** Write a build artifact packager. Given a `dist/` directory, copy all files to a versioned directory (`releases/v{version}/`), create a `.tar.gz` archive, and delete the working directory. Rotate: keep only the last 5 releases.

---

## Knowledge Check

1. What's the difference between `os.environ.get("KEY")` and `os.environ["KEY"]`?
2. How do you create nested directories with `pathlib`?
3. What does `shutil.rmtree()` do and what's the danger?
4. How do you find all `.yaml` files recursively in a directory with `pathlib`?
5. Why should you never hardcode file paths or credentials in a script?

**Answers:** (1) `.get()` returns `None` if missing; `["KEY"]` raises `KeyError`. (2) `Path("a/b/c").mkdir(parents=True, exist_ok=True)`. (3) Deletes the entire directory tree — no undo, no recycle bin. (4) `list(Path(dir).glob("**/*.yaml"))`. (5) Hardcoded paths break on other machines; hardcoded secrets end up in git history.

---

> **Next:** Lesson 5 — requests → Call any REST API, check service health, trigger webhooks.

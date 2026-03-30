# 🏛️ DevOps Module | Lesson 2: Navigate the File System with os, pathlib, and shutil

## *"The Filing Cabinet"*

> **DevOps Objective:** *Work with file paths, environment variables, directory operations, and file management using `os`, `pathlib`, and `shutil`.*

> **Previously Learned:** File I/O (`open`, `read`, `write`), context managers, strings, exception handling

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

*The deployment script hardcodes `/home/mike/app/config.yaml`. It works on Mike's laptop but crashes on every server. Meanwhile, secrets are hardcoded in the script instead of using environment variables.*

```python
# ❌ Hardcoded paths, hardcoded secrets
config_path = "/home/mike/app/config.yaml"           # Breaks on servers!
db_password = "SuperSecret123"                        # In source code!

# ✅ Portable paths, environment variables
from pathlib import Path
import os

config_path = Path(__file__).parent / "config.yaml"   # Works everywhere!
db_password = os.environ["DB_PASSWORD"]                # From environment!
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

| Module | Purpose | When to Use |
|--------|---------|-------------|
| **`os`** | Environment vars, basic file ops, process info | Env vars, `os.walk` |
| **`pathlib`** | Object-oriented path manipulation | Path building, file ops ✅ |
| **`shutil`** | High-level file operations | Copy, move, archive |

**Harvey:** "Use `pathlib` for paths. Use `os` for environment. Use `shutil` for heavy lifting."

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 Environment Variables — The DevOps Lifeline ⭐⭐⭐

```python
import os

# === Read environment variables ===
# os.environ is a DICT of all environment variables!

# Get with default (safe — no crash)
aws_region = os.environ.get("AWS_REGION", "us-east-1")
log_level = os.environ.get("LOG_LEVEL", "INFO")
debug = os.environ.get("DEBUG", "false").lower() == "true"

# Get required (crash if missing — good for secrets!)
db_host = os.environ["DB_HOST"]          # KeyError if not set!
api_key = os.environ["API_KEY"]          # Fail fast if missing!

# Safer required check with helpful errors
def require_env(name):
    """Get required env var or crash with clear message."""
    value = os.environ.get(name)
    if value is None:
        raise EnvironmentError(f"Required environment variable '{name}' is not set!")
    return value

db_password = require_env("DB_PASSWORD")

# === Set environment variables ===
os.environ["DEPLOY_ENV"] = "staging"     # Set for this process + children
os.environ["BUILD_VERSION"] = "1.2.3"

# === Check if variable exists ===
if "KUBERNETES_SERVICE_HOST" in os.environ:
    print("Running inside Kubernetes!")
else:
    print("Running locally")

# === List all environment variables ===
for key, value in sorted(os.environ.items()):
    if key.startswith("AWS_"):
        print(f"  {key} = {value}")

# === .env file loading ===
# pip install python-dotenv
# from dotenv import load_dotenv
# load_dotenv()    # Loads .env file into os.environ
```

---

### 3.2 `pathlib.Path` — Modern Path Handling ⭐⭐⭐

```python
from pathlib import Path

# === Creating paths ===
config = Path("/etc/nginx/nginx.conf")
home = Path.home()                          # /home/username
cwd = Path.cwd()                            # Current working directory
here = Path(__file__).parent                 # Directory of THIS script

# === The / operator builds paths! ===
project = Path("/app")
config = project / "configs" / "production" / "app.yaml"
print(config)    # /app/configs/production/app.yaml

log_dir = Path("/var/log") / "myapp"
latest = log_dir / "latest.log"

# === Path components ===
p = Path("/var/log/nginx/error.log")
print(p.name)       # error.log
print(p.stem)       # error
print(p.suffix)     # .log
print(p.parent)     # /var/log/nginx
print(p.parents[0]) # /var/log/nginx
print(p.parents[1]) # /var/log
print(p.parts)      # ('/', 'var', 'log', 'nginx', 'error.log')

# === Checking existence ===
p = Path("config.yaml")
p.exists()           # True/False
p.is_file()          # True if it's a file
p.is_dir()           # True if it's a directory
p.is_symlink()       # True if it's a symbolic link

# === File info ===
stat = p.stat()
print(f"Size: {stat.st_size} bytes")
print(f"Modified: {stat.st_mtime}")

# === Resolve to absolute path ===
relative = Path("../configs/app.yaml")
absolute = relative.resolve()    # /full/absolute/path/configs/app.yaml
```

---

### 3.3 Directory Operations ⭐⭐

```python
from pathlib import Path

# === Create directories ===
log_dir = Path("/var/log/myapp")
log_dir.mkdir(parents=True, exist_ok=True)
# parents=True  → create parent dirs too (like mkdir -p)
# exist_ok=True → don't crash if already exists

# === List directory contents ===
configs = Path("/app/configs")

# All files in directory
for item in configs.iterdir():
    kind = "📁" if item.is_dir() else "📄"
    print(f"  {kind} {item.name}")

# Filter by extension
yaml_files = list(configs.glob("*.yaml"))
print(f"YAML files: {len(yaml_files)}")

# Recursive search (all subdirectories)
all_yamls = list(configs.glob("**/*.yaml"))
print(f"All YAML files (recursive): {len(all_yamls)}")

# Multiple patterns
for pattern in ["*.yaml", "*.yml", "*.json"]:
    files = list(configs.glob(pattern))
    print(f"  {pattern}: {len(files)} files")

# === os.walk — deep directory traversal ===
import os

for root, dirs, files in os.walk("/app"):
    depth = root.replace("/app", "").count(os.sep)
    indent = "  " * depth
    print(f"{indent}📁 {os.path.basename(root)}/")
    for f in files:
        print(f"{indent}  📄 {f}")
```

---

### 3.4 Reading and Writing with pathlib ⭐⭐

```python
from pathlib import Path

# === Quick read/write (no open() needed!) ===
# Read entire file
content = Path("config.yaml").read_text(encoding="utf-8")

# Write entire file
Path("output.json").write_text('{"status": "ok"}', encoding="utf-8")

# Read binary
data = Path("image.png").read_bytes()

# Write binary
Path("copy.png").write_bytes(data)

# === Still use open() for large files or line-by-line ===
with open(Path("/var/log/app.log")) as f:
    for line in f:
        if "ERROR" in line:
            print(line.strip())

# === Create a config file ===
import json

config = {
    "database": {
        "host": os.environ.get("DB_HOST", "localhost"),
        "port": int(os.environ.get("DB_PORT", "5432")),
        "name": os.environ.get("DB_NAME", "app"),
    },
    "logging": {"level": "INFO"},
}

config_path = Path("generated-config.json")
config_path.write_text(json.dumps(config, indent=2))
print(f"Config written to {config_path.resolve()}")
```

---

### 3.5 `shutil` — Copy, Move, Archive ⭐⭐

```python
import shutil
from pathlib import Path

# === Copy files ===
shutil.copy("app.conf", "/etc/nginx/app.conf")          # Copy file
shutil.copy2("app.conf", "/etc/nginx/app.conf")         # Copy + metadata

# === Copy entire directory tree ===
shutil.copytree("configs/", "/deploy/configs/")
# With filtering:
shutil.copytree(
    "project/",
    "/deploy/project/",
    ignore=shutil.ignore_patterns("*.pyc", "__pycache__", ".git"),
)

# === Move / rename ===
shutil.move("old_name.yaml", "new_name.yaml")
shutil.move("app.log", "/var/log/archive/app.log")

# === Delete directory tree ===
shutil.rmtree("/tmp/build")                # ⚠️ Deletes EVERYTHING in it!
shutil.rmtree("/tmp/build", ignore_errors=True)    # Don't crash if missing

# === Create archives ===
shutil.make_archive(
    "backup-2026-03-29",     # Archive name (no extension)
    "tar",                    # Format: 'zip', 'tar', 'gztar', 'bztar'
    "/var/data",              # Directory to archive
)
# Creates: backup-2026-03-29.tar

shutil.make_archive("logs-backup", "gztar", "/var/log/app")
# Creates: logs-backup.tar.gz

# === Extract archives ===
shutil.unpack_archive("backup.tar.gz", "/restore/path")

# === Disk usage ===
usage = shutil.disk_usage("/")
print(f"Total: {usage.total / 1024**3:.1f} GB")
print(f"Used:  {usage.used / 1024**3:.1f} GB")
print(f"Free:  {usage.free / 1024**3:.1f} GB")
```

---

### 3.6 Temporary Files and Directories ⭐

```python
import tempfile
from pathlib import Path

# === Temporary file — auto-deleted ===
with tempfile.NamedTemporaryFile(mode='w', suffix='.yaml', delete=False) as f:
    f.write("apiVersion: v1\nkind: ConfigMap\n")
    temp_path = f.name

print(f"Temp file: {temp_path}")
# Use it, then clean up
Path(temp_path).unlink()    # Delete

# === Temporary directory — auto-deleted ===
with tempfile.TemporaryDirectory() as tmpdir:
    # tmpdir is a string path to a temporary directory
    config = Path(tmpdir) / "config.yaml"
    config.write_text("key: value")
    
    # Do work with the temp directory...
    print(f"Working in: {tmpdir}")

# Directory is automatically deleted when the with block exits!

# === Common DevOps use: build in temp dir ===
with tempfile.TemporaryDirectory(prefix="build-") as build_dir:
    build_path = Path(build_dir)
    # Copy sources, build, test — dir is cleaned up automatically
```

---

### 3.7 `hashlib` — File Checksums & Integrity ⭐

```python
import hashlib
from pathlib import Path

# === Hash a string ===
digest = hashlib.sha256(b"Hello World").hexdigest()
print(f"SHA-256: {digest}")

# === Hash a file (for integrity verification) ===
def file_hash(filepath, algorithm="sha256"):
    """Calculate hash of a file (handles large files)."""
    h = hashlib.new(algorithm)
    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()

# Verify a downloaded artifact
expected = "a1b2c3d4e5f6..."
actual = file_hash("terraform_1.7.0_linux_amd64.zip")
if actual == expected:
    print("✅ Checksum matches — file is authentic")
else:
    print("❌ Checksum mismatch — file may be corrupted!")

# === Compare two files ===
def files_identical(file1, file2):
    return file_hash(file1) == file_hash(file2)

# === Generate checksum manifest ===
def generate_checksums(directory):
    """Generate SHA-256 checksums for all files in a directory."""
    checksums = {}
    for path in Path(directory).rglob("*"):
        if path.is_file():
            checksums[str(path.relative_to(directory))] = file_hash(path)
    return checksums
```

---

### 3.8 File Permissions and Ownership ⭐

```python
import os
import stat
from pathlib import Path

# === Check permissions ===
p = Path("/app/deploy.sh")
mode = p.stat().st_mode

print(f"Readable: {bool(mode & stat.S_IRUSR)}")
print(f"Writable: {bool(mode & stat.S_IWUSR)}")
print(f"Executable: {bool(mode & stat.S_IXUSR)}")

# === Set permissions ===
os.chmod("deploy.sh", 0o755)     # rwxr-xr-x (owner can execute)
os.chmod("secret.key", 0o600)    # rw------- (owner only!)

# Using pathlib
Path("deploy.sh").chmod(0o755)

# === Make a script executable ===
script = Path("deploy.sh")
script.write_text("#!/bin/bash\necho 'Deploying...'\n")
script.chmod(0o755)    # Now it can be executed!
```

---

### 3.9 Watching for File Changes ⭐

```python
import time
from pathlib import Path

# === Simple file watcher ===
def watch_directory(path, pattern="*", interval=2):
    """Watch a directory for new/modified files."""
    seen = {}
    watch_path = Path(path)
    
    print(f"👀 Watching {watch_path} for {pattern} changes...")
    
    while True:
        for filepath in watch_path.glob(pattern):
            mtime = filepath.stat().st_mtime
            
            if filepath not in seen:
                print(f"  🆕 New: {filepath.name}")
                seen[filepath] = mtime
            elif seen[filepath] != mtime:
                print(f"  ✏️ Modified: {filepath.name}")
                seen[filepath] = mtime
        
        # Check for deleted files
        for filepath in list(seen.keys()):
            if not filepath.exists():
                print(f"  🗑️ Deleted: {filepath.name}")
                del seen[filepath]
        
        time.sleep(interval)

# watch_directory("/app/configs", "*.yaml")

# For production: use 'watchdog' library
# pip install watchdog
```

---

### 3.10 Solving the Firm's Problem — Portable Config Manager

```python
# ============================================
# Pearson Specter Litt — Config Manager
# Portable, environment-aware configuration
# ============================================

import os
import json
import hashlib
import shutil
from pathlib import Path
from datetime import datetime

class ConfigManager:
    """Manage configs using environment variables and pathlib."""
    
    def __init__(self, app_name="psl-billing"):
        self.app_name = app_name
        self.env = os.environ.get("DEPLOY_ENV", "development")
        
        # Portable paths — work on any machine!
        self.base_dir = Path(__file__).parent.resolve()
        self.config_dir = self.base_dir / "configs" / self.env
        self.backup_dir = self.base_dir / "backups"
        self.log_dir = Path(os.environ.get("LOG_DIR", "/var/log")) / app_name
    
    def setup(self):
        """Create required directories."""
        for d in [self.config_dir, self.backup_dir, self.log_dir]:
            d.mkdir(parents=True, exist_ok=True)
        print(f"✅ Directories ready for '{self.env}'")
    
    def load_config(self):
        """Load environment-specific config."""
        config_file = self.config_dir / "app.json"
        
        if not config_file.exists():
            print(f"⚠️ No config for '{self.env}', using defaults")
            return self._defaults()
        
        config = json.loads(config_file.read_text())
        
        # Override with environment variables
        config["database"]["host"] = os.environ.get("DB_HOST", config["database"]["host"])
        config["database"]["password"] = os.environ.get("DB_PASSWORD", "UNSET")
        
        return config
    
    def backup_configs(self):
        """Backup all configs with timestamp and checksum."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"configs_{self.env}_{timestamp}"
        
        archive = shutil.make_archive(
            str(self.backup_dir / backup_name),
            "gztar",
            str(self.config_dir),
        )
        
        checksum = self._file_hash(archive)
        print(f"📦 Backup: {Path(archive).name} ({checksum[:12]}...)")
        return archive
    
    def list_configs(self):
        """List all config files with sizes and checksums."""
        print(f"\n📁 Configs for '{self.env}':")
        for f in sorted(self.config_dir.rglob("*")):
            if f.is_file():
                size = f.stat().st_size
                rel = f.relative_to(self.config_dir)
                print(f"  📄 {rel} ({size:,} bytes)")
    
    def _defaults(self):
        return {
            "app": {"name": self.app_name, "env": self.env},
            "database": {"host": "localhost", "port": 5432, "password": "UNSET"},
        }
    
    @staticmethod
    def _file_hash(filepath):
        h = hashlib.sha256()
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(8192), b""):
                h.update(chunk)
        return h.hexdigest()

# Demo
print("=" * 55)
print(f"{'🏛️  CONFIG MANAGER':^55}")
print("=" * 55)

mgr = ConfigManager()
mgr.setup()
config = mgr.load_config()
mgr.list_configs()

print(f"\n📊 Active config:")
print(f"  App:  {config.get('app', {}).get('name', 'N/A')}")
print(f"  Env:  {config.get('app', {}).get('env', 'N/A')}")
print(f"  DB:   {config.get('database', {}).get('host', 'N/A')}")
print(f"{'='*55}")
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

### Pitfall 1: Hardcoded Paths 🔥
```python
# ❌ Breaks on other machines
config = open("/home/mike/app/config.yaml")

# ✅ Relative to script location
config = open(Path(__file__).parent / "config.yaml")
```

### Pitfall 2: Secrets in Source Code 🔥
```python
# ❌ NEVER put secrets in code!
DB_PASSWORD = "SuperSecret123"

# ✅ Use environment variables
DB_PASSWORD = os.environ["DB_PASSWORD"]
```

### Pitfall 3: Not Using `exist_ok=True` 🔥
```python
# ❌ Crashes if dir exists
Path("/var/log/app").mkdir()    # FileExistsError!

# ✅ Safe
Path("/var/log/app").mkdir(parents=True, exist_ok=True)
```

### Pitfall 4: Windows vs Linux Paths 🔥
```python
# ❌ Hardcoded separators
path = "/var/log" + "/" + "app.log"

# ✅ Path handles it automatically
path = Path("/var/log") / "app.log"    # Works on ALL platforms!
```

### Pitfall 5: `shutil.rmtree` on Wrong Directory 🔥
```python
# ❌ Catastrophic if variable is wrong!
shutil.rmtree(some_variable)    # What if it's "/" ?!

# ✅ Validate first
target = Path(some_variable)
assert target != Path("/"), "Refusing to delete root!"
assert str(target).startswith("/tmp/"), "Only delete from /tmp!"
shutil.rmtree(target)
```

---

## 5. 🧠 Mental Model — Donna Paulsen

```
PATHLIB = GPS NAVIGATION:
  Path("/var/log") / "app" / "error.log"
  Like giving directions: "Go to var, then log, then app."
  Works on every machine, every OS.

OS.ENVIRON = THE FIRM'S SECURE VAULT:
  Secrets and settings stored OUTSIDE the code.
  Different vault for each environment (dev/staging/prod).
  os.environ["DB_PASSWORD"] = "open the vault, get the key."

SHUTIL = THE MOVING COMPANY:
  copy, move, archive, delete — heavy lifting.
  "Pack up the office" = make_archive()
  "Move to new building" = shutil.move()

DONNA'S RULE:
  "Never hardcode a path. Never hardcode a secret.
   Let the environment tell you where things are."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

### 🟢 Beginner
1. **Env Inspector**: Print all environment variables starting with `PATH`, `HOME`, and `USER`.
2. **File Finder**: Find all `.yaml` files in a directory tree. Print paths and sizes.
3. **Directory Builder**: Create a project structure: `src/`, `tests/`, `configs/`, `logs/`.

### 🟡 Intermediate
4. **Config Loader**: Load config from JSON, override values with env vars.
5. **Backup Tool**: Archive a directory with timestamp, verify checksum.
6. **Disk Reporter**: Scan a directory tree, report sizes by file type.

### 🔴 Advanced
7. **Sync Tool**: Sync two directories (like rsync) — copy new/modified, delete removed.
8. **Secret Validator**: Check that all required env vars are set before deployment starts.
9. **File Watcher**: Watch a config directory and reload when files change.

---

## 9. ✅ Knowledge Check

**Q1.** How do you safely read an optional environment variable?
> `os.environ.get("VAR_NAME", "default_value")`.

**Q2.** What is `pathlib.Path` and why prefer it over string paths?
> An object-oriented path that works cross-platform. The `/` operator builds paths without worrying about separators.

**Q3.** How do you create nested directories safely?
> `Path("a/b/c").mkdir(parents=True, exist_ok=True)`.

**Q4.** What does `shutil.copytree()` do?
> Copies an entire directory tree (recursively), including all files and subdirectories.

**Q5.** Why should secrets never be in source code?
> Source code is committed to git, shared, and visible. Environment variables are set per-machine and excluded from repos.

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | Environment variables (`os.environ`) | ✅ |
| 2 | `pathlib.Path` — modern path handling | ✅ |
| 3 | Directory operations (create, list, walk) | ✅ |
| 4 | Reading/writing with pathlib | ✅ |
| 5 | `shutil` — copy, move, archive, delete | ✅ |
| 6 | Temporary files and directories | ✅ |
| 7 | `hashlib` — checksums and integrity | ✅ |
| 8 | File permissions | ✅ |
| 9 | File watching | ✅ |
| 10 | Portable config management | ✅ |

---

> **Next Lesson →** DevOps Lesson 3: *Make HTTP Requests with the requests Library*

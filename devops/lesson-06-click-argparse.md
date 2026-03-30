# 🏛️ DevOps Module | Lesson 6: Build CLI Tools with Click and argparse

## *"The Command Center"*

> **DevOps Objective:** *Build professional command-line tools using `argparse` (standard library) and `Click` (popular framework) — deploy scripts, health checkers, config managers, and automation utilities.*

> **Previously Learned:** Functions, decorators, exception handling, f-strings, type hints, packaging

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

*The firm's deployment process involves running 5 different Python scripts with positional arguments nobody remembers.*

```bash
# ❌ Unnamed arguments — which is the environment? Which is the version?
python deploy.py production billing-api 1.2.3 3 true
#                 ^          ^           ^     ^  ^
#                 What is what?!

# ❌ Hard-coded input()
# version = input("Enter version: ")   # Can't automate this!
```

**Mike:** "Professional tools have `--help`, named arguments, validation, and subcommands. Like `kubectl`, `docker`, or `terraform`."

```bash
# ✅ Professional CLI
python deploy.py deploy billing-api \
    --version 1.2.3 \
    --environment production \
    --replicas 3 \
    --dry-run

python deploy.py status billing-api --environment production
python deploy.py rollback billing-api --environment production
python deploy.py --help
```

---

## 2. 🎯 Concept Introduction — Harvey Specter

**Harvey:** "If your tool needs a README to explain how to run it, it's not finished."

| Feature | `argparse` | `Click` |
|---------|:----------:|:-------:|
| Built-in | ✅ | ❌ (`pip install click`) |
| Auto `--help` | ✅ | ✅ |
| Type validation | ✅ | ✅ |
| Subcommands | ✅ (complex) | ✅ (simple) |
| Colored output | ❌ | ✅ |
| Progress bars | ❌ | ✅ |
| File arguments | Basic | ✅ |
| Testing | Manual | Built-in runner |
| Decorator syntax | ❌ | ✅ |

**Rule of thumb:**
- Quick scripts → `argparse` (no dependencies)
- Professional tools → `Click` (cleaner, more features)

---

## 3. 🔧 Technical Breakdown — Mike Ross

---

### 3.1 `argparse` — Basic Arguments ⭐⭐⭐

```python
import argparse

# === Create parser ===
parser = argparse.ArgumentParser(
    description="Pearson Specter Litt — Deployment Tool",
    epilog="Example: deploy.py billing-api --version 1.2.3 --env production",
)

# === Positional argument (required, no --) ===
parser.add_argument("service", help="Service name to deploy")

# === Optional arguments (with --) ===
parser.add_argument("--version", "-v", required=True, help="Version to deploy")
parser.add_argument("--environment", "-e", default="staging",
                    choices=["dev", "staging", "production"],
                    help="Target environment (default: staging)")
parser.add_argument("--replicas", "-r", type=int, default=3,
                    help="Number of replicas (default: 3)")
parser.add_argument("--dry-run", action="store_true",
                    help="Simulate without deploying")
parser.add_argument("--timeout", type=int, default=120,
                    help="Deployment timeout in seconds")

# === Parse ===
args = parser.parse_args()

# === Use the values ===
print(f"🚀 Deploying {args.service} v{args.version}")
print(f"   Environment: {args.environment}")
print(f"   Replicas: {args.replicas}")
print(f"   Dry run: {args.dry_run}")
print(f"   Timeout: {args.timeout}s")

# === Auto-generated --help ===
# $ python deploy.py --help
# usage: deploy.py [-h] --version VERSION [-e {dev,staging,production}]
#                   [-r REPLICAS] [--dry-run] [--timeout TIMEOUT]
#                   service
#
# Pearson Specter Litt — Deployment Tool
#
# positional arguments:
#   service               Service name to deploy
#
# options:
#   -h, --help            show this help message and exit
#   --version, -v         Version to deploy
#   --environment, -e     Target environment (default: staging)
#   ...
```

---

### 3.2 `argparse` — Subcommands ⭐⭐

```python
import argparse
import sys

def cmd_deploy(args):
    """Handle deploy subcommand."""
    print(f"🚀 Deploying {args.service} v{args.version} to {args.environment}")
    if args.dry_run:
        print("   [DRY RUN] No changes made")

def cmd_status(args):
    """Handle status subcommand."""
    print(f"📊 Status of {args.service} in {args.environment}")

def cmd_rollback(args):
    """Handle rollback subcommand."""
    print(f"⏪ Rolling back {args.service} in {args.environment}")
    if not args.force:
        confirm = input("   Are you sure? (y/N): ")
        if confirm.lower() != 'y':
            print("   Cancelled.")
            sys.exit(0)

# === Main parser ===
parser = argparse.ArgumentParser(description="PSL Deployment Tool")
subparsers = parser.add_subparsers(dest="command", required=True)

# === deploy subcommand ===
deploy_parser = subparsers.add_parser("deploy", help="Deploy a service")
deploy_parser.add_argument("service")
deploy_parser.add_argument("--version", "-v", required=True)
deploy_parser.add_argument("--environment", "-e", default="staging",
                           choices=["dev", "staging", "production"])
deploy_parser.add_argument("--dry-run", action="store_true")
deploy_parser.set_defaults(func=cmd_deploy)

# === status subcommand ===
status_parser = subparsers.add_parser("status", help="Check service status")
status_parser.add_argument("service")
status_parser.add_argument("--environment", "-e", default="staging")
status_parser.set_defaults(func=cmd_status)

# === rollback subcommand ===
rollback_parser = subparsers.add_parser("rollback", help="Rollback a service")
rollback_parser.add_argument("service")
rollback_parser.add_argument("--environment", "-e", default="staging")
rollback_parser.add_argument("--force", "-f", action="store_true",
                             help="Skip confirmation")
rollback_parser.set_defaults(func=cmd_rollback)

args = parser.parse_args()
args.func(args)

# Usage:
# python tool.py deploy billing-api --version 1.2.3 --environment production
# python tool.py status billing-api
# python tool.py rollback billing-api --force
# python tool.py --help
```

---

### 3.3 `Click` — Decorator-Based CLI ⭐⭐⭐

```python
# pip install click

import click

@click.command()
@click.argument("service")
@click.option("--version", "-v", required=True, help="Version to deploy")
@click.option("--environment", "-e", default="staging",
              type=click.Choice(["dev", "staging", "production"]),
              help="Target environment")
@click.option("--replicas", "-r", default=3, type=int, help="Number of replicas")
@click.option("--dry-run", is_flag=True, help="Simulate without deploying")
def deploy(service, version, environment, replicas, dry_run):
    """Deploy a service to the specified environment.

    SERVICE is the name of the service to deploy.
    """
    click.echo(f"🚀 Deploying {service} v{version}")
    click.echo(f"   Environment: {environment}")
    click.echo(f"   Replicas: {replicas}")
    
    if dry_run:
        click.secho("   [DRY RUN] No changes made", fg="yellow")
        return
    
    # Simulate deployment steps
    with click.progressbar(range(5), label="   Deploying") as steps:
        import time
        for step in steps:
            time.sleep(0.5)
    
    click.secho(f"   ✅ {service} v{version} deployed!", fg="green", bold=True)

if __name__ == "__main__":
    deploy()

# Usage:
# python deploy.py billing-api --version 1.2.3 --environment production
# python deploy.py --help
#
# KEY INSIGHT: Notice the decorator pattern?
# @click.command()       → Makes function a CLI command
# @click.argument()      → Positional arg
# @click.option()        → Named option (--flag)
# Function parameters    → Automatically match arg/option names!
```

---

### 3.4 `Click` — Command Groups (Subcommands) ⭐⭐⭐

```python
import click

# === Group = container for subcommands (like kubectl, docker) ===
@click.group()
@click.option("--verbose", "-V", is_flag=True, help="Enable verbose output")
@click.pass_context
def cli(ctx, verbose):
    """PSL Deployment Tool — Manage service deployments."""
    ctx.ensure_object(dict)
    ctx.obj["verbose"] = verbose

# === deploy subcommand ===
@cli.command()
@click.argument("service")
@click.option("--version", "-v", required=True)
@click.option("--environment", "-e", default="staging",
              type=click.Choice(["dev", "staging", "production"]))
@click.option("--replicas", "-r", default=3, type=int)
@click.option("--dry-run", is_flag=True)
@click.pass_context
def deploy(ctx, service, version, environment, replicas, dry_run):
    """Deploy SERVICE to an environment."""
    if ctx.obj["verbose"]:
        click.echo(f"[DEBUG] Deploy called with: {service}, {version}, {environment}")
    
    click.secho(f"🚀 Deploying {service} v{version} → {environment}", fg="cyan")
    
    if dry_run:
        click.secho("   [DRY RUN] Simulated only", fg="yellow")
        return
    
    click.secho(f"   ✅ Deployed with {replicas} replicas", fg="green")

# === status subcommand ===
@cli.command()
@click.argument("service")
@click.option("--environment", "-e", default="staging")
@click.option("--format", "-f", "fmt", default="table",
              type=click.Choice(["table", "json", "yaml"]))
def status(service, environment, fmt):
    """Check the status of SERVICE."""
    click.echo(f"📊 {service} in {environment}:")
    click.echo(f"   Status: Running")
    click.echo(f"   Replicas: 3/3 ready")
    click.echo(f"   Version: 1.2.3")

# === rollback subcommand ===
@cli.command()
@click.argument("service")
@click.option("--environment", "-e", default="staging")
@click.confirmation_option(prompt="Are you sure you want to rollback?")
def rollback(service, environment):
    """Rollback SERVICE to the previous version."""
    click.secho(f"⏪ Rolling back {service} in {environment}...", fg="yellow")
    click.secho(f"   ✅ Rolled back to v1.2.2", fg="green")

# === logs subcommand ===
@cli.command()
@click.argument("service")
@click.option("--tail", "-n", default=50, type=int, help="Number of lines")
@click.option("--follow", "-f", is_flag=True, help="Follow log output")
def logs(service, tail, follow):
    """Show logs for SERVICE."""
    click.echo(f"📋 Logs for {service} (last {tail} lines):")
    for i in range(min(tail, 5)):
        click.echo(f"   [2026-03-30 10:0{i}:00] INFO: Processing request #{i+1}")

if __name__ == "__main__":
    cli()

# Usage:
# python tool.py deploy billing-api --version 1.2.3
# python tool.py status billing-api
# python tool.py rollback billing-api
# python tool.py logs billing-api --tail 100 --follow
# python tool.py --verbose deploy billing-api --version 1.2.3
# python tool.py --help
# python tool.py deploy --help
```

---

### 3.5 `Click` — Output, Colors, and Progress ⭐⭐

```python
import click
import time

# === Colored output ===
click.echo("Normal text")
click.secho("Success!", fg="green", bold=True)
click.secho("Warning!", fg="yellow")
click.secho("Error!", fg="red", bold=True)
click.secho("Info", fg="cyan")
click.secho("Highlighted", fg="white", bg="blue")

# === Styled inline ===
name = click.style("billing-api", fg="cyan", bold=True)
status = click.style("RUNNING", fg="green")
click.echo(f"Service {name}: {status}")

# === Progress bar ===
items = ["billing", "users", "search", "auth", "gateway"]

with click.progressbar(items, label="Deploying services") as bar:
    for service in bar:
        time.sleep(0.5)    # Simulate work

# === Spinner (manual) ===
import itertools
spinner = itertools.cycle(['⠋', '⠙', '⠹', '⠸', '⠼', '⠴', '⠦', '⠧', '⠇', '⠏'])

# === User confirmation ===
if click.confirm("Deploy to production?"):
    click.echo("Deploying...")
else:
    click.echo("Cancelled.")

# === User input ===
name = click.prompt("Service name", type=str)
replicas = click.prompt("Replicas", type=int, default=3)

# === Password input (hidden) ===
password = click.prompt("API Token", hide_input=True)

# === File output ===
@click.command()
@click.option("--output", "-o", type=click.File("w"), default="-")
def export(output):
    """Export data. Use -o to write to file, default is stdout."""
    output.write("service,status,replicas\n")
    output.write("billing,running,3\n")
```

---

### 3.6 Argument Types and Validation ⭐⭐

```python
import click
from pathlib import Path

# === Built-in types ===
@click.command()
@click.option("--count", type=int)                      # Integer
@click.option("--rate", type=float)                      # Float
@click.option("--name", type=str)                        # String
@click.option("--debug", type=bool)                      # Boolean
@click.option("--env", type=click.Choice(["dev", "prod"]))  # Enum-like
@click.option("--port", type=click.IntRange(1, 65535))   # Bounded int
@click.option("--config", type=click.Path(exists=True))  # File path (must exist)
@click.option("--output", type=click.File("w"))          # Writable file
def serve(**kwargs):
    """Start the server."""
    for k, v in kwargs.items():
        click.echo(f"  {k}: {v}")

# === Custom validation with callbacks ===
def validate_version(ctx, param, value):
    """Validate semantic version format."""
    import re
    if not re.match(r'^\d+\.\d+\.\d+$', value):
        raise click.BadParameter(f"Must be semver format (e.g., 1.2.3), got: {value}")
    return value

@click.command()
@click.option("--version", callback=validate_version, required=True)
def deploy(version):
    click.echo(f"Deploying v{version}")

# === Custom type ===
class SemverType(click.ParamType):
    """Custom type for semantic versions."""
    name = "semver"
    
    def convert(self, value, param, ctx):
        import re
        if not isinstance(value, str) or not re.match(r'^\d+\.\d+\.\d+$', value):
            self.fail(f"'{value}' is not a valid semver (expected X.Y.Z)", param, ctx)
        return value

SEMVER = SemverType()

@click.command()
@click.option("--version", type=SEMVER, required=True)
def release(version):
    click.echo(f"Releasing {version}")

# === Multiple values ===
@click.command()
@click.option("--tag", "-t", multiple=True, help="Add tags (repeatable)")
@click.option("--env-var", "-e", nargs=2, multiple=True,
              help="Environment variable as KEY VALUE pair")
def build(tag, env_var):
    click.echo(f"Tags: {tag}")         # ('latest', 'v1.2.3')
    click.echo(f"Env vars: {env_var}") # (('DB_HOST', 'localhost'), ...)

# Usage: python build.py --tag latest --tag v1.2.3 -e DB_HOST localhost -e LOG debug
```

---

### 3.7 Testing CLI Tools ⭐⭐

```python
# === Testing Click commands ===
from click.testing import CliRunner

def test_deploy_success():
    runner = CliRunner()
    result = runner.invoke(deploy, [
        "billing-api",
        "--version", "1.2.3",
        "--environment", "staging",
    ])
    assert result.exit_code == 0
    assert "Deploying billing-api" in result.output
    assert "v1.2.3" in result.output

def test_deploy_dry_run():
    runner = CliRunner()
    result = runner.invoke(deploy, [
        "billing-api",
        "--version", "1.2.3",
        "--dry-run",
    ])
    assert result.exit_code == 0
    assert "DRY RUN" in result.output

def test_deploy_missing_version():
    runner = CliRunner()
    result = runner.invoke(deploy, ["billing-api"])
    assert result.exit_code != 0
    assert "Missing option" in result.output or "required" in result.output.lower()

def test_deploy_invalid_environment():
    runner = CliRunner()
    result = runner.invoke(deploy, [
        "billing-api",
        "--version", "1.2.3",
        "--environment", "invalid",
    ])
    assert result.exit_code != 0

# === Testing argparse (with pytest) ===
import pytest
import sys

def test_argparse_deploy():
    sys.argv = ["tool", "deploy", "billing-api", "--version", "1.2.3"]
    args = parser.parse_args()
    assert args.service == "billing-api"
    assert args.version == "1.2.3"
```

---

### 3.8 Making an Installable CLI ⭐⭐

```python
# === Entry point in pyproject.toml ===
# After pip install, the command is available globally!

# pyproject.toml:
# [project.scripts]
# psl = "psl_tool.cli:cli"
#
# Now: $ psl deploy billing-api --version 1.2.3
# Instead of: $ python src/psl_tool/cli.py deploy ...

# === Project structure ===
# psl-tool/
# ├── pyproject.toml
# ├── src/
# │   └── psl_tool/
# │       ├── __init__.py
# │       ├── cli.py          ← Click group lives here
# │       ├── deploy.py       ← Deploy logic
# │       ├── status.py       ← Status logic
# │       └── config.py       ← Config helpers
# └── tests/
#     └── test_cli.py

# === cli.py ===
import click
from psl_tool.deploy import run_deploy
from psl_tool.status import check_status

@click.group()
@click.version_option("1.0.0")
def cli():
    """PSL Tool — Infrastructure management CLI."""
    pass

@cli.command()
@click.argument("service")
@click.option("--version", "-v", required=True)
def deploy(service, version):
    """Deploy a service."""
    run_deploy(service, version)

@cli.command()
@click.argument("service")
def status(service):
    """Check service status."""
    check_status(service)
```

---

### 3.9 Real-World CLI Patterns ⭐

```python
import click
import json
import sys

# === Pattern 1: Output format switching ===
@click.command()
@click.option("--format", "-f", "fmt", default="table",
              type=click.Choice(["table", "json", "csv"]))
def list_services(fmt):
    """List all services."""
    services = [
        {"name": "billing-api", "status": "running", "replicas": 3},
        {"name": "user-svc", "status": "running", "replicas": 2},
        {"name": "search", "status": "stopped", "replicas": 0},
    ]
    
    if fmt == "json":
        click.echo(json.dumps(services, indent=2))
    elif fmt == "csv":
        click.echo("name,status,replicas")
        for s in services:
            click.echo(f"{s['name']},{s['status']},{s['replicas']}")
    else:
        click.echo(f"{'Service':<20} {'Status':<12} {'Replicas':>8}")
        click.echo(f"{'─'*20} {'─'*12} {'─'*8}")
        for s in services:
            color = "green" if s["status"] == "running" else "red"
            status = click.style(s["status"], fg=color)
            click.echo(f"{s['name']:<20} {status:<21} {s['replicas']:>8}")

# === Pattern 2: Config file + CLI override ===
@click.command()
@click.option("--config", "-c", type=click.Path(exists=True), default="deploy.yaml")
@click.option("--override", "-o", multiple=True, help="Override as key=value")
def apply(config, override):
    """Apply config with optional overrides."""
    import yaml
    from pathlib import Path
    
    cfg = yaml.safe_load(Path(config).read_text())
    
    for item in override:
        key, value = item.split("=", 1)
        cfg[key] = value
    
    click.echo(f"Applied config: {json.dumps(cfg, indent=2)}")

# Usage: python tool.py apply --config prod.yaml -o replicas=5 -o image=billing:1.3

# === Pattern 3: Exit codes for scripting ===
@click.command()
@click.argument("service")
def check(service):
    """Check if service is healthy. Exit 0=healthy, 1=unhealthy."""
    healthy = check_health(service)
    if healthy:
        click.secho(f"✅ {service} is healthy", fg="green")
        sys.exit(0)
    else:
        click.secho(f"❌ {service} is unhealthy", fg="red")
        sys.exit(1)

# Can be used in bash: if python tool.py check billing; then echo "OK"; fi
```

---

### 3.10 Solving the Firm's Problem — Full Deployment CLI

```python
# ============================================
# Pearson Specter Litt — Deployment CLI
# Professional tool replacing ad-hoc scripts
# ============================================

import click
import time
import json
from datetime import datetime

# ─── Shared state ───
@click.group()
@click.version_option("2.0.0", prog_name="psl-deploy")
@click.option("--verbose", "-V", is_flag=True, help="Show debug output")
@click.pass_context
def cli(ctx, verbose):
    """🏛️ PSL Deploy — Infrastructure management for Pearson Specter Litt.

    \b
    Examples:
      psl deploy billing-api --version 1.2.3 --env production
      psl status billing-api --env production
      psl rollback billing-api --env production
      psl health --all
    """
    ctx.ensure_object(dict)
    ctx.obj["verbose"] = verbose
    ctx.obj["start_time"] = time.perf_counter()

# ─── deploy ───
@cli.command()
@click.argument("service")
@click.option("--version", "-v", required=True, help="Semver to deploy")
@click.option("--env", "-e", "environment", default="staging",
              type=click.Choice(["dev", "staging", "production"]),
              help="Target environment")
@click.option("--replicas", "-r", default=3, type=click.IntRange(1, 20))
@click.option("--dry-run", is_flag=True, help="Simulate deployment")
@click.option("--wait/--no-wait", default=True, help="Wait for rollout")
@click.pass_context
def deploy(ctx, service, version, environment, replicas, dry_run, wait):
    """Deploy SERVICE to an environment."""
    click.secho(f"\n🚀 Deploying {service} v{version} → {environment}", fg="cyan", bold=True)
    click.echo(f"   Replicas: {replicas}")
    
    if environment == "production" and not dry_run:
        if not click.confirm(click.style("   ⚠️ Deploy to PRODUCTION?", fg="yellow")):
            click.secho("   Cancelled.", fg="red")
            raise SystemExit(1)
    
    if dry_run:
        click.secho("   📋 [DRY RUN] Simulated steps:", fg="yellow")
        steps = ["Build image", "Push to registry", "Update manifest", "Apply to cluster"]
        for step in steps:
            click.echo(f"      → {step}")
        return
    
    # Simulated deployment
    steps = ["Pulling image", "Updating deployment", "Waiting for pods", "Health check"]
    with click.progressbar(steps, label="   Deploying", show_pos=True) as bar:
        for step in bar:
            time.sleep(0.5)
    
    elapsed = time.perf_counter() - ctx.obj["start_time"]
    click.secho(f"\n   ✅ {service} v{version} live in {environment} ({elapsed:.1f}s)", fg="green", bold=True)

# ─── status ───
@cli.command()
@click.argument("service")
@click.option("--env", "-e", "environment", default="staging")
@click.option("--format", "-f", "fmt", default="table",
              type=click.Choice(["table", "json"]))
def status(service, environment, fmt):
    """Check the status of SERVICE."""
    info = {
        "service": service,
        "environment": environment,
        "version": "1.2.3",
        "replicas": {"desired": 3, "ready": 3, "available": 3},
        "status": "Running",
        "uptime": "3d 14h 22m",
        "last_deploy": "2026-03-29T10:00:00Z",
    }
    
    if fmt == "json":
        click.echo(json.dumps(info, indent=2))
    else:
        click.secho(f"\n📊 {service} — {environment}", fg="cyan", bold=True)
        click.echo(f"   Version:     {info['version']}")
        click.echo(f"   Status:      {click.style(info['status'], fg='green')}")
        r = info['replicas']
        click.echo(f"   Replicas:    {r['ready']}/{r['desired']} ready")
        click.echo(f"   Uptime:      {info['uptime']}")
        click.echo(f"   Last Deploy: {info['last_deploy']}")

# ─── rollback ───
@cli.command()
@click.argument("service")
@click.option("--env", "-e", "environment", default="staging")
@click.confirmation_option(prompt="⚠️ Are you sure you want to rollback?")
def rollback(service, environment):
    """Rollback SERVICE to the previous version."""
    click.secho(f"\n⏪ Rolling back {service} in {environment}...", fg="yellow")
    time.sleep(1)
    click.secho(f"   ✅ Rolled back to v1.2.2", fg="green")

# ─── health ───
@cli.command()
@click.option("--all", "check_all", is_flag=True, help="Check all services")
@click.option("--service", "-s", multiple=True, help="Specific services")
def health(check_all, service):
    """Check service health."""
    services = list(service) if service else ["billing-api", "user-svc", "search", "auth"]
    
    click.secho(f"\n🏥 Health Check — {datetime.now():%H:%M:%S}", fg="cyan", bold=True)
    click.echo(f"   {'Service':<20} {'Status':>10} {'Latency':>10}")
    click.echo(f"   {'─'*20} {'─'*10} {'─'*10}")
    
    healthy = 0
    for svc in services:
        is_healthy = svc != "search"    # Simulate one down
        ms = 42 if is_healthy else 0
        
        if is_healthy:
            status = click.style("✅ UP", fg="green")
            healthy += 1
        else:
            status = click.style("❌ DOWN", fg="red")
        
        click.echo(f"   {svc:<20} {status:>19} {ms:>8}ms")
    
    click.echo(f"\n   {healthy}/{len(services)} healthy")
    
    if healthy < len(services):
        raise SystemExit(1)

# ─── Entry point ───
if __name__ == "__main__":
    cli()

# Install as 'psl' command via pyproject.toml:
# [project.scripts]
# psl = "psl_tool.cli:cli"
#
# $ psl deploy billing-api --version 1.2.3 --env production --dry-run
# $ psl status billing-api --format json
# $ psl health --all
# $ psl rollback billing-api --env staging
# $ psl --help
```

---

## 4. ⚡ Edge Cases & Pitfalls — Louis Litt

### Pitfall 1: Positional Arguments Confuse Users 🔥
```python
# ❌ Too many positional args — unclear
parser.add_argument("service")
parser.add_argument("version")
parser.add_argument("environment")
# python deploy.py billing 1.2.3 production ← which is which?

# ✅ Use options for optional/ambiguous values
parser.add_argument("service")                    # One positional is OK
parser.add_argument("--version", required=True)   # Named = clear
parser.add_argument("--environment", default="staging")
```

### Pitfall 2: Hard Exit Without Cleanup 🔥
```python
# ❌ sys.exit() skips cleanup
sys.exit(1)    # Context managers and finally blocks may not run!

# ✅ Raise SystemExit or use Click's ctx.exit()
raise SystemExit(1)

# Or with Click:
@click.pass_context
def deploy(ctx, ...):
    ctx.exit(1)    # Proper cleanup
```

### Pitfall 3: Using `input()` Instead of Arguments 🔥
```python
# ❌ Can't automate!
version = input("Enter version: ")

# ✅ Command-line arguments — scriptable
@click.option("--version", required=True)
def deploy(version): ...

# Now: deploy.py --version 1.2.3  (works in CI/CD!)
```

### Pitfall 4: Not Setting Exit Codes 🔥
```python
# ❌ Always exits 0, even on failure
print("Deployment failed!")

# ✅ Exit 1 on failure — scripts and CI depend on this!
click.secho("Deployment failed!", fg="red")
sys.exit(1)
```

### Pitfall 5: Click Parameter Name vs Python Name 🔥
```python
# ❌ Python keywords clash
@click.option("--format")     # 'format' shadows built-in!
@click.option("--class")      # 'class' is a keyword!

# ✅ Use a different Python name
@click.option("--format", "-f", "fmt")    # CLI: --format, Python: fmt
@click.option("--class", "klass")         # CLI: --class, Python: klass
```

---

## 5. 🧠 Mental Model — Donna Paulsen

```
CLI TOOLS = THE FIRM'S FRONT DESK:
  Every visitor (user) gets:
  - Greeted properly (--help)
  - Asked what they need (arguments)
  - Guided to the right person (subcommands)
  - Told if something's wrong (error messages)

ARGPARSE = THE RECEPTIONIST'S MANUAL:
  Thorough, by the book, built-in.
  Gets the job done. A bit formal.

CLICK = DONNA HERSELF:
  Elegant, powerful, handles everything.
  Colors, progress bars, confirmations.
  Makes it look effortless.

DONNA'S RULE:
  "A professional tool has --help, validates input,
   and NEVER asks the user to remember argument order.
   If it doesn't have --version, it's not ready."
```

---

## 6. 💪 Practice Exercises — Rachel Zane

### 🟢 Beginner
1. **Greeter**: CLI that takes `--name` and `--greeting` and prints a message.
2. **Calculator**: CLI with subcommands `add`, `subtract`, `multiply` taking two numbers.
3. **File Counter**: CLI that takes a directory path and counts files by extension.

### 🟡 Intermediate
4. **Deploy Tool**: Build deploy/status/rollback subcommands with environment and version.
5. **Health Checker**: CLI that checks URLs and outputs table/JSON with exit codes.
6. **Config Validator**: CLI that reads YAML files and validates required fields.

### 🔴 Advanced
7. **Multi-Format Output**: Support table, JSON, CSV, and YAML output for a list command.
8. **Installable Tool**: Package your CLI with `pyproject.toml` entry points. Install and test.
9. **Plugin System**: CLI that discovers and loads subcommands from entry points.

---

## 7. 🐛 Debugging Scenario

### Buggy Code 🐛
```python
@click.command()
@click.option("--format", default="table")         # Bug 1: shadows built-in
def list(format):                                    # Bug 2: shadows built-in list()!
    print(f"Format: {format}")                       # Bug 3: should use click.echo
    data = get_data()
    if not data:
        print("No data found")                       # Bug 4: no exit code!
```

### Fixed Code ✅
```python
@click.command("list")                               # Fix 2: CLI name, not function name
@click.option("--format", "-f", "fmt", default="table")  # Fix 1: different Python name
def list_services(fmt):
    click.echo(f"Format: {fmt}")                     # Fix 3: click.echo
    data = get_data()
    if not data:
        click.secho("No data found", fg="yellow")
        raise SystemExit(1)                          # Fix 4: proper exit code
```

---

## 8. 🌍 Real-World Application

| Tool | Framework | What It Does |
|------|-----------|-------------|
| **aws-cli** | argparse | AWS cloud management |
| **docker-compose** | Click | Container orchestration |
| **ansible** | argparse | Configuration management |
| **httpie** | Click | HTTP client |
| **black** | Click | Code formatter |
| **cookiecutter** | Click | Project scaffolding |
| **pip** | argparse | Package management |
| **pytest** | argparse + plugins | Test runner |

---

## 9. ✅ Knowledge Check

**Q1.** What is `argparse`?
> Python's built-in module for parsing command-line arguments. Auto-generates `--help` and validates input.

**Q2.** What is Click?
> A third-party Python library for building CLIs using decorators. Offers colors, progress bars, testing utilities, and cleaner syntax than argparse.

**Q3.** What is the difference between `@click.argument` and `@click.option`?
> Arguments are positional and required (`mycommand VALUE`). Options are named with `--` flags and usually optional (`--value VALUE`).

**Q4.** How do you add subcommands in Click?
> Use `@click.group()` on the parent, then `@group.command()` on each subcommand function.

**Q5.** Why are exit codes important?
> CI/CD pipelines and shell scripts check exit codes to determine success (0) or failure (non-zero). Without proper exit codes, failures go undetected.

---

## 📌 Concepts Mastered

| # | Concept | Status |
|---|---------|--------|
| 1 | `argparse` basics (arguments, options, types) | ✅ |
| 2 | `argparse` subcommands | ✅ |
| 3 | `Click` decorator syntax | ✅ |
| 4 | `Click` command groups | ✅ |
| 5 | Colors, progress bars, prompts | ✅ |
| 6 | Argument validation and custom types | ✅ |
| 7 | Testing CLI tools | ✅ |
| 8 | Installable CLI (entry points) | ✅ |
| 9 | Real-world CLI patterns | ✅ |
| 10 | Full deployment CLI tool | ✅ |

---

## 🎬 End of DevOps Module

**Harvey:** "So we went from copying commands by hand to a professional toolkit?"

**Mike:** "`subprocess` runs the commands. `pathlib` handles the files. `requests` talks to APIs. `PyYAML` reads the configs. `Jinja2` generates them. And `Click` wraps it all in a CLI anyone can use."

**Harvey:** "Six tools. Zero manual work."

**Mike:** "And every one of them is just Python dictionaries, functions, and decorators — concepts we learned in Level 1 through 4."

---

> 🎉 **DevOps Module Complete!**
>
> **6 lessons covering the missing Python concepts for DevOps engineers:**
>
> 1. `subprocess` — Run shell commands
> 2. `os` / `pathlib` / `shutil` — File system & environment
> 3. `requests` — HTTP & API calls
> 4. `PyYAML` — YAML config parsing
> 5. `Jinja2` — Config templating
> 6. `Click` / `argparse` — Build CLI tools

# Phase 2 — DevOps Automation Tools
### 9 Lessons · The Practical Toolkit

> Complete these 9 lessons after Phase 1. Each lesson = 10 minutes.
> After Phase 2 you can write real DevOps automation scripts.

---

## Lesson Order

| # | File | Concept | DevOps Use |
|---|------|---------|------------|
| 1 | `lesson-01-exception-handling.md` | try/except/else/finally, raise, custom exceptions | Resilient pipelines — one bad server doesn't kill the rest |
| 2 | `lesson-02-logging.md` | logging module, handlers, formatters | Production-grade scripts with timestamps and file logs |
| 3 | `lesson-03-subprocess.md` | subprocess.run(), Popen, shell commands | Run kubectl, docker, terraform, git from Python |
| 4 | `lesson-04-os-pathlib-shutil.md` | env vars, pathlib, file ops, shutil | Config from env, file manipulation, artifact packaging |
| 5 | `lesson-05-requests.md` | HTTP GET/POST, auth, retry, download | Call REST APIs, health checks, Slack alerts |
| 6 | `lesson-06-yaml.md` | PyYAML read/write, K8s manifests | Parse and generate Kubernetes, Ansible, GitHub Actions configs |
| 7 | `lesson-07-jinja2.md` | Jinja2 templates, loops, conditionals | Generate nginx configs, K8s manifests, systemd units |
| 8 | `lesson-08-venv-pip.md` | venv, pip, requirements.txt | Isolated dependencies, reproducible environments |
| 9 | `lesson-09-debugging.md` | print/f=, pdb, assert, logging | Find bugs in scripts that run but give wrong output |

---

## Your 10-Minute Sprint Formula (same as Phase 1)

1. **Read** "Why This Matters" (2 min)
2. **Copy** one code block that looks useful (1 min)
3. **Run** it (2 min)
4. **Change** one value and run again (2 min)
5. **Write** one line in your own words (3 min)

---

## What You Can Build After Phase 2

By the time you finish these 9 lessons, you'll be able to write:

- A deployment script that runs `kubectl`, checks rollout status, and alerts Slack on failure
- A health checker that pings all your services and emails if any are down
- A config generator that reads a YAML services file and generates nginx configs for all of them
- A log analyzer that reads your server logs, counts errors by type, and writes a report
- A CLI tool with proper argument handling, logging to file, and exception handling

---

## The Build Project (after all 9 lessons)

Write one real script that combines at least 5 of these lessons:

**Suggestion — "psl-deploy" CLI tool:**
- Reads config from env vars (Lesson 4)
- Parses a YAML services file (Lesson 6)
- Generates K8s manifests from Jinja2 templates (Lesson 7)
- Runs `kubectl apply` using subprocess (Lesson 3)
- Calls a health check API to verify deployment (Lesson 5)
- Logs everything to file + console (Lesson 2)
- Handles errors gracefully — bad YAML, kubectl failure, health check timeout (Lesson 1)
- Sends a Slack alert with the result (Lesson 5)

That's a real DevOps tool. That's what Python is for.

---

> **Phase 3 next:** Decorators, generators, async, classes — the patterns that power AI agent frameworks like LangGraph.

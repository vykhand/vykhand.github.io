# Introducing tinystructlog: Minimalistic Context-Aware Logging for Python

## Why Another Logging Library?

I've spent way too many hours trying different structured logging libraries for Python. Every time, I'd get excited about the features, then realize I'm spending more time configuring the logger than actually writing code. Configuration files, processor pipelines, custom formatters, a dozen dependencies just to log some messages—it all felt like overkill.

What I actually needed was pretty simple: timestamped, colored output with some context (like `request_id` or `user_id`) automatically included in every log line. That's it. No fancy serialization, no log rotation schemes, no external dependencies that might break in six months.

So I did what any developer does—I wrote my own tiny logging wrapper and started copy-pasting it between projects. For a while, this worked great. But after copying the same 200 lines of code for the tenth time, I thought "this is ridiculous" and finally published it as **tinystructlog**.

---

## The Problem: Tracing Context Across Your Application

If you've built async web services or multi-tenant apps, you know the pain: you need to track which request, user, or tenant generated each log line. Otherwise, when something breaks in production, you're stuck grep-ing through logs without any way to filter by context.

The obvious solution is to manually add context to every log statement:

```python
def process_order(order_id, user_id):
    logger.info(f"[order_id={order_id}] [user_id={user_id}] Processing order")
    validate_order(order_id, user_id)
    charge_payment(order_id, user_id)
    ship_order(order_id, user_id)

def validate_order(order_id, user_id):
    logger.info(f"[order_id={order_id}] [user_id={user_id}] Validating order")
    # ... and so on, in every single function
```

This gets old fast. You're either passing context through every function signature (polluting your API), or you're manually formatting strings everywhere and inevitably forgetting to include context in half your log statements.

---

## The Solution: tinystructlog

So I built [**tinystructlog**](https://github.com/Aprova-GmbH/tinystructlog). It's a small Python library that uses `contextvars` to automatically inject context into all your log messages. Set the context once at the start of your request handler, and it magically appears in every log line—no manual formatting, no passing context through function parameters.

### Installation

```bash
pip install tinystructlog
```

Python 3.11+, zero runtime dependencies. That's the whole installation story.

---

## How It Works

```python
from tinystructlog import get_logger, set_log_context

log = get_logger(__name__)

# Set context once
set_log_context(user_id="12345", request_id="abc-def")

# All subsequent logs automatically include context
log.info("Processing order")
# Output: [2026-01-17 10:30:45] [INFO] [main:10] [request_id=abc-def user_id=12345] Processing order

log.info("Validating payment")
# Output: [2026-01-17 10:30:46] [INFO] [main:12] [request_id=abc-def user_id=12345] Validating payment
```

That's it. Set the context once, and it shows up in every log line automatically. No more manual string formatting, no more function parameter pollution.

---

## Key Features

### 1. Thread & Async Safe
Built on Python's `contextvars`, tinystructlog provides perfect isolation across threads and async tasks:

```python
import asyncio
from tinystructlog import get_logger, set_log_context

log = get_logger(__name__)

async def handle_request(user_id: str, request_id: str):
    set_log_context(user_id=user_id, request_id=request_id)
    log.info("Processing request")
    await do_work()

# These run concurrently with isolated contexts
await asyncio.gather(
    handle_request("user1", "req001"),
    handle_request("user2", "req002"),
)
```

Each async task maintains its own isolated context. No cross-contamination.

### 2. Colored Terminal Output
Logs are color-coded by level (DEBUG=cyan, INFO=green, WARNING=yellow, ERROR=red, CRITICAL=magenta) with dimmed source locations for better readability.

### 3. Temporary Context with Context Managers

```python
from tinystructlog import log_context

with log_context(operation="cleanup", task_id="task-123"):
    log.info("Starting cleanup")  # Includes operation and task_id
    perform_cleanup()
# Context automatically removed after the block
```

### 4. Zero Configuration
Works out of the box with sensible defaults. Configure via `LOG_LEVEL` environment variable if needed.

---

## Real-World Use Cases

### FastAPI Integration

```python
from fastapi import FastAPI, Request
from tinystructlog import get_logger, set_log_context, clear_log_context
import uuid

app = FastAPI()
log = get_logger(__name__)

@app.middleware("http")
async def add_context(request: Request, call_next):
    set_log_context(
        request_id=str(uuid.uuid4()),
        path=request.url.path,
        method=request.method,
    )

    log.info("Request started")
    response = await call_next(request)
    log.info("Request completed")

    clear_log_context()
    return response
```

### Multi-Tenant SaaS
Track which tenant owns each log entry:

```python
set_log_context(tenant_id="tenant-123", user_id="user-456")
log.info("Fetching tenant data")  # Automatically tagged with tenant_id
```

### Background Workers
Each job gets its own isolated context:

```python
async def process_job(job_id: str, job_type: str):
    set_log_context(job_id=job_id, job_type=job_type, worker_id="worker-1")

    log.info("Job started")
    await execute_job()
    log.info("Job completed")
```

---

## Why Not structlog?

**structlog** is excellent but feature-heavy with external dependencies. If you need JSON output, structured event processing, or extensive customization, use structlog.

**tinystructlog** is minimalistic:
- Zero runtime dependencies
- Focused purely on context management
- Sensible defaults for 90% of use cases
- Smaller footprint, faster to learn

Think of it as the logging equivalent of `requests` (simple, focused) versus `httpx` (feature-rich, complex).

---

## Project Status

- **Version:** 0.1.0
- **License:** MIT
- **Repository:** [github.com/Aprova-GmbH/tinystructlog](https://github.com/Aprova-GmbH/tinystructlog)
- **Documentation:** Coming soon on ReadTheDocs
- **Test Coverage:** 100%
- **Python Support:** 3.11, 3.12, 3.13

---

## Design Philosophy

tinystructlog follows a few simple principles:

1. **Zero dependencies for zero friction**: No runtime dependencies means no version conflicts, no security vulnerabilities from transitive deps, and instant installation.

2. **Do one thing well**: Context management. That's it. No JSON encoding, no syslog handlers, no custom formatters beyond colored output.

3. **Type hints everywhere**: Full type hint support for better IDE autocomplete and static analysis.

4. **Async-first, but sync-compatible**: Built on `contextvars`, which works seamlessly in both sync and async code.

---

## Try It

```bash
pip install tinystructlog
```

Or check out the [GitHub repository](https://github.com/Aprova-GmbH/tinystructlog) for examples and documentation.

If you're building Python applications with async workers, web frameworks, or multi-tenant systems, give tinystructlog a try. It solves one problem—context propagation—and solves it well.

---

**Questions or feedback?** Open an issue on [GitHub](https://github.com/Aprova-GmbH/tinystructlog/issues) or reach out at vya@aprova.ch.

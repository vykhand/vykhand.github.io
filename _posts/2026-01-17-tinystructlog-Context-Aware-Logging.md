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

## What Makes It Different

### Thread & Async Safe by Default
Because it's built on Python's `contextvars`, each async task or thread gets its own isolated context. No weird bugs where logs from concurrent requests get mixed up:

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

Each task has its own context. No cross-contamination, no mutexes, no headaches.

### Colored Output That Doesn't Hurt
Logs are color-coded by level (DEBUG=cyan, INFO=green, WARNING=yellow, ERROR=red, CRITICAL=magenta) with dimmed source locations. It's just nicer to read.

### Temporary Context When You Need It

```python
from tinystructlog import log_context

with log_context(operation="cleanup", task_id="task-123"):
    log.info("Starting cleanup")  # Includes operation and task_id
    perform_cleanup()
# Context automatically removed after the block
```

Perfect for adding temporary context that only applies to a specific code block.

### Zero Configuration
Literally zero configuration. Import it, use it. If you want to change the log level, set the `LOG_LEVEL` environment variable. That's the entire configuration story.

---

## Real-World Examples

### FastAPI Integration

Here's how I use it in FastAPI apps—just add a middleware that sets context at the start of each request:

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

Now every log line in your entire request handling code will include the request ID automatically.

### Multi-Tenant SaaS
If you're building a multi-tenant app, this is a lifesaver. Just set the tenant ID once and filter logs by tenant in your log aggregation tool:

```python
set_log_context(tenant_id="tenant-123", user_id="user-456")
log.info("Fetching tenant data")  # Automatically tagged with tenant_id
```

### Background Workers
Each background job gets its own isolated context. Perfect for Celery, RQ, or whatever task queue you're using:

```python
async def process_job(job_id: str, job_type: str):
    set_log_context(job_id=job_id, job_type=job_type, worker_id="worker-1")

    log.info("Job started")
    await execute_job()
    log.info("Job completed")
```

---

## How Does It Compare to Other Libraries?

### vs. loguru

[loguru](https://github.com/Delgan/loguru) is probably the most popular "better logging" library for Python. It's great, but it's solving a different problem.

**loguru** gives you advanced features like automatic log rotation, better exception formatting, and structured JSON output. But for context management, you need to manually call `.bind()` every time:

```python
from loguru import logger

# You need to bind context explicitly
context_logger = logger.bind(request_id="abc-def", user_id="12345")
context_logger.info("Processing request")
```

With **tinystructlog**, you set the context once using `contextvars`, and it automatically propagates to all log calls in the current async task or thread—without explicit binding:

```python
from tinystructlog import get_logger, set_log_context

log = get_logger(__name__)
set_log_context(request_id="abc-def", user_id="12345")
log.info("Processing request")  # Context automatically included
```

**When to use loguru:** You need log rotation, structured JSON output, or really nice exception traces.

**When to use tinystructlog:** You just want automatic context propagation with zero dependencies and zero configuration.

### vs. structlog

**structlog** is the heavyweight champion of structured logging. It's incredibly powerful but also complex—processors, formatters, event dictionaries, binding contexts. If you need that level of control, use structlog.

**tinystructlog** is the opposite: zero dependencies, zero configuration, four functions in the entire API. If you just need context-aware logging without the ceremony, tinystructlog is simpler.

Think of it like `requests` (simple, focused) vs `httpx` (feature-rich, complex). Both are great, just different philosophies.

---

## Project Status

- **Version:** 0.1.2
- **License:** MIT
- **Repository:** [github.com/Aprova-GmbH/tinystructlog](https://github.com/Aprova-GmbH/tinystructlog)
- **Documentation:** [tinystructlog.readthedocs.io](https://tinystructlog.readthedocs.io)
- **Test Coverage:** 100%
- **Python Support:** 3.11, 3.12, 3.13

---

## Design Philosophy

I kept tinystructlog intentionally small:

1. **Zero dependencies**: No runtime dependencies means no version conflicts, no surprise security vulnerabilities from transitive deps, and instant installation. Just pure Python.

2. **Do one thing**: Context management. That's it. I'm not trying to build log rotation, syslog handlers, or custom serialization. There are already libraries for that.

3. **Type hints everywhere**: Full type hints for better IDE autocomplete. If your editor supports it, you'll get nice autocompletion.

4. **Async-first, but sync works too**: Built on `contextvars`, which works seamlessly in both sync and async code. You don't need to think about it.

---

## Try It Out

```bash
pip install tinystructlog
```

Check out the [GitHub repository](https://github.com/Aprova-GmbH/tinystructlog) for more examples and full documentation.

If you're building async web services, background workers, or multi-tenant apps and you're tired of manually threading context through every function call, give it a shot. It's a small library that does one thing well.

---

**Questions or feedback?** Open an issue on [GitHub](https://github.com/Aprova-GmbH/tinystructlog/issues) or email me at vya@aprova.ch.

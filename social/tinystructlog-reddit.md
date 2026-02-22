# tinystructlog Reddit Posts

## r/Python - Main Post

**Title:** [P] tinystructlog: Context-aware logging that doesn't get in your way

After copying the same 200 lines of logging code between projects for the tenth time, I finally published it as a library.

**The problem:** You need context (request_id, user_id, tenant_id) in your logs, but you don't want to:
1. Pass context through every function parameter
2. Manually format every log statement
3. Use a heavyweight library with 12 dependencies

**The solution:**
```python
from tinystructlog import get_logger, set_log_context

log = get_logger(__name__)

# Set context once
set_log_context(request_id="abc-123", user_id="user-456")

# All logs automatically include context
log.info("Processing order")
# [2026-01-28 10:30:45] [INFO] [main:10] [request_id=abc-123 user_id=user-456] Processing order

log.info("Charging payment")
# [2026-01-28 10:30:46] [INFO] [main:12] [request_id=abc-123 user_id=user-456] Charging payment
```

**Key features:**
- Built on `contextvars` - thread & async safe by default
- Zero runtime dependencies
- Zero configuration (import and use)
- Colored output by log level
- Temporary context with `with log_context(...):`

**FastAPI example:**
```python
@app.middleware("http")
async def add_context(request: Request, call_next):
    set_log_context(
        request_id=str(uuid.uuid4()),
        path=request.url.path,
    )
    response = await call_next(request)
    clear_log_context()
    return response
```

Now every log in your entire request handling code includes the request_id automatically. Perfect for multi-tenant apps, microservices, or any async service.

**vs loguru:** loguru is great for advanced features (rotation, JSON output). tinystructlog is focused purely on automatic context propagation with zero config.

**vs structlog:** structlog is powerful but complex. tinystructlog is 4 functions, zero dependencies, zero configuration.

GitHub: https://github.com/Aprova-GmbH/tinystructlog
PyPI: `pip install tinystructlog`
Blog: https://vykhand.github.io/tinystructlog-Context-Aware-Logging/

MIT licensed, Python 3.11+, 100% test coverage.

---

## r/FastAPI - FastAPI-focused Post

**Title:** Simple middleware for automatic request context in logs

Been using this pattern in all my FastAPI projects - thought others might find it useful.

Problem: You want every log line to include request_id, user_id, etc., but you don't want to pass context through every function.

Solution using tinystructlog (tiny library with zero dependencies):

```python
from fastapi import FastAPI, Request
from tinystructlog import get_logger, set_log_context, clear_log_context
import uuid

app = FastAPI()
log = get_logger(__name__)

@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    set_log_context(
        request_id=str(uuid.uuid4()),
        path=request.url.path,
        method=request.method,
    )

    log.info("Request started")
    response = await call_next(request)
    log.info("Request completed", status=response.status_code)

    clear_log_context()
    return response

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    log.info("Fetching user")  # Automatically includes request_id!
    # ... your code
```

Built on contextvars so each async request gets isolated context (no cross-contamination).

GitHub: https://github.com/Aprova-GmbH/tinystructlog
`pip install tinystructlog`

---

## r/asyncio - Async-focused Post

**Title:** Context-aware logging for async Python using contextvars

Built a tiny logging wrapper that uses `contextvars` to automatically propagate context across async tasks.

Example:
```python
from tinystructlog import get_logger, set_log_context
import asyncio

log = get_logger(__name__)

async def handle_request(user_id: str, request_id: str):
    set_log_context(user_id=user_id, request_id=request_id)

    log.info("Processing request")
    await do_work()
    log.info("Request complete")

# Run concurrent tasks - each has isolated context
await asyncio.gather(
    handle_request("user1", "req001"),
    handle_request("user2", "req002"),
)
```

Each task gets its own context automatically (thanks to contextvars). No global state, no locks, no cross-contamination between tasks.

Useful for:
- Web frameworks (FastAPI, Starlette, aiohttp)
- Async background workers
- Multi-tenant applications
- Any async service where you need to track context

Zero dependencies, zero configuration, MIT licensed.

GitHub: https://github.com/Aprova-GmbH/tinystructlog
PyPI: `pip install tinystructlog`

---

## r/webdev - Web Developer Perspective

**Title:** Stop passing request_id through every function - use context vars instead

Quick tip for async web services (FastAPI, Flask, etc.):

Instead of this:
```python
def process_order(order_id, request_id):
    logger.info(f"[{request_id}] Processing {order_id}")
    validate_order(order_id, request_id)
    charge_payment(order_id, request_id)

def validate_order(order_id, request_id):
    logger.info(f"[{request_id}] Validating {order_id}")
```

Use context variables:
```python
set_log_context(request_id=request_id)

# All logs automatically include request_id
log.info("Processing order", order_id=order_id)
log.info("Validating order")
```

I packaged this pattern into tinystructlog (zero dependencies, zero config):
- Set context once in middleware
- Every log automatically includes it
- Thread & async safe (each request isolated)

`pip install tinystructlog`
https://github.com/Aprova-GmbH/tinystructlog

Works with any Python web framework.

---

## r/madeinpython - Show & Tell Style

**Title:** tinystructlog - Finally packaged my logging snippet after copying it 10+ times

Hey r/madeinpython!

You know when you have a code snippet you keep copying between projects? I finally turned mine into a library.

**The problem I kept solving:** Every FastAPI/async service needs request_id in logs, but passing it through every function is annoying:

```python
def process_order(order_id, request_id):  # Ugh
    logger.info(f"[{request_id}] Processing {order_id}")
    validate_order(order_id, request_id)  # Still passing it
```

**My solution - tinystructlog:**

```python
from tinystructlog import get_logger, set_log_context

log = get_logger(__name__)

# Set context once (e.g., in FastAPI middleware)
set_log_context(request_id="abc-123", user_id="user-456")

# Every log automatically includes it
log.info("Processing order")
# [2026-01-28 10:30:45] [INFO] [main:10] [request_id=abc-123 user_id=user-456] Processing order
```

**Why it's nice:**
- Built on `contextvars` (thread & async safe)
- Zero dependencies
- Zero configuration
- Colored output
- 4 functions in the whole API

Perfect for FastAPI, multi-tenant apps, or any service where you need to track context across async tasks.

**Stats:**
- 0.1.2 on PyPI (`pip install tinystructlog`)
- MIT licensed
- 100% test coverage
- Python 3.11+

GitHub: https://github.com/Aprova-GmbH/tinystructlog
PyPI: `pip install tinystructlog`
Blog: https://vykhand.github.io/tinystructlog-Context-Aware-Logging/

It's tiny (hence the name) but saves me so much time!
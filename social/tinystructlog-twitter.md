# tinystructlog - Twitter Thread

## Tweet 1 (Main)
I got tired of copy-pasting the same logging code between projects, so I published it as a package.

tinystructlog: Context-aware logging for Python with zero dependencies.

Set context once, it appears in all your logs automatically. Perfect for async web services, background workers, multi-tenant apps.

📝 Blog post: https://vykhand.github.io/tinystructlog-Context-Aware-Logging/
🔗 Repo: https://github.com/Aprova-GmbH/tinystructlog

## Tweet 2
The problem: You need to trace which user/request/tenant generated each log line.

The usual solution: Manually add [user_id=123] to every log statement.

It's tedious and you'll forget half the time.

## Tweet 3
tinystructlog uses Python's `contextvars` to automatically inject context:

```python
from tinystructlog import get_logger, set_log_context

log = get_logger(__name__)
set_log_context(user_id="12345", request_id="abc-def")

log.info("Processing order")
# Context included automatically ✨
```

## Tweet 4
Thread & async safe by default. Each async task gets its own isolated context.

No weird bugs where logs from concurrent requests get mixed up. It just works.

## Tweet 5
Why not loguru or structlog?

loguru: Great features, but requires explicit .bind() for context
structlog: Powerful but complex (processors, formatters, etc.)

tinystructlog: Does one thing—automatic context propagation. Zero deps, zero config.

## Tweet 6
Perfect for:
✅ FastAPI/Flask apps (middleware adds context per-request)
✅ Multi-tenant SaaS (filter logs by tenant_id)
✅ Background jobs (Celery, RQ, etc.)
✅ Any async Python code

Python 3.11+, MIT licensed, 100% test coverage

## Tweet 7 (Final)
Install:
```
pip install tinystructlog
```

Repo: https://github.com/Aprova-GmbH/tinystructlog
Docs: https://tinystructlog.readthedocs.io

If you've built async services, you know the pain of tracing context. This library solves it cleanly.

Would love your feedback! 🙏
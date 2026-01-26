# I just published tinystructlog 🎉

After copy-pasting the same 200 lines of logging code between projects for the last few years, I finally got tired of it and published it on PyPI.

**The problem:** When you're debugging production issues in async web services or multi-tenant apps, you need to know which request, user, or tenant generated each log line. Otherwise you're stuck grep-ing through logs with no way to filter.

The usual solutions are tedious:
- Manually add `[user_id=123]` to every log statement
- Pass context through every function signature (polluting your API)
- Use a complex logging library with dozens of dependencies

**tinystructlog** is simpler: Set context once using `contextvars`, and it automatically appears in all your logs. No manual formatting, no parameter pollution.

```python
from tinystructlog import get_logger, set_log_context

log = get_logger(__name__)
set_log_context(user_id="12345", request_id="abc-def")

log.info("Processing order")
# Output: [2026-01-17 10:30:45] [INFO] [request_id=abc-def user_id=12345] Processing order
```

## Why not loguru or structlog?

**loguru** is great for log rotation and exception formatting, but requires explicit `.bind()` calls for context. **structlog** is powerful but complex with processors and formatters.

**tinystructlog** does one thing: automatic context propagation using Python's `contextvars`. Zero dependencies, zero config, four functions in the entire API.

## The details:

✅ Zero runtime dependencies
✅ Thread & async-safe (each task gets isolated context)
✅ Works with FastAPI, background workers, multi-tenant apps
✅ Colored terminal output
✅ Python 3.11+, MIT licensed, 100% test coverage

If you're tired of manually threading context through every function call, give it a try:

```bash
pip install tinystructlog
```

**Repository:** https://github.com/Aprova-GmbH/tinystructlog
**Docs:** https://tinystructlog.readthedocs.io

Would love to hear your feedback! 🙏

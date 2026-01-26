# Introducing tinystructlog

After years of trying every structured logging library in Python and being overwhelmed by their complexity, I did what every developer does: wrote my own.

For the last few years, I've been copying the same 200-line logging package between my Python projects. It does exactly what I need—**context-aware logging with zero configuration**—and nothing more.

I finally stopped the copy-paste madness and published it on PyPI as **tinystructlog**.

## The pitch is simple:

- **Zero dependencies**
- **Thread and async-safe** context propagation
- One line to set context, automatic injection everywhere
- Colored output that doesn't hurt your eyes
- Works with FastAPI, background workers, multi-tenant apps

Built on `contextvars`. Python 3.11+. MIT licensed. 100% test coverage.

---

If you've ever needed to trace requests across function calls without polluting every signature with `user_id` and `request_id` parameters, this might save you some time.

## Install

```bash
pip install tinystructlog
```

**Repository:** https://github.com/Aprova-GmbH/tinystructlog

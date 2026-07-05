# Task 2: Dependency Chaos

## Scenario (Mode B - Production Outage)
A developer on your team was reading about systemd dependencies. They wanted to make sure the `web-api.service` didn't start until the `redis-cache.service` was up. They added the following to `web-api.service`:

```ini
[Unit]
Requires=redis-cache.service
After=redis-cache.service
```

At 3:00 AM, the Redis cache ran out of memory and was OOM-Killed by the kernel. Because of the `Requires=` directive, systemd immediately and forcibly shut down the `web-api.service` as well, taking down the entire public-facing website instead of just degrading gracefully.

## Your Task
1. Explain the fundamental difference between `Requires=` and `Wants=`.
2. How would you rewrite the `[Unit]` section of `web-api.service` so that it always starts *after* Redis on boot, but stays running even if Redis suddenly crashes?

## Solution
1. Requires is a strict critera whereas Wants is optional. In this context, it means that if redis-cache is required, then if redis-cache crashes, web-api will also crash. If redis-cache is wanted, then if redis-cache crashes, the web-api will continue to run, though the parts of web-api that uses the crash will return a error.
2. I will replace Requires with Wants.

**Correction:**
Spot on! `Wants=` allows for graceful degradation. 
One small detail: You would keep `After=redis-cache.service` in the file. `Wants=` ensures systemd *tries* to start Redis, and `After=` ensures it waits for Redis to start before starting the API. But if Redis crashes later, `Wants=` prevents the API from being dragged down with it.

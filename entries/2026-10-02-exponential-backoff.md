# 🔄 Async Retry with Exponential Backoff & Full Jitter

## Overview
When interacting with third-party APIs, webhooks, or distributed databases, transient network hiccups and HTTP 429 (Rate Limit) errors are inevitable. Simple fixed retries can cause a **"thundering herd"** problem where multiple failing requests retry simultaneously, crashing downstream servers.

The optimal industry solution combines **Exponential Backoff** with **Full Jitter**.

---

## The Formula
Instead of a static backoff:
$$\text{delay} = \text{base\_delay} \times 2^{\text{attempt}}$$

Full Jitter randomizes the delay uniformly between 0 and the exponential cap:
$$\text{sleep} = \text{random}(0, \min(\text{max\_delay}, \text{base\_delay} \times 2^{\text{attempt}}))$$

---

## 💻 TypeScript Implementation

```typescript
export interface RetryOptions {
  maxRetries?: number;
  baseDelayMs?: number;
  maxDelayMs?: number;
}

export async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  options: RetryOptions = {}
): Promise<T> {
  const { maxRetries = 4, baseDelayMs = 500, maxDelayMs = 8000 } = options;

  let attempt = 0;

  while (true) {
    try {
      return await fn();
    } catch (error) {
      attempt++;
      if (attempt > maxRetries) {
        throw error;
      }

      // Calculate exponential ceiling
      const cap = Math.min(maxDelayMs, baseDelayMs * Math.pow(2, attempt));
      // Apply full jitter
      const sleepMs = Math.random() * cap;

      console.warn(`[Retry] Attempt ${attempt} failed. Retrying in ${Math.round(sleepMs)}ms...`);
      await new Promise((resolve) => setTimeout(resolve, sleepMs));
    }
  }
}
```

---

## 🐍 Python Implementation

```python
import time
import random
from typing import Callable, TypeVar, Any

T = TypeVar("T")

def retry_with_backoff(
    func: Callable[..., T],
    max_retries: int = 4,
    base_delay: float = 0.5,
    max_delay: float = 8.0,
    *args: Any,
    **kwargs: Any
) -> T:
    attempt = 0
    while True:
        try:
            return func(*args, **kwargs)
        except Exception as exc:
            attempt += 1
            if attempt > max_retries:
                raise exc
            
            # Exponential ceiling
            cap = min(max_delay, base_delay * (2 ** attempt))
            # Full Jitter
            sleep_sec = random.uniform(0, cap)
            
            print(f"[Retry] Attempt {attempt} failed: {exc}. Retrying in {sleep_sec:.2f}s...")
            time.sleep(sleep_sec)
```

---

## Key Takeaways
1. **Never retry non-idempotent mutations** without idempotency keys.
2. **Respect `Retry-After` headers** whenever returned by the server.
3. **Full Jitter** provides the highest throughput and lowest queue delay under contention compared to equal jitter or no jitter.

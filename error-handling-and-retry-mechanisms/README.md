<h1>
  <span class="headline">Selenium: Debugging and Stabilizing Tests</span>
  <span class="subhead">Error Handling and Retry Mechanisms</span>
</h1>

**Learning objective:** Implement retry patterns and exception handling to stabilize tests in dynamic or flaky environments.

## Why retry logic matters

Even with good waits and selectors, tests sometimes fail due to:

- Dynamic UI behavior (ex: element disappears and reappears)
- Flaky timing in CI environments
- Transient browser states or animations

Retries give your test **a second chance** when something rare and temporary causes a failure.

> 🧠 Think of waits as "wait until ready" and retries as "try again if it briefly fails."

## When to use retries vs. waits

| Use waits when...                            | Use retries when...                                 |
| -------------------------------------------- | --------------------------------------------------- |
| Waiting for an element to appear or be ready | An action fails sporadically (ex: click fails once) |
| You can define a clear UI condition          | The failure is non-deterministic and rare           |
| You want to pause until the UI is ready      | You want to recover from timing hiccups             |

> ⚠ Avoid using retries to patch over poor waits or bad selectors.

## Reusable retry decorator

Use this decorator to retry any function if it raises specific exceptions:

```python
import time
from functools import wraps
from selenium.common.exceptions import TimeoutException, StaleElementReferenceException

def retry_on_exception(exceptions, tries=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, tries + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == tries:
                        raise
                    print(f"Attempt {attempt} failed: {e}. Retrying in {delay}s...")
                    time.sleep(delay)
        return wrapper
    return decorator
```

## Applying it in a Selenium test

Let’s say this line sometimes fails with a `StaleElementReferenceException`:

```python
driver.find_element(By.ID, 'checkout').click()
```

Refactor it:

```python
from selenium.common.exceptions import StaleElementReferenceException

@retry_on_exception((StaleElementReferenceException,), tries=3, delay=2)
def click_checkout(driver):
    driver.find_element(By.ID, 'checkout').click()

click_checkout(driver)
```

## Practice scenario: Retry a flaky button click

Here’s a brittle test:

```python
def test_place_order(driver):
    driver.get("https://shop.example.com")
    driver.find_element(By.ID, "buy-now").click()
    confirmation = driver.find_element(By.ID, "order-confirmed")
    assert "Order confirmed" in confirmation.text
```

### Your refactor:

1. Move the flaky line to a helper function
2. Apply the retry decorator to catch both `StaleElementReferenceException` and `ElementClickInterceptedException`

```python
@retry_on_exception(
    (StaleElementReferenceException, ElementClickInterceptedException),
    tries=3,
    delay=1
)
def click_buy_now(driver):
    driver.find_element(By.ID, "buy-now").click() # flaky line

def test_place_order(driver):
    driver.get("https://shop.example.com")
    click_buy_now(driver)
    confirmation = driver.find_element(By.ID, "order-confirmed")
    assert "Order confirmed" in confirmation.text
```

## Final advice

✅ Use retries only when:

- The error is rare
- The cause is environmental or timing-based
- You’ve already validated waits and selectors

🚫 Don’t use retries:

- As a replacement for proper waits
- On every action (it slows tests and hides bugs)
- If the error is consistent (that’s not flaky—it’s broken)

> 🔍 Be intentional. Retrying the right way means fewer random failures, better diagnostics, and stronger trust in your test suite.

<div class="activity discussion">
  <h2 class="title">Retry Patterns: Thought Exercise</h2>
  <span class="minutes">5 min</span>
</div>

We've explored how retries can help stabilize flaky Selenium tests—especially when actions fail due to rare timing issues, like animations or network delays in CI environments.

For now, rather than diving into code, we’ll use this scenario to **think through how you’d approach a problem**. You’ll get more chances to practice this pattern in real-world tests and larger projects later in the course.

### Scenario:

You have a Selenium test where clicking a **Login** button **sometimes fails** with a `TimeoutException` during CI runs, even though you’ve already validated your waits and selectors.

- What retry strategy would you use to handle this **without making the entire test slower or harder to maintain**?
- Would you:
  - Retry every interaction?
  - Retry just the click?
  - Take a different approach?
- How would you make sure that if retries still fail, the **error is clear and helpful** for whoever reads the report?
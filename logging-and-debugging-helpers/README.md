<h1>
  <span class="headline">Selenium: Debugging and Stabilizing Tests</span>
  <span class="subhead">Logging and Debugging Helpers</span>
</h1>

**Learning objective:** Write logging statements and debug helpers that make future test failures easier to analyze.

Logging is your lens into test automation. While previous lessons laid the foundation for stabilizing and refactoring your Selenium tests, in this lesson you will bridge the gap between failures and fast, effective solutions. Whether you are running tests on your own computer or in a shared environment, strong logging and reusable debug helpers empower you and your team to fix problems before they grow.

## Why logs matter

When a test fails, a vague message like “Clicked button” doesn’t help. But a message like:

`[INFO] Clicked submit button (By.ID: 'submit') on login page`

...tells you exactly what went wrong and where. Good logs save time, help you diagnose failures faster, and are essential for team collaboration and CI debugging.

## What makes logs helpful in real projects?

Robust logs are not about capturing more—they are about capturing better. To be truly helpful, log output should be:

- **Organized by level:**  
  Use logging levels such as INFO, WARNING, or ERROR to prioritize what matters most.
- **Context-rich:**  
  Include details like the test name, action, locator used, and current browser or environment.
- **Readable at a glance:**  
  Write skimmable, meaningful messages rather than generic statements.
- **Actionable:**  
  Provide the right clues to swiftly reproduce, diagnose, and fix issues. This could be the selector used, the input provided, or the page visited.

> 🏆 Quality over quantity: A clear log of one failed click with step details, test name, and timestamp beats a screen of scrolling stack traces every time.

## Basic logging with context

### Vague log messages:

```python
print('Clicked button')
print('Login tested')
```

If the test breaks, this tells you almost nothing—what page, which button, which user?

### Improved, context-rich logs:

```python
print('[INFO] Clicking submit button on login page (By.ID: "submit")')
print('[INFO] Successfully logged in as user "learner2024"')
```

Notice how these improved logs quickly answer: What did I try to do? Where? With what data or selector?

## Structuring logs with Python’s logging module

Reliance on `print()` statements often makes logs hard to search, sort, or analyze. Python’s built-in _logging_ module helps you add levels, timestamps, and consistent structure—making logs more useful for both solo and collaborative debugging.

Replace `print()` with Python’s `logging` module to get structured, timestamped output:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='[%(asctime)s] [%(levelname)s] %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

logging.info("Navigating to login page")
logging.error("Submit button not found")
```

The `%` tokens in your `logging.basicConfig()` format string are placeholders used by Python’s logging module to inject dynamic values into your log messages at runtime.

## Reusable logging with decorators

Repeating similar log statements or error handling across dozens of Selenium steps is not only tedious—it invites mistakes. Instead, you can _abstract_ repeating logic into a logging decorator or helper function, giving every test step consistent, contextual logs.

Instead of repeating log statements, use a helper like this:

```python
from functools import wraps

def log_action(action_name):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            logging.info(f'Starting: {action_name}')
            try:
                result = func(*args, **kwargs)
                logging.info(f'Finished: {action_name}')
                return result
            except Exception as error:
                logging.error(f'Failed: {action_name} | {error}')
                if args and hasattr(args[0], 'save_screenshot'):
                    filename = f'fail_{action_name}.png'
                    args[0].save_screenshot(filename)
                    logging.error(f'Screenshot saved: {filename}')
                raise
        return wrapper
    return decorator
```

Now wrap key test steps:

```python
@log_action("Click Checkout")
def click_checkout(driver):
    driver.find_element(By.ID, 'checkout').click()
```

> Use decorators to avoid copy-pasting logging logic across tests.

## Optional: Capture screenshots or page source

Occasionally, understanding _why_ a step failed requires more than words. Capturing a screenshot or page source at the moment of failure provides crucial visual evidence—especially for diagnosing timing problems, unexpected overlays, or missing elements.

Add a helper like:

```python
def capture_artifacts(driver, step):
    driver.save_screenshot(f'{step}.png')
    with open(f'{step}.html', 'w') as f:
        f.write(driver.page_source)
```

> 💡 Save artifacts only when needed (ex: inside `except` blocks or failure decorators).

### Example: Upgrading a real Selenium test

```python
def test_checkout(driver):
    logging.info("Start test_checkout")
    driver.get("https://shop.example.com")
    try:
        click_checkout(driver)
    except Exception:
        capture_artifacts(driver, "checkout_fail")
        raise
```

Now, a failure generates rich logs and artifacts, such as:

```bash
[2024-06-21 13:22:10] [INFO] Navigating to shopping site
[2024-06-21 13:22:15] [INFO] Starting step: Click checkout button
[2024-06-21 13:22:16] [ERROR] Step failed: Click checkout button | Exception: no such element: ...
[2024-06-21 13:22:16] [ERROR] Screenshot saved: screenshot_failure_Click_checkout_button.png
```

From here, you instantly know where and why the test broke and have evidence ready for deeper investigation or team review.

> This approach turns every test into a learning opportunity—giving you clues for fixes now, and a roadmap for faster diagnosis in the future.

<div class="activity guided-walkthrough">
  <h2 class="title">Practice adding robust logging and helpers</h2>
  <span class="minutes">10 min</span>
</div>
 
Apply structured logging and reusable helpers to transform a basic Selenium test into a maintainable, diagnosable one—gaining confidence in debugging for both individual and team use.

**For this exercise, use the 📄 `example_3_login.html` file created during setup.**

<img src="./assets/login.png" alt="" width="700" />

<br>

1. **Set up the example test case:**

**Create a new 📄 `login_test.py` test file and use the following template:**

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()

driver.get("file:///Users/username/path/to/example_file.html")  # Replace with the full absolute path to your local HTML file

driver.find_element(By.ID, 'username').send_keys('learner2024')
driver.find_element(By.ID, 'password').send_keys('securetest!')
driver.find_element(By.ID, 'submit').click()
driver.quit()
```

1. **Configure structured logging:**  
   At the top of your script, initialize the logging module to show timestamps and levels.

2. **Create and use debug helpers:**

   - Build a logging decorator like `log_action` for major Selenium actions.
   - Construct a `capture_artifacts_on_failure` helper for failed steps.

3. **Refactor test steps:**

   - Move each Selenium step (filling username, password, clicking submit) into its own function.
   - Decorate each with your logging decorator.
   - Use try/except in your main test to collect artifacts when failures happen.

4. **Run and diagnose:**
   - Deliberately break a selector (such as the submit button) to trigger a failure.
   - Observe the logs.
   - Check the saved screenshot and HTML to see moment of failure.

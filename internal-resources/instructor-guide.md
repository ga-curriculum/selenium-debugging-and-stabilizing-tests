<h1>
  <span class="headline">Selenium: Debugging and Stabilizing Tests</span>
  <span class="subhead">Instructor Guide</span>
</h1>

## Common Selenium Errors

**Purpose of the microlesson:**  
Help learners recognize, interpret, and begin troubleshooting common Selenium errors by connecting technical definitions to scenarios and real-world analogies.

**Knowledge check answers (from self-check section):**

- Exception type: `NoSuchElementException`
- Selenium is trying to find: An `<input>` element with the `name` attribute “email” using XPath.
- Code location: `test_user_login.py`, line 18

**Delivery tips:**

- Emphasize that diagnosing errors is about building pattern recognition rather than memorizing definitions.
- Use a collaborative, problem-solving tone to promote partnership and diverse approaches.
- Incorporate visual assets and analogies for maximum relatability.
- Encourage learners to look for patterns in error logs and to tie the errors back to potential fixes in their own application context.

---

## Debugging Techniques with Browser DevTools

**Purpose:**  
Support learners in applying browser DevTools and debugging artifacts to efficiently investigate and resolve Selenium test failures, building practical diagnostic skills.

**Interaction/knowledge check guidance (from “Quick interaction”):**

- Expected answers should note:
  - In the screenshot: Whether UI overlays, pop-ups, or missing/hidden elements are visible.
  - In the HTML: Whether the targeted selector is present, missing, disabled, or not rendered at all.
  - Learners might observe, for example, that an overlay is covering a button, or that the expected element's HTML exists but the style makes it invisible.

**Suggested solution for the activity:**

- **Artifacts review:**

- The screenshot will show the page UI as expected, including the visible "Launch Modal" button.
- The HTML does not include id="confirm-action" anywhere. Searching for it in DevTools will return no matches.
- A similar, working element (id="launch-action") is present, which may be what the script was supposed to target.

**Additional facilitation tips:**

- Ensure all students have access to a system where they can open and interact with both screenshots and HTML files.
- Encourage exploration in the DevTools panels (Elements, Console, Network) and allow time for learners to practice actively investigating failures, not just reading about steps.
- Prompt learners to label their files with unique, descriptive names.
- In group work or discussion, share sample artifacts that illustrate common points of confusion (hidden elements, overlays, mis-timed elements, etc.).
- Use the discussion prompt to encourage learners to reflect on habits that promote maintainable, team-friendly test code and artifacts.

---

## Best Practices for Flaky Test Prevention

### Expected Student Outcomes

Flaky test symptoms:

- Fails when the spinner is still active
- Clicks the button before it’s enabled
- Tries to read a message that hasn’t appeared yet

Corrected version should:

- Wait for spinner to disappear
- Wait for button to be clickable
- Validate button text (optional)
- Wait for and assert confirmation message text

### 💬 Discussion Prompts

Ask students to reflect or share:

- What made the original version brittle?
- Did their refactored test pass more consistently?
- How would they apply this same fix to another app they’ve tested?

Encourage pairing or quick peer reviews of test logic.

---

## Refactoring for Resilience

Maintainable test design tips

- Group selectors in constants or helper methods
- Use visible text or test IDs over long DOM chains
- Refactor duplicated locator logic
- Leave comments when using non-obvious or fragile selectors

---

## Error Handling and Retry Mechanisms

**Purpose:**  
Facilitate learners’ application of defensive retry patterns and exception handling to real Selenium test scenarios, building toward robust, maintainable automation scripts.

**Knowledge check answer (Mini-challenge):**

- Add a retry decorator (like `retry_on_exception`) only to the function that performs the **Login** button click. Limit the exceptions to `TimeoutException` or other validated rare errors. Ensure you log each attempt and clearly raise the final exception if retries fail, so failures are still diagnosable. Do not add retries to every action.

**Additional facilitation recommendations:**

- Emphasize the distinction between occasional, environment-induced flakiness and issues resolved by good structure or waits.
- Encourage code reviews where retry logic is discussed openly—learners should justify retry choices and document their reasoning.
- Use the scenario-based assets and visuals to walk through failure and recovery, supporting learners as they adapt retry concepts to diverse projects.
- Promote global discussion about what makes error handling “good” in tests for international, user-facing software.
- Provide feedback on the reflection step, focusing on the learner’s rationale for targeted retries and the balance with overall test maintainability.

---

## Logging and Debugging Helpers

**Guided activity solution:**

- Structured logging setup in place (`logging.basicConfig` with informative format).
- Decorators in use to log action start, completion, errors, and save screenshots on failure.
- Broken selector triggers a clear error, an error log, and an image or HTML file.
- Learner shows, ex: screenshot displaying missing or hidden “submit” button, and HTML file confirming absence.
- Reflection addresses:
  - “Logs immediately showed the submit step as failing, with context and error type.”
  - “The screenshot captured the login form missing a submit button, while the HTML also lacked the expected selector.”
  - “This made it quick to spot that the locator was either wrong, or the site changed UI.”

**Facilitation recommendations:**

- Encourage all learners to run a test with a deliberate error and review the log/artifact output.
- Support lagging learners by pairing them for log analysis, ensuring understanding is based on context over content memorization.
- Use the activity as a springboard for a group conversation: “What would help future team members understand our test history?”
- Highlight the importance of log clarity and maintainable helper code for supporting international or distributed teams.

---

🏗️ **Under Construction**

We are constantly working to improve our resources for instructors and students.

**Have something to contribute to this Instructor Guide?** [Let us know](https://pages.git.generalassemb.ly/modular-curriculum-all-courses/universal-resources-internal/module-feedback.html).

# Senior QA Automation (Python / Playwright) - Answers to Say to the Interviewer

# PART 1 - Introduction and Project

### Q1. Tell me about yourself.
"I am a QA automation engineer with X years of experience, Y of them in automation. For the last Z years I have worked mainly with Python, Playwright and pytest, building UI, API and end-to-end suites for [product/domain]. I have also worked with Java, Selenium and Cucumber, so I am comfortable across stacks. Recently I have been using Claude and Copilot to speed up test design and script writing. What attracted me to this role is the mix of analytics workflows, automation and quality strategy."

### Q2. Describe your current project and your role.
"I work on [product], used by [users]. The team has [N] people. My stack is Python, pytest and Playwright, running in [Jenkins / GitHub Actions] on dev, stage and prod-like environments. I own the automation framework, write UI and API tests, do some manual and exploratory testing, define regression scope and mentor juniors. One result I am proud of: I reduced regression time from X to Y."

### Q3. Which automation framework have you built or owned?
"I built a Python framework on pytest and Playwright. It has page objects and components, API clients for test setup, data factories, environment config through env variables, and Allure reporting. It runs in parallel with pytest-xdist in CI. It cut regression time from X to Y and brought flaky failures down from X% to Y%."

### Q4. Biggest technical challenge and how you solved it?
"**Situation:** our regression suite had X% flaky tests and nobody trusted the results. **Task:** I had to bring that down and stabilise the pipeline. **Action:** I classified failures by cause, mostly shared data and fixed waits. I replaced sleeps with web-first assertions, created data per test through the API, and mocked an unstable third-party service. **Result:** flakiness dropped from X% to Y% and runtime fell by Z%."

### Q5. Which JD requirement is your weakest?
"I am strongest in Python, Playwright and API testing. Capital markets is newer to me, so I have been studying the trade lifecycle and market data, and I learn domains fast by working closely with product and data teams."

---

# PART 2 - Playwright with Python

### Q6. Why Playwright, and what are its key features?
"Playwright gives me auto-waiting, isolated browser contexts, Chromium, Firefox and WebKit through one API, native network interception, built-in API testing, and strong debugging with Trace Viewer, codegen and video."

### Q7. Explain Browser, BrowserContext and Page.
"A Browser is the browser process. A BrowserContext is an isolated profile with its own cookies and storage, and it is cheap to create. A Page is a tab inside a context. With pytest-playwright every test gets a fresh context, so I get isolation without relaunching the browser."

### Q8. What is auto-waiting?
"Before every action, Playwright waits until the element is attached, visible, stable, enabled and able to receive events. That removes most explicit waits and a big share of timing flakiness."

### Q9. Which locators do you prefer?
"I prefer `get_by_role`, `get_by_label`, `get_by_text` and `get_by_test_id`, because they mirror how users find elements and survive DOM changes. CSS and XPath are my last resort."
```python
page.get_by_role("button", name="Run analysis").click()
page.locator("tr").filter(has_text="AAPL").get_by_role("button", name="Edit").click()
```

### Q10. Web-first assertions: why `expect` instead of `assert`?
"`expect` retries until the condition is true or times out. A plain `assert` on `is_visible()` checks only once, so it is flaky on dynamic pages."
```python
expect(page.get_by_text("Report ready")).to_be_visible()
expect(page.locator(".row")).to_have_count(5)
```

### Q11. Locator vs ElementHandle?
"Locators are lazy and re-resolve on every action, so they survive re-renders. ElementHandles point to one DOM node and can go stale. I always prefer locators."

### Q12. How do you reuse login across tests and roles?
"I log in once and save `storage_state` to a file. Then I create contexts from that state, usually in a session-scoped fixture. I keep one state file per role, such as admin, analyst and viewer, and I can open two contexts in one test to verify permissions."
```python
@pytest.fixture(scope="session")
def browser_context_args(browser_context_args):
    return {**browser_context_args, "storage_state": "auth/analyst.json"}
```

### Q13. How do you mock or intercept network calls?
"I use `page.route` to fulfill, modify or abort requests. I use it for unstable third-party services, error simulation and edge cases."
```python
page.route("**/api/prices", lambda r: r.fulfill(status=200, json={"AAPL": 190.5}))
```

### Q14. Iframes, popups, dialogs and downloads?
"For iframes I use `frame_locator`. For new tabs, `expect_popup`. For alerts, `page.on('dialog')`. For files, `expect_download`."
```python
page.frame_locator("#chart").get_by_text("Export").click()
with page.expect_download() as d: page.get_by_text("Download CSV").click()
```

### Q15. Dynamic content and slow loads?
"I rely on auto-wait and `expect`. If I need a specific backend call, I use `page.expect_response`. I avoid `time.sleep`, and I avoid `networkidle` as a default because it is fragile."

### Q16. Parallel and cross-browser runs?
"I run `pytest -n auto` with xdist and `--browser chromium --browser firefox`. Tests must have independent data. I run the full suite on Chromium and a critical-path suite on Firefox and WebKit."

### Q17. Does Playwright for Python have built-in retries?
"No. Config-level retries belong to the Node runner. In Python I use pytest-rerunfailures with `--reruns 2`, sparingly, and I report reruns so they do not hide flakiness."

### Q18. How do you debug Playwright tests?
"I use `PWDEBUG=1` for the Inspector, `--headed --slowmo`, `page.pause()`, and `--tracing retain-on-failure` with `playwright show-trace`. I also capture screenshots and video on failure, and use codegen to find locators."

### Q19. What does Trace Viewer show?
"An action timeline, DOM snapshots before and after each step, network requests, console logs and the source line. It is the best tool for failures that happen only in CI."

### Q20. API testing with Playwright?
"I use `playwright.request.new_context`. My favourite pattern is to set up data through the API and verify it in the UI."
```python
api = playwright.request.new_context(base_url=BASE,
        extra_http_headers={"Authorization": f"Bearer {TOKEN}"})
r = api.post("/orders", data={"symbol": "AAPL", "qty": 10})
assert r.ok and r.json()["status"] == "NEW"
```

### Q21. Playwright vs Selenium?
"Playwright has built-in auto-wait, fast isolated contexts, tracing, native network control and three engines. Selenium has a bigger ecosystem, Grid, and wider language and legacy-browser support. I choose based on project constraints."

### Q22. Dynamic tables, date pickers, custom dropdowns?
"For tables I filter rows with `locator('tr').filter(has_text=...)`. For custom dropdowns I click the trigger and then `get_by_role('option', name=...)`. For very large grids I validate against API or DB data instead of scraping every cell."

### Q23. Running Playwright in CI?
"I install dependencies with `playwright install --with-deps`, run headless, parallelise or shard, and upload traces, videos and reports as artifacts. The official Playwright Python Docker image avoids missing-dependency problems."

### Q24. Common Playwright mistakes?
"Sleeps, brittle XPath chains, shared state between tests, using `assert` instead of `expect`, logging in through the UI in every test, and relying on `networkidle`."

---

# PART 3 - pytest

### Q25. Why pytest over unittest?
"Plain asserts with rich failure output, powerful fixtures, parametrization, markers and a big plugin ecosystem such as xdist, rerunfailures, allure-pytest and pytest-playwright, all with less boilerplate."

### Q26. What is a fixture and what scopes exist?
"A fixture gives setup and teardown through dependency injection. Scopes are function, class, module, package and session. Code after `yield` is teardown."
```python
@pytest.fixture(scope="session")
def api_token():
    token = login()
    yield token
    logout(token)
```

### Q27. What is conftest.py?
"It holds shared fixtures and hooks. pytest finds it automatically through the directory tree, so no import is needed."

### Q28. How do you parametrize tests?
```python
@pytest.mark.parametrize("qty,valid", [(1, True), (0, False), (-5, False)])
def test_qty(qty, valid):
    assert validate_qty(qty) is valid
```
"This gives me data-driven tests with one function."

### Q29. Markers, skip and xfail?
"I tag tests with markers like `@pytest.mark.smoke` and select with `-m 'smoke and not slow'`. I register markers in `pytest.ini`. I use `skipif` with a reason, and `xfail` for known defects."

### Q30. Parallel runs and re-running failures?
"`pytest -n auto` for xdist and `--reruns 2` for rerunfailures. Tests must be independent for parallel runs."

### Q31. How do you handle test data?
"I use factories or builders, JSON or YAML files, and API or DB seeding through fixtures. I create unique data per test, clean up in teardown, and never share mutable data across parallel tests."

### Q32. Reports?
"`--junitxml` for CI, `--html` for a quick view, and allure-pytest for rich reports with steps, attachments and history."

### Q33. Asserting exceptions?
```python
with pytest.raises(ValueError, match="invalid qty"):
    place_order(qty=-1)
```

---

# PART 4 - Framework Design and Flaky Tests

### Q34. How would you design a framework for this analytics platform?
"I would use layers: pytest tests on top, then page objects and components, then API and service clients, then utilities like data factories, DB helpers and config. Fixtures handle auth and data setup and teardown through APIs, so UI tests cover only the UI. Config comes from env variables for dev, stage and prod. Markers split smoke, regression and critical path. I would use Allure for reporting, xdist for parallelism, and CI would publish traces and reports."
```text
project/
  conftest.py  pytest.ini
  pages/   api/   tests/(ui, api, e2e)   utils/(data, db, config)
```

### Q35. Which design patterns have you used?
"Page Object and Component Object for UI, Factory and Builder for test data, Strategy for swapping environments or browsers, Facade for simple workflow APIs, Singleton carefully for config, and dependency injection through pytest fixtures."

### Q36. Page Object Model: benefits and pitfalls?
"Benefits are reuse, readability and a single point of change. Pitfalls are god-classes, assertions buried in page objects and returning raw elements. I keep assertions in tests and use components for repeated widgets."

### Q37. How do you decide what to automate?
"I automate stable, repetitive, high-risk and data-driven flows: smoke, regression, API and critical end-to-end. I keep exploratory, one-off and fast-changing areas manual. I follow the test pyramid: more API and unit tests, fewer UI tests."

### Q38. How do you reduce flaky tests?
"First I classify the cause: timing, shared data, environment, order dependency or a real intermittent bug. Then I fix it with web-first assertions instead of sleeps, role and test-id locators, isolated data per test, mocks for unstable services, and removing order dependencies. I track flake rate in CI, quarantine the worst offenders with a ticket and an owner, and use retries only as a diagnostic."

### Q39. Test passes locally but fails in CI?
"I download the trace and logs, compare viewport, browser version and resources, check timing and data dependencies, rerun in the same Docker image, and look for parallelism and ordering issues."

### Q40. Root-cause analysis on a defect?
"I reproduce it, narrow the scope between UI, API, data and environment, check logs and traces, compare with the last good build, confirm the cause with the developer, document it, and add a regression test."

### Q41. How do you reduce suite runtime?
"I parallelise, set up data through the API instead of the UI, reuse auth state, move checks down to API level, shard in CI, remove duplicates, and run only the relevant subset on PRs."

### Q42. What if 20% of tests are flaky?
"I triage by failure frequency and fix the top causes first, usually data and waits. I quarantine the rest, add a flake dashboard, and require new tests to pass N consecutive runs before merging."

### Q43. Multiple environments?
"Env variables or config files per environment, selected with a CLI option like `--env stage`. Secrets come from a secrets manager. Prod gets read-only smoke tests only."

### Q44. Keeping a framework maintainable?
"Code reviews, linting with ruff or flake8, type hints, small focused page objects, naming conventions, documentation, deleting dead tests, and a regular flaky-test review."

---

# PART 5 - API Testing

### Q45. How do you automate API tests in Python?
"I use pytest with requests, or Playwright's request context, a session-scoped auth fixture and reusable clients."
```python
def test_create_order(api_base, token):
    r = requests.post(f"{api_base}/orders",
                      json={"symbol": "AAPL", "qty": 100, "side": "BUY"},
                      headers={"Authorization": f"Bearer {token}"}, timeout=10)
    assert r.status_code == 201
    assert r.json()["status"] == "NEW"
```

### Q46. What do you validate in a response?
"Status code, response time, headers, body fields and types, schema with jsonschema, business rules, and side effects in the database."

### Q47. Negative cases?
"Missing or invalid token (401), no permission (403), bad payload (400 or 422), not found (404), duplicate (409), large payloads, wrong content type and rate limiting (429)."

### Q48. Common HTTP status codes?
"200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests, 500 Server Error, 503 Service Unavailable."

### Q49. Path vs query parameters? PUT vs PATCH vs POST?
"Path parameters identify a resource, like `/orders/123`. Query parameters filter it, like `/orders?status=NEW&limit=10`. POST creates, PUT replaces and is idempotent, PATCH updates part of a resource."

### Q50. Serialization and deserialization?
"Serialization turns an object into a transferable format like JSON with `json.dumps`. Deserialization rebuilds the object with `json.loads`."

### Q51. Postman vs code-based API tests?
"Postman is quick for exploration and sharing. Code-based tests with pytest and requests are version-controlled, reviewable, data-driven and fit CI better. I explore in Postman and regress in pytest."

### Q52. Authentication and contract testing?
"I get the token in a session fixture for OAuth or JWT, refresh it when it expires, and never hardcode credentials. For contracts I validate responses against OpenAPI or JSON schema, and consumer-driven contracts with Pact catch breaking changes between services."

---

# PART 6 - AI Tools in Testing

### Q53. How do you use Claude, Copilot or Microsoft Copilot?
"I use Claude to turn acceptance criteria into test matrices and edge cases, and for first-pass failure triage from logs. I use GitHub Copilot for boilerplate like page objects, parametrized tests and utilities. I always review the output, verify assertions against requirements, run for stability, and keep sensitive client data out of prompts."

### Q54. Give a concrete example.
"For a new analytics screen I gave Claude the acceptance criteria and got about 40 candidate scenarios. I kept 28 after review, added 6 it missed, and automated the critical ones. It saved me about X hours."

### Q55. Risks of AI-generated tests?
"Hallucinated locators or APIs, assertions that always pass, tests coupled to implementation instead of requirements, data and IP leakage, and false confidence. I mitigate with human review, approved tools only, and checking that a test fails when I deliberately break the behaviour."

### Q56. How do you validate AI-generated test cases?
"I trace each one to a requirement, check the assertions are meaningful, make the test fail on purpose by breaking behaviour, and run it repeatedly for stability."

### Q57. What is agentic AI in test automation?
"Agents that plan and execute multi-step tasks, such as exploring an app, generating tests, running them and triaging failures, with access to the browser, repo and CI. They need guardrails: scoped permissions, review before merge and audit logs."

### Q58. Self-healing tests: would you use them?
"They adjust locators automatically when the UI changes. They reduce maintenance but can hide real regressions, so I would use them with logging and review, not silently."

### Q59. How would you test an AI / ML system?
"I validate data quality, check accuracy metrics against thresholds on a held-out set, test edge cases and bias, monitor consistency and drift, and keep a golden dataset for regression. Outputs are probabilistic, so I assert on ranges or properties instead of exact values."

---

# PART 7 - SQL and Data Testing

### Q60. How do you test data transformations?
"I define rules from the spec and build a golden dataset with edge cases: nulls, duplicates, late-arriving records, currency conversion. I assert counts, field mappings and aggregates at each stage, and reconcile source to target with SQL. I automate the checks in CI so every pipeline change reruns them."

### Q61. Very large datasets?
"I reconcile with SQL aggregates, counts and checksums instead of row-by-row checks, sample strategically with risk-based edge cases, and test performance at realistic volume."

### Q62. SQL: find duplicates.
```sql
SELECT trade_id, COUNT(*) FROM trades GROUP BY trade_id HAVING COUNT(*) > 1;
```
"I group by the key and filter groups with HAVING."

### Q63. SQL: records in source but missing in target.
```sql
SELECT s.id FROM source s LEFT JOIN target t ON s.id = t.id WHERE t.id IS NULL;
-- or: SELECT id FROM source EXCEPT SELECT id FROM target;
```

### Q64. SQL: second-highest value.
```sql
SELECT MAX(price) FROM trades WHERE price < (SELECT MAX(price) FROM trades);
```

### Q65. SQL: total notional per symbol above a threshold.
```sql
SELECT symbol, SUM(qty*price) AS notional
FROM trades GROUP BY symbol
HAVING SUM(qty*price) > 1000000 ORDER BY notional DESC;
```

### Q66. JOIN types? WHERE vs HAVING?
"INNER returns matches only. LEFT returns all left rows plus matches. RIGHT is the mirror. FULL returns all rows from both. CROSS is the Cartesian product. WHERE filters rows before grouping; HAVING filters groups after aggregation."

### Q67. Validating numeric precision?
"I use Decimal or exact comparison with a defined tolerance, never float equality. I check rounding rules, currency scale and aggregate totals."

---

# PART 8 - CI/CD and AWS

### Q68. Describe a CI/CD pipeline for tests.
"On a PR: lint, unit tests and a parallel smoke suite. Nightly: full regression sharded across workers. After each run I publish Allure reports, traces and videos as artifacts and notify the team on failure. Merge is gated on smoke passing."

### Q69. Running across dev / stage / prod?
"Config by env variable and CLI option, per-environment credentials from a secrets manager, and tags so prod runs only safe, read-only smoke tests."

### Q70. Sample GitHub Actions job?
```yaml
- uses: actions/setup-python@v5
  with: { python-version: "3.12" }
- run: pip install -r requirements.txt && playwright install --with-deps
- run: pytest -n auto --tracing retain-on-failure --alluredir=allure-results
- uses: actions/upload-artifact@v4
  if: always()
  with: { name: results, path: allure-results }
```
"It installs, runs in parallel with traces, and always uploads artifacts."

### Q71. AWS touchpoints for QA?
"CodeBuild and CodePipeline for runs, S3 for artifacts and test data, IAM roles for access, Secrets Manager or Parameter Store for credentials, CloudWatch for logs, and EC2 or ECS for runners."

### Q72. Secrets, and why Docker?
"I never store secrets in the repo. I use CI secrets or a secrets manager and inject them as env variables at runtime. A Docker image with fixed Python, dependencies and browsers makes CI and local runs identical."

### Q73. What is a quality gate?
"A pipeline rule that blocks promotion when a threshold fails, such as smoke pass rate, critical defects or coverage of critical flows."

---

# PART 9 - Java, Selenium and BDD

### Q74. Implicit vs explicit vs fluent wait?
"Implicit is a global polling time when finding elements. Explicit waits for a specific condition with ExpectedConditions. Fluent is explicit with a custom polling interval and ignored exceptions. I never mix implicit and explicit waits."

### Q75. StaleElementReferenceException?
"I re-locate the element, use explicit waits, avoid caching elements, and make page-object methods find elements fresh each time."

### Q76. What is Cucumber / BDD?
"BDD writes scenarios in Gherkin, Given, When, Then, and step definitions execute them. It improves shared understanding between product, dev and QA, but adds overhead if overused for purely technical tests."

### Q77. Scenario vs Scenario Outline? Hooks and tags?
"A Scenario runs once. A Scenario Outline with an Examples table runs once per data row. Hooks like `@Before` and `@After` handle setup and teardown. Tags like `@smoke` select which scenarios run."

### Q78. Java concepts?
"OOP, collections like ArrayList, HashMap and HashSet, exceptions, String vs StringBuilder, `==` vs `equals`, interface vs abstract class, and Java 8 streams."
```java
List<Integer> evens = nums.stream().filter(n -> n % 2 == 0).collect(Collectors.toList());
```

### Q79. TestNG / JUnit vs pytest?
"They play similar roles. pytest has simpler fixtures and plain asserts. TestNG has built-in groups, data providers and XML suites."

### Q80. Migrating Selenium to Playwright?
"I would start with the flakiest or highest-value flows, run both in parallel for a while, port page objects, compare results, then retire the old tests gradually."

---

# PART 10 - Manual Testing, Strategy and Agile

### Q81. Test design techniques?
"Equivalence partitioning, boundary-value analysis, decision tables, state transition, pairwise testing and error guessing."

### Q82. Severity vs priority?
"Severity is the technical impact of a defect. Priority is how urgently it should be fixed. A homepage typo can be low severity but high priority."

### Q83. A good bug report?
"Title, environment and build, steps to reproduce, expected versus actual, evidence such as screenshot, logs, trace or API response, and severity."

### Q84. Which KPIs would you define?
"Defect escape rate, automation coverage of critical flows, suite runtime, flake rate, defect leakage per release, and mean time to detect and triage."

### Q85. Regression scope?
"Risk-based: critical paths, recently changed code, defect-prone modules and customer impact, plus a fast smoke subset for every build."

### Q86. QA role in Agile? Test plan vs strategy?
"I join refinement to challenge acceptance criteria, plan tests early, automate within the sprint, run regression and share quality metrics. A strategy is the high-level, long-lived approach. A plan is release- or project-specific, covering scope, schedule and resources."

---

# PART 11 - Performance and Capital Markets

### Q87. Performance test types and JMeter?
"Load is expected volume, stress is beyond capacity, spike is a sudden surge, soak is long duration. In JMeter I use Thread Groups for users, ramp-up and loops, plus Samplers, Config elements, Assertions, Timers and Listeners, and I run non-GUI mode in CI."

### Q88. Which metrics matter?
"Response time including average, p95 and p99, throughput, error rate, and resource usage like CPU, memory and DB connections."

### Q89. Sudden traffic increase?
"I find the bottleneck with load tests first. Then I apply caching, horizontal autoscaling, load balancing, queues, rate limiting and database optimisation, and validate again with spike and soak tests."

### Q90. Explain the trade lifecycle.
"An order is placed, validated, executed on a venue, confirmed, cleared and settled, then reported and reconciled."

### Q91. What would you test in a financial analytics platform?
"Calculation accuracy and precision, data lineage and reconciliation, timezone and holiday handling, permissions and audit trails, performance on large volumes, and report and export correctness."

### Q92. Why does precision matter? What if you lack domain experience?
"Floating-point errors cause cent-level mismatches that grow in aggregate, so I use Decimal and defined rounding rules. On domain experience: I have studied the trade lifecycle and key terms, I ask product and data teams early, and my data-validation skills transfer directly."

---

# PART 12 - Python Core

### Q93. Mutable vs immutable?
"Mutable: list, dict, set. Immutable: int, str, tuple, frozenset. A classic trap is a mutable default argument, because that object is shared across calls."
```python
def add(item, bucket=None):
    if bucket is None: bucket = []
    bucket.append(item); return bucket
```

### Q94. List vs tuple vs set vs dict? `==` vs `is`?
"List is ordered and mutable. Tuple is ordered and immutable. Set is unordered, unique and fast for membership. Dict is key-value and insertion-ordered since 3.7. `==` compares values; `is` compares identity, and I use it only for None."

### Q95. Shallow vs deep copy?
"A shallow copy copies the outer container but shares nested objects. `deepcopy` copies everything recursively."

### Q96. *args and **kwargs?
"`*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dict. They are handy for flexible helpers and wrappers."

### Q97. Decorator with a QA example?
"A decorator wraps a function to add behaviour. Here is a retry."
```python
import functools, time
def retry(times=3, delay=1):
    def deco(fn):
        @functools.wraps(fn)
        def wrapper(*a, **kw):
            for i in range(times):
                try: return fn(*a, **kw)
                except AssertionError:
                    if i == times - 1: raise
                    time.sleep(delay)
        return wrapper
    return deco
```

### Q98. Generator and context manager?
"A generator uses `yield` to produce values lazily, which saves memory on large files. A context manager guarantees setup and cleanup with `with`."
```python
def read_rows(path):
    with open(path) as f:
        for line in f:
            yield line.strip().split(",")
```

### Q99. What is the GIL?
"In CPython only one thread runs Python bytecode at a time, which limits CPU-bound threading. For CPU work I use multiprocessing. Test parallelism usually comes from processes, like pytest-xdist."

### Q100. OOP pillars and MRO?
"Encapsulation, inheritance, polymorphism through duck typing, and abstraction with `abc.ABC`. MRO is the order Python searches base classes, using C3 linearisation, visible through `__mro__`."

### Q101. Exception handling best practice?
"Catch specific exceptions, never a bare `except`. Use `else` for the no-exception path, `finally` for cleanup, and `raise ... from e` to keep the cause."

### Q102. staticmethod vs classmethod vs instance method? `__str__` vs `__repr__`?
"An instance method gets `self`. A classmethod gets `cls` and is good for alternative constructors. A staticmethod gets neither. `__str__` is the readable string; `__repr__` is the unambiguous developer string."

### Q103. Dataclasses, type hints, logging?
"Dataclasses give typed test-data objects without boilerplate. Type hints improve readability and enable mypy, though they are not enforced at runtime. I use the logging module with levels instead of print, and attach logs to reports."
```python
@dataclass
class Order:
    symbol: str
    qty: int
    side: str = "BUY"
```

### Q104. Output-prediction traps.
```python
print(type([]) is list)                  # True
print([1,2,3][::-1])                     # [3, 2, 1]
print(0.1 + 0.2 == 0.3)                  # False
a = [1,2]; b = a; b.append(3); print(a)  # [1, 2, 3]
print("abc"[5:])                         # ''  (no IndexError)
```

---

# PART 13 - Live Coding (think aloud, ask about edge cases first)

### Q105. Pairs that sum to a target.
"I will use a set of seen values so it is O(n)."
```python
def pairs(nums, target):
    seen, out = set(), []
    for n in nums:
        if target - n in seen:
            out.append((target - n, n))
        seen.add(n)
    return out
# pairs([1,9,3,7,5,5], 10) -> [(1, 9), (3, 7), (5, 5)]
```

### Q106. Longest run of consecutive positive numbers.
"I track the current run and the best run, and reset on a non-positive number."
```python
def longest_positive_run(nums):
    best, cur = [], []
    for n in nums:
        if n > 0:
            cur.append(n)
            if len(cur) > len(best): best = cur[:]
        else:
            cur = []
    return best
```

### Q107. Duplicates, word frequency, first unique character.
```python
from collections import Counter
def duplicates(items): return [k for k, v in Counter(items).items() if v > 1]
def word_freq(text):   return Counter(text.lower().split())
def first_unique(s):
    c = Counter(s)
    return next((ch for ch in s if c[ch] == 1), None)
```

### Q108. Palindrome and anagram.
"I normalise first, then compare."
```python
def is_palindrome(s):
    s = "".join(c.lower() for c in s if c.isalnum())
    return s == s[::-1]
def is_anagram(a, b):
    return Counter(a.replace(" ", "").lower()) == Counter(b.replace(" ", "").lower())
```

### Q109. OTP-style string like A23DR.
"For real security tokens I would use the `secrets` module, but for test data this is fine."
```python
import random, string
def make_otp():
    return (random.choice(string.ascii_uppercase)
            + "".join(random.choices(string.digits, k=2))
            + "".join(random.choices(string.ascii_uppercase, k=2)))
```

### Q110. Card type from the sum of the first four digits.
```python
def card_type(card: str) -> str:
    assert len(card) == 16 and card.isdigit()
    return "visa" if sum(int(d) for d in card[:4]) == 16 else "master"
```

### Q111. Compare two dicts and flatten a nested list.
```python
def diff(exp, act):
    return {"missing": exp.keys() - act.keys(),
            "extra": act.keys() - exp.keys(),
            "changed": {k: (exp[k], act[k]) for k in exp.keys() & act.keys() if exp[k] != act[k]}}
def flatten(lst):
    for x in lst:
        if isinstance(x, list): yield from flatten(x)
        else: yield x
```

### Q112. Full Playwright test with Page Object and API setup.
"The API fixture creates and cleans up data, the page object finds elements, and the test uses `expect`."
```python
class OrdersPage:
    def __init__(self, page): self.page = page
    def open(self, base): self.page.goto(f"{base}/orders")
    def status_of(self, oid):
        return self.page.locator("tr").filter(has_text=oid).get_by_test_id("status")

@pytest.fixture
def order_id(api_base, token):
    h = {"Authorization": f"Bearer {token}"}
    oid = requests.post(f"{api_base}/orders", json={"symbol": "AAPL", "qty": 10},
                        headers=h, timeout=10).json()["id"]
    yield oid
    requests.delete(f"{api_base}/orders/{oid}", headers=h, timeout=10)

def test_new_order_visible(logged_in_page, base_url, order_id):
    orders = OrdersPage(logged_in_page)
    orders.open(base_url)
    expect(orders.status_of(str(order_id))).to_have_text("NEW")
```

---

# PART 14 - Behavioral and HR

### Q113. Disagreement with a developer about a defect?
"I stay evidence-based. I share steps, logs and the requirement, and ask for their view. If we still disagree, I involve the product owner. In the end we [outcome], and we added [process improvement]."

### Q114. A bug escaped to production?
"I owned it. The impact was [X]. The root cause was [Y]. We fixed it in [time], and I added a regression test and a checklist item so it cannot happen again."

### Q115. Tight deadlines?
"I prioritise by risk, communicate trade-offs early, automate the critical path, and defer low-risk items with stakeholder agreement."

### Q116. Mentoring juniors?
"I pair with them on tests, explain in code reviews, share checklists and templates, and give small tasks with real ownership."

### Q117. Working with Product, Developers and Data / AI engineers?
"I clarify expected behaviour early through refinement and example mapping, share test results openly, and agree on acceptance criteria and data before development finishes."

### Q118. Why are you leaving? Why Luxoft?
"I want broader ownership of quality strategy and exposure to capital-markets analytics. Luxoft offers engineering depth, complex client projects, and the chance to combine automation with AI-assisted testing."

### Q119. Where do you see yourself in 3-5 years?
"Leading quality engineering for a platform: framework architecture, mentoring, and AI-assisted testing strategy."

### Q120. Notice period, expected CTC, location?
"My notice period is [X]. My expected CTC is [range], based on market research and my experience. I am comfortable with Noida and can discuss hybrid or remote expectations."

---

# Questions to Ask the Interviewer
- What does the current automation framework look like, and what is the flake rate?
- How does the team use AI tools today, and is there an approved-tools policy?
- What are the release cadence and environment setup?
- How much of the role is manual testing versus automation?
- What does success look like in the first 90 days?

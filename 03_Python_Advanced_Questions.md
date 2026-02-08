# Python Interview Prep — SDET (8+ Years) — PART 3: ADVANCED
## Covers: Noida | Hyderabad | Gurugram | Delhi | Pune | Chennai

---

# SECTION C — ADVANCED LEVEL

---

## C1. Metaclasses

### Q46. What are metaclasses?
A metaclass is a class whose instances are classes. Default metaclass is `type`.

```python
print(type(int))   # <class 'type'>
print(type(str))   # <class 'type'>

# Creating class dynamically
MyClass = type('MyClass', (object,), {'x': 10, 'greet': lambda self: "Hello"})

# Custom metaclass — Singleton example
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = "connected"

db1 = Database()
db2 = Database()
print(db1 is db2)  # True
```

### Q47. `__new__` vs `__init__`?
- `__new__`: **Creates** and returns a new instance (static method, called before `__init__`)
- `__init__`: **Initializes** the instance (called after `__new__`)

```python
class Singleton:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    def __init__(self):
        self.value = 42

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

---

## C2. GIL (Global Interpreter Lock)

### Q48. What is the GIL?
The **Global Interpreter Lock** is a mutex in CPython allowing only one thread to execute Python bytecode at a time.

**Impact:**
- CPU-bound multithreaded programs don't benefit from multiple cores
- I/O-bound programs are fine (GIL released during I/O)

**Solutions for CPU-bound:**
```python
from multiprocessing import Pool

def cpu_heavy(n):
    return sum(i * i for i in range(n))

with Pool(4) as pool:
    results = pool.map(cpu_heavy, [10**6] * 4)
```

**I/O-bound (threading works):**
```python
from concurrent.futures import ThreadPoolExecutor
import requests

def fetch(url):
    return requests.get(url).status_code

with ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(fetch, ["https://httpbin.org/get"] * 10))
```

---

## C3. Concurrency & Async

### Q49. Threading vs Multiprocessing vs Asyncio?
| Feature | Threading | Multiprocessing | Asyncio |
|---------|-----------|-----------------|---------|
| Type | Preemptive | Parallelism | Cooperative |
| GIL | Affected | Not affected | Affected |
| Best for | I/O-bound | CPU-bound | Many I/O connections |
| Memory | Shared | Separate | Shared |
| Overhead | Low | High | Very low |

### Q50. What are coroutines?
**Coroutines** are special functions defined with `async def` that can be **paused** and **resumed** using `await`. Unlike threads (preemptive multitasking), coroutines use **cooperative multitasking** — they voluntarily yield control at `await` points, allowing other coroutines to run.

**Key concepts:**
- `async def` — declares a coroutine function
- `await` — pauses execution until the awaited task completes
- `asyncio.gather()` — runs multiple coroutines **concurrently** (not in parallel)
- `asyncio.run()` — entry point to start the event loop

**SDET use:** Running multiple API health checks concurrently, parallel page load verification, bulk data validation.

```python
import asyncio

async def fetch_data(delay, name):
    print(f"Starting {name}")
    await asyncio.sleep(delay)
    print(f"Finished {name}")
    return f"{name}: data"

async def main():
    results = await asyncio.gather(
        fetch_data(2, "API-1"),
        fetch_data(1, "API-2"),
        fetch_data(3, "API-3"),
    )
    print(results)

asyncio.run(main())
# Total time ≈ 3s (not 6s)
```

### Q51. Async API testing example?
Using `aiohttp` (async HTTP client) with `asyncio` to make **concurrent API calls**. This is significantly faster than sequential `requests` calls when testing multiple endpoints. For 100 API calls, async can complete in the time of 1 call vs 100x time with synchronous code.

```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.json()

async def main():
    urls = ["https://httpbin.org/get"] * 10
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
    return results

asyncio.run(main())
```

---

## C4. Descriptors & Slots

### Q52. What are descriptors?
**Descriptors** are objects that define any of `__get__()`, `__set__()`, or `__delete__()` methods to **customize attribute access** on another class. When a descriptor is assigned as a class variable, Python calls its descriptor methods instead of normal attribute access. They are the mechanism behind `@property`, `@classmethod`, `@staticmethod`, and `__slots__`.

**SDET use:** Creating reusable validators for test configuration (e.g., ensuring timeout values are within range, URLs are valid format).

```python
class Validator:
    def __init__(self, min_val, max_val):
        self.min_val = min_val
        self.max_val = max_val

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        return getattr(obj, f"_{self.name}", None)

    def __set__(self, obj, value):
        if not self.min_val <= value <= self.max_val:
            raise ValueError(f"{self.name} must be {self.min_val}-{self.max_val}")
        setattr(obj, f"_{self.name}", value)

class TestConfig:
    timeout = Validator(1, 300)
    retries = Validator(0, 10)

config = TestConfig()
config.timeout = 30   # OK
config.retries = 3    # OK
config.timeout = 500  # ValueError!
```

**Descriptor types:**
- **Data descriptor:** `__get__` + `__set__`/`__delete__`
- **Non-data descriptor:** only `__get__`
- **Priority:** Data descriptor > Instance `__dict__` > Non-data descriptor

### Q53. What are `__slots__`?
`__slots__` is a class variable that **explicitly declares** which instance attributes are allowed. By defining `__slots__`, Python skips creating a `__dict__` for each instance, resulting in **significant memory savings** (up to 40%) and slightly **faster attribute access**. However, you lose the ability to dynamically add new attributes.

**When to use:** Classes with many instances (e.g., test data objects, page element wrappers), performance-critical code, and when you want to prevent accidental attribute creation.

```python
class WithSlots:
    __slots__ = ['x', 'y']
    def __init__(self, x, y):
        self.x = x
        self.y = y

b = WithSlots(1, 2)
b.z = 3  # AttributeError!
# No __dict__ → less memory per instance
```

---

## C5. Design Patterns for SDET

### Q54. Singleton Pattern
The **Singleton pattern** ensures a class has **only one instance** throughout the application's lifecycle. It provides a global access point to that instance. Implemented by overriding `__new__()` to check if an instance already exists.

**SDET use:** Managing a single WebDriver instance, database connection pool, configuration manager, or logger across all test classes.

```python
class WebDriverManager:
    _instance = None
    _driver = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def get_driver(self):
        if self._driver is None:
            self._driver = "ChromeDriver()"
        return self._driver
```

### Q55. Factory Pattern
The **Factory pattern** creates objects without exposing the creation logic to the client. It delegates instantiation to a factory method that returns the appropriate object based on input parameters.

**SDET use:** Creating browser drivers (Chrome/Firefox/Safari) based on config, generating different types of test data, creating API clients for different environments.

```python
class BrowserFactory:
    @staticmethod
    def create_driver(browser_type):
        drivers = {
            "chrome": lambda: "ChromeDriver()",
            "firefox": lambda: "FirefoxDriver()",
            "safari": lambda: "SafariDriver()",
        }
        creator = drivers.get(browser_type.lower())
        if not creator:
            raise ValueError(f"Unsupported: {browser_type}")
        return creator()
```

### Q56. Page Object Model (POM)
**POM** is the most important design pattern in test automation. Each web page is represented by a **separate class** that contains the page's **locators** (as class variables) and **interaction methods**. Benefits:
- **Single point of maintenance** — if a locator changes, update only one place
- **Readable tests** — `loginPage.login("admin", "pass")` instead of raw Selenium commands
- **Reusability** — page methods can be reused across multiple test cases
- **Separation of concerns** — tests focus on "what to test", page objects handle "how to interact"

```python
class BasePage:
    def __init__(self, driver):
        self.driver = driver
    def find_element(self, locator):
        return self.driver.find_element(*locator)
    def click(self, locator):
        self.find_element(locator).click()
    def enter_text(self, locator, text):
        el = self.find_element(locator)
        el.clear()
        el.send_keys(text)

class LoginPage(BasePage):
    USERNAME = ("id", "username")
    PASSWORD = ("id", "password")
    LOGIN_BTN = ("id", "login-btn")

    def login(self, username, password):
        self.enter_text(self.USERNAME, username)
        self.enter_text(self.PASSWORD, password)
        self.click(self.LOGIN_BTN)
        return DashboardPage(self.driver)
```

### Q57. Strategy Pattern
The **Strategy pattern** defines a family of algorithms (strategies), encapsulates each one, and makes them **interchangeable**. The client can switch between strategies at runtime without changing the code that uses them.

**SDET use:** Different wait strategies (explicit/fluent/custom), different locator strategies (CSS/XPath/ID), different reporting strategies (Allure/Extent/HTML), or different data sources (CSV/JSON/DB) for data-driven tests.

```python
from abc import ABC, abstractmethod

class WaitStrategy(ABC):
    @abstractmethod
    def wait_for(self, driver, locator, timeout): pass

class ExplicitWait(WaitStrategy):
    def wait_for(self, driver, locator, timeout=10):
        pass  # WebDriverWait implementation

class FluentWait(WaitStrategy):
    def wait_for(self, driver, locator, timeout=10):
        pass  # Custom polling

class ElementFinder:
    def __init__(self, strategy: WaitStrategy):
        self.strategy = strategy
    def find(self, driver, locator, timeout=10):
        return self.strategy.wait_for(driver, locator, timeout)
```

### Q58. Builder Pattern (for test data)
The **Builder pattern** constructs complex objects **step by step** using method chaining. Instead of a constructor with many parameters, you call readable builder methods. The `build()` method returns the final object.

**SDET use:** Creating test data with many optional fields (user profiles, API payloads, test configurations). Makes test data creation readable and avoids telescoping constructors.

```python
class UserBuilder:
    def __init__(self):
        self._name = "Default User"
        self._email = "default@test.com"
        self._role = "user"
        self._active = True

    def with_name(self, name):
        self._name = name
        return self

    def with_email(self, email):
        self._email = email
        return self

    def with_role(self, role):
        self._role = role
        return self

    def inactive(self):
        self._active = False
        return self

    def build(self):
        return {
            "name": self._name,
            "email": self._email,
            "role": self._role,
            "active": self._active,
        }

# Usage
admin = UserBuilder().with_name("Admin").with_role("admin").build()
inactive = UserBuilder().with_name("Old User").inactive().build()
```

---

## C6. Dataclasses & Typing

### Q59. Dataclasses?
**Dataclasses** (Python 3.7+) automatically generate `__init__`, `__repr__`, `__eq__`, and other boilerplate methods based on class annotations. They're ideal for **data containers** — objects that primarily hold data with minimal behavior.

**Key features:**
- `@dataclass` — auto-generates `__init__`, `__repr__`, `__eq__`
- `field(default_factory=list)` — mutable default values
- `@dataclass(frozen=True)` — creates immutable instances (hashable, usable as dict keys)
- `asdict(obj)` — converts to dictionary (useful for JSON serialization)

**SDET use:** Test results, test configurations, API response models, test data objects.

```python
from dataclasses import dataclass, field, asdict
from typing import List, Optional

@dataclass
class TestResult:
    test_name: str
    status: str
    duration: float
    error_message: Optional[str] = None
    tags: List[str] = field(default_factory=list)

    @property
    def passed(self):
        return self.status == "PASS"

result = TestResult("test_login", "PASS", 2.5, tags=["smoke"])
print(asdict(result))
print(result.passed)  # True

# Frozen (immutable)
@dataclass(frozen=True)
class Config:
    base_url: str
    timeout: int = 30
```

### Q60. Type hinting?
**Type hints** (PEP 484, Python 3.5+) add optional type annotations to function parameters, return values, and variables. They don't enforce types at runtime but enable **IDE autocompletion**, **static analysis** (mypy), and serve as **living documentation**.

**Key types:** `List[str]`, `Dict[str, Any]`, `Optional[int]` (can be None), `Union[str, int]`, `Callable[[args], return]`, `Tuple[int, str]`, `TypeVar` (generics), `Protocol` (structural typing).

**SDET benefit:** Catch type errors before running tests, better code readability, easier framework maintenance.

```python
from typing import List, Dict, Optional, Union, Callable, Any, TypeVar, Protocol

def run_tests(
    test_names: List[str],
    config: Dict[str, Any],
    timeout: Optional[int] = None,
    callback: Callable[[str, bool], None] = None
) -> Dict[str, bool]:
    results: Dict[str, bool] = {}
    for name in test_names:
        results[name] = True
        if callback:
            callback(name, True)
    return results

# TypeVar for generics
T = TypeVar('T')
def first_element(items: List[T]) -> Optional[T]:
    return items[0] if items else None

# Protocol (structural subtyping, Python 3.8+)
class Clickable(Protocol):
    def click(self) -> None: ...

def interact(element: Clickable) -> None:
    element.click()
```

---

## C7. Advanced Concepts

### Q61. `__getattr__` vs `__getattribute__` vs `__setattr__`?
These magic methods control **attribute access** on objects:
- **`__getattribute__(self, name)`** — called on **every** attribute access. Use with caution (easy to cause infinite recursion). Always call `super().__getattribute__()` inside.
- **`__getattr__(self, name)`** — called **only when** the attribute is **not found** through normal mechanisms. Good for default values or proxying.
- **`__setattr__(self, name, value)`** — called on **every** attribute assignment. Use `super().__setattr__()` to actually set the value.

**SDET use:** Logging attribute access for debugging, creating proxy objects for API clients, implementing lazy-loading page elements.

```python
class LoggingObject:
    def __getattribute__(self, name):
        # Called for EVERY attribute access
        print(f"Accessing: {name}")
        return super().__getattribute__(name)

    def __getattr__(self, name):
        # Called ONLY when attribute NOT found
        return f"{name} not found"

    def __setattr__(self, name, value):
        # Called for EVERY attribute assignment
        print(f"Setting {name} = {value}")
        super().__setattr__(name, value)
```

### Q62. Monkey patching?
**Monkey patching** is dynamically modifying a class, module, or object **at runtime** by replacing its attributes or methods. While generally considered bad practice in production code, it's extremely useful for **testing** — replacing real implementations with mocks or stubs without modifying source code.

**SDET use:** Mocking API calls, replacing database connections with test doubles, simulating error conditions. Python's `unittest.mock.patch` is the preferred way (structured monkey patching with automatic cleanup).

```python
class APIClient:
    def get(self, url):
        import requests
        return requests.get(url)

# Patch for testing
def mock_get(self, url):
    return {"status": 200, "data": "mocked"}

APIClient.get = mock_get
client = APIClient()
print(client.get("https://api.example.com"))  # mocked
```

### Q63. `collections` module?
The `collections` module provides **specialized container datatypes** that extend Python's built-in containers (dict, list, tuple, set):

- **`defaultdict(type)`** — dict that auto-creates missing keys with a default value
- **`Counter(iterable)`** — counts occurrences, has `most_common(n)` method
- **`namedtuple(name, fields)`** — lightweight immutable object with named fields
- **`deque`** — double-ended queue with O(1) append/pop on both ends
- **`ChainMap`** — combines multiple dicts into a single view (first match wins)
- **`OrderedDict`** — dict that remembers insertion order (redundant since Python 3.7+)

```python
from collections import defaultdict, Counter, namedtuple, deque, ChainMap, OrderedDict

# defaultdict
word_count = defaultdict(int)
for w in "hello world hello".split():
    word_count[w] += 1  # {'hello': 2, 'world': 1}

# Counter
c = Counter("abracadabra")
c.most_common(3)  # [('a', 5), ('b', 2), ('r', 2)]

# namedtuple
TC = namedtuple("TestCase", ["id", "name", "priority"])
tc = TC(1, "Login Test", "High")
print(tc.name)  # Login Test

# deque (O(1) both ends)
d = deque([1, 2, 3], maxlen=5)
d.appendleft(0)
d.append(4)

# ChainMap
defaults = {"timeout": 30, "retries": 3}
overrides = {"timeout": 60}
config = ChainMap(overrides, defaults)
print(config["timeout"])  # 60
print(config["retries"])  # 3
```

### Q64. `itertools` module?
The `itertools` module provides **memory-efficient iterator building blocks** for working with sequences. All functions return **lazy iterators**, making them perfect for large datasets.

**Key functions for SDET:**
- `chain()` — combine multiple iterables into one
- `product()` — cartesian product (generate all browser + environment combinations)
- `combinations()` / `permutations()` — test input combinations
- `groupby()` — group test results by status/category
- `islice()` — slice an iterator without loading all data
- `accumulate()` — running totals (useful for performance trend analysis)

```python
import itertools

# chain
list(itertools.chain([1,2], [3,4]))  # [1,2,3,4]

# product (cartesian)
list(itertools.product(["chrome","firefox"], ["staging","prod"]))
# [('chrome','staging'), ('chrome','prod'), ('firefox','staging'), ('firefox','prod')]

# combinations & permutations
list(itertools.combinations([1,2,3], 2))  # [(1,2), (1,3), (2,3)]
list(itertools.permutations([1,2,3], 2))  # 6 results

# groupby
data = sorted([("pass","t1"), ("fail","t2"), ("pass","t3")], key=lambda x: x[0])
for status, tests in itertools.groupby(data, key=lambda x: x[0]):
    print(status, list(tests))

# islice
list(itertools.islice(range(100), 5, 10))  # [5,6,7,8,9]

# accumulate
list(itertools.accumulate([1,2,3,4]))  # [1,3,6,10]
```

### Q65. `functools` module?
The `functools` module provides **higher-order functions** that act on or return other functions:

- **`lru_cache(maxsize)`** — memoization decorator that caches function results (Least Recently Used eviction)
- **`partial(func, *args)`** — creates a new function with some arguments pre-filled (currying)
- **`reduce(func, iterable)`** — cumulatively applies function to pairs
- **`wraps(func)`** — preserves metadata when creating decorators
- **`total_ordering`** — auto-generates comparison methods from `__eq__` + one other

**SDET use:** `lru_cache` for expensive API lookups, `partial` for creating pre-configured HTTP clients, `wraps` in custom test decorators.

```python
from functools import lru_cache, partial, reduce, wraps, total_ordering

# lru_cache — memoization
@lru_cache(maxsize=128)
def expensive_call(n):
    return sum(range(n))

# partial — freeze some arguments
import requests
get_api = partial(requests.get, headers={"Accept": "application/json"})
# get_api("https://api.example.com")  # headers already set

# reduce
total = reduce(lambda a, b: a + b, [1,2,3,4,5])  # 15

# total_ordering — define __eq__ and one comparison, get the rest free
@total_ordering
class Priority:
    def __init__(self, level):
        self.level = level
    def __eq__(self, other):
        return self.level == other.level
    def __lt__(self, other):
        return self.level < other.level
```

### Q66. Weak references?
A **weak reference** allows you to refer to an object **without preventing garbage collection**. Normal references increase the reference count; weak references don't. When the object is garbage collected, the weak reference returns `None`.

**Use cases:** Caching expensive objects (they get cleaned up when memory is needed), observer patterns (observers don't keep subjects alive), and preventing circular references in parent-child relationships.

```python
import weakref

class ExpensiveObject:
    def __init__(self, name):
        self.name = name

obj = ExpensiveObject("test")
weak = weakref.ref(obj)

print(weak())       # <ExpensiveObject object>
print(weak().name)  # test

del obj
print(weak())       # None — object garbage collected
```

### Q67. Abstract Base Classes (ABC)?
**ABCs** define a **blueprint/contract** for subclasses. Methods decorated with `@abstractmethod` **must be implemented** by any concrete subclass — attempting to instantiate a class with unimplemented abstract methods raises `TypeError`.

**SDET use:** Defining a base test runner contract (setup/run/teardown), base page object interface, base API client with required methods. Ensures all implementations follow a consistent interface.

**Template Method Pattern:** The ABC defines the algorithm skeleton (`execute`), while subclasses fill in the specific steps — one of the most common patterns in test frameworks.

```python
from abc import ABC, abstractmethod

class TestRunner(ABC):
    @abstractmethod
    def setup(self):
        pass

    @abstractmethod
    def run(self):
        pass

    @abstractmethod
    def teardown(self):
        pass

    def execute(self):  # template method
        self.setup()
        try:
            result = self.run()
        finally:
            self.teardown()
        return result

class SeleniumTestRunner(TestRunner):
    def setup(self):
        self.driver = "ChromeDriver()"

    def run(self):
        return "test passed"

    def teardown(self):
        pass  # self.driver.quit()
```

### Q68. Enums?
**Enums** (Enumerations) represent a fixed set of **named constants**. They prevent magic strings/numbers in code, provide type safety, and make code self-documenting. Use `Enum` for string/custom values, `IntEnum` for integer values that support comparison.

**Key features:**
- `.value` — the assigned value ("pass", 1, etc.)
- `.name` — the constant name ("PASS", "HIGH")
- Iterable — loop through all members
- Comparable — `IntEnum` supports `<`, `>`, `==`
- `auto()` — auto-assign incrementing integer values

**SDET use:** Test statuses, priority levels, browser types, environment names — anything with a fixed set of valid values.

```python
from enum import Enum, auto, IntEnum

class TestStatus(Enum):
    PASS = "pass"
    FAIL = "fail"
    SKIP = "skip"
    ERROR = "error"

class Priority(IntEnum):
    LOW = auto()      # 1
    MEDIUM = auto()   # 2
    HIGH = auto()     # 3
    CRITICAL = auto() # 4

# Usage
status = TestStatus.PASS
print(status.value)  # "pass"
print(status.name)   # "PASS"

# Comparison
print(Priority.HIGH > Priority.LOW)  # True

# Iteration
for s in TestStatus:
    print(s.name, s.value)
```

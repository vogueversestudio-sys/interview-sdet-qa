# Python Interview Prep — SDET (8+ Years) — PART 2: INTERMEDIATE
## Covers: Noida | Hyderabad | Gurugram | Delhi | Pune | Chennai

---

# SECTION B — INTERMEDIATE LEVEL

---

## B1. Object-Oriented Programming (OOP)

### Q25. Four pillars of OOP in Python?
OOP (Object-Oriented Programming) is built on four fundamental pillars that help organize code into reusable, maintainable structures. In SDET work, OOP is crucial for designing **Page Object Models**, **test frameworks**, and **utility classes**.

**1. Encapsulation:**
Bundling data (attributes) and methods that operate on that data into a single unit (class), while restricting direct access to internal state. Python uses naming conventions: `_protected` (single underscore) and `__private` (double underscore, name-mangled).
```python
class BankAccount:
    def __init__(self):
        self.__balance = 0       # private (name mangled)
        self._transactions = []  # protected (convention)

    def deposit(self, amount):
        self.__balance += amount

    def get_balance(self):
        return self.__balance
```

**2. Inheritance:**
A mechanism where a child class inherits attributes and methods from a parent class, promoting code reuse. In SDET, we use inheritance for base test classes, base page objects, and shared utilities.

```python
class Animal:
    def speak(self):
        raise NotImplementedError

class Dog(Animal):
    def speak(self):
        return "Woof!"
```

**3. Polymorphism:**
"Many forms" — the ability of different classes to respond to the same method call in their own way. Achieved through method overriding (runtime) and duck typing in Python.

```python
class Cat(Animal):
    def speak(self):
        return "Meow!"

for animal in [Dog(), Cat()]:
    print(animal.speak())  # Woof! then Meow!
```

**4. Abstraction:**
Hiding complex implementation details and exposing only the necessary interface. Python uses the `abc` module (Abstract Base Class) to enforce that child classes implement required methods.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    def area(self):
        return 3.14159 * self.radius ** 2
```

### Q26. What is MRO (Method Resolution Order)?
Python uses **C3 Linearization** to determine method lookup order in multiple inheritance.

```python
class A:
    def show(self): print("A")

class B(A):
    def show(self): print("B")

class C(A):
    def show(self): print("C")

class D(B, C):
    pass

d = D()
d.show()  # "B"
print(D.__mro__)
# D → B → C → A → object
```

### Q27. `@staticmethod` vs `@classmethod` vs instance method?
- **Instance method** — takes `self` as first parameter, can access both instance and class data. Most common type.
- **Class method** (`@classmethod`) — takes `cls` as first parameter, can access class-level data but NOT instance data. Used for factory methods and alternative constructors.
- **Static method** (`@staticmethod`) — no `self` or `cls`. Just a regular function namespaced inside the class. Used for utility functions logically related to the class.

**SDET example:** In a framework, `@staticmethod` for utility helpers, `@classmethod` for browser factory methods, instance methods for page interactions.

```python
class MyClass:
    class_var = 0

    def instance_method(self):       # accesses instance & class
        return f"instance: {self}"

    @classmethod
    def class_method(cls):           # accesses class only
        cls.class_var += 1
        return cls.class_var

    @staticmethod
    def static_method(x, y):        # no access to instance/class
        return x + y
```

| Feature | Instance Method | Class Method | Static Method |
|---------|----------------|--------------|---------------|
| First param | `self` | `cls` | None |
| Access instance | Yes | No | No |
| Access class | Yes | Yes | No |

### Q28. Dunder (magic) methods?
**Dunder** (double underscore) or **magic methods** are special methods surrounded by `__` that Python calls automatically in response to certain operations. They allow you to define how objects behave with built-in operations like `+`, `==`, `len()`, `print()`, `in`, and `for` loops.

**Why important for SDET:** Custom assertion messages (`__str__`, `__repr__`), comparable test results (`__eq__`, `__lt__`), iterable test data (`__iter__`, `__next__`), and callable test objects (`__call__`).

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __len__(self):
        return int((self.x**2 + self.y**2)**0.5)

    def __getitem__(self, index):
        return (self.x, self.y)[index]

    def __call__(self):
        return self.x, self.y

    def __contains__(self, value):
        return value in (self.x, self.y)
```

**Key dunders:**
- `__init__`, `__new__`, `__del__`
- `__str__`, `__repr__`
- `__add__`, `__sub__`, `__mul__`, `__eq__`, `__lt__`, `__gt__`
- `__len__`, `__getitem__`, `__setitem__`, `__delitem__`
- `__iter__`, `__next__`
- `__enter__`, `__exit__` (context managers)
- `__call__`, `__hash__`

### Q29. `__str__` vs `__repr__`?
- `__str__`: Human-readable, used by `print()`, `str()`
- `__repr__`: Unambiguous, used in REPL, containers. Ideally valid Python expression.

```python
import datetime
d = datetime.date(2026, 2, 7)
print(str(d))    # 2026-02-07
print(repr(d))   # datetime.date(2026, 2, 7)
```
**Rule:** Always implement `__repr__`. `__str__` falls back to `__repr__` if missing.

### Q30. Properties in Python?
**Properties** allow you to define getter, setter, and deleter methods that are accessed like regular attributes. They enable **controlled access** to private attributes — you can add validation, logging, or computed values without changing the calling code.

**Why use properties instead of getters/setters?** Pythonic approach — `obj.name` is cleaner than `obj.get_name()`. Properties let you start with simple attributes and add logic later without breaking existing code.

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Below absolute zero!")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

t = Temperature(100)
print(t.fahrenheit)  # 212.0
t.celsius = 37       # uses setter
```

### Q31. Multiple inheritance & diamond problem?
**Multiple inheritance** means a class inherits from more than one parent class. The **diamond problem** occurs when two parent classes inherit from the same grandparent, creating ambiguity about which method to call.

Python resolves this using **MRO (Method Resolution Order)** with **C3 Linearization** algorithm — it creates a deterministic, left-to-right, depth-first order. Use `ClassName.__mro__` or `ClassName.mro()` to inspect the resolution order.

```python
class A:
    def greet(self): print("A")
class B(A):
    def greet(self): print("B")
class C(A):
    def greet(self): print("C")
class D(B, C):
    pass

d = D()
d.greet()  # "B" — resolved by MRO: D → B → C → A → object
```

Use `super()` for cooperative multiple inheritance.

---

## B2. Decorators

### Q32. What are decorators?
A **decorator** is a function that takes another function as input, adds some functionality (wraps it), and returns the enhanced function — all **without modifying the original function's code**. The `@decorator` syntax is syntactic sugar for `func = decorator(func)`.

**Key concepts:**
- Always use `@functools.wraps(func)` inside your decorator to preserve the original function's `__name__`, `__doc__`, and other metadata
- Decorators are heavily used in test frameworks: `@pytest.fixture`, `@pytest.mark.parametrize`, `@retry`, `@allure.step`
- They follow the **Open/Closed Principle** — open for extension, closed for modification

```python
# Basic decorator
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time()-start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

slow_function()
```

```python
# Decorator with arguments
def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt} failed: {e}")
                    if attempt == max_attempts:
                        raise
        return wrapper
    return decorator

@retry(max_attempts=5)
def flaky_api_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("timeout")
    return "Success"
```

### Q33. Class-based decorators?
Instead of using a function, you can create decorators using a **class with `__call__`** method. This is useful when you need to **maintain state** across multiple calls (e.g., counting invocations, caching results). The `__init__` receives the decorated function, and `__call__` is invoked every time the decorated function is called.

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"{self.func.__name__} called {self.count} times")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    print("Hello!")

say_hello()  # say_hello called 1 times
say_hello()  # say_hello called 2 times
```

---

## B3. Generators & Iterators

### Q34. Generators?
A **generator** is a special function that uses `yield` instead of `return` to produce a sequence of values **lazily** (one at a time, on demand). Unlike regular functions that compute all values and return a list (consuming memory), generators produce values only when requested — making them **memory-efficient** for large datasets.

**How it works:** When `yield` is encountered, the function's state is **frozen** (local variables, instruction pointer) and the yielded value is returned. On the next call to `next()`, execution resumes from where it left off.

**SDET use cases:** Reading large log files line-by-line, generating test data streams, paginated API responses, and processing large CSV test datasets.

```python
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

for num in fibonacci(10):
    print(num, end=" ")  # 0 1 1 2 3 5 8 13 21 34

# Generator expression
squares = (x**2 for x in range(1000000))  # lazy, no memory
```

| Feature | Function | Generator |
|---------|----------|-----------|
| Returns | Value via `return` | Iterator via `yield` |
| Memory | All at once | One at a time |
| State | Lost after return | Preserved between yields |

### Q35. Iterator vs Iterable?
- **Iterable:** has `__iter__()` → `list`, `str`, `dict`
- **Iterator:** has `__iter__()` AND `__next__()`

```python
my_list = [1, 2, 3]         # iterable
my_iter = iter(my_list)      # iterator
print(next(my_iter))  # 1
print(next(my_iter))  # 2
```

```python
# Custom iterator
class Countdown:
    def __init__(self, start):
        self.start = start
    def __iter__(self):
        return self
    def __next__(self):
        if self.start <= 0:
            raise StopIteration
        self.start -= 1
        return self.start + 1

for num in Countdown(5):
    print(num, end=" ")  # 5 4 3 2 1
```

### Q36. `yield from`?
`yield from` **delegates** iteration to another iterable or sub-generator. Instead of manually looping and yielding each item from a nested generator, `yield from` does it in a single statement — cleaner and more efficient. It also properly propagates `send()`, `throw()`, and `close()` calls to the sub-generator.

```python
def inner():
    yield 1
    yield 2

def outer():
    yield from inner()
    yield 3

list(outer())  # [1, 2, 3]
```

---

## B4. Exception Handling

### Q37. Exception handling syntax?
Python uses `try/except/else/finally` blocks for exception handling:
- **`try`** — code that might raise an exception
- **`except`** — handles specific exceptions (catch most specific first)
- **`else`** — runs **only if no exception** occurred (good for code that should run on success)
- **`finally`** — **always runs** regardless of exceptions (cleanup: close files, release resources)

**Best practices:** Never use bare `except:` — always catch specific exceptions. Use `except Exception as e` to log the error message.

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (TypeError, ValueError) as e:
    print(f"Type/Value Error: {e}")
except Exception as e:
    print(f"Unexpected: {e}")
else:
    print("No exception")    # only if no exception
finally:
    print("Always executes") # cleanup
```

### Q38. Custom exceptions?
Create custom exceptions by inheriting from `Exception` (or a more specific built-in exception). Custom exceptions provide **meaningful error context** specific to your application or test framework. They make error handling more precise and debugging easier.

**SDET pattern:** Create a base exception for your framework (e.g., `TestFrameworkError`), then specific exceptions like `ElementNotFoundError`, `APITimeoutError`, `TestDataError` that inherit from it.

```python
class TestExecutionError(Exception):
    def __init__(self, test_name, message="Test failed"):
        self.test_name = test_name
        super().__init__(f"{message}: {test_name}")

class ElementNotFoundError(TestExecutionError):
    def __init__(self, locator):
        super().__init__("Element Lookup", f"Not found: {locator}")

try:
    raise ElementNotFoundError("//button[@id='submit']")
except TestExecutionError as e:
    print(e)
```

### Q39. `raise` vs `raise from`?
- **`raise ExceptionType(msg)`** — raises a new exception. If inside an `except` block, Python automatically chains it with the original exception as `__context__`.
- **`raise NewException() from original`** — explicitly sets the cause (`__cause__`), showing "The above exception was the direct cause of the following exception."
- **`raise NewException() from None`** — suppresses the exception chain entirely, hiding the original error.

**When to use `from`:** When you want to clearly indicate that one error directly caused another, providing a clean traceback for debugging.

```python
try:
    int("abc")
except ValueError as original:
    raise RuntimeError("Conversion failed") from original
    # "The above exception was the direct cause..."

try:
    int("abc")
except ValueError:
    raise RuntimeError("Failed") from None
    # Suppresses the original exception chain
```

---

## B5. File Handling

### Q40. File operations?
Python uses the built-in `open()` function with **mode flags**: `r` (read), `w` (write/overwrite), `a` (append), `rb`/`wb` (binary). Always use the `with` statement (context manager) — it **automatically closes** the file even if an exception occurs.

**SDET use cases:** Reading test data from files, writing test reports, parsing log files for error analysis, generating CSV/JSON test results.

```python
# Write
with open("report.txt", "w") as f:
    f.write("Test Results\n")

# Read
with open("report.txt", "r") as f:
    content = f.read()           # entire file
    # lines = f.readlines()      # list of lines
    # for line in f:             # line by line (memory efficient)

# Append
with open("report.txt", "a") as f:
    f.write("PASS: test_search\n")
```

### Q41. JSON and CSV handling?
**JSON** (JavaScript Object Notation) is the standard format for API request/response bodies. **CSV** (Comma-Separated Values) is used for tabular test data and data-driven testing.

**Key JSON functions:**
- `json.dump(obj, file)` — write Python dict to JSON file
- `json.load(file)` — read JSON file to Python dict
- `json.dumps(obj)` — convert to JSON string
- `json.loads(string)` — parse JSON string to dict

**SDET use:** JSON for API test payloads and config files. CSV for parametrized test data (login credentials, search queries, etc.).

```python
import json, csv

# JSON
data = {"test": "login", "status": "pass"}
with open("results.json", "w") as f:
    json.dump(data, f, indent=4)
with open("results.json", "r") as f:
    loaded = json.load(f)

json_str = json.dumps(data)
parsed = json.loads(json_str)

# CSV
with open("results.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["test", "status"])
    writer.writeheader()
    writer.writerow({"test": "login", "status": "pass"})

with open("results.csv", "r") as f:
    for row in csv.DictReader(f):
        print(row["test"], row["status"])
```

---

## B6. Context Managers

### Q42. What are context managers?
A context manager is an object that manages **resource setup and teardown** automatically using the `with` statement. It ensures resources (files, DB connections, browser sessions) are properly **acquired** on entry and **released** on exit, even if exceptions occur.

**Two ways to create:**
1. **Class-based** — implement `__enter__()` (setup, returns resource) and `__exit__()` (teardown, handles exceptions)
2. **Function-based** — use `@contextmanager` decorator from `contextlib` with a `yield` statement

**SDET use cases:** Managing browser sessions, database connections, temporary test data, API authentication tokens, and timing test execution.

```python
# Class-based
class DBConnection:
    def __init__(self, conn_str):
        self.conn_str = conn_str
    def __enter__(self):
        self.conn = "connected"
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn = None
        return False  # propagate exceptions

with DBConnection("localhost:5432") as db:
    print(db.conn)

# Using contextlib
from contextlib import contextmanager

@contextmanager
def timer(label):
    import time
    start = time.time()
    try:
        yield
    finally:
        print(f"{label}: {time.time()-start:.4f}s")

with timer("API Test"):
    time.sleep(0.5)
```

---

## B7. Regular Expressions

### Q43. Regex basics in Python?
**Regular expressions (regex)** are patterns used to match, search, and manipulate strings. Python's `re` module provides regex support. Key for SDETs when **validating data formats**, **parsing logs**, **extracting test data**, and **API response validation**.

**Key functions:**
- `re.search(pattern, string)` — finds **first match** anywhere in string
- `re.match(pattern, string)` — matches only at the **beginning** of string
- `re.findall(pattern, string)` — returns **all matches** as a list
- `re.sub(pattern, replacement, string)` — **replaces** all matches
- `re.compile(pattern)` — pre-compiles for **reuse** (performance optimization)

```python
import re

text = "Error 404: Page not found at 2026-02-07 13:00:00"

# Search
match = re.search(r'\d{4}-\d{2}-\d{2}', text)
if match:
    print(match.group())  # 2026-02-07

# Find all
numbers = re.findall(r'\d+', text)  # ['404', '2026', '02', '07', '13', '00', '00']

# Match (anchored to start)
if re.match(r'Error', text):
    print("Starts with Error")

# Sub (replace)
clean = re.sub(r'\d+', 'X', text)  # "Error X: Page not found at X-X-X X:X:X"

# Compile (reusable)
pattern = re.compile(r'(\d{4})-(\d{2})-(\d{2})')
m = pattern.search(text)
print(m.group(1))  # 2026 (year)
print(m.groups())  # ('2026', '02', '07')

# Named groups
pattern = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})')
m = pattern.search(text)
print(m.group('year'))  # 2026
```

**Common patterns for SDET:**
```python
# Email validation
email_pat = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

# Phone (Indian)
phone_pat = r'^(\+91|91|0)?[6-9]\d{9}$'

# URL
url_pat = r'https?://(?:www\.)?[\w.-]+\.\w{2,}(?:/\S*)?'

# IP address
ip_pat = r'\b(?:\d{1,3}\.){3}\d{1,3}\b'

# Log timestamp
log_pat = r'\d{4}-\d{2}-\d{2}\s\d{2}:\d{2}:\d{2}'
```

---

## B8. Multithreading Basics

### Q44. Threading in Python?
**Threading** allows concurrent execution of multiple tasks. Python's `threading` module creates threads that share the same memory space. Due to the **GIL (Global Interpreter Lock)**, threads don't achieve true parallelism for CPU-bound tasks but are excellent for **I/O-bound** operations (file reading, network requests, database queries).

**Key concepts:**
- `Thread(target=func, args=())` — creates a thread
- `thread.start()` — begins execution
- `thread.join()` — waits for thread to finish
- `Lock()` — prevents race conditions when multiple threads access shared data

**SDET use cases:** Parallel API calls, concurrent file downloads, running browser tests alongside API checks.

```python
import threading
import time

def download_file(filename):
    print(f"Downloading {filename}...")
    time.sleep(2)
    print(f"Downloaded {filename}")

# Create threads
threads = []
for f in ["file1.txt", "file2.txt", "file3.txt"]:
    t = threading.Thread(target=download_file, args=(f,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()  # wait for all to finish

print("All downloads complete")
```

```python
# Thread-safe with Lock
lock = threading.Lock()
counter = 0

def increment():
    global counter
    for _ in range(100000):
        with lock:
            counter += 1

t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)
t1.start(); t2.start()
t1.join(); t2.join()
print(counter)  # 200000 (correct with lock)
```

### Q45. concurrent.futures?
The `concurrent.futures` module provides a **high-level interface** for asynchronous execution using thread or process pools. It's simpler than raw `threading`/`multiprocessing`.

- **`ThreadPoolExecutor`** — pool of threads for **I/O-bound** tasks (API calls, file I/O)
- **`ProcessPoolExecutor`** — pool of processes for **CPU-bound** tasks (data processing, heavy computation)
- **`as_completed(futures)`** — yields futures as they finish (not in submission order)
- **`executor.map(func, iterable)`** — parallel map (results in submission order)

**SDET use:** Running multiple API tests concurrently, parallel data validation, bulk file processing.

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed

# Thread pool (I/O-bound)
def fetch(url):
    import requests
    return requests.get(url).status_code

urls = ["https://httpbin.org/get"] * 5

with ThreadPoolExecutor(max_workers=5) as executor:
    futures = {executor.submit(fetch, url): url for url in urls}
    for future in as_completed(futures):
        url = futures[future]
        print(f"{url}: {future.result()}")

# Process pool (CPU-bound)
def cpu_heavy(n):
    return sum(i*i for i in range(n))

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_heavy, [10**6]*4))
```

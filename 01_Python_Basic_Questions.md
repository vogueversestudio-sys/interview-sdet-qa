# Python Interview Prep — SDET (8+ Years) — PART 1: BASIC
## Covers: Noida | Hyderabad | Gurugram | Delhi | Pune | Chennai

---

# SECTION A — BASIC LEVEL (Theoretical)

## Q1. What is Python? Why popular for test automation?
Python is a high-level, interpreted, dynamically-typed language. Popular for SDET because:
- Simple readable syntax
- Rich testing ecosystem (pytest, unittest, robot framework, behave)
- Libraries for API (requests), UI (Selenium), mobile (Appium)
- Cross-platform, large community

## Q2. Python 2 vs Python 3?
| Feature | Python 2 | Python 3 |
|---------|----------|----------|
| Print | `print "hi"` | `print("hi")` |
| Division | `5/2 = 2` | `5/2 = 2.5` |
| Unicode | ASCII default | Unicode default |
| range() | Returns list | Returns iterator |
| Support | EOL Jan 2020 | Active |

## Q3. Built-in data types?
| Category | Types |
|----------|-------|
| Numeric | `int`, `float`, `complex`, `bool` |
| Sequence | `list`, `tuple`, `range` |
| Text | `str` |
| Set | `set`, `frozenset` |
| Mapping | `dict` |
| Binary | `bytes`, `bytearray`, `memoryview` |
| None | `NoneType` |

## Q4. list vs tuple vs set vs dict?
| Feature | List | Tuple | Set | Dict |
|---------|------|-------|-----|------|
| Mutable | Yes | No | Yes | Yes |
| Ordered | Yes | Yes | No | Yes (3.7+) |
| Duplicates | Yes | Yes | No | Keys:No |
| Syntax | `[1,2]` | `(1,2)` | `{1,2}` | `{k:v}` |

## Q5. `is` vs `==`?
- `==` checks **value equality** (calls `__eq__`)
- `is` checks **identity** (same object in memory)

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)   # True  (same value)
print(a is b)   # False (different objects)
```

## Q6. Mutable vs Immutable?
- **Immutable:** `int`, `float`, `str`, `tuple`, `frozenset`, `bytes`, `bool`
- **Mutable:** `list`, `dict`, `set`, `bytearray`

## Q7. Shallow copy vs Deep copy?
**Shallow copy** creates a new object but inserts references to the same inner objects. Changes to nested objects affect both copies. **Deep copy** creates a completely independent clone — all objects are recursively copied, so changes to one don't affect the other. In testing, deep copy is essential when you need independent test data copies.

```python
import copy
original = [[1, 2], [3, 4]]
shallow = copy.copy(original)       # inner objects shared
deep = copy.deepcopy(original)      # completely independent
original[0][0] = 999
print(shallow)  # [[999, 2], [3, 4]]
print(deep)     # [[1, 2], [3, 4]]
```

## Q8. `*args` and `**kwargs`?
- `*args` — positional args → **tuple**
- `**kwargs` — keyword args → **dict**

```python
def demo(*args, **kwargs):
    print(args)    # (1, 2, 3)
    print(kwargs)  # {'name': 'Vikrant'}
demo(1, 2, 3, name="Vikrant")
```

## Q9. List comprehensions?
A **list comprehension** is a concise, Pythonic way to create lists in a single line. It replaces multi-line `for` loops with a readable `[expression for item in iterable if condition]` syntax. It's faster than traditional loops because it's optimized internally by Python. You can also create **dict comprehensions** `{k:v for ...}` and **set comprehensions** `{x for ...}`.

```python
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
sq_dict = {x: x**2 for x in range(5)}
```

## Q10. `append()` vs `extend()` vs `insert()`?
- **`append(x)`** — adds `x` as a **single element** to the end (even if `x` is a list, it becomes a nested list)
- **`extend(iterable)`** — adds **each element** of the iterable individually to the end
- **`insert(index, x)`** — inserts `x` at a **specific position**, shifting existing elements right

**Time complexity:** `append()` is O(1), `extend()` is O(k) where k is length of iterable, `insert()` is O(n) due to shifting.

```python
lst = [1, 2, 3]
lst.append([4, 5])   # [1, 2, 3, [4, 5]]
lst = [1, 2, 3]
lst.extend([4, 5])   # [1, 2, 3, 4, 5]
lst = [1, 2, 3]
lst.insert(1, 99)    # [1, 99, 2, 3]
```

## Q11. `remove()` vs `pop()` vs `del`?
- **`remove(value)`** — removes the **first occurrence** of the value. Raises `ValueError` if not found.
- **`pop(index)`** — removes element at the given **index** and **returns it**. Default is last element (-1). Raises `IndexError` if empty.
- **`del list[index]`** — deletes element at index. No return value. Can also delete slices: `del list[1:3]`.

**Key difference for interviewer:** `remove()` searches by **value**, while `pop()` and `del` work by **index**.

```python
lst = [10, 20, 30, 20]
lst.remove(20)       # Removes first occurrence → [10, 30, 20]
lst = [10, 20, 30]
val = lst.pop(1)     # Removes by index, returns value → val=20
del lst[0]           # Removes by index, no return
```

## Q12. Python memory management?
- **Private heap** for all objects
- **Reference counting** — primary mechanism
- **Garbage collector** — handles cyclic references (3 generations)
- **pymalloc** — small object allocator (≤512 bytes)

## Q13. `break`, `continue`, `pass`?
- `break` — exits loop entirely
- `continue` — skips current iteration
- `pass` — does nothing (placeholder)

## Q14. Namespaces & LEGB rule?
1. **L — Local:** Inside current function
2. **E — Enclosing:** Enclosing function (closures)
3. **G — Global:** Module-level
4. **B — Built-in:** Python built-ins

## Q15. `global` and `nonlocal`?
- **`global`** — declares that a variable inside a function refers to the **module-level (global)** variable, not a local one. Without it, assigning to a variable inside a function creates a new local variable.
- **`nonlocal`** — used in **nested functions** to refer to a variable in the **enclosing (outer) function's** scope. It bridges the gap between local and global.

**Interview tip:** Overusing `global` is considered bad practice — prefer passing parameters and returning values. `nonlocal` is commonly used in closures and decorators.

```python
count = 0
def increment():
    global count
    count += 1

def outer():
    x = 10
    def inner():
        nonlocal x
        x += 5
    inner()
    print(x)  # 15
```

## Q16. f-strings (Formatted String Literals)?
Introduced in **Python 3.6**, f-strings are the **fastest and most readable** way to format strings. They allow embedding expressions directly inside `{}` braces with an `f` prefix. They support format specifiers like `{value:.2f}` for decimals and `{value:>10}` for alignment.

**Three string formatting methods (newest to oldest):**
1. **f-strings** `f"{var}"` — fastest, most readable (Python 3.6+)
2. **str.format()** `"{}".format(var)` — flexible but verbose (Python 2.6+)
3. **% operator** `"%s" % var` — legacy C-style, not recommended

```python
name = "Vikrant"
print(f"{name} has {8}+ years exp")  # fastest, Python 3.6+
print("{} has {}+ years".format(name, 8))
print("%s has %d+ years" % (name, 8))
```

## Q17. Walrus operator `:=` (Python 3.8+)?
The **walrus operator** (`:=`) is an **assignment expression** — it assigns a value to a variable **and** returns that value in a single expression. Named after its resemblance to a walrus's eyes and tusks. It reduces code duplication by combining assignment and condition checking.

**Common use cases:** while loops with input, list comprehensions with computed values, and avoiding redundant function calls.

```python
while (data := input("Enter: ")) != "quit":
    print(data)
results = [y for x in range(10) if (y := x**2) > 20]
```

## Q18. Lambda functions?
A **lambda** is an **anonymous (unnamed) inline function** defined with the `lambda` keyword. It can take any number of arguments but has only **one expression** (no statements, no multi-line logic). Commonly used as short callback functions in `sorted()`, `map()`, `filter()`, and UI event handlers.

**Lambda vs def:** Lambda is for simple one-liners; use `def` for anything complex. Lambda cannot contain assignments, loops, or multiple statements.

```python
square = lambda x: x ** 2
sorted_nums = sorted([3,1,4], key=lambda x: -x)
```

## Q19. `map()`, `filter()`, `reduce()`?
These are **functional programming** tools for transforming and aggregating data:
- **`map(func, iterable)`** — applies `func` to **every element** and returns an iterator of results
- **`filter(func, iterable)`** — returns only elements where `func` returns **True**
- **`reduce(func, iterable)`** — cumulatively applies `func` to pairs, reducing the iterable to a **single value** (must import from `functools`)

**Interview tip:** In modern Python, **list comprehensions** are preferred over `map`/`filter` for readability. `reduce` is still useful for cumulative operations.

```python
from functools import reduce
nums = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, nums))
evens = list(filter(lambda x: x % 2 == 0, nums))
total = reduce(lambda a, b: a + b, nums)  # 15
```

## Q20. Closures?
A **closure** is a nested function that **remembers and has access to variables** from its enclosing function's scope, even after the outer function has finished executing. The inner function "closes over" the free variables.

**Three conditions for a closure:**
1. There must be a **nested function** (function inside function)
2. The inner function must **reference a variable** from the enclosing scope
3. The enclosing function must **return** the inner function

**Use in SDET:** Closures are used in decorators, factory functions (creating parametrized test functions), and callback patterns.

```python
def multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply
double = multiplier(2)
print(double(5))  # 10
```

## Q21. String methods cheat sheet?
Python strings are **immutable** — every string method returns a **new string** without modifying the original. Key categories: **trimming** (`strip`), **case conversion** (`lower`, `upper`, `title`), **searching** (`find`, `index`, `startswith`), **splitting/joining** (`split`, `join`), and **validation** (`isdigit`, `isalpha`, `isalnum`).

**Interview tip:** `find()` returns `-1` if not found, while `index()` raises `ValueError`. Always prefer `find()` for safe lookups.

```python
s = "  Hello, World!  "
s.strip()       # "Hello, World!"
s.lower()       # "  hello, world!  "
s.split(",")    # ['  Hello', ' World!  ']
",".join(["a","b"])  # "a,b"
s.find("World") # 9 (-1 if not found)
s.replace("World", "Python")
s.startswith("  He")  # True
s.isdigit()     # False
```

## Q22. Reverse a string?
**Three approaches:**
1. **Slicing** `s[::-1]` — most Pythonic and fastest (O(n))
2. **`reversed()` + join** — built-in iterator approach
3. **Loop** — manual character-by-character (least efficient)

**Interview tip:** Slicing `[::-1]` works because it creates a new string stepping backward: `[start:stop:step]` where step is -1.

```python
s = "automation"
print(s[::-1])              # "noitamotua"
print("".join(reversed(s)))
```

## Q23. Check palindrome?
A **palindrome** reads the same forward and backward (e.g., "madam", "racecar"). The approach is to normalize the string (lowercase, remove spaces/special chars) and then compare it with its reverse.

**Time complexity:** O(n) — we traverse the string once for reversal.
**Space complexity:** O(n) — for the reversed copy.

```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]
```

## Q24. Function vs Method?
- **Function:** standalone `def greet(): ...`
- **Method:** bound to object `obj.greet()` — receives `self` or `cls`

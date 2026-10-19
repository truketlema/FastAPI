# Python Type Hints: Making Your Code Self-Documenting and Safer

## What is it?

Python is a **dynamically typed** language. This means you don't declare the type of a variable when you create it, and a variable can hold values of different types throughout its lifetime.

```python
user_id = 42        # int
user_id = "42"      # str — perfectly valid in Python!
```

**Type hints** (introduced in PEP 484, Python 3.5) let you *optionally* annotate your code with expected types. They are **not enforced at runtime** by the Python interpreter itself—your code will run exactly the same whether hints are present or not. Instead, they serve as **static metadata** that tools (linters, type checkers like `mypy`, IDEs) can analyze *before* you run your code.

Think of type hints as **executable documentation**. A docstring tells a human what a function expects; a type hint tells a tool—and therefore the human reading the tool's output—what the function expects, with machine precision.

Basic syntax uses colons for arguments and `->` for return values:

```python
def greet(name: str) -> str:
    return f"Hello, {name}"
```

This reads: "`greet` takes one argument `name` of type `str`, and returns a value of type `str`."

## Why does it matter?

If Python ignores them at runtime, why bother? Three practical reasons dominate real-world engineering:

### 1. Catch bugs before runtime
A type checker (like `mypy`, `pyright`, or your IDE's built-in analyzer) can spot entire classes of errors statically:

```python
def calculate_area(width: float, height: float) -> float:
    return width * height

calculate_area("10", 5)  # mypy: Argument 1 has incompatible type "str"; expected "float"
```

Without hints, this fails at runtime with a `TypeError` (or worse, succeeds silently if the operation is valid for the wrong type). With hints + a checker, you catch it while editing.

### 2. IDE superpowers
Modern editors (VS Code + Pylance, PyCharm, Zed) use type hints to provide:
- **Autocomplete** that knows `.strip()` exists on `str` but not on `int`
- **Go-to-definition** that actually works
- **Inline error squiggles** for mismatched types
- **Safe refactoring** (rename a method, update all call sites confidently)

### 3. Living documentation
Function signatures become contracts. Compare:

```python
# Without hints — what does `data` look like? What comes back?
def process(data):
    ...

# With hints — the shape is explicit
def process(data: list[dict[str, int]]) -> tuple[int, list[str]]:
    ...
```

The second version answers "what do I pass?" and "what do I get?" without reading the implementation or hunting for a docstring that may be outdated.

---

## How does it work?

### The mental model: Two layers
| Layer | What happens | Tools |
|-------|--------------|-------|
| **Runtime** | Python executes code; hints are stored in `__annotations__` but ignored | `inspect`, `typing.get_type_hints()` |
| **Static analysis** | Type checker reads hints, builds a type graph, verifies consistency | `mypy`, `pyright`, `pytype`, IDEs |

You write hints → run a checker (or let your IDE run it continuously) → get feedback → fix mismatches.

---

### Built-in types (Python 3.9+)

Since Python 3.9, you can use standard collection types directly—no `typing.List` import needed:

```python
def top_scores(scores: list[int]) -> list[int]:
    return sorted(scores, reverse=True)[:3]

def user_lookup(users: dict[str, int], name: str) -> int | None:
    return users.get(name)
```

**Common built-ins:**
| Hint | Meaning |
|------|---------|
| `int`, `float`, `str`, `bool` | Primitive types |
| `list[T]`, `tuple[T, ...]`, `set[T]` | Homogeneous containers |
| `dict[K, V]` | Mapping from keys `K` to values `V` |
| `T | None` or `Optional[T]` | Value or `None` (Python 3.10+) |
| `Callable[[ArgTypes], ReturnType]` | Function signatures |

---

### The `typing` module: Advanced building blocks

For types beyond built-ins, import from `typing`:

```python
from typing import Optional, Union, Sequence, Mapping, Any

# Before Python 3.10, Union was required for "or"
def parse_id(raw: Union[str, int]) -> Optional[int]:
    try:
        return int(raw)
    except ValueError:
        return None
```

**Key special forms:**

| Form | Purpose |
|------|---------|
| `Optional[T]` | Shorthand for `T \| None` (pre-3.10) |
| `Union[T1, T2, ...]` | One of several types |
| `Sequence[T]` | Read-only iterable (list, tuple, str…) — prefer over `list[T]` for parameters |
| `Mapping[K, V]` | Read-only dict-like — prefer over `dict[K, V]` for parameters |
| `Any` | "Turn off checking for this value" — use sparingly |
| `TypeVar` | Generics (see below) |

**Why `Sequence` over `list` for parameters?**  
If a function only *reads* a collection, annotate it as `Sequence[T]`. Callers can pass a `list`, `tuple`, or even a `range`—your function doesn't care, and the type checker enforces read-only usage.

---

### Generics: Reusable type-safe components

`TypeVar` lets you write functions/classes that preserve type relationships:

```python
from typing import TypeVar, Sequence

T = TypeVar("T")

def first_item(items: Sequence[T]) -> T | None:
    return items[0] if items else None

# Type checker knows:
# first_item([1, 2, 3])      -> int | None
# first_item(["a", "b"])     -> str | None
# first_item([1, "oops"])    -> int | str | None  (inferred as Union)
```

Without `TypeVar`, you'd have to write `Sequence[Any]` and lose the connection between input and output types.

---

### Classes and `Self`

Annotating methods in classes:

```python
class User:
    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email

    def with_email(self, new_email: str) -> "User":  # Forward reference (pre-3.11)
        return User(self.name, new_email)
```

**Python 3.11+**: Use `typing.Self` for fluent APIs:

```python
from typing import Self

class QueryBuilder:
    def __init__(self) -> None:
        self._filters: list[str] = []

    def where(self, clause: str) -> Self:
        self._filters.append(clause)
        return self

    def build(self) -> str:
        return " AND ".join(self._filters)
```

Now `QueryBuilder().where("a=1").where("b=2").build()` type-checks perfectly—the checker knows each `.where()` returns the same concrete type.

---

### Protocols: Structural subtyping (duck typing, formalized)

Python's "if it walks like a duck" philosophy is captured by `Protocol`:

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("○")

class Square:
    def draw(self) -> None:
        print("□")

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())   # OK
render(Square())   # OK
render("not a shape")  # mypy error: "str" has no "draw" method
```

No inheritance required—`Circle` and `Square` implicitly satisfy `Drawable` by having a compatible `draw` method. This is **structural typing**, and it fits Python's dynamic nature far better than nominal (class-based) interfaces.

---

### Practical workflow

1. **Add hints incrementally** — Start at module boundaries (public APIs, function signatures). Internal helpers can wait.
2. **Run a type checker in CI** — `mypy` or `pyright` as a GitHub Action / GitLab job / pre-commit hook.
3. **Use `py.typed` marker** — If you publish a package, include a `py.typed` file so downstream users get your hints.
4. **Strictness levels** — `mypy --strict` enables all checks. Start looser (`--disallow-untyped-defs` is a good middle ground) and tighten over time.

---

### Common mistakes & mental models

| Mistake | Why it's wrong | Better approach |
|---------|----------------|-----------------|
| `list` instead of `list[int]` | Loses element type info | Always parameterize generics |
| `dict` instead of `dict[str, int]` | Same problem | Parameterize both key and value |
| `Any` everywhere | Defeats the purpose | Use `object` for "any value," `Any` only at dynamic boundaries (JSON, ORM) |
| Annotating `*args: str` | Means *each* arg is `str`, not the tuple | Use `*args: str` for variadic str args; `args: tuple[str, ...]` for a tuple parameter |
| Mutable default arguments + hints | `def f(items: list[int] = []):` still shares the list | `def f(items: list[int] | None = None): items = items or []` |

**Mental model: Types are sets of values.**  
`int` = {..., -1, 0, 1, ...}  
`str` = {"", "a", "hello", ...}  
`int | str` = union of those sets.  
`list[int]` = all lists where *every* element is in the `int` set.  
Subtyping = subset relationship. `bool` is a subtype of `int` (since `True == 1`), which is a frequent source of subtle bugs—be aware.

---

## Key Takeaways

- **Type hints are optional, static metadata** — they don't change runtime behavior but enable powerful static analysis.
- **Run a type checker (mypy, pyright) in your editor and CI** — hints without checking are just comments.
- **Use built-in generics (`list[int]`, `dict[str, int]`)** — cleaner than `typing.List`/`Dict` (Python 3.9+).
- **Prefer `Sequence[T]` and `Mapping[K, V]` for read-only parameters** — accepts more types, prevents accidental mutation.
- **`TypeVar` preserves type relationships** — essential for reusable utilities that return what they receive.
- **`Protocol` formalizes duck typing** — no inheritance needed; structural compatibility is enough.
- **Avoid `Any` except at system boundaries** — it opts out of checking entirely.
- **Adopt incrementally** — start at public interfaces, expand inward, tighten strictness over time.
- **Types are sets; subtyping is subset** — this mental model explains variance, `bool`/`int` quirks, and why `list[Dog]` is not a subtype of `list[Animal]`.
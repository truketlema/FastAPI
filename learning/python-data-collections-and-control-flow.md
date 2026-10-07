# Python Data Collections and Control Flow

## What is it?

At the most basic level, Python programs are just instructions for manipulating data. However, a single variable usually holds only one piece of data. In real-world software, you rarely work with a single number or string at a time. You work with lists of users, groups of settings, or collections of results.

Python provides **collections** (data structures) to group multiple values into a single variable, and **control flow** statements to dictate the order in which code runs.

The four core collection types are:
*   **Lists:** Ordered groups of items.
*   **Tuples:** Ordered groups of items that cannot change.
*   **Sets:** Groups of unique items with no specific order.
*   **Dictionaries:** Groups of data linked by unique keys.

The two main control flow mechanisms are:
*   **Conditionals (`if`):** Making decisions based on data.
*   **Loops (`for`/`while`):** Repeating actions.

Think of collections as different types of storage containers, and control flow as the logic that decides what you put in them or how you retrieve things from them.

## Why does it matter?

You cannot write meaningful software without these concepts. Almost every application needs to store a user's session, validate input, or process a database query.

Collections matter because they allow you to organize data logically. If you have a list of 1,000 customers, you don't write 1,000 variables named `customer1`, `customer2`. You write one variable, `customers`, and manipulate the whole group.

Control flow matters because it turns a static script into a dynamic program. Without `if` statements, a login form cannot distinguish between a correct and incorrect password. Without loops, you cannot process every item in a dataset without copy-pasting the same logic thousands of times. Together, these features allow you to scale a program from solving one small problem to solving a complex, repeatable task.

## How does it work?

### The Four Containers

The main distinction between collections lies in their **ordering**, **mutability**, and **identity rules**.

**Lists** are ordered and mutable. You can add, remove, or reorder items. Think of this as a to-do list where you might cross things off or add new tasks.

```python
shopping_list = ["milk", "eggs", "bread"]
shopping_list.append("cheese")  # Adds item
```

**Tuples** are ordered but immutable. Once created, you cannot change the contents. This is useful for data that should remain constant, like geographic coordinates or database configuration. Because they cannot change, they are also safer to use as keys in other structures.

```python
coordinates = (40.7128, -74.0060)
# coordinates[0] = 50.0  # This raises an error: 'tuple' object does not support item assignment
```

**Sets** contain unique items with no guaranteed order. If you add a duplicate, Python ignores it. This is ideal for filtering out repetition, such as tracking which users have already logged in.

```python
seen_users = {"alice", "bob"}
seen_users.add("alice")  # No change: 'alice' is already there
```

**Dictionaries** map a unique **key** to a **value**. This is the most powerful collection for lookup. Instead of searching through a list for a specific item, you ask for the item by its label.

```python
user = {"username": "alice", "role": "admin"}
user["role"] = "moderator"  # Update the value associated with the key
```

### Control Flow

Conditionals allow your program to branch. The `if` keyword checks a **boolean expression** (something that is `True` or `False`). If the expression is true, the indented block runs.

```python
age = 18
if age >= 18:
    print("Adult access granted")
else:
    print("Access denied")
```

Loops allow repetition. The `for` loop is specifically designed to iterate over a collection. It takes one item at a time from the list, tuple, set, or dictionary keys, and runs the block of code for that item.

```python
guests = ["Alice", "Bob", "Charlie"]
for name in guests:
    print(f"Hello, {name}")
```

### Putting It Together

The real power comes from combining collections with control flow. For example, iterating through a dictionary to calculate an average grade:

```python
scores = {"Alice": 85, "Bob": 90, "Charlie": 78}
total = 0
count = 0

for name, grade in scores.items():  # .items() gives keys AND values
    if grade >= 80:
        print(f"{name} passed")  # Conditional logic inside the loop
    total += grade
    count += 1

print(f"Average: {total / count}")
```

In this example, `.items()` unpacks the dictionary into pairs so we can use both the student's name and their score. The `if` statement filters which names to announce, and the loop handles the math for every student automatically.

### Common Mistakes

1.  **Confusing Assignment with Comparison:** Inside a conditional, use `==` to check equality. Using a single `=` inside an `if` statement (e.g., `if x = 5`) will cause a syntax error in Python because it tries to *set* a value rather than *check* one.
2.  **Modifying a Tuple:** Remember that tuples are read-only. If you need to change data, use a list.
3.  **Wrong Indentation:** Python uses indentation to define code blocks. If your code inside a loop or conditional is not indented, it will run once after the statement rather than repeatedly or conditionally.
4.  **Unhashable Keys:** In a dictionary, keys must be unique and "hashable." You generally cannot use a list as a key, but you can use a tuple.

## Key Takeaways

*   **Lists** are ordered and changeable; use them for collections of items you need to modify.
*   **Tuples** are ordered but fixed; use them for data that should stay constant.
*   **Sets** remove duplicates automatically; use them for uniqueness checks.
*   **Dictionaries** store key-value pairs; use them for fast lookups by label.
*   **Conditionals (`if`)** make decisions; **loops (`for`)** iterate over collections.
*   Always use `==` for comparison and be careful with indentation to control code blocks.
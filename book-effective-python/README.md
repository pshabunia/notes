# Effective Python
by Brett Slatkin

[Chapter 1. Pythonic Thinking](1_pythonic_thinking.md)
* Item 1: Know Which Version of Python You’re Using
* Item 2: Follow the PEP 8 Style Guide
* Item 3: Know the Differences Between bytes and str
* Item 4: Prefer Interpolated F-Strings Over C-style Format Strings and str.format
* Item 5: Write Helper Functions Instead of Complex Expressions
* Item 6: Prefer Multiple Assignment Unpacking Over Indexing
* Item 7: Prefer `enumerate` Over `range`
* Item 8: Use `zip` to Process Iterators in Parallel
* Item 9: Avoid else Blocks After for and while Loops
* Item 10: Prevent Repetition with Assignment Expressions

[Chapter 2. Lists and Dictionaries](2_lists_and_dicts.md)
* Item 11: Know How to Slice Sequences
* Item 12: Avoid Striding and Slicing in a Single Expression
* Item 13: Prefer Catch-All Unpacking Over Slicing
* Item 14: Sort by Complex Criteria Using the key Parameter
* Item 15: Be Cautious When Relying on dict Insertion Ordering
* Item 16: Prefer `get` Over `in` and KeyError to Handle Missing Dictionary Keys
* Item 17: Prefer `defaultdict` Over `setdefault` to Handle Missing Items in Internal State
* Item 18: Know How to Construct Key-Dependent Default Values with `__missing__`

[Chapter 3. Functions](3_functions.md)
* Item 19: Never Unpack More Than Three Variables When Functions Return Multiple Values
* Item 20: Prefer Raising Exceptions to Returning `None`
* Item 21: Know How Closures Interact with Variable Scope
* Item 22: Reduce Visual Noise with Variable Positional Arguments (varargs)
* Item 23: Provide Optional Behavior with Keyword Arguments (kwargs)
* Item 24: Use `None` and Docstrings to Specify Dynamic Default Arguments
* Item 25: Enforce Clarity with Keyword-Only and Positional-Only Arguments
* Item 26: Define Function Decorators with `functools.wraps`

[Chapter 4. Comprehensions and Generators](4_comprehensions_and_generators.md)
* Item 27: Use Comprehensions Instead of `map` and `filter`
* Item 28: Avoid More Than Two Control Subexpressions in Comprehensions
* Item 29: Avoid Repeated Work in Comprehensions by Using Assignment Expressions
* Item 30: Consider Generators Instead of Returning Lists
* Item 31: Be Defensive When Iterating Over Arguments
* Item 32: Consider Generator Expressions for Large List Comprehensions
* Item 33: Compose Multiple Generators with `yield from`
* Item 34: Avoid Injecting Data into Generators with `send`
* Item 35: Avoid Causing State Transitions in Generators with `throw`
* Item 36: Consider `itertools` for Working with Iterators and Generators

[Chapter 5. Classes and Interfaces](5_classes_and_interfaces.md)
* Item 37: Compose Classes Instead of Nesting Many Levels of Built-in Types
* Item 38: Accept Functions Instead of Classes for Simple Interfaces
* Item 39: Use `@classmethod` Polymorphism to Construct Objects Generically
* Item 40: Initialize Parent Classes with `super`
* Item 41: Consider Composing Functionality with Mix-in Classes
* Item 42: Prefer Public Attributes Over Private Ones
* Item 43: Inherit from `collections.abc` for Custom Container Types

[Chapter 6. Metaclasses and Attributes](6_metaclasses_and_attributes.md)
TBD

[Chapter 7. Concurrency and Parallelism](7_concurrency_and_parallelism.md)
TBD


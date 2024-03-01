# Chapter 3. Functions

## Item 19: Never Unpack More Than Three Variables When Functions Return Multiple Values

```python
def get_avg_ratio(numbers):
    average = sum(numbers) / len(numbers)
    scaled = [x / average for x in numbers]
    scaled.sort(reverse=True)
    return scaled
longest, *middle, shortest = get_avg_ratio(lengths)
```

* Functions return multiple values in a tuple and the caller can unpack them.
* Multiple return values can be unpacked by catch-all starred expressions.
* Unpacking into four or more variables is error prone and should be avoided; instead, use class or namedtuple instance.

## Item 20: Prefer Raising Exceptions to Returning `None`

(two separate topics: `None` is problematic output. Exception is design concept)

Misinterpretation of a False-equivalent return value is a common mistake in Python code 
when None has special meaning. This is why returning None from a function like careful_divide is error prone. 
There are two ways to reduce the chance of such errors.

1. Return tuple (status, value) - requires if-handler
2. Raise exception - requires try-handler, plus documentation and type annotations

## Item 21: Know How Closures Interact with Variable Scope


* Python supports **closures** - a functions that refer to variables from the scope in which they were defined.
* Functions are **first-class objects** in Python, which means you can refer to them directly, assign them to variables, 
  pass them as arguments to other functions, compare them in expressions and if statements, and so on.
* Python has specific rules for comparing sequences (including tuples). It first compares items at index zero; 
  then, if those are equal, it compares items at index one; if they are still equal, it compares items at index two, and so on.
* When you reference a variable in an expression, the Python interpreter traverses the scope to resolve the reference in this order:
  1. The current function’s scope.
  2. Any enclosing scopes (such as other containing functions).
  3. The scope of the module that contains the code (also called the global scope).
  4. The built-in scope (that contains functions like len and str).
  5. And if none of these places has defined a variable with the referenced
name, then a NameError exception is raised
* Assigning a value to a variable works differently. If the variable is already defined in the current scope, 
  it will just take on the new value. If the variable doesn’t exist in the current scope, Python treats the assignment 
  as a variable definition. Critically, the scope of the newly defined variable is the function that contains the assignment.
* The `nonlocal` statement is used to indicate that scope traversal should happen upon assignment for a specific variable name. 
The only limit is that nonlocal won’t traverse up to the module-level scope (to avoid polluting globals).
* The `global` statement indicates that a variable’s assignment should go directly into the module scope.
* Global is anti-pattern. Nonlocal is code smell.
* When your usage of nonlocal starts getting complicated, it’s better to wrap your state in a helper class.

```python
class Sorter:
    def __init__(self, group):
        self.group = group
        self.found = False
    def __call__(self, x):
        if x in self.group:
            self.found = True
            return (0, x)
        return (1, x)
sorter = Sorter(group)
numbers.sort(key=sorter)
assert sorter.found is True
```

Summary:
* Closure functions can refer to variables from any of the scopes in which they were defined.
* By default, closures can’t affect enclosing scopes by assigning variables.
* Use the nonlocal statement to indicate when a closure can modify a variable in its enclosing scopes.
* Avoid using nonlocal statements for anything beyond simple functions.


## Item 22: Reduce Visual Noise with Variable Positional Arguments (varargs)

* Functions can accept a variable number of positional arguments by
using *args in the def statement.
* You can use the items from a sequence as the positional arguments
for a function with the * operator.
* Using the varargs operator with a generator may cause a program to run
out of memory and crash.
* Adding new positional parameters to functions that accept *args
can introduce hard-to-detect bugs.

## Item 23: Provide Optional Behavior with Keyword Arguments (kwargs)

* Function arguments can be specified by position or by keyword.
* Keywords make it clear what the purpose of each argument is when
it would be confusing with only positional arguments.
* Keyword arguments with default values make it easy to add new
behaviors to a function without needing to migrate all existing
callers.
* _Optional keyword arguments should always be passed by keyword
instead of by position. (??)_
  * note: seems fine to me if using readable variables as positional arguments


## Item 24: Use `None` and Docstrings to Specify Dynamic Default Arguments

* A default argument value is evaluated only once: during function
definition at module load time. This can cause odd behaviors for
dynamic values (like {}, [], or datetime.now()).
* Use None as the default value for any keyword argument that has a
dynamic value. Document the actual default behavior in the function’s docstring.
* Using None to represent keyword argument default values also
works correctly with type annotations.

also: https://web.archive.org/web/20200221224620id_/http://effbot.org/zone/default-values.htm

## Item 25: Enforce Clarity with Keyword-Only and Positional-Only Arguments

```python
def safe_division(numerator, denominator, 
                  /, # Position-only before
                  ndigits=10,
                  *, # Keyword-only after
                  ignore_overflow=False,
                  ignore_zero_division=False):
    try:
      return round(numerator / denominator, ndigits)
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
      if ignore_zero_division:
          return float('inf')
      else:
          raise
```

* Keyword-only arguments force callers to supply certain arguments
by keyword (instead of by position), which makes the intention of a
function call clearer. Keyword-only arguments are defined after a
single `*` in the argument list.
* Positional-only arguments ensure that callers can’t supply
certain parameters using keywords, which helps reduce coupling.
Positional-only arguments are defined before a single `/` in the argument list.
* Parameters between the `/` and `*` characters in the argument list
may be supplied by position or keyword, which is the default for
Python parameters.


## Item 26: Define Function Decorators with `functools.wraps`

* Decorators in Python are syntax to allow one function to modify
another function at runtime.
* Using decorators can cause strange behaviors in tools that do introspection, 
  such as debuggers and serializers
* Use the `functools.wraps` decorator (as `@wraps(func)`) when you define 
your own decorators to avoid issues.

P.S. Check out `help(functools)`
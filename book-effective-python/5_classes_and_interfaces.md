# Chapter 5. Classes and Interfaces

## Item 37: Compose Classes Instead of Nesting Many Levels of Built-in Types

```python
import collections

help(collections.namedtuple)
# namedtuple(typename, field_names, *, rename=False, defaults=None, module=None)
#     Returns a new subclass of tuple with named fields.
# 
#     >>> Point = namedtuple('Point', ['x', 'y'])
#     >>> Point.__doc__                   # docstring for the new class
#     'Point(x, y)'
#     >>> p = Point(11, y=22)             # instantiate with positional args or keywords
#     >>> p[0] + p[1]                     # indexable like a plain tuple
#     33
#     >>> x, y = p                        # unpack like a regular tuple
#     >>> x, y
#     (11, 22)
#     >>> p.x + p.y                       # fields also accessible by name
#     33
#     >>> d = p._asdict()                 # convert to a dictionary
#     >>> d['x']
#     11
#     >>> Point(**d)                      # convert from a dictionary
#     Point(x=11, y=22)
#     >>> p._replace(x=100)               # _replace() is like str.replace() but targets named fields

import dataclasses

help(dataclasses.dataclass)
# dataclass(cls=None, /, *, init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False, match_args=True, kw_only=False, slots=False)
#     Returns the same class as was passed in, with dunder methods
#     added based on the fields defined in the class.
# 
#     Examines PEP 526 __annotations__ to determine fields.
# 
#     If init is true, an __init__() method is added to the class. If
#     repr is true, a __repr__() method is added. If order is true, rich
#     comparison dunder methods are added. If unsafe_hash is true, a
#     __hash__() method function is added. If frozen is true, fields may
#     not be assigned to after instance creation. If match_args is true,
#     the __match_args__ tuple is added. If kw_only is true, then by
#     default all fields are keyword-only. If slots is true, an
#    __slots__ attribute is added.
```

https://docs.python.org/3/library/dataclasses.html

* Avoid making dictionaries with values that are dictionaries, long
  tuples, or complex nestings of other built-in types.
* Use `namedtuple` for lightweight, immutable data containers before
  you need the flexibility of a full class. Consider `dataclasses`.
* Move your bookkeeping code to using multiple classes when your
  internal state dictionaries get complicated.

## Item 38: Accept Functions Instead of Classes for Simple Interfaces

* Instead of defining and instantiating classes, you can often simply
  use functions for simple interfaces between components in Python.
* References to functions and methods in Python are first class,
  meaning they can be used in expressions (like any other type).
* The `__call__` special method enables instances of a class to be
  called like plain Python functions.
* When you need a function to maintain state, consider defining a
  class that provides the `__call__` method instead of defining a stateful closure.

## Item 39: Use `@classmethod` Polymorphism to Construct Objects Generically

* Python only supports a single constructor per class: the `__init__` method.
* Use `@classmethod` to define alternative constructors for your classes.
* Use class method polymorphism to provide generic ways to build and connect
  many concrete subclasses.

## Item 40: Initialize Parent Classes with `super`

* Python’s standard method resolution order (MRO) solves the problems
  of superclass initialization order and diamond inheritance.
* Use the `super` built-in function with zero arguments to initialize
  parent classes.

The only time you should provide parameters to super is in situations where
you need to access the specific functionality of a superclass’s implementation
from a child class (e.g., to wrap or reuse functionality).

https://snarky.ca/unravelling-pythons-classes/

## Item 41: Consider Composing Functionality with Mix-in Classes

```python
class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output

    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value
```

Summary

* Avoid using multiple inheritance with instance attributes and
  `__init__` if mix-in classes can achieve the same outcome.
* Use pluggable behaviors at the instance level to provide per-class
  customization when mix-in classes may require it.
* Mix-ins can include instance methods or class methods, depending on your needs.
* Compose mix-ins to create complex functionality from simple behaviors.

## Item 42: Prefer Public Attributes Over Private Ones

* Private attributes aren’t rigorously enforced by the Python compiler.
* Plan from the beginning to allow subclasses to do more with your
  internal APIs and attributes instead of choosing to lock them out.
* Use documentation of protected fields to guide subclasses instead of
  trying to force access control with private attributes.
* Only consider using private attributes to avoid naming conflicts
  with subclasses that are out of your control.

> https://docs.python.org/3/reference/expressions.html#atom-identifiers
**Private name mangling**: When an identifier that textually occurs in a class definition begins with two or more
> underscore characters and does not end in two or more underscores, it is considered a private name of that class.
> Private names are transformed to a longer form before code is generated for them. The transformation inserts the class
> name, with leading underscores removed and a single underscore inserted, in front of the name. For example, the
> identifier __spam occurring in a class named Ham will be transformed to _Ham__spam. This transformation is independent
> of the syntactical context in which the identifier is used. If the transformed name is extremely long (longer than 255
> characters), implementation defined truncation may happen. If the class name consists only of underscores, no
> transformation is done.

## Item 43: Inherit from `collections.abc` for Custom Container Types

* Inherit directly from Python’s container types (like list or dict) for simple use cases.
* Beware of the large number of methods required to implement custom container types correctly.
* Have your custom container types inherit from the interfaces defined in `collections.abc` to ensure that your classes
  match required interfaces and behaviors.

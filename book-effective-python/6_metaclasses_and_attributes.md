# Chapter 6. Metaclasses and Attributes

## Item 44: Use Plain Attributes Instead of Setter and Getter Methods

* Define new class interfaces using simple public attributes and avoid defining setter and getter methods.
* Use `@property` to define special behavior when attributes are accessed on your objects, if necessary.
* Follow the rule of least surprise and avoid odd side effects in your `@property` methods.
* Ensure that `@property` methods are fast; for slow or complex work — especially involving I/O or causing side
  effects — use normal methods instead.

## Item 45: Consider `@property` Instead of Refactoring Attributes



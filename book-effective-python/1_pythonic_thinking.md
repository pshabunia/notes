# Chapter 1. Pythonic Thinking

## Item 1: Know Which Version of Python You’re Using

## Item 2: Follow the PEP 8 Style Guide

- Use pylint
- Avoid single-line if statements, for and while loops, and except
  compound statements. Spread these over multiple lines for
  clarity.
- If you must do relative imports, use the explicit syntax `from . import foo`.
- Imports should be in sections in the following order: standard
  library modules, third-party modules, your own modules. Each
  subsection should have imports in alphabetical order.

## Item 3: Know the Differences Between bytes and str

```python
import locale
print(locale.getpreferredencoding())
```

## Item 4: Prefer Interpolated F-Strings Over C-style Format Strings and str.format

```python
f_string = f'{key:<10} = {value:.2f}'
c_tuple = '%-10s = %.2f' % (key, value)
str_args = '{:<10} = {:.2f}'.format(key, value)
str_kw = '{key:<10} = {value:.2f}'.format(key=key, value=value)
c_dict = '%(key)-10s = %(value).2f' % {'key': key, 'value': value}
```

## Item 5: Write Helper Functions Instead of Complex Expressions

## Item 6: Prefer Multiple Assignment Unpacking Over Indexing

```python
item = ('Peanut butter', 'Jelly')
first, second = item  # Unpacking
print(first, 'and', second)
```

```python
favorite_snacks = {
    'salty': ('pretzels', 100),
    'sweet': ('cookies', 180),
    'veggie': ('carrots', 20),
}
((type1, (name1, cals1)),
 (type2, (name2, cals2)),
 (type3, (name3, cals3))) = favorite_snacks.items()
```

```python
def bubble_sort(a):
    for _ in range(len(a)):
        for i in range(1, len(a)):
            if a[i] < a[i - 1]:
                a[i - 1], a[i] = a[i], a[i - 1]  # Swap
```

## Item 7: Prefer `enumerate` Over `range`

```python
for i, flavor in enumerate(flavor_list, 1):
    print(f'{i}: {flavor}')
```

## Item 8: Use `zip` to Process Iterators in Parallel

- zip keeps yielding tuples until any one of the wrapped iterators is exhausted. Its output is as long as its shortest
  input.
- use itertools

```python
import itertools

for name, count in itertools.zip_longest(names, counts):
    print(f'{name}: {count}')
```

## Item 9: Avoid else Blocks After for and while Loops

## Item 10: Prevent Repetition with Assignment Expressions

```python
if (count := fresh_fruit.get('banana', 0)) >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
elif (count := fresh_fruit.get('apple', 0)) >= 4:
    to_enjoy = make_cider(count)
elif count := fresh_fruit.get('lemon', 0):
    to_enjoy = make_lemonade(count)
else:
    to_enjoy = 'Nothing'
```

The walrus operator obviates the need for the *loop-and-a-half idiom* 
by allowing the fresh_fruit variable to be reassigned and then conditionally
evaluated each time through the while loop.

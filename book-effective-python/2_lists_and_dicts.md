# Chapter 2. Lists and Dictionaries

Item 11: Know How to Slice Sequences

```python
a[:]  # ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
a[:5]  # ['a', 'b', 'c', 'd', 'e']
a[:-1]  # ['a', 'b', 'c', 'd', 'e', 'f', 'g']
a[4:]  # ['e', 'f', 'g', 'h']
a[-3:]  # ['f', 'g', 'h']
a[2:5]  # ['c', 'd', 'e']
a[2:-1]  # ['c', 'd', 'e', 'f', 'g']
a[-3:-1]  # ['f', 'g']
```

Slicing deals properly with start and end indexes that are beyond the
boundaries of a list by silently omitting missing items.

The result of slicing a list is a whole new list. References to the
objects from the original list are maintained. Modifying the result of
slicing won’t affect the original list

When used in assignments, slices replace the specified range in the original list.
Unlike unpacking assignments, the lengths of slice assignments don’t need to be the
same. The values before and after the assigned slice will be preserved.

If you leave out both the start and the end indexes when slicing, you
end up with a copy of the original list.

```python
b = a[:]
assert b == a and b is not a
```

If you assign to a slice with no start or end indexes, you replace the
entire contents of the list with a copy of what’s referenced (instead of
allocating a new list).

```python
b = a
print('Before a', a)
print('Before b', b)
a[:] = [101, 102, 103]
assert a is b
```

Item 12: Avoid Striding and Slicing in a Single Expression

Python has special syntax for the stride of a slice in
the form `somelist[start:end:stride]`. This lets you take every nth item
when slicing a sequence.

```python
odds = x[::2]
evens = x[1::2]
```

```python
a = list(range(0, 5))
n = 3
shards = [a[i::n] for i in range(0, n)]
assert shards == [[0, 3], [1, 4], [2]]
```

- Specifying start, end, and stride in a slice can be extremely confusing.
- Prefer using positive stride values in slices without start or end
  indexes. Avoid negative stride values if possible.
- Avoid using start, end, and stride together in a single slice. If you
  need all three parameters, consider doing two assignments (one
  to stride and another to slice) or using islice from the itertools
  built-in module.

Item 13: Prefer Catch-All Unpacking Over Slicing

Python also supports catch-all
unpacking through a starred expression. This syntax allows one part
of the unpacking assignment to receive all values that didn’t match
any other part of the unpacking pattern.

```python
import csv

with open('file.csv') as file:
    header, *rows = csv.reader(file)
```

(!!) Keep in mind, however, that because a starred expression is always
turned into a list, unpacking an iterator also risks the potential of using up
all of the memory on your computer and causing your program to crash.

## Item 14: Sort by Complex Criteria Using the key Parameter

The list built-in type provides a sort method for ordering the items
in a list instance based on a variety of criteria. By default, sort will
order a list’s contents by the natural ascending order of the items.

The sort method doesn’t work for objects unless they define a natural
ordering using special methods, which is uncommon.

Often there’s an attribute on the object that you’d like to use for sorting.
To support this use case, the sort method accepts a key parameter that’s expected to be a function.

```python
tools.sort(key=lambda x: x.name)
places.sort(key=lambda x: x.lower())
```

Returning a tuple from the key function allows you to combine multiple
sorting criteria together. The unary minus operator can be
used to reverse individual sort orders for types that allow it.

```python
saw = (5, 'circular saw')
jackhammer = (40, 'jackhammer')
assert not (jackhammer < saw)  # Matches expectations

drill = (4, 'drill')
sander = (4, 'sander')
assert drill[0] == sander[0]  # Same weight
assert drill[1] < sander[1]  # Alphabetically less
assert drill < sander  # Thus, drill comes first

power_tools = [
    Tool('drill', 4),
    Tool('circular saw', 5),
    Tool('jackhammer', 40),
    Tool('sander', 4),
]
power_tools.sort(key=lambda x: (x.weight, x.name))
power_tools.sort(key=lambda x: (x.weight, x.name), reverse=True)
power_tools.sort(key=lambda x: (-x.weight, x.name))
```

Python provides a *stable* sorting algorithm.
The sort method of the list type will preserve the order of the input
list when the key function returns values that are equal to each
other. This means that I can call sort multiple times on the same
list to combine different criteria together.

```python
power_tools.sort(key=lambda x: x.name, reverse=True)  # Name descending
power_tools.sort(key=lambda x: -x.weight)  # Weight descending
```

You can combine as many different types of sorting criteria as you’d like
in any direction, respectively. You just need to make sure that you execute
the sorts in the opposite sequence of what you want the final list to contain.

## Item 15: Be Cautious When Relying on dict Insertion Ordering

- Since Python 3.7, you can rely on the fact that iterating a dict
  instance’s contents will occur in the same order in which the keys
  were initially added.
- Python makes it easy to define objects that act like dictionaries but
  that aren’t dict instances. For these types, you can’t assume that
  insertion ordering will be preserved.
  -There are three ways to be careful about dictionary-like classes:
  Write code that doesn’t rely on insertion ordering, explicitly check
  for the dict type at runtime, or require dict values using type anno-
  tations and static analysis.

## Item 16: Prefer `get` Over `in` and KeyError to Handle Missing Dictionary Keys

```python
count = counters.get(key, 0)
counters[key] = count + 1
```

(!) If you’re maintaining dictionaries of counters like this, it’s worth considering
the Counter class from the collections built-in module, which provides
most of the facilities you are likely to need.

```python
if key in votes:
    names = votes[key]
else:
    votes[key] = names = []
names.append(who)
# vs
if (names := votes.get(key)) is None:
    votes[key] = names = []
names.append(who)
# vs
names = votes.setdefault(key, [])
names.append(who)
```

```python
data = {}
key = 'foo'
value = []
data.setdefault(key, value)
print(data)  # {'foo': []}
value.append('hello')
print(data)  # {'foo': ['hello']}
```

* There are four common ways to detect and handle missing keys
  in dictionaries: using in expressions, KeyError exceptions, the get
  method, and the setdefault method.
* The get method is best for dictionaries that contain basic types
  like counters, and it is preferable along with assignment expressions
  when creating dictionary values has a high cost or may raise exceptions.
* When the setdefault method of dict seems like the best fit for your
  problem, you should consider using defaultdict instead.

## Item 17: Prefer `defaultdict` Over `setdefault` to Handle Missing Items in Internal State

```python
visits.setdefault('France', set()).add('Arles')  # Short
# or 
if (japan := visits.get('Japan')) is None:  # Long
    visits['Japan'] = japan = set()
japan.add('Kyoto')
```

```python
data = defaultdict(set)
data[country].add(city)
```

* If you’re creating a dictionary to manage an arbitrary set of potential
  keys, then you should prefer using a defaultdict instance from
  the collections built-in module if it suits your problem.
* If a dictionary of arbitrary keys is passed to you, and you don’t control
  its creation, then you should prefer the get method to access its
  items. However, it’s worth considering using the setdefault method
  for the few situations in which it leads to shorter code.

## Item 18: Know How to Construct Key-Dependent Default Values with `__missing__`

```python
class Pictures(dict):
    def __missing__(self, key):
        value = open_picture(key)
        self[key] = value
        return value
```

* The setdefault method of dict is a bad fit when creating the default
  value has high computational cost or may raise exceptions.
* The function passed to defaultdict must not require any arguments,
  which makes it impossible to have the default value depend
  on the key being accessed.
* You can define your own dict subclass with a `__missing__` method
  in order to construct default values that must know which key was
  being accessed.

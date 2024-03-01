# Chapter 4. Comprehensions and Generators

## Item 27: Use Comprehensions Instead of `map` and `filter`

* List comprehensions are clearer than the map and filter built-in
  functions because they don’t require lambda expressions.
* List comprehensions allow you to easily skip items from the input
  list, a behavior that map doesn’t support without help from filter.
* Dictionaries and sets may also be created using comprehensions.

```python
# Dict comprehension
d = dict(one=1, two=2, three=3)
d_ = {f"{k}_": v for k, v in d.items()}
# >> {'one_': 1, 'two_': 2, 'three_': 3}

# Set comprehension
s_ = {v // 2 for v in range(0, 10)}
# >> {0, 1, 2, 3, 4}
```

## Item 28: Avoid More Than Two Control Subexpressions in Comprehensions

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
# >> [1, 2, 3, 4, 5, 6, 7, 8, 9]
squared = [[x ** 2 for x in row] for row in matrix]
# >> [[1, 4, 9], [16, 25, 36], [49, 64, 81]]
```

The rule of thumb is to avoid using more than two control subexpressions in a comprehension.
This could be two conditions, two loops, or one condition and one loop.
As soon as it gets more complicated than that, you should use normal `if` and `for` statements
and write a helper function.

## Item 29: Avoid Repeated Work in Comprehensions by Using Assignment Expressions

```python
stock = {
    'nails': 125,
    'screws': 35,
    'wingnuts': 8,
    'washers': 24,
}
order = ['screws', 'wingnuts', 'clips']


def get_batches(count, size):
    return count // size


found1 = {name: get_batches(stock.get(name, 0), 8)
          for name in order
          if get_batches(stock.get(name, 0), 8)}
# better with walrus
found2 = {name: batches for name in order
          if (batches := get_batches(stock.get(name, 0), 8))}
```

If a comprehension uses the walrus operator in the value part of the
comprehension and doesn’t have a condition, it’ll leak the loop variable
into the containing scope.

```python
half = [(last := count // 2) for count in stock.values()]
print(f'Last item of {half} is {last}')
# >> Last item of [62, 17, 4, 12] is 12
```

This leakage of the loop variable is similar to what happens with a
normal for loop.

* Assignment expressions make it possible for comprehensions and
  generator expressions to reuse the value from one condition elsewhere
  in the same comprehension, which can improve readability and performance.
* Although it’s possible to use an assignment expression outside
  a comprehension or generator expression’s condition, you should
  avoid doing so.

## Item 30: Consider Generators Instead of Returning Lists

When called, a generator function does not actually run but instead
immediately returns an iterator. With each call to the next built-in
function, the iterator advances the generator to its next yield expression.
Each value passed to yield by the generator is returned by the iterator to the caller

* Using generators can be clearer than the alternative of having a
function return a list of accumulated results.
* The iterator returned by a generator produces the set of values
passed to yield expressions within the generator function’s body.
* Generators can produce a sequence of outputs for arbitrarily large
inputs because their working memory doesn’t include all inputs and
outputs.
* (!!) Similar to iterator, the output of generator can be traversed only once

## Item 31: Be Defensive When Iterating Over Arguments

Confusingly, you also won’t get errors when you iterate over an
already exhausted iterator. for loops, the list constructor, and many
other functions throughout the Python standard library expect the
StopIteration exception to be raised during normal operation. These
functions can’t tell the difference between an iterator that has no output 
and an iterator that had output and is now exhausted.

To solve this problem, you can explicitly convert iterator into a list.
But it can be heavy on memory. One way around this is to accept a function
that returns a new iterator each time it’s called. A better way to achieve 
the same result is to provide a new container class that implements the 
iterator protocol.

The iterator protocol is how Python for loops and related expressions
traverse the contents of a container type. When Python sees a statement 
like `for x in foo`, it actually calls `iter(foo)`. The `iter` built-in
function calls the `foo.__iter__` special method in turn. The `__iter__`
method must return an iterator object (which itself implements the
`__next__` special method). Then, the for loop repeatedly calls the
`next` built-in function on the iterator object until it’s exhausted 
(indicated by raising a `StopIteration` exception).

It sounds complicated, but practically speaking, you can achieve all of
this behavior for your classes by implementing the `__iter__` method
as a generator.

The protocol states that when an iterator is passed to the iter built-in function, 
`iter` returns the iterator itself. In contrast, when a container type is passed to `iter`, 
a new iterator object is returned each time. Thus, you can test an input value for this 
behavior and raise a `TypeError` to reject arguments that can’t be repeatedly iterated over:

```python
def normalize_defensive(numbers):
    if iter(numbers) is numbers: # An iterator -- bad!
        raise TypeError('Must supply a container')
```
    
Alternatively, the `collections.abc` built-in module defines an `Iterator`
class that can be used in an isinstance test to recognize the potential problem:

```python
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator):  # Another way to check
        raise TypeError('Must supply a container')
```

Summary:
* Beware of functions and methods that iterate over input arguments multiple times.
  If these arguments are iterators, you may see strange behavior and missing values.
* Python’s iterator protocol defines how containers and iterators interact with the `iter`
  and `next` built-in functions, for loops, and related expressions.
* You can easily define your own iterable container type by implementing the `__iter__`
  method as a generator.
* You can detect that a value is an iterator (instead of a container) if calling `iter` on it 
  produces the same value as what you passed in. Alternatively, you can use the `isinstance` 
  built-in function along with the `collections.abc.Iterator` class.


## Item 32: Consider Generator Expressions for Large List Comprehensions

```python
l : list = ... # very long list
l1 = (x*2 for x in l)
l2 = (x*x for x in l1)
# etc
```

* List comprehensions can cause problems for large inputs by using
too much memory.
* Generator expressions avoid memory issues by producing outputs
one at a time as iterators.
* Generator expressions can be composed by passing the iterator from
one generator expression into the for subexpression of another.
* Generator expressions execute very quickly when chained together
and are memory efficient.


## Item 33: Compose Multiple Generators with `yield from`

```python
def child():
    for i in range(1_000_000):
        yield i

def fast():
    yield from child()
```

* The `yield from` expression allows you to compose multiple nested
generators together into a single combined generator.
* `yield from` provides better performance than manually iterating
nested generators and yielding their outputs.


## Item 34: Avoid Injecting Data into Generators with `send`

* https://dabeaz.com/coroutines/Coroutines.pdf
* https://peps.python.org/pep-0342

* The `send` method can be used to inject data into a generator by giving
the `yield` expression a value that can be assigned to a variable.
* Using `send` with `yield from` expressions may cause surprising
behavior, such as `None` values appearing at unexpected times in the
generator output.
* Providing an input iterator to a set of composed generators is a better
approach than using the `send` method, which should be avoided.
  * IOW, distinguish generators and coroutines. Don't mix the two concepts together.


## Item 35: Avoid Causing State Transitions in Generators with `throw`

```python
g = (i for i in range(10))
print(g)
print(next(g))
print(next(g))
```
* The `throw` method can be used to re-raise exceptions within
generators at the position of the most recently executed yield
expression.
* Using `throw` harms readability because it requires additional nesting
and boilerplate in order to raise and catch exceptions.
* A better way to provide exceptional behavior in generators is to use
a class that implements the `__iter__` method along with methods to
cause exceptional state transitions.


## Item 36: Consider `itertools` for Working with Iterators and Generators

* The itertools functions fall into three main categories for working 
  with iterators and generators: linking iterators together, filtering items
  they output, and producing combinations of items.
* There are more advanced functions, additional parameters, and
useful recipes available in the documentation at `help(itertools)`.
* See examples at https://www.geeksforgeeks.org/python-itertools


```python
Infinite iterators:
    count(start=0, step=1) --> start, start+step, start+2*step, ...
    cycle(p) --> p0, p1, ... plast, p0, p1, ...
    repeat(elem [,n]) --> elem, elem, elem, ... endlessly or up to n times

Iterators terminating on the shortest input sequence:
    accumulate(p[, func]) --> p0, p0+p1, p0+p1+p2
    chain(p, q, ...) --> p0, p1, ... plast, q0, q1, ...
    chain.from_iterable([p, q, ...]) --> p0, p1, ... plast, q0, q1, ...
    compress(data, selectors) --> (d[0] if s[0]), (d[1] if s[1]), ...
    dropwhile(pred, seq) --> seq[n], seq[n+1], starting when pred fails
    groupby(iterable[, keyfunc]) --> sub-iterators grouped by value of keyfunc(v)
    filterfalse(pred, seq) --> elements of seq where pred(elem) is False
    islice(seq, [start,] stop [, step]) --> elements from
           seq[start:stop:step]
    pairwise(s) --> (s[0],s[1]), (s[1],s[2]), (s[2], s[3]), ...
    starmap(fun, seq) --> fun(*seq[0]), fun(*seq[1]), ...
    tee(it, n=2) --> (it1, it2 , ... itn) splits one iterator into n
    takewhile(pred, seq) --> seq[0], seq[1], until pred fails
    zip_longest(p, q, ...) --> (p[0], q[0]), (p[1], q[1]), ...

Combinatoric generators:
    product(p, q, ... [repeat=1]) --> cartesian product
    permutations(p[, r])
    combinations(p, r)
    combinations_with_replacement(p, r)
```

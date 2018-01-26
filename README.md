# deque
This module is a pure Python implementation of a double-ended queue, or deque (pronounced "deck"), optimized for operations near either end and primarily [coded live on video](https://www.youtube.com/playlist?list=PLQ7bGgvf9FtFQ_E4g6Th2F45oY8qUzp65). The API follows [`collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque) for the most part, except for the note below. However, there are several convenience features that [`collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque) does not have.

## API divergence warning
As of commit #15, this module does not exactly duplicate the standard [`collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque) API. The divergence is in the addition operator, `+`, e.g. `a_deque + other_deque`. This performs concatenation in [`collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque) and several other Python data types including lists, strings, and tuples. This module, however, uses the `+` operator for vector (elementwise) addition, e.g. `deque.DQ([1, 2, 3]) + deque.DQ([4, 5, 6]) == deque.DQ([5, 7, 9])`.

## Extra features
* O(1) reversal time with the `reverse` method. The standard [`collections.deque.reverse`](https://docs.python.org/3/library/collections.html#collections.deque.reverse) is O(n) for _n_ elements, while this module doesn't particularly care which way is up and will simply do a handstand to look at your data in the other direction rather than shuffling values around.
* The `remove` method has an additional optional argument `count` for the number of removals to perform, defaulting to 1. When `count` is positive, `count` items are removed, starting from the beginning of the deque. When `count` is negative, it starts from the end of the deque and proceeds backwards. When `count` is 0, all matching items are removed.
* The `replace` method works similarly to `str.replace`, with an optional `count` argument for the number of items to replace. A positive `count` starts from the beginning, a negative `count` starts from the end, and a `count` of 0 replaces all matching items.
* Concatenation is now performed via instance calls (the `__call__` special method). This is a replacement for the standard [`collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque) concatenation that would use `+` (which performs vector addition in this module). It sends every passed object to `extend` and then returns the deque back, for convenient chaining. This method has the following aliases, but they are not connected to call syntax and must be manually invoked: `plus`, `p`, `concatenate`, `concat`, `c`.

For example:

    d = deque.DQ('123')('456', '789')
    d == deque.DQ('123456789') -> True

Note that the first deque in the chain is mutated, so this:

    d = save = deque.DQ('123')
    e = d('456')

is equivalent to this:

    d = save = deque.DQ('123')
    d('456')
    e = d

resulting either way in this:

    save is d is e -> True
    d == e == deque.DQ('123456') -> True
    
* Vector arithmetic with every operator - `+`, `-`, `*`, `/`, `//`, `%`, `divmod()`, `pow()`, `**`, `<<`, `>>`, `&`, `^`, `|` (see the [Python data model documentation](https://docs.python.org/3/reference/datamodel.html#emulating-numeric-types) for details).

For example:

    deque.DQ([1, 2, 3]) + (0.5,)*3
    -> deque.DQ([1.5, 2.5, 3.5])
    'abc' * deque.DQ([1, 2, 3])
    -> deque.DQ(['a', 'bb', 'ccc'])
    deque.DQ([1]) / (3,)
    -> deque.DQ([0.3333333333333333])
    (1,) / deque.DQ([3])
    -> deque.DQ([0.3333333333333333])
    
* Matrix multiplication via the `@` operator.

For example:

    deque.DQ([[1, 2, 3], [4, 5, 6]]) @ deque.DQ([[11, 12, 13, 14], [21, 22, 23, 24], [31, 32, 33, 34]])
    -> deque.DQ([deque.DQ([146, 152, 158, 164]), deque.DQ([335, 350, 365, 380])])

Row vectors and column vectors are promoted to 2D matrices and then the result is demoted appropriately. Note that this means there is no attempt to preserve any references with augmented assignment, e.g. `matrix1 @= matrix2`, as `matrix1` could start out as a row vector (like a deque or a list) and become a scalar (like an integer or float).

For example:

    deque.DQ([1, 2, 3]) @ deque.DQ([4, 5, 6]) -> 32

When the maximum dimension of both matrices exceeds the threshold set by the `invoke_mp` attribute (default 150) of the left operand (or right operand, if the left operand is of a different but supported class, like a list), the task will be distributed among all available CPU cores. Smaller problem sets are more quickly solved without the overhead of multiprocessing. This threshold is suited to square matrices of small integers, so it's advisable to reduce the threshold for small matrices containing large values.

## Internal methods
There are a few methods primarily intended for internal use, but which are still accessible. Use the built-in function `help()` for documentation.

* `distance_to_index`
* `norm_index`
* `_neighbors`
* `_slice`
* `_getsetdel`
* `_getsetdel`
* `_remove_node`
* `_insert_after`
* `_insert_before`
* `_remove_replace`
* `_iter`
* `_reviter`
* `findrow`

## Please
When you need a deque in real code, use the standard [`collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque), not this module. When you need features like vector arithmetic or matrix multiplication, use a library like [NumPy](http://www.numpy.org/) or [pandas](https://pandas.pydata.org/), not this module.

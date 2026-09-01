# Python Iterables vs. Iterators

## Table of Contents
- [Python Iterables vs. Iterators](#python-iterables-vs-iterators-1)
- [Under the Hood of Iteration](#under-the-hood-of-iteration)
- [Creating a Custom Iterator Class](#creating-a-custom-iterator-class)
- [Generators: A Cleaner Way to Create Iterators](#generators-a-cleaner-way-to-create-iterators)
- [Memory Efficiency & Infinite Iterators](#memory-efficiency--infinite-iterators)

## Python Iterables vs. Iterators <a name="python-iterables-vs-iterators-1"></a>

An **iterable** is something that can be looped over (like a list, tuple, dictionary, string, or file). However, just because something is iterable does not mean it is an iterator.

* **Iterable:** An object that has a `__iter__` (dunder iter) method. This method returns an iterator object.
* **Iterator:** An object with a state that remembers where it is during iteration. It must have a `__next__` method to fetch the next value and it raises a `StopIteration` exception when it is exhausted. Iterators can only go forward.

## Under the Hood of Iteration <a name="under-the-hood-of-iteration"></a>

When you try to use the `next()` function on a standard list, it throws a `TypeError: 'list' object is not an iterator` because a list does not have a `__next__` method. To iterate over it manually, you must first convert it to an iterator using `iter()`.

**Basic List Iteration:**
```python
nums = [1, 2, 3]

# Getting the iterator from the iterable
i_nums = iter(nums) # In the background, this calls nums.__iter__()

print(next(i_nums)) # Outputs: 1
print(next(i_nums)) # Outputs: 2
print(next(i_nums)) # Outputs: 3
print(next(i_nums)) # Raises StopIteration
```

**How a `for` loop actually works:**
In the background, a standard `for` loop requests an iterator from the object and continuously calls `next()` until it hits a `StopIteration` exception.

```python
nums = [1, 2, 3]
i_nums = iter(nums)

while True:
    try:
        item = next(i_nums)
        print(item)
    except StopIteration:
        break
```

## Creating a Custom Iterator Class <a name="creating-a-custom-iterator-class"></a>

You can make your own objects iterable and act as iterators by defining both the `__iter__` and `__next__` methods. Here is an example of creating a custom class that mimics Python's built-in `range()` function.

```python
class MyRange:
    def __init__(self, start, end):
        self.value = start
        self.end = end
        
    def __iter__(self):
        # Returns the iterator object itself
        return self
        
    def __next__(self):
        # Checks if we have reached the end
        if self.value >= self.end:
            raise StopIteration
            
        # Store current, increment state, return current
        current = self.value
        self.value += 1
        return current

# Using the custom iterator
nums = MyRange(1, 10)
for num in nums:
    print(num)
```

## Generators: A Cleaner Way to Create Iterators <a name="generators-a-cleaner-way-to-create-iterators"></a>

Generators are special functions that allow you to create easy-to-read iterators. Instead of returning a final result, they `yield` a value and pause their state until the generator is called again. Generators automatically create the `__iter__` and `__next__` methods for you.

**Custom Range using a Generator:**
```python
def my_range(start, end):
    current = start
    while current < end:
        yield current
        current += 1

# Works exactly the same as the class-based version
nums = my_range(1, 10)
for num in nums:
    print(num)
```

## Memory Efficiency & Infinite Iterators <a name="memory-efficiency--infinite-iterators"></a>

Because iterators only fetch one value at a time, they are highly memory efficient. They do not hold all values in memory simultaneously. This allows you to create infinite iterators or process massive datasets (like brute-forcing passwords) without crashing your computer.

**Example of an Infinite Generator:**
```python
def infinite_range(start):
    current = start
    while True: # No end condition
        yield current
        current += 1

# This will count up by 1 forever until manually interrupted
infinite_nums = infinite_range(1)
```

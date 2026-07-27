---
title: Using iterators to insert new elements in a container
section: Standard library containers, algorithms, and iterators
section_href: /#standard-library-containers-algorithms-and-iterators
---

Throughout the previous pages we have been passing `std::back_inserter()` to algorithms without stopping to explain it. It is time to do that, because the idea behind it answers a question that comes up as soon as you write to an output range: how does an algorithm add elements to a container it knows nothing about?

Algorithms never touch containers directly. They see a range through iterators, and when they produce output they assign through an output iterator. Assigning through an ordinary iterator overwrites an element that is already there; it cannot make the container grow, which is why `std::fill()` requires the destination to be sized correctly in advance. Insert iterators close that gap. They are adapters that wrap a container and turn every assignment into an insertion, so an algorithm written only to assign ends up growing the container instead.

The standard library provides three of them, one for each way a container can accept a new element.

- Use `std::back_inserter()` to insert elements at the end of a container; it requires the container to have a `push_back()` method, which `std::vector`, `std::deque`, `std::list`, and `std::string` all do:

```cpp run
#include <algorithm>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> v {1, 2, 3};

    std::fill_n(std::back_inserter(v), 3, 0);
    // v = { 1, 2, 3, 0, 0, 0 }
}
```

- Use `std::front_inserter()` to insert elements at the beginning of a container; it requires a `push_front()` method, which `std::deque`, `std::list`, and `std::forward_list` provide but `std::vector` does not:

```cpp run
#include <algorithm>
#include <iterator>
#include <list>

int main() {
    std::list<int> l {1, 2, 3};

    std::fill_n(std::front_inserter(l), 3, 0);
    // l = { 0, 0, 0, 1, 2, 3 }
}
```

- Use `std::inserter()` to insert elements anywhere in a container; unlike the other two it takes a second argument, an iterator marking the position to insert before, and it requires an `insert()` method, which every standard container has except `std::forward_list` and the container adapters:

```cpp run
#include <algorithm>
#include <iterator>
#include <list>

int main() {
    std::list<int> l {1, 2, 3};

    auto it = l.begin();
    std::advance(it, 2);

    std::fill_n(std::inserter(l, it), 3, 0);
    // l = { 1, 2, 0, 0, 0, 3 }
}
```

## How it works

All three are output iterators, and each is a thin wrapper holding a pointer to the container it was built from. What makes them work is that they redefine the three operations an algorithm performs on an output iterator so that only one of them does anything at all:

- `operator*` returns a reference to the iterator itself rather than to any element, because there is no element yet to refer to.
- `operator++` does nothing and also returns the iterator. There is no position to advance past; the container decides where the next element lands.
- `operator=` is the one that acts, calling `push_back()`, `push_front()`, or `insert()` on the stored container with the value assigned to it.

This is why the ordinary output-iterator idiom `*it++ = value` compiles and behaves correctly: the dereference and the increment both evaluate to the iterator, and the assignment inserts. An algorithm written against the generic output-iterator interface therefore drives an insert iterator without containing a single line of special-case code, which is the whole point of the adapter.

The helper functions `std::back_inserter()`, `std::front_inserter()`, and `std::inserter()` construct the corresponding `std::back_insert_iterator`, `std::front_insert_iterator`, and `std::insert_iterator` types. Before C++17 these functions saved you from spelling out the container type as a template argument; class template argument deduction has since made the explicit form bearable, but the helpers remain the idiomatic spelling. All three became `constexpr` in C++20.

Two behaviours are worth committing to memory, because both surprise people.

The first is that `std::front_inserter()` reverses the sequence it copies. Each element is pushed onto the front, so the one inserted last ends up first:

```cpp run
#include <algorithm>
#include <deque>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> source {1, 2, 3, 4, 5};
    std::deque<int> destination;

    std::copy(source.begin(), source.end(),
              std::front_inserter(destination));
    // destination = { 5, 4, 3, 2, 1 }
}
```

The second is that `std::inserter()` does *not* have that problem, even though it inserts before a position rather than at the end. After each insertion it advances its stored iterator past the element it just added, so the next one goes after it and the copied elements keep their original relative order. That is exactly what the third example above shows: the three zeros arrive in order between `2` and `3`, not reversed.

> A consequence of these adapters calling `push_back()` and `insert()` is that they can reallocate. Every iterator you were holding into a `std::vector` may be invalidated part-way through an algorithm, and inserting one element at a time into a vector that grows repeatedly is slower than reserving the space up front. When you know how many elements are coming, call `reserve()` before the algorithm runs; the insert iterator still does the appending, but without the repeated reallocation.

## There is more...

Insert iterators are not limited to sequence containers. `std::inserter()` works with the associative containers too, where the position argument is treated as a *hint* rather than an instruction — the container puts each element where its ordering requires, and merely uses the hint to start looking:

```cpp run
#include <algorithm>
#include <iterator>
#include <set>
#include <vector>

int main() {
    std::vector<int> source {1, 2, 3};
    std::set<int> destination {10, 20};

    std::copy(source.begin(), source.end(),
              std::inserter(destination, destination.end()));
    // destination = { 1, 2, 3, 10, 20 }
}
```

The hint here is `end()`, which is as wrong as it could be, since every inserted value sorts before what is already in the set. The result is still correctly ordered. A good hint makes the insertion faster; a bad one costs a little time but can never corrupt the container.

This is the mechanism behind every `std::back_inserter()` on the set operations page. Algorithms such as `std::set_union()` and `std::set_difference()` cannot know how many elements they will produce, so they are given an output iterator and left to assign through it as many times as the result requires — and the insert iterator turns each of those assignments into a new element.

## See also

- [Using vector as a default container](/containers-algorithms-iterators/vector/), to learn how to use the `std::vector` standard container.
- [Using set operations on a range](/containers-algorithms-iterators/set-operations/), to learn about the standard algorithms for performing unions, intersections, and differences of sorted ranges.
- [Initializing a range](/containers-algorithms-iterators/initializing/), to learn about the standard algorithms that fill a range with values.
- [Finding elements in a range](/containers-algorithms-iterators/finding-elements/), to learn about the standard algorithms for searching through a sequence of values.

---
title: Using set operations on a range
section: Standard library containers, algorithms, and iterators
section_href: /#standard-library-containers-algorithms-and-iterators
---

The standard library provides several algorithms for set operations and enables us to do the union, intersection, or difference of sorted ranges. On this page, we will see what these are and how they work.

```cpp
#include <algorithm>
#include <iterator>
#include <vector>

std::vector<int> v1 {1, 2, 3, 4, 4, 5};
std::vector<int> v2 {2, 3, 3, 4, 6, 8};
std::vector<int> v3;
```

- Use `std::set_union()` to compute the union of two ranges into a third range; the output range will contain the elements that are present in either or both of the input ranges:

```cpp
std::set_union(v1.cbegin(), v1.cend(),
               v2.cbegin(), v2.cend(),
               std::back_inserter(v3));
// v3 = {1, 2, 3, 3, 4, 4, 5, 6, 8}
```

- Use `std::merge()` to merge the content of two ranges into a third one; this is similar to `std::set_union()` except that it copies the entire content of the input ranges into the output one, not just their union:

```cpp
std::merge(v1.cbegin(), v1.cend(),
           v2.cbegin(), v2.cend(),
           std::back_inserter(v3));
// v3 = {1, 2, 2, 3, 3, 3, 4, 4, 4, 5, 6, 8}
```

- Use `std::set_intersection()` to compute the intersection of two ranges into a third range:

```cpp
std::set_intersection(v1.cbegin(), v1.cend(),
                      v2.cbegin(), v2.cend(),
                      std::back_inserter(v3));
// v3 = {2, 3, 4}
```

- Use `std::set_difference()` to compute the difference of two ranges into a third range; the output range will contain elements from the first range that are not present in the second range:

```cpp
std::set_difference(v1.cbegin(), v1.cend(),
                    v2.cbegin(), v2.cend(),
                    std::back_inserter(v3));
// v3 = {1, 4, 5}
```

- Use `std::set_symmetric_difference()` to compute the dual difference of two ranges:

```cpp
std::set_symmetric_difference(v1.cbegin(), v1.cend(),
                              v2.cbegin(), v2.cend(),
                              std::back_inserter(v3));
// v3 = {1, 3, 4, 5, 6, 8}
```

- Use `std::includes()` to check if one range is a subset of another range, that is, if all its elements are present in the other range:

```cpp
std::vector<int> v4 {1, 4, 5};

auto i1 = std::includes(v1.cbegin(), v1.cend(),
                        v2.cbegin(), v2.cend());
// i1 = false

auto i2 = std::includes(v1.cbegin(), v1.cend(),
                        v4.cbegin(), v4.cend());
// i2 = true
```

## How it works

- They take two input ranges, each defined by a first and last input iterator.
- They take an output iterator to the output range where the elements are inserted.
- They have an overload that takes an extra argument representing a comparison binary function object, which must return true if the first argument is less than the second.
- When a comparison function object is not specified, `operator<` is used.
- They return an iterator past the end of the constructed output range.
- The input ranges must be sorted, either using `operator<` or the provided comparison function object, depending on the overload that is used.
- The output range must not overlap the input ranges.

On the other hand, `std::includes()` does not produce an output range; it only checks whether the second range is included in the first range. It returns a boolean value that is true if the second range is empty or all its elements are included in the first range, and false otherwise. It also has two overloads, one of which specifies a comparison binary function object.

We will demonstrate the way they work with additional examples, using a vector of the POD type `task` from the previous documentation:

```cpp
struct task {
    int priority;
    std::string name;
};

bool operator<(const task& lhs, const task& rhs) {
    return lhs.priority < rhs.priority;
}

std::vector<task> v1 {
    { 10, "Task 1.1" },
    { 20, "Task 1.2" },
    { 20, "Task 1.3" },
    { 30, "Task 1.4" },
    { 30, "Task 1.5" },
    { 50, "Task 1.6" }
};

std::vector<task> v2 {
    { 10, "Task 2.1" },
    { 20, "Task 2.2" },
    { 20, "Task 2.3" },
    { 30, "Task 2.4" },
    { 30, "Task 2.5" },
    { 50, "Task 2.6" }
};
```

- Use `std::set_union()` to compute the union of the two ranges of tasks; for every priority, the output range holds as many tasks as whichever input contains the most of them:

```cpp
std::vector<task> v3;

std::set_union(v1.cbegin(), v1.cend(),
               v2.cbegin(), v2.cend(),
               std::back_inserter(v3));
```

The output range holds six tasks, all drawn from the first range:

```text
{ 10, "Task 1.1" },
{ 20, "Task 1.2" }, { 20, "Task 1.3" },
{ 30, "Task 1.4" }, { 30, "Task 1.5" },
{ 50, "Task 1.6" }
```

Both ranges carry the same priorities with the same multiplicities, so the union comes out looking exactly like the intersection did — the same six `Task 1.x` names. When an element appears *m* times in the first range and *n* times in the second, `std::set_union()` copies all *m* from the first range and then max(*n* − *m*, 0) from the second. Here *m* equals *n* for every priority, so nothing is ever taken from the second range and no `Task 2.x` name reaches the output.

- Use `std::merge()` to merge the content of two ranges into a third one; every element from both input ranges is copied to the output:

```cpp
std::vector<task> v3;

std::merge(v1.cbegin(), v1.cend(),
           v2.cbegin(), v2.cend(),
           std::back_inserter(v3));
```

The output range contains all twelve tasks, ordered by priority:

```text
{ 10, "Task 1.1" }, { 10, "Task 2.1" },
{ 20, "Task 1.2" }, { 20, "Task 1.3" }, { 20, "Task 2.2" }, { 20, "Task 2.3" },
{ 30, "Task 1.4" }, { 30, "Task 1.5" }, { 30, "Task 2.4" }, { 30, "Task 2.5" },
{ 50, "Task 1.6" }, { 50, "Task 2.6" }
```

Because `operator<` compares only the `priority` member, the four tasks with priority 20 are all equivalent as far as the algorithm is concerned; their names are what let us see the order it actually chose. `std::merge()` is stable, so when elements from the two ranges compare equivalent, the ones from the first range are copied first — which is why `Task 1.2` and `Task 1.3` precede `Task 2.2` and `Task 2.3`. That guarantee is what makes `std::merge()` usable as the combining step of a merge sort.

- Use `std::set_difference()` to compute the difference of the two ranges of tasks; every priority in the first range is also present in the second, so no task survives and the output range is left empty:

```cpp
std::vector<task> v3;

std::set_difference(v1.cbegin(), v1.cend(),
                    v2.cbegin(), v2.cend(),
                    std::back_inserter(v3));
// v3 is empty
```

This result is worth pausing on. Every name in the two vectors is different, yet nothing is copied, because `operator<` looks only at `priority`: the algorithm considers each task in `v1` to have a match in `v2`. Membership here means whatever the comparison function says it means, not whole-object equality.

To see the difference produce something, compare against a range that does not cover every priority:

```cpp
std::vector<task> v4 {
    { 20, "Task 4.1" },
    { 30, "Task 4.2" }
};

std::vector<task> v5;

std::set_difference(v1.cbegin(), v1.cend(),
                    v4.cbegin(), v4.cend(),
                    std::back_inserter(v5));
// v5 = { 10, "Task 1.1" }, { 20, "Task 1.3" },
//      { 30, "Task 1.5" }, { 50, "Task 1.6" }
```

Priorities 10 and 50 are missing from `v4` entirely, so their tasks are copied. Priorities 20 and 30 appear twice in `v1` and once in `v4`, so one of each survives — and it is worth noticing which one: `Task 1.3` and `Task 1.5`, not `Task 1.2` and `Task 1.4`. When the first range holds *m* equivalent elements and the second holds *n*, `std::set_difference()` copies the **final** max(*m* − *n*, 0) of them.

- Use `std::set_intersection()` to compute the intersection of the two ranges of tasks:

```cpp
std::set_intersection(v1.cbegin(), v1.cend(),
                      v2.cbegin(), v2.cend(),
                      std::back_inserter(v3));
```

This time the output range holds only six tasks:

```text
{ 10, "Task 1.1" },
{ 20, "Task 1.2" }, { 20, "Task 1.3" },
{ 30, "Task 1.4" }, { 30, "Task 1.5" },
{ 50, "Task 1.6" }
```

Every priority in the first range is also present in the second one, so all six tasks are part of the intersection. When elements from the two ranges compare equivalent, `std::set_intersection()` copies the ones from the first range, which is why the output contains only `Task 1.x` names and none of their `Task 2.x` counterparts.

- Use `std::set_symmetric_difference()` to compute the dual difference of the two ranges of tasks; as with `std::set_difference()`, the two ranges hold matching priorities, so neither contributes anything and the output range is left empty:

```cpp
std::vector<task> v3;

std::set_symmetric_difference(v1.cbegin(), v1.cend(),
                              v2.cbegin(), v2.cend(),
                              std::back_inserter(v3));
// v3 is empty
```

Comparing against a range that has a priority of its own shows both directions at work:

```cpp
std::vector<task> v6 {
    { 20, "Task 6.1" },
    { 40, "Task 6.2" }
};

std::vector<task> v7;

std::set_symmetric_difference(v1.cbegin(), v1.cend(),
                              v6.cbegin(), v6.cend(),
                              std::back_inserter(v7));
// v7 = { 10, "Task 1.1" }, { 20, "Task 1.3" }, { 30, "Task 1.4" },
//      { 30, "Task 1.5" }, { 40, "Task 6.2" }, { 50, "Task 1.6" }
```

The output is drawn from both inputs and stays sorted by priority as it interleaves them. Priority 40 exists only in `v6`, so `Task 6.2` is copied out of the second range, while everything else comes from the first. Priority 20 appears twice in `v1` and once in `v6`, leaving one task, and again it is the later `Task 1.3` rather than `Task 1.2`.

- Use `std::includes()` to check whether every task in one range has a counterpart in another:

```cpp
auto i1 = std::includes(v1.cbegin(), v1.cend(),
                        v2.cbegin(), v2.cend());
// i1 = true

auto i2 = std::includes(v1.cbegin(), v1.cend(),
                        v6.cbegin(), v6.cend());
// i2 = false
```

The first result is the sharpest illustration of the rule that governs this whole page. Not one task in `v1` shares a name with any task in `v2`, and yet `std::includes()` reports `true`, because the only question it ever asks is whether `operator<` orders one element before another. By that measure the two ranges are indistinguishable. The second result is `false` for a single reason: no task in `v1` carries priority 40.

## See also

- [Using vector as a default container](/containers-algorithms-iterators/vector/), to learn how to use the `std::vector` standard container.
- [Sorting a range](/containers-algorithms-iterators/sorting/), to learn about the standard algorithms for sorting ranges.
- [Using iterators to insert new elements in a container](/containers-algorithms-iterators/inserting-elements/), to learn how to use iterators and iterator adapters to add elements in a range.
- [Finding elements in a range](/containers-algorithms-iterators/finding-elements/), to learn about the standard algorithms for searching through a sequence of values.

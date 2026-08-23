---
title: Writing your own random-access iterator
section: Standard library containers, algorithms, and iterators
section_href: /#standard-library-containers-algorithms-and-iterators
---

Back in chapter 1, [Enabling range-based for on your own types](/core-language/range-for-custom-types/) made a custom type iterable by giving it a small nested iterator. That iterator was deliberately minimal — three operations and no more: `operator*` to read the current element, prefix `operator++` to advance, and `operator!=` to tell when the end had been reached. That trio is the entire protocol range-based `for` asks for.

What is worth noticing now is everything that iterator could *not* do. You could not hand it to `std::sort`, `std::accumulate`, or any other standard algorithm, because it was not a model of any standard iterator category. It could not be copy-constructed and assigned the way the library requires, and it could not be incremented in any form beyond the single prefix `++` — no post-increment, no stepping backwards, no jumping *n* positions at once, no subscript, and none of the member type aliases the algorithms inspect. Range-based `for` is forgiving; the algorithms are not. To write an iterator the whole library will accept, you have to satisfy the full set of requirements for the category you are targeting.

This page builds the most capable of those categories — the *random-access iterator*, the one that can move to any element in constant time — starting from an empty container. Before diving in, it helps to have the requirements in front of you: a compact overview of every iterator category and the operations each one is obliged to provide is at [cplusplus.com/reference/iterator](https://www.cplusplus.com/reference/iterator/).

## A container to iterate over

First we need something to iterate over. `dummy_array` is a thin wrapper around a fixed-size C array — just enough of a container to be worth giving an iterator, and nothing that would distract from the iterator itself:

```cpp
template <typename Type, size_t const SIZE>
class dummy_array
{
    Type data[SIZE] = {};

public:
    Type& operator[](size_t const index)
    {
        if (index < SIZE)
            return data[index];

        throw std::out_of_range("index out of range");
    }

    Type const& operator[](size_t const index) const
    {
        if (index < SIZE)
            return data[index];

        throw std::out_of_range("index out of range");
    }

    size_t size() const { return SIZE; }
};
```

The element count is a template parameter, so the size is baked into the type and the array lives inline with no allocation. Two subscript operators give read-write access to a `dummy_array` and read-only access to a `const dummy_array`, each bounds-checked so an out-of-range index throws rather than reads past the storage, and `size()` reports the fixed length. What it does *not* have yet is any `begin()` or `end()` — so at this point it works with neither range-based `for` nor a single algorithm. Supplying that is the whole job of the iterator we are about to write.


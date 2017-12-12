## The Algorithm Library (STL)

A suite of template functions, many of these work over Iterators

`#include <algorithm>`

Examples: `for_each`

```c++
template <typename Iter, typename T>
Iter find(Iter begin, Iter last, const T &val) {
    // returns iterator to the first element in [begin, last)
    // if it matches val
    // returns last if not
}
```

Other functions:

- `count` - like find but returns the number of occurences of values in [begin, last)
- `copy`
- `transform` - like copy, but also applies a function

```c++
template <typename InIter, typename OutIter>
OutIter copy(InIter begin, InIter last, OutIter result) {
    // copies one container range from [begin, last) to the containter that result points to
}
```

**Note**: Does **NOT** allocate more space if needed in the container result points at, you must make sure you have enough.

Example:

```c++
vector<int> v{1,2,3,4,5,6,7};
vector<int> w(4);
// creates a vector with space for 4 ints
copy(v.begin(), v.begin() + 5, w.begin());

// -------------------

template <typename InIter, typename OutIter, typename Func>
OutIter transform(InIter first, InIter last, OutIter result, Func f) {
    while (first != last) {
        *result = f(*first);
        ++first;
        ++result;
    }
    return result;
}
```

Focusing on transform, just how generalized is out template code?

1. What can we use for func?
2. What can we use for InIter/OutIter

So Func needs to be callable as a function. WHat other than functions can provide that behaviour?

We can overload `operator()` for our classes

```c++
class Plus {
    int m;

public:
    Plus(int m) : m{m} {}
    int operator(int n) { return m + n; }
};

transform(v.begin(), v.end(), w.begin(), Plus{5});

The big advantage of using function objects over functions is that they can maintain state

class IncreasingPlus {
    int m = 0;
public:
    int operator()(int m) {
        return n + (m++);
    }

    void reset() {
        m = 0;
    }
};
```

How many ints in a vector are even?

```c++
vector<int> v{...};

bool even(int n) {
    return !(n % 2);
}

int x = count_if(v.begin(), v.end(), even);
```

## Lambdas

```c++
int x = count_if(v.begin(), v.end(), [](int n) {
    return !(n % 2);
});
```

## Iterators

An iterator is anything that supports these operators: `*`, `++` and `!=`

So that means that we can apply the notion of iterators to other no container types, such as streams.

Example:

```c++
#include <iterators>

vector<int> v {1,2,3,4,5};

ostream_iterator<int> out {cout, ", "};

copy(v.begin(), v.end(), out);
// prints 1, 2, 3, 4, 5, 
```

Consider:

```c++
vector<int> v {1, 2, 3, 4, 5};
vector<int> w;

copy(v.begin(), v.end(), w.begin());
// WRONG!
```

Remember, copy can't allocate new space for you since it doesn't even know what `w` is iterating over.

But what if we had a specialized iterator whose assigment operator inserts a new item?

```c++
vector<int> v {1,2,3,4,5};
vector<int> w;

copy(v.begin(), v.end(), back_iterator(w));
```

Now `v` is copied to `w` with space allocated as needed.

> Take the time to learn to work with Iterators and the algorithm library, and get confortable with them (also the rest of the STL). They can dramatically reduce the length of your code and reduce opportunities for bugs in your programs.

## Template Meta Programming

Offload computation to the compiler

```c++
template<int n>
struct Fact {
    enum { val = Fact<n - 1>::val * n };
};

template<>
struct Fact<0> {
    enum { val = 1 };
}

int main() {
    // runs in O(1) !!!
    int n = Fact<10>::val;
}
```

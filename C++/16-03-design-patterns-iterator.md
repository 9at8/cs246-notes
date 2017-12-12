# Iterator Pattern

Our guiding principle is to program interfaces, not implementations.

Abstract base classes define the interface. Work with pointers or references to Abstract Base classes and call the methods declared in the interface. So this way, concrete class objects in and out at will. 

```c++
// It can be templated

class AbstractIterator {
public:
  virtual int operator*() = 0;
  virtual bool operator!=(const AbstractIterator &o) = 0;
  virtual ~AbstractIterator();
}

class List {
  struct Node;
  ...

public:
  class Iterator :: public AbstractIterator {
    ...
  };
};

class Set {
  ...

public:
  class Iterator : public AbstractIterator {
    ...
  };
};
```

We can write functions that operate on Iterators that are derived from `AbstractIterator`. In this way, we can use the same functions to iterator over a variety of different data structures, withour knowing what those underlying structures are.

```c++ 
template <typename Fn> {
  void foreach(AbstractIterator &start, AbstractIterator &end, Fn f) {
    while (start != end) {
      f(*start);
      ++start;
    }
  }
}
```

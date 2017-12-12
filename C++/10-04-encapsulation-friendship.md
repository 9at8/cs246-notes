# Friendship

```c++
class List {
  class Node {
    int data;
    Node *next;
  };

  Node *next;
public:
  class Iterator {
    Node *p;
  public:
    Iterator(Node *p);
    int &operator*();
    Iterator& operator++();
    bool operator!=(const Iterator &it);
    Iterator begin();
    Iterator end();
    void addToFront(int n);
  };
};

// user:
auto it = List::Iterator(nullptr);
```

This breaks encapsulation, the user should only be able to create iterators through the methods we've provided to them in our class, `begin()` and `end()`.

If we make the constructor private, `List` can no longer create `Iterator` objects to return in `begin()` and `end()`, if we leave it public we break encapsulation. The solution is **_Friendship!_**

We can declare a class X to be a friend of another class Y and in doing so we give X access to Y's _private parts_.

```c++
Iterator {
  Iterator(Node *p)
  ...
  ...
public:
  ...
  friend class List;
};
```

Try to have as few friends as possible, **friends weaken encapsulation**.
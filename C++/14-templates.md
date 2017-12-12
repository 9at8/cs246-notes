# Templates

Note: We are **not** covering the concept of templates in entirety, just some stuff.

```c++
class List {
  struct Node;
  Node *head;
  ...
  ...
};

struct List::Node {
  int data;
  Node *next;
}
```

What if we want to store something other than `int`s? Do we have to write a whole new `class`? Well, a template class is a class that is parametized by a type.

## Example: A stack

```c++
template <typename T> class Stack {
  int size;
  int capacity;
  T *data;

public:
  stack() ...
  void push(T *) ...
  T top() const ...
  void pop() ...
}
```

## Template for our List

```c++
template <typename T> class List {
  struct Node {
    T data;
    Node *next;
  }

  Node *head;

public:
  class Iterator {
    Node *p;
    Iterator(Node *p) : p{p} {}
    T& operator*() { return p->data; }
    bool operator!=(const Iterator &o) { return p != o.p }
    Iterator& operator++() { p = p->next; return *this; }
    friend class List;
  };

  Iterator begin() { return Iterator{head}; }
  Iterator end() { return Iterator(nullptr); }
}
```

But how do I use this template?

```c++
List <int> l1;
List <List <int> > l2;
List <List <List <int> > > l3;
// need spaces around the closing angular braces

l1.addToFront(3);
l2.addToFront(l1);
l3.addToFront(l2);

for (auto &n : l1) {
  cout << n << endl;
}

// or this

for (auto &l : l2) {
  for (auto n: l) {
    cout << n << endl;
  }
}
```

How do templates work? We make the compiler do it.

Roughly speaking, the compiler makes a separate class each time a templated class is parameterized with a new type (at the machine code level (not really))

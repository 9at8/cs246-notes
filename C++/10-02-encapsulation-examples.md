# Encapsulation Examples

## Example - Stack

A stack requires that the next thing popped will be the most recent thing that was pushed. But if the client can modify the internal structure of the stack, we have no way of maintaining this invariant.

## Example - Linked list

Now, let's fix our Linked List by introducing encapsulation, the key is to create a wrapper class `List` that has exclusive rights to the underlying `Node` objects

```c++
// list.h

class List {
  struct Node;
  Node *theList;
  
public:
  void addToFront(int n);
  int ith(int i);
  ~List();
}
```

```c++
// list.cc

struct List::Node {
  int data;
  Node *next;
  Node (int data, Node *next) : data{data}, next{next} {}
  ~Node() { delete next; }
}

List::~List() { delete theList; }

List::addToFront(int n) { theList = new Node { n, theList }; }

int List::ith(int i) {
  Node *p = theList;
  for (int j = 0; j < i && p; ++j, p = p->next);
  return p->data;
}

// this fn is trash for a number of reasons and has an error
```

Since only `class List` can manipulate `Node` objects we can now hold the invariant that the next is either the `nullptr` or head allocated memory.

But now, we can't traverse the list from node to node as we would a linked list

```c++
for (int i = 0; i < size; ++i) { myList.ith(i); }
```

Repeatedly calling `ith` is `O(n^2)`, which is bad.

# Design Patterns

This is a common problem and some problems come up again and again. So we keep track of good solutions to these problems. Solutions are called design patterns.

A design pattern says, if you have a problem like *X* then *Y* may be a good solution.

## Iterator Pattern

Our current problem is known as the iterator pattern. We create a new class that manages access to the nodes, so essentially we are abstracting pointers.

```c++
class List {
  struct Node;
  Node *theList;

public:
  class Iterator {
    Node *p;
  
  public:
    explicit Iterator (Node *p) : p{p} {}
    Iterator &operator++() { p = p->next; }
    int &operator*() { return p->data; }
    bool operator==(const Iterator &rhs) { return p == o.p; }
    bool operator!=(const Iterator &rhs) { return !(p == o.p); }
  }
  
  Iterator begin() { return Iterator{theList}; }
  Iterator end() { return Iterator{nullptr}; }
}

int main() {
  List lst;
  lst.addToFront(3);
  lst.addToFront(2);
  lst.addToFront(1);

  for (List::Iterator it = lst.begin(); it != lst.end(); ++i) {
    cout << *it << endl;
    *it = (*it) * 2;
  }
}
```

We can shorten the iteration loop somewhat by taking advantage of type deduction.

A definition like `auto x = y`; defines `x` to have the same return type as `y`. The main advantage is we don't have to write the exact type of x, which may be complex;

```c++
...
for (auto it = lst.begin(); it != lst.end(); ++i) { ... }
...
```

We can shorten the loop even further by taking advantage of C++'s build in support of the Iterator pattern.

```c++
...
for (auto n : lst) {
  cout << n << endl;
}
...
```

These are called range based loops. They are available for any class C with the following properties:
- C has methods `begin` and `end`, that produce `Iterators`
- The `Iterator` supports prefix `++`, `operator!=` and the unary `*` operator.

The range based loop above declares the variable `n` by value as a copy of successive items in a list. What this means is, that we can't update the node anymore.

To update the data, we can use a reference.

```c++
for (auto &n : lst) {
  n = n * 2;
}
```

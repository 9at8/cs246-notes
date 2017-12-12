# The big 5
In summary, if you need one of the following in your classes, then you almost certainly need all 5. (Big 5)

- Copy Ctor
- Copy Assignment
- Move Ctor
- Move Assignment
- Dtor

## Copy Ctor

```c++
struct Node {
    int data;
    Node *next = nullptr;

    // this is the copy ctor
    Node(Node &other) {
        ...
    }
}
```

## Copy Assignment Operator

```c++
Student billy{60, 70, 80};
Student jane = billy;   // Copy ctor
Student mohammed;       // default ctor
mohammed = billy;       // copy, but not ctor - copy assignment operator
```

Classes come with a copy assignment operator that just copy assigns each field (a shallow copy). Thus we may need to write our own.

```c++
struct Node {
  ...
  Node &operator=(const Node &other) {
    data = other.data;
    // free up old memory
    delete next;
    next = other.next ? new Node{*other.next} : nullptr;
    return *this;
  }
}

// What happens when we do this?

Node n{5, new Node{2, new Node{1, nullptr}}};
n = n;

// When writing operator=
// **Always be wary of the possibility of self-assignments**

Node &operator=(const Node &other) {
  if (this == &other) return *this;
  data = other.data;
  delete next;
  next = other.next ? new Node{Node *other.next} : nullptr;
  return *this;
}
```

We must not change the object if `new` fails

```c++
Node &operator=(const Node &other) {
  if (this == &other) return *this;
  Node *tmp = next;
  next = other.next ? new Node{*other.next} : nullptr;
  delete tmp;
  data = other.data;
  return *this;
}
```

Alternatively, we can employ a copy-and-swap idiom to achieve all of these advantages of this version more concisely.

```c++
#include <utility>

struct Node {
  void swap(Node &other) {
    using std::swap;
    swap(data, other.data);
    swap(next, other.next);
  }

  Node &operator=(const Node &o) {
    Node tmp{o};  // deep copy via copy ctor
    swap(tmp);    // same as this->swap(tmp);
    return *this;
  }
}
```

## Move Constructor

```c++
// Node.h

struct Node {
  int data;
  Node *next;
  ...
  ...
  Node (Node &&o); // Move constructor
}

// Node.cc
Node::Node (Node &&o) : data{o.data}, next{o.next} {
  o.next = nullptr;
  // set o.next to nullptr so that the chain of data isn't removed from memory
}
```

## Move Assignment operator

What happens when we already have an object which in initialized? We also need to clear up that memory as well.

```c++
Node &operator=(Node &&o) {
  using std::swap;
  swap(data, o.data);
  swap(next, o.next);
}
```

If you don't define a move ctor/assignment op, then the copying versions are used. (bad because there is no reason to deep copy as the r values are going to be destroyed)

If the move ctor/assignment op is defined, it will replace calls to the copying versions when the argument is an r value (temporary)

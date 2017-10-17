# OOP: Classes
The big movation of OOP: can put functions inside structs.

```c++
struct Student {
  int assms, mt, fn;
  float grade() {
    return assms + mt + fn;
  }
}

Student Billy = Student{60, 70, 80}
cout << Billy.grade() << endl;
```

In object oriented terminology, a class is essentially a struct type that can potentially contain functions. Student above is a class.

An object is an instance of a class

The functions inside a class are called member functions or methods.

But inside the function grade, what do the variables mean? Since these don't exist until a Student is instantiated? They refer to the fields of the object on which the method was invoked.

```c++
Student Billy{ ... };

Billy.grade(); // uses Billy's variables
```

Formally, methods take a hidden extra parameter with the name `this` which is a pointer to the object on which the method was invoked.

```c++
struct Student {
  int assms, mt, final;
  float grade() {
    return this->assms * 0.4 + this->mt * 0.2 + this->final * 0.4;
  }
}
```


## Initialization
```c++
Student billy{60, 70, 80};
// This is fine, but is limited
```

Better thing to have: a function that does initialization: Constructor

```c++
struct Student {
  int assms, mt, final;
  float grade() { ... }
  Student(int assms, int mt, int final) {
    this->assms = assms;
    this->mt = mt;
    this->final = final;
  }
}

Student Billy{60, 70, 80}; // better!
```

If the constructor has been defined, the members above are passed as arguments to the constructor. If no constructor has been defined, these initialize the individual files of Student. Alternatively, `Student Billy = Student{60, 70, 80};` does the same thing.

### Uniform Initialization
Sometimes, variables are initialized with `=`

```c++
int x = 9;
string s = "hello";

// We can also initialize by (), especially when invoking constructors

int x(5);
string s("hello");
Student Billy(60, 70, 80);
```

The rules involving when we may use `=` v/s when we may use `()` are somewhat arbitrary. C++ now has uniform initialization syntax, using `{}`, which is meant to be used in (nearly) all initialization cases

```c++
int x{5};
string s{"hello"};
...
...
```

This is the **preferred** syntax

##### For allocation on the heap
```c++
Student billy = new Student{60, 70, 80};
```

## Advantages of using a constructor
  - Default params
  - Overloading
  - Sanity checks
  - Logic other than simple field init

The built in constructor goes away if you define any constructor

```c++
struct Vec {
  int x, y;
  Vec (int x, int y) {
    this->x = x;
    this->y = y;
  }
}

Vec v1{1,2};
// calls the ctor we gave

Vac v2;
// error, no matching fn defn
```


## What happens when an object gets created?

What if a structure contains `const`or a reference variable? Does every instance of my struct need to have the same values?

Where do we initialize? Ctor body? Too late for that.

  1. Space is allocated
  2. Fieldsare constructed (need to put our initializations here)
  3. ctor body runs

How do we do this? Using the **Member Initialization (MIL)**

```c++
struct Student {
  const int id,
  int assns, mt, final,
  Student(int id, int assms, int mt, int final) : id{id}, assms{assms}, mt{mt}, final{final} {
    // blank
  }
}
```

### Notes
  - You can .... Check picture taken on October 3

**_Next lecture (missed - Oct 5 2017 thursday):_**
  - destructor


## Destructor
...
...
...


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
// Always be wary of the possibility of self-assignments

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
  Node &operator=(Node &other) {
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


## Member operator
The operator `=` is a member function, and not a standalone function. When an operator is declared as a member function, `this` plays the role of the left hand side.

```c++
n = m; // same as n.operator=(m);

struct Vec {
  int x,y;
  ...
  ...
  Vec operator+(const Vec &o) {
    return Vec{x + o.x, y + o.y};
  }
  ...
  ...
}
```

But if we have to do something like `int * Vec` then we have to create a standalone function. Same thing for I/O as well, as the streams are on the left hand side.

```c++
struct Vec {
  ...
  ...
  ostream &operator<< (ostream) {
    out << << "," << y;
    return out;
  }
}
```

^THIS SHOULD NOT BE DONE.

### Some operators that must be member functions
  - `operator=`
  - `operator->`
  - `operator[]`
  - `operator()`
  - `operatorT`


## Separate compilation for classes
```c++
// Node.h
#ifndef __NODE_H__
#define __NODE_H__

struct Node {
  int data;
  Node *next;
  explicit Node(int, Node *next=nullptr);
  Node(const Node &o);
}
```

Don't create `struct Node` again in `Node.cc`

```c++
// Node.cc
#include "Node.h"

Node::Node(int data, Node *next) : data{data}, next{next} {}
Node::Node(const Node &o) : data{o.data} next{o.data ? new Node{o.next} : nullptr} {}
Node &Node::operator=( ... ){ ... }
```


## DK what this is
```c++
Node plusOne(Node N) {
  for (Node *p = &n; p p = p->next) {
    ++p->data;
  }

  return n;
}

Node m3 = plusOne(n);   // copy ctor(?)

If the definition of m3 calls the copy ctor (it does), then what is other? It is an lvalue reference, but what object with an address does it reference?

The compiler creates a temporary object to hold the result of `plusOne`. Other is a ref to that temp object, and out copy ctor deep copies that temp.

But:
  - The temp is going to be discussed anyway, as soon as the stmt `Node m3 = plusOne(n);` is done
  - It is wasteful to copy data from a temp; why not just steal it instead and save cost of a deep copy
  - We need to be able to tell if other is a ref to a temp or a real object
```

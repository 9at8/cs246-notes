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

**_Pictures from 12 Oct 2017_**


## More constructors

R values are temp values; such as the return value from a function; Accessed using double `&` (`&&`)

### Move Constructor
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

### Move Assignment operator
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


### Copy Elision
```c++
Vec MakeAVec { return Vec{0,0}; } // uses the basic 2 int ctor
...
...
Vec v = MakeAVec(); // what would be run here?
```

**Neither**

Not sure! In G++, only the basic ctor runs, no copy, no move.

In some circumstances, the compiler is allowed to (but doesn't to) skip calling the copy/move ctors. In the example above, `MakeAVec` writes the result directly into the location of memory where `v` is.

Rather than write it to the local stack frame, return it, and then copy/move ctor

Another example:
```c++
void doSomething(Vec v) {
...
...
}
...
...
doSomething(MakeAVec()); // same thing as above
```

Under `copy elision`, the result of `MakeAVec()` is written directly into `doSomething`s stack frame.

To disable `copy elision` use flag `-fno-elide-constructors` with g++

Copy elision is allowed even if the ctors have some side effects. _We are not responsible for knowing when copy elision can happen, only that it can happen_

### The big 5
In summary, if you need one of the following in your classes, then you almost certainly need all 5. (Big 5)

  - Copy Ctor
  - Copy Assignment
  - Move Ctor
  - Move Assignment
  - Dtor

## More Scenarios
```c++
struct Vec {
  int x, y;
  Vec(int x, int y) : x{x}, y{y} {}
};

Vec *vp = new Vec[10];
Vec moreVecs[3];
vec v;
```

This will create an error

What are our options to fix this?

  1. Give values to initialize with
  2. Provide a default ctor
  3. For heap arrays, switch to array of ptrs
    ```c++
Vec **vp = new Vec*[10];
vp[0] = new Vec{1,2};
// doesn't initialize vecs, provides an array of pointers
    ```

int f(const Node &n) { ... }

Constant objects arise often, especially as functions parameters. If an object is const we can't change its fields. But can we call methods on a const objects? The issue is a method may modiify the objects fields through `this` ptr.

The answer is to only allow methods to be caled which have promised not to modify `*this`

```c++
struct Student {
  int assms, mt, final;
  float grade() const { ... }
}
```

// Now grade is a const method so the compiler makes sure you don't modify `*this` in `grade`

## Encapsulation
```c++
// Consider this

struct Node {
  int data;
  Node *next;
  Node (int data, Node *next) : data{data}, next{next} {}
  ~Node () { delete next; }
}

int main() {
  Node n1{1, new Node{2, nullptr}};
  Node n2{3, nullptr};
  Node n3{4, &n2};
}
```

But what happens when these objects go out of scope? All three are stack allocated. So all three have their dtors run. When n3's dtor runs, it tries to delete stack allocated `n2`, and the result is undefined behaviour.

So the dtor has an assumption, `next` is either a `nullptr` or heap allocated memory. This is the essence of an **invariant**, a statement that is supposed to hold true for a class, in order for it to work as advertised.

For example, consider a stack. A stack obeys the variant that the first thing to be removed off a stack was the most recent added. This fails if the user can rearrange the order of the stack's data.

It is very hard to reason about code if you can't rely on invariants.

To help enforce out invariants, we introduce encapsulation, so named bacause we treat our classes like closed capsules that wrap around their data. So the user cant access it, and implementation details are hidden from the user.

```c++
struct Vec {
  Vec(int x, int y) x{x}, y{y} {}
private:
  int x, y;
public:
  Vec operator+(const Vec &rhs) { ... }
};

// by default, members of a struct are public.
```

In general, we want our fields to be private. This prevents clients from tampering with our internals and destroying invariance. Out methods (typically) should be public


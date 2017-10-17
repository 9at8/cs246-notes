# Lecture 3, CS 246


## Constants
```c++
const int maxGrade = 100;
//Declare anything you don't need to change as const, helps in catching errors


Node n1 {5, nodeptr};
// Syntax for null pointers in c++, Do not use NULL or 0 in this class!


const Node n2 = n1;
// Immutable copy of n1


// Pointer to a 'const Node'
const Node *n3 = &n1;
// Pointer to a const Node

(*n3).data = 10;
// NOT OKAY

n1.data = 10;
// Now (*n3).data is 10

n3 = &n4;
// This is okay


// 'const' pointer to a 'const' node
const Node const *n5 = &n1;

n5 = &n4;
// Not okay

(*n5).data = 10;
// Also not okay, node is const

n1.data = 10;
//updates affect (*n5)
```


## Call by Value
```c++
int x;
cin >> x;
```

Question: Why can we say `cin >> x` and not `cin >> &c`?

Answer: C++ has another **pointer like** type **References**


## References
```c++
int y = 10;
int &z = y;

// z is an lvalue reference to y
// Reference are like aliases

z = 5;
// y is now 5
```

  - References are **like** constant pointers with automatic dereferences
  - References aren't guaranteed to have memory
  - In all cases `z` behaves exactly like y
  - Things you can't do with lvalue references
    - Leave them uninitiated; eg `int &x;`
    - Must be initialized with something that has an address (lvalue)
      - `int &x = y + z` is invalid
    - Create a pointer to a reference `int &*x;`
    - We **can** create a reference to a pointer `int *&x;`
    - Create a reference to a reference
    - Create an array of references eg `int &n[3] = {n,n,n};`
  - What **CAN** you do?
    - Pass as function parameter
      ```c++
      void inc (int &n) { n++; }
      
      int x = 5;
      inc(x); // x is passed as a reference
      cout << x << endl; // print 6
      ```
    - So why does `cin >> x work`? Because it takes x by reference


## Pass by Value
If the argument is big, the copy could be expensive.

```c++
struct ReallyBig { ..... };
int f(ReallyBig { ..... })
// copies, potentially slow (see copy/move elision later)

int g(ReallyBig &r) { ..... }
// alias. fast. but could change data

int h(const ReallyBig &r) { ..... }
// alias. fast. const.
```

Note: Prefer passing by const reference over pass by value for anything larger that an int. Unless the function needs a copy anyway.

Also:
```c++
int f(int &n) { ..... }
f(5);
// Not okay

int g(const int &n) { ..... }
g(5);
// WORKS!
```

  - Since 5 is never changed, compiler allows this.
  - How does `g(5)` work? The compiler creates a temporary location in memory to hold the 5, so reference has something to refer to


## Dynamic Memory Allocation
```c
int *p = malloc(... * sizeof(int));
free(p);
```

Don't use these in C++. Instead use `new/delete`

Reasons:
  - Type aware
  - Less error prone

```c++
struct Node {
  int data;
  Node *next;
};

Node *np = new Node;
...
...
delete np;
```

  - All local variables are on the stack
  - `np` is on the stack, but the actual struct is on the heap
  - Variables are deallocated when they fo out of scope (stack is popped)
  - Allocated memory resides on the heap
  - Remains allocated until delete is called
  - If you don't delete all allocated memory - Memory Leak. Program will fail eventually

### Array forms:

```c++
Node *np = new Node[10]; 
// Allocates memory for 10 nodes on the heap
...
...
delete [] np;
```

  - It is important to match the correct form of `delete` with the corresponding `new`. If Memory was allocated with ordinary `new`, it must be deallocated with ordinary `delete`
  - Mixing the teo forms of delete results in undefined behaviour


```c++
Node getMeANode() {
  Node n;
  return n;
}
// Potentially expensive
// n is copied to the caller stack


// return a pointer or a reference instead

Node *getMeANode() {
  Node n;
  return &n;
}
// Bad: returns a ptr to stack allocated data

Node *getMeANode() {
  Node *np = new Node;
  return np
}
// okay; but user's job to delete
```

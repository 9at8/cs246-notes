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

## The Standard Template Library

It is a large collection of template classes provided as part std c++

### Dynamic size arrays - Vector

```c++
#include <vector>

using namespace std;

vector <int> v{4, 5};
// creates the array [4, 5]
// calls the list initialisation ctor

vector <int> v2(4, 5);
// This calls vector(int, T)
// v(4, 5) creates [5, 5, 5, 5]

v.emplace_back(6); // 4,5,6
v.emplace_back(7); // 4,5,6,7

for (int i = 0; i < v.size(); ++i) {
  cout << v[i] << endl;
}
```

Vector implements the iterator pattern

```c++
for (vector<int>::iterator it = v.begin(); it != v.end(); ++i) {
  cout << *it << endl;
}

// or this

for (auto n : v) {
  cout << *it << endl;
}

// or the reverse iterator

for (vector<int>::reverse_iterator it = v.rbegin(); it != v.rend(); ++it) {
  cout << *it << endl;
}
```

`v.pop_back()` removes the last element of `v`

Vectors are generated to be implemented as an array, so from now on, if you use a dynamic array, use vector. There is no more reason to use the array forms of new[] and delete[]

Other vector operations are based on iterators such as `erase`, which erases the element pointed to by the iterator

```c++
auto it = v.erase(v.begin());
// erases the item as index 0, returns the iterator to the first item after the erase

it = v.erase(v.begin() + 3);
// erases the item at the 3rd index

it = v.erase(it);
// erases the item pointed to by it

it = v.erase(v.end() - 1);
// erases the last item on our list
```

`v[i]` returns the `ith` element of v. If you go out of bounds, vector doesn't check for you - undefined behaviour.

We can instead use `v.at(i)`, which is a checked version.

But what does it do when you go out of bounds? What should happen?

Problem: Vector code can detect the error, but doesn't know what to do about it. The client knows what they do if it occurs, but can't detect it. Error recovery, by nature is a non-local problem.

**C Solution:** fn returns a status code or if output eange completely used set a global variable. These are awkward and clunky solutions, which lead to wither bad code or ignored errors.

**C++:** when an error condition arises, the function can raise an exception. What happens when an exception is raised? The program terminates by default. However the client can catch the exception and handle it.

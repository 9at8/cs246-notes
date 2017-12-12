# Member operator

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
// THIS SHOULD NOT BE DONE.
```

## Some operators that must be member functions

- `operator=`
- `operator->`
- `operator[]`
- `operator()`
- `operatorT`

---

# r-values

```c++
Node plusOne(Node N) {
  for (Node *p = &n; p p = p->next) {
    ++p->data;
  }

  return n;
}

Node m3 = plusOne(n);   // copy ctor(?)
```

If the definition of m3 calls the copy ctor (it does), then what is other? It is an lvalue reference, but what object with an address does it reference?

The compiler creates a temporary object to hold the result of `plusOne`. Other is a ref to that temp object, and out copy ctor deep copies that temp.

But:

- The temp is going to be discussed anyway, as soon as the stmt `Node m3 = plusOne(n);` is done
- It is wasteful to copy data from a temp; why not just steal it instead and save cost of a deep copy
- We need to be able to tell if other is a ref to a temp or a real object

**_Pictures from 12 Oct 2017_**

R values are temp values, such as the return value from a function; Accessed using double `&` (`&&`)

---

# Copy Elision

```c++
Vec MakeAVec { return Vec{0,0}; } // uses the basic 2 int ctor
...
...
Vec v = MakeAVec(); // what would be run here?
```

**Neither**

Not sure! In `g++`, only the basic ctor runs, no copy, no move.

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

---

# More Scenarios

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

Constant objects arise often, especially as functions parameters. If an object is const we can't change its fields. But can we call methods on a const objects? The issue is a method may modify the objects fields through `this` ptr.

The answer is to only allow methods to be caled which have promised not to modify `*this`

```c++
struct Student {
  int assms, mt, final;
  float grade() const { ... }
}
```

// Now grade is a const method so the compiler makes sure you don't modify `*this` in `grade`

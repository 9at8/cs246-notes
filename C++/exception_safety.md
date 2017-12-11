# Exception Safety

```c++
void f() {
    MyClass *p = new MyClass;
    MyClass mc;
    g();
    delete p;
}
```

No leaks in this code, but what if `g` throws an exception. What is guaranteed?

During stack unwindng, all stack allocated data is cleared up, dtor runs, memory is reclaimed. But heap allocated memory is not freed.

Therefore, if `g()` throws, `p` is leaked.

```c++
void f() {
    MyClass *p = new MyClass;
    MyClass mc;
    try {
        g();
    } catch (...) {
        delete p;
        throw;
    }
}
```

This is tedious and error prone as we have duplication of code. How else can we guarantee something (here `delete p;`) will execute, no matter how the function exists.

All C++ guarantees is that the dtors for stack allocated objects will run.

So, put to your advantage and use stack allocated objects as much as possible, and when stack allocation won't work...

## RAII: C++ Idiom

> Resource Acquisition Is Initialisation

Every resource should be wrappedd in a stack allocated object, whose destructor will clean it up.

```c++
// Example files
{
    ifstream{"file"};
}
```

Acquiring the resource (file) happens by initializing the object `f`

The file is guaranteed to be closed when `f` is popped from the stack (`f`'s dtor is run)

This can be done with dynamic memory. Class `std::unique_ptr<T>` is a class that holds a `T*` which you supply in the constructor.

Also include `<memory>`

The dtor will delete the ptr. In between you can use the smart pointer just like you would a raw pointer. (example dereference it, etc)

```c++
void f() {
    auto p = std::make_unique<MyClass>();
    std::unique_ptr<MyClass> p{new MyClass}; // also works
    MyClass mc;
    g(); // No leaks, even if `g` throws.
}
```

### Difficulty with Unique Pointers

```c++
class C { ... };
auto p = make_unique<C>();
unique_ptr<C> p2 = p;
```

What does it mean to copy a unique ptr? Well, its nonsense - its supposed to be unique. We don't want to delete the same ptr twice. So copying is disallowed for `unique_ptr`s - They can only be moved

If you need to be able to copy ptrs, Use `std::shared_ptr`

```c++
void f() {
    auto p1 - std::make_shared<MyClass>();
    if ( ... ) {
        auto p2 = p1; // Two pts to same ob;
    } // p2 is popped off the stack, but the object is not deleted.
} // p1 is popped, the object is freed
```

Shared pointers maintain a reference count. A count of all `shared_ptr`s pointing at the same object. The memory is freed when the number of `shared_ptr`s pointing at it reaches.

Use `shared_ptr`s and `unique_ptr`s instead of raw pointers whenever you can. They will dramatically reduce the chance of memory leaks.

For other kinds of resources, you should similarly follow `RAII` by creating or using classes to manage resources.

Example `fstream` objects for files or `XWindow` objects for `Window`

## Back to Exception Safety

There are three levels of exception safety that a funtion `f` can offer.

1. Basic Guarantee: if an exception occurs, the program will left in a valid, but unspecified state. Nothing is leaked and any class invariants are maintained.
2. Strong Guarantee: If an exception is raised while executing `f`, the state of the program will be as it was had `f` not been called.
3. No throw guarantee: `f` will never throw an exception and will always accomplish its task.

```c++
class A { ... };
class B { ... };
class C {
    A a;
    B b;
public:
    void f() {
        a.method1(); // may throw (provides strong guarantee)
        b.method1(); // may throw (provides strong guarantee)
    }
};
```

Is `C::f` exception safe?

- If `a.method1()` throws, nothing happens. So ok.
- If `b.method2()` throws, the effects of a `method1` must be undone to offer the strong safety. This is very hard or impossible to offer the string guarantee if `a.method1` has non local side-effects.

If `A::method1` and `B::method1` do not have non local side effects, we can provide exception safety using copy and swap.

```c++
class C {
    A a;
    B b;
public:
    void f() {
        A atemp = a;
        B btemp = b;
        
        atemp.method1();
        btemp.method1();
        
        a = atemp;
        b = btemp;
        // What if copy assignment throws??
    }
}
```

Because copy assignments could throw, we don'tyet have exception safety. It would be better if we could guarantee that the `swap` part was a no-throw operation. A non-throwing swap is at the heart of writing exn safe code.

```c++
class C {
    unique_ptr<cImpl> pImpl;
public:
    void f() {
        auto temp = make_unique<cImpl>(*pImpl);
        temp->a.method1();
        temp->b.method1();
        std::swap(pImpl, temp);
    }
};
```

## Exception Safety and the STL Vectors

### Vectors

- Encapsulate a heap allocated array
- Follow RAII - when a stack allocated vector goes out of scope, the internal heap allocated array is freed.

```c++
void f() {
    vector<MyClass> v;
    ...
}

// v goes out of scope, MyClass dtor runs
// All items in the array
// Array itself is freed

// But:

void g() {
    vector<MyClass *> v;
}

// v goes out of scope,
// The pointers are deleted but the memory is not freed
```

In this case any objects pointed to by the pointers in `v` are not freed. The vector `v` has no way of knowing if it is appropriate to delete these pointers. The pointers might not be the owners of the objects they point might not be the owners of the objects they point at, the objects they point at might not even be on the heap.

So if these objects need to be freed, you'll have to do it.

```c++
void g() {
    vector<MyClass *> v;
    ...
    for (auto &n: v) { delete n; }
}

void h() {
    vector<shared_pointer<MyClass> > v;
    ...
}

// v goes out of scope, shared pointer
// dtor frees MyClass objects if appropriate, array is freed.
```

In this case we don't hace to do amy explicit deallocation.

Consider the method:
`vector<T>::emplace_back

- offers strong guarantee
- `if (size == cap)`
    - allocate new layer array
    - copy the objects over (copy ctor)
        - if a copy ctor throws
            - delete the new array
            - old array is still intact
        - strong guarantee
    - delete the old array

But copying is expensive, if we're going to throw the old data away anyway, why not move the objects to the new array?

- Allocate a larger array
- Move the objects over (move ctor)
- delete the old array

The problem is if the move ctor throws, then `vector<T>::emplace_back` can no longer offer the strong guarantee because the original array is not longer intact. But `emplace_back` **DOES** offer strong guarantee.

Therefore, `vector<T>::emplace_back` will use the move ctor if it promises not to throw any exceptions, otherwise it uses the copy ctor, which may be slower.

Therefore your move operations should provide the no throw guarantee and they should indicate that they do.

```c++
class MyClass {
public:
    MyClass(MyClass &&other) noexcept {

    }
};
```

If you know a function will never throw, or propagate an exception, declare it `noexcept`, this will allow for optimisation of yout code. At the very least, you move operation should be `noexcept`

## Casting

In C:

```c++
Node n;
int *ip = (int *) &n;
```

This is a cast, forces the compiler to lie to itself about types and treat a `Node *` like an `int *`

Casts should be avoided, and in particular, C-Style casts should be avoided. If you **MUST** cast, use C++ casts.

C++ has four _horrible flavours_ of casts:

### Static Cast

`static_cast` is for "sensible casts", with well defined behaviour

```c++
double d;
int i = static_cast<int>(d);

Book *b = new Text{...};
Text *t = static_cast<Text *>(b);
```

You are taking responsibility for the fact that `b` really does point at a text object.

### Reinterpret Cast

It is for unsafe, system/implementation dependent, _weird_ conversions. Nearly all uses of a `reinterpret_cast` result in undefined behaviour.

```c++
Student s;
Turtle *t = reinterpret_cast<Turtle *>(&s);
// force this student to be treating like a turtle
```

### Const Cast

<mark>"It's sole goal is to fuck shit up"</mark>

It is for converting between `const` and `non cost`. It is the only c++ cast that can "cast away const"

```c++
void g(int *p);

void f(const int *p) {
    ...
    g(const_cast<int *>(p));
    ...
}
```

### Dynamic Cast

Is it safe to convert from a `Book *` to a `Text *`?

```c++
Book *pb = ...;
Text *pt = static_cast<Text *>(pb);

// depends on what pb actually points at. Would be better to tentativey try to cast, and see if it succeeds.

Book *pb = ...;
Text *pt = dynamic_cast<Text *>(pb);
```

If the cast works, `pt` points at the same thing as pb, if not it is the `nullptr`.

```c++
if (pt) cout << pt->getTopic();
else cout << "Not a text";
```

Note: Dynamic casting only works on classes that at least one virtual function. But these casts have been operating on raw pointers, can we cast smart pointers?

Yes!

```c++
static_pointer_cast
const_pointer_cast
dynamic_pointer_cast
```

Thes functions (defined in `<memory>`) cast shared pointers to shared pointers

We can use dynamic casting to make decisions based on an objects run time type information (RTTI)

```c++
void WhatIsIt(shared_ptr<Book> b) {
    if (dynamic_pointer_cast<Comic>(b)) {
        cout << "Comic";
    } else {
        cout << "Book";
    }
}
```

Code like this is highly coupled to the Book class hierarchy, and may (probably) indicates bad design.

Better: Use virtual methods or visitor (if possible)

Dynamic Casting also works on references.

```c++
Text t {...};
Book &b = t;
...
Text &t2 = dynamic_cast<Text &>(b);
```

If `b` is a reference to a text object then `t2` will be assignmed to be aa reference to that same text.

What if `b` doesn't actually refer to a text?

No such thing as a null reference, we can't continue.

In that case, dynamic cast will throw a `bad_const` exception.

With dynamic reference casting, we can now solve the polymorphic assignment problem.

```c++
Text &Text::operation=(const Book &b) {
    const Text &textOther = dynamic_cast<Text &>(b);
    // throw if b is not a text.
    Book::operator=(other);
    topic = textOther.topic;
    return *this;
}
```

## How do virtual functions work?

*Get notes from Yatin*

*vtables*

*Dynamic Casting*

### Steps in calling a virtual

1. Follow the vtable ptr to the vtable
2. Fetch the appropriate fn ptr from table
3. Execute that fn through the ptr

This means that the decision is made based on the run time type because the compiler stored the appropriate vtable ptr in the object when creating it

Therefore, virtual method incur a small overhead cost.

Also, declaring at least one virtual method means your objects all just got 8 bytes bigger.

## Concretely how does `g++` lay out your objects

In g++:

|||
|---|---|
|fields|vptr|
||...|
||...|
||...|

```c++
class A {
    int a, c;
    virtual void f();
};

class B: public A {
    int b, d;
};
```
### A Object

||
|---|
|vptr|
|a|
|c|

### B Object

||
|---|
|vptr|
|a|
|c|
|b|
|d|

## Multiple Inheritance

A class can inherit from more than one base classes

```c++
class A {
public:
    int a;
};

class B {
public:
    int b;
};

class C: public A, public B {
public:
    void f() {
        cout << a << b << endl;
    }
};
```

*Picture from November 28*

```c++
D obj;
obj.a = 5; // which a is this??
```

This access is ambiguous, so the compiler rejects it. We don't throw if its the `a` hat comes from `B` or our `C`. So we must specify.

`obj.B::a or obj.C::a;`

But is this really what we want? It may be that we do want to distinct `A` components. But more often we probably only want one copy of `A` in our `d`.

This is called the **the deadly diamond of death**

*Picture from November 28*

If we do want only one `A` component. Make `A` a virtual base class and employ `virtual` inheritance.

```c++
class B: virtual public A {...};
class c: virtual public A {...};
```

Example: I/O Stream Hierarchy

*Picture from November 28*

Consider A, B, C, D example usinig virtual inheritance

||
|---|
|vptr|
|A fields|
|B fields|
|C fields|
|D fields|

This should look like a `A*` `B*` `C*` and `D*`

But it doesn't!

So what does g++ do?

|||
|---|---|
|vptr|<- ptr to a B or a D (not quite)|
|B fields|
|vptr|<- ptr to a C (not quite)|
|C fields|
|D fields|
|vptr|<- ptr to a A|
|A fields|

B needs to be laid out. So it can find it's A part, but the offset to the A component cannot be known (because you don't know how many classes there are in the hierarchy and the more there are, the greater offset is)

Observe the layout of a simple of a simple B object with virtual inheritance. Remember it has no idea C and D exist.

Thus the distance between the B part and the A part is unknowable at compile time.

Solution: the location of the base class part of the object is stored in the vtables (hence virtual inheritance)

The diagram doesn't look like an A, B, C, D object simultaneously, but slices of it to resemble A, B, C, D.

So what happens is that pointer assignment among A, B, C, D pointers changes the address stored in the ptr.

D *d; // points at the D part
A *a = d; // changes the address to point at the a part

Under multiple inheritance, `static_cast`, `const_cast`, `dynamic_cast` under multiple inheritance exhibit this behaviour of adjusting the pointers value, `reinterpret_cast` does NOT, and this should exemplify some of the many risks inherent in using this cast.

## Template Functions

```c++
template <typename T>
T min(T x, T y) {
    return x < y ? x : y;
}

// In order to use this function

int f() {
    int x = 1, y = 1;
    int y = min(x, y);
    return z;
}
```

In this case, there's no need to say `min<int>(x, y)`. The compiler can figure it out, based on the types of `x` and `y`.

If the compiler is unable to do type inference, you can always explicitly paramaterize the template type.

char w = min('a', 'c'); // T = char
auto f = min(1.0, 3.0); // T = double

### For what types `T` can min be used?

For any type where the `operator<` is defined.

Recall:

```c++
void foreach(abstractIterator &start, &end, int(*f)(int)) {
    while (start != end) {
        f(*start);
        ++start;
    }
}
```

This could work so long:
- `AbstractIterator` supports `operator*`, `++` and `!=`
- `f` can be called as a function.

So we can generalize with templates

```c++
template <typename Iter, typename Func>
void foreach(Iter start, Iter end, Func f) {
    // exactly as before
}
```

Now `Iter` can be anytype supporting those operators, including raw pointers.

```c++
void f(int n) {
    cout << n << endl;
}
...
int a[] = { 1, 2, 3, 4, 5 };
...
foreach(a, a+5, f);
```

## The Algorithm Library (STL)

A suite of template functions, many of these work over Iterators

`#include <algorithm>`

Examples: `for_each`

```c++
template <typename Iter, typename T>
Iter find(Iter begin, Iter last, const T &val) {
    // returns iterator to the first element in [begin, last)
    // if it matches val
    // returns last if not
}
```

Other functions:

- `count` - like find but returns the number of occurences of values in [begin, last)
- `copy`
- `transform` - like copy, but also applies a function

```c++
template <typename InIter, typename OutIter>
OutIter copy(InIter begin, InIter last, OutIter result) {
    // copies one container range from [begin, last) to the containter that result points to
}
```

**Note:** Does **NOT** allocate more space if needed in the container result points at, you must make sure you have enough.

Example:

```c++
vector<int> v{1,2,3,4,5,6,7};
vector<int> w(4);
// creates a vector with space for 4 ints
copy(v.begin(), v.begin() + 5, w.begin());

// -------------------

template <typename InIter, typename OutIter, typename Func>
OutIter transform(InIter first, InIter last, OutIter result, Func f) {
    while (first != last) {
        *result = f(*first);
        ++first;
        ++result;
    }
    return result;
}
```

Focusing on transform, just how generalized is out template code?

1. What can we use for func?
2. What can we use for InIter/OutIter

So Func needs to be callable as a function. WHat other than functions can provide that behaviour?

We can overload `operator()` for our classes

```c++
class Plus {
    int m;

public:
    Plus(int m) : m{m} {}
    int operator(int n) { return m + n; }
}

transform(v.begin(), v.end(), w.begin(), Plus{5});

The big advantage of using function objects over functions is that they can maintain state

class IncreasingPlus {
    int m = 0;
public:
    int operator()(int m) {
        return n + (m++);
    }

    void reset() {
        m = 0;
    }
}
```

How many ints in a vector are even?

```c++
vector<int> v{...};

bool even(int n) {
    return !(n % 2);
}

int x = count_if(v.begin(), v.end(), even);
```

## Lambdas

```c++
int x = count_if(v.begin(), v.end(), [](int n) {
    return !(n % 2);
});
```

## Iterators

An iterator is anything that supports these operators: `*`, `++` and `!=`

So that means that we can apply the notion of iterators to other no container types, such as streams.

Example:

```c++
#include <iterators>

vector<int> v {1,2,3,4,5};

ostream_iterator<int> out {cout, ", "};

copy(v.begin(), v.end(), out);
// prints 1, 2, 3, 4, 5, 
```

Consider:

```c++
vector<int> v {1, 2, 3, 4, 5};
vector<int> w;

copy(v.begin(), v.end(), w.begin());
// WRONG!
```

Remember, copy can't allocate new space for you since it doesn't even know what `w` is iterating over.

But what if we had a specialized iterator whose assigment operator inserts a new item?

```c++
vector<int> v {1,2,3,4,5};
vector<int> w;

copy(v.begin(), v.end(), back_iterator(w));
```

Now `v` is copied to `w` with space allocated as needed.

> Take the time to learn to work with Iterators and the algorithm library, and get confortable with them (also the rest of the STL). They can dramatically reduce the length of your code and reduce opportunities for bugs in your programs.

## Template Meta Programming

Offload computation to the compiler

```c++
template<int n>
struct Fact {
    enum { val = Fact<n - 1>::val * n };
};

template<>
struct Fact<0> {
    enum { val = 1 };
}

int main() {
    // runs in O(1) !!!
    int n = Fact<10>::val;
}
```

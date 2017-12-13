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

Resource Acquisition Is Initialisation

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

What does it mean to copy a unique ptr? Well, its nonsense - its supposed to be unique. We don't want to delete the same ptr twice. So copying is disallowed for `unique_ptr`s - They can only be moved.

If you need to be able to copy ptrs, Use `std::shared_ptr`

```c++
void f() {
    auto p1 = std::make_shared<MyClass>();
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

Because copy assignments could throw, we don't yet have exception safety. It would be better if we could guarantee that the `swap` part was a no-throw operation. A non-throwing swap is at the heart of writing exn safe code.

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
    vector<shared_ptr<MyClass> > v;
    ...
}

// v goes out of scope, shared pointer
// dtor frees MyClass objects if appropriate, array is freed.
```

In this case we don't hace to do any explicit deallocation.

Consider the method:
`vector<T>::emplace_back`

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

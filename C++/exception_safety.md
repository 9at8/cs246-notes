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
- If `b.method2()` throws, the effects of a method1 must 
be undone to offer the strong safety

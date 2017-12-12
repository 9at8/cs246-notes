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

Consider A, B, C, D example using virtual inheritance

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

`B` needs to be laid out. So it can find it's `A` part, but the offset to the `A` component cannot be known (because you don't know how many classes there are in the hierarchy and the more there are, the greater offset is)

Observe the layout of a simple of a simple `B` object with virtual inheritance. Remember it has no idea `C` and `D` exist.

Thus the distance between the `B` part and the `A` part is unknowable at compile time.

Solution: the location of the base class part of the object is stored in the vtables (hence virtual inheritance)

The diagram doesn't look like an `A`, `B`, `C`, `D` object simultaneously, but slices of it to resemble `A`, `B`, `C`, `D`.

So what happens is that pointer assignment among `A`, `B`, `C`, `D` pointers changes the address stored in the ptr.

```c++
D *d; // points at the D part
A *a = d; // changes the address to point at the a part
```

Under multiple inheritance, `static_cast`, `const_cast`, `dynamic_cast` under multiple inheritance exhibit this behaviour of adjusting the pointers value, `reinterpret_cast` does NOT, and this should exemplify some of the many risks inherent in using this cast.

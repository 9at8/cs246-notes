# Compilation Dependency

When does a compilation dependency exist, i.e. when does a file really need to include another?

```c++
class A {
  ...
};

// Has a true compilation dependency
// `B` needs to know the size of A
class B : public A {
  ...
};

// Has a true compilation dependency
// `C` also needs to know the size of A
class C {
  A myA;
  ...
};

// Does not have a true compilation dependency
// `A*` is a pointer; We don't need to know the size
class D {
  A* myA;
  ...
};

// Does not have a true compilation dependency
// `A` is a type signature. Not an implementation. We don't need to know the size
class E {
  A f(A a);
  ...
};
```

Which of these need to include `a.h` and which only need a forward declare?

Classes B and C have compilation dependencies because in order for the compiler to know how large `B` and `C` are, it needs to know the size of `A`

Classes D and E do not, the compilaer knows the size of a pointer for `a`, and function prototypes are only used for typechecking purposes.

If there is no compilation dependency, then do not include the file, just forward declare.

If `A` changes, `B` and `C` also need to be recompiled. But `D` and `E` don't.

Now in the implementation files a true compilation dependency is almost certain.

```c++
void D::f() {
  myA->somefunc();
}
```

Do the includes in your `cc` files whenever possible.

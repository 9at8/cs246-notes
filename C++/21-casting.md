# Casting

In C:

```c++
Node n;
int *ip = (int *) &n;
```

This is a cast, forces the compiler to lie to itself about types and treat a `Node *` like an `int *`

Casts should be avoided, and in particular, C-Style casts should be avoided. If you **MUST** cast, use C++ casts.

C++ has four _horrible flavours_ of casts:

## Static Cast

`static_cast` is for "sensible casts", with well defined behaviour

```c++
double d;
int i = static_cast<int>(d);

Book *b = new Text{...};
Text *t = static_cast<Text *>(b);
```

You are taking responsibility for the fact that `b` really does point at a text object.

## Reinterpret Cast

It is for unsafe, system/implementation dependent, _weird_ conversions. Nearly all uses of a `reinterpret_cast` result in undefined behaviour.

```c++
Student s;
Turtle *t = reinterpret_cast<Turtle *>(&s);
// force this student to be treated like a turtle
```

## Const Cast

> It's sole goal is to fuck shit up

It is for converting between `const` and `non cost`. It is the only c++ cast that can "cast away const"

```c++
void g(int *p);

void f(const int *p) {
    ...
    g(const_cast<int *>(p));
    ...
}
```

## Dynamic Cast

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

If `b` is a reference to a text object then `t2` will be assignmed to be a reference to that same text.

What if `b` doesn't actually refer to a text?

No such thing as a null reference, we can't continue.

In that case, dynamic cast will throw a `bad_const` exception.

With dynamic reference casting, we can now solve the polymorphic assignment problem.

```c++
Text &Text::operation=(const Book &b) {
    const Text &textOther = dynamic_cast<Text &>(b);
    // throw if b is not a text.
    Book::operator=(b);
    topic = textOther.topic;
    return *this;
}
```

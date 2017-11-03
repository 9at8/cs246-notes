# Oct 31 - Get Rob's notes for 26th Oct

```c++
class Book {
  ...
public:
  ...
  virtual void isItHeavy() {
    return numpages > 200;
  }
};

class Comic : public Book {
  ...
public:
  void isItHeavy() override {
    return numpages > 40;
  }
}
```

How to make a comic object act like a comic object regardless of the pointer or reference type it is accessed through?

- Solution is to declare the `isItHeavy` method as `virtual`

With `virtual` methods, we choose when class's method to run, based on the actual type of the object, not based on the type of whatever we're using to access that object.

## Example

```c++
// My book collections
Book *MyBooks[20];
...
for (int i = 0; i < 20; ++i) {
  cout << MyBooks[i]->isItHeavy;
}
```

This code for iterating the array accomodates multiple types under one abstraction(Book). This is called Polymorphism, "many forms"

### Notes

- This way we have been able to write functions that take in `istream&` and pass them an `ifstream&`, because an `ifstream` is an `istream`
- Never use arrays of objects polymorphically. If you need polymorphic arrays, use an array of ptrs.

## Destructor Revisited

```c++
class X{
  int *x;
  public:
  X(int x) : X{new int {x}} {}
  ~X() { delete x; }
}

class Y{
  int *y;
  public:
  Y(int x, int y) : X{new int {x}}, y{new int{y}} {}
  ~Y() { delete y; }
}

Y *py = new Y{1,2,3};
delete py;  // this is fine
X *px - new Y{1,2};
delete px; // leak the y position of the object
```

How can we ensure deletion through a base class ptr calls the right destructor? Make it virtual.

Always declare your destructor virtual when you have inheritance.

- If a class `Y` is not meant to ever be derived, declare `final` - `class Y final : public X { ... }`

## Pure Virtual and abstract classes

```c++
class Student {
  protected:
    int numCourses;
  public:
    // declares Student Fees as pure virtual
    virtual int fees()=0;
};

class Regular final : public Student {
  public:
    int fees() override {
      // calc fees for reg stud
    }
};

class Coop final : public Student {
  public:
    int fees() override {
      // calc fees for coop stud
    }
};
```

What should `Student::Fees` do?

Nothing. It makes no sense to create a student object, all students are either regular or coops. So we explicitly give `Student::fees` no implementation by setting it equal to 0, declaring it pure virtual.

```c++
Student s; // error
Student *reg = new Regular{ ... }; // okay
```

Any class with at least 1 pure virtual method, cannot be instantiated as an object. Such a class is called an **Abstract Class**

Subclasses of an abstract class must implement the pure virtual methods, or else they're also abstract classes.

A class that is not an abstract class is called a **Concrete Class**.

In UML, an abstract class and virtual methods are denoted by _italicizing_ their names.

## Copy and Move ctors/ops with inheritance

```c++
class Book {
  ...
  public:
  // defines copy/move fns
};

class Text : public Book {
  ...
  public:
  // does not define copy/move fns
};

Text t{"algorithms 1", "A programmer", 250, "Algorithms"};
Text t2 = t; // What happens??
```

This just does the build in behaviour, which just copy constructs our base class component with other, and then copy constucts all fields with the fields of other. This is okay in this case, but it may not always be.

To define your own:

```c++
Text::Text(const Text &other) : Book{other}, topic{other.topic} {}

Text &Text:operator=(const Text &o) {
  Book::operator=(o);
  topic = o.topic;
  return *this;
}

Text::Text(Text &&other) : Book{std::move(other)}, topic{other.topic} {}

Text &Text:operator=(const Text &o) {
  Book::operator=(std::move(o));
  topic = o.topic;
  return *this;
}
```

**Note**: Even though the thing other (ot o) reference is an r value, it itself is not. It has a name! So use `std::move(o)` on it to return an rvalue version of it.

The funcs above are same as the built in behaviour, adapt accordingly for Node, etc.

Now, consider:

```c++
Text t1{ ... }, t2{ ... };
Book *pb1 = &t1, *pb2 = &t2;
*pb1 = *pb2;
// this calls Books Copy =; incomplete assignment.
```

That's a partial assignment! We didn't assign the topic fields.

How can we fix it? We could declare it virtual...

```c++
class Book {
  ...
  public:
    virtual Book& operator=(const &o) {
      ...
    }
}

class Text : public Book {
  ...
  public:
    Text& operator=(const Book &o) {
      Book::operator=(o);
      topic = o.topic;
      return *this;
    }
};

// if assigning a text to a book call this function.
```

We are damned if we do and damned if we don't. So we don't create concrete superclasses

```c++
class AbstractBook {
  int numpages;
  string title, author;

protected:
  AbstractBook& operator=(const AbstractBook &o) ...

public:
  AbstractBook(...) ...
  virtual ~AbstractBook()=o;
}
```

Even though we declared `~AbstractBook` pure virtual, we still need to provide an implementation or else our program won't link.

```c++
// AbstractBook.cc
AbstractBook::~AbstractBook() { }
```

For other classes, headers/code would be similar, adapt accordingly.

This design prevents partial assignment and mixed assignments.

**Don't make concrete superclasses, make them abstract**

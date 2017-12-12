# Static Members

A static member is a member of a class whose existence is tied to that class, not to any one individual object.

What if we want to create of type T, where T is some class?

```c++
// student.h

struct Student {
  int assns, mt, final;
  static int numObject;
  Student(int assns, int mt, int final) :
    assns{assns},
    mt{mt},
    final{final} {
    ++numObject;
  }
}

// student.cc
int Student::numObject = 0;

int main() {
  cout << Student::numObject << endl;
  Student s{85, 60, 90};
  cout << Student::numObject << endl;
}
```

We not only have static fields, we also have static members

```c++
class Student {
  int assns, mt, final;
  static numObjects;

public:
  static int getNumObjects() { return numObjects; }
}
```

**Note**: `static` methods receive no `this` pointer as they can't reference non static fields, they aren't declared in that scope!

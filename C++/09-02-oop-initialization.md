# Initialization

```c++
Student billy{60, 70, 80};
// This is fine, but is limited
```

Better thing to have: a function that does initialization: Constructor

```c++
struct Student {
  int assms, mt, final;
  float grade() { ... }
  Student(int assms, int mt, int final) {
    this->assms = assms;
    this->mt = mt;
    this->final = final;
  }
}

Student Billy{60, 70, 80}; // better!
```

If the constructor has been defined, the members above are passed as arguments to the constructor. If no constructor has been defined, these initialize the individual files of Student. Alternatively, `Student Billy = Student{60, 70, 80};` does the same thing.

## Uniform Initialization
Sometimes, variables are initialized with `=`

```c++
int x = 9;
string s = "hello";

// We can also initialize by (), especially when invoking constructors

int x(5);
string s("hello");
Student Billy(60, 70, 80);
```

The rules involving when we may use `=` v/s when we may use `()` are somewhat arbitrary. C++ now has uniform initialization syntax, using `{}`, which is meant to be used in (nearly) all initialization cases.

```c++
int x{5};
string s{"hello"};
...
...
```

This is the **preferred** syntax

## For allocation on the heap

```c++
Student billy = new Student{60, 70, 80};
```

## Advantages of using a constructor
- Default params
- Overloading
- Sanity checks
- Logic other than simple field init

The built in constructor goes away if you define any constructor

```c++
struct Vec {
  int x, y;
  Vec (int x, int y) {
    this->x = x;
    this->y = y;
  }
}

Vec v1{1,2};
// calls the ctor we gave

Vac v2;
// error, no matching fn defn
```

## What happens when an object gets created?

What if a structure contains `const` or a reference variable? Does every instance of my struct need to have the same values?

Where do we initialize? Ctor body? Too late for that.

1. Space is allocated
2. Fields are constructed (need to put our initializations here)
3. ctor body runs

How do we do this? Using the **Member Initialization List (MIL)**

```c++
struct Student {
  const int id,
  int assms, mt, final,
  Student(int id, int assms, int mt, int final) : id{id}, assms{assms}, mt{mt}, final{final} {
    // blank
  }
}
```

### Notes
- You can and should initialize any field in the MIL, not just const and refs.
- Fields listed in the MIL are initialized in the order in which they're declared in the struct, regardless of the ordering in MIL.
- Member initializer lists allow us to initialize our members rather than assign values to them. This is the only way to initialize members that require values upon initialization, such as const or reference members, and it can be more performant than assigning values in the body of the constructor. Member initializer lists work both with fundamental types and members that are classes themselves.

**_Next lecture (missed - Oct 5 2017 thursday):_**
  - destructor

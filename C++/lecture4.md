# CS 246


## Operator Overloading
We can give meanings to C++ operators for types that we define

```c++
struct Vec {
  int x, y;
}

Vec operator+ (const Vec &lhs, const Vec &rhs) {
  Vec V { lhs.x + rhs.y, lhs.y + rhs.y };
  return V;
}
```

### Overloading `<<` and `>>` (I/O)
```c++
struct Grade {
  int theGrade;
}

ostream& operator>> (ostream &out, const Grade &g) {
  out << y.theGrade << "%";
  return out;
}

istream& operator>> (istream &in, Grade &g) {
  in >> g.theGrade;
  if (g.theGrade > 100) g.theGrade = 100;
  if (g.theGrade < 0) g.theGrade = 0;
  return in;
}
```


## The pre-processor
It transforms your C++ program before the compiler

```c++
// General form of a preprocessor directive
#.....

// Example: include
#include<....>
```

When a preprocessor sees `#include` file, it copies the contents of that file into your program

To include old `c` headers:
```c++
#include <cstdio>
```

### `#define VAR VAL`
Sets a preprocessor variable. Then all occurrences of `VAR` in your program are replaced with `VAL`

```c++
#define MAX 10

int x[MAX]

// It's basically like replacing all instances of VAR with VAL
```

There is no good reason to do this in C++, use const definition instead.

`#define FLAG` sets the variable flag to empty string

Defined constants are useful for conditional compilation

```c++
#define ANDROID 1
#define IOS 2
#define OS ANDROID

#if OS == ANDROID
long long int publicKey;  // suppressed if OS is not ANDROID
#elif OS==IOS             // spacing doesn't matter
short int publicKey;      // suppressed if OS is not IOS
#endif

```

Special case
`#if 0` is never true, so all the inner text is removed before the compiler sees it. Acts as a comment.

We can also define symbols via a compiler flag

```c++
int main() {
  cout << X <<< endl;
}
```

This won't work on it's own. We can use the `-D` flag to define `X`

```bash
$ g++ -DX=15 filename.cc
```

#### Checking if something is defined or not `#ifdef` and `#ifndef`
```c++
int main() {
  #ifdef DEBUG
  cout << "Setting x to 1" << endl;
  #endif
  
  int x = 1;
  while (x < 10) {
    ++x
    #ifdef DEBUG
    cout << "x is now " << x << endl;
    #endif
  }
}
```


## Separate Compilation
Split program into composable modules. Each of these modules provides:
  - Interface: type definitions, and prototypes for functions (.h file)
  - Implementation: full definition for every provided function

```c++
// vec.h
struct Vec {
  int x, y;
};

Vec operator+(const Vec &v1, const Vec &v2);
```

```c++
// vec.cc
#include "vec.h"

Vec operator+ (const Vec &v1, const Vec &v2) {
  // ...
``// ...
}
```

What about `main()`?

```c++
// main.cc (client)
#include "vec.h"

int main() {
  Vec v { 1, 2 };
  v = v + v;
}
```

Recall: An entity can be declared as many times as you want, but defined at most once

**Compiling Seperately**
```bash
$ g++ -c vec.cc
$ g++ -c main.cc
$ g++ vec.o main.o -o main
# This 'links' the files together into an executable ???
```

Compiler flag `-c` "compile only" supresses linking, does not attempt to build executables, produces an object file (`.o`)

**What if we want to provide a global var?**
```c++
// abc.h
int globalVar;
```

Problem: every file that includes `abc.h` defines a separate `globalVar`, program will not link.

Solution: put the definition in `cc` file, and declare only in the header file.

```c++
// abc.h
extern int globalVar;

// abc.cc
int globalVar;
```


## Include guard
Need to prevent files from being included more than once.

```c++
// vec.h
#ifndef _VEC_H_
#define _VEC_H_

...
...
...

#endif
```

The first time vec.h is included, the symbol `_VEC_H_` is not defined, so the files contents are included and the symbol is defined.

  - We should **always** wrap all the header files with an inclusion guard
  - **NEVER** put `using namespace std;` in `.h` files, as the using directive will be forced upon any client who includes the file
  - Always refer to to `cin`, `cout`, `end`, `string`, etc as `std::cin`, `std::cout`, etc in header files

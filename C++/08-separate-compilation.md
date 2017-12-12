# Separate Compilation

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
  // ...
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
# This 'links' the files together into an executable
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

We need to prevent files from being included more than once.

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

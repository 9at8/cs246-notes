# The pre-processor

It transforms your C++ program before the compiler

```c++
// General form of a preprocessor directive
#.....

// Example: include
#include <....>
```

When a preprocessor sees `#include` file, it copies the contents of that file into your program

To include old `c` headers:

```c++
#include <cstdio>
```

## `#define VAR VAL`

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

**Special case**: `#if 0` is never true, so all the inner text is removed before the compiler sees it. Acts as a comment.

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

### Checking if something is defined or not `#ifdef` and `#ifndef`

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

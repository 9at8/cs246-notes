# C++
C++ is a compiled language (on student environment)

To compile:
```bash
g++ -5 -std=c++ program.cc -o program
```


## Hello World in C
```c
#include <stdio.h>

int main() {
  printf("Hello World\n")
  return 0;
}
```


## Hello World in C++
```c++
#include <iostream>
using namespace std;

int main() {
  cout << "Hello World" << endl;
  return 0;
}
```


## Notes:
  - Main must return type `int` in c++
  - `stdio.h`, `printf` available in c++, `<cstdio>`
  - Preferred C++ I/O header `<iostream>`
  - Return statement, returns status code to shell `$?`; If left out, returns 0

## Input/Output Streams
  1. `cout` for printing to `stdout`
  2. `cin` for reading from `stdin`
  3. `cerr` for printing to `stderr`

### Operators
  1. `<<` means put to (output): example: `cerr << x`
  2. `>>` read (input): exmaple `cin >> x`

  - The operator is **pointing** in the flow of information

## Examples
  1. Add two numbers
```c++
#include <iostream>

using namespace std;

int main() {
  int x, y;
  cin >> x >> y;
  cout << x + y << endl;
  return 0;
}
```

Note:
  - What if an input doesn't contain an integer next?
      Statement fails, value of either 0, min int or max int is assigned to the variable
  - What if input is exhausted before reading two integers? Same thing.
  - If read fails: `cin.fail()` will be true
  - If `EOF`: `cin.eof()` and `cin.fail()` will both be true, but not until the attempted read fails
  - There is an inplicit conversion from `cin` to a boolean, `cin` evaluates to true if neither the fail or bad bits are set otherwise false, so `cin` is true if reads haven't failed.
```c++
int main() {
  int i;
  while (true) {
    cin >> i;
    if (!cin) break;
    cout << i << endl;
  }
}
```
  - The operator `>>`, when used for input, has `cin` (type istream) as its first operand, and data (could be of seven different possible types) as its second operand
  - `(cin >> x) >> y >> z` return the stream given (`cin`)

  2. Read all ints from stdin until `EOF`, ignoring bad inputs.
```c++
int main() {
  int i;
  while (true) {
    if (!(cin >> i)) {
      if (cin.eof()) break; // at EOF, we're done
      cin.clear()           // clears the fail bit so cin.fail() is false
      cin.ignore()          // skips over the offending character in the stream
    } else {
      cout << i << endl;
    }
  }
}
```


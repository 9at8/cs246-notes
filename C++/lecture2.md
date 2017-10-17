# Lecture 2 - C++ - CS 246


Read about Stream Abstraction and StringStreams

## Strings
### In `C`
  - Array of chars, terminated with \0
  - Must explicity manage memory, allocate more mem as string grows
  - Easy to overwrite \0 and corrupt memory

### In `C++`
  - Grows as needed
  - Safer to manipulate
  
  `string s = "hello";`
  
  Even in C++, the string literal "hello" represents a C-style string, i.e. a null terminated char array.
  
#### Operations
  - `==` and `!=`
  - `<=`, `<`, etc
  - Fetch individual chars: `s[i]`
  - Concatenation: `+`
  - More online!


## Default function Parameters
```c++
void printSuiteFile(string name="suite.txt") {
  ifstream file {name};
  string s;
  while (file >> s) cout << s << endl;
}

printSuiteFile("greg.txt"); // prints greg.txt
printSuiteFile(); // prints default thing
```

## Overloading
```c++
int neg(int n) { return -n; }
int neg(bool b) { return !b; }
```

Another example:
The `+` operator

Compiler uses the number and type of args to decide which neg to call overloads

**MUST** differ in number of args or type. They cannot differ in just return type

```c++
int add(int x, int y=10) {...}
int add(int x) {...}

// This is invalid
```


## Structures
```c++
struct Node {
  int data;
  Node *next;
}

// In C we have to do 'struct Node *next;' instead of 'Node *next;' in C++
```


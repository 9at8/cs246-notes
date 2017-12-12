# Default function Parameters

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

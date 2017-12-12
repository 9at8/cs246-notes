## STL Maps - for creating dicts

Example: For creating an `array` that can be indexed by an arbitrary type, i.e. a `string`

```c++
#include <map>
using namespace std;

map<string, int> m;
m["abc"] = 1; // if key doesn't exist, make it, and assign it 1
m["df"] = 4;

cout << m["abc"] << endl; // prints 1
cout << m["ghi"] << endl; // ? -> prints 0

// if a key doesn't exist, it is created and is
//  default initialized

m.erase("abc"); // removed key/value pair

if (m.count("abc")) { // returns 0 or 1
  // do something
}

for (auto &p : m) {
  cout << p.first << "," << p.second << endl;
}

// prints in order based on the key values
// p's type here is a std::pair<string, int>
```

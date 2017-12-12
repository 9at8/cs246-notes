### Dynamic size arrays - Vector

```c++
#include <vector>

using namespace std;

vector <int> v{4, 5};
// creates the array [4, 5]
// calls the list initialisation ctor

vector <int> v2(4, 5);
// This calls vector(int, T)
// v(4, 5) creates [5, 5, 5, 5]

v.emplace_back(6); // 4,5,6
v.emplace_back(7); // 4,5,6,7

for (int i = 0; i < v.size(); ++i) {
  cout << v[i] << endl;
}
```

Vector implements the iterator pattern

```c++
for (vector<int>::iterator it = v.begin(); it != v.end(); ++i) {
  cout << *it << endl;
}

// or this

for (auto n : v) {
  cout << *it << endl;
}

// or the reverse iterator

for (vector<int>::reverse_iterator it = v.rbegin(); it != v.rend(); ++it) {
  cout << *it << endl;
}
```

`v.pop_back()` removes the last element of `v`

Vectors are generated to be implemented as an array, so from now on, if you use a dynamic array, use vector. There is no more reason to use the array forms of new[] and delete[]

Other vector operations are based on iterators such as `erase`, which erases the element pointed to by the iterator

```c++
auto it = v.erase(v.begin());
// erases the item as index 0, returns the iterator to the first item after the erase

it = v.erase(v.begin() + 3);
// erases the item at the 3rd index

it = v.erase(it);
// erases the item pointed to by it

it = v.erase(v.end() - 1);
// erases the last item on our list
```

# Operator Overloading

We can give meanings to C++ operators for types that we define

```c++
struct Vec {
  int x, y;
}

Vec operator+(const Vec &lhs, const Vec &rhs) {
  Vec V { lhs.x + rhs.y, lhs.y + rhs.y };
  return V;
}
```

## Overloading `<<` and `>>` (I/O)

```c++
struct Grade {
  int theGrade;
}

ostream& operator<<(ostream &out, const Grade &g) {
  out << y.theGrade << "%";
  return out;
}

istream& operator>>(istream &in, Grade &g) {
  in >> g.theGrade;
  if (g.theGrade > 100) g.theGrade = 100;
  if (g.theGrade < 0) g.theGrade = 0;
  return in;
}
```

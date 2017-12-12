# Final review session

## Q4: Big 5 Revisited, Smart Pointers

```c++
Node(int data, unique_ptr<Node> &next)
: data{data}, next{std::move(next)}, id{this->next ? this->next->id : nullptr} {
    
}

// Copy ctor
Node(const Node &other)
: data{other.data},
  next{other.next ? make_unique<Node>(other.next) : nullptr},
  id{other.next ? this->next->id : make_shared<int>(*other.id)} {
  
}

// Copy Assignment
Node &operator=(const Node &other) {
  // this is copy ctor and not copy assignment
  // It is the same as `Node copy{other};`
  Node copy = other;
  using std::swap;
  swap(data, copy.data);
  swap(next, cop.next);
  return *this;
}

// Move ctor
Node(Node &&other)
: data{other.data},
  next{std::move(other.next)},
  id{std::move(other.id)} {

}

// Move Assignment
Node &operator=(Node &&other) {
  std::swap(data, other.data);
  stdt::swap(next, other.next);
  stdt::swap(id, other.id);
  return *this;
}
```

Notes:

1. Private fields of an object of a particular class can be accessed inside the same class.

## Q7: vtables

```
    A
    |
  -----
  |   |
  B   C
  |
  D
```

### A object

||
|---|
|`vptr`|

### A vtable

|"A"||
|---|---|
|`f()`|-> `nullptr`|
|`g()`|-> `A::g()`|
|`~A()`|-> `A::~A()`|

### B object

||
|---|
|`vptr`|
|`x`|
|`y`|

### B vtable

|"B"||
|---|---|
|`f()`|-> `B::f()`|
|`g()`|-> `B::g()`|
|`~B()`|-> `B::~B()`|

### C object

||
|---|
|`vptr`|
|`x`|

### C vtable

|"C"||
|---|---|
|`f()`|-> `B::f()`|
|`g()`|-> `A::g()` (make an arrow on the final)|
|`~C()`|-> `C::~C()`|

### D object

||
|---|
|`vptr`|
|`x`|
|`y`|
|`z`|

### D vtable

|"D"||
|---|---|
|`f()`|-> `B::f()` (make an arrow on the final)|
|`g()`|-> `B::g()` (make an arrow on the final)|
|`h()`|-> `D::h()`|
|`~D()`|-> `D::~D()`|

## Q6: Template Functions

```c++
template<typename Fn, typename Y, typename XIter>
Y foldl(Fn f, Y init, XIter begin, XIter end) {
  for (; begin != end; ++begin) {
    init = f(*begin, init);
  }

  return init;
}

template<typename Fn, typename Y, typename XIter>
Y foldr(Fn f, Y init, XIter begin, XIter end) {
  auto next = begin;
  next++;

  // `(next == end)` can't be used because that's not a specification for the Iterator pattern
  if (!(next != end)) {
    return f(*begin, init);
  }

  return f(*begin, foldr(f, init, next, end));
}
```

## Q3: Exceptions and Exception Safety

1. Strong as `return n` could throw something.
1. No guarantee because `make_unique` could throw.
1. No guarantee because of segfaults.
1. Cases:
    1. If D is able to `move` and `move` is no throw, the function gives a strong guarantee
    1. If D is able to `copy` and `copy` is strong, the function gives a basic guarantee
    1. Otherwise no guarantee

Notes:

- Assume `move` is no throw if stated otherwise

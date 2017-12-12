# Encapsulation

Consider this

```c++
struct Node {
  int data;
  Node *next;
  Node (int data, Node *next) : data{data}, next{next} {}
  ~Node () { delete next; }
}

int main() {
  Node n1{1, new Node{2, nullptr}};
  Node n2{3, nullptr};
  Node n3{4, &n2};
}
```

But what happens when these objects go out of scope? All three are stack allocated. So all three have their dtors run. When n3's dtor runs, it tries to delete stack allocated `n2`, and the result is undefined behaviour.

So the dtor has an assumption, `next` is either a `nullptr` or heap allocated memory. This is the essence of an **invariant**, a statement that is supposed to hold true for a class, in order for it to work as advertised.

For example, consider a stack. A stack obeys the variant that the first thing to be removed off a stack was the most recent added. This fails if the user can rearrange the order of the stack's data.

It is very hard to reason about code if you can't rely on invariants.

To help enforce out invariants, we introduce encapsulation, so named bacause we treat our classes like closed capsules that wrap around their data. So the user cant access it, and implementation details are hidden from the user.

```c++
struct Vec {
  Vec(int x, int y) x{x}, y{y} {}

private:
  int x, y;

public:
  Vec operator+(const Vec &rhs) { ... }
};

// by default, members of a struct are public.
```

## Definition

Encapsulation is the idea that we hide implementation details from the client and only allow them to modify our objects through provided methods.

## Why do we need it?

We need encapsulation to be able to enforce our invariants, since otherwise if the cllient can modify our data however they please, then they can break our invariants.

It is very hard to reason about a program if you can't rely on invariants.

## Enforcing Invariants

```c++
struct Vec {
  Vec(int x, int y) : x{x}, y{y} {}

private:
  int x, y;

public:
  Vec operator+(const Vec &other) {
    return {x + other.x, y + other.y};
  }
}
```

In general, we want our fields to be private. This prevents clients from tampering with our internals and destroying invariance. Out methods (typically) should be public

The default visibility of a `class` is private and a `struct` is public. That's the only difference between `class`es and `struct`s.

## Getters and Setters

```c++
class Vec {
  int x, y;
 public:
  ...
  getX() { return x; }
  getY() { return y; }
  setX(int n) { x = n; }
  setY(int n) { y = n; }
};
```

  - If we want our client to be able to read your fields, use accessor methods (getters)
  - If we want our client to be able to modify our fields, we use mutator methods (setters)

### But what about `operator<<`?

It needs access to the fields of your class but it can't be a member function. So if you already have `getters` then we can use them. But if we don't, and we don't want to add them to the interface, declare `operator<<` as a `friend`

```c++
class Vec {
  int x, y;
 public:
  ...
  friend std::ostream& operator<<(ostream &o, const Vec &v);
  // doesn't matter if it's in a public or private.
};
```

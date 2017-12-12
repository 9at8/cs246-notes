## Exception Handling

`v[i]` returns the `ith` element of v. If you go out of bounds, vector doesn't check for you - undefined behaviour.

We can instead use `v.at(i)`, which is a checked version.

But what does it do when you go out of bounds? What should happen?

Problem: Vector code can detect the error, but doesn't know what to do about it. The client knows what they do if it occurs, but can't detect it. Error recovery, by nature is a non-local problem.

**C Solution**: fn returns a status code or if output eange completely used set a global variable. These are awkward and clunky solutions, which lead to wither bad code or ignored errors.

**C++**: when an error condition arises, the function can raise an `exception`. What happens when an exception is raised? The program terminates by default after unwinding the call stack. But, we can write handlers to resolve the exception and deal with them.

`vector<T>::at` raises an exception of `type::out_of_range`

If we want to deal with it, we include `<stdexcept>`

```c++
#include <stdexcept>
#include <vector>

...
...

try {
  cout << v.at(99999999999) << endl;
} catch (out_of_range) {
  cerr << "Index out of range" << endl;
}

...
...
// Program continues execution
```

Now, consider:
```c++
void f() {
  throw std::out_of_range("f");
}

void g() { f(); }
void h() { g(); }

int main() {
  try {
    h();
  } catch (out_of_range) {
    ...
  }
}
```

Here, `main` calls `h` which calls `g` which calls `f` and `f` throws the exception. Control goes backwards through the call chain (unwindins the stack) until an appropriate handler is found in, this case in `main`, `main` then handles the execution. If no such handler is found the program crashed.

What is `out_of_range`? It's a class, so the statement. The statement `out_of_range("f")` creates an `out_of_range` object using a single string parameter ctor. The string is for including auxillary information.

```c++
try {
  ...
  ...
} catch (out_of_range ex) {
  cout << ex.what() << endl;
  // this prints the message from the constructor
}
```

### Handlers

A handler can do part of the recovery job, i.e. execute some corrective code and throw another exn:

```c++
try {
  ...
} catch (someErrorType s) {
  ...
  throw someOtherError(...);
}
```

```c++
try {
  ...
} catch (someErrorType s) {
  ...
  throw;
}
```

Why `throw` instead of `throw s`?

The exception could actually be a type of derived class `SomeErrorType` in which case we created `s` by slicing through object. If we throw `s` we are throwing the sliced object. If we just say throw, it just throws the original exception we received.

A handler can act as a catch.

```c++
try {
  ...
  ...
} catch (...) {   // literally `...` here
  // but here we can't reference the error thrown
  ...
  ...
}
```

An exception can be anything, not just objects. It can be a `POD` type if you want. Typically though, will be a class you've made to be meaningfull ir an appropriate std exception.

```c++
class BadInput{};
...
try {
  int n;
  if (!(cin >> n)) throw BadInput{};
} catch (BadInput &b) {
  cerr << "Input not well formed!" << endl;
}
```

**Note**: The exception is caught by reference. This prevents the exn from being sliced so if we had actually caught an object of some derived class of bad input. We could get the actual correct behaviour of that object in our catch.

So the general rule of reference is that you catch by reference and throw by value.

Some std exceptions:

- `length_error`: attempting to resize strings/vectors that are too long.
- `bad_alloc`: is thrown when new fails.

### Warning!

Never let a dtor throw.

Since when an exn is thrown, any statically allocated objects inside a fn being unwound will be destroyed if their destructors throw, you will have two unhandled exceptions and the program terminates immediately.

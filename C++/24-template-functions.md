# Template Functions

```c++
template <typename T>
T min(T x, T y) {
    return x < y ? x : y;
}

// In order to use this function

int f() {
    int x = 1, y = 1;
    int y = min(x, y);
    return z;
}
```

In this case, there's no need to say `min<int>(x, y)`. The compiler can figure it out, based on the types of `x` and `y`.

If the compiler is unable to do type inference, you can always explicitly paramaterize the template type.

```c++
char w = min('a', 'c'); // T = char
auto f = min(1.0, 3.0); // T = double
```

## For what types `T` can min be used?

For any type where the `operator<` is defined.

Recall:

```c++
void foreach(abstractIterator &start, &end, int(*f)(int)) {
    while (start != end) {
        f(*start);
        ++start;
    }
}
```

This could work so long:
- `AbstractIterator` supports `operator*`, `++` and `!=`
- `f` can be called as a function.

So we can generalize with templates

```c++
template <typename Iter, typename Func>
void foreach(Iter start, Iter end, Func f) {
    // exactly as before
}
```

Now `Iter` can be anytype supporting those operators, including raw pointers.

```c++
void f(int n) {
    cout << n << endl;
}
...
int a[] = { 1, 2, 3, 4, 5 };
...
foreach(a, a+5, f);
```
# pImpl and Bridge Pattern 

## `XWindow` class

```c++
class XWindow {
  Display *d;       // This is private data
  Window w;         // We don't know what this means and we don't care
  int s;
  GC gc;
  unsigned long color[10];
  ...
}
```

What if we add or change these private fields? Our client code must now be recompiled. How can we make it so this isn't the case.

The solution is to use the pointer to implemenatation (pImpl) idiom. Create a second class XWindowImpl which stores the implementation details, `XWindow` just has a ptr to it.

```c++
// XWindowImpl.h

#include <X11/XLib.h>

struct XWindowImpl {
  Display *d;
  Window w;
  ints;
  GC gc;
  unsigned long colors[10];
}

// window.h

class XWindowImpl;

class XWindow {
  XWindowImpl *pImpl;

public:
  // no change
}

// window.cc

#include "window.h"
#include "XWindowImpl.h"
XWindow::XWindow(...)
: pImpl{new} 
```

In other methods replace fields `w`, `d`, `s` etc with `pImpl->w`, `pImpl->d` etc

If you confine all `XWindow` private fields within `XWindowImpl`, then only `Window.cc` needs to be recompiled if you change `XWindow` implementation

Generalization: What if there's a muliple types of Windows? E.ge `XWindow` and `YWindow`? Then maje the Impl struct a superclass.

<mark>UML goes here</mark>

The `pImpl` with a hierarchical impl structures is called the bridge pattern

# Midterm Review
  - Half trivia and half programming

## Programming
  - 1 bash programming question
  - Write the big five
  
### Bash question
Q: Takes a program and args on command line; Run it in Valgrind.

Will do these tasks as well:
  - If no program waas supplied, print a usage and exit with status 2
  - If there are errors, print a "Error" message and print it to std error
  - If there aren't any errors, end with status 0.

```bash
#!/bin/bash

program=$1

if [ $# -lt 1 ]
  echo "Usage: $0 [program]" 1>&2
  exit 2
fi

tmpfile=$(mktemp)

valgrind "$@" 2> "$tmpfile"

# $@ means all the arguements from $1 onwards.

egrep "0 errors in 0 contexts" "$tmpfile" &>/dev/null
egrep_out = $?

rm "$tmpfile"

if [ $egrep_out -ne 0 ]; then
  echo "Program has memory errors" 1>&2
  exit 1
fi

exit 0
```

### Binary Tree

```c++
class BinTree {
  int data;
  BinTree *left, *right;
  BinTree *parent = nullptr;
  
  public:
    ~BinTree() {
      delete left;
      delete right;
    }

    BinTree(const BinTree &other) :
      data{other.data},
      left{other.left ? new BinTree(*other.left) : nullptr},
      right{other.right ? new BinTree(*other.right) : nullptr} {
      // parents have new memory locations so we can't have the older addresses
      if (left) left->parent = this;
      if (right) right->parent = this;
    }

    // Not using MIL here because it is not a ctor
    // They might explicitly ask us to use copy and swap or they might not, we should be familiar with both.
    BinTree &operator=(const BinTree &other) {
      // remember to check for self assignment
      if (this == &other) return *this;
      
      // We can also use the copy and swap method
      // using std::swap;
      // BinTree tmp{other};
      // swap(left, tmp.left);
      // swap(right, tmp.right);
      // swap(data, tmp.data);
      // swap(parent, tmp.parent);
      
      delete left;
      delete right;
      data = other.data;
      left = other.left ? new BinTree(*other.left) : nullptr;
      right = other.right ? new BinTree(*other.right) : nullptr;
      
      if (left) left->parent = this;
      if (right) right->parent = this;
      
      return *this;
    }

    BinTree(BinTree &&other) :
      data{other.data},
      left{other.left},
      right{other.right} {
      if (left) left->parent = this;
      if (right) right->parent = this;
      other.left = nullptr;
      other.right = nullptr;
      // Alternatively
      // other.left = other.right = nullptr;
    }

    BinTree &operator=(BinTree &&other) {
      swap(left, other.left);
      swap(right, other.right);
      swap(data, other.data);
      // updateParents
      return *this;
    }
}
```


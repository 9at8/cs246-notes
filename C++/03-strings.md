# Strings

Read about Stream Abstraction and StringStreams

## In `C`

- Array of chars, terminated with \0
- Must explicity manage memory, allocate more mem as string grows
- Easy to overwrite \0 and corrupt memory

## In `C++`

- Grows as needed
- Safer to manipulate

`string s = "hello";`

Even in C++, the string literal "hello" represents a C-style string, i.e. a null terminated char array.
  
## Operations

- `==` and `!=`
- `<=`, `<`, etc
- Fetch individual chars: `s[i]`
- Concatenation: `+`
- More online!

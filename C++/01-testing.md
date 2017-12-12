# Testing
- Essential part of software development
- Ongoing
- Begins before you begin coding
- Test suites - expected behaviour
- It is not **Debugging**
- Testing can not guarantee correctness, only prove wrongness
  
## Human Testing
- Human looks over code
- Code inspection, code walkthrough
- Test suites, expected behaviour

## Machine Testing
- Run a program on a selected input, check it against specifications
- Can't check everything; choose tests carefully

|Black Box|White Box|Grey Box|
|---|---|---|
|No knowledge|Full knowledge|Some knowledge|

Start with black box testing, then supplement with white box

- Various classes of input, eg: positive/negative
- Boundaries of valid ranges, edge cases, etc
- Multiple simultaneous boundaries at some time (corner case)
- Intuition/experience - guess at likely errors
- Extreme cases
- White Box testing
  - Execute every logical path through prog
  - Make sure every function works

## Performance Testing
Is the program efficient enough?

## Regression Testing
- Make sure new changes to the program don't break old test cases
- Test suite/tesing script
- Always add tests, never take away 

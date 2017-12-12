# Measures of Design Quality

Coupling and Cohesion

## Coupling

The degree to which distinct program modules depend on each other

- (low) modules communicate through fn calls with basic params/results
- modules pass arrays/structs back and forth
- modules affect each other's control flow
- modules share global data
- (high) modules have access to each other implementation (friends)

High Coupling:

- Changes to one module require greater changes to other modules
- Harder to reuse individual modules

## Cohesion

How closely related the elements of a specific module are

- (low) Arbitrary grouping of unrelated elements (eg `<utility>`)
- Elements share a common theme, may be some reuse code, but are otherwise unrelated (eg `<algorithm>`)
- Elements manipulate the state over a lifetime of an object
- Elements pass data to each other
- (high) Elements cooperate to perform exactly one task

Low Cohesion:

- Poorly organized code
- Harder to maintain and use

Our goal is **Low coupling** and **high cohesion**

## MVC: Decouple the Interface (Model-View-Controller)

Your primary program classes should not be printing/displaying things

Example:

```c++
class ChessBoard {
  ...
  ...
  cout << "Your move" << ...
  ...
  ...
};
```

This is bad design as it inhibits code reuse. What if we want to reuse the `ChessBoard` class but not communicate via std out?

A better solution, give the class stream objects with which it can o input/output.

```c++
class ChessBoard {
  istream &in;
  ostream &out;

public:
  ChessBoard(istream &in, ostream &out) {
    ...
    ...
    out << "Your move" << ...
    ...
    ...
  }
  ...
};
```

This is better, but what if we don't want to use streams at all, eg graphics

Your ChessBoard should not be doing any communication at all.

### Single Responsibility Principle

> A class should only have one reason to change

Game state and communication are two reasons.

So, our chess board should communicate via `params` and `results` and occasionally by raising exceptions. Confine the actual user interaction outside the game class. Then you have total freedom to change how the communication is done.

*Question*: So should `main` do all the communication to/from the user/chessboard?

*Answer*: No. You may want to expand or reuse your communication code, and it's hard to reuse that's in main.

A better solution was to create a `ChessBoard` class just for the chess logic. A MUCH BETTER solution would be to create a new class that handles all user interaction. This class should be distinct from our game state class.

Separate the distinct notions of our data(state), the presentation of the data, and the control of the data.

- Model : This is the main data we are manipulating. (example `ChessBoard`)
- View : How the model is manipulated
- Controller : How the model is manipulated

*Picture from Nov 21*

### Model

- Can have multiple views (example text and graphics)
- Doesn't need to know about their details
- Can be implemented as a classic observer pattern (or could comminicate through the controller)

### Controller

- Mediates the flow of communication between the view and the model
- May encapsulate the idea of turn taking, or full-game rules (trade off with model)
- May communicate with the user for input (or could be done by the view)

By decoupling presentation and control, MVC enhances opportunities for code reuse and adaptability/customization

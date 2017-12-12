# Template Method Pattern

We want subclasses to override some behaviour but still have some common behaviour across all our derived classes.

Example: We have red turtles and we have green turtles. We want to draw our turtles:

```c++
class Turtle {
public:
  void draw() {
    drawhead();
    drawshell(); // virtual
    drawfeet();
  }

private:
  void drawhead() {...}
  void drawfeet() {...}
  virtual void drawshell() = 0;
};

class RedTurtle : public Turtle {
  void drawshell() {
    // draw a red shell
  }
};

class GreenTurtle: public Turtle {
  void drawshell() {
    // draw green shell
  }
}
```

Here, subclasses can't handle the way a turtle is drawn, but can specialize drawings shell.

A public virtual method is really 2 things:

1. public:
  - It is an interface to the client
  - It indicates the behaviour we're providing to the clients
  - Upholds class variants and <mark>pre/post conditions</mark>
2. virtual method:
  - an interface to out subclass
  - A "hook" for our derived class to insert specialized behaviour

It is hard to separate these two ideas from each other if they're wrapped up in one method

- What if we later decide one of our virtual methods has grown too large and decide to split it into two.
  - We now have to change not only our class code, but also the client code.
- How could we make sure our overriding classes conform pre/post condition? What if they change later? Ugly code duplication

## The Non-Virtual Interface (NVI) idiom

NVI says:
- All public methods should be non-virtual
- All virtual methods should be private, except obviously the `dtor`

Example: Digital Media

```c++
class DigitalMedia {
public:
  virtual void play() = 0;
  virtual ~DigitalMedia();
};

// Using the NVI idiom

class DigitalMedia {
public:
  void doplay();
  virtual ~DigitalMedia() {}

private:
  virtual void Play() = 0;
};

void DigitalMedia::Play() {
  if (valid_subscription) {
    doplay();
  }
}
```

Now if we need exert extra control overplay, we can do it:

- We can add before/after code to run around `doplay()` -> also could be specialized
- can add more hooks later through more virtual method calls
- all of this without changing the public interface

It is much easier to keep control over our interface and virtual methods

So the `NVI` idiom extends the template method pattern saying all virtual methods should be wrapped in a non-virtual method

This is a huge benefit without any downside

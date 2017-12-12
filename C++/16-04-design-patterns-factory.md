# Factory Method Problem

Problem: Write a video game with two types of enemies, bullets and turtles. The system should randomly spawns Bullets and Turtles, but Bullets should become more frequent in later levels.

*Picture from November 9*

Since we never know what enemy is going to spawn next we can't call constructors directly. We also don't want to hard code the Policy.

We want it to be customizable and adaptable.

So we put a Factory method in Level that creates our enemies for us.

```c++
class Level {
public:
  virtual Enemy *createEnemy() = 0;
  // factory method
  ...
}

class NormalLevel : public Level {
public:
  Enemy *createEnemy() override {
    // create more turtles
  }
  ...
}

class Castle : public Level {
public:
  Enemy *createEnemy() override {
    // create more turtles
  }
  ...
}
```

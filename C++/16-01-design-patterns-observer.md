# Observer Pattern

Publish subscribe model

- One class: the publisher or subject generates data
- One or more class: called subscribers or observers - receive the data and react.

## Example

Publishers in the form of a spreadsheet cells and subscribers in the form of graphs.

*Picture from November 7*

The abstract class `subject` has all the common code to all subjects. The abstract class `observer` holds only the interface common to all observers.

Sequence to method calls:

1. Subjects state is updated
2. `Subject::notifyObservers()` calls each `Observers` notify method
3. Each observer calls `ConcreteSubject::getState` to get the data and reacts to it

## Example: Horse races

The subject publishes the winners, the observers are individually bettors. They will declare victory if their horse wins.

```c++
class Subject {
  Vector<Observer *> observer;

public:
  Subject();
  void attach(Observer *ob) { observers.emplace_back(ob); }
  void detach(Observer *ob) { ... } // remove from vector
  void notifyObservers() {
    for (auto &ob : observer) {
      ob->notify();
    }
  }
  ~Subject() = 0;
};

Subject::~Subject() {}

class Observer {
public:
  virtual void notify() = 0;
  virtual ~observer() {}
};

class HorseRace : public Subject {
  ifstream in; // source of data
  string LastWinner;

public:
  HorseRace(string source) : in{source} {}
  ~HorseRace() {}
  bool runRace() { return in >> LastWinner; }
  string getState() const { return LastWinner; }
};

class Bettor : public Observer {
  HorseRace *subject;
  string name, myHorse;

public:
  Bettor(...) {
    // we set the subject in MIL or in ctor body
    subject->attach(this);
  }
  
  void notify() {
    string winner = subject.getState();
    cout << (winner == myHorse ? "Win!" : "Lose") << endl;
  }
};
```

Simplifications and adaptations are possible in certain circumstances.

- If there is only one subject, you can merge `Subject` and `ConcreteSubject`. But do not do so lightly, as this reduces your ability to generalizes in the future.
- If the state is trivial, that is being notified is all the information we need, then we can leave out `getState`
- If `subject` and an `observer` can be the same thing (example cells in a spreadsheet) you can have it be one class

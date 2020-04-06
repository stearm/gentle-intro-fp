# A gently introduction to functional programming

## Goal of fp

Mastering complexity of a system using formal models

## Pillars

- referential transparency
- composition

## Referential transparency

An expression has the referential transparency property if it can be replaced with the corrispondent value, without alters program behaviour.
Therefore we can understand what code does without worrying about weird behaviours.

We are saying implicitly that in a functional world functions that return nothing cannot exists. This is totally opposed to what we do in OOP. Just think to setter methods, through we mutate object data over-time.

## Expression vs Statement

What is an expression? Everything that returns a value, we love expressions in FP. For example, in Scala everything is an expression, even the `if` statement!

**Referential transparency is all about expressions**

is this function referential transparent?

```scala
val area = (radius: Int) => Math.PI * math.pow(radius, 2)
```

and what about this?

```scala
var balance = 0
def updateBalance(amount: Int): Int = {
  balance += amount
  balance
}
updateBalance(150) == updateBalance(150)
```

In the last snippet we can face against unexpected results caused by an hidden global dependency. User cannot understand how computation end reading only function body.
Also think about what can happen if multiple threads call `updateBalance`.

## Pure functions

_A pure function is a function that given same input returns always the same output without produce any side-effect._

In other words, we would like to get closer to the mathematical definition of function, in which we are sure that everything works, thanks to formal models proven over the centuries.
What is a "side-effect"? Think about something that mutate external systems like IO operations, access to variables outside local scope or throwing exceptions.
But real life is full of side-effect, how can we face against this problem? Well, it is not possible to avoid side-effects, but we can model it, describe it through a model (to make "things" referential transparent) and stem them to the borders of our programs.
We will face this kind of problem in next sessions. 🤓

Then, we have to compromise and write pure functions where possibile mixing it together with referential transparency.

### Why pure functions are so important?

They give us guarantees: we know that they take some types of parameters and return some others stuff. We also know that, as we said, they don't touch external variables or fire side-effects. We are sure that composing pure functions we will obtain the expected results!
Others benefits are:

- parallelization
- memoization

### Partial functions

Functions that could not return values exists in math? Obviously not.
Lets consider this function:

```scala
val inverse: (Double) => (Double) = (x) => 1 / x
```

and try to run `inverse(0)`. What do you expect? Not so functional right?
Lets try to refactor it using a functional approach!

```scala
val inverse: (Double) => Option[Double] = {
  case 0 => None
  case x => Some(1 / x)
}
```

Using `Option[T]` we have a function with a total codomain, throwing away exceptions.

### From OOP to FP: an example

```java
public class Bird {
  private Double weight;

  public Bird(Double weight) {
    this.weight = weight;
  }
}

public class Cat {
  private Bird catch;
  private boolean sated;
  private boolean poo;
  private Double weight;

  public Cat(Double weight) {
    this.catch = null;
    this.sated = false;
    this.poo = false;
    this.weight = weight;
  }

  public void capture(Bird bird) {
    this.catch = bird;
  }

  public void eat() {
    this.sated = true;
    this.weight = this.weight + this.catch.weight;
    this.catch = null;
  }

  public void poo() {
    this.poo = true;
  }
}

Cat cat = new Cat(10);
Bird bird = new Bird(1);

cat.capture(bird);
cat.eat();
cat.poo();
```

```scala
object Chasing extends App {
  final case class Bird(weight: Double)
  final case class Cat(weight: Double)
  final case class CatWithBird(cat: Cat, capturedBird: Bird)
  final case class SatedCat(weight: Double)
  final case class HealthyCat(weight: Double)

  val capture: ((Cat, Bird)) => CatWithBird = (chase) => CatWithBird(chase._1, chase._2)
  val eat: (CatWithBird) => SatedCat = (catWithBird) => SatedCat(catWithBird.cat.weight + catWithBird.capturedBird.weight)
  val poo: (SatedCat) => HealthyCat = (satedCat) => HealthyCat(satedCat.weight - 1)

  // we can obtain the same result with poo(eat(capture((Cat(10), Bird(1)))))
  // the thing is, we want to highlight sequential operations through composition
  val foodChain: ((Cat, Bird)) => HealthyCat = capture andThen eat andThen poo
  val healthyCat: HealthyCat = foodChain((Cat(10), Bird(1)))

  // omg side effect
  println(healthyCat.weight)
}
```

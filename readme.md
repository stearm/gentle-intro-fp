# A gentle introduction to functional programming

Welcome to this light introduction to functional programming. Here we will try to explain in few words why functional
programming can be applied successfully to reduce the complexity of algorithms and ensure the correctness of the code.

## Goals of fp

The primary goal of functional programming is expressing complex flows with simple expressions, or, functions.
FP heavily borrows from mathematics, as its primary unit is a "function", that can be defined as a building brick
that given an input produces an output.

## Pillars

The founding principle on which FP is based are:

1. The ability to replace any block of code with the calculated value without altering the program's behaviour
   (a.k.a.) referential transparency. This implicitly means that a function that returns nothing cannot exist.
2. The ability to compose functions one with another to obtain a completely new function that does a composite
   behaviour
3. Using no side effects: every function should be "pure", in a sense that there are no side effects or mutable
   states. For example there are no setter functions (which do not return values, so they are not purely functional),
   there are no functions that make IO, and most importantly, there are NO EXCEPTIONS. A function will always
   return a value, whether the computation goes well, or not.

Any side effect, such as saving to a DB or file system will be postponed until the very end of the program, in order
to leave the functions pure.

## Anything is a function

Or, better, an expression. The definition of expression is "anything that returns a value".
In purely functional languages everything is an expression, so there is no need for a `return` statement (even the
`if` statement is an expression)
In non purely functional languages, such as Java, we can (since java 8) simulate functional programming using lambdas
and method references, though it's a bit more difficult and verbose.

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

In the last snippet we can face unexpected results caused by a hidden global dependency.
An external programmer cannot understand how the computation will result by only reading the function body.
Also think about what can happen if multiple threads call `updateBalance`.

## Pure functions

_A pure function is a function that, given the same input, always returns the same output without producing any side-effect._

--- TODO

In other words, we would like to get closer to the mathematical definition of function, in which we are sure that everything works, thanks to formal models proven over the centuries.
What is a "side-effect"? Think about something that mutate external systems like IO operations, access to variables **outside local scope** or throwing exceptions. It is legal to have local mutations.
But real life is full of side-effect, how can we face against this problem? Well, it is not possible to avoid side-effects, but we can model it, describe it through a model (to make "things" referential transparent) and stem them to the borders of our programs.
We will face this kind of problem in next sessions. 🤓

Then, we have to compromise and write pure functions where possibile mixing it together with referential transparency.

### Why pure functions are so important?

They give us guarantees: we know that they take some types of parameters and return _always_ a value. We also know that, as we said, they don't touch external variables or fire side-effects. We are sure that composing pure functions we will obtain the expected results!
Others benefits are:

- parallelization
- memoization

When we can compose two functions `f` and `g` (`f ∘ g == f(g(x))`)? The domain of `f` must be a subset of `g` domain.

### Partial functions

We want to live a world where every function is a total function.
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

And what about reason of failure? You can use `Either[L, R]`!

// TODO

### From OOP to FP: an example

OOP with Java

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
    this.sated = false;
  }
}

Cat cat = new Cat(10);
Bird bird = new Bird(1);

cat.capture(bird);
cat.eat();
cat.poo();
```

FP with Scala

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

**References:**

- https://github.com/gcanti/functional-programming
- https://www.youtube.com/watch?v=tKfVI2hGtGQ
- https://medium.com/@olxc/referential-transparency-93352c2dd713
- https://www.sitepoint.com/functional-programming-pure-functions/

**Misc:**

- https://stackoverflow.com/questions/4865616/purity-vs-referential-transparency
- https://www.reddit.com/r/haskell/comments/21y560/-

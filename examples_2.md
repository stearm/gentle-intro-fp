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

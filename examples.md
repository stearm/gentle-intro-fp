# FP in practice
It's time to put all the words into something more practical.
We have chosen for this demonstration to use Java, since it's the most common language between us.

If there is interest, we might think about starting a series of presentations about Scala.

## The basics
Since Java 8, we are able to use java in a more functional oriented way, thanks to some new
interfaces.

The interface we are introducing now, and that is the most important for this demonstration,
is called (no surprise) `Function<T, R>` and is part of the `java.util.function` package.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

`Function` is a "Functional interface", the concept of functional interface was introduced in
Java 8 and it basically means that it is an interface that exposes only one method that must
be implemented in order to implement that interface. In this case the method is called
`apply`, which takes as input a `T` and returns as output `R`.

The fact that there's only one function to implement means that we can create an instance
of a function by just writing:

```
Function<Integer, Integer> fun = x -> x + 2
```

instead of the more verbose

```
Function<Integer, Integer> fun = Function<> {
    Integer apply(Integer x) {
        return x + 2;
    }
}
```

So, now we have our "building block" for the functional world.

## Properties of a function

As we have said, we can use composition to create new functions. This can of course be applied
to our `Function` interface, because it exposes two default methods that allow us to do this:
`compose` and `andThen`.

* `compose` takes as input another function and the result can be expressed mathematically
  like this: `f.compose(g) = f(g(x))`. Meaning that g is applied first, and then f.
* `andThen` is the opposite and works like this: `f.andThen(g) = g(f(x))`: first f is applied,
  then g.
  
There is also an utility definition called `identity` that returns the identity function:
`static <T> Function<T, T> identity() { return t -> t; }`

## Example 1: Anything can be a function
If you try hard enough.

Now that we have grasped the basics of our function, let's see some use cases to apply it.

Let's say we want to make a salary calculator that must take into account various modifiers
such as bonuses and taxes. We could implement it like this:

```java
class SalaryCalculator {
    private double calculateBonus(double d) { return d * 1.2; }
    private double calculateTax(double d) { return d * 0.7; }
    
    public double calculateSalary(double base, boolean bonus, boolean tax) {
        double salary = base;
        if (bonus) {
            salary = calculateBonus(salary);
        }
        if (tax) {
            salary = calculateTax(salary);
        }
        return salary;
    }
}
```

As it's easy to notice, there are many reasons we don't want to implement it like this:
1. It is really verbose: each "if" increases the complexity of the class.
2. To use the function, I need to remember the order of the boolean parameters.
3. If I have to add another modifier, I need to
    1. Add another function with the operation
    2. Add another boolean to the main function
    3. Add another `if` in the main function's body (the code grows longer)
    4. Modify all the places where that function is used to add the new boolean, even
       if in that particular place I don't want to use the new modifier

So, in definitive, it's not easily maintainable.

How can we improve this? Let's try to refactor it.

```java
class SalaryCalculator {
    private boolean bonus;
    private boolean taxes;

    public SalaryCalculator withBonus(boolean bonus) {
        this.bonus = bonus;
        return this;
    }

    public SalaryCalculator withTaxes(boolean taxes) {
        this.taxes = taxes;
        return this;
    }

    private double calculateBonus(double d) { return d * 1.2; }
    private double calculateTaxes(double d) { return d * 0.7; }

    public double calculateSalary(double base) {
        double salary = base;
        if (bonus) {
            salary = calculateBonus(salary);
        }
        if (taxes) {
            salary = calculateTaxes(salary);
        }
        return salary;
    }
    
}
```

This is a bit better, though it's still long and the codebase will grow, we solved
problems number 2, number 3.ii and 3.iv as we don't need to modify the places where
we don't intend to use a new modifier.

But we can do better than this, so let's give it a try with our new friend `Function`!

```java
class SalaryCalculator {

    private final List<Function<Double, Double>> rules = new ArrayList<>();

    public SalaryCalculator with(Function<Double, Double> rule) {
        rules.add(rule);
        return this;
    }

    public double calculateSalary(double base) {
        double salary = base;
        for (Function<Double, Double> rule: rules) {
            salary = rule.apply(salary);
        }
        return salary;
    }
}
```

Here we have solved more problems: the code is not verbose at all now, we will never
have to touch the calculateSalary function again even if we need to add another rule: when
we unit test this once, we will never have to do it again.

We will use this code like this:

```java
class App {
    public void main(String[] args) {
        Function<Double, Double> bonusRule = x -> x * 1.2;
        Function<Double, Double> taxesRule = x -> x * 0.7;
        
        new SalaryCalculator()
            .with(bonusRule)
            .with(taxesRule)
            .calculateSalary(2000.0);
    }
}
```

But there's one last thing we can do: in this example we are still thinking with the old
imperative mindset. Let's work on that and use a purely functional style.

As we've seen before, functions can be combined, and in this case all we have to do is
combine all the functions and then apply the result to the base salary, a bit like this:
`taxesRule(bonusRule(salaryBase))`.

So let's rewrite our calculate salary function in light of this:


```java
class SalaryCalculator {

    // omissis

    public double calculateSalary(double baseSalary) {
        Function<Double, Double> result = salary -> salary;
        for (Function<Double, Double> rule: rules) {
            result = result.andThen(rule);
        }
        return result.apply(baseSalary);
    }
}
````

Also, as we said before, `salary -> salary` can be replaced with `Function.identity()

```java
class SalaryCalculator {

    // omissis

    public double calculateSalary(double baseSalary) {
        Function<Double, Double> result = Function.identity();
        for (Function<Double, Double> rule: rules) {
            result = result.andThen(rule);
        }
        return result.apply(baseSalary);
    }
}
````

Scrumptious. That was easy, wasn't it? But we can do even better!

How? We will remove the mutable state with the Streaming API, another great functional facility
introduced by Java8.

On any `Collection` object, we can call the `stream()` function that allows us to define
a series of operations to execute on that collection. These operations are chained until a terminal
operation is specified; this means that if you do not specify a terminal operation, the stream
will be in a pending state, and all code inside it will not be executed.

An example would be taking all elements into a list, transforming them and then collecting
them again into another list:

```
aList.stream()
    .map(x -> x + 1)
    .collect(Collectors.toList());
```

Back to our case, what we need is the function called `reduce`: this function will take all
elements of a stream and perform a "reduction" operation, meaning it will take each
element and combine (or accumulate) it with the result of the former iteration.
The "zero-th" iteration result must be provided, along with the accumulator which will combine
the iteration results.

The definition of the reduce is as follows (pseudo code):

```
T result = identity;
for (T element: thisStream)
    result = accumulator.apply(result, element);
return result;
``` 

does it ring a bell? Of course it does, it's the same code we wrote before. Here are our
substitutions:

* `T` = `Function<Double, Double>`
* `identity` = `Function.identity()`
* `accumulator` is a bit more tricky: as you can see by the apply method, it's a function,
  but this one takes **two** parameters and returns a result,
  in our case `(result, rule) -> result.andThen(rule)`. This is also called `BiFunction<T, U, R>`
  
Time to wrap up then!

```java
class SalaryCalculator {
   
    // omissis

    public double calculateSalary(double baseSalary) {
        return rules.stream()
            .reduce(Function.identity(), (result, rule) -> result.andThen(rule))
            .apply(baseSalary);
    }
}
```

There we go. A single return with one chained stream call. This is a purely functional
approach.
Also, notice one thing: `(result, rule) -> result.andThen(rule)` can be further simplified
because of the two parameters that we have as input, we use both of them and in that precise
order, so we can substitute this with a **method reference**: `Function::andThen`
resulting in our full class:

```java
class SalaryCalculator {
   
    private final List<Function<Double, Double>> rules = new ArrayList<>();
    
    public SalaryCalculator with(Function<Double, Double> rule) {
        rules.add(rule);
        return this;
    }

    public double calculateSalary(double baseSalary) {
        return rules.stream()
            .reduce(Function.identity(), Function::andThen)
            .apply(baseSalary);
    }
}
```

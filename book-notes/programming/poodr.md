# Practical Object Oriented Design in Ruby

---

# In Praise Of Design
#### Chapter 1

---

# Designing Classes with a Single Responsibility
#### Chapter 2

---

- **Transparent** The consequences of change should be obvious in the code that is changing and in distant code that relies upon it.
- **Reasonable** The cost of any change should be proportional to the benefits the change achieves.
- **Usable** Existing code should be usable in new and unexpected contexts
- **Exemplary** The code itself should encourage those who change it to perpetuate these qualities.

---

Code that is Transparent, Reasonable, Usable, and Exemplary (TRUE) not only meets today's needs but can also be changed to meet future needs.

---

# Single Responsibility Principle

Everthing a class does should be highly related to its purpose.

---

# When to Make Design Decisions

What is the future cost of doing nothing today?

When the future cost of doing nothing is the same as the current cost, postpone the decision. Make the decision only when you must with the information you have at that time

---

# Writing Changeable Code

* Depend on behavior, not data

---

# Hide Instance Variables

Don't expose access to data unless you really need to (private methods)

---

# Hide Data Structures

Don't depend on complicated data structures.

Examples:

* Using arrays as constructor arguments
* Each sender (method) of the data must know what piece of data is at what index in the array

---

# Hide Data Structures
## Bad

```php
// Not easy to change, not "TRUE"
class Greeter
{
    protected $data;

    public function __construct(array $data)
    {
        $this->data = $data;
    }

    public function greet()
    {
        return "Hello {$this->data[0]}, you are {$this->data[1]} years old!"
    }

    // more methods that use other array indexes to get data
}
```

---

# Hide Data Structures

The previous example **depends** upon the arrays structure.

---

# Hide Data Structures

In the next example, we would prefer passing a Student object directly, but in the real world you don't always have control of your input. For we now we settle for limiting the knowledge of the array indexes to only one method in the class by converting every item in the array into a simple object.

---

```php
class Greeter
{
    protected $person;

    public function __construct(array $data)
    {
        $this->person = $this->personify($data);
    }

    public function greet()
    {
        $person = $this->person;
        return "Hello {$person->name}, you are {$person->age} years old!"
    }

    protected function personify(array $data)
    {
        $person = new \stdClass;
        $person->name = $data[0];
        $person->age = $data[1];
        // ... more attributes
        return $person;
    }

    // other methods that depend on person
}
```

---

Yes the example is contrived and it might make more sense to pass person to greet at runtime, but in the interest of brevity, I think the above example clearly illustrates how hiding complex data structures can make code much easier to understand and to change.

---

# Enforce Single Responsibility Everywhere

* Creating classes with a single responsibility is important for design but you shouldn't stop there

---

## Extract Extra Responsibilities from Methods

This method has two responsibilities. It iterates over the wheels AND calculates the diameter of each wheel.

```php
// return an array of wheel diameters
public function diameters()
{
    return $this->wheels->map(function($wheel) {
        return $wheel->rim + ($wheel->tire * 2);
    });
}
```

---

## Extract Extra Responsibilities from Methods

```php
// iterate over the wheels
public function diameters()
{
    return $this->wheels->map(function($wheel) {
        return $this->diameter($wheel);
    })
}

// calculate diameter of ONE wheel
protected function diameter($wheel)
{
    return $wheel->rim + ($wheel->tire * 2);
}
```

---

## Extract Extra Responsibilities from Methods

The impact of a small refactoring like in the previous example is small but the cumulative effect of this coding style is huge. Methods that have a single responsibility confer the following benefits.

---

## Extract Extra Responsibilities from Methods

- **Expose previously hidden qualities** Refactoring a class so that all of its methods have a single responsibility has a clarifying effect on the class. You can now more easily extract a class from this class, it doesn't have to be right now.

---

## Extract Extra Responsibilities from Methods

- **Avoid the need for comments** Comments are usually just decaying documentation. Extract the comment into a named method.

---

## Extract Extra Responsibilities from Methods

- **Encourage Reuse** Small methods encourage coding behavior that is healthy for your application. Other programmers will reuse the methods instead of duplicating the code.

---

# Isolate Extra Responsibilities in Classes

Let's say you have a `Gear` class that has some `Wheel` like behavior. Does the application need a `Wheel` class?

---

# Isolate Extra Responsibilities in Classes

There are reasons why you might not be able to create a wheel class right now:

* Some design restriction has been imposed, you can't create wheel right now
* You don't really know where you are going yet and you aren't ready to introduce a new class

---

# Isolate Extra Responsibilities in Classes

Because you are writing changeable code, you are best served by postponing decisions until you are absolutely forced to make them. Any decision you make in advance of an explicit requirement is just a guess. Don’t decide; preserve your ability to make a decision later.

---

```php
class Gear
{
    protected $chainring;
    protected $cog;
    protected $wheel;

    public function __construct($chainring, $cog, $rim, $tire)
    {
        $this->chainring = $chainring;
        $this->cog = $cog;
        $this->wheel = $this->wheelInstance($rim, $tire);
    }

    public function ratio()
    {
        return $this->chainring / (float) $this->cog;
    }

    public function inches()
    {
        return $this->ratio() * $this->wheel->diameter();
    }

    protected function wheelInstance($rim, $tire)
    {
        return new class($rim, $tire) {
            protected $rim;
            protected $tire;
            function __construct($rim, $tire) {
                $this->rim = $rim;
                $this->tire = $tire;
            }
            function diameter() {
                return $this->rim + ($this->tire * 2);
            }
        };
    }
}
```

---

# Isolate Extra Responsibilities in Classes

Now you have a `Wheel` that can calculate its own diameter. Embedding this `Wheel` in `Gear` is obviously not the long-term design goal; it’s more an experiment in code organization. It cleans up `Gear` but defers the decision about `Wheel`.

---

# Summary

The path to changeable and maintainable object-oriented software begins with classes that have a single responsibility. Classes that do one thing isolate that thing from the rest of your application. This isolation allows change without consequence and reuse without duplication.

---

# Managing Dependencies
#### Chapter 3

---

* Because well designed objects have a single responsibility, their very nature requires that they collaborate to accomplish complex tasks.
* This collaboration is powerful and perilous. To collaborate, an object must know something know about others.
* Knowing creates a dependency. If not managed carefully, these dependencies will strangle your application.

---

# Recognizing Dependencies

* The name of another class. Gear expects a class named Wheel to exist
* The name of a message that it intends to send to someone other than self.
* The arguments that a message requires.
* The order of those arguments.

---

* Unnecessary dependencies make code less reasonable. Because they increase the chance that other classes will be forced to change, these dependencies turn minor code tweaks into major undertakings where small changes cascade through the application, forcing many changes.
* Your design challenge is to manage dependencies so that each class has the fewest possible; a class should know just enough to do its job and not one thing more.

---

# Other Dependencies
## Law of Demeter
* knowing the name of a message you plan to send to someone other than self
* Don't go two deep...
* Bad: `$a->b->foo()`
* Ok: `$a->foo()`

---

# Other Dependencies
## Tightly Couple Test Code

* Tests break every time the code is refactored
* Tests begin to seem costly relative to their value

---

# Writing Loosely Coupled Code
## Inject Dependencies

`gearInches` method contains an explicit reference to class Wheel

```php
class Gear
{
    public function gearInches()
    {
        $this->ratio * (new Wheel($this->rim, $this->tire))->diameter();
    }
}
```

---

# Writing Loosely Coupled Code
## Inject Dependencies

Refactor using DI. Gear no longer cares about class wheel. It just needs something that responds to the method `diameter`.

```php
class Gear
{
    protected $chaingring;
    protected $cog;
    protected $wheel;

    public function construct($chainring, $cog, $wheel)
    {
        $this->chaingRing = $chainring;
        $this->cog = $cog;
        $this->wheel = $wheel;
    }

    public function gearInches()
    {
        return $this->ratio() * $this->wheel->diameter();
    }

    # ... code
}
```

---

# Writing Loosely Coupled Code

* In the real world you often times can't make the changes you would like to make. In these situations, just do the best you can given the constraints.
* There are many things you can do to make a codebase more flexible.
* Expose dependencies as mucha as possible.

---

# Writing Loosely Coupled Code
## Working under constraints

There are strategies you could use when you can't just inject a wheel class into Gear. These strategies aim to REVEAL dependencies since we can't just inject or remove them right now. This lowers the barrier to re-use later.

---

* Strategy 1: Move the creation of `Wheel` into `Gear` construct. This exposes the dependency in the constructor which is better than hiding it down inside a class method.
* Strategy 2: Isolate the creation of `Wheel` inside of a private method.

---

# Remove Argument-Order Dependencies

* You can use named constructors or accept associative arrays (hashes) as constructor arguments to avoid argument-order dependencies.

---

# Accept associative array as constructor param

```php
class Wheel
{
    protected $gear;
    protected $bar;

    public function _construct(array $args)
    {
        $this->gear = $args['gear'];
        $this->foo = $args['bar'];
    }
}
```

---

# Named constructor

```php
class Wheel
{
    protected $gear;
    protected $bar;

    public function __construct($gear, $bar)
    {
        $this->gear = $gear;
        $this->bar = $bar;
    }

    public static function create(array $args)
    {
        return new static($args['gear'], $args['bar']);
    }
}
```

---

# Isolate Multiparameter Initialization

* Imagine that Gear is part of a framework and that its initialization method requires fixed-order arguments. Imagine also that your code has many places where you must create a new instance of Gear. Gear’s initialize method is external to your application; it is part of an external interface over which you have no control.

---

# Isolate Multiparameter Initialization

* As dire as this situation appears, you are not doomed to accept the dependencies. Just as you would DRY out repetitive code inside of a class, DRY out the creation of new Gear instances by creating a single method to wrap the external interface

---

```php
class GearFactory
{
    public static function create(array $args)
    {
        return new Gear($args['foo'], $args['bar']);
    }
}
```

---

# Managing Dependency Direction

Classes should depend on things that change less often than themselves.

---

* Some classes are more likely than others to have changes in requirements.
* Concrete classes are more likely to change than abstract classes.
* Changing a class that has many dependents will result in widespread consequences.

---

This chapter really didn't seem to talk much about dependency direction.

---

# Creating flexible interfaces
#### Chapter 4


---
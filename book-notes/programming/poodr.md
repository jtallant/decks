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
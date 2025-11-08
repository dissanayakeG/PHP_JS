# C# OOP

# Classes and Objects

In C#, an object (or instance) is just a concrete “thing” created from a class (the blueprint). You use the `new` keyword to allocate memory and initialize it.

## Objects can be created in below ways

**1. Basic Object Creation**

```c#
public class Person
{
    public string Name;
    public int Age;
}

// Create object
Person p1 = new Person();
p1.Name = "Madz";
p1.Age = 28;

Console.WriteLine($"{p1.Name}, {p1.Age}");
```

**Note** `p1` is an instance of `Person`.

**2. Object Initializer Syntax**

Cleaner way to set properties when creating objects:

```c#
Person p2 = new Person { Name = "Alice", Age = 25 };
```

**3. Using a Constructor**

You can define constructors to force initialization when creating objects:

```c#
public class Person
{
    public string Name { get; }
    public int Age { get; }

    // Constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
// Create object with constructor
Person p3 = new Person("Bob", 30);
```

**4. Anonymous Objects (Quick Ad-hoc)**

For temporary use, you can create anonymous objects (no explicit class):

```c#
var anon = new { Name = "Charlie", Age = 35 };
Console.WriteLine(anon.Name);
```

**Note** Common in LINQ queries when shaping data.

**5. `new()` Target-Typed Expressions (C# 9+)**

If the type is obvious from context, you can use shorthand:

```c#
Person p4 = new("Dana", 22); // uses constructor
```

**6. Static Classes (No Instances)**

Static classes cannot be instantiated:

```c#
public static class MathUtils
{
    public static int Add(int a, int b) => a + b;
}

// MathUtils utils = new MathUtils(); ❌ not allowed
int sum = MathUtils.Add(2, 3); // use directly
```

**7. With `Activator.CreateInstance`**

For reflection-heavy scenarios:

```c#
Type t = typeof(Person);
object obj = Activator.CreateInstance(t, "Eve", 29);
```

**Note** Useful in frameworks like ASP.NET Core and EF when objects are created dynamically.

# Interfaces

## What is an Interface in C#?

- An interface is a contract — it defines _what_ a class can do, but not _how_ it does it.
- Think of it as a checklist:
- If a class says, “I implement this interface,” then it must provide all the members listed in that interface.

## Declaring an Interface

```c#
public interface IVehicle
{
    void Drive();
    void Refuel(int liters);
}
```

This doesn’t implement logic — it just declares what methods exist.

## Implementing an Interface

```c#
public class Car : IVehicle
{
    public void Drive()
    {
        Console.WriteLine("The car is driving...");
    }

    public void Refuel(int liters)
    {
        Console.WriteLine($"The car is refueled with {liters} liters.");
    }
}
```

If you miss a method, the compiler screams. That’s the contract.

## Using Interfaces

```c#
IVehicle myCar = new Car();
myCar.Drive();
myCar.Refuel(50);
```

**Notice:** you reference it by the interface, not the concrete type → this is polymorphism.  
Later you can swap `Car` for `Bike`, `Truck`, or anything else that implements `IVehicle` without changing calling code.

## Why Interfaces Matter

- Abstraction → focus on _what_ things do, not _how_.
- Loose coupling → your code depends on contracts, not concrete classes.
- Dependency Injection (DI) → almost all modern C# apps rely on interfaces to inject services.
- Testability → you can mock interfaces easily for unit tests.

## Advanced Tricks

### Multiple Interfaces

```c#
public interface IFlyable { void Fly(); }
public interface ISwimmable { void Swim(); }

public class Duck : IFlyable, ISwimmable
{
    public void Fly() => Console.WriteLine("Duck is flying!");
    public void Swim() => Console.WriteLine("Duck is swimming!");
}
```

### Default Interface Methods (C# 8+)

Interfaces can now have default method bodies (like Java).

```c#
public interface ILogger
{
    void Log(string message);

    // Default method
    void LogError(string message) => Log("ERROR: " + message);
}
```

### Explicit Interface Implementation

Sometimes two interfaces have methods with the same name.

```c#
public interface IPrinter { void Print(); }
public interface IScanner { void Print(); }

public class MultiFunctionDevice : IPrinter, IScanner
{
    void IPrinter.Print() => Console.WriteLine("Printing...");
    void IScanner.Print() => Console.WriteLine("Scanning...");
}

# Now you must call via the interface type:

IPrinter p = new MultiFunctionDevice();
p.Print(); // Printing...

IScanner s = new MultiFunctionDevice();
s.Print(); // Scanning...
```

## What Are Default Interface Methods?

Traditionally, interfaces in C# could only declare members — never implement them.
But in C# 8, Microsoft introduced default interface methods (DIMs):  
**Note** You can now provide a _default implementation_ for a method inside the interface itself.
That means classes implementing the interface don’t have to implement every method anymore — they can just use the default if they want.

### Example

```c#
public interface ILogger
{
    void Log(string message);
    // New: default method
    void LogError(string message)
    {
        Log($"ERROR: {message}");
    }
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
    // No need to implement LogError — default kicks in
}

##Usage:
ILogger logger = new ConsoleLogger();
logger.Log("Normal log");
logger.LogError("Something went wrong"); // Uses default method
```

## Why Did Microsoft Add This?

- Backward compatibility → Imagine you have 100s of classes implementing `ILogger`.  
     If you add a new method, all those classes break.  
     With default implementations, you can evolve interfaces without breaking everything.

- Easier API evolution → Libraries can add new functionality while keeping old code running.

## Override if You Want

Implementing classes can still override the default if they need custom logic:

```c#
public class FileLogger : ILogger
{
    public void Log(string message) => File.AppendAllText("log.txt", message + "n");
    // Override default method
    public void LogError(string message)
    {
        File.AppendAllText("log.txt", $"[CRITICAL] {message}n");
    }
}
```

## Gotchas

1. Don’t abuse it  
   Interfaces aren’t meant to become full-blown abstract classes. Keep defaults minimal.
2. Diamond problem  
   If a class implements two interfaces that both have the same default method, you must resolve the conflict explicitly.

```c#
public interface IA { void Speak() => Console.WriteLine("A"); }
public interface IB { void Speak() => Console.WriteLine("B"); }
public class C : IA, IB
{
    // Must disambiguate
    void IA.Speak() => Console.WriteLine("A impl in C");
    void IB.Speak() => Console.WriteLine("B impl in C");
}
```

# Encapsulation

## What is Encapsulation?

Encapsulation = bundling data + behavior into a single unit (class), while restricting direct access to some of that data.  
**Note** It’s about controlling how your data is accessed and modified.

### How C# Does Encapsulation

C# gives you access modifiers to control visibility:

- `public` → accessible everywhere.
- `private` → accessible only inside the class.
- `protected` → accessible in the class and derived classes.
- `internal` → accessible only inside the same assembly.
- `protected internal` → accessible in derived classes OR same assembly.
- `private protected` → accessible in derived classes but only in the same assembly.

## Example Without Encapsulation (Bad Design)

```c#
public class BankAccount
{
 public double Balance; // exposed to the world
}

#Usage:

var account = new BankAccount();
account.Balance = -1000; // 👎 Invalid state, no restrictions
```

## Example With Encapsulation

```c#
public class BankAccount
{
 private double balance; // hide the field

    public double Balance
    {
        get => balance; // controlled access
        private set
        {
            if (value >= 0)
                balance = value;
        }
    }

    public void Deposit(double amount)
    {
        if (amount > 0)
            Balance += amount;
    }

    public void Withdraw(double amount)
    {
        if (amount > 0 && amount <= Balance)
            Balance -= amount;
    }

}

#Usage:

var account = new BankAccount();
account.Deposit(500);
account.Withdraw(200);
Console.WriteLine(account.Balance); // 300
// account.Balance = -9999; // ❌ Not allowed
```

**Note** `balance` is protected inside the class, and only controlled methods decide how it changes.

## Benefits of Encapsulation

1. Data protection → prevent invalid states (e.g., negative balance).
2. Flexibility → you can change implementation later without breaking consumers.
3. Controlled access → expose only what’s necessary.
4. Maintenance → one place to add validation / logging.

# Polymorphism

## What is Polymorphism?

The word literally means “many forms.”  
 In C#, polymorphism lets you treat different objects the same way, as long as they share a common base type (class or interface).

That way, the _same code_ can work with different types.

## Two Main Types in C#

### 1. Compile-Time Polymorphism (Method Overloading)

Happens when you define multiple methods with the same name but different signatures (different params).

```c#
public class MathUtils
{
 public int Add(int a, int b) => a + b;
 public double Add(double a, double b) => a + b;
 public int Add(int a, int b, int c) => a + b + c;
}

#Usage:

var math = new MathUtils();
Console.WriteLine(math.Add(2, 3)); // Calls int version
Console.WriteLine(math.Add(2.5, 3.5)); // Calls double version
Console.WriteLine(math.Add(1, 2, 3)); // Calls 3-param version
```

Decided at compile time which overload to call.

### 2. Runtime Polymorphism (Method Overriding)

This is the _classic OOP polymorphism_.  
You declare a base class with a `virtual` method and then override it in derived classes.

```c#
public class Animal
{
 public virtual void Speak()
 {
    Console.WriteLine("Some generic animal sound");
 }
}

public class Dog : Animal
{
 public override void Speak()
 {
    Console.WriteLine("Woof!");
 }
}

public class Cat : Animal
{
 public override void Speak()
 {
    Console.WriteLine("Meow!");
 }
}

#Usage:
Animal a1 = new Dog();
Animal a2 = new Cat();
a1.Speak(); // Woof!
a2.Speak(); // Meow!
```

**Note** Even though both variables are typed as `Animal`, the runtime type decides which method executes.

## Interfaces & Polymorphism

Polymorphism also shines with interfaces.

```c#
public interface IVehicle
{
 void Drive();
}

public class Car : IVehicle
{
 public void Drive() => Console.WriteLine("Car is driving");
}

public class Bike : IVehicle
{
 public void Drive() => Console.WriteLine("Bike is riding");
}

#Usage:

List<IVehicle> vehicles = new() { new Car(), new Bike() };
foreach (var v in vehicles){
 v.Drive();
}

#Output:
//Car is driving
//Bike is riding
```

Same method call (`Drive()`), different behavior depending on the object.

## Summary

- Compile-time polymorphism → Method overloading.
- Runtime polymorphism → Method overriding (`virtual`, `override`).
- Interface polymorphism → Different classes sharing the same contract.

**Note** Polymorphism = _“one interface, many implementations.”_

## virtual/override and new keywords

### `virtual` / `override` keys

This is true runtime polymorphism.

- `virtual` → says _“this method can be overridden by subclasses.”_
- `override` → says _“I’m overriding a virtual method from the base class.”_

### Example:

```c#
public class Animal
{
    public virtual void Speak(){Console.WriteLine("Some generic animal sound");}
}

public class Dog : Animal
{
    public override void Speak(){Console.WriteLine("Woof!");}
}

##Usage:
Animal a = new Dog();
a.Speak(); // Woof! (runtime polymorphism)
```

**Note** Even though `a` is typed as `Animal`, the runtime picks Dog’s override.

### `new` (Method Hiding) key

This is not polymorphism — it’s method hiding.

- `new` → says _“I’m not overriding the base method; I’m hiding it with a new one.”_
- Calls depend on the compile-time type, not runtime type.

### Example:

```c#
public class Animal
{
    public void Speak(){Console.WriteLine("Some generic animal sound");}
}

public class Dog : Animal
{
    public new void Speak(){Console.WriteLine("Woof!");}
}

##Usage:
Dog d = new Dog();
d.Speak(); // Woof!
Animal a = new Dog();
a.Speak(); // Some generic animal sound ❌ (base version, not Dog)
```

**Note** The base method gets called if the variable is typed as `Animal`, because `new` does not override.

## Side-by-Side Comparison

| Feature                  | `virtual/override`                 | `new` (hiding)                                                                      |
| ------------------------ | ---------------------------------- | ----------------------------------------------------------------------------------- |
| Runtime polymorphism?    | Yes                                | ❌ No                                                                               |
| Chosen by                | Runtime type                       | Compile-time type                                                                   |
| Base method replaceable? | Yes                                | No (just hidden)                                                                    |
| Use case                 | Extensible APIs, true polymorphism | Rare — only when you _intentionally_ want different behavior based on variable type |

## When to Use What

- Use `virtual/override` → when designing extensible, polymorphic behavior (the normal case).
- Use `new` → when you want to explicitly hide a base method (very rare, usually signals poor design).
    Most of the time, if you find yourself typing `new` in method declarations, you should rethink your class design.

Quick rule of thumb:

- `override` = polymorphism (runtime decides which method runs).
- `new` = hiding (compiler decides which method runs, based on the variable type).

# Inheritance

## What is Inheritance?

Inheritance = creating a new class from an existing one, reusing and extending its behavior.  
**Note** A derived class (child) automatically gets the fields, properties, and methods of the base class (parent).

## Basic Example

```c#
public class Animal
{
    public string Name { get; set; }
    public void Eat()
    {
        Console.WriteLine($"{Name} is eating");
    }
}

public class Dog : Animal
{
    public void Bark(){ Console.WriteLine($"{Name} is barking: Woof!");}
}

#Usage:

var dog = new Dog { Name = "Rex" };
dog.Eat(); // inherited from Animal
dog.Bark(); // Dog-specific
```

`Dog` didn’t need to re-implement `Eat()` — it inherits it.

## Types of Inheritance in C#

### 1. Single Inheritance (C# only allows this for classes — one base class per child).

```c#
public class A {}
public class B : A {}
```

### 2. Multilevel Inheritance

```c#
public class Animal { }
public class Mammal : Animal { }
public class Dog : Mammal { }
```

### 3. Hierarchical Inheritance

```c#
public class Animal { }
public class Dog : Animal { }
public class Cat : Animal { }
```

### 4. Multiple Inheritance (via Interfaces only)

```c#
public interface IFly { void Fly(); }
public interface ISwim { void Swim(); }

public class Duck : IFly, ISwim
{
    public void Fly() => Console.WriteLine("Duck flying");
    public void Swim() => Console.WriteLine("Duck swimming");
}
```

## Virtual & Override in Inheritance

This is where polymorphism plugs in.

```c#
public class Animal
{
    public virtual void Speak(){ Console.WriteLine("Some generic sound");}
}

public class Dog : Animal
{
    public override void Speak(){Console.WriteLine("Woof!"); }
}

public class Cat : Animal
{
    public override void Speak(){Console.WriteLine("Meow!");}
}

#Usage:

Animal a1 = new Dog();
Animal a2 = new Cat();

a1.Speak(); // Woof!
a2.Speak(); // Meow!
```

**Note** Parent defines general behavior, child overrides with specific behavior.

## Access Modifiers in Inheritance

- `public` → accessible everywhere.
- `protected` → accessible in base + derived classes.
- `private` → only inside the same class (not inherited).

Example:

```c#
public class Base
{
    protected int number = 10;
}

public class Derived : Base
{
    public void ShowNumber(){Console.WriteLine(number); // allowed}
}
```

## Abstract Classes in Inheritance

Abstract classes enforce inheritance contracts.

```c#
public abstract class Shape
{
    public abstract double Area(); // must be overridden
}

public class Circle : Shape
{
    private double r;
    public Circle(double radius) => r = radius;
    public override double Area() => Math.PI * r * r;
}
```

## Quick Summary

- Inheritance = reuse + extend existing code.
- C# supports single inheritance for classes, multiple via interfaces.
- Used with `virtual`/`override` → enables polymorphism.
- Abstract classes enforce a contract while still sharing implementation.

# Abstraction

## What is Abstraction?

Abstraction = hiding the “how” and showing only the “what.”  
You expose _essential behavior_ without forcing consumers to care about implementation details.

In C#, you achieve abstraction mainly with:

1. Abstract classes
2. Interfaces

## Abstract Classes

- Can’t be instantiated directly.
- Can contain abstract members (no implementation) and regular members (with implementation).
- Derived classes must implement abstract members.

### Example

```c#
public abstract class Shape
{
    // Abstract method (must be implemented in subclasses)
    public abstract double Area();
    // Regular method (shared implementation)
    public void Describe(){Console.WriteLine($"This shape has an area of {Area()}");}
}

public class Circle : Shape
{
    private double radius;
    public Circle(double r) => radius = r;
    public override double Area() => Math.PI * radius * radius;
}

public class Square : Shape
{
    private double side;
    public Square(double s) => side = s;
    public override double Area() => side * side;
}

##Usage:
Shape s1 = new Circle(5);
Shape s2 = new Square(4);

s1.Describe(); // This shape has an area of 78.5398...
s2.Describe(); // This shape has an area of 16
```

**Note** Consumers only care about `Area()`, not how it’s calculated.

## Abstract Class vs Interface

| Feature                         | Abstract Class                             | Interface                                         |
| ------------------------------- | ------------------------------------------ | ------------------------------------------------- |
| Can have method implementations | Yes                                        | (since C# 8 with default methods, but usually ❌) |
| Fields                          | Yes                                        | ❌ No                                             |
| Constructors                    | Yes                                        | ❌ No                                             |
| Multiple inheritance            | ❌ Only one base abstract class            | A class can implement many                        |
| Use case                        | When classes share common state + behavior | When unrelated classes must share a contract      |

**Note** Rule of thumb:

- Use interface if you just need a contract (e.g., `IDisposable`).
- Use abstract class if you need a base class with both _default behavior_ and _mandatory overrides_.

## Quick Summary

- Abstraction in C# = exposing “what” (methods, properties) without “how” (implementation details).
- Achieved via abstract classes and interfaces.
- Helps reduce complexity, enforce contracts, and build scalable systems.

# Other Advance Topics

## 1. Constructors & Destructors

### Constructors

- Special methods used to initialize objects.
- Same name as the class. Can be overloaded.

```c#
public class Person
{
    public string Name { get; }
    public int Age { get; }
    // Default constructor
    public Person()
    {
        Name = "Unknown";
        Age = 0;
    }
    // Parameterized constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
// Usage
Person p1 = new Person();
Person p2 = new Person("Madz", 28);
```

### Static Constructors

- Called once per class, used for initializing static members.

```c#
public class Logger
{
    public static string LogFile;
    static Logger()
    {
        LogFile = "log.txt";
    }
}
```

### Destructors / Finalizers

- `~ClassName()` called by GC before memory is released.
- Rarely needed because C# has `IDisposable`.

## 2. Properties & Auto-Properties

- Encapsulate fields with controlled access.
- Modern C#: auto-properties, `init`-only, expression-bodied.

```c#
public class Person
{
    public string Name { get; private set; }
    public int Age { get; init; }
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
// Usage

var p = new Person("Madz", 28);
// p.Name = "New"; ❌ not allowed (private setter)
// p.Age = 30; ❌ init-only cannot change after initialization
```

Use case: Enforce immutability or validate data when setting fields.

## 3. Static Classes & Members

- Belong to the class itself, not instances.
- Useful for utility/helper methods.

```c#
public static class MathUtils
{
    public static int Add(int a, int b) => a + b;
}

// Usage
int result = MathUtils.Add(5, 7);
```

Use case: Math libraries, extension methods, shared constants.

## 4. Readonly & Const

```c#
public class Config
{
    public const double Pi = 3.1415; // compile-time constant
    public readonly string AppName;   // set in constructor
    public Config(string appName)
    {
        AppName = appName;
    }
}
```

Use case: Constants that never change (`const`) or values initialized once per object (`readonly`).

## 5. Sealed Classes & Methods

```c#
public class Base
{
    public virtual void DoWork() => Console.WriteLine("Base work");
}

public sealed class FinalClass : Base
{
    public override void DoWork() => Console.WriteLine("FinalClass work");
}
// public class Derived : FinalClass {} ❌ Not allowed
```

Use case: Prevent class inheritance for security or design reasons.

## 6. Partial Classes & Partial Methods

```c#
// File1.cs

public partial class Person
{
    public string Name { get; set; }
    partial void OnCreated();
}

// File2.cs

public partial class Person
{
    partial void OnCreated() => Console.WriteLine("Person created");
}
```

Use case: Split large classes, especially with code generation in WinForms, EF Core, or source generators.

## 7. Generics

- Create type-safe, reusable classes or methods.
- Avoid boxing/unboxing, runtime errors.

```c#
public class Repository<T>
{
    private List<T> items = new();
    public void Add(T item) => items.Add(item);
    public T Get(int index) => items[index];
}

// Usage
Repository<int> intRepo = new Repository<int>();
intRepo.Add(5);
Repository<Person> personRepo = new Repository<Person>();
personRepo.Add(new Person("Madz", 28));
```

Use case: Collections, repositories, utilities.

## 8. Delegates & Events

### Delegate

Type-safe function pointer.

```c#
public delegate void Notify(string message);
class Messenger
{
    public Notify OnNotify;
}

var messenger = new Messenger();
messenger.OnNotify = msg => Console.WriteLine(msg);
messenger.OnNotify("Hello!");
```

### Event

Publisher-subscriber pattern.

```c#
public class Button
{
    public event Action Clicked;
    public void Click() => Clicked?.Invoke();
}

var button = new Button();
button.Clicked += () => Console.WriteLine("Button clicked!");
button.Click();
```

Use case: UI events, callbacks, observer pattern.

## 9. Operator Overloading

```c#
public class Point
{
    public int X, Y;
    public Point(int x, int y) { X = x; Y = y; }
    public static Point operator +(Point a, Point b)
        => new Point(a.X + b.X, a.Y + b.Y);
}

// Usage

Point p1 = new(1, 2), p2 = new(3, 4);
Point sum = p1 + p2; // X=4, Y=6
```

Use case: Mathematical or domain-specific classes like `Vector`, `Matrix`.

## 10. Indexers

```c#
public class Team
{
    private string[] members = new string[3];
    public string this[int index]
    {
        get => members[index];
        set => members[index] = value;
    }
}

// Usage
Team team = new Team();
team[0] = "Alice";
Console.WriteLine(team[0]);
```

Use case: Custom collection-like classes.

## 11. Explicit Interface Implementation

```c#
public interface IPrinter { void Print(); }
public interface IScanner { void Print(); }
public class MultiDevice : IPrinter, IScanner
{
    void IPrinter.Print() => Console.WriteLine("Printing...");
    void IScanner.Print() => Console.WriteLine("Scanning...");
}

// Usage
IPrinter p = new MultiDevice();
p.Print(); // Printing
IScanner s = new MultiDevice();
s.Print(); // Scanning
```

Use case: Resolve conflicts when multiple interfaces have same method names.

## 12. Record Types & Immutability (C# 9+)

```c#
public record Person(string Name, int Age);
// Usage
var p1 = new Person("Madz", 28);
var p2 = p1 with { Age = 29 }; // copy with modification
Console.WriteLine(p1); // Person { Name = Madz, Age = 28 }
Console.WriteLine(p2); // Person { Name = Madz, Age = 29 }
```

Use case: DTOs, immutable objects, functional programming patterns.

## 13. IDisposable & Resource Management

```c#
public class FileManager : IDisposable
{
    private StreamWriter writer = new("file.txt");
    public void Write(string text) => writer.WriteLine(text);
    public void Dispose() => writer.Dispose();
}

// Usage
using var file = new FileManager();
file.Write("Hello");
```

Use case: Manage unmanaged resources (files, DB connections, sockets).

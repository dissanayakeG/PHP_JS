# C# Advance

Learn from open source : https://github.com/bitwarden
[https://www.programiz.com/csharp-programming/getting-started](https://www.programiz.com/csharp-programming/getting-started)
[https://www.csharptutorial.net/](https://www.csharptutorial.net/)

# Stack vs Heap in C#

## Stack

- Small, fast memory.
- Stores _value types_ (like `int`, `double`, `struct`) directly.
- Stores _references_ (pointers) to objects on the heap.
- Automatically cleaned up when the method ends (LIFO).

## Heap

- Larger, slower memory.
- Stores _reference type objects_ (`class`, `array`, `string`, etc.).
- Garbage Collector (GC) cleans it up when no references remain.

Example 1: Value Types (Stack Only)

```c#
int x = 10;   // stored directly on the stack
int y = x;    // copy of x, so y = 10 (independent)
y = 20;
Console.WriteLine(x); // 10
Console.WriteLine(y); // 20
```

- Value types are copied by value.

Example 2: Reference Types (Heap + Stack)

```c#
public class Person
{
    public string Name;
}

Person p1 = new Person();
p1.Name = "Alice";

Person p2 = p1;  // copies the reference (pointer), not the object
p2.Name = "Bob";

Console.WriteLine(p1.Name); // Bob (same object on heap)
Console.WriteLine(p2.Name); // Bob
```

- Both `p1` and `p2` point to the same heap object.

Example 3: Struct vs Class

```c#
public struct Point { public int X; public int Y; }
public class PointClass { public int X; public int Y; }

Point s1 = new Point { X = 1, Y = 2 };
Point s2 = s1; // copy of struct

s2.X = 99;

Console.WriteLine(s1.X); // 1 **Note** separate copies
Console.WriteLine(s2.X); // 99

PointClass c1 = new PointClass { X = 1, Y = 2 };
PointClass c2 = c1; // reference copy

c2.X = 99;

Console.WriteLine(c1.X); // 99 ❌ same object
Console.WriteLine(c2.X); // 99
```

Note: Structs live on the stack (copied).  
Note: Classes live on the heap (referenced).

## Strings Are Special

- Strings are reference types but immutable.
- Modifying a string creates a _new_ object on the heap.

```c#
string a = "Hello";
string b = a;
b += " World";

Console.WriteLine(a); // Hello
Console.WriteLine(b); // Hello World
```

## Garbage Collection (GC)

- Stack memory → cleared automatically when scope ends.
- Heap memory → GC runs in the background, cleaning objects that have no active references.
- That’s why you don’t usually need `delete` like in C++ — but you do need `IDisposable` for unmanaged resources.

## Quick Summary

- Value types (structs, ints, bools) → live on the stack (fast, copied by value).
- Reference types (classes, arrays, strings) → object on the heap, reference on the stack.
- Strings → reference types but immutable (always new object on change).
- Garbage Collector manages heap cleanup automatically.

# boxing & unboxing

## What is Boxing & Unboxing?

- Boxing = converting a value type (like `int`, `struct`) into a reference type (object), so it can live on the heap.
- Unboxing = extracting the value type back from the object.
    Think of it like putting a small item into a box (`boxing`) so it can be shipped, then taking it out (`unboxing`) to use it.

Example: Boxing

```c#
int x = 42;        // value type on the stack
object obj = x;     // boxing → x is wrapped in a heap object
```

- `x` stays on the stack.
- `obj` points to a heap object containing the value `42`.

Example: Unboxing

```c#
int y = (int)obj;  // unboxing → extract value from heap back to stack
```

- Cast is mandatory when unboxing.
- The runtime checks the type — invalid cast throws `InvalidCastException`.

## Why Boxing Happens

Boxing is automatic when:

1. Passing value types to methods expecting `object`:

```c#
void PrintObject(object o) => Console.WriteLine(o);
int n = 10;
PrintObject(n); // boxing occurs
```

2. Storing value types in collections of objects (before generics):

```c#
ArrayList list = new ArrayList();
list.Add(5); // boxing
int a = (int)list[0]; // unboxing
```

**Performance Note**
- Boxing allocates on the heap and adds GC pressure.
- Frequent boxing/unboxing in loops can hurt performance.

**Note** Prefer generics to avoid it:

```c#
List<int> list = new List<int>();
list.Add(5); // no boxing
int a = list[0]; // no unboxing
```

## Structs & Boxing

- Structs are value types, so boxing occurs when cast to `object` or an interface.

```c#
struct Point { public int X; public int Y; }
Point p = new Point { X = 1, Y = 2 };
object o = p; // boxing
Point p2 = (Point)o; // unboxing
```

## Quick Summary

- Boxing → stack value → heap object.
- Unboxing → heap object → stack value (cast required).
- Avoid unnecessary boxing by using generics and keeping value types as value types when possible.

# LINQ (Language Integrated Query)

LINQ lets you query collections (arrays, lists, XML, databases) using a SQL-like syntax.

```c#
var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };
// Query syntax
var evensQuery = from n in numbers
                 where n % 2 == 0
                 select n;
// Method syntax
var evensMethod = numbers.Where(n => n % 2 == 0);
```

**Note** Both return: `2, 4, 6`

## Common LINQ Operations:

- Where → filter items
- Select → transform items
- OrderBy / OrderByDescending → sort
- Count / Sum / Average / Max → aggregate
- Join → combine collections

Example with objects:

```c#
var people = new List<Person>
{
    new Person { Name = "Alice", Age = 25 },
    new Person { Name = "Bob", Age = 17 },
    new Person { Name = "Charlie", Age = 30 }
};

var adults = people
    .Where(p => p.Age >= 18)
    .OrderBy(p => p.Name)
    .Select(p => p.Name);
// Output: Alice, Charlie
```

# Lambda Expressions

A lambda expression is a short way to write anonymous methods.

**Syntax:**
```c#
(parameters) => expression
(parameters) => { statements }
```

```c#
Func<int, int> square = x => x * x;
Func<int, int, int> add = (a, b) => a + b;
Func<string> greet = () => "Hello World!";
```

Used with LINQ:

```c#
var evens = numbers.Where(n => n % 2 == 0);
var squares = numbers.Select(n => n * n);
```

**Note** Think of lambdas as “mini functions on the fly.”

# Delegates vs Func / Action / Predicate vs Lambdas

## Delegates

Type-safe references to methods. They are somewhat similar to JS Functions, and PHP Callables.
**Somewhat similar** is more accurate because delegates are type-safe and strongly typed, whereas JavaScript functions and PHP callables are more loosely typed.

```c#
using System;
public delegate void MyDelegate(string message);

class Program {
    static void PrintMessage(string msg) => Console.WriteLine("Message: " + msg);
    static void PrintUpper(string msg) => Console.WriteLine("Upper: " + msg.ToUpper());
    static void Main() {
        MyDelegate d1 = PrintMessage;
        MyDelegate d2 = PrintUpper;
        d1("Hello C#!");
        d2("Hello C#!");
        // Multicast delegate
        MyDelegate multi = d1 + d2;
        multi("Multicast Example");
    }
}
//Output:
//Message: Hello C#!
//Upper: HELLO C#!
//Message: Multicast Example
//Upper: MULTICAST EXAMPLE
```

```c#
public delegate int MathOperation(int a, int b);
MathOperation op = (x, y) => x + y;
Console.WriteLine(op(3, 4)); // 7
```

## Func / Action / Predicate

- `Func<T1,..., TResult>` → returns value (last type is return)
- `Action<T1,...>` → returns void
- `Predicate<T>` → returns bool

```c#
Func<int, int, int> add = (a, b) => a + b;
Action<string> print = s => Console.WriteLine(s);
Predicate<int> isEven = n => n % 2 == 0;
```

## Lambda Expressions

Shorthand for anonymous methods that match delegate signatures.

```c#
Func<int, bool> isPositive = x => x > 0;
```

**Note** Connection:

- Delegates = contracts
- Func/Action/Predicate = built-in delegate types
- Lambdas = easy syntax for inline functions

# Async / Await

- Makes asynchronous code easier to write and avoids blocking threads.

```c#
public async Task FetchDataAsync()
{
    var httpClient = new HttpClient();
    string result = await httpClient.GetStringAsync("https://example.com");
    Console.WriteLine(result);
}
```

# Generics
- Write reusable, type-safe code.

```c#
List<int> numbers = new List<int> { 1, 2, 3 };
List<string> names = new List<string> { "Alice", "Bob" };
```

# Null Safety

- Use `?`, `??`, and `?.` to handle null values safely.

```c#
int? age = null;
int safeAge = age ?? 18; // default if null
string? name = null;
Console.WriteLine(name?.Length); // safe navigation
```

# Pattern Matching

- Cleaner conditions and switch expressions.

```c#
object obj = 42;
string result = obj switch
{
    int n when n > 0 => "Positive number",
    int n => "Some int",
    _ => "Unknown"
};
```

# Extension Methods

- Add methods to existing classes without modifying them.

```c#
public static class StringExtensions
{
    public static bool IsCapitalized(this string s) =>
        !string.IsNullOrEmpty(s) && char.IsUpper(s[0]);
}
Console.WriteLine("Hello".IsCapitalized()); // true
```

# Records (C# 9+)

- Immutable data types with value-based equality.

```c#
public record Person(string Name, int Age);
var p1 = new Person("Alice", 30);
var p2 = p1 with { Age = 31 }; // clone with new Age
```

## Why record for DTO?

In C# 9+, you usually define a record either with _positional parameters_ or with _init-only properties_.
Using record class with set; is allowed, but the idiomatic approach for DTOs is often init;, because it makes the object immutable after initialization (which is nice for data transfer objects).

**Note** Option 1: Positional record (concise)

```c#
public record Item(int Id, string Name);
var item = new Item(1, "Book");
```

**Note** Option 2: Record with init-only properties

```c#
public record class Item
{
    public int Id { get; init; }
    public string Name { get; init; } = string.Empty;
}
var item = new Item { Id = 1, Name = "Book" };
```

# Tuples & Deconstruction

- Return multiple values easily.

```c#
(string, int) GetPerson() => ("Alice", 30);
var (name, age) = GetPerson();
```

# Delegates & Events

- Built-in way to handle callbacks and event-driven programming.

```c#
public class Button
{
    public event Action Clicked;
    public void Click() => Clicked?.Invoke();
}
```

# Understanding Delegates in C#, JavaScript, and PHP

## 1. C# Delegates

In C#, a delegate is a type-safe object that references a method (or multiple methods). Delegates allow methods to be passed as parameters, stored, and invoked indirectly.

```c#
using System;
public delegate void MyDelegate(string message);

class Program {
    static void PrintMessage(string msg) => Console.WriteLine("Message: " + msg);
    static void PrintUpper(string msg) => Console.WriteLine("Upper: " + msg.ToUpper());
    static void Main() {
        MyDelegate d1 = PrintMessage;
        MyDelegate d2 = PrintUpper;
        d1("Hello C#!");
        d2("Hello C#!");
        // Multicast delegate
        MyDelegate multi = d1 + d2;
        multi("Multicast Example");
    }
}
//Output:
//Message: Hello C#!
//Upper: HELLO C#!
//Message: Multicast Example
//Upper: MULTICAST EXAMPLE
```

## 2. Comparison: C# Delegates, JS Functions, and PHP Callables

| Feature / Concept  | C# Delegates                                 | JavaScript Functions                          | PHP Callables                           |
| ------------------ | -------------------------------------------- | --------------------------------------------- | --------------------------------------- |
| Declaration        | `public delegate void Printer(string msg);`  | Functions are first-class; no special keyword | Function, `[obj, "method"]`, or closure |
| Assign a method    | `Printer p = PrintMessage;`                  | `let printer = printMessage;`                 | `$printer = 'printMessage';`            |
| Invoke             | `p("Hello");`                                | `printer("Hello");`                           | `$printer("Hello");`                    |
| Anonymous / Lambda | `Printer p = msg => Console.WriteLine(msg);` | `let printer = msg => console.log(msg);`      | `$printer = fn($msg) => print($msg);`   |
| Multicast          | Supported (`p += AnotherMethod;`)            | Manual (store in array and loop)              | Manual (array of callables, then loop)  |
| Type Safety        | Strongly typed                               | Dynamic                                       | Dynamic                                 |
| Events             | Built on delegates (`event`)                 | Built-in (DOM events, Node.js `EventEmitter`) | Built-in via libraries                  |

## 3. Why JavaScript Functions Are First-Class Citizens

In JavaScript, functions are first-class citizens, meaning they are treated like any other value. They can:

1. Be assigned to variables:

```js
const greet = function (name) {
	return `Hello, ${name}`;
};
console.log(greet("Alice"));
```

2. Be passed as arguments (callbacks):

```js
function run(fn) {
	fn("Bob");
}
run(greet);
```

3. Be returned from other functions:

```js
function multiplier(factor) { return function(x) { return x \* factor; }; }
const double = multiplier(2);
console.log(double(5)); // 10
```

4. Be stored in arrays or objects:

```js
const actions = {
	sayHi: () => console.log("Hi"),
	sayBye: () => console.log("Bye"),
};
actions.sayHi();
```

**In short:** JavaScript functions can be assigned, passed, returned, and stored just like numbers or strings.

## 4. PHP Callables

In PHP, a callable is any value you can invoke like a function. No special keyword is needed. Callables include:

- Function name as string
- `[object, "method"]` or `[ClassName, "staticMethod"]`
- Anonymous functions / closures

### 4.1 Function Name as String

```php
<?php

function greet($name) {
    echo "Hello, $name!n";
}
$printer = "greet";
$printer("Alice");
?>
```

### 4.2 Object Method or Static Method

```php
<?php
class Greeter {
    public function sayHello($name) { echo "Hello from object, $name!n"; }
    public static function sayStatic($name) { echo "Hello from static, $name!n"; }
}
$greeter = new Greeter();
$printer1 = [$greeter, "sayHello"];
$printer1("Bob");
$printer2 = ["Greeter", "sayStatic"];
$printer2("Charlie");
?>
```

### 4.3 Anonymous Function / Closure

```php
<?php
$printer = function($msg) { echo "Closure says: $msgn"; };
$printer("Hello from closure!");
?>
```

### 4.4 PHP Arrow Functions (PHP 7.4+)

```php
<?php
$factor = 3;
$mult = fn($x) => $x * $factor;
echo $mult(5); // 15
?>
```
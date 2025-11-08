# C# Fundamentals

# What to install for .NET Dev

> On your dev machine (Ubuntu) → install the SDK only.
> On a server that runs console apps only → install .NET Runtime.
> On a server that runs Web API / MVC apps → install ASP.NET Core Runtime.
> On a Windows machine running desktop apps → install .NET Desktop Runtime.

## .NET Install in Windows 
https://dotnet.microsoft.com/en-us/download

## .NET Install in Ubuntu
Reference : https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?tabs=dotnet9&pivots=os-linux-ubuntu-2204

```bash
dotnet --version
```

## Create a simple console app and run

```bash
dotnet new console -o MyApp
cd MyApp
code .
dotnet run
```

# C# data types

In C#, every piece of data you work with has a type. Data types fall into two main categories:

## 1. Value types

Value types store data directly in memory.
They live on the stack (fast, but size-limited).

### (a) Numeric types

* Integers (whole numbers):

  * `byte` (0 to 255)
  * `sbyte` (-128 to 127)
  * `short` (-32,768 to 32,767)
  * `ushort` (0 to 65,535)
  * `int` (-2,147,483,648 to 2,147,483,647)
  * `uint` (0 to \~4 billion)
  * `long` (big numbers)
  * `ulong` (big positive numbers)

* Floating-point numbers (decimals, fractions):

  * `float` (7-digit precision)
  * `double` (15–16 digit precision, most common)
  * `decimal` (28–29 digits, high precision, good for money)

Example:

```csharp
int age = 25;
double price = 19.99;
decimal salary = 1000.50m;
```

### (b) Other value types

* `bool` → `true` / `false`
* `char` → single character `'A'`
* `struct` → custom value type
* `enum` → named constants

Example:

```csharp
bool isAlive = true;
char grade = 'A';

enum Day { Mon, Tue, Wed }
Day today = Day.Mon;
```

## 2. Reference types

Reference types store a reference (address) to the data, not the data itself.
They live on the heap (slower, but flexible).

### Common reference types:

* `string` → sequence of characters `"Hello"`
* `object` → base type of all types in C#
* `class` → user-defined reference type
* `interface` → contract for classes
* `dynamic` → type checked at runtime
* `array` → collection of same type elements
* `delegate` → reference to a method

Example:

```csharp
string name = "Alice"; 
object anything = 42;
int[] numbers = { 1, 2, 3 };
```

## 3. Nullable types

Value types normally can’t be `null`.
But you can use `?` to allow null:

```csharp
int? optionalAge = null;  // nullable int
```

## 4. Var, dynamic, and object

* `var` → type inferred by compiler (still strongly typed).

  ```csharp
  var number = 10; // int
  ```
* `dynamic` → type decided at runtime.

  ```csharp
  dynamic x = 1;
  x = "Now a string!";
  ```
* `object` → parent of all types, can hold anything.

  ```csharp
  object y = 5;
  y = "Hello";
  ```

## 5. Value vs Reference in action

```csharp
int a = 5;        // value type
int b = a;        // copy of value
b = 10;
Console.WriteLine(a); // 5 (unchanged)

string s1 = "Hi"; // reference type
string s2 = s1;   // reference to same object
s2 = "Hello";
Console.WriteLine(s1); // "Hi" (strings are immutable, new object created)
```

# Strings & Escaping

```csharp
//Escape quotes
Console.WriteLine("Hello \"World\"!");

//Escape backslash
Console.WriteLine("c:\\source\\repos");

// Verbatim strings (`@`) → keeps whitespace & ignores escape sequences
Console.WriteLine(@"    c:\source\repos
	(this is where your code goes)");
```

# Operators

- Compound Assignment Operators
    `+=` `-=` `*=` `++` `--`

    ```csharp
    int value = 1;
    value++;  // 2
    Console.WriteLine("First: " + value);     // 2
    Console.WriteLine($"Second: {value++}");  // 2 (then increments to 3)
    Console.WriteLine("Third: " + value);     // 3
    Console.WriteLine("Fourth: " + (++value));// 4
    ```

## Integer Division vs Decimal Division

- Integer division truncates fractional part

    ```csharp
    decimal x = 7 / 5;  // result = 1 (integer division), then cast to decimal
    ```

- To get fractional result, use `decimal` in operands:

    ```csharp
    decimal x = 7m / 5;         // 1.4
    decimal y = (decimal)7 / 5; // 1.4
    ```

## Casting

- Casting from `decimal` to `int` truncates (does not round):

    ```csharp
    decimal gradePointAverage = 3.99872831m;
    int value = (int)gradePointAverage; // 3
    ```

# Arrays

## Array Declaration

```csharp
// Explicit size
string[] fraudulentOrderIDs = new string[3];
fraudulentOrderIDs[0] = "A123";

// Inline initialization
string[] fraudulentOrderIDs = { "A123", "B456", "C789" };
```

# Loops

## foreach Loop

```csharp
string[] names = { "Rowena", "Robin", "Bao" };
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

# Collections in .NET

1. Non-generic collections (older, in `System.Collections`)
2. Generic collections (modern, in `System.Collections.Generic`)
3. Concurrent/thread-safe collections (in `System.Collections.Concurrent`)
4. Specialized collections (queues, stacks, sets, linked lists, etc.)

### 1. Non-Generic Collections (`System.Collections`)

These predate generics and store items as `object`. They allow mixed types but require casting, which can be error-prone.

### Examples

```c#
using System;
using System.Collections;
class Program {
    static void Main() {
        // ArrayList (dynamic array, can store mixed types)
        ArrayList list = new ArrayList();
        list.Add(1);
        list.Add("hello");
        foreach (var item in list) Console.WriteLine(item);
        // Hashtable (key-value, unordered)
        Hashtable ht = new Hashtable();
        ht["id"] = 101;
        ht["name"] = "Alice";
        Console.WriteLine(ht["name"]); // Alice
    }
}
```

**Note** Use when working with legacy code, but prefer generics.

### 2. Generic Collections (`System.Collections.Generic`)

Type-safe, more performant, modern. These are what you’ll use 95% of the time.

### List

Dynamic array, fast index-based access.

```c#
var numbers = new List<int> {1, 2, 3};
numbers.Add(4);
Console.WriteLine(numbers[2]); // 3
```

### Dictionary<TKey,TValue>

Key-value pairs, fast lookups.

```c#
var dict = new Dictionary<int, string>();
dict[1] = "One";
dict[2] = "Two";
Console.WriteLine(dict[2]); // Two
```

### HashSet

Unique values, no duplicates.

```c#
var set = new HashSet<int> {1, 2, 2, 3};
Console.WriteLine(set.Count); // 3
```

### Queue

FIFO (first in, first out).

```c#
var q = new Queue<string>();
q.Enqueue("A");
q.Enqueue("B");
Console.WriteLine(q.Dequeue()); // A
```

### Stack

LIFO (last in, first out).

```c#
var stack = new Stack<int>();
stack.Push(10);
stack.Push(20);
Console.WriteLine(stack.Pop()); // 20
```

### LinkedList

Doubly linked list.

```c#
var ll = new LinkedList<string>();
ll.AddLast("A");
ll.AddLast("B");
ll.AddFirst("Start");
foreach (var item in ll) Console.WriteLine(item);
```

### 3. Concurrent Collections (`System.Collections.Concurrent`)

Thread-safe for multi-threaded environments.

- ConcurrentDictionary<TKey,TValue>

```c#
var cdict = new ConcurrentDictionary<int, string>();
cdict.TryAdd(1, "One");
Console.WriteLine(cdict[1]); // One
```

- ConcurrentQueue, ConcurrentStack, BlockingCollection  
     Provide thread-safe FIFO/LIFO structures.

### 4. Specialized Collections

- SortedList<TKey,TValue> → Sorted by key, O(log n) lookups.
- SortedDictionary<TKey,TValue> → Like `Dictionary`, but keeps keys sorted.
- SortedSet → Keeps elements sorted and unique.
- ObservableCollection → Notifies UI when items change (used in WPF/MVVM).

Example:

```c#
var sorted = new SortedDictionary<int, string>();
sorted[2] = "Two";
sorted[1] = "One";
foreach (var kv in sorted) Console.WriteLine(kv.Key + ": " + kv.Value);
```

**Note** 
- Use generic collections for most modern development.
- Use concurrent collections in multi-threaded environments.
- Use specialized collections when you need ordering, uniqueness, or UI binding.
- Avoid non-generic collections unless maintaining old code.

**Quick rules of thumb:**

- Reference types (`class`, `record`, `delegate`, `interface`, etc.) → live on the heap, variables hold a reference.
- Value types (`struct`, `enum`, `record struct`) → stored on the stack (or inline in objects/arrays).
- ref struct → forced to the stack only, never heap.
- static → members live in a shared type-level memory space, not instance-based.

# Target-Typed Constructor Invocation

```csharp
Random dice = new(); // no need to repeat type name
```

# Stateless vs Stateful Methods

* Stateless → Call method directly (no object needed).
* Stateful → Create an instance, then call method on that object.

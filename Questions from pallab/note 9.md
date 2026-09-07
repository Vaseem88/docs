## 42. If Exception Occurs in `finally` Block

When an unhandled exception is thrown inside a `finally` block:

* The execution of the `finally` block immediately aborts at the fault line.
* If an exception was already thrown inside the `try` block and had not been caught, **the original exception is discarded and masked** by the new exception emerging from the `finally` block.
* The new exception bubbles up the call stack to the nearest enclosing `catch` handler or terminates the process if unhandled.

```csharp
try
{
    throw new InvalidOperationException("Original failure in try");
}
finally
{
    // The InvalidOperationException will be completely lost!
    throw new NullReferenceException("Secondary failure in finally");
}

```

> **Best Practice**: Always guard operations within `finally` blocks (or dispose methods) with defensive code or dedicated nested `try-catch` blocks if they can throw.

---

## 43. Constructor Allocation Sequence in Case of Inheritance

When an instance of a derived class is created, object allocation and initialization follow this strict deterministic sequence:

1. **Derived class static field initializers** (if first reference).
2. **Base class static field initializers** (if first reference).
3. **Derived class static constructor** runs.
4. **Base class static constructor** runs.
5. **Derived class instance field initializers** evaluate.
6. **Base class instance field initializers** evaluate.
7. **Base class instance constructor body** executes.
8. **Derived class instance constructor body** executes.

```
Creation: new Child()
  │
  ├──► [Child Instance Fields Initialized]
  │
  ├──► [Parent Instance Fields Initialized]
  │
  ├──► [Parent Constructor Body Executes]  (Base initializes first)
  │
  └──► [Child Constructor Body Executes]   (Derived completes last)

```

```csharp
public class Parent
{
    public Parent() => Console.WriteLine("Parent ctor");
}

public class Child : Parent
{
    public Child() : base() => Console.WriteLine("Child ctor");
}

// Output of new Child():
// Parent ctor
// Child ctor

```

---

## 44. How to Prevent Inheritance

Inheritance is prevented in C# using the **`sealed`** keyword on classes.

* **Sealed Class**: Cannot be inherited by any subclass. Attempting to derive triggers compile error `CS0509`.
* **Sealed Member**: Applied to an overridden virtual method or property in a subclass (`sealed override`) to prevent further derived classes down the chain from overriding it.
* **Private Constructors**: Creating a class with only private constructors and no nested classes also prevents external derivation.

```csharp
public sealed class SecurityTokenProvider
{
    // Cannot be inherited
}

public class Base
{
    public virtual void Render() { }
}

public class Intermediate : Base
{
    public sealed override void Render() { } // Cannot be overridden further
}

```

---

## 45. `IEnumerable` vs. `IQueryable`

| Feature | `IEnumerable<T>` | `IQueryable<T>` |
| --- | --- | --- |
| **Namespace** | `System.Collections.Generic` | `System.Linq` |
| **Execution Location** | **In-Memory (Client-side)** | **Out-of-Memory (Server**42. If Exception Occurs in finally Block** |

* If an unhandled exception is thrown inside a `finally` block, execution immediately halts and unwinds up the call stack for an outer handler.
* If another exception was already in-flight inside the `try` block, the exception from `finally` supersedes and masks it, causing the original exception to be lost unless explicitly handled.

---

**43. Constructor Execution Sequence in Case of Inheritance**

* **Field initializers** execute first from derived to base.
* **Constructor bodies** execute strictly from **base to derived** (Base constructor runs completely before the Derived constructor body starts).

---

**44. How to Prevent Inheritance**

* Mark the class with the **`sealed`** keyword:
```csharp
public sealed class NonInheritable { }

```


* Making all constructors `private` also prevents external subclassing.

---

**45. IEnumerable vs IQueryable**

| Feature | `IEnumerable<T>` | `IQueryable<T>` |
| --- | --- | --- |
| **Namespace** | `System.Collections.Generic` | `System.Linq` |
| **Execution** | In-memory (LINQ to Objects) | Out-of-memory / Remote (e.g., LINQ to Entities) |
| **Paging & Filtering** | Executes on client side; fetches all rows first | Translates expressions into native queries (SQL) server-side |
| **Under the Hood** | Uses `Func<T, bool>` delegates | Uses `Expression<Func<T, bool>>` expression trees |

---

**46. IEnumerable vs IEnumerator**

* **`IEnumerable`**: An interface representing a collection that can be iterated over. It exposes a single method: `GetEnumerator()`.
* **`IEnumerator`**: The actual cursor/iterator object holding the state. It exposes `Current`, `MoveNext()`, and `Reset()`.

---

**47. Parse vs TryParse**

* **`int.Parse(str)`**: Converts the string to an integer; throws `FormatException` or `OverflowException` if conversion fails.
* **`int.TryParse(str, out int val)`**: Returns a `bool` (`true` on success, `false` on failure) without throwing exceptions, making it faster and safer for user input.

---

**48. is vs as**

* **`is`**: Checks type compatibility and returns `bool` (or binds variables via pattern matching: `if (obj is string s)`). Works on both value and reference types.
* **`as`**: Attempts a safe cast. Returns the cast object if valid, or `null` if incompatible. Works only with nullable reference or nullable value types.

---

**49. var vs dynamic**

* **`var`**: Statically typed at **compile-time** by type inference. Renaming and type checking happen before build.
* **`dynamic`**: Resolved at **runtime** using the DLR (Dynamic Language Runtime). Bypasses compile-time type safety; invalid calls throw runtime exceptions.

---

**50. const vs readonly**

* **`const`**: Compile-time constant. Must be initialized at declaration; its value is hardcoded directly into the calling IL assemblies.
* **`readonly`**: Run-time constant. Can be assigned either at declaration or within the class constructor; values are resolved dynamically at runtime.

---

**51. Method Hiding**

* Uses the **`new`** keyword on a derived method to suppress compiler warnings and hide an inherited base method with the same signature.
* Unlike overriding (`virtual`/`override`), calling the method via a base class reference executes the **base** implementation, not the derived one.

---

**52. Static Members in C#**

* Belong to the **type itself** rather than any specific object instance.
* Allocated once in memory on the High Frequency Heap upon first access and shared universally across the entire application domain.

---

Would you like code snippets or interview-focused edge cases for any specific question here?

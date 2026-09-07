Here are professional, interview-ready answers framed from the perspective of a senior .NET engineer.

---

### 17. Difference between Virtual and Abstract Method

| Feature | `virtual` Method | `abstract` Method |
| --- | --- | --- |
| **Implementation** | Provides a default implementation. | Contains no body/implementation. |
| **Class Requirement** | Can reside in standard or abstract classes. | Can only reside inside an `abstract` class. |
| **Override Requirement** | Optional to override in derived classes. | **Must** be overridden by non-abstract derived classes. |

```csharp
public abstract class Payment
{
    public abstract void Process(); // Must implement
    public virtual void PrintReceipt() => Console.WriteLine("Standard receipt"); // Optional
}

```

---

### 18. Using Statement

The `using` statement ensures deterministic disposal of unmanaged resources by wrapping an `IDisposable` object in a hidden `try-finally` block, automatically invoking `.Dispose()` even if an unhandled exception occurs.

```csharp
// Modern C# 8+ declaration syntax
using var connection = new SqlConnection(connString);
connection.Open();
// Dispose() is guaranteed at the end of the enclosing scope

```

```
[ Code Scope Begins ]
        │
        ▼
   Allocates Resource (IDisposable)
        │
   Executes Work
        │
   Scope Exits / Exception Thrown
        │
        ▼
   [ finally { resource.Dispose(); } ]

```

---

### 19. Boxing and Unboxing

* **Boxing:** Implicit conversion of a **Value Type** (stack) into an **`object` / Reference Type** (heap), allocating memory and copying the value.
* **Unboxing:** Explicit conversion of an `object` back into a concrete **Value Type**.

```csharp
int count = 42; 
object boxed = count;        // Boxing (Allocates on Heap)
int unboxed = (int)boxed;    // Unboxing (Extracts value to Stack)

```

```
  STACK                        HEAP
┌───────────┐                ┌───────────────────────┐
│ count: 42 │                │ Object Header + 42    │
├───────────┤   Boxing       ├───────────────────────┤
│ boxed: ref├───────────────>│ (Managed Object Body) │
└───────────┘                └───────────────────────┘

```

> **Production Insight:** Frequent boxing/unboxing causes heap fragmentation and GC pressure. Always prefer generic collections (`List<T>`) over legacy non-generic types (`ArrayList`).

---

### 20. Jagged Array

A jagged array is an **array of arrays**, meaning each row can contain a distinct length, unlike a rectangular 2D array (`int[,]`).

```csharp
int[][] jagged = new int[3][];
jagged[0] = new int[2] { 1, 2 };
jagged[1] = new int[4] { 3, 4, 5, 6 };
jagged[2] = new int[1] { 7 };

```

---

### 21. Array vs. ArrayList

* **Array (`int[]`):** Fixed size, strongly-typed, stored contiguously in memory with direct index access. No boxing for primitives.
* **`ArrayList`:** Legacy collection (`System.Collections`), dynamically resized, stores everything as `System.Object`. Requires boxing/unboxing for primitives, lacks compile-time type safety. (Superseded by `List<T>`).

---

### 22. Collection

A collection in .NET is an in-memory data structure used to group, manage, iterate, and manipulate related items dynamically. They implement root interfaces such as `IEnumerable`, `ICollection`, and `IList`/`IDictionary`, supporting operations like dynamic sizing, sorting, searching, and filtering.

---

### 23. Delegate

A **Delegate** is a type-safe, object-oriented function pointer in .## 17. Difference Between Virtual and Abstract Method

An **abstract method** has no implementation in the base class; it acts as a pure contract that any non-abstract derived class *must* override using the `override` keyword. Abstract methods can only exist inside an abstract class.

A **virtual method** provides a default implementation in the base class. Derived classes have the option to override it if custom behavior is needed, or simply inherit the base implementation.

```csharp
public abstract class ReportGenerator
{
    public abstract void ParseData(); // Must be implemented by derived class

    public virtual void Export()      // Default behavior, optionally overridden
    {
        Console.WriteLine("Exporting as generic CSV.");
    }
}

```

---

## 18. Using Statement

The `using` statement ensures deterministic disposal of unmanaged resources by guaranteeing that `Dispose()` is called on types implementing `IDisposable`, even if an unhandled exception occurs. Under the hood, the compiler transforms a `using` block into a `try-finally` block.

```csharp
// Modern C# 8+ using declaration
using var stream = new FileStream("data.bin", FileMode.Open);
stream.ReadByte();
// stream.Dispose() is invoked deterministically at the end of the enclosing scope

```

---

## 19. Boxing and Unboxing

* **Boxing:** Implicit conversion of a value type (stored on the stack) into a reference type (`object` or an interface) allocated on the managed heap.
* **Unboxing:** Explicit conversion from an `object` back down to the original value type.

```text
[Stack]                 [Heap]
+-----------+           +-------------------+
| int x = 42| --Box-->  | Object Header     |
+-----------+           | MethodTable Ptr   |
      ^                 | Value: 42         |
      |                 +-------------------+
      +---Unbox (explicit cast)-+

```

```csharp
int val = 42;
object boxed = val;        // Boxing: allocates reference object on heap
int unboxed = (int)boxed;  // Unboxing: extracts value back to stack

```

Frequent boxing/unboxing causes heap churn and increases GC pressure, which is why generics were introduced.

---

## 20. Jagged Array

A **jagged array** is an "array of arrays." Unlike a multidimensional rectangular array (`int[,]`), each inner array row can have a completely distinct size and allocates its own contiguous memory block independently.

```text
Row 0: [ 10 | 20 ]
Row 1: [ 30 | 40 | 50 | 60 ]
Row 2: [ 70 ]

```

```csharp
int[][] jagged = new int[3][];
jagged[0] = new int[2] { 10, 20 };
jagged[1] = new int[4] { 30, 40, 50, 60 };
jagged[2] = new int[1] { 70 };

```

---

## 21. Array and ArrayList

| Feature | `System.Array` (`T[]`) | `System.Collections.ArrayList` |
| --- | --- | --- |
| **Type Safety** | Strongly typed at compile-time | Non-generic; stores items as `System.Object` |
| **Performance** | High; no boxing for value types | High GC pressure due to boxing/unboxing |
| **Size** | Fixed size at initialization | Dynamically resizes as elements are added |
| **Namespace** | `System` | `System.Collections` (Legacy) |

---

## 22. Collection

A **collection** in .NET is an in-memory data structure that manages groups of related objects. Collections provide structured mechanisms to store, organize, search, and iterate over items. They root from core interfaces such as:

* `IEnumerable` / `IEnumerable<T>`: Forward iteration support via `GetEnumerator()`.
* `ICollection<T>`: Adds counts, add, remove, and copy semantics.
* `IList<T>`: Adds positional indexing (`[i]`).
* `IDictionary<TKey, TValue>`: Provides key-value lookup structures.

---

## 23. Delegate

A **delegate** is a type-safe, object-oriented function pointer in .NET. It holds a reference to a method with a specific signature and return type, allowing methods to be passed as arguments or executed dynamically.

```csharp
public delegate int MathOperation(int a, int b);

public class Calculator
{
    public static int Add(int x, int y) => x + y;
}

// Usage
MathOperation op = Calculator.Add;
int result = op(5, 10); // 15

```

---

## 24. Finalize and Dispose

```text
Dispose Pattern:
Application Code ---> calls Dispose() ---> Cleans unmanaged + managed handles immediately
                                       ---> Calls GC.SuppressFinalize(this)

Garbage Collector:
Object unreachable -> (If not suppressed) -> Placed in Finalization Queue -> Finalizer Thread runs Finalize()

```

* **Dispose:** Declared in `IDisposable`. Explicitly invoked by consumer code or `using` statements to clean up unmanaged resources (e.g., file descriptors, database connections, sockets) immediately.
* **Finalize:** A protected method on `System.Object` (written as a destructor `~MyClass()`). Called asynchronously by the Garbage Collector's finalizer thread when an object is being collected, acting as a non-deterministic fallback if consumers forgot to call `Dispose()`.

---

## 25. What is an Event?

An **event** is an encapsulation layer built on top of a delegate. It implements the publisher-subscriber pattern. While a raw multicast delegate can be invoked or overwritten directly from outside the defining class, an event restricts external callers to only the subscription (`+=`) and unsubscription (`-=`) operators. Only the publisher class can trigger the event.

---

## 26. Async and Await

`async` and `await` enable non-blocking, asynchronous execution via the Task-based Asynchronous Pattern (TAP).

* `async`: Modifies a method signature, telling the Roslyn compiler to rewrite the method into an underlying state machine.
* `await`: Suspends execution of the current method at the yield point and releases the calling thread back to the thread pool to handle other work. Once the awaited `Task` completes, execution resumes on an available thread (or captured synchronization context).

```csharp
public async Task<string> DownloadPayloadAsync(string url)
{
    using var client = new HttpClient();
    // Frees the thread while awaiting network I/O
    string data = await client.GetStringAsync(url);
    return data;
}

```

---

## 27. Race Condition

A **race condition** occurs in concurrent code when two or more threads attempt to read and mutate shared data simultaneously without synchronization, making the final outcome dependent on non-deterministic thread execution timing.

```csharp
private int _counter = 0;

public void Increment()
{
    // Non-atomic operation (read -> compute -> write)
    // Multiple threads hitting this simultaneously cause missed increments
    _counter++; 
    
    // Fix: Interlocked.Increment(ref _counter); or lock(_syncRoot) { _counter++; }
}

```

---

## 28. Importance of Garbage Collector (GC)

The .NET Garbage Collector automates dynamic memory management:

* **Prevents Memory Leaks:** Automatically detects and reclaims objects that are no longer referenced in the application graph.
* **Eliminates Dangling Pointers:** Removes risks of pointers addressing freed memory blocks.
* **Heap Compaction:** Compacts surviving objects in Generations 0, 1, and 2 to mitigate memory fragmentation and optimize allocation speed.
* **Generational Optimization:** Operates under the heuristic that freshly created objects have short lifespans (Gen 0), avoiding full-heap scans during every cycle.

---

## 29. Difference Between Stack and Heap

```text
[ STACK MEMORY ]                         [ HEAP MEMORY ]
(Fast, contiguous, LIFO per-thread)      (Global dynamic storage, tracked by GC)
+-----------------------+                +-------------------------------------+
| Frame: Main()         |                | Gen 0 / Gen 1 / Gen 2 / LOH         |
|   int x = 10;         |                |                                     |
|   Order ref ------->--+--------------> | [Order Instance: Id=101, Value=$50] |
+-----------------------+                +-------------------------------------+

```

* **Stack:** Fast, per-thread memory managed in a Last-In, First-Out (LIFO) model. Stores local primitive value types and reference pointers. Cleaned up immediately when execution exits the stack frame.
* **Heap:** Large pool of global memory where all reference types and long-lived instances reside. Managed non-deterministically by the Garbage Collector.

---

## 30. Value Types and Reference Types

* **Value Types:** Derived from `System.ValueType` (e.g., `int`, `double`, `bool`, `struct`, `enum`). Directly contain their data. Typically allocated on the stack (unless scoped within a reference type instance). Assignment copies the actual value.
* **Reference Types:** Derived from `System.Object` (e.g., `class`, `interface`, `string`, `delegate`, arrays). Store a pointer pointing to the actual data residing on the managed heap. Assignment copies the memory pointer, not the underlying object.

---

## 31. Casting: Implicit Casting vs. Explicit Casting

* **Implicit Casting:** Safe, widening conversions where no data loss or overflow can occur. Handled automatically by the compiler.
* **Explicit Casting:** Potentially unsafe, narrowing conversions where precision loss or runtime exceptions (`InvalidCastException`, `OverflowException`) can occur. Requires explicit cast syntax.

```csharp
// Implicit: int (4 bytes) -> long (8 bytes)
int small = 500;
long large = small; 

// Explicit: double -> int (truncates fractional digits)
double pi = 3.14159;
int truncatedPi = (int)pi; // Result: 3

```

---

## 32. Generic Collection

Generic collections (`System.Collections.Generic`) are strongly typed data structures parameterized with a type variable `T` (e.g., `List<T>`, `Dictionary<TKey, TValue>`).

* **Type Safety:** Compiler guarantees type validation at build time.
* **Performance:** Eliminates boxing and unboxing overhead when handling value types.
* **Code Reusability:** One class implementation works across any type.

```csharp
List<int> numbers = new();
numbers.Add(10);
// numbers.Add("text"); // Compile-time error CS1503

```

---

## 33. Threads

A **thread** is the smallest executable unit scheduled by the Operating System kernel. In .NET, `System.Threading.Thread` allows direct control over an independent execution path with its own dedicated call stack (typically 1MB of memory overhead) and thread context. Creating bare OS threads manually is resource-expensive, which led to the creation of the .NET Managed Thread Pool.

---

## 34. Thread vs. Task

| Criterion | `System.Threading.Thread` | `System.Threading.Tasks.Task` |
| --- | --- | --- |
| **Abstraction Level** | Low-level direct OS thread abstraction | Higher-level abstraction representing an async operation |
| **Resource Weight** | Heavy (~1MB stack overhead, slow start) | Lightweight; pulled from and returned to the ThreadPool |
| **Async Support** | Cannot be awaited | Fully integrated with `async` / `await` |
| **Continuations** | Manual synchronization primitives | Built-in via `.ContinueWith()`, combinators (`WhenAll`, `WhenAny`) |
| **Return Values** | Cannot return values directly | Supports direct returns via `Task<TResult>` |

---

## 35. Func Delegate

`Func<...>` is a built-in generic delegate defined in the `System` namespace that points to a method taking zero to 16 input parameters and **always returns a value**. The final generic type argument signifies the return type.

```csharp
// Func<in T1, in T2, out TResult>
Func<int, int, string> formatSum = (a, b) => $"Sum: {a + b}";

string output = formatSum(4, 6); // "Sum: 10"

```

*(Note: Use `Action<...>` if the method returns `void`.)*

---

## 36. Dependency Container

A **Dependency Container** (IoC / DI container) manages the inversion of control principle by automating the registration, resolution, and lifetime management of dependent services throughout an application's object graph.

In ASP.NET Core (`Microsoft.Extensions.DependencyInjection`), services are registered under three core lifetimes:

* **Transient:** Created brand new every time requested.
* **Scoped:** Created once per incoming client request/lifetime scope.
* **Singleton:** Created once on initial resolution and reused throughout the application lifetime.

---

## 37. Multiple Finally Block

A single `try-catch-finally` construct in C# **cannot have multiple finally blocks**. A single `try` block allows zero or many `catch` blocks to handle distinct exception types, but it can only be paired with **at most one** `finally` block, which guarantees execution cleanup.

To achieve sequential finally-like stages, try blocks must be nested:

```csharp
try 
{
    try 
    {
        // Work
    }
    finally 
    {
        // Inner cleanup
    }
}
finally 
{
    // Outer cleanup
}

```

---

## 38. Out and Ref Keyword

Both keywords pass parameters by reference rather than by value, modifying the variable in the caller's stack frame.

* **`ref`:** The argument **must be initialized** before being passed into the method. The method may read and optionally modify the value.
* **`out`:** The argument **does not need to be initialized** before calling. The invoked method **must assign** a value to the parameter before returning.

```csharp
public void ProcessData(ref int currentCount, out bool isSuccess)
{
    currentCount += 10; // Read and write allowed
    isSuccess = true;   // Must be assigned before exit
}

```

---

## 39. Hashing and Encryption

```text
Hashing:
Plaintext [ "secret123" ] -----> [ SHA-256 Hash Function ] -----> Digest [ 8d969... ] (One-way)

Encryption:
Plaintext [ "secret123" ] -----> [ AES + Secret Key ] ----------> Ciphertext [ a8X#9... ]
                          <----- [ Decrypt + Secret Key ] <-------

```

* **Hashing:** A **one-way**, deterministic mathematical function that maps arbitrary-length data to a fixed-size digest (e.g., SHA-256, BCrypt). It cannot be decrypted back to plaintext. Primary use: password storage and data integrity checks.
* **Encryption:** A **two-way** reversible cryptographic operation that converts plaintext into ciphertext using an algorithm (AES, RSA) and a cryptographic key. The original plaintext can be recovered by providing the appropriate decryption key.

---

## 40. Bundle and Minification

Performance optimization techniques traditionally used in web applications to reduce page load latency:

* **Bundling:** Combines multiple individual static assets (CSS or JavaScript files) into a single unified file. This minimizes the total number of roundtrip HTTP requests the browser must make.
* **Minification:** Strips out non-essential characters from source files without changing code semantics (e.g., removing whitespace, newlines, comments, and shortening local variable names). This shrinks file sizes and cuts download bandwidth.

---

## 41. Window.Ready and Window.Unload

These represent distinct event hooks in the client browser lifecycle:

* **`window.ready` (e.g., jQuery `$(document).ready()` / `DOMContentLoaded`):** Fires as soon as the HTML Document Object Model (DOM) is fully loaded and parsed into memory. Scripts can safely query and manipulate DOM nodes immediately without waiting for external stylesheets, images, or frames to finish downloading.
* **`window.unload` (`window.onunload`):** Fires right before the user navigates away from the page, closes the browser tab, or submits a form. It is historically used for cleanup routines, aborting active network connections, or firing analytics pings (`navigator.sendBeacon()`).

---

Would you like to drill down into the internals of any of these topics, such as the .NET Garbage Collector's generation phases or custom implementations of the IDisposable pattern?

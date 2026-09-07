### Top Section: Core .NET Concepts

#### .NET Framework Architecture

The .NET runtime model separates application code from hardware through managed execution, abstraction layers, and an integrated garbage collector.

```
┌────────────────────────────────────────────────────────┐
│               C# / F# / VB Source Code                 │
└───────────────────────────┬────────────────────────────┘
                            │ Roslyn Compiler
                            ▼
┌────────────────────────────────────────────────────────┐
│      Common Intermediate Language (CIL / IL) Code       │
│                + Assembly Metadata                     │
└───────────────────────────┬────────────────────────────┘
                            │ Class Loader / Verifier
                            ▼
┌────────────────────────────────────────────────────────┐
│             Common Language Runtime (CLR)              │
│  ┌────────────────────────┐  ┌──────────────────────┐  │
│  │ JIT Compiler (RyuJIT)  │  │  Garbage Collector   │  │
│  └───────────┬────────────┘  └──────────────────────┘  │
└──────────────┼─────────────────────────────────────────┘
               ▼
┌────────────────────────────────────────────────────────┐
│                  Native Machine Code                   │
└────────────────────────────────────────────────────────┘

```

* **CLR (Common Language Runtime):** The execution engine that handles execution threads, memory allocation, exception dispatching, and type safety checks.
* **CTS (Common Type System):** Defines cross-language type standards ensuring an `int` in C# maps directly to a `System.Int32` across all .NET languages.
* **CLS (Common Language Specification):** A subset of CTS rules that guarantees library interoperability across .NET consumers.

---

#### Delegates

A **delegate** is a type-safe, object-oriented function pointer holding a reference to a method with a specific signature and return type.

```csharp
// Standard Custom Delegate
public delegate int CalculationHandler(int a, int b);

// Built-in Generic Delegates:
Func<int, int, int> addFunc = (x, y) => x + y;       // Returns a value
Action<string> logAction = msg => Console.WriteLine(msg); // Returns void
Predicate<int> isEven = n => n % 2 == 0;            // Returns bool

```

---

#### Extension Methods

Extension methods allow adding new methods to existing types without modifying the underlying source code, creating derived types, or recompiling the original assembly.

```csharp
// Rules: Must be a static class, static method, with 'this' on the first parameter
public static class StringExtensions
{
    public static bool IsValidEmail(this string input)
    {
        return !string.IsNullOrWhiteSpace(input) && input.Contains('@');
    }
}

// Invocation looks identical to an instance method:
string email = "dev@enterprise.com";
bool valid = email.IsValidEmail();

```

---

#### Anonymous Methods & Lambda Expressions

Introduced in C# 2.0 via the `delegate` keyword, then modernized in C# 3.0 via lambda expressions (`=>`), anonymous methods define inline logic without requiring a declared method name.

```csharp
// C# 2.0 Anonymous Method syntax
Func<int, bool> checkEvenOld = delegate(int x) { return x % 2 == 0; };

// C# 3.0+ Expression Lambda
Func<int, bool> checkEvenNew = x => x % 2 == 0;

```

---

#### SOLID Principles

```
  S ──► Single Responsibility     (One reason to change)
  O ──► Open/Closed               (Open for extension, closed for modification)
  L ──► Liskov Substitution       (Subtypes must substitute base types cleanly)
  I ──► Interface Segregation     (Client-specific, lean interfaces)
  D ──► Dependency Inversion      (Depend on abstractions, not concretions)

```

* **S - Single Responsibility Principle (SRP):** A class should have one, and only one, reason to change.
* **O - Open/Closed Principle (OCP):** Software entities should be open for extension, but closed for modification (e.g., using strategy patterns or plugins).
* **L - Liskov Substitution Principle (LSP):** Derived classes must be substitutable for their base classes without altering program correctness.
* **I - Interface Segregation Principle (ISP):** Clients should not be forced to depend on interfaces they do not use. Prefer small, focused interfaces over large, monolithic ones.
* **D - Dependency Inversion Principle (DIP):** High-level modules should not depend on low-level modules; both should depend on abstractions (e.g., Constructor Injection via `Microsoft.Extensions.DependencyInjection`).

---

### Main C# Interview Questions

#### 1. Difference Between `public`, `static`, and `void`

These three keywords serve different roles when declaring types and members:

| Keyword | Category | Function |
| --- | --- | --- |
| `public` | Access Modifier | Removes visibility restrictions; accessible from any assembly referencing the project. |
| `static` | Member Modifier | Binds the member to the type itself rather than an instantiated object. Allocated once in high-frequency heap memory. |
| `void` | Return Type | Specifies that the method completes its operation without returning any value to the caller. |

```csharp
// Program Entry Point demonstration:
public static void Main(string[] args)
{
    // public: Callable externally by the CLR
    // static: CLR executes it without instantiating 'Program'
    // void:   Returns nothing upon termination
}

```

---

#### 2. Code Compilation in C#

Compilation in C# is a two-step process that ensures platform independence and runtime optimization.

```
[ C# Source Code ] 
       │ 
       │ Roslyn Compiler (csc.exe)
       ▼
[ Assembly (.dll / .exe) ] ──► Contains CIL (Intermediate Language) + Metadata
       │ 
       │ Loaded by CLR onto Host Machine
       ▼
[ RyuJIT Compiler ] ──► Compiles IL methods to native instructions on-demand
       │ 
       ▼
[ Native Machine Code ] (x64 / ARM64 execution)

```

1. **Build Time:** The Roslyn compiler (`csc`) converts C# source code into Common Intermediate Language (CIL/IL) and writes assembly metadata into a Portable Executable (`.dll` or `.exe`).
2. **Runtime:** The CLR loads the assembly. The Just-In-Time (JIT) compiler compiles the IL instructions into native machine code targeting the host architecture (e.g., x64, ARM64) right before the method runs for the first time.

---

#### 3. Access Modifiers in C#

```
┌──────────────────────────────┬─────────────────────────────────────────────────┐
│ Modifier                     │ Accessibility Scope                             │
├──────────────────────────────┼─────────────────────────────────────────────────┤
│ private                      │ Inside the containing class/struct only         │
│ protected                    │ Inside containing class + derived child classes │
│ internal                     │ Anywhere within the same compiled assembly      │
│ protected internal           │ Same assembly OR derived classes in other assm. │
│ private protected            │ Derived classes located ONLY inside same assm.  │
│ public                       │ Unrestricted; accessible globally               │
└──────────────────────────────┴─────────────────────────────────────────────────┘

```

---

#### 4. `continue` vs. `break`

* **`break`:** Terminates the loop or switch block immediately. Execution jumps to the statement following the loop.
* **`continue`:** Terminates only the current iteration, skipping subsequent statements within the loop body and moving execution directly to the next iteration evaluation.

```csharp
for (int i = 0; i < 5; i++)
{
    if (i == 2) continue; // Skips 2; prints 0, 1, 3, 4
    if (i == 4) break;    // Halts the loop entirely
    Console.WriteLine(i);
}

```

---

#### 5. Passing Parameters to a Method

C# provides four primary semantics for passing parameters:

* **Pass by Value (Default):** Passes a copy of the argument. Modifications to value types within the method do not affect the original caller variable.
* **`ref`:** Passes a reference to the existing storage location. The parameter must be initialized before passing, and method changes write back to the caller.
* **`out`:** Passes a reference intended to emit data. The caller variable does not need initialization, but the called method is required to assign a value before exiting.
* **`in`:** Passes by reference for performance (avoiding copying large `readonly struct` instances), but enforces read-only access—the method cannot mutate the value.
* **`params`:** Allows passing a variable number of arguments as a comma-separated list or an array.

```csharp
public void MethodSignatures(int val, ref int refVal, out int outVal, in decimal inVal, params string[] list)
{
    outVal = val + refVal; // 'out' must be assigned
    // inVal = 10m;        // Compile Error: 'in' is read-only
}

```

---

#### 6. `finally` vs. `Finalize`

| Dimension | `finally` Block | `Finalize()` Method |
| --- | --- | --- |
| **Purpose** | Deterministic resource cleanup paired with `try-catch`. | Non-deterministic final cleanup before GC reclamation. |
| **Execution Trigger** | Executes immediately after `try` or `catch`, regardless of exceptions. | Executed by the GC Finalizer thread during collection cycles. |
| **C# Implementation** | `try { } finally { /* runs guaranteed */ }` | Implemented using destructor syntax: `~MyClass() { }`. |
| **Best Practice** | Used for closing streams, database connections, and manual cleanup. | Avoid in production; implement `IDisposable` with `Dispose()` instead. |

---

#### 7. Managed vs. Unmanaged Code

```
┌───────────────────────────────────────┐  ┌───────────────────────────────────────┐
│             MANAGED CODE              │  │            UNMANAGED CODE             │
├───────────────────────────────────────┤  ├───────────────────────────────────────┤
│ • Targeted to CLR environment         │  │ • Compiled straight to machine code   │
│ • Automatic Memory (Garbage Collector)│  │ • Manual alloc/free (malloc, free)    │
│ • Type Safety & Bounds-Checking       │  │ • Risk of buffer overflows & leaks    │
│ • Examples: C#, F#, VB.NET            │  │ • Examples: C, C++, Win32 APIs        │
└───────────────────────────────────────┘  └───────────────────────────────────────┘

```

* **Managed Code:** Runs inside the execution environment of the CLR. The runtime automatically allocates heap memory, manages object lifetimes via the Garbage Collector, enforces type safety, and handles array bounds checking.
* **Unmanaged Code:** Executes directly on the operating system outside the CLR sandbox. The developer is directly responsible for manual memory allocation/deallocation, thread synchronization, and safety (e.g., C/C++ libraries, COM objects, Native OS System calls).

---

#### 8. Abstract Class

An **abstract class** is a base class marked with the `abstract` modifier that cannot be directly instantiated. It serves as an incomplete structural blueprint intended for inheritance.

* Can declare both abstract members (signatures with no implementation) and concrete members (fully implemented methods, fields, properties).
* Can define constructors, access modifiers, static members, and state fields.
* Derived non-abstract classes must provide concrete implementations using the `override` keyword for all inherited abstract members.

```csharp
public abstract class PaymentProcessor
{
    public string ProcessorName { get; set; } = "BaseProcessor";
    
    // Abstract method: Derived classes MUST implement
    public abstract void Process(decimal amount);

    // Concrete method: Reusable as-is
    public void LogTransaction(decimal amount) => Console.WriteLine($"Logged {amount}");
}

```

---

#### 9. Sealed Class

A class declared with the `sealed` modifier prevents other classes from inheriting from it.

* **Primary Uses:** Preserves design integrity by preventing unintended overrides, and provides runtime performance benefits through JIT devirtualization optimizations (converting virtual calls to direct method calls).
* Methods can also be marked `sealed` in derived classes to prevent further overrides down the inheritance hierarchy.

```csharp
public sealed class SecurityTokenProvider
{
    // No other class can extend this implementation
}

```

---

#### 10. Partial Class

The `partial` keyword allows a single class, struct, interface, or method definition to be split across multiple physical files within the same assembly.

```
      FileA.cs                    FileB.cs
┌──────────────────┐        ┌──────────────────┐
│ partial class    │        │ partial class    │
│ CustomerAccount  │        │ CustomerAccount  │
│ {                │        │ {                │
│   public int Id; │        │   public void    │
│ }                │        │   Save() { }     │
└─────────┬────────┘        └─────────┬────────┘
          │                           │
          └─────────────┬─────────────┘
                        ▼ Compiled Together
         ┌─────────────────────────────┐
         │       CustomerAccount       │
         │ (Combined single type in IL)│
         └─────────────────────────────┘

```

* **Compilation:** During the build process, the Roslyn compiler combines all partial segments into a single unified type inside the assembly metadata.
* **Common Use Cases:** Auto-generated tooling code (such as Entity Framework migrations or Windows Forms/WPF designers) kept separate from manual business logic extensions.

---

#### 11. Core OOP Concepts

```
              ┌──────────────────────────────────────────────┐
              │           Object-Oriented Design             │
              └──────┬──────────┬──────────┬──────────┬──────┘
                     │          │          │          │
         ┌───────────┘          │          │          └───────────┐
         ▼                      ▼          ▼                      ▼
  [ Encapsulation ]     [ Abstraction ]  [ Inheritance ]   [ Polymorphism ]
   Hides state via       Exposes high-     Promotes reuse    Enables dynamic
   access boundaries.    level contracts.  via hierarchies.  behavioral dispatch.

```

* **Encapsulation:** Bundles data and the methods operating on that data into a single unit, hiding internal state through access modifiers and public properties.
* **Abstraction:** Hides underlying implementation complexity by exposing only the essential behavioral contract (via interfaces or abstract classes).
* **Inheritance:** Enables code reuse and hierarchical relationships, where a child class acquires the properties and methods of its parent class.
* **Polymorphism:** The ability of different types to respond to the same interface or method signature in their own specialized way (compile-time via overloading; runtime via overriding).

---

#### 12. Method Overloading

**Method Overloading** is compile-time (static) polymorphism, where multiple methods within the same class share the exact same name but have distinct parameter signatures (different types, counts, or order of parameters).

```csharp
public class Logger
{
    public void Log(string message) => Console.WriteLine(message);
    public void Log(string message, LogLevel level) => Console.WriteLine($"[{level}] {message}");
    public void Log(Exception ex) => Console.WriteLine(ex.Message);
    
    // Return-type-only divergence is NOT permitted:
    // public bool Log(string message) { ... } // Compile Error
}

```

---

#### 13. Method Overriding

**Method Overriding** is runtime (dynamic) polymorphism, where a derived class replaces the implementation of a base class method using the `override` keyword.

* The base method must be marked with `virtual`, `abstract`, or `override`.
* The method signature, name, and return type must match the base definition.
* Resolved dynamically at runtime using the object's Virtual Method Table (vtable).

```csharp
public class Document
{
    public virtual void Print() => Console.WriteLine("Printing default layout");
}

public class PdfDocument : Document
{
    public override void Print() => Console.WriteLine("Printing PDF format with margins");
}

```

---

#### 14. `StreamReader` and `StreamWriter`

Helper classes in the `System.IO` namespace designed for reading and writing character-based data to and from byte streams using specific character encodings (defaulting to UTF-8).

* **`StreamWriter`:** Converts primitive text strings into bytes and writes them to a destination backing stream (e.g., `FileStream`, `MemoryStream`).
* **`StreamReader`:** Reads bytes from an underlying stream and converts them back into strings.
* **Best Practice:** Always wrap them in `using` declarations or statements to guarantee that underlying file handles and system streams are flushed and closed properly.

```csharp
string path = "enterprise_metrics.log";

// Writing text
using (var writer = new StreamWriter(path, append: true))
{
    writer.WriteLine($"Audit timestamp: {DateTime.UtcNow}");
}

// Reading text
using (var reader = new StreamReader(path))
{
    string? line;
    while ((line = reader.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
}

```

---

#### 15. Interface

An **interface** is a reference type defining a contract that implementing classes or structs must fulfill.

* **Characteristics:** Contains signatures for methods, properties, indexers, and events. By default, members are `public` and `abstract`.
* **Multiple Implementation:** While C# allows a class to inherit from only one base class, a class can implement multiple interfaces.
* **Modern Features (C# 8.0+):** Supports Default Interface Methods (DIM), allowing you to provide a fallback implementation on the interface without breaking existing implementers.

```csharp
public interface IRepository<T>
{
    Task<T?> GetByIdAsync(int id);
    Task AddAsync(T entity);
}

```

---

#### 16. Difference Between `struct` and `class`

```
           STRUCT (Value Type)                         CLASS (Reference Type)
       ┌────────────────────────┐                   ┌────────────────────────┐
Stack: │ Data values stored     │            Stack: │ Pointer (0x7FFE)       │
       │ directly in-place      │                   └───────────┬────────────┘
       └────────────────────────┘                               │ Points to
                                                    Heap:       ▼
                                                    ┌────────────────────────┐
                                                    │ Object Instance Data   │
                                                    │ + Method Table Pointer │
                                                    │ + Sync Block Index     │
                                                    └────────────────────────┘

```

| Feature | `struct` | `class` |
| --- | --- | --- |
| **Type System** | Value Type (inherits implicitly from `System.ValueType`) | Reference Type (inherits from `System.Object`) |
| **Memory Allocation** | Allocated inline where declared: usually on the **Stack**, or embedded within the containing object on the Heap. | Object instance is allocated on the **Managed Heap**; reference pointer lives on the Stack. |
| **Assignment Semantics** | Copy-by-value. Mutating a copy does not affect the original. | Copy-by-reference. Variables point to the same shared heap instance. |
| **Inheritance** | Does not support class inheritance (can implement interfaces). | Supports single base-class inheritance and multiple interface implementation. |
| **Default Constructor** | Param-less constructor allowed (C# 10+), but values zero-initialized by default. | Fully customizable constructor lifecycle. |
| **Recommended Use** | Small (typically $\le$ 16 bytes), immutable data entities (e.g., `Vector2D`, `DateTime`, `Point`). | Complex business models, entities with mutable lifecycles, and large data structures. |

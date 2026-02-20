# Rust Learning Notes

## Variable Shadowing

**What it is:** Declaring a new variable with the same name as a previous one. The new variable "shadows" the old one.

```rust
let number = "T-H-R-E-E";      // String
let number = 3;                 // i32 - shadows the first
```

**Key insight:** Shadowing creates a *new* variable that can be a *different type*. This is why it works even though you're switching from String to i32.

**Shadowing vs Mutation:**

| Shadowing | Mutation |
|-----------|----------|
| `let x = 5; let x = 3;` | `let mut x = 5; x = 3;` |
| Creates a **new variable** | **Reassigns** the same variable |
| Can change **type** | Must keep **same type** |
| Immutable (unless you explicitly use `mut`) | Explicitly mutable |

**When to use:** When you need to transform data through multiple steps (e.g., string → parse → number) while keeping things immutable.

---

## Integer Types

**i32 vs u8 and others:**

- **i32:** 32-bit signed integer. Range: -2,147,483,648 to 2,147,483,647. Default integer type.
  - "i" = signed (can be negative)

- **u8:** 8-bit unsigned integer. Range: 0 to 255.
  - "u" = unsigned (cannot be negative)

- **Other types:** `i8, i16, i64, i128, u16, u32, u64, u128, isize, usize`

**Key insight:** Type annotations are about precision and safety. Using `u8` for small values tells future readers "this is deliberately small" and prevents overflow bugs.

---

## Floating Point Types

**f32 vs f64:**

| Feature | f32 | f64 |
|---------|-----|-----|
| **Bits** | 32 bits | 64 bits |
| **Precision** | ~7 decimal digits | ~15 decimal digits |
| **Can be negative?** | Yes | Yes |
| **Default?** | No | Yes (3.14 defaults to f64) |

**Key insight:** Both f32 and f64 are *signed* (can be negative). There's no unsigned float variant in Rust. f64 is the default because it's more precise. Use f32 only when you need to save memory.

**Important:** Floating point is approximate. Both types can't represent every decimal perfectly (0.1 is actually infinite in binary). f64 just has smaller rounding errors.

---

## Common Patterns: Reassigning Variables

When you need to reassign a variable of the **same type**, you have options:

### Pattern 1: Use `mut` (straightforward)

```rust
let mut apple_price = 2;
if quantity > 40 {
  apple_price = 1;
}
```

### Pattern 2: Use `if` expression (more idiomatic) ⭐

```rust
let apple_price = if quantity > 40 { 1 } else { 2 };
```

### Pattern 3: Use `match` (for multiple conditions)

```rust
let apple_price = match condition {
  true => 1,
  false => 2,
};
```

**Key insight:** Rust prefers *expressions* over *statements*. The `if` expression pattern is more idiomatic because it avoids mutation entirely. You compute the final value at declaration time rather than mutating it later. This makes logic clearer: "apple_price IS either 1 or 2" rather than "apple_price STARTS at 2, then MAYBE changes to 1."

**When to use `mut`:** Use it when you're building up state across multiple operations. When you just need to pick between values, use an expression instead.

---

## Shadowing vs Expressions

- **Shadowing:** Use when *changing types* (`String` → `i32`)
- **Expressions/`if`:** Use when *same type, different value*
- **`mut`:** Fallback for complex state building, but less idiomatic

---

## Slices: Views vs Copies

**What is a slice?**

A **slice** is a view into a portion of an array or string. It's created with the `&` operator and a range:

```rust
let a = [1, 2, 3, 4, 5];
let nice_slice = &a[1..4];  // Points to elements at indices 1, 2, 3
```

**Slices vs Shallow Copies:**

| Concept | Slices (`&`) | Shallow Copy (Python/JS) | StructureClone |
|---------|--------------|--------------------------|-----------------|
| **What it is** | A *view* into memory | A *new array object* | A *deep copy* |
| **Memory allocation** | Zero new allocations | New array created | New structures created |
| **How it works** | Pointer + length | Copy of values | Recursive copy of all data |
| **Independence** | Shares memory with original | Independent object | Independent object |
| **Cost** | Free | O(n) for n elements | O(n) recursively |

**The key difference:** A slice is **not** a copy at all—it's just metadata (a pointer + length) pointing to the original data in memory. Think of it like a window frame on the original, not a painting.

**Example in memory:**

```text
Original array:   [1][2][3][4][5]
                       ↑  ↑  ↑
Rust slice &a[1..4]:   └──┴──┘  (just a pointer to this section)

Python slice a[1:4]:   [2][3][4]  (NEW array object with copied values)
```

**Value Equality vs Identity:**

When you test a slice with `assert_eq!`, Rust compares **values**, not memory addresses:

```rust
let a = [1, 2, 3, 4, 5];
let nice_slice = &a[1..4];

// Rust dereferences the pointer and compares VALUES
assert_eq!([2, 3, 4], nice_slice);  // ✓ Values match: [2, 3, 4] == [2, 3, 4]
```

The slice **points to** the original memory, but when compared, Rust looks at **what's at those addresses** and compares the values.

**Key insight:** The `&` in `&a[1..4]` means "give me a borrowed reference to this slice, not a copy." This is why Rust's slices are so efficient—zero runtime cost, just a pointer to existing data.

---

## Macros vs Functions: Understanding the `!`

**What is a macro?**

A **macro** looks like a function but is actually compile-time code transformation. You can tell it's a macro because it ends with `!`:

```rust
println!("{name} is {age} years old");  // println! is a MACRO
println!("{}", name);                    // Still a macro, even if it looks simple
```

**Macros vs Functions:**

| Feature | Macro (`!`) | Function |
|---------|------------|----------|
| **When it runs** | Compile time (transformation) | Runtime (execution) |
| **What it does** | Transforms code before compilation | Executes at runtime |
| **Type checking** | Can inspect the input at compile time | Standard runtime type checking |
| **Flexibility** | Can do metaprogramming, custom syntax | Limited to normal function behavior |

**Why `println!` is a macro (not a function):**

At compile time, Rust transforms:

```rust
println!("{name} is {age} years old");
```

Into something roughly like:

```rust
io::_print(format_args!("{} is {} years old", name, age));
```

This happens **before** your code runs. The macro:

1. Inspects the string literal
2. Captures variable names from scope (`name`, `age`)
3. Type-checks everything at compile time
4. Generates optimized code

If `println!` were just a function, you'd lose this compile-time safety.

**Other common macros:**

```rust
vec![1, 2, 3]               // Creates a vector (more convenient than Vec::new())
assert_eq!(a, b)            // Compares with type checking at compile time
format!("{x}")              // Like println! but returns a string instead of printing
```

**Key insight:** Macros are Rust's way of saying "this looks like normal code, but we're doing something special at compile time." The `!` is the signal that something meta is happening.

---

## The `::` Namespace Operator

**What is `::`?**

The `::` operator is a **path separator** that navigates through Rust's module hierarchy. Think of it like a file path:

```rust
std::io::println        // Go into `std`, then `io`, then access `println`
std::collections::HashMap   // Navigate to HashMap in the collections module
Vec::new()              // Access the `new` method on the Vec type
```

**Common patterns:**

```rust
// Accessing library functions
std::fs::read_to_string("file.txt")

// Accessing type methods
String::from("hello")
HashMap::new()

// Accessing nested modules
std::collections::HashMap
std::io::BufReader
```

**How it combines with macros:**

```rust
// `::` finds the location, `!` identifies it as a macro
println!("{}", std::fs::read_to_string("file.txt"));
         ↑     ↑                                      ↑
       macro   namespace path                   macro ends
```

**Key insight:** `::` is about *where* something lives (its path in the module system). The `!` tells you *what* it is (a macro). Together, they let Rust be precise about both location and type.

---

## Arrays vs Vectors: Fixed Size in Rust

**The fundamental difference:**

In **Rust**, arrays have a **fixed size that is part of the type**:

```rust
let a = [1, 2, 3];  // Type: [i32; 3] — the size (3) is BAKED INTO THE TYPE
                     // Compiler knows this will always be exactly 3 elements
```

In **Python and TypeScript**, arrays/lists are always **flexible**:

```python
# Python
a = [1, 2, 3]
a.append(4)  # ✓ Works — now a = [1, 2, 3, 4]
```

```typescript
// TypeScript
const a = [1, 2, 3];
a.push(4);  // ✓ Works — now a = [1, 2, 3, 4]
```

**Why the difference?**

| Aspect | Rust | Python/TypeScript |
|--------|------|------------------|
| **Compilation** | Compiled (static) | Interpreted (dynamic) |
| **Size decision** | Compile time | Runtime |
| **Array type** | Includes size: `[i32; 3]` | Doesn't include size: `int[]`, `number[]` |
| **Memory** | Fixed stack allocation | Dynamic heap allocation |

Rust compiles to machine code ahead of time, so it needs to know array sizes **before** your program runs. Python/TypeScript figure out sizes **as the program runs**.

**Key insight:** This is why Rust has **two** collection types:

- **Array `[T; N]`** — fixed size, known at compile time, stack allocated
- **Vector `Vec<T>`** — dynamic size, grows/shrinks at runtime, heap allocated

Python and TypeScript only have one flexible collection type because they don't need a compile-time distinction.

---

## The `vec!` Macro

**Creating vectors with `vec!`:**

The `vec!` macro has two main forms:

```rust
// Form 1: List of elements
let v = vec![1, 2, 3];  // Creates a vector with 3 elements

// Form 2: Element repeated N times (syntax: element; count)
let v = vec![5; 4];     // Creates [5, 5, 5, 5] — the value 5, repeated 4 times
```

**Why it's a macro:**

Like `println!`, `vec!` is a compile-time macro that generates optimized vector initialization code. You can tell because of the `!`.

**Common use case:**

```rust
let a = [10, 20, 30, 40];
let v = vec![10, 20, 30, 40];  // Manual: create a vector with the same elements

// In production code, you'd use:
let v = a.to_vec();  // Convert array to vector (cleaner, more maintainable)
```

**Key insight:** `vec!` is convenient for writing literal vectors quickly. For converting from arrays, `.to_vec()` is more idiomatic because it stays synchronized with the original data.

---

## Implicit Returns: Expressions Without Semicolons

**The core concept:**

In Rust, the **last expression in a scope** (without a semicolon) is automatically returned as a value. This is a fundamental language feature—statements end with semicolons and discard their value, while expressions without semicolons produce values.

```rust
fn simple() -> i32 {
    5 + 6  // No semicolon → this expression's value (11) is returned
}

fn with_semicolon() -> i32 {
    5 + 6;  // ⚠️ ERROR! Semicolon makes it a statement → returns (), not i32
}
```

**Implicit returns in nested scopes:**

You can have implicit returns at multiple levels. Each scope (if block, else block, match arm, loop, etc.) can produce a value:

```rust
fn example(x: i32) -> i32 {
    if x > 5 {
        x * 2      // if block returns x * 2
    } else {
        x + 1      // else block returns x + 1
    }              // The if/else expression itself returns a value
}
```

**Why this matters:**

The `if` expression (not `if` statement) returns a value from one of its branches. This is why you can assign directly:

```rust
let result = if condition { 10 } else { 20 };  // result is either 10 or 20
```

**Key insight:** The semicolon is Rust's way of saying "I don't care about the value of this expression—execute it for its side effects only." No semicolon means "I want the value from this expression to flow out to the caller."

---

## Iterators: The `.iter()` Bridge

**The problem:**

Collections (slices, arrays, vectors) don't have `.map()` as a direct method. Only **iterators** do. So you need a way to convert a collection into an iterator first.

```rust
let slice = &[1, 2, 3];
slice.map(|x| x + 1)  // ⚠️ ERROR! Slices don't have .map()
slice.iter().map(|x| x + 1)  // ✓ Works! .iter() creates an iterator
```

**The three main iterator types:**

| Iterator | Method | What you get | Use case |
|----------|--------|--------------|----------|
| **Immutable borrow** | `.iter()` | `&T` (borrowed reference) | Read-only transformations |
| **Mutable borrow** | `.iter_mut()` | `&mut T` (mutable reference) | Modify elements in place |
| **Take ownership** | `.into_iter()` | `T` (owned value) | Consume the collection |

**Examples:**

```rust
let vec = vec![1, 2, 3];

// .iter() - borrow immutably
vec.iter().map(|x| x + 1).collect()  // ✓ Works

// .iter_mut() - borrow mutably (modify values)
vec.iter_mut().map(|x| *x = x * 2).collect()  // ✓ Doubles each element

// .into_iter() - take ownership
vec.into_iter().map(|x| x + 1).collect()  // ✓ Consumes vec (can't use it again)
```

**Slices specifically:**

Slices (`&[T]`) require `.iter()` because a slice is already a borrowed view. You can't own the elements, so you iterate by borrowing them:

```rust
let slice: &[i32] = &[1, 2, 3];
slice.iter().map(|x| x + 1).collect()  // Slice elements are borrowed
```

**Key insight:** `.iter()` (and its variants) are the **bridge** between collections and functional methods like `.map()`, `.filter()`, etc. Different scenarios call for different iterator types depending on what you need to do with the elements.

---

## Ownership vs Mutability: Two Separate Concerns

**The fundamental principle:**

In Rust, **ownership** and **mutability** are independent concepts:

- **Ownership** = "Who controls this data?"
- **Mutability** = "Can this owner modify it?"

You can own something without permission to change it. These are two separate decisions.

**Examples:**

```rust
// Ownership WITHOUT mutability (immutable parameter)
fn process(vec: Vec<i32>) -> Vec<i32> {
    // Ownership: YES (parameter owns the vec)
    // Permission to modify: NO (no `mut` keyword)
    // vec.push(88);  // ❌ ERROR - can't modify
    vec
}

// Ownership WITH mutability (mutable parameter)
fn fill_vec(mut vec: Vec<i32>) -> Vec<i32> {
    // Ownership: YES (parameter owns the vec)
    // Permission to modify: YES (`mut` keyword)
    vec.push(88);  // ✓ OK - can modify
    vec
}
```

**Where ownership transfers:**

When you call a function, **ownership transfers at the function call**, regardless of whether the parameter is mutable:

```rust
let vec0 = vec![1, 2, 3];
let vec1 = process(vec0);
// At this point, vec0's ownership has moved into the function
// This is true whether process() has `mut vec` or just `vec`
```

The `mut` keyword only controls **whether the function is allowed to modify** what it owns. It doesn't enable ownership transfer—that happens automatically.

**Comparison: Ownership with `&mut` references:**

```rust
// Taking ownership + making it mutable
fn fill_vec(mut vec: Vec<i32>) -> Vec<i32> {
    vec.push(88);  // Modify because we own it AND it's mutable
    vec
}

// Borrowing mutably (no ownership transfer)
fn fill_vec(vec: &mut Vec<i32>) -> &mut Vec<i32> {
    vec.push(88);  // Modify because we borrowed mutably
    vec
}
```

In the first, ownership moves. In the second, it doesn't—you're just borrowing temporarily.

**Key insight:** The compiler enforces two separate checks:

1. **"Do you own this or borrow it?"** (ownership question)
2. **"Are you allowed to modify it?"** (mutability question)

Both must be satisfied. You can have ownership without mutability (immutable parameter), or borrowing with mutability (`&mut`), but the rules are always: check ownership first, then check mutability permission.

---

## Why Ownership & Borrowing Matter: The Memory Layout Story

**The Problem: Heap Data Without Rules**

Stack data (integers, booleans) can be copied freely—each copy is independent and safe. But heap data (String, Vec) is different:

```rust
// Without ownership rules:
let s1 = String::from("hello");  // s1 points to heap memory
let s2 = s1;                     // What do we copy? Just the pointer?
// Now s1 and s2 point to the SAME heap memory!
// When s1 leaves scope: drop(s1) frees the heap
// When s2 leaves scope: drop(s2) tries to free SAME memory
// → CRASH (double free)
```

**Why the difference?** Because of memory layout:

| Data | Layout | Issue |
|------|--------|-------|
| `u32` | Value stored directly on stack | Copying is cheap & safe. Each copy independent. |
| `String` | Metadata on stack + data on heap | Copying just the pointer = multiple owners of same heap memory |
| `Vec<T>` | Metadata on stack + data on heap | Same problem: multiple pointers = danger |

**Rust's Solution: Ownership**

Only ONE owner can exist for heap data at a time:

```rust
let s1 = String::from("hello");  // s1 OWNS the heap memory
let s2 = s1;                     // Ownership MOVES to s2
// s1 is invalid now—can't use it
// Only s2 owns the heap. When s2 leaves scope: ONE drop() call.

println!("{}", s1);  // ❌ ERROR
println!("{}", s2);  // ✅ OK
```

This prevents double free because ownership is clear and exclusive.

**Borrowing: Multiple Readers, No Ownership Transfer**

If you want the original owner to keep the data but let others *look at* it:

```rust
let s1 = String::from("hello");
let s2 = &s1;  // BORROW: s1 still owns it, s2 can read
let s3 = &s1;  // Another borrow: s1 still owns it

println!("{}", s1);  // ✅ s1 still owns
println!("{}", s2);  // ✅ s2 reading through reference
println!("{}", s3);  // ✅ s3 reading through reference
// Only s1's drop() runs when scope ends
```

**Visual: Why Both Are Needed**

```text
Stack data (no ownership needed):
let x = 5;     let y = x;
┌─────┐        ┌─────┐
│ x=5 │        │ y=5 │  (independent, both stay valid)
└─────┘        └─────┘

Heap data WITH ownership:
let s1 = String::from("hello")    let s2 = s1
┌──────────────┐                  ┌──────────────┐
│ s1: ptr ─────┼──→ [heap]        │ s2: ptr ─────┼──→ [heap]
└──────────────┘                  └──────────────┘
(s1 valid)                        (s1 INVALID, ownership moved)

Heap data WITH borrowing:
let s1 = String::from("hello")    let s2 = &s1
┌──────────────┐                  ┌──────────┐
│ s1: ptr ─────┼──→ [heap]        │ s2: &ptr │
└──────────────┘                  └──────────┘
(s1 still owns, s2 just reads)
```

**Mutable Borrowing: Protecting Data Integrity**

If you borrow mutably (allow modifications), Rust prevents other readers from existing:

```rust
let mut s1 = String::from("hello");
let s2 = &s1;           // Immutable borrow
s1.push_str(" world");  // ❌ ERROR: can't mutate while s2 borrows
println!("{}", s2);     // s2 expects "hello", not "hello world"!
```

**Why?** If `s1` mutates while `s2` is reading, `s2` sees inconsistent data.

**Stack Data: No Ownership Problem**

```rust
let x: u32 = 5;
let y = x;    // Copy the value (cheap, 4 bytes)
let z = x;    // Copy again (still valid)
// All independent. Stack is cleaned up automatically (LIFO).
// No heap = no need for ownership.
```

**Key Insight:**

Ownership and borrowing exist *specifically because of heap memory*:

- **Heap = shared resource** that could have multiple pointers
- **Ownership = exactly ONE responsible owner** (prevents double free)
- **Borrowing = temporary access** without transferring responsibility
- **Stack = no problem** (automatic cleanup, cheap copies, independent)

The rules aren't arbitrary—they're Rust's solution to preventing memory bugs that plague languages like C. Every rule traces back to: "If multiple owners exist for heap data, it crashes."

---

## Structs: Custom Data Structures

**What is a struct?**

A **struct** is a way to group related data together under one custom type name. Instead of passing individual variables around, you bundle them into a single container. It's the Rust equivalent of a TypeScript `interface`, `type`, or `class`—a custom data structure.

**Why structs?**

Structs enforce structure and clarity. Instead of having loose variables floating around, you say: "These fields logically belong together, and here's what they are."

**The Three Struct Flavors:**

| Flavor | Syntax | Field Access | Use Case |
|--------|--------|--------------|----------|
| **Named Field** | `struct Name { field: Type }` | By name: `.field_name` | Most common; clear, self-documenting |
| **Tuple** | `struct Name(Type1, Type2)` | By position: `.0`, `.1` | Obvious from context; lightweight |
| **Unit** | `struct Name;` | No fields | Marker types; compile-time logic |

**Examples of all three:**

```rust
// 1️⃣ NAMED FIELD STRUCT (clarity is important)
struct Person {
    name: String,
    age: u32,
    email: String,
}

let ethan = Person {
    name: String::from("Ethan"),
    age: 30,
    email: String::from("ethan@example.com"),
};
println!("{}", ethan.name);  // Access by name


// 2️⃣ TUPLE STRUCT (meaning is obvious from context)
struct Point(f64, f64);  // Coordinates, order matters
let origin = Point(0.0, 0.0);
println!("{}", origin.0);  // Access by position

struct Color(u8, u8, u8);  // RGB - order is understood
let red = Color(255, 0, 0);


// 3️⃣ UNIT STRUCT (marker type, no data)
struct DatabaseConnection;

let db = DatabaseConnection;  // Just a marker, no fields to set
```

**When to use each:**

| Use Named Fields | Use Tuple | Use Unit |
|------------------|-----------|----------|
| `struct BankAccount { account_number: String, balance: f64 }` | `struct Point(f64, f64)` | `struct Marker;` |
| Complex data with many fields | Simple data where order is obvious | Type marker for compile-time logic |
| Self-documenting code | Lightweight, minimal structure | Traits and generic logic later |

**TypeScript Analogues:**

```typescript
// TypeScript interface (like named field struct)
interface Person {
  name: string;
  age: number;
  email: string;
}

const ethan: Person = {
  name: "Ethan",
  age: 30,
  email: "ethan@example.com",
};

// TypeScript type (similar)
type Person = {
  name: string;
  age: number;
  email: string;
};

// TypeScript class (also similar, but with methods)
class Person {
  name: string;
  age: number;
  email: string;

  constructor(name: string, age: number, email: string) {
    this.name = name;
    this.age = age;
    this.email = email;
  }
}

// TypeScript type alias for tuple struct (closest equivalent)
type Point = [number, number];
const p: Point = [3.14, 2.71];
```

**Key insight:** If you've been using TypeScript interfaces or types to organize data, you already understand what a Rust struct does. The main differences:

- TypeScript interfaces are **compile-time only** (they disappear after compilation)
- Rust structs are **runtime concepts** (baked into the compiled binary)
- Rust gives you three flavors for different scenarios; TypeScript mostly just gives you objects

---

## Stack vs Heap: Memory Regions

**The fundamental principle:**

Both the **stack** and **heap** are regions of your program's memory (RAM). The key difference is how they organize data and what they're optimized for:

| Aspect | Stack | Heap |
|--------|-------|------|
| **Organization** | Organized (LIFO - Last In First Out) | Flexible (unorganized) |
| **Speed** | Fast (simple stack pointer moves) | Slower (allocation/deallocation is complex) |
| **Size** | Fixed per thread | Grows dynamically |
| **Data stored** | Simple values (i32, bool, pointers) | Complex data (String, Vec, structs with heap data) |
| **Allocation** | Automatic (variable declarations) | Manual (you request memory) |
| **Cleanup** | Automatic (scope exit) | Automatic via Rust ownership (drop trait) |

**String vs &str: A Critical Distinction**

This is one of the most important distinctions in Rust:

| Aspect | `String` | `&str` |
|--------|----------|--------|
| **Ownership** | Owns the data | Borrowed reference (no ownership) |
| **Mutability** | Can be mutable (`mut String`) | Always immutable |
| **Memory** | Heap (data) + Stack (metadata: ptr, len, capacity) | Just a reference (pointer + length on stack) |
| **Size at compile time** | Unknown (grows/shrinks at runtime) | Pointer size is known (always 16 bytes: 8-byte pointer + 8-byte length) |
| **Example** | `String::from("hello")` | `"hello"` or `&owned_string[..]` |

**The key insight:** `String` owns data and can change it. `&str` borrows a reference to data (whether on the heap, in the binary, or anywhere) and cannot change it. Immutability isn't because it's "stack data"—it's because **it's a borrowed reference, and borrowed references can't mutate**.

```rust
let literal = "hello";                       // &str → pointer to binary's read-only section
let owned = String::from("hello");           // String → Stack (metadata) + Heap (data)
let borrowed = &owned[0..5];                 // &str → borrowed reference to heap data

// You can modify owned:
let mut s = String::from("hello");
s.push_str(" world");  // ✓ owns the data, so can mutate

// You cannot modify borrowed:
borrowed.push_str("!");  // ✗ error, &str is immutable (borrowed)
```

**Why `String` when you could use `&str`?**

- **`String`:** Use when you need to own the data and modify it (user input, building strings dynamically)
- **`&str`:** Use when you're passing references around (function parameters, avoiding copies)

**Visual memory layout:**

```text
STACK (organized, fast)          HEAP (flexible, slower)
┌─────────────────────┐          ┌──────────────────────────┐
│ x: 5                │          │                          │
├─────────────────────┤          │                          │
│ y: String           │          │ "hello world" data       │
│   ptr: 0x7fff ─────────────────→ (at address 0x7fff)    │
│   capacity: 11      │          │                          │
│   length: 11        │          │                          │
└─────────────────────┘          └──────────────────────────┘
```

**How it works:**

When you declare a simple variable like `let x: i32 = 5;`, the value `5` lives **directly on the stack**. It's stored right there, ready to access instantly.

When you create a `String` like `let y = String::from("hello");`, here's what happens:

1. The **metadata** (pointer, capacity, length) goes on the **stack**
2. The **actual text data** (`"hello"`) goes on the **heap**
3. The pointer on the stack points to the heap location

```rust
let x = 5;                    // Stack: stores 5 directly
let y = String::from("hello"); // Stack: stores pointer, capacity, length
                              // Heap: stores "hello" at the address the pointer points to
```

**Why two regions?**

- **Stack:** Perfect for fixed-size data. Fast because it's just a pointer moving up/down.
- **Heap:** Perfect for variable-size data. Flexible because you can allocate/deallocate anywhere, but slower because of that complexity.

**Ownership and cleanup:**

Rust uses the **drop** trait to automatically clean up heap memory when the owner goes out of scope. When `y` (the String) leaves scope, Rust calls `drop(y)`, which frees the heap memory that `y`'s pointer was pointing to.

```rust
{
    let s = String::from("hello");  // s allocated on stack, "hello" on heap
    // s is used here
}  // s goes out of scope, drop(s) runs, heap memory is freed
```

**What data lives where (the specific rule):**

| Data Type | Where? | Why? | Size Known? |
|-----------|--------|------|-------------|
| `u32`, `i64`, `bool`, `f64` | **Stack** | Fixed size at compile time | ✅ Yes (4, 8, 1, 8 bytes) |
| `char` | **Stack** | Always 4 bytes | ✅ Yes |
| `[T; N]` (fixed array) | **Stack** | Size is compile-time constant | ✅ Yes |
| `&T` (reference/pointer) | **Stack** | Always 8 bytes (address) | ✅ Yes |
| `String` | **Stack** (metadata) + **Heap** (data) | Size can grow at runtime | ❌ No |
| `Vec<T>` | **Stack** (metadata) + **Heap** (data) | Size can grow at runtime | ❌ No |
| `Box<T>` | **Stack** (pointer) + **Heap** (data) | Heap allocation of any type | ❌ No |
| String literals `"hello"` | **Binary's read-only section** | Compile-time constant (embedded in binary) | ✅ Yes |

**The core principle:**

- **If size is known at compile time** → Stack (fast & predictable)
- **If size is unknown/variable at runtime** → Heap (flexible & slower)

**String literal special case:**
String literals (`"hello"`) are **NOT** stored on the heap—they're embedded directly in your binary's read-only data section. But when you do `String::from("hello")`, you create a *new* String struct that allocates heap memory and copies the literal's data into it:

```rust
let literal = "hello";                    // Stack: pointer to binary read-only section
let heap_string = String::from("hello");  // Stack: metadata (ptr, len, capacity)
                                          // Heap: copy of "hello" (allocated at runtime)
```

**Key insight:** The stack/heap distinction exists because **simple data is fixed-size** (put it on the fast stack) while **complex data is variable-size** (put it on the flexible heap). Rust makes you aware of which is which—if you're moving data that uses the heap, ownership rules apply. If it's pure stack data (integers, booleans), copying is free because the compiler can just duplicate the simple value.

---

## Ownership, Mutability, and Memory Location Are Independent

**Critical realization:** These three concepts are completely separate in Rust:

| Concept | Question | Options |
|---------|----------|---------|
| **Ownership** | "Who controls this data?" | One owner (for heap) OR borrowed (&) |
| **Mutability** | "Can the owner modify it?" | `mut` (yes) OR immutable (no) |
| **Memory location** | "Where does it live?" | Stack OR Heap |

These choices are **orthogonal**—you decide each one independently:

```rust
// Owned, immutable, on heap
let s = String::from("hello");

// Owned, MUTABLE, on heap
let mut s = String::from("hello");

// Owned, immutable, on stack
let x = 5i32;

// Owned, MUTABLE, on stack
let mut x = 5i32;

// Borrowed immutable, from anywhere
let ref_s = &s;
let ref_x = &x;

// Borrowed MUTABLE, from anywhere
let mut_ref_s = &mut s;
let mut_ref_x = &mut x;
```

**Why this matters:** You might have owned data (a `String`) that's immutable. You might have borrowed data that's mutable. You might have stack data that's mutable. Don't conflate these concepts—they're independent decisions.

**Example:** In strings2, `word` is a `String` (owned), but it doesn't need to be `mut` to pass `&word` to a function expecting `&str`. The ownership and mutability are separate from the type mismatch you're solving.

---

## Strings in Rust: The Easy Version

There are two types of text in Rust:

**`&str` (string slice) — Read-only borrowing**

- It's text you're *borrowing* to look at
- You can't change it
- Examples: `"hello"`, borrowed text from a `String`

```rust
let text = "hello";  // This is &str
```

**`String` — You own it**

- It's text *you own*
- You can change it if you add `mut`
- Examples: text from user input, text you're building

```rust
let mut text = String::from("hello");
text.push_str(" world");  // ✓ Can change it
```

**Why the rule about borrowing?**

Imagine you lend a friend a comic book page to read. While they're reading it, you start erasing and rewriting it. That's confusing!

So Rust says: **"If someone's reading something, you can't change it while they're reading."**

That's why borrowed text (`&str`) is always read-only.

**The rule:** Borrow = read-only. Own = can change.

---

## String Methods and Operations

**Common string methods:**

| Method | Input | Output | What it does | Example |
|--------|-------|--------|--------------|---------|
| `.trim()` | `&str` | `&str` | Removes whitespace from both ends | `"  hello  ".trim()` → `"hello"` |
| `.replace(old, new)` | `&str`, `&str` | `String` | Replaces all occurrences of a substring | `"cars are cool".replace("cars", "balloons")` → `"balloons are cool"` |
| `.to_string()` | (any type with ToString trait) | `String` | Converts to an owned String | `"hello".to_string()` or `42.to_string()` |
| `.to_lowercase()` | `&str` | `String` | Converts to lowercase | `"HELLO".to_lowercase()` → `"hello"` |
| `.to_uppercase()` | `&str` | `String` | Converts to uppercase | `"hello".to_uppercase()` → `"HELLO"` |

**Key insight:** Notice the difference in return types. Methods that **return `&str`** (like `.trim()`) are just borrowing a slice of the original string—they don't allocate new memory. Methods that **return `String`** (like `.replace()`, `.to_lowercase()`) create new owned strings on the heap.

**String concatenation with `+`:**

The `+` operator works with strings, but it has specific rules due to ownership:

```rust
// Pattern: owned String + borrowed &str = new String
let result = String::from("Hello") + " world";
let result = "Hello".to_string() + " world";  // Same thing
```

Why the asymmetry? The `+` operator **consumes** (takes ownership of) the left side to build a new String. The right side just needs to be readable, so a borrowed `&str` is fine.

```rust
let s1 = String::from("Hello");
let s2 = String::from(" world");
// ❌ let result = s1 + s2;  // ERROR: can't use owned String on right side
// ✓ let result = s1 + &s2;  // OK: borrow s2
```

**`to_string()` vs `String::from()`:**

Both convert `&str` to `String`, but they work on different types:

```rust
// String::from() — specifically for &str and string literals
String::from("hello")
String::from(&my_str)

// .to_string() — works on any type with the ToString trait
"hello".to_string()       // &str → String
42.to_string()            // i32 → "42"
true.to_string()          // bool → "true"
```

For `&str`, both work identically. Choose based on readability.

**String vs &str in method return types:**

When a function returns `&str`, it's returning a borrowed slice—no new heap allocation. When it returns `String`, it's returning a newly owned value on the heap:

```rust
fn trim_me(input: &str) -> &str {
    input.trim()  // Borrows a slice, no new allocation
}

fn compose_me(input: &str) -> String {
    input.to_string() + " world!"  // Creates new String on heap
}

fn replace_me(input: &str) -> String {
    input.replace("cars", "balloons")  // Creates new String on heap
}
```

**Key insight:** The return type tells you about memory—`&str` means "I'm giving you a view into existing data," while `String` means "I've created new data that you own."

---

## Methods vs Functions: `impl` Blocks and `&self`

**The distinction:**

In Rust, **functions** and **methods** are different:

- **Function:** Standalone, takes explicit arguments
- **Method:** Attached to a type, has special `self` parameter

```rust
// FUNCTION (standalone)
fn calculate(weight: u32, rate: u32) -> u32 {
    weight * rate
}

// METHOD (attached to Package via impl block)
impl Package {
    fn get_fees(&self, cents_per_gram: u32) -> u32 {
        self.weight_in_grams * cents_per_gram
    }
}

// Call them differently:
calculate(1500, 3);           // Function call
package.get_fees(3);          // Method call (on an instance)
```

**What is `&self`?**

`&self` is **not** an argument you pass—it's a special parameter that represents the struct instance itself. When you call a method, Rust automatically provides it:

```rust
let package = Package::new(...);
package.get_fees(3);  // Rust converts this to: Package::get_fees(&package, 3)
```

Inside the method, you have access to:

1. **`&self`** = the entire Package struct (all its fields)
2. **Explicit arguments** = what you pass in the parentheses

```rust
fn get_fees(&self, cents_per_gram: u32) -> u32 {
    // self.weight_in_grams is available because of &self
    // cents_per_gram is available because we passed it
    self.weight_in_grams * cents_per_gram
}
```

**The three flavors of `self`:**

| Form | Meaning | Use Case |
|------|---------|----------|
| `&self` | Immutable borrow (read-only access to struct) | Most common; need to read fields |
| `&mut self` | Mutable borrow (can modify struct fields) | Need to mutate the struct |
| `self` | Take ownership (consume the struct) | Converting/transforming the struct into something else |

```rust
impl Package {
    // Read-only: inspect the package
    fn get_fees(&self, rate: u32) -> u32 {
        self.weight_in_grams * rate
    }

    // Mutable: modify the package
    fn update_weight(&mut self, new_weight: u32) {
        self.weight_in_grams = new_weight;
    }

    // Consume: turn the package into something else
    fn into_shipment(self) -> Shipment {
        Shipment::new(self.weight_in_grams, /* ... */)
    }
}
```

**Why `&self` instead of extracting parameters?**

You might think: "Why not just take `weight_in_grams` as an argument?"

```rust
// ❌ Less flexible
fn get_fees(weight: u32, cents_per_gram: u32) -> u32 { /* ... */ }

// ✓ More flexible
fn get_fees(&self, cents_per_gram: u32) -> u32 { /* ... */ }
```

With `&self`, you have the **entire struct available**. If later you need to check `is_international()` or access other fields, you can—without changing the function signature.

**Why no type annotation on `&self`?**

Inside an `impl Package` block, Rust **implicitly knows** that `self` is of type `Package`. You don't need to write `&self: &Package` because it's a language feature:

```rust
impl Package {
    // &self type is implicit — Rust knows it's &Package
    fn get_fees(&self, cents_per_gram: u32) -> u32 { /* ... */ }
}
```

Regular function arguments always need explicit types:

```rust
fn regular(arg: u32) -> u32 { /* needs explicit type */ }
```

**Real-world concrete example:**

Let's say you have a Camera struct and want to add methods:

```rust
struct Camera {
    megapixels: u32,
    battery_level: u32,
    lens_type: String,
}

impl Camera {
    // Every Camera has this capability
    fn take_photo(&self) -> String {
        // Can always access self.megapixels, self.battery_level, etc.
        if self.battery_level > 0 {
            format!("Photo taken at {} MP", self.megapixels)
        } else {
            "Battery dead".to_string()
        }
    }

    fn charge(&mut self) {
        // Can MODIFY the camera's battery
        self.battery_level = 100;
    }

    fn into_box(self) -> Box {
        // CONSUME the camera and turn it into a Box
        Box::new(self)
    }
}

let mut camera = Camera { megapixels: 12, battery_level: 50, lens_type: "50mm".to_string() };

camera.take_photo();        // ✓ Borrows camera immutably to read data
camera.charge();            // ✓ Borrows camera mutably to modify battery_level
let boxed = camera.into_box(); // ✓ Takes ownership—camera no longer exists after this
// camera.take_photo();      // ❌ ERROR! camera was consumed by into_box
```

**Analogy: The Film Camera Manual**

Think of it this way:

- **Struct** = Your camera (physical object with properties: megapixels, battery, lens)
- **impl block** = The instruction manual that comes with it
- **Methods** = The specific instructions ("How to take a photo", "How to charge")
- **Every camera instance** = Has those instructions available to it
- **`&self`** = "Read your own properties to follow these instructions"
- **`&mut self`** = "You're allowed to change your own state while following these instructions"
- **`self`** (consuming) = "These instructions destroy the camera and turn it into something else"

When you call `camera.take_photo()`, you're essentially saying: "Follow the take_photo instruction from the manual, and use YOUR properties (battery, megapixels) to do it."

**Key insight:** Methods are Rust's way of attaching behavior to data. `impl` blocks create the connection, and `&self` gives the method access to the struct's data. This is similar to classes in TypeScript/JavaScript, but Rust separates data (struct) from behavior (impl), giving you more flexibility.

**The flexibility advantage:**

In TypeScript, data and methods are locked together in the class:

```typescript
class Camera {
  megapixels: number;
  takePhoto() { /* ... */ }
}
```

In Rust, you can split them up:

```rust
struct Camera { megapixels: u32 }

// Later, in one part of your app:
impl Camera {
    fn take_photo(&self) { /* ... */ }
}

// Later, in another part of your app:
impl Camera {
    fn charge(&mut self) { /* ... */ }
}
```

You can even have methods in different files! The separation of data and behavior gives you more organizational freedom.

---

## Enums: One of Many Variants

**What is an enum?**

An **enum** (enumeration) is a type that can be **one of several possible variants**. Unlike a struct which bundles multiple fields together, an enum says "this value is either this variant, or that variant, or this other variant—but only ONE at a time."

**The three enum variants (matching how you define them):**

Enums can hold different *shapes* of data. The syntax you use to define each variant affects how you later destructure it:

| Variant Type | Definition | Destructuring | Use Case |
|--------------|-----------|----------------|----------|
| **Unit variant** | `Quit` | `Message::Quit` | No data attached |
| **Struct variant** | `Move { x: i32, y: i32 }` | `Message::Move { x, y }` | Named fields for clarity |
| **Tuple variant** | `Write(String)` or `ChangeColor(u8, u8, u8)` | `Message::Write(text)` or `Message::ChangeColor(r, g, b)` | Unnamed fields; meaning is obvious from context or type |

**Full example:**

```rust
enum Message {
    Quit,                              // Unit variant — no data
    Move { x: i32, y: i32 },           // Struct variant — named fields
    Write(String),                     // Tuple variant — single unnamed field
    ChangeColor(u8, u8, u8),          // Tuple variant — three unnamed fields
}
```

**Key insight about struct vs tuple syntax:**

The difference between `Move { x: i32, y: i32 }` and `ChangeColor(u8, u8, u8)` isn't about the *types* of data—it's about **whether you name the fields**:

- **Struct syntax `{}`** = "I want to name these fields for clarity"
  - `Move { x: i32, y: i32 }` — clearly coordinates
  - Destructure: `Message::Move { x, y }`

- **Tuple syntax `()`** = "The meaning is obvious from context or type"
  - `ChangeColor(u8, u8, u8)` — clearly RGB (three u8s in order)
  - Destructure: `Message::ChangeColor(r, g, b)`

If you had a struct like `Point { x: i32, y: i32 }` and used it in an enum, the enum only sees it as one unnamed thing:

```rust
enum Message {
    Move(Point),  // ← Tuple variant! One unnamed field (happens to be a Point)
}

// Destructure:
Message::Move(point) => {  // Extract the Point, but Move itself is a tuple variant
    println!("{}", point.x);  // The Point's internal structure is separate
}
```

**Pattern matching and destructuring:**

When you use a `match` expression on an enum, the **pattern mirrors the variant's structure**:

```rust
match msg {
    Message::Quit => {
        // Unit variant — nothing to destructure
        println!("Quit");
    }

    Message::Move { x, y } => {
        // Struct variant — destructure with { } to extract named fields
        println!("Move to ({}, {})", x, y);
    }

    Message::Write(text) => {
        // Tuple variant — destructure with ( ) to extract the value
        println!("Write: {}", text);
    }

    Message::ChangeColor(r, g, b) => {
        // Tuple variant with multiple values — extract all three
        println!("RGB({}, {}, {})", r, g, b);
    }
}
```

**Real-world concrete example: State machine**

Imagine you're building a state machine for a UI. Here's an enum representing different actions:

```rust
enum Action {
    Click,                          // Just a marker — no data needed
    Resize { width: u32, height: u32 },  // Named fields — clear what they represent
    Input(String),                  // Single value — a text input
    SetColor(u8, u8, u8),          // Three values — RGB
}

// Handling actions:
match action {
    Action::Click => handle_click(),
    Action::Resize { width, height } => resize_window(width, height),
    Action::Input(text) => process_input(text),
    Action::SetColor(r, g, b) => apply_color(r, g, b),
}
```

**Why enums matter:**

Without enums, you'd need separate types for each action (separate structs), making it
hard to pass them around uniformly:

```rust
// ❌ Without enums — each is a different type
struct ClickAction;
struct ResizeAction { width: u32, height: u32 }
struct InputAction(String);

// Can't easily put these in one collection or function
// let actions: Vec<???> = vec![...];  // What type?

// ✓ With enums — all the same type
enum Action { Click, Resize { ... }, Input(...), ... }
let actions: Vec<Action> = vec![...];  // Works!
```

**Key insight:** Enums are Rust's way of saying "a value is one of these possibilities." The
syntax you use to define each variant—unit, struct, or tuple—tells you exactly how to destructure
it when matching. This is one of Rust's most powerful features for representing different states
or variants of data.

---

## Functional Programming: `.map()`

**What is `.map()`?**

`.map()` is a universal functional programming pattern. It takes a transformation function and
applies it to every element, producing a new collection with transformed values. The concept
is identical across languages—only the syntax differs.

```typescript
// TypeScript
const input = [1, 2, 3];
const output = input.map(element => element + 1);  // [2, 3, 4]
```

```rust
// Rust
let input = [1, 2, 3];
let output: Vec<i32> = input.iter().map(|element| element + 1).collect();
// [2, 3, 4]
```

**Breaking down the Rust version:**

1. **`.iter()`** — Convert the array into an iterator
2. **`.map(|element| element + 1)`** — Apply the transformation to each element
   - `|element|` is Rust's closure syntax (like `=>` in TypeScript/JavaScript)
3. **`.collect()`** — Gather the transformed values into a concrete collection (`Vec`, array, etc.)

**Lazy evaluation:**

`.map()` is **lazy** in both Rust and TypeScript—it doesn't execute the transformation until you consume the iterator:

```rust
let iter = vec![1, 2, 3].iter().map(|x| x + 1);  // Nothing happens yet
let result: Vec<i32> = iter.collect();             // NOW it transforms
```

**Key insight:** `.map()` is about **transforming every element uniformly**. It's the functional
alternative to manual loops. In Rust, the pattern is always: iterator → transformation → `.collect()`.

## Modules and `pub use`: Creating API Boundaries [L1329-end]

The `pub use` statement is a **re-export** tool that decouples your internal code organization
from your public API. Instead of forcing users to know your folder structure, you can flatten
and rename exports to create a cleaner interface.

Imagine you're building a game engine library with internal modules (rendering/graphics,
physics/collision, audio/sound). Without `pub use`, users write verbose imports tied to
your internals: `use my_game_engine::rendering::graphics::Renderer`. If you reorganize
internally, their code breaks. With `pub use` at the library's root level, users import
cleanly: `use my_game_engine::Renderer`. You can reorganize your internal modules
freely—users never know or care where things actually live. The `as` keyword adds semantic
clarity (like `pub use fruits::PEAR as fruit`) when renaming imports.

```rust
// # Internal structure (hidden from users)
// my_game_engine/
// ├── rendering/graphics.rs → Renderer
// ├── physics/collision.rs → CollisionDetector
// └── audio/sound.rs → SoundEngine

// # Public API (via pub use aliases)
use rendering::graphics::Renderer;      // Render lives in rendering/graphics
use physics::collision::CollisionDetector;  // CollisionDetector lives in physics
use audio::sound::SoundEngine;          // Users just import the top-level names
```

**Key insight:** `pub use` is about **separating your internal architecture from your public API**. It's a stability tool—you can refactor internals without breaking user code, and users get a clean, discoverable interface instead of navigating your module hierarchy.

## HashMap vs Map: Key-Value Collections [L1372-end]

HashMap (Rust) and Map (TypeScript) are conceptually the same—both are key-value stores optimized for lookups. The key difference is that Rust's HashMap is **type-strict** (all keys and values must be the same type), while TypeScript's Map is flexible. Both have their own specialized methods instead of using array methods like `.pop()` or `.push()`.

**TypeScript Map methods:**
```javascript
map.set(key, value)      // Add or update entry
map.get(key)             // Retrieve value (returns undefined if not found)
map.delete(key)          // Remove entry
map.has(key)             // Check if key exists
map.clear()              // Remove all entries
map.size                 // Get number of entries
```

**Rust HashMap methods:**
```rust
map.insert(key, value)   // Add or update entry (returns old value)
map.get(key)             // Retrieve value (returns Option<&V>)
map.remove(key)          // Remove entry (returns Option<V>)
map.contains_key(key)    // Check if key exists
map.clear()              // Remove all entries
map.len()                // Get number of entries
map.entry(key).or_insert(default)  // Conditional add (Rust-specific)
```

**Key insight:** Both use **specialized methods for key-based lookups** instead of sequential operations. The `.entry()` method in Rust is particularly powerful—it's a concise way to add a default value only if the key is missing, avoiding redundant lookups.

## Tricky Patterns: Closures, String Borrowing, and Numeric Types [L1399-end]

Three concepts that trip up learners working with functional patterns and enums:

**1. Closure syntax vs match arrow:**

Closures use pipes `| |` (not `=>`). This is different from `match` which uses `=>`:

```rust
match command {
    Command::Uppercase => { /* ... */ }  // match uses =>
}

collection.into_iter().map(|item| transform(item))  // closure uses | |
```

In a closure like `.map(|n| n + 1)`, the `|n|` extracts the parameter, and what comes after is the body.

**2. String `+` operator requires understanding when to borrow:**

The `+` operator works with `String + &str`. Here's where it gets confusing:
- `"bar"` is already `&str` (string literals are automatically borrowed)
- `"bar".repeat(n)` returns a **`String`** (owned), so you need `&` to make it compatible

```rust
let s = String::from("foo");
s + "bar"                    // ✓ "bar" is already &str
s + &"bar".repeat(3)         // ✓ & converts String to &str
// Result: "foobarbarbar"
```

**3. Numeric types: `usize` and when to use them:**

`usize` is an unsigned integer for sizes and counts. When destructuring an enum variant like `Command::Append(usize)`, the extracted variable holds that count:

```rust
enum Command {
    Append(usize),  // Holds a number representing "repeat this N times"
}

match command {
    Command::Append(n) => {  // n is now a usize (e.g., 3, 5, 10)
        // Use n as "how many times to repeat"
    }
}
```

Other numeric types: `u8` (0–255, like RGB), `u32` (general), `i32` (negative allowed).

**Key insight:** Functional patterns (closures, `.map()`, `.collect()`) combined with enum extraction create the most idiomatically Rust code. But pay attention to type details—especially `&` for borrowing and understanding what type each method returns.

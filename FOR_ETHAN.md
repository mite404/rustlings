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

---
title: "[Swift] Swift Docs - 01. The Basics"
excerpt: "본문의 주요 내용을 여기에 입력하세요"

categories: # 카테고리 설정
  - Swift Basics
tags: # 포스트 태그
  - [SwiftDocs]

permalink: /swift/basics/swift-docs/01-The-Basics/ # 포스트 URL

toc: true
toc_sticky: true

date: 2025-12-17
last_modified_at: 2025-12-17
---

# Constants and Variables

Associates a name with a value of a particular type (`welcomMessage` == "Hello")

* constant : value can't be changed once set
* variable : value can change

## Declaring Constants and Variables

Constants and Variables must be declared before being used.

```swift
let maxLoginAttempt = 10 // constant
var currentLoginAttempt = 5 // variable
```
<br>

The declaration and assigning initial value of constant can be separated.
```swift
var environment = "development"
let maximumNumberOfLoginAttempts: Int // declaration

// assign initial value
if environment == "release" {
    maximumNumberOfLoginAttempts = 3
} else {
    maximumNumberOfLoginAttempts = 100
}
```
<br>
  
Single-line declaration of multiple constants/variables

```swift
var x = 0.0, y = 0.0, z = 0.0
```

## Type Annotations

Define the kind of the values the constants / varaibles can store.

```swift
// {name of constant/variable}: {name of the type}
var welcomeMessage: String

var red, green, blue: Double
```

> It's rare to write type annotations in practice. Swift can almost always infer the type to be used for that const/var at the point when it's defined with an initial value. 

## Naming Constants and Variables

Names can contain almost any character, including Unicode characters.

```swift
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow"
```

* Banned
  * whitespace characters
  * mathematical symbols (+ - * /)
  * arrows
  * private-use Unicode scalar values (usually come from custom fonts, ...)
  * line
  * box-drawing characters
  * **begin with a number**
  * **declare with a same name which is declared before**

> To give the same name as a reserved Swift keyword(default, var, ...), surround the keyword with backticks (`)

## Printing Constants and Variables

`print(_:separator:terminator:)` global function : prints output in Xcode's "console" pane
  * terminator : line break by default

**String interpolation** : include name as a placeholder in a string,
prompt Swift the replace it with the value

```swift
print(friendlyWelcome)

// string interpolation
print("The current value of friendlyWelcome is \(friendlyWelcome)")
```

# Comments

Nonexecutable text in code. Ignored by the Swift compiler.

```swift
// This is a comment.

/* This is also a comment
but is written over multiple lines. */

// nested comments
/* This is the start of the first multiline comment.
    /* This is the second, nested multiline comment. */
This is the end of the first multiline comment. */
```

# Semicolons

Use semicolons if you want to write multiple separate statements on a single line.

```swift
let vibe = "good"; // ok
let cat = "🐱"; print(cat)
```

# Integers

Whole numbers with no fractional component.

* signed : positive / zero / negative
* unsigned : positive / zero

Max, min value depends on size (number of bits to store values)

e.g. `UInt8` : 8-bit unsigned integer, `Int32` : 32-bit signed integer

## Integer Bounds

`min`, `max`

```swift
let minValue = UInt8.min  // minValue is equal to 0, and is of type UInt8
let maxValue = UInt8.max  // maxValue is equal to 255, and is of type UInt8
```

Calculations that produce out-of bounds results stop the program's execution. Or use the operation overflow.

## Int

`Int` : Has the same size as the current platform's native words size

e.g. 32-bit platform : `Int` == `Int32`, 64-bit platform : `Int` == `Int64`

Why use `Int`? : adds code consistency, interoperatbility

## UInt

e.g. 32-bit platform : `UInt` == `UInt32`, 64-bit platform : `UInt` == `UInt64`

> Even the values to be stored are known to be nonnegative, use `Int` for code interoperability, avoids the nee to convert between different number types.

## Floating-Point Numbers

Numbers have a fractional component. Unlike `Int` calculations, floating-point math rounds the result to the nearest representable number.

* Use `Double` when don't need to specify an exact size.
* `Float` : 32 bits, `Double` : 64 bits
* Includes -infinity(underflow), infinity(overflow), NaN(not-a-number to present an invalid, undefined result. e.g. 0/0)

```swift
let intNumber = 0/0 // error
let doubleNumber: Double = 0/0 // nan
```

# Type Safety and Type Inference

Every value in Swift program has a type.

* Explicit : type annotation
* Implicit : Swift infers from an initial value

**Type safe** : Swift check the value's type matches the place you use it when building code.

* Pros of type safe language
  * Be clear about the types of values working with.
  * Values are never implicitly converted to another type, avoiding errors.

**Type Inference** : Compiler deduce the type of the expression when it compiles. Swift use type inference when the type is not specified.

```swift
// assigning a literal value (42 here)
let meaningOfLife = 42 // inferred to be of type Int

let anotherPi = 3 + 0.14159 // Double
```

# Numeric Literals

* Int literals
  * decimal : no prefix (17)
  * binary : `0b` prefix (0b10001)
  * octal : `0o` prefix (0o21)
  * hexadecimal : `0x` (0x11)
* Floating-point literals (must always have a number on both sides of the decimal point)
  * decimal : with exponent `e` / `E`
  * hexadecimal : with exponent `p` / `P`

e.g. 

* 1.25e2 means 1.25 x 10², or 125.0.
* 1.25e-2 means 1.25 x 10⁻², or 0.0125.
* 0xFp2 means 15 x 2², or 60.0.
* 0xFp-2 means 15 x 2⁻², or 3.75.

Extra formatting for readability.

```swift
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
```

# Numeric Type Conversion

Use `Int` for general-purpose for interoperable code. 

Use other int types only when needed. (external source/performance/memeory usage...)

## Integer Conversion

Range of numbers can be stored differs for numeric type.

Compile error occurs when number doesn't fit in.

```swift
let cannotBeNegative: UInt8 = -1
// UInt8 can't store negative numbers, and so this will report an error
let tooBig: Int8 = Int8.max + 1
// Int8 can't store a number larger than its maximum value
```


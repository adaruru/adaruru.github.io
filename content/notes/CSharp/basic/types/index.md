---
title: Basic Types
weight: 10
menu:
  notes:
    name: Basic Types
    identifier: notes-csharp-basics-types
    parent: notes-csharp-basics
    weight: 10
---

{{< note title="Variables" >}}
**Normal Declaration:**
```csharp
string msg;
msg = "Hello";
```

---

**Implicit Type (var):**
```csharp
var msg = "Hello";
```
{{< /note >}}


<!-- Declaring Constants -->

{{< note title="Constants" >}}
```csharp
const double Phi = 1.618;
```
{{< /note >}}

<!-- String Type -->
{{< note title="Strings" >}}
```csharp
string str = "Hello";
```

Multiline string (verbatim)
```csharp
string str = @"Multiline
string";
```

String interpolation
```csharp
string name = "World";
string greeting = $"Hello, {name}!";
```
{{< /note >}}

<!-- Number Types -->
{{< note title="Numbers" >}}

**整數 Integer**

| 類型 | 大小 | 數學式 | 實際範圍 | 約莫 |
|------|------|------|------|------|
| `byte` | 8-bit | 0 ~ 2⁸-1 | 0 ~ 255 | 250 |
| `short` | 16-bit | -2¹⁵ ~ 2¹⁵-1 | -32,768 ~ 32,767 | 正負 3 萬，4 位數 |
| `int` | 32-bit | -2³¹ ~ 2³¹-1 | -2.1B ~ 2.1B | 正負 21 億，10 位數|
| `long` | 64-bit | -2⁶³ ~ 2⁶³-1 | -9.2E ~ 9.2E | 正負 922 京，19 位數|

```csharp
byte b = 255;
short s = 32000;
int num = 3;
long l = 100000L;
```

---

**浮點 Floating**

| 類型 | 大小 | 數學式 | 精度 |
|------|------|------|------|
| `float` | 32-bit | ±3.4×10³⁸ | ~7 位數 |
| `double` | 64-bit | ±1.7×10³⁰⁸ | ~15 位數 |
| `decimal` | 128-bit | ±7.9×10²⁸ | ~28 位數 |

```csharp
float f = 3.14f;
double d = 3.14;
decimal m = 3.14m;
```

{{< /note >}}

<!----------- Arrays  ------>

{{< note title="Arrays" >}}

```csharp
// Fixed size array
int[] numbers = new int[5];

// Array with initializer
int[] numbers = { 1, 2, 3, 4, 5 };

// Multi-dimensional array
int[,] matrix = new int[3, 3];
```

{{< /note >}}

<!-- Nullable Types -->

{{< note size="medium" title="Nullable Types">}}

```csharp
int? nullableInt = null;
nullableInt = 5;

// Null coalescing
int value = nullableInt ?? 0;

// Null conditional
string? name = null;
int? length = name?.Length;
```

Nullable types allow value types to represent undefined values.

{{< /note >}}
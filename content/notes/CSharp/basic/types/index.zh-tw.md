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
Typical types

```csharp
int num = 3;           // int (32-bit)
double d = 3.14;       // double (64-bit)
float f = 3.14f;       // float (32-bit)
decimal m = 3.14m;     // decimal (128-bit)
```

Other Types

```csharp
long l = 100000L;      // long (64-bit)
byte b = 255;          // byte (8-bit unsigned)
short s = 32000;       // short (16-bit)
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
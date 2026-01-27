---
title: 介紹
weight: 10
menu:
  notes:
    name: 介紹
    identifier: notes-csharp-basics-intro
    parent: notes-csharp-basics
    weight: 10
---
<!-- A Sample Program -->
{{< note title="Hello World">}}
A sample C# program is shown here.

```csharp
using System;

class Program
{
    static void Main(string[] args)
    {
        string message = GreetMe("world");
        Console.WriteLine(message);
    }

    static string GreetMe(string name)
    {
        return "Hello, " + name + "!";
    }
}
```

Run the program as below:

```bash
$ dotnet run
```
{{< /note >}}

<!-- Declaring Variables -->

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

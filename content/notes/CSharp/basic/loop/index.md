---
title: Loop
weight: 20
menu:
  notes:
    name: Loop
    identifier: loop
    parent: notes-csharp-basics
    weight: 20
---

{{< note title="for" >}}

**Basic for loop**

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);  // 0, 1, 2, 3, 4
}
```

**Reverse iteration**

```csharp
for (int i = 4; i >= 0; i--)
{
    Console.WriteLine(i);  // 4, 3, 2, 1, 0
}
```

**Step increment**

```csharp
for (int i = 0; i < 10; i += 2)
{
    Console.WriteLine(i);  // 0, 2, 4, 6, 8
}
```

{{< /note >}}

{{< note title="foreach" >}}

**Iterate array**

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
foreach (int num in numbers)
{
    Console.WriteLine(num);
}
```

**Iterate List**

```csharp
List<string> names = new List<string> { "Alice", "Bob" };
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

**Iterate Dictionary**

```csharp
Dictionary<string, int> dict = new Dictionary<string, int>
{
    { "one", 1 },
    { "two", 2 }
};
foreach (KeyValuePair<string, int> kvp in dict)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}
```

{{< /note >}}

{{< note title="while / do-while" >}}

**while - 先判斷再執行**

```csharp
int i = 0;
while (i < 5)
{
    Console.WriteLine(i);  // 0, 1, 2, 3, 4
    i++;
}
```

**do-while - 先執行再判斷，至少執行一次**

```csharp
int j = 0;
do
{
    Console.WriteLine(j);  // 0, 1, 2, 3, 4
    j++;
} while (j < 5);
```

{{< /note >}}

{{< note title="break / continue" >}}

**break - 跳出迴圈**

```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 5) break;
    Console.WriteLine(i);  // 0, 1, 2, 3, 4
}
```

**continue - 跳過本次，繼續下一次**

```csharp
for (int i = 0; i < 5; i++)
{
    if (i == 2) continue;
    Console.WriteLine(i);  // 0, 1, 3, 4
}
```

{{< /note >}}

{{< note title="Recursive" >}}

**Factorial 階乘遞迴: 一般遞迴，堆疊累積**

```csharp
int Factorial(int n)
{
    if (n <= 1) return 1;
    return n * Factorial(n - 1);
}
// Factorial(5) = 5 * 4 * 3 * 2 * 1 = 120
```

**Fibonacci 斐波那契數列遞迴: 值等於前兩項相加**

```csharp
int Fibonacci(int n)
{
    if (n <= 1) return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
// Fibonacci(6) = 8  (0, 1, 1, 2, 3, 5, 8)
```

**FactorialTail 尾遞迴: 累加器攜帶中間結果，不累積堆疊**

```csharp
int FactorialTail(int n, int acc = 1)
{
    if (n <= 1) return acc;
    return FactorialTail(n - 1, n * acc);
}
```

{{< /note >}}

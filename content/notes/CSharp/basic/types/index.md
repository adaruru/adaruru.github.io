---
title: Basic Types
weight: 20
menu:
  notes:
    name: Basic Types
    identifier: notes-csharp-basics-types
    parent: notes-csharp-basics
    weight: 20
---
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

<!-- Type Conversion -->

{{< note title="Type Conversion" >}}

```csharp
int i = 2;
double d = (double)i;      // Explicit cast
string s = i.ToString();   // To string
int parsed = int.Parse("42"); // From string
```

{{< /note >}}

<!-- Collections -->

{{< note size="large" title="Collections" >}}

**List**
```csharp
List<int> list = new List<int> { 1, 2, 3 };
list.Add(4);
list.Remove(2);
list.Contains(3);       // true
list.IndexOf(1);        // 0
list.Sort();
list.Reverse();
```

---

**Dictionary**
```csharp
Dictionary<string, int> dict = new Dictionary<string, int>
{
    { "one", 1 },
    { "two", 2 }
};
dict["three"] = 3;
dict.TryGetValue("one", out int val);
dict.ContainsKey("one");   // true
dict.ContainsValue(1);     // true
dict.Keys;                 // ICollection<string>
dict.Values;               // ICollection<int>
```

---

**HashSet**
```csharp
HashSet<int> set = new HashSet<int> { 1, 2, 3 };
set.Add(4);
set.Remove(2);
set.Contains(3);           // true

// Set operations
HashSet<int> other = new HashSet<int> { 3, 4, 5 };
set.UnionWith(other);      // { 1, 3, 4, 5 }
set.IntersectWith(other);  // { 3, 4 }
set.ExceptWith(other);     // { 1 }
```

---

**Stack (LIFO)**
```csharp
Stack<int> stack = new Stack<int>();
stack.Push(1);
stack.Push(2);
stack.Push(3);
int top = stack.Pop();     // 3
int peek = stack.Peek();   // 2 (不移除)
stack.Count;               // 2
stack.Contains(1);         // true
```

---

**Queue (FIFO)**
```csharp
Queue<int> queue = new Queue<int>();
queue.Enqueue(1);
queue.Enqueue(2);
queue.Enqueue(3);
int first = queue.Dequeue();  // 1
int peek = queue.Peek();      // 2 (不移除)
queue.Count;                  // 2
queue.Contains(3);            // true
```

---

**LinkedList**
```csharp
LinkedList<int> linked = new LinkedList<int>();
linked.AddFirst(1);
linked.AddLast(3);
linked.AddAfter(linked.First, 2);  // 1 -> 2 -> 3
linked.RemoveFirst();
linked.RemoveLast();
```

---

**SortedList / SortedDictionary**
```csharp
// SortedList - 依 key 排序，較省記憶體
SortedList<string, int> sortedList = new SortedList<string, int>
{
    { "banana", 2 },
    { "apple", 1 }
};
// Keys: apple, banana

// SortedDictionary - 依 key 排序，插入/刪除較快
SortedDictionary<string, int> sortedDict = new SortedDictionary<string, int>();
```

{{< /note >}}

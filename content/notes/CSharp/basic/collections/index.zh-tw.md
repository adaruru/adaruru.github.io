---
title: Collections
weight: 10
menu:
  notes:
    name: Collections
    identifier: collections
    parent: notes-csharp-basics
    weight: 10
---

<!-- Collections -->

{{< note title="List" >}}

**List\<T\> 一維動態陣列**

```csharp
List<int> list = new List<int> { 1, 2, 3 };
list.Count;         // 3
list.Add(4);        // { 1, 2, 3, 4 }
list.Remove(2);     // 移除值 2，{1, 3, 4}
list.RemoveAt(2);   // 移除索引 2，{1, 3}
list.Contains(3);   // true
list.IndexOf(1);    // 0
list.Sort();
list.Reverse();
```

**List\<List\<T\>\> 巢狀動態陣列，完全動態**

```csharp
List<List<int>> nested = new List<List<int>>
{
    new List<int> { 1, 2 },
    new List<int> { 3, 4, 5 }
};
nested.Add(new List<int> { 6, 7 });  // 新增一列
nested[0].Add(3);                     // 第一列加元素
nested[1][2];                         // 5
nested.Count;                         // 3
```

{{< /note >}}


{{< note title="Array" >}}

**Single 一維陣列，固定長度**

```csharp
int[] numbers = new int[5];
int[] numbers = { 1, 2, 3, 4, 5 };

Array.Sort(numbers);
Array.Reverse(numbers);
var l = numbers.Length; //5
```

---

**Multi-dimensional 多維陣列，矩形固定**

```csharp
int[,] matrix = new int[3,4]
{
    { 1,  2,  3,  4 },
    { 5,  6,  7,  8 },
    { 9, 10, 11, 12 }
};
int val = matrix[1, 2];  // 7
```

---

**Jagged 鋸齒陣列，每列可不同長，Array of Array**

```csharp
int[][] jagged = new int[4][] // 只能固定第一元長度
{
    new int[] { 1, 2 },
    new int[] { 3, 4, 5 },
    new int[] { 6 }
};
int val2 = jagged[1][2];        // 5
jagged[3];                      // null
jagged[3][0];                   // NullReferenceException
jagged[3] = new int[] { 1, 2 }; // 指定位置值，不能 add
jagged.Length;                  // 4

```

{{< /note >}}

{{< note title="Dictionary" >}}

```csharp
Dictionary<string, int> dict = new Dictionary<string, int>
{
    { "one", 1 },
    { "two", 2 }
};
dict["three"] = 3;
var key = dict.FirstOrDefault(x => x.Value == 1).Key; // key = "one"
int val = dict["one"]; // val = 1，throws if missing
dict.TryGetValue("one", out int val); //val = 1，safe, returns bool
dict.ContainsKey("one");   // true
dict.ContainsValue(1);     // true
dict.Keys;                 // ICollection<string>
dict.Values;               // ICollection<int>
```
{{< /note >}}

{{< note title="HashSet" >}}

```csharp
HashSet<int> set = new HashSet<int> { 1, 2, 3 };
set.Add(4);
set.Remove(2);
set.Contains(3);           // true
```

**Set operations**

```csharp
// 
HashSet<int> other = new HashSet<int> { 3, 4, 5 };
set.UnionWith(other);      // { 1, 3, 4, 5 }
set.IntersectWith(other);  // { 3, 4 }
set.ExceptWith(other);     // { 1 }
```

{{< /note >}}

{{< note title="Stack (LIFO)" >}}

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
{{< /note >}}

{{< note title="Queue (FIFO)" >}}

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

{{< /note >}}

{{< note title="LinkedList" >}}

```csharp
LinkedList<int> linked = new LinkedList<int>();
linked.AddFirst(1);
linked.AddLast(3);
linked.AddAfter(linked.First, 2);  // 1 -> 2 -> 3
linked.Count;                      // 3
var f = linked.First;              // 1
var l = linked.Last;               // 3

linked.RemoveFirst();              // 2 -> 3
linked.RemoveLast();               // 2


```

{{< /note >}}

{{< note title="SortedList / SortedDictionary" >}}

**SortedList - 依 key 排序，較省記憶體**

```csharp
SortedList<string, int> sortedList = new SortedList<string, int>
{
    { "banana", 2 },
    { "apple", 1 }
};
// Keys: apple, banana
```

**SortedDictionary - 依 key 排序，插入/刪除較快**

```csharp
SortedDictionary<string, int> sortedDict = new SortedDictionary<string, int>();
```

{{< /note >}}

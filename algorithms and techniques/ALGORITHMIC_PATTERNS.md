# Алгоритмічні патерни — повний довідник

> Шпаргалка для співбесід та практичного кодування. Усі приклади на C#.
> Патерни згруповано за категоріями. Усередині категорії — від базових до складніших.

## Зміст

1. [Two Pointers (Два вказівники)](#1-two-pointers)
2. [Sliding Window (Ковзне вікно)](#2-sliding-window)
3. [Prefix Sum (Префіксні суми)](#3-prefix-sum)
4. [Binary Search (Бінарний пошук)](#4-binary-search)
5. [Hash Map patterns](#5-hash-map-patterns)
6. [Stack patterns](#6-stack-patterns)
7. [Queue & Deque patterns](#7-queue--deque-patterns)
8. [Heap / Priority Queue](#8-heap--priority-queue)
9. [Linked List patterns](#9-linked-list-patterns)
10. [Tree Traversal](#10-tree-traversal)
11. [Graph Traversal (DFS/BFS)](#11-graph-traversal)
12. [Topological Sort](#12-topological-sort)
13. [Union-Find (DSU)](#13-union-find-dsu)
14. [Shortest Paths](#14-shortest-paths)
15. [Minimum Spanning Tree](#15-minimum-spanning-tree)
16. [Dynamic Programming](#16-dynamic-programming)
17. [Greedy](#17-greedy)
18. [Backtracking](#18-backtracking)
19. [Divide and Conquer](#19-divide-and-conquer)
20. [String algorithms](#20-string-algorithms)
21. [Bit Manipulation](#21-bit-manipulation)
22. [Math & Number Theory](#22-math--number-theory)
23. [Advanced Data Structures](#23-advanced-data-structures)
24. [Special tricks](#24-special-tricks)

---

## 1. Two Pointers

Підтримуємо два індекси, які рухаються масивом/списком за певними правилами. Це базовий патерн для оптимізації з `O(n²)` до `O(n)`.

### 1.1 Slow / Fast (заяць і черепаха)

**Ідея:** один вказівник рухається швидше за інший. Використовується для пошуку середини, виявлення циклів, знаходження n-го з кінця.

**Коли використовувати:**
- Знайти середину linked list
- Виявити цикл у linked list (Floyd's cycle detection)
- Знайти початок циклу
- Видалити n-й вузол з кінця
- Виявити дублікат у масиві (Find the Duplicate Number — трактуємо індекси як вказівники)

```csharp
// Пошук середини linked list
ListNode FindMiddle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast?.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}

// Виявлення циклу (Floyd's)
bool HasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast?.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

### 1.2 Opposite ends (два кінці)

**Ідея:** два вказівники стартують з початку і кінця, рухаються назустріч.

**Коли використовувати:**
- Відсортований масив: знайти пару з сумою X (Two Sum II)
- Палідром
- Container With Most Water
- Trapping Rain Water (одна з версій)
- 3Sum, 4Sum (після сортування)

```csharp
// Two Sum II — відсортований масив
int[] TwoSum(int[] nums, int target) {
    int left = 0, right = nums.Length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return new[] { left, right };
        if (sum < target) left++;
        else right--;
    }
    return Array.Empty<int>();
}
```

### 1.3 Same direction (однаковий напрямок)

**Ідея:** два вказівники рухаються в одному напрямку з різною логікою. Часто один "пише" результат, інший "читає".

**Коли використовувати:**
- Remove Duplicates from Sorted Array
- Move Zeroes
- Partition (поділ масиву за умовою)
- Merge sorted arrays in-place

```csharp
// Remove duplicates in-place
int RemoveDuplicates(int[] nums) {
    if (nums.Length == 0) return 0;
    int write = 1;
    for (int read = 1; read < nums.Length; read++) {
        if (nums[read] != nums[read - 1]) {
            nums[write++] = nums[read];
        }
    }
    return write;
}
```

---

## 2. Sliding Window

Підмножина Two Pointers, але з явним поняттям "вікна" — підмасиву чи підрядка, що ковзає.

### 2.1 Fixed-size window (фіксоване вікно)

**Коли використовувати:**
- Максимальна сума підмасиву довжини k
- Середнє значення підмасиву довжини k
- Кількість анаграм довжини k у рядку

```csharp
// Максимальна сума підмасиву довжини k
int MaxSumK(int[] nums, int k) {
    int sum = 0;
    for (int i = 0; i < k; i++) sum += nums[i];
    int max = sum;
    for (int i = k; i < nums.Length; i++) {
        sum += nums[i] - nums[i - k];
        max = Math.Max(max, sum);
    }
    return max;
}
```

### 2.2 Variable-size window (змінне вікно)

**Ідея:** розширюємо вікно, поки умова виконується; звужуємо, коли порушується.

**Коли використовувати:**
- Longest Substring Without Repeating Characters
- Minimum Window Substring
- Longest substring with at most K distinct characters
- Permutation in String
- Subarray sum problems з додатніми числами

```csharp
// Найдовший підрядок без повторів
int LengthOfLongestSubstring(string s) {
    var seen = new Dictionary<char, int>();
    int left = 0, max = 0;
    for (int right = 0; right < s.Length; right++) {
        if (seen.TryGetValue(s[right], out int idx) && idx >= left) {
            left = idx + 1;
        }
        seen[s[right]] = right;
        max = Math.Max(max, right - left + 1);
    }
    return max;
}
```

**Загальний шаблон variable-size window:**

```csharp
int left = 0;
for (int right = 0; right < n; right++) {
    // 1. Додати nums[right] до стану вікна

    while (/* вікно невалідне */) {
        // 2. Видалити nums[left] зі стану
        left++;
    }

    // 3. Оновити відповідь на основі валідного вікна
}
```

---

## 3. Prefix Sum

**Ідея:** попередньо обчислити кумулятивні суми для `O(1)` відповіді на запити суми на діапазоні.

### 3.1 1D Prefix Sum

**Коли використовувати:**
- Range Sum Query (immutable)
- Subarray Sum Equals K
- Continuous Subarray Sum
- Product of Array Except Self (prefix + suffix)

```csharp
// Range sum: prefix[i] = sum of nums[0..i-1]
// Сума nums[l..r] = prefix[r+1] - prefix[l]
int[] BuildPrefix(int[] nums) {
    var prefix = new int[nums.Length + 1];
    for (int i = 0; i < nums.Length; i++)
        prefix[i + 1] = prefix[i] + nums[i];
    return prefix;
}

// Subarray Sum Equals K — комбінація prefix + hashmap
int SubarraySum(int[] nums, int k) {
    var counts = new Dictionary<int, int> { [0] = 1 };
    int sum = 0, result = 0;
    foreach (int n in nums) {
        sum += n;
        if (counts.TryGetValue(sum - k, out int c)) result += c;
        counts[sum] = counts.GetValueOrDefault(sum, 0) + 1;
    }
    return result;
}
```

### 3.2 2D Prefix Sum

Для матриць — сума прямокутника за `O(1)`.

```csharp
// prefix[i+1, j+1] = сума підматриці [0..i, 0..j]
// Сума [r1..r2, c1..c2] = p[r2+1,c2+1] - p[r1,c2+1] - p[r2+1,c1] + p[r1,c1]
```

### 3.3 Difference Array (масив різниць)

**Ідея:** зворотне до prefix sum. Для багатьох діапазонних оновлень.

**Коли використовувати:**
- Range Addition (багато оновлень типу `nums[l..r] += val`)
- Car Pooling
- Corporate Flight Bookings

```csharp
// diff[l] += val; diff[r+1] -= val для кожного запиту
// Потім prefix sum дає фінальний масив
```

---

## 4. Binary Search

**Умова застосування:** монотонність (впорядкованість або монотонна функція).

### 4.1 Класичний бінарний пошук

```csharp
int BinarySearch(int[] nums, int target) {
    int lo = 0, hi = nums.Length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2; // уникаємо переповнення
        if (nums[mid] == target) return mid;
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

### 4.2 Lower bound / Upper bound

Знайти першу позицію, де `nums[i] >= target` (або `> target`).

```csharp
// Перший індекс, де nums[i] >= target
int LowerBound(int[] nums, int target) {
    int lo = 0, hi = nums.Length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```

### 4.3 Binary Search on Answer

**Ідея:** коли відповідь — це число в деякому діапазоні, і ми можемо перевірити, чи підходить кандидат. Шукаємо бінарним пошуком найменше/найбільше, що задовольняє умову.

**Коли використовувати:**
- Koko Eating Bananas (мінімальна швидкість)
- Split Array Largest Sum
- Capacity to Ship Packages
- Find K-th Smallest in sorted matrix
- Median of Two Sorted Arrays

```csharp
// Типовий шаблон
int lo = minPossible, hi = maxPossible;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (CanAchieve(mid)) hi = mid;
    else lo = mid + 1;
}
return lo;
```

### 4.4 Binary Search на обертаному масиві

Rotated Sorted Array — одна з двох половин завжди впорядкована.

---

## 5. Hash Map patterns

### 5.1 Complement search

**Коли використовувати:** Two Sum і похідні.

```csharp
int[] TwoSum(int[] nums, int target) {
    var map = new Dictionary<int, int>();
    for (int i = 0; i < nums.Length; i++) {
        int need = target - nums[i];
        if (map.TryGetValue(need, out int j)) return new[] { j, i };
        map[nums[i]] = i;
    }
    return Array.Empty<int>();
}
```

### 5.2 Frequency counting

**Коли використовувати:**
- Anagram detection
- Top K Frequent Elements
- First Unique Character
- Longest Consecutive Sequence

```csharp
var freq = new Dictionary<char, int>();
foreach (char c in s)
    freq[c] = freq.GetValueOrDefault(c, 0) + 1;
```

### 5.3 Grouping by key

**Коли використовувати:** Group Anagrams, Group Shifted Strings.

```csharp
// Ключ — відсортований рядок, значення — список анаграм
var groups = new Dictionary<string, List<string>>();
foreach (var s in strs) {
    var key = new string(s.OrderBy(c => c).ToArray());
    if (!groups.ContainsKey(key)) groups[key] = new List<string>();
    groups[key].Add(s);
}
```

---

## 6. Stack patterns

### 6.1 Matching / balancing

**Коли:** Valid Parentheses, Simplify Path, Remove Duplicates.

```csharp
bool IsValid(string s) {
    var stack = new Stack<char>();
    var pairs = new Dictionary<char, char> { [')'] = '(', [']'] = '[', ['}'] = '{' };
    foreach (char c in s) {
        if (pairs.ContainsValue(c)) stack.Push(c);
        else if (stack.Count == 0 || stack.Pop() != pairs[c]) return false;
    }
    return stack.Count == 0;
}
```

### 6.2 Monotonic Stack

**Ідея:** стек, у якому елементи впорядковані. Використовується для задач типу "наступний більший/менший елемент".

**Коли використовувати:**
- Next Greater Element
- Daily Temperatures
- Largest Rectangle in Histogram
- Trapping Rain Water (stack-версія)
- Sum of Subarray Minimums
- Remove K Digits

```csharp
// Daily Temperatures — скільки днів чекати теплішого дня
int[] DailyTemperatures(int[] temps) {
    var result = new int[temps.Length];
    var stack = new Stack<int>(); // зберігаємо індекси
    for (int i = 0; i < temps.Length; i++) {
        while (stack.Count > 0 && temps[stack.Peek()] < temps[i]) {
            int j = stack.Pop();
            result[j] = i - j;
        }
        stack.Push(i);
    }
    return result;
}
```

### 6.3 Expression evaluation

Shunting-yard, evaluation of postfix / infix. Basic Calculator задачі.

---

## 7. Queue & Deque patterns

### 7.1 BFS queue (див. розділ про графи)

### 7.2 Monotonic Deque

**Ідея:** дек, який підтримує монотонний порядок. Дає `O(n)` для "максимум/мінімум у ковзному вікні".

**Коли використовувати:**
- Sliding Window Maximum
- Shortest Subarray with Sum at Least K
- Constrained Subsequence Sum

```csharp
// Sliding Window Maximum
int[] MaxSlidingWindow(int[] nums, int k) {
    var dq = new LinkedList<int>(); // індекси, значення спадають
    var result = new int[nums.Length - k + 1];
    for (int i = 0; i < nums.Length; i++) {
        while (dq.Count > 0 && dq.First.Value <= i - k) dq.RemoveFirst();
        while (dq.Count > 0 && nums[dq.Last.Value] < nums[i]) dq.RemoveLast();
        dq.AddLast(i);
        if (i >= k - 1) result[i - k + 1] = nums[dq.First.Value];
    }
    return result;
}
```

---

## 8. Heap / Priority Queue

**Коли використовувати:**
- Top K elements
- K-й найменший/найбільший
- Merge K sorted lists
- Task scheduler
- Median of stream (two heaps)
- Dijkstra
- Huffman coding

```csharp
// Top K Frequent — min-heap розміру K
int[] TopKFrequent(int[] nums, int k) {
    var freq = new Dictionary<int, int>();
    foreach (var n in nums) freq[n] = freq.GetValueOrDefault(n, 0) + 1;

    var heap = new PriorityQueue<int, int>(); // (item, freq)
    foreach (var kv in freq) {
        heap.Enqueue(kv.Key, kv.Value);
        if (heap.Count > k) heap.Dequeue();
    }

    var result = new int[k];
    for (int i = k - 1; i >= 0; i--) result[i] = heap.Dequeue();
    return result;
}
```

### Two Heaps (медіана потоку)

Min-heap для більшої половини, max-heap для меншої. Медіана — корінь одного з них.

---

## 9. Linked List patterns

### 9.1 Reverse

```csharp
ListNode Reverse(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        var next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

### 9.2 Dummy node (dummy head)

**Коли:** операції, що можуть змінити head (Remove, Merge).

```csharp
ListNode MergeTwo(ListNode l1, ListNode l2) {
    var dummy = new ListNode(0);
    var tail = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { tail.next = l1; l1 = l1.next; }
        else { tail.next = l2; l2 = l2.next; }
        tail = tail.next;
    }
    tail.next = l1 ?? l2;
    return dummy.next;
}
```

### 9.3 Split & merge (Reorder List, Palindrome Linked List)

Комбінація: знайти середину → реверснути другу половину → об'єднати чи порівняти.

---

## 10. Tree Traversal

### 10.1 DFS: preorder, inorder, postorder

```csharp
// Recursive inorder
void Inorder(TreeNode root, List<int> result) {
    if (root == null) return;
    Inorder(root.left, result);
    result.Add(root.val);
    Inorder(root.right, result);
}

// Iterative inorder зі стеком
List<int> InorderIter(TreeNode root) {
    var result = new List<int>();
    var stack = new Stack<TreeNode>();
    var curr = root;
    while (curr != null || stack.Count > 0) {
        while (curr != null) { stack.Push(curr); curr = curr.left; }
        curr = stack.Pop();
        result.Add(curr.val);
        curr = curr.right;
    }
    return result;
}
```

### 10.2 BFS / Level order

```csharp
List<List<int>> LevelOrder(TreeNode root) {
    var result = new List<List<int>>();
    if (root == null) return result;
    var queue = new Queue<TreeNode>();
    queue.Enqueue(root);
    while (queue.Count > 0) {
        int size = queue.Count;
        var level = new List<int>();
        for (int i = 0; i < size; i++) {
            var node = queue.Dequeue();
            level.Add(node.val);
            if (node.left != null) queue.Enqueue(node.left);
            if (node.right != null) queue.Enqueue(node.right);
        }
        result.Add(level);
    }
    return result;
}
```

### 10.3 Tree DP / Post-order accumulation

**Коли:** задачі, де відповідь вузла залежить від відповідей дітей (діаметр дерева, максимальна сума шляху, LCA).

```csharp
// Diameter of Binary Tree
int diameter = 0;
int Depth(TreeNode node) {
    if (node == null) return 0;
    int l = Depth(node.left), r = Depth(node.right);
    diameter = Math.Max(diameter, l + r);
    return 1 + Math.Max(l, r);
}
```

### 10.4 Lowest Common Ancestor (LCA)

Базовий випадок — рекурсія: якщо знайшли p чи q — повертаємо його; якщо обидва з різних піддерев — це LCA.

### 10.5 Morris Traversal

Ін-order обхід без рекурсії і без стеку — `O(1)` пам'яті. Використовує тимчасові посилання в листі.

---

## 11. Graph Traversal

### 11.1 DFS (рекурсивний та ітеративний)

```csharp
void Dfs(int node, List<int>[] graph, bool[] visited) {
    visited[node] = true;
    foreach (int next in graph[node])
        if (!visited[next]) Dfs(next, graph, visited);
}
```

**Застосування:** connected components, cycle detection, path finding, flood fill.

### 11.2 BFS

**Застосування:** shortest path in unweighted graph, level-by-level exploration.

```csharp
int BfsShortestPath(int start, int end, List<int>[] graph) {
    var queue = new Queue<(int node, int dist)>();
    var visited = new HashSet<int> { start };
    queue.Enqueue((start, 0));
    while (queue.Count > 0) {
        var (node, d) = queue.Dequeue();
        if (node == end) return d;
        foreach (int next in graph[node]) {
            if (visited.Add(next)) queue.Enqueue((next, d + 1));
        }
    }
    return -1;
}
```

### 11.3 Multi-source BFS

Стартуємо BFS з кількох джерел одночасно. Приклад: "Rotting Oranges", "Walls and Gates", "01 Matrix".

### 11.4 Bidirectional BFS

BFS з обох кінців назустріч. Використовується для Word Ladder — сильно зменшує час.

### 11.5 0-1 BFS

Коли ваги ребер тільки 0 або 1 — використовуємо Deque: ребра з вагою 0 додаємо в початок, з вагою 1 — в кінець. Дає `O(V+E)` замість Dijkstra.

---

## 12. Topological Sort

**Умова:** тільки для DAG (орієнтований ациклічний граф).

### 12.1 Kahn's algorithm (BFS-based)

**Ідея:** беремо вузли з in-degree = 0, додаємо в результат, зменшуємо in-degree сусідів.

```csharp
int[] TopoSort(int n, int[][] edges) {
    var graph = new List<int>[n];
    for (int i = 0; i < n; i++) graph[i] = new List<int>();
    var inDegree = new int[n];
    foreach (var e in edges) {
        graph[e[0]].Add(e[1]);
        inDegree[e[1]]++;
    }
    var queue = new Queue<int>();
    for (int i = 0; i < n; i++) if (inDegree[i] == 0) queue.Enqueue(i);
    var result = new List<int>();
    while (queue.Count > 0) {
        int node = queue.Dequeue();
        result.Add(node);
        foreach (int next in graph[node])
            if (--inDegree[next] == 0) queue.Enqueue(next);
    }
    return result.Count == n ? result.ToArray() : Array.Empty<int>(); // виявлення циклу
}
```

**Застосування:** Course Schedule, Alien Dictionary, Task Scheduling з залежностями, порядок збірки проєкту.

### 12.2 DFS-based

DFS з позначенням вузлів (white/gray/black), додавання у стек при виході. Розгорнутий стек = топологічний порядок.

---

## 13. Union-Find (DSU)

**Ідея:** структура для динамічного об'єднання множин і швидкої перевірки "чи в одній множині".

**Коли використовувати:**
- Кількість зв'язних компонент
- Connected Components in Graph
- Number of Islands II
- Kruskal's MST
- Accounts Merge
- Redundant Connection
- Перевірка двочасткового графа (через DSU на парах)

```csharp
class DSU {
    int[] parent, rank;
    public DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    public int Find(int x) {
        if (parent[x] != x) parent[x] = Find(parent[x]); // path compression
        return parent[x];
    }
    public bool Union(int a, int b) {
        int ra = Find(a), rb = Find(b);
        if (ra == rb) return false;
        if (rank[ra] < rank[rb]) (ra, rb) = (rb, ra);
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;
        return true;
    }
}
```

**Амортизована складність:** майже `O(α(n))` — практично константа.

---

## 14. Shortest Paths

| Алгоритм | Коли застосовувати | Складність |
|----------|-------------------|------------|
| BFS | Невагомі графи | O(V+E) |
| 0-1 BFS | Ваги 0 або 1 | O(V+E) |
| Dijkstra | Додатні ваги | O((V+E) log V) |
| Bellman-Ford | Від'ємні ваги (без від'ємних циклів) | O(V·E) |
| SPFA | Bellman-Ford з оптимізацією черги | ~O(E) у середньому |
| Floyd-Warshall | Усі пари вершин | O(V³) |
| Johnson | Усі пари, розріджений граф | O(V²·log V + V·E) |

### 14.1 Dijkstra

```csharp
int[] Dijkstra(int n, List<(int to, int w)>[] graph, int start) {
    var dist = new int[n];
    Array.Fill(dist, int.MaxValue);
    dist[start] = 0;
    var pq = new PriorityQueue<int, int>(); // node, dist
    pq.Enqueue(start, 0);
    while (pq.Count > 0) {
        int node = pq.Dequeue();
        // pq.TryDequeue(out int node, out int d); if (d > dist[node]) continue;
        foreach (var (next, w) in graph[node]) {
            int nd = dist[node] + w;
            if (nd < dist[next]) {
                dist[next] = nd;
                pq.Enqueue(next, nd);
            }
        }
    }
    return dist;
}
```

### 14.2 Bellman-Ford

`V-1` разів релаксуємо всі ребра. На `V-й` ітерації, якщо ще є оновлення — є від'ємний цикл.

### 14.3 Floyd-Warshall

Потрійний цикл: `for k, for i, for j: dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`.

---

## 15. Minimum Spanning Tree

### 15.1 Kruskal's

Сортуємо ребра за вагою, додаємо по черзі, якщо не створюємо цикл (через DSU). `O(E log E)`.

### 15.2 Prim's

Аналог Dijkstra, але замість найкоротшої відстані до start — мінімальна вага ребра від поточного MST до нової вершини. `O((V+E) log V)`.

---

## 16. Dynamic Programming

**Основний принцип:** знайти рекурентність + мемоізація (top-down) або табуляція (bottom-up).

### 16.1 1D DP

**Шаблон:** `dp[i]` — відповідь до індексу i.

**Приклади:** Climbing Stairs, House Robber, Maximum Subarray (Kadane), Decode Ways, Word Break.

```csharp
// House Robber
int Rob(int[] nums) {
    int prev2 = 0, prev1 = 0;
    foreach (int n in nums) {
        int curr = Math.Max(prev1, prev2 + n);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

### 16.2 2D DP (матричне)

**Шаблон:** `dp[i][j]` — відповідь для пари (i,j) або стану (i,j).

**Приклади:** Unique Paths, Min Path Sum, Edit Distance, Longest Common Subsequence.

```csharp
// Edit Distance (Levenshtein)
int MinDistance(string w1, string w2) {
    int m = w1.Length, n = w2.Length;
    var dp = new int[m + 1, n + 1];
    for (int i = 0; i <= m; i++) dp[i, 0] = i;
    for (int j = 0; j <= n; j++) dp[0, j] = j;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (w1[i-1] == w2[j-1]) dp[i,j] = dp[i-1,j-1];
            else dp[i,j] = 1 + Math.Min(dp[i-1,j-1], Math.Min(dp[i-1,j], dp[i,j-1]));
        }
    return dp[m, n];
}
```

### 16.3 Knapsack

**0/1 Knapsack:** кожен предмет береш або ні. `dp[i][w]` — макс. цінність з перших i предметів і ємності w.

**Unbounded:** предмети можна брати необмежено. Приклад: Coin Change.

**Варіації:** Partition Equal Subset Sum, Target Sum, Last Stone Weight II.

### 16.4 LIS / LCS

- **LIS (Longest Increasing Subsequence):** класичне `O(n²)` DP або `O(n log n)` з бінарним пошуком.
- **LCS (Longest Common Subsequence):** 2D DP.

```csharp
// LIS за O(n log n)
int LIS(int[] nums) {
    var tails = new List<int>();
    foreach (int n in nums) {
        int idx = tails.BinarySearch(n);
        if (idx < 0) idx = ~idx;
        if (idx == tails.Count) tails.Add(n);
        else tails[idx] = n;
    }
    return tails.Count;
}
```

### 16.5 Interval DP

**Шаблон:** `dp[i][j]` — відповідь на інтервалі [i,j]. Перебираємо інтервали за зростанням довжини.

**Приклади:** Matrix Chain Multiplication, Burst Balloons, Palindromic Substrings, Stone Game варіації.

### 16.6 Bitmask DP

**Коли:** стан містить підмножину (n ≤ 20). Бітова маска кодує, які елементи вже використані.

**Приклади:** Travelling Salesman, Assignment Problem, Count Ways to Distribute Candies.

```csharp
// TSP — O(2^n * n^2)
int Tsp(int[,] dist) {
    int n = dist.GetLength(0);
    var dp = new int[1 << n, n];
    // dp[mask, i] — мін. вартість відвідати множину mask, закінчуючи в i
    // base: dp[1, 0] = 0; решта MaxValue
    // transition: для кожної маски та її біта i, для кожного j не в масці — оновлюємо dp[mask | (1<<j), j]
    return 0; // скелет
}
```

### 16.7 Digit DP

**Коли:** рахувати числа в діапазоні [L, R] з певною властивістю на цифрах.

**Стан:** позиція, tight-bound прапор, інформація про попередні цифри.

### 16.8 Tree DP

DP на дереві з post-order обходом. Розглянуто в розділі Tree Traversal.

### 16.9 State Machine DP

Декілька станів у кожній позиції. Приклад: Best Time to Buy/Sell Stock з cooldown або fees.

---

## 17. Greedy

**Принцип:** на кожному кроці вибираємо локально оптимальний вибір. Треба доводити коректність (exchange argument).

### 17.1 Interval scheduling

**Задача:** максимум непересічних інтервалів. **Жадний вибір:** сортування за часом завершення, беремо перший, пропускаємо всі, що перетинаються.

### 17.2 Interval merging

Сортуємо за початком, об'єднуємо перекриття.

### 17.3 Huffman coding

Жадне об'єднання двох найменших частот через min-heap.

### 17.4 Task scheduling з дедлайнами

Сортуємо за дедлайнами + priority queue.

**Приклади:** Jump Game II, Gas Station, Assign Cookies, Candy, Non-overlapping Intervals.

---

## 18. Backtracking

**Шаблон:** будуємо рішення інкрементально, "відкочуємось" при невалідному стані.

```csharp
void Backtrack(стан поточний) {
    if (рішення) { Зберегти; return; }
    foreach (варіант vr) {
        якщо (valid(vr)) {
            застосувати(vr);
            Backtrack(оновлений стан);
            скасувати(vr); // backtrack
        }
    }
}
```

### 18.1 Permutations

```csharp
void Permute(List<int> nums, List<int> current, bool[] used, List<List<int>> result) {
    if (current.Count == nums.Count) {
        result.Add(new List<int>(current));
        return;
    }
    for (int i = 0; i < nums.Count; i++) {
        if (used[i]) continue;
        used[i] = true;
        current.Add(nums[i]);
        Permute(nums, current, used, result);
        current.RemoveAt(current.Count - 1);
        used[i] = false;
    }
}
```

### 18.2 Combinations / Subsets

Інклюд/екслюд кожного елемента.

### 18.3 Constraint Satisfaction

**Приклади:** N-Queens, Sudoku Solver, Word Search, Combination Sum.

### Оптимізації backtracking
- **Pruning:** ранній відсів невалідних гілок
- **Ordering:** розглядати "важливі" варіанти першими
- **Memoization:** якщо стан повторюється — перетворюємо на DP

---

## 19. Divide and Conquer

### 19.1 Merge Sort

`O(n log n)` стабільне сортування. Основа для Count of Smaller Numbers After Self, Reverse Pairs, Count Inversions.

### 19.2 Quickselect

Пошук k-го найменшого за середнє `O(n)`.

### 19.3 Binary search variants

### 19.4 Задачі на D&C

- **Median of Two Sorted Arrays:** D&C на partitions.
- **Skyline Problem:** D&C по будівлях.
- **Closest Pair of Points:** геометричний D&C за `O(n log n)`.

---

## 20. String algorithms

### 20.1 KMP (Knuth-Morris-Pratt)

Пошук підрядка за `O(n+m)` з префікс-функцією.

```csharp
int[] BuildLps(string pattern) {
    var lps = new int[pattern.Length];
    int len = 0;
    for (int i = 1; i < pattern.Length; ) {
        if (pattern[i] == pattern[len]) lps[i++] = ++len;
        else if (len > 0) len = lps[len - 1];
        else lps[i++] = 0;
    }
    return lps;
}
```

### 20.2 Z-algorithm

Для кожної позиції i — довжина найдовшого спільного префікса з s та суфікса, що починається в i. `O(n)`.

### 20.3 Rolling Hash (Rabin-Karp)

Хеш підрядка за `O(1)` після препроцесингу. Використовується для пошуку підрядка, Longest Duplicate Substring, порівняння підрядків.

### 20.4 Trie (префіксне дерево)

**Коли використовувати:**
- Word Search II
- Autocomplete / Prefix matching
- Replace Words
- Design Search Autocomplete
- Stream of Characters
- XOR-trie для задач Maximum XOR

```csharp
class Trie {
    class Node { public Dictionary<char, Node> Children = new(); public bool IsEnd; }
    Node root = new();
    public void Insert(string word) {
        var curr = root;
        foreach (char c in word) {
            if (!curr.Children.ContainsKey(c)) curr.Children[c] = new Node();
            curr = curr.Children[c];
        }
        curr.IsEnd = true;
    }
    public bool Search(string word) { /* подібно */ return true; }
}
```

### 20.5 Manacher's

Усі паліндромні підрядки за `O(n)`.

### 20.6 Suffix Array / Suffix Automaton

Просунуті структури для рядкових задач.

---

## 21. Bit Manipulation

### 21.1 Базові трюки

```csharp
x & 1                     // парність
x & (x - 1)               // скинути наймолодший біт (Brian Kernighan)
x & -x                    // ізолювати наймолодший біт (lowbit)
x ^ x == 0                // XOR з собою = 0
a ^ b ^ a == b            // XOR пар взаємно знищуються
(mask >> i) & 1           // перевірити i-й біт
mask | (1 << i)           // встановити i-й біт
mask & ~(1 << i)          // скинути i-й біт
mask ^ (1 << i)           // перемкнути i-й біт
```

### 21.2 Класичні задачі

- **Single Number** — XOR всіх елементів
- **Single Number II, III** — бітові лічильники
- **Count Set Bits** — Brian Kernighan's
- **Power of Two** — `x > 0 && (x & (x-1)) == 0`
- **Missing Number** — XOR діапазону

### 21.3 Bitmask як множина

Використовується в bitmask DP. Для `n ≤ 20` зручно зберігати підмножину в int.

### 21.4 Binary Exponentiation

`a^n mod m` за `O(log n)`. Узагальнюється на матричне експоненціювання.

```csharp
long Power(long a, long n, long mod) {
    long result = 1;
    a %= mod;
    while (n > 0) {
        if ((n & 1) == 1) result = result * a % mod;
        a = a * a % mod;
        n >>= 1;
    }
    return result;
}
```

---

## 22. Math & Number Theory

### 22.1 GCD / LCM (Euclidean)

```csharp
long Gcd(long a, long b) => b == 0 ? a : Gcd(b, a % b);
long Lcm(long a, long b) => a / Gcd(a, b) * b;
```

### 22.2 Sieve of Eratosthenes

Усі прості до N за `O(N log log N)`.

### 22.3 Modular arithmetic

- Фермата (для простого `p`): `a^(p-1) ≡ 1 (mod p)` → обернений: `a^(p-2)`
- Обернений елемент через розширений Евклідів алгоритм

### 22.4 Combinatorics

Precompute `fact[]`, `inv_fact[]` для `C(n,k)` mod p за `O(1)` запит.

### 22.5 Pigeonhole, parity, invariants

Роздуми, що часто спрощують задачі.

---

## 23. Advanced Data Structures

### 23.1 Segment Tree

Range queries + point/range updates за `O(log n)`.

**Застосування:** Range Sum / Min / Max Query, Count of Smaller Numbers, My Calendar I/II/III.

### 23.2 Fenwick Tree (BIT)

Простіший за Segment Tree, тільки для кумулятивних операцій. `O(log n)`.

```csharp
class BIT {
    int[] tree;
    public BIT(int n) => tree = new int[n + 1];
    public void Update(int i, int val) {
        for (; i < tree.Length; i += i & -i) tree[i] += val;
    }
    public int Query(int i) {
        int sum = 0;
        for (; i > 0; i -= i & -i) sum += tree[i];
        return sum;
    }
}
```

### 23.3 Sparse Table

`O(n log n)` preprocessing, `O(1)` range min/max (immutable).

### 23.4 LRU Cache (HashMap + Doubly Linked List)

Класичний design-patter на співбесідах.

### 23.5 LFU Cache

Доданий лічильник частот + додаткові структури.

### 23.6 Skip List

Ймовірнісна альтернатива збалансованим деревам.

### 23.7 Segment Tree з lazy propagation

Для діапазонних оновлень за `O(log n)`.

---

## 24. Special tricks

### 24.1 Kadane's algorithm

Максимальна сума підмасиву за `O(n)`.

```csharp
int MaxSubArray(int[] nums) {
    int max = nums[0], curr = nums[0];
    for (int i = 1; i < nums.Length; i++) {
        curr = Math.Max(nums[i], curr + nums[i]);
        max = Math.Max(max, curr);
    }
    return max;
}
```

### 24.2 Boyer-Moore Majority Vote

Знайти елемент, що з'являється > n/2 разів за `O(n)` час і `O(1)` пам'ять.

```csharp
int Majority(int[] nums) {
    int candidate = 0, count = 0;
    foreach (int n in nums) {
        if (count == 0) candidate = n;
        count += (n == candidate) ? 1 : -1;
    }
    return candidate;
}
```

### 24.3 Reservoir Sampling

Випадковий вибір K елементів зі стріму невідомого розміру.

### 24.4 Meet in the Middle

Ділимо вхід навпіл, обчислюємо обидві половини окремо, об'єднуємо. Використовується, коли `2^n` завеликий, а `2^(n/2)` нормальний.

### 24.5 Coordinate Compression

Заміняємо значення їхніми рангами у відсортованому порядку. Потрібно, коли значення великі, а кількість унікальних мала.

### 24.6 Matrix Exponentiation

Рекурентності Фібоначчі-подібні за `O(k³ log n)` замість `O(n)`.

### 24.7 Sqrt Decomposition

Розбиваємо масив на блоки розміру `√n`. Запити та оновлення за `O(√n)`.

### 24.8 Mo's Algorithm

Оффлайн-обробка запитів на діапазонах. Сортуємо запити і обробляємо за `O((n+q)·√n)`.

### 24.9 Two-pass техніка

Часто допомагає уникнути складних структур: один прохід зліва направо, другий — справа наліво. Приклад: Product of Array Except Self, Candy, Trapping Rain Water.

### 24.10 Cyclic Sort

Коли масив містить числа 1..n — можна сортувати за `O(n)` без додаткової пам'яті, просто ставлячи кожне на своє місце. Використовується для Find Missing / Duplicate Number задач.

---

## Як обрати патерн — швидкий чек-лист

| Ознака задачі | Ймовірний патерн |
|---------------|------------------|
| Відсортований масив, шукати пару/трійку | Two Pointers |
| Підмасив/підрядок з умовою | Sliding Window |
| "Сума на діапазоні" багато разів | Prefix Sum |
| Впорядкована структура + пошук | Binary Search |
| "Чи існує таке значення…" | Hash Set |
| "Скільки разів з'являється…" | Hash Map (frequency) |
| Парність дужок, next greater | Stack |
| Max/min у ковзному вікні | Monotonic Deque |
| Top K, k-th element | Heap |
| Linked list: середина, цикл | Slow/Fast |
| Тree: рівень за рівнем | BFS |
| Tree: сума/глибина/діаметр | DFS + post-order |
| Граф без ваг: найкоротший шлях | BFS |
| Граф з додатніми вагами | Dijkstra |
| Граф з від'ємними вагами | Bellman-Ford |
| DAG + порядок виконання | Topological Sort |
| Зв'язні компоненти, динамічне об'єднання | Union-Find |
| Оптимізація з перекриттям підзадач | DP |
| "Максимум/мінімум на всіх розбиттях" | DP (часто 2D або interval) |
| Перебір варіантів зі скасуванням | Backtracking |
| n ≤ 20, підмножини | Bitmask DP |
| Рядок: пошук, повторення | KMP / Rolling Hash / Trie |
| Range queries + updates | Segment Tree / BIT |

---

## Що вивчати в якому порядку (якщо з нуля)

1. **Фундамент:** Two Pointers, Sliding Window, Prefix Sum, Hash Map, Binary Search
2. **Структури:** Stack, Queue, Heap, Linked List patterns
3. **Дерева:** DFS, BFS, Tree DP
4. **Графи:** DFS/BFS, Topological Sort, Union-Find, Dijkstra
5. **DP:** 1D → 2D → Knapsack → LIS/LCS → Interval → Bitmask
6. **Жадні + Backtracking** (паралельно з DP)
7. **Рядкові:** Trie, KMP, Rolling Hash
8. **Просунуте:** Segment Tree / BIT, Monotonic Stack/Deque, спецтрюки
9. **Рідкісне (лише для топ-компаній):** Manacher's, Suffix Array, Mo's, Heavy-Light Decomposition

---

## Корисні джерела

- **Книга:** "Competitive Programming 4" (Halim & Halim) — вичерпно для змагань
- **Книга:** "Introduction to Algorithms" (CLRS) — академічно
- **Книга:** "Algorithm Design Manual" (Skiena) — практично
- **Сайт:** cp-algorithms.com — лаконічні пояснення з кодом
- **Сайт:** leetcode.com + neetcode.io — структурована практика
- **Sайт:** codeforces.com — змагальний спорт, найкраща практика для складних задач

# ☕ Java Collections: Enterprise Cheatsheet

Quick reference for choosing, using, and optimizing collections in production.

---

## Decision Tree: What Collection to Use?

```
NEED A LIST?
    ├─ Random access critical? 
    │   └─ YES → ArrayList
    ├─ Frequent inserts/deletes at ends?
    │   └─ YES → LinkedList or ArrayDeque
    ├─ Insertion order matters + concurrency?
    │   └─ YES → CopyOnWriteArrayList
    └─ Always sorted?
        └─ YES → Collections.sort(list) on ArrayList

NEED A SET?
    ├─ Fastest, unordered?
    │   └─ YES → HashSet
    ├─ Must be sorted?
    │   └─ YES → TreeSet
    ├─ Insertion order matters?
    │   └─ YES → LinkedHashSet
    ├─ Only enums?
    │   └─ YES → EnumSet
    └─ Thread-safe sorted?
        └─ YES → ConcurrentSkipListSet

NEED A MAP?
    ├─ Fast lookups, unordered?
    │   └─ YES → HashMap
    ├─ Sorted by key?
    │   └─ YES → TreeMap
    ├─ Insertion order matters?
    │   └─ YES → LinkedHashMap
    ├─ LRU cache?
    │   └─ YES → LinkedHashMap(accessOrder=true)
    ├─ Thread-safe?
    │   └─ YES → ConcurrentHashMap
    ├─ Weak references?
    │   └─ YES → WeakHashMap
    └─ Only enums as keys?
        └─ YES → EnumMap

NEED A QUEUE?
    ├─ FIFO, double-ended?
    │   └─ YES → ArrayDeque
    ├─ Priority-based?
    │   └─ YES → PriorityQueue
    ├─ Thread-safe blocking?
    │   └─ YES → ArrayBlockingQueue
    ├─ Unbounded, thread-safe?
    │   └─ YES → ConcurrentLinkedQueue
    └─ Task scheduling?
        └─ YES → Use Executor framework
```

---

## Performance Quick Reference

| Operation | ArrayList | LinkedList | HashSet | TreeSet | HashMap | TreeMap |
|-----------|-----------|-----------|---------|---------|---------|---------|
| **get/access** | **O(1)** | O(n) | — | — | **O(1)** | O(log n) |
| **add/put** | O(n) | O(1) | **O(1)** | O(log n) | **O(1)** | O(log n) |
| **remove/delete** | O(n) | O(1) | **O(1)** | O(log n) | **O(1)** | O(log n) |
| **contains** | O(n) | O(n) | **O(1)** | O(log n) | **O(1)** | O(log n) |
| **iteration** | **O(n)** | **O(n)** | O(n+cap) | **O(n)** | O(n+cap) | **O(n)** |

---

## Critical Gotchas & Anti-Patterns

### ❌ LinkedList for Random Access (O(n²) trap)

```java
// BAD: Each get(i) traverses from head → O(n²)
for (int i = 0; i < list.size(); i++) {
  process(list.get(i));
}

// GOOD: Uses iterator → O(n)
for (String item : list) {
  process(item);
}
```

### ❌ Collections.synchronizedMap() (Whole-map lock)

```java
// BAD: Serial execution, contention bottleneck
Map<K,V> m = Collections.synchronizedMap(new HashMap<>());

// GOOD: Segment-level locking, parallel access
Map<K,V> m = new ConcurrentHashMap<>();
m.compute(k, (key, v) -> v + 1);
```

### ❌ Inconsistent Comparator in TreeSet

```java
// BAD: Breaks TreeSet invariant → duplicates!
TreeSet<Integer> s = new TreeSet<>(
  (a, b) -> (a % 10) - (b % 10)
);
s.add(5); s.add(15); // Both hash to bucket 5, causes duplicates!

// GOOD: Comparator matches equals() contract
Comparator<User> c = 
  Comparator.comparing(User::getName)
    .thenComparing(User::getId);
```

### ❌ Iterating While Removing

```java
// BAD: ConcurrentModificationException
for (String s : list) {
  if (s.startsWith("x")) list.remove(s);
}

// GOOD: Use iterator.remove()
for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
  if (it.next().startsWith("x")) it.remove();
}

// GOOD: Use streams
list.removeIf(s -> s.startsWith("x"));
```

### ❌ PriorityQueue Iteration Order

```java
// BAD: Items NOT sorted!
System.out.println(pq); // Unpredictable order

// GOOD: Use poll() to extract in order
while (!pq.isEmpty()) {
  System.out.println(pq.poll()); // Sorted order
}
```

### ❌ No Null in TreeSet/TreeMap

```java
// BAD: NullPointerException
TreeSet<String> s = new TreeSet<>();
s.add(null); // NPE during comparison

// GOOD: HashSet allows single null
HashSet<String> s = new HashSet<>();
s.add(null); // OK
```

---

## Thread Safety Patterns

### ConcurrentHashMap (Segment/Bucket-level locking)

**When:** High-contention shared cache/map

```java
Map<K,V> m = new ConcurrentHashMap<>();

// Atomic operations
m.putIfAbsent(k, v);
m.compute(k, (key, old) -> old == null ? 1 : old + 1);
m.getOrDefault(k, defaultValue);
```

**Key Points:**
- Segment/bucket-level locking (JDK 8+: node-level with CAS)
- No null keys or values allowed (by design)
- High throughput under contention
- Weakly consistent iterator

### CopyOnWriteArrayList (Read-heavy, write-rare)

**When:** Listeners, configuration snapshots, mostly reads

```java
List<String> listeners = new CopyOnWriteArrayList<>();

// Slow: full array copy O(n)
listeners.add(handler);

// Fast: no lock O(1)
for (String h : listeners) { ... }
```

**Key Points:**
- Every write copies entire array
- Only use if writes are rare
- Perfect for listener patterns
- Iteration never sees concurrent modifications

### BlockingQueue (Producer-consumer with backpressure)

**When:** Rate-limiting, task queues, producer-consumer pattern

```java
BlockingQueue<Task> q = new ArrayBlockingQueue<>(100);

// Producer (blocks if full → backpressure)
q.put(task);

// Consumer (blocks if empty)
Task t = q.take();
```

**Key Points:**
- Built-in blocking on full/empty
- Natural backpressure
- Thread-safe by design

### ReadWriteLock Pattern (Many reads, rare writes)

```java
ReadWriteLock lock = new ReentrantReadWriteLock();
Map<K,V> map = new HashMap<>();

// Read (parallel, concurrent)
lock.readLock().lock();
try { 
  return map.get(k); 
} finally { 
  lock.readLock().unlock(); 
}

// Write (exclusive)
lock.writeLock().lock();
try { 
  map.put(k, v); 
} finally { 
  lock.writeLock().unlock(); 
}
```

### Fail-Fast vs Weakly Consistent

| Collection | Iterator Behavior |
|------------|-------------------|
| ArrayList, HashMap | **Fail-fast:** Throws `ConcurrentModificationException` |
| ConcurrentHashMap | **Weakly consistent:** Reflects concurrent modifications |
| CopyOnWriteArrayList | **Snapshot:** Immutable view at iteration time |

### Synchronized Wrappers (Legacy, avoid)

```java
// BAD: Whole-map lock, slow
Map<K,V> m = Collections.synchronizedMap(new HashMap<>());

// GOOD: Use concurrent collections
Map<K,V> m = new ConcurrentHashMap<>();
```

Only use synchronized wrappers if you need null keys or must wrap an existing collection.

---

## Enterprise Patterns & Recipes

### LRU Cache (LinkedHashMap with accessOrder)

```java
class LRUCache<K,V> extends LinkedHashMap<K,V> {
  int capacity;
  
  public LRUCache(int cap) {
    super(cap, 0.75f, true); // accessOrder = true
    this.capacity = cap;
  }
  
  @Override
  protected boolean removeEldestEntry(Map.Entry eldest) {
    return size() > capacity;
  }
}

LRUCache<String, String> cache = new LRUCache<>(100);
cache.get(key);           // Updates position (most recent)
cache.put(key, value);    // Evicts LRU if full
```

### Batch Processing & Pre-allocation

```java
List<String> items = new ArrayList<>();

// Pre-allocate if size known → avoids resizing
List<String> items = new ArrayList<>(1000);

// Bulk operations
items.addAll(sourceList);
items.retainAll(keepSet);
items.removeAll(removeSet);

// Single-pass transformation
List<String> upper = items.stream()
  .map(String::toUpperCase)
  .collect(Collectors.toList());
```

### Defensive Copying (Encapsulation)

```java
public class UserCache {
  private List<User> users;
  
  // BAD: Exposes internals
  public List<User> getUsers() { 
    return users; 
  }
  
  // GOOD: Defensive copy
  public List<User> getUsers() { 
    return new ArrayList<>(users); 
  }
  
  // BEST: Unmodifiable view
  public List<User> getUsers() { 
    return Collections.unmodifiableList(users); 
  }
}
```

### Grouping & Partitioning (Streams)

```java
List<Order> orders = ...;

// Group by status
Map<String, List<Order>> byStatus = orders.stream()
  .collect(Collectors.groupingBy(Order::getStatus));

// Partition: true/false split
Map<Boolean, List<Order>> shipped = orders.stream()
  .collect(Collectors.partitioningBy(Order::isShipped));
```

### Sorted Collection Patterns

```java
// Always-sorted list
List<String> sorted = new ArrayList<>(Arrays.asList("c", "a", "b"));
Collections.sort(sorted);

// Sorted set (no duplicates)
Set<String> unique = new TreeSet<>(unsortedList);

// Multi-key sort (last name → first name → age)
Collections.sort(users, 
  Comparator.comparing(User::getLastName)
    .thenComparing(User::getFirstName)
    .thenComparingInt(User::getAge)
);
```

### Stream Collectors Recipes

```java
// To Map
Map<Integer, String> m = items.stream()
  .collect(Collectors.toMap(Item::getId, Item::getName));

// Count
long count = items.stream()
  .collect(Collectors.counting());

// Join strings
String csv = items.stream()
  .map(Item::toString)
  .collect(Collectors.joining(","));

// Sorted set
TreeSet<String> sorted = items.stream()
  .collect(Collectors.toCollection(TreeSet::new));

// Min/Max by comparator
Optional<Item> smallest = items.stream()
  .collect(Collectors.minBy(Comparator.comparing(Item::getPrice)));
```

---

## Capacity & Memory Optimization

### Resizing Costs

| Collection | Strategy | Cost |
|------------|----------|------|
| **ArrayList** | Resize at 1.5× when exceeded | O(n) copy of all elements |
| **HashMap** | Resize at load factor 0.75 | Rehash all entries, capacity 2× |
| **LinkedHashMap** | Same as HashMap | + doubly-linked overhead |

### Space Usage & Optimization Tips

| Collection | Space | Overhead | Tip |
|-----------|-------|----------|-----|
| ArrayList | O(n) | ~12% over capacity | Pre-allocate: `new ArrayList<>(expectedSize)` |
| LinkedList | O(n) | 2 pointers/node + header | Only for queue/deque, not random access |
| HashSet | O(n) | Bucket array + entries | Load factor 0.75 → capacity ≈ 1.33× size |
| TreeSet | O(n) | RB-tree pointers + color | No resizing, use for sorted access |
| HashMap | O(n) | Bucket array + entries | Capacity = size / 0.75 |
| LinkedHashMap | O(n) | HashMap + 2 ptrs/node | Use only for LRU; avoid if simple order needed |

---

## Special Collections

### EnumSet & EnumMap (Enums only, O(1) array-backed)

```java
// Super fast O(1) operations
EnumSet<Color> primary = EnumSet.of(RED, BLUE, YELLOW);
EnumMap<Color, String> names = new EnumMap<>(Color.class);

// vs slower HashSet
Set<Color> s = new HashSet<>();
```

### WeakHashMap (GC-friendly, soft caches)

```java
Map<Resource, Metadata> meta = new WeakHashMap<>();
meta.put(resource, data); 
// Entry automatically removed when resource is GC'd
```

### IdentityHashMap (Reference equality, not .equals())

```java
Map<Object, String> m = new IdentityHashMap<>();
Integer a = 1; Integer b = 1;
m.put(a, "x"); m.put(b, "y");
// Two entries (different objects, same primitive value)
```

### ConcurrentSkipListMap/Set (Lock-free sorted)

```java
NavigableMap<String, String> m = new ConcurrentSkipListMap<>();
// Thread-safe sorted, better concurrency than TreeMap
m.subMap("a", "z"); // Lazy range view
```

### ArrayDeque (Circular buffer, both ends)

```java
// Stack (LIFO)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1); 
stack.pop();

// Queue (FIFO)
Deque<Integer> queue = new ArrayDeque<>();
queue.addLast(1); 
queue.removeFirst();
```

### PriorityQueue (Binary min-heap)

```java
// Min-heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max-heap (reverse)
PriorityQueue<Integer> maxHeap = 
  new PriorityQueue<>((a, b) -> b - a);

// Extract in order
while (!maxHeap.isEmpty()) {
  System.out.println(maxHeap.poll());
}
```

---

## Null Handling Across Collections

| Collection | Null Keys | Null Values | Notes |
|-----------|-----------|------------|-------|
| HashMap | ✓ 1 | ✓ many | Single null key allowed |
| HashSet | ✓ 1 | — | Single null element allowed |
| TreeMap | ✗ | ✗ | NullPointerException on compare |
| TreeSet | ✗ | — | NullPointerException on compare |
| ConcurrentHashMap | ✗ | ✗ | By design: distinguish absence from null |
| ArrayList | — | ✓ | Nulls allowed in lists |
| PriorityQueue | — | ✗ | Heap needs comparisons |
| LinkedHashMap | ✓ 1 | ✓ many | Single null key allowed |

---

## Streams Integration

### Complete Collector Recipes

```java
List<String> items = Arrays.asList("apple", "banana", "apricot");

// Filter + map + collect
List<String> result = items.stream()
  .filter(s -> s.startsWith("a"))
  .map(String::toUpperCase)
  .collect(Collectors.toList());

// Group by attribute
Map<Integer, List<String>> byLen = items.stream()
  .collect(Collectors.groupingBy(String::length));

// To Map (key-value pair extraction)
Map<String, Integer> lengths = items.stream()
  .collect(Collectors.toMap(s -> s, String::length));

// Partition (true/false split)
Map<Boolean, List<String>> partition = items.stream()
  .collect(Collectors.partitioningBy(s -> s.length() > 5));

// Sorted set
TreeSet<String> sorted = items.stream()
  .collect(Collectors.toCollection(TreeSet::new));

// Join strings
String csv = items.stream()
  .collect(Collectors.joining(", "));

// Count elements
long count = items.stream()
  .filter(s -> s.startsWith("a"))
  .count();

// Find first (Optional)
Optional<String> first = items.stream()
  .filter(s -> s.startsWith("b"))
  .findFirst();

// Find any (Optional, parallel-friendly)
Optional<String> any = items.stream()
  .filter(s -> s.length() > 5)
  .findAny();

// Reduce (fold/aggregate)
String concatenated = items.stream()
  .reduce("", (a, b) -> a + b);

// Custom collector
String uppercaseCommaJoined = items.stream()
  .map(String::toUpperCase)
  .collect(Collectors.joining(","));
```

---

## Interview Q&A (Canonical Questions)

### Q1: HashMap vs TreeMap vs LinkedHashMap?

| Aspect | HashMap | TreeMap | LinkedHashMap |
|--------|---------|---------|---------------|
| **Ordering** | None | By key (sorted) | Insertion order |
| **get/put/remove** | O(1) | O(log n) | O(1) |
| **Iteration** | O(n + capacity) | O(n) sorted | O(n) ordered |
| **Null key** | 1 ✓ | ✗ | 1 ✓ |
| **Use case** | Cache/lookup | Sorted/range views | JSON serialization |

### Q2: ArrayList vs LinkedList?

**ArrayList:** Choose for random access via index (`get(i)`)
- O(1) random access
- Slow insertion in middle O(n)
- Cache-friendly, contiguous memory

**LinkedList:** Choose for queue/deque operations
- O(1) insertion/deletion at ends
- O(n) random access (NO!)
- High memory overhead (2 pointers per node)

⚠️ **Never loop with `list.get(i)` on LinkedList** — it's O(n²)

### Q3: Why power-of-2 capacity in HashMap?

Index calculation: `hash(key) & (capacity - 1)` instead of modulo

```
& 0x1F (31 = 0b11111)  ← Fast bitwise AND
% 32                   ← Slower division
```

Power-of-2 also ensures even distribution if hash function spreads high bits.

### Q4: Fail-fast iterator: what & why?

**What:** Iterator throws `ConcurrentModificationException` if collection changes structurally during iteration.

**Why:** Detects accidental concurrent modification bugs (NOT a synchronization mechanism).

**Fix:**
```java
// Use iterator.remove()
for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
  if (it.next().startsWith("x")) it.remove();
}

// Or use CopyOnWriteArrayList
List<String> safe = new CopyOnWriteArrayList<>(original);
```

### Q5: ConcurrentHashMap vs Collections.synchronizedMap()?

**ConcurrentHashMap:**
- Segment/bucket-level locking
- High concurrency, parallel access
- No nulls allowed (by design)
- Weakly consistent iterator

**Collections.synchronizedMap():**
- Whole-map lock
- Serial execution, contention bottleneck
- Allows single null key
- Fail-fast iterator

**Use ConcurrentHashMap in production. Always.**

### Q6: TreeSet comparator consistency rule?

**Rule:** If `comparator.compare(a, b) == 0`, then `a.equals(b)` MUST be true.

Otherwise TreeSet treats them as duplicates and violates the set invariant:

```java
// BAD: Breaks invariant
TreeSet<Integer> set = new TreeSet<>(
  (a, b) -> (a % 10) - (b % 10)
);
set.add(5);  // Added
set.add(15); // comparator returns 0, but !5.equals(15)
// Set now has "duplicates" → contract violated!

// GOOD: Consistent
Comparator<User> c = 
  Comparator.comparing(User::getName)
    .thenComparing(User::getId);
```

---

## Summary: Collection Selection Flowchart

```
START
  │
  ├─ Need ORDERED elements?
  │   ├─ YES → List (ArrayList/LinkedList/CopyOnWriteArrayList)
  │   │         └─ Random access critical?
  │   │             ├─ YES → ArrayList
  │   │             └─ NO  → Frequent inserts at ends?
  │   │                       ├─ YES → LinkedList or ArrayDeque
  │   │                       └─ NO  → ArrayList
  │   │
  │   └─ NO → Need UNIQUE elements?
  │       ├─ YES → Set (HashSet/TreeSet/LinkedHashSet)
  │       │         └─ Must be SORTED?
  │       │             ├─ YES → TreeSet
  │       │             ├─ NO  → Insertion order matters?
  │       │             │         ├─ YES → LinkedHashSet
  │       │             │         └─ NO  → HashSet
  │       │
  │       └─ NO → Need KEY-VALUE pairs?
  │           └─ YES → Map (HashMap/TreeMap/LinkedHashMap)
  │                    └─ Must be SORTED by key?
  │                        ├─ YES → TreeMap
  │                        ├─ NO  → Insertion order matters?
  │                        │        ├─ YES → LinkedHashMap
  │                        │        └─ NO  → HashMap
  │
  └─ Need FIFO/Priority order?
      └─ YES → Queue (PriorityQueue/ArrayDeque/BlockingQueue)
               └─ Priority-based?
                   ├─ YES → PriorityQueue
                   ├─ NO  → FIFO + blocking?
                   │        ├─ YES → BlockingQueue
                   │        └─ NO  → ArrayDeque
```

---

## Cheat Sheet: Copy-Paste Ready

### Default choices for most scenarios:

```java
// List: usually ArrayList
List<String> items = new ArrayList<>();

// Set: usually HashSet
Set<String> unique = new HashSet<>();

// Map: usually HashMap
Map<String, Integer> cache = new HashMap<>();

// Sorted: usually TreeMap
Map<String, Integer> sorted = new TreeMap<>();

// Queue: usually ArrayDeque
Deque<Integer> stack = new ArrayDeque<>();

// Thread-safe map
Map<String, Integer> concurrent = new ConcurrentHashMap<>();

// Thread-safe list
List<String> safeList = new CopyOnWriteArrayList<>();

// LRU Cache
class LRU<K,V> extends LinkedHashMap<K,V> {
  int cap;
  public LRU(int c) { super(c, 0.75f, true); cap = c; }
  protected boolean removeEldestEntry(Map.Entry e) {
    return size() > cap;
  }
}
```

---

**Last Updated:** 2025  
**Purpose:** Production-grade reference for Java Collections Framework  
**Audience:** Enterprise developers, interviewees, architects

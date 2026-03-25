# 🎯 Java Collections Framework - Complete Enterprise Guide (Mermaid Edition)

## 📋 Table of Contents
1. Collections Hierarchy Overview
2. Memory & Internal Implementation
3. Decision Tree
4. List Implementations
5. Set Implementations
6. Map Implementations
7. Queue Implementations
8. HashCode & Equals
9. HashMap Internals
10. Garbage Collection

---

## 1️⃣ Collections Hierarchy Overview

```mermaid
flowchart TD
    Iterable --> Collection

    Collection --> List
    Collection --> Set
    Collection --> Queue
    Collection --> Map

    List --> ArrayList
    List --> LinkedList
    List --> Vector

    Set --> HashSet
    Set --> LinkedHashSet
    Set --> TreeSet

    Queue --> PriorityQueue
    Queue --> Deque
    Deque --> ArrayDeque

    Map --> HashMap
    Map --> LinkedHashMap
    Map --> TreeMap
    Map --> Hashtable
```

---

## 2️⃣ Memory & Internal Implementation

```mermaid
flowchart LR
    subgraph STACK
        A[list ref]
        B[map ref]
    end

    subgraph HEAP
        C[ArrayList Object]
        D[elementData Array]
        E[String Object]
    end

    A --> C
    C --> D
    D --> E
```

**Key Concepts:**
- Stack holds references
- Heap holds actual objects
- ArrayList uses dynamic arrays

---

## 3️⃣ Decision Tree

```mermaid
flowchart TD
    Start --> KV{Need Key-Value?}

    KV -->|Yes| MapChoice{Need Sorting?}
    KV -->|No| Unique{Need Unique Elements?}

    MapChoice -->|Yes| TreeMap
    MapChoice -->|No| HashMap

    Unique -->|Yes| SetChoice{Need Order?}
    Unique -->|No| ListChoice{Frequent Insert/Delete?}

    SetChoice -->|Yes| LinkedHashSet
    SetChoice -->|No| HashSet

    ListChoice -->|Yes| LinkedList
    ListChoice -->|No| ArrayList
```

---

## 4️⃣ ArrayList

```mermaid
graph TD
    A[ArrayList] --> B[elementData Array]
    B --> C["A"]
    B --> D["B"]
    B --> E["C"]
```

- Backed by dynamic array
- Fast random access O(1)
- Slow insert in middle O(n)

---

## 5️⃣ LinkedList

```mermaid
graph LR
    A["null"] <-->|prev| B["A"] <-->|next|
    B <-->|prev| C["B"] <-->|next|
    C <-->|prev| D["C"] <-->|next|
    D --> E["null"]
```

- Doubly linked list
- Fast insert/delete O(1)
- Slow access O(n)

---

## 6️⃣ HashSet

- Uses HashMap internally
- No duplicates
- O(1) operations

---

## 7️⃣ TreeSet

```mermaid
graph TD
    A[Bob] --> B[Alice]
    A --> C[Dan]
```

- Red-Black Tree
- Sorted order
- O(log n)

---

## 8️⃣ HashMap

```mermaid
flowchart TD
    Key --> Hash
    Hash --> Bucket
    Bucket --> Value
```

- Key-value storage
- O(1) average

---

## 9️⃣ Collision Handling

```mermaid
graph TD
    Bucket --> A["Alice"]
    A --> B["Bob"]
    B --> C["Charlie"]
```

- Uses chaining
- Converts to tree after threshold

---

## 🔟 Resizing

```mermaid
flowchart LR
    A[Capacity 16] --> B{Threshold Exceeded}
    B --> C[Resize to 32]
    C --> D[Rehash]
```

---

## 1️⃣1️⃣ PriorityQueue

```mermaid
graph TD
    A[1] --> B[3]
    A --> C[2]
    B --> D[7]
    B --> E[4]
```

- Min Heap
- O(log n)

---

## 1️⃣2️⃣ ArrayDeque

```mermaid
flowchart LR
    A --> B --> C --> D --> A
```

- Circular buffer
- Fast operations both ends

---

## 1️⃣3️⃣ HashCode & Equals

```mermaid
flowchart TD
    A[Object] --> B[hashCode]
    B --> C[Bucket]
    C --> D[equals]
    D --> E[Result]
```

**Rules:**
- Equal objects must have same hashCode
- Always override both

---

## 1️⃣4️⃣ Garbage Collection

```mermaid
flowchart TD
    A[Object Created] --> B[Referenced]
    B --> C{Reachable?}
    C -->|Yes| Alive
    C -->|No| GC
    GC --> Freed
```

---

## 🚀 Summary

| Type | Best Use |
|------|---------|
| ArrayList | Fast read |
| LinkedList | Frequent insert/delete |
| HashSet | Unique fast lookup |
| TreeSet | Sorted data |
| HashMap | Fast key-value |
| TreeMap | Sorted map |
| PriorityQueue | Priority processing |
| ArrayDeque | Stack/Queue |

---

## 🔥 Final Tip

> Default choice:  
- List → ArrayList  
- Set → HashSet  
- Map → HashMap  

Switch only when needed 🚀

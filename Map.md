
# Java Map


---

## Level 1: Absolute Basics

### 1. What is a Map in Java?
A `Map` is a data structure that stores data as **key–value pairs**, where keys are unique and each key maps to exactly one value.

### 2. Why does Map not extend Collection?
`Collection` represents a group of individual elements, while `Map` represents key–value associations. Operations like `add(E)` don’t conceptually apply to a Map.

### 3. Duplicate keys / values?
- Duplicate keys: Not allowed  
- Duplicate values: Allowed


### 4. What happens if same key is inserted again?
- Old value is **replaced**
- `put()` returns the **previous value**

### 5. Common Map implementations
| Map | Use case |
|----|----|
| HashMap | Fast lookup, no order |
| LinkedHashMap | Order-sensitive or LRU cache |
| TreeMap | Sorted keys |
| Hashtable | Legacy synchronized map* |
| ConcurrentHashMap | High-concurrency access |
> *A Map implementation from early Java (pre-Collections framework) that provides thread safety by synchronizing the entire map.
---


---

## Level 2: HashMap Fundamentals

### 6. How does HashMap store data?
Internally uses:
- **Array of buckets**
- Each bucket contains:   Each bucket stores one or more key-value pairs (Node<K,V> entries).
  - Linked list (Java 7) 
  - Linked list  (default) or Red-Black Tree (Java 8+)   (when collision chain grows large)
```
// LinkedList bucket:
table[]
+----+----+----+----+
| 0  | 1  | 2  | 3  |
+----+----+----+----+
             |
             v
         Node(hash,k,v)
               |
         Node(hash,k,v)
               |
         Node(hash,k,v)
```
```
//Treeified bucket:
        (k,v)
       /     \
   (k,v)     (k,v)

```
### 7. What is a bucket?
A bucket is an **array index** where entries with same hash index are stored.
> ***Hashing & Bucket Indexing***\
> The key’s hashCode() method is invoked.\
> The hash is spread (JDK 8+) to reduce collisions: ```h = hashCode() ^ (hashCode() >>> 16)```\
> The final bucket index is calculated using: ```index = (n - 1) & h```\
> where `n` is the current bucket array capacity.

### 8. Role of hashCode() and equals()
- `hashCode()` → participate to decides bucket
- `equals()` → resolves collisions inside bucket

### 9. Internal steps of put(k, v)
1. Compute `hash(key)`
2. Calculate index: `(n - 1) & hash`
3. Check bucket
4. If key exists → replace value
5. Else → insert new node
6. Resize if threshold exceeded

### 10. What and Why hash spreading?
**What:**
- Hash spreading in HashMap is the XOR operation where the high bits of the hashCode are shifted and mixed into the low bits before bucket indexing or Hash spreading is the operation where Java takes the original hashCode() and MIXES its high bits into low bits using bit operations.\
**Why?**, To:
- Reduce collisions
- Improve distribution
- Protect against poor hashCode implementations
- Hash spreading exists because HashMap uses only the low bits of the hash to compute the bucket index; spreading mixes high-entropy bits into low bits to reduce collisions and maintain O(1) performance.
  -  Bucket index is computed as:
    -  ```java index = (n - 1) & hash ```
    -  n = capacity (power of 2)
    -  Only lower bits of hash are used
    -  If low bits are poorly distributed → massive collisions
 
### Timeline of what happens in put()
1. Step 1: Get raw hash
   ```java
       int h = key.hashCode();
          // This is NOT spreading.
          // This is just whatever the key gives you (often bad).
   ```
  2. Step 2: 🔥 Hash spreading (THIS is the answer)
     ```java
        int spread = h ^ (h >>> 16);
         // ✔ High 16 bits are mixed into low 16 bits
         // ✔ This is called hash spreading
    ```
3. Step 3: Bucket index calculation
   ```java
      index = (n - 1) & spread; 
       // ❌ This is NOT hash spreading
       // ✔  This is just bucket selection
    ```
> “HashMap uses hashCode and modulo to find bucket” ----> ***BIG NO***
#### One-line mental model 🧠
> hashCode() gives the number → hash spreading mixes the bits → index picks the bucket\
> HashMap uses an array of buckets indexed by spread hash values, resolves collisions via linked lists or red-black trees, resizes based on load factor, and provides near constant-time performance for well-distributed keys.

### Null Value in HashMap
> At most one null key  :  Stored at bucket index 0  : Multiple null values allowed
---

## Level 3: Capacity, Load Factor, Resizing

### 11. Initial capacity & load factor
Capacity is bucket count; load factor controls resize threshold.

### 12. Defaults
Capacity = 16, Load factor = 0.75

### 13. When resize happens?
When size exceeds capacity * load factor.

### 14. Why power of 2?
Enables fast index calculation using bitwise AND.

### 15. Resize cost
O(n), expensive due to rehashing.

### 16. Improper sizing issues
Frequent resizing, GC pressure, latency spikes.

### On resize:
- Capacity doubles
- Entries are redistributed using:
- new index  =    oldIndex OR (oldIndex + oldCapacity)
- No need to recompute full hash — optimization in JDK 8+
### Why HashMap (JDK 8+) does NOT recompute full hash during resize
- Key idea (one sentence) : Because capacity is always a power of 2, doubling the table lets HashMap decide a new bucket using one extra bit, not a new hash calculation.
-  The Math Behind It :
  - index = (n - 1) & hash        ( n = capacity , hash = spread hash of key )
  - When resizing:
    - oldCapacity = n
    - newCapacity = 2n
    - What Actually Changes? Only one bit of the index calculation changes — the bit corresponding to oldCapacity. So for every existing entry, only two outcomes are possible:
      - newIndex = oldIndex OR   newIndex = oldIndex + oldCapacity
      - Before resize
        ```bash
        capacity = 16  -> binary 10000
        mask     = 15  -> binary 01111
        index = hash & 01111
        ```
      - after resize:
        ```bash
        capacity = 32  -> binary 100000
        mask     = 31  -> binary 11111
        index = hash & 11111
        ```
      - Key Observation :
        - Only the 5th bit (value 16) is newly considered.
        - So HashMap checks:
          ```bash
          (hash & oldCapacity) == 0 ?
          ✅ 0 → stays in same bucket index
          ❌ 1 → moves to index + oldCapacity
          ```
        - Inside HashMap:
          ```java
          if ((hash & oldCapacity) == 0) {
              newIndex = oldIndex;
          } else {
              newIndex = oldIndex + oldCapacity;  
          }
          ```
          >This is why resize is O(n) but cheap per entry.
          ---
         | Approach             | Cost                    |
         | -------------------- | ----------------------- |
         | Recompute hashCode() | Expensive, user-defined |
         | Modulo (%)           | Slow                    |
         | Bit mask (&)         | Very fast               |
         | One-bit check        | Extremely fast          |

---

## Level 4: Collision Handling

### 17. What is collision?
Different keys mapping to the same bucket.

### 18. Before Java 8
Linked list, worst case O(n).

### 19. Java 8+
Treeifies bucket into Red-Black Tree when threshold exceeded.

### 20. Tree conversion thresholds
Treeify at 8, untreeify at 6.

### 21. Why Red-Black Tree?
Better insertion/deletion balance than AVL.

### 22. Time complexity
Best/Average O(1), Worst O(log n) after Java 8.

---

## Level 5: equals & hashCode Contract

### 23. Contract
Equal objects must have same hashCode.

### 24. Common mistakes
Mutable keys or overriding equals only.

### 25. Why keys should be immutable?
Ensures consistent hashCode.

### 26. Real bug example
Using mutable POJO as key leads to lookup failures.

---

## Level 6: Iteration & Views

### 27. keySet / values / entrySet
Different views of map content.

### 28. Why entrySet faster?
Avoids extra lookup.

### 29. Fail-fast iterator
Throws ConcurrentModificationException.

### 30. modCount
Detects structural modifications.

### 31. Modification during iteration
Only via iterator.remove().

---

## Level 7: LinkedHashMap

### 32. How order is maintained?
Doubly linked list.

### 33. Insertion vs access order
Insertion keeps insert order; access moves entry to end.

### 34. LRU Cache
Use accessOrder=true.

### 35. removeEldestEntry()
Override to evict entries.

---

## Level 8: TreeMap

### 36. How sorting works?
Uses comparator or natural ordering.

### 37. Internal DS
Red-Black Tree.

### 38. Comparable vs Comparator
Natural vs external ordering.

### 39. Comparator inconsistency
Breaks map behavior.

### 40. Time complexity
O(log n).

### 41. Null keys?
Not allowed with natural ordering.

---

## Level 9: Hashtable vs HashMap

### 42. Why legacy?
Synchronized and slow.

### 43. Synchronization model
Method-level locking.

### 44. Why slower?
Single lock bottleneck.

### 45. Use today?
Almost none.

---

## Level 10: ConcurrentHashMap

### 46. Difference from HashMap
Thread-safe with fine-grained locking.

### 47. Locking
Java 8 uses CAS and synchronized bins.

### 48. Lock striping
Locks only part of map.

### 49. Why no nulls?
Avoid ambiguity in concurrency.

### 50. compute / merge APIs
Atomic compound operations.

---

## Level 11: Performance & Memory

### 51. Memory overhead
Nodes + references.

### 52. Objects per entry
At least one node object.

### 53. Large-scale danger
Memory bloat and GC pressure.

### 54. Cache locality
Poor due to pointers.

### 55. GC impact
Long pauses with many entries.

---

## Level 12: Edge Cases

### 56. Same hash for all keys
Worst-case collisions.

### 57. Infinite loop bug
Java 7 resize race.

### 58. Why not thread-safe?
No synchronization.

### 59. What breaks?
Lost updates.

### 60. Debugging prod issue
Heap dump + hash analysis.

---

## Level 13: Design Questions

### 61. Map vs List vs Array
Depends on access pattern.

### 62. Special Maps
EnumMap, IdentityHashMap.

### 63. Cache vs lookup
Eviction & concurrency matter.

### 64. High-performance lookup
Pre-size, immutable keys.

---

## Final Boss

### 65. Design your own Map
Buckets + hashing.

### 66. Trade-offs
Speed vs memory.

### 67. Optimization goals
Reads, writes, memory.

### 68. Build HashMap from scratch
Array + hash + collision handling.

---

End of file.

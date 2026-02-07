
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
- Each bucket contains:
  - Linked list (Java 7)
  - Linked list or Red-Black Tree (Java 8+)

### 7. What is a bucket?
A bucket is an **array index** where entries with same hash index are stored.

### 8. Role of hashCode() and equals()
- `hashCode()` → decides bucket
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
> hashCode() gives the number → hash spreading mixes the bits → index picks the bucket
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

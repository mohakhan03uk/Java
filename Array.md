
# Java Array

---

## 1. What Is an Array?

An array is a **fixed-size**, **index-based**, **contiguous memory** data structure that stores elements of the **same type**.

Key properties:
- Fixed size (decided at creation)
- O(1) random access
- Homogeneous elements
- Zero-based indexing

---

## 2. Arrays vs Collections

| Aspect | Array | Collection |
|------|------|------------|
| Size | Fixed | Dynamic |
| Performance | Faster | Slight overhead |
| Primitives | Supported | Not supported |
| API | Minimal | Rich |
| Memory | Compact | Extra object overhead |

---

## 3. Array Creation Syntax

```java
int[] a = new int[5];
int b[] = new int[5]; // valid but discouraged
int[] c = {1, 2, 3};
```

### Declaration Trap

```java
int[] a, b; // both arrays
int a[], b; // a is array, b is int
```

---

## 4. JVM Memory Model for Arrays

```java
int[] arr = new int[3];
```

- Reference stored on **stack**
- Array object stored on **heap**
- Primitive values stored directly in array
- Object arrays store **references**

---

## 5. Time & Space Complexity

| Operation | Complexity |
|--------|-----------|
| Access | O(1) |
| Update | O(1) |
| Search | O(n) |
| Insert/Delete (middle) | O(n) |

---

## 6. Traversal Techniques

```java
for (int i = 0; i < arr.length; i++)
for (int x : arr)
Arrays.stream(arr).forEach(System.out::println);
```

⚠ Streams add overhead for simple loops — don’t blindly use them in hot paths.\
Arrays win in CPU cache locality.

---

## 7. Array vs LinkedList

| Feature | Array | LinkedList |
|------|------|-----------|
| Memory | Contiguous | Non-contiguous |
| Cache friendly | Yes | No |
| Random access | O(1) | O(n) |
| Insert/Delete | O(n) | O(1) |

---

## 8. Multidimensional Array

```java
int[][] matrix = new int[3][];
matrix[0] = new int[2];
matrix[1] = new int[4];
```

- Java supports **jagged array**, not true matrices.
- Java does not guarantee contiguous memory for 2D arrays.

---

## 9. Array Utility Class

```java
Arrays.sort(arr);  arr = int[]        // Dual-Pivot QuickSort (primitives)
Arrays.sort(Object[]) ;               // → TimSort
Arrays.binarySearch(arr);            // Sorted only
Arrays.copyOf(arr, len);
Arrays.equals(a, b);
```

---

## 10. Common Pitfalls

```java
arr[arr.length]; // ArrayIndexOutOfBoundsException
int[] b = a;     // Shallow copy
```

Correct copy:
```java
int[] b = Arrays.copyOf(a, a.length);
```

---

# JVM DEEP DIVE SECTION

---
## Are arrays objects in Java?
```
arr instanceof Object // true
```
YES 

## Why lenght is a filed, not a method ?
- Because arrays are built-in JVM objects, not normal Java classes.
- They don’t go through normal method dispatch.

## All arrays share same class 
❌ Wrong
int[] and long[] are different runtime classes



## “Object[] can hold int[]”

❌ Wrong — primitive arrays are not Object arrays
```java
Object[] o = new int[5]; // ❌ compile-time error
Object o = new int[5];   // ✅
```

## 11. Runtime Classes of Arrays
```java
int[] a = new int[5];
System.out.println(a.getClass()); // class [I
```
- Have length field
- Have a runtime class: [I, [Ljava.lang.String;

```java
String[] s = new String[3];
System.out.println(s.getClass()); // class [Ljava.lang.String;
```
> In Java, every array has a JVM-generated runtime class.\
> Primitive arrays use descriptors like [I, and object arrays use [L<classname>;.\
> These descriptors encode array dimensionality and element type directly into the class name
### Primitive Type Codes

| Code | Type |
|----|------|
| I | int |
| B | byte |
| S | short |
| J | long |
| F | float |
| D | double |
| C | char |
| Z | boolean |

---

## 12. Multi-Dimensional Runtime Types

```java
int[][] a → [[I
String[][] s → [[Ljava.lang.String;
```

Each `[` adds one dimension.

---

## 13. Arrays Are Objects

```java
a instanceof Object        // true
a instanceof Cloneable     // true
a instanceof Serializable  // true
```

- `length` is a **field**
- Arrays are JVM-native objects

---

## 14. Method Descriptors (JVM Signatures)

Format:
```
(parameter-descriptors)return-descriptor
```

Example:
```
(I[Ljava/lang/String;)V
```

Java equivalent:
```java
void method(int x, String[] arr)
```

---

## 15. Descriptor Examples

| Descriptor | Java Method |
|----------|------------|
| ()V | void m() |
| (I)I | int m(int) |
| ([I)V | void m(int[]) |
| ([[Ljava/lang/String;)Ljava/lang/Object; | Object m(String[][]) |

---

## 16. Serialization Impact

- Runtime array class is serialized
- `[I` ≠ `[Ljava.lang.Integer;`

```java
int[] a = {1,2,3};
Integer[] b = (Integer[]) (Object) a; // ClassCastException
```

---

## 17. JNI Impact

```java
native int sum(int[] arr);
```

JNI signature:
```
([I)I
```

Descriptor mismatch → `UnsatisfiedLinkError`

---

## 18. ArrayStoreException & Runtime Checks

```java
Object[] arr = new String[3];
arr[0] = "hello";
arr[1] = 100; // ArrayStoreException
```

Reason:
- Arrays are **covariant**
- JVM enforces runtime type safety

Primitive arrays are invariant and safer.

---

## 19. Big Picture Connection

| Concept | Purpose |
|------|--------|
| `[I`, `[Ljava.lang.String;` | Runtime type |
| Method descriptors | JVM linking |
| Serialization | Compatibility |
| JNI | Native binding |
| ArrayStoreException | Safety |

---

## 20. Summary

> Arrays are JVM-level constructs with real runtime classes.  
> Their descriptors power method invocation, serialization, JNI binding, and runtime type checks.
> Arrays  are preferred in system-level code, Because they offer predictable memory, minimal overhead, and better CPU cache utilization.
> Arrays are the most primitive, performance-oriented data structure in Java — simple, fast, but inflexible.

---

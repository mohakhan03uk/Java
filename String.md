# Strings
---

## 1. What is a String in Java?

- `String` is a **final class** representing an immutable sequence of characters.
- Internally (Java 8):
  - `String` object → Heap
  - `char[]` → Heap
- From Java 9+:
  - Uses `byte[]` + encoding flag (Compact Strings)

---

## 2. Why is String Immutable?

Key reasons (interview-grade):

1. **HashCode Caching**
   - Hashcode computed once
   - Enables safe use as `Map` keys

2. **String Pool Optimization**
   - Same literals reused
   - Saves memory

3. **Security**
   - Used in:
     - Class loading
     - File paths
     - Network URLs
   - Prevents tampering after creation

4. **Thread Safety**
   - Safe to share without synchronization
   - Side effect, not the primary goal

---

## 3. String Memory Layout (Evolution)

### Java 6
- String Pool → **PermGen**
- `String` object → Heap
- `char[]` → Heap
- Risk: `OutOfMemoryError: PermGen space`

### Java 7
- String Pool moved to **Heap**
- Interned strings GC-managed

### Java 8+
- PermGen removed
- Metaspace introduced (class metadata)
- String Pool still in Heap

---

## 4. String.intern() – Deep Internals

### What `intern()` Does
- Checks String Pool
- If exists → returns pooled reference
- Else → adds to pool and returns reference

### Java 6 vs Java 7+
- Java 6: copies string into PermGen
- Java 7+: stores reference in Heap pool

### Production Guidance
❌ Avoid for dynamic/unbounded data  
✅ Safe for finite, enum-like values

---

## 5. Reference Comparison Trap

```java
String a = new String("java");
String b = a.intern();
String c = "java";
```

Results (Java 7+):

```java
a == b   // false
b == c   // true
```

Why:
- `a` → heap object
- `b`, `c` → pooled object

---

## 6. String Concatenation Performance Bug

### Buggy Code

```java
String result = "";
for (int i = 0; i < 1_000_000; i++) {
    result += i;
}
```

### What Happens Internally

Each iteration:
```java
result = new StringBuilder(result)
            .append(i)
            .toString();
```

### Complexity
- **Time**: O(n²)
- **Space**: Excessive temporary objects
- Causes GC pressure and latency issues

⚠️ No String Pool involvement here.

---

## 7. Correct Pattern

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1_000_000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### Why This Is Better
- Single `StringBuilder`
- Avoids repeated copying
- Linear time complexity
- Minimal object creation

---

## 8. One-Liner

> Using a single StringBuilder avoids repeated object creation, repeated character array copying, and quadratic time complexity caused by rebuilding the string on every iteration.

---

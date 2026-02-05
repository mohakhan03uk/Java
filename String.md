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
- Problem:
- > PermGen was fixed-size \
  > Too many interned strings → OutOfMemoryError: PermGen space

### Java 7
- String Pool moved to **Heap**
- `String` object -> Heap
- `char[]` -> Heap
- Why this change?
- > Heap is GC-managed \
  > Easier tuning \
  > Reduced PermGen pressu

### Java 8+
- PermGen removed
- Metaspace introduced (class metadata)
- String Pool still in Heap
- Bonus change (Java 9):
- > char[] → byte[] (Compact Strings) \
  > Saves memory for Latin-1 strings

---

## 4. String.intern() – Deep Internals

### What `intern()` Does
- Checks String Pool
- If exists → returns pooled reference
- Else → adds to pool and returns reference
- So:
- > String s = new String("abc").intern(); \
  > `s` points to the pooled instance

### Java 6 vs Java 7+
- Java 6:
- > Pool in PermGen \
  > copies string into PermGen\
  > High Risk of OutOfMemoryError: PermGen space\
- Java 7+:
- > Pool moved to Heap\
  > stores reference in Heap pool\
  > intern() stores reference, not a copy\
  > Much cheaper & GC-friendly\
  > This change made intern() usable — but still dangerous.

### Production Guidance
- Short answer: Usually no
- > Why?
> Pool entries are effectively immortal\
> Dynamic data (usernames, request IDs, headers):\
> Never reused\
> Pollutes the pool\
> Memory leak in disguise

- > **When it is acceptable**
> Finite, well-known strings:\
> Protocol names\
> Enum-like values

**Keywords**\
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
**Step-by-step**

1. "java" \              
 String literal\
 Goes into String Pool
 ***This is the trap, literal will go to pool when JVM will come up, hence the 'java' is already there and intern() will return its reference***
2. new String("java")\
Creates a new object on the heap\
a → heap object (NOT pooled)

3. a.intern()
   Pool already contains "java"\
   Returns reference to pooled string\
   b → pooled instance
4. String c = "java"\
  Directly references pooled string


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

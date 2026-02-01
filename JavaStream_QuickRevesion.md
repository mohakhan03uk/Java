
# Java Streams — Quick Revision with Co‑Located Examples

---

## 1) What Streams Are For
- **Streams let you write complex data‑processing pipelines in a clear, functional style (filter → map → reduce).**

  ```java
  List<Integer> data = List.of(1, 2, 3, 4, 5);
  int sumOfEvenSquares = data.stream()
      .filter(n -> n % 2 == 0)  // keep evens: 2,4
      .map(n -> n * n)          // square: 4,16
      .reduce(0, Integer::sum); // sum: 20
  ```

---

## 2) Filtering & Slicing (Selecting Elements)
- **`filter` — keep elements that match a predicate.**
  ```java
  List<Integer> out = List.of(1,2,3,4,5).stream()
      .filter(n -> n % 2 == 0)
      .toList(); // [2, 4]
  ```

- **`distinct` — remove duplicates (stateful).**
  ```java
  List<Integer> out = List.of(1,2,2,3).stream()
      .distinct()
      .toList(); // [1, 2, 3]
  ```

- **`skip(n)` — drop the first n elements.**
  ```java
  List<Integer> out = List.of(10,20,30,40).stream()
      .skip(2)
      .toList(); // [30, 40]
  ```

- **`limit(n)` — take only the first n elements.**
  ```java
  List<Integer> out = List.of(1,2,3,4,5).stream()
      .limit(3)
      .toList(); // [1, 2, 3]
  ```

- **`takeWhile` (Java 9+) — take elements while predicate stays true (great on sorted data).**
  ```java
  List<Integer> out = List.of(1,2,3,7,8).stream()
      .takeWhile(n -> n < 5)
      .toList(); // [1, 2, 3]
  ```

- **`dropWhile` (Java 9+) — drop elements while predicate is true, then keep the rest.**
  ```java
  List<Integer> out = List.of(1,2,3,7,8).stream()
      .dropWhile(n -> n < 5)
      .toList(); // [7, 8]
  ```

---

## 3) Why `takeWhile` / `dropWhile` Beat `filter` on Sorted Sources
- **On a sorted stream, `takeWhile`/`dropWhile` can short‑circuit early, while `filter` must scan all elements.**
  ```java
  List<Integer> sorted = List.of(1,2,3,10,11,12);
  // takeWhile stops as soon as it sees 10
  List<Integer> a = sorted.stream().takeWhile(n -> n < 5).toList(); // [1,2,3]
  // filter still checks every element
  List<Integer> b = sorted.stream().filter(n -> n < 5).toList();    // [1,2,3]
  ```

---

## 4) Transforming Data
- **`map` — one‑to‑one transformation.**
  ```java
  List<String> words = List.of("a", "bb", "ccc");
  List<Integer> lengths = words.stream()
      .map(String::length)
      .toList(); // [1, 2, 3]
  ```

- **`flatMap` — flatten nested content (list of lists → single list).**
  ```java
  List<List<Integer>> matrix = List.of(List.of(1,2), List.of(3,4));
  List<Integer> flat = matrix.stream()
      .flatMap(List::stream)
      .toList(); // [1, 2, 3, 4]
  ```

---

## 5) Finding & Matching
- **`findFirst` / `findAny` — retrieve an element wrapped in `Optional`.**
  ```java
  int first = Stream.of(9,8,7).findFirst().orElse(-1); // 9
  int any   = Stream.of(9,8,7).findAny().orElse(-1);   // any element (non‑deterministic in parallel)
  ```

- **`anyMatch` / `allMatch` / `noneMatch` — test predicates with short‑circuiting.**
  ```java
  boolean anyEven  = Stream.of(1,2,3).anyMatch(n -> n % 2 == 0);  // true
  boolean allEven  = Stream.of(2,4,6).allMatch(n -> n % 2 == 0);  // true
  boolean noneEven = Stream.of(1,3,5).noneMatch(n -> n % 2 == 0); // true
  ```

---

## 6) Short‑Circuiting (Stops Early When Answer Known)
- **Operations like `anyMatch`, `allMatch`, `noneMatch`, `findFirst`, `findAny`, `limit`, `takeWhile` may stop without scanning the whole stream.**
  ```java
  // Stops the moment it sees 6 (even)
  boolean foundEven = Stream.of(1,3,5,6,7,8)
      .anyMatch(n -> n % 2 == 0); // true

  // Creating a finite view over an infinite stream
  Stream<Double> infinite = Stream.generate(Math::random);
  List<Double> three = infinite.limit(3).toList(); // exactly 3 numbers
  ```

---

## 7) Reducing (Combine to One Result)
- **`reduce` folds elements into a single value (sum, min, max, product, concat).**
  ```java
  int sum = Stream.of(1,2,3,4).reduce(0, Integer::sum);                 // 10
  int max = Stream.of(5,1,9,2).reduce(Integer.MIN_VALUE, Integer::max); // 9
  ```
- **Tip:** Use `reduce` for associative operations. For mutable accumulation (e.g., building lists/maps), prefer `collect`/`Collectors` (see Bonus).

---

## 8) Stateless vs Stateful Operations
- **Stateless intermediates** don’t retain history across elements (e.g., `map`, `filter`).
  ```java
  List<Integer> doubled = Stream.of(1,2,3).map(n -> n * 2).toList(); // [2,4,6]
  ```
- **Stateful intermediates** need buffering (e.g., `sorted`, `distinct`), which can require processing the whole stream before producing output.
  ```java
  List<Integer> sorted = Stream.of(3,1,2).sorted().toList();        // [1,2,3]
  List<Integer> uniq   = Stream.of(1,2,2,3).distinct().toList();    // [1,2,3]
  ```

---

## 9) Primitive Streams (Avoid Boxing)
- **Use `IntStream`, `LongStream`, `DoubleStream` for numeric workloads.**
  ```java
  int total = IntStream.of(1,2,3).sum();                            // 6
  double avg = DoubleStream.of(1.0, 2.0, 4.0).average().orElse(0); // 2.333...
  long count = LongStream.rangeClosed(1, 3).count();                // 3

  // Convert between object and primitive streams
  int lenSum = Stream.of("a","bb","ccc")
      .mapToInt(String::length)   // to IntStream
      .sum(); // 6
  ```

---

## 10) Creating Streams
- **From collections**
  ```java
  Stream<String> s1 = List.of("a","b","c").stream();
  ```
- **From arrays**
  ```java
  int sumArr = Arrays.stream(new int[]{1,2,3}).sum(); // 6
  ```
- **From values**
  ```java
  Stream<String> s2 = Stream.of("x","y");
  ```
- **From files** *(use try‑with‑resources in real code)*
  ```java
  // long lines = Files.lines(Path.of("data.txt")).count();
  ```
- **Infinite generators**
  ```java
  Stream<Integer> inc = Stream.iterate(1, n -> n + 1).limit(3); // 1,2,3
  Stream<Double> rnd = Stream.generate(Math::random).limit(2);  // 2 randoms
  ```
- **Other handy sources** (good to know)
  ```java
  // Regex split as a stream
  List<String> tokens = Pattern.compile(",").splitAsStream("a,b,c").toList();

  // Optional to stream (Java 9+)
  List<String> present = Optional.of("val").stream().toList(); // ["val"]
  ```

---

## 11) Infinite Streams (Laziness)
- **Streams produce elements on demand. Convert infinite → finite with `limit` or `takeWhile`.**
  ```java
  // Simple example
  List<Integer> firstFive = Stream.iterate(1, n -> n + 1)
      .limit(5)
      .toList(); // [1,2,3,4,5]

  // Tiny Fibonacci (pairs)
  List<Integer> fib = Stream.iterate(new int[]{0,1}, p -> new int[]{p[1], p[0]+p[1]})
      .limit(7)
      .map(p -> p[0])
      .toList(); // [0,1,1,2,3,5,8]
  ```

---

## Bonus: `collect` vs `reduce`, and `toList()` vs `Collectors.toList()`
- **Prefer `collect` for mutable accumulation** (e.g., building lists/maps/grouping), and keep `reduce` for mathematical/associative folds.
  ```java
  // Build a list using collect
  List<Integer> evens = Stream.of(1,2,3,4)
      .filter(n -> n % 2 == 0)
      .collect(Collectors.toList()); // [2,4]

  // Grouping example
  Map<Integer, List<String>> byLen = Stream.of("a","bb","ccc")
      .collect(Collectors.groupingBy(String::length));
  // {1=[a], 2=[bb], 3=[ccc]}
  ```
- **`toList()` vs `Collectors.toList()`**
  - `stream.toList()` (Java 16+) returns an **unmodifiable** list.
  - `collect(Collectors.toList())` returns a **mutable** list (implementation not specified).
  ```java
  List<Integer> a = Stream.of(1,2,3).toList();               // unmodifiable
  List<Integer> b = Stream.of(1,2,3).collect(Collectors.toList()); // mutable
  ```

---

## Copy‑Paste Imports
```java
import java.util.*;
import java.util.regex.*;
import java.util.stream.*;
import java.nio.file.*; // if using Files.lines
```

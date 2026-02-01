
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

## Imports
```java
import java.util.*;
import java.util.regex.*;
import java.util.stream.*;
import java.nio.file.*; // if using Files.lines
```





# Java Streams: Finding & Matching — Optional<T> Examples

> Java 17+ assumed. Import helpers:
>
> ```java
> import java.util.*;
> import java.util.stream.*;
> import java.util.regex.*;
> ```
>
> Each example **returns `Optional<T>`** and includes a short description of what the code does and why.

---

## 1) `findFirst()` — first element that satisfies a condition
**What this does:** Filters the stream for names longer than 3 characters and returns the **first** one wrapped in `Optional<String>`.
```java
List<String> names = List.of("Al", "Alice", "Bob", "Charlie");
Optional<String> first = names.stream()
    .filter(s -> s.length() > 3)
    .findFirst(); // Optional["Alice"]
```
**Why:** `findFirst()` preserves encounter order on ordered streams and returns `Optional.empty()` if none match.

---

## 2) `findAny()` — any matching element (parallel-friendly)
**What this does:** In parallel, returns **any** element > 15 (non-deterministic which one) as `Optional<Integer>`.
```java
List<Integer> nums = List.of(10, 20, 30, 40);
Optional<Integer> any = nums.parallelStream()
    .filter(n -> n > 15)
    .findAny(); // Optional[20] or Optional[30] or Optional[40]
```
**Why:** `findAny()` allows more optimization in parallel pipelines and does not guarantee the first element.

---

## 3) `max` / `min` — terminal ops that return `Optional<T>`
**What this does:** Finds the user with the **maximum score** and the one with the **minimum name (lexicographically)**.
```java
record User(String name, int score) {}

List<User> users = List.of(new User("Ann", 75), new User("Ben", 92), new User("Cara", 88));
Optional<User> top = users.stream()
    .max(Comparator.comparingInt(User::score));        // Optional[User("Ben",92)]
Optional<User> minByName = users.stream()
    .min(Comparator.comparing(User::name));            // Optional[User("Ann",75)]
```
**Why:** `max`/`min` return `Optional` because the stream might be empty; handle with `orElse*`.

---

## 4) `reduce` without identity — Optional because stream may be empty
**What this does:** Sums integers and picks the **longest** string, both using `reduce(BinaryOperator)` (no identity).
```java
Optional<Integer> sum = Stream.of(1, 2, 3, 4)
    .reduce(Integer::sum); // Optional[10]

Optional<String> longest = Stream.of("a", "bbb", "cc")
    .reduce((a, b) -> a.length() >= b.length() ? a : b); // Optional["bbb"]
```
**Why:** No identity value → result is undefined for empty streams → `Optional<T>`.

---

## 5) Map → Find pattern — compute then find
**What this does:** Maps words to their lengths, then finds the first whose length is exactly 5.
```java
List<String> words = List.of("x", "alpha", "beta");
Optional<Integer> len = words.stream()
    .map(String::length)
    .filter(n -> n == 5)
    .findFirst(); // Optional[5]
```
**Why:** Often you transform (`map`) before deciding which element to select.

---

## 6) Primitive streams → `Optional<T>` via boxing
**What this does:** Gets the **maximum** from an `IntStream` but returns `Optional<Integer>` by boxing.
```java
Optional<Integer> maxEven = IntStream.of(2, 4, 6)
    .boxed()                     // IntStream -> Stream<Integer>
    .max(Integer::compareTo);    // Optional[6]
```
**Why:** Primitive ops like `IntStream.max()` return `OptionalInt`; boxing keeps a uniform `Optional<T>` type.

---

## 7) `Optional`-producing lookups + `flatMap(Optional::stream)`
**What this does:** Tries a key lookup that may fail (`Optional`), flattens successes, and returns the **first** found value.
```java
Optional<String> lookup(String key) {
    return "id".equals(key) ? Optional.of("FOUND") : Optional.empty();
}

List<String> keys = List.of("x", "y", "id", "z");
Optional<String> firstFound = keys.stream()
    .map(k -> lookup(k))          // Stream<Optional<String>>
    .flatMap(Optional::stream)    // Stream<String>
    .findFirst();                 // Optional["FOUND"]
```
**Why:** `Optional::stream` (Java 9+) is perfect to flatten a `Stream<Optional<T>>` into a `Stream<T>`.

---

## 8) Deduplicate + sort + `findFirst()` — smallest that matches
**What this does:** Gets the **smallest even** number as `Optional<Integer>`.
```java
Optional<Integer> smallestEven = Stream.of(9, 4, 6, 2, 7, 4)
    .filter(n -> n % 2 == 0)   // 4, 6, 2, 4
    .distinct()                // 4, 6, 2
    .sorted()                  // 2, 4, 6
    .findFirst();              // Optional[2]
```
**Why:** Ordering + `findFirst` reliably picks the minimal element that satisfies your predicate.

---

## 9) Wrap a collected result into `Optional`
**What this does:** Collects items > 100 into a list; if the list is empty, returns `Optional.empty()` instead.
```java
List<Integer> data = List.of();
Optional<List<Integer>> maybeList = data.stream()
    .filter(n -> n > 100)
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        lst -> lst.isEmpty() ? Optional.empty() : Optional.of(lst)
    )); // Optional.empty
```
**Why:** Useful when you want to signal “no result” without returning an empty container.

---

## 10) Stay in `Optional` after `findFirst` (no premature unwrapping)
**What this does:** Finds a 3‑letter word and **maps** it to uppercase without leaving the `Optional` world.
```java
Optional<String> upper = Stream.of("one", "two", "three")
    .filter(s -> s.length() == 3)
    .findFirst()               // Optional["one"]
    .map(String::toUpperCase); // Optional["ONE"]
```
**Why:** Prefer `Optional.map/flatMap/orElse*` to keep null‑safety and readability.

---

## 11) Guard with `findFirst().or(() -> ...)` (Java 9+)
**What this does:** Try primary source; if empty, try a fallback source — all with `Optional<T>`.
```java
Optional<String> primary = Stream.<String>of().findFirst(); // empty
Optional<String> fallback = Stream.of("backup").findFirst();

Optional<String> result = primary.or(() -> fallback); // Optional["backup"]
```
**Why:** `Optional.or` composes fallbacks elegantly without unwrapping.

---

## 12) Using `Optional.filter` to post‑validate a found value
**What this does:** Finds any user, then keeps it only if active.
```java
record User(String name, boolean active) {}

Optional<User> maybeActive = Stream.of(
        new User("Ann", false),
        new User("Ben", true)
    )
    .findFirst()          // Optional[User("Ann", false)]
    .filter(User::active); // Optional.empty (Ann is not active)
```
**Why:** `Optional.filter` lets you chain an additional predicate after selection.

---

### Notes / Pitfalls
- `findFirst`/`findAny`/`max`/`min`/no‑identity `reduce` return `Optional.empty()` on empty streams → handle with `orElse*`, `ifPresent`, `orElseThrow`.
- `findAny` is **non‑deterministic** on parallel streams; prefer `findFirst` when order matters.
- Avoid side effects in predicates — short‑circuiting & parallelism can make behavior surprising.




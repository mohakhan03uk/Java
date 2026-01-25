
# Java Streams
Goal: Learn Streams by solving realistic, teacher‑designed tasks that reveal concepts like filter, map, limit, short‑circuiting (findFirst, anyMatch, limit), loop fusion (single‑pass pipelines), flatMap, collectors, and more.

---

## Setup (Models & Sample Data)

Create a project with a class `StreamsScenarioLab` and paste the following shared dataset. You will use it across scenarios.

```java
import java.util.*;
import java.util.stream.*;

public class StreamsScenarioLab {
    // Domain models
    public record Person(String name, int age, String city) {}
    public record LineItem(String sku, int quantity, double unitPrice) {}
    public record Order(int id, String city, List<LineItem> items) {}

    // Shared datasets
    static final List<Integer> numbers = List.of(3, 9, 2, 7, 12, 8, 5, 6, 11, 4, 10, 1);
    static final List<String> words = List.of("alpha", "beta", "gamma", "delta", "epsilon", "zeta", "eta", "theta");

    static final List<Person> people = List.of(
        new Person("Aman", 28, "Pune"),
        new Person("Riya", 34, "Mumbai"),
        new Person("Karan", 22, "Delhi"),
        new Person("Neha", 29, "Pune"),
        new Person("Irfan", 41, "Bengaluru"),
        new Person("Zoya", 23, "Pune"),
        new Person("Asha", 30, "Mumbai")
    );

    static final List<List<Integer>> matrix = List.of(
        List.of(1, 2, 3),
        List.of(4, 5, 6),
        List.of(7, 8, 9, 10, 11)
    );

    static final List<Order> orders = List.of(
        new Order(101, "Pune", List.of(
            new LineItem("SKU-RED", 2, 499.0),
            new LineItem("SKU-GRN", 1, 1299.0)
        )),
        new Order(102, "Delhi", List.of(
            new LineItem("SKU-BLU", 5, 199.0),
            new LineItem("SKU-RED", 1, 499.0)
        )),
        new Order(103, "Mumbai", List.of(
            new LineItem("SKU-YLW", 3, 799.0),
            new LineItem("SKU-GRN", 2, 1299.0)
        )),
        new Order(104, "Pune", List.of(
            new LineItem("SKU-PRP", 4, 299.0),
            new LineItem("SKU-RED", 1, 499.0),
            new LineItem("SKU-GRN", 1, 1299.0)
        ))
    );
}
```

---

## Part I — Basic Stream Concepts

Focus: filter, map, sorted, limit, skip, findFirst, anyMatch, allMatch, primitive streams.

### Scenario 1 — Even Squares (Filter → Map → Limit)
From `numbers`, produce the first three squares of even numbers greater than 5, ascending.

### Scenario 2 — Uppercase and Sort Words
From `words`, keep words with length ≥ 5, convert to uppercase, sort lexicographically, and return the list.

### Scenario 3 — Paging with skip/limit
Treat sorted `numbers` as a list. Return page 2 with pageSize = 4.

### Scenario 4 — findFirst after Sort
From `words`, after sorting, find the first word that contains 't' and has length > 4. Return `Optional<String>`.

### Scenario 5 — Validations with Match Operations
Using `numbers`:
1) Does any number end with digit 1?
2) Do all numbers lie between 1 and 12 inclusive?
3) Are there no negative numbers?

### Scenario 6 — Custom Sort by Length then Alphabetical
Sort `words` by length ascending, then alphabetically for ties. Return the sorted list.

### Scenario 7 — distinct + count
Given `List<Integer> nums = List.of(5,2,2,3,3,3,4,4,4,4);` compute the count of distinct squares.

### Scenario 8 — mapToInt Sum and Average
From `numbers`, compute the sum of all values and the average of odd numbers (as `OptionalDouble`).

### Scenario 9 — Ranges and Boxing
Create integers 1..20, filter multiples of 3, collect into a list of their cubes.

### Scenario 10 — Observe Execution Order (peek)
Using `numbers` sorted ascending, build a pipeline: peek → filter even → map square → limit 3. Print diagnostic messages to observe per‑element processing order and early stop at limit.

---

## Part II — Intermediate Stream Patterns

Focus: flatMap, Optional mapping, collect with grouping/partitioning, distinct, encounter order.

### Scenario 11 — Flatten Nested Lists
From `matrix`, flatten into a single stream and return the first number > 8 (as `Optional<Integer>`).

### Scenario 12 — Orders: First Expensive Line Item (short‑circuit)
From `orders`, find the first line item with `unitPrice * quantity > 2000`, returning `Optional<LineItem>`.

### Scenario 13 — Group People by City, Top‑N by Age
Group `people` by `city`. For each city, return the top 2 oldest names (descending age), as a `Map<String, List<String>>` with names only.

### Scenario 14 — Join Names of Pune Residents
From `people`, all in Pune: uppercase names, sort alphabetically, then join with ", " into a single `String`.

### Scenario 15 — Partition by Age ≥ 30
Partition `people` into `true/false` by `age >= 30`. For each partition, return just the names list, preserving encounter order.

### Scenario 16 — First 3 Distinct Squares
From `numbers` sorted ascending, compute the first 3 distinct squares as a list (use `distinct` + `limit`).

### Scenario 17 — Encounter Order Matters
Show that `findFirst` on a parallel stream can still return a deterministic result if the source has encounter order and the terminal operation honors it. Implement both: sequential `findFirst` and parallel `findAny` on `words` after sorting, and compare behavior.

### Scenario 18 — Optional Mapping
Return the uppercase name of the first Pune resident aged ≥ 25, or "NONE" if not found. Use `map` on `Optional`.

### Scenario 19 — Multi‑criteria Sorting
From `people`, sort by city ascending, then by age descending, then by name ascending. Return the sorted list.

### Scenario 20 — Finite Slice of Infinite Stream
Generate an infinite stream of Fibonacci numbers, take the first 10 even Fibonacci numbers, and return their sum.

---

## Part III — Advanced: Performance, Collectors, Parallelism

Focus: loop fusion vs materialization, custom collectors, reduce, parallel pitfalls.

### Scenario 21 — Loop Fusion vs Materialization
Compute the sum of (double of even numbers) in `numbers` in two ways: (A) materialize intermediate lists after filter/map; (B) a single fused pipeline. Compare the two approaches in comments.

### Scenario 22 — reduce to Longest Word
Using `words`, find the longest word with `reduce`. If tie, prefer lexicographically smaller. Return `Optional<String>`.

### Scenario 23 — Custom Merge in toMap
Collect `people` into a `Map<String, Integer>` of `name -> age`. If duplicate names occur, keep the max age.

### Scenario 24 — Downstream Collector with Ordering and Limit
For each `city`, collect uppercased names into a `TreeSet` to get natural order, then return only the first 2 names per city as `List`. (Hint: `groupingBy` + `collectingAndThen`).

### Scenario 25 — Parallel Streams and Short‑Circuit Caveat
Demonstrate that `findAny` on a parallel stream of `numbers` filtered to `> 8` can return any such element (non‑deterministic). Print several runs to observe.

### Scenario 26 — Windowed Computation (Moving Average)
Given `List<Integer> series = List.of(3, 5, 7, 6, 8, 10, 9);` compute a moving average with window=3 using index ranges and `mapToDouble`. Return a `List<Double>` with 5 values.

---

# Solutions and Explanations (with Functional Interfaces Used)

Import note: All code assumes the Setup section has been added to your project. You can place each solution in a `main` method or separate test methods.

### Solution 1 — Even Squares
```java
List<Integer> out = numbers.stream()
    .sorted()
    .filter(n -> n % 2 == 0 && n > 5)
    .map(n -> n * n)
    .limit(3)
    .collect(Collectors.toList());
```
Functional interfaces: `Comparator<Integer>` (sorted), `Predicate<Integer>` (filter), `Function<Integer,Integer>` (map), internal short‑circuit via `limit` terminal operation. Why: `Predicate` expresses boolean test; `Function` expresses value transform; `Comparator` defines order.

### Solution 2 — Uppercase and Sort Words
```java
List<String> out = words.stream()
    .filter(w -> w.length() >= 5)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```
Functional interfaces: `Predicate<String>` (filter), `Function<String,String>` (map via method reference), `Comparator<String>` (natural order). Why: matching, transforming, then ordering the stream.

### Solution 3 — Paging with skip/limit
```java
int page = 2, pageSize = 4;
List<Integer> page2 = numbers.stream()
    .sorted()
    .skip((page - 1) * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
```
Functional interfaces: `Comparator<Integer>` (sorted). `skip`/`limit` are terminal/short‑circuit helpers and do not introduce new functional interfaces here.

### Solution 4 — findFirst after Sort
```java
Optional<String> first = words.stream()
    .sorted()
    .filter(w -> w.contains("t") && w.length() > 4)
    .findFirst();
```
Functional interfaces: `Comparator<String>` (sorted), `Predicate<String>` (filter). Why: filter decides eligibility; `findFirst` short‑circuits on first match in encounter order.

### Solution 5 — Match Operations
```java
boolean anyEndsWith1 = numbers.stream().anyMatch(n -> n % 10 == 1);
boolean allBetween1And12 = numbers.stream().allMatch(n -> n >= 1 && n <= 12);
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);
```
Functional interfaces: `Predicate<Integer>` for each match operation. Why: predicates provide boolean tests consumed by short‑circuiting terminal ops.

### Solution 6 — Custom Sort by Length then Alphabetical
```java
List<String> sortedWords = words.stream()
    .sorted(Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder()))
    .collect(Collectors.toList());
```
Functional interfaces: `Comparator<String>` (composed). Why: multi‑criteria ordering via comparator composition.

### Solution 7 — distinct + count
```java
List<Integer> nums = List.of(5,2,2,3,3,3,4,4,4,4);
long distinctSquares = nums.stream()
    .map(n -> n * n)
    .distinct()
    .count();
```
Functional interfaces: `Function<Integer,Integer>` (map). Why: transform values before distinct; `distinct` relies on equals/hashCode, not a functional interface.

### Solution 8 — mapToInt Sum and Average
```java
int sum = numbers.stream().mapToInt(Integer::intValue).sum();
OptionalDouble avgOdd = numbers.stream()
    .filter(n -> n % 2 != 0)
    .mapToInt(Integer::intValue)
    .average();
```
Functional interfaces: `Predicate<Integer>` (filter), `ToIntFunction<Integer>` (mapToInt). Why: use primitive specialization to avoid boxing and for numeric reductions.

### Solution 9 — Ranges and Boxing
```java
List<Integer> cubes = IntStream.rangeClosed(1, 20)
    .filter(n -> n % 3 == 0)
    .map(n -> n * n * n)
    .boxed()
    .collect(Collectors.toList());
```
Functional interfaces (primitive specializations): `IntPredicate` (filter), `IntUnaryOperator` (map). Why: efficient primitive pipeline, then box once at the end for List.

### Solution 10 — Observe Execution Order (peek)
```java
List<Integer> out = numbers.stream()
    .sorted()
    .peek(n -> System.out.println("S:" + n))
    .filter(n -> { System.out.println("F:" + n); return n % 2 == 0; })
    .map(n -> { System.out.println("M:" + n); return n * n; })
    .limit(3)
    .collect(Collectors.toList());
```
Functional interfaces: `Consumer<Integer>` (peek), `Predicate<Integer>` (filter), `Function<Integer,Integer>` (map), `Comparator<Integer>` (sorted). Why: observe, decide, transform, and short‑circuit via `limit`.

### Solution 11 — Flatten Nested Lists
```java
Optional<Integer> firstGt8 = matrix.stream()
    .flatMap(List::stream)
    .filter(n -> n > 8)
    .findFirst();
```
Functional interfaces: `Function<List<Integer>, Stream<Integer>>` (flatMap), `Predicate<Integer>` (filter). Why: flatten nested structure, then test sequentially for early exit.

### Solution 12 — Orders: First Expensive Line Item
```java
Optional<StreamsScenarioLab.LineItem> expensive = orders.stream()
    .flatMap(o -> o.items().stream())
    .filter(li -> li.unitPrice() * li.quantity() > 2000.0)
    .findFirst();
```
Functional interfaces: `Function<Order, Stream<LineItem>>` (flatMap), `Predicate<LineItem>` (filter). Why: traverse all line items linearly and short‑circuit on first qualifying cost.

### Solution 13 — Group People by City, Top‑N by Age
```java
Map<String, List<String>> top2ByCity = people.stream()
    .collect(Collectors.groupingBy(
        StreamsScenarioLab.Person::city,
        Collectors.collectingAndThen(
            Collectors.toList(),
            lst -> lst.stream()
                .sorted(Comparator.comparingInt(StreamsScenarioLab.Person::age).reversed())
                .limit(2)
                .map(StreamsScenarioLab.Person::name)
                .collect(Collectors.toList())
        )
    ));
```
Functional interfaces: `Function<Person,String>` (classifier for groupingBy), `Comparator<Person>` (sorted), `Function<Person,String>` (map). Why: classify by city, then post‑process each bucket with ordering and slicing.

### Solution 14 — Join Names of Pune Residents
```java
String puneJoined = people.stream()
    .filter(p -> p.city().equals("Pune"))
    .map(p -> p.name().toUpperCase())
    .sorted()
    .collect(Collectors.joining(", "));
```
Functional interfaces: `Predicate<Person>` (filter), `Function<Person,String>` (map), `Comparator<String>` (sorted). Why: narrow by city, transform, order, and join with a collector.

### Solution 15 — Partition by Age ≥ 30
```java
Map<Boolean, List<String>> partition = people.stream()
    .collect(Collectors.partitioningBy(
        p -> p.age() >= 30,
        Collectors.mapping(StreamsScenarioLab.Person::name, Collectors.toList())
    ));
```
Functional interfaces: `Predicate<Person>` (partitioningBy key), `Function<Person,String>` (mapping). Why: boolean split and transform values per bucket.

### Solution 16 — First 3 Distinct Squares
```java
List<Integer> first3 = numbers.stream()
    .sorted()
    .map(n -> n * n)
    .distinct()
    .limit(3)
    .collect(Collectors.toList());
```
Functional interfaces: `Comparator<Integer>` (sorted), `Function<Integer,Integer>` (map). Why: transform then deduplicate; `limit` short‑circuits after three unique values.

### Solution 17 — Encounter Order Matters
```java
Optional<String> seqFirst = words.stream()
    .sorted()
    .filter(w -> w.contains("t"))
    .findFirst();

Optional<String> parAny = words.parallelStream()
    .sorted()
    .filter(w -> w.contains("t"))
    .findAny();
```
Functional interfaces: `Comparator<String>` (sorted), `Predicate<String>` (filter). Why: demonstrate deterministic `findFirst` vs non‑deterministic `findAny` in parallel context.

### Solution 18 — Optional Mapping
```java
String nameOrNone = people.stream()
    .filter(p -> p.city().equals("Pune") && p.age() >= 25)
    .map(StreamsScenarioLab.Person::name)
    .map(String::toUpperCase)
    .findFirst()
    .orElse("NONE");
```
Functional interfaces: `Predicate<Person>` (filter), `Function<Person,String>` then `Function<String,String>` (map and map). Why: filter then transform twice before optional extraction.

### Solution 19 — Multi‑criteria Sorting
```java
List<Person> sortedPeople = people.stream()
    .sorted(Comparator
        .comparing(StreamsScenarioLab.Person::city)
        .thenComparing(StreamsScenarioLab.Person::age, Comparator.reverseOrder())
        .thenComparing(StreamsScenarioLab.Person::name))
    .collect(Collectors.toList());
```
Functional interfaces: `Comparator<Person>` (composed from key extractors, which are `Function<Person,?>` under the hood). Why: stable, readable multi‑key ordering.

### Solution 20 — Finite Slice of Infinite Stream
```java
// Fibonacci stream: [0,1,1,2,3,5,8,13,...]
Stream<long[]> fib = Stream.iterate(new long[]{0,1}, a -> new long[]{a[1], a[0]+a[1]});
long sumEvenFirst10 = fib
    .map(a -> a[0])
    .filter(n -> n % 2 == 0)
    .limit(10)
    .mapToLong(Long::longValue)
    .sum();
```
Functional interfaces: `UnaryOperator<long[]>` (iterate), `Function<long[],Long>` (map), `Predicate<Long>` (filter), `ToLongFunction<Long>` (mapToLong). Why: generate infinite sequence, slice finitely with limit.

### Solution 21 — Loop Fusion vs Materialization
```java
// (A) Materialization
List<Integer> evens = numbers.stream().filter(n -> n % 2 == 0).collect(Collectors.toList());
List<Integer> doubled = evens.stream().map(n -> n * 2).collect(Collectors.toList());
int sumA = doubled.stream().mapToInt(Integer::intValue).sum();

// (B) Fused single pass
int sumB = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * 2)
    .mapToInt(Integer::intValue)
    .sum();
```
Functional interfaces: `Predicate<Integer>` (filter), `Function<Integer,Integer>` (map), `ToIntFunction<Integer>` (mapToInt). Why: (B) avoids intermediate collections and multiple traversals.

### Solution 22 — reduce to Longest Word
```java
Optional<String> longest = words.stream()
    .reduce((a, b) -> {
        if (a.length() > b.length()) return a;
        if (b.length() > a.length()) return b;
        return (a.compareTo(b) <= 0) ? a : b; // tie: lexicographically smaller
    });
```
Functional interfaces: `BinaryOperator<String>` (reduce accumulator). Why: combine two candidates into one based on custom rule.

### Solution 23 — Custom Merge in toMap
```java
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        StreamsScenarioLab.Person::name,
        StreamsScenarioLab.Person::age,
        Integer::max
    ));
```
Functional interfaces: `Function<Person,String>` (key mapper), `Function<Person,Integer>` (value mapper), `BinaryOperator<Integer>` (merge). Why: resolve duplicate keys by keeping max age.

### Solution 24 — Downstream Collector with Ordering and Limit
```java
Map<String, List<String>> first2UpperPerCity = people.stream()
    .collect(Collectors.groupingBy(
        StreamsScenarioLab.Person::city,
        Collectors.collectingAndThen(
            Collectors.mapping(p -> p.name().toUpperCase(), Collectors.toCollection(TreeSet::new)),
            set -> set.stream().limit(2).collect(Collectors.toList())
        )
    ));
```
Functional interfaces: `Function<Person,String>` (classifier), `Function<Person,String>` (mapping to upper), plus `Supplier<TreeSet<String>>` for the collection factory. Why: ordered set per group, then limit.

### Solution 25 — Parallel Streams and Short‑Circuit Caveat
```java
for (int i = 0; i < 5; i++) {
    Optional<Integer> any = numbers.parallelStream()
        .filter(n -> n > 8)
        .findAny();
    System.out.println(any.orElse(-1));
}
```
Functional interfaces: `Predicate<Integer>` (filter). Why: show non‑deterministic selection with `findAny` in parallel.

### Solution 26 — Windowed Computation (Moving Average)
```java
List<Integer> series = List.of(3, 5, 7, 6, 8, 10, 9);
int w = 3;
List<Double> movingAvg = IntStream.rangeClosed(0, series.size() - w)
    .mapToDouble(i -> (series.get(i) + series.get(i+1) + series.get(i+2)) / 3.0)
    .boxed()
    .collect(Collectors.toList());
```
Functional interfaces (primitive): `IntToDoubleFunction` (mapToDouble). Why: compute arithmetic on windows using indices, then box to List.

---

## Cheat Sheet — Short‑Circuiting and Loop Fusion

Short‑circuiting terminal operations: findFirst, findAny, anyMatch, allMatch, noneMatch, limit. These can stop traversal early when the result is determined.
Loop fusion: A stream pipeline processes each element through all stages before moving to the next element—avoid materializing intermediate results unless necessary.
Ordering: sorted imposes encounter order; findFirst respects it; findAny may not (especially in parallel).
Performance tips: Place cheap, selective filters early; use primitive streams for numeric reductions; avoid peek in production logic.

---

## How to Practice

Implement each scenario in a separate method (for example, `solveScenario1()`) and call from `main`.
Add quick assertions using `System.out.println` or a simple assertion utility.
Convert them into JUnit tests when comfortable.

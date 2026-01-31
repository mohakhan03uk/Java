# Java Streams Operations (Java 8 → 21): Intermediate & Terminal

This cheat sheet summarizes the **core Stream API operations** from Java 8 through 21.
It lists each operation’s type (intermediate/terminal), return type, functional interface(s), canonical
function descriptor(s), and practical guidance on when to use each.

> **Legend (function descriptors):**
> - `T`: stream element type
> - `R`: result element type
> - `U`, `A`: accumulator/container types
> - `int/long/double`: primitive counterparts
> - `Optional<T>` / `OptionalInt` / `OptionalLong` / `OptionalDouble`: optional wrappers

---

## Intermediate Operations

| Operation & Type | Method Signature | Return Type | Interface Used | Descriptor & Usage | Example |
|---|---|---|---|---|---|
| **filter** (Intermediate) | `Stream<T> filter(Predicate<? super T> predicate)` | Stream<T> | Predicate<T> | **T → boolean**<br>Filter elements based on condition. | `list.stream().filter(x -> x > 10)` |
| **map** (Intermediate) | `Stream<R> map(Function<? super T,? extends R> mapper)` | Stream<R> | Function<T,R> | **T → R**<br>Transform elements. | `list.stream().map(String::length)` |
| **flatMap** (Intermediate) | `Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)` | Stream<R> | Function<T,Stream<R>> | **T → Stream<R>**<br>Flatten nested collections. | `list.stream().flatMap(List::stream)` |
| **mapMulti** (Intermediate) | `Stream<R> mapMulti(BiConsumer<T,Consumer<R>> mapper)` | Stream<R> | BiConsumer<T,Consumer<R>> | **(T, Consumer<R>) → void**<br>Efficient flattening. | `stream.mapMulti((n,c)->{if(n%2==0)c.accept(n);})` |
| **distinct** (Intermediate, stateful) | `Stream<T> distinct()` | Stream<T> | — | Remove duplicates. | `list.stream().distinct()` |
| **sorted** (Intermediate, stateful) | `Stream<T> sorted()` / `Stream<T> sorted(Comparator)` | Stream<T> | Comparator<T> | **(T,T)→int**<br>Sort elements. | `list.stream().sorted()` |
| **peek** (Intermediate) | `Stream<T> peek(Consumer<? super T> action)` | Stream<T> | Consumer<T> | **T→void**<br>Debug/log. | `stream.peek(System.out::println)` |
| **limit** (Intermediate, SC) | `Stream<T> limit(long maxSize)` | Stream<T> | — | Take first N. | `stream.limit(5)` |
| **skip** (Intermediate) | `Stream<T> skip(long n)` | Stream<T> | — | Skip N. | `stream.skip(3)` |
| **takeWhile** | `Stream<T> takeWhile(Predicate)` | Stream<T> | Predicate<T> | **T→boolean**<br>Take while true. | `stream.takeWhile(x<10)` |
| **dropWhile** | `Stream<T> dropWhile(Predicate)` | Stream<T> | Predicate<T> | Drop while true. | `stream.dropWhile(x<10)` |
| **unordered** | `Stream<T> unordered()` | Stream<T> | — | Remove ordering constraint. | `stream.unordered()` |
| **onClose** | `Stream<T> onClose(Runnable closeHandler)` | Stream<T> | Runnable | Cleanup on close. | `stream.onClose(() -> log)` |
| **sequential** | `Stream<T> sequential()` | Stream<T> | — | Switch to sequential. | `stream.sequential()` |
| **parallel** | `Stream<T> parallel()` | Stream<T> | — | Switch to parallel. | `stream.parallel()` |
| **mapToInt/Long/Double** | `IntStream mapToInt(ToIntFunction)` etc. | Primitive streams | ToXFunction<T> | Numeric mapping. | `stream.mapToInt(String::length)` |
| **flatMapToInt/Long/Double** | `IntStream flatMapToInt(Function)` | Primitive streams | Function<T,IntStream> | Flatten primitives. | `stream.flatMapToInt(x->x.stream())` |
| **boxed** | `Stream<T> boxed()` | Stream<T> | — | Primitive → boxed. | `IntStream.range(1,5).boxed()` |

---

## Terminal Operations

| Operation & Type | Method Signature | Return Type | Interface Used | Descriptor & Usage | Example |
|---|---|---|---|---|---|
| **forEach** | `void forEach(Consumer)` | void | Consumer<T> | **T→void**<br>Side effects. | `stream.forEach(System.out::println)` |
| **forEachOrdered** | `void forEachOrdered(Consumer)` | void | Consumer<T> | Ordered side effects. | `stream.forEachOrdered(...)` |
| **toArray** | `Object[] toArray()` / `<A>A[] toArray(IntFunction<A[]>)` | Array | IntFunction | Convert to array. | `stream.toArray()` |
| **reduce(BinaryOperator)** | `Optional<T> reduce(BinaryOperator)` | Optional<T> | BinaryOperator<T> | Fold without identity. | `stream.reduce((a,b)->a+b)` |
| **reduce(id, op)** | `T reduce(T identity, BinaryOperator)` | T | BinaryOperator<T> | Fold with identity. | `stream.reduce(0, a+b)` |
| **reduce(id, acc, combiner)** | `U reduce(U id, BiFunction, BinaryOperator)` | U | BiFunction, BinaryOperator | Parallel-friendly reduction. | `stream.reduce(0,sum,combine)` |
| **collect(collector)** | `R collect(Collector)` | R | Collector | Collect into list/map/group. | `stream.collect(toList())` |
| **collect(s,a,c)** | `A collect(Supplier, BiConsumer, BiConsumer)` | A | Supplier, BiConsumer | Manual collection. | `stream.collect(ArrayList::new, add, addAll)` |
| **min/max** | `Optional<T> min(Comparator)` | Optional<T> | Comparator | Smallest/largest. | `stream.min(cmp)` |
| **count** | `long count()` | long | — | Count elements. | `stream.count()` |
| **anyMatch** | `boolean anyMatch(Predicate)` | boolean | Predicate | SC: any matches. | `stream.anyMatch(x>10)` |
| **allMatch** | `boolean allMatch(Predicate)` | boolean | Predicate | SC: all match. | `stream.allMatch(x>10)` |
| **noneMatch** | `boolean noneMatch(Predicate)` | boolean | Predicate | SC: none match. | `stream.noneMatch(x>10)` |
| **findFirst** | `Optional<T> findFirst()` | Optional<T> | — | First element. | `stream.findFirst()` |
| **findAny** | `Optional<T> findAny()` | Optional<T> | — | Any element. | `stream.findAny()` |
| **iterator** | `Iterator<T> iterator()` | Iterator<T> | — | Convert to iterator. | `stream.iterator()` |
| **spliterator** | `Spliterator<T> spliterator()` | Spliterator<T> | — | Split traversal. | `stream.spliterator()` |
| **toList** | `List<T> toList()` | List<T> | — | Unmodifiable list. | `stream.toList()` |

---

## Notes & Guidance

### Choosing between `map`, `flatMap`, and `mapMulti`
- Use **`map`** when each element becomes exactly one element.
- Use **`flatMap`** when each element becomes **a stream** of zero/one/many elements and you need flattening.
- Use **`mapMulti`** when you can push zero/one/many results directly to a provided `Consumer` without constructing intermediate streams
  (often more efficient than `flatMap`).

### Short‑circuiting & Statefulness
- Short‑circuiting: `limit`, `takeWhile` (intermediate) and `anyMatch` / `allMatch` / `noneMatch` / `findFirst` / `findAny` (terminal) can finish early.
- Stateful: `sorted`, `distinct` may buffer; be cautious with very large or unbounded streams.

### Primitive Streams
- Prefer `mapToInt/Long/Double`, `flatMapTo*`, and primitive terminals like `sum`, `average`, `min`, `max`, `summaryStatistics` to avoid boxing.
- Use `boxed()` only when you truly need object semantics/collectors.

### Parallel Streams
- Only consider parallel when data size is large, operations are CPU‑bound, and work is easily partitionable.
- Ensure accumulators/reducers are **associative**, **side‑effect‑free**, and have correct **combiners**.
- Prefer `forEachOrdered` if you must preserve encounter order (with potential performance tradeoffs).

---

## Quick Reference: Functional Interfaces & Descriptors

| Interface | Descriptor |
|---|---|
| `Predicate<T>` | `T -> boolean` |
| `Function<T,R>` | `T -> R` |
| `BiFunction<U,T,R>` | `(U, T) -> R` |
| `Consumer<T>` | `T -> void` |
| `BiConsumer<A,T>` | `(A, T) -> void` |
| `Supplier<A>` | `() -> A` |
| `Comparator<T>` | `(T, T) -> int` |
| `BinaryOperator<T>` | `(T, T) -> T` (alias of `BiFunction<T,T,T>`) |
| `IntFunction<A[]>` | `int -> A[]` |
| `Runnable` | `() -> void` |

---

## Frequently Used Primitive Terminals (for completeness)

| Primitive Stream | Terminals |
|---|---|
| `IntStream` | `sum`, `average` → `OptionalDouble`, `min`/`max` → `OptionalInt`, `summaryStatistics`, `boxed` |
| `LongStream` | `sum`, `average` → `OptionalDouble`, `min`/`max` → `OptionalLong`, `summaryStatistics`, `boxed` |
| `DoubleStream` | `sum`, `average` → `OptionalDouble`, `min`/`max` → `OptionalDouble`, `summaryStatistics`, `boxed` |

---

### Practical Tips
- Prefer `stream().toList()` over `collect(Collectors.toList())` when you just need a list.
- Use `Collectors.groupingBy`, `partitioningBy`, `mapping`, `flatMapping`, `teeing` (Collectors) for rich aggregations—paired with `collect`.
- Be mindful of `peek`—great for debugging but avoid mixing with side‑effects in production logic.
- For large data, watch for stateful ops (`sorted`, `distinct`) and consider pre‑sorting or using specialized data structures.

---

**End of file.**

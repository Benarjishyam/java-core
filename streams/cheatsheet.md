# The Cheat Sheet: When to Use Which Category

| If your goal is to... | Use this built-in Collector | Equivalent SQL concept |
|---|---|---|
| Combine everything into a single value (Count, Sum, Avg, Max) | `counting()`, `summingDouble()`, `averagingInt()`, `maxBy()`, `summarizingDouble()` | `COUNT()`, `SUM()`, `AVG()`, `MAX()` |
| Group data into a Map (by a specific property/key) | `groupingBy(classifier)` | `GROUP BY column_name` |
| Split data into exactly two groups (True vs. False) | `partitioningBy(predicate)` | `WHERE condition` |
| Change the container type (to a specific List/Set/String) | `toList()`, `toSet()`, `joining()`, `toCollection()` | N/A (Java-specific) |

## 1. The Summary / Math Collectors

Use these when you want to reduce an entire stream down to a single primitive value (a number, an average, or a text summary).

- **`counting()`:** Use this when you need to know how many items match a criterion.
- **`averagingDouble()` / `averagingInt()`:** Use these when you need a mathematical mean.
- **`summarizingDouble()` / `summarizingInt()`:** 💡 **Pro Tip:** If you need the count, sum, min, max, and average all at once, don't run multiple streams. Use `summarizingDouble()`. It returns a single `DoubleSummaryStatistics` object containing all five metrics.

## 2. The Multi-Level Collectors (`groupingBy`)

Use `groupingBy` when you want your final output to be a `Map<Key, Value>`. Think of `groupingBy` as a two-step factory:

1. **Step 1 (The Key):** Pass a function to decide how to categorize the rows (e.g., `Employee::getDepartment`).
2. **Step 2 (The Value):** If you provide only the key, Java defaults to giving you a `List` of items matching that key (`Map<String, List<Employee>>`).

### How to Know When to Chain "Downstream" Collectors

If you don't want a `List` as the Map's value, you pass a second collector inside `groupingBy`.

- **"I want a map of departments, but I want the count of employees, not the list."**

  ➡️ Use `groupingBy(Employee::getDepartment, Collectors.counting())`

- **"I want a map of departments, but I want the average salary, not the list."**

  ➡️ Use `groupingBy(Employee::getDepartment, Collectors.averagingDouble(Employee::getSalary))`

## 3. The Boolean Filter (`partitioningBy`)

Use `partitioningBy` when you are splitting a dataset based on a strict Yes/No or True/False condition. It returns a `Map<Boolean, List<T>>` when used with the default downstream collector.

- **When to use:** Tracking "Active vs. Inactive", "Passed vs. Failed", or "Joined after 2020 vs. Joined before 2020".
- **Why not just use `filter()`?** `filter()` throws away the data that doesn't match. `partitioningBy()` keeps both sides of the coin so you can analyze both groups side by side.

## How to Practice for Interviews

The easiest way to identify which collector to use during an interview is to look at the requested return type of the problem:

1. **Does the interviewer want a single Boolean, String, or numeric value?** Look at Category 1 (`counting`, `joining`, `maxBy`).
2. **Does the interviewer want a Map categorized by a property?** Look at Category 2 (`groupingBy`).
3. **Does the interviewer want a Map categorized by a condition?** Look at Category 3 (`partitioningBy`).

# 📋 Java Streams: Bucket 1 & Sorting Cheat Sheet

## 1. Extreme Values: `Collectors.maxBy()` / `minBy()`

- **What it takes:** A `Comparator<T>`.
- **What it returns:** An `Optional<T>`.

### How to Build the Comparator

- `Comparator.comparingInt(Employee::getAge)` ➡️ Efficient sorting for primitive `int`.
- `Comparator.comparingLong(Employee::getId)` ➡️ Efficient sorting for primitive `long`.
- `Comparator.comparingDouble(Employee::getSalary)` ➡️ Efficient sorting for primitive `double`.
- `Comparator.comparing(Employee::getName)` ➡️ Sorting objects using their natural order (e.g., `String`).
- `Comparator.comparing(Employee::getName, String.CASE_INSENSITIVE_ORDER)` ➡️ Sorting objects using a custom comparator rule.
- `comparator.reversed()` ➡️ Inverts the order of any comparator you just built.

### Code Example

```java
Optional<Employee> maxSal = empList.stream()
    .collect(Collectors.maxBy(
        Comparator.comparingDouble(Employee::getSalary)
    ));
```

## 2. Element Counting: `Collectors.counting()`

- **What it takes:** Nothing.
- **What it returns:** A `Long` representing the total element count in the stream.

### Code Example

```java
Long totalCount = empList.stream()
    .collect(Collectors.counting());
```

## 3. Mathematical Sums: `Collectors.summingX()`

- **What it takes:** A functional interface mapping the object to a primitive number (`ToIntFunction`, `ToLongFunction`, or `ToDoubleFunction`).
- **What it returns:** The wrapper object of the matching type (`Integer`, `Long`, or `Double`).

### Methods Available

- `Collectors.summingInt(Employee::getAge)`
- `Collectors.summingLong(Employee::getId)`
- `Collectors.summingDouble(Employee::getSalary)`

### Code Example

```java
Double totalPayroll = empList.stream()
    .collect(Collectors.summingDouble(Employee::getSalary));
```

## 4. Averages: `Collectors.averagingX()`

- **What it takes:** A functional interface mapping the object to a primitive number.
- **What it returns:** Always returns a `Double`, regardless of whether the source input was an `int`, `long`, or `double`.

### Methods Available

- `Collectors.averagingInt(Employee::getAge)`
- `Collectors.averagingLong(Employee::getId)`
- `Collectors.averagingDouble(Employee::getSalary)`

### Code Example

```java
Double avgAge = empList.stream()
    .collect(Collectors.averagingInt(Employee::getAge));
```

## 5. All-in-One Statistics: `Collectors.summarizingX()`

- **What it takes:** A functional interface mapping the object to a primitive number.
- **What it returns:** A specialized statistics carrier object:
  - `IntSummaryStatistics`
  - `LongSummaryStatistics`
  - `DoubleSummaryStatistics`

These objects provide the following methods:

- `getCount()`
- `getSum()`
- `getMin()`
- `getMax()`
- `getAverage()`

### Methods Available

- `Collectors.summarizingInt(Employee::getAge)`
- `Collectors.summarizingLong(Employee::getId)`
- `Collectors.summarizingDouble(Employee::getSalary)`

### Code Example

```java
DoubleSummaryStatistics stats = empList.stream()
    .collect(Collectors.summarizingDouble(Employee::getSalary));
```

## 6. In-Stream Order: `Stream.sorted()`

- **What it takes:** A `Comparator<T>`.
- **What it is:** An intermediate stream operation, not a terminal collector.
- **What it returns:** A new sorted `Stream<T>`.

### Code Example

```java
List<Employee> sortedByAge = empList.stream()
    .sorted(Comparator.comparingInt(Employee::getAge).reversed())
    .collect(Collectors.toList());
```

---

## Quick Revision Table

| Operation | Purpose | Return Type |
|---|---|---|
| `Collectors.maxBy()` | Find the maximum element | `Optional<T>` |
| `Collectors.minBy()` | Find the minimum element | `Optional<T>` |
| `Collectors.counting()` | Count stream elements | `Long` |
| `Collectors.summingInt()` | Calculate an integer sum | `Integer` |
| `Collectors.summingLong()` | Calculate a long sum | `Long` |
| `Collectors.summingDouble()` | Calculate a double sum | `Double` |
| `Collectors.averagingInt()` | Calculate the average of integers | `Double` |
| `Collectors.averagingLong()` | Calculate the average of longs | `Double` |
| `Collectors.averagingDouble()` | Calculate the average of doubles | `Double` |
| `Collectors.summarizingInt()` | Get integer statistics | `IntSummaryStatistics` |
| `Collectors.summarizingLong()` | Get long statistics | `LongSummaryStatistics` |
| `Collectors.summarizingDouble()` | Get double statistics | `DoubleSummaryStatistics` |
| `Stream.sorted()` | Sort stream elements | `Stream<T>` |

## Interview Tips

1. **Need the highest or lowest element?** Use `maxBy()` or `minBy()`.
2. **Need only the total number of elements?** Use `counting()`.
3. **Need the sum of a numeric property?** Use `summingInt()`, `summingLong()`, or `summingDouble()`.
4. **Need the average of a numeric property?** Use `averagingInt()`, `averagingLong()`, or `averagingDouble()`.
5. **Need count, sum, min, max, and average together?** Use `summarizingInt()`, `summarizingLong()`, or `summarizingDouble()`.
6. **Need the entire stream sorted?** Use `Stream.sorted()` with an appropriate comparator.
7. **Need descending order?** Chain `.reversed()` onto the comparator.

# 📋 Java Streams: Bucket 2 (groupingBy) Cheat Sheet

## 1. Single-Argument Grouping: `Collectors.groupingBy()`

- **What it takes:** A classifier function `Function<T, K>` to group elements by a key.
- **What it returns:** A `Map<K, List<T>>`, where each key maps to a list of elements matching that key.

### Code Example

```java
// Group employees by department name
Map<String, List<Employee>> empByDept = empList.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

## 2. Two-Argument Grouping: `Collectors.groupingBy()` with Downstream Collectors

- **What it takes:**
  1. A classifier function `Function<T, K>`.
  2. A downstream `Collector` to process the elements inside each group.
- **What it returns:** A `Map<K, Downstream_Result_Type>`, where each key maps to the result produced by the downstream collector.

### Common Downstream Combinations

#### A. Element Counting: `Collectors.counting()`

- **What it takes:** Nothing.
- **What it returns:** A `Long` representing the number of elements in each group.
- **Resulting Map Type:** `Map<K, Long>`.

##### Code Example

```java
Map<String, Long> countByDept = empList.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));
```

#### B. Averages: `Collectors.averagingDouble()`

- **What it takes:** A `ToDoubleFunction<T>` mapping each element to a `double` value.
- **What it returns:** A `Double` representing the average of the mapped values in each group.
- **Resulting Map Type:** `Map<K, Double>`.

##### Code Example

```java
Map<String, Double> avgSalaryByDept = empList.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

#### C. Mathematical Sums: `Collectors.summingInt()`

- **What it takes:** A `ToIntFunction<T>` mapping each element to an `int` value.
- **What it returns:** An `Integer` representing the sum of the mapped values in each group.
- **Resulting Map Type:** `Map<K, Integer>`.

##### Code Example

```java
Map<String, Integer> totalAgeByDept = empList.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summingInt(Employee::getAge)
    ));
```

#### D. Extreme Values: `Collectors.maxBy()` / `Collectors.minBy()`

- **What it takes:** A `Comparator<T>`.
- **What it returns:** An `Optional<T>` containing the maximum or minimum element in each group, if present.
- **Resulting Map Type:** `Map<K, Optional<T>>`.

##### Code Example

```java
Map<String, Optional<Employee>> maxSalaryByDept = empList.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(
            Comparator.comparingDouble(Employee::getSalary)
        )
    ));
```

## 3. Three-Argument Grouping: `Collectors.groupingBy()` with a Custom Map Type

- **What it takes:**
  1. A classifier function `Function<T, K>`.
  2. A map factory function (e.g., `TreeMap::new` if you want sorted keys).
  3. A downstream `Collector`.
- **What it returns:** A specific map implementation of your choice, with values determined by the downstream collector.

### Code Example

```java
// Keep departments sorted alphabetically by using a TreeMap
TreeMap<String, Long> sortedCountByDept = empList.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        TreeMap::new,
        Collectors.counting()
    ));
```

## 4. Special Chained Collector: `Collectors.mapping()` Inside `groupingBy()`

- **What it takes:** A mapping function to transform each element into a different value, followed by a downstream collector.
- **When to use:** When you want a list of a specific field (like names) instead of the full `Employee` objects inside your map values.
- **What it returns:** A `Map<K, List<R>>` when used with `Collectors.toList()`, where `R` is the mapped value type.

### Code Example

```java
// Returns a Map<String, List<String>> containing department names mapped to employee names
Map<String, List<String>> namesByDept = empList.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(
            Employee::getName,
            Collectors.toList()
        )
    ));
```

---

## Quick Revision Table

| Operation | Purpose | Return Type |
|---|---|---|
| `Collectors.groupingBy(classifier)` | Group elements by a key | `Map<K, List<T>>` |
| `Collectors.groupingBy(classifier, counting())` | Count elements in each group | `Map<K, Long>` |
| `Collectors.groupingBy(classifier, averagingDouble())` | Calculate the average in each group | `Map<K, Double>` |
| `Collectors.groupingBy(classifier, summingInt())` | Calculate the integer sum in each group | `Map<K, Integer>` |
| `Collectors.groupingBy(classifier, maxBy())` | Find the maximum element in each group | `Map<K, Optional<T>>` |
| `Collectors.groupingBy(classifier, minBy())` | Find the minimum element in each group | `Map<K, Optional<T>>` |
| `Collectors.groupingBy(classifier, mapFactory, downstream)` | Group elements using a custom map implementation | `Map<K, D>` |
| `Collectors.mapping()` | Transform elements before downstream collection | Depends on the downstream collector |

## Interview Tips

1. **Need to group elements by a property?** Use `Collectors.groupingBy()` with a classifier function.
2. **Need a list of elements for each group?** Use the single-argument `groupingBy()`, which collects values into lists by default.
3. **Need the count of elements in each group?** Use `groupingBy()` with `Collectors.counting()`.
4. **Need the average of a numeric property in each group?** Use `groupingBy()` with `Collectors.averagingDouble()`.
5. **Need the sum of a numeric property in each group?** Use `groupingBy()` with `Collectors.summingInt()`, `Collectors.summingLong()`, or `Collectors.summingDouble()`.
6. **Need the maximum or minimum element in each group?** Use `groupingBy()` with `Collectors.maxBy()` or `Collectors.minBy()`.
7. **Need sorted keys in the resulting map?** Use the three-argument `groupingBy()` with `TreeMap::new`.
8. **Need only a specific field from each element?** Use `Collectors.mapping()` as a downstream collector.
9. **Need custom processing inside each group?** Choose the appropriate downstream collector.
10. **Remember:** The classifier determines the map keys, while the downstream collector determines the values associated with those keys.

# 📋 Java Streams: Bucket 3 (partitioningBy) Cheat Sheet

`Collectors.partitioningBy()` always transforms your stream into a `Map<Boolean, V>`. It evaluates every element against a `Predicate` (a condition that returns `true` or `false`).

## 1. Single-Argument Partitioning: `Collectors.partitioningBy()`

- **What it takes:** A `Predicate<T>` condition.
- **What it returns:** A `Map<Boolean, List<T>>`, where:
  - `true` maps to a list of elements that satisfy the condition.
  - `false` maps to a list of elements that do not satisfy the condition.

### Code Example

```java
// Partition employees into: Joined after 2020 (true) vs Joined in/before 2020 (false)
Map<Boolean, List<Employee>> splitByYear = empList.stream()
    .collect(Collectors.partitioningBy(e -> e.getYearOfJoining() > 2020));

// Accessing the buckets:
List<Employee> joinedAfter2020 = splitByYear.get(true);
List<Employee> joinedBeforeOrIn2020 = splitByYear.get(false);
```

## 2. Two-Argument Partitioning: `Collectors.partitioningBy()` with Downstream Collectors

- **What it takes:**
  1. A `Predicate<T>` condition.
  2. A secondary downstream `Collector` to process the elements inside the `true` and `false` buckets.

- **What it returns:** A `Map<Boolean, Downstream_Result_Type>`, where each Boolean key maps to the result produced by the downstream collector.

### Common Downstream Combinations

### A. Element Counting: `Collectors.counting()`

- **What it takes:** Nothing.
- **What it returns:** A `Long` representing the number of elements in each bucket.
- **Resulting Map Type:** `Map<Boolean, Long>`.

#### Code Example

```java
Map<Boolean, Long> countBySeniority = empList.stream()
    .collect(Collectors.partitioningBy(
        e -> e.getYearOfJoining() > 2020,
        Collectors.counting()
    ));
```

### B. Averages: `Collectors.averagingDouble()`

- **What it takes:** A `ToDoubleFunction<T>` mapping each element to a `double` value.
- **What it returns:** A `Double` representing the average of the mapped values in each bucket.
- **Resulting Map Type:** `Map<Boolean, Double>`.

#### Code Example

```java
// Average salary of employees who joined after 2020 vs those who didn't
Map<Boolean, Double> avgSalarySplit = empList.stream()
    .collect(Collectors.partitioningBy(
        e -> e.getYearOfJoining() > 2020,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

### C. Extreme Values: `Collectors.maxBy()` / `Collectors.minBy()`

- **What it takes:** A `Comparator<T>`.
- **What it returns:** An `Optional<T>` containing the maximum or minimum element in each bucket, if present.
- **Resulting Map Type:** `Map<Boolean, Optional<T>>`.

#### Code Example

```java
// Highest paid employee among those who joined after 2020 vs those who didn't
Map<Boolean, Optional<Employee>> maxSalarySplit = empList.stream()
    .collect(Collectors.partitioningBy(
        e -> e.getYearOfJoining() > 2020,
        Collectors.maxBy(
            Comparator.comparingDouble(Employee::getSalary)
        )
    ));
```

---

## 💡 Interview Catch: `groupingBy()` vs `partitioningBy()`

Interviewers often ask:

"Why use `partitioningBy(e -> e.getAge() > 30)` when `groupingBy(e -> e.getAge() > 30)` produces the same map structure?"

### 1. Performance

`partitioningBy()` is designed specifically for two buckets (`true` and `false`). It avoids the need for a general-purpose classification map and can be more efficient for Boolean partitioning. However, actual performance depends on the workload and implementation.

### 2. Guaranteed Keys

`partitioningBy()` always returns a map containing both Boolean keys, even if one bucket is empty.

For example:

```java
{
    true = [],
    false = [...All Employees]
}
```

If no employees satisfy the condition, the `true` bucket is still present as an empty list.

By contrast, `groupingBy()` only includes keys for which at least one element was classified. If no employees satisfy the condition, the `true` key will be absent.

---

## Quick Revision Table

| Operation | Purpose | Return Type |
|---|---|---|
| `Collectors.partitioningBy(predicate)` | Split elements into two lists based on a condition | `Map<Boolean, List<T>>` |
| `Collectors.partitioningBy(predicate, counting())` | Count elements in both buckets | `Map<Boolean, Long>` |
| `Collectors.partitioningBy(predicate, averagingDouble())` | Calculate averages for both buckets | `Map<Boolean, Double>` |
| `Collectors.partitioningBy(predicate, maxBy())` | Find the maximum element in both buckets | `Map<Boolean, Optional<T>>` |
| `Collectors.partitioningBy(predicate, minBy())` | Find the minimum element in both buckets | `Map<Boolean, Optional<T>>` |

## Interview Tips

1. **Need to split elements into two groups based on a condition?** Use `Collectors.partitioningBy()`.
2. **Need elements that satisfy the condition?** Access the `true` bucket using `.get(true)`.
3. **Need elements that do not satisfy the condition?** Access the `false` bucket using `.get(false)`.
4. **Need the count of elements in both buckets?** Use `partitioningBy()` with `Collectors.counting()`.
5. **Need the average of a numeric property in both buckets?** Use `partitioningBy()` with `Collectors.averagingDouble()`.
6. **Need the highest- or lowest-valued element in both buckets?** Use `partitioningBy()` with `Collectors.maxBy()` or `Collectors.minBy()`.
7. **Need to group by multiple possible values rather than just `true` and `false`?** Use `Collectors.groupingBy()`.
8. **Remember:** `partitioningBy()` always includes both Boolean keys, even when one bucket contains no elements.

# Spark Fundamentals: Repartition and Coalesce

## Key Concepts
- **Repartitioning** and **Coalesce** are DataFrame transformations for managing partition counts in Spark.
- Proper partitioning is crucial for parallelism, resource utilization, and avoiding data skew or out-of-memory (OOM) errors.

## Repartition
- Used to **increase** the number of partitions or repartition on specific columns.
- Methods:
  - `repartition(numPartitions)`
  - `repartition(numPartitions, columns...)`
  - `repartitionByRange(numPartitions, columns...)`
- **repartition()** uses a hash function (or range for `repartitionByRange`) and always triggers a **shuffle** (wide dependency).
- **Uniform partitioning**: When repartitioning by number, partitions are evenly sized.
- **Column-based repartitioning**: May result in uneven partition sizes; controlled by `spark.sql.shuffle.partitions` unless overridden by passing numPartitions.
- **repartitionByRange()** uses data sampling, so partition ranges may vary between runs.
- **Expensive operation**: Causes shuffle/sort; avoid unless necessary.

### When to Use
- Reusing a DataFrame multiple times with filters on specific columns.
- Data is poorly distributed across cluster nodes.
- Existing partitions are large or skewed; need more uniform distribution.

## Coalesce
- Used to **reduce** the number of partitions **without shuffle**.
- Method: `coalesce(numPartitions)`
- Combines local partitions on the same worker node; does not move data between nodes.
- **No shuffle**: Efficient for reducing partition count.
- **Cannot increase** partition count; only reduces.
- Avoid using `repartition()` to reduce partitions (causes unnecessary shuffle).

### Caveats
- Drastically reducing partitions with coalesce can cause **skewed partitions** and potential OOM errors.
- Use with caution; avoid reducing to very few partitions.

## Comparison Table
| Method                      | Purpose                | Causes Shuffle | Uniform Partitions | Can Increase Partitions | Can Reduce Partitions |
|-----------------------------|------------------------|---------------|--------------------|------------------------|----------------------|
| repartition(numPartitions)  | Increase/redistribute  | Yes           | Yes                | Yes                    | Yes (but expensive)  |
| repartition(columns...)     | Redistribute by column | Yes           | No                 | Yes                    | Yes (but expensive)  |
| repartitionByRange()        | Range-based partition  | Yes           | No                 | Yes                    | Yes                  |
| coalesce()                  | Reduce partitions      | No            | No                 | No                     | Yes                  |

## Additional Notes
- Always test partitioning changes for performance and memory impact.
- Prefer `repartition()` for increasing partitions or redistributing data; use `coalesce()` for reducing partitions efficiently.
- Avoid drastic reductions with coalesce to prevent skew and OOM.
- Partitioning strategy affects shuffle, parallelism, and resource usage.

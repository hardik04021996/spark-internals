# Spark Fundamentals: DataFrame Hints

## Key Concepts
- **Hints** in Spark allow users to suggest strategies for partitioning and joining DataFrames or Spark SQL queries.
- Hints are advisory; Spark may ignore them depending on execution context and configuration.

## Partitioning Hints
- **COALESCE**: Reduces the number of partitions to a specified value. Syntax: `COALESCE(n)`
- **REPARTITION**: Changes the number of partitions, optionally by columns. Syntax: `REPARTITION(n, columns...)`
- **REPARTITION_BY_RANGE**: Partitions data by range of column values. Syntax: `REPARTITION_BY_RANGE(n, columns...)`
- **REBALANCE** (Spark 3.0+): Attempts to balance partition sizes, splitting skewed partitions. Syntax: `REBALANCE(columns...)`
  - Only effective if AQE (Adaptive Query Execution) is enabled.

## Join Hints
- **BROADCAST / BROADCASTJOIN / MAPJOIN**: Suggests using broadcast join for small tables.
- **MERGE**: Suggests sort-merge join.
- **SHUFFLE_HASH**: Suggests shuffle hash join.
- **SHUFFLE_REPLICATE_NL**: Suggests shuffle-and-replicate nested loop join.
- Spark prioritizes join hints in a specific order if multiple are present.

## How to Apply Hints
- **Spark SQL**: Use hints in SQL queries, e.g., `SELECT /*+ BROADCAST(table) */ ...`
- **DataFrame API**:
  - Use `DataFrame.hint()` method, e.g., `df.hint("broadcast")`
  - Use Spark SQL functions, e.g., `broadcast(df)`

## Example Usage
- Apply broadcast hint to a small DataFrame before joining with a large DataFrame to avoid shuffle sort.
- Apply coalesce hint to reduce the number of output partitions after a join.

## Execution Plan Impact
- Without hints: Spark may use default shuffle sort and sort-merge join, resulting in many partitions (e.g., 200).
- With hints: Spark may use broadcast join and coalesce output to fewer partitions (e.g., 5), improving efficiency.

**Example (PySpark):**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import broadcast

spark = SparkSession.builder.getOrCreate()

# Large DataFrame
df1 = spark.read.parquet("large_table.parquet")
# Small DataFrame
df2 = spark.read.parquet("small_table.parquet")

# Apply broadcast hint to small DataFrame and coalesce hint to join result
join_df = df1.join(broadcast(df2), "key").hint("coalesce", 5)

# Now join_df will use broadcast join and coalesce output to 5 partitions
```

## Comparison Table: Partitioning Hints
| Hint                 | Purpose                        | Parameters         | Shuffle | AQE Required |
|----------------------|--------------------------------|--------------------|---------|--------------|
| COALESCE             | Reduce partitions              | numPartitions      | No      | No           |
| REPARTITION          | Change partitions (by columns) | numPartitions, cols| Yes     | No           |
| REPARTITION_BY_RANGE | Range-based partitioning       | numPartitions, cols| Yes     | No           |
| REBALANCE            | Balance partition sizes        | cols               | Yes     | Yes          |

## Additional Notes
- Hints are best-effort; Spark may ignore them based on context or configuration.
- Use partitioning hints to optimize resource usage and output file sizes.
- Use join hints to improve join performance, especially with small tables.
- Always review execution plans to verify if hints are applied as expected.

# Spark Fundamentals: Dynamic Partition Pruning

## Key Concepts
- Dynamic Partition Pruning (DPP) is a feature introduced in Spark 3.0+ to optimize partitioned table queries.
- DPP is enabled by default (`spark.sql.optimizer.dynamicPartitionPruning.enabled=true`).
- DPP injects filter conditions from dimension tables into fact table scans, reducing data read and improving performance.

## Architecture Overview
- Traditional partition pruning only works if the filter is directly on the partitioned column of the fact table.
- In star-schema joins (fact + dimension), filters on the dimension table do not automatically prune partitions in the fact table.
- DPP dynamically pushes down filters from the dimension table to the fact table during query execution.
- DPP is most effective when:
  - There is a fact table (large, partitioned) and a dimension table (small).
  - The fact table is partitioned on the join column.
  - The dimension table is broadcasted (usually <10MB, or forced with `broadcast()`).

## Components
- **Predicate Pushdown**: Applies filter conditions as early as possible in the scan step.
- **Partition Pruning**: Reads only relevant partitions of a partitioned table based on filter conditions.
- **Dynamic Partition Pruning**: Injects subqueries from the dimension table into the fact table scan, enabling partition pruning even when the filter is on the dimension table.

## Tables
| Feature                | Description                                                      |
|------------------------|------------------------------------------------------------------|
| Predicate Pushdown     | Applies filters early in scan step                                |
| Partition Pruning      | Reads only relevant partitions based on direct filter             |
| Dynamic Partition Pruning | Pushes filters from dimension to fact table for pruning        |
| Broadcast Table        | Small dimension table sent to all executors for join              |
| Fact Table             | Large, partitioned table (e.g., Orders)                           |
| Dimension Table        | Small table (e.g., Dates)                                         |

## Additional Notes
- DPP is automatic if the fact table is partitioned, the dimension table is broadcasted, and the join is on the partition column.
- DPP reduces I/O and speeds up queries by avoiding unnecessary partition scans.
- If the dimension table is not broadcasted, DPP may not be applied.
- DPP is especially useful in star-schema and data warehouse scenarios.
- You can disable DPP with `spark.sql.optimizer.dynamicPartitionPruning.enabled=false` if needed.

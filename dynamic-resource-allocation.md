# Spark Fundamentals: Dynamic Resource Allocation

## Key Concepts
- **Dynamic Resource Allocation** allows Spark applications to request and release executor resources dynamically based on workload needs.
- Improves resource utilization in shared clusters by scaling resources up or down as stages require.
- Disabled by default; must be enabled via configuration.

## Resource Allocation Strategies
- **Static Allocation**
  - Executors are requested at application start and held for the entire duration.
  - Can lead to inefficient resource usage if stages have varying parallelism needs.
- **Dynamic Allocation**
  - Executors are acquired and released as needed during application execution.
  - Spark monitors task demand and executor idleness to adjust resources.

## How Dynamic Allocation Works
- Spark requests more executors when there are pending tasks and not enough slots.
- Spark releases idle executors after a configurable timeout period.
- The cluster manager (e.g., YARN, Kubernetes) provides and manages the actual resources.

## Configuration Parameters
| Config Key                                 | Default | Description                                                      |
|--------------------------------------------|---------|------------------------------------------------------------------|
| spark.dynamicAllocation.enabled            | false   | Enable/disable dynamic allocation                                |
| spark.dynamicAllocation.idleTimeout        | 60s     | Time before idle executor is released                            |
| spark.dynamicAllocation.executorAllocationRatio | 1.0 | Ratio of tasks to executors for allocation                       |
| spark.dynamicAllocation.minExecutors       | 0       | Minimum number of executors                                      |
| spark.dynamicAllocation.maxExecutors       | (unset) | Maximum number of executors                                      |
| spark.dynamicAllocation.schedulerBacklogTimeout | 1s  | Time before requesting more executors for pending tasks           |

## Example: Enabling Dynamic Allocation
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .config("spark.dynamicAllocation.enabled", "true") \
    .config("spark.dynamicAllocation.idleTimeout", "60s") \
    .config("spark.dynamicAllocation.schedulerBacklogTimeout", "1s") \
    .getOrCreate()
```

## Comparison Table: Static vs Dynamic Allocation
| Strategy         | Resource Usage | Scalability | Efficiency | Use Case                |
|------------------|---------------|-------------|------------|-------------------------|
| Static           | Fixed         | Limited     | May waste  | Simple, predictable jobs|
| Dynamic          | Adaptive      | High        | Optimized  | Shared, variable jobs   |

## Additional Notes
- Dynamic allocation is recommended for shared clusters and variable workloads.
- Executors are released and acquired without cluster manager intervention; Spark handles requests.
- Proper configuration is essential for optimal performance and resource savings.
- Monitor Spark UI to observe executor allocation and release during job execution.

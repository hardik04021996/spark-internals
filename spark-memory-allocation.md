# Spark Fundamentals: Memory Allocation

## Key Concepts
- Spark containers (driver/executor) have two main memory portions: JVM heap memory and overhead (non-JVM) memory.
- Memory allocation is critical for avoiding out-of-memory (OOM) errors, especially overhead memory.
- Overhead memory is used for shuffle exchange, network buffers, and non-JVM processes (e.g., Python workers).

## Memory Allocation for Driver
- Configured via `spark.driver.memory` (JVM heap) and `spark.driver.memoryOverhead` (container overhead).
- YARN allocates JVM heap + overhead (default overhead: max of 10% of heap or 384MB).
- Driver JVM uses only heap; overhead is reserved for container and non-JVM processes.
- Exceeding either limit causes OOM errors.

## Memory Allocation for Executor
- Configured via:
  - `spark.executor.memory` (JVM heap)
  - `spark.executor.memoryOverhead` (container overhead)
  - `spark.memory.offHeap.size` (off-heap)
  - `spark.executor.pyspark.memory` (PySpark worker)
- Total executor container memory = sum of all above.
- YARN allocates container based on requested total; physical node limits apply (`yarn.scheduler.maximum-allocation-mb`).
- JVM heap is for executor JVM; overhead is for Python workers, network buffers, and other non-JVM processes.
- PySpark memory comes from overhead; limited if overhead is small.

## UDF Memory Usage Comparison
| UDF Type         | Executes in   | Uses JVM Heap | Uses Overhead | Notes                       |
|------------------|--------------|--------------|---------------|-----------------------------|
| Python UDF       | Python Proc  | For serialization and exchange | Heavily       | Slowest, per-row overhead   |
| Pandas UDF (Arrow)| Python+Arrow | Less as no serialization is required        | Heavily       | Fast, vectorized            |
| JVM UDF (Scala)  | JVM          | Fully        | Not needed    | Fastest, no serialization   |

## Additional Notes
- Always check cluster physical memory limits before requesting container memory.
- PySpark applications rely on overhead memory for Python workers; insufficient overhead leads to OOM errors.
- Overhead memory is also used for shuffle exchange and network read buffers.
- Both JVM and overhead memory portions are critical for stable Spark execution.

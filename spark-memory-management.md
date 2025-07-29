# Spark Fundamentals: Memory Management

## Key Concepts
- Spark executor JVM heap is divided into Reserved Memory, Spark Memory Pool, and User Memory.
- Memory pool sizes are controlled by configuration parameters (`spark.memory.fraction`, `spark.memory.storageFraction`).
- Spark uses a unified memory manager for dynamic allocation between execution and storage pools.
- Off-heap memory and PySpark memory are managed separately from JVM heap.

## Architecture Overview
- Executor JVM heap is allocated via `spark.executor.memory`.
- Reserved Memory (300MB) is set aside for Spark engine internals.
- Remaining heap is split into Spark Memory Pool (default 60%) and User Memory (default 40%).
- Spark Memory Pool is further divided into Storage Memory (for caching) and Execution Memory (for computations).
- Storage and Execution pools have a flexible boundary; memory can be borrowed if free.
- Off-heap memory (`spark.memory.offHeap.size`) extends storage/execution pools and reduces JVM GC pressure.
- PySpark workers use overhead/off-heap memory, set via `spark.executor.pyspark.memory`.

## Components
- **Reserved Memory**: Fixed 300MB for Spark engine.
- **Spark Memory Pool**: Main pool for DataFrame operations and caching.
  - **Storage Memory**: For caching DataFrames (long-term).
  - **Execution Memory**: For computation buffers (short-term).
- **User Memory**: For user-defined data structures, RDD operations, Spark metadata.
- **Off-Heap Memory**: Optional, for large workloads and reduced GC delays.
- **PySpark Memory**: Overhead memory for Python workers (not JVM heap).

## Tables
| Memory Type         | Configuration Parameter           | Purpose/Usage                        |
|---------------------|-----------------------------------|--------------------------------------|
| Reserved Memory     | Fixed (300MB)                     | Spark engine internals               |
| Spark Memory Pool   | spark.memory.fraction (default 60%)| DataFrame ops, caching               |
| Storage Memory      | spark.memory.storageFraction (50%) | Caching DataFrames                   |
| Execution Memory    | spark.memory.storageFraction (50%) | Computation buffers                  |
| User Memory         | Remaining JVM heap                 | User data structures, RDD ops        |
| Off-Heap Memory     | spark.memory.offHeap.size          | Large workloads, reduce GC           |
| PySpark Memory      | spark.executor.pyspark.memory      | Python worker process (overhead)     |

## Additional Notes
- Unified memory manager dynamically allocates memory between execution and storage pools.
- Storage pool is long-term (caching); execution pool is short-term (computations).
- Off-heap memory is disabled by default; enable for large jobs to reduce GC delays.
- PySpark workers do not use JVM heap; set extra memory via `spark.executor.pyspark.memory` if needed.
- More than 5 executor cores can cause excessive memory contention; recommended is 2 to 5.
- Insufficient overhead or off-heap memory can lead to OOM errors, especially for PySpark workloads.

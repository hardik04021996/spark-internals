# Spark Fundamentals: Data Caching

## Key Concepts
- Spark can cache DataFrames in memory to speed up repeated computations.
- Caching uses the executor storage memory pool.
- Two main methods: `cache()` (default MEMORY_AND_DISK) and `persist()` (customizable storage level).
- Caching is a lazy transformation; data is cached only after an action is executed so spark tries to
only cache partitions that are needed to be accessed.

## Architecture Overview
- Storage memory is used for caching, while executor memory is used for computations.
- In Spark, caching is done at the partition level. Spark will only cache a partition if the entire partition fits in the available storage memory. If a partition is too large to fit, it will not be cached at all (Spark does not cache partial partitions). This behavior ensures data consistency and avoids partial or corrupted cache blocks.
- Caching can be in memory, disk, or off-heap, and can be serialized or deserialized.
  - **What is Serialization/Deserialization?**
    - Serialization is the process of converting an object (like a Python dict, Java object, or Scala case class) into a byte stream, so it can be:
      - Stored (e.g., on disk)
      - Sent over a network (e.g., to another Spark executor or worker)
    - Deserialization is the opposite process — turning the byte stream back into the original object.
  - **Why is Serialization Needed in Spark?**
    - Spark is distributed — it sends data between:
      - Driver ↔ Executors
      - Executor ↔ Executor (e.g., during shuffles)
      - Memory ↔ Disk (when spilling)
    - To do this efficiently, it serializes data into a compact format.
  - **Serialization vs Deserialization**:
    - **Deserialized (default for memory):**
      - Pros: Fastest access, no CPU overhead for deserialization when reading from cache.
      - Cons: Consumes more memory since data is stored as Java objects.
    - **Serialized:**
      - Pros: More compact, uses less memory.
      - Cons: Requires CPU to deserialize data when accessed, which can slow down processing.
    - **Default:** Spark caches data in memory in deserialized format. Disk and off-heap storage always use serialized format.
  - **In-Memory vs Disk Caching Format**:
    - Spark’s in-memory cache stores data in deserialized format for fast access and computation.
    - When you call `.cache()` or `.persist(MEMORY_ONLY)`, Spark keeps objects in deserialized JVM format, ready for computation (fast, but uses more memory).
    - If memory isn't sufficient (e.g., during a shuffle or join), Spark spills to disk.
    - During spill, data is serialized using the configured serializer (e.g., Kryo, Java). This makes it compact and avoids keeping full Java objects on disk.
    - When read back later, the data is deserialized again into memory, if needed.
    - **Summary Table:**
      | Location      | Format         |
      |--------------|---------------|
      | In-Memory    | Deserialized  |
      | Spilled Disk | Serialized    |
- Replication factor can be set for cached blocks (default is 1).

## Components
- **cache()**: Caches DataFrame using default MEMORY_AND_DISK storage level; no arguments.
- **persist()**: Caches DataFrame with a specified storage level (e.g., MEMORY_ONLY, MEMORY_AND_DISK, OFF_HEAP, etc.).
- **unpersist()**: Removes DataFrame from cache.
- **Storage Level**: Defines where and how data is cached (memory, disk, off-heap, serialized/deserialized, replication).

## Tables
| Storage Level         | Description                                      |
|----------------------|--------------------------------------------------|
| MEMORY_ONLY          | Cache in memory, deserialized                     |
| MEMORY_AND_DISK      | Cache in memory, spill to disk if needed          |
| MEMORY_ONLY_SER      | Cache in memory, serialized                       |
| MEMORY_AND_DISK_SER  | Cache in memory/disk, serialized                  |
| DISK_ONLY            | Cache only on disk                                |
| OFF_HEAP             | Cache in off-heap memory (if configured)          |

## Additional Notes
- Only accessed partitions are cached; Spark does not cache unused partitions.
- Caching is beneficial for large DataFrames accessed multiple times across actions.
- Avoid caching if DataFrame is small, not reused, or does not fit in memory.
- Use `unpersist()` to free up memory when cache is no longer needed.
- Replication factor >1 can improve data locality but increases memory usage.
- Disk and off-heap storage always use serialized format; the option of serialization and deserialization applies only to memory. The default for memory is deserialized (MEMORY_ONLY, MEMORY_AND_DISK).

  - **KryoSerializer Advantages**:
    1. **Faster Serialization/Deserialization**
       - Kryo is much faster than Java’s built-in serialization.
       - Uses efficient binary encoding and avoids Java reflection overhead.
       - Especially helpful in large, iterative jobs (like ML or graph processing).
    2. **More Compact Output (Less Data to Transfer)**
       - Kryo generates smaller byte streams, reducing disk I/O (when spilling) and shuffle data over the network.
       - Improves performance across the entire Spark pipeline.
    3. **Less Memory Usage**
       - Smaller serialized size = less memory pressure during shuffles and caching.
       - Helps reduce executor OOMs and GC overhead.
    4. **Custom Registration (Optional)**
       - You can register classes to further reduce overhead and improve speed.
       - Registered classes avoid writing full class metadata every time.
    **Why Not Use Kryo Always?**
    | Tradeoff                  | Explanation                                                      |
    |---------------------------|------------------------------------------------------------------|
    | Slightly more setup     | You may need to register custom classes for full efficiency       |
    | Limited out-of-box support | Some complex or generic types may need tweaking to serialize |

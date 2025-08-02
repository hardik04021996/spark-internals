# Spark Fundamentals: Broadcast Variables

## Key Concepts
- **Broadcast variables** are shared, immutable variables distributed to all worker nodes in a Spark cluster.
- Used to efficiently share reference data (e.g., lookup tables) with tasks, especially in UDFs and joins.
- Reduce serialization overhead compared to closures.
- Must fit into executor memory; otherwise, can cause OOM errors.

## Usage Scenarios
- **RDD API:** Broadcast variables are primarily used with Spark's low-level RDD APIs.
- **DataFrame API:** Less common, but Spark uses broadcast internally for broadcast joins.
- **UDFs:** Useful for making reference data available to UDFs without passing as function arguments.

## How Broadcast Variables Work
- Reference data (e.g., Python dictionary) is loaded on the driver.
- Data is broadcast to all worker nodes using `SparkContext.broadcast()`.
- Each worker node caches the broadcast variable locally; tasks access it as needed.
- Serialization is **lazy**: variable is only sent to a worker node if a task there needs it.

## Broadcast vs Closure
| Approach         | Serialization | Memory Usage | Scalability | Use Case                  |
|------------------|---------------|-------------|------------|---------------------------|
| Closure          | Per task      | High        | Poor       | Small reference data      |
| Broadcast Var    | Per node      | Low         | Good       | Large reference data/UDFs |

- **Closure:** Reference data is serialized with every task (e.g., 1000 tasks across 30 nodes → 1000 copies).
- **Broadcast:** Data is serialized once per worker node (e.g., 1000 tasks across 30 nodes → 30 copies).

## Example Workflow
1. Load reference data (e.g., lookup table) on driver.
2. Broadcast using `sc.broadcast(data)`.
3. Access broadcast variable in UDF or task via `.value` property.
4. Spark caches the broadcast variable on each worker node as needed.

## Broadcast Joins
- Spark DataFrame API uses broadcast variables for **broadcast joins**.
- Small table is collected on driver, broadcast to workers, and used for efficient join.
- Avoids shuffling large tables across the cluster.

## Additional Notes
- Broadcast variables must fit in executor memory.
- Use for large reference data in UDFs or joins to reduce serialization and network overhead.
- Not needed for small reference data; closures may suffice.
- Always test for memory impact and performance.

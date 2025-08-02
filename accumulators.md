# Spark Fundamentals: Accumulators

## Key Concepts
- **Accumulators** are global, mutable variables used for aggregating information (e.g., counters, sums) across Spark tasks.
- Primarily used with Spark's low-level RDD APIs; rarely needed with DataFrame APIs.
- Accumulators are updated by tasks and maintained at the driver.
- Useful for counting events (e.g., bad records) without triggering extra shuffle or aggregation stages.

## Usage Scenario
- Example: Counting bad records while cleaning a DataFrame column using a UDF.
- Avoids the need for a separate count() aggregation, which would cause a shuffle.

## How Accumulators Work
1. Create an accumulator variable using the Spark context (e.g., `sc.longAccumulator()`).
2. Use the accumulator inside a UDF or lambda to increment its value when a condition is met.
3. At the end of the job, read the accumulator value from the driver.

## Action vs Transformation
- **Action (Recommended):**
  - Using accumulators inside actions (e.g., `forEach`) guarantees accurate results.
  - Spark ensures each task's update is applied only once, even if tasks are retried or duplicated.
- **Transformation (Not Recommended):**
  - Using accumulators inside transformations (e.g., `withColumn`) can lead to over-counting due to task retries or duplicates.
  - Spark does not guarantee accuracy in this case.

## Comparison Table: Accumulator Usage
| Usage Context   | Accuracy Guarantee | Recommended |
|----------------|-------------------|-------------|
| Inside Action  | Yes               | Yes         |
| Inside Transformation | No          | No          |

## Types of Accumulators
- **LongAccumulator**: For integer counts.
- **FloatAccumulator**: For floating-point sums.
- **Custom Accumulators**: Possible, but advanced and rarely needed.

## Spark UI Visibility
- Scala: Accumulators can be named and shown in Spark UI.
- PySpark: Accumulators are unnamed and do not appear in Spark UI.

## Additional Notes
- Accumulators are best for simple counters or sums during distributed processing.
- Use for monitoring, debugging, or lightweight aggregation.
- Not suitable for complex aggregations or as a replacement for DataFrame aggregations.

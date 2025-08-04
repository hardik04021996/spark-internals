# Spark Fundamentals: Job Scheduling (Schedulers)

## Key Concepts
- **Job scheduling within a Spark application** determines how multiple jobs (actions) are executed: sequentially or in parallel.
- By default, Spark jobs run sequentially (FIFO), but parallel execution is possible using multithreading.
- Resource allocation and competition become important when running jobs in parallel.

## Scheduling Strategies
- **FIFO Scheduler (Default):**
  - Jobs are queued and executed one after another.
  - The first job gets priority for all available resources.
  - Later jobs may be delayed if earlier jobs are large.
- **FAIR Scheduler:**
  - Tasks from multiple jobs are assigned resources in a round-robin fashion.
  - All jobs get a roughly equal share of cluster resources.
  - Can be configured with multiple pools for finer control.
  - Default pool uses FIFO within the pool.

## How to Run Jobs in Parallel
- Use multithreading in your Spark application to submit independent jobs concurrently.
- Each thread can trigger a separate Spark action (job).
- Example (Python):
```python
import threading
from pyspark.sql import SparkSession

def do_job(file1, file2):
    spark = SparkSession.builder.getOrCreate()
    df1 = spark.read.parquet(file1)
    df2 = spark.read.parquet(file2)
    result = df1.join(df2, "key").count()
    print(result)

threads = []
for files in [("data1.parquet", "data2.parquet"), ("data3.parquet", "data4.parquet")]:
    t = threading.Thread(target=do_job, args=files)
    threads.append(t)
    t.start()
for t in threads:
    t.join()  # Waits for each thread to finish before continuing; ensures all jobs complete before exit
```

## Scheduler Configuration
| Scheduler   | Default Behavior | Resource Sharing | Configuration Key         |
|-------------|------------------|------------------|--------------------------|
| FIFO        | Sequential       | None             | spark.scheduler.mode=FIFO|
| FAIR        | Parallel (fair)  | Round-robin      | spark.scheduler.mode=FAIR|

- To enable FAIR scheduler:
```bash
--conf spark.scheduler.mode=FAIR
```

## Additional Notes
- FAIR scheduler pools can be configured for more granular resource sharing, but default pool is often sufficient.
- Monitor Spark UI to observe job scheduling and resource allocation.
- Consider resource competition and executor slot availability when running jobs in parallel.
- Proper scheduler configuration improves cluster utilization and job throughput.

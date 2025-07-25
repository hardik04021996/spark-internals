# Spark Fundamentals: spark-submit Options & Deploy Modes

## Key Concepts
- `spark-submit` is the main command-line tool for submitting Spark applications to clusters.
- Most common cluster managers: YARN, local, Kubernetes.
- Resource allocation and application configuration are controlled via command-line options.

## Common spark-submit Options
- `--class`: Main class for Java/Scala applications (not needed for PySpark).
- `--master`: Specifies cluster manager (`yarn`, `local`, etc.).
- `--deploy-mode`: Determines where the driver runs (`client` or `cluster`).
- `--conf`: Set additional Spark configuration properties (e.g., `spark.executor.memoryOverhead`).
- `--driver-cores`: Number of CPU cores for the driver container.
- `--driver-memory`: RAM for the driver container.
- `--num-executors`: Number of executor containers.
- `--executor-cores`: CPU cores per executor container.
- `--executor-memory`: RAM per executor container.

## Resource Allocation Example
| Option             | Purpose                                 | Example Value |
|--------------------|-----------------------------------------|--------------|
| --driver-cores     | CPU cores for driver                    | 2            |
| --driver-memory    | RAM for driver                          | 8G           |
| --num-executors    | Number of executors for whole cluster   | 4            |
| --executor-cores   | CPU cores per executor                  | 2            |
| --executor-memory  | RAM per executor                        | 16G          |

## Deploy Modes
- **Cluster Mode**
  - Driver runs in the cluster (AM container on a worker node).
  - Executors run in containers on worker nodes.
  - Allows you to log off client machine; job continues in cluster.
  - Lower network latency between driver and executors.
- **Client Mode**
  - Driver runs on the client machine (where spark-submit is executed).
  - Executors run in containers on cluster worker nodes.
  - Used for interactive workloads (e.g., spark-shell, notebooks).
  - If client logs off, driver dies and executors are terminated.

## Additional Notes
- Cluster mode is preferred for production and batch jobs.
- Client mode is mainly for interactive and development use cases.
- The driver and executors communicate heavily; cluster mode reduces network latency.
- Resource manager (YARN) allocates containers for driver and executors based on submitted options.
- Logging off the client in cluster mode does not affect running jobs; in client mode, it terminates the job.

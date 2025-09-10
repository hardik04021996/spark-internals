# Question

There is a parquet file stored in a datalake. Do the following:
1. Write PySpark code to read it into a DataFrame
2. Remove duplicate records
3. Write the DataFrame back to the datalake

# Answer
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("example").getOrCreate()

source = "path/to/input/file"
destination = "path/to/output/file"

df = spark.read.parquet(source)
df_without_duplicates = df.dropDuplicates()
df_without_duplicates.write.parquet(destination, mode="overwrite")

spark.stop()
```

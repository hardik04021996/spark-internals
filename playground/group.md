# Question

Given the following input DataFrame:

| col1 | col2 | col3 |
|------|------|------|
| a    | aa   | 1    |
| a    | aa   | 2    |
| b    | bb   | 5    |
| b    | bb   | 3    |
| b    | bb   | 4    |

Expected output:

| col1 | col2 | col3    |
|------|------|---------|
| a    | aa   | [1,2]   |
| b    | bb   | [5,3,4] |

# Answer (PySpark DataFrame API)
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import collect_list

spark = SparkSession.builder.appName("example").getOrCreate()

data = [
    ("a", "aa", 1),
    ("a", "aa", 2),
    ("b", "bb", 5),
    ("b", "bb", 3),
    ("b", "bb", 4),
]

schema = ["col1", "col2", "col3"]

df = spark.createDataFrame(data, schema=schema)
df_grouped = df.groupBy("col1", "col2").agg(collect_list("col3").alias("col3"))
df_grouped.show()

spark.stop()
```

# Answer (Spark SQL)
```sql
-- Register the DataFrame as a temporary view (in PySpark):
-- df.createOrReplaceTempView("my_table")

SELECT col1, col2, collect_list(col3) AS col3
FROM my_table
GROUP BY col1, col2;
```

# Answer (PySpark SQL)
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("example").getOrCreate()

data = [
    ("a", "aa", 1),
    ("a", "aa", 2),
    ("b", "bb", 5),
    ("b", "bb", 3),
    ("b", "bb", 4),
]

schema = ["col1", "col2", "col3"]

df = spark.createDataFrame(data, schema=schema)
df.createOrReplaceTempView("my_table")

result = spark.sql("""
    SELECT col1, col2, collect_list(col3) AS col3
    FROM my_table
    GROUP BY col1, col2
""")
result.show()

spark.stop()
```

# Notes
- All solutions are correct and produce the expected output.
- In Spark SQL, use `collect_list()` for array aggregation after registering the DataFrame as a view.
- You must register a DataFrame as a temporary view (using `df.createOrReplaceTempView("name")`) to run SQL queries on it in PySpark. You cannot run a SQL query directly on a DataFrame without registering it as a view.
- If you use the DataFrame API, you do not need to register a view.

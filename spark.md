# PySpark ETL Scripts

Python project containing modules of Apache Spark ETL scripts as well as common code:

- job1.py
- job2.py

## Launching Jobs on an Existing Spark Cluster

Ignoring end-to-end Spark solutions such as Databricks and managed offerings on AWS and Azure, then the easiest was to submit a Python Spark job to an existing Spark cluster - e.g. `spark:/localhost:7077` if developing locally - is to use `spark-submit`, which exists in the `bin` folder of every Spark download. To execute `job1.py` for example, we would call `spark-submit` as follows,

```bash
SPARK_HOME/bin/spark-submit \
   --master spark://localhost:7077 \
   --packages 'org.elasticsearch:elasticsearch-spark-20_2.11:5.3.2' \
   --py-files packages.zip \
   leaseplan/job1.py arg1 arg2
```

Briefly, the options supplied serve the following purposes:

- `--master spark://localhost:7077` - the address of the Spark to start the job on;
- `--packages 'org.elasticsearch:elasticsearch-spark-20_2.11:5.3.2,...'` - Maven coordinates for any JAR dependencies required by the job;
- `--py-files packages.zip` - archive containing Python dependencies (modules) referenced by the job; and,
- `leaseplan/job1.py 1.2 ...` - the Python job file followed by any input arguments.

Full details of all possible options can be found [here](http://spark.apache.org/docs/latest/submitting-applications.html). Note, that we have left some options to be defined within the job (which is actually a Spark application) - e.g. `spark.cores.max` and `spark.executor.memory` are defined in the Python script as it is felt that the job should explicitly contain the requests for the required cluster rescources.

## Managing Python Dependencies

In this project, functions that can be used across clients are kept in a module called `common` and referenced in specific job files using, for example,

```python
from common import utils
```

This package, together with any additional dependencies referenced within it, must be to copied to each Spark node for all jobs that use `common` to run. This can be achieved in one of several ways:

1. send all dependencies as a `zip` archive together with the job, using `--py-files` with Spark submit;
2. formally package and upload `common` to somewhere like the `PyPi` archive (or a private version) and then run `pip3 install common` on each node; or,
3. a combination of manually copying new modules (e.g. `common`) to the Python path of each node and using `pip3 install` for additional dependencies (e.g. for `requests`).

Option (1) is by far the easiest and most flexible approach, so we will make use of this for now. To make this task easier, especially when modules such as `common` have additional dependencies (e.g. the `requests` package), we have provided the `create_py_packages_zip.sh` bash script for automating the production of `packages.zip`, given a list of dependencies documented in `Pipfile` and managed by the `pipenv` python application.

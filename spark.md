## Project Structure

The basic project structure is as follows:

```bash
root/
 |-- common/
 |   |-- logging.py
 |   |-- spark.py
 |-- jobs/
 |   |-- etl_job.py
 |-- configs/
 |   |-- etl_config.json
 |   tests/
 |   |-- test_data/
 |   |-- | -- employees/
 |   |-- | -- employees_report/
 |   |-- test_etl_job.py
 |   build_dependencies.sh
 |   packages.zip
```

The main Python module containing the ETL job (which will be sent to the Spark cluster), is `etl_job.py`. Any external configuration parameters required by `etl_job.py` are stored in JSON format in `etl_config.json`. Additional modules that support this job can be kept in the `common/` folder (more on this later). In the project's root we include `build_dependencies.sh`, which is bash script for building these dependencies into a zip-file to be sent to the cluster (`packages.zip`). Unit test modules are kept in the `tests/` folder and small chunks of representative input and output data, to be use with the tests, are kept in `tests/test_data` folder.

## Running the ETL job

Assuming that the `$SPARK_HOME` environment variable points to your local Spark installation folder, then the ETL job can be run from the project's root directory using the following command from the terminal,

```bash
$SPARK_HOME/bin/spark-submit \
--master local[*] \
--packages 'com.somesparkjar.dependency:1.0.0' \
--py-files dependencies.zip \
--files configs/etl_config.json \
jobs/etl_job.py
```

Briefly, the options supplied serve the following purposes:

- `--master spark://localhost:7077` - the address of the Spark cluster to start the job on. If you have a Spark cluster in operation (either in single-executor mode locally or something larger in the cloud) and want to send the job there, then modify this with the apprioriate Spark IP - e.g. `spark://localhost:7077`;
- `--packages 'com.somesparkjar.dependency:1.0.0,...'` - Maven coordinates for any JAR dependencies required by the job (e.g. JDBC driver for connecting to a relational database);
- `--py-files packages.zip` - archive containing Python dependencies (modules) referenced by the job; and,
- `jobs/etl_job.py` - the Python module file containing the ETL job to execute.

Full details of all possible options can be found [here](http://spark.apache.org/docs/latest/submitting-applications.html). Note, that we have left some options to be defined within the job (which is actually a Spark application) - e.g. `spark.cores.max` and `spark.executor.memory` are defined in the Python script as it is felt that the job should explicitly contain the requests for the required cluster rescources.

## Passing Configuration Parameters to the ETL Job

Although it is possible to pass arguments to `etl_job.py`, as you would for any generic Python module running as a 'main' program  - by specifying them after the module's filename and then parsing these command line arguments - this can get very complicated, very quickly, especially when there are lot of parameters (e.g. credentials for multiple databases, table names, SQL snippets, etc.). This also makes debugging the code from within a Python interpreter extremely awkward, as you don't have access to the command line arguments that would ordinarily be passed to the code, when calling it from the command line.

A much more effective solution is to send Spark a separate file - e.g. using the `--files etl_config.json` flag with `spark-subit` - containing the configuration in JSON format, which can be parsed into a Python dictionary in one line of code with `json.loads(config_file_contents)`. Testing the code from within a Python interactive console session is also greatly simplified, as all one has to do to access configuration parameters for testing, is to copy and paste the contents of the file - e.g.,

```python
import json

config = json.loads("""{"field": "value"}""")
```

For the exact details of how the configuration file is located, opened and parsed, please see the `start_spark()` function in `common/spark.py` (also discussed further below), which in addition to parsing the configuration file sent to Spark (and returning it as a Python dictionary), also launches the Spark driver program (the application) on the cluster and retrieves the Spark logger at the same time.

## Testing and Debugging Spark Jobs Using `start_spark`

It is not pracital to test and debug Spark jobs by sending them to a cluster using `spark-submit` and examining stack traces for clues on what went wrong. A more productive workflow is to use an interactive console session (e.g. IPython) or a debugger (e.g. the Python dubugger in Visual Studio Code). In practice, however, it can be hard to test and debug Spark jobs in this way, as they implicity rely on arguements that are sent to `spark-submit`, which are not available in a console or debug session.

We wrote the `start_spark` function - found in `common/spark.py` - to facilitate the development of Spark jobs that are aware of the context in which they are being executed - i.e. as `spark-submit` jobs or within an IPython console, etc. The expected location of the Spark and job configuration parameters required by the job, is contingent on which execution context has been detected. The doscstring for `start_spark` gives the precise details,

```python
def start_spark(app_name='my_spark_app', master='local[*]', jar_packages=[],
                files=[], spark_config={}):
    """Start Spark session, get Spark logger and load config files.

    Start a Spark session on the worker node and register the Spark
    application with the cluster. Note, that only the app_name argument
    will apply when this is called from a script sent to spark-submit.
    All other arguments exist solely for testing the script from within
    an interactive Python console.

    This function also looks for a file ending in 'config.json' that
    can be sent with the Spark job. If it is found, it is opened,
    the contents parsed (assuming it contains valid JSON for the ETL job
    configuration), into a dict of ETL job configuration parameters,
    which are returned as the last element in the tuple returned by
    this function. If the file cannot be found then the return tuple
    only contains the Spark session and Spark logger objects and None
    for config.

    The function checks the enclosing environment to see if it is being
    run from inside an interactive console session or from an
    environment which has a DEBUG environment varibale set (i.e. from
    within a DEBUG run with custom environment variables). In this
    scenario, the function uses all available function arguments to
    start a PySpark driver from the local PySpark package as opposed to
    using the spark-submit and Spark cluster defaults. This will also
    use local module imports, as opposed to those in the zip archive
    sent to spark via the --py-files flag in spark-submit.

    :param app_name: Name of Spark app.
    :param master: Cluster connection details (defaults to local[*].
    :param jar_packages: List of Spark JAR package names.
    :param files: List of files to send to Spark cluster (master and
        workers).
    :param spark_config: Dictionary of config key-value pairs.
    :return: A tuple of references to the Spark session, logger and
        config dict (only if available).
    """

    # ...

    return spark_sess, spark_logger, config_dict
```

For example, the following code snippet,

```python
spark, log, config = start_spark(
    app_name='my_etl_job',
    jar_packages=['com.somesparkjar.dependency:1.0.0'],
    files=['configs/etl_config.json'])
```

Will use the arguments provided to `start_spark` to setup the Spark job if executed from an interactive console session or debugger, but will look for the same arguments sent via `spark-submit` if that is how the job has been executed.

## Structure of an ETL Job

In order to facilitate easy debugging and testing, we recommend that the 'Transformation' step be isolated from the 'Extract' and 'Load' steps, into it's own function - taking input data arguments in the form of DataFrames and returning the transformed data as a single DataFrame. Then, the code that surrounds the use of the transformation function in the `main()` job function, is concerned with Extracting the data, passing it to the transformation function and then Loading (or writing) the results to their ultimate destination. Testing is simplified, as mock or test data can be passed to the transformation function and the results explicitly verified, which would not be possible if all of the ETL code resided in `main()` and referenced production data sources and destinations.

More generally, transformation functions should be designed to be idempotent. This is technical way of saying that the repeated application of the transformation function should have no impact on the fundamental state of output data, until the instance when the input data changes. One of the key advantages of idempotent ETL jobs, is that they can be set to run repeatedly (e.g. by using `cron` to trigger the `spark-submit` command above, on a pre-defined schedule), rather than having to factor-in potential dependencies on other ETL jobs completing successfully.

## Testing

In order to test with Spark, we use the `pyspark` Python package, which is bundled with the Spark JARs required to programmatically start-up and tear-down a local Spark instance, on a per-test-suite basis (we recommend using the `setUp` and `tearDown` methods in `unittest.TestCase` to do this once per test-suite). Note, that using `pyspark` to run Spark is an alternative way of developing with Spark as opposed to using the PySpark shell or `spark-submit`. 

Given that we have chosen to structure our ETL jobs in such a way as to isolate the 'Transformation' step into its own function (see 'Structure of an ETL job' above), we are free to feed it a small slice of 'real-world' production data that has been persisted locally - e.g. in `tests/test_data` or some easily accessible network directory - and check it against known results (e.g. computed manually or interactively within a Python interactive console session).

## Packaging ETL Job Dependencies

In this project, functions that can be used across different ETL jobs are kept in a module called `common` and referenced in specific job modules using, for example,

```python
from common import utils
```

This package, together with any additional dependencies referenced within it, must be to copied to each Spark node for all jobs that use `common` to run. This can be achieved in one of several ways:

1. send all dependencies as a `zip` archive together with the job, using `--py-files` with Spark submit;
2. formally package and upload `common` to somewhere like the `PyPi` archive (or a private version) and then run `pip3 install common` on each node; or,
3. a combination of manually copying new modules (e.g. `common`) to the Python path of each node and using `pip3 install` for additional dependencies (e.g. for `requests`).

Option (1) is by far the easiest and most flexible approach, so we will make use of this for now. To make this task easier, especially when modules such as `common` have additional dependencies (e.g. the `requests` package), we have provided the `build_dependencies.sh` bash script for automating the production of `packages.zip`, given a list of dependencies documented in `Pipfile` and managed by the `pipenv` python application.

```bash
#!/usr/bin/env bash

# check to see if pipenv is installed
if [ -x "$(which pipenv)" ]
then
    # check that there Pipfile exists in root directory
    if [ ! -e Pipfile ]
    then
        echo 'ERROR - cannot find Pipfile'
        exit 1
    fi

    # use Pipenv to create a requirement.txt file
    echo '... creating requirements.txt from Pipfile'
    pipenv lock -r > requirements.txt

    # install packages to a temporary directory and zip it
    pip3 install -r requirements.txt --target ./packages

    # zip dependencies
    if [ ! -d packages ]
    then 
        echo 'ERROR - pip failed to import dependencies'
        exit 1
    fi

    cd packages
    zip -9mrv packages.zip .
    mv packages.zip ..
    cd ..

    # remove temporary directory and requirements.txt
    rm -rf packages
    rm requirements.txt
    
    # add local modules
    echo '... adding all modules from local utils package'
    zip -ru9 packages.zip utils

    exit 0
else
    echo 'ERROR - pipenv is not installed --> run pip3 install pipenv to load pipenv into global site packages or install via a system package manager.'
    exit 1
fi
```

## Static Type Checking

We have used the Python type annotation framework, together with the [MyPy package](http://mypy-lang.org), to perform static type checks on the codebase. Analagous to any linter or unit testing framwork, MyPy can be run from the command line as follows,

```bash
pipenv run python -m mypy my_module_name/*.py
```

MyPy options for this project are kept in the `mypy.ini` file that MyPy will look for by default. For more information on the fulls set of options, see the [mypy documentation](https://mypy.readthedocs.io/en/stable/config_file.html).

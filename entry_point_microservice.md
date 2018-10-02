## Starting the REST API Service

The package's `service` module contains entry points (defined in `setup.py`), that start a Flask service exposing some functions via a REST API (defined in `api.py`). This can be started via the command line, from the root directory using,

```bash
pipenv run mypackage
```

Which will start the server at `http://localhost:5000`. To start the service in debug mode run,

```bash
pipenv run mypackage_debug
```

This will re-load the server any time changes are made to the codebase. Note, that this only applies when installing development dependencies, which installs the project as an editable dependency.

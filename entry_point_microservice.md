## Starting the Crowdfill Matching Service

The package's `microservice` module contains entry points (defined in `setup.py`), that start a Flask microservice that exposes the matching engine via a REST API (defined in `api.py`). This can be started via the command line, from the root directory using,

```bash
pipenv run crowdfill
```

Which will start the server at `http://localhost:5000/crowdfill`. To start the service in debug mode run,

```bash
pipenv run crowdfill_debug
```

This will re-load the server any time changes are made to the codebase. Note, that this only applies when installing development dependencies, which installs the project as an editable dependency.

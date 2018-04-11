## Starting the Crowdfill Matching Service

The package's default entry point - `__main__.py` - has been setup to start a Flask microservice that exposes the matching engine via a RESTful API. This can be started via the command line with,

```bash
pipenv run python -m crowdfill
```

Which will start the server at `http://localhost:5000/`.

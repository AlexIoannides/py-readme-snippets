## Building Deployable Distributions

The easiest and most pragmatic way to deploy this package is to build a Python [wheel](https://wheel.readthedocs.io/en/stable/), and to then to install it in a fresh virtual environment on the target system. The build configuration is determined by the parameters in `setup.py`. Note, that this requires that all package dependencies to also be specified here, regardless of their entry in `Pipfile`. For more information on Python packaging refer to the [Python Packaging User Guide](https://packaging.python.org) and the accompanying [sample project](https://github.com/pypa/sampleproject). To create the Python wheel run,

```bash
pipenv run python setup.py bdist_wheel
```

This will create `build`, `crowdfill.egg-info` and `dist` directories and the wheel can be found in the latter. This needs to be copied to the target system (which we are assuming has Python and Pipenv available as a minimum), where it can be installed into a new virtual environment, together with all downstream dependencies, using,

```bash
pipenv install path/to/your-package.whl
```

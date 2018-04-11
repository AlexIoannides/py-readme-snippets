## Documentation

The documentation in the `docs` folder has have been built using [Sphinx](http://www.sphinx-doc.org). We have used the default 'quickstart' automatic configuration, which is triggered by executing,

```bash
pipenv run sphinx-quickstart
```

The output is based primarily on the Docstrings in the sourcecode, using the `autodoc` extension within Sphinx (specified during the 'quickstart'). The documentation can be built by running the following command,

```bash
pipenv run sphinx-build -b html docs/source docs/build
```

A third party theme from [Read the Docs](https://readthedocs.org) has also been used, by installing the `sphinx_rtd_theme` as a development dependecy and modifying `docs/source/config.py` as follows:

```python
import sphinx_rtd_theme
html_theme = "sphinx_rtd_theme"
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
```

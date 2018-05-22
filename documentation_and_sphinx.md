## Documentation

The documentation in the `docs` folder has have been built using [Sphinx](http://www.sphinx-doc.org). We have used the default 'quickstart' automatic configuration, which is triggered by executing,

```bash
pipenv run sphinx-quickstart
```

The output is based primarily on the Docstrings in the sourcecode, using the `autodoc` extension within Sphinx (specified during the 'quickstart'). The documentation can be built by running the following command,

```bash
pipenv run sphinx-build -b html docs/source docs/build_html
```

A third party theme from [Read the Docs](https://readthedocs.org) has also been used, by installing the `sphinx_rtd_theme` as a development dependency and modifying `docs/source/config.py` as follows:

```python
import sphinx_rtd_theme
html_theme = "sphinx_rtd_theme"
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
```

### Creating a PDF Version Using LaTeX

So long as a LaTex distribution is present on your system (e.g. MikTeX for Mac OS X), then it is possible to create a PDF version of the documentation, as well. Start by building the prerequistite LaTex version from the ReStructured Text originals,

```bash
pipenv run sphinx-build -b latex docs/source docs/build_latex
```

Then, navigate to `docs/build_latex` and run,

```bash
make latexpdf
```

Both LaTeX and PDF versions can then be found in `docs/build_latex`.
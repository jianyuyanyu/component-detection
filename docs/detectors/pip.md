# Pip Detection

## Requirements

Pip detection depends on the following to successfully run:

- Python 2 or Python 3
- Internet connection
- One or more `setup.py` or `requirements.txt` files

## Detection strategy

### Installation Report (PipReportDetector)
The `--report` option of the `pip install` command produces a detailed JSON report of what it did install (or what it would have installed). 
See https://pip.pypa.io/en/stable/reference/installation-report/#specification for more details.

Serialization specifications:
- https://packaging.python.org/en/latest/specifications/core-metadata/
- https://peps.python.org/pep-0508/
- https://peps.python.org/pep-0301/

### Legacy Detection (PipDetector, SimplePipDetector)

Pip detection is performed by running the following code snippet on every *setup.py*:

```python
    import distutils.core;
    setup = distutils.core.run_setup({setup.py});
    print(setup.install_requires);
```

The code above allows Pip detection to detect any runtime dependencies.

`requirements.txt` files are parsed; a Git component is created for every `git+` url.

For every top level component, Pip detection makes http calls to Pip in order to determine latest version available, as well as to resolve the dependency tree by parsing the `METADATA` file on a given release's `bdist_wheel` or `bdist_egg`.

Full dependency graph generation is supported.

## Known limitations

Dev dependency tagging is not supported.

Pip detection will not run if `python` is unavailable.

If no `bdist_wheel` or `bdist_egg` are available for a given component, dependencies will not be fetched.

If no internet connection or a component cannot be found in PyPi, said component and its dependencies will be skipped.

## Environment Variables

The environment variable `PyPiMaxCacheEntries` is used to control the size of the in-memory LRU cache that caches responses from PyPi.
The default value is 4096.

The enviroment variable `PIP_INDEX_URL` is used to determine what package feed should be used for `pip install --report` detection.
The default value will use the PyPi index unless pip defaults have been configured globally.

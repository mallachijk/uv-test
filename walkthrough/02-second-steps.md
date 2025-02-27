# UV dependency adding and resolution

## venv

In case that `.venv` was not created automatically lets run

```bash
$ uv venv
```

And **DON'T** activate the environment.

Instead let's just try to add a dependency

```bash
$ uv add --dev ruff
$ uv tree
Resolved 2 packages in 0.68ms
uv-test v0.1.0
└── ruff v0.9.8 (group: dev)
```

But where is it installed?

```bash
$ pip list
Package    Version
---------- -------
pip        25.0.1
setuptools 65.5.0
$ source .venv
```

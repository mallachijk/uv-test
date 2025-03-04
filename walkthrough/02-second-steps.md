# UV dependency adding and resolution

## venv & project

In case that `.venv` was not created automatically lets run
```bash
$ uv venv --seed
```

The `--seed` is there to add 'pip' to the virtual environment. We will use it later

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
$ ls -l .venv/Lib/site-packages/
total 18
drwxr-xr-x 1 jakub 197609    0 Feb 27 17:40 __pycache__/
-rw-r--r-- 2 jakub 197609   39 Feb 27 17:46 _uv_test.pth
-rw-r--r-- 1 jakub 197609   18 Feb 27 17:40 _virtualenv.pth
-rw-r--r-- 1 jakub 197609 4342 Feb 27 17:40 _virtualenv.py
drwxr-xr-x 1 jakub 197609    0 Feb 27 17:46 ruff/
drwxr-xr-x 1 jakub 197609    0 Feb 27 17:46 ruff-0.9.8.dist-info/
drwxr-xr-x 1 jakub 197609    0 Feb 27 17:46 uv_test-0.1.0.dist-info/
```

So it automatically installed packages in the local virtual environment using the Python for this project and
this is a feature.

## Dependency resolution and package management

Let's use following::
```bash
$ .venv/Scripts/python -m pip install pandas[parquet]==2.2.3
$ .venv/Scripts/python -m pip install "python-dateutil<2.8.2"
```

Who knows what went wrong there?

































Well pip happily allowed to install python-dateutil version that is not compatible with pandas version.

At least it gave a warning but still overwrote the version. On older versions of Python and pip it was
not given.

Let's try uv now (after cleaning this mess):
```bash
$ .venv/Scripts/python -m pip uninstall -y numpy pandas pyarrow python-dateutil pytz six tzdata
$ uv add pandas[parquet]==2.2.3
$ uv add "python-dateutil<2.8.2"
```

uv prevented adding an incompatible version. Which might be both good and bad thing:
- Prevents less experienced devs from messing up the environment,
- However, some older projects with badly chosen dependencies might fail to resolve.

Still for most of the use cases it is safer.

Same applies for `requirements.txt` format
```bash
$ uv add -r walkthrough/files/reqs_wrong.txt
```

...will not resolve, while:
```bash
$ uv add -r walkthrough/files/reqs_correct.txt
```

...and in most cases allows you to use the `requirements.txt` format to apply uv to the existing
project, as well as allows to create this format with:
```bash
$ uv export > requirements.txt
```

## Viewing dependencies

Sometimes knowing if there are dependencies between installed packages helps and uv also allows
to examine them easily with:
```bash
$ uv tree
Resolved 9 packages in 0.71ms
uv-test v0.1.0
├── numpy v2.2.3
├── pandas[parquet] v2.2.3
│   ├── numpy v2.2.3
│   ├── python-dateutil v2.9.0.post0
│   │   └── six v1.17.0
│   ├── pytz v2025.1
│   ├── tzdata v2025.1
│   └── pyarrow v19.0.1 (extra: parquet)
├── pyarrow v19.0.1
└── ruff v0.9.8 (group: dev)
```

Which becomes quite nice in larger projects
# TODO add Flex Toolkits deps...
```bash
$ uv remove numpy pandas pyarrow
$ uv tree
uv-test v0.1.0
└── ruff v0.9.8 (group: dev)
$ uv add -r /files/reqs_a_lot.txt
...
```

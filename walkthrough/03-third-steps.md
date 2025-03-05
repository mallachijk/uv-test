# More advanced tricks

## Running modules without adding it to the environment

Running modules from outside of environment is possible with `uv run --with ...`
```bash
$ uv run --with mypy mypy ./src
Success: no issues found in 2 source files
$ uv tree --package mypy
Resolved 146 packages in 0.59ms
```

Though this example is not correct because mypy should probably be added as dev dependency:
```bash
$ uv add mypy --dev
Resolved 12 packages in 39ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test
Prepared 4 packages in 1.27s
Uninstalled 1 package in 3ms
Installed 4 packages in 349ms
 + mypy==1.15.0
 + mypy-extensions==1.0.0
 + typing-extensions==4.12.2
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test)
$ uv tree --depth 1
Resolved 148 packages in 0.55ms
uv-test v0.1.0
├── apscheduler v3.10.4
├── azure-identity v1.17.1
├── databricks-sdk v0.29.0
├── databricks-sql-connector v3.2.0
├── fastapi[standard] v0.115.11
├── flask v3.0.2
├── geopandas v0.14.4
├── mkdocs v1.6.1
├── plotly v5.21.0
├── pydantic v2.10.6
├── pyodbc v5.1.0
├── pyyaml v6.0.2
├── scipy v1.14.1
├── shapely v2.0.4
├── sqlalchemy v2.0.25
├── taipy v3.1.1
├── mypy v1.15.0 (group: dev)
└── ruff v0.9.9 (group: dev)
```

This command also allows you to use different Python version in isolated environment:
```bash
$ uv run -p 3.12.1 --isolated --frozen -m timeit
Installed 145 packages in 489ms
50000000 loops, best of 5: 5.57 nsec per loop
$ uv run -p 3.12.9 --isolated --frozen -m timeit
Installed 145 packages in 367ms
50000000 loops, best of 5: 8.12 nsec per loop
```

Though a lot of it depends on caching so it can grow quite quickly:
```bash
$ du -h -d 1 $(uv cache dir) | sort -hr
1.5G    /home/codespace/.cache/uv
1.3G    /home/codespace/.cache/uv/archive-v0
171M    /home/codespace/.cache/uv/sdists-v8
45M     /home/codespace/.cache/uv/simple-v15
2.1M    /home/codespace/.cache/uv/wheels-v5
556K    /home/codespace/.cache/uv/builds-v0
80K     /home/codespace/.cache/uv/interpreter-v4
8.0K    /home/codespace/.cache/uv/environments-v2
```




# TODO:

Also [Microsoft already kinda endorses it](https://learn.microsoft.com/en-us/azure/app-service/configure-language-python#using-uv).
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
Resolved 148 packages in 86ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
Prepared 3 packages in 1.16s
Uninstalled 1 package in 1ms
Installed 3 packages in 391ms
 + mypy==1.15.0
 + mypy-extensions==1.0.0
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
$ uv tree --depth 1
Resolved 148 packages in 0.88ms
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
Installed 147 packages in 4.84s
50000000 loops, best of 5: 5.49 nsec per loop
$ uv run -p 3.12.9 --isolated --frozen -m timeit
Installed 147 packages in 4.38s
50000000 loops, best of 5: 8.75 nsec per loop
```

Though a lot of it depends on caching so it can grow quite quickly:
```bash
$ du -h -d 1 $(uv cache dir) | sort -hr
974M    C:\Users\jakub\AppData\Local\uv\cache
866M    C:\Users\jakub\AppData\Local\uv\cache/archive-v0
64M     C:\Users\jakub\AppData\Local\uv\cache/sdists-v8
42M     C:\Users\jakub\AppData\Local\uv\cache/simple-v15
2.0M    C:\Users\jakub\AppData\Local\uv\cache/wheels-v4
40K     C:\Users\jakub\AppData\Local\uv\cache/interpreter-v4
4.0K    C:\Users\jakub\AppData\Local\uv\cache/builds-v0
1.0K    C:\Users\jakub\AppData\Local\uv\cache/environments-v2
```

## Workspaces

Let's say that we have some API to develop but we want to isolate it from the rest of the code.

First remove fastapi from the general codebase
```bash
$ uv remove fastapi[standard]
Resolved 131 packages in 420ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
Prepared 1 package in 517ms
Uninstalled 17 packages in 62ms
Installed 1 package in 4ms
 - anyio==4.8.0
 - email-validator==2.2.0
 - fastapi==0.115.11
 - fastapi-cli==0.0.7
 - httpcore==1.0.7
 - httptools==0.6.4
 - httpx==0.28.1
 - python-multipart==0.0.20
 - rich-toolkit==0.13.2
 - shellingham==1.5.4
 - sniffio==1.3.1
 - starlette==0.46.0
 - typer==0.15.2
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
 - uvicorn==0.34.0
 - watchfiles==1.0.4
 - websockets==15.0.1
```

Let's create a folder for it and initialize uv again into it.
```bash
$ mkdir packages/uv-test-api -p
$ uv init packages/uv-test-api
Adding `uv-test-api` as member of workspace `C:\Users\jakub\repositories\uv-test-actual`
Initialized project `uv-test-api` at `C:\Users\jakub\repositories\uv-test-actual\packages\uv-test-api`
$ cat pyproject.toml
[project]
name = "uv-test"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
authors = [
]
requires-python = ">=3.12"
dependencies = [
    "apscheduler==3.10.4",
    "azure-identity==1.17.1",
    "databricks-sdk==0.29.0",
    "databricks-sql-connector==3.2.0",
    "flask>=3.0.2",
    "geopandas==0.14.4",
    "mkdocs==1.6.1",
    "plotly==5.21.0",
    "pydantic==2.10.6",
    "pyodbc==5.1.0",
    "pyyaml~=6.0",
    "scipy==1.14.1",
    "shapely==2.0.4",
    "sqlalchemy==2.0.25",
    "taipy==3.1.1",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[dependency-groups]
dev = [
    "mypy>=1.15.0",
    "ruff>=0.9.9",
]

[tool.uv.workspace]
members = ["packages/uv-test-api"]
```

And now we can have a separate package with it's own dependencies
```bash
$ uv add fastapi[standard] --package uv-test-api
Resolved 149 packages in 62ms
Installed 16 packages in 66ms
 + anyio==4.8.0
 + email-validator==2.2.0
 + fastapi==0.115.11
 + fastapi-cli==0.0.7
 + httpcore==1.0.7
 + httptools==0.6.4
 + httpx==0.28.1
 + python-multipart==0.0.20
 + rich-toolkit==0.13.2
 + shellingham==1.5.4
 + sniffio==1.3.1
 + starlette==0.46.0
 + typer==0.15.2
 + uvicorn==0.34.0
 + watchfiles==1.0.4
 + websockets==15.0.1
$ uv tree -d 1
uv-test v0.1.0
├── apscheduler v3.10.4
├── azure-identity v1.17.1
├── databricks-sdk v0.29.0
├── databricks-sql-connector v3.2.0
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
uv-test-api v0.1.0
└── fastapi[standard] v0.115.11
```

And now funny thing:
```bash
$ uv add ./packages/uv-test-api
Resolved 149 packages in 69ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
Prepared 1 package in 483ms
Uninstalled 1 package in 1ms
Installed 1 package in 5ms
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
$ uv tree -d 1
Resolved 149 packages in 1ms
uv-test v0.1.0
├── apscheduler v3.10.4
├── azure-identity v1.17.1
├── databricks-sdk v0.29.0
├── databricks-sql-connector v3.2.0
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
├── uv-test-api v0.1.0
├── mypy v1.15.0 (group: dev)
└── ruff v0.9.9 (group: dev)
```

We have added `uv-test-api` library as the dependency for the `uv-test`.

Benefits of this approach are more likely for larger codebases but still cool.
However, it still has some limitations:
- shared lockfile
- shared virtual environment (so no conflicting dependencies)

## Endorsment

While it's still kind of fresh tool, `uv` offers quite some benefits for creating Python projects and is a bit
smarter with managing dependencies.

Also [Microsoft already kinda endorses it](https://learn.microsoft.com/en-us/azure/app-service/configure-language-python#using-uv).
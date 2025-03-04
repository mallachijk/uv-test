# More advanced tricks

## Running modules without adding it to the environment

Running modules from outside of environment is possible with `uv run --with ...`
```bash
$ uv run --with mypy mypy ./src
Success: no issues found in 2 source files
$ uv tree
# TODO - correct tree
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
$ uv tree
Resolved 12 packages in 0.74ms
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
├── mypy v1.15.0 (group: dev)
│   ├── mypy-extensions v1.0.0
│   └── typing-extensions v4.12.2
└── ruff v0.9.8 (group: dev)
```

This command also allows you to use different Python version in isolated environment:
```bash
$ uv run -p 3.12.0 --isolated --frozen -m timeit
Installed 12 packages in 550ms
50000000 loops, best of 5: 7.13 nsec per loop
$ uv run -p 3.13 --isolated --frozen -m timeit
Installed 12 packages in 447ms
50000000 loops, best of 5: 5.7 nsec per loop
```

Though a lot of it depends on caching so it can grow quite quickly:
```bash
$ du -h -d 1 $(uv cache dir)
835M    C:\Users\jakub\AppData\Local\uv\cache/archive-v0
476K    C:\Users\jakub\AppData\Local\uv\cache/builds-v0
8.0K    C:\Users\jakub\AppData\Local\uv\cache/environments-v2
64K     C:\Users\jakub\AppData\Local\uv\cache/interpreter-v4
38M     C:\Users\jakub\AppData\Local\uv\cache/sdists-v8
25M     C:\Users\jakub\AppData\Local\uv\cache/simple-v15
1.4M    C:\Users\jakub\AppData\Local\uv\cache/wheels-v4
898M    C:\Users\jakub\AppData\Local\uv\cache
```

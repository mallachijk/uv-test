# UV dependency adding and resolution

## venv & project

In case that `.venv` was not created automatically lets run
```bash
$ uv venv --seed
Using CPython 3.12.9
Creating virtual environment with seed packages at: .venv
 + pip==25.0.1
Activate with: source .venv/Scripts/activate
```

The `--seed` is there to add 'pip' to the virtual environment. We will use it later

And **DON'T** activate the environment yet.

Instead let's just try to add a dependency
```bash
$ uv add --dev ruff
Resolved 2 packages in 95ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
Prepared 2 packages in 848ms
Installed 2 packages in 9ms
 + ruff==0.9.9
 + uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
$ uv tree
Resolved 2 packages in 0.72ms
uv-test v0.1.0
└── ruff v0.9.9 (group: dev)
```

But where is it installed?
```bash
$ pip list
Package    Version
---------- -------
pip        25.0.1
setuptools 65.5.0
$ ls -l .venv/Lib/site-packages/
total 26
-rw-r--r-- 2 jakub 197609   46 Mar  6 20:03 _uv_test.pth
-rw-r--r-- 1 jakub 197609   18 Mar  6 20:03 _virtualenv.pth
-rw-r--r-- 1 jakub 197609 4342 Mar  6 20:03 _virtualenv.py
drwxr-xr-x 1 jakub 197609    0 Mar  6 20:03 pip/
drwxr-xr-x 1 jakub 197609    0 Mar  6 20:03 pip-25.0.1.dist-info/
drwxr-xr-x 1 jakub 197609    0 Mar  6 20:03 ruff/
drwxr-xr-x 1 jakub 197609    0 Mar  6 20:03 ruff-0.9.9.dist-info/
drwxr-xr-x 1 jakub 197609    0 Mar  6 20:03 uv_test-0.1.0.dist-info/
```

So it automatically installed packages in the local virtual environment using the Python for this project and
this is a feature.

## Dependency resolution and package management

Let's use following::
```bash
$ .venv/Scripts/python -m pip install pandas[parquet]==2.2.3
...
$ .venv/Scripts/python -m pip install "python-dateutil<2.8.2"
Collecting python-dateutil<2.8.2
  Using cached python_dateutil-2.8.1-py2.py3-none-any.whl.metadata (8.0 kB)
Requirement already satisfied: six>=1.5 in c:\users\jakub\repositories\uv-test-actual\.venv\lib\site-packages (from python-dateutil<2.8.2) (1.17.0)
Using cached python_dateutil-2.8.1-py2.py3-none-any.whl (227 kB)
Installing collected packages: python-dateutil
  Attempting uninstall: python-dateutil
    Found existing installation: python-dateutil 2.9.0.post0
    Uninstalling python-dateutil-2.9.0.post0:
      Successfully uninstalled python-dateutil-2.9.0.post0
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
pandas 2.2.3 requires python-dateutil>=2.8.2, but you have python-dateutil 2.8.1 which is incompatible.
Successfully installed python-dateutil-2.8.1
```

Who knows what went wrong there?

































Well pip happily allowed to install python-dateutil version that is not compatible with pandas version.

At least it gave a warning but still overwrote the version. On older versions of Python and pip it was
not given.

Let's try uv now (after cleaning this mess):
```bash
$ .venv/Scripts/python -m pip uninstall -y numpy pandas pyarrow python-dateutil pytz six tzdata
...
$ uv add pandas[parquet]==2.2.3
Resolved 9 packages in 129ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
Prepared 8 packages in 4.00s
Uninstalled 1 package in 1ms
Installed 8 packages in 829ms
 + numpy==2.2.3
 + pandas==2.2.3
 + pyarrow==19.0.1
 + python-dateutil==2.9.0.post0
 + pytz==2025.1
 + six==1.17.0
 + tzdata==2025.1
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
$ uv add "python-dateutil<2.8.2"
warning: `VIRTUAL_ENV=C:/Users/jakub/repositories/uv-test/.venv` does not match the project environment path `.venv` and will be ignored; use `--active` to target the active environment instead
  × No solution found when resolving dependencies:
  ╰─▶ Because pandas==2.2.3 depends on python-dateutil>=2.8.2 and your project depends on pandas[parquet]==2.2.3, we can conclude that your project depends on python-dateutil>=2.8.2.
      And because your project depends on python-dateutil<2.8.2, we can conclude that your project's requirements are unsatisfiable.
  help: If you want to add the package regardless of the failed resolution, provide the `--frozen` flag to skip locking and syncing.
```

uv prevented adding an incompatible version. Which might be both good and bad thing:
- Prevents less experienced devs from messing up the environment,
- However, some older projects with badly chosen dependencies might fail to resolve.

Still for most of the use cases it is safer.

Let's clear up this
```bash
$ uv remove pandas[parquet]
Resolved 2 packages in 10ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
Prepared 1 package in 464ms
Uninstalled 8 packages in 474ms
Installed 1 package in 4ms
 - numpy==2.2.3
 - pandas==2.2.3
 - pyarrow==19.0.1
 - python-dateutil==2.9.0.post0
 - pytz==2025.1
 - six==1.17.0
 - tzdata==2025.1
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
$ uv tree
Resolved 2 packages in 0.69ms
uv-test v0.1.0
└── ruff v0.9.9 (group: dev)
```

It also removed all dependencies of Pandas that were no longer required!

Let's see how to use standard `requirements.txt` format
```bash
$ uv add -r ../uv-test/walkthrough/files/reqs_wrong.txt
  × No solution found when resolving dependencies:
  ╰─▶ Because pandas[parquet]==2.2.3 depends on pyarrow>=10.0.1 and your project depends on pandas[parquet]==2.2.3, we can conclude that your project depends on pyarrow>=10.0.1.
      And because your project depends on pyarrow<10, we can conclude that your project's requirements are unsatisfiable.
  help: If you want to add the package regardless of the failed resolution, provide the `--frozen` flag to skip locking and syncing.
```

...will not resolve, while:
```bash
$ uv add -r ../uv-test/walkthrough/files/reqs_correct.txt
```

...and in most cases allows you to use the `requirements.txt` format to apply uv to the existing
project, as well as allows to create this format with:
```bash
$ uv export > requirements.txt
Resolved 9 packages in 13ms
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
Prepared 1 package in 465ms
Uninstalled 1 package in 1ms
Installed 8 packages in 381ms
 + numpy==2.2.3
 + pandas==2.2.3
 + pyarrow==19.0.1
 + python-dateutil==2.9.0.post0
 + pytz==2025.1
 + six==1.17.0
 + tzdata==2025.1
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
```

## Viewing dependencies

Sometimes knowing if there are dependencies between installed packages helps and uv also allows
to examine them easily with:
```bash
$ uv tree
Resolved 9 packages in 0.75ms
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
└── ruff v0.9.9 (group: dev)
```

Which becomes quite nice in larger projects and is quite faster than pip with better progress tracking.
```bash
$ uv remove numpy pandas pyarrow
$ uv tree
Resolved 2 packages in 0.77ms
uv-test v0.1.0
└── ruff v0.9.9 (group: dev)
$ uv add -r ../uv-test/walkthrough/files/reqs_a_lot.txt
Resolved 146 packages in 8.31s
      Built taipy-gui==3.1.4
      Built taipy-core==3.1.1
      Built uv-test @ file:///C:/Users/jakub/repositories/uv-test-actual
      Built taipy-config==3.1.1
      Built taipy-rest==3.1.1
      Built taipy-templates==3.1.1
      Built gitignore-parser==0.1.11
      Built thrift==0.20.0
Prepared 140 packages in 27.95s
Uninstalled 1 package in 1ms
Installed 144 packages in 4.57s
 + aniso8601==10.0.0
 + annotated-types==0.7.0
 + anyio==4.8.0
 + apispec==6.4.0
 + apispec-webframeworks==1.0.0
 + apscheduler==3.10.4
 + arrow==1.3.0
 + attrs==25.1.0
 + automat==24.8.1
 + azure-core==1.32.0
 + azure-identity==1.17.1
 + bidict==0.23.1
 + binaryornot==0.4.4
 + blinker==1.9.0
 + boto3==1.34.34
 + botocore==1.34.162
 + cachetools==5.5.2
 + certifi==2025.1.31
 + cffi==1.17.1
 + chardet==5.2.0
 + charset-normalizer==3.4.1
 + click==8.1.8
 + click-plugins==1.1.1
 + cligj==0.7.2
 + colorama==0.4.6
 + constantly==23.10.4
 + cookiecutter==2.5.0
 + cryptography==44.0.2
 + databricks-sdk==0.29.0
 + databricks-sql-connector==3.2.0
 + deepdiff==6.7.1
 + dnspython==2.7.0
 + email-validator==2.2.0
 + et-xmlfile==2.0.0
 + fastapi==0.115.11
 + fastapi-cli==0.0.7
 + fiona==1.10.1
 + flask==3.0.2
 + flask-cors==4.0.0
 + flask-restful==0.3.10
 + flask-socketio==5.3.6
 + geopandas==0.14.4
 + gevent==23.9.1
 + gevent-websocket==0.10.1
 + ghp-import==2.1.0
 + gitignore-parser==0.1.11
 + google-auth==2.38.0
 + greenlet==3.1.1
 + h11==0.14.0
 + httpcore==1.0.7
 + httptools==0.6.4
 + httpx==0.28.1
 + hyperlink==21.0.0
 + idna==3.10
 + incremental==24.7.2
 + itsdangerous==2.2.0
 + jinja2==3.1.6
 + jmespath==1.0.1
 + kthread==0.2.3
 + lz4==4.4.3
 + markdown==3.5.2
 + markdown-it-py==3.0.0
 + markupsafe==3.0.2
 + marshmallow==3.20.2
 + mdurl==0.1.2
 + mergedeep==1.3.4
 + mkdocs==1.6.1
 + mkdocs-get-deps==0.2.0
 + msal==1.31.1
 + msal-extensions==1.2.0
 + networkx==3.2.1
 + numpy==2.2.3
 + oauthlib==3.2.2
 + openpyxl==3.1.2
 + ordered-set==4.1.0
 + packaging==24.2
 + pandas==2.1.1
 + passlib==1.7.4
 + pathspec==0.12.1
 + platformdirs==4.3.6
 + plotly==5.21.0
 + portalocker==2.10.1
 + pyarrow==14.0.2
 + pyasn1==0.6.1
 + pyasn1-modules==0.4.1
 + pycparser==2.22
 + pydantic==2.10.6
 + pydantic-core==2.27.2
 + pygments==2.19.1
 + pyjwt==2.10.1
 + pymongo==4.6.1
 + pyodbc==5.1.0
 + pyproj==3.7.1
 + python-dateutil==2.9.0.post0
 + python-dotenv==1.0.1
 + python-engineio==4.11.2
 + python-multipart==0.0.20
 + python-slugify==8.0.4
 + python-socketio==5.12.1
 + pytz==2023.3.post1
 + pywin32==308
 + pyyaml==6.0.2
 + pyyaml-env-tag==0.1
 + requests==2.32.3
 + rich==13.9.4
 + rich-toolkit==0.13.2
 + rsa==4.9
 + s3transfer==0.10.4
 + scipy==1.14.1
 + setuptools==75.8.2
 + shapely==2.0.4
 + shellingham==1.5.4
 + simple-websocket==1.0.0
 + six==1.17.0
 + sniffio==1.3.1
 + sqlalchemy==2.0.25
 + starlette==0.46.0
 + taipy==3.1.1
 + taipy-config==3.1.1
 + taipy-core==3.1.1
 + taipy-gui==3.1.4
 + taipy-rest==3.1.1
 + taipy-templates==3.1.1
 + tenacity==9.0.0
 + text-unidecode==1.3
 + thrift==0.20.0
 + toml==0.10.2
 + twisted==23.10.0
 + twisted-iocpsupport==1.0.4
 + typer==0.15.2
 + types-python-dateutil==2.9.0.20241206
 + typing-extensions==4.12.2
 + tzdata==2025.1
 + tzlocal==5.2
 + urllib3==2.3.0
 ~ uv-test==0.1.0 (from file:///C:/Users/jakub/repositories/uv-test-actual)
 + uvicorn==0.34.0
 + watchdog==6.0.0
 + watchfiles==1.0.4
 + websockets==15.0.1
 + werkzeug==3.1.3
 + wsproto==1.2.0
 + zope-event==5.0
 + zope-interface==7.2
```

And allows to clearly what depend on what:
```bash
$ uv tree
Resolved 146 packages in 0.89ms
uv-test v0.1.0
├── apscheduler v3.10.4
│   ├── pytz v2023.3.post1
│   ├── six v1.17.0
│   └── tzlocal v5.2
│       └── tzdata v2025.1
├── azure-identity v1.17.1
│   ├── azure-core v1.32.0
│   │   ├── requests v2.32.3
│   │   │   ├── certifi v2025.1.31
│   │   │   ├── charset-normalizer v3.4.1
│   │   │   ├── idna v3.10
│   │   │   └── urllib3 v2.3.0
│   │   ├── six v1.17.0
│   │   └── typing-extensions v4.12.2
│   ├── cryptography v44.0.2
│   │   └── cffi v1.17.1
│   │       └── pycparser v2.22
│   ├── msal v1.31.1
│   │   ├── cryptography v44.0.2 (*)
│   │   ├── pyjwt[crypto] v2.10.1
│   │   │   └── cryptography v44.0.2 (extra: crypto) (*)
│   │   └── requests v2.32.3 (*)
│   ├── msal-extensions v1.2.0
│   │   ├── msal v1.31.1 (*)
│   │   └── portalocker v2.10.1
│   │       └── pywin32 v308
│   └── typing-extensions v4.12.2
├── databricks-sdk v0.29.0
│   ├── google-auth v2.38.0
│   │   ├── cachetools v5.5.2
│   │   ├── pyasn1-modules v0.4.1
│   │   │   └── pyasn1 v0.6.1
│   │   └── rsa v4.9
│   │       └── pyasn1 v0.6.1
│   └── requests v2.32.3 (*)
├── databricks-sql-connector v3.2.0
│   ├── lz4 v4.4.3
│   ├── numpy v2.2.3
│   ├── oauthlib v3.2.2
│   ├── openpyxl v3.1.2
│   │   └── et-xmlfile v2.0.0
│   ├── pandas v2.1.1
│   │   ├── numpy v2.2.3
│   │   ├── python-dateutil v2.9.0.post0
│   │   │   └── six v1.17.0
│   │   ├── pytz v2023.3.post1
│   │   └── tzdata v2025.1
│   ├── pyarrow v14.0.2
│   │   └── numpy v2.2.3
│   ├── requests v2.32.3 (*)
│   ├── thrift v0.20.0
│   │   └── six v1.17.0
│   └── urllib3 v2.3.0
├── fastapi[standard] v0.115.11
│   ├── pydantic v2.10.6
│   │   ├── annotated-types v0.7.0
│   │   ├── pydantic-core v2.27.2
│   │   │   └── typing-extensions v4.12.2
│   │   └── typing-extensions v4.12.2
│   ├── starlette v0.46.0
│   │   └── anyio v4.8.0
│   │       ├── idna v3.10
│   │       ├── sniffio v1.3.1
│   │       └── typing-extensions v4.12.2
│   ├── typing-extensions v4.12.2
│   ├── email-validator v2.2.0 (extra: standard)
│   │   ├── dnspython v2.7.0
│   │   └── idna v3.10
│   ├── fastapi-cli[standard] v0.0.7 (extra: standard)
│   │   ├── rich-toolkit v0.13.2
│   │   │   ├── click v8.1.8
│   │   │   │   └── colorama v0.4.6
│   │   │   ├── rich v13.9.4
│   │   │   │   ├── markdown-it-py v3.0.0
│   │   │   │   │   └── mdurl v0.1.2
│   │   │   │   └── pygments v2.19.1
│   │   │   └── typing-extensions v4.12.2
│   │   ├── typer v0.15.2
│   │   │   ├── click v8.1.8 (*)
│   │   │   ├── rich v13.9.4 (*)
│   │   │   ├── shellingham v1.5.4
│   │   │   └── typing-extensions v4.12.2
│   │   ├── uvicorn[standard] v0.34.0
│   │   │   ├── click v8.1.8 (*)
│   │   │   ├── h11 v0.14.0
│   │   │   ├── colorama v0.4.6 (extra: standard)
│   │   │   ├── httptools v0.6.4 (extra: standard)
│   │   │   ├── python-dotenv v1.0.1 (extra: standard)
│   │   │   ├── pyyaml v6.0.2 (extra: standard)
│   │   │   ├── watchfiles v1.0.4 (extra: standard)
│   │   │   │   └── anyio v4.8.0 (*)
│   │   │   └── websockets v15.0.1 (extra: standard)
│   │   └── uvicorn[standard] v0.34.0 (extra: standard) (*)
│   ├── httpx v0.28.1 (extra: standard)
│   │   ├── anyio v4.8.0 (*)
│   │   ├── certifi v2025.1.31
│   │   ├── httpcore v1.0.7
│   │   │   ├── certifi v2025.1.31
│   │   │   └── h11 v0.14.0
│   │   └── idna v3.10
│   ├── jinja2 v3.1.6 (extra: standard)
│   │   └── markupsafe v3.0.2
│   ├── python-multipart v0.0.20 (extra: standard)
│   └── uvicorn[standard] v0.34.0 (extra: standard) (*)
├── flask v3.0.2
│   ├── blinker v1.9.0
│   ├── click v8.1.8 (*)
│   ├── itsdangerous v2.2.0
│   ├── jinja2 v3.1.6 (*)
│   └── werkzeug v3.1.3
│       └── markupsafe v3.0.2
├── geopandas v0.14.4
│   ├── fiona v1.10.1
│   │   ├── attrs v25.1.0
│   │   ├── certifi v2025.1.31
│   │   ├── click v8.1.8 (*)
│   │   ├── click-plugins v1.1.1
│   │   │   └── click v8.1.8 (*)
│   │   └── cligj v0.7.2
│   │       └── click v8.1.8 (*)
│   ├── numpy v2.2.3
│   ├── packaging v24.2
│   ├── pandas v2.1.1 (*)
│   ├── pyproj v3.7.1
│   │   └── certifi v2025.1.31
│   └── shapely v2.0.4
│       └── numpy v2.2.3
├── mkdocs v1.6.1
│   ├── click v8.1.8 (*)
│   ├── colorama v0.4.6
│   ├── ghp-import v2.1.0
│   │   └── python-dateutil v2.9.0.post0 (*)
│   ├── jinja2 v3.1.6 (*)
│   ├── markdown v3.5.2
│   ├── markupsafe v3.0.2
│   ├── mergedeep v1.3.4
│   ├── mkdocs-get-deps v0.2.0
│   │   ├── mergedeep v1.3.4
│   │   ├── platformdirs v4.3.6
│   │   └── pyyaml v6.0.2
│   ├── packaging v24.2
│   ├── pathspec v0.12.1
│   ├── pyyaml v6.0.2
│   ├── pyyaml-env-tag v0.1
│   │   └── pyyaml v6.0.2
│   └── watchdog v6.0.0
├── plotly v5.21.0
│   ├── packaging v24.2
│   └── tenacity v9.0.0
├── pydantic v2.10.6 (*)
├── pyodbc v5.1.0
├── pyyaml v6.0.2
├── scipy v1.14.1
│   └── numpy v2.2.3
├── shapely v2.0.4 (*)
├── sqlalchemy v2.0.25
│   ├── greenlet v3.1.1
│   └── typing-extensions v4.12.2
├── taipy v3.1.1
│   ├── cookiecutter v2.5.0
│   │   ├── arrow v1.3.0
│   │   │   ├── python-dateutil v2.9.0.post0 (*)
│   │   │   └── types-python-dateutil v2.9.0.20241206
│   │   ├── binaryornot v0.4.4
│   │   │   └── chardet v5.2.0
│   │   ├── click v8.1.8 (*)
│   │   ├── jinja2 v3.1.6 (*)
│   │   ├── python-slugify v8.0.4
│   │   │   └── text-unidecode v1.3
│   │   ├── pyyaml v6.0.2
│   │   ├── requests v2.32.3 (*)
│   │   └── rich v13.9.4 (*)
│   ├── taipy-gui v3.1.4
│   │   ├── flask v3.0.2 (*)
│   │   ├── flask-cors v4.0.0
│   │   │   └── flask v3.0.2 (*)
│   │   ├── flask-socketio v5.3.6
│   │   │   ├── flask v3.0.2 (*)
│   │   │   └── python-socketio v5.12.1
│   │   │       ├── bidict v0.23.1
│   │   │       └── python-engineio v4.11.2
│   │   │           └── simple-websocket v1.0.0
│   │   │               └── wsproto v1.2.0
│   │   │                   └── h11 v0.14.0
│   │   ├── gevent v23.9.1
│   │   │   ├── cffi v1.17.1 (*)
│   │   │   ├── greenlet v3.1.1
│   │   │   ├── zope-event v5.0
│   │   │   │   └── setuptools v75.8.2
│   │   │   └── zope-interface v7.2
│   │   │       └── setuptools v75.8.2
│   │   ├── gevent-websocket v0.10.1
│   │   │   └── gevent v23.9.1 (*)
│   │   ├── gitignore-parser v0.1.11
│   │   ├── kthread v0.2.3
│   │   ├── markdown v3.5.2
│   │   ├── pandas v2.1.1 (*)
│   │   ├── python-dotenv v1.0.1
│   │   ├── pytz v2023.3.post1
│   │   ├── simple-websocket v1.0.0 (*)
│   │   ├── taipy-config v3.1.1
│   │   │   ├── deepdiff v6.7.1
│   │   │   │   └── ordered-set v4.1.0
│   │   │   └── toml v0.10.2
│   │   ├── twisted v23.10.0
│   │   │   ├── attrs v25.1.0
│   │   │   ├── automat v24.8.1
│   │   │   ├── constantly v23.10.4
│   │   │   ├── hyperlink v21.0.0
│   │   │   │   └── idna v3.10
│   │   │   ├── incremental v24.7.2
│   │   │   │   └── setuptools v75.8.2
│   │   │   ├── twisted-iocpsupport v1.0.4
│   │   │   ├── typing-extensions v4.12.2
│   │   │   └── zope-interface v7.2 (*)
│   │   └── tzlocal v5.2 (*)
│   ├── taipy-rest v3.1.1
│   │   ├── apispec[yaml] v6.4.0
│   │   │   ├── packaging v24.2
│   │   │   └── pyyaml v6.0.2 (extra: yaml)
│   │   ├── apispec-webframeworks v1.0.0
│   │   │   └── apispec[yaml] v6.4.0 (*)
│   │   ├── flask v3.0.2 (*)
│   │   ├── flask-restful v0.3.10
│   │   │   ├── aniso8601 v10.0.0
│   │   │   ├── flask v3.0.2 (*)
│   │   │   ├── pytz v2023.3.post1
│   │   │   └── six v1.17.0
│   │   ├── marshmallow v3.20.2
│   │   │   └── packaging v24.2
│   │   ├── passlib v1.7.4
│   │   └── taipy-core v3.1.1
│   │       ├── boto3 v1.34.34
│   │       │   ├── botocore v1.34.162
│   │       │   │   ├── jmespath v1.0.1
│   │       │   │   ├── python-dateutil v2.9.0.post0 (*)
│   │       │   │   └── urllib3 v2.3.0
│   │       │   ├── jmespath v1.0.1
│   │       │   └── s3transfer v0.10.4
│   │       │       └── botocore v1.34.162 (*)
│   │       ├── networkx v3.2.1
│   │       ├── openpyxl v3.1.2 (*)
│   │       ├── pandas v2.1.1 (*)
│   │       ├── pyarrow v14.0.2 (*)
│   │       ├── pymongo v4.6.1
│   │       │   └── dnspython v2.7.0
│   │       ├── sqlalchemy v2.0.25 (*)
│   │       ├── taipy-config v3.1.1 (*)
│   │       └── toml v0.10.2
│   └── taipy-templates v3.1.1
└── ruff v0.9.9 (group: dev)
(*) Package tree already displayed
```

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
$ uv add -r walkthrough/files/reqs_a_lot.txt
Resolved 146 packages in 0.53ms
Installed 142 packages in 9.43s
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
 + jinja2==3.1.5
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
 + typer==0.15.2
 + types-python-dateutil==2.9.0.20241206
 + typing-extensions==4.12.2
 + tzdata==2025.1
 + tzlocal==5.2
 + urllib3==2.3.0
 + uv-test==0.1.0 (from file:///workspaces/uv-test)
 + uvicorn==0.34.0
 + uvloop==0.21.0
 + watchdog==6.0.0
 + watchfiles==1.0.4
 + websockets==15.0
 + werkzeug==3.1.3
 + wsproto==1.2.0
 + zope-event==5.0
 + zope-interface==7.2
```

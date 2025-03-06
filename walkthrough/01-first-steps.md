# UV first steps

## Installing UV

With pip
```bash
pip install uv
```

Normaly - Unix
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Less normally - Windows (should be fine without admin)
```bash
winget install --id=astral-sh.uv  -e
```

For now let's just quickly see what is available
```bash
$ uv -h
```

## Python

Ok, there is this python command, what does it do?
```bash
$ uv python list
cpython-3.14.0a5+freethreaded-linux-x86_64-gnu    <download available>
cpython-3.14.0a5-linux-x86_64-gnu                 <download available>
cpython-3.13.2+freethreaded-linux-x86_64-gnu      <download available>
cpython-3.13.2-linux-x86_64-gnu                   <download available>
cpython-3.12.9-linux-x86_64-gnu                   /root/.local/share/uv/python/cpython-3.12.9-linux-x86_64-gnu/bin/python3.12
cpython-3.11.11-linux-x86_64-gnu                  <download available>
cpython-3.10.16-linux-x86_64-gnu                  <download available>
cpython-3.10.12-linux-x86_64-gnu                  /usr/bin/python3.10
cpython-3.10.12-linux-x86_64-gnu                  /usr/bin/python3 -> python3.10
cpython-3.10.12-linux-x86_64-gnu                  /bin/python3.10
cpython-3.10.12-linux-x86_64-gnu                  /bin/python3 -> python3.10
cpython-3.9.21-linux-x86_64-gnu                   <download available>
cpython-3.8.20-linux-x86_64-gnu                   <download available>
cpython-3.7.9-linux-x86_64-gnu                    <download available>
pypy-3.11.11-linux-x86_64-gnu                     <download available>
pypy-3.10.19-linux-x86_64-gnu                     <download available>
pypy-3.9.19-linux-x86_64-gnu                      <download available>
pypy-3.8.16-linux-x86_64-gnu                      <download available>
pypy-3.7.13-linux-x86_64-gnu                      <download available>
```

Quite some Python versions here, let's see later what does it enable!

## Project

Ok so let's setup a project using one of the python versions that we don't have.
```bash
$ cd ../
$ uv init -p 3.12 --name uv-test ./uv-test-actual
Initialized project `uv-test` at `/workspaces/uv-test-actual`
```

This creates a new project using Python 3.12 (`-p 3.12`) with name `uv-test` in folder `./uv-test`.
```bash
$ ls -al ./uv-test-actual
total 28
drwxrwxrwx+ 3 codespace codespace 4096 Mar  6 18:53 .
drwxr-xrwx+ 6 codespace root      4096 Mar  6 18:53 ..
drwxrwxrwx+ 7 codespace codespace 4096 Mar  6 18:53 .git
-rw-rw-rw-  1 codespace codespace  109 Mar  6 18:53 .gitignore
-rw-rw-rw-  1 codespace codespace    5 Mar  6 18:53 .python-version
-rw-rw-rw-  1 codespace codespace    0 Mar  6 18:53 README.md
-rw-rw-rw-  1 codespace codespace   85 Mar  6 18:53 main.py
-rw-rw-rw-  1 codespace codespace  153 Mar  6 18:53 pyproject.toml
```

This already provide quite some structure for your project, together with initialization of Git. However, almost
everything there is customizable and to start without it you can use `--bare` argument

The defaults assume that you are building app (web servers, scripts, and command-line interfaces). Let's say that we want to
create a library to be distributed and used by others.
```bash
$ rm -r ./uv-test-actual
$ uv init -p 3.12 --name uv-test ./uv-test-actual --lib
Initialized project `uv-test` at `/workspaces/uv-test-actual`
$ cd ./uv-test-actual
$ ls -hlR
.:
total 8.0K
-rw-rw-rw-  1 codespace codespace    0 Mar  6 18:55 README.md
-rw-rw-rw-  1 codespace codespace  300 Mar  6 18:55 pyproject.toml
drwxrwxrwx+ 3 codespace codespace 4.0K Mar  6 18:55 src

./src:
total 4.0K
drwxrwxrwx+ 2 codespace codespace 4.0K Mar  6 18:55 uv_test

./src/uv_test:
total 4.0K
-rw-rw-rw- 1 codespace codespace 53 Mar  6 18:55 __init__.py
-rw-rw-rw- 1 codespace codespace  0 Mar  6 18:55 py.typed
```

Now we have our directory with the project already created setup for the project

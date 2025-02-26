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
```
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

Huh, so there are few Python versions here. What about outside of WSL?

```bash
$ uv python list
cpython-3.14.0a5+freethreaded-windows-x86_64-none    <download available>
cpython-3.14.0a5-windows-x86_64-none                 <download available>
cpython-3.13.2+freethreaded-windows-x86_64-none      <download available>
cpython-3.13.2-windows-x86_64-none                   <download available>
cpython-3.12.9-windows-x86_64-none                   <download available>
cpython-3.11.11-windows-x86_64-none                  <download available>
cpython-3.11.9-windows-x86_64-none                   AppData\Local\Programs\Python\Python311\python.exe
cpython-3.10.16-windows-x86_64-none                  <download available>
cpython-3.9.21-windows-x86_64-none                   <download available>
cpython-3.8.20-windows-x86_64-none                   <download available>
cpython-3.7.9-windows-x86_64-none                    <download available>
pypy-3.11.11-windows-x86_64-none                     <download available>
pypy-3.10.19-windows-x86_64-none                     <download available>
pypy-3.9.19-windows-x86_64-none                      <download available>
pypy-3.8.16-windows-x86_64-none                      <download available>
pypy-3.7.13-windows-x86_64-none                      <download available>
```

## Project

Ok so let's setup a project using one of the python versions that we don't have.

```bash
$ uv init -p 3.12 --name uv-test ./uv-test
```

This creatres a new projecte using Python 3.12 (`-p 3.12`) with name `uv-test` in folder `./uv-test`.

```bash
$ ls -al
drwxr-xr-x 7 root root 4096 Feb 26 19:15 .git
-rw-r--r-- 1 root root  109 Feb 26 19:07 .gitignore
-rw-r--r-- 1 root root    5 Feb 26 19:07 .python-version
drwxr-xr-x 4 root root 4096 Feb 26 19:12 .venv
-rw-r--r-- 1 root root  253 Feb 26 19:17 README.md
-rw-r--r-- 1 root root   85 Feb 26 19:07 main.py
-rw-r--r-- 1 root root  153 Feb 26 19:07 pyproject.toml
-rw-r--r-- 1 root root  127 Feb 26 19:12 uv.lock
```

This already provide quite some stucture for your project, toghether with initalization of Git. However, almost
everything there is customizable and to start without it you can use `--bare` argument

The defaults assume that you are building app (web servers, scripts, and command-line interfaces). Let's say that we want to
create a library to be distributed and used by others.

```bash
$ cd ..
$ rm -r ./uv-test
$ uv init -p 3.12 --name uv-test ./uv-test --lib
$ cd ./uv-test
$ ls -hr
.:
total 12K
-rw-r--r-- 1 root root    0 Feb 26 19:52 README.md
-rw-r--r-- 1 root root  305 Feb 26 19:52 pyproject.toml
drwxr-xr-x 3 root root 4.0K Feb 26 19:52 src
drwxr-xr-x 2 root root 4.0K Feb 26 19:52 walkthrough

./src:
total 4.0K
drwxr-xr-x 2 root root 4.0K Feb 26 19:52 uv_test

./src/uv_test:
total 4.0K
-rw-r--r-- 1 root root 53 Feb 26 19:52 __init__.py
-rw-r--r-- 1 root root  0 Feb 26 19:52 py.typed
```

Hmm, we have our directory

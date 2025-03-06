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
cpython-3.14.0a5+freethreaded-windows-x86_64-none    <download available>
cpython-3.14.0a5-windows-x86_64-none                 <download available>
cpython-3.13.2+freethreaded-windows-x86_64-none      C:\Users\jakub\AppData\Roaming\uv\python\cpython-3.13.2+freethreaded-windows-x86_64-none\python.exe
cpython-3.13.2-windows-x86_64-none                   C:\Users\jakub\AppData\Roaming\uv\python\cpython-3.13.2-windows-x86_64-none\python.exe
cpython-3.12.9-windows-x86_64-none                   C:\Users\jakub\AppData\Roaming\uv\python\cpython-3.12.9-windows-x86_64-none\python.exe
cpython-3.12.0-windows-x86_64-none                   C:\Users\jakub\AppData\Roaming\uv\python\cpython-3.12.0-windows-x86_64-none\python.exe
cpython-3.11.11-windows-x86_64-none                  <download available>
cpython-3.11.9-windows-x86_64-none                   C:\Users\jakub\AppData\Local\Programs\Python\Python311\python.exe
cpython-3.10.16-windows-x86_64-none                  <download available>
cpython-3.9.21-windows-x86_64-none                   C:\Users\jakub\AppData\Roaming\uv\python\cpython-3.9.21-windows-x86_64-none\python.exe
cpython-3.8.20-windows-x86_64-none                   <download available>
cpython-3.7.9-windows-x86_64-none                    <download available>
pypy-3.11.11-windows-x86_64-none                     <download available>
pypy-3.10.19-windows-x86_64-none                     <download available>
pypy-3.9.19-windows-x86_64-none                      <download available>
pypy-3.8.16-windows-x86_64-none                      <download available>
pypy-3.7.13-windows-x86_64-none                      <download available>
```

Quite some Python versions here, let's see later what does it enable! If you are running it on linux you will see
linux platform specific distributions!

## Project

Ok so let's setup a project using one of the python versions that we don't have.
```bash
$ cd ../
$ uv init -p 3.12 --name uv-test ./uv-test-actual
Initialized project `uv-test` at `C:\Users\jakub\repositories\uv-test-actual`
```

This creates a new project using Python 3.12 (`-p 3.12`) with name `uv-test` in folder `./uv-test`.
```bash
$ ls -al ./uv-test-actual
total 12
drwxr-xr-x 1 jakub 197609   0 Mar  6 20:01 ./
drwxr-xr-x 1 jakub 197609   0 Mar  6 20:01 ../
drwxr-xr-x 1 jakub 197609   0 Mar  6 20:01 .git/
-rw-r--r-- 1 jakub 197609 109 Mar  6 20:01 .gitignore
-rw-r--r-- 1 jakub 197609   5 Mar  6 20:01 .python-version
-rw-r--r-- 1 jakub 197609  85 Mar  6 20:01 main.py
-rw-r--r-- 1 jakub 197609 153 Mar  6 20:01 pyproject.toml
-rw-r--r-- 1 jakub 197609   0 Mar  6 20:01 README.md
```

This already provide quite some structure for your project, together with initialization of Git. However, almost
everything there is customizable and to start without it you can use `--bare` argument

The defaults assume that you are building app (web servers, scripts, and command-line interfaces). Let's say that we want to
create a library to be distributed and used by others.
```bash
$ rm -r ./uv-test-actual
$ uv init -p 3.12 --name uv-test ./uv-test-actual --lib
Initialized project `uv-test` at `C:\Users\jakub\repositories\uv-test-actual`
$ cd ./uv-test-actual
$ ls -hlR
.:
total 1.0K
-rw-r--r-- 1 jakub 197609 309 Mar  6 20:02 pyproject.toml
-rw-r--r-- 1 jakub 197609   0 Mar  6 20:02 README.md
drwxr-xr-x 1 jakub 197609   0 Mar  6 20:02 src/

./src:
total 0
drwxr-xr-x 1 jakub 197609 0 Mar  6 20:02 uv_test/

./src/uv_test:
total 1.0K
-rw-r--r-- 1 jakub 197609 53 Mar  6 20:02 __init__.py
-rw-r--r-- 1 jakub 197609  0 Mar  6 20:02 py.typed
```

Now we have our directory with the project already created setup for the project

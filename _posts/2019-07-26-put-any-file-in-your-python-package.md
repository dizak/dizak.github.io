---
title: "Distribute any file with your Python package"
published: true
---

# Introduction

The mechanism for embedding the files that need to be used during the installation process is simple:
if you need ```requirements.txt``` - just put its path inside the ```MANIFEST.in```.
But the files listed inside ```MANIFEST.in``` file are **NOT** placed anywhere in the user's' system!
For actually distributing *non-python* executables, configs or any other kind of file - you need proper arguments inside your ```setup``` function.

# Goals

Prepare Python package which installs Python module and non-python executable.

# Environment and Set-up

This is the structure of our test module:

```bash
├── bin
│   └── neofetch
├── foomod
│   ├── foomod.py
│   └── __init__.py
├── MANIFEST.in
├── requirements.txt
└── setup.py
```

- Inside ```bin/``` there is neofetch - bash script which print the system info and logo in a really cool way ;)

- foomod is a dummy module that holds only one function - called - ```get_google_response_code``` - it fetches google.com and returns the request's status code.
If the installation process of ```requirements``` fails then it will throw an error.

Here is how ```foomod.__init__``` looks like:

```python
__version__ = 'v0.0.0'
__author__ = 'dizak'

__all__ = ['foomod']

try:
    from .foomod import get_google_response_code
except ImportError:
    print('Failed to import package. Ignore if running setup')
```

The  ```__author__``` and ```__version__``` variables which are imported by ```setup.py``` are defined there.
Also, ```foomod``` is imported to the top-level of the module.

...and here is ```foomod.py```:

```python
import requests as rq


def get_google_response_code():
    res = rq.get('https://www.google.com')
    return res.status_code
```

- There is only one dependency listed in ```requirements.txt``` - ```requests``` module.

Here is the content of ```MANIFEST.in```:

```bash
include bin/neofetch
include requirements.txt
```

- finally, ```setup.py```:

```python
from setuptools import find_packages
from setuptools import setup
from foomod import __version__
from foomod import __author__


setup(
    name='foobar',
    version=__version__,
    author=__author__,
    packages=find_packages(),
    include_package_data=True,
    data_files=[
        ('bin', ['bin/neofetch']),
    ],
    install_requires=open('requirements.txt').readlines(),
    py_modules = ['foomod'],
)
```


# Installation Process

As you can see, there is nothing really special about ```setup.py```.
There are only two key things:

1. To tell the ```setup``` function that it should include the package data.
This is what ```include_package_data``` argument is for.

2. To tell the ```setup``` function where the package data files should be placed and where can be found in you package.
This is what ```data_files``` argument is for.

In this example, file found under relative path ```bin/neofetch``` is placed inside ```bin/``` directory relative to the directory where Python packages are installed.
If you run ```pip install --user foobar-0.0.0.tar.gz``` then ```neofetch``` will be placed in ```$HOME/.local/bin/```.

What's insteresting, you should stick to the structure of list of tuples, each composed of string and list. If you change list to tuple, then you will see an error like this:

```bash
Command "/usr/bin/python -u -c "import setuptools, tokenize;__file__='/tmp/pip-req-build-V3a4S3/setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record /tmp/pip-record-RvSfQT/install-record.txt --single-version-externally-managed --compile --user --prefix=" failed with error code 1 in /tmp/pip-req-build-V3a4S3/
```

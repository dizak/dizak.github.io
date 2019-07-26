---
title: "Distribute any file with your Python package"
published: true
---

# Introduction

The mechanism for embedding the files that need to be used during the installation process is simple:
if you need ```requirements.txt``` to be present - just put its name inside the ```MANIFEST.in```.
But the files listed inside ```MANIFEST.in``` file are **NOT** placed anywhere in your system!
For actually distributing *non-python* executables, configs or any other kind of file - you need proper arguments inside your ```setup``` function.

# Goals

Prepare Python package which installs Python module and non-python executable.

# Environment and set-up

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

- foomod is a module that holds only one function - it fetches google.com and returns the HTML request status code. 

- There is only one dependency listed in ```requirements.txt``` - requests module.

Here is the content of ```MANIFEST.in```:

```bash
include bin/neofetch
include requirements.txt
```

There is the content of setup.py:

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
)
```

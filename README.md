# Xx

## Motivation

Users will typically used [pyenv](https://github.com/pyenv/pyenv) to
install multiple versions of Python, and switch amongst them dynamically.
Pyenv provides a rich set of commands, but at its heart the package
implements the following:
* A set of shims (eg `python`, `pip`, etc) installed at `$PYENV_ROOT/shims`
* A multiplexor that is sensitive to `PYENV_VERSION` environment variable,
  and mechanisms to provide a default version within a directory,
  directory tree, or per user.

Pyenv provides a collection of curated recipes to build different
versions of Python. While the recipes work in many cases, it is
hard to debug and support the compilation process when the
host environment causes problems.

Xx supports the use case of dynamically selecting the installed
version to run, and leaves aside all other scenarios, in particular
that building corresponding distribution. Users are left to find their
own recipes to build Python, Ruby, and Perl respectively. In its
simplest form, for example, the recipe to integrate Python 2.7.18
with xx is:

```
./configure --prefix="$(/usr/local/python/bin/xx root)/2.7.18"
make
make install
```

Most importantly, xx does not require the user to add a shim
directory to the PATH environment variable. The only requirement
is a symlink to the xx executable in one of the directories named
in PATH, for example in `/usr/local/bin` or `$HOME/bin`.

## Use

Xx relies in a symlink from a `bin/` directory. Suppose that multiple
versions of Python are installed at `/usr/local/python/versions`, and
xx is installed at `/usr/local/python/versions/bin`:
```
% cd /usr/local/python/version/bin
% ln -s $PWD/xx /usr/local/python/bin/python
% ln -s $PWD/xx /usr/local/python/bin/xx
```

For convenience, an additional symlink is created from a common
bin directory (eg `/usr/local/bin`) to avoid cluttering the
PATH environment variable with each installed language:
```
% ln -s /usr/local/python/bin/python /usr/local/bin/
% PATH="/usr/local/bin:$PATH"
```

Once xx installed, a version of Python (or correspondingly
Ruby, or Perl) is selected from the the environment variable
`PYXX_VERSION` (or `PYENV_VERSION`):
```
% PYXX_VERSION=2.7.18 python --version
Python 2.7.18
```

The key idea is that target application is wrapped in a small
script that configures the required version before launching the
underlying program. In the case of Python:
```
#!/bin/sh

PYXX_VERSION=2.7.18 exec python "$0.py"
```

Xx will look for the matching Python version in a peer directory:
```
% ls "$(/usr/local/python/bin/xx root)"
2.7.18@  3.8.7@  3.9.1@  bin/
```

The xx root directory can use symlinks to point at the
corresponding Python installations as shown here, or each
version can be installed directly into a subdirectory named
after the version.

Xx provides a more convenient way to list the installed versions.
Using the Python use case as an example:
```
% /usr/local/python/bin/xx versions
2.7.18
3.8.7
3.9.1
```

## Installation

To install xx in `/usr/local/python/versions/bin`:
```
cd "$(git rev-parse --show-toplevel)" &&
  cp bin/llxx bin/xx /usr/local/python/versions/bin/
```

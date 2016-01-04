# Python on Nix

You're a Python dev, and recently you've heard about [Nix](http://nixos.org/nix/).
Immediately convinced by sane package management you decide you want to start
using Nix. But before you do so, there's still an important question that you
want an answer to. **How does Nix affect my Python development workflow?**
In this tutorial we show you how you use and develop with Python on Nix.

## Introduction to Nix

While we assume you've already read a bit on Nix before going through this
tutorial, we recap a couple of things first. [Nix](http://nixos.org/nix/)
is a purely functional package manager for *Nix systems. Packages are specified
using the [Nix expression language](http://nixos.org/nix/manual/#chap-writing-nix-expressions).
[NixOS](http://nixos.org) is an operating system built around the Nix package
manager. On NixOS not only packages but also services (called
[modules](http://nixos.org/nixos/manual/index.html#sec-writing-modules)) are
described using the Nix expression language.
[Nixpkgs](http://nixos.org/nixpkgs/) is a collection of packages (and NixOS
modules).


## Getting started

If you don't have Nix yet, and you would like to install it on your *Nix system,
then head on to the [Nix Quick
Start](http://nixos.org/nix/manual/#chap-quick-start).
if instead you want to install NixOS, then please continue now with the [NixOS
installation guide](http://nixos.org/nixos/manual/index.html#ch-installation).

## Using Python

<!-- You want to be able to use the Python interpreter, along with a bunch of Python packages? Read on! -->


### Installing software

Now that you've installed Nix you're ready to start installing software.
With the Nix package manager you can [ad hoc install](http://nixos.org/nixos/manual/index.html#sec-ad-hoc-packages) software in your profile
using `nix-env -iA`. e.g.

    $ nix-env -iA nixpkgs.pkgs.pandoc

would install the [Pandoc](http://pandoc.org/) tool in your profile. From then on you will have the
possibility to run `pandoc` from within a shell.

If you are running NixOS and you have `root`/`sudo` access, then you can specify
[declaratively](http://nixos.org/nixos/manual/index.html#sec-declarative-package-mgmt) which packages need to be installed by appending those packages to
`environment.systemPackages` in the `/etc/nixos/configuration.nix` file

    ...

    environment.systemPackages = with pkgs; [
      busybox
      chromium
      pandoc
    ];

    ...

A common method on Nix is however not to install all the software you need, but
instead using the [Nix shell](https://nixos.org/nix/manual/#sec-nix-shell)
(`nix-shell`) to open a shell with exactly those packages that you need when you
need them. E.g., say you want to convert some files and so you want to use
Pandoc, then you could run

    $ nix-shell -p pandoc

which opens a shell from which you can run pandoc

    [nix-shell:~] pandoc tutorial.md -o tutorial.pdf


### Installing Python?

You might be wondering now why, if this tutorial is about Python, we are using
Pandoc as an example and not Python? Well, that's because with Python you're
generally interested in not just the interpreter, but also Python packages. On
Nix most software can be installed in a profile, either ad hoc or declaratively.
However, this is not possibly with Python and packages. Actually, to be precise,
the tools allow you to install to your profile e.g. Python 3.5 using

    $ nix-env -iA nixpkgs.pkgs.python35

Most likely, now running

    $ python35

will actually open the interpreter. Nothing wrong, right? Well, not entirely.
There is a problem though with installing Python modules/packages in this way;
generally they cannot be accessed from the interpreter.
Obviously you do not want this. What's the solution to that, you might ask?

Installing Python declaratively perhaps? How about the following in your
`/etc/nixos/configuration.nix` in case you're running NixOS?

    ...

    environment.systemPackages = with pkgs; [
      busybox
      chromium
      pandoc
      python35
      python35Packages.numpy
      python35Packages.toolz
    ];

    ...
Nope, it might work, but likely not always. But *why* does it not work then?

With Nix you install only applications. Because with Nix you can have multiple
versions of a library/application at the same time, an application needs to know
which exact libraries to use, that is, which exact entries in the Nix store
(`/nix/store`). Libraries that are needed by an application are defined as
`buildInputs` in the Nix expression of the application. When building/installing
the application, the libraries are also built/installed. You won't ever manually
install libraries using `nix-env -iA` or in `environment.systemPackages`.

But now let's consider Python. You might want to install the interpreter
system-wide, along with maybe some Python packages. As user, you realise you
want to have some additional packages so you install them using `nix-env -iA`.
Remember, with Nix, you can have multiple versions of libraries because
different applications might require different versions. How now, would the
interpreter decide which version of say [`numpy`](http://www.numpy.org/) to use, when multiple are
installed?

The bottomline is that **installing Python and packages is not supported**.
The way to go though is environments...


### Python using nix-shell

Perhaps the easiest way to get a functional Python environment is by using
[`nix-shell`](http://nixos.org/nix/manual/#sec-nix-shell).

Executing

    $ nix-shell -p python35Packages.numpy python35Packages.toolz

opens a Nix shell which has available the requested packages and dependencies.
Now you can launch the Python interpreter (which is itself a dependency)

    [nix-shell:~] python3

If the packages were not available yet in the Nix store, Nix would download or
compile them automatically.
A convenient option with `nix-shell` is the `--run` option, with which you can
execute a command in the `nix-shell`. Let's say we want the above environment
and directly run the Python interpreter

    $ nix-shell -p python35Packages.numpy python35Packages.toolz --run "python3"

You can also use the `--run` option to directly execute a script

    $ nix-shell -p python35Packages.numpy python35Packages.toolz --run "python3 myscript.py"

For this specific case there is another convenient method; you can add a shebang
to your script specifying which dependencies Nix shell needs. With the following
shebang, you can use `nix-shell myscript.py` and it will make available all
dependencies and run the script in the `python3` shell.

    #! /usr/bin/env nix-shell
    #! nix-shell -i python3 -p python35Packages.numpy python35Packages.toolz

The first line here is a standard shebang. We say we want to use `nix-shell`.
With the Nix shell you are however not limited to only a single line, but you
can have multiple. The second line instructs Nix shell to create an environment
with the `-p` packages in it, as we did before, and then run the `-i`
interpreter. Note that the `-i` option can only be used as part of a shebang. In
other cases you will have to use the `--run` option as shown above.

By default all installed applications are still accessible from the Nix shell.
If you do not want this, you can use the `--pure` option.

    $ nix-env -iA nixpkgs.pkgs.pandoc
    $ nix-shell -p python35Packages.numpy python35Packages.toolz --pure
    [nix-shell:~] pandoc
    The program ‘pandoc’ is currently not installed. You can install it by typing:
      nix-env -iA nixos.pandoc

Likely you do not want to type your dependencies each and every time. What you
can do is write a simple Nix expression which sets up an environment for you,
requiring you only to type `nix-shell`. Say we want to have Python 3.5, `numpy`
and `toolz`, like before, in an environment. With a `shell.nix` file
containing

    with import <nixpkgs> {};

    ( pkgs.python35.buildEnv.override  {
    extraLibs = with pkgs.python35Packages; [ numpy toolz ];
    }).env

executing `nix-shell` gives you again a Nix shell from which you can run Python.
So what do those lines here mean? Let's consider line by line:

1. We begin with importing the Nix Packages collections. `import <nixpkgs> {}` does the actual import and the `with` statement brings all attributes of `nixpkgs` in the local scope. Therefore we can now use `pkgs`.
2. Then we say we want a Python 3.5 environment, so we use the derivation [`pkgs.python35.buildEnv`](http://nixos.org/nixpkgs/manual/#ssec-python-build-env). Because we want to use it with a custom set of Python packages, we override it.
3. The `extraLibs` argument of the original `buildEnv` function can be used to specify which packages you want. We want `numpy` and `toolz`. Again, we use the `with` statement to bring a set of attributes into the local scope.
4. **EXPLAIN**


### Declarative environment using myEnvFun

Using Nix shell means you either need to add a bunch of arguments to the
`nix-shell` invocation, or executing a specific file with Nix shell. Another
option is to instead define your environments [declaratively in your user
profile](http://nixos.org/nixpkgs/manual/#chap-packageconfig). As user you have
a `~/.nixpkgs/config.nix` file in which you can include
[overrides](http://nixos.org/nixpkgs/manual/#sec-modify-via-packageOverrides)
specifically for yourself. Here we can add our declarative environments as well.
Let's say we already have the following `config.nix`.

    with <nixpkgs> {};
    {
    allowUnfree = true;
    }

This expression imports the Nix packages collections, and says that we allow
unfree software. Let's extend this now with two environments. We add one
environment that we use for development, and another for blogging with
[Pelican](http://blog.getpelican.com/).

    with <nixpkgs> {};
    {
    allowUnfree = true;
    allowBroken = true;

    packageOverrides = pkgs: with pkgs; {
        devEnv = pkgs.myEnvFun {
            name = "work";
            buildInputs = with python34Packages; [
              python34
              numpy
              toolz
            ];
        };

        blogEnv = pkgs.myEnvFun {
            name = "blog";
            buildInputs = with python27Packages; [
              python27
              pelican
              markdown
            ];
        };
    };
    }

For the first environment we want Python 3.4, and for the second Python 2.7.
Note that we have to explicitly include the interpreter when using `myEnvFun`!
We can install these environments with `nix-env -i env-<name>` and use them by
calling `load-env-<name>`. In both cases `<name>` is the argument `name` of the
function `myEnvFun`.

    $ nix-env -i env-work
    installing ‘env-work’

    $ load-env-work
    env-work loaded
    work:[~]$

You can now start the interpreter, `python3`.

### Missing Python modules?

At this point you might have gone ahead using the Nix shell or `myEnvFun` to create Python environments, and got some very surprising import errors, unrelated to those explained before.
If you haven't encountered these yet, try running

    $ nix-shell -p python27 --run python

and then

    >>> import sqlite3

You will notice that you get an `ImportError`.

    >>> import sqlite3
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/nix/store/v9pq6f0s1r5fdybqc7pbv7mkb33lx9yy-python-2.7.10/lib/python2.7/sqlite3/__init__.py", line 24, in <module>
        from dbapi2 import *
    File "/nix/store/v9pq6f0s1r5fdybqc7pbv7mkb33lx9yy-python-2.7.10/lib/python2.7/sqlite3/dbapi2.py", line 28, in <module>
        from _sqlite3 import *
    ImportError: No module named _sqlite3


What's going on here? In Nixpkgs the Python 2.x interpreters were sent on a diet. To [reduce dependency bloat](http://nixos.org/nixpkgs/manual/#sec-python) some modules were removed from the 2.x interpreters.
If you do want some of these modules, then you have to include them explicitly, e.g.

    $ nix-shell -p python27 python27.modules.sqlite3 --run python

to include the `sqlite3` module. For convenience, there is also a `python27Full` package which includes all these modules

    $ nix-shell -p python27Full python27Packages.numpy python27Packages.toolz --run python


### How to find Python packages?

So far we only considered two python packages, `numpy` and `toolz`. At this point you might be wondering how to find packages.
You can search for packages with `nix-env -qa`. The `-q` stands for query and `-a` for available derivations. Let's search for `numpy`

    $ nix-env -qa '.*numpy.*'
    pypy2.6-numpydoc-0.5
    python2.7-numpy-1.10.1
    python2.7-numpydoc-0.5
    python3.4-numpy-1.10.1
    python3.4-numpydoc-0.5
    python3.5-numpy-1.10.1
    python3.5-numpydoc-0.5

A tool that is generally easier to begin with is [Nox](https://github.com/madjar/nox). You can install Nox with `nix-env -i nox` or try it with `nix-shell -p nox`.
Let's search for `numpy` with Nox.

    $ nox numpy
    Refreshing cache
    1 pypy2.6-numpydoc-0.5 (nixos.pypyPackages.numpydoc)
	Sphinx extension to support docstrings in Numpy format
    2 python2.7-numpy-1.10.1 (nixos.python27Packages.numpy)
	Scientific tools for Python
    3 python2.7-numpydoc-0.5 (nixos.python27Packages.numpydoc)
	Sphinx extension to support docstrings in Numpy format
    4 python3.4-numpy-1.10.1 (nixos.python34Packages.numpy)
	Scientific tools for Python
    5 python3.4-numpydoc-0.5 (nixos.python34Packages.numpydoc)
	Sphinx extension to support docstrings in Numpy format
    6 python3.5-numpy-1.10.1 (nixos.python35Packages.numpy)
	Scientific tools for Python
    7 python3.5-numpydoc-0.5 (nixos.python35Packages.numpydoc)
	Sphinx extension to support docstrings in Numpy format
    Packages to install:

Nox provides, among other things, an easier interface to `nix-env` for querying and installing packages.
Nox shows you the name with version of packages, along with the Nix attribute, e.g. `nixos.python34Packages.numpy`.
The first part is the identifier of the channel, in this case `nixos`, since

    $ nix-channel --list
    nixos https://nixos.org/channels/nixos-unstable

Another example

    $ nox pandas
    1 pypy2.6-pandas-0.17.0 (nixos.pypyPackages.pandas)
	Python Data Analysis Library
    2 python2.7-pandas-0.17.0 (nixos.python27Packages.pandas)
	Python Data Analysis Library
    3 python3.4-pandas-0.17.0 (nixos.python34Packages.pandas)
	Python Data Analysis Library
    4 python3.5-pandas-0.17.0 (nixos.python35Packages.pandas)
	Python Data Analysis Library
    Packages to install:

### Alternative interpreters and shells

So far we considered only the CPython interpreter, but in the examples shown just before, you could see that packages for the PyPy interpreter also show up.
Indeed, you can use the PyPy interpreter on Nix as well

    $ nix-shell -p pypy --run pypy
    Python 2.7.9 (295ee98b69288471b0fcf2e0ede82ce5209eb90b, Sep 21 2015, 22:02:02)
    [PyPy 2.6.0 with GCC 4.9.3] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>>>

We can get an environment with PyPy just like we did before.

    $ nix-shell -p pypyPackages.numpy pypyPackages.toolz

but this results in an error

    error: numpy-1.10.1 not supported for interpreter pypy
    (use ‘--show-trace’ to show detailed location information)

Why is that? As the message explains, `numpy` is
[not supported](http://pypy.org/numpydonate.html) for `pypy`, just like many
other packages that include extension types.
This is however not a Nix issue, but a PyPy issue. Even so, you will encounter
these kind of errors more often, since also with CPython certain packages are
supported on certain versions, but not all.

Included in the Nix packages collection are also alternative Python shells, like Jupyter/IPython.
Say we want to use `numpy` and `toolz` again but now using the [IPython](http://ipython.org/) interpreter

    $ nix-shell -p python34Packages.ipython python34Packages.numpy python34Packages.toolz --run ipython
    Python 3.4.3 (default, Jan 01 1970, 00:00:01)
    Type "copyright", "credits" or "license" for more information.

    IPython 4.0.0 -- An enhanced Interactive Python.
    ?         -> Introduction and overview of IPython's features.
    %quickref -> Quick reference.
    help      -> Python's own help system.
    object?   -> Details about 'object', use 'object??' for extra details.

    In [1]:

We can also use the [Jupyter QtConsole](http://jupyter.org/qtconsole/stable/)

    nix-shell -p python34Packages.qtconsole --run "jupyter qtconsole"

or the [Jupyter Notebook](http://jupyter.org/)

    nix-shell -p python34Packages.notebook --run "jupyter notebook"

## Developing a Python package

Now that you know how to get a working Python environment on Nix, it is time to go forward and start actually developing with Python.
We will first have a look at how Python packages are packaged on Nix. Then, we will look how you can use development mode with your code.

### Python packaging on Nix

On Nix all packages are built by functions. The main function in Nix for building Python packages is [`buildPythonPackage`](https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/python-modules/generic/default.nix).
Let's see how we would build the `toolz` package. According to [`python-packages.nix`](https://raw.githubusercontent.com/NixOS/nixpkgs/master/pkgs/top-level/python-packages.nix) `toolz` is build using

    toolz = buildPythonPackage rec{
      name = "toolz-${version}";
      version = "0.7.4";

      src = pkgs.fetchurl{
        url = "https://pypi.python.org/packages/source/t/toolz/toolz-${version}.tar.gz";
        sha256 = "43c2c9e5e7a16b6c88ba3088a9bfc82f7db8e13378be7c78d6c14a5f8ed05afd";
      };

      meta = {
        homepage = "http://github.com/pytoolz/toolz/";
        description = "List processing tools and functional utilities";
        license = licenses.bsd3;
        maintainers = with maintainers; [ fridh ];
      };
    };

What happens here? The function `buildPythonPackage` is called and as argument
it accepts a set. In this case the set is a recursive set ([`rec`](http://nixos.org/nix/manual/#sec-constructs)). One of the
arguments is the name of the package, which consists of a basename (generally
following the name on PyPi) and a version. Another argument, `src` specifies the
source, which in this case is fetched from an url. `fetchurl` not only downloads
the target file, but also validates its hash. Furthermore, we specify some
(optional) [meta information](http://nixos.org/nixpkgs/manual/#chap-meta).

The output of the function is a derivation, which is an attribute with the name
`toolz` of the set `pythonPackages`. Actually, sets are created for all interpreter versions,
so `python27Packages`, `python34Packages`, `python35Packages` and `pypyPackages`.

The above example works when you're directly adding or modifying packages to `python-packages.nix`.
Often though, you will want to test a Nix expression outside of the Nixpkgs tree. If you create a `shell.nix` file with the following contents

    with import <nixpkgs> {};

    pkgs.python35Packages.buildPythonPackage rec {
      name = "toolz-${version}";
      version = "0.7.4";

      src = pkgs.fetchurl{
        url = "https://pypi.python.org/packages/source/t/toolz/toolz-${version}.tar.gz";
        sha256 = "43c2c9e5e7a16b6c88ba3088a9bfc82f7db8e13378be7c78d6c14a5f8ed05afd";
      };

      meta = {
        homepage = "http://github.com/pytoolz/toolz/";
        description = "List processing tools and functional utilities";
        license = licenses.bsd3;
        maintainers = with maintainers; [ fridh ];
      };
    }
and then execute `nix-shell` will result in an environment in which you can use
Python 3.5 and the `toolz` package. As you can see we had to explicitly mention
for which Python version we want to build a package.

Often though, you will want to use a package in environments together with other packages.
If we create a `shell.nix` file with the following contents

    with import <nixpkgs> {};

    ( let
        toolz = pkgs.python35Packages.buildPythonPackage rec{
          name = "toolz-${version}";
          version = "0.7.4";

          src = pkgs.fetchurl{
            url = "https://pypi.python.org/packages/source/t/toolz/toolz-${version}.tar.gz";
            sha256 = "43c2c9e5e7a16b6c88ba3088a9bfc82f7db8e13378be7c78d6c14a5f8ed05afd";
          };

          meta = {
            homepage = "http://github.com/pytoolz/toolz/";
            description = "List processing tools and functional utilities";
            license = licenses.bsd3;
            maintainers = with maintainers; [ fridh ];
          };
        };

    in pkgs.python35.buildEnv.override rec {

      extraLibs = [ numpy toolz ];
    }
    ).env

and again execute `nix-shell`, then we get a Python 3.5 environment with our
locally defined package as well as `numpy` which is build according to the
definition in Nixpkgs. What did we do here? Well, we took the Nix expression
that we used earlier to build a Python environment, and said that we wanted to
include our own version of `toolz`. To introduce our own package in the scope of
`buildEnv.override` we used a
[`let`](http://nixos.org/nix/manual/#sec-constructs) expression.

### Handling dependencies

So far, the example `toolz`, didn't have any dependencies on other Python
packages or system libraries. According to the manual, the `buildPythonPackage`
uses the arguments `buildInputs` and `propagatedBuildInputs`. If something is
exclusively a build-time dependency, then the dependency should be included as a
`buildInput`, but if it is (also) a runtime dependency, then it should be added
to `propagatedBuildInputs`.

The following example shows which arguments are given to `buildPythonPackage` in order to be build [`datashape`](https://github.com/blaze/datashape).

    datashape = buildPythonPackage rec {
      name = "datashape-${version}";
      version = "0.4.7";

      src = pkgs.fetchurl {
        url = "https://pypi.python.org/packages/source/D/DataShape/${name}.tar.gz";
        sha256 = "14b2ef766d4c9652ab813182e866f493475e65e558bed0822e38bf07bba1a278";
      };

      buildInputs = with self; [ pytest ];
      propagatedBuildInputs = with self; [ numpy multipledispatch dateutil ];

      meta = {
        homepage = https://github.com/ContinuumIO/datashape;
        description = "A data description language";
        license = licenses.bsd2;
        maintainers = with maintainers; [ fridh ];
      };
    };

We can see several runtime dependencies, `numpy`, `multipledispatch`, and
`dateutil`. Furthermore, we have one `buildInput`, i.e. `pytest`. `pytest` is a
test runner and is only used during the `checkPhase` and is therefore not added
to `propagatedBuildInputs`.

In the previous case we had only dependencies on other packages to consider.
Occasionally you have also system libraries to consider. E.g., `lxml` provides
Python bindings to `libxml2` and `libxslt`. These libraries are only required
when building the bindings and are therefore added as `buildInputs`.

    lxml = buildPythonPackage rec {
      name = "lxml-3.4.4";

      src = pkgs.fetchurl {
        url = "http://pypi.python.org/packages/source/l/lxml/${name}.tar.gz";
        sha256 = "16a0fa97hym9ysdk3rmqz32xdjqmy4w34ld3rm3jf5viqjx65lxk";
      };

      buildInputs = with self; [ pkgs.libxml2 pkgs.libxslt ];

      meta = {
        description = "Pythonic binding for the libxml2 and libxslt libraries";
        homepage = http://lxml.de;
        license = licenses.bsd3;
        maintainers = with maintainers; [ sjourdois ];
      };
    };

In this example `lxml` and Nix are able to work out exactly where the relevant
files of the dependencies are. This is not always the case.

The example below shows bindings to The Fastest Fourier Transform in the West, commonly known as
FFTW. On Nix we have separate packages of FFTW for the different types of floats
(`"single"`, `"double"`, `"long-double"`). The bindings need all three types,
and therefore we add all three as `buildInputs`. The bindings don't expect to
find each of them in a different folder, and therefore we have to set `LDFLAGS`
and [`CFLAGS`](https://en.wikipedia.org/wiki/CFLAGS).

    pyfftw = buildPythonPackage rec {
      name = "pyfftw-${version}";
      version = "0.9.2";

      src = pkgs.fetchurl {
        url = "https://pypi.python.org/packages/source/p/pyFFTW/pyFFTW-${version}.tar.gz";
        sha256 = "f6bbb6afa93085409ab24885a1a3cdb8909f095a142f4d49e346f2bd1b789074";
      };

      buildInputs = [ pkgs.fftw pkgs.fftwFloat pkgs.fftwLongDouble];

      propagatedBuildInputs = with self; [ numpy scipy ];

      # Tests cannot import pyfftw. pyfftw works fine though.
      doCheck = false;

      preConfigure = ''
        export LDFLAGS="-L${pkgs.fftw}/lib -L${pkgs.fftwFloat}/lib -L${pkgs.fftwLongDouble}/lib"
        export CFLAGS="-I${pkgs.fftw}/include -I${pkgs.fftwFloat}/include -I${pkgs.fftwLongDouble}/include"
      '';

      meta = {
        description = "A pythonic wrapper around FFTW, the FFT library, presenting a unified interface for all the supported transforms";
        homepage = http://hgomersall.github.com/pyFFTW/;
        license = with licenses; [ bsd2 bsd3 ];
        maintainer = with maintainers; [ fridh ];
      };
    };

Note also the line `doCheck = false;`, we explicitly disabled running the test-suite.

### Developing with Nix shell

When building a package, Nix sequentially goes through a series of
[phases](http://nixos.org/nixpkgs/manual/#sec-stdenv-phases). Separate phases
exist for unpacking the source code, patching it, building it and
installing it. `buildPythonPackage`
[modifies](http://nixos.org/nixpkgs/manual/#ssec-build-python-package) these
generic phases slightly in order to use the Python infrastructure.

If you tested the above examples, then hopefully everything ran without
problems. Unfortunately, that is not always the case. Sometimes you encounter
problems when building a package. When this happens Nix aborts the build, and
removes all build data. This can be annoying, especially when after a long time
of compiling just a single test fails and Nix decides to abort (example: `scikitlearn`).

Luckily there is the `-K` option to





meaning sometimes there's maybe a single test failing, and youE.g., if you're building a Python package which took a long time to compile (e.g. `scikitlearn`) In such cases you might want to quickly change


### Develop local package

As a Python developer you're likely aware of [development mode](http://pythonhosted.org/setuptools/setuptools.html#development-mode) (`python setup.py develop`);
instead of installing the package this command creates a special link to the project code.
That way, you can run updated code without having to reinstall after each and every change you make.
Development mode is also available on Nix as [explained](http://nixos.org/nixpkgs/manual/#ssec-python-development) in the Nixpkgs manual.
Let's see how you can use it.

In the previous Nix expression the source was fetched from an url. We can also refer to a local source instead using

    src = ./path/to/source/tree;

Now, if we create a `shell.nix` file which calls `buildPythonPackage`, and if `src`
is a local source, and if the local source has a `setup.py`, then development
mode is activated.

In the following example we create a simple environment that
has a Python 3.5 version of our package in it, as well as its dependencies and
other packages we like to have in the environment, all specified with `propagatedBuildInputs`.
Indeed, we can just add any package we like to have in our environment to `propagatedBuildInputs`.


    with import <nixpkgs>;
    with pkgs.python35Packages;

    buildPythonPackage rec {
        name = "mypackage";
        src = ./path/to/package/source;
        propagatedBuildInputs = [ pytest numpy pkgs.libsndfile ];
    };

It is important to note that due to how development mode is implemented on Nix it is not possible to have multiple packages simultaneously in development mode.


## Organising your packages

So far we discussed how you can use Python on Nix, and how you can develop with
it. We've looked at how you write expressions to package Python packages, and we
looked at how you can create environments in which specified packages are
available.

At some point you'll likely have multiple packages which you would
like to be able to use in different projects. In order to minimise unnecessary
duplication we now look at how you can maintain yourself a repository with your
own packages. The important functions here are `import` and `callPackage`.

In previous examples we used `import` generally in combination with the `with`
statement, thereby introducing the attributes of the imported attribute set into
the local scope. We can also simply assign the imported attribute set using a `let` expression.

Consider the following `shell.nix` file



## Python on Nix internals


### Important files and folders

  - [`buildPythonPackage`](https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/python-modules/generic/default.nix) is a function to build Python packages.
  - [`python-packages.nix`](https://raw.githubusercontent.com/NixOS/nixpkgs/master/pkgs/top-level/python-packages.nix) is a Nix expression listing most of Python packages that are available in Nix.
  - Folder with [expressions for Python interpreters](https://github.com/NixOS/nixpkgs/tree/master/pkgs/development/interpreters/python).
   - ['wrapper.nix'](https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/interpreters/python/wrapper.nix) wraps Python binaries.

## FAQ

### How can I prevent my packages from being garbage-collected?

So you're using `nix-shell` and whenever you run `nix-collect-garbage -d` your packages are gone?

With `nix-shell`...

On NixOS the `system.extraDependencies` option for `configuration.nix` exists.
Packages added here are added to the Nix store, but not made available to users.

    system.extraDependencies = with pkgs.python35Packages; [
      numpy
      toolz
    ];


### Why should I use buildEnv for creating environments instead of buildPythonPackage or mkDerivation?

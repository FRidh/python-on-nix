# Python on Nix

You're Python dev, and recently you've heard about [Nix](http://nixos.org/nix/).
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
then head on to the [Quick
Start](http://nixos.org/nix/manual/#chap-quick-start).
if instead you want to install NixOS, then please continue now with the [NixOS
installation guide](http://nixos.org/nixos/manual/index.html#ch-installation).

## Using Python

### Installing software

Now that you've installed Nix you're ready to start installing software.
With the Nix package manager you can ad hoc install software in your profile
using `nix-env -iA`. e.g.

    $ nix-env -iA nixpkgs.pkgs.pandoc

would install the Pandoc tool in your profile. From then on you will have the
possibility to run `pandoc` from within a shell.

If you are running NixOS and you have `root`/`sudo` access, then you can specify
which packages need to be installed by appending those packages to
`environment.systemPackages` in the `/etc/nixos/configuration.nix` file

    ...

    environment.systemPackages = with pkgs; [
      busybox
      chromium
      pandoc
    ];

    ...

A common method on Nix is however not to install all the software you need, but
instead using the `nix-shell` to open a shell with exactly those packages that
you need when you need them.
E.g., say you want to convert a some files and so you want to use Pandoc, then
you could run

    $ nix-shell -p pandoc

which opens a shell from which you can run pandoc

    [nix-shell:~] pandoc tutorial.md -o tutorial.pdf


### Installing Python?

You might be wondering now why, if this tutorial is about Python, we are using
Pandoc as an example and not Python?
Well, that's because with Python you're generally interested in not just the interpreter, but also Python packages.
On Nix most software can be installed in a profile, either
ad hoc or declaratively. However, this is not possibly with Python and packages.
Actually, to be precise, the tools allow you to install to your profile e.g.
Python 3.5 using

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
interpreter decide which version of say `numpy` to use, when multiple are
installed?

The bottomline is that simply installing Python and packages is not supported.
The way to go though is environments...


### Python using nix-shell

Perhaps the easiest way to get a functional Python environment is by using `nix-shell`.
Running

    $ nix-shell -p python35Packages.numpy python35Packages.toolz

opens a Nix shell from which you can launch Python

    [nix-shell:~] python35

A convenient option with `nix-shell` is the `-i` flag, with which you can directly launch another shell.

    $ nix-shell -i python35 python35Packages.numpy python35Packages.toolz


## Developing a Python package

When developing a Python package, one commonly uses `python setup.py develop`.
The package is build



### buildPythonPackage




## Python on Nix internals



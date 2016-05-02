# python-on-nix

This repository contains a [tutorial](http://python-on-nix.readthedocs.org/en/latest/) to [Python](https://www.python.org/) on [Nix](http://nixos.org/nix/).

## Building the docs

Documentation is build from CommonMark/Markdown and ReStructuredText using [Sphinx](http://sphinx-doc.org/).
When using Nix/NixOS, just run `nix-shell` in this directory to obtain a shell with all dependencies.
To build html docs, just run `make html`.

## Merged into Nixpkgs documentation

These docs are not maintained anymore since they we're merged into the Nixpkgs manual.
See http://hydra.nixos.org/job/nixpkgs/trunk/manual/latest


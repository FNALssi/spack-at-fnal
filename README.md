# spack-at-fnal
Documentation describing Spack use at Fermilab.
Now rendered to [fnalssi.github.io](https://fnalssi.github.io/spack-at-fnal/).

## Set up the documentation environment

Install [Miniforge](https://github.com/conda-forge/miniforge) or another Conda
distribution, and ensure that `conda` is initialized in your shell. Git must
also be available for CMake to fetch some packages upon which the build depends.

From the root of this repository, create and activate the environment:

```sh
conda env create --name for-spack-at-fnal --file environment.yml
conda activate for-spack-at-fnal
```

The `environment.yml` file lists the tools required to build the HTML
documentation, including CMake, Ninja, Sphinx, and Graphviz. The example
environment name is `for-spack-at-fnal`; you may choose your own name by
replacing it in the creation command and all activation commands.

In a new shell, activate the existing environment before building:

```sh
conda activate for-spack-at-fnal
```

## Build the HTML documentation

With the environment active, run these commands from the repository root:

```sh
cmake --preset=default -G Ninja -S . -B build \
  -DSPHINX-BUILD_EXECUTABLE="$CONDA_PREFIX/bin/sphinx-build"
cmake --build build --target doc
```

CMake uses Sphinx from the active Conda environment. It fetches Cetmodules
during configuration, so the first configuration requires network access.
Use an empty build directory for the initial
configuration. You may choose another build directory by replacing `build`
in both commands.

Open `build/doc/html/index.html` in a web browser to view the documentation.
After editing the sources, run `cmake --build build --target doc` again to
update the HTML.

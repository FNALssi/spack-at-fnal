# spack-at-fnal
Documentation describing Spack use at Fermilab.

## Set up to build spack-at-fnal

## On a Mac

### Build prerequisites

* Building prerequisites will be _much_ faster if you have Spack
  installed, and `git`, `cmake` (at least v3.24) and `gmake` installed
  via [homebrew](https://brew.sh/).
  
* For a full list of dependencies, see `spack.yaml`.

* A Python version in the range 3.10--3.12 (spack does not yet support 3.13).
  We recommend

      brew install python@3.12

  Look at the output of `python3 -V`. If it is in the range above, all is well.
  If it is not, then export the environment variable `SPACK_PYTHON` to point at
  the correct executable, e.g.:

      export SPACK_PYTHON=python3.12

```
# This will make several directories under your current working directory.
# You should first `cd` to the directory under which you want these new
# directories to be made. A new, empty directory is recommended.

# One environment variable should be set to define the location of your
# clone of the spack_at_fnal repository.

export SPACK_AT_FNAL_DIR=/dev/null  # set this correctly for your own installation

export SPACK_DISABLE_LOCAL_CONFIG=true
export SPACK_USER_CACHE_PATH=`pwd`/spack-user-cache
export SPACK_PYTHON=python3.12  # or a different version, if you must

git clone https://github.com/FNALssi/spack  # we use our own clone
git clone https://github.com/FNALssi/fnal_art.git  # repo for our recipes

source spack/share/spack/setup-env.sh
spack compiler find
spack bootstrap now
# spack external find cmake git gmake ninja
spack repo add fnal_art

spack mirror add --type binary --signed --scope site fnal-develop https://scisoft.fnal.gov/scisoft/spack-mirror/spack-fnal-develop
spack buildcache keys -it

spack env create docgen "${SPACK_AT_FNAL_DIR}"/spack.yaml
spacktivate docgen
spack concretize
spack install
```

Note that after building the environment, you should first deactivate (`despacktivate`) and then reactivate the environment to ensure that the newly-built packages are available in your `PATH` in the current shell.

### Set up an already-created installation

```
# cd to the working directory that you chose, above.
# It will have the build, fnal_art, spack, and spack-user-cache
# subdirectories.

export SPACK_AT_FNAL_DIR=/dev/null  # set this correctly for your own installation
export SPACK_DISABLE_LOCAL_CONFIG=true
export SPACK_USER_CACHE_PATH=`pwd`/spack-user-cache
export SPACK_PYTHON=python3.12  # use the same as you did above
source spack/share/spack/setup-env.sh
spacktivate docgen
```

### Build spack-at-fnal locally

```
mkdir <build-dir>
cd <build-dir>
cmake [-GNinja] -DBUILD_DOCS:BOOL=YES "${SPACK_AT_FNAL_DIR}"
ninja
```

Top level index HTML will be found as `doc/html/index.html`.

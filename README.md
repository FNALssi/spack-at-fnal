# spack-at-fnal
Documentation describing Spack use at Fermilab.

## Setting up to build spack-at-fnal

## On a Mac

## Building prerequisites

* Building prerequisites will be _much_ faster if you have Spack
  installed, and `git`, `cmake` (at least v3.24) and `gmake` installed
  via [homebrew](https://brew.sh/).
  
* For a full list of dependencies, see `spack.yaml`.

```
git clone https://github.com/spack/spack.git
export SPACK_DISABLE_LOCAL_CONFIG=true
export SPACK_USER_CACHE_PATH=~/scratch/spack-user-cache
git clone https://github.com/FNALssi/fnal_art.git
cd spack
. share/spack/setup-env.sh
spack compiler find
spack bootstrap now
spack external find apple-libuuid cmake git gmake ninja
spack repo add ../fnal_art
spack env create docgen <spack-at-fnal-src>/spack.yaml
spacktivate spack.yaml
spack concretize
spack install
```

### Building spack-at-fnal locally

```
export SPACK_DISABLE_LOCAL_CONFIG=true
export SPACK_USER_CACHE_PATH=~/scratch/spack-user-cache
. <spack-install-dir>/share/spack/setup-env.sh
spacktivate doc
mkdir <build-dir>
cd <build-dir>
cmake [-GNinja] -DBUILD_DOCS:BOOL=YES <spack-at-fnal-src>
ninja
```

Top level index HTML will be found as `doc/html/index.html`.

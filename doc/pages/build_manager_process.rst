===========================================================================
Build Manager Processes for Experiment software using CSAID Shared Software
===========================================================================

The Fermilab CSAID directorate develops various software packages that are shared by multiple experiments, and built and distributed using the Spack package manager.
These packages are used to build experiment software stacks.
To allow a single reference build of those software packages to work for our various experiment customers, we request they follow the following procedures when building and distributing their software.

This document will discuss the files that the experiment needs to maintain to facilitate these shared Spack builds as well as experiment software releases, as well as how to distribute the packages thus built to CVMFS.
For purposes of illustration, this document will refer to the Hypothetical Experiment **hypot**: filenames, etc. using the string **hypot** would, for any particular experiment, use their own experiment name in place of **hypot**.

Maintaining build environment configuration files and build recipes
===================================================================

This will involve (at least) 2 Git repositories (besides the repositories for the experiment code packages themselves)

* one for Spack build recipes, and 
* one for Spack build environment configuration files and fragments.

These two repositories for each experiment, properly configured, will allow CSAID to build releases of shared packages that can be reused by all the various groups, by combining them all into a *global solve*.
This *global solve* is the mechanism by which Spack can ensure that we produce a single consistent set of packages that will work together and which satisfy the set of constraints imposed by the recipes of all the different participating experiments.

N.B. For finding new versions, Spack works best with dotted versions for packages. It is recommended that dotted tags, e.g. XX.YY.ZZ.PP, be added at the same time that vXX_YY_ZZ_PP tagged versions are added.

Build environment configuration files
-------------------------------------

The reference builds and the various experiment software stacks should be built in Spack build environments [1]_, which will have configuration files (`spack.yaml`) which include (by URL) multiple shared config fragments, which are owned and controlled by the various organizations whose software is being included. 
In particular we expect each experiment to have, under version control, a build configuration repository of at least 3 files:

* A file defining lists of specs `hypot_specs.yaml`
* A file listing package version and variant requirements, `hypot_packages.yaml`
* An actual build configuration file which includes the above, along with including similar files for the **larsoft** and **fife** projects, etc., needed.

These files should be under version control in order to provide different branched versions of those files for different purposes, to be explained in more detail below.

The files above for a particular experiment should be in a repository controlled by that experiment.
Similarly there will be build configuration files for the various CSAID projects, like **larsoft**, **phlex**, **art**, **fife**, etc.

These configuration repositories should have branches with variants of the above files, allowing checking out a version of the above files which use `@develop` (or similar tags) for the version, (or building CI test builds, etc.), as well as tagged versions for their various software releases.

We will now discuss the format and usage of these various files in more detail.

Specs files `hypot_specs.yaml`
------------------------------

These files will be YAML [2]_ files whose contents will be a dictionary named *definitions*, defining a list of *specs*.
These are usually just as package names, for example:

.. code-block:: yaml

  definitions:
  - hypot_specs:
    - hypotcode
    - hypotreco
    - hypotutils
    ...

This definition list will be used in the experiment build configuration file, but also by the CSAID teams building the share packages in a *global solve*. 

Packages files `hypot_packages.yaml`
------------------------------------

These YAML [2]_ files will contain a dictionary named *packages*, mapping package names to requirements lists, for example:

.. code-block:: yaml

  packages:
    hypotcode:
      require:
      - '@=1.2.02:'
    hypotreco:
      require:
      - '@=1.3.01:'
    hypotutils:
      require:
      - '@=1.2.09:'
    art:
    require:
    - '@=3.14:'
  boost:
    require:
    - '+iostreams+json'

Note that this file may add requirements to non-experiment packages (i.e. **boost** and **art**, above), if they are required for the experiment software. 
Version restrictions (like the above example for **art**) should really be included in the dependency definitions in the appropriate package's recipe file; but can be included here if needed.

.. [1] https://spack.readthedocs.io/en/latest/environments_basics.html

.. [2] https://en.wikipedia.org/wiki/YAML

The Build configuration file
----------------------------

This will be an Spack environment `spack.yaml` file, with includes of the various fragments, above, and combines the various definitions from those files.  For example:

.. code-block:: yaml

  spack:
    config:
      deprecated: true
    include:
    # Toolchain definition.
    # Defines the compiler (gcc or llvm/clang) and version.
    - path: https://raw.githubusercontent.com/art-framework-suite/art-release-configs/refs/tags/art-3.14.04/included_yaml/gcc-12-toolchain.yaml
      sha256: b769ddee1cff92c88b22cc968bc192a37d5fe863e6e224aeac5697a89480c5ca

    # Packages definitions.
    # Hypot packages
    - path: https://raw.githubusercontent.com/Hypot/hypot-release-configs/b5d45909e3c3e2efb9930798df4ba2363986f42b/included_yaml/hypot-packages.yaml
      sha256: f36085e9de736feb765305bebc170593d767a046c94b5489f4ca0082ab4d6754
    # These are version and variant requirements for a package suite
    - path: https://raw.githubusercontent.com/LArSoft/larsoft-release-configs/refs/tags/larsoft-10.11.01/included_yaml/larsoft-packages.yaml
      sha256: 79189dce1ceee3db1672da87b2af878c113f28f5041e7b29f4ff755d9a017f94
    - path: https://raw.githubusercontent.com/NuSoftHEP/nusofthep-release-configs/refs/tags/nusofthep-3.17.01/included_yaml/nusofthep-packages.yaml
      sha256: 2d8dea0b6eabcd23d54e4db44a2e20aa4a7742b06a4107bca5ca3d1bc02f29e5
    - path: https://raw.githubusercontent.com/art-framework-suite/art-release-configs/refs/tags/art-3.14.04/included_yaml/packages-art.yaml
      sha256: 32bfd398fcb27780263e6d593decbad885995b43852ccaf15788d82c7aed49de
    - path: https://github.com/fnal-fife/fife-release-configs/raw/refs/heads/main/included_yaml/fife_packages.yaml
      sha256: b7b7ef7ceaaa947c2cead036409f04aac94f5de357f6b7407699d00d511cfaa3


    # Spec definitions.
    # These are definitions of the root specs for a package suite
    - path: https://raw.githubusercontent.com/LArSoft/larsoft-release-configs/refs/tags/larsoft-10.11.01/included_yaml/larsoft-specs.yaml
      sha256: a3d4c8646e15a02b284150fe67c9d582ef73c3c4a70f58cc17cbb40334af5752
    - path: https://raw.githubusercontent.com/NuSoftHEP/nusofthep-release-configs/refs/tags/nusofthep-3.17.01/included_yaml/nusofthep-specs.yaml
      sha256: d9c0e0827d133ac6e44c207352baca83bb8d601ecbed260b5980d400571a0e94
    - path: https://raw.githubusercontent.com/art-framework-suite/art-release-configs/refs/tags/art-3.14.04/included_yaml/specs-art.yaml
      sha256: 16e776ee6fea578385fe5ccfd67924fdb442f9219ea0cf3beaad5f6a211b7281
    - path: https://github.com/fnal-fife/fife-release-configs/raw/refs/heads/main/included_yaml/fife_specs.yaml
      sha256: b0306775ce16510f7d099d597b3e07cd118f0a178362fdaea0273a8b788eca37 
    # Hypot specs
    - path: https://raw.githubusercontent.com/Hypot/hypot-release-configs/b5d45909e3c3e2efb9930798df4ba2363986f42b/included_yaml/hypot-specs.yaml
      sha256: 61739bad1be5af0aafdc287cd1263652a4bdfaa7a413b8d6fa2a3135381f6486

    upstreams:
      cvmfs-larsoft:
        install_tree: /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/opt/spack
    specs:
      - $specs_art
      - $specs_nusofthep
      - $specs_larsoft
      - $fife_specs
      - $hypot_specs
    concretizer:
      unify: true
      reuse:
        from:
        - type: environment
          path: /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/var/spack/environments/larsoft-v10_11_01-unified-cuda-python-3_10-trimmed-m3
    packages:
      all:
        providers:
          libc:
          - glibc
          'zlib-api:':
          - zlib
        variants:
        - generator=ninja
        - build_type=Release
        - cxxstd=17
        target:
        - x86_64_v3

This file includes the various organizations' specs and packages files, and combines the various definitions from the specs files into the `spec:` list for this build.
It also sets concretizer and upstreams values to reuse the packages from the CVMFS Larsoft area in the build.
Note that the includes all include the SHA256 hashes of the included files, to be sure we're using the expected versions.

Experiment Build Recipe Repository
----------------------------------

The experiment should have a Spack build recipe repository, with the build/install recipes for the experiment packages, and any third-party packages used by only this experiment, and not available in the main Spack builtin repository.
Spack `documents <https://spack.readthedocs.io/en/latest/repositories.html>`__ the recipe repository structure in some detail, as well as the recipe creation `process <https://spack.readthedocs.io/en/latest/packaging_guide_creation.html>`__ and we have some Fermi specific `recommendations <https://github.com/FNALssi/fermi-spack-tools/wiki/Spack-recipe-guidance-for-CMake-packages>`__
for CMake recipes using Fermi cetmodules, etc.  Experiment repositories should be maintained by the experiment, and included in the fermi-spack-tools configs installed by our bootstrap / make_spack scripts.

Build process
=============

With some version of the above files and repositories available, building a version of the software consists of:

* preparing a Spack build instance if you don't have one
* creating a build environment for this release
* updating a `hypot_package.yaml`  with suitable version information
* putting the Build config / `spack.yaml` file in the environment
* doing a `spack concretize` and `spack install` to do a test build of the software
* committing and tagging the versions of the `hypot_package.yaml` and build config `spack.yaml` files used in the build 
* doing an actual release build, using the checksummed, tagged, version of the files from the from the experiment config repository.
* making a *buildcache* of the built binary packages
* installing the buildacache images into CVMFS package areas.

We will now discuss these steps in more detail.

Preparing a Spack build instance
--------------------------------

There are two approaches to setting up a Spack build instance:

* a standalone instance or
* a *subspack* chained instance

which we will discuss below; in either case we want to configure such an instance which:

* is in a read-write file system
* is configured with Spack path-padding so binary packages are redistribable
* knows about appropriate compilers
* has signing keys installed for signing binary packages for distribution

A *subspack* / chained instance will generally take less disk space and setup faster than a standalone instance.
However, if you don't have access to an existing intance to base it from, or you want to be insulated from the compilers available, existing packages, etc., in the existing instance, you may find a standalone instance preferable.


Creating a Standalone instance
------------------------------

At Fermilab, we recommend using our `bootstrap` script from our fermi-spack-tools package.
If you go to the `fermi-spack-tools wiki <https://github.com/FNALssi/fermi-spack-tools/wiki#getting-started>`__ you will find links to download the latest bootstrap script, which you can copy the link for in your browser and paste into a terminal to get the latest version, and use like this:

.. code-block:: shell

  $ wget https://github.com/FNALssi/fermi-spack-tools/raw/refs/heads/fnal-v1.1.1/bin/bootstrap
  $ sh ./bootstrap --with_padding /path/to/location

where you give a path to where you want the spack instance to appear.

You then want to make sure to install the compilers you want into your standalone instance. 
Unless you ahve a lot of time to kill, you probalby want to install one that is already built from the build cache.

.. code-block:: shell

  source /path/to/location/setup-env.sh
  spack buildcache list -al gcc llvm
  spack install --cache-only gcc@1.2.3


Creating a subspack instance
----------------------------

If you have access to one of our existing cvmfs Spack instances that is running Spack 1.0 or later, you can use our `spack subspack` extension to make a chained spack instance which shares the existing instance's packages, but lets you install new things into the local, writable one.

.. code-block:: shell

   source /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/setup-env.sh
   spack subspack --with-padding /path/to/location

In this subspack, you will already have access to the compilers that are available in the upstream instance, so you'll only need to install a compiler if you want to use one that isn't already in that upstream instance.

Setting up the Spack instance
-----------------------------

Both of the above will give you a writable Spack instance at `/path/to/location` with our standard Fermi path padding turned on, for building redistributable binary packages.
Having created the instance, you need to set it up, and then we can create the build environment

.. code-block:: shell

  source /path/to/location/setup-env.sh
  spack env create build_hypotcode_vx_y

Getting Ready for a build
-------------------------

We can now populate our build environment with the `spack.yaml` file for our build, and concretize it and install.
In a perfect world, this looks like:
  
.. code-block:: shell

  spack env activate build_hypotcode_vx_y
  spack cd --env
  wget https://github.com/hypot/hypot-release-configs/blob/main/hypot-release.yaml
  mv hypot-release.yaml spack.yaml
  spack concretize | tee conc.out
  spack install

However, we probalby wish instead to customize the `spack.yaml` file, and possibly several of the included files, to be ready for the next release.  
Most frequently, this involves customizing the `hypot_packages.yaml` file that the release file gets from the repository.   
To avoid repeated updates of the config file and checksums to test a new version number, etc. you want to download that file from Git as well, and change your `spack.yaml` file to use the local copy. 

.. code-block:: shell

  wget https://github.com/hypot/hypot-release-configs/blob/main/included_yaml/hypot-packages.yaml

Then you want to edit the `spack.yaml` file and change it to use the local file instead; this changes the entry under include: to look like:

.. code-block:: yaml

  include:

    # Toolchain
    - path: https://raw.githubusercontent.com/art-framework-suite/art-release-configs/refs/heads/main/included_yaml/gcc-12-toolchain.yaml
      sha256: b769ddee1cff92c88b22cc968bc192a37d5fe863e6e224aeac5697a89480c5ca

    # hypot
    #- path: https://raw.githubusercontent.com/hypot/hypot-release-configs/refs/heads/main/included_yaml/hypot-packages.yaml
    #  sha256: f36085e9de736feb765305bebc170593d767a046c94b5489f4ca0082ab4d6754
    - ./hypot-packages.yaml

You see above, we have commented out both the URL path and the sha256 checksum line, and just put the local filename in its place.   
Now we can update version numbers, add or remove required variants, etc. to assist in getting the build we want, without a lot of git commit/push operations and sha256 hash updates.

Once we have a version of the `hypot_packages.yaml` file we like, we can commit it to our config repository, compute the new sha256 sum for it, commit the `spack.yaml` file to the repository, and tag it, and do a build which is getting the tagged config file from the repository. 

Adding built packages to a buildcache
=====================================

Once you have packages built, you can make signed *buildcache* images, and upload them to the appropriate *buildcache* mirror directory. 
First you will need a *GnuPG* signing key installed in your spack instance, if you don't have one already.

Installing a signing key
------------------------

If this is a new Spack build instance, you likely do not have any signing keys installed in it to sign your packages.  
If your experiment has a *pgp* key or key(s) for signing official binaries already, you can add it; if you don't have one you can create one.

.. code-block:: shell

  # making a new key
  spack gpg create --comment "HYPOT official package signing key" hypotpro hypotpro@fnal.gov
  spack export public-key-file hypotpro@fnal.gov
  spack export --secret secret-key-file hypotpro@fnal.gov
  scp secret-key-file public-key-file somehost:/some/place/safe
  rm secret-key-file public-key-file

  # adding an existing key
  scp somehost:/some/place/safe/*-key-file .
  spack gpg trust public-key-file
  spack gpg trust secret-key-file
  rm secret-key-file public-key-file

where the `somehost:/some/place/safe` should be somwhere like `/opt/hypotpro` on one of the experiment gpvm machines, etc. and should be readonly to the experiment production account, or similar.

Or you may want to use a signing key specific to yourself, personally, and keep it wherever you keep such things.

Packing up your build for distribution
======================================

Now you can activate your spack build environment, and make signed buildcache images for the new build:

.. code-block:: shell

  spack env activate build_hypotcode_vx_y
  spack cd --env
  spack localbuildcache --key hypotcode@fnal.gov --local --not-bc --with-build-dependencies

This will 

* create a *buildcache* in the `bc` subdirectory
* sign the binaries with your hypotcode@fnal.gov *GnuPG* key
* only include packages from the local spack instance
* omit packages installed from a *buildcache* 
* include build dependencies, not just runtime ones

Moving the buildcache images to SciSoft
=======================================

Now (assuming you have suitable permissions) you can upload the *buildcache* files to SciSoft or spack-cache-1 and update the *buildcache* index.  


.. code-block:: shell

  spack env activate build_hypotcode_vx_y
  spack cd --env
  scp -r bc/* scisoftbuild02.fnal.gov:/SciSoft/spack-mirror/spack-binary-cache-v3
  ssh scisoftbuild02.fnal.gov <<EOF
   source /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/setup-env.sh
   spack buildcache update-index /SciSoft/spack-mirror/spack-binary-cache-v3
  EOF

Installing the built packges in /cvmfs
======================================

Now if you have a spack instance in /cvmfs, you can install these new packages into /cvmfs.  
(This assumes you have permissions to use the cvmfshypot@oasiscfs.fnal.gov account to update cvmfs)
The most effective way is to 
* start a cvmfs transaction
* recreate the environment from the `spack.lock` file from your build
* use `spack install --cache-only` to populate it
* publish the cvmfs transaction
This looks like:

.. code-block:: shell

  spack env activate build_hypotcode_vx_y
  spack cd --env
  scp spack.lock cvmfshypot@oasiscfs.fnal.gov:/tmp/hypotcode_vx_y.lock
  ssh cvmfshypot@oasiscfs.fnal.gov

  cvmfs_server transaction hypot.opensciencegrid.org
  source /cvmfs/hypot.opensciencegrid.org/spack-dir/setup-env.sh
  spack env create build_hypotcode_vx_y /tmp/hypotcode_vx_y.lock
  spack env activate build_hypotcode_vx_y
  spack install --cache-only --include-build-deps

  cd ; cvmfs_server publish hypot.opensciencegrid.org

Useful spack install options
----------------------------

In our above examples, we have generally just done a ``spack concretize`` and a ``spack install`` but there are numerous `options <https://spack.readthedocs.io/en/latest/command_index.html#spack-install>`__ to ``spack install`` which can be useful, including:

``--source``
    install source files as well as package
``--add``
    add the given spec to the current environment, concretize, and install
``--test {root,all}``
    run recipe defined tests on either the root package(s) only, or all installed packages
``--force``
    Re-concretize before installing.  Useful if you have changed a package requirement, etc. 


Automating builds with FNAL Jenkins and build-spack-env.sh
==========================================================

[DRAFT section, may not be included] 

The `fermi-spack-tools <https://github.com/FNALssi/fermi-spack-tools>` package has a ``build-spack-env.sh`` script that is very useful for doing automated Spack builds on our site Jenkins infrastructure -- it runs through pretty much the entire process described in this document of 

* setting up a spack instance, 
* fetchin an environment `spack.yaml` file from a given URL
* installing neccesary compilers
* creating an environment from the downloded `spack.yaml`
* concretizing it
* doing the `spack install` (with optional --test arguments)
* creating the *buildcache* images of the build as Jenkins artifacts

If you have gotten a Jenkins account (link needed), and have VPN, you can examine the project here (link needed) for an example of using this script. 

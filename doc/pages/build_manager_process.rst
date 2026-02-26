
Build Manager Processes for Experiment software using CSAID Shared Software
===========================================================================

The Fermilab CSAID division develops various software packages that are shared by multiple experiments, and built and distributed using the Spack package manager,  which are used to build experiment software frameworks, etc.
To allow a single reference build of those software packages to work for our various experiment customers, we request they follow the following procedures when building and distributing their software.
This document will discuss the files that the experiment needs to maintain to facilitate these shared Spack builds as well as experiment software releases, as well as how to distribute the packages thus built to CVMFS.
For purposes of illustration, this document will refer to the Hypothetical Experiment "hypot": filenames, etc. using the string "hypot" would, for any particular experiment, use their own experiment name in place of "hypot".


Maintaining build environment config files and build recipes
============================================================

This will involve (at least) 2 Git repositories (besides the repositories for the experiment code packages themselves)

* one for Spack build recipes and 
* one for Spack build environment config files and fragments

These two repositories for each experiment, properly configured, will allow CSAID to build releases of shared packages that can be reused by all the various groups, by combining them all into a "global solve".
More details on these below:

Build environment config files
------------------------------

Both the reference builds, and the various experiment frameworks, should be built in Spack build environments[1]_, which will have config files ("spack.yaml") which include (by URL) multiple shared config fragments, which are owned and controlled by the various organizaitions whose software is being included. 
In particular we expect each experiment to have, under version control, a build config repository of at least 3 files:

* A file defining lists of specs "hypot_specs.yaml"
* A file listing package version and variant requirements, "hypot_packages.yaml"
* An actual build configuration file which includes the above, along with including similar files for the larsoft and fife projects, etc. needed.

These files should be under version control in order to provide different branched versions of those files for different purposes, to be explained in more detail below.

The files above for a particular experiment should be in a repository controlled by that experiment.
Similarly there will be build configuration files for the various CSAID projects, like "larsoft", "phlex", "art", "fife", etc.

These config repositories should have branches with variants of the above files, allowing checking out a version of the above files which use "@develop" (or similar tags) for the version, (((((((((for building CI test builds, etc.), as well as tagged versions for their various software releases.

We will now discuss the format and usage of these various files in more detail.

Specs files hypot_specs.yaml
----------------------------

These files will be YAML[2]_ files whose contents will be a "definitions" dictionary defining a list of specs, usually just as package names, for example:

.. code-block:: yaml

  definitions:
  - hypot_specs:
    - hypotcode
    - hypotreco
    - hypotutils
    ...

This definition list will be used in the experiment build configuration file, but also by the CSAID teams building the share packages in a "global solve". 

Packages files hypot_packages.yaml
----------------------------------

These YAML[2]_ files will contain a "packages" dictionary mapping package names to requirements lists, for example:

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

Note that this file may add requirements to non-experiment packages (i.e. boost and art, above), if they are required for the experiment software. 
Version restrictions (like the above example for "art") should really be included in the dependency definitions in the appropriate package's recipe file; but can be included here if needed.

..[1] https://spack.readthedocs.io/en/latest/environments_basics.html
..[2] https://en.wikipedia.org/wiki/YAML

The Build config file
---------------------

This will be an Spack environment "spack.yaml" file, with includes of the various fragments, above, and combines the various definitions from those files.  For example:

.. code-block:: yaml

  spack:
    config:
      deprecated: true
    include:
    # Toolchain defintion.
    # Defines the compiler (gcc or llvm/clang) and version.
    - path: https://raw.githubusercontent.com/art-framework-suite/art-release-configs/refs/tags/art-3.14.04/included_yaml/gcc-12-toolchain.yaml
      sha256: b769ddee1cff92c88b22cc968bc192a37d5fe863e6e224aeac5697a89480c5ca

    # Packages defintions.
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
      - $specs_dune

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

This file includes the various organizations' specs and packages files, and combines the various definitions from the specs files into "spec:" list for this build.  It also sets concretizer and upstreams values to reuse the packages from the CVMFS Larsoft area in the build.  Note that the includes all include the SHA256 hashes of the included files, to be sure we're using the expected versions.`

Experiment Build Recipe Repository
----------------------------------

The experiment should have a Spack build recipe repository, with the build/install recipes for the experiment packages, and any third-party packages used by only this experiment, and not available in the main Spack builtin repository.
Spack [documents](https://spack.readthedocs.io/en/latest/repositories.html#structure-of-an-individual-package-repository) the recipe repository structure in some detail, as well as the recipe creation [process](https://spack.readthedocs.io/en/latest/packaging_guide_creation.html), and we have some Fermi specific [recommendations]() for CMake recipes using Fermi cetmodules, etc.  Experiment repositories should be maintained by the experiment, and included in the fermi-spack-tools configs installed by our bootstrap / make_spack scripts.

Build process
=============

With some version of the above files and repositories available, building a version of the software consists of:

* preparing a Spack build instance if you don't have one
* creating a build environment for this release
* updating a hypot_package.yaml  with suitable version information
* putting the Build config / spack.yaml file in the environment
* doing a spack concretize and spack install to do a test build of the software
* committing and tagging the versions of the hypot_package.yaml and build config spack.yaml files used in the build 
* doing an actual release build, using the checksummed, tagged, version of the files from the from the experiment config repository.
* making a buildcache of the built binary packages
* installing the buildacache images into CVMFS package areas.

We will now discuss these steps in more detail.

Preparing a Spack build instance
--------------------------------

There are two approaches to setting up a Spack build instance:

* a standalone instance or
* a "subspack" chained instance

which we will discuss below; in either case we want to configure such an instance which:

* is in a read-write file system
* is configured with Spack path-padding so binary packages are redistribable
* knows about appropriate compilers
* has signing keys installed for signing binary packages for distribution

A "subspack" / chained instance will generally take less disk space and setup faster than a standalone instance, but if you don't have access to an existing intance to base it from, or you want to be insulated from the compilers available, existing packages, etc. in the existing instance, you may find a standalone instance is your choice.


Creating a Standalone instance
------------------------------

At Fermilab, we recommend using our "bootstrap" script from our fermi-spack-tools package.
If you go to the `fermi-spack-tools wiki <https://github.com/FNALssi/fermi-spack-tools/wiki#getting-started>`_ you will find links to download the latest bootstrap script, which you can copy the link for in your browser and paste into a terminal to get the latest version, and use like this:

.. code-block:: shell

  $ wget https://github.com/FNALssi/fermi-spack-tools/raw/refs/heads/fnal-v1.1.1/bin/bootstrap
  $ sh ./bootstrap --with_padding /path/to/location

where you give a path to where you want the spack instance to appear.

Creating a "subspack" instance
------------------------------

If you have access to one of our existing cvmfs Spack instances that is running Spack 1.0 or later, you can use our "spack subspack" extension to make a chained spack instance which shares the existing instance's packages, but lets you install new things into the local, writable one.

.. code-block:: shell

   source /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/setup-env.sh
   spack subspack --with-padding /path/to/location

Setting up the Spack instance
-----------------------------

Both of the above will give you a writable Spack instance at /path/to/location with our standard Fermi path padding turned on, for building redistributable binary packages.
Having created the instance, you need to set it up, and then we can create the build environment

.. code-block:: shell

  source /path/to/location/setup-env.sh
  spack env create build_hypotcode_vx_y

Getting Ready for a test build
------------------------------

We can now populate our build environment with the spack.yaml file for our build, and concretize it and install.  In a perfect world, this looks like:
  
.. code-block:: shell

  spack env activate build_hypotcode_vx_y
  spack cd --env
  wget https://github.com/hypot/hypot-release-configs/blob/main/hypot-release.yaml
  mv hypot-release.yaml spack.yaml
  spack concretize | tee conc.out
  spack install

However, we wish instead to customize the spack.yaml file, and possibly several of the included files, to be ready for the next release.  
Most frequently, this involves customizing the hypot_packages.yaml file that the release file gets from the repository.   
To avoid repeated updates of the config file and checksums to test a new version number, etc. you want to download that file from Git as well, and change your spack.yaml file to use the local copy. 

.. code-block:: shell

  wget https://github.com/hypot/hypot-release-configs/blob/main/included_yaml/hypot-packages.yaml

Then you want to edit the spack.yaml file and change it to use the local file instead; this changes the entry under include: to look like:

.. code-block:: yaml

  include:

    # Toolchain
    - path: https://raw.githubusercontent.com/art-framework-suite/art-release-configs/refs/heads/main/included_yaml/gcc-12-toolchain.yaml
      sha256: b769ddee1cff92c88b22cc968bc192a37d5fe863e6e224aeac5697a89480c5ca

    # hypot
    #- path: https://raw.githubusercontent.com/hypot/hypot-release-configs/refs/heads/main/included_yaml/hypot-packages.yaml
    #  sha256: f36085e9de736feb765305bebc170593d767a046c94b5489f4ca0082ab4d6754
    - ./hypot-packages.yaml

You see above, we huave commented out both the URL path and the sha256 checksum line, and just put the local filename in its place.   
Now we can update version numbers, add/remove required variants, etc. to assist in getting the build we want, without a lot of git commit/push operations and sha256 hash updates.



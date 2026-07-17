==========================
Build Manager Instructions
==========================

A *build manager* is a designated person responsible for producing and publishing official builds of experiment or project software.

The Fermilab SCD division develops various software packages that are shared by multiple experiments, and built and distributed using the Spack package manager.
These packages are used to build experiment software stacks.
To allow a single *reference build* of those software packages to work for the many experiments we support, we ask experiment's build managers to follow the procedures below when building and distributing their software.

This document will discuss the files that the experiment needs to maintain to facilitate these shared Spack builds as well as experiment software releases, as well as how to distribute the packages thus built to CVMFS.
For purposes of illustration, this document will refer to the Hypothetical Experiment **hypot**: filenames, etc. using the string **hypot** would, for any particular experiment, use their own experiment name in place of **hypot**.

Maintaining build environment configuration files and build recipes
===================================================================

This will involve two Git repositories in addition to the repositories for the experiment code packages themselves:

* one for Spack build recipes, and 
* one for Spack build environment configuration files and fragments.

These two repositories for each experiment, properly configured, will allow SCD to build releases of shared packages that can be reused by all the various groups, by combining them all into a *global solve*.
This *global solve* is the mechanism by which Spack can ensure that we produce a single consistent set of packages that will work together and which satisfy the set of constraints imposed by the recipes of all the different participating experiments.

N.B. For finding new versions, Spack works best with dotted versions for packages.
It is recommended that dotted tags, e.g. XX.YY.ZZ.PP, be added at the same time that vXX_YY_ZZ_PP tagged versions are added to all experiment code repositories.

Build environment configuration files
-------------------------------------

The reference builds and the various experiment software stacks should be built in Spack build environments [#environments]_, which will have configuration files (`spack.yaml`) which include (by URL) multiple shared config fragments, which are owned and controlled by the various organizations whose software is being included. 
In particular we expect each experiment to have, under version control, a build configuration repository of at least two files:

* A file defining lists of specs `hypot-specs.yaml`
* A file listing package version and variant requirements, `hypot-packages.yaml`

We recommand an additional build configuration file `hypot_release.yaml` which directs the experiment's release build process.
It includes the two files above,

* An build configuration file which includes the above, along with including similar files for the **larsoft** and **fife** projects, etc., needed.

These files should be under version control in order to provide different branched versions of those files for different purposes, to be explained in more detail below.

The files above for a particular experiment should be in a repository controlled by that experiment.
Similarly there will be build configuration files for the various SCD projects, like **larsoft**, **phlex**, **art**, **fife**, etc.

These configuration repositories should have branches with variants of the above files, allowing checking out a version of the above files which use `@develop` (or whatever tag your experiment uses to mark the branch used for development) for the version, (or building CI test builds, etc.), as well as tagged versions for their various software releases.

We will now discuss the format and usage of these various files in more detail.

Specs files `hypot-specs.yaml`
------------------------------

These files will be YAML [#yaml]_ files whose contents will be a dictionary named *definitions*, defining a list of *specs*.
These are usually just as package names, for example:

.. code-block:: yaml

  definitions:
  - hypot_specs:
    - hypotcode
    - hypotreco
    - hypotutils
    - justin

This definition list will be used in the experiment build configuration file, but also by the SCD teams building the shared packages in a *global solve*. 

Note that in this example `justin` is not part of the Hypot experiment software, but is a third party package that needs to be available with the experiment's software.

Packages files `hypot-packages.yaml`
------------------------------------

These YAML [#yaml]_ files will contain a dictionary named *packages*, mapping package names to requirements lists, for example:

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
    justin:
      require:
      - '@=4.5.6:'
    boost:
      require:
      - '+iostreams+json'

A non-experiment package should only be included here if the experiment requires a specific variant of that package that is *not* required by any recipe in the experiment's recipe repository.
This may happen, for example, if analysis code that is not part of the repository requires the variant.

Specifying a version for a shared package (*e.g.* `art`) is discouraged.
Strict API requirements should be expressed in package recipes, not here.

The Build configuration file
----------------------------

This will be an Spack environment `hypot_release.yaml` file, with includes of the various fragments, above, and combines the various definitions from those files.
For example:

.. code-block:: yaml

  # Note that the SHA256 sums and the version number in this file are only
  # examples, and are not recommended values of any real build.

  spack:
    config:
      deprecated: true  # This allows use of deprecated versions of packages
    include:
      # Hypot packages
      - git: https://github.com/Hypot/hypot-release-configs
        tag: hypot-1.2.3
        paths:
        - included_yaml/hypot-packages.yaml
        - included_yaml/hypot-specs.yaml

      # Shared packages
      - git: https://github.com/art-framework-suite/art-release-configs
        tag: art-3.14.04
        paths:
        - included_yaml/gcc-12-toolchain.yaml  # compiler specifications
        - included_yaml/packages-art.yaml
        - included_yaml/specs-art.yaml

      - git: https://github.com/LArSoft/larsoft-release-configs
        tag: larsoft-10.20.03
        paths:
        - included_yaml/larsoft-packages.yaml
        - included_yaml/larsoft-specs.yaml

      - git: https://github.com/NuSoftHEP/nusofthep-release-configs
        tag: nusofthep-3.22.00
        paths:
        - included_yaml/nusofthep-packages.yaml
        - included_yaml/nusofthep-specs.yaml

      - git: https://github.com/fnal-fife/fife-release-configs
        branch: main
        paths:
        - included_yaml/fife_packages.yaml
        - included_yaml/fife_specs.yaml

    upstreams:
      cvmfs-larsoft:
        install_tree: /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.1/opt/spack

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
          path: /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.1/var/spack/environments/larsoft-v10_20_03-unified-cuda-python-3_11-trimmed-rc2
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
        - cxxstd=17  # This is needed here, and should be the same as in the unified build
        target:
        - x86_64_v3  # Specified so that the build is not specialized for the build machine

This file includes the various organizations' specs and packages files, and combines the various definitions from the specs files into the `spec:` list for this build.
It also sets concretizer and upstreams values to reuse the packages from the CVMFS Larsoft area in the build.

Experiment Build Recipe Repository
----------------------------------

The experiment should have a Spack build recipe repository, with the Spack recipes for:

1.  the experiment packages
2. any third-party packages used by only this experiment and not available in the main Spack builtin repository.

Spack `documents <https://spack.readthedocs.io/en/latest/repositories.html>`__ the recipe repository structure in some detail, as well as the recipe creation `process <https://spack.readthedocs.io/en/latest/packaging_guide_creation.html>`__.
We have some Fermi specific `recommendations <https://github.com/FNALssi/fermi-spack-tools/wiki/Spack-recipe-guidance-for-CMake-packages>`__ for CMake recipes using Fermi cetmodules.
Experiment repositories should be maintained by the experiment, and included in the fermi-spack-tools configs installed by our bootstrap / make_spack scripts.

Build process
=============

With some version of the above files and repositories available, building a version of the software consists of:

* preparing a Spack instance in which you will build your experiment's software
* creating a build environment for this release
* updating a `hypot_package.yaml`  with suitable version information
* putting the Build config / `hypot_release.yaml` file in the environment
* doing a `spack concretize` and `spack install` to do a test build of the software
* committing and tagging the versions of the `hypot_package.yaml` and build config `hypot_release.yaml` files used in the build 
* doing an actual release build, using the tagged version of the files from the from the experiment config repository.
* making *build cache tarballs* of the built binary packages
* installing the *build cache tarballs* into CVMFS package areas via a *build cache mirror*.

We will now discuss these steps in more detail.

Preparing a Spack build instance
--------------------------------

There are two approaches to setting up a Spack build instance:

* a standalone instance or
* a *subspack* chained instance

which we will discuss below; in either case we want to configure such an instance which:

* is in a read-write file system
* is configured with Spack path-padding so binary packages are redistributable
* knows about appropriate compilers
* has signing keys installed for signing binary packages for distribution

A *subspack* / chained instance will generally take less disk space and setup faster than a standalone instance.
However, if you don't have access to an existing instance to base it from, or you want to be insulated from the compilers available, existing packages, etc., in the existing instance, you may find a standalone instance preferable.
This would happen, for example, if you want to build with a compiler that has not yet been used in an upstream global build.
The result will be that your build will not have any packages available for reuse.

Creating a Standalone instance
------------------------------

At Fermilab, we recommend using our `bootstrap` script from our fermi-spack-tools package.
Instructions are available at `Installing Spack at Fermilab <./using_bootstrap.html>`_.

.. code-block:: shell

  $ wget https://github.com/FNALssi/fermi-spack-tools/raw/refs/heads/fnal-v1.1.1/bin/bootstrap
  $ sh ./bootstrap --with_padding /path/to/location

where you give a path to where you want the spack instance to appear.

You then want to make sure to install the compilers you want into your standalone instance. 
Unless you have a lot of time to kill, you probably want to install one that is already built from the *build cache mirror*.

.. code-block:: shell

  source /path/to/location/setup-env.sh
  spack buildcache list -al gcc llvm
  spack install --cache-only gcc@1.2.3


Creating a subspack instance
----------------------------

If you have access to one of our existing cvmfs Spack instances that is running Spack 1.0 or later, you can use our `spack subspack` extension to make a chained spack instance which shares the existing instance's packages, but lets you install new things into the local, writable one.
When doing this, you have to first source the setup-env.sh for spack instance that has the software upon which you wish to base your build, and then run spack subspack with the --with-padding flag to have it pad the paths of new packages to a standard length for relocatability. 

.. code-block:: shell

   source /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.1/setup-env.sh
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

Note that, if your upstream Spack instance is in /cvmfs, when running the 'spack concretize' you will likely receive some warnings of the form:

.. code-block:: shell

  ==> Warning: Ignoring write error on readonly /cvmfs/.../config/bootstrap.yaml

You can safely ignore these; Spack has a habit of overwriting some config files with the same content, and this obviously doesn't work in /cvmfs which is read-only.

However, things do not always come out perfectly in or cocretization, and  we probably wish instead to customize the `spack.yaml` file, and possibly several of the included files, to be ready for the next release.  
Most frequently, this involves customizing the `hypot-packages.yaml` file that the release file gets from the repository.
To avoid repeated updates of the config file to test a new version number, etc. you want to download that file from Git as well, and change your `spack.yaml` file to use the local copy. 


.. code-block:: shell

  wget https://github.com/hypot/hypot-release-configs/blob/main/included_yaml/hypot-packages.yaml

Then you want to edit the `spack.yaml` file and change it to use the local file instead; this changes the entry under include: to look like:

.. code-block:: yaml

  include:
    # Hypot packages
    - git: https://github.com/Hypot/hypot-release-configs
      tag: hypot-1.2.3
      paths:
      # - included_yaml/hypot-packages.yaml
      - included_yaml/hypot-specs.yaml
    - hypot-packages.yaml

You see above, we have commented out the repository path and put the local filename in its place.   
Now we can update version numbers, add or remove required variants, etc., in the local file, to assist in getting the build we want, without a lot of git commit/push operations.

Once we have a version of the `hypot-packages.yaml` file we like, we can commit it to our config repository, commit the `spack.yaml` file to the repository, and tag it, and do a build which is getting the tagged config file from the repository. 

Adding built packages to a build cache mirror
=============================================

Once you have packages built, you can make signed *build cache tarballs*, and upload them to the appropriate *build cache mirror* directory. 
First you will need a *GnuPG* signing key installed in your spack instance, if you don't have one already.

Installing a signing key
------------------------

If this is a new Spack build instance, you likely do not have any signing keys installed in it to sign your packages.  
If your experiment has a *pgp* key or key(s) for signing official binaries already, you can add it; if you don't have one you can create one.
Adding your own key, or an experiment one, is the same process, for a personal one you would of course use your own username and email address

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

where the `somehost:/some/place/safe` for an experiment account should be somewhere like `/opt/hypotpro` on one of the experiment gpvm machines, etc. or $HOME/.gnupg/spack for a personal key,  and in either case should be only readable  by  the relevant account.

Packing up your build for distribution
======================================

Now you can activate your spack build environment, and make signed build cache tarballs for the new build:

.. code-block:: shell

  spack env activate build_hypotcode_vx_y
  spack cd --env
  spack localbuildcache --key hypotcode@fnal.gov --local --not-bc --with-build-dependencies

This will 

* create a *build cache mirror* in the `bc` subdirectory
* sign the binaries with your hypotcode@fnal.gov *GnuPG* key
* only include packages from the local spack instance
* omit packages installed from a *build cache mirror* 
* include build dependencies, not just runtime ones

Moving the build cache images to SciSoft
========================================

Now (assuming you have suitable permissions) you can upload the *build cache tarballs* to SciSoft or spack-cache-1 and update the *build cache mirror* index.  


.. code-block:: shell

  spack env activate build_hypotcode_vx_y
  spack cd --env
  scp -r bc/* scisoftbuild02.fnal.gov:/SciSoft/spack-mirror/spack-binary-cache-v3
  ssh scisoftbuild02.fnal.gov <<EOF
   source /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.1/setup-env.sh
   spack buildcache update-index /SciSoft/spack-mirror/spack-binary-cache-v3
  EOF

Installing the built packages in /cvmfs
=======================================

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


.. rubric:: Footnotes

.. [#environments] `Spack Environments documentation <https://spack.readthedocs.io/en/latest/environments_basics.html>`__
.. [#yaml] `YAML on Wikipedia <https://en.wikipedia.org/wiki/YAML>`__

Building Compatible Updated and New Packages
============================================

Often we need to update versions of packages in our environment, like 
data management tools, etc. that are relased on a different schedule
than the main software releases; maintainers may wish to distribute
a new version of a package compatible with the existing distribution.
Or we may need to build an altogether new package which we would like
to be compatible with an existing package suite.
Then users can ``spack load`` the new package version without changing 
other packages out from underneath their current environment.

Setting up
----------

Make a build spack instance if you don't have it, and put your spack signing keys in it.  If you don't have a spack signing key, see `these instructions <https://fnalssi.github.io/spack-at-fnal/pages/build_manager_process.html#installing-a-signing-key>`__

.. code-block:: shell

   . /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/setup-env.sh
   spack subspack --with-padding $PWD/build_instance
   spack gpg trust $HOME/.gnupg/spack/pubring.gpg
   spack gpg trust $HOME/.gnupg/spack/secring.gpg

make and configure a build environment

.. code-block:: shell

   . $PWD/build_instance/setup-env.sh
   spack env create my_package_1_2_3
   spack env activate my_package_1_2_3
   spack cd --env
   vi spack.yaml
   
(interestingly, it is the "spack env activate" step here that is slowest!)

where the spack.yaml ends up looking like

.. code-block:: yaml

  spack:
    specs:
    - my-package@1.2.3

    concretizer:
      unify: true
      reuse:
        from:
        - type: environment
          path: /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/var/spack/environments/larsoft-v10_11_01-unified-cuda-python-3_10-trimmed-m3
    packages:
      all:
        require:
        - target=x86_64_v3

And now when you concretize your package, it should be basically the only package rebuilt... but sometimes it isn't.

Reading the spack concretize output
-----------------------------------

When you concretize, spack should give you output that looks like:

.. code-block:: bash

    ==> Concretized 1 spec:
     -   wzwrx6v  metacat@4.1.4+client_only build_system=python_pip platform=linux os=almalinux9 target=x86_64_v3 
     -   ahw4ylu      ^py-lark@1.1.2 build_system=python_pip platform=linux os=almalinux9 target=x86_64_v3 
     -   eyadajq      ^py-pip@25.1.1 build_system=generic platform=linux os=almalinux9 target=x86_64_v3 
    [^]  l7q24lu      ^py-pyjwt@2.4.0~crypto build_system=python_pip platform=linux os=almalinux9 target=x86_64_v2 
    [^]  yezicjn          ^py-pip@25.1.1 build_system=generic platform=linux os=almalinux9 target=x86_64_v2 
    [^]  rjohrzr          ^py-setuptools@80.9.0 build_system=generic platform=linux os=almalinux9 target=x86_64_v2 
    [^]  6qyc252          ^py-wheel@0.45.1 build_system=generic platform=linux os=almalinux9 target=x86_64_v2 
    [^]  rbt2w2m      ^py-pythreader@2.15.0 build_system=python_pip platform=linux os=almalinux9 target=x86_64_v
    ...

where each dependent package with all of its version, variant, etc flags are listed.  The lines all start with a status, one of:

.. |nbsp| unicode:: U+00A0 .. nbsp

* |nbsp| ``-`` |nbsp| a dash in the first 3 columns is a package spack thinks it should rebuild
* ``[^]`` bracketed carat indicates a package reused from the upstream spack instance
* ``[+]`` bracketed plus indicates a package installed in the current spack instance
* ``[e]`` brackeded e indicates an external (system) package

Basically, our goal is for the output of our concretize to only show a dash-ed entry for our new version of our package, and possibly one or two supporting packages not used by anything else in our software environments.
You might also find spack wanting to rebuild/reinstall some build dependencies that didn't get put into the upstream spack instance -- packages like py-setuptools and py-wheel -- this is okay.

Getting more reuse
------------------

If you're finding the spack concretize step is wanting to rebuild a lot of packages that already exist in the upstream environment, you can update the spec in the spack.yaml file to be more specific about reuse.
Basically you want to look in the reuse environment for the exact hash of the package it didn't reuse, and add that as an explicit dependency.  This looks like:

.. code-block:: shell

   $ spack -e larsoft-v10_11_01-unified-cuda-python-3_10-trimmed-m3 find --long py-webpie python
   ==> In environment larsoft-v10_11_01-unified-cuda-python-3_10-trimmed-m3
   ==> 61 root specs
   ...
   -- linux-almalinux9-x86_64_v2 / %c,cxx=gcc@12.5.0 ---------------
   ppik5zk python@3.10.16

   -- linux-almalinux9-x86_64_v2 / no compilers --------------------
   lojz5wk py-webpie@5.16.3
   ==> 2 installed packages

and then update the spack.yaml file with dependencies on the specific hashed versions:

.. code-block:: yaml

   spack:
     specs:
     - my-package@1.2.3 ^python/ppik5zk ^py-webpie/lojz5wk 
   ...

And do a ``spack concretize -f`` to reconcretize it.
With just a few iterations like this, you should be able to get a minimal
rebuild.

A short note on strategy here; it is usually fastest to specify the dependencies that are least indented 
in the "spack concretize" output.
It can also be very helpful to do 

.. code-block:: shell

   spack concretize -f | tee concretize.out

when concretizing to save a copy of the concretizer output to review what is being rebuilt and what isn't. 
Then you can do things like 

.. code-block:: shell

   grep '[a-z0-9]      ^' concretize.out

(that is with 6 blanks) to see all the first-level dependencies  to see which of them might need specifying.

Distributing the package
------------------------

Now that you have the concretization issues settled, you should be able to
build the package, make a buildcache image, and put it onto scisoft.fnal.gov or
spack-cache-1.
This looks like:
  
.. code-block:: shell

   spack cd --env
   spack install
   spack localbuildcache --local --key mykey
   scp -r bc/* scisoftbuild02.fnal.gov:/SciSoft/spack-mirror/spack-binary-cache-v3
   ssh scisoftbuild02.fnal.gov <<EOF
     source /cvmfs/larsoft.opensciencegrid.org/spack-fnal-v1.1.0/spack_env/setup-env.sh
     spack buildcache update-index /SciSoft/spack-mirror/spack-binary-cache-v3
   EOF

Now you or the experiment package maintainers can install the package into cvmfs following
`these instructions <https://fnalssi.github.io/spack-at-fnal/pages/build_manager_process.html#installing-the-built-packges-in-cvmfs>`__

Replacing package dependencies
------------------------------

Sometimes there is a package that needs to be updated in existing environments, either for security reasons, 
or because it reflects a configuration change that represents real world changes, where you need to 
flat out replace a package with a new version.   
A common example of this is the data management package ``ifdhc_config``, which knows thinks like service 
servernames, etc. and needs to be updated when those servers change.
Having built a compatible version of the package, as above, one can make a change like this using
the ``spack deprecate`` command, specifying that the old version of the package be replaced by the
new one. 
In a CVMFS instance this would be done right after installing the new version, but before you
publish the CVMFS transaction; and looks like

.. code-block:: shell

   $ spack find --long ifdhc_config
   -- linux-almalinux9-x86_64_v2 / no compilers --------------------
   vtkszyk ifdhc-config@2.6.20  zgxkyy3 ifdhc-config@2.7.2
   kebqcx2 ifdhc-config@2.6.20  gnj5lfy ifdhc-config@2.7.2
   qxgzxem ifdhc-config@2.6.20  t7gveqv ifdhc-config@2.7.3
   ...
   $ spack deprecate -D ifdhc-config/zgxkyy3 ifdhc-config/gnj5lfy ifdhc-config/t7gveqv

where the two 2.7.2 versions of ifdhc-config would be pointed at ifdhc-config@2.7.3.
It is recommended to do these explicitly by hash values like this so you don't inadvertently
modify the wrong instances.
The -D flag says to *not* similarly deprecate the dependencies of those packages.
Again, this is to not modify more things than you meant to.

You can read more about ``spack deprecate`` in the `Spack documentation <https://spack.readthedocs.io/en/latest/packaging_guide_creation.html#deprecating-old-versions>`__



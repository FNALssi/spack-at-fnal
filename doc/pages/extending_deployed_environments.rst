Extending Environments
======================

Sometimes you need to build a new package that is compatible with the package builds exposed by an existing Spack environment.
To do this we recommend creating a new Spack environment based on the existing Spack environment and which adds the new package.

Setting up
----------

If you do not already have your own *subspack*, you will need to create one; see :doc:`working_with_deployed_software`.
You have to run the `setup-env.sh` activation script for your *subspack*.

Creating the new environment
----------------------------

Making the new environment is done by:

1. cloning the environment you wish to extend
2. adding the new packages to your clone

.. code-block:: shell
  
  spack env create my_environment <name of the existing environment>
  spack env activate my_environment
  spack concretize # This should tell you there are no new specs to concretize
  spack add <new package>
  spack concretize # This will tell you what new things are to be installed
  # If the concretization succeeds, continue with
  spack install

If the concretization fails then you need need to rebuild some of the packages that are in the original environment: see :doc:`building_updated_packages`.

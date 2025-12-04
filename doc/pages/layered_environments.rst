Layered Environments
====================

The mechanism through which we will deliver built distributions of CSAID-supported software is *Spack environments*.
In order to facilitate sharing of built software products, we will use *layered environments*.
This document explains what a layered environment is, now we build them, and how we use them.

Spack environments
------------------

A Spack environment is a feature provided by the Spack package manager that allows users to manage and isolate collections of software packages, dependencies, and configurations.
To activate an existing environment, one uses the ``spack env activate`` command:

.. code-block:: bash

  spack env activate myenv

After activating an environment, the libraries and headers for the packages contained within the environment will be available for use.
To deactivate an environment, one uses the ``spack env deactivate`` command:

.. code-block:: bash

  spack env deactivate

Spack environments are fully documentted at https://spack.readthedocs.io/en/latest/environments_basics.html.

 
Layered Environments
--------------------

A goal of CSAID is to provide reliable builds of software stacks relied upon by experiments.
To this end, we build and distribute consistent versions of software environments that can be shared by as many experiments as feasible.
Because different experiments have different amounts of software in common, we have identified a set of "layers" that we will build and distribute.
Each of these layers is realized as a Spack environment that can either be activated directly or, more commonly, be *resused* in an experiment-specific Spack environment.


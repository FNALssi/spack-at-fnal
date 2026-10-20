Layered Environments
====================

SCD will distribute built packages through build caches and CVMFS package areas, and use *Spack environments* to expose selected package builds to users.
In order to facilitate sharing of built software products, we will use *layered environments*.
This document explains what a layered environment is, how we build them, and how we use them.

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

Spack environments are fully documented at https://spack.readthedocs.io/en/latest/environments_basics.html.


Layered Environments
--------------------

A goal of SCD is to provide reliable builds of software stacks relied upon by experiments.
To this end, we build and distribute consistent collections of built packages that can be shared by as many experiments as feasible.
Because different experiments have different amounts of software in common, we have identified a set of "layers" that we will build and distribute.
Each of these layers is realized as a Spack environment that can either be activated directly or, more commonly, be *reused* in an experiment-specific Spack environment.
Most experimenters working with their experiment's software will only directly see the experiment-specific environment.

A *standard build* contains all packages in a software suite, selected
*ancillary packages* needed by supported experiments, and the complete
build-time and run-time dependency closure of both. Ancillary packages
are outside the suite and its dependency closure; for example,
*fife-utils* is ancillary to LArSoft. They are included so that the
combined collection supports coexistence and supported workflows without
package conflicts, including ABI compatibility where applicable.
See :doc:`plan_for_standard_builds` for the full definition and guarantee.

A layer need not contain the whole suite or its standard build.
Dependencies and ancillary packages may be exposed through other layers,
and all package builds in the standard build must be available even if
they are not all installed in one Spack instance or exposed by one
environment. Layer membership does not define suite membership.

To produce an environment that can be shared between experiments it is necessary to allow Spack to see *all* of the packages that will be used by each experiment.
This allows Spack to concretize a consistent set of package builds that can be shared.
In the proposed workflow, this is the *global solve*, and its new or
reused built results form the *global build*. Standard builds drawn from
that global build may overlap through shared package builds; using a
global solve is not part of the definition of a standard build.

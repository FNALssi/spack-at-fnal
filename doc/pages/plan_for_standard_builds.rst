=====================================
A Plan for Standard Builds of LArSoft
=====================================

Introduction
============

For more than 30 years the Fermilab scientific software stacks have been
packaged and distributed using UPS and related tools. Fermilab specific
scripts and procedures were developed to build several hundred
third-party software packages (i.e., software created by neither
Fermilab scientific software developers nor by the experiments) into
UPS-compatible forms. Because UPS is a Fermilab-specific tool, it has
been difficult for non-Fermilab personnel to develop the expertise
needed to contribute to building this software stack. As a result, the
time taken to update versions of software, and to add new packages, is
longer than it would be if the software stack were built and distributed
using more widely-used tools.

In order to make it easier for non-Fermilab personnel to contribute to
our shared scientific software stack, and because of the diminishing
availability of personnel to maintain support of the specialized
infrastructure needed, we have decided to move to using Spack as the
primary tool for building and distributing the software stack. Because
Spack is supported by a larger community than is UPS, we will gain the
advantage of being able to use the work of others in the community who
have already created the necessary recipes for building much of the
software we need. We will also gain the advantage that thorough
documentation is available on the web. In addition, it will be easier
for members of Fermilab experiments to add new recipes for third-party
software that they need to use, and to contribute those additions back
to the rest of the community. Finally, Spack is better placed than UPS
to handle the needs of current and upcoming experiments, as UPS is very
rigid and has high maintenance overhead with respect to dependency
chains, and has other limitations with regards to issues such as
relocatability and release distribution.

In this document we describe a plan for how to make use of the software
built, packaged, and distributed using Spack and related tools. The
goals of this plan are several:

1. To allow the SciSoft team to build, package, and distribute to the
   experiments the software the team develops or contributes to,
   including the *art* framework, LArSoft, and eventually \*phlex.
2. To provide the experiments greater flexibility in building software
   not provided in the collection of packages delivered by the SciSoft team,
   while also providing a clear path for sharing effort between
   experiments.
3. To provide more flexibility for experiments to support building their
   software stack on platforms on operating systems not directly
   supported by Fermilab, for example, at supercomputing facilities.

A central feature of the new plan is what we call a *standard build*,
described in the next section.

Definition of a Standard Build
==============================

A *standard build* is a consistent collection of built packages created,
designated, and published by SCD for shared use by supported experiments
or projects. It includes:

* every package in a software suite;
* selected *ancillary packages* needed by the supported experiments; and
* the complete build-time and run-time dependency closure of both the
  suite and the selected ancillary packages, including transitive
  dependencies.

The selected versions and variants determine which package builds
realize this collection; they do not permit required dependencies to be
omitted. The collection contains actual shared libraries, executables,
and other installed files, not just descriptions of how to produce them.
It includes reused package builds, not only packages newly built for
that standard build. Teams within SCD have primary responsibility for
different suites; SciSoft is responsible for the *art* and LArSoft suites.

A *software suite* is a named collection of packages intended to be developed and used
together. A *project* coordinates maintenance of software packages and
need not correspond to exactly one suite. Dependencies of a suite need
not be members of that suite: a standard build of LArSoft includes the
selected *art* and ROOT package builds, but those packages are not
members of the LArSoft suite.

An *ancillary package* is outside the suite and its build-time and
run-time dependency closure, but is selected for inclusion in the
standard build to support experiments' needs. For example, SciSoft's
standard build of LArSoft includes *fife-utils*, which the experiments
need but on which LArSoft does not depend. Including it does not make it
part of the LArSoft suite. Ancillary status is relative to a suite: a
package ancillary to one suite may be a dependency of another. The
dependencies of an ancillary package belong in the standard build even
when the suite itself does not need them.

The responsible SCD team and supported experiments jointly select the
ancillary packages. They may agree on different membership for different
supported platforms or configurations; membership must be documented
for each standard build. Inclusion does not extend to every package any
experiment might need.

Consistency includes ABI compatibility where applicable and compatible
shared dependencies, but is not limited to these properties. The suite
and ancillary packages must coexist in the same supported software
environment and perform their supported workflows without conflicts
caused by the combined package collection. This includes Python modules,
command-line tools, shared libraries, and their relevant interactions.
The guarantee does not cover external-service availability, unsupported
uses, or experiment physics validation. A successful global solve or
ABI compatibility alone does not establish this functional compatibility;
the LArSoft PR CI checks described below have a narrower scope. This
definition does not specify an additional validation procedure.

A standard build is a built result, distinct from the build process,
the *build configuration* that selects it, and any *Spack environment*
used to expose its packages. An individual package build within a
standard build is not itself a standard build. An independent
experiment build is not a standard build merely because it uses the
same configuration as an SCD-published build.

A standard build is one particular collection of package builds.
Changing its membership or any selected package build produces a
different standard build, even if the suite's release version is
unchanged. This includes changes to ancillary-package versions,
compilers, targets, Python versions, CUDA support, variants, or package
build modes. Independently distributing a compatible package update
does not automatically include it in an existing standard build.

The move away from UPS to Spack is intended to make it easier for
non-SciSoft personnel to make their own builds of software stacks. At
the same time, the SciSoft team must retain the ability to build and
test LArSoft, which requires building and testing the software stacks
that depend upon LArSoft. Separately, the SciSoft team will also need to
be able to build and test the software stack for *phlex*. The framework
for allowing these distinct efforts to proceed relies on the idea of
*standard builds*. The SciSoft team will produce the SCD-designated
standard builds of the LArSoft and *art* suites for publication to users;
test or partial builds are not themselves standard builds. Ancillary
packages include selected data management clients. The “data management”
software stacks will need to package data management clients, etc., so
they do not have conflicting dependencies with the scientific software
stacks. It is inconvenient for users of the data
management tools (e.g., *rucio-clients*, *sam-web-client*, etc.) to have
to “unsetup” their experiment’s software environment in order to “setup”
the data management tools. Having the tools built with the same
underlying libraries will allow them to be active in the same shell
session.

Each package within a standard build is built with a specific set of
variants [1]_. For example, if one experiment requires ROOT with support
for Apache Arrow, and another requires support for Graphviz, then the
ROOT package build shared by those experiments would include both
options. It will be up to the collaborating LArSoft experiments to
determine the set of options for each package included in their standard
builds.

In the proposed global-solve workflow, each suite's standard build is a
subset of the *global build* described below. This is a property of the
workflow, not a prerequisite for being a standard build; the definition
also applies to SciSoft's standard builds produced without a global
solve. Standard builds of different suites may overlap through shared
package builds. For example, standard builds of *art* and LArSoft can
include the same ROOT package build.

The SciSoft team will create a Spack environment for each standard build
of a suite. Its package builds will be made available through appropriate
build caches [2]_ and CVMFS package areas. The SciSoft team will make
available build configuration and environment definition files for each
standard build through CVMFS, or
``https://scisoft.fnal.gov``, or both. Publishing only those files does
not publish a standard build; all its built packages must also be available.
Distributing the same package builds through both a build cache and
CVMFS does not create two different standard builds.

The collection may span distribution locations and layered environments;
no single Spack installation, environment, or build cache is required to
contain all of it. For example, build-only dependencies may be available
separately from a runtime installation. An upload containing only newly
built packages is not the complete standard build: reused package builds
and all required dependencies remain part of the collection.

The Plan for Standard Builds and Releases
=========================================

Here we describe the “steady state” plan for releases, the standard
builds that will be created for each release, and guidance on how the
experiments should use them. Some adjustments to the process might be
needed during the transition from UPS to Spack.

1. A new release may be created when one of the LArSoft experiments or
   the SciSoft team submits a successful pull request (PR) on one or
   more of the LArSoft repositories, or when one of the underlying
   dependencies is updated, which may occur at the discretion either of
   the experiments or the SciSoft team. The code of the experiment
   making a release request must be consistent with the current LArSoft
   release, or for a bug fix release, consistent with the most recent
   bug fix release on the associated minor release branch of LArSoft.

   PRs from experiments to recipes for 3rd party packages, and to the
   *art* and LArSoft packages will be welcomed. The procedure for
   maintaining recipes is described in :doc:`recipe_maintenance`;
   additional PR requirements are described separately.

2. For each new release of the *art* or LArSoft suites, the SciSoft team
   will create a number of standard builds. Each of these builds will
   use a specific, single source code version for each of the packages
   in the software stack. Across these standard builds, first-party
   software will be built in both *debug* and *profile* modes.
   Third-party packages for which *debug* builds are useful will also
   be built in both *debug* and *profile* modes. Others will be built in
   *release* mode only (i.e., in the
   native build mode for the package). Across these builds, each package
   will be built using a small number of supported compilers (and
   specific versions of those compilers). The LArSoft collaboration and
   the SciSoft team will together decide on compilers to be supported.

   Each selected, consistent combination of compiler versions and
   package build modes produces a distinct standard build. Thus one
   software release can have several standard builds; the supported
   compiler and build-mode combinations are not all combined into one
   standard build.

3. A Spack environment will be created corresponding to each standard
   build of a suite. Users of the standard builds will be able to use
   ``spack env activate`` (or the alias, ``spacktivate``) to activate
   the corresponding environment, and also build their experiment
   software against the package builds it exposes. Users of the standard
   builds can also do development with Spack MPD, which does not require
   activation of the environment. Users who are developing all or part
   of LArSoft itself will be able to activate the corresponding
   environment, then build the relevant parts of LArSoft in addition to
   their own experiment’s software stack.

4. Experiments that are part of the LArSoft collaboration should use
   standard builds, preferably one of the *most recent*, as the base for
   their own development builds. Doing so makes it possible to test new
   releases of LArSoft against the experiment code. If an experiment
   builds does not use a standard build of LArSoft, then testing by the
   SciSoft team becomes infeasible, which risks wasting effort creating
   standard builds that do not work.

5. Experiments that are part of the LArSoft collaboration are encouraged
   to use *one of* the standard builds as a base for their own
   production builds. This need not be the most recent standard build,
   since the pace of releases for production is determined by other
   factors than the pace of the releases of the LArSoft suite.

6. Experiments may choose to build alternative builds of LArSoft. The
   SciSoft team will be available for consulting on such builds on a
   best-effort basis.

The use of the LArSoft and Experiment CI systems to verify LArSoft Pull Requests
================================================================================

Changes to LArSoft code are made via PRs to the relevant repositories.
PRs that pass a review and testing process are merged into the main body
of code, where they can be tagged and used in a release. The review
process uses the CI system to trigger builds and two phases of tests
that are built into each of the LArSoft and experiment code
repositories. (In the following, we refer to the build and tests
collectively “CI tests”.) As in the past, the SciSoft team will rely on
the results from these CI tests to determine whether the pull request
behaves in the expected way. Only when test results are understood can
PRs be accepted. Details of the CI testing process, the conditions that
must be met for a PR to be accepted, and the responsibilities on various
parties in maintaining the tests are described below.

The CI tests run for LArSoft PRs are not intended to be *physics
validation tests* for the experiments. The purpose of the CI tests is to
verify that the code changes do not break experiment software. Physics
validation testing of each experiment’s code is the responsibility of
that experiment.

How the CI system we be used to verify LArSoft PRs
--------------------------------------------------

1. The SciSoft team will use the LArSoft CI system to build and test the
   development head of each LArSoft repository as needed, such as when
   triggered by a PR to a LArSoft repository from any source (the
   experiments or SciSoft). Each PR or collection of associated PRs to
   LArSoft repositories must pass all CI tests for all LArSoft
   repositories. In addition, PRs must meet a set of additional
   requirements before they can be accepted. Those additional
   requirements will be proposed, documented and maintained separately
   by the SciSoft team. It is the responsibility of the submitter of the
   PR to fix test failures address issues related to the other
   requirements, or to arrange to have them fixed.

2. Once the CI tests for LArSoft are passed, the CI tests for each
   experiment that uses LArSoft are run. As in the case of the LArSoft
   CI tests, all CI tests for all experiments must succeed for a PR to
   be accepted. It is the responsibility of whatever experiment
   submitted or requested the PR to fix CI test failures or make
   arrangements to have them fixed.

3. Note that commits to an experiment’s repositories, and not only
   changes to LArSoft, might break that experiment’s CI tests. It is the
   responsibility of each experiment to maintain their CI system by
   keeping the branches of the repositories they use for development
   up-to-date with the development head of LArSoft. If an experiment
   does not keep their CI tests up-to-date with the most recent release
   of LArSoft, or that branch otherwise becomes incompatible with the
   most recent release, then their CI tests will be removed from the
   workflow of the CI system used to verify new PRs in LArSoft. It will
   be re-enabled in the workflow as soon as the experiment updates their
   code to work with the most recent release of LArSoft.

4. Bug fix PRs for old releases of LArSoft can be accepted for declared
   production releases of LArSoft only. A PR that fixes a bug in a
   production release of LArSoft will be merged into the bug-fix branch
   for that release only if the CI tests for the production release of
   LArSoft and the experiment passes, or the PR breaks no CI tests that
   passed immediately before the PR. Each experiment is expected to keep
   a branch in their own repositories for CI testing of bug fixes for
   each of the LArSoft production releases they use.

What will be built and tested for each LArSoft PR
-------------------------------------------------

Each PR to a LArSoft repository will trigger a build of:

1. the PR version of the code for that repository
2. the development head for each LArSoft repository that has a compile
   or link dependency (either directly or transitively) on the
   repository being changed by the PR
3. the development head of each active LArSoft experiment’s repositories
   that have a compile or link dependency (either directly or
   transitively) on the repository being changed by the PR

The LArSoft repositories that do not have a compile or link dependency
on the repository being changed will not be built. Instead, their most
recent releases will be used in the build.

Upon successful build of the above, the ``ctest`` test suite for each
built repository will be run.

How we will organize Spack environments
=======================================

Layered Spack environments
--------------------------

SCD will create Spack environments for its own use and for the use of
the experiments. To ensure binary compatibility across the several
environments, SCD will perform a *global solve*: Spack concretizes the
abstract specs and constraints from participating experiments and
projects into one consistent set of concrete specs.

A *spec* describes a package configuration, including its version,
variants, compiler, flags, architecture, and dependencies. An *abstract
spec* leaves choices unresolved; a *concrete spec* fixes all those
choices and the full dependency graph, describing exactly one package
build. A spec is not the built libraries or executables. These terms
follow the `latest Spack documentation
<https://spack.readthedocs.io/en/latest/spec_syntax.html#software-model>`_.

SCD will realize the concrete specs through new or reused package
builds. The resulting collection of built packages is the *global
build*. In this workflow, each standard build is a subset of that global
build containing all the suite's package builds, selected ancillary
package builds, and the complete build-time and run-time dependency
closure of both. Standard builds may overlap by sharing package builds.

A *layered environment* is a Spack environment providing one shareable
layer of software. Each layered environment exposes a subset of the
global build, rather than selecting an independent set of package
builds. A layer need not contain all the packages in a suite's standard
build, since dependencies and ancillary packages may be exposed through
other layers. Layer membership is distinct from suite membership and
from the contents of a standard build. This organization allows
experiments to reuse consistent package builds, while still allowing
them to replace portions of the dependency graph
when they have a special need. The modified experiment build is not
itself an SCD standard build. The figure titled
:ref:`The proposed Spack environment layers <spack_environments>` shows for illustrative
purposes how spack environments might be layered. A description of the
layers shown in this figure is provided below. Details of the layers and
their contents may change in the final implementation. Note that the
packages that are named are for illustration only; they are not
exhaustive. Link dependencies are allowed within each layer, and from
higher layers to lower layers, but never from lower layers to higher
layers. The arrows in :ref:`the figure <spack_environments>`
show the direction of the allowed dependencies. The connection to the
*tools* layer has no arrow because there is no link dependency upon
*tools*.

.. graphviz:: ../static/environment_graph.gv
   :align: center
   :caption: The proposed Spack environment layers. The *art*, *fife*, and *nulite* layers are also used by experiments and projects that do not depend on LArSoft. Yellow boxes indicate packages directly maintained by SCD.
   :name: spack_environments

1. The *tools* layer includes tools that are used for development.
   Examples include compilers (but not compiler runtimes), *git*,
   *cmake*, and *ninja*. Packages in other layers do not have link
   dependencies on this layer.

2. The *substrate* layer includes products for which we (SCD) are not
   in control of the source code. Examples include *ROOT*, *Geant4*, and
   *Catch2*. *Python* should probably be included here because some code
   needs to link against the *libpython.so*, and because the Python
   interpreter is a runtime dependency for Python programs. Compiler
   runtimes are included in this layer.

3. The *fife* layer includes products, excluding *art*, needed to build
   higher-level layers, along with data management and authentication
   client packages. These products are developed by other groups in
   SCD. One example is *ifdhc*.

4. The *art* layer includes all of the products that are currently part
   of the *critic* suite, and which are all products that are developed
   by the SciSoft team. The *art* layer also includes *ifdh_art*, which
   introduces the dependency on *ifdh*.

5. The *nulite* layer includes products that are needed to build
   *larsoft* (and *novasoft*)but which are not part of *art* or *fife*,
   and which are controlled by other groups in SCD. Examples are
   *GENIE* and *nusimdata*.

6. The *larsoft* layer includes all of the LArSoft products. It also
   includes *artdaq_core* which is used by the experiments that use
   LArSoft.

7. The *experiment* layers indicates the software of one of the
   supported experiments, and any 3rd party products that are used only
   by that one experiment. Each experiment layer is independent of other
   experiment layers.

Experiments are free to introduce addtional layers if code sharing
between limited subgroups of experiments is desired.

Building and distributing layered environments
----------------------------------------------

SCD is responsible for creating and publishing standard builds, with
its teams primarily responsible for different suites. The SciSoft team
will be responsible for building and distributing
releases of the *tools*, *substrate*, and *art* layers, while the Data
Management team will be responsible for building and distributing the
*fife* layer. Releases of *tools* and *substrate* will be made as needed
in response to requests for newer version of packages in those layers.
Updates to the *tools* layer will be made in response to requests from
experiments or the SciSoft team, not automatically when new versions of
the source code for those packages are released. Updates to the
*substrate* layer will be made in response to requests from experiments
or the SciSoft team, or when needed to support new versions of
higher-level layers. The requesting party (experiment or the SciSoft
team) is responsible for demonstrating that the update is
self-consistent in that the updated *substrate* builds without error.
The requesting party is also responsible for demonstrating that the
higher-level layers also build and pass CI tests without error.

Updates to the *art* layer will be made as needed, either because of new
versions of *tools* or *substrate*, or due to pull requests for bug
fixes or new features in the *art* suite. The SciSoft team will remain
responsible for ensuring that the *art* layer builds and passes CI tests
without error.

The SciSoft team will build and distribute releases of the *fife* layer,
if needed, or reuse recent builds from Data Management, who will be
responsible for maintenance of the code for the packages in that layer.
Similarly, the SciSoft team will build and distribute releases of the
*nulite* layer, but maintenance of the code for the packages in that
layer remains the responsibility of the authors of that code.

The SciSoft team will build and distribute releases of the *larsoft*
layer as described above. A LArSoft standard build spans this layer and
the relevant package builds in other layers, including selected ancillary
packages and the complete build-time and run-time dependency closure;
it is not just the contents of the *larsoft* layer.

Each experiment retains the responsibility for building and distributing
its own *experiment* layer.

It is not yet determined whether a given Spack installation would host
multiple layers and/or releases of same, and what the
complications/advantages of such might be. It is also not yet determined
how many binary build caches we will need and when different caches
should be used.

Other Notes on the Use of Spack
===============================

The release of Spack 1.0 has introduced the promise from the Spack
principal developers that new releases of Spack 1.x will be compatible
with the recipes from early 1.x versions. This was not true for pre-1.0
versions of Spack. In addition, Spack package recipes have been moved
out of the Spack core repository and into
`spack/spack-packages <https://github.com/spack/spack-packages>`_, which
is versioned separately. Upgrades to the Spack infrastructure should no
longer require rebuild of the entire software stack.

In order to reduce unnecessary rebuilding of the software stack,
experiments are encouraged to use either the appropriate Spack
installation from CVMFS, or to use SciSoft-provided scripts and/or
procedures to produce an appropriately versioned, patched (where
necessary), and configured Spack installation. Recommended procedures
for doing this will be detailed in another document.

Change log
==========

Only significant changes to the plan will be included in the change log.
The git repository can be consulted for a full record of all changes.

Version 1 (2025-03-28)
----------------------

Original version.

Version 2 (2025-08-20)
----------------------

1. Introduce use of the name chosen for the DUNE framework, *phlex*.

2. Add statement in section 3, item 2, about bug fix releases.

3. Substantial changes to section 5 to reflex the change in our approach
   to building environments.

Version 3 (2026-06-09)
----------------------

1. Revise section 4 to describe the use of the LArSoft and experiment CI
   systems for verifying LArSoft pull requests rather than releases.

2. Clarify that CI testing for LArSoft pull requests is intended to
   detect software breakage, not to serve as physics validation for
   experiment code.

3. Add a description of which dependent LArSoft and experiment
   repositories will be built and tested for each LArSoft pull request.

Version 4 (2026-10-20)
----------------------

1. Define standard builds as SCD-designated and published suite-level
   built results rather than individual package builds or Spack
   environments. A standard build contains the entire suite, selected
   ancillary packages, and the complete build-time and run-time
   dependency closure of both, including reused package builds.

2. Define ancillary packages relative to a suite and describe joint
   selection, configuration-specific membership, and the supported
   workflow compatibility guarantee.

3. Distinguish global solves, global builds, software suites, projects,
   and layered environments; clarify that suite-level standard builds
   are potentially overlapping subsets of the global build in the
   proposed global-solve workflow, not as a prerequisite for being a
   standard build.

4. Separate standard-build identity from suite release versions,
   distribution locations, and compatible package updates.

5. Use current SCD terminology and clarify spec usage against the latest
   Spack documentation.

.. [1]
   In Spack, a variant is a way to specify build options or
   configuration choices for a package. Variants allow the user to
   customize how a package is built and installed, giving flexibility to
   enable or disable certain build options and configuration choices,
   choose dependencies, or set specific configuration parameters.

.. [2]
   A *build cache* is a collection of Spack binary package archives and
   associated spec metadata made available for reuse by other Spack
   installations. A ``spack install`` command can download and install
   an already-built package rather than downloading source code and
   building the package.

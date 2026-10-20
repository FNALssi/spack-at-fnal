# Spack at Fermilab

This context describes how Fermilab's Scientific Computing Division (SCD) and supported experiments use Spack to build, share, and distribute software.

## Participants and software

**Build manager**:
A designated person responsible for producing and publishing official builds of experiment or project software.

**Experiment software stack**:
The collection of experiment-specific, shared, and third-party software needed for an experiment's computing operations, including the experiment software itself and its required build and runtime dependencies. The stack is independent of its distribution mechanism; a software release is a particular versioned realization of it.
_Avoid_: software environment (when referring to the whole stack rather than its Spack environment)

**Shared package**:
A software package developed or maintained for use by multiple experiments or projects.

**SCD**:
Fermilab's Scientific Computing Division, which develops and distributes software packages and is responsible for creating standard builds. Teams within SCD have primary responsibility for different software suites.

**Software suite**:
A named collection of software packages intended to be developed and used together. Suite membership is distinct from dependency relationships and from the project that coordinates maintenance.

**Project**:
A coordinated maintenance grouping for software packages. A project need not correspond to exactly one software suite.

## Spack configuration and builds

**Build configuration**:
The Spack environment configuration and included fragments that select software specs and set requirements for a build or release.
_Avoid_: build recipe (for environment configuration)

**Build recipe**:
A Spack package definition that describes how to build and package a software package.
_Avoid_: build configuration (for a package definition)

**Build configuration repository**:
A version-controlled repository of an experiment's Spack build configuration files, including its spec lists and package requirements.

**Global solve**:
The Spack concretization of abstract specs and constraints from participating experiments and projects into one consistent set of concrete specs describing package builds that can be shared.
_Avoid_: global build (when referring specifically to constraint solving)

**Global build**:
The consistent collection of built packages realizing a global solve, including any reused package builds. Suite-level standard builds are subsets of this collection and may overlap through shared dependency builds.

**Standard build**:
A consistent subset of a global build containing a software suite's built packages and their selected dependencies, created, designated, and published by SCD for shared use by supported experiments or projects. Each standard build realizes one selected build configuration and is distinct from that configuration, any individual package build within it, and any Spack environment or distribution location used to expose its packages.
_Avoid_: reference build

**Spec**:
A Spack description of a package configuration, specifying or constraining its name, version, variants, compiler, flags, architecture, and dependencies. A spec describes a package build, not the built files themselves.

**Abstract spec**:
A spec whose package configuration is not fully resolved. Concretization fills in the missing choices and dependency graph.

**Concrete spec**:
A fully resolved spec with all package configuration choices and the complete dependency graph fixed. It describes exactly one package build, whether or not that build has been installed.

These spec definitions follow the [latest Spack glossary](https://spack.readthedocs.io/en/latest/glossary.html#term-spec) and [Spec Syntax documentation](https://spack.readthedocs.io/en/latest/spec_syntax.html#software-model).

## Environments and distribution

**Spack environment**:
A Spack-managed collection of software specs and configuration used to concretize, build, or use software.

**Layered environment**:
A Spack environment that provides one shareable layer of software and can be reused in an experiment-specific environment.

**Upstream environment**:
A Spack environment designated as a source of installed packages that another environment may reuse.

**Build cache**:
A collection of Spack binary package archives and associated spec metadata made available for reuse by other Spack installations.
_Avoid_: binary mirror (when referring to the package collection rather than its storage endpoint)

**CVMFS package area**:
A CVMFS-hosted location where built Spack packages are installed for distribution to users.

**Software release**:
A versioned distribution of an experiment's or project's software, built from versioned Spack configuration and package recipes.

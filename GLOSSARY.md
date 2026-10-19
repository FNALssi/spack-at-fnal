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
Fermilab's Scientific Computing Division, which develops and distributes software packages and supports shared software builds.

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
The Spack concretization of constraints from participating experiments and projects into one consistent set of package builds that can be shared.
_Avoid_: global build (when referring specifically to constraint solving)

**Reference build**:
A build of shared software packages intended to serve multiple supported experiments or projects.

**Spec**:
A Spack description of a package and the constraints on its version, variants, dependencies, and build configuration.

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

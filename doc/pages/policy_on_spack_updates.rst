============================
SCD Policy and Spack Updates
============================


Spack Minor Version Updates
---------------------------

When a new minor version of Spack is released SCD will test the new release for backward compatibility with the distributed software built with the previous release of Spack.

If the new release of spack produces a build that is consistent with that produced by the previous version of Spack, SCD will:

1. Update the SCD-provided tools to ensure they work using the new Spack release.
2. Make public in SCD's fork the new Spack release
3. Recommend that experiments and projects upgrade to the new Spack release, and update their Spack instances to the new version
4. SCD will upgrade the installed Spack instances to the new version of Spack
5. SCD will begin using the new version of the creation of new releases of supported software

If the new release is *not* compatible, SCD will not update the tools or the fork.  
We will pursue with the Spack developers a resolution to the issue.
If the issue can not be resolved, we will treat it as if it were a major release version.

Spack Major Version Updates
---------------------------

A new major version of Spack will require consultation with the experiment and project community to plan a way forward.


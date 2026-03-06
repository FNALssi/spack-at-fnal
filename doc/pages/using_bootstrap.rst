Using the fermi-spack-tools bootstrap script
============================================

Introduction
------------

At Fermilab, we recommend using our `bootstrap` script from our fermi-spack-tools package.

* pick a directory path  where you want to place your spack instance
* fetch a release branch of our bootstrap script with wget or curl, one of

  * `fnal-v1.1.1 <https://github.com/FNALssi/fermi-spack-tools/raw/refs/heads/fnal-v1.1.1/bin/bootstrap>`__ — gets the matching fnal-v1.1.1 branch of Spack by default.
  * `fnal-v1.1.0 <https://github.com/FNALssi/fermi-spack-tools/raw/refs/heads/fnal-v1.1.0/bin/bootstrap>`__ — gets the matching fnal-v1.1.0 branch of Spack by default   
  * `fnal-develop <https://github.com/FNALssi/fermi-spack-tools/raw/refs/heads/fnal-develop/bin/bootstrap>`__ 

* run "bash bootstrap.sh /install/location" 
* source /install/location/setup-env.sh 

Sample session:

.. code-block:: bash

   $ wget https://github.com/FNALssi/fermi-spack-tools/raw/v2_21_0/bin/bootstrap
   ...
   2024-08-07 13:38:21 (15.4 MB/s) - 'bootstrap' saved [4489/4489]

   $ bash ./bootstrap /install/location
   Putting detail log in /tmp/bootstrap2688477.log
   Cloning FNALssi fermi-spack-tools repository             
   Setting up with make_spack                               
   Setting up new instance                                  
   Finding compilers                                        
   Bootstrapping...                                         


Common options to the bootstrap script
--------------------------------------

Besides the mandatory destination directory argument, the bootstrap script 
takes a short list of optional arguments:

\--help                              
  Prints a help message describing these options and exits
\--query-packages    
  Run make_packages_yaml rather than putting in templated packages.yaml
\--with_padding
  Set padding in spack config
\--fermi_spack_tools_release ver
  fetch the labeled version of fermi_spack_tools
\--fermi_spack_tools_repo url  
  set the git reposity for the above
\--spack_release ver                 
  fetch the labeled version/branch of Spack
\--spack_repo url                    
  set the git repository for the above

If Something Goes Wrong
-----------------------

If the bootstrap says that it failed to complete successfully, you can 
check for a few obvious problems, and if those aren't the issue, you should
note the detail log file it prints at the beginning, i.e.

.. code-block:: shell

  Putting detail log in /tmp/bootstrap2688477.log

and report the issue with that logfile as an attachment.
This document will describe these in more detail below.

Common Issue: disk space
------------------------

You should check that there is sufficient disk space for the destination
directory, and in your temporary storage.

.. code-block:: shell

   df -h /install/path ${TMPDIR:-/tmp}
   Filesystem                         Size  Used Avail Use% Mounted on
   /dev/mapper/vg_belkwinith-lv_home  405G  405G    0G 100% /install
   /dev/mapper/vg_belkwinith-lv_root   49G   18G   29G  39% /

In the above example, you can see that the install area is out of space,
you need to remove the installation directory you attempted and try someplace
with some more space.
If the temporary area (i.e. /tmp) is full, you can clean up any files you 
have there you don't need, and/or set TMPDIR in the environment to some alternate location that should be used for temporary storage

If you have located more space, and want to retry the bootstrap, you should 
remove the entire filetree at the install location, and rerun the bootstrap

.. code-block:: shell

   $ rm -rf /install/location
   $ bash ./bootstrap /new-install/location
   

Reporting a failed bootstrap
----------------------------

Make a servicedesk ticket to the Software Packaging group in 
fermi.servicenowservices.com, and upload the "detail log" file 
(whose name the bootstrap prints) out as an attachment.



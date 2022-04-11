.. _user_setup:

Client Setup
------------

The BenchPRO site package should already be installed on most TACC systems. If it is not, contact ``mcawood@tacc.utexas.edu`` or install it from the benchpro-site_ repository. Assuming the site package is available, you need to install a local copy of the configuration and template files to use BenchPRO.

.. _benchpro-site: https://github.com/TACC/benchpro-site

.. list-table::
    :header-rows: 1

    * - System
      - Module Path
    * - Frontera
      - /scratch1/hpc_tools/benchpro/modulefiles
    * - Stampede2
      - /scratch/hpc_tools/benchpro/modulefiles
    * - Lonestar6
      - /scratch/projects/benchtool/modulefiles
    * - Longhorn
      - TBD

#. Load the BenchPRO site package using the appropriate system path above, this module adds BenchPRO to PYTHONPATH and sets up your environment.

.. code-block::

    ml python3
    ml use [module_path]
    ml benchpro

#. You will likely get a warning stating you need to install missing user files. Follow the instructions to clone those files from this repository:

.. code-block::

    ml git
    git clone https://github.com/TACC/benchpro.git $HOME/benchpro

#. Finally, you need to run a validation process to confirm that your system, environment and directory structures are correctly configured before using BenchPRO for the first time. Run this with:

.. code-block::

    benchpro --validate

NOTE: Some of the hardware data collection functionality provided by BenchPRO requires root access, you can either run the permissions script below to privilege said scripts, or manage without this data collection feature.

.. code-block::

    sudo -E $BP_HOME/resources/scripts/change_permissions.sh

#. Print help & version information:

.. code-block::

    benchpro --help
    benchpro --version

#. Display some useful defaults

.. code-block::

    benchpro --defaults


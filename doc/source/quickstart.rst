========
Quick Start
========

Install
========

The BenchPRO site package should already be installed on most TACC systems. If it is not, contact ``mcawood@tacc.utexas.edu`` or install it from the benchpro-package_ repository. Assuming the site package is available, you need to install a local copy of the configuration and template files to use BenchPRO.

.. _benchpro-package: https://github.com/TACC/benchpro-package

  =====                =====
 System              Module Path     
  =====                =====
 Frontera            /scratch1/hpc_tools/benchpro/modulefiles 
 Stampede2           /scratch/hpc_tools/benchpro/modulefiles             
 Lonestar6           /scratch/projects/benchtool/modulefiles             
 Longhorn            TBD             
 =====                 =====

#. Load the BenchPRO site package using the appropriate system path above, this module adds BenchPRO to PYTHONPATH and sets up your environment.

.. code-block:: bash
    ml python3
    ml use [module_path]
    ml benchpro

#. You will likely get a warning stating you need to install missing user files. Follow the instructions to clone those files from this repository:

.. code-block:: bash
    ml git
    git clone https://github.com/TACC/benchpro.git $HOME/benchpro

#. Finally, you need to run a validation process to confirm that your system, environment and directory structures are correctly configured before using BenchPRO for the first time. Run this with:

.. code-block:: bash
    benchpro --validate

NOTE: Some of the hardware data collection functionality provided by BenchPRO requires root access, you can either run the permissions script below to privilege said scripts, or manage without this data collection feature.

.. code-block:: bash
    sudo -E $BP_HOME/resources/scripts/change_permissions.sh

#. Print help & version information:

.. code-block:: bash
    benchpro --help
    benchpro --version

#. Display some useful defaults 

.. code-block:: bash
    benchpro --defaults


Build an Application
============

This section will walk you through building your first application with BenchPRO using an included application profile.

1. List all available applications and benchmarks with:
.. code-block:: bash
    benchpro -a

#. Install LAMMPS:

.. code-block:: bash
    benchpro -b lammps

#. List applications currently installed:

.. code-block:: bash
    benchpro -la

You will see that LAMMPS is labelled as `DRY RUN` because `dry_run=True` in `$BP_HOME/settings.ini` by default. Therefore BenchPRO generated a LAMMPS compilation script but did not submit it to the scheduler to execute the build process. You can obtain more information about your LAMMPS deployment with:

.. code-block:: bash
    benchpro -qa lammps     

You can examine the build script `build.batch` located in the `build_prefix` directory. Submit your LAMMPS compilation script to the scheduler manually, or
#. Remove the dry_run build:

.. code-block:: bash
    benchpro -da lammps

#. Overload the default 'dry_run' value and rebuild LAMMPS with: 

.. code-block:: bash
    benchpro -b lammps o dry_run=False

#. Now check the details and status of your LAMMPS compilation job with:

.. code-block:: bash
    benchpro -qa lammps

In this example, parameters in `$BP_HOME/config/build/lammps.cfg` were used to contextualize the build template `$BP_HOME/templates/build/lammps.template` and produce a job script. Parameters for the job, system architecture, compile time optimizations and a module file were automatically generated. You can load your LAMMPS module with `ml lammps`. For each application that is built, a 'build_report' is generated in order to preserve metadata about the application. This build report is referenced whenever the application is used to run a benchmark, and also when this application is captured to the database. You can manually examine this report in the application directory or by using the `--queryApp / -qa` flag.


Run a Benchmark
============

We can now run a benchmark with our LAMMPS installation. There is no need to wait for the LAMMPS build job to complete because BenchPRO is able create job dependencies between tasks when needed. In fact, if `build_if_missing=True` in `$BP_HOME/settings.ini`, BenchPRO would detect that LAMMPS is not installed for the current system when attempting to run a benchmark and build it automatically without us doing the steps above. The process to run a benchmark is similar to compilation; a configation file is used to populate a template script. A benchmark run is specified with `--bench / -B`. The argument may be a single benchmark label, or a benchmark 'suite' (i.e collection of benchmarks) defined in `settings.ini`. Once again you can check for available benchmarks with `--avail / -a`.  

1. If you haven't already, modify '$BP_HOME/settings.ini' to disable the dry_run mode.

.. code-block:: bash
    dry_run = False

#. Generate the LAMMPS Lennard-Jones benchmark with: 

.. code-block:: bash
    benchpro -B ljmelt 

We changed `settings.ini` so we don't need to use the `--overload / -o` flag to disable the dry_run mode. 
Note that BenchPRO will use the default scheduler parameters for your system from a file defined in `$BP_HOME/config/system.cfg`. You can overload individual parameters using `--overload`, or use another scheduler config file with the flag `--sched [FILENAME]`. 

#. Check the benchmark report with:

.. code-block:: bash
    benchpro -qr ljmelt

#. Because this Lennard-Jones benchmark was the last BenchPRO job executed, a useful shortcut is available to check this report:

.. code-block:: bash
    benchpro --last


In this example, parameters in `$BP_HOME/config/bench/lammps_ljmelt.cfg` were used to contetualize the template `$BP_HOME/templates/bench/lammps.template`
Much like the build process, a 'bench_report' was generated to store metadata associated with this benchmark run. It is stored in the benchmark result direcotry and will be used in the next step to capture the result to the database.

### Capture Benchmark Result

A benchmark result exists in four states, during scheduler queueing and execution it is considered in `running` state, upon completion it will remain on the local system in a `complete` state, until it is captured it to the database when its state changes to `captured` or `failed`. 

1. We can check on the status of all benchmark runs with:

.. code-block:: bash
benchpro -lr 

#. Once your LAMMPS benchmark result is in the complete state, capture all complete results to the database with:

.. code-block:: bash
    benchpro -C

#. You can now query your result in the database with :

.. code-block:: bash
    benchpro --dbResult 

#. You can provide search criteria to narrow the results and export these results to a .csv file with:

.. code-block:: bash
    benchpro --dbResult username=$USER system=$TACC_SYSTEM submit_time=$(date +"%Y-%m-%d") --export

Because your LAMMPS application was recently compiled and not present in the database, it was also added automatically.

#. Query your application details using the [APPID] from above:

.. code-block:: bash
    benchpro --dbApp [APPID]

#. Once you are satisfied the benchmark result and its associated files have been uploaded to the database, you can remove the local copy with:

.. code-block:: bash
    benchpro --delResult captured


Web frontend
============

The captured applications and benchmark results are available through a web frontend here http://benchpro.tacc.utexas.edu/. 

Useful commands
============

You can print the default values of several important parameters with:

.. code-block:: bash
    benchpro --setup


It may be useful to review your previous BenchPRO commands, do this with:

.. code-block:: bash
    benchpro --history

You can remove tmp, log, csv, and history files by running:

.. code-block:: bash
    benchpro --clean

clean will NOT remove your all installed applications, to do that run:

.. code-block:: bash
    benchpro --delApp all


# BenchTool
BenchTool is a framework to automate and standardize application compilation, benchmarking and result collection on large scale HPC systems. Current list of applications included with BenchTool:

| Applications               | Synthetic Benchmarks     |
|----------------------------|--------------------------|
| AMBER 20                   | HPL                      |
| GROMACS 2020.2             | HPCG                     |
| LAMMPS 3Mar20              | STREAM                   |
| MILC 7.8.1                 | GPCNET                   |
| NAMD 2.14                  |                          |
| OpenFOAM v2012             |                          |
| Quantum Espresso 6.5       |                          |
| SWIFTsim 0.9.0             |                          |
| WRF 4.2                    |                          |
| SpecFEM3D Globe 7.0.2      |                          |

These application profiles have been created with the expectation that corresponding source code and datasets are available in the local repository directory which is the case for TACC systems. New applications are being added.

## Getting Started

### Install

The BenchTool site package should already be installed on most TACC systems. If it is not, contact mcawood@tacc.utexas.edu or install it from the [benchtool-package](https://github.com/TACC/benchtool-package) repository. Assuming the site package is available, you need to install a local copy of the configuration and template files to use BenchTool.

| System             | Module Path     |
|--------------------|--------------------------|
| Frontera           | /scratch1/hpc_tools/benchtool/modulefiles |
| Stampede2          | TBD                      |
| Lonestar6          | TBD                      |
| Longhorn           | TBD                      |


1 Load the BenchTool site package using the appropriate system path above, this module adds BenchTool to PYTHONPATH and sets up your environment.
```
ml python3
module use [module_path]
ml benchtool
```
2 You will likely get a warning stating you need to install missing user files. Follow the instructions to clone those files from this repository:
```
ml git
git clone https://github.com/TACC/benchtool.git $HOME/benchtool
```
3 Finally, you need to run a validation process to confirm that your system, environment and directory structures are correctly configured before using BenchTool for the first time. Run this with:
```
benchtool --validate
```
NOTE: Some of the hardware data collection functionality provided by BenchTool requires root access, you can either run the permissions script below to privilege said scripts, or manage without this data collection feature.
```
sudo -E $BT_HOME/resources/scripts/change_permissions.sh
```
4 Print help & version information:
```
benchtool --help
benchtool --version
```
5 Display some useful defaults 
```
benchtool --defaults
```
### Build an Application
This section will walk you through building your first application with BenchTool using an included application profile.

1 List all available applications and benchmarks with:
```
benchtool -a
```
2 Install LAMMPS:
```
benchtool -b lammps
```
3 List applications currently installed:
```
benchtool -la
```
You will see that LAMMPS is labelled as `DRY RUN` because `dry_run=True` in `$BT_HOME/settings.ini` by default. Therefore BenchTool generated a LAMMPS compilation script but did not submit it to the scheduler to execute the build process. You can obtain more information about your LAMMPS deployment with:
```
benchtool -qa lammps
```
You can examine the build script `build.batch` located in the `build_prefix` directory. Submit your LAMMPS compilation script to the scheduler manually, or
4 Remove the dry_run build:
```
benchtool -da lammps
```
5 Overload the default 'dry_run' value and rebuild LAMMPS with: 
```
benchtool -b lammps -o dry_run=False
```
6 Now check the details and status of your LAMMPS compilation job with:
```
benchtool -qa lammps
```
In this example, parameters in `$BT_HOME/config/build/lammps.cfg` were used to contextualize the build template `$BT_HOME/templates/build/lammps.template` and produce a job script. Parameters for the job, system architecture, compile time optimizations and a module file were automatically generated. You can load your LAMMPS module with `ml lammps`. For each application that is built, a 'build_report' is generated in order to preserve metadata about the application. This build report is referenced whenever the application is used to run a benchmark, and also when this application is captured to the database. You can manually examine this report in the application directory or by using the `--queryApp / -qa` flag.

### Run a Benchmark

We can now run a benchmark with our LAMMPS installation. There is no need to wait for the LAMMPS build job to complete because BenchTool is able create job dependencies between tasks when needed. In fact, if `build_if_missing=True` in `$BT_HOME/settings.ini`, BenchTool would detect that LAMMPS is not installed for the current system when attempting to run a benchmark and build it automatically without us doing the steps above. The process to run a benchmark is similar to compilation; a configation file is used to populate a template script. A benchmark run is specified with `--bench / -B`. The argument may be a single benchmark label, or a benchmark 'suite' (i.e collection of benchmarks) defined in `settings.ini`. Once again you can check for available benchmarks with `--avail / -a`.  

1 If you haven't already, modify `$BT_HOME/settings.ini' to disable the dry_run mode.
```
dry_run = False
```
2 Generate the LAMMPS Lennard-Jones benchmark with: 
```
benchtool -B ljmelt 
```
We changed `settings.ini` so we don't need to use the `--overload / -o` flag to disable the dry_run mode. 
Note that BenchTool will use the default scheduler parameters for your system from a file defined in `$BT_HOME/config/system.cfg`. You can overload individual parameters using `--overload`, or use another scheduler config file with the flag `--sched [FILENAME]`. 

3 Check the benchmark report with:
```
benchtool -qr ljmelt
```
4 Because this Lennard-Jones benchmark was the last BenchTool job executed, a useful shortcut is available to check this report:
```
benchtool --last
```

In this example, parameters in `$BT_HOME/config/bench/lammps_ljmelt.cfg` were used to contetualize the template `$BT_HOME/templates/bench/lammps.template`
Much like the build process, a 'bench_report' was generated to store metadata associated with this benchmark run. It is stored in the benchmark result direcotry and will be used in the next step to capture the result to the database.

### Capture Benchmark Result

A benchmark result exists in four states, during scheduler queueing and execution it is considered in `running` state, upon completion it will remain on the local system in a `complete` state, until it is captured it to the database when its state changes to `captured` or `failed`. 
1 We can check on the status of all benchmark runs with:
```
benchtool -lr 
```
2 Once your LAMMPS benchmark result is in the complete state, capture all complete results to the database with:
```
benchtool -C
```
3 You can now query your result in the database with :
```
benchtool --dbResult 
```
4 You can provide search criteria to narrow the results and export these results to a .csv file with:
```
benchtool --dbResult username=$USER system=$TACC_SYSTEM submit_time=$(date +"%Y-%m-%d") --export
```
Because your LAMMPS application was recently compiled and not present in the database, it was also added automatically.

5 Query your application details using the [APPID] from above:
```
benchtool --dbApp [APPID]
```
6 Once you are satisfied the benchmark result and its associated files have been uploaded to the database, you can remove the local copy with:
```
benchtool --delResult captured
```

### Web frontend

The captured applications and benchmark results are available through a web frontend here http://tacc-stats03.tacc.utexas.edu/. 

### Useful commands

You can print the default values of several important parameters with:
```
benchtool --setup
```

It may be useful to review your previous BenchTool commands, do this with:
```
benchtool --history
```

You can remove tmp, log, csv, and history files by running:
```
benchtool --clean
```

clean will NOT remove your all installed applications, to do that run:
```
benchtool --delApp all
```

## Adding a new Application
BenchTool requires two input files to build an application: a config file containing contextualization parameters, and a build template file which will be populated with these parameters and executed. 

### 1. Build config file

A full detailed list of config file fields are provided at the bottom of this README. A config file is seperated into the following sections:
 - `[general]` where information about the application is specified. `module_use` can be provided to add a nonstandard path to MODULEPATH. By default BenchTool will attempt to match this config file with its corresponsing template file. You can overwrite this default filename by adding the `template` field to this section. 
 - `[modules]` where `compiler` and `mpi` are required, while more modules can be specified if needed. Every module must be available on the local machine, if you are cross compiling to another platform (e.g. to frontera-rtx) and require system modules not present on the current node, you can set `check_modules=False` in settings.ini to bypass this check. 
 - `[config]`  where variables used in the build template script can be added.

You can define as many additional parameters as needed for your application. Eg: additional modules, build options, etc. All parameters `[param]` defined here will be used to fill `<<<[param]>>>` variables of the same name in the template file, thus consistent naming is important.
This file must be located in `$BT_HOME/config/build`, preferably with the naming scheme `[label].cfg`. 

### 2. Build template file

This template file is used to gerenate a contextualized build script which will executed to compile the application.
Variables are defined with `<<<[param]>>>` syntax and populated with the variables defined in the config file above.
If a `<<<[param]>>>` in the build template in not successfully populated and `exit_on_missing=True` in settings.ini, an expection will be raised.
You are able to make use of the `local_repo` variable defined in `$BT_HOME/settings.ini` to store and use files locally. 
This file must be located in `$BT_HOME/templates/build`, with the naming scheme `[label].template` 

### 3. Module template file (optional)

You can define your own .lua module template, otherwise a generic one will be created for you.
This file must be located in `$BT_HOME/templates/build`, with the naming scheme `[label].module` 

The application added above would be built with the following command:
```
benchtool --build [code]
```
Note: BenchTool will attempt to match your application input to a unique config filename. The specificity of the input will depend on the number of similar config files.
It may be helpful to build with `dry_run=True` initially to confirm the build script was generated as expected, before `--removing` and rebuilding with `dry_run=False` to compile.

## Adding a new Benchmark

The process of setting up an application benchmark is much the same as the build process; a config file is used to populate a benchmark template. 

### 1. Benchmark config file

A full detailed list of config file fields is provided below. A config file is seperated into the following sections:
 - `[requirements]` where fields are defined to create requirements to an application. More fields produce a finer, more specific application selection criteria.
 - `[runtime]` where job setup parameters are defined.
 - `[config]` where bench script parameters are defined.
 - `[result]` where result collection parameters are defined.

Any additional parameters may be defined in order to setup the benchmark, i.e dataset label, problem size variables etc.
This file must be located in `$BT_HOME/config/bench`, preferably with the naming scheme `[label].cfg`.

### 2. Benchmark template file  

As with the build template. The benchmark template file is populated with the parameters defined in the config file above. This file should include setup of the dataset, any required pre-processing or domain decomposition steps if required, and the appropriate mpi_exec command.
You are able to make use of the `local_repo` variable defined in `$BT_HOME/settings.ini` to copy local files. 

This file must be located in `$BT_HOME/templates/bench`, with the naming scheme `[label].template`. 

The benchmark added above would be run with the following command:
```
benchtool --bench [dataset]
```
Note: BenchTool will attempt to match your benchmark input to a unique config filename. The specificity of the input will depend on the number of similar config files.
It may be helpful to build with `dry_run=True` initially to confirm the build script was generated as expected, before `--removing` and rebuilding with `dry_run=False` to launch the build job.

## Advanced Features

BenchTool supports a number of additional features which may be of use.

### Overloading parameters

Useful for changing a setting for a onetime use. 
Use `benchtool --setup` to confirm important default params from $BT_HOME/settings.ini
You can overload any of the default parameters read from file with the `-o`/`--overload` flag. 
Parameters are overloaded with a list of key-value pairs, or by provided a filename with overloads defined.
An exception will be raised if a provided overload key does not match an existing parameter read from file.

Example 1: overload dry_run and build locally rather than via sched:
```
benchtool -b lammps --overload dry_run=False build_mode=local
```

Example 2: run LAMMPS benchmark with modified nodes, ranks and threads:
```
benchtool -B ljmelt --overload nodes=16 ranks_per_node=8 threads=6
```

Example 3: run a collection of benchmarks across a range of hardware, allowing only 1 active task at a time:
```
vim layout.txt
> nodes = 16,32,64
> ranks_per_node=2
> threads=28

benchtool -B ljmelt gromacs_stmv new_conus12km --overload layout.txt max_running_jobs=1
```

### Input list support

Comma delimited lists of nodes, ranks and threads are supported which can be useful for automating scaling and optimization investigations.
These lists can be specified in the config file, or via the overload feature detailed above.
A list of nodes will be iterated over, and for each, the list of threads and ranks will both be iterated over.
If the single thread and multiple ranks are specified, the same thread value will be used for all ranks, and vice versa. If ranks and threads and both larger than a single value but not equal length, an exception will be raised.

Example 1: Run LAMMPS on 4, 8 and 16 nodes, first using 4 ranks per node with 8 threads each, and then 8 ranks per node using 4 threads each:
```
benchtool --bench ljmelt --overload nodes=4,8,16 ranks_per_node=4,8 threads=8,4
```
From this example, the resulting set of runs would look like:
```
Nodes=  4, ranks= 4, threads= 8 
Nodes=  4, ranks= 8, threads= 4 
Nodes=  8, ranks= 4, threads= 8 
Nodes=  8, ranks= 8, threads= 4 
Nodes= 16, ranks= 4, threads= 8 
Nodes= 16, ranks= 8, threads= 4 
```

### Local build and bench modes

Allows you to run the generated scripts in a shell on the local machine rather than  via the scheduler.
Useful for evaluating hardware which is not integrated into the scheduler.

In settings.ini `build_mode` and `bench_mode` are responsible for selecting this feature. Values `sched` or `local` are accepted, or an exception will be raised. 
You can opt to build locally and run via the scheduler, or vice a versa.

### Benchmarks with no application dependency

Some benchmarks such as synthetics are microbenchmarks do require an application be compiled and module created.
You are able to create a benchmark without any dependency to an application. 
This is done by not specifying any values in the [requirements] section of the benchmark config file.


# Inputs & settings format

## Command line arguments

| Argument                                              | Description                                                   |
|-------------------------------------------------------|---------------------------------------------------------------|
| --help                                                | Print usage info.                                             |
| --validate                                            | Confirm the installation is correctly configured.             |
| --clean                                               | Remove logs and other temp files left after an execption.     |
| --avail                                               | Print the available application and benchmark profiles.       |
| --build [LABEL]                                       | Compile an available application.                             |
| --listApps                                            | Print a list of currently installed applications.             |
| --queryApp [LABEL]                                    | Print compilation information for an installed app.           |
| --delApp [LABEL]                                      | Remove application installation matching inpout.              |
| --bench [LABEL]                                       | Run a benchmark.                                              |
| --sched [LABEL]                                       | Use with '--build' or '--bench' to specify a custom scheduler config file instead of the system default. |
| --listResults [all/running/pending/captured/failed]   | List all benchmark results in requested state.                |
| --queryResult [LABEL]                                 | Print config and result of a benchmark run.                   |
| --capture                                             | Validate and capture all pending results to the database.     |
| --dbResult [all/LIST]                                 | Display either all results from DB or results matching colon delimited search list, eg "username=mcawood:code=lammps". |
| --dbApp [APPID]                                       | Display application details                                   |
| --delResult [all/captured/failed/LABEL]               | Remove local benchmark results matching input criteria.       |
| --overload [LIST]                                     | Replace options in settings.ini or any config file, acceptes a colon delimited list. |

## Global settings
Global settings are defined in the file `settings.ini`

| Label             | Default                       | Description                                                                       |
|-------------------|-------------------------------|-----------------------------------------------------------------------------------|
| **[paths]**       |                               | -                                                                                 |
| install_dir       |                               | Populated by installer                                                            |
| build_dir         |                               | Populated by installer                                                            |
| bench_dir         |                               | Populated by installer                                                            |
| **[common]**      |                               | -                                                                                 |
| dry_run           | True                          | Generates job script but does not submit it, useful for testing                   |
| debug             | True                          | Prints additional nonessential messages                                           |
| timeout           | 5                             | Delay in seconds after warning and before file deletion event                     |
| sl                | /                             | Filesystem separator                                                              |
| system_env        | $TACC_SYSTEM                  | Environment variable contained system label (eg: stampede2)                       |
| sched_mpi         | ibrun                         | MPI launcher to use in job script                                                 |
| local_mpi         | mpirun                        | MPI launcher to use on local machine                                              |
| tree_depth        | 6                             | Determines depth of app installation tree                                         |
| topdir_env_var    | $BT_HOME                   | BenchTool's working directory environment variable (exported in from sourceme)    |
| log_dir           | ./log                         | Log file directory                                                                |
| script_basedir    | ./scripts                     | Result validation and system check script directory                               |
| ssh_key_dir       | ./auth                        | Directory containing SSH keys for server access                                   |
| mpi_blacklist     | login,staff                   | Hostnames containing these strings are forbidden from executing MPI code          |
| **[config]**      |                               | -                                                                                 |
| config_basedir    | ./config                      | Top directory for config files                                                    |
| build_cfg_dir     | build                         | Build config file subdirectory                                                    |
| bench_cfg_dir     | bench                         | Benchmark config file subdirectory                                                |
| sched_cfg_dir     | sched                         | Scheduler config file subdirectory                                                |
| system_cfg_file   | system.cfg                    | File containing system default architecture and core count                        |
| arch_cfg_file     | architecture_defaults.cfg     | File containing default compile optimization flags                                |
| compile_cfg_file  | compiler.cfg                  | File containing compiler environment variables                                    |
| **[templates]**   |                               | -                                                                                 |
| exit_on_missing   | True                          | Exit if template is not fully populates (missing parameters found)                |
| template_basedir  | ./templates                   | Top directory for template files                                                  |
| build_tmpl_dir    | build                         | Build template file subdirectory                                                  |
| sched_tmpl_dir    | sched                         | Scheduler template file subdirectory                                              |
| bench_tmpl_dir    | bench                         | Benchmark template file subdirectory                                              |
| compile_tmpl_file | compiler.template             | Template for setting environment variables                                        |
| **[builder]**     |                               | -                                                                                 |
| app_env_var       | $BT_APPS                      | Application directory environment variable                                        |
| overwrite         | False                         | If existing installation  is found in build path, replace it                      |
| build_mode        | sched                         | Accepts 'sched' or 'local', applications compiled via sched job or local shell    |
| build_basedir     | ./build                       | Top directory for application installation tree                                   |
| build_subdir      | build                         | Application subdirectory for build files                                          |
| install_subdir    | install                       | Application subdirectory for installation (--prefix)                              |
| build_log_file    | build                         | Label for build log                                                               |
| build_report_file | build_report.txt              | Application build report file name                                                |
| max_build_jobs    | 5                             | Maximum number of concurrent running build jobs allowed in the scheduler          |
| **[bencher]**     |                               |                                                                                   |
| result_env_var    | $BT_RESULTS                   | Application directory environment variable                                        |
| bench_mode        | sched                         | Accepts 'sched' or 'local', benchmarks run via sched job or local shell           |
| build_if_missing  | True                          | If application needed for benchmark is not currently installed, install it        |
| local_repo    | /scratch/06280/mcawood/local_repo  | Directory containing benchmark datasets                                          |
| bench_basedir     | ./results                     | Top directory containing bechmark runs                                            |
| bench_log_file    | bench                         | Label for run log                                                                 |
| bench_report_file | bench_report.txt              | Benchmark report file                                                             |
| output_file       | output.log                    | File name for benchmark stdout                                                    |
| **[results]**     |                               |                                                                                   |
| move_failed_result| True                          | Move failed results to subdir                                                     |
| result_scripts_dir| results                       | Subdirectory inside [script_basedir] containing result validation scripts         |
| results_log_file  | capture                       | Label for capture log                                                             |
| pending_subdir    | pending                       | Subdirectory for pending results                                                  |
| captured_subdir   | captured                      | Subdirectory for captured results                                                 |
| failed_subdir     | failed                        | Subdirectory for failed results                                                   |
| **[database]**    |                               |                                                                                   |
| db_host           | tacc-stats03.tacc.utexas.edu  | Database host address                                                             |
| db_name           | bench_db                      | Database name                                                                     |
| db_user           | postgres                      | Database user                                                                     |
| db_passwd         | postgres                      | Datanase user password                                                            |
| result_table      | results_result                | Postgres results table name                                                       |
| app_table         | results_application           | Django application table name                                                     |
| file_copy_handler | scp                           | File transfer method, only scp working currently                                  |
| ssh_user          | mcawood                       | Username for SSH access to database host                                          |
| ssh_key           | id_rsa                        | SSH key filename (stored in ./auth)                                               |
| django_static_dir | /home/mcawood/benchdb/static  | Directory for Django static directory (destination for file copies)               |
| **[system]**      |                               | -                                                                                 |
| system_scripts_dir| system                        | Subdirectory in which hardware info collection tools are located                  |
| system_utils_dir  | hw_utils                      |                                                                                   |
| **[suites]**      |                               |                                                                                   |
| [Suite label]     | [list of apps/benchmarks]     | Several example included for 

## Application config files
These config files contain parameters used to populate the application build template file, config files are broken in sections corresponding to general settings, system modules and configuration parameters.

| Label             | Required? | Description                                                                      |
|-------------------|-----------|----------------------------------------------------------------------------------|
| **[general]**     |           |                                                                                  |
| code              | Y         | Application identifier.                                                          |
| version           | Y         | Application version label, accepts x.x, x-x, or strings like 'stable'.           |
| system            | N         | TACC system identifier, if left blank will use $TACC_SYSTEM.                     |
| build_prefix      | N         | Custom build (outside of default tree).                                          |
| build_template    | N         | Overwrite default build template file.                                           | 
| **[modules]**     |           | NOTE: user may add as many custom fields to this section as needed.              |
| compiler          | Y         | Module name of compile, eg: 'intel/18.0.2' or just 'intel' for LMod default.     |
| mpi               | Y         | Module name of MPI, eg: 'impi/18.0.2' or just 'impi' for LMod default.           |
| **[config]**      |           | NOTE: user may add as many fields to this section as needed.                     |
| arch              | N         | Generates architecture specific optimization flags. If left blank will use system default, set to 'system' to combine with 'opt_flags' below  | 
| opt_flags         | N         | Used to add additional optimization flags, eg: '-g -ipo'  etc.  If arch is not set, this will be only optimization flags used.        |
| build_label       | N         | Custom build label, replaces arch default eg: skylake-xeon. Required if 'opt_flags' is set and 'arch' is not                 |
| bin_dir           | N         | Set bin dir suffix to add executable to PATH, eg: bin, run etc.                  | 
| exe               | Y         | Name of application executable, used to check compilation was successful.        |
| collect_hw_stats  | N         | Runs the hardware stats collection tool after build.                             |

## Benchmark config file
These config files contain parameters used to populate the benchmark template script. The file structure is:

| Label                 | Required?  | Description                                                                      |
|-----------------------|------------|----------------------------------------------------------------------------------|
| **[requirements]**    |            | NOTE: user may add as many fields to this section as needed.                     |
| code                  | N          | This benchmark requires an installed application matching code=""                |
| version               | N          | This benchmark requires an installed application matching version=""             |
| label                 | N          | This benchmark requires an installed application matching label=""               |
| **[runtime]**         |            |                                                                                  |
| nodes                 | Y          | Number of nodes on which to run, accepts comma-delimited list.                   |
| ranks_per_node        | N          | MPI ranks per node.                                                              |
| threads               | Y          | Threads per MPI rank.                                                            |
| max_running_jobs      | N          | Sets maximum number of concurrent running scheduler jobs.                        |
| hostlist              | Depends    | Either hostlist or hostfile required if benchmarking on local system (no sched). |
| hostfile              | Depends    |                                                                                  |    
| **[config]**          |            | NOTE: user may add as many fields to this section as needed.                     |
| label                 | Depends    | Required if this benchmark has no application dependency.                        | 
| exe                   | Y          | Application executable.                                                          |
| dataset               | Y          | Benchmark dataset label.                                                         |
| collect_hw_stats      | N          | Run hardware info collection after benchmark.                                    |
| output_file           | N          | File to redirect stdout, if empty will use stdout for sched jobs, or 'output_file' from settings.ini for local job.  | 
| **[result]**          |            |                                                                                  |
| description           | N          | Result explanation/description.                                                  |
| method                | Y          | Results extraction method. Currently 'expr' or 'script' modes supported.         |
| expr                  | Depends    | Required if 'method=expr'. Expression for result extraction (Eg: "grep 'Performance' <file> | cut -d ' ' -f 2")"|
| script                | Depends    | Required if 'method=script'. Filename of script for result extraction.           |
| unit                  | Y          | Result units.                                                                    |


## Directory structure

| Directory         | Purpse                                                    |
|-------------------|-----------------------------------------------------------|
| $BT_APPS                 | Application build basedir.                                |
| $BT_HOME/config          | config files containing template parameters.              |
| $BT_HOME/log             | Build, bench and catpure log files.                       |
| $BT_HOME/resources       | Contains useful content including modulefiles, hardware collection and result validation scripts.    |
| $BT_RESULTS              | Benchmark result basedir.                                 |
| $BT_HOME/templates       | job template files                                        |

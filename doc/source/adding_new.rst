Contributions
===

BenchPRO requires two input files to build an application: a config file containing contextualization parameters, and a build template file which will be populated with these parameters and executed. 

### 1. Build config file

A full detailed list of config file fields are provided at the bottom of this README. A config file is seperated into the following sections:
 - `[general]` where information about the application is specified. `module_use` can be provided to add a nonstandard path to MODULEPATH. By default BenchPRO will attempt to match this config file with its corresponsing template file. You can overwrite this default filename by adding the `template` field to this section. 
 - `[modules]` where `compiler` and `mpi` are required, while more modules can be specified if needed. Every module must be available on the local machine, if you are cross compiling to another platform (e.g. to frontera-rtx) and require system modules not present on the current node, you can set `check_modules=False` in settings.ini to bypass this check. 
 - `[config]`  where variables used in the build template script can be added.

You can define as many additional parameters as needed for your application. Eg: additional modules, build options, etc. All parameters `[param]` defined here will be used to fill `<<<[param]>>>` variables of the same name in the template file, thus consistent naming is important.
This file must be located in `$BP_HOME/config/build`, preferably with the naming scheme `[label].cfg`. 

### 2. Build template file

This template file is used to gerenate a contextualized build script which will executed to compile the application.
Variables are defined with `<<<[param]>>>` syntax and populated with the variables defined in the config file above.
If a `<<<[param]>>>` in the build template in not successfully populated and `exit_on_missing=True` in settings.ini, an expection will be raised.
You are able to make use of the `local_repo` variable defined in `$BP_HOME/settings.ini` to store and use files locally. 
This file must be located in `$BP_HOME/templates/build`, with the naming scheme `[label].template` 

### 3. Module template file (optional)

You can define your own .lua module template, otherwise a generic one will be created for you.
This file must be located in `$BP_HOME/templates/build`, with the naming scheme `[label].module` 

The application added above would be built with the following command:
```
benchpro --build [code]
```
Note: BenchPRO will attempt to match your application input to a unique config filename. The specificity of the input will depend on the number of similar config files.
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
This file must be located in `$BP_HOME/config/bench`, preferably with the naming scheme `[label].cfg`.

### 2. Benchmark template file  

As with the build template. The benchmark template file is populated with the parameters defined in the config file above. This file should include setup of the dataset, any required pre-processing or domain decomposition steps if required, and the appropriate mpi_exec command.
You are able to make use of the `local_repo` variable defined in `$BP_HOME/settings.ini` to copy local files. 

This file must be located in `$BP_HOME/templates/bench`, with the naming scheme `[label].template`. 

The benchmark added above would be run with the following command:
```
benchpro --bench [dataset]
```
Note: BenchPRO will attempt to match your benchmark input to a unique config filename. The specificity of the input will depend on the number of similar config files.
It may be helpful to build with `dry_run=True` initially to confirm the build script was generated as expected, before `--removing` and rebuilding with `dry_run=False` to launch the build job.


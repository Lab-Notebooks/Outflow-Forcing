## Reproducibility Capsule for Multiphase Simulations using Flash-X

A reproducibility archive of incompressible multiphase flow simulations for the following papers:
- [A Vortex Damping Outflow Forcing for Multiphase Flows with Sharp Interfacial Jumps](https://arxiv.org/pdf/2306.10174.pdf)
- [BubbleML: A Multi-Physics Dataset and Benchmarks for Machine Learning](https://arxiv.org/pdf/2307.14623.pdf)

The corresponding data repositories are hosted here:
- https://anl.box.com/s/9fkwioyxhgr1dyoxeg5j3mv571r0ec7h
- https://anl.box.com/s/4c18o04hj05rxa2dle9lx4iads93o9vj

<p align="center"> <img src="./icon.gif" width="350" style="border:none;background:none;"/> </p>

## Overview

This repository provides a lab notebook for running multiphase fluid dynamics simulations using Flash-X (https://flash-x.org), a multiphysics simulation software instrument. Design and organization of this notebook serves as a tutorial to setup production simulations for multiphase fluid-flow problems, archive data, perform visualization and data analysis, and maintain records/notes for scientific reproducibility and accountability. 

## Dependencies

A comprehensive lab environment to work with high-fidelity simulations should include,

- A scalable and performant computational solver.
- A well organized design of computational experiments that can handle complexities of parametric studies.
- Visualization tools to generate animations/images.
- Tools to extract/process information from simulation data to perform statistical/data analysis.

Flash-X (https://github.com/Flash-X/Flash-X) addresses the first requirement by using AMReX (https://github.com/AMReX-Codes/amrex) $\textemdash$ a framework for block-structured Adaptive Mesh Refinement (AMR) $\textemdash$ for grid management, along with its native Application Programming Interface (API) to solve complex multiphase problems. Details of the numerical solver can be found in our previous publications,

- [A Formulation for High-Fidelity Simulations of Pool Boiling in Low Gravity](https://www.sciencedirect.com/science/article/abs/pii/S030193221930165X)
- [An Investigation of The Gravity Effects on Pool Boiling Heat Transfer via High-Fidelity Simulations](https://www.sciencedirect.com/science/article/abs/pii/S0017931021009315?dgcid=author#!)

Building Flash-X in AMReX mode requires that Message Passing Interface (MPI) library is installed and available for use on platforms where users wish to run their simulations. Path to location of the library should be defined in ``environment.sh`` located in the project root directory. Details for setting these variables are provided in subsequent sections.

Organization of computational experiments is implemented using Jobrunner, which enables reuse of files/scripts along directory trees and ensures strict organization rules. Details on Jobrunner are provided separately in its own repository (https://github.com/akashdhruv/Jobrunner), and can be installed by running,

```
pip install PyJobRunner==2023.8 # Version only for reference, there is no explicit version constraint for this software
```

`pip` should point to Python3.8+ package installer `pip3`

Software dependencies for this notebook are divided into two categories: 

- Generic: Dependencies that are independent of version constraints and are often pre-installed on HPC cluster or can be installed using package installers.
- Specific: Dependencies that have strict version constraints and are provided under the `software.` directory.

HDF5, which is used for I/O exists in both categories to allow for custom installation if a pre-existing installation on a HPC cluster does not suffice. We have faced this situation where HDF5 was not available on a cluster and we had to build it ourselves.

For visualization we rely on ParaView (https://www.paraview.org), which is treated as a generic dependency, and assume that it has already been installed on the platform by admins/users of this lab notebook. FlashKit (https://github.com/GWU-CFD/FlashKit) can be used to generate ParaView compatible files from HDF5 data generated by Flash-X. FlashKit  provides a slew of tools to work with Flash-X, however, we only rely on the command ``flashkit create xdmf`` to generate XDMF hyperslabs. FlashKit is a specific dependency and is provided under `software` and should be installed using the same Python version as Jobrunner.

To perform analysis of simulations results, and integrate them with Python packages for machine learning and statistics we use BoxKit (https://github.com/akashdhruv/BoxKit), which is an ongoing project to parallelize and scale data analysis of block-structured datasets. BoxKit is a generic dependency and should be installed using the same Python version as Jobrunner and FlashKit.

```
pip install BoxKit==2023.6.7 # Version only for reference, there is no explicit version constraint for this software
```
Following is a full list of dependencies:

1. Generic: Make (4.2.1), Python3.8+, GCC (9.4.0) /Intel MPI (openmpi-4.1.1) compilers with Fortran support, HDF5 with Fortran support, Paraview, Jobrunner, BoxKit
2. Specific: AMReX, FlashKit, Flash-X, HDF5 (if not statisfied through HPC system).

Note that versions for generic softwares are listed only to enhance knowledge related to data created for the existing experiments. 

## Organization 

Directory strucuture of this repository plays an important role in repoducibility and consitency of the computational experiments, and is one of the most important aspects that a user may want to focus on when extending/contributing towards current work. To understand the nuances of the design of the directory tree provided here, we recommened reading details of Jobrunner (https://github.com/akashdhruv/Jobrunner). 

```
$ tree Boiling-Simulations

├── Jobfile
├── environment.sh
├── sites
    ├── sedona
        ├── Makefile.h.FlashX
        ├── modules.sh
├── software
    ├── Jobfile
    ├── setupAMReX.sh
    ├── setupFlashX.sh
    ├── setupFlashKit.sh
├── simulation
    ├── PoolBoiling
        ├── SingleBubble
            ├── Jobfile
            ├── flashOptions.sh
            ├── flashBuild.sh
            ├── flashRun.sh
            ├── flash.toml
    ├── FlowBoiling
├── analysis
```

The directory tree is divided into three major components $\textemdash$ software, simulation, and analysis $\textemdash$ which rely on a common environment configuration defined in `environment.sh`. 

The ``software/`` component provides scripts to install software packages with compatible configuration. 

The ``simulation/`` component contains specific simulations as directory objects, with the ability to configure each of them with different flavors. As an example, ``simulation/PoolBoiling`` can be configured for single and multiple bubble problems by creating sub-directories ``simulation/PoolBoiling/SingleBubble`` and ``simulation/PoolBoiling/Gravity-FC72`` with their respective options, Jobfiles, and commands.

The ``analysis/`` component is designed to setup data analysis and machine learning workflows and is currently a work in progress.

## Usage

Once a user has installed necessary libaries/tools, i.e., Jobrunner, MPI, HDF5, and ParaView, and designed their customized `environment.sh`, they can install the remaining software stack by running the following command from the project root directory. Note that a new `sites/<site-name>` directory should be created using `sites/sedona` as an example and its name should be specified in `config.sh`. `config.sh` also contains SHA-1 information for all the specific software components. 

```
jobrunner setup software
```

This command will checkout appropriate SHA-1 for Flash-X, AMReX, and FlashKit, and install them using base libraries and paths provided in ``environment.sh``

Setting up a simulation is done in similar way by running setup command as,

```
jobrunner setup simulation/PoolBoiling/SingleBubble
```

and then running it using,

```
jobrunner submit simulation/PoolBoiling/SingleBubble
```

Make sure to edit Jobfiles as desired to change/update your schedular configuration.

TIP: use `--show` with `jobrunner setup` and `jobrunner submit` to see the parsed configuration for a working directory derived from Jobfiles along the directory tree.

To visualize data using ParaView run following from the working directory of a job run,

```
flashkit create xdmf -b <begin_number> -e <end_number>
```

The `<begin_number>` and `<end_number>` refer to the files containing the pattern `*_hdf5_plt_cnt_*`. The resulting `*.xmf` file is ParaView compatible.

BoxKit is used to access raw simulation data in Python. Several IPython notebooks and scripts in the `analysis/` folder use it to post processing. 

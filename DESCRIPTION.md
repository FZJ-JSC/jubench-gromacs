# GROMACS

## Purpose

GROMACS is a versatile package to perform molecular dynamics, i.e., simulate the Newtonian equations of motion for systems with hundreds to millions of
particles. It is primarily designed for biochemical molecules like proteins, lipids and nucleic acids that have a lot of complicated bonded interactions, but since GROMACS is extremely fast at calculating the nonbonded interactions (that usually dominate simulations) many groups are also using it for research on non-biological systems, e.g. polymers.

The benchmarks are taken from the Unified European Applications Benchmark Suite
(https://repository.prace-ri.eu/git/UEABS/ueabs/-/tree/r2.2-dev/gromacs). The `ion_channel` and `STMV` benchmarks are used.

## Source
The source code for GROMACS 2023 can be downloaded at ftp://ftp.gromacs.org/pub/gromacs/gromacs-2023.tar.gz. A copy of the tar ball is available in the `src` directory.

## Building

Instructions for building GROMACS can be found on its [web page](https://manual.gromacs.org/current/install-guide/index.html).

To build GROMACS with support for CUDA-capable GPUs you can use the following.

```bash
VERSION=2023
wget ftp://ftp.gromacs.org/pub/gromacs/gromacs-${VERSION}.tar.gz
tar axf gromacs-${VERSION}.tar.gz
cd gromacs-${VERSION}
mkdir build
cd build
cmake .. -DGMX_MPI=on -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON -DGMX_GPU=CUDA
make -j 8
```

You can also use JUBE to download and build GROMACS: `jube run benchmark/jube/jube_gromacs_build.xml`.

If you use JUBE, you need to adjust the CMake and module parameters in `jube_gromacs_build.xml` for your platform.

You may choose optimized FFT and BLAS libraries in accordance with the general rules for the exchange and usage of libraries. In particular the STMV benchmark requires a build with multi-GPU PME to scale (c.f. [Using CuFFTMp](https://manual.gromacs.org/current/install-guide/index.html#using-cufftmp) in the GROMACS manual, or sketched quickly below.)

### Modifications

GROMACS supports a wide range of architectures and changes to the source code should not be necessary. But if such changes are necessary to achieve performance, for example, a new particle-particle kernel that
exploits the architecture, the patch must be submitted with the results and be placed under an LGPL v. 2.1 license so it can be included in future versions of GROMACS.

You may also use a later released version of GROMACS.

### Example Compilation with Distributed FFT

As GROMACS is a widely used application, many compile optimization are available. As an example, instructions are sketched here for using a distributed FFT library, cuFFTmp, for the STMV sub-benchmark. As this is a highly platform-specific optimization, it is not included automatically in the JUBE script, but hints are given at the according places in the JUBE file.

Taking JUWELS Booster as an example, cuFFTmp needs to be available in the environment and enabled through the proper compiler switches.

```bash
ml NVHPC OpenMPI NCCL CMake Python-bundle-PyPI
export LD_LIBRARY_PATH=$EBROOTNVHPC/Linux_x86_64/2023/comm_libs/12.2/nvshmem_cufftmp_compat/lib:$EBROOTNVHPC/Linux_x86_64/2023/math_libs/12.2/lib64:$LD_LIBRARY_PATH
cmake ../ -DGMX_OPENMP=ON -DGMX_MPI=ON -DGMX_BUILD_OWN_FFTW=ON -DGMX_GPU=CUDA  -DGMX_CUDA_TARGET_SM=80 -DCMAKE_BUILD_TYPE=Release -DGMX_DOUBLE=off -DGMX_USE_CUFFTMP=ON -DcuFFTMp_ROOT=$EBROOTNVHPC/Linux_x86_64/2023/math_libs/12.2
```

(The environment variables `$EBROOTNVHPC` are set through the `NVHPC` environment module and point to the NVIDIA HPC SDK.)


## Execution

GROMACS uses a single executable to perform many different tasks.
The default name for an MPI enabled build is `gmx_mpi`. The different tasks are given as argument. To run one of the above examples you could use the following command line:

```bash
mpiexec -n 128 gmx_mpi mdrun -s ion_channel.tpr
```
You *must* use an MPI-enabled build even if you only use a single node.

### Multithreading

Any combinations of threads and MPI tasks are allowed as long as the specified number of nodes is used. The number of nodes, MPI tasks per node, and threads per task are given in the parameterset "systemParameter"
separately for each benchmark. These should be adjusted. You may run with fewer ranks than requested, for example, if you use 8 nodes with 4 ranks each and use one rank for PME, GROMACS would complain that 31 is a large prime. You can instead launch 31 MPI ranks.

### Input Data

The input files are `ion_channel.tpr` (TestCaseA) and `stmv.28M.tpr` (TestCaseC), which are automatically downloaded by JUBE when building GROMACS. They are also available at https://repository.prace-ri.eu/git/UEABS/ueabs/-/tree/r2.2-dev/gromacs as [GROMACS_TestCaseA.tar.xz](https://repository.prace-ri.eu/ueabs/GROMACS/2.2/GROMACS_TestCaseA.tar.xz) and [GROMACS_TestCaseC.tar.xz](https://repository.prace-ri.eu/ueabs/GROMACS/2.2/GROMACS_TestCaseC.tar.xz). You can download them using

```bash
wget https://repository.prace-ri.eu/ueabs/GROMACS/2.2/GROMACS_TestCaseA.tar.xz
wget https://repository.prace-ri.eu/ueabs/GROMACS/2.2/GROMACS_TestCaseC.tar.xz
```

A copy of the files is available in the `src` directory.

The archives unpack into corresponding subdirectories using, e.g.,

```bash
tar axvf GROMACS_TestCaseA.tar.xz
```

creates a subdirectory `GROMACS_TestCaseA` with the file `ion_channel.tpr` in it.

This is done automatically when using `jube`.

### Command Line

The benchmarks can be run with the following commands. Replace `mpiexec` with the proper launcher for your MPI. You may change, remove, and add options that do not change the result, for example, options necessary to run GROMACS on a GPU or other accelerator. The command used to launch the benchmark must be provided with the results.


#### Ion channel

```bash
mpiexec -n 22 gmx_mpi mdrun -s ion_channel.tpr -maxh 0.50 -resetstep 60000 -noconfout -nsteps 600000 -noconfout -g logfile -ntomp 6 -dlb yes -pme gpu -npme 1 -nstlist 100
```

#### STMV

```bash
mpiexec -n 8 gmx_mpi mdrun -s stmv.28M.tpr -maxh 2.90 -resetstep 10000 -noconfout -nsteps 170000 -g logfile -ntomp $threadspertask -tunepme no -pme gpu -npme 1 -bonded gpu -nstlist 400
```

##### Example Execution with Distributed FFT Library

Picking up the previous example of using cuFFTmp as a highly-optimized library with platform-specific tuning, an example execution of the case looks like the following on JUWELS Booster:

```bash
ml NVHPC OpenMPI NCCL CMake Python-bundle-PyPI
export LD_LIBRARY_PATH=$EBROOTNVHPC/Linux_x86_64/2023/comm_libs/12.2/nvshmem_cufftmp_compat/lib:$EBROOTNVHPC/Linux_x86_64/2023/math_libs/12.2/lib64:$LD_LIBRARY_PATH
export GMX_ENABLE_DIRECT_GPU_COMM=ON
export GMX_GPU_PME_DECOMPOSITION=1

srun -N 128 -n 512 -c 6 --time 00:30:00 --gres=gpu:4 --threads-per-core=2 gromacs/build/bin/gmx_mpi mdrun -s GROMACS_TestCaseC/stmv.28M.tpr -maxh 0.5 -resetstep 10000 -noconfout -nsteps 170000 -g logfile -ntomp 6 -tunepme no -pme gpu -npme 128 -bonded gpu -nstlist 400 -cpt -1 -pin on 
```

#### JUBE

Using JUBE you start the benchmarks with

```bash
jube run benchmark/jube/jube_gromacs_ionchannel_bench.xml
```

and

```bash
jube run benchmark/jube/jube_gromacs_STMV_bench.xml
```

Common options used for both benchmarks are defined in `jube_gromacs_common.xml`.


## Verification

The build used for the benchmark must be verified using regression
testing. This needs to be done by hand (see [Testing GROMACS for correctness](https://manual.gromacs.org/current/install-guide/index.html#testing-gromacs-for-correctness) in the installation guide of the GROMACS manual.)

Note, that some tests require multiple MPI ranks and must run on a node that is capable of running MPI jobs. If you use a GPU build, the node that runs the test may also need a GPU.

After adding the `gmx_mpi` binary to your PATH, you can run the regression tests from the build directory:

```bash
# Starting from your build directory
cd tests/regressiontests-2023
# This starts MPI runs using srun if applicable. Adjust to your system as necessary.
./gmxtest.pl -np 4 -mpirun srun all &> regression_2023.log
```

If you use a later version of GROMACS, it must pass the corresponding version of the regression test.

You need to provide the output of the regression tests (`regression_2023.log` in the example above).

## Results

The performance numbers are found at the end of the logfile.

```
               Core t (s)   Wall t (s)        (%)
       Time:    24382.088       84.660    28799.9
                 (ns/day)    (hour/ns)
Performance:       10.208        2.351
```

You need to provide the results for **both** benchmarks. If you use `jube` you can generate the table for the last benchmark runs including some extra information by running

```bash
jube result -a ion_channel_run
jube result -a STMV_run
```

## Baseline

The baseline configuration must be chosen such that the runtime metric (`Wall`) is less than or equal to 443 s (for `ion_channel` benchmark) and 418 s (`STMV` benchmark). The results were obtained with 3 and 128 nodes, respectively, of JUWELS Booster each with 4 A100 GPUs.

| Benchmark Name | Nodes | Performance [ns/d] | Wall t[s] |
| -------------- | ----- | ------------------ | --------- |
| ion_channel    |   3   |           263.4    |  443      |
| STMV           | 128   |            66.1    |  418      |





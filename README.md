# JUPITER Benchmark Suite: GROMACS

[![DOI](https://zenodo.org/badge/831351357.svg)](https://zenodo.org/badge/latestdoi/831351357) [![Static Badge](https://img.shields.io/badge/DOI%20(Suite)-10.5281%2Fzenodo.12737073-blue)](https://zenodo.org/badge/latestdoi/764615316)

This benchmark is part of the [JUPITER Benchmark Suite](https://github.com/FZJ-JSC/jubench). See the repository of the suite for some general remarks.

This repository contains the GROMACS benchmark. [`DESCRIPTION.md`](DESCRIPTION.md) contains details for compilation, execution, and evaluation.

[GROMACS](https://www.gromacs.org/) is a versatile package to perform molecular dynamics, i.e., simulate the Newtonian equations of motion for systems with hundreds to millions of particles.  
It is primarily designed for biochemical molecules like proteins, lipids, and nucleic acids that have a lot of complicated bonded interactions, but since GROMACS is extremely fast at calculating the nonbonded interactions (that usually dominate simulations) many groups are also using it for research on non-biological systems, e.g. polymers.


## Quickstart

### Compilation

The JUBE script automatically downloads GROMACS from the official website, and the input for the two possible runs from the [PRACE Unified European Applications Benchmark Suite](https://repository.prace-ri.eu/git/UEABS/ueabs). It's just a matter of using the following commands:

```bash
module load JUBE
jube run jube_gromacs_build.xml

```

This should take about an hour.

### Execution

In similar fashion, execute the two sub-benchmarks with the according JUBE scripts:

```bash
jube run jube_gromacs_STMV_bench.xml
jube run jube_gromacs_ionchannel_bench.xml
```


### JUBE sample output

#### ionchannel_run (CPU)

|  Nodes | MPI tasks | Threads per Task | Performance [ns/d] | Wall t[s] | MFLOPs        |
| ------ | --------- | ---------------- | ------------------ | --------- | ------------- |
|     64 |       256 |                6 |            354.035 |    15.253 | 136650833.463 |

#### STMV_run (GPU)

| Nodes | MPI tasks | Threads per Task | GPUs per Node | Performance [ns/d] | Wall t[s] |          MFLOPs | Version | Compiler |
|-------|-----------|------------------|---------------|--------------------|-----------|-----------------|---------|----------|
|   256 |      1024 |                6 |             4 |             72.172 |    23.945 | 43979566371.330 | 2023RC1 |    NVHPC |

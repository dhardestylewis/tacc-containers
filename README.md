# TACC Containers

A curated set of starter containers for building containers to eventually run on TACC systems.

| Image                                                             | Frontera | Stampede2 | Maverick2 | Longhorn | Local Dev |
|-|:-:|:-:|:-:|:-:|:-:|
| [tacc/tacc-centos7](#minimal-base-images)                              | X | X | X |   | X |
| [tacc/tacc-centos7-ppc64le](#minimal-base-images)                              |   |   |   | X | X |
| [tacc/tacc-centos7-mvapich2.3-ib](#infiniband-base-mvapich2-images)    | X |   | X |   | X |
| [tacc/tacc-centos7-ppc64le-mvapich2.3-ib](#infiniband-base-mvapich2-images)    |   |   |   | X* | X |
| [tacc/tacc-centos7-mvapich2.3-psm2](#omni-path-base-mvapich2-images)   |   | X |   |   |   |
| [tacc/tacc-centos7-impi19.0.7-common](#common-base-intel-mpi-images)   | X | X |   |   | X |
| [tacc/tacc-ubuntu18](#minimal-base-images)                             | X | X | X |   | X |
| [tacc/tacc-ubuntu18-mvapich2.3-ib](#infiniband-base-mvapich2-images)   | X |   | X |   | X |
| [tacc/tacc-ubuntu18-mvapich2.3-psm2](#omni-path-base-mvapich2-images)  |   | X |   |   |   |
| [tacc/tacc-ubuntu18-impi19.0.7-common](#common-base-intel-mpi-images)  | X | X |   |   | X |

> The singularity version of these containers should be invoked with `singularity run`, and any modifications to `ENTRYPOINT` on the docker side may disrupt function.
>
> \* Must be used with the `mvapich2-gdr` module

## Contents

* [Container Descriptions](#container-descriptions)
* [Running the Containers](#running-the-containers)
  * [Running on Docker](#running-on-docker)
  * [Running on TACC](#running-on-tacc)
* [Building from our Containers](#building-from-our-containers)
* [Performance](#performance)
* [Troubleshooting](#troubleshooting)
* [Known Issues](#known-issues)
* [Frequently Asked Questions](#frequently-asked-questions)

## Container Descriptions

### Minimal base images

* tacc/tacc-centos7
  * [Dockerfile](containers/tacc-centos7) - [Container](https://hub.docker.com/r/tacc/tacc-centos7)
* tacc/tacc-ubuntu18
  * [Dockerfile](containers/tacc-ubuntu18) - [Container](https://hub.docker.com/r/tacc/tacc-ubuntu18)
* tacc/tacc-centos7-ppc64le
  * [Dockerfile](containers/tacc-centos7) - [Container](https://hub.docker.com/r/tacc/tacc-centos7-ppc64le)

> ubuntu18 [does not support](https://packages.ubuntu.com/bionic/libfabric-dev) IB libraries for ppc64le architectures

These are the starting point for our downstream images, and the operating systems we support.
They are meant to be extremely light and only contain the following:

* TACC mount points (for legacy containers)
* [docker-clean](containers/extras/docker-clean) script for cleaning up temporary files between layers
  * Usage: `RUN apt-get install less && docker-clean`
* System GCC toolchains (build-essential)
* Generic `$CFLAGS/$CXXFLAGS` that will work on both _your_ build system and fairly well on ours
  * **x86_64 images** `-O2 -pipe -march=x86-64 -ftree-vectorize -mtune=core-avx2`
  * **ppc64le images** `-mcpu=power8 -O2 -pipe`
* Version recorded in /etc/tacc-[OS]-release for troubleshooting

> The architecture flags in our `$CFLAGS` are not more system specific due to the age of the system compilers.
As we support newer operating systems, those flags will better match the contemporary hardware at TACC

### InfiniBand base MVAPICH2 images

* tacc/tacc-centos7-mvapich2.3-ib
  * [Dockerfile](containers/tacc-centos7-mvapich2.3-ib) - [Container](https://hub.docker.com/r/tacc/tacc-centos7-mvapich2.3-ib)
* tacc/tacc-ubuntu18-mvapich2.3-ib
  * [Dockerfile](containers/tacc-ubuntu18-mvapich2.3-ib) - [Container](https://hub.docker.com/r/tacc/tacc-ubuntu18-mvapich2.3-ib)
* tacc/tacc-centos7-ppc64le-mvapich2.3-ib
  * [Dockerfile](containers/tacc-centos7-mvapich2.3-ib) - [Container](https://hub.docker.com/r/tacc/tacc-centos7-ppc64le-mvapich2.3-ib)
  * Must be used with the `mvapich2-gdr` module on Longhorn

Each image starts from their respective minimal base, and inherits those base features.
The goal of these images is to provide a base MPI development environment that will work on our InfiniBand systems, and will specifically contain the following:

* Version recorded in /etc/tacc-[OS]-mvapich2.3-ib for troubleshooting
* InfiniBand system development libraries
* [MVAPICH2 v2.3](http://mvapich.cse.ohio-state.edu/downloads/)
  * configured with
    ```
    --with-device=ch3 --with-ch3-rank-bits=32 \
    --enable-fortran=yes --enable-cxx=yes \
    --enable-romio --enable-fast=O3
    ```
* [`hellow`](containers/extras/hello.c) - A simple "Hello World" test program on the system path
* [OSU micro benchmarks](http://mvapich.cse.ohio-state.edu/benchmarks/)
  * Installed in /opt/osu-micro-benchmarks
  * Not on system `$PATH`

### Omni-Path base MVAPICH2 images

* tacc/tacc-centos7-mvapich2.3-psm2
  * [Dockerfile](containers/tacc-centos7-mvapich2.3-psm2) - [Container](https://hub.docker.com/r/tacc/tacc-centos7-mvapich2.3-psm2)
* tacc/tacc-ubuntu18-mvapich2.3-psm2
  * [Dockerfile](containers/tacc-ubuntu18-mvapich2.3-psm2) - [Container](https://hub.docker.com/r/tacc/tacc-ubuntu18-mvapich2.3-psm2)

Each image starts from their respective minimal base, and inherits those base features.
The goal of these images is to provide a base MPI development environment that will work on our [Intel Omni-Path](https://www.intel.com/content/www/us/en/high-performance-computing-fabrics/omni-path-driving-exascale-computing.html) (psm2) systems, and will specifically contain the following:

* Version recorded in /etc/tacc-[OS]-mvapich2.3-psm2 for troubleshooting
* InfiniBand system development libraries
* [PSM2 development library](https://github.com/intel/opa-psm2)
* [MVAPICH2 v2.3](http://mvapich.cse.ohio-state.edu/downloads/)
  * configured with
    ```
    --with-device=ch3:psm --with-ch3-rank-bits=32 \
    --enable-fortran=yes --enable-cxx=yes \
    --enable-romio --enable-fast=O3
    ```
* [`hellow`](containers/extras/hello.c) - A simple "Hello World" test program on the system path
* [OSU micro benchmarks](http://mvapich.cse.ohio-state.edu/benchmarks/)
  * Installed in /opt/osu-micro-benchmarks
  * Not on system `$PATH`

Please note that while you can build software in these images, they will **not** run on systems without Omni-Path devices, which probably includes your development system.

### Common base Intel MPI images

* tacc/tacc-centos7-impi19.0.7-common
  * [Dockerfile](containers/tacc-centos7-impi19.0.7-common) - [Container](https://hub.docker.com/r/tacc/tacc-centos7-impi19.0.7-common)
* tacc/tacc-ubuntu18-impi19.0.7-common
  * [Dockerfile](containers/tacc-ubuntu18-impi19.0.7-common) - [Container](https://hub.docker.com/r/tacc/tacc-ubuntu18-impi19.0.7-common)

Each image starts from their respective minimal base, and inherits those base features.
The goal of these images is to provide a base MPI development environment that will work on both our InfiniBand systems and Omni-Path systems, and will specifically contain the following:

* Version recorded in /etc/tacc-[OS]-impi19.0.7-common for troubleshooting
* InfiniBand system development libraries
* [PSM2 development library](https://github.com/intel/opa-psm2)
* [Mellanox OpenFabrics Enterprise Distribution](https://www.mellanox.com/support/mlnx-ofed-public-repository)
* Intel MPI 19.0.7 ([yum repo](https://software.intel.com/content/www/us/en/develop/articles/installing-intel-free-libs-and-python-yum-repo.html), [apt repo](https://software.intel.com/content/www/us/en/develop/articles/installing-intel-free-libs-and-python-apt-repo.html))
* [`hellow`](containers/extras/hello.c) - A simple "Hello World" test program on the system path
* [OSU micro benchmarks](http://mvapich.cse.ohio-state.edu/benchmarks/)
  * Installed in /opt/osu-micro-benchmarks
  * Not on system `$PATH`
* `/entry.sh` - To initialize necessary environment variables for Intel MPI.

> Please note that you will need to use `singularity run` to get the `/entry.sh` invoked properly under Singularity.

## Running the Containers

### Running on Docker

<details><summary>&#x2705; mvapich2.3-ib images</summary>

```
$ docker run -e MV2_SMP_USE_CMA=0 --rm -it tacc/tacc-ubuntu18-mvapich2.3-ib:0.0.5 mpirun -n 2 hellow

Hello world!  I am process-1 on host 7984e55ceba6
Hello world!  I am process-0 on host 7984e55ceba6
```

> Don't forget the to set `MV2_SMP_USE_CMA=0` when running locally

</details>
<details><summary>&#x26D4; mvapich2.3-psm2 images</summary>

This container does **not** run locally.

</details>
<details><summary>&#x2705; impi19.0.7-common images</summary>

```
$ docker run --rm -it tacc/tacc-centos7-impi19.0.7-common:latest mpirun -n 2 hellow

WARNING: release_mt library was used but no multi-ep feature was enabled. Please use release library instead.
Hello world!  I am process-0 on host c05666685143
Hello world!  I am process-1 on host c05666685143
```

> Please ignore this warning

</details>

### Running on TACC
Mult-node jobs need to be invoked with the system `ibrun`.

> Single-node, multi-core applications _can_ be invoked with the container's `mpirun`, but we do not recommend it unless absolutely necessary.

Large MPI applications **must** be run on our high-performance filesystems (not `$WORK`) in the following manner:

1. Pull container once, using a single process <br>
```
[login]$ idev -N 1
[compute]$ singularity pull docker://tacc/tacc-centos7-mvapich2.3-psm2:latest
[compute]$ exit
[login]$
```

2. Move the container to a high-performance filesystem like `$SCRATCH` or maybe `$HOME` <br>
```
[login]$ mv tacc-centos7-mvapich2.3-psm2_latest.sif $SCRATCH/
```
> For large MPI jobs, consider using [`sbcast`](https://slurm.schedmd.com/sbcast.html) to stage the image to `/tmp`

3. Launch MPI application with `singularity run` to load the correct environment<br>
```
[login]$ cd $SCRATCH
[login]$ idev -N 2
[compute]$ ibrun singularity run tacc-centos7-mvapich2.3-psm2_latest.sif hellow
```

#### Running on Stampede 2

<details><summary>impi19.0.7-common images</summary>

```
# Start 2-node compute session
[login]$ idev -N 2 -n 2

# Load the tacc-singularity module
[compute]$ module load tacc-singularity

# Pull your desired image
[compute]$ singularity pull docker://tacc/tacc-centos7-impi19.0.7-common:latest

# Run Hello World
[compute]$ ibrun singularity run tacc-centos7-impi19.0.7-common_latest.sif hellow
TACC:  Starting up job 6848404
TACC:  Starting parallel tasks...
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
WARNING: release_mt library was used but no multi-ep feature was enabled. Please use release library instead.
Hello world!  I am process-1 on host c460-003.stampede2.tacc.utexas.edu
Hello world!  I am process-0 on host c460-002.stampede2.tacc.utexas.edu
TACC:  Shutdown complete. Exiting.
```

> The ERROR messages can be ignored or eliminated by unloading the xalt module.

</details><details><summary>mvapich2.3-psm2 images</summary>

```
# Start 2-node compute session
[login]$ idev -N 2 -n 2

# Load the tacc-singularity module
[compute]$ module load tacc-singularity

# Pull your desired image
[compute]$ singularity pull docker://tacc/tacc-centos7-mvapich2.3-psm2:latest

# Run Hello World
[compute]$ ibrun singularity run tacc-centos7-impi19.0.7-common_latest.sif hellow
TACC:  Starting up job 6848404
TACC:  Starting parallel tasks...
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
WARNING: release_mt library was used but no multi-ep feature was enabled. Please use release library instead.
Hello world!  I am process-0 on host c460-002.stampede2.tacc.utexas.edu
Hello world!  I am process-1 on host c460-003.stampede2.tacc.utexas.edu
TACC:  Shutdown complete. Exiting.
```

> The ERROR messages can be ignored or eliminated by unloading the xalt module.

</details>

#### Running on Frontera

<details><summary>impi19.0.7-common images</summary>

```
# Start 2-node compute session
[login]$ idev -N 2 -n 2

# Load the tacc-singularity module
[compute]$ module load tacc-singularity

# Pull your desired image
[compute]$ singularity pull docker://tacc/tacc-centos7-impi19.0.7-common:latest

# Run Hello World
[compute]$ ibrun singularity run tacc-centos7-impi19.0.7-common_latest.sif hellow
TACC:  Starting up job 2019250
TACC:  Starting parallel tasks...
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
WARNING: release_mt library was used but no multi-ep feature was enabled. Please use release library instead.
Hello world!  I am process-0 on host c191-074.frontera.tacc.utexas.edu
Hello world!  I am process-1 on host c191-081.frontera.tacc.utexas.edu
TACC:  Shutdown complete. Exiting.
```

> The ERROR messages can be ignored or eliminated by unloading the xalt module.

</details><details><summary>mvapich2.3-ib images</summary>

```
# Start 2-node compute session
[login]$ idev -N 2 -n 2

# Load the tacc-singularity module
[compute]$ module load tacc-singularity

# Pull your desired image
[compute]$ singularity pull docker://tacc/tacc-centos7-mvapich2.3-ib:latest

# Run Hello World
[compute]$ ibrun singularity run tacc-centos7-mvapich2.3-ib_latest.sif hellow
TACC:  Starting up job 2019250
TACC:  Starting parallel tasks...
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
ERROR: ld.so: object '/opt/apps/xalt/xalt/lib64/libxalt_init.so' from LD_PRELOAD cannot be preloaded: ignored.
Warning: Process to core binding is enabled and OMP_NUM_THREADS is set to non-zero (1) value
If your program has OpenMP sections, this can cause over-subscription of cores and consequently poor performance
To avoid this, please re-run your application after setting MV2_ENABLE_AFFINITY=0
Use MV2_USE_THREAD_WARNING=0 to suppress this message
Hello world!  I am process-1 on host c191-081.frontera.tacc.utexas.edu
Hello world!  I am process-0 on host c191-074.frontera.tacc.utexas.edu
TACC:  Shutdown complete. Exiting.
```

> The ERROR messages can be ignored or eliminated by unloading the xalt module. The MVAPICH warning can also be suppressed by setting the appropriate environment variables.

</details>

## Building from our Containers

In the [examples](examples) directory, we have a file called `run_julia.py`, which computes the [Julia set](https://en.wikipedia.org/wiki/Julia_set) and was adapted from one of [mpi4py's examples](https://mpi4py.readthedocs.io/en/stable/mpi4py.futures.html#examples).

To build a container to run `run_julia.py` on an InfiniBand system at TACC, a new Docker container needs to be built with the following requirements:

* Starts **FROM** `tacc-[OS]-mvapich2.3-ib`
* Installs necessary python dependencies
  * pip/setuptools
  * mpi4py
* Adds the `run_julia.py` program and updates the permissions

> Do not modify the `ENTRYPOINT` of these containers. Otherwise, the MPI environments may not work correctly.

```
ARG VER=latest
FROM tacc/tacc-ubuntu18-mvapich2.3-ib:${VER}

# Install dependencies
RUN apt-get update \
        && apt-get install -yq --no-install-recommends python3-dev python3-pip \
                python3-setuptools python3-wheel python3-numpy \
        && docker-clean

RUN pip3 install mpi4py \
        && docker-clean

# Add/compile application
ADD run_julia.py /usr/local/bin/run_julia.py

# Make sure permissions are correct for singularity
RUN chmod a+rx /usr/local/bin/run_julia.py
```

You can either manually recreate this, or take advantage of the provided [`Makefile`](examples/Makefile).

```
$ make ORG=[your dockerhub username] julia
```

After the image is done being pushed to dockerhub, you can pull it down to the InfiniBand system of your choice.

```
$ idev -N 2 -n 4
$ module load tacc-singularity
$ singularity pull docker://gzynda/julia:latest
$ ibrun -np 4 singularity run julia_latest.sif run_julia.py
```

<details><summary>Results</summary>

```
$ ibrun -np 4 singularity run julia_latest.sif run_julia.py
TACC: Starting up job 48657
TACC: Starting parallel tasks...
Running COMM
Running COMM
Running COMM
Loaded Executor
c262-169.hikari.tacc.utexas.edu - Julia Set 1600x1200 in 1.92 seconds.
Running COMM
TACC: Shutdown complete. Exiting.

$ ibrun -np 2 singularity run julia_latest.sif run_julia.py
TACC: Starting up job 48657
TACC: Starting parallel tasks...
Running COMM
Loaded Executor
c262-169.hikari.tacc.utexas.edu - Julia Set 1600x1200 in 5.72 seconds.
Running COMM
TACC: Shutdown complete. Exiting.

$ ibrun -np 1 singularity run julia_latest.sif run_julia.py
TACC: Starting up job 48657
TACC: Starting parallel tasks...
Running COMM
Loaded Executor
c262-169.hikari.tacc.utexas.edu - Julia Set 1600x1200 in 5.59 seconds.
TACC: Shutdown complete. Exiting.
```

</details>

> Note: The [MPIPoolExecutor](https://mpi4py.readthedocs.io/en/stable/mpi4py.futures.html#mpipoolexecutor) version of `run_julia.py` does not work

## Performance

There should be no serial performance loss when running from a single node container - assuming the same compilers, libraries, and flags were used.
We did want to measure communication latency to confirm that the correct fabric devices were used and no significant communication performance was lost when programs were compiled against container MPI libraries.

Performance was measured using [osu_latency](http://mvapich.cse.ohio-state.edu/benchmarks/) which exists in all of our tacc-[OS]-mvapich2.3-[fabric] containers at:

 * `/opt/osu-micro-benchmarks/pt2pt/osu_latency`

### Frontera Performance

| Size    | inter-native | inter-centos7 | inter-ubuntu18 | intra-native | intra-centos7 | intra-ubuntu18 |
|---------|--------------|---------------|----------------|--------------|---------------|----------------|
| 0       | 1.15         | 1.16          | 1.16           | 0.42         | 0.22          | 0.21           |
| 1       | 1.14         | 1.2           | 1.19           | 0.4          | 0.22          | 0.21           |
| 2       | 1.14         | 1.19          | 1.19           | 0.4          | 0.22          | 0.22           |
| 4       | 1.14         | 1.19          | 1.19           | 0.4          | 0.22          | 0.22           |
| 8       | 1.13         | 1.19          | 1.19           | 0.4          | 0.22          | 0.21           |
| 16      | 1.13         | 1.23          | 1.22           | 0.41         | 0.23          | 0.22           |
| 32      | 1.15         | 1.23          | 1.22           | 0.42         | 0.25          | 0.22           |
| 64      | 1.2          | 1.23          | 1.22           | 0.42         | 0.27          | 0.24           |
| 128     | 1.22         | 1.28          | 1.27           | 0.51         | 0.3           | 0.27           |
| 256     | 1.7          | 1.69          | 1.69           | 0.59         | 0.32          | 0.31           |
| 512     | 1.53         | 1.77          | 1.78           | 0.79         | 0.4           | 0.4            |
| 1024    | 1.7          | 1.93          | 1.93           | 0.88         | 0.51          | 0.49           |
| 2048    | 2.25         | 2.3           | 2.31           | 1.03         | 0.66          | 0.63           |
| 4096    | 3            | 3.4           | 3.36           | 1.48         | 1             | 0.95           |
| 8192    | 3.98         | 4.56          | 4.64           | 1.94         | 1.7           | 1.87           |
| 16384   | 5.49         | 7.36          | 7.39           | 3.27         | 3.09          | 3.27           |
| 32768   | 9.76         | 9.53          | 9.54           | 4.69         | 4.86          | 5.32           |
| 65536   | 12.74        | 12.43         | 12.44          | 7.81         | 4.06          | 4.64           |
| 131072  | 18.44        | 21.63         | 21.55          | 13.78        | 6.79          | 8.12           |
| 262144  | 33.23        | 32.55         | 32.47          | 25.86        | 12.43         | 15.46          |
| 524288  | 56.55        | 54.73         | 54.28          | 60.53        | 26.54         | 32.34          |
| 1048576 | 100.07       | 96.91         | 97.03          | 117.08       | 81.49         | 80.25          |
| 2097152 | 184.95       | 182.18        | 185.47         | 226.64       | 207.01        | 214.86         |
| 4194304 | 356.57       | 352.09        | 352.55         | 447.13       | 431.6         | 437.29         |

[Full run logs](run_metrics/frontera)

## Troubleshooting

TODO

## Known Issues

* [mpi4py.futures](https://mpi4py.readthedocs.io/en/stable/mpi4py.futures.html) fails on `*psm2` - please submit a pull request if you find a solution
* [MPIPoolExecutor](https://mpi4py.readthedocs.io/en/stable/mpi4py.futures.html#mpipoolexecutor) fails on `*ib` - please submit a pull request if you find a solution
  * Still true with mvapich 2.3.1 in release 0.0.3
* `*psm2` containers cannot run locally
* The `tacc-centos7-ppc64le-mvapich2.3-ib` container is not compatible with Longhorn's default spectrum MPI and only works with the `mvapich2-gdr` module.
* Running with `MV2_ENABLE_AFFINITY=0` in your environment is sometimes required for some code if it fails and you see the following warning <blockquote>
```
Warning: Process to core binding is enabled and OMP_NUM_THREADS is set to non-zero (1) value
If your program has OpenMP sections, this can cause over-subscription of cores and consequently poor performance
To avoid this, please re-run your application after setting MV2_ENABLE_AFFINITY=0
Use MV2_USE_THREAD_WARNING=0 to suppress this message
```
</blockquote>

## Frequently asked questions

<details><summary>What happens if I run a `*ib` container on an OmniPath system like Stampede2?</summary>

Multi-node
```
gzynda@Sc460-031[osu-bench]$ ibrun singularity run tacc-centos7-mvapich2.3-ib_0.0.2.sif hellow
TACC:  Starting up job 4784577
TACC:  Starting parallel tasks...
[c460-032.stampede2.tacc.utexas.edu:mpi_rank_1][error_sighandler] Caught error: Segmentation fault (signal 11)
[c460-031.stampede2.tacc.utexas.edu:mpi_rank_0][error_sighandler] Caught error: Segmentation fault (signal 11)
TACC:  MPI job exited with code: 139
TACC:  Shutdown complete. Exiting.
```

Single-node
```
gzynda@Sc460-031[osu-bench]$ singularity run tacc-centos7-mvapich2.3-ib_0.0.2.sif bash -c 'mpirun -n 2 -launcher fork hellow'
Hello world!  I am process-0 on host c460-031.stampede2.tacc.utexas.edu
Hello world!  I am process-1 on host c460-031.stampede2.tacc.utexas.edu
```

</details>
<details><summary>What happens if I run a `*psm2` container on an InfiniBand system?</summary>


Multi-node
```
c262-169.hikari(44)$ ibrun singularity run tacc-centos7-mvapich2.3-psm2_0.0.2.sif hellow
TACC: Starting up job 48655
TACC: Starting parallel tasks...
psm2_init failed with error: PSM Unresolved internal error
psm2_init failed with error: PSM Unresolved internal error
[cli_1]: aborting job:
Fatal error in MPI_Init: Internal MPI error!, error stack:
MPIR_Init_thread(490):
MPID_Init(395).......: channel initialization failed
(unknown)(): Internal MPI error!
[cli_0]: aborting job:
Fatal error in MPI_Init: Internal MPI error!, error stack:
MPIR_Init_thread(490):
MPID_Init(395).......: channel initialization failed
(unknown)(): Internal MPI error!
TACC: MPI job exited with code: 16

TACC: Shutdown complete. Exiting.
```

Single-node
```
c262-169.hikari(51)$ singularity run tacc-centos7-mvapich2.3-psm2_0.0.2.sif bash -c 'MV2_USE_CMA=0; mpirun -n 2 -launcher fork hellow'
psm2_init failed with error: PSM Unresolved internal error
psm2_init failed with error: PSM Unresolved internal error
[cli_0]: aborting job:
Fatal error in MPI_Init: Internal MPI error!, error stack:
MPIR_Init_thread(490):
MPID_Init(395).......: channel initialization failed
(unknown)(): Internal MPI error!
[cli_1]: aborting job:
Fatal error in MPI_Init: Internal MPI error!, error stack:
MPIR_Init_thread(490):
MPID_Init(395).......: channel initialization failed
(unknown)(): Internal MPI error!
```

</details>

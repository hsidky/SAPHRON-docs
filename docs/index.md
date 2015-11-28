# SAPHRON

Thanks for visiting the official documentation for SAPHRON 
(**S**tatistical **A**pplied **PH**ysics through **R**andomized **O**n-the-fly **N**umerics), 
a lightweight C++11-based Monte Carlo molecular simulation package. **SAPHRON is currently 
under development and may lack some basic features**. As a result, the documentation may also 
not be up-to-date. If you are feeling up to it, then please, dive right in!

## Overview

The primary goal of SAPHRON is to provide a simple, extensible and *user friendly* API and 
interface for carrying out Monte Carlo simulations. It is both a library and a standalone 
application, with an emphasis on density of states (DOS) based sampling. A few key features 
are listed below.

* Lattice simulations support (Ising, Lebwohl-Lasher, etc...)
* Off-lattice atomistic simulations.
* OpenMP parallelization.
* DOS sampling on various order parameters.
* Native JSON support for input files.
* Extensive suite of unit tests.
* Clear user error reporting 

One point of frustration for us using various molecular simulation packages was the lack 
of clear error reporting. We found that packages often dumped meaningless errors, sometimes
even contradictory messages in response to semantic errors in input files. In a few cases, 
applications seg-faulted on syntax errors. **That is NOT okay!**

SAPHRON was designed to provide thoughtful, meaningful and helpful error messages to the user
as a way of guiding them towards crafting a valid input file. As SAPHRON develops, this feature 
will be built out to provide general notices and hints as well. Our hope is that SAPHRON will
be easy enough to use so that with a little bit of practice, it can be operated by a high-school 
student. If you feel any aspect of SAPHRON's usability can be improved, please let us know 
immediately by [opening a ticket](https://github.com/hsidky/SAPHRON/issues/new).

## Dependencies

SAPHRON requires GCC 4.9.x or Clang 3.4+ **at minimum**. This is not an issue for most 
systems, but if you are running Ubuntu 14.04 LTS, please read 
[this](http://askubuntu.com/questions/466651/how-do-i-use-the-latest-gcc-4-9-on-ubuntu-14-04).
SAPHRON also depends on [Armadillo](http://arma.sourceforge.net/) to provide an interface to 
low level linear algebra routines. It links directly to any linear algebra provider 
available on the system. BLAS and LAPACK are the default providers, but SAPHRON 
also supports the Intel MKL. Typically, no configuration on the user's end is required. 
All dependencies are automatically detected by the CMake build system. 

If you just want to get things going, it is easy enough to install the required libraries.
For a Debian-based distro simply type:

```bash
$ sudo apt-get install libblas3 liblapack3 libarmadillo-dev
```

This will install the BLAS/LAPACK shared libraries and Armadillo headers. As mentioned, 
if you have access to Intel's MKL you don't have to do anything else and SAPHRON should 
find it. If the MKL is available via an environment module, it is recommended that it be 
loaded since it offers superior performance. Just make sure that wherever the binary is
deployed also has access.

**SAPHRON will NOT compile until you perform the next step!**
By default, Armadillo assumes you want to link against its shared library. SAPHRON does not do this, 
but instead links directly to BLAS and LAPACK or MKL as mentioned above. To fix this, use your editor 
of choice (here we use vim) to modify the Armadillo config header:

```bash
$ vim /usr/include/armadillo_bits/config.hpp
```

Just comment out the line that says `#define ARMA_USE_WRAPPER` and you're good to go!

#### OpenMP 

SAPHRON uses OpenMP to parallelize energy evaluations, neighbor list generation and other 
calculations. It relies on compilers that support the `-fopenmp` flag. OpenMP support is enabled
by default. If you do not want to run SAPHRON multi-threaded, see the advanced configuration 
section below.

## Installation

The first step is to clone the repository locally

```bash	
$ git clone https://github.com/hsidky/SAPHRON.git
```

SAPHRON uses a CMake build system. It is recommended that you perform an out of source build. 
To do so, enter the following commands

```bash	
$ cd SAPHRON
$ mkdir build && cd build
$ cmake .. 
$ make
$ sudo make install
```

This will install a `saphron` binary into `/usr/local/bin` and a static library in `/usr/local/lib/libsaphron.a`.
Now that SAPHRON is installed, hop on over to the [Getting Started](getting-started.md) page to
set up and run your first simulation!

#### Advanced installation

If you do not have root access or would like to build SAPHRON for a cluster, it is possible 
to manually specify the location of the Armadillo headers by specifying the ARMA_PATHS variable
in CMake

```bash
$ cmake -DARMA_PATHS=/path/to/arma ..
```

OpenMP support can also be disabled if desired

```bash
$ cmake -DSAPHRON_OPENMP=false ..
```

By default, SAPHRON is built optimized without debug info, to change the build type to 
include debug info for debugging or profiling

```bash
$ cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
```

Of course, any of the default CMake variables are available and can be combined as desired.

## Testing 

SAPHRON has a comprehensive, but not *exhaustive* suite of unit tests. They range from 
simple behavioral verification, to full fledged simulations verified against NIST data. 
The [Google C++ Testing Framework](https://code.google.com/p/googletest/) is used for 
unit testing. It is automatically downloaded as part of the build process. It is a good idea 
if deploying SAPHRON for the first time to run the unit tests. This can be done in the build 
directory by typing 

```bash	
$ make test
```

It is also a great way to make sure that any customizations you make do not introduce new 
bugs into the code and to get a feel for the SAPHRON API.

## Acknowledgements 

**SAPHRON** utilizes the following packages: 

Armadillo (http://arma.sourceforge.net/)

JSONCpp (https://github.com/open-source-parsers/jsoncpp)

Sitmo Random Number Generator (http://www.sitmo.com/article/multi-threaded-random-number-generation-in-c11/)

**SAPHRON** would also not be possible without the support and expertise of [Eliseo Marin-Rimoldi](https://github.com/emarinri), 
developer of [Cassandra](http://cassandra.nd.edu/).

## Contributing

Feel free to fork this project on [GitHub](https://github.com/hsidky/SAPHRON). Any pull-requests,
feature requests or other form of contributions are welcome. We especially encourage users to 
report bugs or request features; you can guide the development of this project!


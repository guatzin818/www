Here is the guide for the build of CLI version.

For the build of python-package and R-package, please refer to [python-package build](https://github.com/Microsoft/LightGBM/tree/master/python-package) and [R-package build](https://github.com/Microsoft/LightGBM/tree/master/R-package) respectively.

## Windows

LightGBM can use Visual Studio, MSBuild with CMake or MinGW to build in Windows.

### Visual Studio (Or MSBuild)

1. Install [git for windows](https://git-scm.com/download/win), [cmake](https://cmake.org/) and [MS Build](https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2017) (MSbuild is not needed if *Visual Studio* is installed).

2. Run following command:

```
git clone --recursive https://github.com/Microsoft/LightGBM
cd LightGBM
mkdir build
cd build
cmake -DCMAKE_GENERATOR_PLATFORM=x64 ..
cmake --build . --target ALL_BUILD --config Release
```

The exe and DLL will be in ```LightGBM/Release``` folder.

### MinGW64

1. Install [git for windows](https://git-scm.com/download/win), [cmake](https://cmake.org/) and MinGW64.
2. Run following command:
```
git clone --recursive https://github.com/Microsoft/LightGBM
cd LightGBM
mkdir build
cd build
cmake -G "MinGW Makefiles" ..
mingw32-make.exe -j4
```

The exe and DLL will be in ```LightGBM/``` folder.

## Linux

LightGBM use ***cmake*** to build. Run following: 

```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake .. 
make -j4
```

Note: glibc >= 2.14 requirement.

## OSX

LightGBM depends on OpenMP for compiling, which isn't supported by Apple Clang.

Please install gcc/g++ by using following command:

```
brew install cmake
brew install gcc --without-multilib
```

Then install LightGBM:
```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake .. 
make -j4 
```

## Build MPI Version

The default build version of LightGBM is based on socket. LightGBM also support [MPI](https://en.wikipedia.org/wiki/Message_Passing_Interface). MPI is a high performance communication approach with [RDMA](https://en.wikipedia.org/wiki/Remote_direct_memory_access) supported. 

If you need to run a parallel learning application with high performance communication, you can build the LightGBM with MPI support.

### Windows

1. You need to install [MSMPI](https://www.microsoft.com/en-us/download/details.aspx?id=49926) first. Both ```msmpisdk.msi``` and ```MSMpiSetup.exe``` are needed.

3. Install [git for windows](https://git-scm.com/download/win), [cmake](https://cmake.org/) and [MS Build](https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2017) (Not need the MSbuild if you already install *Visual Studio*).

3. Run following command:

```
git clone --recursive https://github.com/Microsoft/LightGBM
cd LightGBM
mkdir build
cd build
cmake -DCMAKE_GENERATOR_PLATFORM=x64 -DUSE_MPI=ON ..
cmake --build . --target ALL_BUILD --config Release
```

Note: Build mpi version by MinGW is not supported due to the miss of MPI Library in MinGW.

### Linux

You need to install [OpenMPI](https://www.open-mpi.org/) first.

Then run:

```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake -DUSE_MPI=ON .. 
make -j4 
```

Note: glibc >= 2.14 requirement.

### OSX

Please install gcc and openmpi first
```
brew install openmpi 
brew install cmake
brew install gcc --without-multilib
```

Then run:
```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake -DUSE_MPI=ON .. 
make -j4 
```

## With GPU support

### Linux

The following dependencies should be installed before compilation:

- OpenCL 1.2 headers and libraries, which is usually provided by GPU manufacture.  
  The generic OpenCL ICD packages (for example, Debian package
  `ocl-icd-libopencl1` and `ocl-icd-opencl-dev`) can also be used.

- libboost 1.56 or later (1.61 or later recommended). We use Boost.Compute as
  the interface to GPU, which is part of the Boost library since version 1.61.
  However, since we include the source code of Boost.Compute as a submodule, we
  only require the host has Boost 1.56 or later installed. We also use
  Boost.Align for memory allocation. Boost.Compute requires Boost.System
  and Boost.Filesystem to store offline kernel cache. The following Debian 
  packages should provide necessary Boost libraries: 
  `libboost-dev, libboost-system-dev, libboost-filesystem-dev`.

- CMake 3.2 or later

To build LightGBM-GPU, use the following procedure:

First clone this repository:

```
git clone --recursive https://github.com/Microsoft/LightGBM
```

Then run `cmake` and `make`:

```
cd LightGBM
mkdir build ; cd build
cmake -DUSE_GPU=1 .. 
make -j4 
```

### Windows

If you use MinGW in windows, the build procedure are similar to the build in Linux. Visit [here](https://github.com/Microsoft/LightGBM/blob/master/docs/GPU-Windows.md) to get more details.


Following procedure is for the MSVC(Microsoft Visual C++) build. 

1. Install OpenCL for windows. The installation depend on the brand(Nvidia, AMD, Intel) of your GPU card. 

    * For running on Intel, get Intel SDK for OpenCL: https://software.intel.com/en-us/articles/opencl-drivers
    * For running on AMD, get AMD APP SDK: http://developer.amd.com/tools-and-sdks/opencl-zone/amd-accelerated-parallel-processing-app-sdk/
    * For running on NVIDIA, get CUDA Toolkit: https://developer.nvidia.com/cuda-downloads

2. Install Boost Binary: https://sourceforge.net/projects/boost/files/boost-binaries/1.64.0/ .
   (Note: match your Visual C++ version.  Visual studio 2013 -> msvc-12.0-64.exe, 2015-> msvc-14.0-64.exe, 2017 -> msvc-14.1-64.exe). 
3. run following in the command line:
```
Set BOOST_ROOT=C:\local\boost_1_64_0\
Set BOOST_LIBRARYDIR=C:\local\boost_1_64_0\lib64-msvc-14.0
git clone --recursive https://github.com/Microsoft/LightGBM
cd LightGBM
mkdir build
cd build
cmake -DCMAKE_GENERATOR_PLATFORM=x64 -DUSE_GPU=1 ..
cmake --build . --target ALL_BUILD --config Release
```
Note: `C:\local\boost_1_64_0\` and `C:\local\boost_1_64_0\lib64-msvc-14.0` is the location of your Boost binaries. You also can set them to the environment variable to avoid `set ...` when build. 


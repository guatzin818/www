LightGBM is implemented by standard C++ 11. It doesn't need additional packages to build.

## Windows

### Visual Studio
LightGBM use Visual Studio (2013 or higher) to build in Windows.

1. Clone or download latest source code.
2. Open ```./windows/LightGBM.sln``` by Visual Studio.
3. Set configuration to ```Release``` and ```x64``` (set to ```DLL``` and ```x64``` for building library(.dll file)).
4. Press ```Ctrl+Shift+B``` to build.
5. The exe file is in ```./windows/x64/Release/``` after built (library(.dll) file is in ```./windows/x64/DLL/```).

### cmake with Visual Studio


```
git clone --recursive https://github.com/Microsoft/LightGBM
cd LightGBM
mkdir build
cd build
cmake -DCMAKE_GENERATOR_PLATFORM=x64 ..
cmake --build . --target ALL_BUILD --config Release
```

### cmake with MinGW
```
git clone --recursive https://github.com/Microsoft/LightGBM
cd LightGBM
mkdir build
cd build
cmake -G "MinGW Makefiles" ..
mingw32-make.exe -j
```

## Linux

LightGBM use ***cmake*** to build in Unix. Run following: 

```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake .. 
make -j 
```

Note: glibc >= 2.14 requirement.

## OSX

LightGBM depends on OpenMP for compiling, which isn't supported by Apple Clang.

Please use gcc/g++ instead. 

Run following: 

```
brew install cmake
brew install gcc --without-multilib
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake -DCMAKE_CXX_COMPILER=g++-6 -DCMAKE_C_COMPILER=gcc-6 .. 
make -j 
```

## Build MPI Version

The default build version of LightGBM is based on socket. LightGBM also support [MPI](https://en.wikipedia.org/wiki/Message_Passing_Interface). MPI is a high performance communication approach with [RDMA](https://en.wikipedia.org/wiki/Remote_direct_memory_access) supported. 

If you need to run a parallel learning application with high performance communication, you can build the LightGBM with MPI support.

### Windows

You need to install [MSMPI](https://www.microsoft.com/en-us/download/details.aspx?id=49926) first. Both ```msmpisdk.msi``` and ```MSMpiSetup.exe``` are needed.

Then:

1. Clone or download latest source code.
2. Open ```./windows/LightGBM.sln``` by Visual Studio.
3. Set configuration to ```Release_mpi``` and ```x64``` .
4. Press ```Ctrl+Shift+B``` to build.
5. The exe file is in ```./windows/x64/Release_mpi/``` after built.

Note: Doesn't support build mpi version by MinGW due to the miss of MPI Library in MinGW.
### Linux

You need to install [OpenMPI](https://www.open-mpi.org/) first.

Then run following:

```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake -DUSE_MPI=ON .. 
make -j 
```

### OSX


Run following: 

```
brew install openmpi 
brew install cmake
brew install gcc --without-multilib
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake -DCMAKE_CXX_COMPILER=g++-6 -DCMAKE_C_COMPILER=gcc-6 -DUSE_MPI=ON .. 
make -j 
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
make -j 
```

### Windows

If you use MinGW in windows, the build procedure are similar to the build in Linux. Visit [here](https://github.com/Microsoft/LightGBM/blob/master/docs/GPU-Windows.md) to get more details.


Following procedure is for the MSVC build. 

1. Install OpenCL for windows. The installation depend on the brand(Nvidia, AMD, Intel) of your GPU card. 
2. Install Boost Binary: https://sourceforge.net/projects/boost/files/boost-binaries/1.64.0/ .
   (Note: match your VC version.  Visual studio 2013 -> msvc-12.0-64.exe, 2015-> msvc-14.0-64.exe, 2017 -> msvc-14.1-64.exe). 
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


LightGBM is implemented by standard C++ 11. It doesn't need additional packages to build.

## Windows

LightGBM use Visual Studio (2013 or higher) to build in Windows.

1. Clone or download latest source code.
2. Open ```./windows/LightGBM.sln``` by Visual Studio.
3. Set configuration to ```Release``` and ```x64``` (set to ```DLL``` for building library) .
4. Press ```Ctrl+Shift+B``` to build.
5. The exe file is in ```./windows/x64/Release/``` after built (library(.dll) file is in ```./windows/x64/DLL/```).

Use MinGW:
```
git clone --recursive https://github.com/Microsoft/LightGBM
cd LightGBM
cmake -G "MinGW Makefiles" .
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

## OSX

LightGBM depends on OpenMP for compiling, which isn't supported by Apple Clang.

Please use gcc/g++ instead. 

Run following: 

```
brew install cmake
brew install gcc --without-multilib
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake -DCMAKE_CXX_COMPILER=g++-6 .. 
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
cmake -DCMAKE_CXX_COMPILER=g++-6 -DUSE_MPI=ON .. 
make -j 
```


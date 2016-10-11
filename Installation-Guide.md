LightGBM is implemented by standard C++ 11. It doesn't need additional packages to build.

## Windows

LightGBM use Visual Studio (2013 or higher) to build in Windows.

1. Clone or download latest source code.
2. Open ```./windows/LightGBM.sln``` by Visual Studio.
3. Set configuration to ```Release``` and ```x64``` .
4. Press ```Ctrl+Shift+B``` to build.
5. The exe file is in ```./windows/x64/Release/``` after built.

## Unix

LightGBM use ***cmake*** to build in unix. Run following: 

```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake .. 
make -j 
```

## Build MPI Version

The default build version of LightGBM is based on socket. LightGBM also support [MPI](https://en.wikipedia.org/wiki/Message_Passing_Interface). MPI is a high performance communication approach with [RDMA](https://en.wikipedia.org/wiki/Remote_direct_memory_access) supported. 

If you need to run parallel learning application with high performance communication, you can build the LightGBM with MPI support.

### Windows

You need to install [MSMPI](https://www.microsoft.com/en-us/download/details.aspx?id=49926) first. Both ```msmpisdk.msi``` and ```MSMpiSetup.exe``` are needed.

Then:

1. Clone or download latest source code.
2. Open ```./windows/LightGBM.sln``` by Visual Studio.
3. Set configuration to ```Release_mpi``` and ```x64``` .
4. Press ```Ctrl+Shift+B``` to build.
5. The exe file is in ```./windows/x64/Release_mpi/``` after built.

### Unix

You need to install [OpenMPI](https://www.open-mpi.org/) first.

Then run following:

```
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM
mkdir build ; cd build
cmake -DUSE_MPI=ON .. 
make -j 
```


This is a guide for parallel learning of LightGBM.

Follow the [Quick Start](https://github.com/Microsoft/LightGBM/wiki/Quick-Start) to know how to use LightGBM first.

## Choose appropriate parallel algorithm

LightGBM provides 2 parallel learning algorithms now. 

|Parallel algorithm| How to use |
|----------------|---------------------|
|Data parallel   | tree_learner=data   |
|Feature parallel| tree_learner=feature|
|Voting parallel| tree_learner=voting|

These algorithms are suited for different scenarios, which is listed in following table:

|                     | #data is small| #data is large| 
|---------------------|------------------|-----------------|
|**#feature is small**| Feature Parallel | Data Parallel   |
|**#feature is large**| Feature Parallel | Voting Parallel |


More details about these parallel algorithms can be found in [optimization in parallel learning](https://github.com/Microsoft/LightGBM/wiki/Features#optimization-in-parallel-learning).

## Build parallel version
Default build version support parallel learning based on the socket.

If you need to build parallel version with MPI support, please refer to [this](https://github.com/Microsoft/LightGBM/wiki/Installation-Guide#build-mpi-version).

## Preparation

### socket version

It needs to collect IP of all machines that want to run parallel learning in and allocate one TCP port (assume 12345 here) for all machines, and change firewall rules to allow income of this port (12345). Then write these IP and ports in one file (assume mlist.txt), like following:

```
machine1_ip 12345
machine2_ip 12345
```

### MPI version

It needs to collect IP (or hostname) of all machines that want to run parallel learning in. Then write these IP in one file (assume mlist.txt) like following:

```
machine1_ip
machine2_ip
```

Note: For Windows users, need to start "smpd" to start MPI service. More details can be found in [here](https://blogs.technet.microsoft.com/windowshpc/2015/02/02/how-to-compile-and-run-a-simple-ms-mpi-program/).

## Run parallel learning

### Socket version

1. Edit following parameters in config file:

    ```tree_learner=your_parallel_algorithm```, edit "your_parallel_algorithm"(e.g. feature/data) here.

    ```num_machines=your_num_machines```, edit "your_num_machines"(e.g. 4) here.

    ```machine_list_file=mlist.txt```,  mlist.txt is created in Preparation.
    
    ```local_listen_port=12345```,  12345 is allocated in Preparation.

    
2. Copy data file, executable file, config file and "mlist.txt" to all machines.
3. Run following command on all machines, need to change "your_config_file" to real config file.

   For windows:```lightgbm.exe config=your_config_file```
  
   For linux:```./lightgbm config=your_config_file```

### MPI version

1. Edit following parameters in config file:

    ```tree_learner=your_parallel_algorithm```, edit "your_parallel_algorithm"(e.g. feature/data) here.

    ```num_machines=your_num_machines```, edit "your_num_machines"(e.g. 4) here.
    
2. Copy data file, executable file, config file and "mlist.txt" to all machines. Note: MPI need run in **same path on all machines**.
3. Run following command on one machine (not need to run on all machines), need to change "your_config_file" to real config file.

   For windows:```mpiexec.exe /machinefile mlist.txt lightgbm.exe config=your_config_file```
  
   For linux:```mpiexec --machinefile mlist.txt ./lightgbm config=your_config_file```

### Example

* [A simple parallel example](https://github.com/Microsoft/lightgbm/tree/master/examples/parallel_learning).


offloading to GPU
===================================

.. questions::

   - What functions should you use for RMA?

.. objectives::

   - Learn how to create memory windows.
   - Learn how to access remote memory windows.

target
------------------
Since version 4.0 , OpenMP supports heterogeneous systems
OpenMP uses target construct to offload execution from the host to the target device(s), and hence the directive name.

Such a host/device model is generally used by OpenMP for target:
- Generally one single host: CPU
   - Generally one single host: CPU
   - one or multiple target devices of the same kind: CPU, GPU, FPGA, ...
- Generally one single host: CPU
 - Generally one single host: CPU
  - Generally one single host: CPU


gpu-derectives +
Device Execution Model
Device: An implementation-defined logical execution unit.

The general model used by OpenMP for target is a single host and one or more target devices.
The host is where the  thread begins execution

Can have a single host and one or more target devices 
Host and Device have separate data environment (except with managed memory or unified shared memory).

The execution model is host-centric
Host creates/destroys data environment on the device(s)
Host maps data to the device data environment.
Host then offloads accelerator regions to the device for execution
Host updates the data between the host and the device.
Host destroys data environment on device.


To begin, 
the host creates the data environments on the device(s). 
The host then maps data to the device data environment, which is data movement to the device. 
The host then offloads OpenMP target regions to the target device; that is, the code is executed on the device. 
After execution, the host updates the data between the host and the device, which is transferring data from the device to the host. 
The host then destroys the data environment on the device.



Creating Parallelism on the Target Device
The target construct transfers the control flow to the target device
▪ Transfer of control is sequential and synchronous

OpenMP separates offload and parallelism
▪Programmers need to explicitly create parallel regions on the target device
In theory, this can be combined with any OpenMP construct
▪
In practice, there is only a useful subset of OpenMP features for a target device such 
as a GPU, e.g., no I/O, limited use of base language features.


teams distribute Construct Bristol
The teams construct
–
Similar to the parallel construct
It starts a league of teams
Each team in the league starts with one initial thread – i.e. a team of one thread
Threads in different teams cannot synchronize with each other
The construct must be “perfectly” nested in a target construct
• The distribute construct

Similar to the for construct
Loop iterations are workshared across the initial threads in a league
No implicit barrier at the end of the construct
dist_schedule(kind[, chunk_size])
– If specified, scheduling kind must be static
– Chunks are distributed in round-robin fashion in chunks of size chunk_size
– If no chunk size specified, chunks are of (almost) equal size; each team receives at least one chunk


2020ecp
The target construct offloads the enclosed code to the accelerator: single thread on a device (GPU)
The teams construct creates a league of teams: one thread each, concurrent (not parallel) execution (on SMs)
The parallel construct creates a new team of threads: parallel execution (by hardware threads)
The simd construct indicates SIMD execution is allowed



gpu-derectives
Structured
Has explicit start and end points
Within a single function
Memory exist within the data region
Unstructured
Can have multiple start and end points
Can branch across multiple functions
Memory exists until explicitly deallocated

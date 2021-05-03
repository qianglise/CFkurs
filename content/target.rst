offloading to GPU
===================================

.. objectives::

   - Learn how to create .
   - Learn how to access 

.. prereq::

   1. You need
   2. Basic understanding of Git.
   3. You need a 


.. _host_device_model:
host-device model
------------------
Since version 4.0 , OpenMP supports heterogeneous systems. OpenMP uses target  construct to offload execution from the host to the target device(s), and hence the directive name. In addition, the associated data needs to be transferred to the device(s) as well.  By default all variables within the lexical scope of the construct are copied to and from the device and this will be discussed more in the next chapter. Once transferred, the target device owns the data and  accesses by the host during the execution of the target region is forbidden.

.. note::
  1. device is host
  2. data already exists on the device from a previous execution


A host/device model is generally used by OpenMP for offloading:
 - Normally there is only one single host: e.g. CPU
 - but one or multiple target devices of the same kind: e.g. CPU, GPU, FPGA, ...
 - unless with unified shared memory, the host and device have separate memory address spac






.. note::
	Device: An implementation-defined logical execution unit.


The host is where the  thread begins execution


.. _device_execution_model:
device execution model
------------------
The execution on the device is host-centric and 
1.the host creates the data environments on the device(s) 
2.the host maps data to the device data environment, which is data movement to the device 
3.the host offloads OpenMP target regions to the target device to be  executed
4.the host transfers data from the device to the host 
5.The host destroys the data environment on the device.


code::
    >>> binary_blob = b"Hello\x00Hello\x00"
    >>> dset.attrs["attribute_name"] = np.void(binary_blob)
    >>> out = dset.attrs["attribute_name"]
    >>> binary_blob = out.tobytes()


Target construct
------------------
The target construct consists of a target directive and an execution region. It is used to transfer both the control flow  from the host to the device and the data between the host and device.

Syntax
 - (C/C++)

#pragma omp target [clause[[,] clause],...]
structured-block
 - (Fortran)
!$omp target [clause[[,] clause],...]
structured-block
!$omp end target
 - Clauses
  -- device(scalar-integer-expression)
  -- map([{alloc | to | from | tofrom}:] list)
  -- if(scalar-expr)


Target construct Code Example
https://github.com/OpenMP/Examples/blob/v4.5.0/sources/Example_target.1.c
https://github.com/OpenMP/Examples/blob/v4.5.0/sources/Example_target.1.f90

To execute code on a target device
 ``target``
 ``declear target``

Syntax
The ``declear target`` directive specifies that variables, functions (C, C++ and Fortran), and subroutines (Fortran) are mapped to a device. The syntax  is as follows:  
    - C/C++: `#pragma target [clauses]`
    - Fortran: `!$OMP target [clauses]`
Commonly used clauses 
#pragma omp target [clause[[,]clause]...]
structured-block
if(scalar-expression)
– If the scalar-expression evaluates to false then the target region is executed by the
host device in the host data environment.
device(integer-expression)
– The value of the integer-expression selects the device when a device other than
the default device is desired.
private(list) firstprivate(list)
– creates variables with the same name as those in the list on the device. In the
case of firstprivate, the value of the variable on the host is copied into the private
variable created on the device.
map(map-type: list)
– map-type may be to, from, tofrom, or alloc. The clause defines how the variables
in list are moved between the host and the device. (Lots more on this later)...
nowait
– The target task is deferred which means the host can run code in parallel to the
target region on the device.




## Pure and impure functions





Creating Parallelism on the Target Device
------------------
OpenMP separates offload and parallelism
 -The target construct transfers the control flow to the device in a sequential and synchronous manner
 -Programmers need to explicitly create parallel regions on the target device
 -In practice, there is only a useful subset of OpenMP features for a target device such 
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



The creation of ``MPI_Win`` objects is a collective operation: each process in
the communicator will reserve the specified memory for remote memory accesses.

.. signature:: |term-MPI_Win_allocate|

   Use this function to *allocate* memory and *create* a window object out of it.

   .. code-block:: c

      int MPI_Win_allocate(MPI_Aint size,
                           int disp_unit,
                           MPI_Info info,
                           MPI_Comm comm,
                           void *baseptr,
                           MPI_Win *win)

.. note::

   - The memory window is usually a single array: the size of the window object
     then coincides with the size of the array.  If the base type of the array
     is a simple type, then the displacement unit is the size of that type,
     *e.g.* ``double`` and ``sizeof(double)``.  You should use a displacement
     unit of 1 otherwise.


.. challenge:: Window creation

   Let's look again at the initial example in the type-along. There we published
   an already allocated buffer as memory window. Use the examples above to
   figure out how to switch to using |term-MPI_Win_allocate|


composite directive
------------------
Each compiler supports different levels of parallelism
– LLVM/clang 10, AMD, CCE9, IBM, PGI: teams, parallel
– (Planned) LLVM/Clang 11, Intel: teams, parallel, simd
– CCE8: teams, parallel or teams, simd
•
Caveats:
– Real applications will have algorithms that are structured such that they can’t
immediately use the combined construct.
– It may also make collapse hard to do
– Performance can be achieved without combined directives, but likely won’t be portable

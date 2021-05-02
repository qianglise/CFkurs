offloading to GPU
===================================

.. questions::

   - What functions should you use for RMA?

.. objectives::

   - Learn how to create memory windows.
   - Learn how to access remote memory windows.

.. prereq::

   1. You need `pytest <http://doc.pytest.org>`__ (as part of Anaconda or Miniconda or Virtual Environment).
   2. Basic understanding of Git.
   3. You need a `GitHub <https://github.com>`__ or a `Gitlab <https://gitlab.com/>`__ account.



host-device model
------------------
Since version 4.0 , OpenMP supports heterogeneous systems
OpenMP uses ''target construct'' to offload execution from the host to the target device(s), and hence the directive name.
In addition, the associated data needs to be transferred to the device(s) as well and this will be discussed in the next chapter. Once transferred, the target device owns the data and  accesses by the host during the execution of the target region is forbidden.

Such a host/device model is generally used by OpenMP for offloading:
 - Normally there is only one single host: CPU
 - but one or multiple target devices of the same kind: CPU, GPU, FPGA, ...
 - unless with unified shared memory, the host and device have separate memory address space






gpu-derectives +


.. note::
	Device: An implementation-defined logical execution unit.


The host is where the  thread begins execution


device execution model
------------------
The execution model is host-centric and
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


Target construct
------------------
The target construct is used to transfer
 - the control flow  from the host to the device
 - data between the host and device ``DEVICE`` 
 - data between ``HOST``  and  ``DEVICE`` 

To execute code on a target device
 ``target``
 ``declear target``
Syntax
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




``````{challenge} 1. Function that receives a number and returns a number
````{tabs}
   ```{code-tab} py
   def factorial(n):
       """
       Computes the factorial of n.
       """
       if n < 0:
           raise ValueError('received negative input')
       result = 1
       for i in range(1, n + 1):
           result *= i
       return result
   ```

   ```{code-tab} c++
   /* Computes the factorial of n recursively. */
   constexpr unsigned int factorial(unsigned int n) {
      return (n <= 1) ? 1 : (n * factorial(n - 1));
   }
   ```

   ```{code-tab} r R
   #' Computes the factorial of n
   #'
   #' @param n The number to compute the factorial of.
   #' @return The factorial of n
   factorial <- function(n) {
     if (n < 0)
       stop('received negative input')
     if (n == 0)
       return(1)

     result <- 1
     for (i in 1:n)
       result <- result * i
     result
   }
   ```

   ```{code-tab} julia
   """
       factorial(n::Int)

   Compute the factorial of n.
   """
   function factorial(n::Int)
       if n < 0
           throw(DomainError("n must be non-negative"))
       end
       result = 1
       for i in 1:n
           result *= i
       end
       return result
   end
   ```

   ```{code-tab} fortran
   module factorial_mod
   contains
      ! computes the factorial of n
      integer function factorial(n)
         implicit none
         integer, intent(in) :: n
         integer r
         integer i
         if(n < 0) then
            write(*,*) 'Received negative input'
            stop
         end if
         r = 1
         do i = 1,n
            r = r*i
         end do
         factorial=r
      end function factorial
   end module factorial_mod
   ```
````
``````




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

data environment
===================================

.. objectives::

   - Understand what are data movement
   - Understand what are structured and unstructured data clauses
   - Learn how to move data explicitly
   - Learn how to compile and run OpenACC code with data movement directives

.. prereq::

   1. You need ...
   2. Basic understanding of ...

# data environment

## data mapping

data mapping
------------------
The map cluase on a device construct explicitly specifies how items  are mapped from the host to the device data environment.  The common mapped items consist of  arrays(array sections), scalars, pointers, and structure elements.
Note that a C/C++ pointer is mapped as a zero-length array section. OpenMP uses a combination of implicit and explicit data mapping.

implicit data movement 
within a target region
 - scalar variables are implicitly mapped as firstprivate and not copied back to the host at the end of the target region
 - non-scalar/non-pointer variables which have a complete type are copied to the device at the start of the target region, and copied back at the end, the so-called  mapped with a map-type of tofrom
 - Pointers are implicitly copied, but not the data they point to.

explicit data movement
explicit data movement is controled by the map clause
Data allocated on the heap needs to be explicitly copied to/from the device:
the various forms of the map cluase are
 – map(to:list): On entering the region, variables in the list are initialized on the device using the
original values from the host (host to device copy).
 – map(from:list): At the end of the target region, the values from variables in the list are copied
into the original variables on the host (device to host copy). On entering the region, the initial
value of the variables on the device is not initialized.
 – map(tofrom:list): the effect of both a map-to and a map-from (host to device copy at start of
region, device to host copy at end).
 – map(alloc:list): On entering the region, data is allocated and uninitialized on the device.
 – map(list): equivalent to map(tofrom:list).


.. csv-table::
   :widths: auto
   :delim: ;

   10 min ; aaa
   10 min ; bbb



  map(to:list)     ; On entering the region, variables in the list are initialized on the device using the original values from the host (host to device copy) 
 map(from:list) ;   At the end of the target region, the values from variables in the list are copied into the original variables on the host (device to host copy). On entering the region, the initial value of the variables on the device is not initialized.         
 map(tofrom:list)    ;    the effect of both a map-to and a map-from (host to device copy at start of region, device to host copy at end). map(alloc:list)     ;       On entering the region, data is allocated and uninitialized on the device.   
 map(list)           ;  equivalent to map(tofrom:list)      





data shaping
------------------
To mapping data arrays  and pointers, one must use array section notation:
C/C++: pointer [lower-bound:length]
Fortran:
OpenMP defines that the pointer itself (a) is mapped as a zero-length array section.
– Zero length arrays: A[:0]

data region
------------------
Due to distinct memory spaces on host and device, transferring data becomes inevitable. 
From version 4.0, by default all variables within the lexical scope of the construct are copied to and from the device,
.. exceptions::
  1. device is host
  2. data already exists on the device from a previous execution

How the target construct  creates storage, transfer data, and remove storage on the device  are clasiffied as two categories:  structured data region and unstructured data region

Structured Data Regions
------------------
The target data construct is used to create a structured data region which is convenient for providing persistent data on the device which could be used for subseqent target constructs.

unstructured Data Regions
------------------
The unstructured data constructs have much mor freedom in creating and deleting of data on the device at any appropriate point.


Optimize Data Transfers
------------------
◼ Reduce the amount of time spent transferring data
▪ Use map clauses to enforce direction of data transfer.
▪ Use target data, target enter data, target exit data constructs to keep
data environment on the target device.
Inefficient to move data around all the time
Want to keep data resident on the device between target regions

# kernel-hax
My journey into the Linux kernel.

## the what?
- modified linux 5.15 in the following ways to learn more about the various aspect of the linux os

## memory management
The patch of all changes is available in `mm.patch`
- Modified memory management to capture major page faults per process
- Injected a per process tree like data structure, `extents`, to track contiguous pages virtual to physical page allocations
- Here is the idea:
    - By keeping track of the range of pages that are contiguous, we can
      1. Free up space in the TLB, because the translations of contiguous allocations can be handled by the software based `extent`
      2. Reduce the number of page table traversals specifically for those pages that reside in contiguous regions. We can obtain the desired physical page based off of an offset from the first page in the contiguous range
      - The freed up space in the TLB and the software based management of page faults of pages which are contiguous allows for reduced traversals of the page table and more TLB hits of pages that would otherwise not be in the TLB
- Measured the feasibility of pruning TLB translations under different workloads

## scheduler
The goal here was to implement cooperative scheduling without simply calling `yield`, thus forcing the understanding of the internals of the scheduler. This was done in two ways by registering syscalls to:
1. Add a penalty to the `vruntime` of selected tasks. The oenalty could then be disabled with another syscall. Check out `vruntime.patch`
2. Put selected threads onto kernel wait queues by safely removing the corresponding `task_struct` from CPU runqueues. These tasks could then be returned to corresponding runqueues with another syscall. Check out `waitq.patch`

## other interesting stuff
- The implementation of kernel data structures is unorthodox and genius at the same time. The thing that fascinated me the most was the `container_of` macro
- Syscalls have a lot of overhead since due to a proviledge escalation occurring at the boundary of user space and kernel land. Furthremore, any pointers passing between this boundary add additional overhead as they have to processed by special functions to ensure that a user is not able to address stuff inside the kernel
- The usage of macros was a learning experience on its own as I learned that besides simply defining constants, the are also used for:
    - turning specific functions and functionality on and off
    - changing the bounds and access patterns of loops based on what functionality is turned on
- Locks, locks, locks. There is a lot to consider in terms of locking, especialy with respect to the scheduler becasue data is constantly moving between different CPU runqueues making synchronization that much more difficult and costly

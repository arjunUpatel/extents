# kernl-hax
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
The goal here was to implement a form of cooperative scheduling 
- talk about both the approaches and what you learned about in each one

## other interesting stuff
- talk about the papers you read, syscall integration, ssafely copying pointers from userland
- how macros can be used to turn on different functionality
- smart macro work to traverse the linkedlists and get the containing node of a given ll node
- Macros are very helpful
- Syscalls have a lot of overhead since due to a proviledge escalation occurring at the boundary of user space and kernel land. 

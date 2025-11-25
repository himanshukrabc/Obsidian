The OS takes up resources and transforms it into a more general and easy to use resource for processes. This is called virtualisation.

### User Mode and Kernel Mode
- There are certain hardware instructions that only the OS can execute. These are sensitive calls that cannot be digressed to a user process as it can harm the system.
- Thus there are two modes in a computer.
- Any process running in user mode cannot send instructions directly to the hardware.
  Processes running in Kernel mode can do so.
### System Calls
- If a process needs to perform actions with hardware(eg I/O), it has to make a system call.
- These are procedure calls which transfer control to the OS while simultaneously raising the privilege to the Kernel Mode.
##### fork()
- Used to create a new process. It creates an exact copy of the currently running process.
- The new child process starts executing from the exact same point from where the fork was called. Just that the value returned from the fork call is different.
- Child -> 0, 
  Parent -> pid, 
  If fork fails -> -1
``` C++
int rc = fork();
if (rc < 0) { // fork failed; exit
	fprintf(stderr, "fork failed\n");
	exit(1);
} else if (rc == 0) { // child (new process)
	printf("hello, I am child (pid:%d)\n", (int) getpid());
} else { // parent goes down this path (main)
	int wc = wait(NULL);
	printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
	rc, wc, (int) getpid());
}
```
##### wait()
- By calling the wait(), a parent process can pause its execution and wait for the child process to execute.
- Once the child process is done, the wait() returns the exit status of the child process.
##### exec()
- It is used to start up a new process. It loads the code up from an executable and takes in the arguments to pass to the new process.
- Then it overwrites the current process to run the new process.
###### To start a new process, a prev process has to run fork() and then exec() this separation allows the calling program to alter environment of the to be run process.
###### Shell
Shell basically waits for you to give some input. It then runs a fork command and based on your input, runs an exec() on the child. The parent calls the wait() to display the exit status of the child process.

#### Trap Handler
- Whenever a system call is triggered, a **trap** instruction is called. The hardware handles control to the trap Handler. 
- The trap handler uses a **trap table**, initialized on boot by OS to make sense of what instruction to perform based on the system call.
- When the OS is done a **return-from-trap** instruction is called.
- When trap is called, OS takes over the CPU cycles. The processor will push the registers onto a per-process **kernel stack**; the return-from-trap will pop these values off the stack and resume execution.
# Virtualising the CPU
#### Process
- Any running program is called a **process**.
- When a process runs it has - 
  **code** - The code written by user in executable format is loaded up in the memory. 
  **stack** - Contains the local variables, functions, return addresses etc/
  **heap** - Contains memory which dynamically allocated.
- The register context will hold, for a stopped process, the contents of its registers.
- There are also some registers including the 
	- instruction pointer - The instrcution the process is currently executing.
	- Stack and Heap pointer - location of the stack and the heap.
	- File discriptors - Where to read, write output and errors.
##### Process States
- **Running**: Process is running on a processor.
- **Ready**: Process is ready to run but for some the OS has chosen not to run it at this given moment.
- **Blocked**: Process has performed some kind of operation and is not ready to run until some other event takes place. Eg- I/O
![[Screenshot 2025-11-15 at 4.49.06 PM.png|400]]
##### Zombie Process
- When a child process finishes, it becomes a zombie process until the parent process reads its exit status by calling wait().
- If the parent exits without calling wait(), It is adopted by the init process(Parent of all processes).
### Limited Direct Execution
- Means a process should not be allowed to hog the CPU until it completes execution.
	- Process might run infinitely or do IO where CPU cycles are wasted
### How to take control back?
#### Cooperative Approach
- OS waits for the process to make system calls.
- Once system call is made control transfers to OS and it makes the switch to another process
- **CONS**
	- Process may get stuck in infinite loop.
	- Process may not do system calls at all.
#### Non Cooperative Approach
- All processes are preempted to make other processes run.
- A **timer interrupter** tells a **interrupt handler** when a process has to be interrupted.
- There is a **scheduler** which decides what process has to run next.
##### Context Switching
- Whenever a process is preempted a timer interrupt happens and control transfers to OS.
- All data(registers etc) related to it has to be moved out of the memory and stored in a per process data structure called **kernel stack**.
- The kernel stack pointer is updated to the pointer of the to be run process.
- It is a costly process for the CPU to perform.
### Scheduling
- We make the following assumptions when starting our algo trials and we relax as we go on.
	1. Each job runs for the same amount of time.
	2. All jobs arrive at the same time.
	3. Once started, each job runs to completion.
	4. All jobs only use the CPU (i.e., they perform no I/O)
	5. The run-time of each job is known.
#### Scheduling Metrics
- **Turnaround Time** = T<sub>completion</sub>− T<sub>arrival</sub>
- **Response Time** = T<sub>firstrun</sub>− T<sub>arrival</sub>
#### First In First Out (FIFO)
- If we relax assumption 1, there is **Convoy Effect**
- **CON** - ***Convoy Effect*** - If a heavy process is run first, short jobs coming just after that will get queued up even if they dont take a lot of time. In this case the average turnaround time will be high.
#### Shortest Job First (SJF)
- This is the optimal scheduling algorithm. 
- However, it is not possible to know the length of job beforehand.
- And jobs can arrive at any point of time(Assumption 2 revoked) so effectively a shorter job can arrive and the CPU is busy.
#### Shortest Time-to-Completion First (STCF)
- A preemptive SFJ. When a shorter job arrives you preempt the running process and run the shorter job.
- This has a bad response time. If multiple jobs arrive at the same time, the longer job has to wait for the other jobs to run through its entirety for it to execute.
#### Round Robin
- Jobs are preempted when they have run for a **time slice(scheduling quantum)** then switches to the next job in queue.
- If we reduce the time slice, **Response Time is improved** which is good for processes with a lot of user interaction.
- However it leads to a lot of context switching which is an overhead. This also means that RR is has the **worst Turnaround time**.
- RR is a fair scheduler. Any fair scheduler will have a bad average Turnaround time.
#### Multi Level Feedback Queue (MLFQ)
- It is an improvement on RR - 
	1. Tries to reduce turnaround time
	2. Tries to improve response time to accommodate interactive processes
- Has multiple queues with different priority levels.
- **Rules**
	1. If Priority(A) > Priority(B), A runs 
	2. If Priority(A) Priority(B), A & B run in RR.
	3. When a job enters the system, it is placed at the highest priority (the topmost queue).
	4. Once a job uses up its time allotment at a given level its priority is reduced
	5. After some time period, move all the jobs in the system to the topmost queue
- Earlier it was proposed that if a job gives up the control voluntarily(By I/O etc), it can stay in the same priority queue
	- Process can game the scheduler by giving up control just before the quantum expires.
	- Interactive processes which do I/O frequently will give up control and always stay in the top queue. This will lead to **starvation** of other processes. => Improved by **Priority Boosting(Rule5)**
- The time slice and number of queues are variables which are tuned per computer basis.
#### Proportional Share Schedulers
##### Ticket Based Lottery Scheduler
- Each process is assigned some tickets. The scheduler randomly draws a ticket and that process is allowed to execute.
- In the long run each process gets a proportional share in the run time on CPU.
- **Ticket Transfer** - A process will transfer its tickets to another process to make it more likely to execute.
- **Ticket Inflation** - A process can temporarily inflate its ticket value.
- For longer length jobs this is a very fair scheduler. For short jobs this can be very unfair.
- Unfairness Metric = Worst time to complete / Best time to complete.
  This varies a lot and can be very bad is many cases. It is unpredictable as well.
##### Stride and Pass Scheduler
- Stride = 1/#(tickets with process)
- Initially pass is set to 0. The scheduler will pick the process with the least pass value.
- Process with the most no of tickets is more likely to execute. And this is precise in scheduling as even short jobs will be treated fairly.
- This is predictable and fair.
#### Scheduling in Multiprocessor Systems
##### Caching
- Caches are based on notion of locality
	- **Spatial Locality** - If data is accessed in a specific location, another location in the locality will be accessed.
	- **Temporal Locality** - If a specific location is accessed it is likely that it will be accessed again.
- **Cache Coherence** - In case of multiple processors, multiple processes can update the same cache entry. This will lead to falsification of cache data.
- **Bus Snooping** - Check if the process trying to access the cache is the same as the one which stored it. If not remove it.
- **Cache Affinity** - It is advantageous to run the process on the same processor after preemption. This will help in reuse of the cache entries.
##### Single Queue Scheduling
- There is a single queue with all the jobs and the scheduler is responsible for picking which job runs on which scheduler.
- There will be locks involved as multiple processors will be trying to run the same jobs making it costly.
##### Multiple Queue Scheduling
- There will be a queue associated with each processor so no locks will be involved.
- There will be **load imbalance**. This can be reduced by migrating jobs between queues. This can be implemented by Work Stealing
- **Work Stealing** - Processor will look into other queues and if they have more load then they will take up a job from the queue.
# Virtualising the Memory
##### Address Space
- Each process has its own virtual memory which starts at 0. This is called the address space.
- It consists of 3 parts - code, stack and heap.
- Heap and Stack grow. Heap grows downwards while the stack grows upwards.
##### Goals of memory virtualisation
1. **Transparency** - For the process the virtualisation should be invisible.
2. **Efficiency**
3. **Protection** - Other processes should not be able to access the address space of one process.
##### Common Memory Errors
1. **Segmentation Fault** - When you access memory before allocating it.
2. **Buffer Overflow** - Not Allocating Enough Memory
3. **Uninitialised Read** - Forgetting to Initialise Allocated Memory
4. **Memory Leak** - Forget to free up the memory
5. **Dangling Pointer** - Referencing memory that was freed.
6. **Invalid Free** - Called free on already freed up memory.
#### Static Relocation
- A program was written assuming memory started at 0. Then a loader was run on it which manually changed all addresses to reflect the actual address based on the memory the process was allocated.
### Base and Bounds(Dynamic Relocation)
- Each memory referenced by the process will be a **virtual address**. This will be translated into a **physical address**.
- The address space will be defined by 
  **base register** - storing the location where the address space starts on the memory.
  **bound register** - stores the size/last location of the address space(can be done in 2 ways).
- *physical address = virtual address + base*
- Whenever a context switch happens these registers will be updated.
- A lot of space in the address space is wasted if the heap and stack is small. There is space free but any new process cannot use it. This is called **internal fragmentation**.
#### Free List
- The OS maintains a list of free addresses left on the memory. This is stored in the form of a linked list.
- When a process is created, the list is traversed to find a space for the address space of the process.
- When memory is deallocated, it needs to added back to the list and merged with adjoining free memory if any.
#### Process Control Block
- When a process stops running its base and bounds values are stored in a per process structure called the Process Control Block.
- It contains - PID, CPU registers, Program Counter and other info.
*Kernel stack on the other hand stores the values when the process does a system call or interrupt or fault*
### Segmentation
- We have a base and bounds register per logical segment of the process.
- **Ways**-
	- *Explicit* - The top 2 bits of the address decide what segment we are referring to.00 -> Code, 01->Heap, 11->Stack.
	  Some systems use just 1bit and store the code in the stack segment.
	- *Implicit* - Based on the fetch was generated. Program Counter->Code, Stack/Base Pointer->Stack else Heap.
- **Segment Registers** - Store the Base, Bounds(Size) and growth(as stack grows upwards)
  ![[Screenshot 2025-11-20 at 9.45.35 AM.png|400]]
```C++
SEG_SHIFT = 12
SEG_MASK = 3<<SEG_SHIFT //11000000000000
// get top 2 bits of 14-bit VA
Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT
// now get offset
Offset = VirtualAddress & OFFSET_MASK
if (Offset >= Bounds[Segment])
    RaiseException(PROTECTION_FAULT)
else
    PhysAddr = Base[Segment] + Offset
    Register = AccessMemory(PhysAddr)
```
##### OS Support Required
- During Context Switch, segment registers must be saved and restored.
- OS must find space for the new segments. Segments vary in size, a algo must find space in the available memory.
#### External Fragmentation 
- Variable sized segments -> Memory becomes full of holes of free spaces which cannot accommodate an entire segment although overall there is space available.
**Solution**
1. Copy the memory of running processes into contiguous memory blocks and consolidate free memory.
2. Use policies to prevent external fragmentation.
##### Free List
- It contains a list of free locations on memory
- ![[Screenshot 2025-11-20 at 10.07.14 AM.png|400]]
- **Splitting** - When memory is demanded, an entry of free list will split and the first part will be used for the segment.
- **Coalescing** - When memory is freed, it will be joined with the neighbouring free memory segments if any.
##### Best Fit
- Search through the free list to find the smallest entry just greater that the requested memory
- Costly as the entire list has to be searched.
##### Worst Fit
- Search through the free list to find the largest entry and allocate from there.
- Costly as the entire list has to be searched.
- Leads to excessive fragmentation
##### First Fit
- Find the first block larger than requested memory.
- Faster as no need to search the list entirely.
- This leads to several small free entries polluting the top of the list.
##### Next Fit
- Start searching from the address from the entry that you stopped at last.
- Faster and does not pollute the list.
##### Segragated Lists
- Maintain lists for memory sizes which are frequently requested.
##### Buddy Algorithm
- Free memory can be through of a big space of size 2<sup>N</sup>.
- When a memory request is made, you divide the memory space repeatedly until you find a memory just more than requested.
- When reclaiming memory, we can judge from the memory size who its buddy will be(As size will indicate the depth and the address will indicate which sibling will be its buddy). Add it as the sibling to the buddy and keep merging free memory until you cannot.
- Suffers from *internal fragmentation* as more than required memory is allocated.
### Paging
- The memory is chopped up into fixed size pages.
- The memory of a process is thus an array of slots(called **page frames**) where a single page can fit in.
- Can suffer from **internal fragmentation** if the page size is large.
##### Page Table
- Per process data structure where the address translation(called **Page Frame Numbers(PFN)**) of each virtual page is stored.
- Since Virtual Pages are numbered from 0, the page table is actually an array, with **Virtual Page** **Number** as index.
- A **Page Table Entry(PTE)** contains - 
  **valid bit** -> If the translation is valid or not. Unused pages are marked invalid.
  **protection bits** -> If we can read write or execute from that page.
  **present bit** -> If the page has been swapped out.
  **reference/accessed bit** -> If a page has been accessed. Used to keep popular pages in memory.
- *Page Tables can get quite bulky specially when the page size is small.*
- If we have a large page table, a lot of memory will be wasted to just store the page table.
##### Address Translation Example
- Lets say we have a 32bit address space with 4KB pages. 4KB page = 2<sup>12</sup> addresses per page.
- Thus there will be 2<sup>20</sup> pages. Now if we have an address, The top 20 bits will be used as the VPN and the next 12 will be offset.
```C++
// Extract the VPN from the virtual address
VPN = (VirtualAddress & VPN_MASK) >> SHIFT
// Form the address of the page-table entry (PTE)
PTEAddr = PTBR + (VPN * sizeof(PTE))
// Fetch the PTE
PTE = AccessMemory(PTEAddr)

// Check if process can access the page
if (PTE.Valid == False)
    RaiseException(SEGMENTATION_FAULT)
else if (CanAccess(PTE.ProtectBits) == False)
    RaiseException(PROTECTION_FAULT)
else
    // Access is OK: form physical address and fetch it
    offset = VirtualAddress & OFFSET_MASK
    PhysAddr = (PTE.PFN << PFN_SHIFT) 
```

##### Cons
- Can suffer from **internal fragmentation** for large pages.
- Small pages make the **page table too bulky** and occupy large space in physical address of process.
- When translating, you need to access the page table to get the translation. This is an **extra memory access per translation** which makes paging slow.
#### Translation Lookaside Buffer(TLB) - Speed Up Paging
- Helps in speeding up Paging
- It is a hardware cache that stores the translations of frequently referenced pages.
```C++
VPN = (VirtualAddress & VPN_MASK) >> SHIFT
(Success, TlbEntry) = TLB_Lookup(VPN)

if (Success == True) // TLB Hit
    if (CanAccess(TlbEntry.ProtectBits) == True)
        Offset = VirtualAddress & OFFSET_MASK
        PhysAddr = (TlbEntry.PFN << SHIFT) | Offset
        AccessMemory(PhysAddr)
    else
        RaiseException(PROTECTION_FAULT)
else // TLB Miss
    PTEAddr = PTBR + (VPN * sizeof(PTE))
    PTE = AccessMemory(PTEAddr)
    if (PTE.Valid == False)
        RaiseException(SEGMENTATION_FAULT)
    else if (CanAccess(PTE.ProtectBits) == False)
        RaiseException(PROTECTION_FAULT)
    else
        TLB_Insert(VPN, PTE.PFN, PTE.ProtectBits)
        RetryInstruction()
```
##### TLB Miss
- It is quite costly as a TLB miss would lead to perform extra memory references.
- First look for translation in Page Table then, Update the TLB and then get the translation. This is slow.
 ###### Who handles TLB Miss?
 - **Hardware approach** - Hardware must maintain a register storing location of Page Table. Navigate to translation, update TLB and then translate.
 - **Software Approach** 
   - TLB Miss raises a trap call. The trap handler then performs the required steps to update the TLB
   - When this trap is called, the hardware needs to save PC to the previous instruction so that it can be retried with the updated TLB when control returns from the trap handler.
- Ensure not to cause series of TLB Misses. 
  -> TLB miss handlers could directly use physical memory and not page tables.
  -> Use permanently valid translations for TLB miss handlers.
##### TLB Entry
- VPN, PFN
- **Valid bit** - Indicates is a TLB entry is valid or not.(Not PTE valid bit which indicates if page is allocated by process)
- **Protection bit** - How a page can be accessed.
##### TLB Issues - Context Switch
- When context switches OS ensures that the new process does not use the OLD translation values.
- Thus we also store **ASID(Address Space Identifier)** which is essentially a PID but has fewer bits. Checked whenever a process tries to access a TLB entry.
#### Reducing Page Table Size
- Most of the PTEs are useless when the process starts as they are empty.
##### Segmentation with Paging
- We use base and bound registers to store the page table physical address and the number of pages allocated to each segment(code, heap and stack).
- Here as well the top 2 bits will be used to identify which segment the address reference belongs to.
  However this also
  ![[Screenshot 2025-11-21 at 9.52.37 AM.png|800]]
- Still segmentation is inflexible and each segment page table will still have unused pages.
##### Multi Level Page Tables
- Chop up the page table in page size units. Then do not allocate that page itself if all the page table entries are invalid.
- To track if the page of the page table is valid, use a **Page Directory**.
- **Page Directory** - tells the translation of the page of the page table or that the entire page of the page table is invalid.
###### Page Directory Entry
- A Page Frame Number, a valid bit.
- A valid bit in PDE indicates whether any page corresponding to the Page Table portion this PDE points to has an entry or not.
###### Pros - 
- If constructed carefully it Multilevel Paging makes it easier to manage memory as the page table is chopped up into page sized units.
- It allocates just enough memory to accommodate the process's memory requirements.
###### Cons - 
- TLB miss would be costlier as it would require 2 new memory accesses.
###### Example -
- 16KB(14bit) - Address Space, 64byte(6bit) - pages. => VPN = 8bit.
- Code -page 0,1; Heap -page 4,5; Stack -page 254,255
- Assuming each page table entry to be 4bytes, page table will be 256x4 bytes = 16pages. Each page will contain 16 PTEs
- Thus PDE will have 2 entries -> contianing pages for table 0,1,3,4 and 254,255
```C++
	PTEAddr = (PDE.PFN << SHIFT) + (PTIndex * sizeof(PTE))
```
- Lot of space is saved as 2 pages are only allocated for Page Table and Page Directory will take up 1 page.![[Screenshot 2025-11-22 at 12.22.11 PM.png|550]]

```C++
VPN = (VirtualAddress & VPN_MASK) >> SHIFT

(Success, TlbEntry) = TLB_Lookup(VPN)
if (Success == True) // TLB Hit
    if (CanAccess(TlbEntry.ProtectBits) == True)
        Offset = VirtualAddress & OFFSET_MASK
        PhysAddr = (TlbEntry.PFN << SHIFT) | Offset
        Register = AccessMemory(PhysAddr)
    else
        RaiseException(PROTECTION_FAULT)
else // TLB Miss
    // first, get page directory entry
    PDIndex = (VPN & PD_MASK) >> PD_SHIFT
    PDEAddr = PDBR + (PDIndex * sizeof(PDE))
    PDE = AccessMemory(PDEAddr)
    if (PDE.Valid == False)
        RaiseException(SEGMENTATION_FAULT)
    else
        // PDE is valid: now fetch PTE from page table
        PTIndex = (VPN & PT_MASK) >> PT_SHIFT
        PTEAddr = (PDE.PFN << SHIFT) + (PTIndex * sizeof(PTE))
        PTE = AccessMemory(PTEAddr)
        if (PTE.Valid == False)
            RaiseException(SEGMENTATION_FAULT)
        else if (CanAccess(PTE.ProtectBits) == False)
            RaiseException(PROTECTION_FAULT)
        else
            TLB_Insert(VPN, PTE.PFN, PTE.ProtectBits)
            RetryInstruction()
```
- From PDE_Mask & PDE_Shift, get the PDE Index. Then to get PDE, PDBR + PDE_Index\*Size(PDE)
  From PDE get the PFN of the page that contains the page table portion. Now get PT_Index from PTE_Mask and PTE_Shift.
  Get PTE from (PDE.PFN)<<SHIFT + PT_Index\*Size(PTE). From PTE -> PTE.PFN << SHIFT + Offset.
##### Inverted Page Tables
- Instead of having per page page table, we maintain a page table indicating which process's which virtual page is mapped to this physical page.
### Swapping
- It would be wise for OS to support large address space as the programmer would not have to think about memory when coding.
- Instead of storing all the pages in memory, swap out few pages into the disk in a **swap space**.
- All code pages are loaded as required.
##### Present Bit
- Indicates whether the page is in memory or has been swapped into the swap space.
- If present bit =0, then the OS would look store the disk page address in the PTE so that it can swap the page in. This is called **pagee fault**
##### Page Fault
- If a page needs to be accessed that is not in memory, a **page fault** occurs.
- The control transfers to a Page Fault Handler. It is responsible to swap the required page in form memory.
- During this time the process is blocked and the OS can run other processes.
##### Page/Swap Daemon
- It is a process running to keep the memory space free.
- If the number of pages exceed **High Watermark(HW)** it swaps pages out to the swap space. If it is lower than **Low Watermark(LW)** it swaps pages in.
- Which pages to swap in is decided by swapping policies. Thus the memory is essentially a cache, storing the frequently accessed pages.
#### Cache Management and Replacement Policies
- Avg. Memory Access Time (AMAT) = P<sub>Hit</sub>\*T<sub>M</sub> + P<sub>Miss</sub>\*T<sub>D</sub>
- The most optimal approach is to replace the page which will be used furthest in the future. But it is difficult to implement.
- Types of cache misses -
	- **Cold Start/Compulsory Miss** - when cache is empty
	- **Capacity Miss** - when cache is full
	- **Conflict Miss** - When cache is empty but the section used incoming item is full. Occurs in cases where cache entries are non associative.
##### FIFO
- Pages are placed in a queue and the first page in queue will be removed when space is required.
- Does not have **Stack Property** - If cache size increases, efficiency might decrease.
##### Random
- Pages removed at random. Performs better than FIFO.
##### History Based
###### Least Recently Used(LRU)
- Takes into account the history of the entry and evicts the Least Recently Used entry.
###### Least Frequently Used(LFU)
- Evicts the page which is least frequently used.
##### Testing under different workloads
- **Random Workload**- Pages are referenced at random. All policies perform identically.
- **80-20 Workload** - 80% calls to 20% pages. LRU performs much better than others.
- **Looping Workload** - Pages are referenced 1-n then from 1 again. All policies perform badly except Random.
- Since 80-20 is the workload generally observed. LRU is best.
##### Implementing LRU
- If we implement LRU, it is very costly and demanding. You need to update a data structure everytime a page is accessed and move the page entry to the top(Most recently used side).
- If we use hardware support and for each page maintain when the page was last referenced, OS just needs to scan and find the one with the earliest entry. But this requires traversal through the entire set of pages and is costly.
###### Approximating LRU - Use bit
- You set use bit to 1 to all the pages and implement an clock algo
- Set the pointer to any one of the pages. If the current page has use bit = 0, swap it. Else set use bit to 0 and move to the next page.
###### Dirty Bit
- It is easier to replace places which have not been written to/modified as they dont require copying data.
- So some algos consider dirty bit while swapping pages
#### Page Selection
- Which page to swap into memory. Most OS just use demand paging and swap in pages when their demand arises.
#### Writing to Disk
- Another policy is to determine when to write out to disk. 
- It is better to cluster all the writes onto disks and execute them together. This is called **grouping/clustering of writes**
#### Thrashing
- Sometimes while running memory intensive processes the memory may fall short and OS will be **busy paging constantly**.
- To prevent this some OS may decide not to run a subset of processes and execute others. This is called **admission control**.
- Other OS may choose to kill some memory intensive process as in Linux.
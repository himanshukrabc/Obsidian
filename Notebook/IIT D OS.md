- It is a piece of software that allows other softwares to use the hardware.

- It consists up of a bunch of drivers. Drivers are interfaces which allow applications to interact with the hardware. This enables multiple applications to work together without the need of trust.

- Uniprocessing OS ⇒ 1 app at a time.

- It allows to run different programs while allowing them to coexist on the disc.

> [!important] _**KERNEL is another name for the OS, it is used for disc and memory management.**_

> [!important] Program ⇒ executable file stored on the disc.
> 
>   
> Process ⇒ any running program.  
> Each process has an Id called the PID.

# UNIX

### File System

- it is on disc Data Structure to store/retrieve files and an interface to access them.

```SQL
fd = open("filename") => gives a file descriptor. 
read(fd,buf,100) => read from fd, 100 chars into buf(a pointer in memory)
read(fd,buf,100) => write into fd, 100 chars from buf
```

## SHELL

> [!important] It is better to use shell as an app because it can create multiple copies of itself when opening files due to fork.

- It is an applications that performs certain tasks on given prompts.

- When you give a command, it searches for the file and opens it. Then the control is transfered to the opened app.

- Now the SHELL performs KERNEL FUNCTIONS/ SYSTEM CALLS which allows it to communicate with the OS Kernel.

- Functions of SHELL
    
    - When opening an app, control must go to it and return back when the app exits
    

> [!important] fork() creates a copy of the calling process, which seems wasteful. Thus, widows OS uses a system call called createProcess(), which doesnot require copying.

### The FD table

- There is a file descriptor table fro each process, having entries of pairs of (key,FDs), which is not accessible by any process.

- rocesses can manipulate the table through syste calls.

- The first 3 FDs are spl.
    
    - 0 : Standard Input, STDIN
    
    - 1 : Standard Output, STDOUT
    
    - 2 : Standard Error, STDERR
    

- The STDIN,-OUT,-ERR are decided by the OS. Any program will read from STDIN and write to STDOUT.

  

### Inter Process Communication(IPC)

1. Through exit and wait calls.

1. Using fork, the child also has access to the all that parent has access to.

1. Using a pipe.

  

### System Calls

- **exec(”__filename”)**
    
    - Searches the filename, If found, closes the process and runs the file.
    
    - For the shell, as it is closed the control cannot return to the shell. Hence directly calling the exec call wont serve shell’s functions.
    

- **fork(”__filename”)**
    
    - It creates a duplicate(_**child**_) process of the app that calls it.
    
    - The parent process gets stored in disc.
    
    - The child process calls the exec(), which allows the control to return to SHELL on exit.
    
    - Note that the child process has its own copy of the FD table.
    
    > [!important] A fork() call leads to 2 identical applications, however the return value of the fork call is different on the processes. Based on this the process can decide which app will call exec and which will continue to exist.
    
    ![[/Untitled 3.png|Untitled 3.png]]
    

- **exit(exitcode)**
    
    - exits the current process, frees the allocated resources and sends the control back.
    
    - Control goes back to the parent process.
    
    > [!important] A process’s exitcode is sustained untill the parent process calls wait(). Such a process which exited but wait has not yet been called is called a
    > 
    > ==**Zombie Process.**  
    > ==_**This occourance is called a leak.**_  
    > All zombie processes are killed on shutdown of system.
    

- wait(0)
    
    - wait until the child process exits.
    
    - return value is the exitcode of the child process.
    
    - Helps in Inter Process Communication (IPC).
    

- dup(FD)
    
    - It selects the first available FD and assigns it the passed FD.
    
    > [!important] ==**Appending into files:**==
    > 
    > - Each file has its pointer in the file descriptor which also stores an offset. This indicaes from what location on disc the write operation can be performed to append data.
    > 
    > - However, in case where both the parent and child are trying to do a write operation, the offsets will not be in sync. This will lead to failure in appending.
    > 
    > - Another case is when we have two FDs pointing to same file but different pointers.
    > 
    > - ==Example==
    >     
    >     2 > &1 ⇒ Error stream is now written on STDOUT.
    >     
    >     ```C++
    >     //If this method is used,
    >     close(1);
    >     open("foo");
    >     close(2);
    >     open("foo);
    >     ```
    >     
    >     ![[/Untitled 1 2.png|Untitled 1 2.png]]
    >     
    >     As offsets will not get updated for both, it will lead to overwiting of write operation by error operation and vise versa.
    >     
    >     ![[/Untitled 2 2.png|Untitled 2 2.png]]
    >     
    >     This is what we want. This is implemented by _**dup(FD)**_ system call.
    >     
    > 
    > - Hence in such a case the dup() is made to copy the pointer of the file into the child process.
    

- open(”__filename”)
    
    - It traverses the FD table and returns the first available FD, pointing to the file.
    
    - If -ve value returned, there is an error.
    

- **read**(fd, buf, size)
    
    - Reads content of FD file, into buffer upto size sz.
    

- **write**(fd, buf, size)
    
    - Writes into the FD file, from buffer upto size sz.
    

- close(fd)

### pipe(fdarray)

Consider the following sequence of operations

```C++
$ sort < file.txt > tmp1
$ uniq < tmp1 > tmp2
$ couhnt_words < tmp2
```

If the file size of file.txt is large, we would require a lot of memory. Ideally we require a system to connect STDINs, STDOUTs of processes.

![[/Untitled 3 2.png|Untitled 3 2.png]]

This is facilitated by the _**pipe()**_ system call.

==_**pipe( fdarray )**_==

- We are supposd to pass an **int fdarray[2]; array. ⇒ must be global in nature**

- It allocates the first 2 availavble FDs, fdarray[1] ⇒ I/P, fdarray[0] ⇒ O/P

- When a child process is created for a process with pipe, the pointes of fdarray[] are same. Thus, they can communicate with each other.

![[Untitled 4.png]]

Code for above problem

```C++
int fdarray_sh[2],fdarray_so[2],fdarray_u[2],fdarray_w[2];
//shell.
while(1){
	pipe(fdarray_s);
	pid = fork();
	if(pid == 0) {
		//we are reading from the child process
		close(1);
		dup(fdarray_s[1]);
		read(0,buf,100);
		write(1,buf,100);
		exec(sort);
	}
}

//sort => uniq and wc implementations will also be similar.
// In wc, no need to connect O/P
while(1){
	pipe(fdarray_so);
	pid = fork();
	if(pid == 0){
		//output connected to sort pipe.
		close(1);
		dup(fdarray_so[1])
		//now 0 will be connected to shell pipe
		close(0);
		dup(fdarray_sh[0]);
		/*sort implementation*/
		exec(uniq);
	}
}
```

_**sort < file.txt | uniq | wc**_ ⇒ creates the setup

  

### Scheduling Problem

- I/P process produces more data than O/P process uses. ⇒ Pipe size inc.

- O/P process uses data faster than produced by I/P process ⇒ waste of time and space.

- Thus the schedule(time allowed to run) of the processes must ensure max utilization.

  

  

### BASIC SHELL PROGRAM:

```C++
while(1){
	write(1,"$ ",2);
	read(0,command,args);//command maybe to open a file, arg => filename
	pid = fork();
	if(pid == 0){
		exec(command, args);
	}
	else if (pid > 0) wait(0);
	else printf("Error in fork");
}
```

  

### Redirection of write (”>” opeator) & Redirection of read(”<” operator)

$ _**sh < foo > bar :**_  
It reads the file “foo” into the shell and writes onto file “bar”

```C++
//Write
close(1);
open(bar,wr); //attached to 1st available FD => 1
//Read
close(0);
open(foo,r);
exec();
```

  

  

## Processes

- ==Program ⇒ executable file stored on the disc.  
    Process ⇒ any running program.==

- The first process that is generated on start is called init process.

- ==Each process has an Id called the PID and== an address space which is private to it and cannot be accessed by other processes.

- A process is an adress space accompanied by a thread(Execution flow).

![[Untitled 5.png]]

## Signals

- It is like a asynchronous function call.

- Every process has a ==signal handler== which executes in the address space.

- When a signal is sent, the program stops its execution, makes a call to the signal handler and resumes once a value is returned.

- Signals can be overwritten. However to ensure functionality OS restricts some signals from override.

- There are 2 system calls that help in thi==**s**==
    
    - ==**signal**==(int signum, void (*handler)(int v))
        
        - signum is the signal number, handler in the handler funtion. It takes in an integer argument which helps us use the same handler for multiple signals
        
        - SIGINT ⇒ Interrupt
        
        - SIGSTOP ⇒ Stops the process
        
        - SIGSEGV ⇒ Segmentation fault ⇒ Process accesses a oction it is not allowed to access.
        
        - SIGCHLD ⇒ Generated when a child process exits. This helps when a child and parent process run concurrently. If this signal has been recieved, the parent process can call the wait function thereby eliminating zombie processes.
        
        - When the parent exits without calling wait() ⇒ child becomes ==orphaned process.== This process is then attached to the init process, which calls wait() on it.
        
    
    - ==kill====(pid, signum)==
        
        - It is used to send a signal to another process(identified by pid).
        
    

  

### /proc

- It is a pseudo file system that has all the information regarding the running processes.

- It has directories for all the currently running processes.

- It also has other important info like /num_active_cpus file.

- These directories can be modified using open, read, write and close calls. To pass various instructions to OS.

- Eg - if we write 2 in num_active_cpus file, OS shuts all but 2 CPUs on its own.

  

## Synchronization & Threads

- This happens when using pipes. When data is shared using pipes, the reading and writing processes must be in sync to prevent ineffiiency.

- Also this way of sync is costly as it leads to 2 system calls, write then read.

- A better way would be if 2 processes could share a address space in which case it would be just setting and reading bits in memory(not sys calls).

- This can be done by adding 2 ==threads(Execution flows)== in a address space.

![[Untitled 6.png]]

- Multiple threads allow multiple actions to be carried out simultaneously as is the case with multiple processes(more accurately address spaces with single threads).

- Adv of multithreading is that inter process communication is very cheap. Disadv is that there is no isolation.

- Creation of threads
    
    1. Using a system call ⇒ OS knows there are multiple threads in the process  
        such threads are called kernel level threads.
    
      
    
    ![[Untitled 7.png]]
    
    1. Internal imlpementation of threads ⇒ OS does not know about the threads  
        This means that if one thread calls exit, all the threads will be exited.  
        If one thread calls read all the threads will be stopped.  
        Thus there is no concurrency. Such threads are called user level therads.
    
      
    
      
    
    ![[Untitled 8.png]]
    

> [!important] Another way of IPC is by sending signals, along with to the process and recieving data when signal is read.

  

## PC Architecture (x86)

- Bus acts as the communication medium. Each part is assigned a bandwidth on the bus based on the volume of communication through it.

![[Untitled 9.png]]

  

![[Untitled 10.png]]

### Registers in x86

- The x86(8086) architecture, has 4 different registers for calculations and computation. All these were 16bit registers. Each of them were divided into two 8bit parts. Any process can choose to run on the 16bit or 8bit system. :
    
    - AX(AL|AH)
    
    - BX(BL|BH)
    
    - CX(CL|CH)
    
    - DX(DL|DH)
    

  

- It also had 4 16bit registers for reading from memory :
    
    - SP(Stat Pointer)
    
    - BP(Base Pointer)
    
    - SI(Source Index)
    
    - DI(Destination Index)
    

  

- **==IP Register==** There is another 16bit register called IP(Instruction Pointer)/PC(Program Counter). It's a register that holds the memory address of the next instruction to be executed in a program.  
    The OS works as an endless loop. It checks out the IP which comes back with a set of instructions on what action needs to be performed next and then increments the IP. This helps in making programs run on memory while utilizing the CPU resources through IP.

  

- ==**FLAGS Register**== It stores info about various actions carried out. Eg- if last conputation had carry or not etc.

  

- ==**Transform Registers**== : The 16bit CPU can actually support 1mb(2^20) memory. This is done by transforming the 16bit output to 20bit adreses on memory. This was done using :
    
    - CS ⇒ FOR COMPUTATIONAL WORK
    
    - DS ⇒ FOR DATA STORE
    
    - SS ⇒ FOR STACK INSTRUCTIONS
    
    - ES ⇒ FOR SOME SPL PROCESSES
    
    This was done using ⇒ (%cs)*16 + (%ip) ⇒ based on CS the memory was divided into chunks, where access was done sing IP. (%cs ⇒ val of cs)
    

  

> [!important] It was later realized the 16bit is too low. Hence the new OS was developed in such a way to provide backward compatibility. The same OS could run on 16, 32 and 64 architectures.
> 
> When the OS boots, it starts in 16bit then switches to 32bit and then to 64 bit mode.

  

> [!important] The 32 bit architecture also has the same regs but their name is now EAX, EBX, ECX, EDX, ESP, EBP, ESI, EDI (E⇒ Extended). If you call AX, it gets the lower 16bits.

  

### AT&T Syntax

- This is the syntax through which we can manipulate data in the CPU registers.

- It works as __**function**__**suf** %src_reg %dest_reg  
    Eg: add %eax %ebx ⇒ adds val at eax to ebx.

- suf : l ⇒ last 32bit, w ⇒ last 16bit, b ⇒ last 8bit.

- functions ⇒
    
    - **arithmetic :** add, sub, inc, dec, mul, div, neg
    
    - **data movement :** mov, xchng(exchange val in ptrs), push, pop(stack)
    
    - **control :** jmp(jump to addresss), je(jump equal to), jl(jump grtr than) etc.
    
    - **I/O :** in,out etc.
    
    - **string :** movsb⇒(copy a bit from src to dest.)
    
    - **system :** int(interrupt), irt etc.
    

  

> [!important] prefix addl %eax %ebx
> 
>   
> x066 ⇒ fetches only 16bits of data  
> x067 ⇒ extended memory is not used, eax⇒ax, ebx⇒bx  
> These are used when we have to operate on 16bit regs in a 32 bit system.

  

movl %eax %ebx ebx = eax

movl $123 %ebx ebx = 0x123 ⇒ $→constant value, no dereferencing.

movl 0x123 %ebx ebx = *((int32 *) 0x123)) ⇒ typecast address to int32 and dereference.

movl (%eax) %ebx ebx = *((int32 *) eax))

movl 4(%eax) %ebx ebx = *((int32 *) eax+4))

  

## How function calls work?

- Every process has an esp(extended stack pointer), which points to the top of the stack.

- When new element is added, the pointer moves downwards.

- **==pushl %eax== ⇒**  
    movl %eax %esp  
    sub $4 %esp

![[Untitled 11.png]]

- **==popl %eax== ⇒**  
    movl %esp %eax  
    add $4 %esp

  

![[Untitled 12.png]]

![[Untitled 13.png]]

- All system calls are made using **==call==** instruction.**call _sys_func_address** Eg ⇒ call 0x12345  
    > pushl $eip  
    > jmp 0x12345

- When a function call is made, we push the $eip(extended instruction pointer) into the stack and the control shifts to the called program. This is also called ==Return address.==

- When the program exits, the control is then passed onto the pushed eip.**ret**  
    > pop %eip

- Thus when a function call is made,  
    %eip ⇒ points to the first instr of the func  
    %esp ⇒ points to the return address.  
    %esp + 4 ⇒ 1st arg  
    %esp + 8 ⇒ 2nd arg …

![[Untitled 14.png]]

- On return,  
    %eip ⇒ points to the return address  
    %eax ⇒ return value of the function

- %eax, %cx, %edx can be trashed.  
    %ebp, %ebx, %esi, %edi must contain same value as at the call time.
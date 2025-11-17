The OS takes up resources and transforms it into a more general and easy to use resource for processes. This is called virtualisation.

#### User Mode and Kernel Mode
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
###### To start a new process, a prev process has to run fork() and then exec() this separation allows the calling program to execute certain instructions before start of new process if required.
#### Trap Handler
- Whenever a system call is triggered, a **trap** instruction is called. The hardware handles control to the trap Handler. 
- This trap handler is responsible for interpreting what kind of instructions the OS has to execute in response to that system call.
- When the OS is done a **return-from-trap** instruction is called.
# Virtualising the CPU
##### Process
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
##### Time Sharing
- OS can preemptly stop a process and make another process run. This leads to multiple processes sharing the CPU. This is called Time Sharing.
##### Context Switching
- Whenever a process is preempted, all data(registers etc) related to it has to be moved out of the memory and stored elsewhere.
- The data of the to-be-run process has to copied in. This is called context switching. 
- It is a costly process for the CPU to perform.
#### Process States
- **Running**: Process is running on a processor.
- **Ready**: Process is ready to run but for some the OS has chosen not to run it at this given moment.
- **Blocked**: Process has performed some kind of operation and is not ready to run until some other event takes place. Eg- I/O
![[Screenshot 2025-11-15 at 4.49.06 PM.png|400]]
##### Zombie Process
- When a child process finishes, it becomes a zombie process until the parent process reads its exit status by calling wait().
- If the parent exits without calling wait(), It is adopted by the init process(Parent of all processes).

---
layout: post
title: "Notes: Operating Systems - Three Easy Pieces  "
date:   2020-03-09
author: Faushine
tags: 
- Operating system
---



# CPU

### Problem 1: How to provide the illusion of many CPUs?

**Answer:** Run one process and then stop it to run another one, which called time sharing

### Process API:

**Create**: which is used to create new process when you click the app or type a command into the shell.

**Destroy**: which can halt a running process as needed.

**Wait**: which waits for a process to stop running.

**Miscellaneous Control**: other than killing or waiting, some kind of operation like suspending a process for a while then resuming it.

**Status**: records how long the process has run and what states is in.


### Problem 2: How program are transformed into process?

**Answer**:

 - Load its code and any static data into memory. (from disk to memory cause program reside on disk)
 - Allocated memory for run-time stack which stores variables, function parameters and return addresses.
 - Allocate memory for heap which is used for data structures such as linked list, hash table, trees.
 - Doing other works related to I/O setup.
 - Start the program from the entry point, namely main().
 - OS transfer control of CPU to this process and then it begins execution.


### Process states

Running 
Ready
Blocked

What we know: 

 1. OS has to decide to run a process when the current process is issued an I/O. 
 2. If OS decide not to switch to process it runs before, which is not clear if it is a good decision. Such type of decisions are made by OS scheduler.

### Data structure

OS is a program which has some key data structures to track relevant information. To track the state of process, OS will keep some lists of process to tell him which process is running, ready or blocked. The structure is uesd to store information of process called PCB process control block.


### Process APIs

Question: How to create and control processes?

**Fork()**: create a new process

**Wait()**: parent process calls wait() to delay its execution until the child finish executing.

**Exec()**: run a program which is different from the calling program.

### Why would we build such interfaces to what should be the simple act of creating a new process?

Answer:
Think about that the shell is a program. It shows you a prompt and waits you to type something to run. When it recieves you command, it calls fork() to create a new process to execute it.

### Process Control And Users

**Kill():** Sends a signal to a process by kill() system call which can do pause, die and other imperatives to a process.

Keystrokes:

 - Ctrl+z: sends a stop signal which pauses the process for a while which means you can resume it after that.
 - Ctrl+c: sends a interrupt to terminate the process 

Process could use signal() to catch various signals. Now we think about the another question: who can send a signal to a process, and who cannot? OS provides a notion of user to parcel out resources including processes.

# Mechanism: Limited Direct Execution 

Recall the previous chapter, what virtualization achieves is to run a process for a while and then stop it to run another one in a time sharing mode. But how could we guarantee the performance in this situation? Something like how to avoid adding excessive overhead to the system. The second question is control: how to run the process efficiently while retaining control over the CPU? 

In this chapter, we deal with this problem: maintaining high performance while keeping control.

### Basic method: Limited Direct Execution

Direct execution means run a program on CPU directly. When a OS wants to run a program it will follow these steps:

 - Create entry for process list
 - Allocate memory for program
 - Load program into memory
 - Set up stack with argc/argv
 - Clear registers
 - Execute call main()

### Problem 1: Restricted operations
	
	When a process wishes to perform some kind of access request to disk or other resources like memory, how could OS achieves that if run process on CPU natively?
	
	One answer is to allow all requests related to IO. It discards permission verify which might not desirable.
	
	The another answer is to introduce a new processor mode: user mode. Code runs in the user mode is restricted in what it can do. In contrast to user mode is kernel mode, code can do what it likes, including privileged operations. The user process performs privileged operations by calling system call (process runs in user mode tells kernel to do what it wants).
	
### Problem 2: Switching between processes
	
	CPU can do nothing when it is running a process. So how to regain control of the CPU?
	
**Approach 1: Cooperative approach**
	
	OS trusts the process to assume it behaves reasonably. The process periodically give up the CPU to avoid runs too long and lets CPU do other tasks. Such operation can be called explicitly by system call yield(). Think about if a process ends up in an infinite loop and never make a system call, what can OS do ?
	
**Approach 2: Non-cooperative approach: the OS takes control**
	
	Use a timer interrupt to raise an interrupt every so many milliseconds and when it happens, the currently running process is halted. Hardware is responsible for saving states of the program to make sure it can be resumed correctly.
	
	
### Saving and Restoring Context
	
    Now that the OS are able to regain the control, a decision has to be made: whether to continue running the currently-running process or switch to another one. If the decision is made to switch, the OS then execute context switch. Mostly OS will execute some assembly code to save registers, PC and the kernel stack pointer of currently-running process.

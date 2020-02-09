# CS 6200 Introduction to Operating Systems

## What is an Operating System

### Shop Manager Metaphor

A shop manager:

- Directs operational resources: use of employee time, parts, tools
- Enforces working policies: fairness, safety, cleanup
- Mitigates difficulty of complex tasks: simplifies operation, optimizes performance

An operating system:

- Directs operational resources: use of CPU, memory, peripheral devices
- Enforces working policies: fair resource access, limits to resource usage
- Mitigates difficulty of complex tasks: abstract hardware details (system calls)

### Attempt do define an operating system

Hardwares (CPU, Memory, GPU, Storage, Network Card) are used by multiple applications. The operating system sits in between the underlying hardwares and the applications. It:

- hides hardware complexity
- manages resource (schedule tasks on CPU, manages memory usage)
- provide isolation and protection (so application do not interfere with each other)

### Operating System Definition

An **operating system** is a layer of systems software that:

- directly has privileged access to the underlying hardware
- hides the hardware complexity (abstraction)
- manages hardware on behalf of one or more applications according to some predefined policies (arbitration)
- ensures that applications are isolated and protected from one another

### Operating Systems Examples

Desktop:

- Microsoft Windows
- Unix-based
  - Mac OSX (BSD)
  - Linux (Ubuntu, Centos, Fedora)

Embedded:

- Android (Linux)
- iOS
- Symbian

Mainframe

### Operating System Elements

- Abstractions:
  - application: process, thread
  - hardware: file, socket, memory page
- Mechanisms:
  - application: create, schedule
  - hardware: open, write, allocate
- Policies (how mechanisms will be used):
  - least-recently used (LRU)
  - earliest deadline first (EDF)

#### Memory management example

- Abstractions: memory page
- Mechanism: allocate, map to a process
- Policies: least recently used (least recently used memory page will be moved to disk, a.k.a swap)

### Design Principles

1. Separation of mechanism and policy

   Implement flexible mechanisms to support many policies, e.g. LRU, LFU, random

2. Optimize for common case

   - Where will the OS be used?
   - What will user want to execute on that machine?
   - What are the workload requirements?

### User/Kernel Protection Boundary

- Kernel level: OS Kernel, privileged mode, direct hardware access
- User level: unprivileged mode, applications

user-kernel switch is supported by hardware:

- trap instructions
- system call: open (file), send (socket), mmap (memory)
- signals

### System Call Flowchart

Flow of synchronous mode: wait until the sys call completes

1. _(user process)_ User process executing
2. _(user process)_ Call system call
3. _(kernel)_ Trap
4. _(kernel)_ Execute system call
5. _(kernel)_ Return
6. _(user process)_ Return from system call

To make a system call an application must:

- write arguments
- save relevant data at well-defined location
- make system call

### Crossing the User/Kernel Protection Boundary

User/Kernel Transitions

- hardware supported: e.g. traps on illegal instructions or memory accesses requiring special privilege
- involves a number of instructions: e.g. 50-100ns on a 2Ghz machine running linux
- switches locality: affects hardware cache!

#### Note on cache

Because context switches will swap the data/addresses currently in cache, the performance of applications can benefit or suffer based on how a context switch changes what is in cache at the time they are accessing it.

A cache would be considered hot if an application is accessing the cache when it contains the data/addresses it needs.

Likewise, a cache would be considered cold if an application is accessing the cache when it does not contain the data/addresses it needs -- forcing it to retrieve data/addresses from main memory.

### Basic OS Services

- Scheduler: access to CPU
- Memory manager
- Block device driver: e.g. disk
- File system

Windows vs Unix

- Process Control:
  - Windows: CreateProcess, ExitProcess, WaitForSingleObject
  - Unix: fork, exit, wait
- File Manipulation
  - Windows: CreateFile, ReadFile, WriteFile, CloseHandle
  - Unix: open, read, write, close
- Device Manipulation
  - Windows: SetConsoleMode, ReadConsole, WriteConsole
  - Unix: ioctl, read, write
- Information Maintenance
  - Windows: GetCurrentProcessID, SetTime, Sleep
  - Unix: getpid, alarm, sleep
- Communication
  - Windows: CreatePipe, CreateFileMapping, MapViewofFile
  - Unix: pipe, shmget, mmap
- Protection
  - Windows: SetFileSecurity, InitializeSecurityDescriptor, SetSecurityDescriptorGroup
  - Unix: chmod, umask, chown

[Linux System Calls Table](https://thevivekpandey.github.io/posts/2017-09-25-linux-system-calls.html)

### Monolithic OS

Historically, OS had a monolithic design. Every possible service any application can require or hardware will make are already included.

- **+** everything included
- **+** inlining, compile-time optimization
- **-** customization, portability, manageability
- **-** memory footprint
- **-** performance

### Modular OS

e.g. Linux

OS specifies interface for any module to be included in the OS. Modules can be installed dynamically.

- **+** maintainability
- **+** smaller footprint
- **+** less resource needs
- **-** indirection can impact performance
- **-** maintenance can still be an issue

### Microkernel

Only require the most basic primitives at OS level. File system, disk driver, etc will run at unprivileged application level.

- **+** size
- **+** verifiability
- **-** portability
- **-** complexity of software development
- **-** cost of user/kernel crossing (more frequent)

## Processes and Process Management

Simple Process Definition:

- Instance of an executing program
- Synonymous with "task" or "job"

### Toy Shop Metaphor

An order of toys:

- state of execution: completed toys, waiting to be built
- parts and temporary holding area: plastic pieces, containers
- may require special hardware: sewing machine, glue gun

A process:

- state of execution: program counter, stack
- parts and temporary holding area: data, register state occupies state in memory
- may require special hardware: I/O devices

### What is a process

OS manages hardware on behalf of applications. Applications are programs on disk, flash memory, etc. They are static entities. Once an application is loaded, it becomes an active entity and is called a process. Process represents the execution state of an active entity.

### What does a process look like

Process encapsulate all states from an application, e.g. text, data, heap, stack. Every state has to be uniquely identified by its address. Address is defined as a range of address from $V_0$ to $V_\text{max}$.

Types of state:

- text and data: static state when process first loads
- heap: dynamically created during execution
- stack: grows and shrinks (LIFO queue)

What have been said above regarding the encapsulation can be called the **address space**. It is the "in memory" representation of a process. We call $V_0$ to $V_\text{max}$ used by the process virtual addresses, in contrast with physical addresses which correspond to locations in physical memory (e.g. DRAM). A Page table created by the OS maintains a mapping of virtual to physical addresses.

32bits: Virtual address space up to 4GB.

Some times applications can share the same physical addresses, and additional states will be swapped in and out of the disk.

### How does the OS know what a process is doing

OS has to have idea of what a process is doing, so it can stop and restart the process at the same point.

- Program Counter: where in the instruction sequence the current is with
- CPU register: maintains program counter while the process is runner
- Stack pointer: where the top of the stack is. It's important because stack is LIFO.

### What is a Process Control Block (PCB)

A data structure that the operating system maintains for every one of the processes that it manages.

- process state, process number, program counter, registers, memory limits, list of open files, priority, signal mask, CPU scheduling info
- PCB created when process is created
- Certain fields are updated when process state changes
- Other fields change too frequently (e.g. registers)

### What is a context switch

A context switch is the mechanism used by the operating system to switch the CPU from the context of one process to the context of another.

They are expensive:

- direct costs: number of cycles for load and store instructions
- indirect costs: cold cache! cache misses

### Process Lifecycle

States of a process:

- New: when a process is created. OS will perform admission control, and create PCB.
- Ready: process is admitted, and ready to start executing. It isn't actually running on the CPU and has to wait until the scheduler is ready to move it into a running state when it schedules it on the CPU.
- Running: ready state process is dispatched by the scheduler. It can be interuppted so that a context switch can be performed. This will move running process back into the ready state.
- Waiting: when a longer operation is initiated (e.g. reading data from disk, waiting for keyboard input), process enters waiting state. When the operation is complete, it will be moved back to ready state.
- Terminated: when all instructions are complete or errors are encountered, process will exit.

### Process creation

Mechanism for process creation

- fork: copies the parent PCB into new child PCB. Child continues execution at instruction after fork.
- EXEC: replace child image. Load new program and start from first instruction.

Note:
The parent of all processes: `init`

### What is the role of the CPU scheduler

A CPU scheduler determines which one of the currently ready processes will be dispatched to the CPU to start running, and how long it should run for.

OS must be able to do the following tasks efficiently:

- preempt: interrupt and save current context
- schedule: run scheduler to choose next process
- dispatch: dispatch process and switch into its context

### How long should a process run for/ How frequently should we run the scheduler

Longer the process runs means less often we are invoking the task scheduler. Consider the following case:

- running process for amount of time $T_p$
- schedular takes amount of $T_\text{sched}$

$$\text{Useful CPU Work} = \frac{\text{Total Processing Time}}{\text{Total time}} = \frac{T_p}{T_p + T_\text{sched}}$$

### Can Processes Interact

An operating system must provide mechanisms to allow processes to interact with each other. Many applications we see today are structured as multiple processes.

Inter Process Communication (IPC) mechanisms:

- transfer data/info between address spaces
- maintain protection and isolation
- provide flexibility and performance

#### Message-passing IPC

- OS provides communication channel, like shared buffer
- Processes write/read messages to.from channel
- **+** OS manages
- **-** overheads: everything needs to go through the OS

#### Shared memory IPC

- OS establishes a shared channel and maps it into each process address space
- Processes directly read/write from this memory
- **+**: OS is out of the way
- **-**: No shared and well defined API because OS is not involved. Can be more error-prone, and developers may need to re-implement code. Mapping memory between processes can also be expensive.

## Threads and Concurrency

### Toy shop worker metaphor

A worker in a toy shop:

- is an active entity: executing unit of toy order
- works simultaneously with others: many workers completing toy orders
- requires coordination: sharing of tools, parts, workstations

A thread:

- is an active entity: executing unit of a process
- works simultaneously with others: many threads executing
- requires coordination: sharing of I/O devices, CPUs, memory

### Process vs Thread

Threads are part of the same virtual address space. Each thread will have a separate program counter, registers, and stack pointer.

### Why are threads useful

- Each thread can execute the same code but for different portions of the input. Parallelization helps to speed up processes.
- Threads can execute different parts of the program.
- Specialization with multiple cores: hotter cache.
- Efficiency: lower memory requirement and cheaper IPC.

### Are threads useful on a single CPU (or # of threads > # of CPUs)

- If $t_\text{idle} > 2 t_\text{context switch}$, then context switch to hide idling time.
- When switching between threads, it is not necessary to recreate virtual to physical memory maps (the most costly step). Therefore, $t_\text{thread context switch} << t_\text{process context switch}$.
- Multithreading is specially useful to high latency in IO operations.

### Benefits to applications and OS code

Multithreaded OS kernel

- threads working on behalf of apps
- OS-level services like daemons or device drivers

### What do we need to support threads?

- thread data structure: identify threads, keep track of resource usage
- mechanisms to create and manage threads
- mechanisms to safely coordinate among threads running concurrently in the same address space

### Threads and concurrency

Processes: Operating in own address spaces. Opearting system prevents overwriting physical addresses.

Threads: They both use the same virtual and physical addresses. Multiple threads can attempt to access the same data at the same time.

### Concurrency Control and Coordination

Synchronization Mechanisms

- Mutual Exclusion
  - exclusive access to only one thread at a time
  - mutex
- Waiting on other threads
  - specific condition before proceeding
  - condition variable
- Waking up other threads from wait state.

### Threads and Thread Creation

- Thread type: thread data structure (threadId, PC, SP, registers, stack, attributes)
- `Fork(proc, args)`: create a thread, not UNIX fork. `t1 = fork(proc, args)` creates a new thread.
- `Join(thread)`: terminate a thread.

### Thread Creation Example

```c
Thread thread1;
Shared_list list;
thread1 = fork(safe_insert, 4);
safe_insert(6);
join(thread1); // Optional, blocks parent thread
```

Result can be:

- 4 $\rightarrow$ 6 $\rightarrow$ NIL
- 6 $\rightarrow$ 4 $\rightarrow$ NIL

How is the list updated?

```
create new list element e
set e.value = X
read list and list.p_next
set e.point = list.p_next
set list.p_next = e
```

### Mutual Exclusion

Mutex data structure:

- whether it's locked
- owner of mutex
- blocked threads

Birrell's Lock API

```c
lock(m) {
  // critical section
} // unlock;
```

Common Thread API

```c
lock(m);
// critical section
unlock(m);
```

### Making safe_insert safe

```c
list<int> my_list;
Mutex m;
void safe_insert(int i) {
  lock(m) {
    my_list.insert(i);
  } // unlock;
}
```

### Producer/Consumer Example

What if the processing you wish to perform with mutual exclusion needs to occur only under certain conditions?

```c
//main
for i=0..10
   producers[i] = fork(safe_insert, NULL) //create producers
consumer = fork(print_and_clear, my_list) //create consumer

// producers: safe_insert
lock(m) {
  list->insert(my_thread_id)
} // unlock;

// consumer: print_and_clear
lock(m) {
  if my_list.full -> print; clear up to limit of elements of list (WASTEFUL!)
  else -> release lock and try again later
} // unlock;
```

### Condition Variable

```c
// consumer: print and clear
lock(m) {
  while (my_list.not_full()) Wait(m, list_full);
  my_list.print_and_remove_all();
}

// producers: safe_insert
lock(m) {
  my_list.insert(my_thread_id);
  if my_list.full()
    Signal(list_full)
}

```

### Condition Variable API

- Wait(mutex, cond): mutex is automatically released and re-acquired on wait
- Signal(cond): notify only one thread waiting on condition
- Broadcast(cond): notify all waiting threads

### Reader/Writer Problem

Readers: 0 or more can access
Writer: 0 or 1 can access

```python
if read_counter == 0 and write_counter == 0:
  read okay, write okay
if read_counter > 0:
  read okay
if write_counter > 0:
  read no, write no
```

State of shared file/resource:

- free: resource.counter = 0
- reading: resource.counter > 0
- writing: resource.counter = -1

### Reader/Writer Example

```c
Mutex counter_mutex;
Condition read_phase, write_phase;
int resource_counter = 0;
```

READERS

```c
Lock(counter_mutex) {
  while (resource_counter == -1)
    Wait(counter_mutex, read_phase);
  resource_counter++;
} // unlock;

// ... read data ...

Lock(counter_mutex) {
  resource_counter--;
  if (resource_counter == 0)
    Signal(write_phase)
} // unlock;
```

WRITER

```c
Lock(counter_mutex) {
  while (resource_counter != 0)
    Wait(counter_mutex, write_phase);
  resource_counter = -1;
} // unlock;

// ... write data ...

Lock(counter_mutex) {
  resource_counter = 0;
  Broadcast(read_phase);
  Broadcast(write_phase);
}
```

### Critical Sections in Readers/Writer Example

![Critical Section](img/P2L2.23.png)

### Critical Section Structure with Proxy Variable

![Critical Section Proxy Variable](img/P2L2.24.png)

### Avoiding Common Mistakes

- Keep track of mutex/conditional variables used with a resource
  - e.g. mutex_type m1; //mutex for file 1
- Check that you are always (and correctly) using lock and unlock
  - e.g. did you forget to lock/unlock? What about compilers?
- Use a single mutex to access a single resource!
  - e.g. read and write of the same file has 2 mutexs
- Check that you are signaling correct condition
- Check that you are not using signal when broadcast is needed
  - signal: only 1 thread will proceed... remaining threads will continue to wait... possibly indefinitely!
- Ask yourself: do you need priority guarantees?
  - thread execution order not controlled by signals to condition variables!

### Spurious Wake Ups

![Spurious Wake-Ups](img/P2L2.26.png)
![Unlock Before Broadcast](img/P2L2.26.2.png)

### Deadlocks

Definition:

> Two or more competing threads are waiting on each other to complete, but none of them ever do.

To avoid deadlocks: Maintain lock order across threads, so there is no dead cycles.
![Deadlocks](img/P2L2.28.png)

In summary:

> A cycle in the wait graph is necessary and sufficient for a deadlock to occur. (edges from thread waiting on a resource to thread owning a resource)

What can we do about it?

- Deadlock prevention (EXPENSIVE)
- Deadlock detection & recovery (ROLLBACK)
- Apply the ostrich Algorithm (DO NOTHING!) => if all else fails... just REBOOT

### Kernel vs User-Level Threads

#### One to One Model

![One to One](img/P2L2.32.png)

- **+** OS sees/understands threads, synchronization, blocking
- **-**
  - Must go to OS for all operations (may be expensive)
  - OS may have limits on policies, thread #
  - portability

#### Many to One model

![Many to One](img/P2L2.32.1.png)

- **+** totally portable, doesn't depend on OS limits and policies
- **-**
  - OS has no insights into application needs
  - OS may block entire process if one user-level thread blocks on I/O

#### Many to Many Model

![Many to Many](img/P2L2.32.2.png)

- **+**
  - can be best of both worlds
  - can have bound or unbound threads
- **-**
  - requires coordination between user and kernel level thread managers

#### Scope

- System Scope: System-wide thread management by OS-level thread managers (e.g. CPU scheduler)
- Process Scope: User-level library manages threads within a single process

### Multithreading Patterns

Toy shop example: for each wooden toy order, we:

1. accept the order
2. parse the order
3. cut wooden parts
4. paint and add decorations
5. assemble the wooden toys
6. ship the order

#### Boss/Workers Pattern

![Boss/Workers](img/P2L2.35.png)

Boss-Workers:

- boss: assigns work to workers (step 1)
- workers: performs entire task (step 2-6)

Throughput of the system limited by boss thread => must keep boss efficient
$$\text{Throughput} = \frac{1}{\text{boss time per order}}$$

**Boss assigns work by**

- directly signalling specific worker
  - **+** workders don't need to synchronize
  - **-** boss must track what each worker is doing, and throughput will go down
- placing work in producer/consumer queue (more often used)
  - **+** boss doesn't need to know details about workers
  - **-** queue synchronization: workers need to synchronize access to the queue, etc

**How many workers**

- on demand
- pool of workers
  - static or dynamic

Boss-Workers:

- boss: assigns work to workers (step 1)
- workers: performs entire task (step 2-6)
- placing work in producer/consumer queue (more often used)
- pool of workers
- **+** simplicity
- **-** thread pool management
- **-** locality

Boss-Workers Variants:

- all workders created equal vs workers specialized for certain tasks
- **+** better locality (hotter cache); quality of service management
- **-** load balancing

#### Pipeline Pattern

![Pipeline Patter](img/P2L2.38.png)

Pipeline:

- threads assigned one subtask in the system
- entire tasks == pipeline of threads
- multiple tasks concurrently in the system, in different pipeline stages
- throughput == weakest link => pipeline stage == thread pool
- shared-buffer based communication between stages

In summary:

- sequence of stages
- stage == subtask
- each stage == thread pool
- buffer-based communication
- **+** specialization and locality
- **-** balancing and synchronization overheads

#### Layered Pattern

![Layered Pattern](img/P2L2.39.png)

Layered:

- each layer group of related subtasks
- end-to-end task must pass up and down through all layers
- **+** specialization and locality, but less fine-grained than pipeline
- **-** not suitable for all applications
- **-** synchronization

### Time to finish orders

![Process Time](img/P2L2.40.png)

Boss-Worker:

$$ \text{Time to Finish 1 Order} \times \text{Ceiling}(\text{num orders}/\text{num threads}) $$

Pipeline:

$$ \text{time to finish first order} + (\text{remaining orders} \times \text{time to finish last stage})$$
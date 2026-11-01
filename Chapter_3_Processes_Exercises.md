---
course: Operating Systems
title: Chapter 3 -- Processes
type: Exercise Report
---

# Chapter 3: Processes

## Exercise 3.12

**Question:** Describe the actions taken by a kernel to context-switch
between processes.

**Answer:**

A context switch occurs when the CPU stops executing one process and
begins executing another. The kernel performs the following actions:

1.  Saves the current process's CPU state, including its registers and
    program counter.
2.  Stores the saved information in the current process's Process
    Control Block (PCB).
3.  Updates the current process's state, such as changing it from
    running to ready or waiting.
4.  Selects another process from the ready queue using the CPU
    scheduler.
5.  Loads the selected process's saved CPU state from its PCB.
6.  Updates memory-management information if necessary.
7.  Resumes execution of the selected process.

This allows multiple processes to share the CPU.

## Exercise 3.13

**Question:** Construct a process tree similar to Figure 3.7. To obtain
process information for a UNIX or Linux system, use `ps -ael`.

**Answer:**

On Linux, the command `ps -ael` displays detailed process information,
including process IDs and parent process IDs. The command `pstree -p`
can also display the process hierarchy in a tree format.

Example process tree:

``` text
systemd (PID 1)
├── sshd
│   └── sshd
│       └── bash
│           └── pstree
├── cron
├── NetworkManager
└── systemd-logind
```

The root process in this example is `systemd` (PID 1). Processes can
create child processes, and each process has a parent process identified
by its Parent Process ID (PPID). The actual process tree depends on the
Linux system.

## Exercise 3.14

**Question:** Explain the role of the init (or systemd) process on UNIX
and Linux systems in regard to process termination.

**Answer:**

The `init` or `systemd` process adopts orphaned processes. When a parent
process terminates while its child processes are still running, those
child processes become orphan processes and are adopted by `init` or
`systemd`.

The init or systemd process also collects the exit status of terminated
adopted processes. This helps prevent terminated processes from
remaining as unreaped zombies.

## Exercise 3.15

**Question:** Including the initial parent process, how many processes
are created by the program shown in Figure 3.1?

**Answer:**

The program executes `fork()` four times:

``` c
for (i = 0; i < 4; i++)
    fork();
```

Each call to `fork()` duplicates every process that currently exists.
Therefore, the number of processes doubles during each iteration.

  Stage                         Number of processes
  --------------------------- ---------------------
  Initial process                                 1
  After the first `fork()`                        2
  After the second `fork()`                       4
  After the third `fork()`                        8
  After the fourth `fork()`                      16

The total number of processes is:

\[ 2\^n = 2\^4 = 16 \]

**Final answer:** 16 processes, including the initial parent process.
Therefore, 15 child processes are created.

## Exercise 3.16

**Question:** Explain the circumstances under which the line of code
marked `printf("LINE J")` in Figure 3.2 will be reached.

**Answer:**

The `printf("LINE J")` statement is located immediately after the
`execlp()` function.

The `execlp()` function replaces the current process image with the
specified program, `/bin/ls`.

There are two possible situations:

1.  If `execlp()` succeeds, the child process is replaced by the
    `/bin/ls` program. The next statement, `printf("LINE J")`, is not
    executed.
2.  If `execlp()` fails, the function returns to the original program,
    so `printf("LINE J")` is executed.

**Final answer:** LINE J is reached only if the `execlp()` function
fails.

## Exercise 3.17

**Question:** Using the program in Figure 3.3, identify the values of
`pid` at lines A, B, C, and D. Assume that the actual PIDs of the parent
and child are 2600 and 2603, respectively.

**Answer:**

The `fork()` function returns different values depending on which
process receives the return value:

-   The child process receives `0`.
-   The parent process receives the PID of the child.
-   The `getpid()` function returns the PID of the process that calls
    it.

  Line   Statement                                Value
  ------ -------------------------------------- -------
  A      `printf("child: pid = %d", pid);`            0
  B      `printf("child: pid1 = %d", pid1);`       2603
  C      `printf("parent: pid = %d", pid);`        2603
  D      `printf("parent: pid1 = %d", pid1);`      2600

**Final answers:**

-   A = 0
-   B = 2603
-   C = 2603
-   D = 2600

## Exercise 3.18

**Question:** Give an example of a situation in which ordinary pipes are
more suitable than named pipes and an example of a situation in which
named pipes are more suitable than ordinary pipes.

**Answer:**

### Ordinary Pipes (Unnamed Pipes)

Ordinary pipes are more suitable for communication between related
processes, such as a parent process and its child process.

**Example:** A parent process sends data to its child process after
creating it using `fork()`.

**Advantages:** - Simple to implement. - Suitable for communication
between related processes. - Does not require a separate named pipe in
the file system.

### Named Pipes (FIFOs)

Named pipes are more suitable for communication between unrelated
processes.

**Example:** Two independent applications running on the same computer
communicate by sending and receiving data through a named pipe.

**Advantages:** - Allow communication between unrelated processes. -
Have a name in the file system. - Can be accessed by processes with the
appropriate permissions.

**Final answer:** Ordinary pipes are suitable for communication between
related processes, while named pipes are suitable for communication
between unrelated processes.

## Exercise 3.19

**Question:** Consider the RPC mechanism. Describe the undesirable
consequences that could arise from not enforcing either the "at most
once" or "exactly once" semantic. Describe possible uses for a mechanism
that has neither of these guarantees.

**Answer:**

Remote Procedure Call (RPC) allows a program to execute a procedure on
another computer or remote system.

### At-Most-Once Semantics

The procedure is executed zero or one time.

-   Prevents duplicate execution.
-   A request might not be executed if a communication failure occurs.
-   The client might not receive a response even if the server has
    processed the request.

### Exactly-Once Semantics

The procedure is executed exactly one time.

-   Ensures that the requested operation is completed once.
-   Prevents duplicate execution.
-   Is difficult to guarantee because network failures can occur between
    execution and response delivery.

### Consequences of Having Neither Guarantee

-   A procedure may execute multiple times.
-   A procedure may never execute.
-   Duplicate operations can cause incorrect results.
-   Requests or responses can be lost during communication failures.

### Possible Uses

A mechanism without either guarantee can be used for operations where
occasional lost or duplicated requests are acceptable, such as sending
periodic status updates, refreshing information, or transmitting
non-critical monitoring data.

## Exercise 3.20

**Question:** Using the program shown in Figure 3.4, explain what the
output will be at lines X and Y.

**Answer:**

The program initializes the array as follows:

``` c
#define SIZE 5

int nums[SIZE] = {0, 1, 2, 3, 4};
```

The child process executes:

``` c
nums[i] *= -i;
```

### Line X: Child Process

The child modifies each array element using the value of `i`.

    i   Original value Calculation     Result
  --- ---------------- ------------- --------
    0                0 0 × 0                0
    1                1 1 × -1              -1
    2                2 2 × -2              -4
    3                3 3 × -3              -9
    4                4 4 × -4             -16

**Output at Line X:**

``` text
CHILD: 0
CHILD: -1
CHILD: -4
CHILD: -9
CHILD: -16
```

### Line Y: Parent Process

The parent executes `wait(NULL)` before printing its array. Although the
child modifies its own array, the parent retains its original array
because `fork()` gives the child a separate process address space.

**Output at Line Y:**

``` text
PARENT: 0
PARENT: 1
PARENT: 2
PARENT: 3
PARENT: 4
```

**Final answer:** The child prints the modified array values, while the
parent prints the original values because the two processes have
separate address spaces.

## Exercise 3.21

**Question:** What are the benefits and disadvantages of each of the
following? Consider both the system level and the programmer level.

### A. Synchronous and Asynchronous Communication

  -----------------------------------------------------------------------
  Type                    Benefits                Disadvantages
  ----------------------- ----------------------- -----------------------
  Synchronous             Simple to understand    The sender may have to
                          and coordinate. Sender  wait until the receiver
                          and receiver            is ready, which can
                          communicate in an       reduce performance.
                          organized manner.       

  Asynchronous            Allows processes to     More difficult to
                          continue executing      coordinate and manage.
                          without waiting for an  Requires additional
                          immediate response.     mechanisms for handling
                          Improves concurrency.   messages.
  -----------------------------------------------------------------------

**Explanation:** Synchronous communication requires coordination between
the sender and receiver. Asynchronous communication allows the sender to
continue working while the receiver processes the message later.

### B. Automatic and Explicit Buffering

  -----------------------------------------------------------------------
  Type                    Benefits                Disadvantages
  ----------------------- ----------------------- -----------------------
  Automatic Buffering     The operating system    Provides less control
                          manages message         over memory usage and
                          storage, making         buffer management.
                          communication easier    
                          for programmers.        

  Explicit Buffering      Programmers can control Requires additional
                          buffer allocation and   programming effort and
                          memory usage.           proper buffer
                                                  management.
  -----------------------------------------------------------------------

**Explanation:** Automatic buffering allows the system to handle message
storage, while explicit buffering requires the programmer to manage the
buffers.

### C. Send by Copy and Send by Reference

  -----------------------------------------------------------------------
  Type                    Benefits                Disadvantages
  ----------------------- ----------------------- -----------------------
  Send by Copy            The receiver gets an    Requires additional
                          independent copy of the memory and copying
                          data, preventing        time.
                          unintended changes to   
                          the original data.      

  Send by Reference       Avoids copying large    Can introduce
                          amounts of data,        synchronization
                          improving performance   problems and risks when
                          and reducing memory     multiple processes
                          usage.                  access shared data.
  -----------------------------------------------------------------------

**Explanation:** Send by copy transfers an actual copy of the message,
while send by reference transfers a reference to the original data.

### D. Fixed-Sized and Variable-Sized Messages

  -----------------------------------------------------------------------
  Type                    Benefits                Disadvantages
  ----------------------- ----------------------- -----------------------
  Fixed-Sized Messages    Easier to manage,       Can waste memory when
                          allocate memory for,    messages contain less
                          and process.            data than the allocated
                                                  size.

  Variable-Sized Messages Uses memory more        More complicated to
                          efficiently because     manage and requires
                          messages can have       additional memory
                          different lengths.      allocation and
                                                  message-length
                                                  handling.
  -----------------------------------------------------------------------

**Explanation:** Fixed-sized messages always have the same length, while
variable-sized messages can have different lengths depending on the
information being transmitted.

------------------------------------------------------------------------

## Summary of Answers

  -----------------------------------------------------------------------
  Exercise                            Main Answer
  ----------------------------------- -----------------------------------
  3.12                                The kernel saves and restores
                                      process state during a context
                                      switch.

  3.13                                Use `ps -ael` or `pstree -p` to
                                      display the process hierarchy.

  3.14                                Init/systemd adopts orphaned
                                      processes and collects the exit
                                      status of terminated adopted
                                      processes.

  3.15                                16 total processes, including the
                                      initial parent.

  3.16                                LINE J executes only if `execlp()`
                                      fails.

  3.17                                A = 0, B = 2603, C = 2603, D =
                                      2600.

  3.18                                Ordinary pipes are suitable for
                                      related processes; named pipes are
                                      suitable for unrelated processes.

  3.19                                Without execution guarantees, RPC
                                      requests can be lost or duplicated.

  3.20                                Line X: `0, -1, -4, -9, -16`; Line
                                      Y: `0, 1, 2, 3, 4`.

  3.21                                Compare the benefits and
                                      disadvantages of the four
                                      communication methods.
  -----------------------------------------------------------------------

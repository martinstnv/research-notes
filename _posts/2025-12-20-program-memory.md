---
layout: single
title: Program Memory
date: 2025-12-06
classes: wide
tags:
  - Stack
  - Heap
---

> Operating systems use virtual memory to allow processes to work as if they have the full addressable space, even if physical RAM is smaller.
- [Memory Addressing: Physical and Virtual Memory](/memory-addressing/#physical-and-virtual-memory)

The following diagram illustrates a typical program memory layout on a 32-bit system, with virtual addresses spanning from `0x00000000` to `0xFFFFFFFF`.

```
(High Address)
0xFFFFFFFF  ->  |-----------------------------|
                |   Command-line arguments    |
                |  and environment variables  |
                |-----------------------------|
                |            Stack            | 
                |.............................|
                |                             |
                |.............................|
                |                             |
                |            Heap             | 
                |                             |
                |-----------------------------|
                |     Data Segment (BSS)      |
                |       (Uninitialised)       | 
                |-----------------------------|
                |         Data Segment        |
                |         (Initialised)       | 
                |-----------------------------|
                |         Text Segment        |
                |        (Program Code)       |
0x00000000  ->  |-----------------------------|
(Low Address)
```

At the bottom of the address space lies the Text Segment, also called the Code Segment, which stores the program’s instructions.

Directly above the Text segment is the Data segment, which is divided into two areas:
	1.	Initialized data - where variables with predefined values are stored.
	2.	Uninitialized data (BSS) - where variables without initial values reside.

> Note: Global variables that are not explicitly initialized by the program are automatically set to 0 by the process model. This behavior does not apply to local variables.

All contents of the Text and Data segments are determined at compile time, so the compiler knows their locations and encodes this information in the executable.

At the very top of the memory space are the command-line arguments and environment variables, which are set when the process starts. Immediately below them is the Stack, which holds local variables as well as metadata used for function calls and returns.

Between the Data segment and the Stack lies the Heap, the area managed by dynamic memory functions.

Unlike the Text and Data segments, the contents of the Stack and Heap are determined at runtime, and their organization and growth depend on the program’s execution.

As the program requires more memory for the Heap, it expands upward toward higher memory addresses, whereas the Stack grows downward toward lower addresses when more space is needed.

During execution, the program keeps track of the Stack Pointer, which marks the current top of the Stack.

```
0xFFFFFFFF  |-----------------------------|
            |             ...             |
            |-----------------------------|
            |            Stack            | 
            |.............................| <- Stack Pointer
            |                             |
            |                             |
            |                             |
            |.............................|
            |            Heap             | 
            |-----------------------------|
            |             ...             |
            |-----------------------------|
            |        section .text        |
            |        global _start        |
            |                             |
            |        _start:              |
            |            ...              |
0x00000000  |-----------------------------|
```

When a push instruction is executed, the value is placed onto the Stack, and the Stack Pointer is updated accordingly.

```
0xFFFFFFFF  |-----------------------------|                      0xFFFFFFFF  |-----------------------------|
            |             ...             |                                  |             ...             |
            |-----------------------------|                                  |-----------------------------|
            |            Stack            |                                  |            Stack            |
            |.............................|                                  |.............................|
            |            1337             |                                  |            1337             |                    
            |.............................| <- Stack Pointer                 |.............................|
            |                             |                                  |            7331             | 
            |                             |                                  |.............................| <- Stack Pointer
            |                             |                                  |                             |
            |                             |                                  |                             |
            |                             |                                  |                             |
            |.............................|                                  |.............................|
            |            Heap             |                                  |            Heap             |
            |-----------------------------|                                  |-----------------------------|
            |             ...             |                                  |             ...             |
            |-----------------------------|                                  |-----------------------------|
            |        section .text        |                                  |        section .text        |
            |        global _start        |                                  |        global _start        |
            |                             |                                  |                             |
            |        _start:              |                                  |        _start:              |
            |            push 0x539;      |                                  |            push 0x539;      |
            |            ...              |                                  |            push 0x1CB3;     |
            |            ...              |                                  |            ...              |
0x00000000  |-----------------------------|                                  |-----------------------------|
```

In addition to storing local variables, the stack is used to manage function calls and returns, keeping track of return addresses and other bookkeeping information.

Now, let’s examine the basic stack layout for the following function.

```
void func(char *arg1, int arg2, int arg3)
{
    char loc1[4];
    int loc2;
    loc2++;
}
```

The function receives three arguments and contains two local variables. The diagram below illustrates the process memory, beginning with the caller’s data, that is, the data belonging to the function that invoked this one.

```
0xFFFFFFFF  |-----------------------------|
            |             ...             |
            |-----------------------------|
            |        Caller's locals      | 
            |.............................| <- Stack Pointer
            |                             |
            |                             |
            |                             |
```

When a CALL instruction is executed, the function arguments are first pushed onto the stack in reverse order, followed by the return address, which indicates the instruction to execute once the called function completes.


```
0xFFFFFFFF  |-----------------------------|
            |             ...             |
            |-----------------------------|
            |        Caller's locals      | 
            |.............................|
            |             arg3            |
            |.............................|
            |             arg2            |
            |.............................|
            |             arg1            |
            |.............................|
            |        Return Address       |
            |.............................| <- ESP
            |                             |
            |                             |
```

To ensure the function can return correctly to its caller and restore the caller’s Frame Pointer, the callee saves the previous EBP value onto the stack.

For the compiler to accurately access a variable within a function, it needs to know the variable’s fixed offset relative to the Frame Pointer, usually stored in the EBP register. This is why the Frame Pointer is set to the current stack pointer’s value, and then all local variables are pushed onto the stack in the order they appear in the program.

As a result, no matter when or from where the function is invoked, the compiler can reliably determine that a local variable, such as loc2, will always be located at a fixed offset (e.g., 8 bytes) from the Frame Pointer.

```
0xFFFFFFFF  |-----------------------------|
            |             ...             |
            |-----------------------------|
            |        Caller's locals      | 
            |.............................| <- start of stack frame of the called function
            |             arg3            |
            |.............................|
            |             arg2            |
            |.............................|
            |             arg1            |
            |.............................|
            |        Return Address       |
            |.............................|
            |    Caller's Frame Pointer   |
            |.............................| <- EBP
            |             loc1            |
            |.............................|
            |             loc2            |
            |.............................| <- ESP
            |                             |
            |                             |
```

When a called function completes execution, it uses the stored return address to continue execution in the caller.

> In summary, a function call works as follows: the caller first pushes the function arguments onto the stack in reverse order, after which the CALL instruction pushes the return address. Within the called function, the previous Frame Pointer is saved on the stack, a new Frame Pointer is established, and space is allocated for local variables. To return, the function restores the previous stack frame by resetting the Frame Pointer and then jumps back to the return address stored on the stack.

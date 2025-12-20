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
- (Memory Addressing: Physical and Virtual Memory)[/memory-addressing/#physical-and-virtual-memory]

The following diagram illustrates a typical program memory layout on a 32-bit system, with virtual addresses spanning from `0x00000000` to `0xFFFFFFFF`.

```
High Address (0xFFFFFFFF)  ->  |-----------------------------|
                               |                             |
                               |   Command line arguments    |
                               |  and environment variables  |
                               |                             |
                               |-----------------------------|
                               |                             |
                               |            Stack            | 
                               |                             |
                               |.............................|
                               |              |              |
                               |              V              |
                               |                             |
                               |                             |
                               |                             |
                               |              ^              |
                               |              |              |
                               |.............................|
                               |                             |
                               |            Heap             | 
                               |                             |
                               |-----------------------------|
                               |                             |
                               |      Data (Uninitialised)   | 
                               |                             |
                               |-----------------------------|
                               |                             |
                               |      Data (Initialised)     | 
                               |                             |
                               |-----------------------------|
                               |                             |
                               |         Text Segment        | 
                               |                             |
Low Address  (0x00000000)  ->  |-----------------------------|
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

```
0x00000000                                                       0xFFFFFFFF
---------------------------------------------------------------------------
|      ...      |   HEAP   |  ->          <-  |   STACK   |      ...      |
---------------------------------------------------------------------------
```

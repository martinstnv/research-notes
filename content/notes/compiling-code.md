---
title: Compiling C++ Code
date: 2023-11-01
tags:
    - Assembly
    - C++
---

> Visualizing simple C functions compiled with GCC 15.2 into x86-64 assembly to analyze stack frame layouts, register usage, argument passing, and control-flow decisions, all interactively explored through the [Compiler Explorer](https://godbolt.org)

## Assignment to Local Variables

Consider the following function, which assigns an integer value to a local variable:

```cpp
int func() {
    int x = 3;
}
```

This code compiles to the following assembly instructions:

```
func():
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 1337
        ud2
```

`push rbp` saves the current value of the base pointer (rbp) onto the stack. This preserves the caller’s frame pointer so it can be restored when the function returns.

`mov rbp, rsp` establishes a new stack frame for the current function by copying the stack pointer (rsp) into the base pointer (rbp). The base pointer provides a stable reference point for accessing local variables and function parameters within the stack frame.

Because the base pointer remains constant throughout the function’s execution, the compiler can reference local variables using fixed offsets relative to rbp. For example, the variable x is stored at a fixed offset from the frame pointer (in this case, 4 bytes below it).

`mov DWORD PTR [rbp-4], 1337` writes the value 1337 to the memory location at rbp - 4. The DWORD qualifier specifies that the operation writes 4 bytes, which corresponds to the size of an int.

`ud2` is an undefined instruction that always triggers an invalid instruction exception. It is used as a trap. Since the function is declared to return an int, a return value is expected in the eax register. However, because the function does not return a value, this results in undefined behavior in C. Instead of allowing execution to continue with an unspecified return value, the compiler emits ud2 so that execution will immediately fault if the instruction is reached.


## Assignment to Function Arguments

Now, consider the following function, which assigns a value to a function argument:

```cpp
int func(int num) {
    num = 1337;
}
```

The compiler generates the following assembly instructions:

```
func(int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     DWORD PTR [rbp-4], 1337
        ud2
```

`mov DWORD PTR [rbp-4], edi` copies the function argument num from the edi register into a local stack slot at `[rbp-4]`.

> Under the System V AMD64 calling convention (used on Linux and macOS), the first integer argument to a function is passed in the edi register. In unoptimized builds, compilers often store such arguments in the stack frame so they can be accessed consistently via fixed offsets from the base pointer.


## Conditional Statements

Next, consider a simple comparison between a local variable and a function argument:

```cpp
int func(int num) {
    int x = 1337;
    if (x < num) {
        x = 7331;
    }
}
```

The corresponding assembly introduces an additional label, .L2, to manage the conditional branch:

```
func(int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-20], edi
        mov     DWORD PTR [rbp-4], 1337
        mov     eax, DWORD PTR [rbp-4]
        cmp     eax, DWORD PTR [rbp-20]
        jge     .L2
        mov     DWORD PTR [rbp-4], 7331
.L2:
        ud2
```

`mov DWORD PTR [rbp-20], edi` copies the function argument num from the `edi` register into memory at `[rbp-20]`.

> The offset of 20 bytes from rbp is chosen to allocate space for all local variables, maintain proper alignment (typically 16-byte alignment on x86-64), and leave room for temporary variables or spilled registers.

`mov eax, DWORD PTR [rbp-4]` loads the value of the local variable x into the eax register. This prepares it for comparison, as the `cmp` instruction operates on registers.

`cmp eax, DWORD PTR [rbp-20]` compares eax (holding x) with the stored argument `num`.

> The CPU sets status flags based on the result, which are then used by the conditional jump to determine control flow.

`jge .L2` jumps to the label `.L2` if the comparison indicates that eax is greater than or equal to the argument.

> In other words, if the condition `x < num` is false, execution skips the body of the `if` statement.
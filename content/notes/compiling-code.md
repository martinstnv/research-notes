---
title: Compiling Native Code
date: 2026-03-05
tags:
    - Assembly
    - C++
---

> Visualizing simple C functions compiled with GCC 15.2 into x86-64 assembly to analyze stack frame layouts, register usage, argument passing, and control-flow decisions, all interactively explored through the [Compiler Explorer](https://godbolt.org)


## Assignment to Local Variables

```cpp
void func() {
    int x = 3;
}
```

```
func():
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 3
        nop
        pop     rbp
        ret
```

`push rbp` saves the current value of the base pointer (rbp) onto the stack. This preserves the caller's frame pointer so it can be restored when the function returns.

`mov rbp, rsp` establishes a new stack frame for the current function by copying the stack pointer (rsp) into the base pointer (rbp). The base pointer provides a stable reference point for accessing local variables and function parameters within the stack frame.

Because the base pointer remains constant throughout the function's execution, the compiler can reference local variables using fixed offsets relative to rbp. For example, the variable x is stored at a fixed offset from the frame pointer (in this case, 4 bytes below it).

`mov DWORD PTR [rbp-4], 1337` writes the value 1337 to the memory location at rbp - 4. 

> The DWORD qualifier specifies that the operation writes 4 bytes, which corresponds to the size of an int.

The `nop` instruction is simply padding inserted by the compiler and does not affect the program's logic.

`pop rbp` restores the previous stack frame pointer.

Finally, `ret` returns control to the calling function.

> Internally it pops the return address from the stack and jumps to that address


## Assignment to Function Arguments

```cpp
int func(int num) {
    num = 1337;
}
```

```
func(int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     DWORD PTR [rbp-4], 1337
        nop
        pop     rbp
        ret
```

`mov DWORD PTR [rbp-4], edi` copies the function argument num from the edi register into a local stack slot at `[rbp-4]`.

> Under the System V AMD64 calling convention (used on Linux and macOS), the first integer argument to a function is passed in the edi register. In unoptimized builds, compilers often store such arguments in the stack frame so they can be accessed consistently via fixed offsets from the base pointer.


## Conditional Statements

```cpp
int func(int num) {
    int x = 1337;
    if (x < num) {
        x = 7331;
    }
}
```

```
func(int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-20], edi
        mov     DWORD PTR [rbp-4], 1337
        mov     eax, DWORD PTR [rbp-4]
        cmp     eax, DWORD PTR [rbp-20]
        jge     .L3
        mov     DWORD PTR [rbp-4], 7331
.L3:
        nop
        pop     rbp
        ret
```

`mov DWORD PTR [rbp-20], edi` copies the function argument num from the `edi` register into memory at `[rbp-20]`.

> The offset of 20 bytes from rbp is chosen to allocate space for all local variables, maintain proper alignment (typically 16-byte alignment on x86-64), and leave room for temporary variables or spilled registers.

`mov eax, DWORD PTR [rbp-4]` loads the value of the local variable x into the eax register. This prepares it for comparison, as the `cmp` instruction operates on registers.

`cmp eax, DWORD PTR [rbp-20]` compares eax (holding x) with the stored argument `num`.

> The CPU sets status flags based on the result, which are then used by the conditional jump to determine control flow.

`jge .L2` jumps to the label `.L2` if the comparison indicates that eax is greater than or equal to the argument.

> In other words, if the condition `x < num` is false, execution skips the body of the `if` statement.


## Loops

```cpp
void func(int num) {
    int x = 1337;
    while (x < num) {
        x += 1;
    }
}
```

```
func(int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-20], edi
        mov     DWORD PTR [rbp-4], 1337
        jmp     .L2
.L3:
        add     DWORD PTR [rbp-4], 1
.L2:
        mov     eax, DWORD PTR [rbp-4]
        cmp     eax, DWORD PTR [rbp-20]
        jl      .L3
        nop
        nop
        pop     rbp
        ret
```

After performing the initial assignments, the program jumps to the `.L2` label, which marks the beginning of the loop's condition check. The value of the local variable `x` is moved into the `eax` register so it can be compared with the function argument. If the comparison shows that `x` is less than the argument, execution jumps to the `.L3` label, where `x` is incremented. Control then flows back to the condition check, repeating the same instructions until the comparison fails. Once the condition is no longer true, the jump to `.L3` is skipped and the loop terminates.


## Function Calls

```cpp
int factorial(int x) {
	if (x == 0) {
	    return 1;
	} else {
        return x * factorial(x-1);
    }
}

void func() {
    int x, y;
    x = 3;
    y = factorial(x);
}
```

```
factorial(int):
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     DWORD PTR [rbp-4], edi
        cmp     DWORD PTR [rbp-4], 0
        jne     .L2
        mov     eax, 1
        jmp     .L3
.L2:
        mov     eax, DWORD PTR [rbp-4]
        sub     eax, 1
        mov     edi, eax
        call    factorial(int)
        imul    eax, DWORD PTR [rbp-4]
.L3:
        leave
        ret
func():
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     DWORD PTR [rbp-4], 3
        mov     eax, DWORD PTR [rbp-4]
        mov     edi, eax
        call    factorial(int)
        mov     DWORD PTR [rbp-8], eax
        nop
        leave
        ret
```

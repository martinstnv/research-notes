---
title: Compiling Code
date: 2026-02-22
tags:
    - Assembly
    - C++
---

## Assignments

Here is an example of a variable assignment:

```cpp
int main() {
    int x;
    x = 3;
}
```

... and here is the exact equivalent in assembler:

```nasm
pushq   %rbp
movq    %rsp, %rbp
movl    $3, -4(%rbp)
movl    $0, %eax
popq    %rbp
ret
```

It's important to not that most of the instructions are responsible for starting and ending the `main` function, which determines where the program starts and ends.

This is the instruction that corresponds to the assignment of the  `x` variable:

```nasm
movl    $3, -4(%rbp)
```

The instruction says to "move" the number 3 into this memory location.

## Conditional Statements

Here is an example of an `if` statement:

```cpp
int main() {
    int x;
    x = 3;
    if (x < 10) {
        x = x + 1;
    }
}
```

... and here is the exact equivalent in assembler:

```nasm
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $3, -4(%rbp)
        cmpl    $9, -4(%rbp)
        jmp     ENDIF
        addl    $1, -4(%rbp)
ENDIF:  movl    $0, %eax
        popq    %rbp
        ret
```

Before the regular block, we have instructions to evaluate the condition, and then we have a conditional jump statement. The processor knows whether or not to skip this jump based on the result of the previous instruction. That instruction temporarily sets some flags in the processor, so that we could remember the result by the time we get to this conditional jump to skip over the block.

Here is an example of an `if-else` statement:

```cpp
int main() {
    int x;
    x = 3;
    if (x < 10) {
        x = x + 1;
    } else {
        x = 42;
    }
}
```

... and here is the exact equivalent in assembler:

```nasm
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $3, -4(%rbp)
        cmpl    $9, -4(%rbp)
        jg      ELSE
        addl    $1, -4(%rbp)
        jmp     ENDIF
ELSE:   movl    $42, -4(%rbp)
ENDIF:  movl    $0, %eax
        popq    %rbp
        ret
```

Before the first block we have a comparison and a conditional jump, like in the previous example. In between the blocks we have a regular and non-conditional jump instruction. This is so the execution skips the second block if it just finished the first one.

## Loops

Here is an example of a `while` statement:

```cpp
int main() {
    int x;
    x = 3;
    while (x < 10) {
        x = x + 1;
    }
}
```

... and here is the exact equivalent in assembler:

```nasm
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $3, -4(%rbp)
        jmp     WHILE
DO:     addl    $1, -4(%rbp)
WHILE:  cmpl    $9, -4(%rbp)
        jle     DO
        movl    $0, %eax
        popq    %rbp
        ret
```

## Functions

Here is an example of a function named `factorial`, which returns the factorial of a given number:

```cpp
int factorial(int x) {
	if (x == 0) {
	    return 1;
	} else {
        return x * factorial(x-1);
    }
}

int main() {
    int x, y;
    x = 3;
    y = factorial(x);
}
```

... and here is the exact equivalent in assembler:

```nasm
FACTORIAL:
        pushq   %rbp
        movq    %rsp, %rbp
        subq    $16, %rsp
        movl    %edi, -4(%rbp)
        cmpl    $0, -4(%rbp)
        jne     ELSE
        movl    $1, %eax
        jmp     ENDIF
ELSE    movl    -4(%rbp), %eax
        subl    $1, %eax
        movl    %eax, %edi
        call    FACTORIAL
        imull   -4(%rbp), %eax
ENDIF   leave
        ret
MAIN:
        pushq   %rbp
        movq    %rsp, %rbp
        subq    $16, %rsp
        movl    $3, -8(%rbp)
        movl    -8(%rbp), %eax
        movl    %eax, %edi
        call    FACTORIAL
        movl    %eax, -4(%rbp)
        movl    $0, %eax
        leave
        ret
```
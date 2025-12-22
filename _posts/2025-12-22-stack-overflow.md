---
layout: single
title: Stack Overflow
date: 2025-12-22
classes: wide
tags:
  - Stack
---

This function resembles a Fibonacci recurrence, but lacks a base case.

```
void fibonacci(int a, int b) {
    int c;
    c = a + b;
    fibonacci(b,c);
}
```

Because of the infinite recursion, each call consumes stack space until the program crashes with a stack overflow.

```
High Address
              +-----------------------------+
              |                             | 
              |.............................|
              |              a              |
              |.............................|
              |              b              |
              |.............................|
              |        Return Address       |
              |.............................|
              |   Previous Frame Pointer    |
              |.............................| <- Current Frame Pointer
              |              c              |
              |.............................|
Low Address
```

```
High Address
              +-----------------------------+
              |                             | 
              |.............................|
              |              a              |  // 0
              |.............................|
              |              b              |  // 1
              |.............................|
              |        Return Address       |
              |.............................|
              |       Frame Pointer #1      |
              |.............................|
              |              c              |  // 1
              |.............................|
              |              a              |  // 1
              |.............................|
              |              b              |  // 2
              |.............................|
              |        Return Address       |
              |.............................|
              |      Frame Pointer (#3)     |
              |.............................| <- Frame Pointer #3
              |              c              |  // 3
              |.............................|
Low Address
```

The stack is limited by the operating system, not by C itself.

```
High Address
              +-----------------------------+
              |                             | 
              |.............................|
              |              a              |  // 0
              |.............................|
              |              b              |  // 1
              |.............................|
              |        Return Address       |
              |.............................|
              |       Frame Pointer #1      |
              |.............................|
              |              c              |  // 1
              |.............................|
              |              a              |  // 1
              |.............................|
              |              b              |  // 1
              |.............................|
              |        Return Address       |
              |.............................|
              |       Frame Pointer #2      |
              |.............................| <- Frame Pointer #3
              |              c              |  // 2
              |.............................|
              |                             |
              |             ...             |
              |                             |
              |.............................|
Low Address
```

Typical defaults:
 - Linux: ~8 MB (often configurable via ulimit -s)
 - Windows: ~1 MB (default)
 - Embedded systems: much smaller (KBs)

```
High Address
              +-----------------------------+
              |        Caller's data        |
              |.............................|
              |              a              |  // 0
              |.............................|
              |              b              |  // 1
              |.............................|
              |        Return Address       |
              |.............................|
              |       Frame Pointer #1      |
              |.............................|
              |              c              |  // 1
              |.............................|
              |              a              |  // 1
              |.............................|
              |              b              |  // 1
              |.............................|
              |        Return Address       |
              |.............................|
              |       Frame Pointer #2      |
              |.............................| <- Frame Pointer #3
              |              c              |  // 2
              |.............................|
              |                             |
              |             ...             |
              |                             |
              |.............................|
              |              a              | // -1586439200
              |.............................|
              |              b              | // 301789297
              |.............................|
              |        Return Address       |
              |.............................|
              |       Frame Pointer #n-2    |
              |.............................| <- Frame Pointer #n-1
              |              c              |  // -1284649903
              |.............................|
              |              a              | // 301789297
              |.............................|
              |              b              | // -1284649903
              |.............................|
              |        Return Address       |
              |.............................|
              |       Frame Pointer #n-1    |
              |.............................| <- Frame Pointer #n
              |              c              |  // -982860606
              +-----------------------------+ <- End of the Stack
Low Address
```

Once that memory is exhausted, the stack cannot grow anymore.

When the stack exceeds its allowed region, the function call tries to write past the stack boundary where the OS detects an invalid memory access and terminates the process.

Common symptoms:
 - Segmentation fault (Linux/macOS)
 - Stack overflow exception (Windows)
 - Program aborts or crashes silently (embedded systems)

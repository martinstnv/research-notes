---
layout: single
title: Understanding CPU Architecture and Memory Addressing
date: 2025-11-30
classes: wide
tags:
  - CPU Architecture
  - Memory Addressing
---

> How a CPU’s architecture determines the size of its memory space, how memory addresses work at the byte level, and how these addresses are represented in binary and hexadecimal formats, showing the maximum memory each architecture can handle.

## Memory Addressing

A 32-bit CPU architecture means that the process memory space is represented using 32 bits. The lowest memory address in this space can be visualized in binary as `0000 0000 0000 0000 0000 0000 0000 0000`. Following the same logic, the highest memory address in binary is `1111 1111 1111 1111 1111 1111 1111 1111`.

By convention, memory addresses are often written in hexadecimal format. In this notation, the lowest address is `0x00000000` and the highest address is `0xFFFFFFFF`, since each group of four binary digits corresponds to a single hexadecimal digit, with `0000` equaling 0x0 and `1111` equaling 0xF.

The highest address in a 32-bit memory space can also be expressed as 2^{32} - 1, since the address count starts from 0, or as 4,294,967,295 in decimal. Using this, we can calculate the total size of the memory space as follows:

|  Binary representation                    |  Power of 2  |  Calculation           |  Decimal representation  |
|  ---------------------------------------  |  ----------  |  --------------------  |  ----------------------  |
|  0000 0000 0000 0000 0000 0000 0000 0000  |  0           |  0                     |  0                       |
|  0000 0000 0000 0000 0000 0000 0000 0001  |  2^{0}       |  1                     |  1                       |
|  0000 0000 0000 0000 0000 0000 0000 0010  |  2^{1}       |  2 x 1                 |  2                       |
|  0000 0000 0000 0000 0000 0000 0000 0100  |  2^{2}       |  2 x 2                 |  4                       |
|  0000 0000 0000 0000 0000 0000 0000 1000  |  2^{3}       |  2 x 2 x 2             |  8                       |
|  0000 0000 0000 0000 0000 0000 0001 0000  |  2^{4}       |  2 x 2 x 2 x 2         |  16                      |
|  0000 0000 0000 0000 0000 0001 0000 0000  |  2^{8}       |  16 x 16               |  256                     |
|  0000 0000 0000 0000 0001 0000 0000 0000  |  2^{12}      |  16 x 16 x 16          |  4,096                   |
|  0000 0000 0000 0001 0000 0000 0000 0000  |  2^{16}      |  16 x 16 x 16 x 16     |  65,536                  |
|  0000 0000 0001 0000 0000 0000 0000 0000  |  2^{20}      |  16 x 65,536           |  1,048,576               |
|  0000 0001 0000 0000 0000 0000 0000 0000  |  2^{24}      |  16 x 1,048,576        |  16,777,216              |
|  0001 0000 0000 0000 0000 0000 0000 0000  |  2^{28}      |  16 x 16,777,216       |  268,435,456             |
|  1111 1111 1111 1111 1111 1111 1111 1111  |  2^{32} - 1  |  16 x 268,435,456 - 1  |  4,294,967,295           |
         
In comparison, the highest memory address of a 64-bit architecture would be:

|  Binary representation                                                            |  Power of 2  |  Calculation                    |  Decimal representation      |
|  -------------------------------------------------------------------------------  |  ----------  |  -----------------------------  |  --------------------------  |
|  1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111 1111  |  2^{64} - 1  |  268,435,456 x 268,435,456 - 1  |  18 446 744 065 119 617 025  |

So what exactly does this memory space mean?

Each memory address corresponds to 1 byte in memory. For example:

|  Address (binary)                         |  Address (hex)  |  Data (binary)  |  Data (hex)  |
|  ---------------------------------------  |  -------------  |  -------------  |  ----------- |
|  0000 0000 0000 0000 0000 0000 0000 0000  |  0x00000000     |  0000 0000      |  0x00        |
|  0000 0000 0000 0000 0000 0000 0000 0001  |  0x00000001     |  0000 1111      |  0x0F        |
|  0000 0000 0000 0000 0000 0000 0000 0010  |  0x00000002     |  0001 0000      |  0x10        |
|  0000 0000 0000 0000 0000 0000 0000 0011  |  0x00000003     |  0001 0001      |  0x11        |
|  0000 0000 0000 0000 0000 0000 0000 0100  |  0x00000004     |  0001 0010      |  0x12        |
|  0000 0000 0000 0000 0000 0000 0000 0101  |  0x00000005     |  0001 1010      |  0x1A        |
|  0000 0000 0000 0000 0000 0000 0000 0110  |  0x00000006     |  0001 1111      |  0x1F        |
|  0000 0000 0000 0000 0000 0000 0000 0111  |  0x00000007     |  0010 0000      |  0x20        |
|  0000 0000 0000 0000 0000 0000 0000 1000  |  0x00000008     |  0010 0100      |  0x24        |
|  0000 0000 0000 0000 0000 0000 0000 1001  |  0x00000009     |  0010 1111      |  0x2F        |
|  0000 0000 0000 0000 0000 0000 0000 1010  |  0x0000000A     |  0011 1111      |  0x3F        |
|  0000 0000 0000 0000 0000 0000 0000 1011  |  0x0000000B     |  0100 1111      |  0x4F        |
|  0000 0000 0000 0000 0000 0000 0000 1100  |  0x0000000C     |  1000 1111      |  0x8F        |
|  0000 0000 0000 0000 0000 0000 0000 1101  |  0x0000000D     |  1010 1111      |  0xAF        |
|  0000 0000 0000 0000 0000 0000 0000 1110  |  0x0000000E     |  1111 1111      |  0xFF        |
|  0000 0000 0000 0000 0000 0000 0000 1111  |  0x0000000F     |  1010 1010      |  0xAA        |
|  0000 0000 0000 0000 0000 0000 0001 0000  |  0x00000010     |  0101 0101      |  0x55        |
|  0000 0000 0000 0000 0000 0000 0001 0001  |  0x00000011     |  1001 1001      |  0x99        |
|  0000 0000 0000 0000 0000 0000 0001 0010  |  0x00000012     |  1110 1110      |  0xEE        |
|  0000 0000 0000 0000 0000 0000 0001 0010  |  0x00000013     |  1100 1100      |  0xDD        |


A 32-bit CPU can reference up to 2^{32} = 4,294,967,296 memory locations, which equals exactly 4 GB of memory. This is called addressable memory and it is the maximum amount of memory that a CPU can reference using its addresses.

Physical memory is the actual RAM installed in the system and it is often less than the maximum addressable memory. Some addresses are reserved for system devices or hardware I/O, so not all addresses map to physical RAM.

> Operating systems use virtual memory to allow processes to work as if they have the full addressable space, even if physical RAM is smaller.

In comparison, a 16-bit CPU can address up to 65,536 bytes, which equals 64 KB of addressable memory, while a 64-bit CPU can theoretically address up to 18,446,744,065,119,617,025 KB, or 16,384 PB of memory.


## Endianess

When a CPU stores values larger than one byte—such as 2-byte, 4-byte, or 8-byte numbers—it must decide the order in which the bytes are stored in memory. This ordering is called endianness:

 - Big-endian: the most significant byte (MSB) is stored at the lowest memory address.
 - Little-endian: the least significant byte (LSB) is stored at the lowest memory address.

For example, using the table above, the values would be arranged differently in memory depending on the endianness.

|  Address     |  Data        |
|  ----------  |  ----------  |
|  0x00000000  |  0x000F1011  |
|  0x00000004  |  0x121A1F20  |
|  0x00000008  |  0x242F3F4F  |
|  0x0000000C  |  0x8FAFFFAA  |
|  0x00000010  |  0x5599EEDD  |


While in litte endian, it would be:

|  Address     |  Data        |
|  ----------  |  ----------  |
|  0x00000000  |  0x11100F00  |
|  0x00000004  |  0x201F1A12  |
|  0x00000008  |  0x4F3F2F24  |
|  0x0000000C  |  0xAAFFAF8F  |
|  0x00000010  |  0xDDEE9955  |

As another example, the 16-bit value `0xCAFE` would be stored as `FECA0000` in little-endian format and as `0000CAFE` in big-endian format. Note that the leading zeros are typically displayed when showing memory contents in fixed-width formats.

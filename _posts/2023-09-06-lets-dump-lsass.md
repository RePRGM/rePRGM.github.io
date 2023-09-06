---
layout: post
title: Lets Dump LSASS
---

![Lets Make Malware](/assets/lets_make_malware_header.jpg)

# Introduction
Local Subsystem Authority Subsystem Service (LSASS) is one of the most important parts of Windows as it handles authentication.

This also makes it a wonderful target for threat actors and your friendly, neighborhood pentest team. 

So, in this post, we’re going to discuss various methods of dumping the process’s memory as well as some pitfalls associated with some of these methods.

# Task Manager

We will begin our discussion by talking about the easiest method of dumping lsass.

By finding the process in task manager and right-clicking, we will find the option to create a dump file.

This method works great under the condition there is no anti-virus running on the system.

Otherwise, the resulting dump file is detected quickly. In the case of Windows Defender and some other AV, the detection is the dump file itself.

If we look through the file’s strings, we will find various strings mentioning “lsass”. Our detection likely stems from these strings being found along the MDMP file signature within the header.

As with a lot of the methods we will discuss, an easy fix  is obfuscating our dump file. We can do this by removing our file signature, removing the references to lsass, or encrypting the file entirely.

Keep in mind, however, if we remove things like the file header or the lsass strings, tools such as mimikatz may not be able to read the dump file afterwards, meaning we must also restore these bytes at some point.

# Comsvcs.dll

Our next method is also simple. We will make use of the Comsvcs dll. Of course, as with any dll, we cannot directly execute it. 

We must use another executable to call the functions within the dll. 

In this case, we will simply use rundll32, but we could also use reflectively loading or even regularly loading the dll with a custom executable file.

Within Comsvcs, there is a function conveniently named MiniDump that we can use.

The command we will use is `rundll32 C:\windows\system32\comsvcs.dll MiniDump [LSASS_PID] [out file] full`

This, of course, presents several detection opportunities.

In addition to detecting the resulting dump file, there is also an opportunity to detect rundll32.exe which is a commonly abused executable as well as another detection opportunity by observing the command line arguments. 

This method is arguably even less stealthy than simply going through task manager.

# Third-Party Executables

We can, of course, use third-party executables and scripts to dump LSASS. The sky is the limit here as there are plenty to choose from. 

Some examples are ProcDump from Sysintenals, nanodump, mimikatz, secretsdump, and of course our own [EvilLsassTwin](https://github.com/RePRGM/Nimperiments/tree/main/EvilLsassTwin).

The problem with most of these is that AVs have signatures for them and will block them from running. 

# Custom

If we want a decent chance at successfully dumping LSASS, our only real option is to go custom. There are numerous methods we can utilize and combine, but most tools to dump LSASS have two things in common.

The first is the use of the `MiniDumpWriteDump` Win32 API function. This is a function of convenience. It will read the memory of the process we choose and create a dump file in the proper format for us. 

The second thing these tools have in common is to dump LSASS, we need to get a handle to it one way or another.

## Handles

Gaining a handle to LSASS in the first place is usually where scrutiny lies. `OpenProcess` and `NtOpenProcess` are likely hooked to begin with. 

So, any EDR worth the cash will see a process attempting to gain a handle to LSASS of all things. That’s a bit suspicious on its own, but not necessarily a grave offense. 

Next, the EDR will look at what rights we are requesting either through the hook or through a kernel callback for object handle creation/duplication. 

PROCESS_VM_READ? PROCESS_ALL_ACCESS? To LSASS? Highly suspicious.

_Note: it may not be game over simply for requesting a highly permissioned handle to LSASS. In fact, it may be what happens next that determines whether EDRs kill our process or not._

If we want to avoid scrutiny, we must find another way to gain a handle.

Luckily, we have some options here. 

We can duplicate an existing handle. This will not usually be possible, but it can happen. Or we can force it to happen. 

Kernel callbacks will still trigger doing this, so we are not completely safe, but it is a safer option than a brand-new handle through `OpenProcess`.

The other option is to use `NtGetNextProcess`. This is slightly better than going through `NtOpenProcess` or `OpenProcess`, however can still be hooked. The main benefit is that we are not requesting a handle to our target process _directly_. 

Instead, we are cycling through all processes running and stopping when we find our target.

# MiniDumpWriteDump
```c
BOOL MiniDumpWriteDump(
  [in] HANDLE                            hProcess,
  [in] DWORD                             ProcessId,
  [in] HANDLE                            hFile,
  [in] MINIDUMP_TYPE                     DumpType,
  [in] PMINIDUMP_EXCEPTION_INFORMATION   ExceptionParam,
  [in] PMINIDUMP_USER_STREAM_INFORMATION UserStreamParam,
  [in] PMINIDUMP_CALLBACK_INFORMATION    CallbackParam
);
```

Now that we have a handle to LSASS, what do we do with it? 

We either give it to `MiniDumpWriteDump` or implement our own logic to read memory and create a dump file.

There are two problems with using this API function.

First, it takes not only a handle to our target process, but also needs the target process’s PID, which it uses to request another handle. 

Thankfully, we can get around this second requirement by simply passing in 0 or our own process’s PID.

The other issue is that not only are we writing to disk, but we are _also_ writing a clear-text LSASS dump file to disk. 

Writing to disk should generally be avoided for OPSEC, but what’s worse is that the dump file will be scanned and removed almost immediately.

Thankfully, yet again we have options. 

We can use a callback to obfuscate the contents or even keep everything in memory.

Alternatively, we can make use of “Delete on Close” functionality, which is the approach EvilLsassTwin takes.

Delete on Close is a component of the Process Ghosting technique in which we create a file and prevent other threads from accessing it while our process has a handle to it. 

Once our process closes its handle to the file, the file is deleted. 

This does not seem useful at first glance, but what we can do once the file is created, but prior to deletion, is map the file to memory and encrypt it. 

From there, we can do any number of things with the data.

The same thing can also be accomplished using a callback as well.

## Alternatives

Finally, let’s talk about some alternatives. 

Instead of passing a handle directly to LSASS to `MiniDumpWriteDump`, we can do things a bit more indirectly.

### PssCaptureSnapshot

We can first create a snapshot of the process with `PssCaptureSnapshot` and then pass this snapshot’s handle to `MiniDumpWriteDump`. 

This has the benefit of not reading LSASS’s memory directly. 

### NtCreateProcessEx

We can also fork / clone LSASS (or any other process) with `NtCreateProcessEx` and then pass the resulting handle from this fork to `MiniDumpWriteDump`. 

There are a couple benefits to doing this. 

First, this function does not create any threads for the fork. That must be done manually by the developer. 

Because of this, no process creation events or callbacks are triggered if we do not create threads. 

While the process will show up in Task Manager, it will be suspended as there are no threads.

So, we have a duplicate LSASS process that EDRs do not know exists. 

Second, the fork is a child process of the original. In other words, even if EDR knows about this fork, it does not look associated to our malicious process.

This is the method EvilLsassTwin takes.

The other alternative we will discuss is an alternative to the `MiniDumpWriteDump` function itself. 

There is another function we can use instead. 

### RtlReportSilentProcessExit
```c
NTSTATUS RtlReportSilentProcessExit(
  [in] HANDLE                            ProcessHandle,
  [in] NTSTATUS                             ExitStatus
);
```
We will not be diving too deep into how this function works. 

We only need to know that with some registry modifications, we can use this function to have the Windows Error Reporting (WER) service create a dump file for us. 

The benefit to this function is that we can disassociate our malicious executable from the resulting dump file of our target process.

The downsides are three-fold. 

First, we need to make registry changes. Kernel callbacks can be triggered because of this. Event logs will also be generated.

The second downside is that we have little control over the dump file. It will be created in plaintext.

Lastly, a dump file of the process that calls the `RtlReportSilentProcessExit` function will _also_ be created. 

So, while we _have_ disassociated our malicious executable from the process of dumping our target process somewhat, there _is_ still an association. 

# Resources
[DeepInstinct - LSASS Memory Dumps are Stealthier Than Ever Part 2](https://www.deepinstinct.com/blog/lsass-memory-dumps-are-stealthier-than-ever-before-part-2)

[IRedTeam - Dumping LSASS Passwords without Mimikatz](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-lsass-passwords-without-mimikatz-minidumpwritedump-av-signature-bypass#minidumpwritedump-+-psscapturesnapshot)

[MSDN - MiniDumpWriteDump](https://learn.microsoft.com/en-us/windows/win32/api/minidumpapiset/nf-minidumpapiset-minidumpwritedump)

[Splintercod3 - The Hidden Side of SecLogon Part 2](https://splintercod3.blogspot.com/p/the-hidden-side-of-seclogon-part-2.html)

[PentestParters - Dumping LSASS in Memory Undetected using Mirrordump](https://www.pentestpartners.com/security-blog/dumping-lsass-in-memory-undetected-using-mirrordump/)

[Hexacorn - SilentProcessExit - Quick Look Under the Hood](https://www.hexacorn.com/blog/2019/09/19/silentprocessexit-quick-look-under-the-hood/)

[Rasta Mouse - Dumping LSASS with Duplicated Handles](https://rastamouse.me/dumping-lsass-with-duplicated-handles/)

[Bussink - LSASS Minidump File Seen as Malicious by McAfee AV](https://www.bussink.net/lsass-minidump-file-seen-as-malicious-by-mcafee-av/)

[Powerseb - LSASS Parsing without a Cat](https://powerseb.github.io/posts/LSASS-parsing-without-a-cat/)

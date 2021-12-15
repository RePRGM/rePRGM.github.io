---
layout: post
title: Road to Red-Teaming: Defense Evasion
---

Defense evasion is a topic that interests me and as such I’ve been spending some time researching various methods of doing so. This is a very in-depth topic and I have only scratched the surface of what is possible. 

-----

## How AV/EDR Work

To understand how to evade defenses, one must first understand how malware is caught. There are quite a few different ways, but most can be boiled down to signatures and behavior/heuristics. 

## Signatures

Signatures can be a hash (typically, a MD5 sum) of a file itself or certain strings within the file itself. These strings could be comments, or variable or function names. Bypassing this type of detection tends to be much easier than bypassing heuristics. This will typically involve obfuscation. This includes renaming variable and function names, removing comments, and utilizing encoding or encryption.

## Behavior/Heuristics

This category boils down to “watch what the executable does”. AV and EDR do this in a few different ways, which is why it is a bit more difficult to bypass compared to bypassing signatures. These methods include API hooking, sandboxing, and etw monitoring.

### API Hooking

Calling certain Windows API functions will trigger modern-day AV/EDR to monitor what the executable is trying to do and determine if it is malicious or not. It does so by “hooking” Windows API functions contained in ntdll.dll. Hooking means modifying the function’s Assembly instructions and inserting a JMP instruction that leads to the AV/EDR’s code. Thus, altering program flow. If the AV/EDR determines the call is benign, it will then allow the original function call to go through. 

There are a few methods of bypassing hooking. The easiest to implement would be unhooking. This can be done either by unhooking the specific functions the executable uses or by remapping ntdll.dll. Some AV/EDR may be able to detect this method now however.

Another method involves using direct syscalls. This means avoiding the hooked function calls altogether. This is harder to implement as it involves including assembly instructions in your code. These instructions also vary, not only between major Windows versions, but between Service Packs and Updates as well.

### Sandboxing

Sandboxing involves running an arbitrary executable in an isolated virtual environment. AV/EDR will watch what it does and determine if it is malicious or not.

There are a few ways to bypass this. An attacker can add time-delays to their code before executing anything malicious. While AV/EDR can’t, or rather won’t, maintain a sandbox indefinitely, this is a known bypass so some will “fast-forward” in time past a delay.

An attacker can also add some checks to their code. Most sandboxes will have a few characteristics that make them stand-out compared to a live, production system. These checks could be for hard-drive size, ram, cpu cores, etc. Sandboxing can be bypassed by checking for these characteristics and only then further executing.

### ETW Monitoring

AV/EDR can also monitor system logs to detect malware. Performing actions such starting cmd.exe, and then running commands commonly used for enumeration such as net user, whoami, sysinfo, etc are all logged and can cause an AV/EDR to determine malicious activity is occurring. 

Interestingly enough, bypassing ETW involves hooking/patching certain functions within ntdll.dll.

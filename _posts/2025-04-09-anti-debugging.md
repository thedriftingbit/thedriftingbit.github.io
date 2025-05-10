---
layout: post
title: "Basic Anti-Debugging Techniques"
author: <author_id> 
categories: write-up
tags: [cybersecurity,debugging,malware,windbg,blog]
---

There are many anti-debugging techniques that can be used to prevent the reverse engineering of malware. This post will detail two simple and commonplace techniques used to thwart a debugger - the `IsDebuggerPresent()` function and the `ICEBP` instruction - as well as provide practical examples of their circumvention. The examples are taken from a WinDbg lab debugging an executable called WinDbgDemo.exe, but the concepts presented can be applied in your debugger of choice.

## The IsDebuggerPresent() Function

This function is seen in Windows and is part of the operating system's API. The purpose of this function, as the name suggests, is to detect whether a user-mode debugger has been attached to the given process. This is done by various API calls made at the time of attachment and is a bit beyond the scope of this article - if you are interested, [this](https://xorl.wordpress.com/2017/11/20/reverse-engineering-isdebuggerpresent/) is a great explanation. The function checks the `BeingDebugged` flag which will be set to 1 if the process is being debugged. This flag lives in what is called the **Process Environment Block**.

### Process Environment Block

The Process Environment Block (PEB) is a structure that exists for each process and contains data related to that process. In WinDbg, you can view a nicely-formatted version of the PEB with the command `!peb`.

![the process environment block displayed in WinDbg](assets\img\anti-debugging\PEB_Format.jpg)

The third value down is the `BeingDebugged` flag, which we can see has been set to "Yes." Another way to view the PEB in WinDbg is by using the `dt _peb @$peb` command, which will display the information about a specified structure - in this case, the `_peb` structure located at the variable `$peb`. Structures like the PEB are extensively documented by Microsoft, so be sure to look up any that you may be unfamiliar with. [This](https://learn.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-peb) is the link to the structure documentation for the PEB.

![the process environment block structure](assets\img\anti-debugging\PEB1.jpg)

Moreso than the `!peb` command, this `dt` command shows the actual values set for each of these flags. We can see that the `BeingDebugged` flag is set to 1. Once the IsDebuggerPresent() function sees this flag, it will call ExitProcess and our debugging attempt will be cut short. However, WinDbg allows you to edit the content of specific memory addresses - in our case, we will use the `eb $peb+0x2 0x0` command to set the byte value of the `BeingDebugged` flag to 0. This command targets the flag at location 2 in the PEB - in this case, `BeingDebugged`, and sets it to 0.

![setting the Being Debugged flag to 0](assets\img\anti-debugging\PEB2.jpg)

## The ICEBP Instruction

The next hurdle we encounter is the `ICEBP` instruction. This has an undocumented op code of `0xF1` that can be seen in x86 assembly. Originally, it was meant to be used in debugging an In-Circuit Emulator. Essentially, this instruction will create a breakpoint in the code, creating a single-step interrupt. A very thorough examination of this instruction can be found [here](https://www.rcollins.org/secrets/opcodes/ICEBP.html). In our case, this means that the exception handler inside of the program will not be called and our debugging attempts will be halted. To prevent this, we need to patch the op code to something else. But how do we find this instruction?

The `ICEBP` instruction is not recognized by x86 assembly, unlike common instructions like `call`, `mov`, etc. WinDbg represents this as an instruction of `???`, so we can use this to narrow down our search.

First, we use the `lm` command to list the modules in the WinDbgDemo.exe executable:

![listing modules in the executable](assets\img\anti-debugging\WinDBG_LM.jpg)

We can see that the WinDbgDemo module starts at memory location `00870000` and ends at `00876000`. This would be a great range to start our search, as it is reasonable to assume that this instruction lives within the code being executed in its namesake's module. To do this, we can search for a pattern over a range of memory addresses with the command `# [\?\?\?] 00870000 00876000`. The bracketed portion of that command is a regular expression searching for three question marks which have to be escaped using `\` since they can be used as regular expression syntax.

![searching for the ??? instruction](assets\img\anti-debugging\WinDBG_SearchF1.jpg)

Within the first few results, we see the `f1` value we are looking for at memory location `00871293` and can use the unassembly (`u`) command to see the assembly at that location with `u 00871293`.

![checking the assembly instructions at memory location](assets\img\anti-debugging\WinDBG_Unassemble.jpg)

This is exactly what we were looking for! We can edit the `f1` portion of the code to something else to avoid this instruction. This is easily done in WinDbg within the Memory window:

![memory window](assets\img\anti-debugging\WinDBG_MemoryPatch.jpg)

You can highlight the hex value shown and edit it as needed - in my case, I used `ff`:

![memory window after patching](assets\img\anti-debugging\WinDBG_AfterPatch2.jpg)

With the `BeingDebugged` flag set to 0 and the `ICEBP` instruction patched out, we can continue our traversal of this executable. The way this lab concludes is by showing the memory location of a deobfuscated string in the console once the appropriate anti-debugging measures are avoided:

![memory location](assets\img\anti-debugging\WinDBG_DeobStringLocation.jpg)

The only trick here is to not let the program run to completion - as part of the process termination, the string we are looking for will be wiped out! So, we can step out of a couple of functions after this memory location was printed and once again look at the location in the Memory window to find the deobfuscated string:

![deobfuscated string at memory location](assets\img\anti-debugging\WinDBG_DeobString.jpg)
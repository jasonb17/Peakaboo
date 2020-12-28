# Peakaboo

Peakaboo is a shellcode implementation of the "Shared Memory Subversion" technique presented by Frank Block at BlackHat USA 2020 (https://i.blackhat.com/USA-20/Wednesday/us-20-Block-Hiding-Process-Memory-Via-Anti-Forensic-Techniques.pdf)

Tested on Windows 10 version 2004, x64

## Usage
```
C:\Projects\Peakaboo\Debug>.\Peakaboo

Spawning Powershell target
Holding until target process is initialized...
Locating existing RWX region that is safe for injection...
Found the target address: (0xAB300000)
RWX section injected with controller shellcode.
Watch console output in Powershell to see the payload get mapped and unmapped!
```

In newly spawned Powershell window:
```
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\Projects\Peakaboo\Debug> Hello from mapped shellcode! Memory visible for 15 seconds
Hello from mapped shellcode!
Hello from mapped shellcode!
Goodbye! Unmapping and sleeping for 30 seconds
Hello from mapped shellcode! Memory visible for 15 seconds
Hello from mapped shellcode!
Hello from mapped shellcode!
Goodbye! Unmapping and sleeping for 30 seconds
```

## Explanation

The crux of the technique is that... \[to fill in\] ... memory scanners can't detect unmapped memory

The injector (main.exe) will first map into a shared section the final shellcode payload (a printf loop for demonstration purposes). The injector then creates a Powershell process, enumerates an existing RWX region in it, and writes into it the controller shellcode, whose responsibility is to continually map the final payload -> execute it -> unmap -> sleep -> repeat. The controller is then initiated in a new thread.

See demo section for a more detailed breakdown.

In an operation, this could be used for \[to fill in\]

#### Side note:
This POC uses a RWX shared section for the final payload, and WriteProcessMemory + CreateRemoteThread to execute the controller shellcode. These are significant IOCs, and were simply used to make the demo clearer + to keep the focus on the "shared memory subversion" technique. An example of a stealthier version would be using phantom DLL hollowing to create the shared section, and stack bombing to force the target process to write into its own memory the controller shellcode (which could be embedded within the ROP chain, loaded from another file dropped, loaded from a registry key created by the injector, etc). See https://github.com/jasonb17/Phantom-Bomber for more detail

Furthermore, an even stealthier variant could entirely handle the controller functionality with a ROP chain written to and executed from the stack after each run through the actual payload. E.g. create a shared section which contains the final shellcode payload, immediately appended with instructions for the thread to write to its stack a ROP chain that performs unmap -> sleep -> re-map -> execute. This works because while execution has been diverted to the ROP chain, the shared section is no longer needed and can be safely unmapped, leaving no in-memory IOC other than the malicious stack. The shared section code could be run for the first time in the target process with stack bombing -> initial mapping and execution.


#### Side note 2:
The controller and final payload shellcodes were extracted from C++ code using the technique described by Hasherzade in following paper, which makes generating complicated position-independent shellcode easy.
https://vxug.fakedoma.in/papers/VXUG/Exclusive/FromaCprojectthroughassemblytoshellcodeHasherezade.pdf


## Credit

https://i.blackhat.com/USA-20/Wednesday/us-20-Block-Hiding-Process-Memory-Via-Anti-Forensic-Techniques.pdf

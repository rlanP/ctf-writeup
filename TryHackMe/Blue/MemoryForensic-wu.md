# Memory Forensic -- Write-up

## Overview
Platform: TryHackMe  
Difficulty: Easy  
Description: Perform memory forensics to find the flags  
Type: Digital Forensic

## Question 1 | What is John's Password?
**Description:**  
The forensic investigator on-site has performed the initial forensic analysis of John's computer and handed you the memory dump he generated on the computer. As the secondary forensic investigator, it is up to you to find all the required information in the memory dump.

---

So the first thing to do in a memory forensic is to identify the Operating-system being used, the reason is because the tool that are used is Volatility and it used different comand for different Operating system.  
**Command Used:**  

    python3 vol.py -f <memory file> windows.info

**Result:**  
```
└─# python3 vol.py -f /media/sf_Documents/TryHackMe/memoryforensic/Snapshot6_1609157562389.vmem windows.info     
Volatility 3 Framework 2.28.1
WARNING  volatility3.framework.layers.vmware: No metadata file found alongside VMEM file. A VMSS or VMSN file may be required to correctly process a VMEM file. These should be placed in the same directory with the same file name, e.g. Snapshot6_1609157562389.vmem and Snapshot6_1609157562389.vmss.
Progress:  100.00               PDB scanning finished                        
Variable        Value

Kernel Base     0xf80002a59000
DTB     0x187000
Symbols jar:file:/home/kali/volatility3/volatility3/symbols/windows.zip!windows/ntkrnlmp.pdb/3844DBB920174967BE7AA4A2C20430FA-2.json.xz
Is64Bit True
IsPAE   False
layer_name      0 WindowsIntel32e
memory_layer    1 FileLayer
KdDebuggerDataBlock     0xf80002c4a0a0
NTBuildLab      7601.17514.amd64fre.win7sp1_rtm.
CSDVersion      1
KdVersionBlock  0xf80002c4a068
Major/Minor     15.7601
MachineType     34404
KeNumberProcessors      1
SystemTime      2020-12-27 06:20:05+00:00
NtSystemRoot    C:\Windows
NtProductType   NtProductWinNt
NtMajorVersion  6
NtMinorVersion  1
PE MajorOperatingSystemVersion  6
PE MinorOperatingSystemVersion  1
PE Machine      34404
PE TimeDateStamp        Sat Nov 20 09:30:02 2010

```
Since the Operating system is Windows, we can use Volatility windows plugin to analyze the memory.  

Volatility Window plugins provide a dedicated plugin to extract a password hash, if the password is weak there is a posibility to crack it using John the ripper or hashcat.


    python3 vol.py -f <memory file> windows.hashdump


**Result:**
```
└─# python3 vol.py -f /media/sf_Documents/TryHackMe/memoryforensic/Snapshot6_1609157562389.vmem windows.hashdump
Volatility 3 Framework 2.28.1
WARNING  volatility3.framework.layers.vmware: No metadata file found alongside VMEM file. A VMSS or VMSN file may be required to correctly process a VMEM file. These should be placed in the same directory with the same file name, e.g. Snapshot6_1609157562389.vmem and Snapshot6_1609157562389.vmss.
/home/kali/volatility3/volatility3/framework/deprecation.py:28: FutureWarning: This API (volatility3.plugins.windows.registry.hashdump.Hashdump.run) will be removed in the first release after 2026-09-25. This plugin has been renamed, please call volatility3.plugins.windows.registry.hashdump.Hashdump rather than volatility3.plugins.windows.hashdump.Hashdump.
  warnings.warn(

User    rid     lmhash  nthash
/home/kali/volatility3/volatility3/framework/deprecation.py:105: FutureWarning: This plugin (volatility3.plugins.windows.hashdump.Hashdump) has been renamed and will be removed in the first release after 2026-09-25. Please ensure all method calls to this plugin are replaced with calls to volatility3.plugins.windows.registry.hashdump.Hashdump
  warnings.warn(

Administrator   500     aad3b435b51404eeaad3b435b51404ee        31d6cfe0d16ae931b73c59d7e0c089c0
Guest   501     aad3b435b51404eeaad3b435b51404ee        31d6cfe0d16ae931b73c59d7e0c089c0
John    1001    <REDACTED>  
HomeGroupUser$  1002    aad3b435b51404eeaad3b435b51404ee        91c34c06b7988e216c3bfeb9530cabfb
                                                    
```
After extracting the NTLM hash, I used John the Ripper with the `rockyou.txt` wordlist to crack it.
```
└─# john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
<REDACTED>    (?)     
1g 0:00:00:00 DONE (2026-07-24 10:56) 1.333g/s 12241Kp/s 12241Kc/s 12241KC/s charmed12b..charmaise
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed. 

```
The password for john has been recovered: **`<REDACTED>`** 


---


**Question 2 | When was the machine last shutdown?**  
**Question 3 | What did John write?**

**Description:**  
On arrival a picture was taken of the suspect's machine, on it, you could see that John had a command prompt window open. The picture wasn't very clear, sadly, and you could not see what John was doing in the command prompt window.

To complete your forensic timeline, you should also have a look at what other information you can find, when was the last time John turned off his computer?

---

Because we are given a new file for this question, we need to check again the Operating system being used.

    python3 vol.py -f <memory file> windows.info

We now know the system are using windows.

The first question ask for the machine last shutdown, this is usually stored in registry hive, windows actually keeps active registry in memory, with that information we can try to find the registry being loaded.

The first thing to do is to find what hive are currently being loaded:

    python3 vol.py -f <memory file> windows.registry.hivelist

***Result:***  
```
└─# python3 vol.py -f /media/sf_Documents/TryHackMe/memoryforensic/Snapshot19_1609159453792.vmem  windows.registry.hivelist
Volatility 3 Framework 2.28.1
WARNING  volatility3.framework.layers.vmware: No metadata file found alongside VMEM file. A VMSS or VMSN file may be required to correctly process a VMEM file. These should be placed in the same directory with the same file name, e.g. Snapshot19_1609159453792.vmem and Snapshot19_1609159453792.vmss.
Progress:  100.00               PDB scanning finished                        
Offset  FileFullPath    File output

0xf8a00000f010          Disabled
0xf8a000024010  \REGISTRY\MACHINE\SYSTEM        Disabled
0xf8a000061010  \REGISTRY\MACHINE\HARDWARE      Disabled
0xf8a0000f7010  \SystemRoot\System32\Config\DEFAULT     Disabled
0xf8a0007ac010  \Device\HarddiskVolume1\Boot\BCD        Disabled
0xf8a001502010  \SystemRoot\System32\Config\SOFTWARE    Disabled
0xf8a001674410  \SystemRoot\System32\Config\SECURITY    Disabled
0xf8a0016dc410  \SystemRoot\System32\Config\SAM Disabled
0xf8a0016f7010  \??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT        Disabled
0xf8a0017a9010  \??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT  Disabled
0xf8a00196c010  \??\C:\Users\John\ntuser.dat    Disabled
0xf8a00197f010  \??\C:\Users\John\AppData\Local\Microsoft\Windows\UsrClass.dat  Disabled
0xf8a0024e4010  \??\C:\System Volume Information\Syscache.hve   Disabled
                                                                                                                                                             
```

The SYSTEM registry hive is loaded in memory. Because the last shutdown time is stored in this hive, we can inspect it to recover the ShutdownTime value.  



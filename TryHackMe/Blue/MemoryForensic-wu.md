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

Windows can contain multiple ControlSet, each one can contain diffrent value, we need to see the one currently being used to find the value we are looking for, we can use this command to find the current control set:  

```
python3 vol.py -f <memory file> windows.registry.printkey --key "Select"

```

```
Volatility 3 Framework 2.28.2
WARNING  volatility3.framework.layers.vmware: No metadata file found alongside VMEM file. A VMSS or VMSN file may be required to correctly process a VMEM file. These should be placed in the same directory with the same file name, e.g. Snapshot19_1609159453792.vmem and Snapshot19_1609159453792.vmss.
Progress:  100.00               PDB scanning finished                        
Last Write Time Hive Offset     Type    Key     Name    Data    Volatile

-       0xf8a00000f010  Key     [NONAME]\Select -       -       -
2009-07-14 05:08:22.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\Select Current 1       False
2009-07-14 05:08:22.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\Select Default 1       False
2009-07-14 05:08:22.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\Select Failed  0       False
2009-07-14 05:08:22.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\Select LastKnownGood   2       False
-       0xf8a000061010  Key     \REGISTRY\MACHINE\HARDWARE\Select       -       -       -
-       0xf8a0000f7010  Key     \SystemRoot\System32\Config\DEFAULT\Select      -       -       -
-       0xf8a0007ac010  Key     \Device\HarddiskVolume1\Boot\BCD\Select -       -       -
-       0xf8a001502010  Key     \SystemRoot\System32\Config\SOFTWARE\Select     -       -       -
-       0xf8a001674410  Key     \SystemRoot\System32\Config\SECURITY\Select     -       -       -
-       0xf8a0016dc410  Key     \SystemRoot\System32\Config\SAM\Select  -       -       -
-       0xf8a0016f7010  Key     \??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT\Select -       -       -
-       0xf8a0017a9010  Key     \??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT\Select   -       -       -
-       0xf8a00196c010  Key     \??\C:\Users\John\ntuser.dat\Select     -       -       -
-       0xf8a00197f010  Key     \??\C:\Users\John\AppData\Local\Microsoft\Windows\UsrClass.dat\Select   -       -       -
-       0xf8a0024e4010  Key     \??\C:\System Volume Information\Syscache.hve\Select    -       -       -
                                                                                          
```
The `Select` registry key indicates which ControlSet is currently active. In this case, the `Current` value is set to `1`, meaning that **ControlSet001** is the active ControlSet. Therefore, registry analysis should use `ControlSet001` to ensure the correct configuration values are examined.

```text
Current = 1
```

Using the active ControlSet, we can inspect the `ControlSet001\Control\Windows` registry key to retrieve the `ShutdownTime` value.

```
python3 vol.py -f <memory_file> windows.registry.printkey --key "ControlSet001\Control\Windows"
```
Result:  
```
┌──(root㉿kali)-[/home/kali/tools/volatility3]
└─# python3 vol.py -f /media/sf_Documents/TryHackMe/memoryforensic/Snapshot19_1609159453792.vmem  windows.registry.printkey --key "ControlSet001\Control\Windows" 
Volatility 3 Framework 2.28.2
WARNING  volatility3.framework.layers.vmware: No metadata file found alongside VMEM file. A VMSS or VMSN file may be required to correctly process a VMEM file. These should be placed in the same directory with the same file name, e.g. Snapshot19_1609159453792.vmem and Snapshot19_1609159453792.vmss.
Progress:  100.00               PDB scanning finished                        
Last Write Time Hive Offset     Type    Key     Name    Data    Volatile

-       0xf8a00000f010  Key     [NONAME]\ControlSet001\Control\Windows  -       -       -
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  ErrorMode       0       False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_EXPAND_SZ   \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  Directory       %SystemRoot%    False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  NoInteractiveServices   0       False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_EXPAND_SZ   \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  SystemDirectory %SystemRoot%\system32False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  ShellErrorMode  1       False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  CSDVersion      256     False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  CSDReleaseType  0       False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  CSDBuildNumber  17514   False
2020-12-27 22:50:12.000000 UTC  0xf8a000024010  REG_DWORD       \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  ComponentizedBuild      1       False
<REDACTED> UTC  0xf8a000024010  REG_BINARY      \REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\Windows  ShutdownTime
<REDACTED>                         ..P.....                False
-       0xf8a000061010  Key     \REGISTRY\MACHINE\HARDWARE\ControlSet001\Control\Windows        -       -       -
-       0xf8a0000f7010  Key     \SystemRoot\System32\Config\DEFAULT\ControlSet001\Control\Windows       -       -       -
-       0xf8a0007ac010  Key     \Device\HarddiskVolume1\Boot\BCD\ControlSet001\Control\Windows  -       -       -
-       0xf8a001502010  Key     \SystemRoot\System32\Config\SOFTWARE\ControlSet001\Control\Windows      -       -       -
-       0xf8a001674410  Key     \SystemRoot\System32\Config\SECURITY\ControlSet001\Control\Windows      -       -       -
-       0xf8a0016dc410  Key     \SystemRoot\System32\Config\SAM\ControlSet001\Control\Windows   -       -       -
-       0xf8a0016f7010  Key     \??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT\ControlSet001\Control\Windows  -       -       -
-       0xf8a0017a9010  Key     \??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT\ControlSet001\Control\Windows    -       -       -
-       0xf8a00196c010  Key     \??\C:\Users\John\ntuser.dat\ControlSet001\Control\Windows      -       -       -
-       0xf8a00197f010  Key     \??\C:\Users\John\AppData\Local\Microsoft\Windows\UsrClass.dat\ControlSet001\Control\Windows    -       -       -
-       0xf8a0024e4010  Key     \??\C:\System Volume Information\Syscache.hve\ControlSet001\Control\Windows     -       -       -
                                                                                                                                                             

```

Since `ShutdownTime` is stored in the Windows FILETIME format, it must be converted into a human-readable timestamp. We can use CyberChef to decode the value.

1. Swap the byte order using [CyberChef](https://gchq.github.io/CyberChef/#recipe=Swap_endianness('Hex',8,true)).
2. Copy the output and paste it into the [Windows FILETIME Converter](https://inventivehq.com/tools/developer/filetime-converter).
3. The converter returns the UTC timestamp.

After converting the `ShutdownTime` value, we obtain the following timestamp:

```
<REDACTED>
```

This indicates that the machine was last shut down on **`<REDACTED>`**.

Next is to find what was written in command prompt.
I first tried `windows.consoles` in Volatility 3, but it returned an error because Windows 7 SP1 isn't supported. `windows.cmdscan` also failed since it uses the same console parser, so I switched to Volatility 2.

```
vol.py -f <Memoru File> --profile=Win7SP1x64 consoles
```

Result:
```
**************************************************
ConsoleProcess: conhost.exe Pid: 2488
Console: 0xffa66200 CommandHistorySize: 50
HistoryBufferCount: 1 HistoryBufferMax: 4
OriginalTitle: %SystemRoot%\System32\cmd.exe
Title: Administrator: C:\Windows\System32\cmd.exe
AttachedProcess: cmd.exe Pid: 1920 Handle: 0x60
----
CommandHistory: 0x21e9c0 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 7 LastAdded: 6 LastDisplayed: 6
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x60
Cmd #0 at 0x1fe3a0: cd /
Cmd #1 at 0x1f78b0: echo THM{<REDACTED>} > test.txt
Cmd #2 at 0x21dcf0: cls
Cmd #3 at 0x1fe3c0: cd /Users
Cmd #4 at 0x1fe3e0: cd /John
Cmd #5 at 0x21db30: dir
Cmd #6 at 0x1fe400: cd John
----
Screen 0x200f70 X:80 Y:300
Dump:
                                                                                
C:\>cd /Users                                                                   
                                                                                
C:\Users>cd /John                                                               
The system cannot find the path specified.                                      
                                                                                
C:\Users>dir                                                                    
 Volume in drive C has no label.                                                
 Volume Serial Number is 1602-421F                                              
                                                                                
 Directory of C:\Users                                                          
                                                                                
12/27/2020  02:20 AM    <DIR>          .                                        
12/27/2020  02:20 AM    <DIR>          ..                                       
12/27/2020  02:21 AM    <DIR>          John                                     
04/12/2011  08:45 AM    <DIR>          Public                                   
               0 File(s)              0 bytes                                   
               4 Dir(s)  54,565,433,344 bytes free                              
                                                                                
C:\Users>cd John                                                                
                                                                                
C:\Users\John>            
```

We found the command that is written on command prompt!  


## Question 4 | What is the TrueCrypt passphrase?
**Description:**  
A common task of forensic investigators is looking for hidden partitions and encrypted files, as suspicion arose when TrueCrypt was found on the suspect's machine and an encrypted partition was found. The interrogation did not yield any success in getting the passphrase from the suspect, however, it may be present in the memory dump obtained from the suspect's computer.

---

Volatility2 have a plugin for truecrypt, it can get the passphrase stored in memory, so the first thing to is to get the profile to use for volatility2.

```
Volatility Foundation Volatility Framework 2.6.1
*** Failed to import volatility.plugins.malware.apihooks (NameError: name 'distorm3' is not defined)
*** Failed to import volatility.plugins.malware.threads (NameError: name 'distorm3' is not defined)
*** Failed to import volatility.plugins.mac.apihooks_kernel (ImportError: No module named distorm3)
*** Failed to import volatility.plugins.mac.check_syscall_shadow (ImportError: No module named distorm3)
*** Failed to import volatility.plugins.ssdt (NameError: name 'distorm3' is not defined)
*** Failed to import volatility.plugins.mac.apihooks (ImportError: No module named distorm3)
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/media/sf_Documents/TryHackMe/memoryforensic/Snapshot14_1609164553061.vmem)
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf80002c4d0a0L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002c4ed00L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2020-12-27 13:41:31 UTC+0000
     Image local date and time : 2020-12-27 05:41:31 -0800
                                               
```

The suggested profile is Win7SP1x64 so im going to use that, as i say volatility2 have a plugin for truecrypt and one of its feature is to find and extract the passphrase.

```
python2 vol.py  -f <memory file> --profile <profile> truecryptpassphrase

```

Result:
```
┌──(root㉿kali)-[/home/kali/tools/volatility2/volatility]
└─# python2 vol.py  -f /media/sf_Documents/TryHackMe/memoryforensic/Snapshot14_1609164553061.vmem --profile Win7SP1x64 truecryptpassphrase
Volatility Foundation Volatility Framework 2.6.1
*** Failed to import volatility.plugins.malware.apihooks (NameError: name 'distorm3' is not defined)
*** Failed to import volatility.plugins.malware.threads (NameError: name 'distorm3' is not defined)
*** Failed to import volatility.plugins.mac.apihooks_kernel (ImportError: No module named distorm3)
*** Failed to import volatility.plugins.mac.check_syscall_shadow (ImportError: No module named distorm3)
*** Failed to import volatility.plugins.ssdt (NameError: name 'distorm3' is not defined)
*** Failed to import volatility.plugins.mac.apihooks (ImportError: No module named distorm3)
Found at 0xfffff8800512bee4 length 11: <REDACTED>
                                                                                                                                                             
┌──(root㉿kali)-[/home/kali/tools/volatility2/volatility]
└─# 

```

And we got the passphrase!

## Conclusion
Even though the room is a little bit old, In my opinion this is a good room for someone who has just the learned the basic of memory forensic, 

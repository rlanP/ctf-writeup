# VulnNet: Internal - Writeup

## Overview
Platform: TryHackMe  
Difficulty: Easy  
Description: VulnNet Entertainment learns from its mistakes, and now they have something new for you...
Type: Boot2Root/Privilege Escalation  

## Reconnaissance

Start with ports scanning using nmap to see all the open ports  
```
nmap -p- -T4 (MACHINE-IP)
```
`-p-` will scan all 65353 ports 
`-T4` sets the timing template to aggressive, making the scan significantly faster by reducing delays between probes and lowering timeouts.

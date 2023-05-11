---
layout: post
title:  "Custom Windows based security distribution"
date:   '2022-07-11'
author: Dan Kir
tags:   
  - security
  - windows
description: >-
  Custom Windows based security distribution
---

Windows based VM for CTFs and ad hoc security testing and development

Base OS is Windows 10 Professional N

Can make this VM available on request. The OVF and disk are 24 GB.

#### Folder for dangerous binaries
1. Create Malware folder at C:\malware

2. Exclude C:\malware folder from real-time scanning in Windows Defender.

3. Set as non executable

   ```bash
   icacls C:\malware /deny "Everyone:(OI)(IO)(X)"
   ```

#### Default Credentials
username: ixg

password: Password1

#### Software List
- 010editor
- 7zip
- Atom Text Editor
- Autopsy
- Burp Suite
- Bloodhound + Neo4j
- CantorDust (Ghidra Plugin)
- CCleaner
- Cyberchef
- Deep Sound
- Detect-it-Easy (DIE)
- dnSpyEx
- Firefox
- FTK Imager
- Ghidra
- Gimp
- gpg4win
- GRASSMARLIN
- Hayabusa
- IDA Free
- ImmunityDebugger (mona.py PyCommand)
- IrfanViewer
- Libreoffice
- Malcode Analyst Pack (MAP)
- Mimikatz
- MobaXterm
- NetworkMiner Free
- Notepad++
- O&O Shutup
- Ollydbg
- PEView
- PE Explorer
- PE Bear
- PE ID
- Putty
- Recuva
- Redline Fireeye
- Regshot
- SQLlitebrowser
- Sysinternals Suite
- Tor Browser
- UPX
- Veracrypt
- VMware VM Tools
- Volatility
- WELA
- Windows Terminal
- Wireshark
- x64dbg

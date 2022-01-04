Pre-class exercises
====================

Goals
------
- Allows students to take as much time as they want
- Helps students to connect with each other through group work
- Strengthens the understanding of technical details through follow up lectures in the later day

Non-goals
----------
- Raises students' levels for the lecture
    - The pre-class exercises offer an opportunity of self-guided learning for those who wish
- Replacing the contents of the lecture
    - The amount of learning through the 4-hour long lecture only is limited
    - To learn and retain new information, one should spend time with hands-on exercises
- Getting the right answers
    - The pre-class exercises are an opportunity for learning

Procedures
-----------
- The trainer will make groups with about 6 students
- Each group discusses and decides how to work on the exercises
- It is encouraged to ask questions to the trainer on Slack
- All exercises are real-world cases and explained in various articles online; do the necessary research
- No need to submit answers, although groups may be asked to talk about their finding, learning, or thoughts during the lecture
- It is not expected to be able to answer all exercises. If you could, you probably do not need to take the lecture
- Focusing on only the exercise(s) that interests you is perfectly fine
- There is no ordering requirement between exercises, although software setup used in the exercise 2 may be required for other exercises

Prerequisites
--------------
- Host: Windows + WSL, macOS on Intel Processors, x64 Ubuntu
- Virtual machine: x64 Windows or Linux is helpful for the exercise 1 but not required
    - Windows 10 ISO image can be [downloaded or created with the media creation tool](https://www.microsoft.com/en-us/software-download/windows10)
    - VMware Workstation Player, Pro, and Fusion is tested
    - VirtualBox and QEMU should work but are untested
    - Make sure to make the VM UEFI-based, as opposed to the legacy-BIOS-based. Some software uses the legacy-BIOS by default


Exercise 1: Analyzing the 3rd party built UEFI module used in game cheating
============================================================================

Tags
-----
Architecture (runtime drivers), UEFI driver code reading

Goals
------
- Familiarize yourself with source code and build procedures of 3rd party UEFI modules
    - Understanding of source code is very helpful for reverse engineering exercise
- Understand the impacts of the UEFI module-based hacking tools based on the threat models of both PC game security and general IT security

Introduction
-------------
UEFI BIOS is capable of executing modules that are built by non-original equipment manufacturer (OEM) entities. This allows, for example, the device owner to build and execute her modules from the USB thumb driver through the UEFI shell. Such modules started to be widely used in hacking communities such as game cheat developers and users due to its low technical hurdle.

[CRZEFI](https://github.com/rasyidabdulhalim/CRZAIMBOT) is one of such modules designed for cheating on the online game called Apex Legends.

Instructions
-------------
1. Build CRZEFI, which is the UEFI part of the project
    ```
    $ sudo apt update
    $ sudo apt install gnu-efi
    $ git clone https://github.com/rasyidabdulhalim/CRZAIMBOT.git
    $ cd CRZAIMBOT/CRZEFI/
    $ git checkout cd04d44a3fdfe38c83c38b96dbb14e810471f8aa
    $ make
    ...
    objcopy -j .text -j .sdata -j .data -j .dynamic \
        -j .dynsym  -j .rel -j .rela -j .reloc \
        --target=efi-rtdrv-x86_64 memory.so memory.efi
    $ ls
    Makefile  definitions.h  dummy.h  main.c  main.o  memory.efi  memory.so
    ```
2. Optionally, execute the module in a VM by following the instruction in the project
3. Explain the high-level purpose (functionality) of CRZEFI
4. Discuss impacts of similar attacks on game security. For example, why do cheat developers use UEFI modules, over kernel-mode modules?
5. Research and discuss possible countermeasures. For example, detection and/or prevention methods and their limitations if any.
6. Consider whether the same approach might be used by malware. And if so, in what attack scenarios with what limitations.


Exercise 2: Reverse engineering the UEFI module used in by malware
===================================================================

Tags
-----
Malware

Goals
------
- Familiarize yourself with UEFI modules and static reverse engineering tools
- Know what the real "UEFI malware" looks like

Introduction
-------------
The below OS layers, particularly UEFI BIOS, have become frequent attack targets in the last few years. This is due to improved security for the above OS layers making existing layers less attractive for attackers, but also, because of greater impacts when attacking UEFI BIOS is successful. For example, an attacker may be able to achieve the following if she was able to install malware into UEFI BIOS:
1. Persisting malware that survives from OS re-installation
2. Executing code without being monitored and deleted by most of security software
3. Bypassing security features implemented by an OS and hypervisor

MosaicRegressor is such a malware reported in 2020, which achieves (1) and (2) above using UEFI modules.

Instructions
-------------
1. Set up a static reverse engineering environment for UEFI BIOS modules
    - Ghidra + efiSeek (free, Windows, some Linux?)
    - IDA Pro + efiXplorer (Windows, macOS, Linux)
    - Binary Ninja + bn-uefi-helper (untested)
2. Reverse engineer SmmInterfaceBase and SmmAccessSub. Then, verify discoveries about their implementations with the [report](https://securelist.com/mosaicregressor/98849/)
    - Many API's details can be found in the [EDK2 repository](https://github.com/tianocore/edk2) or the [UEFI specification](https://uefi.org/sites/default/files/resources/)

Attachments
------------

| File             | SHA256                                                           |
|------------------|------------------------------------------------------------------|
| SmmInterfaceBase | c7c3e039700bc6072f84ff99ecb22557e460dcd2214539938a6a0ef73b9caa88 |
| SmmAccessSub     | b73df2299f1b61629d40e1896efdf170a6c6b44e3fd3f833fad081fcf08a3cbd |
| SmmReset         | 29759388b83c2141bdc224ce1ba348fe3778ffec86b2716bcd6eacc839363737 |
| Ntfs             | dfcdcabd576d8717dcc570a2820947e385f0e10bdb2d9a332e7a5823ea51b3ac |


Exercise 3: Reverse engineering a vulnerable SMM module
========================================================

Tags
-----
Architecture (Firmware File System, System Management Mode), attack surface, vulnerability

Goals
------
- Learn SMM from the perspective of attack surfaces in UEFI BIOS
- Understand the possible impacts of SMM vulnerabilities
- Discuss the possible mitigation approaches against similar vulnerabilities

Introduction
-------------
Part of UEFI modules run at the processor mode called System Management Mode (SMM), which is at the higher privilege level than an OS and a hypervisor. Those modules are referred to as SMM modules, and on System Management Interrupt (SMI), their SMI handlers are executed. The SMI handlers must verify any inputs that may be manipulated by lower privileges (such as the OS) before using them, historically, the SMM developers have been failing to do this. As a result, an attacker may be able to do the following if there is a vulnerability that allows arbitrary code execution on SMM:
1. Persisting malware that survives from OS re-installation
2. Executing code that is not visible from OS, hypervisor, or any software debuggers
3. Bypassing security features implemented by an OS and hypervisor

The attached file is a UEFI BIOS image containing an SMM module with such a vulnerability. This vulnerability (CVE-2021-26943) allows ring-0 code to modify contents of arbitrary physical memory locations including SMM code and data due to the lack of input validation from ring-0.

Instructions
-------------
1. Find the `NvmeSmm` SMM module from the UEFI image using [UEFITool](https://github.com/LongSoft/UEFITool). Extract its `PE32 image section` into a file and confirm its SHA256 value is `A9C762B13FC9C4156603D674C6DAEBC4369520D95ACB1D0FA976078D6F7F1003`.
2. Identify the handler for SMI 0x42.
3. Identify logic with incomplete input validation from the lower privilege controllable location within the SMI handler (ie, find the vulnerability).
    - No need of reading sub-command implementations. The problem exists before that.
4. Consider how to attack and how it could be abused (ie, what impact would be).
5. Consider how one can discover UEFI BIOS versions with the identical (vulnerable) SMM module.
    - Can be from the perspective of any of SMM module developers, security researchers, IT administrators in organizations
6. Consider how one can discover UEFI BIOS version with similar but new SMM vulnerable modules
    - Ditto

Attachments
------------

| File           | SHA256                                                           |
|----------------|------------------------------------------------------------------|
| UX360CA-AS.303 | A2288B225C9C5B3D0FC524CB6DA9A6131EB29A8A73D1FC528F4822841FDC34B9 |
| NvmeSmm        | A9C762B13FC9C4156603D674C6DAEBC4369520D95ACB1D0FA976078D6F7F1003 |


Exercise 4: Reverse engineering Windows malware that infects UEFI BIOS
=======================================================================

Tags
-----
Architecture (security mechanism), attach surface, vulnerability, malware

Goals
------
- Learn how UEFI BIOS is protected from unauthorized modification on the Intel platform
- Understand possible impacts of the malware that infects UEFI BIOS when protection mechanisms are not properly used

Introduction
-------------
BIOS is stored in a storage called an SPI flash that is different from HDD and SSD, and is read and executed by the platform on power-up. BIOS is highly security-sensitive as it contains the SMM modules that run at the high privilege and implementation of security features such as Secure Boot and the use of the Trusted Platform Module (TPM). For this reason, the platform, specifically, Platform Controller Hub (PCH), also known as a chipset, implements several registers that protect the SPI flash from modification outside SMM, and UEFI BIOS code written by OEMs and BIOS vendors must configure those registers before the OS starts to properly protect the flash. However, many OEMs and BIOS vendors have failed to do this historically, and thus, there have been many cases the SPI flash was writable from the OS or a hypervisor.

[Lojax](https://www.welivesecurity.com/2018/09/27/lojax-first-uefi-rootkit-found-wild-courtesy-sednit-group/) discovered in 2018 is the malware that exploits such vulnerabilities and modifies and injects malicious code into UEFI BIOS from kernel-mode.

Instructions
-------------
1. Study Lojax
    1. Explain functionalities of the following 3 bits by referring to the [specification of Intel 9 Series PCH](https://www.intel.com/content/www/us/en/products/docs/chipsets/9-series-chipset-pch-datasheet.html)
        - BIOSWE
        - BLE
        - SMM_BWP
    2. Explain under what condition ReWriter_binary.exe infects the SPI flash concerning those 3 bits
2. Consider and discuss how devices with the vulnerability used by Lojax could be identified and protected
3. Consider and discuss how new malware exploiting the same vulnerabilities can be detected and prevented

Attachment
-----------

| File            | SHA256                                                           |
|-----------------|------------------------------------------------------------------|
| ReWriter_binary | fc28ad61fc748c08e8714cb247e741b736ebf0d9dfbcc3579f66fe3168326f61 |
| SecDxe          | 7ea33696c91761e95697549e0b0f84db2cf4033216cd16c3264b10daa31f598c |

- Do NOT run ReWriter_binary on a physical machine. It is a Windows executable.

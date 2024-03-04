Technical-Background
====================

UEFI
====

The (Unified) Extensible Firmware Interface or (U)EFI is a software interface between an operating system and the platform firmware. In contrast to BIOS based firmware that takes 64kb space and uses a 16-bit processor mode, (U)EFI is 32-bit or 64-bit, allows use of this full range of memory, and in theory positions itself as platform-independent. However, reality is different and achieving a full compatibility to all platforms is impossible.

* * *

Clover
======

Clover is an operating system boot loader for computers already equipped with an UEFI firmware and for those equipped with legacy BIOS firmware. An operating system (OS) may support (U)EFI (macOS, Windows 7, 8, or 10, Linux) or not (Windows XP). Legacy boot is used for the last one, that is, the old BIOS system is used to handle boot sectors.

(U)EFI is not only present during the booting of an OS, but it also creates tables and services that are accessible to the OS, and the operability of the OS depends on the correct functionality of (U)EFI. It is not possible to boot macOS from the built-in UEFI. Neither is it possible to boot macOS with the original DUET firmware emulation. CloverEFI firmware emulation and CloverGUI take care of a great amount of tasks to correct the internal tables and provide a possibility to run macOS.

* * *

Clover's tasks
==============

1.  SMBIOS (DMI) is filled with data emulating a real Apple Macintosh - a requirement for running macOS. Serial numbers are fake, but valid.
2.  ACPI tables - contained in the PC's ROM - are usually not written properly and may contain bugs, mostly because the manufacturer was lazy: an incorrect CPU core count in APIC table, NMI data is missing, missing reset register in table FACP, wrong power profile, missing EIST data in SSDT tables, and it is better to not even mention the DSDT table. Clover attempts to fix these problems.
3.  Further OS X tries to obtain data from the boot loader describing additional devices like the video, ethernet or sound card through so called EFI strings. Clover generates such data.
4.  BIOS-based computers will use USB in legacy mode during the initial boot process, which becomes a problem when passing control to the OS. Clover will change the USB mode.
5.  macOS uses a special memory called NVRAM for information exchange that is included in RuntimeServices (not present in a legacy loader). Clover provides this kind of information exchange, enabling correct Firewire functionality and the use of the Startup Disk preference panel. Additionally NVRAM is used for registration of the iCloud and iMessage services.
6.  ConsoleControl protocol is a necessity and is absent in DUET.
7.  It is necessary to fill certain data in EFI/Platform through the DataHub protocol, which is absent in DUET and not always present in UEFI. Furthermore, the utterly important FSBFrequency value, which sometimes is wrong or completely missing, is set.
8.  The CPU must be correctly initialized before working, but as motherboards are made universal to match a big amount of different CPUs, the internal tables do not contain any correct CPU data. Clover performs a full detection of the installed CPU, corrects the tables and the CPU itself. One side effect is a working turbo mode.
9.  One more small thing: DUET and EDK2 sources are written universally to match different hardware but the hardware dependency itself depends on constants. This implies a compilation process for one specific platform. Clover aims to be universal and to provide an automatic platform detection.

* * *

[Back on top ↑](https://github.com/CloverHackyColor/Clover-Documentation?tab=readme-ov-file#welcome-to-the-cloverbootloader-documentation-clover-documentation-site)

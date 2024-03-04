Installing
==========

Using the installer
-------------------

Each Clover revision contains file Clover\*\*\*.pkg which is macOS executable installer to make all work automatically. ![](https://user-images.githubusercontent.com/485515/167265123-033556f5-80a4-4ecc-a1a8-8949e745d950.png)

If you want to make this manually then read carefully.

Install for legacy boot
-----------------------

When you power on your computer you see BIOS which want to start some operating system. Old computers (legacy computers) have legacy BIOS which is able to boot some drive HDD, CDROM or USB-HDD. The BIOS read first sector (MBR) from the physical drive into memory and start is as a program written in 16bit codes. The program is less then 512 byte. It named boot0 boot sector. The program boot0 searches the partition table of the drive, finds first partition position (PBR) on the drive and reads there first one or two sectors named boot1. Start the program. The program PBR intended to use on the specific file-system. So use boot1hfs on HFS+ filesystem, use boot1f32 on FAT32 filesystem, boot1ex on exFAT file system and more. After years of investigations we decided to choose one case: drive must be formatted to GPT and have first partition EFI formatted to FAT32. Partition signature will be EF00. We have several variants but recommended one is **boot0af** It can be installed from macOS by command

`sudo fdisk440 -f boot0 -u -y /dev/rdisk0`

or by

dd if\=/dev/rdisk0 count\=1 bs\=512 of\=origMBR
cp origMBR newMBR
dd if\=boot0 of\=newMBR bs\=1 count\=440 conv\=notrunc
dd if\=newMBR of\=/dev/rdisk0 count\=1 bs\=512

But the disk should not be system as macOS protect system disk from modifications.

Sector PBR must be updated by commands

dd if\=/dev/rdisk0s1 count\=1 bs\=512 of\=origbs
cp boot1f32alt newbs
dd if\=origbs of\=newbs skip\=3 seek\=3 bs\=1 count\=87 conv\=notrunc
dd if\=newbs of\=/dev/rdisk0s1 count\=1 bs\=512

Install for UEFI boot
---------------------

Modern computers have UEFI BIOS which is much larger then legacy BIOS and able to start some EFI-applications from some drives. Usually an UEFI BIOS searches /EFI/BOOT/BOOTX64.EFI. Some BIOSes tuned to search /EFI/Microsoft/Boot/bootmgfw.efi Some BIOSes have own interface to tune what application to start. As we want Clover then we have to mimic to the BIOS-known variant. First one we copy

`CLOVERX64.EFI -> /EFI/BOOT/BOOTX64.EFI`

Second case we make two copies

`/EFI/Microsoft/Boot/bootmgfw.efi -> /EFI/Microsoft/Boot/bootmgfw-orig.efi`

and

`CLOVERX64.EFI -> /EFI/Microsoft/Boot/bootmgfw.efi`

Clover knows what is bootmgfw-orig.efi and is able to start it if we choose to start Windows from Clover. Moreover Clover is able to write itself as boot option in UEFI BIOS to start the file

`/EFI/CLOVER/CLOVERX64.EFI`

and this is recommended configuration because BOOTX64.EFI and bootmgfw.efi will be rewritten at the next Windows update while CLOVER will stay forever. See picture ![](https://user-images.githubusercontent.com/485515/167264824-e4b45fa2-ca85-4f6c-a505-5ef99f1c8acc.png) Second button in low row will perform the writing /EFI/CLOVER/CLOVERX64.EFI into BIOS as first boot option.

To do this manually you may launch Shell.efi (this is the first button in low row at the Clover GUI).

\> fs0:
> cd \\EFI\\CLOVER
> bcfg boot add 0 cloverx64.efi "start Clover"
> exit

More options about bcfg command you may see by the command

\> help bcfg boot

* * *

[Back on top ↑](https://github.com/CloverHackyColor/Clover-Documentation?tab=readme-ov-file#welcome-to-the-cloverbootloader-documentation-clover-documentation-site)

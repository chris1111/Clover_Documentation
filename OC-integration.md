OC-integration
==============

**Rev 5119 commit 620401d**

Included [OcQuirks](https://github.com/ReddestDream/OcQuirks) and [OpenRuntime](https://github.com/acidanthera/OpenCorePkg/tree/master/Platform/OpenRuntime) into Clover repository.

The drivers will be built at Clover compilation.

**commit 60901993b**

OcQuirks driver and plist now excluded. Now Quirks included into config.plist as separate section

* * *

Quirks: ⇩
---------

\## Config.plist Quirks structure

Set default and most recommended values
<key\>Quirks</key\>
        <dict>
            <key\>AvoidRuntimeDefrag</key\>
            <true/>                            -- recommended for all except native Apple
            <key\>DevirtualiseMmio</key\>
            <false/>                           -- may be useful for Z390
            <key\>DisableSingleUser</key\>
            <false/>                           -- prohibit using Single User Mode. If you are paranoiac.
            <key\>DisableVariableWrite</key\>
            <false/>                           -- prohibit write to NVRAM. If you are paranoiac.
            <key\>DiscardHibernateMap</key\>
            <false/>                           -- in some rare cases memory after hibernation can be 
                                                  wrong mapped. Usually no.
            <key\>EnableSafeModeSlide</key\>
            <false/>                           -- safe mode (-x) by default will use slide=0. You may set
                                                  other value if enable. Mojave crashes with <true\>. Ventura works.
            <key\>EnableWriteUnprotector</key\>
            <true/>                            -- clear memory protect bit. See note below.
            <key\>ForceExitBootServices</key\>
            <false/>                           -- never used
            <key\>MmioWhitelist</key\>
            <array>                            -- list of memory regions to protect. Only good hackers 
                                                  know why.
                <dict>
                    <key\>Address</key\>
                    <integer\>4275159040</integer\>
                    <key\>Comment</key\>
                    <string\>Haswell: SB\_RCBA is a 0x4 page memory region, containing SPI\_BASE at 0x3800 (SPI\_BASE\_ADDRESS)</string\>
                    <key\>Enabled</key\>
                    <false/>
                </dict>
                <dict>
                    <key\>Address</key\>
                    <integer\>4278190080</integer\>
                    <key\>Comment</key\>
                    <string\>Generic: PCI root is a 0x1000 page memory region used by some firmwares</string\>
                    <key\>Enabled</key\>
                    <false/>
                </dict>
            </array>
            <key\>ProtectMemoryRegions</key\>
            <false/>                          -- set true if you use CSM, if your videocard if not UEFI
            <key\>ProtectSecureBoot</key\>
            <false/>                          -- rarely used with InsideH2O 
            <key\>ProtectUefiServices</key\>
            <false/>                          -- needed for VMWare if it will be used with hackintosh 
                                                 bootloader, usually no.
            <key\>ProvideCustomSlide</key\>
            <false/>                          -- if you see OCABC: Only N/256 slide values are usable
                                                 then set this to true
            <key\>ProvideMaxSlide</key\>
            <integer\>0</integer\>              -- for the case above set a value between 1 and 254.
                                                 Never used.
            <key\>RebuildAppleMemoryMap</key\>
            <false/>                          -- see note below
            <key\>ResizeAppleGpuBars</key\>
            <integer\>-1</integer\>             -- only for RX6800 if set in BIOS Resizable Pci Bar,
                                                 which is incompatible with MacOS
            <key\>SetupVirtualMap</key\>
            <true/>                           -- always true
            <key\>SignalAppleOS</key\>
            <false/>                          -- always false. May be assumed for OpenCore that 
                                                 cant start windows.
            <key\>SyncRuntimePermissions</key\>
            <true/>                           -- always true
            <key\>FuzzyMatch</key\>
            <false/>                          -- for system Snow Leopard it may be true
            <key\>KernelCache</key\>
            <string\>Auto</string\>             -- (Auto, Cacheless, Mkext, Prelinked) for system 
                                                 Lion (10.7). No more.
            <key\>AppleXcpmExtraMsrs</key\>
            <false/>                          -- support for XCPM for non-native CPU like Haswell-E 
            <key\>AppleXcpmForceBoost</key\>
            <false/>                          -- set maximum P-state for CPU, maximum frequency.
                                                 Install water cooler for this case
            <key\>DisableIoMapper</key\>
            <false/>                          -- switch off VT-d. It will be better to drop or 
                                                 replace DMAR table. Dont set this if you want Monterey
            <key\>DisableLinkeditJettison</key\>
            <true/>                           -- fix Lilu bug
            <key\>DummyPowerManagement</key\>
            <false/>                          -- block AppleIntelCpuPowerManagement because you have  
                                                 locked 0xE2. Should be resolved other way.
            <key\>ExternalDiskIcons</key\>
            <false/>                          -- usual patch from KextPatches section 
                                                 changing "External" -> "Internal"
            <key\>IncreasePciBarSize</key\>
            <false/>                          -- never use
            <key\>PowerTimeoutKernelPanic</key\>
            <false/>                          -- never use
            <key\>ThirdPartyDrives</key\>
            <false/>                          -- usual patch "Enable Trim on Non-Apple."
                                                 changing "Apple" -> ""
            <key\>XhciPortLimit</key\>
            <false/>                          -- usual patch extended maximum XHCI ports because 
                                                 Apple has limit to 15. Don't do this! 
                                                 The extended limit will break the system.
                                                 Create LegacyUSB.kext to use 15 ports without 
                                                 needless ports.
            <key\>ProvideCurrentCpuInfo</key\>
            <false/>                          -- dangerous patch for the AlderLake CPU. 
                                                 Use if needed with precautions.
        </dict>

Note! If you system contains MATS table then you should set such quirks

EnableWriteUnprotector - False
RebuildAppleMemoryMap - True
SyncRuntimePermissions - True

Otherwise contrary.

* * *

OpenRuntime: ⇩
--------------

Booting macOS 11 and macOS 12 OpenRuntime.efi is need.

![](https://user-images.githubusercontent.com/6248794/164318871-6bc99f4d-5212-4430-98ba-3768a92cf7b8.png)

**Rev 5142+** Using new OpenRuntime.efi.

There is incompatibility with older version. Clover before 5142 requires OpenRuntime.efi version 1.1 while 5142+ requires version 1.2.

What to do if you sometime want to reboot with older version?

![](https://user-images.githubusercontent.com/6248794/164320794-c278aaa1-a91c-47dc-90c4-9791b420c118.png)

[Slice](https://www.insanelymac.com/forum/topic/304530-clover-change-explanations/page/8/#comment-2776376) made special folder drivers/5142 where OpenRuntime-v12.efi is placed. And common folder drivers/UEFI contains OpenRuntime-v11.efi.

**How it works ⤵︎**

Old Clover not see the folder 5142 and uses common folder drivers/UEFI and read here openRuntime-v11.efi.

New Clover see the folder drivers/5142 and read here new OpenRuntime-v12.efi which is in priority in the version.

That's all!

OpenCorePkg: ⇩
--------------

OpenCorePkg repos used by Clover ☞ [OpenCorePkg is used by Clover](https://github.com/CloverHackyColor/OpenCorePkg)

OpenCorePkg is not same as original. Based on OpenCore 0.7.5 it is improved up to latest version. Clover uses some part of the package delegating kext injecting and patching abilities to OC libraries. Big respect to vit9696 for these libraries. Clover keeps own syntax for kernel and kext patching and converted them to OC syntax [Conversion rulez](https://www.insanelymac.com/forum/topic/304530-clover-change-explanations/?do=findComment&comment=2740755) but the method for binary patching in the OC library is replaced by Clover's method because it has more possibilities.

Some new quirks implemented in new OC versions have no analogs in the Clover's config.plist because they are very specific for rare hardware. If you really need them then search how to do the same by common kext patching. Usually the new quirk is just a shortcut to one kernel or kext patch.

* * *

[Back on top ↑](https://github.com/CloverHackyColor/Clover-Documentaion?tab=readme-ov-file#welcome-to-the-cloverbootloader-documentation-clover-documentation-site)

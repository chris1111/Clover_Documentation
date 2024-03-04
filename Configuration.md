Configuration
=============

Creating a configuration file
=============================

Clover will perform an automatic configuration based on the computer's hardware. Nevertheless, an automatic unit is never perfect and this is why the user can permanently change several parameters in the configuration file config.plist or on the fly in the GUI. The configuration file is written in XML but it can be handy to view it as a text file. It can be edited by simple text editors or by plist editors like PlistEdit or Xcode. Clover is shipped with a version with all possible configuration options.

#### The configuration file (config.plist) must be put in the EFI/CLOVER folder.

_\*\*_ Here you will find the config-sample.plist used by the Clover package, you can adapt it to your hardware configuration.

*   View ➢ [config-sample.plist](https://github.com/CloverHackyColor/CloverBootloader/blob/master/CloverPackage/CloverV2/EFI/CLOVER/config-sample.plist) Download ➢ [config-sample.plist](https://raw.githubusercontent.com/CloverHackyColor/CloverBootloader/master/CloverPackage/CloverV2/EFI/CLOVER/config-sample.plist)

* * *

This way you get a nearly complete configuration file with the parameters used to successfully load the OS. Some more manual work is needed to finish it.

* * *

Config.plist structure
======================

ACPI: ⇩
-------

<key\>ACPI</key\>
<dict\>
...
</dict\>

Parameter group affecting various corrections of ACPI tables. This is a rather complex topic. There are several versions of ACPI specifications and additionally Mac has its own requirements. Often vendors are too lazy to write proper tables and internal devices may not be listed or CPU definitions are missing completely.

* * *

ResetAddress and ResetValue
===========================

<key\>ResetAddress</key\>
<string\>0x64</string\>
<key\>ResetValue</key\>
<string\>0xFE</string\>

These two parameters serve a very important purpose: to fix restart. These values theoretically should be in the _FADT_ table, but it is not always the case. Furthermore, FADT may be shorter than required and not contain them at all. Default values are `0x64`/`0xFE`, which means a restart through the PS2 controller. However, this does not work on every system and you can alternatively use `0x0CF9`/`0x06`, which indicates a restart though the PCI rail. This is the default value for real Macs but does not always work on a hackintosh. The difference is clear: a hackintosh additionally has a PS2 controller, which may prevent rebooting, if not disabled. Last but not least you can set them to `0x0`/`0x0` to allow the use of default FACP values. If not present, the default values states above will be used instead.

* * *

smartUPS
========

<key\>smartUPS</key\>
<true/>

This parameter affects the power profile, which will be written into table FADT.

Value|Type     |Power supply type 
---------------------------------
    1|Desktop  |AC 
    2|Mobile   |Battery 
    3|Server   |SmartUPS

Clover will choose between 1 and 2 according to the mobility bit and according to the `Mobile` parameter in SMBIOS. It is, for example, possible to fake a mobile MacMini. Value 3 will be chosen if this parameter is enabled.

* * *

PatchAPIC
=========

<key\>PatchAPIC</key\>
<false/>

Some systems can either be started using the kernel parameter `cpus=1`, or by using a patched kernel (Lapic NMI). It turns out that in these case the table MADT is incomplete and missing the NMI section. Enabling this parameter will cause Clover to automatically correct this table. If the table already is complete, then nothing will be changed.

* * *

HaltEnabler
===========

<key\>HaltEnabler</key\>
<true\>

It works as OpenHaltRestart, clearing SLP\_SMI\_EN at start of OSX system.

* * *

UseSystemIO
===========

<key\>UseSystemIO</key\>
<true\>

Key UseSystemIO at SSDT section will serve to choose in the generated \_CST tables between:

Register (FFixedHW,
Register (SystemIO,

* * *

DSDT
====

<key\>DSDT</key\>
<dict\>
...
</dict\>

Parameter group affecting DSDT.

* * *

DSDT / Name
-----------

<key\>Name</key\>
<string\>DSDT.aml</string\>

File name of the binary DSDT file to load and inject. If you dont want to load extra DSDT you may set BIOS.aml and the DSDT from BIOS will be used.

* * *

DSDT / Fixes
------------

<key\>Fixes</key\>
<dict\>
...
</dict\>

See separate section for fixing DSDT

* * *

DSDT / Patches
--------------

Binary DSDT patching

<key\>Patches</key\>
  <array\>
    <dict\>
      <key\>Find</key\>
      <data\>W4IeQkFUMQhfSElEDEHQDAoIX1VJRAEUCF9TVEEApAA=</data\>
      <key\>Replace</key\>
      <data\></data\>
    </dict\>
    <dict\>
      <key\>Find</key\>
      <data\>UFhTWAhfQURSAAhfUFJXEgYC</data\>
      <key\>Replace</key\>
      <data\>UFhTWAhfQURSAAhfU1VOCgQIX1BSVxIGAg==</data\>
    </dict\>
  </array\>

**Rev 4314**

Limit scope of binary DSDT patch by "Device" section with specified name in additional "TgtBridge" tag by goodwin\_c.

Config.plist sample:

 <dict\>
    <key\>Comment</key\>
    <string\>Rename PXSX to SSD0</string\>
    <key\>Disabled</key\>
    <false/>
    <key\>Find</key\>
    <data\>UFhTWA==</data\>
    <key\>Replace</key\>
    <data\>U1NEMA==</data\>
    <key\>TgtBridge</key\>
    <data\>UlAwOQ==</data\>
</dict\>
<dict\>
    <key\>Comment</key\>
    <string\>Rename PXSX to SSD0 2</string\>
    <key\>Disabled</key\>
    <false/>
    <key\>Find</key\>
    <data\>UlAwOS5QWFNY</data\>
    <key\>Replace</key\>
    <data\>UlAwOS5TU0Qw</data\>
    <key\>Skip</key\>
    <integer\>0</integer\>
</dict\>

**Rev 4468**

More exact renaming Devices in DSDT and SSDT taking into account its bridge.

Example:

<key\>RenameDevices</key\>
        <dict\>
            <key\>\_SB.PCI0.RP02.PSXS</key\>
            <string\>ARPT</string\>
            <key\>\_SB.PCI0.EHC1</key\>
            <string\>EH01</string\>
            <key\>\_SB.PCI0.POP2.PEGP</key\>
            <string\>GFX0</string\>
        </dict\>

Complex case such as this DSDT is also taken into account.

\_SB.PCI0.RP02.PSXS
We have to take into account fields like
Scope(\\\_SB)
{
  Device (PCI0)
  {
    Device(RP02)
    {
      Device(PSXS)  <- to patch
      {
        Method(\_ON)
        {
        }
        Method(\_OFF)
        {
        }

      }
      PSXS.\_ON() <- to patch
    }
    Scope(RP02)
    {
      PSXS.\_OFF() <- to patch
    }
    Device(RP03)
    {
      Device(PSXS) <- to not patch
      {
      }
      PSXS.\_ON() <- to not patch
    }

* * *

DSDT / SuspendOverride
----------------------

<key\>SuspendOverride</key\>
<true/>

Influences the DSDT patch `FixShutdown` and extends the fix from state 5 to 3, 4 and 5 (sleep and suspend).

* * *

DSDT / ReuseFFFF
----------------

<key\>ReuseFFFF</key\>
<true\>

Some OEM DSDT contains some device with `Name (_ADR, 0xFFFF)`. This is a big problem as we can convert it to ADR=0 and inject properties but this is dangerous patch, it may lead to panic on IOPCIFamily.kext. So this key is proposed which will convert this device to (ADR, 0) and reused for injection. (FakeID for example)

* * *

DSDT / PNLF\_UID
----------------

<key\>PNLF\_UID</key\>
<string\>0x0A</string\>

Assign the value for your PNLF device. If you have no PNLF device then you should use DSDT fix AddPNLF. The \_UID value changes it behaviour for brightness control. Each UID corresponds to some MacBook value. If you have exactly the same screen then use the value from those book. If unsure then don't use UID substitution. Clover knows what to do.

* * *

DSDT / Rtc8Allowed
------------------

<key\>Rtc8Allowed</key\>
<false\>

Some users claim that RTC length may be 8 bytes without CMOS reset, but others claim that the reset is still occurring. For those who wants len=8 it should be set to true. Default is false.

* * *

SSDT
====

<key\>SSDT</key\>
<dict\>
...
</dict\>

Parameter group affecting SSDTs.

* * *

SSDT / DropOem
--------------

<key\>DropOem</key\>
<true/>

Drops all internal SSDT tables to avoid conflicts when generating an SSDT for your processor, which contains P- and C-States. Clover can do this automatically or you can specify an external file, which will be loaded from EFI/OEM/\[model\]/ACPI/patched.

* * *

SSDT / Generate
---------------

<key\>Generate</key\>
<false/>

Generate an SSDT with p-states and c-states.

* * *

### SSDT / Generate / CStates

<key\>Generate</key\>
<dict\>
  <key\>CStates</key\>
  <true/>
</dict\>

Automatic SSDT table generation, which extends the processor section with _\_CST_methods for each core. _\_CST_ generation is affected by parameters `EnableC2`, `EnableC4`, `EnableC6`, `EnableISS`, `C3Latency`. There is no need to comment them as everything will work either way. Experiment by yourself. Besides, Clover already has obtained the processor type and core count. Not using this parameter will result in following error message:`ACPI_SMC_PlatformPlugin::pushCPU_CSTData - _CST evaluation failed.`

* * *

### SSDT / Generate / PStates

<key\>Generate</key\>
<dict\>
  <key\>PStates</key\>
  <true/>
</dict\>

Automatic SSDT table generation, which extends the processor section with _\_PPC_, _\_PPC_ and _\_PSS_ methods.

*   `_PCT` - _Performance control_. Controls SpeedStep functions
*   `_PPC` - _Performance Present Capabilities_. SpeedStep capabilities. This method returns a value limiting the frequency. Look further for `PLimitDict`.
*   `_PSS` - _Performance Supported States_. An array containing possible CPU states - P-States. `PLimitDict`, `UnderVoltStep` and `Turbo` will be taken into consideration when generated this array.

* * *

SSDT / EnableC2
---------------

<key\>EnableC2</key\>
<true/>

This key allows you to enable the C2 states generator. Disabled by default.

* * *

SSDT / EnableC4
---------------

<key\>EnableC4</key\>
<true/>

This key allows you to enable the C4 states generator. Disabled by default.

* * *

SSDT / EnableC6
---------------

<key\>EnableC6</key\>
<true/>

This key allows you to enable the C6 states generator.

* * *

SSDT / EnableC7
---------------

<key\>EnableC7</key\>
<true/>

This key allows you to enable the C7 states generator. Disabled by default.

* * *

SSDT / PLimitDict
-----------------

<key\>PLimitDict</key\>
<integer\>1</integer\>

Limits the maximal CPU frequency.

*   `0` - No limit.
*   `1` - Reduce frequency by one step
*   `2` - Reduce frequency by two steps

Example: A Core2Duo T8300 with 2400 MHz operates at a maximal frequency of 2000 MHz when limited by two steps. This parameter might be used to reduce heating in mobile systems.

The same parameter exists in platform plists, for example in:_System/Library/Extensions/IOPlatformPluginFamily.kext/Contents/PlugIns/ACPI\_SMC\_PlatformPlugin.kext/Contents/Resources/MacBook5\_1.plist_. They will be discussed later.

Other CPUs may need other values. This value has a reversed effect on a Core2Quad for instance. The optimal value is `1` in this case. It might be a DSDT error, though.

* * *

SSDT / UnderVoltStep
--------------------

<key\>UnderVoltStep</key\>
<integer\>1</integer\>

This parameter lowers the CPU voltage and indirectly affects the temperature. Possible values are `0`, `1`, `2`, etc. Clover will only allow sane values, meaning it is safe to increase this value until the CPU stops working correctly.

* * *

SSDT / MinMultiplier
--------------------

<key\>MinMultiplier</key\>
<integer\>7</integer\>

Minimal CPU multiplier. Usually a value of 16 is ported, resulting in a frequency of 1600 MHz but you should use lower values when using SpeedStep, like `8` or even `7`.

* * *

SSDT / MaxMultiplier
--------------------

<key\>MaxMultiplier</key\>
<integer\>30</integer\>

Introduces as an analogy to the minimal multiplier but not really necessary. It is not advised to set it.

* * *

DropTables
==========

<key\>DropTables</key\>
<array\>
  <dict\>
    <key\>Signature</key\>
    <string\>SSDT</string\>
    <key\>TableId</key\>
    <string\>SataTabl</string\>
  </dict\>
  <dict\>
    <key\>Signature</key\>
    <string\>BGRT</string\>
  </dict\>
  <dict\>
    <key\>TableId</key\>
    <string\>A M I</string\>   -- WARNING: DO NOT DO THIS!!!!!!!
  </dict\>
  ...
</array\>

Drop OEM ACPI tables by signature and/or table identifier.

Additionally, now you can drop tables by their length. Why? Because we found Acer laptop where all SSDT have the same TableID.

<key\>DropTables</key\>
<array\>
  <dict\>
    <key\>Signature</key\>
    <string\>SSDT</string\>
    <key\>Length</key\>
    <integer\>720</integer\>
  </dict\>
</array\>

* * *

FixHeaders
==========

**Rev 4427-4429**

FixHeaders should be in common ACPI section, it is not DSDT-only fix.

<key\>FixHeaders</key\>
<true/>

The fix is recommended to all users even if you are not going to fix DSDT. Anyway the fix is safe. The reason for the fix is sometimes an ACPI table to have a name with national characters which is impossible for macOS.

Boot: ⇩
-------

Arguments
=========

<key\>Arguments</key\>
<string\>ARGUMENTS</string\>

At this section you can add "Boot Flags" and "Kernel Flags" to be used by the system... Here we will list only Clover's proprietary "Boot Flags", different flags, like for example, npci=0x2000, npci=0x3000, darkwake=0, etc..., should work as expected...

*   `-v` - Verbose Boot.
*   `-s` - Boot OS X into Single User Mode.
*   `-x` - Boot OS X into SafeBoot (Safe Mode).

* * *

Legacy
======

<key\>Legacy</key\>
<string\>PBR</string\>

Necessary for loading old versions of Windows and Linux. Greatly depends on hardware and BIOS. Several algorithms were developed to allow customisation:

*   `LegacyBiosDefault` - for UEFI BIOS variants containing protocol _LegacyBios_
*   `PBRtest` - PBR boot algorithm
*   `PBR` - Another PBR boot algorithm

Additionally for UEFI boot you may specify which HDD to boot (not only the first one).

<key\>Legacy</key\>
<string\>LegacyBiosDefault</string\>
<key\>LegacyBiosDefaultEntry</key\>
<integer\>2</integer\>

* * *

Timeout
=======

<key\>Timeout</key\>
<integer\>5</integer\>

The boot loader will pause for 5 seconds after starting before automatically loading an operating system. If a key is pressed during this period of time, the countdown is stopped. Options:

*   `0` - GUI is not invoked, the OS is loaded instantly
*   `-1` - GUI is invoked, automatic OS loading is turned off

Clover must have be able to find a default entry for it to automatically boot into an OS, see below.

_NOTE:_ If the Timeout integer is set to `0`, hold any key when powering on to invoke the bootloader GUI.

* * *

DefaultVolume
=============

<key\>DefaultVolume</key\>
<string\>LastBootedVolume</string\>

DefaultVolume is used to specify which entry is the default boot entry in Clover GUI. See also \[DefaultLoader\]. It can be set to:

*   Volume Name - The name of the volume. E.g. `Macintosh`.
*   GUID - Globally Unique ID of the volume shown in Clover's boot, preboot or debug log. E.g. `57272A5A-7EFE-4404-9CDA-C33761D0DB3C`.
*   Part of Device Path - Also shown in Clover's logs. E.g. `HD(1,GPT,57272A5A-7EFE-4404-9CDA-C33761D0DB3C,0x800,0xFF000)`.
*   `LastBootedVolume` - The last booted volume will be set as default one in Clover GUI.

OS X Startup Disk can be used to reboot into another volume, but for the following reboot `DefaultVolume` will be used again.

* * *

DefaultLoader
=============

<key\>DefaultLoader</key\>
<string\>boot.efi</string\>

In addition to \[DefaultVolume\] above, the path of the loader can be specified as DefaultLoader. This provides more precise default entry selection for Volumes that have multiple Loaders. The value can be the complete path or a unique portion like like file name.

* * *

Debug
=====

**Formerly known as "Log" before revision 3064**

<key\>Debug</key\>
<false/>

If you are unable to boot into the Clover you can use this setting to produces a debug output to `/EFI/CLOVER/misc/debug.log`. This log then can be assessed to find out what the problem is. **!WARNING** turning on this log will dramatically increase loading time as it has to write the log to the disk as it goes. So please be patient and give plenty of time before reseting your computer.

* * *

Fast
====

<key\>Fast</key\>
<true/>

Similar to setting `Timeout` to `0` but: _nvram.plist will be searched only until first occurrence_ does not search for the best video mode _does not load themes and graphics_ no chance to enter the GUI

Having this parameter enabled might not really have any big effect. Instead, you can try a custom compilation: _disable GUI in CloverEFI (3 seconds difference)_ only search for Sata0 drivers (9 seconds difference)

* * *

XMPDetection
============

<key\>XMPDetection</key\>
<true/> OR <false/> OR <string\>Yes/No</string\>

Detect best eXtreme Memory Profile when detecting memory or disable XMP detection.

<key\>XMPDetection</key\>
<integer\>1/2</integer\>

* * *

Secure
======

<key\>Secure</key\>
<true/>

Secure Boot protocol, a controversial subject, will restrict the booting process to signed binaries only. By enabling this option in your BIOS, booting of many operating systems won't be possible anymore, except Windows 8, 2013 and some Linux distributions.

The aim is to use Clover while Secure Boot is enabled. Clover's binary will need to be signed and a certificate to be loaded.

* * *

NeverHibernate
==============

<key\>NeverHibernate</key\>
<true/>

Disables the hibernation state detection.

* * *

CustomLogo
==========

<key>CustomLogo</key>
<true/> OR <false/\> OR <string>Apple/Alternate/Theme/None/Path</string> OR <data>PNG/BMP/ICNS base64 data</data>

Enable the drawing of the custom boot logo.

*   `true` - Uses the default boot style, Apple.
*   `false` - Disables custom boot logo.
*   `Apple` - Use the default gray on gray apple logo.
*   `Alternate` - Use the alternate white on black apple logo.
*   `Theme` - Use the theme boot screen for entry type - NOT IMPLEMENTED.
*   `None` - Use no logo only background color, gray if not specified by custom entry.
*   `Path` - A file path to load a custom image from
*   `<data>` - A base64 encoded PNG, BMP, or ICNS data.

If no option is specified then the boot screen will be drawn only for >= 10.10 Yosemite, so it remains compatible with previous behavior.

The `CustomLogo` key can also be used under GUI/Custom/Entries in conjunction with BootBgColor for a different screen for every OS. However, the graphics output protocol is not in anyway modified so if the OS draws after it is started then it is after the boot screen is drawn and will overwrite the custom logo, at least for now.

* * *

HibernationFixup
================

**Rev 4223**

<key\>HibernationFixup</key\>
<true/>

To be used for Hibernation modes 25 & 3 with Lilu.kext and HibernationFixup.kext.

* * *

RtcHibernateAware
=================

**Rev 4450**

A data leak issue was identified in the hibernation code, allowing hibernation encryption key to be passed to the system through RTC and preserved till the next hibernation without a subsequent erase. More details could be found in [this message](https://www.insanelymac.com/forum/topic/329777-clover-problems-report-features-request/?do=findComment&comment=2611823). To workaround this issue a new option enabling RTC memory erase upon waking from hibernation was added: `Boot` > `RtcHibernateAware` = YES (BOOLEAN, off by default)

This option relies on a poorly documented (or rather undocumented) RTC memory access, and unspecified RTC memory layout, which is implementation-specific. While it is extremely recommended to be turned on if you rely on hibernation, it may not work on your hardware (should be fine on Ivy Bridge and newer at least), and is thus optional and disabled by default.

Note, that AppleRTC or FixRTC patches effectively break hibernation by reducing the available RTC memory and avoiding encryption key preservation. You should DISABLE them if you have no issues with BIOS preferences afterwards or use HibernationFixup. However, if RtcHibernateAware does not work for you, enabling AppleRTC patch and using HibernationFixup may be a safer workaround.

**Revision 4515**

Revision 4450 introduced a new key (Boot → RtcHibernateAware) which improved the situation with hibernation compatibility and reduced the impact of some security issues in this process. Starting with 10.13.6 a lot of legacy code got (finally) ditched on Apple side, and some changes are necessary to get hibernation to work on hacks. Revision 4515 incorporates them in Clover.

Mandatory stuff: 1) Clover must be r4450 and newer 2) AptioMemoryFix must be R20 (b83c025) or newer 3) Boot → RtcHibernateAware must be set to YES

Some additional notes: — Config changes (and driver updates) will benefit all the systems (starting from Yosemite if I remember correctly), not just 10.13.6 — OsxAptioFix1/2/3 will NOT work with hibernation on 10.13.6 and newer, and they may be removed from the default Clover installer in the future (time to upgrade). — If you can boot with AppleRTC/FixRTC patches OFF without BIOS settings reset, disable them, they break stuff in macOS  
— If you have to use [HibernationFixup](https://github.com/acidanthera/HibernationFixup), you may want to update it to 1.2.1, currently only available from source.

Last, but not least: If Hibernation does not work for you… well, RIP. Believe me or not, but it is not something you should rely on. Especially if you care about security and privacy of your data.

* * *

SignatureFixup
==============

**Rev 4270**

<key\>SignatureFixup</key\>
<false/>

**Rev 4291**

If not SignatureFixup, it will be zeroed.

Anyway, the Signature must be fixed. If True then the value is from hibernate image If False then the value is zero and it will be copied by kernel into the image.

CPU: ⇩
------

<key\>CPU</key\>
<dict\>
...
</dict\>

This group of parameters helps with CPU definitions in case the internal algorithms fail.

* * *

FrequencyMHz
============

<key\>FrequencyMHz</key\>
<string\>3200</string\>

This number is used in the About This Mac window only and not influences on system functions.

* * *

BusSpeedkHz
===========

<key\>BusSpeedkHz</key\>
<integer\>133330</integer\>

Bus speed in kHz. Very important value for a stable system. It is passed from the boot loader to the kernel. **If this value is not correct, the kernel will not start at all. If it is slightly incorrect, clock issues may arise and the system will behave in a strange way.**

An automatic detection was introduced with revision 1060, which relies on the ACPI timer producing much more accurate values than the ones stored in DMI.

DMI stores this value in MHz, which is not accurate in contrast to a value calculated from CPU frequency. You can choose a more accurate value, if needed. For example my DMI has a value of 100 MHz, however manually overriding it to a value of 99790 kHz produced better clocks. Some vendors use a different meaning for BusSpeed and FSBSpeed and use values four times bigger in the BIOS. You can distinguish them according to the scale: either it is from 100 MHz to 400 MHz or it matches the formula CPUFrequency=RailFrequency\*CPUMultiplier. If ASUS uses a rail frequency of 1600 MHz and a multiplier of 8, the formula does not work - a CPU with 12,8 GHz does not exist; a division into 4 is required.

Attention: UEFI booting will produce an inaccurate value. It is recommended manually specify the value, which is calculated more accurately during a boot with Clover EFI.

* * *

UseARTFrequency
===============

<key\>UseARTFrequency</key\>
<true/>

In processors SkyLake and up there is new frequency parameter so called ARTFrequency and macOS will use it for calculating the bus speed. It may be 24MHz or sometimes other values. Clover will calculate it automatically and it will be more precise value then set by other methods.

* * *

QPI
===

<key\>QPI</key\>
<integer\>4800</integer\>

System Profiler calls it _Processor Bus Speed_ or _Bus Speed_. Chameleon has an algorithm for calculating this value for Nehalem CPUs (, which is however not correct). Clover has a corrected algorithm according to Intel datasheets. AppleSmbios sources describe two variants: either SMBIOS already contains this value as specified by the vendor, or it is calculated by the formulae BusSpeed\*4. After a long discussions this value was sourced into the configuration file - write what you want (in MHz). This is a purely cosmetic value. Apparently this value only makes sense for Nehalems, the rest should use the formula stated above - or nothing at all.

Note: Real Mac’s report a hw.busfrequency = 100000000

To achieve that with Clover here’s what to do: 1 - Drop SMBIOS table type 132 for Sandy Bridge and newer CPU’s. Clover does this if you set QPI to a string value of 0. 2 - Set SMBIOS table type 4->ExternalClock to 0 (or 25Mhz as a real Mac). This currently has to be done in the source code and re-compile Clover.

If you don’t do step 2 then for Sandy Bridge and newer CPU’s, AppleSMBIOS.kext will multiply any non zero values reported by SMBIOS table type 4 -> External Clock by 4. [See DHP’s posts for ref.](http://www.insanelymac.com/forum/topic/253642-dsdt-for-asus-p8p67-m-pro/?p=1679440)

* * *

Type
====

<key\>Type</key\>
<string\>0x0201</string\>

This result of this value can only be found in Apple's specification and it is used in the window _About this Mac_, which is displaying the according processor name. Otherwise "Unknown CPU" will be displayed. An invocation of CPUID was not possible due to PowerPC and due to Apple's different view of the world SMBIOS table 4 is not used either. Clover knows most values but due to the fact that hardware development does not stand still, you can specify this value. Again, this is purely cosmetic.

* * *

Latency
=======

<key\>Latency</key\>
<string\>0x03E9</string\>

This parameter value represents the C3 entry latency issued when entering C3 state. The critical value is `0x3E8`(1000). A lower value will allow SpeedStep, a higher one will not allow it. Real Macs always use `0x3E9`, meaning SpeedStep is not turned on. Decide for yourself what you need. Notebook users should use `0x00FA` to enable power management.

Devices: ⇩
----------

<key\>Devices</key\>
<dict\>
...
</dict\>

Parameter group for tweaking setting affecting PCI devices.

* * *

Properties
==========

<key>Properties</key>
<dict>
    <key>PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)</key>
    <dict>
        <key>@0,AAPL,boot-display</key>
        <data>
        AQAAAA==
        </data>
        <key>@0,AAPL,vram-memory</key>
        <data>
        AAAAwAAAABA=
        </data>
        <key>@0,ATY,EFIDisplay</key>
        <string>DP\_INT</string>
        <key>@0,VRAM,memsize</key>
        <data>
        AAAAIAAAACA=
        </data>
        <key>@0,compatible</key>
        <string>ATY,Galago</string>
        <key>@0,connector-type</key>
        <data>
        AAQAAA==
        </data>
        <key>@0,device\_type</key>
        <string>display</string>
        <key>@0,display-inverter-default-cycle</key>
        <data>
        /w==
        </data>
        <key>@0,display-link-component-bits</key>
        <data>
        CAAAAA==
        </data>
        <key>@0,display-pixel-component-bits</key>
        <data>
        CAAAAA==
        </data>
        <key>@0,display-type</key>
        <string>LCD</string>
        <key>@0,name</key>
        <string>ATY,Galago</string>
        <key>@1,AAPL,vram-memory</key>
        <data>
        AAAAwAAAABA=
        </data>
        <key>@1,VRAM,memsize</key>
        <data>
        AAAAAAAAACA=
        </data>
        <key>@1,compatible</key>
        <string>ATY,Galago</string>
        <key>@1,connector-type</key>
        <data>
        AAQAAA==
        </data>
        <key>@1,device\_type</key>
        <string>display</string>
        <key>@1,display-type</key>
        <string>NONE</string>
        <key>@1,name</key>
        <string>ATY,Galago</string>
        <key>AAPL,aux-power-connected</key>
        <data>
        AQAAAA==
        </data>
        <key>AAPL,backlight-control</key>
        <data>
        AAAAAA==
        </data>
        <key>AAPL00,InverterFrequency</key>
        <data>
        yDIAAA==
        </data>
        <key>AAPL00,blackscreen-preferences</key>
        <data>
        AAAACA==
        </data>
        <key>AAPL01,blackscreen-preferences</key>
        <data>
        AAAACA==
        </data>
        <key>ATY,Card#</key>
        <string>109\-B98505\-00</string>
        <key>ATY,Copyright</key>
        <string>Copyright AMD Inc. All Rights Reserved. 2005\-2009</string>
        <key>ATY,DeviceID</key>
        <data>
        wGg=
        </data>
        <key>ATY,EFI-dpcd-post-training</key>
        <data>
        dwABAAAA
        </data>
        <key>ATY,EFI-dpcd-training-result</key>
        <data>
        CoIAAAAAABAB
        </data>
        <key>ATY,EFICompileDate</key>
        <string>Apr 23 2010</string>
        <key>ATY,EFIDispConfig</key>
        <data>
        DxAAAAAAAAA=
        </data>
        <key>ATY,EFIDriverType</key>
        <data>
        Ag==
        </data>
        <key>ATY,EFIEnabledMode</key>
        <data>
        Ag==
        </data>
        <key>ATY,EFIHWInitStatus</key>
        <data>
        AAAAAAAAAAA=
        </data>
        <key>ATY,EFIOrientation</key>
        <data>
        AA8=
        </data>
        <key>ATY,EFIVersion</key>
        <data>
        MDEuMDAuNDE2AA==
        </data>
        <key>ATY,EFIVersionB</key>
        <string>113\-B98505\-101</string>
        <key>ATY,EFIVersionE</key>
        <string>113\-B9850E\-416</string>
        <key>ATY,MRT</key>
        <string></string>
        <key>ATY,MemRevisionID</key>
        <data>
        BAA=
        </data>
        <key>ATY,MemVendorID</key>
        <data>
        BgA=
        </data>
        <key>ATY,PlatformInfo</key>
        <data>
        AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
        </data>
        <key>ATY,RefCLK</key>
        <data>
        jAoAAA==
        </data>
        <key>ATY,Rom#</key>
        <string>113\-B9850H\-133</string>
        <key>ATY,VendorID</key>
        <data>
        AhA=
        </data>
        <key>MVAD</key>
        <data>
        PgQC//+QAEgAAAAAAAAAABo2gAcgCDAAIAA4BFcEAwAFADgEgAdAAQAAAAAA
        AAAAAAAAAAAAAAAAAAAAAAAAAA==
        </data>
        <key>VRAM,totalsize</key>
        <data>
        AAAAIA==
        </data>
        <key>bksv</key>
        <data>
        /wAAAA==
        </data>
        <key>device\_type</key>
        <string>ATY,GalagoParent</string>
        <key>graphic-options</key>
        <data>
        BAAAAA==
        </data>
        <key>model</key>
        <string>ATI Radeon HD 5670</string>
        <key>name</key>
        <string>ATY,GalagoParent</string>
        <key>saved-config</key>
        <data>
        PgQC//+QAEgAAAAAAAAAABo2gAcgCDAAIAA4BFcEAwAFADgEgAdAAQAAAAAA
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAyAAAAMgAAAABAAAAAAAAAJAB
        AADIMgAA/wgICDAADwAAAAAAAAAAAAAAAAAAAAAAHCwAABQsAAAYLAAAJCwA
        ACAsAAAEaAAAEGgAAPiYAABILwAA1AsAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABgAAAAYAAB/
        lwAA73j0AwgBGgAGoEAAAAAYAAIAARICAAESAgABEA==
        </data>
    </dict>
    <key>PciRoot(0x0)/Pci(0x1b,0x0)</key>
    <dict>
        <key>MaximumBootBeepVolume</key>
        <string>M</string>
        <key>PinConfigurations</key>
        <data>
        UEArAUABEJBCARCQIDCLARABoJAw4MsBYOBLAQ==
        </data>
        <key>layout-id</key>
        <data>
        EgAAAA==
        </data>
        <key>platformFamily</key>
        <data>
        AA==
        </data>
    </dict>
    <key>PciRoot(0x0)/Pci(0x1c,0x2)/Pci(0x0,0x0)/Pci(0x0,0x0)</key>
    <dict>
        <key>fwswappedbib</key>
        <data>
        AQAAAA==
        </data>
    </dict>
</dict>

More detailed instructions followed. This way we can deprecate Arbitrary section.

* * *

FakeID
======

  <key\>FakeID</key\>
  <dict\>
    <key\>ATI</key\>
    <string\>0x68181002</string\>
    <key\>IntelGFX</key\>
    <string\>0x01268086</string\>
    <key\>NVidia</key\>
    <string\>0x0</string\>
    <key\>LAN</key\>
    <string\>0x436311ab</string\>
    <key\>SATA</key\>
    <string\>0x25628086</string\>
    <key\>WIFI</key\>
    <string\>0x431214E4</string\>
    <key\>XHCI</key\>
    <string\>0x0</string\>
    <key\>IMEI</key\>
    <string\>0x1E3A8086</string\>
  </dict\>

This is a method to change PCI properties DeviceID and VendorID for the device will work with native drivers. In the example above: - AMDRadeonHD7850 has unsupported DeviceID=0x6819. Change to 0x6818 - Dell Wireless 1595, DeviceID=0x4315 is not supported. Change to 0x4312 - Marvell Yukon 8056, DeviceID=0x4353. Change to 0x4363.

There are other known substitutions for unsupported devices.

This substitution will work if InjectATI (Nvidia, Intel) is set. Or if FixDsdtMask set for the device.

Likewise, the IMEI fix will only work if the DSDT Patch `AddMCHC_0008` is enabled.

* * *

Audio
=====

<key\>Audio</key\>
<dict\>
...
</dict\>

Parameter group for tweaking setting affecting audio devices.

* * *

Audio / Inject
--------------

<key\>Inject</key\>
<string\>Detect</string\>

Property injection for the sound chip. Only works when the DSDT defines `Device (HDEF)`. If you renamed it, you can also inject the other properties differently. Do not use this parameter with VoodooHDA. Possible options are:

*   `No` - Injection is turned off
*   `Detect` - Automatic detection of the sound chip and injection of its ID as layout ID. Actually this is nonsense but still very popular. Does not do any harm and affects the display of sound properties in System Profiler.
*   `883` - Decimal number representing. Refers to Realtek ALC883 in this case.
*   `0x373` - Same as above in hexadecimal.

These numbers are incorrect, you will need to find the correct value and possibly replace the layout file bundled with AppleHDA to get the chip working.

* * *

Audio / ResetHDA
----------------

<key\>ResetHDA</key\>
<true/>

Audio controller initialization. Some users have non-working sound after cold boot but works after restart or wake (even on Windows!). This is a workaround that works at early boot, so it will affect Windows as well.

* * *

Audio / AFGLowPowerState
------------------------

<key\>AFGLowPowerState</key\>
<false/>

This helps remove cracking sounds at audio output after idle mode, so sound card is always on.

* * *

USB
===

<key\>USB</key\>
<dict\>
...
</dict\>

Parameter group for tweaking setting affecting PCI devices.

* * *

USB / Inject
------------

<key\>Inject</key\>
<true/>

Injects USB properties. You can turn it off for whatever reason, if you need. It is also disabled if the _DSDT patch mask_ matches `0x1000` to prevent data duplication.

* * *

USB / FixOwnership
------------------

<key\>FixOwnership</key\>
<true/>

It is possible to leave USB injection enabled and only turn off the ownership fix.

This fix is not relevant for UEFI booting.

* * *

USB / AddClockID
----------------

<key\>AddClockID</key\>
<true/>

*   `<true/>` - Enables a good, deep sleep, which cannot be exited by keyboard or mouse input.
*   `<false/>` - The PC will possibly sleep and it can be woken up by keyboard or mouse; or it will be automatically woken up by some attached device

Injects the property "AAPL,clock-id" with a unique identifier for each device. Set it to your liking.

**Requires USBInjection to be enabled.**

Default value is set to disabled.

USB / HighCurrent
-----------------

<key\>HighCurrent</key\>
<true/>

More power needed to charge iPad from USB ports.

* * *

UseIntelHDMI
============

<key>UseIntelHDMI</key>
<false/>

\### If TRUE, \*hda-gfx=onboard-1\* will be injected into the GFX0 and HDEF devices. Also, if an ATI or Nvidia HDMI device is present, they'll be assigned to onboard-2. If FALSE, then ATI or Nvidia devices will get onboard-1 as well as the HDAU device if present.

* * *

NoDefaultProperties
===================

<key>NoDefaultProperties</key>
<false/>

\### This key will affect DSDT fixes and force them to generate an empty \_DSM. For example, if you enable FIX\_DISPLAY Clover will create a device for the graphics card but with an empty \_DSM. AddProperties and FakeID values will still be injected. This works only for Display, Sound, LAN and WiFi.

* * *

SetIntelMaxBacklight
====================

<key>SetIntelMaxBacklight</key>
<true/>

### Fix MaxBrightness for Intel HD3000/4000 proposed by Dr.Hurt \[here\](https://www.insanelymac.com/forum/topic/284656\-clover-general-discussion/page-722?p=2489740&do\=findComment&comment=2489740)
New setting in config plist

* * *

SetIntelMaxBacklight Rev 4196
=============================

Set Intel Max Backlight Value in config an in menu.

<key\>Devices</key\>
<dict>
<key\>#SetIntelBacklight</key\>
<false/>
<key\>SetIntelMaxBacklight</key\>
<true/>
<key\>IntelMaxValue</key\>
<integer\>1808</integer\>

### The value is proposed to be decimal and common recommendation
- Sandy or IvyBridge:  1808
- Haswell or Broadwell: 2776
- Skylake or Kabylake: 1295

Disable Drivers: ⇩
------------------

DisableDrivers
==============

<key>DisableDrivers</key>
<array>
    <string>CsmVideoDxe</string\>
    <string>VBoxExt4</string\>
</array>

### The purpose of this parameter is to have multiple configurations in the \*OEM\* folder. Only one driver folder exists, though, and it may be necessary to disable specific drivers for different configurations. One may need OsxAptioFixDxe, another one EmuVariableDxe.

GUI: ⇩
------

The GUI settings can me modified by changing parameters in the _GUI_ section of the main configuration file _config.plist_.

<key\>GUI</key\>
</dict\>
...
</dict\>

* * *

TextOnly
========

<key\>TextOnly</key\>
<false/>

Text-only menu mode for a minimal GUI and faster loading times.

* * *

Theme
=====

The design of the GUI depends on the chosen theme, which is set using:

<key\>Theme</key\>
<string\>cesium</string\>

Additionally you can set the theme in the preference panel. This setting overrides the one in the configuration file. In case an invalid theme name is used here (no theme.plist exists at the given path), then the configuration file setting is used again. If even here the theme name is invalid, then you will be greeted with an ugly fallback theme that will still be operational.

* * *

ScreenResolution
================

<key\>ScreenResolution</key\>
<string\>1024x768</string\>

You can set the desired screen resolution if it is supported by the video card and the monitor. Clover will try to set the highest available resolution, however it might fail. You can check the list of supported resolutions in the boot log. In case `PatchVBios=Yes` is used in the _Graphics_ section, you will get the highest resolution supported by the monitor. In that case this parameter may be obsolete. The default value is `1024x768`.

* * *

ConsoleMode
===========

<key\>ConsoleMode</key\>
<string\>\[Number\]/Max/Min</string\>

Will switch the console driver to requested console mode (which is used by shell, boot.efi, dumpueficalls, etc). It is usually not necessary to use this, and should be used only where needed.

*   `0` (default) - Disabled. No switch will be done due to possible incompatibility reasons.
*   `Min` - Selects lowest mode available. Useful for slow consoles, like CloverEFI, with much debug output.
*   `Max` - Select highest mode available. Useful for fast consoles, where it works.
*   `[Number]` - Selects specific mode. See _boot.log_ from revison >= 2496 for a list of available modes.

* * *

Language
========

<key\>Language</key\>
<string\>en:0</string\>

Sets language of Clover's help menu, that can be accessed with shortcut F1. Furthermore, this setting is passed to the OS and can have an influence on the language.

Available languages codes are: `en`, `ru`, `fr`, `it`, `es`, `pt`, `br`, `de`, `nl`, `pl`, `ua`, `cz`, `hr`, `id`, `ko`

* * *

CustomIcons
===========

<key\>CustomIcons</key\>
<true/>

Enabling this key will load the icon from the partition itself. You may assign your own icons to each volume, and you can use different icons for different volumes, even when they have the same OS.

* * *

Mouse
=====

<key\>Mouse</key\>
<dict\>
    <key\>Enabled</key\>
    <true/>
    <key\>Speed</key\>
    <integer\>2</integer\>
    <key\>Mirror</key\>
    <false/>
    <key\>DoubleClick</key\>
    <integer\>500</integer\>
</dict\>

*   `Enabled` - The mouse may not work or even cause the whole GUI to lock up. Turn it off if this is the case.
*   `Speed` - Mouse movement speed. Reasonable values range from `2`\-`8`. Some mice may need a negative speed to reverse the movement. `0` turns off the mouse.
*   `Mirror` - Negative movement speed on a single axis
*   `DoubleClick` - Pause for double click detection in milliseconds. Until now `500`has been a good value for everyone.

* * *

Scan
====

<key\>Scan</key\>
<true/>

Enable or disable all automatic scans.

* * *

Scan / Entries
--------------

<key\>Scan</key\>
<dict\>
  <key\>Entries</key\>
  <true/>
</dict\>

Enable or disable the automatic UEFI entries scan.

* * *

Scan / Tool
-----------

<key\>Scan</key\>
<dict\>
  <key\>Tool</key\>
  <true/>
</dict\>

Enable or disable the automatic UEFI tool scan.

* * *

Scan / Legacy
-------------

<key\>Scan</key\>
<dict\>
  <key\>Legacy</key\>
  <string\>Last</string\>
</dict\>

Enable or disable automatic legacy scan. Also choose from `First` to list in the beginning, `Last` to list it in the end.

* * *

Scan / Kernel
-------------

<key\>Scan</key\>
<dict\>
  <key\>Kernel</key\>
  <string\>All/Newest/Oldest/First/Last/MostRecent/Earliest/None</string\>
</dict\>

Enable or disable automatic Linux kernel scan. Additionally you may choose between several options:

*   `All` - all found kernels (default)
*   `Newest` - newest file modification date
*   `Oldest` - oldest file modification date
*   `First` - first matching
*   `Last` - last matching
*   `MostRecent` - most recent version
*   `Earliest` - earliest version
*   `None` - no kernel scan

* * *

Hide
====

<key>Hide</key>
<array>
  <string>WindowsHDD</string>
  <string>HD(1,GPT,E223FF7F-F2DA-4DBB-B765\-756F2D95B0FE)</string>
</array>

Hide a partition with given name or GUID.

* * *

Custom
======

<key\>Custom</key\>
<dict\>
  ...
</dict\>

If the automatically scan entries are not enough you can add your own custom boot entries.

* * *

Custom / Entries
----------------

<key\>Entries</key\>
<array\>
  <dict\>
    <key\>Disabled</key\>
    <false/>
    <key\>Volume</key\>
    <string\>Volume name or GUID</string\>
    <key\>VolumeType</key\>
    <string\>Volume type</string\>
    <key\>Path</key\>
    <string\>Loader path</string\>
    <key\>Arguments</key\>
    <string\>Load options</string\>
    <key\>AddArguments</key\>
    <string\>Load options</string\>
    <key\>Title</key\>
    <string\>Display title</string\>
    <key\>FullTitle</key\>
    <string\>Full display title</string\>
    <key\>Image</key\>
    <string\>ImagePath</string\>
    <key\>ImageData</key\>
    <string\>Image hex</string\> OR <data\>Image base64</data\>
    <key\>DriveImage</key\>
    <string\>ImagePath</string\>
    <key\>DriveImageData</key\>
    <string\>Image hex</string\> OR <data\>Image base64</data\>
    <key\>Hidden</key\>
    <true/> or <false/> or <string\>Yes/No/Always</string\>
    <key\>InjectKexts</key\>
    <true/> or <false/> or <string\>Yes/No/Detect</string\>
    <key\>NoCaches</key\>
    <false/>
    <key\>Kernel</string\>
    <string\>All/Newest/Oldest/First/Last/MostRecent/Earliest</string\>
    <key\>Type</key\>
    <string\>OSX/OSXInstaller/OSXRecovery/Windows/Linux</string\>
    <key\>BootBgColor</key\>
    <string\>RRGGBBAA</string\>
    <key\>CustomLogo</key\>
    <false/>
    <key\>KernelAndKextPatches</key\>
    <dict\>
          <key\>AppleRTC</key\>
          <false/>
          <key\>KextsToPatch</key\>
          <array\>
          ... 
    </dict\>
    <array\>...</array\> Or <true/> / <false/>
  </dict\>
  ...
</array\>

*   `Disabled` - Disables the entry so it is not taken into account.
*   `Volume` - A volume name or GUID to be used when scanning for loader.
*   `VolumeType` - The type of volume you want your custom entry to match. Can be a string with one type or an array with multiple types. Valid volume types are `Internal`, `External`, `Optical` and `FireWire`.
*   `Path` - Location to scan for the efi loader.
*   `Arguments` - Arguments that are to be passed to the loader. Overides the default boot arguments in [Boot / Arguments](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#boot-).
*   `AddArguments` - Adds to the [default boot arguments](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#boot-). For a SubEntry it adds to the arguments of the main entry.
*   `Title` - Changes the display title. Uses the format "`Boot <Title> from <VolumeName>`"
*   `FullTitle` - Sets the display title to only "`<FullTitle>`" without any format.
*   `Image` - Path to a custom image file. Search path is root directory of entry volume, theme directory, clover directory, clover volume root directory, and finally os icon names.
*   `ImageData` - Embedded custom image. Can be PNG or BMP format.
*   `DriveImage` - Path to a custom drive image file. Search path is root directory of entry volume, theme directory, clover directory, clover volume root directory, and finally os icon names.
*   `DriveImageData` - Embedded custom drive image. Can be PNG or BMP format.
*   `Hidden` - Hides the entry. If value is `true` the entry can be shown by pressing F3. If value is `Always` the entry can never be show.
*   `InjectKexts` - Inject kexts. Valid options are `Yes`, `No` or `Detect`. Use `Detect` to inject kexts only if FakeSMC is not present in KernelCache or /S/L/E. For `OSX`, `OSXInstaller` and `OSXRecovery` type entries.
*   `NoCaches` - Skip caches. For `OSX`, `OSXInstaller` and `OSXRecovery` type entries.
*   `Kernel` - Set the linux kernel scan priority. If this option is not supplied Clover will show all kernels found. Valid options are `Newest`, `Oldest`, `First`, `Last`, `MostRecent`, `Earliest`. For `LinuxKernel` type entries.
*   `Type` - The type of OS that is being scanned for. Valid types are `OSX`, `OSXInstaller`, `OSXRecovery`, `Windows`, `Linux`, `LinuxKernel`. If Type is not one of these it will be treated as all types.
*   `BootBgColor` - Specifies the boot screen's background color. For `OSX`, `OSXInstaller` and `OSXRecovery` type entries.
*   `CustomLogo` - Defines the boot screen logo style. For further information read [Boot / CustomLogo](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#boot-)
*   `SubEntries` - (Default:true) Disables the default sub entries if set to false. Can also be used to create custom sub entries using the same structure as a main entry, any settings that aren't set in the sub entry will be inherited from the main entry.
*   `KernelAndKextPatches` - Uses the same hierarchy and offers the same functionality found in [KernelAndKextPatches](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#kernelandkextpatches-), so you can selectively apply patches for each entry. Only works with revision 2797 or higher.

* * *

Custom / Legacy
---------------

<key\>Legacy</key\>
<array\>
  <dict\>
    <key\>Volume</key\>
    <string\>Volume name or GUID</string\>
    <key\>Title</key\>
    <string\>Display title</string\>
    <key\>Hidden</key\>
    <false/>
    <key\>Disabled</key\>
    <false/>
    <key\>Ignore</key\>
    <true/>
    <key\>Type</key\>
    <string\>Windows/Linux</string\>
  </dict\>
  ...
</array\>

*   `Volume` - A volume name or GUID to be used when scanning for loader.
*   `Title` - Changes the display title. Uses the format "`Boot <Title> from <VolumeName>`"
*   `Hidden` - Hides the entry but can be shown by pressing F3.
*   `Disabled` - Disables the entry so that is never shown.
*   `Ignore` - Entry will be ignored, or not used, so it will not affect anything.
*   `Type` - The type of OS that is being scanned for. Valid types are `OSX`, `OSXInstaller`, `OSXRecovery`, `Windows`, `Linux`, `LinuxKernel`. If Type is not one of these it will be treated as all types.

* * *

Custom / Tool
-------------

<key\>Tool</key\>
<array\>
  <dict\>
    <key\>Volume</key\>
    <string\>Volume name or GUID</string\>
    <key\>Path</key\>
    <string\>Loader path</string\>
    <key\>Arguments</key\>
    <string\>Load options</string\>
    <key\>Hotkey</key\>
    <string\>G</string\>
    <key\>Title</key\>
    <string\>Display title</string\>
    <key\>FullTitle</key\>
    <string\>Full Display title</string\>
    <key\>Hidden</key\>
    <false/>
    <key\>Disabled</key\>
    <false/>
    <key\>Ignore</key\>
    <true/>
  </dict\>
  ...
</array\>

More explanation will come.

* * *

ShowOptimus
===========

<key\>ShowOptimus</key\>
<true/>

**Rev 4222** Tested on Dell Latitude E6430 which can be powered with Optimus enabled or disabled in BIOS.

The criteria is a number of videocard found. 2 = Embedded Intel + Discrete == "Intel" 1 = Discrete only == "Discrete"

I am not sure if someone else needed this feature so default behaviour to be not shown.

* * *

KbdPrevLang
===========

**Rev 4719**

Added prelang kbd option if want to keep language when updating macOS with **native NVRAM**.

*   This is the key to fixing macOS language problems when using native NVRAM.
*   This is macOS bug for long time ago.
*   When using only `AptioMemoryFix` or `OsxAptioFixV3` without `EmuVariableUefi`
*   When using a language other than English

You can fix issues below if use this key:

<key\>GUI</key\>
    <dict\>
        <key\>KbdPrevLang</key\>
        <true/>
        <key\>Language</key\>
        <string\>ru:0</string\>          <----- you want language
    </dict\>

1.  mixed language in restart popup after updating macOS.
    
2.  always keep english when updating macOS in recovery despite to user already use other language in macOS.
    

* * *

[Back on top ↑](https://github.com/CloverHackyColor/Clover-Documentaion)

Dark/Light/Daytime themes
=========================

**Rev 4644** A possibility to choose between **Light** and **Dark** embedded themes

<key\>GUI</key\>
    <dict\>
        <key\>Theme</key\>
        <string\>embedded</string\>
        <key\>EmbeddedThemeType</key\>
        <string\>Dark</string\>

Possible values Dark/Light/SVG /Daytime

The result:

![1](https://user-images.githubusercontent.com/6248794/163851959-35c85d14-9eeb-497f-815f-8fe61d4a7014.gif)
----------------------------------------------------------------------------------------------------------

Daytime Support
===============

**Rev 4773**

Daylight support in Clover GUI (like in Mojave). You should choose

<key\>GUI</key\>
    <dict\>
        <key\>Timezone</key\>
        <integer\>3</integer\>
        <key\>Theme</key\>
        <string\>embedded</string\>
        <key\>EmbeddedThemeType</key\>
        <string\>Daytime</string\>

Then BIOS time + Timezone will be compared to 8:00 of morning and 20:00 of evening and set light type or dark.

**Rev 4774**

Clovy SVG theme has os\_moja\_night icon and Background\_night to choose depending on daylight.

* * *

Disable Logging
===============

**Rev 4794**

Implemented command line to manage **boot.efi** output

Explanations by **vit9696** [**here**](https://www.insanelymac.com/forum/topic/331381-aptiomemoryfix/?do=findComment&comment=2572819).

By default I set command line as `"log=0"`it means you will not see long output from system start to ++++++

The setting can be changed in`Clover GUI`\> `Options`\> `System Parameters`

* * *

PlayAsync
=========

**Rev 4840**

Sync and Async sound play.

**Sync** play - sound play before enter Clover GUI. Clover GUI started after the sound is finished.

**Async** play - sound started to play before Clover GUI started and continue playing when GUI is ready.

To switch edit config.plist

<key\>GUI</key\>
    <dict\>
        <key\>PlayAsync</key\>
        <false/>

* * *

Startup Sound Configuration UI
==============================

**Rev 4847-4852**

There is interface in Clover GUI to tune startup sound parameters.

![](https://user-images.githubusercontent.com/6248794/164117873-44c37912-402b-4afc-82aa-600494ecf2b5.png)

Computer will play a sound before enter Clover GUI. To do this you have to

1.  Use CloverX64.efi revision 4852 or up.
    
2.  Place AudioDxe-64.efi into drivers64 and drivers64UEFI. Playing is possible in both legacy and UEFI modes. AudioDxe.efi is included into Clover project and will be differ from original by Goldfish64 but almost compatible.
    
3.  Place sound.wav and may be sound\_night.wav into your theme folder, for example /EFI/CLOVER/theme/Clovy/. The sound must be 16bit, and 44100 or 48000Hz. But I also made conversion on the fly so 8000Hz is also supported.
    
4.  Tune sound output in Clover GUI as on screen above. The settings will be stored in NVRAM and used next restart. With legacy Clover or systems without nvram you have to use EmuVariableUefi.efi and rc.scripts to save nvram.plist. For very new systems like Z370 where nvram is not working in macOS there is a chance that the nvram is working on Clover level and so all will works. SIC! Legacy Clover saved Variables only with Apple guid so why it will not work. After some discussing I may rebuild the system to use AppleBootGuid. In this case the system will not be compatible with the original driver but will work with legacy systems.
    

No more needs in BootChimeCfg and BootChimeDxe while the compatibility still remains

Graphics: ⇩
-----------

<key\>Graphics</key\>
<dict\>
...
</dict\>

This parameter group is made for injecting video card properties as done, for example, by _Natit.kext_. There are many different parameters that are injected, mostly constants, some of them calculated, some of them defined in the internal table and only a few parameters can be adjusted with the configuration file.

* * *

Inject
======

<key\>Inject</key\>
<true/>

Enables graphics injection, which is necessary for a working system. It is only advised to disable it, if you do not know a better method. Some video cards, for instance Nvidia GTX 6xx or AMD Radeon HD 6xxx do not require injection and it will automatically be disabled without user interaction. At least it is enough to reach the desktop.

Enabled by default.

* * *

Inject / Intel
--------------

<key\>Inject</key\>
<dict\>
  <key\>Intel</key\>
  <true/>
</dict\>

* * *

Inject / ATI
------------

<key\>Inject</key\>
<dict\>
  <key\>ATI</key\>
  <true/>
</dict\>

* * *

Inject / NVidia
---------------

<key\>Inject</key\>
<dict\>
  <key\>NVidia</key\>
  <true/>
</dict\>

* * *

VRAM
====

<key\>VRAM</key\>
<integer\>1024</integer\>

Amount of video memory in MB. It is determined automatically but no one will be harmed if you write a correct value. However, changing it has not helped anyone yet.

* * *

LoadVBios
=========

<key\>LoadVBios</key\>
<true/>

Loads a video bios from a file, which needs to be stored at the directory _EFI/CLOVER/OEM/\[model\]/ROM_ or _EFI/CLOVER/ROM_ with the name _\[vendor\]\_\[device\].rom_, for instance 1002\_68d8.rom. Sometimes it makes sense to load video bios manually, for example when you need a patched version or when the video card does not show it to the system - like mobile Radeon cards. In this case it is enough to set this parameter to `Yes` without providing any file. Clover will read the video bios from legacy memory at 0xC0000, where it is present most of the times, and enable functionality. Apparently the video bios written to the card's ROM does not match the one stored at address 0xC0000 - the shadow ROM. In short:

*   `<true/>` - Use for mobile Radeon cards without any external file. Can also be used to supply a legacy video bios to cards, which only contain a UEFI video bios.
*   `<false/>` - For all other variants

* * *

DualLink
========

<key\>DualLink</key\>
<integer\>0</integer\>

Default is `1`.

Some systems show a quartered screen. Use `0` to solve this problem, as showed in the example above.

* * *

PatchVBios
==========

<key\>PatchVBios</key\>
<true/>

Clover applies corrections to the shadow ROM at address 0xC0000 to allow support of the highest monitor resolution. For instance, a monitor's EDID contains the mode 1920x1080 but the video bios does not. Clover will apply it as the first mode to be used. In case the monitor does not produce an EDID, it can be injected manually.

In case `ScreenResolution` in section `GUI` is set, it will be used for this patch.

Should the automatic patch fail, then you can specify a manual one.

* * *

PatchVBiosBytes
===============

<key\>PatchVBiosBytes</key\>
<array\>
    <dict\>
        <key\>Find</key\>
        <data\>gAeoAqAF</data\>
        <key\>Replace</key\>
        <data\>gAeoAjgE</data\>
    </dict\>
</array\>

This example applied to an AMD Radeon HD 6670 video bios, resulting in a replacement of mode 1920x1440 with mode 1920x1080. It is advised to choose a mode with the same horizontal resolution.

It is possible to apply multiple patches. An Nvidia video bios, for instance, was successfully modified with four patches.

* * *

EDID
====

<key\>EDID</key\>
<dict\>
    <key\>Inject</key\>
    <true/>
    <key\>Custom</key\>
    <data\>AP///////wAyDADfAAAAAAASAQOAIRV4CunVmVlTjigmUFQAAAABAQEBAQEBAQEBAQEBAQEB3iGgcFCEHzAgIFYAS88QAAAY3iGgcFCEHzAgIFYAS88QAAAAAAAA/gBXNjU3RwAxNTRXUDEKAAAA/gAjMz1IZYSq/wIBCiAgAJo=</data\>
    <key\>VendorID</key\>
    <string\>0x1006</string\>
    <key\>ProductID</key\>
    <string\>0x9221</string\>
</dict\>

Old keys InjectEDID and CustomEDID were moved here.

`Inject` parameter is useful when an EDID exists but is not seen by Apple's drivers. In this case it is enough to set this option to `<true/>`. Clover will automatically extract the EDID and provide it to the drivers. See _Mobile Radeons_ for more info. For monitors without DDC and for UEFI-only computers a custom EDID is needed.

`Custom` parameter is for providing custom EDID when it's missing. The main requirement is to have a correct maximal resolution.

Another possibility is to use _ViewSonic EDID Editor v3.1.5_ that can easily be ported to Mac, if needed.

An EDID from a Dell Inspiron will be used in the sample configuration file with a matrix of 1440x900.

A custom EDID will also be used for automatic video bios patching. Look above.

You can override VendorID and ProductID in EDID.

**Rev 4351, 4355**

Additional patches for EDID.

Now config.plist may contain follow EDID section:

<key>EDID</key>
        <dict>
            <key>#Custom</key>
            <data>AP///////wAGECGSAAAAAAASAQOAIRV4CunVmVlTjigmUFQAAAABAQEBAQEBAQEBAQEBAQEB3iGgcFCEHzAgIFYAS88QAAAY3iGgcFCEHzAgIFYAS88QAAAAAAAA/gBXNjU3RwAxNTRXUDEKAAAA/gAjMz1IZYSq/wIBCiAgAJo=</data>
            <key>#Inject</key>
            <true/>
            <key>#ProductID</key>
            <string>0x9221</string>
            <key>#VendorID</key>
            <string>0x1006</string>
            <key>#HorizontalSyncPulseWidth</key>
            <integer>100</integer>
            <key>#VideoInputSignal</key>
            <string>0xA5</string>
        </dict>

`Custom` is EDID obtained or created manually. You mostly don't need do it as Clover usually can obtain it by himself. `Inject`\=true informs Clover to inject Custom or Discovered EDID into video device properties. You have to set it to true is you do patches. `ProductID` and `VendorID`\- may be you want to mimic your display to be Apple manufactured. But you should know what you do. Making this patch I found that Brightness control stop working. Remove this patch and brightness control is working again! `HorizontalSyncPulseWidth` - this patch helps to resolve garbled screen at boot with Intel graphics. Known problem of 8 apples.

I see that the patch is still needed

<dict\>
                <key\>Comment</key\>
                <string\>Patch 8 apples</string\>
                <key\>Disabled</key\>
                <false/>
                <key\>Find</key\>
                <data\>AQAAdRc=</data\>
                <key\>MatchOS</key\>
                <string\>10.11</string\>
                <key\>Name</key\>
                <string\>IOGraphicsFamily</string\>
                <key\>Replace</key\>
                <data\>AQAA6xc=</data\>
            </dict\>

but this patch alone is not full solution. The apple is broken for a several seconds and then restored. With new Clover setting `HorizontalSyncPulseWidth=100` there is a full solution. The image is good and not jumped.

`VideoInputSignal` - this is additional patch for the same problem but I found no inluence. Definition of the value is follow `Bit 7=1` Digital input. If set, the following bit definitions apply:

Bits 6–4 Bit depth: 000=undefined, 001=6, 010=8, 011=10, 100=12, 101=14, 110=16 bits per color, 111=reserved Bits 3–0 Video interface: 0000=undefined, 0001=HDMIa, 0010=HDMIb, 0100=MDDI, 0101=DisplayPort

My default values for desktop 0x80 (Digital, all undefined), for laptop 0x90 -6bits per color?! - not good. It seems value 0xA0 will be better if used. And I don't know interface for intenal screen.

Credits. I was informed by Derek about this patch but initially it was wrong (byte 64 while it must be 63+65). [https://github.com/1and1get2/Clover/commit/ee301efdcce8ae1e47be21267490753a0c145c1d](https://github.com/1and1get2/Clover/commit/ee301efdcce8ae1e47be21267490753a0c145c1d) The discussion was at tmx site as well wrong and without result. And I don't know who is initial inventor because in those matter it can't work properly.

* * *

VideoPorts
==========

<key\>VideoPorts</key\>
<integer\>2</integer\>

The amount of ports on a video card, including TVO and/or HDMI. The chosen Apple framebuffer may not correspond to our real video card.

* * *

FBName
======

<key\>FBName</key\>
<string\>Macaque</string\>

This parameter is specific to AMD Radeon cards, which have a bunch of different framebuffers without any specific pattern. For a wide range of common video cards Clover will automatically choose a suiting framebuffer name. Feel free to set your own if you want. If you do not know what to write here, delete this parameter completely.

**Do not use Macaque! It is solely used for absurdity. But no - people still use it in their configurations!**

Small guidance table:

\*Series v / Type >\*   |Mobile  |Desktop 
----------------------|\--------|--------
HD 5000               |Alouatta|Baboon 
HD 6000               |Cuttail |Ipomoea 
HD 7000               |Pondweed|Futomaki

* * *

NVCAP
=====

<key\>NVCAP</key\>
<string\>04000000000003000C0000000000000A00000000</string\>

This parameter is only useful for Nvidia video cards and configures types and usage of video ports. The example line contains 40 capitalised hexadecimal digits. Theory is missing in this case and praxis shows controversial results.

Following table exists (found somewhere on insanelymac) but its correctness is debatable:

![](https://user-images.githubusercontent.com/485515/184123437-d5a39f55-6e22-4158-89c7-010fce0cc0bb.png)

You can also find other methods on calculating a correct value for this line.

Clover will try to generate one according to the video bios.

* * *

display-cfg
===========

<key\>display\-cfg</key\>
<string\>03010300FFFF0001</string\>

This parameter is also just for Nvidia cards. For details see the [topic on projectosx](http://www.projectosx.com/forum/index.php?showtopic=1105). However the listed examples are controversial. You can see real configurations in [this topic](http://www.projectosx.com/forum/index.php?showtopic=370). Apparently it is best to just let Clover handle this value. Leave it out completely and let Clover do the calculation.

* * *

ig-platform-id
==============

<key\>ig-platform-id</key\>
<string\>0x01620005</string\>

This parameter is used to enable the internal Intel HD 4000 video card. Until now no common rule for selecting a value was found, so either let Clover handle this parameter or use another one if it does not work.

Here is a table with possible values:

Value     |Type   |Ports 
:---      |:---   |:--- 
0x01660000|Mobile |4 ports: 1 LVDS, 3 DP 
0x01660001|Mobile |4 ports: 1 LVDS, 1 HDMI, 2 DP 
0x01660002|Mobile |1 port: LVDS 
0x01660003|Mobile |4 ports: 1 LVDS, 1 HDMI e 2 DP 
0x01660004|Mobile |1 port: LVDS 
0x01620005|Desktop|3 ports: DP 
0x01620006|Desktop|0 ports 
0x01620007|Desktop|0 ports 
0x01660008|Mobile |3 ports: 1 LVDS, 2 DP 
0x01660009|Mobile |3 ports: 1 LVDS, 2 DP 
0x0166000a|Mobile |3 ports: 2 DP, 1 HDMI 
0x0166000b|Mobile |3 ports: 2 DP, 1 HDMI

* * *

RadeonDeInit
============

**Rev 4296**

A possibility to de-init Radeon cards. It was very old problem since ElCapitan that AMD Radeon HD6xxx and up will not wake after sleep if used as first card. If the card is secondary and CSM enabled then the card works fine. So the differences is initialisation. The solution was found by vit9696 and used in private WhateverGreen.kext. Then Mieze proposed the solution as ACPI patch. Now I included this into Clover.

You have to set in config.plist:

<key\>RadeonDeInit</key\>
<true/>

Related threads [No graphics / USB / Audio after wake](https://www.insanelymac.com/forum/topic/308025-no-graphics-usb-audio-after-wake/)

[Tracing back the AMD GPU wakeup issue to its origin](https://www.insanelymac.com/forum/topic/328549-tracing-back-the-amd-gpu-wakeup-issue-to-its-origin/)

KernelAndKextPatches: ⇩
-----------------------

KernelAndKextPatches

<key\>KernelAndKextPatches</key\>
<dict\>
...
</dict\>

This parameter group enables patching on-the-fly.

Requirement: booting **with kernel cache** or with the key **NoCache**. Should the cache be missing because of other reasons, then patching will not be enabled.

* * *

Debug
=====

<key\>Debug</key\>
<true/>

Enables detailed output describing the patching process. This prints kernel patches and kext injection debug messages to the screen, since standard boot log is already closed. Useful for debugging purposes.

* * *

KernelCpu
=========

<key\>KernelCpu</key\>
<true/>

Eliminates kernel panics related to an unsupported CPU like Yonah, Atom or Haswell for old systems. However, the kernel has other algorithms which will not work correctly when using an unsupported CPU and this patch will not solve all problems. It is highly unlikely that this will work with a Pentium M, Pentium 4 or and AMD. The use of a special replacement kernel is advised in such cases.

* * *

FakeCPUID
=========

<key\>FakeCPUID</key\>
<string\>0x010676</string\>

Fake CPUID is intended to be a replacement for the [KernelCPU](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#kernelcpu) patch. It influences the kernel and AppleCPUPowerManagement as well.

If you want to install OSX with an unsupported CPU then you'll need to fake its ID to avoid kernel panics. Useful if you're trying to install OSX with an Atom CPU, or 10.7.2 with an IvyBridge CPU.

For these cases Clover allows you to set FakeCPUID in config.plist or even through the GUI, in the Binaries patching menu.

The following table shows all supported CPUIDs by different OS X versions:

![](https://user-images.githubusercontent.com/6248794/164104704-7e459807-346e-4820-8d7f-0365766b7eec.png)

* * *

AppleIntelCPUPM
===============

<key\>AppleIntelCPUPM</key\>
<true/>

AsusAICPUPM key was renamed to AppleIntelCPUPM.

Some vendors, like ASUS, restrict the use of MSR register 0xE2 in their power management module to ReadOnly. On Sandy/Ivy Bridge systems the kext **AppleIntelCpuPowerManagement** will try to write to this register and cause a kernel panic. This patch will eliminate the kext's write operations.

* * *

KernelPm
========

<key\>KernelAndKextPatches</key\>
<dict\>
   <key\>KernelPm</key\>
   <true/>
</dict\>

Kernel power management patch for Haswell with locked msrs. Works with 10.8.5 and 10.9 kernels.

* * *

AppleRTC
========

<key\>AppleRTC</key\>
<true/>

OS X has compatibility issues with a BIOS CMOS resulting in its reset on each wake after sleep and reboot, which is noticeable in a loss of BIOS settings. Mostly Gigabyte motherboards are affected. Usually it can be solved by patching Device(RTC) in the DSDT. If this does not help, the kext itself can be patched, which is done here.

* * *

KernelLapic
===========

<key\>KernelLapic</key\>
<false/>

HP notebooks have _lapic_ problems, which can be solved by using the boot parameter `cpus=1` or by using this option.

* * *

KernelHaswellE
==============

<key\>KernelHaswellE</key\>
<false/>

Haswell-E setups are currently not supported by the OS X kernel (as of OS X 10.10.2). This patch gets rid of the blocking compatibility checks. Note that this will not enable Power Management or similar features.

* * *

KextsToPatch
============

<key\>KextsToPatch</key\>
<array\>

Apart from the built-in patches you can create your own ones providing following data: the binary file name, the data to find and the data to replace with - both in hexadecimal. The data length must be equal. A smaller replacement data line can be filled with zeroes.

*   The following example shows a patch for VoodooHDA replacing the string _Headphones_ with _Telephones_:
    
    <key\>KextsToPatch</key\>
    <array\>
    <dict\>
        <key\>Name</key\>
        <string\>VoodooHDA</string\>
        <key\>Find</key\>
        <data\>SGVhZHBob25lcwA=</data\>
        <key\>Replace</key\>
        <data\>VGVsZXBob25lcwA=</data\>
    </dict\>
    </array\>
    

Note: the patch data is encoded in base64 due to the _data_ type of the field.

Common patches are:

*   TRIM function for non-Apple SSDs:
    
    <dict\>
        <key\>Name</key\>
        <string\>IOAHCIBlockStorage</string\>
        <key\>Find</key\>
        <data\>QVBQTEUgU1NEAA==</data\>                       
        <key\>Replace</key\>
        <data\>AAAAAAAAAAAAAA==</data\>
    </dict\>
    
*   Define external drivers as internal to fix yellow drive icons:
    
    <dict\>
        <key\>Name</key\>
        <string\>AppleAHCIPort</string\>
        <key\>Find</key\>
        <data\>RXh0ZXJuYWw=</data\>
        <key\>Replace</key\>
        <data\>SW50ZXJuYWw=</data\>
    </dict\>
    
*   Allow booting with a MacPro4,1 or MacPro5,1 SMBIOS definition without ECC memory:
    
    <dict\>
        <key\>Name</key\>
        <string\>AppleTyMCEDriver</string\>
        <key\>Find</key\>
        <data\>cgoATWFjUHJvNCwxAE1hY1BybzUsMQBY</data\>
        <key\>Replace</key\>
        <data\>cgoAAAAAAAAAAAAAAAAAAAAAAAAAAABY</data\>
    </dict\>
    

It is sometimes necessary to additionally modify a kext's Info.plist. In this case following syntax is used:

<dict\>
    <key\>Name</key\>
    <string\>AppleHDAController</string\>
    <key\>Comment</key\>
    <string\>Patch\_to\_not\_load\_this\_driver</string\>
    <key\>InfoPlistPatch</key\>
    <true/>
    <key\>Find</key\>
    <string\>0x04020000</string\>
    <key\>Replace</key\>
    <string\>0x44220000</string\>
</dict\>

Side note: The patch is supposed to be done in the cache. If you patch an Info.plist to allow the loading of a kext, then it is not yet present in the cache. You will need to reboot twice. First with the key _NoCaches_ to allow _FSInject_ to load the kext and a second time for the cache to be patched successfully. Not working in sytem 10.8 and later

* * *

ForceKextsToLoad
================

Not working in BigSur and later

<key\>ForceKextsToLoad</key\>
<array\>
  <string\>\\System\\Library\\Extensions\\AppleHDA.kext</string\>
  <string\>\\Extra\\Extensions</string\>
</array\>

Force load kexts (and plugins) from booted volume so it can be patched. It also accepts directories to force load/inject every kext in the folder.

* * *

ATIConnectorsController
=======================

Not working since 10.11

<key\>ATIConnectorsController</key\>
<string\>6000</string\>

For a fully working AMD video card injecting properties into the registry is not enough. Additionally a modification of the connectors in the according controller binary file is necessary. The example points to the 6000 series controller file:

<key\>ATIConnectorsData</key\>
<string\>000400000403000000010000210302040400000014020000000100000000 040310000000100000000001000000000001</string\>
<key\>ATIConnectorsPatch</key\>
<string\>040000001402000000010000000004040004000004030000000100001102 010500000000000000000000000000000000</string\>

This method works for systems using Mac OS X 10.7 or newer.

More info in the [post of bcc9](http://www.insanelymac.com/forum/topic/249642-editing-custom-personalities-for-ati-radeon-hd45xxx/) on insanelymac.

* * *

KernelXCPM
==========

**Rev 4250**

<key\>KernelXCPM</key\>
<true/>

XCPM patch for IvyBridge CPUs.

RtVariables: ⇩
--------------

<key\>RtVariables</key\>
<dict\>
...
</dict\>

Parameter group for defining runtime variables affecting Apple services or Clover itself.

* * *

MLB
===

<key\>MLB</key\>
<string\>XXXXXXXXXX</string\>

Digits and letters, 17 in length, describing the motherboard's serial number. No special rule exists here. The best option is to choose a real serial number and to replace digits in the middle. For instance, write `…SLICE…` or whatever comes to your mind.

* * *

ROM
===

<key\>ROM</key\>
<data\>AAAAAAAA</data\> OR <string\>UseMacAddrX</string\>

Twelve hexadecimal digits, often corresponding to the MAC address of the ethernet card. According to several reports this value can be random.

_Since revision 3051 Clover can detect the LAN MAC address of an ethernet device and use it as ROM._ For UEFI, it will first check the UEFI protocol for LAN information, but for legacy it will attempt to get it from the hardware instead, and because this procedure is dangerous you'll need to enable the functionality by specifying the LAN device with: `UseMacAddr0` or `UseMacAddr1` (up to 4 devices are calculated but only 2 will be used). If `UseMacAddrX` is not set then the MAC address will not be tested for legacy; the UEFI method is safe and will always be tested.

* * *

CsrActiveConfig
===============

*   Since revision r3259

<key\>CsrActiveConfig</key\>
<string\>0x3</string\>

Controls System Integrity Protection (SIP), see [Wikipedia](https://en.wikipedia.org/wiki/System_Integrity_Protection).

Relevant user options for SIP:

*   csr-active-config 0x0 = SIP Enabled (Default). Some drivers will not work.
*   csr-active-config 0x3 = SIP Partially Disabled (Loads unsigned kexts but not unapproved)
*   csr-active-config 0xFEF = SIP Disabled completely (SIC! bit 0x10 shouldn't be set!)
*   csr-active-config 0xA87 = SIP Disabled enough for hackintosh

* * *

BooterConfig
============

*   Since revision r3259

BooterConfig

0x28

* * *

SMBIOS: ⇩
---------

<key\>SMBIOS</key\>
<dict\>
...
</dict\>

This group of parameters is used to mask your PC as a Mac. Clover will do this task automatically based on the given processor, video card and signs of mobility. However, you may choose differently. Get the MacTracker application, choose a model you like and find matching values. Chameleon Wizard may help you with this task. There is not much to comment here, these values are not for beginners. If you know them - change them; otherwise leave it. Calculating them is mostly not possible.

* * *

ProductName
===========

<key\>ProductName</key\>
<string\>MacBook1,1</string\>

_SMBIOS.table1->ProductName_

You can set only this value and Clover will adjust the rest of the parameters automatically according to the model, so you can leave them out. Otherwise your custom values will be preferred.

Clover has built-in definitions for following models:

*   `MacBook1,1`
*   `MacBook2,1`
*   `MacBook4,1`
*   `MacBook5,2`
*   `MacBook8,1`
*   `MacBook9,1`
*   `MacBook10,1`
*   `MacBookPro5,1`
*   `MacBookPro6,2`
*   `MacBookPro8,1`
*   `MacBookPro8,3`
*   `MacBookPro9,2`
*   `MacBookPro11,4`
*   `MacBookPro12,1`
*   `MacBookPro13,1`
*   `MacBookPro13,3`
*   `MacBookPro14,1`
*   `MacBookPro16,1`
*   `MacBookAir3,1`
*   `MacBookAir5,2`
*   `MacBookAir6,2`
*   `Macmini2,1`
*   `Macmini5,1`
*   `Macmini6,2`
*   `Macmini7,1`
*   `iMac8,1`
*   `iMac10,1`
*   `iMac11,1`
*   `iMac11,2`
*   `iMac11,3`
*   `iMac12,1`
*   `iMac12,2`
*   `iMac13,1`
*   `iMac13,2`
*   `iMac14,2`
*   `iMac15,1`
*   `iMac17,1`
*   `iMac18,2`
*   `iMac18,3`
*   `iMac19,1`
*   `MacPro3,1`
*   `MacPro4,1`
*   `MacPro5,1`
*   `MacPro6,1`
*   `MacPro7,1`

All other models require setting all fields manually.

In case the model is not set, Clover will automatically choose one for you.

Some parameters like `BoardSerialNumber` should be generated and specified manually to be unique though.

* * *

Family
======

<key\>Family</key\>
<string\>MacBookAir</string\>

It is equal to model without model number.

* * *

SmUUID
======

<key>SmUUID</key>
<string>00000000\-0000\-1000\-8000\-010203040506</string>

This is the UUID, which is written into the SMBIOS table. Apparently it makes sense to fill the last pairs of digits with the MAC address of your ethernet card. This UUID will also be used in case CustomUUID is missing. It also be used to initialize ROM RtVariable (use by iMessage) in case ROM is missing.

* * *

FirmwareFeatures
================

<key\>FirmwareFeatures</key\>
<string\>0xC0001403</string\>

_SMBIOS.table128->FirmwareFeatures_

These digits exceed the standard SMBIOS and are part of Apple's specification. Different real Macs show different values and no description of the meaning exists. The only clue is provided by the sources of the _bless_ command:

  && (featureFlags & 0x00000001)) {
  contextprintf(context, kBLLogLevelVerbose,  "Legacy mode suppported\\n");

Therefore, we too need an odd number here.

* * *

ExtendedFirmwareFeatures
========================

        <key\>ExtendedFirmwareFeatures</key\>
    <string\>0x8FE001403</string\>
    <key\>ExtendedFirmwareFeaturesMask</key\>
    <string\>0xFFFFFFFFFF</string\>

_SMBIOS.table128-> ExtendedFirmwareFeatures_

macOS Monterey asked for bit 35 which is not fitted in 32bits of FirmwareFeatures. So new field in SMBIOS and in NVRAM was implemented ExtendedFirmwareFeatures and corresponding ExtendedFirmwareFeaturesMask. And then old FirmwareFeatures will not be used in the case new field is present. Clover automatically assign some value for you hardware but you also may use this sample.

* * *

SerialNumber
============

<key\>SerialNumber</key\>
<string\>W8000AAAAA</string\>

Clover will use one specific value here per each model, which needs to be replaced by your own serial number. There are two formats of serial number, which one to use depends on the model. There are tools to generate a serial number for a given model, or it can be created manually, according to [http://prasys.info/2009/11/understanding-mac-serial-number/](http://prasys.info/2009/11/understanding-mac-serial-number/) or something newer. The serial number used by Clover is most probably banned already.

* * *

BoardSerialNumber
=================

<key\>BoardSerialNumber</key\>
<string\>C02032101R5DC771H</string\>

_SMBIOS.table2->SerialNumber_

Don't mix it with system serial number. Clover will use one specific value here, which needs to be replaced by your own serial number. It is required for working iCloud and iMessage services (initialize MLB RtVariable if missing). The length must be 17 digits, consisting of letters from the Latin alphabet and numbers. The serial number used by Clover is most probably banned already.

* * *

BiosVersion
===========

<key>BiosVersion</key>
<string>MB11.YACC.0061.53PH.B03</string>

We see only the first, third and fifth groups in System Profiler, but really we should specify the full Boot ROM version in config.

* * *

BiosReleaseDate
===============

<key\>BiosReleaseDate</key\>
<string\></string\>

It's the release date of Boot ROM specified in BiosVersion key.

* * *

BoardType
=========

<key\>BoardType</key\>
<integer\>10</integer\>

_SMBIOS.table2->BoardType_

This parameter was introduced for the MacPro, which uses `11` (ProcessorBoard) instead of `10` (Motherboard), apparently for historical reasons. The effect is not clear, however it can be seen in _Profiler_.

* * *

Mobile
======

<key\>Mobile</key\>
<true/>

Usually Clover will automatically detect the mobility of a system, e.g. if is powered by a battery, needs energy saving, or not. Change it to trick the system into thinking that a battery is not available, or otherwise.

* * *

ChassisType
===========

<key\>ChassisType</key\>
<string\>0x10</string\>

_SMBIOS.table3->Type_

Used as an indirect indication for platform mobility. Here is a table according to SMBIOS standards:

Type                             |Value 
:---|\---
MiscChassisTypeOther             |0x01 
MiscChassisTypeUnknown           |0x02
MiscChassisTypeDeskTop           |0x03 
MiscChassisTypeLowProfileDesktop |0x04 
MiscChassisTypePizzaBox          |0x05 
MiscChassisTypeMiniTower         |0x06 
MiscChassisTypeTower             |0x07 
MiscChassisTypePortable          |0x08 
MiscChassisTypeLapTop            |0x09 
MiscChassisTypeNotebook          |0x0A 
MiscChassisTypeHandHeld          |0x0B 
MiscChassisTypeDockingStation    |0x0C 
MiscChassisTypeAllInOne          |0x0D 
MiscChassisTypeSubNotebook       |0x0E 
MiscChassisTypeSpaceSaving       |0x0F 
MiscChassisTypeLunchBox          |0x10

Clover will use a value according to the set Mac model, like it is used in real Macs. The effect, except mobility settings, is unclear.

* * *

ChassisAssetTag
===============

<key\>ChassisAssetTag</key\>
<string\>LatitudeD420</string\>

_SMBIOS.table3->AssetTag_

This field is never used with real Macs. We can use it for our own purpose, for example in the _HWSensors_ project.

* * *

Trust
=====

<key\>Trust</key\>
<true/>

Used to give priority to memory descriptor values found in SMBIOS or SPD. Change if your memory is described incorrectly in Mac. Default is `<true/>`.

* * *

NoRomInfo
=========

<key\>NoRomInfo</key\>
<false/>

_SMBIOS.table11->Apple ROM Info_

Clover generated some info the System Profiler like some real Macs:

Apple ROM Info:    Apple ROM Version.
Board-ID : Mac\-7BA5B2D9E42DDD94
⌘  Powered by Clover revision: 5145 (master, commit a8d020845)

But if you don't want to see this you may use this key with value

* * *

Memory
======

<key\>Memory</key\>
<dict\>
  <key\>Channels</key\>
  <integer\>2</integer\> OR <string\>1</string\>
  <key\>SlotCount</key\>
  <integer\>24</integer\> OR <string\>4</string\>
  <key\>Modules</key\>
  <array\>
    <dict\>
      <key\>Slot</key\>
      <integer\>0</integer\> OR <string\>5</string\>
      <key\>Size</key\>
      <integer\>2048</integer\> OR <string\>4096</string\>
      <key\>Frequency</key\>
      <integer\>1600</integer\> OR <string\>1333</string\>
      <key\>Vendor</key\>
      <string\>Some Company</string\>
      <key\>Part</key\>
      <string\>123456ABCDEF</string\>
      <key\>Serial</key\>
      <string\>ABCDEF123456</string\>
      <key\>Type</key\>
      <string\>DDR/DDR2/DDR3</string\>
   </dict\>
  </array\>
</dict\>

Inject custom memory module tables into SMBIOS if original SPD and SMBIOS provide incorrect or incomplete information.

*   `Channels` - Number of hardware channels supported by your memory chipset, usually two.
    
*   `SlotCount` - Total count of memory slots in your system. Max 24 (for now).
    
*   `Modules` - An array of INSTALLED modules, if your slot is empty don't include an entry for it.
    
    For each module:
    
*   `Slot` - Physical slot location in which the module is installed.
    
*   `Size` - Size of the module in megabytes.
    
*   `Frequency` - Speed of the module in megahertz.
    
*   `Vendor` - Memory manufacturer name
    
*   `Part` - The part number identifier.
    
*   `Serial` - The module's serial number.
    
*   `Type` - The memory module's type.
    

* * *

Slots
=====

<key\>Slots</key\>
<array\>
  <dict\>
     <key\>Device</key\>
     <string\>ATI/NVidia/LAN/WIFI/Firewire</string\>
     <key\>ID</key\>
     <integer\>2</integer\>
     <key\>Name</key\>
     <string\>Device name</string\>
    <key\>Type</key\>
    <integer\>0</integer\>
  </dict\>
</array\>

`AAPL,slot-name` injector. It allows you to add devices into System Profiler's PCI Cards section. This is a property that is usually injected by DSDT or property strings but this is a wrong way to go.

The injection of this property requires `Name (_SUN, 0x02)` to be present in the device's DSDT section. You can set this to any one byte number but 0 and 1 because of compiler optimizations. If you don't use a custom DSDT you may instead set DSDT [Mask Fix](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#dsdt--fixes) bits for those devices. Sample:

Device (GIGE)
{
    Name (\_ADR, 0x00050000)  // \_ADR: Address
    Name (\_SUN, 0x02)  // \_SUN: Slot User Number

*   `Device` - For now it can only be one of these: ATI, NVidia, LAN, WIFI or Firewire.
*   `ID` - Must be the same number defined into your DSDT in `_SUN`.
*   `Name` - The string that you want to assign to `AAPL,slot-name`.
*   `Type` - Set to 0 for PCI, 1 for PCIe 1x, 2 for PCIe 2x, etc.

* * *

SystemParameters: ⇩
-------------------

<key\>SystemParameters</key\>
<dict\>
...
</dict\>

* * *

CustomUUID
==========

<key>CustomUUID</key>
<string>511CE200\-1000\-4000\-9999\-010203040506</string>

Unique identification number of your computer. If not set, an automatically generated UUID will be used. Customise it with hexadecimal digits for full control over your hackintosh.

**Do not use this example value. It is by far not unique, as there are enough fools who copied it already.**

* * *

InjectSystemID
==============

<key\>InjectSystemID</key\>
<false/>

The number described above can be injected differently and transformed by the OS into another ID. The aim is to provide an option for Chameleon users to replicate their UUID. Set to `<true/>` and change `CustomUUID` to match the UUID used with Chameleon found in registry at `IODeviceTree:/efi/platform`\>`system-id`. Profiler will show a different UUID that will match the one generated with Chameleon. Not Chameleon users should simply set .

* * *

BacklightLevel
==============

<key\>BacklightLevel</key\>
<string\>0x0101</string\>

Monitor brightness level. However, only few systems will be affected by this parameter. It also is read from NVRAM. By default a value given by the system is used. Specifying it in the configuration file will override it.

* * *

InjectKexts
===========

<key\>InjectKexts</key\>
<string\>Detect/Yes/No</string\> or <true/> or <false/>

Recommended value `true` This key defines the global policy regarding kext injection.

*   `Yes` or `<true/>` - Always inject kexts from _/EFI/CLOVER/kexts/_
*   `No` or `<false/>`\- Never inject kexts
*   `Detect` - Kexts from _/EFI/CLOVER/kexts/_ will be injected only if FakeSMC is not present in the kernelcache

In case [Custom Entries](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#custom--entries) are defined, its own `InjectKexts` key will override this global one.

* * *

NoCaches
========

<key\>NoCaches</key\>
<string\>Yes/No</string\> or <true/> or <false/>

If enabled, the cache will be skipped on each boot. And just like `InjectKexts`, this key defines the global rule, so the value defined on [Custom Entries](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#custom--entries) will override this one. It works on systems up to 10.7.5. Recent macOS don't allow skip caches.

* * *

NvidiaWeb
=========

<key\>NvidiaWeb</key\>
<true/>

If key value is `true`, it will allow access to load and use Nvidia WebDriver kexts under the new MacOS systems (10.12-13). It is not same as bootarg.

* * *

Quirks
======

**Rev 5119+**

Included ➢ [Quirks](https://github.com/CloverHackyColor/CloverBootloader/wiki/OC-integration#quirks-)

* * *

[Back on top ↑](https://github.com/CloverHackyColor/Clover-Documentation?tab=readme-ov-file#welcome-to-the-cloverbootloader-documentation-clover-documentation-site)

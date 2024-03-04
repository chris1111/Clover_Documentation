Native-speedstep
================

It is more correct to say _power management_ and _CPU frequency control_ or simply EIST - Enhanced Intel SpeedStep Technology.

The topic is for very old computers like Core2Duo.

This topic is more about setting up a hackintosh than about the boot loader, but as Clover performs several steps, they are described separately. Clover does not fully automate this process, you still need to adjust some things manually.

Why is this needed at all? The purpose of it is to allow the CPU to work with its lowest frequency and voltage when it is idling (to reduce power and heat) and to increase both under load again.

EIST can be activated using several options: using a special utility like CoolBookController or GenericCPUPM, or enabling native native SpeedStep where following steps are required:

1.  HPET needs to be fixed, which can be done with mask `0x0010`
2.  A correct CPU section must be present in ACPI (SSDT). See section [CPU](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#cpu-).
3.  A correct Mac model needs to be chosen. EIST does not work on all models. For instance - it will work with MacBook5,1 and will not work with MacBook1,1

Alternatively you can leave any model but change the platform configuration file to enable SpeedStep. Each model has its own file. Look at:

_/System/Library/Extensions/IOPlatformPluginFamily.kext/Contents/PlugIns/ACPI\_SMC\_PlatformPlugin.kext/Contents/Resources/_

Compare different models and choose the right values.

* * *

ConfigArray
===========

<key\>ConfigArray</key\>
<array\>
    <dict\>
        <key\>WWEN</key\>
        <true/>
        <key\>model</key\>
        <string\>MacBook4,1</string\>
        <key\>restart-action</key\>
        <dict\>
            <key\>cpu-p-state</key\>
            <integer\>0</integer\>
        </dict\>
    </dict\>
</array\>

The key `restart-action` specifies the P-State value of the CPU, which will be set on a reboot. Sleep and shutdown only started working after this value was added!

* * *

CtrlLoopArray
=============

<key\>CtrlLoopArray</key\>
<array\>
    <dict\>
        <key\>Description</key\>
        <string\>SMC\_CPU\_Control\_Loop</string\>
        <key\>PLimitDict</key\>
        <dict\>
            <key\>MacBook4,1</key\>
            <integer\>0</integer\>
        </dict\>

The key `PLimitDict` is mentioned at [GeneratePStates](https://github.com/CloverHackyColor/CloverBootloader/wiki/Configuration#ssdt--generate--pstates). It represents the limit of the maximal frequency. If it is missing, the CPU will be stuck at the lowest frequency.

* * *

CStateDict
==========

<key\>CStateDict</key\>
<dict\>
    <key\>MacBook4,1</key\>
    <string\>CSD3</string\>
    <key\>CSD3</key\>
    <dict\>
        <key\>C6</key\>
        <dict\>
            <key\>enable</key\>
            <true/>

It is recommended to delete this section to allow power control through P-States and not through C-States.

* * *

[Back on top ↑](https://github.com/CloverHackyColor/Clover-Documentation?tab=readme-ov-file#welcome-to-the-cloverbootloader-documentation-clover-documentation-site)

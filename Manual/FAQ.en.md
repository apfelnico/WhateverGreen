#### Quick FAQ:
- _When do I need WhateverGreen?_  
If you run macOS 10.11 or newer (possibly latest 10.10 as well) with an ATI/AMD GPU 5xxx or newer, you most likely do.  
Unfortunately it is not possible to test all the GPUs and their configurations, use at your own risk.

- _How do I get the debug log?_  
Install DEBUG versions of WhateverGreen and Lilu, then add `-raddbg -liludbg` to the boot arguments. Once you boot run the following command in terminal:  
`log show --predicate 'process == "kernel" AND (eventMessage CONTAINS "WhateverGreen" OR eventMessage CONTAINS "Lilu")' --style syslog --source`  
If you have macOS 10.11 or earlier, use this command:  
`cat /var/log/system.log | egrep '(WhateverGreen|Lilu)'`  
Please note that in the case you cannot boot if your problem is specific to the GPU you should be able to get the log via SSH. In this case also check the `kextstat` command output.

- _What is the state of 10.13 support?_  
At the time of the release 10.13 is still being tested, so no support could even be thought about. There exist cases of broken AMD graphics on pre-Nehalem CPU chipsets. If you have older hardware please stay away from using 10.13. For other systems WhateverGreen may work if no drastic changes happen in 10.13.

- _What are the hardware requirements for WhateverGreen?_  
Full UEFI without CSM. You are strongly recommended to flash a UEFI-compatible ROM unless your card already has it. Failing to do so will quite likely result in issues in multi-monitor configurations and possibly even in single-monitor configurations. It may stil work for non-UEFI motherboards, try at your own risk. There are known issues when using 2 or more GPUs in multi-monitor configurations.

- _How do I flash my GPU?_  
You could follow [this guide](https://www.techpowerup.com/forums/threads/amd-ati-flashing-guide.212849/) in flashing your UEFI-enabled ROM. To create a UEFI-enabled ROM for your GPU you could follow [this guide](http://www.win-raid.com/t892f16-AMD-and-Nvidia-GOP-update-No-requests-DIY.html). Note, that there is [a tool](http://www.insanelymac.com/forum/topic/299614-asus-eah6450-video-bios-uefi-gop-upgrade-and-gop-uefi-binary-in-efi-for-many-ati-cards/#entry2042163) for macOS, it could also work, but you will likely have to take care of extra padding.

- _When should I use a named framebuffer?_  
Named framebuffers (Baladi, Futomaki, Lotus, etc.), enabled by "Clover GPU injection" or any other methods should _never ever be used_. This way of GPU injection is a common mistake, preventing automatic configuration of various important GPU parameters. This will inavoidably lead to borked GPU functioning in quite a number of cases.

- _When and how should I use custom connectors?_  
In general automatic controller detection written in Apple kexts creates perfect connectors from your VBIOS. The logic of that can be found in [reference.cpp](https://github.com/vit9696/WhateverGreen/blob/master/Manual/reference.cpp). However, some GPU makers physically create different connectors but leave the VBIOS unchanged. This results in invalid connectors that are incompatible with your GPU. The proper way to fix the issues is to correct the data in VBIOS, however, just providing custom connectors can be easier.  
For some GPUs (e.g. 290, 290X and probably some others) WhateverGreen incorporates automatic connector correction that can be enabled via `-raddvi` boot argument. For other GPUs you may specify them as a GPU device property called `connects`, for example, via SSDT. You could pass your connectors in either 24-byte or 16-byte format, they will be automatically adapted to the running system. If you need to provide more or less connectors than it is detected automatically, you are to specify `connector-count` property as well. Please note that automatically detected connectors appear in the debug log to give you a good start.

- _How can I change display priority?_  
With 7xxx GPUs or newer you could simply add `connector-priority` GPU controller property with sense ids (could be seen in debug log) in the order of their importance. This property may help with black screen issues especially with the multi-monitor configurations.  
Without this property specified all the connectors will stay with 0 priority. If there are unspecified connectors they will be ordered by type: LVDS, DVI, HDMI, DP, VGA. Read [SSDT sample](https://github.com/vit9696/WhateverGreen/blob/master/Manual/Sample.dsl) for more details.

- _What properties should I inject for my GPU?_  
Very few! You should inject an `HDAU` device to your GPU controller, `hda-gfx` properties with a corresponding number to the amount of audio codecs you have, and that is basically all. If you need to mask to an unsupported GPU, additionally add `device-id`. It is also recommended to add some cosmetic properties: `AAPL,slot-name` (displayed slot name in system details), `@X,AAPL,boot-display` (boot logo drawing issues), `model` (GPU display name, if detection failed).  
While not pretending to be perfect, there is a [SSDT sample](https://github.com/vit9696/WhateverGreen/blob/master/Manual/Sample.dsl) to get the general idea.

- _How could I tune my GPU configuration?_  
ATI/AMD GPUs could be configured by `aty_config`, `aty_properties` parameters that one could find in AMDxxxxController.kext Info.plist files. Different framebuffers have overrides for these preferences for lower energy consumption, higher performance, or other reasons. Unfortunately in such combinations they often are unsuitable for different GPUs.  
WhateverGreen allows to specify your own preferred parameters via GPU controller properties to achieve best GPU configuration. Use `CFG,` prefix for `aty_config` properties, `PP,` prefix for `aty_properties`, and `CAIL,` prefix for `cail_properties` from AMDRadeonXxxxx.kext Info.plist files. For example, to override `CFG_FB_LIMIT` value in `aty_config` you should use `CFG,CFG_FB_LIMIT` property.  
Further details are available in [SSDT sample](https://github.com/vit9696/WhateverGreen/blob/master/Manual/Sample.dsl) to get the general idea.

- _When do I need to use `radpg` boot argument?_  
This argument is as a replacement for the original igork's AMDRadeonX4000.kext Info.plist patch required for HD 7730/7750/7770/R7 250/R7 250X initialisation (`radpg=15`) GPUs to start. WhateverGreen is not compatible with Verde.kext, and it should be deleted. The argument allows to force-enable certain power-gating flags like CAIL_DisableGfxCGPowerGating. The value is a bit mask of CAIL_DisableDrmdmaPowerGating, CAIL_DisableGfxCGPowerGating, CAIL_DisableUVDPowerGating, CAIL_DisableVCEPowerGating, CAIL_DisableDynamicGfxMGPowerGating, CAIL_DisableGmcPowerGating, CAIL_DisableAcpPowerGating, CAIL_DisableSAMUPowerGating. Therefore `radpg=15` activates the first four keys.  
Starting with 1.0.3 version you could achieve exactly the same result by manually specifying the necessary properties into your GPU controller (e.g. `CAIL,CAIL_DisableGfxCGPowerGating`). This is better, because it allows you to fine-tune each GPU separately instead of changing these parameters for all the GPUs.

- _How to change my GPU model?_  
The controller kext (e.g. AMD6000Controller) replaces GPU model with a generic name (e.g. AMD Radeon HD 6xxx) if it performs the initialisation on its own. Injecting the properties and disabling this will break connector autodetect, and therefore is quite not recommended. WhateverGreen attempts to automatically detect the GPU model name if it is unspecified. If the autodetected model name is not valid (for example, in case of a fake device-id or a new GPU model) please provide a correct one via `model` property. All the questions about automatic GPU model detection correctness should be addressed to [The PCI ID Repository](http://pci-ids.ucw.cz). In special cases you may submit a [patch](https://github.com/vit9696/WhateverGreen/pulls) for [kern_model.cpp](https://github.com/vit9696/WhateverGreen/blob/master/WhateverGreen/kern_model.cpp). GPU model name is absolutely unimportant for GPU functioning.

- _How should I read EFI driver version?_  
This value is by WhateverGreen for debugging reasons. For example, WEAD-102-2017-08-03 stands for WhateverGreen with automatic frame (i.e. RadeonFramebuffer), debug version 1.0.2 compiled on 03.08.2017. Third letter can also be F for fake frame, and B for invalid data. Fourth letter could be R for release builds. 

- _What to do when my GPU does not wake until I start typing on the keyboard?_  
If this bothers you, either wait a bit longer or try adding `darkwake=0` boot argument.

- _How do I know my GPU initialises fine?_  
One of the easiest signs is boot time. If initialised improperly, your boot process will stall for 30 extra seconds to get your display ready.

- _Is it normal to have `Prototype` in OpenGL/OpenCL engine names?_  
Yes. It was discovered during the reverse-engineering that the displayed title has no effect on performance. Furthermore, it was discovered that certain attempts to patch this by modifying the identitiers in kexts (e.g. AMDRadeonX4000) may lead to overall system instability. For the improperly coded apps having issues with such naming use `libWhateverName.dylib`.

- _How do I use my IGPU?_  
In most cases IGPU should be used for hardware video decoding (with a connector-less frame). In case you need extra screens, IGPU may be fully enabled. You should not use `-radlogo` boot argument with IntelGraphicsFixup.

- _How do I get hardware video decoding to work?_  
Generally hardware video decoding is performed by an IGPU, and thus you are required to inject a connector-less frame. For some GPUs (e.g. HD 7870, HD 6670, HD 7970) it is still possible to get AMD hardware video decoding to work. Please refer to [Shiki's FAQ](https://github.com/vit9696/Shiki/blob/master/FAQ.en.md) for further details. You may need `-shikigva` on mac models that have forceOfflineRenderer on.

- _Why would I need to force 24-bit video output?_  
Several screens may not support 30-bit video output, but the GPU may not detect this. The result will look as distorted blinking colours. To resolve the issue either buy a more powerful display or add `-rad24` boot argument.  

- _How do I get HDMI audio to work?_  
In general it should be enough to rely on WhateverGreen automatic HDAU correction. It renames the device to HDAU, and injects missing layout-id and hda-gfx (starting with onboard-2) properties. This will not work well with two or more cards of different vendors (e.g. NVIDIA and ATI/AMD), please manually inject the properties in such a case.  
For identifiers not present in AppleHDAController and AppleHDA you have to add necessary kext patches, see AppleALC [example for 290X](https://github.com/vit9696/AppleALC/commit/cfb8bef310f31fd330aeb4e10623487a6bceb84d#diff-6246954ac288d4f6dd7eb780c006419d).

- _May I access the source code?_  
Model detection code is [open](https://github.com/vit9696/WhateverGreen/blob/master/WhateverGreen/kern_model.cpp) as well as [Lilu](https://github.com/vit9696/Lilu). If you want to contribute a feature you have an implementation for please contact me. For example, getting better handling of AMD Switchable Graphics or providing more complete research on connector detection would be nice.

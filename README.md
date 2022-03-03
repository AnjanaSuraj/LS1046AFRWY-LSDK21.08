# lsdk-frwy

NXP's LSDK does not provide pre-built UEFI firmware image for LS1046AFRWY platform.

This repository holds pre-built UEFI firmware image for LS1046AFRWY platform and also provides details to build your own UEFI firmware image for LS1046AFRWY.

**The Pre-built images are kept in the "Pre-Built Images" folder of this repository.**

Refer to Section "4.1.10 LSDK Quick Start Guide for FRWY-LS1046A" of LSDK 21.08 user Guide - https://www.nxp.com/docs/en/user-guide/LSDKUG_Rev21.08.pdf
to program the images. 

**To build your own UEFI image for LS1046AFRWY,**

1. Follow the Section "5.4.5 LSDK Distro Boot Logs" in LSDK 21.08 user Guide - https://www.nxp.com/docs/en/user-guide/LSDKUG_Rev21.08.pdf
to setup the workspace and toolchain and clone all the required repositories.

2. Checkout to LSDK-21.08 tag using command "git checkout LSDK-21.08 -b LSDK-21.08" in all the repositories.

3. Before compiling the cloned repositories, apply the patch files present in "Patches" folder of this repository.
   These patches have to be applied in edk2-platforms repository. 
   Command to apply the patches:  git am <name.patch> --keep-cr
   
   a. Patch 1 : 48d26b8f Platform/NXP/LS1046aFrwyPkg: Include SmbiosPlatformDxe.inf for static ACPI build
      This patch is required to compile the UEFI source code succesfully with DYNAMIC_ACPI_ENABLE set to FALSE. 
      This patch includes SMBIOS DXE Driver when DYNAMIC_ACPI_ENABLE is set to false. This driver installs SMBIOS information for NXP platforms.

   b. Patch 2 : b1bd5a26 Silicon/NXP: Fix a bug in static ACPI build code
      This patch is required to resolve a runtime error when we program a UEFI image built with DYNAMIC_ACPI_ENABLE set to false. 
      If this patch is not applied, UEFI boot will stop with an assert error in MdePkg/Library/BaseLib/CheckSum.c.


4. After applying the patches, continue the steps mentioned in LSDK 21.08 User guide to compile UEFI. 
Commands for LS1046AFRWY:

To build UEFI image for LS1046AFRWY with Dynamic ACPI enabled -
build -p Platform/NXP/LS1046aFrwyPkg/LS1046aFrwyPkg.dsc -a AARCH64 -t GCC5 -b RELEASE -D FIRMWARE_VER="LSDK21.08_Dynamic" -D FIRMWARE_REV=0x00150008 -D DYNAMIC_ACPI_ENABLE=TRUE -D DYNAMIC_ACPI_PKG=LS1046aFrwyPkg

To build UEFI image for LS1046AFRWY with Dynamic ACPI disabled -
build -p Platform/NXP/LS1046aFrwyPkg/LS1046aFrwyPkg.dsc -a AARCH64 -t GCC5 -b RELEASE -D FIRMWARE_VER="LSDK21.08_Static" -D FIRMWARE_REV=0x00150008

5. ATF build commands for LS1046AFRWY
$ make PLAT=ls1046afrwy bl2 pbl BOOT_MODE=qspi RCW=rcw_1600_qspiboot.bin
$ make PLAT=ls1046afrwy fip BL33=LS1046AFRWY_EFI.fd

This will create fip.bin and bl2_qspi.pbl in /build/ls1046afrwy/release/ folder.

6. For Flashing fip.bin and bl2_qspi.pbl images on LS1046AFRWY refer to Section "5.2.3.3 How to program TF-A binaries on specific boot mode"
LSDK 21.08 user Guide - https://www.nxp.com/docs/en/user-guide/LSDKUG_Rev21.08.pdf

LS1046AFRWY has only one QSPI NOR Flash bank. 
So boot uboot firmware from SD card and then Program QSPI NOR flash: = sf probe 0:0
Follow the document for rest of the commands for programming.



# esx-boot

## Overview

esx-boot is the VMware ESXi bootloader.  

The same source tree builds two different bootloader configurations, one for booting in UEFI mode, the other for booting in legacy BIOS mode.

* UEFI esx-boot: Runs on its own directly on top of the host UEFI firmware.

* Legacy BIOS esx-boot: Runs on top of the open-source bootloader "syslinux".

You probably don't want to use esx-boot to boot anything other than ESXi.  esx-boot has some support for a slightly extended version of the Multiboot standard, but ongoing development is using a new 64-bit boot API specific to VMware called "Mutiboot".

## Try it out

esx-boot is not terribly easy to try out.

On an ESXi installation, the bootloader lives in the UEFI system partition, which ESXi itself currently is not able to mount, so it's not easy for an end user to replace the bootloader with a custom build.  You could boot into the UEFI shell and access the partition that way, or you could copy a partition image elsewhere, modify it with mtools or the like, and then copy it back.

On an ESXi ISO image, see the documentation below for how those work.  If you want to modify an existing ISO, you'll need a tool that can handle an ISO with two El Torito images.  Otherwise you'll lose the feature of being able to boot the same ISO on both UEFI and legacy BIOS.

It's easier to use your own bootloader build if you're network booting.  See https://www.vmware.com/techpapers/2015/installing-vmware-esxi-6.0-using-pxe-10508.html for general instructions about network booting ESXi.

### Prerequisites

* A Linux system to build on.

* An ESXi system to boot.

### Build & Run

1. ./configure

2. Edit the file env/toolchain.mk.  See the comments in the file and
   also see env/toolchain-example.mk.  The latter file is set up to
   use the VMware common development kit toolchain, but you'll
   probably prefer to use the build tool versions packaged with your
   Linux distro.

3. make

## Documentation

esx-boot consists of several modules.

The main bootloader module is called mboot. The mboot module is responsible for:

    1. Reading ESXi's boot configuration file (boot.cfg), and retrieving information about:
    - location of the kernel and boot modules
    - kernel boot options
    2. Loading the kernel and other modules from the boot media into main memory
    3. Verifying the cryptographic signatures on the early modules for Secure Boot purposes.
    4. Setting up system information (Multiboot or Mutiboot) structures and passing them to the kernel
    5. Preparing the firmware for kernel hand-off.
    6. Handing off to the kernel. 

Depending on how you boot ESXi, one or more other esx-boot modules may run prior to mboot.

### UEFI Boot

For UEFI boot from disk, UEFI firmware initially loads a UEFI build of esx-boot's safeboot module from the FAT filesystem in the boot partition (partition 1 or 4 depending whether the disk is partitioned using GPT or MBR).  safeboot then determines which of the two bootbank partitions to boot from and invokes esx-boot's mboot module with either -p 5 or -p 6 on the mboot command line.

For UEFI boot from CD/DVD, the UEFI firmware is not able to read the ISO9660 file format of most of the ESXi disc. To get around this problem, esx-boot's isobounce module is installed in the second El Torito image of the ESXi CD/DVD. It is in the file \EFI\BOOT\BOOTx64.EFI in the FAT filesystem in the El Torito image (not in that filename in the ISO9660 filesystem!). isobounce loads esx-boot's UEFI ISO9660 filesystem driver from the FAT filesystem in the El Torito image, then runs esx-boot's mboot module.

For UEFI boot from the network, UEFI firmware can load esx-boot's mboot module directly.  See https://www.vmware.com/techpapers/2015/installing-vmware-esxi-6.0-using-pxe-10508.html for details.  Alternatively, you could use esx-boot's menu module to set up a menu of different images to boot; see the top of menu.c for documentation.

### Legacy BIOS Boot

Legacy BIOS boot starts with syslinux, and the esx-boot modules run on top of syslinux

For legacy BIOS disk boot, syslinux's configuration file, \syslinux.cfg, tells it to run esx-boot's safeboot.c32 module with suitable command-line options.   The safeboot module then determines which of the two bootbank partitions to boot from (partition 5 or 6) and invokes esx-boot's mboot.c32 with either -p 5 or -p 6 on the mboot command line.

For legacy BIOS boot from CD/DVD, isolinux is installed in the first El Torito image of the CD/DVD. The configuration file for isolinux is /isolinux.cfg. This file tells syslinux to invoke esx-boot's mboot.c32 module with suitable command-line options.

For legacy BIOS boot from the network, pxelinux runs first and loads esx-boot's mboot.c32.  See https://www.vmware.com/techpapers/2015/installing-vmware-esxi-6.0-using-pxe-10508.html for details.

### A note on UEFI Secure Boot

This open source release of esx-boot includes machinery for signing the generated binaries for UEFI Secure Boot purposes.  Two non-secret test certificates are provided in the package as a sample.  Read the source code and Makefiles if you would like to see how to sign the binaries you create with your own certificate(s).

## Releases & Major Branches

We have seeded esx-boot's github repository with the ESXi 6.7u1 version of esx-boot.  Newer esx-boot versions are kept backward compatible with older versions of ESXi, so we do not branch esx-boot even when an older ESXi release needs a bugfix; we simply pull in the newest stable version from the main branch.  So at least to start with, github has only one release and only one branch.

If you are interested in versions of esx-boot that have been used in other ESXi releases, they are available on VMware's ESXi Open Source Disclosure Package ISO images, downloadable from http://www.vmware.com.

## Contributing

The esx-boot project team welcomes contributions from the community. If you wish to contribute code and you have not
signed our contributor license agreement (CLA), our bot will update the issue when you open a Pull Request. For any
questions about the CLA process, please refer to our [FAQ](https://cla.vmware.com/faq). For more detailed information,
refer to [CONTRIBUTING.md](CONTRIBUTING.md).

## License

This product is licensed to you under the GNU GENERAL PUBLIC LICENSE Version 2 license (the "License").  You may not use this product except in compliance with the GPL 2.0 License.  

This product may include a number of subcomponents with separate copyright notices and license terms. Your use of these subcomponents is subject to the terms and conditions of the subcomponent's license, as noted in the LICENSE file. 

# bbb
Device driver development on beagle bone black.
        
        Hey, for the past few days I had been working on getting my own Linux distro to work on the piece of hardware I had at home and I think it would be worth the while to share my findings with you all. If you are new to this just like me, I would recommend reading this article as I break things down and provide references to some of the important aspects. However I must mention, there are other really informative articles / answers out there but this one aims to put all of them under one roof.

When you buy a piece of hardware, usually there would be an image pre flashed onto the emmc so that you could just plug and play. These images get out dated over time and the device vendor might have provided you with latest pre-built images that you would have to flash. But today, we will build our own images and bring up with device.

Advantages of building a custom Linux image:

    You could restrict the image size according to your memory constraints by including or excluding specific modules of your choice.

    You could tweak the configurations to improve the performance of your device.

    Write and include your own device drivers onto the kernel.

    In case you are planning to contribute to the open source community, building and testing your image is a must :)

This is indeed a very vast topic to discuss but I would limit this article to the following areas and the second one is really the prize winner.

    Create custom boot and Kernel images.

    How to interpret errors.

Please do jump into the hyperlinks if you are curious to know more about some of the topics that I cannot get into adhering to the scope of this article.
Create custom boot and Kernel images
Booting Process

Your target device could be either a physical device like Beagle-Bone / Zynq / Vision2 or anything else, as a matter of fact you could even use the QEMU emulator which can emulate any hardware architecture, but before you get started, it is important for you to fathom the hardware specs of it.

    What is the architecture of the core? 

    How much internal memory does it have?

The core architecture is important because the way you instruct the processor to do something is through the ISA and this is distinct to each architecture. Is it ARM based ? or x86 or a RISC-V? On the basis of this, you would have to pick the board support package file which is usually provided by the vendor. This file contains hardware specific info: boot, firmware, on-board RAM, I2C/SPI peripherals and other routines.

You also need to ensure that the size of your first stage boot loader does not exceed the size of the internal SRAM of your device.

From the above block diagram, We would require the following files for our device to just to boot up:

    MLO (.bin) 

    u-boot (.img)

    uEnv (.txt)

    uImage /zImage (.bin)

    Device tree blob (.dtb)

     Root file system

There are multiple ways to build these image, which are:

    u-boot source code ( does not build the kernel or root file system images )

    The Buildroot.

    The Yocto project.

    Linux source code ( Builds the kernel images only )

I would like to pick out the Yocto project here because of the following reasons:

    Most vendors support Yocto directly, which means you already have the BSP files and configurations available.

    The OpenEmbedded build system creates an entire Linux distribution, including the toolchain, from source.

    Yocto allows you to create custom Linux distributions, with a wide range of configuration options and package management tools.

    Reduced build time over successive attempts by caching the build attributes.

However, It is a heavy suite and requires some learning time.
Components of Yocto Project

You could head out to the below link for detailed build procedure:

To brief it up,

    You have installed the required packages based on your host machine.

    Cloned the poky repository and checked out the latest LTS release.

    Modified the /poky/build/config/local.conf file and /poky/build/config/local.conf file in case your device's BSP had to be included manually.

    Initialized the build environment which sets up bitbake.

    Add / remove packages using: bitbake -c menuconfig virtual/kernel

kpranav@Ubuntu22:~/Projects/yocto/poky/build$ bitbake -c menuconfig virtual/kernel
Loading cache: 100% |################################################################################################| Time: 0:00:01
Loaded 1644 entries from dependency cache.
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "2.0.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "universal"
TARGET_SYS           = "arm-poky-linux-gnueabi"
MACHINE              = "beaglebone-yocto"
DISTRO               = "poky"
DISTRO_VERSION       = "4.0.17"
TUNE_FEATURES        = "arm vfp cortexa8 neon callconvention-hard"
TARGET_FPU           = "hard"
meta                 
meta-poky            
meta-yocto-bsp       = "kirkstone:1e0d58c53b7d9c3feb631e46666ae7a3e3614253"

Initialising tasks: 100% |###########################################################################################| Time: 0:00:00
Sstate summary: Wanted 0 Local 0 Mirrors 0 Missed 0 Current 101 (0% match, 100% complete)
NOTE: Executing Tasks

Now, you can select the packages / drivers as: Build-in / Loadable / Not Included.

 You finally star the building the bootable files by issuing : bitbake <recipes>. This would take a couple of hours depending upon your machine. Once it is done, You'll be able to find all your files at the following location:

/poky/build/tmp/deploy/images/<Machine Name>
Output Images

You will also be able to find the root file system file with .tar.bz2 file extension which has to be extracted and mounted on the second portion of the sd-card as shown below.

Lastly, uEnv.txt is a editable file that is used to set up the boot environment and configure the bootloaders to find the kernel images at a particular memory location. 
SD card partition

Now, insert the sd-card and press the boot button to boot via SD-Card (mmc0).

You should see the kernel booting up and now you can login into your device.

[  OK  ] Reached target Hardware activated USB gadget.
         Starting LSB: set CPUFreq kernel parameters...
         Starting A high performanc… and a reverse proxy server...
         Starting OpenBSD Secure Shell server...
         Starting Permit User Sessions...
[   27.149074] musb-hdrc musb-hdrc.1: VBUS_ERROR in a_wait_vrise (88, <AValid), retry #3, port1 0008010c
[  OK  ] Finished Permit User Sessions.
[  OK  ] Started Getty on tty1.
[  OK  ] Started Serial Getty on ttyGS0.
[  OK  ] Started Serial Getty on ttyS0.
[  OK  ] Reached target Login Prompts.
[  OK  ] Started LSB: set CPUFreq kernel parameters.
[  OK  ] Started OpenBSD Secure Shell server.
[  OK  ] Started A high performance…er and a reverse proxy server.
[  OK  ] Reached target Multi-User System.
[  OK  ] Finished Remove Stale Onli…ext4 Metadata Check Snapshots.
[  OK  ] Reached target Graphical Interface.
         Starting Update UTMP about System Runlevel Changes...
[  OK  ] Finished Update UTMP about System Runlevel Changes.

Debian GNU/Linux 11 BeagleBone ttyS0

BeagleBoard.org Debian Bullseye IoT Image 2023-09-02
Support: https://bbb.io/debian
default username:password is [debian:temppwd]

BeagleBone login: debian
Password: 

How to interpret errors:

If you face issues at the time of building the images , it would mostly be related to config mismatch or missing packages for which you will have the debug prints.

You might face issues once the images are flashed. 

Few of the issues you could face are:

    Boot loader fails load the kernel image.

## Flattened Device Tree blob at 88000000
   Booting using the fdt blob at 0x88000000
   Loading Kernel Image ... OK
   Loading Device Tree to 8ffeb000, end 8ffff1df ... OK

Starting kernel ...

    Once the kernel image is located and the kernel initialization starts, You might encounter issues related to symbols not getting loaded.

U-Boot 2019.04-00002-g31a8ae0206 (May 13 2020 - 09:26:17 -0500), Build: jenkins-github_Bootloader-Builder-139

CPU  : AM335X-GP rev 2.1
I2C:   ready
DRAM:  512 MiB
No match for driver 'omap_hsmmc'
No match for driver 'omap_hsmmc'
Some drivers were not found
Reset Source: Global external warm reset has occurred.
Reset Source: Power-on reset has occurred.
RTC 32KCLK Source: External.
MMC:   OMAP SD/MMC: 0, OMAP SD/MMC: 1
Loading Environment from EXT4... 
** Unable to use mmc 0:1 for loading the env **
Board: BeagleBone Black
<ethaddr> not set. Validating first E-fuse MAC
BeagleBone Black:
BeagleBone: cape eeprom: i2c_probe: 0x54:
BeagleBone: cape eeprom: i2c_probe: 0x55:
BeagleBone: cape eeprom: i2c_probe: 0x56:
BeagleBone: cape eeprom: i2c_probe: 0x57:
Net:   eth0: MII MODE
cpsw, usb_ether
Press SPACE to abort autoboot in 0 seconds
=> 
=>help
?         - alias for 'help'
askenv    - get environment variables from stdin
base      - print or set address offset
bdinfo    - print Board Info structure
boot      - boot default, i.e., run 'bootcmd'
bootd     - boot default, i.e., run 'bootcmd'
bootefi   - Boots an EFI payload from memory
bootelf   - Boot from an ELF image in memory
bootm     - boot application image from memory
bootp     - boot image via network using BOOTP/TFTP protocol
bootvx    - Boot vxWorks from an ELF image
bootz     - boot Linux zImage image from memory
btrsubvol - list subvolumes of a BTRFS filesystem
cmp       - memory compare
coninfo   - print console devices and information
cp        - memory copy
crc32     - checksum calculation
dfu       - Device Firmware Upgrade
dhcp      - boot image via network using DHCP/TFTP protocol
dm        - Driver model low level access
echo      - echo args to console
editenv   - edit environment variable
eeprom    - EEPROM sub-system
env       - environment handling commands

Your kernel panics while booting
[    1.548865] Kernel panic - not syncing: Attempted to kill init! exitcode=0x0000000b

---
weight: 4
title: "Custom firmware for Rapsberry Pi 3 using buildroot"
date: 2018-10-05T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: "How to build a custom Linux firmware on Raspberry Pi using buildroot."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["rpi", "buildroot", "ensibs", "firmware", "iot", "embedded"]
categories: ["Shorts"]

lightgallery: true
---

## I. Step 1

In this first part, we had to set up the buildroot system basis.

Goals of this part are:

*   Minimal buildroot configuration
*   Copy the new firmware on RPi SD card

### I.1. Base system specification

After downloaded the last release of buildroot, I prepared the build for Raspberry Pi 3 architecture.

```bash
make raspberrypi3_defconfig
```    

In this interface “**menuconfig**”, I have to initiate the RAM filesystem:

```bash
Filesystem images → initial RAM filesystem linked into linux kernel
```    

Furthermore, as we can see in the picture above, I disabled the ext3/4 filesystem, it’s not necessary here.

I defined the root password:

```bash
System configuration → Root password
```

I also changed the hash function, MD5 or SHA1 are deprecated.

The toolchain requested is Linario ARM, then I have to configure buildroot to use an external toolchain:

```bash
Toolchain → Toolchain type → External toolchain
```

### I.2. dd on the SD card

Now the build is ready for the Raspberry Pi 3, a little shot of **dd** on the SD card, and it’s done.

As mention above, the root filesystem is not required here, then I edited the “genimage-raspberrypi3.cfg” file, and remove this part. I saved the new file in “genimage-raspberrypi3-ENSIBS.cfg”.

Given that we modified the **genimage** name, I have to edit the “post-image.sh” file too (cf. Annexe 1 - Genimage ENSIBS & post-image.sh).

The firmware is ready, I have to build it, just **make** command in the buildroot root folder. When all dependencies are installed, I flashed the SD card:

```bash
dd if=output/images/sdcard.img of=/dev/mmcblk
```

## II. Step 2

The step 2 consists to add the network layer in our firmware. As well as a path that allows us to edit configuration files in the firmware (it’s a _overlays_).

Goals are:

*   Setting up an overlay
*   Configuration of the network layer

### II.1. Setting up on overlay

An overlay is a folder on the host system, it allows us to edit files in the firmware. Typically, we have to edit the **interfaces** file in **/etc/network/** folder to setting up the network layer of our firmware. For this, I just have to create a directory and mention it in buildroot. For example, my path is:

```bash
/home/billy/Documents/Cours/ENSIBS/secuEmbarque
```

In the first instance, the folder is empty, but at the end of the project, here is mine:

```bash
    ▶ tree .
    .
    └── etc
        ├── init.d
        │   └── S40network
        ├── network
        │   └── interfaces
        └── profile
    
    3 directories, 3 files
```

### II.2. Network configuration

On the basis of the previous tree, we had to edit the **interfaces** and **S40network** file.

First of all, I noticed that the startup script for the ethernet interface (**S40network**) is triggered before the network card (hardware). I just add a 5 seconds sleep (cf. Annexe 2 - Sleep in S40Network file).

The “**interface**” file (cf. Annexe 3 - Static network configuration) allows the firmware to get a static address IP, a gateway, static network mask on a specific interface (i.e **eth0** here). The firmware will have the IP address: 10.1.49.4⁄24 and the gateway will be my host system: 10.1.49.3.

### II.3. Dropbear SSH installation

To install a known package in buildroot, it’s pretty easy. I just have to select it in the buildroot menuconfig:

```bash
Target packages → Network Applications → dropbear
```    

Now, I must rebuild the firmware and flash again the SD card.

The default configuration of dropbear allows root connection, in our case, the only user on the firmware is the root user.

I will maybe add a hardening part later :D

Connection as root on the raspberry pi through SSH:

```bash
ssh root@10.1.49.4
```    

### II.4. Update through network

The network connection is necessary to update (through SSH) the Raspberry Pi without unplugging the SD card from the Pi.

I made a little script (cf. Annexe 4 - Update script) to do this.

It following those steps:

*   Copy the new firmware on the Pi (scp)
*   Use the **dd** command to write the firmware on the SD card
*   Restart the Raspberry Pi

```bash
▶ firm_upgrade sdcard.img /dev/mmcblk0 10.1.49.4
```    

This method works because the system is load in the RAM, so it won’t do any call to the SD card. By the way, we could remove the SD card from the raspberry pi during it operates, it will still work.

To improve my script, I could copy only the **zImage** file instead of the complete firmware (**sdcard.img**), for the V2! :p

## III. Step 3

For this final step, I will install an external package (have to download sources, compile them and install, do the same thing with dependencies) and startup this package at boot.

Goals are:

*   Install external package
*   Check dependencies
*   Start the package at boot

### III.1. Install external package

The external package is: **nInvaders**. A Space Invaders for terminal! Very cool! :D

Tells to buildroot that we want to add an external package in the firmware, I have to add an entry **Config.in** (located in the root directory of buildroot). After that, the external package appears in the _menuconfig_ of buildroot.

```bash
    buildroot/Config.in
    

    [...]
    menu "ninvaders [external repo]"
            source package/ninvaders/Config.in
    endmenu
    [...]
```

Then, create two files “Config.in”, “ninvaders.mk” in a folder called “ninvaders” (we have to create the folder).

```bash
    ▶ tree ninvaders 
    ninvaders
    ├── Config.in
    └── ninvaders.mk
```

*   Config.in: This file tells to buildroot name of the external package, dependencies and help menu.
*   ninvaders.mk: It’s a Makefile, it will build the package with all dependencies, but it’s not a standard Makefile, there is some environment variable and other settings manage by buildroot for the architecture (here it’s for Raspberry Pi 3).

Files are available in Annexe 5.

### III.2. Check dependencies

NInvaders package needs **ncurses** dependancies. Fortunately, this package is easy to install and don’t need further dependencies.

In the file **Config.in** of nInvaders, there is the following line:

```bash
    select BR2_PACKAGE_NCURSES
```

It will specify to buildroot that **nInvaders** needs **ncurses**.

To compile this dependency, we just need to add this line in our Makefile **ninvaders.mk**:

```bash
    NINVADERS_DEPENDENCIES = ncurses
```

After a little **make**, the package nInvaders is successfully installed! Now, just connect to the Pi through SSH and execute **ninvaders**:

```bash
ninvaders
```
![](/lib/images/shorts/rpibuildroot/ninvaders.gif)

### III.3. Start the package at boot

When the nInvaders package is successfully installed, we will execute it automatically when a session opens.

In our overlays, we need to add (at the end) the **profile** file in folder **/etc** of the firmware with the following lines:

```bash
export TERM=xterm
ninvaders
```

Complete file is available in Annexe 6 - /etc/profile file.

The **profile** file will be executed when user shell is open. It’s quite like _.bashrc_ with a bash shell.

## Conclusion

To conclude this article, this project was very interesting, it’s a good introduction for people (like me) without knowledge of firmware building.

### TODO

*   Hardening part
*   Update script
*   Build more interesting (honeypot IoT ?)

* * *

Annexes
=======

Annexe 1 - Genimage ENSIBS & post-image.sh
------------------------------------------

> genimage-raspberrypi3-ENSIBS.cfg

```bash
    image boot.vfat {
      vfat {
        files = {
          "bcm2710-rpi-3-b.dtb",
          "rpi-firmware/bootcode.bin",
          "rpi-firmware/cmdline.txt",
          "rpi-firmware/config.txt",
          "rpi-firmware/fixup.dat",
          "rpi-firmware/start.elf",
          "rpi-firmware/overlays",
          "zImage"
        }
      }
      size = 32M
    }
    
    image sdcard.img {
      hdimage {
      }
    
      partition boot {
        partition-type = 0xC
        bootable = "true"
        image = "boot.vfat"
      }
    
    }
```

* * *

> post-image.sh
    

```bash
    #!/bin/sh
    
    BOARD_DIR="$(dirname $0)"
    BOARD_NAME="$(basename ${BOARD_DIR})"
    #GENIMAGE_CFG="${BOARD_DIR}/genimage-${BOARD_NAME}.cfg"
    GENIMAGE_CFG="${BOARD_DIR}/genimage-raspberrypi3-ENSIBS.cfg"
    GENIMAGE_TMP="${BUILD_DIR}/genimage.tmp"
    
    case "${2}" in
        --add-pi3-miniuart-bt-overlay)
        if ! grep -qE '^dtoverlay=' "${BINARIES_DIR}/rpi-firmware/config.txt"; then
            echo "Adding 'dtoverlay=pi3-miniuart-bt' to config.txt (fixes ttyAMA0 serial console)."
            cat << __EOF__ >> "${BINARIES_DIR}/rpi-firmware/config.txt"
    
    # fixes rpi3 ttyAMA0 serial console
    dtoverlay=pi3-miniuart-bt
    __EOF__
        fi
        ;;
    esac
    
    rm -rf "${GENIMAGE_TMP}"
    
    genimage                           \
        --rootpath "${TARGET_DIR}"     \
        --tmppath "${GENIMAGE_TMP}"    \
        --inputpath "${BINARIES_DIR}"  \
        --outputpath "${BINARIES_DIR}" \
        --config "${GENIMAGE_CFG}"
    
    exit $?
```

* * *

Annexe 2 - Sleep in S40network file
-----------------------------------

> overlays/etc/init.d/S40network
    
```bash
    #!/bin/sh
    #
    # Start the network....
    #
    # Debian ifupdown needs the /run/network lock directory
    mkdir -p /run/network
    
    case "$1" in
      start)
            printf "Starting network: "
            sleep 5
        /sbin/ifup -a
            [ $? = 0 ] && echo "OK" || echo "FAIL"
            ;;
      stop)
            printf "Stopping network: "
            /sbin/ifdown -a
            [ $? = 0 ] && echo "OK" || echo "FAIL"
            ;;
      restart|reload)
            "$0" stop
            "$0" start
            ;;
      *)
            echo "Usage: $0 {start|stop|restart}"
            exit 1
    esac
    
    exit $?
```

* * *

Annexe 3 - Set static IP address
--------------------------------

> overlays/etc/network/interfaces

```bash
    auto lo eth0 
    
    iface lo inet loopback
    
    iface eth0 inet static 
        address 10.1.49.4
        netmask 255.255.255.0
        gateway 10.1.49.3
```

* * *

Annexe 4 - Update script
------------------------

```bash
    #!/bin/sh
    
    #############
    # Upgrade script for buildroot
    #############
    
    function upgrade() {
        printf "[+] Upgrade starting...\n"
        scp $1 root@$3:/root/ 
        printf "[+] New firmware copied on the guest system.\n"
        ssh root@$3 "dd if=/root/$1 of=$2 bs=1M && /sbin/reboot"
        printf "[+] Release installed ! Rebooting.\n"
    }
    
    # Main program
    if [[ $# == 0 ]]; then
        printf "[!] Missing args !\n$(basename \"$0\") <new release> <partition path on guest> <guest ip>\nEx : $(basename "$0") sdcard.img /dev/mmcblk0 10.1.49.4"
    else
        # $1 = Path of the new release
        # $2 = Path of the disk on the guest, ex : /dev/mmcblk0
        # $3 = IP of the guest
        upgrade $1 $2 $3    
    fi
```

* * *

Annexe 5 - Config.in and ninvaders.mk
-------------------------------------

> buildroot/package/ninvaders/Config.in
    
```bash
    config BR2_PACKAGE_NINVADERS
        bool "ninvaders"
        select BR2_PACKAGE_NCURSES
        help
          Space invaders for RPi3 with Buildroot.r
```

* * *

> buildroot/package/ninvaders/ninvaders.mk
    
```bash
    ################################################################################
    #
    # ninvaders
    #
    ################################################################################
    
    NINVADERS_VERSION = 0.1.1
    NINVADERS_SITE = https://downloads.sourceforge.net/project/ninvaders/ninvaders/$(NINVADERS_VERSION)
    
    NINVADERS_DEPENDENCIES = ncurses
    
    define NINVADERS_BUILD_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) $(TARGET_CONFIGURE_OPTS) -C $(@D) all
    endef
    
    define NINVADERS_INSTALL_TARGET_CMDS
        $(INSTALL) -m 0755 -D $(@D)/nInvaders $(TARGET_DIR)/usr/bin/ninvaders
    endef
    
    $(eval $(generic-package))
``` 

* * *

Annexe 6 - /etc/profile file
----------------------------

> overlays/etc/profile
    
```bash
    export PATH=/bin:/sbin:/usr/bin:/usr/sbin
    
    if [ "$PS1" ]; then
        if [ "id -u" -eq 0 ]; then
            export PS1='# '
        else    
            export PS1='$ '
        fi
    fi
    
    export PAGER='/bin/more '
    export EDITOR='/bin/vi'
    
    # Source configuration files from /etc/profile.d
    for i in /etc/profile.d/*.sh ; do
        if [ -r "$i" ]; then
            . $i
        fi
        unset i
    done
    
    export TERM=xterm
    ninvaders
```
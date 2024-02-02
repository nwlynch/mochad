# MOCHAD

`mochad` is a Linux TCP gateway daemon for the X10 CM15A RF (radio frequency) and
PL (power line) and the CM19A RF controllers. 


## Why this fork ?

Changes were needed to the [Neil Cherry (linuxha) fork](https://github.com/linuxha/mochad) to compile and install `mochad` on recent versions of Linux with the `systemd` init system. Specifically this fork has been tested on 

- x86_64 GNU/Linux : Mint 20.1, Ubuntu 20.04 LTS (focal), Linux 5.4.0-124

- aarch64 GNU/Linux : Armbian 22.05.3, Ubuntu 22.04.1 LTS (jammy), Linux 5.10.123-meson64 

- aarch64 GNU/Linux : Raspberry Pi OS 2022-04-04, Debian 11.4 (bullseye), Linux 5.15.32-v8+

And also tested on the latest 32 bit version of Raspberry Pi OS for ArmV6 (for Raspberry Pi B and Raspberry Pi Zero)

- armv6l GNU/Linux : Raspberry Pi OS 2024-01-25, Debian 12.1 (bookworm), Linux 6.1.0

## Need for Changes

### Source

`mochad` could not be built because of the following linking errors

    /usr/bin/ld: decode.o:/home/hestia/mochad-master/global.h:31: multiple definition of `RfToRf16'; mochad.o:/home/hestia/mochad-master/global.h:31: first defined here
    /usr/bin/ld: decode.o:/home/hestia/mochad-master/global.h:29: multiple definition of `RfToPl16'; mochad.o:/home/hestia/mochad-master/global.h:29: first defined here
    /usr/bin/ld: decode.o:/home/hestia/mochad-master/global.h:26: multiple definition of `PollTimeOut'; mochad.o:/home/hestia/mochad-master/global.h:26: first defined here
    /usr/bin/ld: decode.o:/home/hestia/mochad-master/global.h:25: multiple definition of `Cm19a'; mochad.o:/home/hestia/mochad-master/global.h:25: first defined here

The problem was solved by declaring those variables as `extern` in `global.h` and defining `PollTimeOut` in `global.c`.

### Configuration

The `mochad` service was not installed properly in `systemd` and it would stop functioning with a 

    usb_claim_interface failed -6

error. 

The solution was to restore the `systemd` and `udev` directories found in the original [mochad-0.1.17](https://sourceforge.net/projects/mochad/files/) repository by mmauka. Modification of the `Makefile.am` was also necessary.

## Patching version 0.1.17 by mmauka

Instead of installing this fork, one could apply a couple of small patches to version 0.1.17 of `mochad` from the original author mmauka. The patches and details are [res directory](res/README.md).  

If IPV6 support is needed then this fork will have to be installed as explained next.

## Installation of `mochad`

**FR:** Il y a une [traduction en français de ces instructions](https://sigmdel.ca/michel/ha/domoticz/mochad_on_recent_linux_distro_fr.html#installation). 


### Prerequisite

    $ sudo apt install libusb-1.0-0-dev

On the Raspberry Pi it was also necessary to install `autoconf`

    $ sudo apt install autoconf

While not strictly necessary, `netcat` is useful when testing the `mochad` deaemon. It was present on two of the test platforms but it had to be added on the Raspberry Pi.    

    $ sudo apt install netcat-openbsd

### Getting the source

The source code on GitHub can be obtained with any one of the usual methods. Notably

#### by cloning the repository

    $ git clone https://github.com/sigmdel/mochad.git
    $ cd mochad


#### by downloading the archive

    $ wget https://github.com/sigmdel/mochad/archive/refs/heads/master.zip
    $ unzip master.zip
    $ cd mochad-master
   

### Enabling IPV6 support

By default, IPV6 is not enabled until the value of the IPV6 macro at the very start of `mochad.c` is changed.

```c
  #define IPV6    1
```
I do not use IPV6 and have not tested that code at all. Any questions about that feature would have to be addressed to its author, [Neil Cherry](https://github.com/linuxha/mochad).

### Compiling the source

From within the directory containing the source

    $ ./autogen.sh
    $ make


While it should be, make sure `autogen.sh` is an executable

    $ chmod +x autogen.sh 

if needed.    

### Installing the package

    $ sudo make install

Again within the directory containing the source.

### Installed files and cleanup

     /usr/local/bin/mochad
     /etc/udev/rules/91-usb-x10-controllers.rules


In `systemd` the service file is also installed

     /etc/systemd/system/mochad.service 

Note that the service will remain inactive until a CM1xA is connected to the system.

Once the installation is completed, all the other downloaded and generated files can be deleted if desired.

## Still getting `usb_claim_interface failed -6` error

If after installation and after the mochad.service is activated by plugging in a CM19A a `usb_claim_interface failed -6` is present check if the ati_remote driver is running.

    $ lsmod | grep ati
    ati_remote              9260  0   

The Lola remote for ATI All-In-Wonder video card has the same `0x0bc7:0x002` id as the CM19A. Because of that, the [ati_remote](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/media/rc/ati_remote.c) driver will be loaded. Blacklist the module and the problem should be solved. 

     $ echo "blacklist ati_remote" | sudo tee /usr/lib/modprobe.d/ati-remote-blacklist.conf

## More information

The [original README](README) text file contains much more information.

The version of `mochad.service` found in this fork comes from Andreas's [2021-09-07 post](https://sourceforge.net/p/mochad/discussion/1320002/thread/764dd1ce44/#76e9) on the flimsy grounds that the unit file looks more sophisticated. Compare it with [another version](https://github.com/ermshiperete/mochad/blob/master/systemd/mochad.service) by Eberhard Beilharz (ermshiperete).

Steve Porter provides a fork of the mmauka 0.0.17 version which he presents as [mochad-0.1.21](https://sourceforge.net/p/mochad/discussion/1320002/thread/9e758b6afc/7c52/attachment/mochad-0.1.21.tgz). It is a different solution to the compilation problem. See details [here](https://sourceforge.net/p/mochad/discussion/1320002/thread/9e758b6afc/). Casey Langen (clangen) has incorporated the changes by Steve Porter into another [mochad](https://github.com/clangen/mochad) GitHub repository.

More information about this fork in excruciating details at [Mochad on Recent Linux Distributions](https://sigmdel.ca/michel/ha/domoticz/mochad_on_recent_linux_distro_en.html).

**FR:** Il a plus de détails au sujet de cette fourche dans un billet intitulé [Mochad sur les distributions Linux récentes](https://sigmdel.ca/michel/ha/domoticz/mochad_on_recent_linux_distro_fr.html). 

## License

GNU General Public License version 3.0 (GPLv3) according to the [original project page on SourceForge](https://sourceforge.net/projects/mochad/).

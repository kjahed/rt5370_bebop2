# RT5370 wireless driver for the BEBOP2

## Prerequisite
The build process below was tested on a 32-bit Ubuntu 16.04 machine. Make sure package ``build-essential`` is installed:
```
apt-get update && apt-get install build-essential
```

##### BEBOP2 toolchain
Download the toolchain from https://github.com/Parrot-Developers/toolchains
```
wget https://raw.githubusercontent.com/Parrot-Developers/toolchains/master/parrot-tools-linuxgnutools-2012.03_1.0.0-1_i386.deb 
```
and install it using:
```
dpkg -i parrot-tools-linuxgnutools-2012.03_1.0.0-1_i386.deb
```
This will install the toolchain under ``/opt/arm-2012.03``

##### BEBOP2 Kernel
Clone the bepop2 kernel:
```
cd /opt
git clone https://github.com/Parrot-Developers/bebop2-opensource.git
cd bebop2-opensource/sources/linux-3.4.11/linux-3.4.11/
```
Edit the Makefile and change
```
EXTRAVERSION =
```
to
```
EXTRAVERSION = +
```
Copy the BEBOP2 Kernel configuration
```
cp ../linux.config .config
```

###### IMPORTANT
The Makefile for the MALI400 driver erroneously adds a leading slash to the drivers subdirectory path resulting in a compilation error. As a workaround, we can create a symbolic link /drivers to references the drivers subdirectory:
```
ln -s $(pwd drivers)/drivers /drivers
```
Moreover, the Kernel configuration has DMA-BUF activated which is incompatible with non-GPL licenses. To overcome this, patch the ``drivers/parrot/gpu/mali400/Kbuild`` to remove the license check. You can do this by applying the provided patch ``mali400-kbuild.patch``:
```
cp /path/to/driver/repo/mali400-kbuild.patch .
patch -p4< mali400-kbuild.patch
```

**Finally**, build the kernel
```
ARCH=arm CROSS_COMPILE=/opt/arm-2012.03/bin/arm-none-linux-gnueabi- make
```

## Build
To build the driver, simple run make
```
ARCH=arm make
```
On success the kernel module ``rt5370sta.ko`` will be placed in ``os/linux/``

## Installation
Simply copy ``rt5370sta.ko`` and ``RT2870STA.dat`` to the drone via FTP and then load the module:
```
mkdir -p /etc/Wireless/RT2870STA
mv /path/to/ftp/RT2870STA.dat /etc/Wireless/RT2870STA/
insmod /path/to/ftp/rt5370sta.ko
```
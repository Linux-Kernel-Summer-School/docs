# Linux Kernel Summer School (LKSS) 2025 edition cheatsheet

## Compiling the Linux kernel for the first time

```bash
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make imx93frdm_defconfig

ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j$(nproc)
```

## Subsequent kernel compilations

***IMPORTANT***: do **NOT** set the default configuration (i.e. ***imx93frdm_defconfig***)
kernel compilations. This will delete all of the configurations you've
selected manually using `menuconfig`.

```bash
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j$(nproc)
```

## Enabling a kernel module

```bash
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make menuconfig

# re-compile the kernel when you're done with the development
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j$(nproc)
```

## Installing the kernel modules in the rootfs

***IMPORTANT: YOU NEED TO DO THIS EVERY TIME YOU COMPILE THE KERNEL***

```bash
# run this inside /home/student/work/repos/lkss-utils/2025. rootfs.ext2 is also placed here
./rootfs_util modules_install rootfs.ext2 ~/work/repos/linux/
```

## Booting the board

```bash
# run this if you're not using the provided VM. You only need to run this once.
export PATH=$PATH:$(pwd)

# run this inside /home/student/work/repos/lkss-utils/2025
./boot_imx93.sh ~/work/repos/linux/arch/arm64/boot/Image ~/work/repos/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dtb rootfs.ext2
```

## Compiling the Linux kernel and booting the board

```bash
# run this inside /home/student/work/repos/linux
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j$(nproc)

cd /home/student/work/repos/lkss-utils/2025

./rootfs_util modules_install rootfs.ext2 ~/work/repos/linux/

./boot_imx93.sh ~/work/repos/linux/arch/arm64/boot/Image ~/work/repos/linux/arch/arm64/boot/dts/freescale/imx93-11x11-frdm.dtb rootfs.ext2
```

## Updating the Linux kernel repository

Run the following commands inside `/home/student/work/repos/linux`:

```bash
git fetch origin
git pull origin dev
```

## Updating the utilities repository

Run the following commands inside `/home/student/work/repos/lkss-utils`:

```bash
git fetch origin
git pull origin main
```

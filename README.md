# Firefly PX30

Specification of Rockchip Core-PX30-JD4  
+ CPU: quad-core Cortex-A35
+ GPU: dual-core Mali-G31
+ NPU: N/A

Host machine 使用 WSL + Ubuntu 18.05，Target board 預計使用 Ubuntu 18.05 做為 rootfs 減少交叉編譯問題。

## Setup build environment

Install dependencies

```bash
# 64-bit Linux system does not have the 32-bit architecture enabled
$ sudo dpkg --add-architecture i386
$ sudo apt update
$ sudo apt install \
    repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools \
    device-tree-compiler gcc-aarch64-linux-gnu mtools parted libudev-dev \
    libusb-1.0-0-dev python-linaro-image-tools linaro-image-tools autoconf \
    autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make binutils \
    build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip \
    rsync file bc wget libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev \
    libglade2-dev cvs git mercurial rsync openssh-client subversion asciidoc \
    w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo \
    liblz4-tool genext2fs lib32stdc++6 expect expect-dev fakeroot
```

## Download SDK

```bash
$ WORK_DIR=~/work/firefly_px30
$ mkdir -p $WORK_DIR; cd $WORK_DIR
# full SDK: px30_linux_release.xml, BSP: px30_linux_bsp_release.xml
$ repo init \
    --repo-url https://gitlab.com/firefly-linux/git-repo.git \
    -u https://gitlab.com/firefly-linux/manifests.git \
    -b master \
    -m px30_linux_release.xml

# sync
$ .repo/repo/repo sync -c --no-tags
$ .repo/repo/repo start firefly --all

# update
$ .repo/repo/repo sync -c --no-tags
```

固定使用 manifests 版本 6377120，以單獨 git repository 簡化管控。

移除非必要檔案，避免超過 Github 倉庫大小限制:  
+ buildroot/
+ prebuilts/gcc/linux-x86/arm/
+ prebuilts/gcc/linux-x86/aarch64/gcc-buildroot-9.3.0-2020.03-x86_64_aarch64-rockchip-linux-gnu/
+ Makefile
+ envsetup.sh

## Download Ubuntu rootfs

Download [ubuntu-base-18.04.5-base-arm64.tar.gz](https://cdimage.ubuntu.com/ubuntu-base/releases/18.04/release/)

```bash
$ wget https://cdimage.ubuntu.com/ubuntu-base/releases/18.04/release/ubuntu-base-18.04.5-base-arm64.tar.gz
$ tar -xf ubuntu-base-18.04.5-base-arm64.tar.gz
```

## Build from SDK

```bash
# Select the board configuration (MIPI 螢幕較容易取得)
#   px30-ubuntu.mk: Ubuntu, MIPI
#   px30-lvds-ubuntu.mk: Ubuntu, LVDS
$ ./build.sh px30-ubuntu.mk

# Build target: all|kernel|uboot|recovery|updateimg 
$ ./build.sh all

# confirm the links in rockdev/ and tools/linux/Linux_Pack_Firmware/rockdev/package-file
$ ./mkfirmware.sh

# clean build
$ ./build.sh cleanall
```


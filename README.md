Basic directory - /home/maxim/hkey/

**0. Toolchain && misc**  
Create folder for toolchain and enter it
```
mkdir toolchain && cd toolchain
```
Clone compile toolchain:
```
git clone https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9
```
Download and unpack gcc 7.1.1
```
wget https://releases.linaro.org/components/toolchain/binaries/7.1-2017.08/aarch64-linux-gnu/gcc-linaro-7.1.1-2017.08-x86_64_aarch64-linux-gnu.tar.xz
tar -xvf gcc-linaro-7.1.1-2017.08-x86_64_aarch64-linux-gnu.tar.xz
```
Packets:
```
sudo apt update;
sudo apt-get install git-core gnupg flex bison gperf build-essential zip \
curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev \
x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev \
libxml2-utils xsltproc unzip python-mako openjdk-8-jdk
```
**1. Bootloader**
```
sudo apt-get install uuid-dev build-essential
mkdir bootloader && cd bootloader
```
```
git clone https://github.com/96boards-hikey/tools-images-hikey970.git
git clone https://github.com/96boards-hikey/OpenPlatformPkg.git -b hikey970_v1.0
git clone https://github.com/96boards-hikey/arm-trusted-firmware.git -b hikey970_v1.0
git clone https://github.com/96boards-hikey/l-loader.git -b hikey970_v1.0
git clone https://github.com/96boards-hikey/edk2.git -b hikey970_v1.0
git clone https://github.com/96boards-hikey/uefi-tools.git -b hikey970_v1.0
```
```
cd edk2
ln -sf ../OpenPlatformPkg
```
Replace path to GCC 7.1.1
/home/maxim/hkey/toolchain/gcc-linaro-7.1.1-2017.08-x86_64_aarch64-linux-gnu/bin
```
vim /home/maxim/hkey/bootloader/l-loader/build_uefi.sh
```
Install openssl headers on ubuntu
```
sudo apt install libssl-dev
```

Compile:
```
l-loader/build_uefi.sh hikey970
```

**2. Kernel**
```
mkdir kernel && cd kernel
```
```
git clone https://github.com/96boards-hikey/linux.git -b hikey970-v4.9
cd linux
```
Setting environment variables:
```text
export ARCH=arm64
export CROSS_COMPILE=/home/maxim/hkey/toolchain/aarch64-linux-android-4.9/bin/aarch64-linux-android-
```
Making:
```
make hikey970_defconfig
make -j4
make hisilicon/kirin970-hikey970.dtb
```

**3. AOSP**
```
mkdir aosp && cd aosp
repo init -u https://android.googlesource.com/platform/manifest -b master
git clone https://github.com/96boards-hikey/android-manifest.git -b hikey970_v1.0 .repo/local_manifest
sudo repo sync
```
*Copy for compilation:*

kirin970-hikey970.dtb -> kirin970-hikey970.dtb-4.9
``` 
cp /home/maxim/hkey/kernel/linux/arch/arm64/boot/dts/hisilicon/kirin970-hikey970.dtb /home/maxim/hkey/aosp/device/linaro/hikey-kernel/kirin970-hikey.dtv-4.9
```
Image.gz -> Image.gz-hikey970-4.9
```
cp /home/maxim/hkey/kernel/linux/arch/arm64/boot/Image.gz /home/maxim/hkey/aosp/device/linaro/hikey-kernel/Image.gz-hikey970-4.9
```
Build:
```
source ./build/envsetup.sh
lunch hikey970-userdebug
make -j$(nproc)
```
*Generate **boot.img**:*

Copy Image and kirin970-hikey970.dtb from kernel directory
copy ramdisk.img from android out/
```
cat Image kirin970-hikey970.dtb > Image-dtb
mkbootimg --kernel Image-dtb --ramdisk ramdisk.img --cmdline "androidboot.hardware=hikey970 firmware_class.path=/system/etc/firmware loglevel=15 buildvariant=userdebug androidboot.selinux=permissive clk_ignore_unused=true initrd=0xBE19D000,0x16677F earlycon=pl011,0xfff32000,115200 console=ttyAMA6 androidboot.serialno=54DA9CD5022525E4 clk_ignore_unused=true" -o boot.img
```

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
```
vim ${BUILD_PATH}/l-loader/build_uefi.sh
```
Specific changes are shown below:

![Changes](https://pic3.zhimg.com/80/v2-06eb6b89334269d4f66d7c410a03a531_hd.jpg)

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
git checkout hikey970-v4.9
```
Compile chain download:
```
git clone https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9
```
Setting environment variables:
```text
export ARCH=arm64
export CROSS_COMPILE=/home/maxim/hikey970source/toolchain/aarch64-linux-android-4.9/bin/aarch64-linux-android-
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
```
```
repo init -u https://android.googlesource.com/platform/manifest -b master
```
```
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b master
```
```
git clone https://github.com/96boards-hikey/android-manifest.git -b hikey970_v1.0 .repo/local_manifest
```
```
sudo repo sync
```
*Compile:*

Copy kirin970-hikey970.dtb (arch/arm64/boot/dts/hisilicon/ kirin970-hikey970.dtb) to the device/linaro/hikey-kernel directory as file: kirin970-hikey970.dtb-4.9  

Copy the Image file (arch/arm64/boot/Image.gz-dtb) to the device/linaro/hikey-kernel directory as file: Image.gz-hikey970-4.9
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

## Build environment setup

Recommend build host is Ubuntu 16.04 64bit, for other hosts, refer official Android documents [Establishing a Build Environment](https://source.android.com/setup/build/initializing).


```shell
$ mkdir -p ~/bin
$ wget 'https://storage.googleapis.com/git-repo-downloads/repo' -P ~/bin
$ chmod +x ~/bin/repo
```

Android's source code primarily consists of Java, C++, and XML files. To compile the source code, you'll need to install OpenJDK 8, GNU C and C++ compilers, XML parsing libraries, ImageMagick, and several other related packages.


```shell
$ sudo apt-get update
$ sudo apt-get install openjdk-8-jdk android-tools-adb bc bison build-essential curl flex g++-multilib gcc-multilib gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc yasm zip zlib1g-dev
```

Configure the JAVA environment

```shell
$ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
$ export PATH=$JAVA_HOME/bin:$PATH
$ export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

## Download source code

```shell
$ mkdir rockpi4-android
$ cd rockpi4-android
```
Then run:

```shell
$ ~/bin/repo init -u https://github.com/radxa/rockpi4-android-tv-7.1.git -m rockchip_tv_nougat_release.xml
$ repo sync -j$(nproc) -c
```
It might take quite a bit of time to fetch the entire AOSP source code(around 86G)!

In China:
Download Repo
```shell
$ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
$ export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

## Build u-boot

```shell
$ cd u-boot
$ make rock-pi-4b-rk3399_defconfig
$ ./mk-uboot.sh
$ cd ..
```

The generated images are **rk3399_loader_v_xxx.bin** , **idbloader.img** and **uboot.img**

## Building kernel

```shell
$ cd kernel
$ make rockchip_defconfig
$ make rk3399-rockpi-4b.img -j$(nproc)
$ cd ..
```

The generated images are **kernel.img** and **resource.img**:

- kernel.img, kernel with rkcrc checksum
- resource.img, contains dtb and boot logo, Rockchip format resource package

## Building AOSP

```shell
$ source build/envsetup.sh
$ lunch rk3399_box-userdebug
$ make -j$(nproc)
```

It takes a long time, take a break and wait...

## Generate  images
```shell
$ ln -s RKTools/linux/Linux_Pack_Firmware/rockdev/ rockdev
$ ./mkimage.sh
```
The generated images under rockdev/Image are

    ├── boot.img
    ├── idbloader.img
    ├── kernel.img
    ├── MiniLoaderAll.bin
    ├── misc.img
    ├── parameter.txt
    ├── pcba_small_misc.img
    ├── pcba_whole_misc.img
    ├── recovery.img
    ├── resource.img
    ├── system.img
    ├── trust.img
    └── uboot.img

```bash
$ cd rockdev
$ ./android-gpt.sh
```
```
IMAGE_LENGTH:3936291
idbloader       64              16383           8.000000       MB
Warning: The resulting partition is not properly aligned for best performance.
uboot           16384           24575           4.000000       MB
trust           24576           32767           4.000000       MB
misc            32768           40959           4.000000       MB
resource        40960           73727           16.000000      MB
kernel          73728           122879          24.000000      MB
boot            122880          188415          32.000000      MB
recovery        188416          253951          32.000000      MB
backup          253952          483327          112.000000     MB
cache           483328          745471          128.000000     MB
system          745472          3891199         1536.000000    MB
metadata        3891200         3923967         16.000000      MB
baseparamer     3923968         3932159         4.000000       MB
userdata        3932160         3932159         0.000000       MB
```
The images under rockdev/Image are `gpt.img`

    ├── boot.img
    ├── gpt.img
    ├── idbloader.img
    ├── kernel.img
    ├── ......
    └── uboot.img

Installation
you can use `tf card` or `emmc module`
```bash
$ sudo umount /dev/<Your device>*
# mac os maybe not supprot progress
$ sudo dd if=Image/gpt.img of=/dev/<Your device> bs=4M status=progress
$ sync
```
through rockusb
```bash
# on device u-boot
# mmc 0 is your emmc module
# mmc 1 is your tf card
$ rockusb 0 mmc 1

# on pc
$ rkdeveloptool wl 0 Image/gpt.img
```
[More](https://wiki.radxa.com/Rockpi4/install)

**There may be some performance loss when using tf card**
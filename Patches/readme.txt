RK3588实时性系统环境搭建
支持平台：RK3562，RK356X，RK3588
一、PREEMPT_RT
  kernel-5.10-base:
  commit cae91899b67b031d95f9163fe1fda74fbe0d931a (tag: linux-5.10-stan-rkr1)
  Author: Lan Honglin <helin.lan@rock-chips.com>
  Date:   Wed Jun 7 15:01:26 2023 +0800
  ARM: configs: rockchip: rv1106 enable sc301iot for battery-ipc

  Signed-off-by: Lan Honglin <helin.lan@rock-chips.com>
  Change-Id: Ib844385bfd58f73eaa5f4e415d598d1f983fa4cd
  a). 在kernel-5.10直接打上三个补丁：
    0001-patch-5.10.180-rt89.patch-on-rockchip-base-cae91899b.patch
    0002-patch-5.10.180-rt89.patch-fix-runtime-error-on-rockc.patch
    0003-arm64-configs-optimize-latency-for-PREEMPT_RT.patch
  b).将arch/arm64/configs/rockchip_rt.config里的配置添加到arch/arm64/configs/rockchip_linux_defconfig中。
  c).编译命令：
    $ cd $sdk/kernel/
    $ export CROSS_COMPILE=../prebuilts/gcc/linux-x86/aarch64/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
    $ make ARCH=arm64 rockchip_linux_defconfig rk3588_linux.config
    $ make ARCH=arm64 rk3588-evb1-lp4-v10-linux.img -j8
d).烧录boot.img 并测试实时性性能
   使用cyclictest测试
   $ cyclictest -m -c 0 -t99 -t4
二、XENOMAI
  kernel-5.10-base:
  commit cae91899b67b031d95f9163fe1fda74fbe0d931a (tag: linux-5.10-stan-rkr1)
  Author: Lan Honglin <helin.lan@rock-chips.com>
  Date:   Wed Jun 7 15:01:26 2023 +0800
  ARM: configs: rockchip: rv1106 enable sc301iot for battery-ipc

  Signed-off-by: Lan Honglin <helin.lan@rock-chips.com>
  Change-Id: Ib844385bfd58f73eaa5f4e415d598d1f983fa4cd
  a).在kernel-5.10上打上dovetail补丁：
     dovetail-core-5.10.161-on-rockchip-base-cae91899b.patch
  b).buildroot打开XENOMAI配置，将补丁0001-xenomai-v3.2.x-on-rockchip.patch放置buildroot/package/xenomai目录，
    并编译rootfs.img
        BR2_PACKAGE_XENOMAI=y
        BR2_PACKAGE_XENOMAI_3_2=y
        BR2_PACKAGE_XENOMAI_VERSION="v3.2.2"
        BR2_PACKAGE_XENOMAI_COBALT=y
        BR2_PACKAGE_XENOMAI_NATIVE_SKIN=y
        BR2_PACKAGE_XENOMAI_TESTSUITE=y
        BR2_PACKAGE_XENOMAI_ANALOGY=y
        BR2_PACKAGE_XENOMAI_ADDITIONAL_CONF_OPTS="--enalbe-demo"
  c).把xenomai系统打到内核上：
    $ cd $sdk/kernel
    $ ../buildroot/output/rockchip_rk3588/build/xenomai-v3.2.2/scripts/prepare-kernel.sh --arch=arm64
  d).编译内核，并烧录boot.img  rootfs.img
  编译命令：
    $ cd $sdk/kernel
    $ export CROSS_COMPILE=prebuilts/gcc/linux-x86/arm/gcc-arm-8.3-2019.03-x86_64-arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
    $ make ARCH=arm64 rockchip_linux_defconfig rk3588_linux.config
    $ make ARCH=arm64 LT0=none LLVM=1 LLVM_IAS=1 rk3588-evb1-lp4-v10-linux.img -j17
  e)测试延时 
    (1).校准latency
        $ echo 0 > /proc/xenomai/latency 
    (2) 使用cyclictest测试
        $ ./usr/demo/cyclctest -m -n -c 0 -t99 -t4

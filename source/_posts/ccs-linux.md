---
title: ccs_linux
date: 2018-09-26 22:05:51
tags:
---

　　发现在linux底下写一些东西比在windows底下方便很多，而且windows的bash环境很难用，而ccs在制作启动镜像时需要用到比较多的命令，所以尝试在linux底下搭建开发环境，记录一下遇到的一些问题。
<!--more-->
        
#### 安装CCS及驱动
　　在官网上下载CCS8.1.0.00011_linux-x64.tar.gz和bios_mcsdk_02_01_02_06_setuplinux.bin。前者是一个压缩包，需要解压，解压后里面有和后者一样的二进制文件包，可以用来安装。
```
tar -xzvf CCS8.1.0.00011_linux-x64.tar.gz
cd CCS8.1.0.00011_linux-x64
sudo chmod 755 ccs_setup_linux64_8.1.0.00011.bin　　添加可执行权限
./ccs_setup_linux64_8.1.0.00011.bin
./bios_mcsdk_02_01_02_06_setuplinux.bin
```
　　正常情况下，上面的过程会开始ccs的安装过程，但是ccs需要32位运行环境和一些依赖所以会报错。opensuse发行版可以在yast >software management >pattern 找到32bit环境安装，其他的linux发行版可以尝试ia32。一般会有两个依赖：
```
sudo zypper in libusb-0_1-4
sudo zypper in libncurses5
```
　　安装完成后，需要执行驱动安装脚本，不然无法识别仿真器。
```
cd /home/ti/ccsv8/install_scripts/
sudo ./install_driver.sh
```
#### 安装bios_mcsdk
　　安装了bios_mcsdk后，打开ccs会提示检测到新文件，需要安装，此时选择安装会发现报错：
```
com.ti.biosmcsdk.pdk.C6678L.p2.feature.group [1.1.2.6] cannot be installed in this environment because its filter is not applicable.
```
　　这是因为这些包中的配置文件配置成了windows环境，需要将报错文件中的feature.xml和plugin.xml中的ws=win32删除。　
　　修改完成后如果直接打开ccs还是会有相同错误，因为ccs保存了一些相关的缓存，需要清除这些缓存
```
cd /home/ti/ccsv8/eclipse
./ccstudio –clean
```
　　还有就是ndk_2_21_01_38这个包是有问题的，安装时会报错：
```
An error occurred while collecting items to be installed
  session context was:(profile=epp.package.cpp, phase=org.eclipse.equinox.internal.p2.engine.phases.Collect, operand=, action=).
  Problems downloading artifact: osgi.bundle,com.ti.rtsc.NDK.product_2.21.1.38,2.21.1.38.
    File has invalid content:/tmp/signatureFile1216046131870858397.jar
      Invalid content:plugin.xml
      The file "plugin.xml" in the jar "/tmp/signatureFile1216046131870858397.jar" has been tampered!
```
　　需要在官网上下载其他较新的版本，我用的是ndk_2_25_01_11，可以正常安装。 　
#### 安装cutecom
　　linux底下的串口调试工具有minicom、cutecom等，之前用过minicom，能够正常使用，不过cutecom是有图形界面的，使用和配置起来要方便一些，于是这次装了cutecom。
```
sudo zypper in cutecom
```

### 补充
#### SYSBIOS版本
　　在新建sysbios工程的时候，发现原来bios_mcsdk_02_01_02_06_setuplinux.bin中的sysbios版本比较低，会报错：

```
None of the currently selected products supplies value for the Target. Please specify the Target manually, or try using a pre-3.30 version of
　　可以到官网下载最新版本。
```
#### LED_PLAY程序
需要使用到pdk包，因而需要添加路径：
File search path路径下：
library file：
> ti.platform.evm6678l.ae66

library search path:
> /home/minbud/ti/pdk_C6678_1_1_2_6/packages/ti/platform/evmc6678l/platform_lib/lib/debug

xdc path(build >xdctools >package repositories):
> /home/minbud/ti/pdk_C6678_1_1_2_6/packages
# HDMI声音异常操作步骤

## 1、打开SecureCRT的时间戳记录功能。

log file设置自动记录文件的名字。

d:\securecrt_log\[%S]-%M-%D-[%h-%m-%s].log

加时间戳：[%h:%m:%s:%t]

## 2、Linux这边要把日志设置到debug级别。

```
echo "all:LOG_DEBUG" > /tmp/AML_LOG_audioservice
echo "all:LOG_DEBUG" > /tmp/AML_LOG_homeapp
```

把上面两句命令放在开机脚本`vi /etc/init.d/S90audioservice`里

## 3、烧录不同版本的镜像文件并抓取log

log 最好按问题分类管理，以便溯源


# 测试AV400开机init时间和audio启动时间

## AV400烧录最新镜像

单独编译uboot 
```
make uboot-dirclean
make uboot-rebuild
```
单独烧录
```
adnl.exe partition -p bootloader -F u-boot.bin.signed

adnl Partition -P boot -F boot.img
```
重启
```
adnl.exe oem "reset"
```

## 关闭部分kernel 打印

在uboot上面执行下列命令

在AV400#下执行：
```
setenv initargs $initargs loglevel=0
saveenv
reset
```
## 重新上电

分析开机打印log，计算各模块启动时间


# a5 soundbar基本功能测试步骤

## HDMI1测试

asplay enable-input HDMI1

连接好HDMI1接口：

本地音乐用window media player打开

喇叭是否有声音
## 测试volume （soft/hardware)
```
asplay set-volume-type 70 1

asplay get-volume-type 0
```
70表示音量，1表示soft;    0表示hardware。

第二句命令表示查看当前soft/hardware的值


## 测试mute /unmute (soft/hardware)
```
asplay set-mute-type 1 1

asplay get-mute-type 1
```
set-mute-type:

第一位数字：1表示mute(静音)，0表示unmute(不静音)

第二位数字：1表示soft; 0表示hardware;

get-mute-type 后面的数字表示查看soft/hardware 是否mute。

## 命令方式播放Dolby

在PC端win+r输入cmd进入电脑命令窗口
```
C:\Users\hongmei.kuang>adb push D:\一些文档\libdolby_atmos.so /usr/lib
```
将杜比库push 到板端

sync 之后重启即可生效

**将杜比库推送到板端之后需要重启才能生效**

命令播放步骤

https://confluence.amlogic.com/pages/viewpage.action?pageId=203235062

atmos完整测试包

http://qa-sz.amlogic.com:8881/#/Streams/Test_Files/Certification/Dolby_DTS_Certification_SDK/Dolby

## 蓝牙

/etc/board_info 查看平台BT名称


# A5-release 操作步骤记录

需要申请开通openlinux 权限，提前发邮件给itsd。

## 一、获取代码

(这一步时间有点长，所以建议晚上下载)

步骤：

- 新建一个文件夹，获取代码
- repo init -u git://git.myamlogic.com/linux/manifest.git -b master -m buildroot-av400-sbr.xml --repo-url=git://git.myamlogic.com/tools/repo.git
- repo sync

## 二、更新代码

如果有修改的话，需要更新代码、

xml文件修改完成后，执行下面的命令来重新同步代码：

```c
repo init -m buildroot-av400-sbr.xml
repo sync
```
## 三、自测

整体编译，然后烧录到板子上测试基本功能。

自己测试完了之后，发邮件给QA测试。

一般仓库单独更新后，先make xx-dirclean在make xx-rebuild，然后整体make。

## 四、测试人员测试

## 五、固定版本

## 六、推送到openlinux

### 首先设置环境变量
```c
export code_dir=/home/hongmei.kuang/work/a113x2/code3/
export xml_file=buildroot-openlinux-202209-a113x2-av400-sbr.xml
export version=buildroot-openlinux-202209-a113x2-av400-sbr
```
然后分别执行脚本01-04

## 七、编写openlinux的xml文件

将上次的openlinux的xml文件拷贝过来，根据情况修改

# adb

查看板端进程，是如何实现的
vi S89usbgadget

## adbd

路径
./build/android-tools-4.2.2+git20130218/

# MRM 

允许多个支持alexa的设备同步播放音频

## 一、MRM库

MRM库是MRM功能的实现

- 管理与Alexa的通信
- 同步网络上的组成员之间的时间
- 从音乐服务提供商处检索和解码音频内容
- 管理组中主从设备之间的音频内容传播
- 将带有时间戳的音频样本缓冲区传递给组成员

主服务器负责维护与Alexa的连接，并通过本地网络将内容传播到组的成员。它还决定了何时应该跨组成员呈现同步的内容。从服务器负责连接到主服务器。主服务器由Alexa定义，主服务器/从服务器/Alexa交互由MRM库处理

**重要提示**：MRM库是为每个硬件目标定制编译的，并作为对象代码提供。

### 1.1 Delegate Classes and the MRM SDK

AVS客户端使用WHA类访问MRM客户端库。在使用WHA类之前，必须实现六个 delegate class

- CloudDelegate:启用在MRM库和AVS客户端之间交换指令和事件
  
- FocusDelegate:在MRM库和AVS客户端之间交换应用程序的焦点和回放状态信息
  
- DeviceInfoDelegate:提供设备信息，如IP地址。
  
- TimeDelegate:表面显示来自设备或操作系统的高分辨率计时器功能。该类还保存了在单元测试期间测量的相对延迟常数。
  
- PlaybackDelegate:从MRM客户端库接收带有时间戳的音频输出。的关键路径的当前延迟的音频管道通过混合器和音频驱动程序，然后使用延迟加上MRM同步时间发送样本到混合器的硬件在正确的时间，这样他们在同步。
  
- LoggingDelegate:捕获由MRM库产生的用于调试的日志记录消息。

### 1.2 使用集群工具包进行单元测试

设备制造商可以请求MRM客户端库的自定义构建，该库无需AVS客户端即可运行。它并不期待来自Alexa的指令，而是连接到一个位于特定网络位置的设备。他的定制构建需要两个定制的亚马逊回声点。当编译成提供的示例应用程序并根据说明在网络上进行配置时，运行此MRM客户端库构建的设备将参与具有提供的点的MRM组。

此配置对于测试音频管道实现的单元测试非常有用。它还允许在实现AVS客户端之前，早期测量设备和回声点之间的相对偏斜和漂移。这个倾斜测试的输出应该包含在TimeDelegate 的实现中，以设置默认的播放偏移量。

### 1.3 AVS设备SDK集成

根据要求，Amazon将提供一个能够与MRM客户端库交互的AVS Device SDK版本。这些版本实现了AVS设备SDK和CloudDelegate and FocusDelegate之间的接口。

使用AVS Device SDK的设备制造商只需要实现操作系统和设备特定的 Delegate Classes：

• DeviceInfoDelegate
• TimeDelegate
• PlaybackDelegate
• LoggingDelegate

## 二、Capabilities API

要使用MRM所需要的全家庭音频接口的1.0版本，必须在对Capabilities API的调用中声明它。

比如：
```json
"type": "AlexaInterface",
"interface": "WholeHomeAudio",
"version": "1.0"
```

# mrm PreTest

sdk-amlogic，目录

1、先修改envsetup.sh 

修改TARGET_PRODUCT=Amlogic_av400_platform，新建一个目录

将其他环境变量设置为我们现在的

然后source toolchain/raspberrypi0/envsetup.sh

2、make fetch 

下载一些东西

3.make

编译出我们测试要链接的库

MRM_preQualTests_Amlogic目录

在build.sh 里修改工具链正确的工具链

然后运行成功，编译出preQualTest 就完成编译了。


## 测试中用到的一些命令

```
set-ad82128-volume.sh 180  音频录制过程中板子音量设置

speaker-test -t sine -D hw:0,1   板端播放一个正弦波

```

```
配置gpio  T9输出高电平

1、pinmux。不用管。默认就是gpio。
2、对应bit写1
PADCTRL_GPIOT_O 0xfe004144

3fff
11 1111 1111 1111

&
gpiot_9的pinmux寄存器是这个。
PADCTRL_PIN_MUX_REGC 0xfe004030

PADCTRL_GPIOT_I 0xfe004140
	只读的。
PADCTRL_GPIOT_O 0xfe004144
	默认都是输出高电平。
PADCTRL_GPIOT_OEN 0xfe004148
	默认都是输出使能的。

PADCTRL_GPIOT_PULL_EN 0xfe00414c
	默认使能
PADCTRL_GPIOT_PULL_UP 0xfe004150
	默认上拉。
PADCTRL_GPIOT_LOCK 0xfe004154
PADCTRL_GPIOT_PROT 0xfe004158
PADCTRL_GPIOT_DS 0xfe00415c

echo none > /sys/kernel/config/usb_gadget/amlogic/UDC
echo fdd00000.crgudc2 > /sys/kernel/config/usb_gadget/amlogic/UDC

 set-ad82128-volume.sh 180  板子音量设置
```

spk设置音量
```
amixer controls
amixer cget numid=1
amixer cset numid=1 120
```

```bash
echo 0 > /proc/sys/kernel/printk 
关闭打印
echo 9 > /proc/sys/kernel/printk
开启打印
```

## 在buildroot里修改默认cpu频率

```c
 vim buildroot/board/amlogic/mesona5_av400/rootfs/etc/inittab  修改cpu频率为500M

null::sysinit:echo 500000 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq 把这句注释删掉
```

## 读写寄存器

```bash
#!/bin/sh

#mount -t debugfs none /sys/kernel/debug

#echo 0xfe004144 > /sys/kernel/debug/aml_reg/paddr;cat /sys/kernel/debug/aml_reg/paddr

#while true ; do 
#echo 0xfe004144  0x3fff > /sys/kernel/debug/aml_reg/paddr;cat /sys/kernel/debug/aml_reg/paddr
#echo 0xfe004144  0x3dff > /sys/kernel/debug/aml_reg/paddr;cat /sys/kernel/debug/aml_reg/paddr
#done 

# 写寄存器
#echo  0xfe330300  0xFE00011F > /sys/kernel/debug/aml_reg/paddr;cat /sys/kernel/debug/aml_reg/paddr
# 读寄存器
#echo 0xfe330300 > /sys/kernel/debug/aml_reg/paddr;cat /sys/kernel/debug/aml_reg/paddr

echo 470 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio470/direction

while true  ;do 
echo 0 > /sys/class/gpio/gpio470/value
sleep 0.5
echo 1 > /sys/class/gpio/gpio470/value
done
```

































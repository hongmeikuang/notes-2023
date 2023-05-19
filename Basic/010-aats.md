# 搭建 AATS 环境并手动测试单个用例

## 1 搭建AATS环境

1.先连接好板子，设置板子的adb id 

在板端进入kernel输入如下命令
```
keyman write usid str AMLA113X2AV400001
```
然后使用adb devices 命令查看是否设置成功
```
List of devices attached
AMLA113X2AV400001       device
```
显示这个则为成功
```
#locate aats_linux_cmd   // 查看本地aats命令工具

#ls /dev/AV400 tab键//查看USB软链接
```
## 手动测试

2.在workdir/workspace/Autotest/S420/Linux_Daily_SZSW_01/AATS_FW目录下输入命令

### 在AV400上通过串口单独测试Post_Test_Check模块
```
./aats_linux_cmd -p A113X2_AV400_a6432bit_DailyBuild.yml -m Post_Test_Check -s /dev/AV400_connector -d AMLA113X2AV400001 -U 921600
```
### 在AV400上通过串口单独测试Basic_Test模块的Basic_Test子模块
```
./aats_linux_cmd -p A113X2_AV400_a6432bit_DailyBuild.yml -m Basic_Test[.DSP_Test] -s /dev/AV400_connector -d 921600AMLA113X2AV400001 -U 921600
```
### 在AV400 上通过串口单独测试特定模块
```
./aats_linux_cmd -p A113X2_AV400_a6432bit_DailyBuild.yml -t Input_HDMI1 -s /dev/AV400_connector -d AMLA113X2AV400001 -U 921600
```
### 在每个命令中需要加上-U 921600 设置波特率，否则会乱码，导致测试结果失败

### 查看串口打印信息
```
minicom -D /dev/AV400_connector -b 921600 

echo none > /sys/kernel/config/usb_gadget/amlogic/UDC
echo fdd00000.crgudc2 > /sys/kernel/config/usb_gadget/amlogic/UDC
```
### 关闭打印
```
CTRL+A 后立即按q 

#av400_on 手动上电

#adb devices  查看设备是否上电
```
## aats 测试脚本.yml

每个空格都有特殊含义，不能用tab键代替空格；

console表示在pc端输入命令，dut表示在板端输入命令；

## AV400_connector

lsusb 查看

connect的串口都是CP210X的，继电器的是CH341的。

ls /dev/AV400_connector查看串口是否正常。

串口板一一对应。

# 重新搭建一套测试环境

1、先添加新板子的信息，并找方岩申请添加

```
https://confluence.amlogic.com/display/SW/AutoTest+Platform+Register
```

2、将adb id 写到板子上

进入uboot 命令行，执行以下命令

```
keyman write usid str AMLA113X2AV400001
```

用adb devices命令可以查看

3、修改usb 设备的名字

  1）、插拔USB设备的同时，用ls /dev/ttyUSB* 命令查看，增加或者减少了哪个设备，即为对应的USB设备。
  2）、将对应的设备生成软链接，比如

```
ln -s /dev/ttyUSB2 /dev/BA400_connector
ln -s /dev/ttyUSB3 /dev/BA400_powerRelay
```
4、修改/etc/udev/rules.d/usb.rules 文件

用udevadm info /dev/ttyUSB1命令

```
amlogic@amlogic-BAD-INDEX:~/work2/test$ udevadm info /dev/ttyUSB1
P: /devices/pci0000:00/0000:00:14.0/usb1/1-2/1-2.3/1-2.3.1/1-2.3.1:1.0/ttyUSB1/tty/ttyUSB1
N: ttyUSB1
S: AV400_powerRelay
S: serial/by-id/usb-1a86_USB_Serial-if00-port0
S: serial/by-path/pci-0000:00:14.0-usb-0:2.3.1:1.0-port0
E: DEVLINKS=/dev/serial/by-path/pci-0000:00:14.0-usb-0:2.3.1:1.0-port0 /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0 /dev/AV400_powerRelay
E: DEVNAME=/dev/ttyUSB1
E: DEVPATH=/devices/pci0000:00/0000:00:14.0/usb1/1-2/1-2.3/1-2.3.1/1-2.3.1:1.0/ttyUSB1/tty/ttyUSB1
E: ID_BUS=usb
E: ID_MM_CANDIDATE=1
E: ID_MODEL=USB_Serial
E: ID_MODEL_ENC=USB\x20Serial
E: ID_MODEL_FROM_DATABASE=HL-340 USB-Serial adapter
E: ID_MODEL_ID=7523
E: ID_PATH=pci-0000:00:14.0-usb-0:2.3.1:1.0
E: ID_PATH_TAG=pci-0000_00_14_0-usb-0_2_3_1_1_0
E: ID_PCI_CLASS_FROM_DATABASE=Serial bus controller
E: ID_PCI_INTERFACE_FROM_DATABASE=XHCI
E: ID_PCI_SUBCLASS_FROM_DATABASE=USB controller
E: ID_REVISION=0264
E: ID_SERIAL=1a86_USB_Serial
E: ID_TYPE=generic
E: ID_USB_CLASS_FROM_DATABASE=Vendor Specific Class
E: ID_USB_DRIVER=ch341
E: ID_USB_INTERFACES=:ff0102:
E: ID_USB_INTERFACE_NUM=00
E: ID_VENDOR=1a86
E: ID_VENDOR_ENC=1a86
E: ID_VENDOR_FROM_DATABASE=QinHeng Electronics
E: ID_VENDOR_ID=1a86
E: MAJOR=188
E: MINOR=1
E: SUBSYSTEM=tty
E: TAGS=:systemd:
E: USEC_INITIALIZED=255589300
```

然后将对应的规则写进/etc/udev/rules.d/usb.rules文件中

```
SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.1.1:1.0", MODE:="0666", SYMLINK+="BA400_powerRelay"
SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.1.3:1.0", MODE:="0666", SYMLINK+="BA400_connector"

SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.2.1:1.0", MODE:="0666", SYMLINK+="T404_powerRelay"
SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.2.3:1.0", MODE:="0666", SYMLINK+="T404_connector"

SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.3.1:1.0", MODE:="0666", SYMLINK+="AV400_powerRelay"
SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.3.3:1.0", MODE:="0666", SYMLINK+="AV400_connector"

SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.4.1:1.0", MODE:="0666", SYMLINK+="S400_sbr_powerRelay"
SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:14.0-usb-0:2.4.3:1.0", MODE:="0666", SYMLINK+="S400_sbr_connector"

SUBSYSTEM=="usb", ATTR{idVendor}=="1b8e", MODE:="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTR{idVendor}=="10c4",ATTR{idProduct}=="ea60", MODE:="0666", GROUP="plugdev", OWNER="amlogic"
```

修改后，执行命令：

```
sudo udevadm trigger
```

av400_on/off的命令是在~/.bash_profile文件中写的

```
 export PATH=$PATH:/home/amlogic/tools/bitbake/bin
 export PYTHONPATH=/home/amlogic/tools/bitbake/lib
 export PATH=$PATH:/home/amlogic/platform-tools
 
 alias ba400_off='/home/amlogic/workdir/workspace/Autotest/AV400/Linux_Daily_SZ_01/AutoFramework/bin/powerRelay /dev/BA400_powerRelay all off'
 alias ba400_on='/home/amlogic/workdir/workspace/Autotest/AV400/Linux_Daily_SZ_01/AutoFramework/bin/powerRelay /dev/BA400_powerRelay all on'
 
 alias t404_on='/home/amlogic/workdir/workspace/Autotest/S420/Linux_Daily_SZSW_01/AutoFramework/bin/powerRelay /dev/T404_powerRelay all on'
 alias t404_off='/home/amlogic/workdir/workspace/Autotest/S420/Linux_Daily_SZSW_01/AutoFramework/bin/powerRelay /dev/T404_powerRelay all off'
 alias sbr_on='/home/amlogic/workdir/workspace/Autotest/S420/Linux_Daily_SZSW_01/AutoFramework/bin/powerRelay /dev/S400_sbr_powerRelay all on'
 alias sbr_off='/home/amlogic/workdir/workspace/Autotest/S420/Linux_Daily_SZSW_01/AutoFramework/bin/powerRelay /dev/S400_sbr_powerRelay all off'
 
 alias av400_on='/home/amlogic/workdir/workspace/Autotest/AV400/Linux_Daily_SZ_01/AutoFramework/bin/powerRelay /dev/AV400_powerRelay all on'
 alias av400_off='/home/amlogic/workdir/workspace/Autotest/AV400/Linux_Daily_SZ_01/AutoFramework/bin/powerRelay /dev/AV400_powerRelay all off'
 
 export NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node
 source ~/.nvm/nvm.sh
 
 git config --global http.proxy http://10.78.20.250:3128
 git config --global https.proxy https://10.78.20.250:3128
 export http_proxy=http://10.78.20.250:3128
 export https_proxy=http://10.78.20.250:3128
 export PATH=/home/amlogic/tools/go/bin:$PATH
 
 echo "this is .bash_profile"
 . "$HOME/.cargo/env"
```

修改后保存退出即可生效。



## 参考资料

先在Confluence上找有用的资料

https://confluence.amlogic.com/display/SW/AATS+FW+User+Guide

操作步骤文档
https://confluence.amlogic.com/display/SW/AATS+CI+Developer+Tutorial

https://confluence.amlogic.com/pages/viewpage.action?pageId=188615774
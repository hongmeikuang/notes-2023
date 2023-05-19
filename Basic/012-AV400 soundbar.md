# AV400 soundbar

基于A113X2 Soc开发的高端演示套件

# 一、套件介绍

## 特点

### 1、多输入接口HDMI  

输入接口：

HDMI                 2

3.5mm耳机口输入       1

同轴线SPDIF输入       1

AIRPLAY源输入         1

蓝牙输入       U盘输入          AVS音源输入。

输出接口：

HDMI输出接口（该接口可以支持HDMIARC/eARC功能） 1个

### 2、多声道输出支持。最多支持16声道音频输出。

### 3、Dolby Atmos Home Audio 1.7.2支持。包括如下格式：DD/DDP/TrueHD/MAT。

## 组成

从上到下依次为

D604麦克风板

AV400核心板

D622 HDMI Repeater板

### AV400核心板

AV400 系统配置

主控：A113X2，64位Cortex A55 4核 SoC

内存：LPDDR4 1GB

Flash： eMMC 4GB

### D622 HDMI Repeater板

D622作为输入接口板和功放驱动板，其作用为：

    接收HDMI/LINEIN/SPDIF等输入信号，传递给AV400核心板处理
    
    接收AV400核心板的输出信号，驱动连接的喇叭发声。

必须连接1号喇叭，才能听见单双声道的声音

### D604麦克风板

麦克风板上有7个数字麦克风，通过排线最终连接到A113X2的PDM接口上。

# 二、套件烧录及运行

### 所需材料

pc 一台    套件 一套     串口板    两个12V电源

usb 线两根

一根用于串口板，一根用于pc 和套件为了使用adb 命令

### 串口设置

波特率设置为 921600

### 上电

连接套件的电源接电，开关为on

### 2.1烧录方法

烧录前需要保证接线完整

进入烧录模式的方式有三种

- 如果AV400没有烧录过镜像，则系统自动进入烧录模式。

- 先将系统上电，按住USB boot键不放，然后按一下RST Key，随后放开USB boot键，则
   系统进入烧录模式。

- 先将系统上电，然后在PC端串口界面按住PC键盘的Enter键不放，打断板端u-boot的自
  动引导过程，然后在u-boot的console里输入 adnl 命令，系统进入到烧录模式。


串口如下显示，即为进入烧录模式
```
a5_av400# adnl
PHY2=00000000fe03a000,phy-base=0xfe03c000
0x10 trim value=0x0000007f
crg cn
```
ps:如果已经进入kernel,则输入reboot 命令重启，然后在PC端串口界面按住PC键盘的Enter键不放，打断板端u-boot的自动引导过程，然后在u-boot的console里输入 adnl 命令，系统进入到烧录模式。

进入烧录模式，则会在烧录工具界面显示设备

设置->导入镜像->开始->勾选烧录完成后重启选项->烧录完成后停止

### 单独烧录

板端进入烧录模式

PC在要烧录文件存在的目录输入命令

#### 烧录bootloader 

```
adnl.exe partition -p bootloader -F u-boot.bin.signed
```
#### 烧录boot.img

```
adnl.exe Partition -P boot -F boot.img
```

#### 烧录设备树dts

```
adnl.exe Partition -P _aml_dtb -F a5_a113x2_av400_1g.dtb
```

烧录之后均需对板端重新上电

```
adnl.exe oem "reset"
```
重启

### 2.2运行

镜像烧录完成后，系统上电后自动运行

### 2.3调试

系统运行日志从AV400的串口输出。串口被A113X2内部集成的CPU、HiFi DSP、AOCPU这3个系统共同使用。串口打印较多，不便于进行命令输入操作。

如果需要进行命令输入、文件编辑等操作，建议通过adb来进行。
板端系统默认启动了adbd进程。将PC跟AV400用USB线连接后即可使用adb命令进行操作。

常见adb命令：
```
adb shell:打开一个adb 会话

adb push  1.txt/data:把PC当前目录下的1.txt文件推送到板端的/data目录下

adb pull /data/1.txt:把板端的/data/1.txt文件拉取到PC当前目录下
```
# 三、网络设置方法

套件主要使用WiFi的方式进行网络连接。

有三种方式可以设置系统的网络信息：手动配置、SoftAP、BLE。

推荐使用手动配置或者BLE方式来进行配置

一般使用手动配置的方式

## 手动配置网络

### 方法1：

1、打开电脑移动网络

2、在板端输入如下命令：

```
wpa_cli -iwlan0 remove_network 0
wpa_cli -iwlan0 add_network 0
wpa_cli -iwlan0 set_network 0 ssid '"hongmei"' # change your ssid here
wpa_cli -iwlan0 set_network 0 key_mgmt WPA-PSK
wpa_cli -iwlan0 set_network 0 psk '"88889999"' # change your password here
wpa_cli -iwlan0 set_network 0 pairwise CCMP
wpa_cli -iwlan0 set_network 0 group CCMP
wpa_cli -iwlan0 set_network 0 proto RSN
wpa_cli -iwlan0 enable_network 0
wpa_cli -iwlan0 status
wpa_cli -iwlan0 save
dhcpcd wlan0
```
3、在板端输入ifconfig查看wlan0是否获得Ip地址。

### 方法2：

系统启动后会自动加载wifi驱动，通过ifconfig检查是否存在wlan0。若存在，则可通过以下命令配置网络：

1. 在 /etc/wpa_supplicant.conf 的最后加入下面的内容，ssid和psk的值修改为正确的AP的名字和密码。
```
network={
ssid="hongmei"
psk="88889999"
}
```
2. 板端执行命令：
```
wpa_cli reconfigure
```
3. 然后板端就会尝试去连接AP，等待连接完成即可，可以用下面的命令查看连接的情况。
```
wpa_cli status
```
# 四、AV400功能体验

## soudbar软件说明

软件名                    作用
```
/etc/init.d/S90audioservice：开机脚本，启动audioservice和homeapp这2个进程

modprobe snd-aloop  //加载模块驱动

audioservice：Soundbar核心进程，处理音频数据并进行输出

homeapp ：Soundbar界面进程，处理输入控制和信息显示

asplay：跟homeapp作用类似，在命令行上对audioservice进行控制
```
## asplay常用命令

命令 作用
```
asplay 查看命令的帮助信息

asplay list 查看当前的输入源

asplay set-volume 60 设置音量为60%

asplay get-volume 查看当前音量

asplay enable-input XX 切换到某个输入源，XX为某个实际的输入模式，例如HDMI1
```
# 五、测试各种输入模式

## 5.1 HDMI1输入测试

1. 首先确保当前是HDMI1输入模式。可以用 asplay list 命令查看确认。

HDMI1(true)表示当前是HDMI1模式

2. 用HDMI线将DVD或者其他支持HDMI音视频输出的设备（例如PC）跟D622的HDMI Input0端口连
接起来。

3. DVD播放声音，就可以听到声音从D622上连接的喇叭输出。


# 六、Soundbar SDK subsystem开发

http://10.28.8.24:8080/a113x2/doc_preview/Soundbar%20SDK%20Integration%20Guide%20%280.1%29_CN.pdf

## 6.1 buildroot 开发

见buildroot学习笔记

## 6.2 bootloader开发

bootloader目录结构

```

    ├── bl2
    │ └── bin
    ├── bl30
    │ ├── bin
    │ └── rtos_sdk
    ├── bl31_1.3
    │ └── bin
    ├── bl32_3.8
    │ └── bin
    ├── bl33
    │ └── v2019
    ├── bl40
    │ └── bin
    └── fip
```

**注**：bootloader编译得到的最终文件是u-boot.bin.signed，该文件生成在
`$SDK/output/xxx_release/images` 目录下

### BL30 开发

bl30位于` $SDK/bootloader/uboot-repo/bl30/rtos_sdk `目录下。

bl30在整个系统中的作用：是运行在A113X2内部集成的RISCV核心上

RISCV核心：被称为AOCPU,AO是aways on的意思。

AOCPU主要对系统进行suspend/resume功能的管理。

入口文件是`$SDK/bootloader/uboot-repo/bl30/rtos_sdk/products/aocpu/main.c `。

### uboot 开发

uboot位于 $SDK/bootloader/uboot-repo/bl33 下，该uboot基于U-boot v2019.01开发。

uboot使用的编译器为 aarch64-elf-gcc ,位于`/opt/gcc-linaro-7.3.1-2018.05-i686_aarch64-elf/bin`下，由`$SDK/bootloader/ubootrepo/bl33/v2019/Makefile`文件中`CROSS_COMPILE`指定。

    CROSS_COMPILE ?=/opt/gcc-linaro-7.3.1-2018.05-i686_aarch64-elf/bin/aarch64-elf-

### 编译

    make uboot-dirclean
    make uboot-rebuild

编译完成后生成的最终文件u-boot.bin.signed在output/.../image 目录下面

## 6.3kernel 开发

内核源码位于SDK根目录下`kernel/aml-5.4`目录。

### dts 修改及添加

Amlogic 64位内核位于`kernel/aml-5.4/arch/arm64/boot/dts/amlogic`目录;
32位内核dts位于`kernel/aml-5.4/arch/arm/boot/dts/amlogic`目录;

AV400 相关DTS包括

$SDK/kernel/aml-5.4/arch/arm64/boot/dts/amlogic/meson-a5.dtsi

$SDK/kernel/aml-5.4/arch/arm64/boot/dts/amlogic/a5_a113x2_av400_1g.dts

`meson-a5.dtsi`为芯片级配置，主要包含SoC内部模块的默认配置。

`a5_a113x2_av400_1g.dts`为板级配置，主要包括对SoC默认配置的调整，以及添加对板级设备的支持。

### 单独烧录dts

```
 adnl Partition -P _aml_dtb -F /d/save-info/MRM/a5_a113x2_av400_1g.dtb
 adnl reboot
```

### kernel 配置修改

kernel的配置文件是通过`buildroot config`里的`BR2_LINUX_KERNEL_DEFCONFIG`变量配置的。


例如，AV400的`buildroot/configs/amlogic/a5_av400_spk.config`的头文件`a5_speaker.config`的头文件->`a5_base.config`里的

BR2_LINUX_KERNEL_DEFCONFIG="meson64_a64_smarthome"

对应的是 $SDK/kernel/aml-5.4/arch/arm64/configs/meson64_a64_smarthome_defconfig 文件。

如果需要调整kernel的配置，就在这个文件里进行修改。

### kernel编译

编译命令

make linux-rebuild

如果修改了kernel 的config文件，则执行：

    make linux-dirclean 
    make linux-rebuild

然后整体make编译，得到boot.img文件及整机烧录镜像
### 单独烧录boot.img
根据需要烧录boot.img或者整机镜像
```
adnl.exe Partition -P boot -F /c/Users/hongmei.kuang/Desktop/boot.img
adnl reboot
```

最终生成的boot.img文件在output/.../image目录下

## 6.4 HiFi DSP　开发

*只适用于A113X2，A113XD没有DSP模块*

DSP在SDK中的代码位置是：`$SDK/vendor/amlogic/rtos/HiFiDSP_rtos_sdk`。

修改代码后的HiFi DSP编译命令：

    make aml-hifi-rtos-sdk-dirclean
    make aml-hifi-rtos-sdk-rebuild

编译后生成的最终文件是`$SDK/output/a5_av400_a6432_release/images/dspbootA.bin`。

然后用adb把该文件推送到板端的/lib/firmware目录下

推送命令为：

    adb push dspbootA.bin /lib/firmware
    adb shell sync

然后重启板子即可生效

DSP RTOS系统的启动是在板端Linux的 /etc/init.d/S71_load_dspa 脚本中

    # load dsp bin
    dsp_util --load --dsp hifi4a -f dspbootA.bin
    # start dsp
    dsp_util -S --dsp hifi4a

## 6.5 NPU 开发

## 6.6 Audio 开发

跟Audio相关的kernel代码有

$SDK/kernel/aml-5.4/sound/soc/amlogic    SoC侧的audio相关驱动，包括
TDM/PDM/SPDIF/VAD等功能。

$SDK/kernel/aml-5.4/sound/soc/codecs/amlogic/       Amlogic集成的codec侧驱动。

$SDK/kernel/aml5.4/arch/arm64/boot/dts/amlogic/a5_a113x2_av400_1g.dts     设备树文件。

$SDK/kernel/aml-5.4/sound/soc/amlogic 和 $SDK/kernel/aml5.4/sound/soc/codecs/amlogic/ ，一般不需要调整。

用户的修改集中在设备树文件 $SDK/kernel/aml5.4/arch/arm64/boot/dts/amlogic/a5_a113x2_av400_1g.dts 。

### 设备树配置

#### DAI 配置

auge_sound是系统声卡对应的设备树节点，节点的具体属性值可参考https://doc.amlogic.com/file/detail?type=1&id=18413

设备树节点信息增删改查均在`$SDK/kernel/aml5.4/arch/arm64/boot/dts/amlogic/a5_a113x2_av400_1g.dts`文件

#### codec配置


### Audio input 

    #arecord -l

 命令可以用来查看所有录音设备

### Audio output

    #aplay -l 

命令用来查看所有音频播放设备

### 调节音量

在调试阶段，用amixer 命令来调节音量

    #amixer controls |grep ad82128|grep Volume|grep Ch

 该命令用来查看系统里音量控件

Soundbar系统里的codec芯片一般较多，用amixer命令逐个调节音量不方便。
系统里带了/usr/bin/set-ad82128-volume.sh脚本，可以一次性调节所有的AD82128芯片的音量。

使用

    #value range: 0-255
    set-ad82128-volume.sh 200

## 6.7 WIFI/BT

WiFi对应有两个网口，wlan0和wlan1，wlan0作为station模式，wlan1作为ap模式。

*注意WiFi模组需要连接天线，否则不能保证无线信号的连接稳定性。*

### 6.7.1 

#### AP 模式：开发板能够开启热点被手机连接上；

wifi ssid默认ssid格式为‘amlogic-audio-’+"特定后缀"，执行 `cat /etc/wifi/ap_name`可获取后缀;wifipsk默认为12345678，热点默认地址为192.168.2.1。

#### station模式：开发板能够主动连接指定的热点。

系统启动后会自动加载wifi驱动，通过ifconfig检查是否存在wlan0。若存在，则可通过以下命令配置网络：

1. 在 /etc/wpa_supplicant.conf 的最后加入下面的内容，ssid和psk的值修改为正确的AP的名字和密码。
```json
 network={
 ssid="your_ssid"
 psk="your_password"
 }
```
2. 板端执行命令：
```c
 wpa_cli reconfigure
```
3. 然后板端就会尝试去连接AP，等待连接完成即可，可以用下面的命令查看连接的情况。
```c
wpa_cli status
```
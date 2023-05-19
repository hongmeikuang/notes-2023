# audioservice代码理解

1、audioservice代码分为audioservice和homeapp两部分，使用DBus通信协议，协议描述位于vender/amlogic/audioservice/src/aml.linux.dbus.xml文件中。

2、audioservice 作为DBus server,homeapp和asplay都是DBus Client.

在命令行输入asplay命令时，比如asplay showversion,调用的是audioservice/src/asplay.c中的func_showversion函数，在这个函数中又调用了as_client.c中的AS_Client_GetVersion函数从dbus_xml文件中解析出version值。其他命令实现类似。asplay list，asplay enable-input HDMI1，
asplay set-volume 50等等。

3、homeapp种中包含各种client.c文件，这些文件均是为了实现与相关功能的通信，控制其音量大小。
比如avs_client.c实现了与avs-sdk的通信，控制avs的音量和播放状态。

这些player client 均由input_manager.c统一进行管理。
player client定义一个 InputHandler_t 结构体变量，并对该结构体变量的函数成员进行实现。
InputHandler_t 结构体定义在 homeapp/input_manage.h 中：
```
typedef struct _tag_input_handler {
  const char *input_name;   //client 的名字

  int (*Init)(void);    //client 初始化函数，进行资源的申请
  int (*Exit)(void);    //client 的退出函数，进行资源的释放
  int (*KeyHandler)(int key, SYSKEY_Event key_event);    //client 对按键的处理
  int (*ASCallback_handler)(AML_AS_NOTIFYID_e type, ASClientNotifyParam_t *param);  处理audioservice发出的system-update的通知事件。
  int (*GetCmd)(InputMgrCmd_e cmd, InputMgrGetParam_u *param);  处理InputMgrCmd_e类型的GET命令。
  int (*SetCmd)(InputMgrCmd_e cmd, InputMgrSetParam_u *param);  处理InputMgrCmd_e类型的SET命令。

  struct _tag_input_handler *next;  所有的client通过next指针组成单链表。
} InputHandler_t;
```
ASCallback_handler是用来处理audioservice的状态变化通知的，通知对应的枚举类型是
AML_AS_NOTIFYID_e 。

# audioservice 中宏是如何传递的

## 以homeapp 中AV400_SBR_ENABLE为例

1 首先在buildroot\package\amlogic\audioservice\audioservice.mk文件里面，
下列语句使能了--enable-av400sbr

```c
ifeq ($(BR2_PACKAGE_AUDIOSERVICE_AV400_SBR),y)
AUDIOSERVICE_CONF_OPTS += --enable-av400sbr
endif
```

2 在vendor\amlogic\audioservice\configure.ac文件里,下列语句使能了AV400SBR_ENABLE

```c
AC_ARG_ENABLE([av400sbr],
[  --enable-av400sbr    Enable av400sbr support in audioservice],
[case "${enableval}" in
	yes) av400sbr=true ;;
	no)  av400sbr=false ;;
	*) AC_MSG_ERROR([bad value ${enableval} for --enable-av400sbr]) ;;
esac],[av400sbr=false])
AM_CONDITIONAL([AV400SBR_ENABLE], [test x$av400sbr = xtrue])
```

3 在vendor\amlogic\audioservice\homeapp\Makefile.am文件里，使能AV400_SBR_ENABLE

```c
if AV400SBR_ENABLE
AM_CFLAGS += -DAV400_SBR_ENABLE -DDISPLAY_LCD_ENABLE -DDISPLAY_PRINT_ENABLE \
				-DDISPLAY_SPEAK_ENABLE
homeapp_SOURCES += display_module/display.c \
	 display_module/print/display_print.c \
	 display_module/lcd/display_lcd.c \
	 display_module/speak/display_speak.c
endif
```

4 这就是AV400_SBR_ENABLE宏的传递过程。

# 一、soundbar system 

http://10.28.8.24:8080/a113x2/doc_preview/Soundbar%20SDK%20Integration%20Guide%20%280.1%29_CN.pdf

## 涉及的软件、脚本以及库文件有这些
-----------------------------------------------
- 板端：/etc/init.d/S90audioservice 

- PC目录： $SDK/vendor/amlogic/audioservice/script/S90audioservice

- 描述：开机脚本，开机时启动audioservice和homeapp。
------------------------------------------------
- 板端：/usr/bin/audioservice 

- PC目录：$SDK/vendor/amlogic/audioservice/src/

- 描述：Soundbar的核心应用。其作用为：

    1.作为D-Bus Server，接收处理homeapp和asplay发出的D-Bus
    消息。
    2.与D622 HDMI Repeater板进行I2C通信，控制D622的输入模
    式。
    3.调用libhalaudio.so里的解码接口，进行Dolby解码操作。
    4.实现DataPlayer。
---------------------------------------------------
- 板端：/etc/default_audioservice.conf

- PC目录：$SDK/vendor/amlogic/audioservice/src/config/br

- 描述：audioservice配置文件。编译时根据
BR2_PACKAGE_AUDIOSERVICE_CONFIG_FILE变量值使用对应
的文件。
--------------------------------------------------

- 板端：/usr/bin/homeapp 

- PC目录：$SDK/vendor/amlogic/audioservice/homeapp/ 

- 描述：Soundbar的UI应用，接收用户的按键、遥控输入，显示系统信
息，实现AVS/AIRPLAY/USB等播放输入模式。
-----------------------------------------------------
- 板端：/usr/lib/libhalaudio.so

- PC目录：$SDK/multimedia/aml_halaudio

- 描述：音频解码库。
--------------------------------------------------------


## audioservice 进程

audioservice.c 作为server端接收 asplay的请求并回复 在`GBusAcquired_Callback`安装回调函数 如：on_handle_get_input_list 的实例；

开启或关闭audioservice 进程： 

        /etc/init.d/S90audioservice stop
        /etc/init.d/S90audioservice start 

## halaudio audioservice 编译：
```
make aml-halaudio-rebuild

make audioservice-rebuild 
```
## homeapp

homeapp是客户端应用程序，它使用 as_client API 来访问音频服务。Control LED 显示音频信息、音量值和输入等。Homeapp具有输入管理机制，用于处理 HDMI、LINEIN、ARC、GVA/C4A、AirPlay、BT、AVS、USB 播放器等之间的切换。Homeapp具有分发远程密钥以纠正 AppClient 的机制。源代码文件位于audioservice/homeapp中。
## HDMI repeater 调试

        i2cdump -f -y 2 0x40

第3，4位下面是版本号

比如，avs_client.c主要与AVS进程进行交互，控制AVS的音量和播放状态。在avs_client.c 中包含AvsClient.h头文件（这个文件在AVS-SDK/Client/目录下），然后实现对avs的音量和播放状态的控制。

# 二、声道切换配置

当前系统默认是使用7.1.4声道的配置。

7.1.4声道和5.1.2声道的不同在于2个配置文件：

板端文件：

        /etc/default_audioservice.conf
        /etc/dap_tuning_files.xml

7.1.4对应文件

        $SDK/vendor/amlogic/audioservice/src/config/br/av400_sbr_7_1_4_datmos_V1_7.conf
        $SDK/vendor/amlogic/audioservice/src/config/br/dap_xml/dap_tuning_files_7_1_4.xml

5.1.2对应文件

        $SDK/vendor/amlogic/audioservice/src/config/br/av400_sbr_5_1_2_datmos_V1_7.conf
        $SDK/vendor/amlogic/audioservice/src/config/br/dap_xml/dap_tuning_files_5_1_2.xml

### 7.1.4声道切换为5.1.2声道配置，操作步骤为：

1.将 $SDK/buildroot/configs/amlogic/a5_av400.config 中
`BR2_PACKAGE_AUDIOSERVICE_CONFIG_FILE="br/av400_sbr_7_1_4_datmos_V1_7.conf"`配置
项修改为
        BR2_PACKAGE_AUDIOSERVICE_CONFIG_FILE="br/av400_sbr_5_1_2_datmos_V1_7.conf"

2.重新编译生成镜像。因为修改了buildroot的配置项，所以需要执行一次 sourse setenv.sh 来保证配置修改生效。

        sourse setenv.sh a5
        make audioservice-dirclean && make audioservice-rebuild && make

3.重新烧录镜像

# 三、适配codec芯片

当前D622上使用的是AD82128 codec芯片，下面以AD82128为例说明如何适配codec芯片。

## kernel 

1.从codec厂家处获取codec驱动代码，根据实际情况修改好之后，放入到 `$SDK/kernel/aml5.4/sound/soc/codecs/amlogic `目录下，并修改下的Makefile和Kconfig。

Makefile加入下面内容：
```
snd-soc-ad82128-objs := ad82128.o
obj-$(CONFIG_AMLOGIC_SND_SOC_AD82128) += snd-soc-ad82128.o
```

Kconfig加入下面内容：
```
config AMLOGIC_SND_SOC_AD82128
tristate "ESMT AD82128"
depends on AMLOGIC_SND_SOC_CODECS
depends on I2C
default n
help
Enable Support for ESMT AD82128 CODEC.
Select this if your AD82128 is connected via an I2C bus.
```
2.在kernel配置文件 $SDK/kernel/aml5.4/arch/arm64/configs/meson64_a64_smarthome_defconfig 里，加上配置项：
```
CONFIG_AMLOGIC_SND_SOC_AD82128=y
```
3.修改设备树，音频输出是连接到TDMB接口上，所以在 aml-audio-card,dai-link@1 节点内部，修
改tdmbcodec为如下内容：

在`$SDK/kernel/aml5.4/arch/arm64/boot/dts/amlogic/a5_a113x2_av400_1g.dts`文件中修改设备树
```
tdmbcodec: codec {
prefix-names = "ad82128_c_30","ad82128_c_31",
"ad82128_c_34","ad82128_c_35",
"ad82128_d_30","ad82128_d_31",
"ad82128_d_34","ad82128_d_35";
对于有多个codec芯片的情况，prefix-names是必须的。
在i2c节点上增加AD82128的配置：
sound-dai = <
&ad82128_c_30
&ad82128_c_31
&ad82128_c_34
&ad82128_c_35
&ad82128_d_30
&ad82128_d_31
&ad82128_d_34
&ad82128_d_35
>;
};
```

对于有多个codec芯片的情况，prefix-names是必须的。
在i2c节点上增加AD82128的配置：
```
&i2c2 {
status = "okay";
pinctrl-names="default";
pinctrl-0=<&i2c2_pins1>;
clock-frequency = <100000>;
ad82128_c_30: ad82128_c_30@30 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x30>;
status = "okay";
reset_pin = <&gpio GPIOD_9 GPIO_ACTIVE_HIGH>;
};
ad82128_c_31: ad82128_c_31@31 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x31>;
status = "okay";
};
ad82128_c_34: ad82128_c_34@34 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x34>;
status = "okay";
};
ad82128_c_35: ad82128_c_35@35 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x35>;
status = "okay";
};
};
&i2c3 {
status = "okay";
pinctrl-names="default";
pinctrl-0=<&i2c3_pins2>;
clock-frequency = <100000>;
ad82128_d_30: ad82128_d_30@30 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x30>;
status = "okay";
};
ad82128_d_31: ad82128_d_31@31 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x31>;
status = "okay";
};
ad82128_d_34: ad82128_d_34@34 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x34>;
status = "okay";
};
ad82128_d_35: ad82128_d_35@35 {
compatible = "ESMT,ad82128";
#sound-dai-cells = <0>;
reg = <0x35>;
status = "okay";
};
};
```
4.然后对kernel进行重新编译并生成镜像。
```
make linux-dirclean
make linux-rebuild
make
```
5.把镜像烧录到板端进行测试。下面命令会依次通过各个喇叭进行发声，看看各个喇叭是否可以正常
出声。
```
speaker-test -D hw:0,1 -t wav
```

当前D622上一共有8颗AD82128 codec芯片，每个芯片支持2个声道，最多可以支持16个声道的音频。

不过业务上当前最多是7.1.4声道(7+1+4=12)，所以还有两颗芯片实际上在业务中并没有被使用。

## audioservice 

audioservice中使用了ALSA amixer接口进行音量控制。

为了可以在audioservice里控制音量，还需要修改`$SDK/vendor/amlogic/audioservice/src/config/br/av400_sbr_7_1_4_datmos_V1_7.conf`文件的channels_volume字段。

ad82128_d_30 Ch1这样的字符串，是通过 amixer scontrols 查询到的。需要注意各个codec芯片跟实际喇叭的位置对应关系。

```
"volume_config": {
"main_volume": "hw_map",
"hw_map": [{
"name": "ad82128",
"vol_type": "amixer",
"linear": true,
"volume": 40,
"mute": false,
各个声道名字的含义如下所示：
用修改后的 av400_sbr_7_1_4_datmos_V1_7.conf 替换板端的/etc/default_audioservice.conf文件
板端把audioservice重启：
用asplay命令调节音量，看看是否起作用。
"map_config": [{
"softmax": 100,
"softmin": 0,
"softstep": 1,
"hwmax": 1,
"hwmin": 0.3,
"hwstep": 0.007
}],
"channels_volume": [
{ "lf_ch": "ad82128_d_30 Ch1" },
{ "rf_ch": "ad82128_d_30 Ch2" },
{ "ls_ch": "ad82128_d_31 Ch1" },
{ "rs_ch": "ad82128_d_31 Ch2" },
{ "lfe_ch": "ad82128_d_34 Ch1" },
{ "c_ch": "ad82128_d_34 Ch2" },
{ "lrs_ch": "ad82128_d_35 Ch1" },
{ "rrs_ch": "ad82128_d_35 Ch2" },
{ "ltf_ch": "ad82128_c_30 Ch1" },
{ "rtf_ch": "ad82128_c_30 Ch2" },
{ "ltr_ch": "ad82128_c_31 Ch1" },
{ "rtr_ch": "ad82128_c_31 Ch2" }
]
}],
```
- 用修改后的 av400_sbr_7_1_4_datmos_V1_7.conf 替换板端的/etc/default_audioservice.conf文件
```
adb push av400_sbr_7_1_4_datmos_V1_7.conf /etc/default_audioservice.conf
```
- 板端把audioservice重启：
```
/etc/init.d/S90audioservice restart
```
- 用asplay命令调节音量，看看是否起作用。
```
asplay set-volume 80
```

# 四、调试

## 4.1 halaudio 日志级别：
```
#define LEVEL_VERBOSE   0
#define LEVEL_INFO      1
#define LEVEL_DEBUG     2
#define LEVEL_WARN      3
#define LEVEL_ERROR     4
#define LEVEL_FATAL     5
#define LEVEL_MAX       0xF

echo "AML_AUDIO_DEBUG=1" > /tmp/AML_AUDIO_DEBUG

echo "AML_AUDIO_DEBUG=4" > /tmp/AML_AUDIO_DEBUG #打开error 级别的日志
echo "AML_AUDIO_DEBUG=5" > /tmp/AML_AUDIO_DEBUG #关闭日志
```

## 4.2 halaudio 调试

- 打开halaudio的调试日志，在板端执行下面的命令：

        echo "AML_AUDIO_DEBUG=4" > /tmp/AML_AUDIO_DEBUG

执行后立刻生效，不需要重启进程。

4表示ERROR级别，

5表示关闭日志。

所以下面的命令就是关闭日志打印

        echo "AML_AUDIO_DEBUG=5" > /tmp/AML_AUDIO_DEBUG

- halaudio的代码在目录 $SDK/multimedia/aml_halaudio 中

- 修改后的编译命令：

        make aml-halaudio-rebuild

- 编译后用adb将libhalaudio.so拷贝到板端，PC端执行命令：
```
adb push $SDK/output/a5_av400_a6432_release/target/usr/lib/libhalaudio.so  /usr/lib
adb shell sync
```

- 然后板端执行下面命令重启audioservice：

        /etc/init.d/S90audioservice restart

## 4.2.1 Dump data

### 4.2.1.1 Input data
```
mkdir -p /data/audio/
chmod 777 /data/audio/ -R
echo "AML_AUDIO_DUMP_FILE=dump_input=1" >/tmp/AML_AUDIO_DEBUG
```
音频数据位于`/data/audio/input.data`，其格式为2ch + S16_LE + 采样率，这些数据来自HDMI Input。

### 4.2.1.2 output data
```
mkdir -p /data/audio/
chmod 777 /data/audio/ -R
echo "AML_AUDIO_DUMP_FILE=dump_output=1" >/tmp/AML_AUDIO_DEBUGG
```
音频数据位于`/data/audio/output.pcm`，是12ch + S32_LE + 48K，这些数据输出到AlSA接口。

### 4.2.1.3 ATMOS data

```
mkdir -p /data/audio/
chmod 777 /data/audio/ -R
echo "AML_AUDIO_DUMP_FILE=dump_decoder=1" >/tmp/AML_AUDIO_DEBUG
```
音频数据位于`/data/audio/datmos_raw_in.data`和`/data/audio/datmos_pcm_out.data`，datmos_raw_in.data是ATMOS解码前的输入音频数据。datmos_pcm_out.data是ATMOS解码的音频数据。

## 4.3 audioservice 或homeapp 日志级别
{ "LOG_QUIET", "LOG_ERR", "LOG_WARNING", "LOG_INFO","LOG_DEBUG", "LOG_VERBOSE", NULL }

```
echo "all:LOG_DEBUG" > /tmp/AML_LOG_audioservice 
//打开audioservice 的日志

echo "all:LOG_DEBUG" > /tmp/AML_LOG_homeapp  
//打开homeapp 的日志
```
## 4.4 audioservice 调试

打开调试日志开关，在板端执行下面命令，即可打开audioservice和homeapp的DEBUG级别的日志打
印。修改立刻生效，不需要重启audioservice和homeapp进程。
```
echo "all:LOG_DEBUG" > /tmp/AML_LOG_audioservice
echo "all:LOG_DEBUG" > /tmp/AML_LOG_homeapp
```
如果要从上电开始就一直打开日志，则在`/etc/init.d/S90audioservice`加上上面两行命令。

```
start() {
printf "Starting audioservice: "
echo "all:LOG_DEBUG" > /tmp/AML_LOG_audioservice
echo "all:LOG_DEBUG" > /tmp/AML_LOG_homeapp
/usr/bin/audioservice /etc/default_audioservice.conf&
/usr/bin/homeapp -D music_vol -s &
echo "OK"
}
```
如果修改了audioservice或者homeapp里的代码，则执行下面的语句把相关文件推送到板端。
```
adb push $SDK/output/a5_av400_a6432_release/target/usr/lib/libasexternal_m031.so.0.0.0 /usr/lib

adb push $SDK/output/a5_av400_a6432_release/target/usr/lib/libasexternal_input.so  /usr/lib

adb push $SDK/output/a5_av400_a6432_release/target/usr/lib/libasclient.so.0.0.0  /usr/lib

adb push $SDK/output/a5_av400_a6432_release/target/usr/bin/audioservice /usr/bin

adb push $SDK/output/a5_av400_a6432_release/target/usr/bin/homeapp /usr/bin

adb shell chmod 755 /usr/bin/audioservice
adb shell chmod 755 /usr/bin/homeapp
adb shell sync
```

然后重启audioservice即可生效：
```
/etc/init.d/S90audioservice restart
```
### 4.4.1 homeapp 与audioservice的 DBus通信协议

DBus通信协议描述是在`$SDK/vendor/amlogic/audioservice/src/aml.linux.dbus.xml`文件中。
audioservice作为DBus Server，homeapp和asplay都是DBus Client。
#### 4.4.1.1 xml生成C code
DBus协议是用aml.linux.dbus.xml这个xml文件描述的，audioservice和homeapp里不能直接使用，需要在编译时使用 gdbus-codegen 工具生成 audioservice_gdbus.c audioservice_gdbus.h 。

这个过程是在 $SDK/vendor/amlogic/audioservice/src/Makefile.am 中完成的。
```
audioservice_gdbus.c audioservice_gdbus.h : aml.linux.dbus.xml gdbus-codegen --interface-prefix=aml.linux.dbus --generate-c- code=audioservice_gdbus $<
```

### 4.4.2 audioservice编译脚本描述
audioservice使用autotools来编译代码。

下面描述的文件以 $SDK/vendor/amlogic/audioservice 作为根目录。
涉及的脚本文件有：

文件：
```
configure.ac：把configure命令的选项作为变量给Makefile.am文件。
Makefile.am ：顶层Makefile.am，控制对哪些目录进行编译。
src/Makefile.am ：使用configure.ac传递过来的变量控制编译哪些文件，并转成宏定义传递给C/C++文件。
homeapp/Makefile.am ：使用configure.ac传递过来的变量控制编译哪些文件，并转成宏定义传递给C/C++文件。
```


### 4.4.3 homeapp调试

homeapp的代码在 $SDK/vendor/amlogic/audioservice/homeapp 目录下。
homeapp的代码包括：

#### avs 

先进行网络配置

然后将配置文件push到板端

```
asplay enable-input AVS
```
切换到AVS模式

```
/etc/init.d/S48avs stop
```
停止进程

- 相关脚本

/etc/init.d/S48avs：执行/etc/init.d/avs_moniter.sh。

/etc/init.d/avs_moniter.sh ：检测SampleApp进程是否存在，如果不存在，则尝试启动该进程。

/etc/init.d/avs_mdns.sh： 启动mdns服务，方便手机APP发现设备并进行配置。

# 五、用户添加player client 

操作步骤：

用户如果要新增自己的player client，按下面步骤操作：
1.以halaudio_client.c、avs_client.c等已有client代码为参考进行修改。例如新增myapp，对应代码文件命名为myapp_client.c，实现 InputHandler_t 结构体要求的函数。并修改Makefile.am文件保证新增文件加入到编译。

2.audioservice.h里增加 AS_Input_e 枚举里对应的枚举值。

3.input_manage.c里 Install_Input_Apps 函数里增加myapp_client的注册。

4./etc/default_audioservice.conf 里input_list增加myapp对应的信息，这样才可以切换到myapp对应的输入方式。


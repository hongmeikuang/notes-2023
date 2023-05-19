# buildroot

## 1、buildroot是什么

```
buildroot 是Linux平台上一个开源的嵌入式Linux系统自动构建框架。
```

## 2、buildroot目录结构

- arch: CPU 构架相关的配置脚本
  
- board: 在构建系统时，board 默认的boot 和Linux kernal配置文件，以及一些板级相关脚本

- boot：uboot 配置脚本目录

- configs: 板级配置文件，该目录下的配置文件记录着该机器平台或者方案使用的工具链，boot、kernal，各种应用软件包的配置。

- dl:  download的简写，下载一些开源包。第一次下载后，下次就不会再去从官网下载了，而是从dl/目录下那开源包，以节约时间。

- docs:

- fs:各种文件系统的自动构建脚本

- Linux：存放linux kernal的自动构建脚本

- package：第三方开源包的自动编译构建脚本，用来配置编译dl目录下载的开源包

- support: 

- system: 存放文件系统目录和设备节点的模板，这些模板会被拷贝到Output/目录下，用以制作根文件系统rootfs
  
- toolchain/ 目录种存放着各种制作工具链的脚本

## 3、编译出的output输出目录介绍

 - iamges: 存储所有映像（内核映像，引导加载程序和根文件系统映像）的位置。这些是需要放在目标系统上的文件
 - build: 构建所有组件的位置（包括主机上buildroot所需工具和针对目标编译的软件包）。该目录为每个组件包含一个子目录。

 - host: 包含为主机构建的工具和目标工具链。

 - staging: 是到内部目标工具链host/的符号链接

 - target:它包含了目标的完整根文件系统，除了设备文件/dev/(buildroot无法创建他们，因为buildroot不能以root身份运行并且不想以root身份运行)之外，所需的一切都存在。

## 4、工具链

buildroot为交叉编译工具提供了两种解决方案：

- 内部工具链，称为buildroot toolchain在配置接口

- 外部工具链external toolchain:

  我们可选择外部工具链，工具链的来源可以选择下载也可以选择指定工具链

   比如 /opt/gcc-linaro-6.3.1-2017.02-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
## 5、buildroot常用make命令

### make 命令通常将执行以下步骤：
1、根据需要下载源文件

2、配置、构建并安装交叉编译工具链，或仅导入外部工作链

3、配置、构建和安装选定的目标软件包。

4、构建内核映像（根据选择）

5、构建引导加载内核映像（如果选择）

6、以选定的格式创建一个根文件系统
### 常用命令

*make help


## 6、与buildroot 平行的目录说明

1、BootLoader ：用于存放u-boot代码，不同的项目在下面会放不同的仓库或分支

2、kernal：用于存放内核代码，包括在内核里面实现的驱动，不同内核版本，会有不同的分支，比如“aml-4.9*”一般表示这是基于Kernel4.9的定制化内核。

3、multimedia:多媒体相关的代码，比如alsa plugin和gstreamer plugin等等相关的代码。（注：后续新增package的代码将统一移到vendor/amlogic下面，建独立仓库维护）

4、toolchain：编译SDK所使用的toolchain。

5、 vendor：芯片相关的package代码，包括Amlogic及3rd party的。Amlogic的package，放在vendor/amlogic目录下面，以独立仓库维护。

6、output：编译后才会产生，调整了编译输出目录到Project root folder下面。

## 7、amlogic Buildroot子目录说明

1、board 目录：和设备相关的改动

  如果需要增加一个硬件环境，需要考虑在board/amlogic下面增加对应的子目录，里面定义相关的rootfs等等内容。

2、configs目录：和项目配置相关的目录

  如果增加一个新的项目，和其他项目功能不一样，需要在这个目录下面增加一个defconfig文件，用于配置新的项目的功能。

3、package目录：组件的配置目录

  如果增加一个功能模块，我们称之为组件（package），可以编译成so供其他package，也可以编译成独立的应用程序，需要在这个目录下面增加对应的config，方便系统可以对这个package进行配置管理。

 ## 8、amlogic buildroot 添加package软件包（本地源码）

  1）要添加自己的本地package，首先在package/amlogic/Config.in中添加指向新增APP目录的Config.in.

  2）然后在package中新增目录（这里以hello world为例），并在里面添加Config.in和hello_world.mk，最后添加对应的hello-world目录。

  3）如果hello-world是一个本地的模块，源码在本地管理，那么源码需要放在本地，也就是下图中的vendor/amlogic/hello-world里面.

  ### 一、package 命名规则

  package目录名称和.mk文件名以及mk里面的宏的开头均需要匹配（相同）

  比如hello，那么package目录名称叫hello，Makefile文件名为hello.mk。

  Package目录下面的Makefile文件hello.mk里面的内容，每个宏都必须以HELLO_开头

ps:建议package目录名中，多个单词之间以"-"（中划线）作为单词分割符，Makefile文件名以“_”（下划线）作为单词分隔符。比如，如果package叫hello world，那么目录名称叫hello-world，Makefile文件名为hello_world.mk。
```
package/   
   +-- amlogic/
     +-- hello-world/
         +-- Config.in
         +-- hello_world.mk
```
Package目录下面的Makefile文件hello_world.mk里面的内容类似下面这样，每个宏都必须以HELLO_WORLD_开头。

```
########################################
#hello world
#
#######################################
HELLO_WORLD_VERSION:= 1.0
HELLO_WORLD_SITE:= $(HELLO_WORLD_TEST_PKGDIR)/helloworld
HELLO_WORLD_SITE_METHOD:=local
HELLO_WORLD_INSTALL_TARGET:=YES
```

### 二、package 本地源码放置目录

    如果pakcage是本地源码，那么请放在vendor/amlogic/目录下，比如vendor/amlogic/hellow-world，源码目录下面放置用于编译的Makefile文件。

### 三、添加package/Condfig.in入口

系统在make menuconfig的时候就可以找到对应的APP的Config.in。
```
    +menu "Private package"
    +       source "package/amlogic/hello-world/Config.in"    
    +endmenu
```
//在package/Condfig.in中添加上述代码，即为入口

    如果在make menuconfig的时候选中hello world，在make savedefconfig的时候就会打开BR2_PACKAGE_HELLO_WORLD=y。

### 四、配置package 对应的Config.in

Config.in文件类似 Kconfig 的配置文件，命名格式遵循Linux Kernel 的 Kconfig 文件规范，如：
```
config BR2_PACKAGE_HELLO_WORLD
bool"hello world"
help
    hello world to you
    ################################################################################
    #
    # hello world
    #
    ################################################################################
```
如果有依赖其他package，请在Config.in里面添加对应的depends on和select选项，详情请参考Kconfig的相关知识

### 五、配置package对应的mk文件

buildroot编译hello world所需要的文件hello_world.mk，需要描述源码位置、安装目录、权限设置等
```
##################################################################################
#helloworld
#################################################################################
HELLO_WORLD_VERSION:= 1.0
HELLO_WORLD_SITE:= $(TOPDIR)/../vendor/amlogic/hello-world
HELLO_WORLD_SITE_METHOD:=local
HELLO_WORLD_INSTALL_TARGET:=YES

define HELLO_WORLD_BUILD_CMDS 
   $(MAKE) CC="$(TARGET_CC)" LD="$(TARGET_LD)" -C $(@D) all
endefdefine HELLO_WORLD_INSTALL_TARGET_CMDS
    $(INSTALL) -D -m 0755 $(@D)/helloworld $(TARGET_DIR)/bin
endef
  
define HELLO_WORLD_PERMISSIONS
      /bin/helloworld f 4755 0 0 - - - - -
endef

$(eval $(generic-package))
```
若应用程序采用cmake编译，则需要将上述hello_world.mk最后一行$(eval $(generic-package))改为$(eval $(cmake-package))

### 六、编写hello world源码(仅本地源码package需要)

简单的编写一个helloworld.c文件：
```
    #include <stdio.h>
    void main(void)
    {    
      printf("Hello world.\n");
    }
```
然后编写Makefile文件：
```
    CPPFLAGS +=
    LDLIBS +=
    all: helloworld
    
    analyzestack: helloworld.o
        $(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ $(LDLIBS)
        
    clean:    rm -f *.o helloworld
    
    .PHONY: all clean
```
若采用cmake编译，则不用编写Makefile文件，需要编写CMakeList.txt文件：
```
    PROJECT(HELLO_WORLD)
    
    SET(SRC_LIST helloworld_test.c)
    
    MESSAGE(STATUS "This is BINARY dir " ${HELLO_WORLD_BINARY_DIR})
    
    MESSAGE(STATUS "This is SOURCE dir " ${HELLO_WORLD_SOURCE_DIR})
    
    ADD_EXECUTABLE(helloworld ${SRC_LIST})
```
### 七、通过make menuconfig 选中APP

通过Target packages -> Private package进入，选中helloworld。

然后save，对helloworld的配置就会保存到.config中。

BR2_PACKAGE_HELLOWORLD等于y时就会生效

PS:如果make menuconfig BR2_PACKAGE_HELLOWORLD=y没生效的话，就在buildroot/configs/a5_av400_release_defconfig里添加写入

BR2_PACKAGE_HELLOWORLD=y

不用make menuconfig 修改。

完成所有设置后，需要source setenv.sh a5 使修改生效。

### 八、编译APP

可以和整个平台一起编译APP，或者make helloworld单独编译。

这两个文件在选中此APP之后，源代码都会被拷贝到output/build/hello-world-1.0.0文件夹中。

然后生成的bin文件拷贝到output/target/bin/helloworld，这个文件会打包到文件系统中。

如果需要清空相应的源文件，通过make hello-world-dirclean。
### 九、运行APP

在shell中输入helloworld，可以得到如下结果。
    
    #which helloworld
    /bin/helloworld
    #helloworld
    hello world.

添加APP工作完成。

## 9、amlogic buildroot添加package软件包（网络下载资源码）

如果hello-word是一个三方的代码，源码在编译的时候，直接从网上下载，那么只要为package提供config文件就可以了。

如果不需要准备源码目录，只要添加对应的package目录下面就可以了，这个可以参考buildroot标准手册中的例子，具体可以参考package/libcurl的内容来实现。

## 10、Amlogic buildroot下如何配置项目defconfig

请根据项目选择对应的config进行编译，比如A213Y公版项目，编译的时候，会选择a213y_au401_a32_release_defconfig作为config文件，这个文件及include相关的文件会包含这个项目的所有config项，有的是一些功能开关config，还有一些用于传递参数，比如，下面这些宏定义很多这个项目依赖的关键路径。

 kernel代码的路径
```
BR2_LINUX_KERNEL_CUSTOM_LOCAL_PATH="$(TOPDIR)/../kernel/aml-4.9-q"
```
U-Boot的路径
```
BR2_TARGET_UBOOT_CUSTOM_LOCAL_LOCATION="$(TOPDIR)/../bootloader/uboot-m805"
```
rootfs的路径
```
BR2_ROOTFS_COMMON_OVERLAY="board/amlogic/common/rootfs/rootfs-49/"BR2_ROOTFS_OVERLAY="board/amlogic/mesona213y_au401/rootfs/"
```
upgrade路径
```
BR2_ROOTFS_UPGRADE_DIR="board/amlogic/common/upgrade/upgrade-m8b/"BR2_ROOTFS_UPGRADE_DIR_OVERLAY="board/amlogic/mesona213y_au401/upgrade/"
```
ota的路径
```
BR2_RECOVERY_OTA_DIR="board/amlogic/common/ota/ota-m8b/"
```
post build脚本
```
BR2_ROOTFS_POST_BUILD_SCRIPT="$(TOPDIR)/board/amlogic/mesona213y_au401/post_build.sh"
```
多媒体相关的模块的路径
``` 
BR2_PACKAGE_MEDIA_MODULES_LOCAL_PATH="$(TOPDIR)/../hardware/aml-4.9/amlogic/media_modules-q"
```
## 11、如何确定当前编译器使用的toolchain
如何确认当前编译使用的toolchain

举例说明，A213Y公版项目

a213y_au401_a32_release_defconfig中包含了下面文件：
```
    2 BR2_arm=y
    
    3 BR2_ARM_KERNEL_32=y
    
    4、BR2_TOOLCHAIN_EXTERNAL_LINARO_ARM_6DOT2_201702_PREINSTALLED=y
    
    5 BR2_TOOLCHAIN_EXTERNAL=y
    
    6 BR2_TOOLCHAIN_EXTERNAL_PREINSTALLED=y
    
    7 BR2_TOOLCHAIN_EXTERNAL_PATH="$(TOPDIR)/../toolchain/gcc/linux-x86/arm/gcc-linaro-6.3.1-2017
    02-x86_64_arm-linux-gnueabihf"
    
    8 BR2_KERNEL_TOOLCHAIN_PATH="$(TOPDIR)/../toolchain/gcc/linux-x86/arm/gcc-linaro-6.3.1-2017.02-x86_64_arm-linux-gnueabihf"
    
    9 BR2_KERNEL_TOOLCHAIN_LINARO_ARM_201702=y
```

arch_a32_6.3.1.config里面包含了toolchain相关的信息。

这个toolchain表示用了ARM32的toolchain，并且用了扩展toolchain，并设定了user space和Kernelspace的toolchain所在的目录。

#include "arch_a32_6.3.1.config"

## 12、如何配置package开关

如果需要打开某个package，请在项目config文件（比如：a213y_au401_a32_release_defconfig）中直接打开即可。
如果需要关闭某个package，请在项目config文件（比如：a213y_au401_a32_release_defconfig）和include的其他config文件中找到对应的config文件中，去掉相应的config定义即可。
```
    # open XXX package for ...
    BR2_XXX=y
```
## 13、amlogic buildroot 下如何让添加设备驱动

### 1）修改dts文件
  先找到DTS文件，先在项目config文件（比如：a213y_au401_a32_release_defconfig）和include的其他config文件中找到下面这个宏，确定DTS文件名

//在buildroot/configs/amlogic/a213y_au401.config文件里
```
  BR2_LINUX_KERNEL_INTREE_DTS_NAME="mesona213y_au401_a330_512m" 
```
打开文件kernel/common/arch/arm/boot/dts/amlogic/mesona213y_au401_a330_512m.dts，在里面添加DTS节点如下：
```
    adc_keypad {
          compatible = "amlogic, adc_keypad";
          status = "okay";    key_name = "enter";
          key_num = <1>;
          io-channels = <&saradc SARADC_CH0>;
          io-channel-names = "key-chan-0";
          key_chan = <SARADC_CH0>;
          key_code = <27>;
          key_val = <389>; //val=voltage/1800mV*1023   
          key_tolerance = <40>;      
    };
```
### 2）添加或者修改按键相关的驱动

打开kernel/aml-4.9/drivers/amlogic/input/keyboard/目录，这里有adc_keypad的驱动文件adc_keypad.h和adc_keypad.c。在对应的Kconfig中增加config说明，用于menuconfig的时候配置。
```
    menuconfig AMLOGIC_INPUT_KEYBOARD
      bool "Keyboards and keypads"
      default n
      help
        Say Y here, and a list of supported keyboards and keypads will bedisplayed.
        This option doesn't affect the kernel.
        
        If unsure, say Y.
        
    if AMLOGIC_INPUT_KEYBOARD
    
    config AMLOGIC_ADC_KEYPADS
      tristate "Meson ADC keypad support"
      depends on AMLOGIC_SARADC
      default n
      help
        Say Y here if you want to use the Meson ADC keypad.
    
    endif # AMLOGIC_INPUT_KEYBOARD
```
在对应的Makefile文件中，增加编译控制：
```
    obj-$(CONFIG_AMLOGIC_ADC_KEYPADS)+= adc_keypad.o
```
### 3)修改kernal 的config文件

先找到Kernel的config文件，先在项目config文件（比如：a213y_au401_a32_release_defconfig）和include的其他config文件中找到下面这个宏，确定DTS文件名：

//在buildroot/coonfigs/amlogic/m8b_base.config文件里
```
    BR2_LINUX_KERNEL_DEFCONFIG="meson32"
```
从而确认Kernel config文件名称为meson32_defconfig，打开这个文件，在里面添加下面的几行，用于打开按键。
```
    CONFIG_AMLOGIC_INPUT=y
    CONFIG_AMLOGIC_INPUT_KEYBOARD=y
    CONFIG_AMLOGIC_ADC_KEYPADS=y
```


https://doc.amlogic.com/file/detail?type=1&id=18183
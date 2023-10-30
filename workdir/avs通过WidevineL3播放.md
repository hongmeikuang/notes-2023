---
data：20230911
目的：avs通过Widevine L3播放
背景：
	之前gaojie.song实现过通过widevine L3 播放音源的功能，不过都是提供的库，现在我们这边要将avs的数据通过Widevine L3的方式播放。
---

# Widevine L3播放的流程

要想使avs通过Widevine L3播放，就要先弄清楚先前Widevine L3的播放流程。

https://confluence.amlogic.com/pages/viewpage.action?pageId=247993646

下面是之前a113l实现的流程。

![image-20230911140715091](images/image-20230911140715091.png)

gaojie.song提供的库文件有：

![image-20230911140910499](images/image-20230911140910499.png)

各个文件的调用顺序如下：

```
wlcdmi_player-> libgstwlcdmi.so  -> WLCDMI Client --> WLCDMI server - -> libwidevine_wlcdmi_plugin.so ---> widevine 
```

# 阅读源码

## 阅读wlcdmi_player和libgstwlcdmi.so源码

wlcdmi_player和libgstwlcdmi.so源码仓库路径为：

```
multimedia/gst-plugin-aml-wlcdmi
```

提交的patch 如下：

```
https://scgit.amlogic.com/#/c/298879/
```

### 首先分析wlcdmi_player和libgstwlcdmi的调用关系

wlcdmi_player设置了一些属性，通过playbin的方式

现在写了一个例子，通过单独创建的方式，但是在与wlcdmi连接的时候出现了错误。

/tmp/gst_player.c是最新的代码，每个element都单独创建，然后来播放加密的音频。

这个目前还有一些问题。先去搭建A1的AATS测试环境。

### 单独创建element搭建pipeline

wlcdmi_player 是通过playbin的方式来搭建pipeline的。playbin是特殊的pipeline，他可以自动选择合适的解码器。

只需指定媒体的URI，playbin会自动创建和配置必要的元素，包括src、decode、convert、sink等。不用手动配置。

现在avs要用的是使用单独创建元素的方式，需要手动配置和连接各种元素

但现在avs里的pipeline是通过单独的element串联起来构建成pipeline的。所以我需要将wlcdmi解密插件插入avs中。由于avs调试起来比较困难。所以本次任务的步骤分为如下几步

1. 先创建一个简单的pipeline，然后将wlcdmi插件插入进去。
2. wlcdmi插件插入到avs中
3. 验证播放

#### 创建test-pipeline并插入wlcdmi插件

wlcdmi的库放在/usr/lib/gstreamer-1.0目录下。这是gstreamer插件加载目录

只要创建了wlcdmi element并将它添加到pipeline中，gstreamer会自动加载该插件。

在pipeline 中插入wlcdmi插件时遇到的问题是：wlcdmi应该解密的是audio数据。但在构建是连接wlcdmi和aacparse失败，通过查看日志发现，是由于wlcdmi的pad和aacparse的pad 格式不匹配。

通过debug发现，wlcdmi和aacparse的连接要在pad-added回调里进行。因为在出发回调时表示上一级的element已经连接完成，数据格式也已经确定，这时候在进行连接，wlcdmi才会接收到正确的数据格式（audio数据），否则，如果预先就将wlcdmi与aacparse进行连接，wlcdmi不确定他接收到的是什么数据，所以会出错。

![wlcdmi_player](images/wlcdmi_player.png)

![](images/error-5-1698115661244.png)

![success](images/success-1698115731935.png)

#### 按照单独构建element的方式，将wlcdmi插入到avs 的pipeline中。

1. 首先，先搞清楚avs pipeline的构建流程。可以通过导出dot图的方式查看。
2. 分析wlcdmi插件插入的合适位置。
3. 将wlcdmi插件添加到avs中
4. 验证

# gstreamer debug

查看某个插件是否加载,例如 wlcdmi

```
rm ~/.cache/gstreamer-1.0/ -rf
gst-inspect-1.0  | grep wlcdmi
```

修改日志级别

```
export GST_DEBUG=*:3
```

导出dot图

```
export GST_DEBUG_DUMP_DOT_DIR=/tmp
```

gstreamer播放完成后会生成XXX.dot 文件在/tmp目录下。

将XXX.dot文件放到ubantu电脑上执行以下命令，生成png图片

```
dot -Tpng 0.00.00.375040292-gst-play.error.dot   -o error-5.png
```

查看某个类型的element的具体属性,比如aacparse。

```
 gst-inspect-1.0 aacparse
```



# A5环境搭建

烧录的镜像是0915-1.img,是已经加密过的。



在cmder_mini目录有一个env_set.sh 脚本，通过该脚本可以快速搭建Widevine环境

1. adb push env_set.sh 

2. 将A5联网   可以ping 通Google

3. 将桌面上的Widevine所需的库push到板子上

4. 将wayland相关的库也push到板子上

5. 修改声卡输出为hw:0,1

   vi /etc/gst-soundcard.conf

   ```
   gst-sound-card =
   {
       hw="hw:0,1";
   };
   ```

6. 先在一个界面运行wlcdmi_server

```
./env_set.sh player
```

7. 另开一个界面运行player

```
./env_set.sh server
```


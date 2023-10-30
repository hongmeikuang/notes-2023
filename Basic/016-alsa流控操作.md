#  alsa

一般操作步骤：

1、打开回放或录音接口

2、设置硬件参数(访问模式，数据格式，信道数，采样率，等等)

3、while 有数据要被处理：

4、读PCM数据(录音) 或 写PCM数据(回放)

5、关闭接口

**编译时需要链接-lasound**

## 函数

```
1、snd_pcm_open(&handle.,pcm_name,stream,open_mode);
```
&handle，是static snd_pcm_t *handle;该结构体指针，在后面的所有操作中，都要用到。

pcm_name，是打开的pcm设备节点，可以是“default” ,也可以是“hw:0,0”。具体有何区别，可网查询。

stream，对应放音与录音操作，SND_PCM_STREAM_PLAYBACK，SND_PCM_STREAM_CAPTURE。

open_mode，是一些相关功能设置，比如：是否阻塞，是否重采样，是否软件调音量等。

```
2、snd_pcm_info(handle, info)；
```
用来获取录放音 声卡与设备等相关信息。不是必须的接口。

```
3、snd_pcm_nonblock(handle, 1);
```
如果需要非阻塞方式，则将第二个参数，设置为1。snd_pcm_open后，默认为阻塞方式，因此，该接口可以不用调用。

```
4、snd_pcm_hw_params_alloca(&params);
```
申请一段snd_pcm_hw_params_t *params;结构体空间。使用该结构体，来配置底层ALSA的相关硬件参数，比如：声道，采样率，位宽，buffer大小等。

stream == SND_PCM_STREAM_PLAYBACK

playback(argv[optind++])--->playback_go(fd, 0, pbrec_count, FORMAT_AU, name--->set_params()-->snd_pcm_hw_params_alloca

```
5、 snd_pcm_hw_params_any(handle, params);
```
硬件参数初始化

```
6、snd_pcm_hw_params_set_access(handle, params,SND_PCM_ACCESS_RW_INTERLEAVED);
```
设置为交错模式
```
7、snd_pcm_hw_params_set_format(handle, params,SND_PCM_FORMAT_S16_LE);
```
设置读取数据格式

```
8、snd_pcm_hw_params_set_channels(handle, params, 2);
```
2通道设置

```
value = 44100
9、snd_pcm_hw_params_set_rate_near(handle, params,&val, &dir);
```
将采样率设置为44100Hz
```
frames = 32;
10、snd_pcm_hw_params_set_period_size_near(handle,params, &frames, &dir);
```
设置period size

```
11、snd_pcm_hw_params(handle, params);
```
将参数写入驱动程序

```
12、snd_pcm_hw_params_get_period_size(params,&frames, &dir);
size = frames * 4; //* 2 bytes/sample, 2 channels */
buffer = (char *) malloc(size);
```
使用足够大的缓冲区来容纳一个周期

```
14、snd_pcm_readi(handle, buffer, frames);
```
从声卡中读取数据
```
15、write(1, buffer, size);
```
将数据写到输出

```
16、snd_pcm_drain(handle)
```
对于播放，会先等待所有挂起没有传输完的数据帧先播完，才会去关闭PCM。

```
17、snd_pcm_close(handle);
```
关闭pcm

## 播放或者录音流程

1、打开设备，设置硬件参数

2、alsa 获取周期大小

3、申请周期大小的缓冲区来存储样本。

4、播放是从标准输入读入数据，并往缓冲区填充一个周期的样本。然后通过snd_pcm_writei来发送数据。直到数据读完。

5、最后调用snd_pcm_drain将所有没有挂起的没有传输完的声音样本传输完全，最后关闭该音频流。释放之前动态分配的缓冲区，退出。


# ALSA架构

ALSA（Advanced Linux Sound Architecture）即高级 Linux 声音架构。

嵌入式移动设备的音频子系统目前主要是ALSA 驱动 asoc 框架，其中包括 codec driver、 platform driver、 machine driver 等。 

`codec driver`只关心 codec 本身； 

`platform driver`主要包括平台 cpu dai（ 如 i2s） ， dma 等部分；

`machine driver`主要将 platform driver 和 codec driver 连接起来。

platform 和 dai driver 一般修改较少， 主要修改machine 和 codec driver。

ALSA是Advanced Linux Sound Architecture 的缩写，是linux的主流音频体系结构。在内核设备驱动层，ALSA 提供了 alsa-driver，同时在应用层，ALSA 为我们提供了 alsa-lib，应用程序只要调用 alsa-lib 提供的 API，即可以完成对底层音频硬件的控制。

## arecord与aplay 

1.查询Linux系统下设备声卡信息

        arecord -l

2.录制音频
```
arecord -D hw:0,0 -r 16000 -c 1 -f S16_LE test.wav
```
参数解析
-D:指定录音设备
0,0:指定card 0, device 0
-r:指定采样率
-c:指定通道（channels)
-f:指定录音格式
-d:录制时长，如果不指定也可以ctrl+c打断
最后是生成文件名

3.播放音频
```
aplay -D hw:0,0 test.wav
```
4.参数说明

        arecord -h
        aplay -h

## amixer

用来设置音频参数，用法可以用-h查询，基本使用方法为：
```
amixer controls 用于查看音频系统提供的操作接口
amixer contents 用于查看接口配置参数
amixer cget + 接口函数
amixer cset + 接口函数 + 设置值
```

### 查询可设置项

        amixer contents

https://blog.csdn.net/lihuan680680/article/details/121941653

# 其他说明

## PCM接口

最简单的音频接口是PCM（脉冲编码调制）接口，该接口由时钟脉冲（BCLK）、帧同步信号（FS）及接收数据（DR）和发送数据（DX）组成。

在FS信号的上升沿数据传输从MSB开始，FS频率等于采样频率。FS信号之后开始数据字的传输，单个数据位按顺序进行传输，一个时钟周期传输一个数据字。PCM接口很容易实现，原则上能够支持任何数据方案和任何采样频率，但需要每个音频通道获得一个独立的数据队列。

## IIS接口

IIS接口在20世纪80年代首先被PHILIPS用于消费音频产品，并在一个称为LRCLK（Left/Right CLOCK）的信号机制中经过转换，将两路音频变成单一的数据队列。当LRCLK为高时，左声道数据被传输；LRCLK为低时，右声道数据被传输。

与PCM相比，IIS更适合于立体声系统，当然，IIS的变体也支持多通道的时分复用，因此可以支持多通道

## Linux ALSA 音频设备驱动

ALSA系统包括包括驱动包`alsa-driver`、开发包alsa-libs、开发包插件alsa-libplugins、设置管理工具包alsa-utils、其他声音相关处理小程序包alsa-tools、特殊音频固件支持包alsa-firmware、OSS接口兼容模拟层工具alsa-oss供7个子项目，其中只有驱动包是必需的。

目前ALSA内核提供给用户空间的接口有：

（1）设备信息接口（/proc/asound）

（2）设备控制接口（/dev/snd/controlCX）

（3）混音器设备接口（/dev/snd/mixerCXDX）

（4）PCM设备接口（/dev/snd/pcmCXDX）

（5）原始MIDI(迷笛)设备接口（/dev/snd/midiCXDX）

（6）声音合成(synthesizer)设备接口（/dev/snd/seq）

（7）定时器接口（/dev/snd/timer）

*这些接口被提供给alsa-lib使用，而不是给应用程序使用，应用程序最好使用alsa-lib，或者更高级的接口比如jack提供的接口。*

## linux ASoC音频设备驱动

ASoC是ALSA在SoC方面的发展和演变，它的本质仍然属于ALSA，但是在ALSA架构基础上对CPU相关的代码和Codec相关的代码进行了分离，其原因是采用传统ALSA架构情况下，同一型号的Codec工作于不同的CPU时，需要不同的驱动，这是不符合代码重用的要求的。

ASoC主要由3部分组成：
（1）Codec驱动，这一部分只关系Codec本身，与CPU相关的特性不由此部分操作;

（2）平台驱动(platform)，这一部分只关心CPU本身，不关系Codec，它主要处理了两个问题：DMA引擎和SoC解除的PCM、IIS或AC’97数字接口控制。

（3）板驱动(Machine)，这一部分将平台驱动和Codec驱动绑定在一起，描述了板一级的硬件特征

以上3部分中，1和2基本都可以仍然是通用的驱动了，即Codec驱动认为自己可以连接任意CPU，而CPU的IIS、PCM、或AC’97接口对应的平台驱动则认为自己可以连接符号其接口类型的Codec，只有3是不通用的，由特定的电路板上具体的CPU和Codec确定，因此它很像一个插座，上面插着Codec和平台这两个插头。ASoC的用户空间编程方法与ALSA完全一致。

# ALSA lib 基本概念

1、channael

通道，即我们熟知的声道数,左/右声道，5.1channel等等

2、sample 

sample即一次采样，通常的sample bit指的是一个channnel上，一次采样的bit数（常见的sample bit 8/16/24/32bits）

3、frame 

一个frame是一次采样时所有channel上的sample bit.即frame = channels * (sample bit).

4、period 周期

A period is the number of frames in between each hardware interrupt.
周期是每个硬件中断之间的帧数。

每当hardware buffer 中有peroid size个frame的空间时，硬件就产生中断，来通知alsa driver来往硬件写数据。

5、buffer size

hardware buffer size 是由多个peroid组成。

buffer size = peroid size * peroids。

缓冲区大小=周期数*一个周期的大小

6、Data access and layout  数据访问和布局

7、Hardware parameter  硬件参数

Hardware parameter是作用于声卡硬件的，包括sample rate, sample format, interupt intervals, data access and layout, buffer size.

8、Software parameter  软件参数

software parameter 是作用于alsa core的。

通常用来控制When to start the device, what to do about xruns, Available minimum space/data for wakeup,Transfer chunk size.

9、 When to start the device

通过`API snd_pcm_sw_params_set_start_threshold`来设置什么时候开始启动声卡。

对于playback，假设第设置start threshold 为320,那么就是说，当用户调用writei，写入的数据，将暂时存在alsa驱动空间里，当这个数据量达到 320帧时，alsa驱动才开始将数据写入hardware buffer，并启动D/A转换。

10、What to do about xruns

xrun指的是，声卡period一到，引发一个中断，告诉alsa驱动，要填入数据，或读走数据，但是，问题在于alsa的读取和写入操作必须用户调用writei和readi才会发生的，它不会去缓存数据。如果上层没有用户调用writei和readi，那么就会产生 overrun（录制时，数据都满了，还没被alsa驱动读走）和underrun（需要数据来播放，alsa驱动却不写入数据），统称为xrun。

当xrun发生时，可以在空余空间超过stop threshold时，stop audio interface.
也可以通过设置silence threshold，当空余空间超过silence threshold时，就hardware buffer 写入silence.


# 音频框架总结

(1).ALSA是 kernel里面管理 audio 的核心，我们的 audio driver部分一般会调用snd_soc_register_card(), 将我们在软件层面抽象出来的声卡注册进audio核心

(2).在用户空间android同样给我们提供了tinyalsa方便 我们操作我们的driver

(3).audio的hal层，这部分也是和具体的厂商有关，

(4).mediaservice,media是在init.rc里面启动的native进程在我们所用的android版本中这个service其实还启动了cameraservice

(5).在 mediaservice 启动之后会zygote进程，zygote紧接着会启动android的核心进程systemserver进程。

在systemserver进程里面会注册AudioService，其实这里AudioService虽然叫做service，其实他是通过调用AudioSystem间接地获得AudioFlinger的代理到具体的audio，

之所以service是由于我们一般在应用程序中又是通过获得AudioService的代理操作的，可见虽然android的binder进程间通讯功能强大但是效率可能并没有其他的进程间通讯机制高效。

比如我们一般在应用程序中通过 AudioManager 操作得到当前的音量会首先通过binder机制先获得 AudioService 代理，然后AudioService在通过AudioSystem获得AudioFlinger的代理才能操作到实际的实现部分会发现这里一次调用却用了两次binder通信，显然没有普通的进程间通信机制高效，优点确实功能很强大。

(6).应用程序中有 AudioTrack，AudioRecoder， AudioManager,MediaPlayer 等接口实际操作我们的audio的。


可参考文档

https://confluence.amlogic.com/pages/viewpage.action?pageId=188636466

https://confluence.amlogic.com/pages/viewpage.action?pageId=193253612

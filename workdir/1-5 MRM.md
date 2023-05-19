# MRM测试

## 2022.12.22

###  OpenSSL 是否影响 没有mrm功能的avs 

在没有MRM 功能的情况下。如果没有OpenSSL ，avs能否正常

删掉libssl **,libcrypto.so* *  ,avs 不能正常启动

### 确保mrm用的库全是 Amazon 编译出来的

删掉板子里所有 与Amazon 相关的库，然后将Amazon编译的库全部推进板端

出现以下错误

```
/usr/bin #  ./SampleApp /etc/AlexaClientSDKConfig.json NONE back 16 /etc/MRMConf
ig.json
./SampleApp: /usr/lib/libasound.so.2: no version information available (required by /usr/lib/libLibSampleApp.so)
./SampleApp: /usr/lib/libasound.so.2: no version information available (required by /usr/lib/libLibSampleApp.so)
configFile /etc/MRMConfig.json
configFile /etc/AlexaClientSDKConfig.json
Running app with log level: NONE
terminate called after throwing an instance of 'std::system_error'
  what():  Invalid argument
Aborted
```

然后我发现是cjson.so.4 这个库的问题，如果运行mrm的时候，将Amazon 编译出来的cjson.so.4 推进去，就会发生上述错误，但是如果用amlogic编译出来的cjson.so.4 ，程序就会crash。

现在我将运行不带mrm的avs ，然后用Amazon的cjson.so.4 代替 amlogic的 cjson.so.4 ，看是否会出现

```
terminate called after throwing an instance of 'std::system_error'
  what():  Invalid argument
```

我运行正常的avs ，将Amazon cjson.so.4的库推进去，会出现同样的问题，说明Amazon cjson.so.4的库是有问题的。但不能确定导致mrm 崩溃的就是amlogic 编译的cjson.so.4



我在 buildroot里把libcurl 回退到7.67 ，尝试了一次，avs是可以起来的，但是还是会crash 

### 开发账户MRM 功能不能选中

需要找Amazon询问，让他们给我们开通这个功能。



##  2022 12 23

 今天我发现 Amazon提供的很多开源库，和amlogic buildroot 里面的库的版本都对不上，我不知道我们的系统使用Amazon的提供的这些版本的库的时候会不会有问题。现在来尝试一下，先将板子里面 mrm要用到的库都先删掉，然后再将Amazon 的开源库全部推进来。看看会有什么问题。



我把Amazon所有的库都推进来之后 homeAPP起不来了

```
/usr/bin/audioservice/usr/bin/homeapp: : error while loading shared librarieserror while loading shared libraries: : libcjson.so.1libcjson.so.1: : cannot open shared object filecannot open shared object file: : No such file or directoryNo such file or directory
```

我把板子里所有cjon 库都删了，然后Amazon那边直推进去了cjson.so ,所以找不到。

我现在将amazon 的cjson.so 创建两个软链接，然后推到板端。

```
/usr/bin/audioservice: symbol lookup error: /usr/bin/audioservice: undefined symbol: snd_mixer_open

 hifi5a task1 is running! 

 hifi5a task2 is running! [341]
audioservice or homeapp is not alive. Try to restart them.
Stopping audioservice: killall: audioservice: no process killed
OK
Starting audioservice: OK
1711.143886 audioservice[17944] default ERR tid:17944 (as_config.c 56 in_AS_Config_CreateJsonRoot): cannot open the default json file /etc/save_audioservice.conf
audio_hw_device_get_module 
```

好像cjson文件audioservice 会解析失败，那我把我们系统下面的cjson 推进去，还是这样，应该不是cjson 的原因。好像是libasound.so.2这个库导致的，于是我又把amlogic 的libasound.so.2推进去，audioservice 就正常了。

然后用手机推avs 配置文件，出现在这样的问题

```
./SampleApp: /usr/lib/libasound.so.2: no version information available (required by /usr/lib/libLibSampleApp.so)
./SampleApp: /usr/lib/libasound.so.2: no version information available (required by /usr/lib/libLibSampleApp.so)
configFile /etc/MRMConfig.json
configFile /etc/AlexaClientSDKConfig.json
Running app with log level: NONE
terminate called after throwing an instance of 'std::system_error'
  what():  Invalid argument
```

现在将amlogic 的 libcurl.so.4 推进来。SampleAPP起来了，但是还没开始认证，audioservice程序crash 了

```
 hifi5a task2 is running! [473]
connect to avs server
client connection arrvied, fd: 7
msgDispatchWorker begin work
in avsClientRegisterServerMsgType
client register types[i] = 0
client register types[i] = 1
client register types[i] = 2
client register types[i] = 3
client register types[i] = 4
bitmap31
in register2Server
register server message succed
receive client register message, bitmap: 31
server receive register 0, fd: 7
serv[ 2371.126235@0]  asoc-aml-card auge_sound: TDM[1] Playback stop
er receive register 1, fd: 7
server receive register 2, fd: 7
server receive register 3, fd: 7
server receive register 4, fd: 7

=========>>>catch signal 11 (Segmentation fault) <<<=========
Dump stack start...
backtrace() returned 3 addresses
  [00] /usr/bin/audioservice() [0x27754]
  [01] /usr/bin/audioservice() [0x277f0]
  [02] /lib/libc.so.6(+0x271c0) [0xf73261c0]
Dump stack end...
[ 2371.133312@1]  aml_tdm_hw_setting_free(), disable mclk for tdm-1
[ 2371.133326@1]  tdm playback mute: 1, lane_cnt = 8
[ 2371.134013@1]  audio_ddr_mngr: frddrs[0] released by device fe330000.audiobus:tdm@1
2371.136575 homeapp[23564] default ERR tid:23798 (as_client.c 296 AS_Client_GetInputList): input_list JSON format is not correct
VAD state 0
VAD state 0
audioservice or homeapp is not alive. Try to restart them.
Stopping audioservice: killall: audioservice: no process killed
OK
Starting audioservice: OK
client close connection
[ 2372.690913@2]  SampleApp[23779]: unhandled exception: 11 DABT (lower EL), ESR 0x92000007, level 3 translation fault in libstdc++.so.6.0.28[f4fdc000+129000]
[ 2372.692084@2]  vma for f506f8d4:
[ 2372.692515@2]  f4fdc000-f5105000 r-xp 00000000 b3:08 347 
[ 2372.692522@2]  /lib/libstdc++.so.6.0.28
[ 2372.693154@2]  
[ 2372.693849@2]  vma for f5c78478:
[ 2372.694247@2]  f5c76000-f5c7a000 r-xp 00000000 b3:08 939 
[ 2372.694251@2]  /usr/lib/libSERVER.so
[ 2372.694919@2]  
[ 2372.695657@2]  vma for d:
[ 2372.695906@2]  00010000-00017000 r-xp 00000000 b3:08 513 
[ 2372.695911@2]  /usr/bin/SampleApp
[ 2372.696857@2]  
[ 2372.697206@2]  PC : 00000000f506f8d4  U
[ 2372.697681@2]  SP : 00000000c7cd7c40  U
[ 2372.698157@2]  FAR : 000000000000000d  U
[ 2372.698649@2]  CPU: 2 PID: 23779 Comm: SampleApp Tainted: G           O      5.4.180-06417-g1c44d21f59b1-dirty #2
[ 2372.699975@2]  Hardware name: Amlogic (DT)
[ 2372.700662@2]  pstate: 80880030 (Nzcv q T32 LE aif)
[ 2372.701040@2]  pc : 00000000f506f8d4
[ 2372.701483@2]  lr : 00000000f5c78478
[ 2372.701927@2]  sp : 00000000c7cd7c40
[ 2372.702372@2]  x12: 00000000f5c8a068 
[ 2372.702826@2]  x11: 00000000f5c8a1b4 x10: 0000000000000000 
[ 2372.703520@2]  x9 : 00000000f5c8a180 x8 : 00000000f5c8a1b4 
[ 2372.704213@2]  x7 : 00000000f5c8a1a0 x6 : 0000000000000007 
2372.700536 audioserv[ 2372.705127@2]  x5 : 00000000f5c8a19c x4 : 00000000f2920fd8 
[ 2372.705827@2]  x3 : 0000000000000001 x2 : 0000000000000000 
ice[[ 2372.706562@2]  x1 : 00000000f2920fd8 x0 : 00000000f2920fd8 
[ 2372.707259@2]  
[ 2372.707259@2]  PC: 0xf506f854:
[ 2372.707853@2]  f854  f853460d 6a0a3c0c 300c50c2 60a36a4b e90ef7ea 4620686b f8536023 69aa3c0c
[ 2372.709044@2]  f874  69eb50e2 692b60a3 f85360a3 696a3c0c 609a4423 602368ab 3c0cf853 50e268ea
[ 2372.709956@2]  f894  60632300 1d29bd38 f7ea4620 f7ebef8a bf00ef3e 4770300c f7ed300c bf00b897
[ 2372.711005@2]  f8b4  f7ec300c bf00bdb7 f7ec300c bf00bc31 b12368c3 689b4618 d1fb2b00 68434770
[ 2372.712094@2]  f8d4  429068da 4618d109 68da685b d0fa4282 429368c2 4618bf18 46184770 bf004770
[ 2372.713131@2]  f8f4  b12368c3 689b4618 d1fb2b00 68434770 429068da 4618d109 68da685b d0fa4282
[ 2372.714187@2]  f914  429368c2 4618bf18 46184770 bf004770 46026803 6843b91b 4298685b 6893d010
[ 2372.715226@2]  f934  4618b123 2b0068db 4770d1fb 68836850 d104429a 68404603 429a6882 4770d0f
```

好像是这个库有问题libc.so.6，我把Amazon 工具链里面的 libc.so.6  libstdc++.so.6.0.28库推进去

mrm 可以正常启动，但是还是会crash 在libcrypto.so.1.1.

我尝试将echo dot 从 amlogic device 所在的组中移除，发现mrm不会再crash了，即使刷新也不会。

我怀疑是由于开发者账户的MRM功能没有打开，所以才导致的这个问题。

但是给Amazon申请，他们说这个配置和MRM功能没关系。



## 2022 12 26

### 用backtrace 打印出crash 地址。

```
=========>>>catch signal 11 (Segmentation fault) <<<=========
Dump stack start...
backtrace() returned 3 addresses
  [00] /usr/lib/libLibSampleApp.so(+0x111ea8) [0xf7ef5ea8]
  [01] /usr/lib/libLibSampleApp.so(+0x111f5c) [0xf7ef5f5c]
  [02] /lib/libc.so.6(+0x271c0) [0xf52ee1c0]
Dump stack end...
[  192.538867@0]  asoc-aml-card auge_sound: Loopback Capture disable
```

用 arm-linux-gnueabihf-readelf -a libLibSampleapp.so 追溯到alexaClientSDK9avsCo，但是定位不到具体函数。

### 将sampleApp 默认的debug级别设置为 debug9

打印出log 目前要反馈给Amazon

根据打印信息，现在要Amazon回ticket，让他们帮忙看看，具体是什么地方出错了。

## 20221227 

https://scgit.amlogic.com/#/c/74124/6

根据之前同时做过的S420 MRM测试，我对比当前的avs_mrm 代码，除了 MRM SDK 升级之外其他内容都相同。可以暂时排除是avs_mrm的代码问题。



## 20221228

1. opensource里面对的开源库包含的 patch都已经打到src里了。

每一个开源库的src版本都是按照Makefile里面写的版本下载的

还有其他五个开源库默认是没有编译的，这些库都是一些基础库，我不确定版本不同会不会影响这个问题。因为amlogic内部是有这些基础库的。在mrm运行时并没有报相关库的错误。



## 20221230

libamazonwha.so 所在的库的的opensource 和opensource2.17只有cjon  不一样，然后我编译出来 使用前者编出来的cjson库，avs连不上server ,opensource2。17 编译出来的 cjson 库才可以。



现在的情况是，使用opensource2.17编译出来的库，加上Amazonwha 第一次提供的，然后用amlogic 的 libcurl.so.4替换opensource 的libcurl.so.4，mrm可以正常启动但会在刷新时crash。这是最好的情况。

在使用amazonwha 最新提供的amazonwha.so时，程序启动就连接不上server。没有第一次提供的Amazonwha 稳定。

但是总体来说，在mrm运行的过程中，除了mrm程序会crash的问题外，我觉得avs在运行时经常会连接不上server也是一个问题。



## 确定编译opensource 附加选项

编译amlogic buildroot a5 的代码

```
[14:44:53]>>> toolchain-external-custom  Building
[14:44:53]/usr/bin/gcc -O2 -I/mnt/fileroot/hongmei.kuang/work/a113x2/code4/output/a5_av400_a6432_release/host/include -DBR_CPU='"cortex-a9"' -DBR_ABI='"aapcs-linux"' -DBR_FPU='"neon"' -DBR_FLOAT_ABI='"hard"' -DBR_MODE='"arm"' -DBR_CROSS_PATH_SUFFIX='""' -DBR_CROSS_PATH_ABS='"/mnt/fileroot/hongmei.kuang/work/a113x2/code4/buildroot/../toolchain/gcc/linux-x86/arm/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf/bin"' -DBR_SYSROOT='"arm-linux-gnueabihf/sysroot"' -DBR_ADDITIONAL_CFLAGS='' -s -Wl,--hash-style=both toolchain/toolchain-wrapper.c -o /mnt/fileroot/hongmei.kuang/work/a113x2/code4/output/a5_av400_a6432_release/build/toolchain-external-custom/toolchain-wrapper
[14:44:53]>>> toolchain-external-custom  Installing to staging directory
```

找出相关信息，添加到envsetup.sh 脚本里，并编译。

```
export CROSS_COMPILER=arm-none-linux-gnueabihf-
export CXXFLAGS="-Wa,-mimplicit-it=arm"
export CFLAGS="-Wa,-mimplicit-it=arm"
export TARGET_CFLAGS='-mfpu=neon -mfloat-abi=hard -mcpu=cortex-a9'
```

将修改后的envsetup.sh 发给 amazon,让他们给我们新编一个MRM SDK.

鹏哥说这几个选项应该看在编译avs-sdk的时候是什么，所以重新编译avs-sdk,看编译的日志，搜索没有发现关于这四项的内容，所以我们的编译选项不需要加这些内容。反馈给Amazon，并等待回复。

## 2023 0102

我以为是getinfo.cpp 文件中的productid没有改成AmlogicA113X2Reference导致的，我修改之后，手机推送配置文件的app使用不了了（连接Amazon服务器错误），鹏哥说跟这个productid 没关系。

## S420 的提交记录
https://scgit.amlogic.com/#/c/74124/6

## MRMapp 运行命令

```
export LD_LIBRARY_PATH=/usr/lib/avs/
MRMApp <path_to_MRMConfig.json>  <log_level>
```


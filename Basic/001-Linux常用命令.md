# 常用Linux 命令
## 在.mk文件中加打印

```
$(warning ========${AVS_KWD_PATH}==========)
 
## 在CMakeLists中加打印
 message(FATAL_ERROR "===================${DSP_KEY_WORD_DETECTOR_INCLUDE_DIR}")
```

## 查找文件
将当前目录及其子目录下所有文件后缀为 .c 的文件列出来:
```
# find . -name "*.c"
```

查找SampleApp进程，排除grep本身
```
ps -fe|grep SampleApp |grep -v grep 1>/dev/null 2>&1
```
1>dev/null：这条命令的作用是将标准输出1重定向到/dev/null中。

/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。那么执行了>/dev/null之后，标准输出就会不再存在，没有任何地方能够找到输出的内容。

2>&1：这条命令用到了重定向绑定，采用&可以将两个输出绑定在一起。这条命令的作用是错误输出将和标准输出同用一个文件描述符，即错误输出将会和标准输出输出到同一个地方。

## 一些命令

测试能不能ping通外网的命令

```
wget www.google.com
```



```
i2cdump -f -y 2 0x40  板端查看版本号
```
3 4 下面即为版本号 01 22表示1.22版本

```
ps | grep Media    查看Media相关进程是否运行

pidof  ProcessName  命令用于查找指定名称的进程的进程号id号。
```


```
printk("[111111111] -- %s -- %d --\n ",__func__, __LINE__);
```
打印函数和行号

```
adnl.exe partition -p bootloader -F u-boot.bin.signed
```
单独烧录bootloader 

```
adnl.exe oem "reset"
```
重启

```
speaker-test -t pink
```
命令测试喇叭是否能够正常工作

```
md5sum filename
```
查看文件序列号

```
echo 0 > /proc/sys/kernel/printk 
kernel关闭打印
echo 9 > /proc/sys/kernel/printk
开启打印
```


用于调试的时候配网，可以直接在board/amlogic/mesona5_av400/rootfs/etc/wpa_supplicant.conf里面将
wifi的账号密码写进去。这样就不用再板端再配一次了。
直接在后面加上。编译烧录即可。

```
 network={
         ssid="Arttest_5G-2"
         psk="Arttest902"
         proto=RSN
         key_mgmt=WPA-PSK
         pairwise=CCMP
         group=CCMP
 }
```

## 解压命令

解压.tar.tgz文件

解压：tar zxvf FileName.tar.tgz

压缩：tar zcvf FileName.tar.tgz FileName

解压.tgz文件

解压：tar zxvf FileName.tgz

解压 tar.bz2 包

tar -xvf code.tar.bz2



## vscode 初始主题设置

```
{
    "remote.SSH.remotePlatform": {
        "hongmei.kuang": "linux"
    },
    "files.autoSave": "onFocusChange",
    "explorer.confirmDelete": false,
    "workbench.colorTheme": "Monokai",
    "editor.renderWhitespace": "all",
    "explorer.confirmDragAndDrop": false,
    "window.zoomLevel": 1,
    "window.commandCenter": true
}
```
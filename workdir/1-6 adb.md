# adb

查看板端进程，是如何实现的
vi S89usbgadget

## adbd

路径
./build/android-tools-4.2.2+git20130218/

## 重启adb 

```
$sudo adb kill-server
$sudo adb start-server
 adb devices
```

# adb概括

adb是client -server架构的，包含三部分：

1、client：用于机器开发，可以在shell中调用adb命令

2、server：作为后台进程同样用于开发机器，server负责管理client和运行于目标机器或者emulator的守护进程之间的通信，类似于一座桥。

3、deamon：运行于目标机或者emulator的守护进程。
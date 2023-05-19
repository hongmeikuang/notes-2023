# 当一个程序crash之后该如何定位

下面是avs_mrm 测试时，一个crash的现象

```
[  386.211967@2]  SampleApp[4768]: unhandled exception: 11 DABT (lower EL), ESR 0x92000007, level 3 translation fault in libcrypto.so.1.1[f5fd7000+1c6000]
[  386.213110@2]  vma for f613c894:
[  386.213458@2]  f5fd7000-f619d000 r-xp 00000000 b3:08 1063 
[  386.213466@2]  /usr/lib/libcrypto.so.1.1
[  386.214158@2]  
[  386.214842@2]  vma for f613d3c8:
[  386.215245@2]  f5fd7000-f619d000 r-xp 00000000 b3:08 1063 
[  386.215274@2]  /usr/lib/libcrypto.so.1.1
[  386.215923@2]  
[  386.216663@2]  vma for 48:
[  386.216979@2]  00010000-00017000 r-xp 00000000 b3:08 513 
[  386.216998@2]  /usr/bin/SampleApp
[  386.217632@2]  
[  386.218250@2]  PC : 00000000f613c894  U
[  386.218738@2]  SP : 00000000c15cc5c0  U
[  386.219214@2]  FAR : 0000000000000048  U
[  386.219708@2]  CPU: 2 PID: 4768 Comm: SampleApp Tainted: G           O      5.4.180-06417-g1c44d21f59b1-dirty #2
[  386.221008@2]  Hardware name: Amlogic (DT)
[  386.221482@2]  pstate: 20830010 (nzCv q A32 LE aif)
[  386.222087@2]  pc : 00000000f613c894
[  386.222529@2]  lr : 00000000f613d3c8
[  386.222974@2]  sp : 00000000c15cc5c0
```

## 计算crash地址

f613c894-f5fd7000 = 16 5894

## 使用命令查看crash库在上述地址处的函数是什么

```
 arm-linux-gnueabihf-readelf -a 库的路径
```

这条命令解析的库必须是当前使用的库，要不然没有意义。

如果找不到16 5894地址，则就找与16 5894最接近的
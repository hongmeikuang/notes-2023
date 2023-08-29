# 程序crash 定位

具体修改是：

```
diff --git a/src/Makefile b/src/Makefile
index 48c1efb..466b775 100755
--- a/src/Makefile
+++ b/src/Makefile
@@ -1,4 +1,6 @@
 #CC=${HOST_GCC}
+CFLAGS+=-g -O0 -rdynamic -funwind-tables
+export CFLAGS
 
 #export CC BUILD_DIR STAGING_DIR TARGET_DIR
 all:
@@ -10,8 +12,8 @@ all:
        -$(MAKE) -C utils install
        -$(MAKE) -C audio_hal all
        -$(MAKE) -C audio_hal install
-       -$(MAKE) -C test all
-       -$(MAKE) -C test install
+       # -$(MAKE) -C test all
+       # -$(MAKE) -C test install
 install:
        -$(MAKE) -C tinyalsa install    
 clean:
```

然后根据串口打印的堆栈地址定位发生错误的位置。

```
=========>>>catch signal 11 (Segmentation fault) <<<=========
Dump stack start...
backtrace() returned 9 addresses
  [00] /usr/bin/audioservice() [0x264f4]
  [01] /usr/bin/audioservice() [0x26590]
  [02] /lib/libc.so.6(+0x271c0) [0xf65611c0]
  [03] /usr/lib/libhalaudio.so(+0x2be3c) [0xf7ac4e3c]
  [04] /usr/bin/audioservice(halaudio_spdif_init+0x120) [0x21358]
  [05] /usr/bin/audioservice(HalAudio_Init+0x6c) [0x28678]
  [06] /usr/bin/audioservice(AS_Input_Init+0x210) [0x2519c]
  [07] /usr/bin/audioservice(main+0xc0) [0x17728]
  [08] /lib/libc.so.6(__libc_start_main+0x83) [0xf6551bc8]
Dump stack end...
```



```
arm-linux-gnueabihf-addr2line -e ./audio_hal/libhalaudio.so 0x2be3c
```

定位结果是：

```
/mnt/fileroot/hongmei.kuang/workdir/a113l2/code4/output/a4_ba400_a6432_release/build/aml-halaudio-0.0.1/audio_hal/audio_hw.c:9123
```




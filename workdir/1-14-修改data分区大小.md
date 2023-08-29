---
作用：修改data分区大小的操作步骤
date：2023-07-06
环境：以S400_K54为例
---

# 操作步骤

修改date分区的空间大小的修改有两处

```
/data # df
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/ubi0_0             245540    174476     71064  71% /rom
devtmpfs                497456         0    497456   0% /dev
tmpfs                   501720        68    501652   0% /dev/shm
tmpfs                   501720       168    501552   0% /tmp
tmpfs                   501720        72    501648   0% /run
/dev/ubi1_0             154136    154128         0 100% /data
overlayfs:/data/overlay/upper
                        154136    154128         0 100% /
```

当前data分区大小为154136/1024≈150Mb,但现在进行swuopdate 需要占用data分区大约200MB的空间，所以要将data分区改到200MB。kernel和Uboot修改如下：

## kernel

修改kernel\aml-5.4\arch\arm64\boot\dts\amlogic\axg_s400_v03.dts

```
diff --git a/arch/arm64/boot/dts/amlogic/axg_s400_v03.dts b/arch/arm64/boot/dts/amlogic/axg_s400_v03.dts
index 48686022b664..393ae222842e 100644
--- a/arch/arm64/boot/dts/amlogic/axg_s400_v03.dts
+++ b/arch/arm64/boot/dts/amlogic/axg_s400_v03.dts
@@ -1289,7 +1289,7 @@
                };
                partition@5 {
                        label = "system";
-                       reg = <0x0 0x11800000>;
+                       reg = <0x0 0xC800000>;
                };
                partition@6 {
                        label = "data";
```

system 分区<0x0 0x11800000>等于280MB，计算方法为：将0x11800000转成十进制=293,601,280，然后293,601,280÷1024÷1024=280。

现在data分区为150MB，但共需要200MB，所以就将system分区改小到200MB，这样data就可以增大80MB变成150+80=230MB。

## BootLoader bl33修改

```
diff --git a/board/amlogic/axg_s400_v1/axg_s400_v1.c b/board/amlogic/axg_s400_v1/axg_s400_v1.c
index b527f10a9f9..36c48e724b0 100644
--- a/board/amlogic/axg_s400_v1/axg_s400_v1.c
+++ b/board/amlogic/axg_s400_v1/axg_s400_v1.c
@@ -528,7 +528,7 @@ static struct mtd_partition normal_partition_info[] = {
     {
         .name = "system",
         .offset = 0,
-        .size = 280*SZ_1M,
+        .size = 200*SZ_1M,
     },
 #endif
        /* last partition get the rest capacity */
```

直接将280改成200 即可。

最后重编uboot 和kernel。
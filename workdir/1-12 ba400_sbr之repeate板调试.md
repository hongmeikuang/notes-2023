# ba400 sbr 没有声音调试

## 用 linein 或者HDMI源播放没有声音

### 先检查D622的音源输出是否正常

输出播放检查：

将播放的.mp3音源push到设备上，然后使用以下命令播放：

```
asplay 1.mp3 audioservice halaudio
```

```
asplay 1.mp3 auduioservice alsa
```

测试是否都能够出声，如果哪条通路不出声，则证明该通路配置有问题。

### 再检查D622音源输入是否正常

音频播放时通过tdma 将将PC的数据录入，然后在通过halaudio或者alsa输出播放的，如果音频输出没有问题的话，再来检查一下音频输入。

录制TDMA口输入的音频，可以用来查看D622输入的音频信号是否正常。

如果提示 Device or resource busy ，则先停止audioservice业务:

```
/etc/init.d/S90audioservice stop
```

录音命令为：

```
arecord -D hw:0,0 -c 8 -r 48000 -f S16_LE /data/tdma.wav
```

录完了之后用ls -l 命令查看录音文件的大小，如果是44的话，则证明文件是空，录不到数据。

## 检查配置信息

定位到问题之后，检查dts配置信息，将dts中的信息与原理图的信息核对。

用以下命令可查看设备上gpio的配置信息

```
mount -t debugfs none /sys/kernel/debug

cat sys/kernel/debug/pinctrl/fe000000.apb4\:pinctrl@4000-pinctrl-meson/pinmux-pins
cat sys/kernel/debug/pinctrl/fe330000.audiobus\:pinctrl-pinctrl-audio/pinmux-pins
```

将打印的信息与dts 中配置的信息，进行核对，看是否符合预期，检查是否是引脚被别的模块占用了。



---

参考资料

https://doc.amlogic.com/file/detail?type=1&id=18413
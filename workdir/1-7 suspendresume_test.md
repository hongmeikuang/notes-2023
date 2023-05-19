# suspend/resume test

# 测试脚本

```
cnt=0
while true; do
echo 0 > /sys/class/rtc/rtc0/wakealarm
echo +5 > /sys/class/rtc/rtc0/wakealarm
echo mem > /sys/power/state
let "cnt=$cnt+1"
echo ------------ cnt=$cnt -----------
echo 0 > /sys/class/rtc/rtc0/wakealarm
sleep 5
done &
```

测试自动休眠唤醒功能是否正常


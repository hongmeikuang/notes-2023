# wifi 连接过程

awk -F ':' '{print $2}' wifi.txt

1 先检查WiFi节点 

如果节点是0x8888，则将AML_WIFI_FLAG="TRUE"

2 如果 AML_WIFI_FLAG为真，则进行初始化

初始化干了什么：

stop_wifi_app

start-stop-daemon -K -o -p $PIDFILE1 1> /dev/null
start-stop-daemon -K -o -p $PIDFILE1 2> /dev/null
start-stop-daemon -K -o -p $PIDFILE1 3> /dev/null
start-stop-daemon -K -o -p $PIDFILE1 4> /dev/null

init_wifi_env

killall wpa_supplicant
killall hostapd
killall dnsmasq
killall dhcpcd

停止wpa_supplicant，hostapd，dnsmasq，dhcpcd这四个进程

usleep 200000

aml_wifi_mode_setup station

aml_wifi_mode_setup
判断是什么模式的，然后做不同的动作

3 parse_paras

将文件里的ssid 和password分别解析出来，如果密码长度小于8则警告

4 
	while true; do
		wpa_cli list_network
		if [ $? -eq 0 ]; then
			echo " wpa_suppliant is ready"
			break
		fi
		sleep 0.1
	done

## soundwave setup 集成

目前已经在av400 soundbar/speaker，s400 soundbar,s420上继承了sound wave network setup 功能。
在AV400 soundbar 和 S400 soundbar 设备上，可以通过长按source按键来启动soundwave_setup 程序；
在设备AV400 speaker 和S420设备上通过长按VolumeUp按键来启动soundwave_setup 程序。
相关gerrit提交如下：


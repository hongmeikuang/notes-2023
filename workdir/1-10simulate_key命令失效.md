操作记录

simulate_key.c 文件在audioservice/homeapp/目录下。

目前的情况是：

之前金荣的提交导致a5调不了声音了，按键和遥控都不行，simulate也不行，但是触发按键、遥控按键或者simulate命令是有打印日志的。simulate的日志和遥控按键的日志相同。

在金荣修改之后，通过按键和遥控可以正常调音量，但是simulate还是不可以，并且也没有日志打印了。

使用evtest工具，设备是收到了simulate事件的，

现在先homeapp里加打印。



目前发现，在金荣代码合进去之后，audioservice是可以正常工作的。换一个最新的代码目录编译一次看看。

最新的autobuild的镜像是不可以的

code7  repo sync 之后，make编译的镜像 simulate key命令是可以工作的，但之前make过。现在是直接make 的。

然后我以为是audioservice没有重新编译导致的，于是重新编译了audioservice。烧录，还是可以。
好像 BootLoader没有重新编译，我试试重新编译BootLoader。看看还能不能工作

重新编译了BootLoader ，simulate 还是可以的。

那就再重编kernel试试。
重编kernel之后 ，就不行了，现在通过回退来找哪个提交导致的。

commit 0afd600fe429ea5c78b1fa3bd7cf14fe1849f332 (HEAD -> amlogic-5.4-dev)
Author: Zelong Dong <zelong.dong@amlogic.com>

最后追溯到是这个提交导致的。
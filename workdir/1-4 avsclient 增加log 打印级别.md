# 操作步骤

因为avsclient中都是用printf来打印的，在日常调试的时候有很多打印看起来会非常不方便，所以增加avsclient 打印log的级别，方便调试。

## 修改内容

在我们的vendor/amlogic/aml-commonlib目录下有aml-log.c和aml-log.h 文件，这就是已经写好的log管理函数。我们可以直接用。

### buildroot的修改

直接在buildroot的avs-sdk.mk文件中加上要引用库及头文件。修改如下：

```
AVS_SDK_CONF_OPTS += -DAML_LOG_LIB_PATH=$(BUILD_DIR)/aml-commonlib-1.0/aml_log/libaml_log.so
AVS_SDK_CONF_OPTS += -DAML_LOG_INCLUDE_DIR=$(BUILD_DIR)/aml-commonlib-1.0/aml_log
```

### avsclient.c 以及CMakeLists的修改

avsclient.c包含上aml_log.h头文件

```
#include "aml_log.h"
```

在avsclient.c中将所有的printf替换成AML_LOGE或者AML_LOGI 分别对应

"ERR", "WARNING", "INFO", "DEBUG", "VERBOSE", "VV", "VVV", "", "", ""};

比如

```
AML_LOGE("avsClientWriteMsg2Server failed\n");
```

CMakeLists的修改，将buildroot里新增的头文件和库路径写进来，但是注意要写在add_executable()的前面。

```c
target_include_directories(avsclient PUBLIC
		"${AML_LOG_INCLUDE_DIR}")

target_link_libraries(avsclient
		"${AML_LOG_LIB_PATH}")
```

# log 级别调整

```
char *log_level[] = {"LOG_QUIET", "LOG_ERR",     "LOG_WARNING", "LOG_INFO",
                       "LOG_DEBUG", "LOG_VERBOSE", NULL};
```

比如想要调整audioservice的打印级别为debug级别，可以在板端输入如下命令

```
cd /tmp/
echo all:LOG_DEBUG > AML_LOG_audioservice
/etc/init.d/S90audioservice restart
```

调整homeapp 的打印级别，则在板端执行如下命令：

```
cd /tmp/
echo all:LOG_DEBUG > AML_LOG_homeapp
/etc/init.d/S90audioservice restart
```




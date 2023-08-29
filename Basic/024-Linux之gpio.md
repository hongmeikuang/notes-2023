---
title:linux之gpio
date:2023-06-21
---

本节主要是对`/sys/class/gpio/`目录下各个文件的解释

#  /sys/class/gpio/

在Linux系统中，`/sys/class/gpio/`目录用于控制和管理gpio(通用输入输出)引脚。每个gpio引脚在该目录下都有一个相应的目录，其名称为`gpioX`(X为引脚号)。

一下是`/sys/class/gpio/`目录下各个目录的作用：

1. `/sys/class/gpio/export`: 通过向该文件写入gpio引脚号，可以将指定的GPIO引脚导出(即使其在用户空间中可见)

   "即使在其在用户空间中可见"指的是，通过导出"export"GPIO引脚，你可以在用户空间中访问和控制这些引脚。导出之后，`/sys/class/gpio/gpioX`目录下的文件（如‘direction’，‘value’等）可以被用户程序读取和写入，以便配置和操作GPIO引脚的状态和值。

2. `/sys/class/gpio/unexport`: 通过向该文件写入GPIO引脚号，可以取消导出指定的GPIO引脚，将其从用户空间中移除。
3. `/sys/class/gpio/gpioX`(X为引脚号)： 每个gpio引脚都有一个以`gpioX`命名的目录，在该目录下，可以找到与该引脚相关文件和目录，包括：
   - `direction`:用于设置或读取GPIO引脚的方向，可以输入("in")或输出("out")。
   - `value`:用于设置或读取GPIO引脚的值。如果引脚被配置为输出，你可以将0或者1写入该文件来控制引脚的状态，如果引脚被配置为输入，你可以读取该文件来获取引脚的当前状态。
   - `edge`:用于配置gpio引脚的边沿触发方式。边沿触发可以是上升沿("rising")、下降沿("failing")、上升和下降沿("both")或禁用边沿检测("none")。


## cmake学习笔记

1、编译一个文件的CMakeLists.txt编写（编译main.c)

CMakeLists.txt

  cmake_minimum_required (VERSION 2.8)

  project(demo)

  add_executable(main main.c)

  第一行：cmake最低版本要求为2.8

  第二行：该工程名叫demo

  第三行：最终生成的elf文件名为main,其所依赖的源文件是main.c

  cmake .  运行

  make    生成main

  ./main    执行程序

 2、 同一目录下多个源文件编写CMakeLists.txt
(main.c调用的函数在testFunc.c中)

 只需要把上面的CMakeLists.txt的第三行改为

 add_executable(main main.c testFunc.c)

 3、但是像步骤2这样写，如果函数很多编写效率就会大大降低，所以使用aux_source_directory把当前目录下的源文件列表存放在变量SRC_LIST中，然后，add_executable里调用SRC_LIST

 CMakeLists.txt编写为：

 cmake_minimum_required (VERSION 2.8)

  project(demo)


aux_source_directory(. SRC_LIST)

add_executable(main ${SRC_LIST})

再次执行cmake 和make 并运行make

set_source_directory 的缺点就是会将指定目录下所有的源文件全都加进去，可能会包括一些不需要的文件

4、不同目录下多个源文件

CMakeLists.txt编写为：

 cmake_minimum_required (VERSION 2.8)

  project(demo)

include_directories (test_func test_func1)

aux_source_directory(test_func SRC_LIST)

aux_source_directory(test_func1 SRC_LIST1)

add_executable(main main.c ${SRC_LIST} ${SRC_LIST1})

在这里命令include_directories的作用是，向工程添加多个头文件的搜索路径，路径之间用空格分隔。
test_func 和test_func1是两个文件夹，分别存放各自的.c函数和头文件。

5、正规组织结构
正规点来说，一般会将源文件都放在src文件夹下，头文件放在include 文件夹下，生成的对象文件放在build目录下，最终输出的elf 文件会放在bin目录下

编写CMakeLists.txt文件
此事需要两个CmakeLists.txt文件，一个在最外层目录，一个在src目录下。

最外层目录下的CMakeLists.txt编写为：

cmake_minimum_required (VERSION 2.8)

project(demo)

add_subdirectory (src)

add_sundirectory （）的作用是向当前工程添加存放源文件的子目录

src目录下的CMakeLists.txt编写如下：

aux_source_directory(. SRC_LIST)


include_dirctories (../include)

add_executable (main ${SRC_LIST})

set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

set的意思是用来定义变量，在这里的意思是把存放elf文件的位置设置为工程根目录下的bin目录。

为了保证生成的附带文件存放在build目录下，而不跟源码混合在一起，所以先切换到build 目录下，然后cmake ..   ,然后再运行make

运行完之后，切换到bin目录下，运行elf生成文件

6、动态库（.so)和静态库（.a)的编译控制

测试.c和.h文件存放在testFunc文件夹下，创建build 和lib文件夹，编写CMakeLists.txt文件

cmake_minimum_required (VERSION 3.5)

project (demo)

set (SRC_LIST ${PROJECT_SOURCE_DIR}/testFunc/testFunc.c)

add_library (testFunc_shared SHARED ${SRC_LIST})
add_library (testFunc_static STATIC ${SRC_LIST})

set_target_properties (testFunc_shared PROPERTIES OUTPUT_NAME "testFunc")
set_target_properties (testFunc_static PROPERTIES OUTPUT_NAME "testFunc")

set (LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

add_library: 生成动态库或静态库(第1个参数指定库的名字；第2个参数决定是动态还是静态，如果没有就默认静态；第3个参数指定生成库的源文件)
set_target_properties: 设置最终生成的库的名称，还有其它功能，如设置库的版本号等等
LIBRARY_OUTPUT_PATH: 库文件的默认输出路径，这里设置为工程目录下的lib目录

切换到build 目录下cmake .. && make

切换到bin目录下就会发现动态库和静态库已经生成

7、链接静态库和动态库

重新建一个工程目录，然后把上节生成的库拷贝过来，然后在在工程目录下新建src目录、bin目录、build目录、bin目录和testFunc目录，，在src目录下添加一个main.c，testFunc目录下新建inc和Lib目录，inc存放头文件，lib里面存放上一步已生成的动态库和静态库。CMakeLists.txt文件在最外层，编写为：

cmake_minimum_required (VERSION 3.5)

project (demo)


set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

set (SRC_LIST ${PROJECT_SOURCE_DIR}/src/main.c)

# find testFunc.h
include_directories (${PROJECT_SOURCE_DIR}/testFunc/inc)

find_library(TESTFUNC_LIB testFunc HINTS ${PROJECT_SOURCE_DIR}/testFunc/lib)

add_executable (main ${SRC_LIST})

target_link_libraries (main ${TESTFUNC_LIB})

find_library: 在指定目录下查找指定库，并把库的绝对路径存放到变量里，其第一个参数是变量名称，第二个参数是库名称，第三个参数是HINTS，第4个参数是路径，其它用法可以参考cmake文档
target_link_libraries: 把目标文件与库文件进行链接

8、添加控制选项

在编译代码时只编译一些指定的源码，可以使用cmake的option命令，主要遇到的情况分为2种：

本来要生成多个bin或库文件，现在只想生成部分指定的bin或库文件
对于同一个bin文件，只想编译其中部分代码（使用宏来控制）

## 第一种情况：
- bin
- build 
- CMakeLists.txt
- src
  - CMakeLists.txt
  - main1.c
  - main2.c

最外层CMakeLists.txt：

cmake_minimum_required(VERSION 3.5)

project(demo)

option(MYDEBUG "enable debug compilation" OFF)

set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

add_subdirectory(src)

在这里option命令，其第一个参数是这个option的名字，第二个参数是字符串，用来描述这个option是来干嘛的，第三个是option的值，ON或OFF，也可以不写，不写就是默认OFF。

src目录下的CMakeLists.txt：

cmake_minimum_required (VERSION 3.5)

add_executable(main1 main1.c)

if (MYDEBUG)
    add_executable(main2 main2.c)
else()
    message(STATUS "Currently is not in debug mode")    
endif()

if-else来根据option来决定是否编译main2.c

然后cd到build目录下输入cmake .. && make就可以只编译出main1，如果想编译出main2，就把MYDEBUG设置为ON，再次输入cmake .. && make重新编译。

每次想改变MYDEBUG时都需要去修改CMakeLists.txt，有点麻烦，其实可以通过cmake的命令行去操作，例如我们想把MYDEBUG设置为OFF，先cd到build目录，然后输入cmake .. -DMYDEBUG=ON，这样就可以编译出main1和main2 （在bin目录下）

## 第二种情况

- bin
- build 
- CMakeLists.txt
- main.c
  
main.c:

#include <stdio.h>

int main(void)
{
#ifdef WWW1
    printf("hello world1\n");
#endif    

#ifdef WWW2     
    printf("hello world2\n");
#endif

    return 0;
}

通过定义宏来控制打印的信息，我们CMakeLists.txt内容如下，

cmake_minimum_required(VERSION 3.5)

project(demo)

set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

option(WWW1 "print one message" OFF)
option(WWW2 "print another message" OFF)

if (WWW1)
    add_definitions(-DWWW1)
endif()

if (WWW2)
    add_definitions(-DWWW2)
endif()

add_executable(main main.c)

这里把option的名字保持和main.c里的宏名称一致，这样更加直观，也可以选择不同的名字。通过与add_definitions()的配合，就可以控制单个bin文件的打印输出了

cd到build目录下执行cmake .. && make，然后到bin目录下执行./main，可以看到打印为空，
接着分别按照下面指令去执行，然后查看打印效果，

cmake .. -DWWW1=ON -DWWW2=OFF && make
cmake .. -DWWW1=OFF -DWWW2=ON && make
cmake .. -DWWW1=ON -DWWW2=ON && make

## 如果option有变化，要么删除上次执行cmake时产生的缓存文件，要么把所有的option都显式的指定其值

https://blog.csdn.net/whahu1989/article/details/82078563?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165788454916782395336376%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165788454916782395336376&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-82078563-null-null.142^v32^experiment_2_v1,185^v2^tag_show&utm_term=cmake&spm=1018.2226.3001.4187
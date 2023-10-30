# Makefile 学习笔记

# makefile介绍

## Makefile简介

当时用make时，工程中以下文件将会被编译

    1、所有源文件没被编译过，则编译各个源文件并进行链接，生成最后的可执行文件
    
    2、在上次执行make后被修改过的文件在本次执行make时将会被重新编译
    
    3、头文件在上次执行make之后被修改，则包含此头文件的所有源文件在本次执行make后将会被重新编译

## makefile 的规则介绍

TARGET... : PREREQUISITES...
    COMMAND 

target：规则的目标。

prerequisites：规则的依赖。生成规则目标所需要的文件名列表。通常一个目标依
赖于一个或者多个文件。

command：规则的命令行。是规则所要执行的动作（任意的 shell 命令或者是可在
shell 下执行的程序）。它限定了 make 执行这条规则时所需要的动作。

一个规则可以有多个命令行，每一条命令占一行。注意：每一个命令行必须以[Tab]
字符开始，[Tab]字符告诉 make 此行是一个命令行。make 按照命令完成相应的动作。
这也是书写 Makefile 中容易产生，而且比较隐蔽的错误。

make 程序根据规则的依赖关系，决定是否执行规则所定义的命令的过程我们称之
为执行规则。

## 简单示例

此例子由3个头文件和8个C文件组成。
我们将书写一个简单的Makefile，来描述如何创建最终的可执行文件“edit”，此可执行
文件依赖于8个C源文件和3个头文件。Makefile文件的内容如下：

        #sample Makefile
        edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
        main.o : main.c defs.h
        cc -c main.c
        kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
        command.o : command.c defs.h command.h
        cc -c command.c
        display.o : display.c defs.h buffer.h
        cc -c display.c
        insert.o : insert.c defs.h buffer.h
        cc -c insert.c
        search.o : search.c defs.h buffer.h
        cc -c search.c
        files.o : files.c defs.h buffer.h command.h
        cc -c files.c
        utils.o : utils.c defs.h
        cc -c utils.c
        clean :
        rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o 

- 首先书写时，可以将一个较长行使用反斜线（\）来分解为多行，这样可以使我们
的Makefile书写清晰、容易阅读理解。

- 但需要注意：反斜线之后不能有空格（这也是大
家最容易犯的错误，错误比较隐蔽）。

- 在这个Makefile中，我们的目标（target）就是可执行文件“edit”和那些.o文件
（main.o,kbd.o….）；依赖（prerequisites）就是冒号后面的那些 .c 文件和 .h文件。
所有的.o文件既是依赖（相对于可执行程序edit）又是目标（相对于.c和.h文件）。命令
包括 “cc –c maic.c”、“cc –c kbd.c”

- 在描述依赖关系行之下通常就是规则的命令行（存在一些些规则没有命令行），命令行定义了规则的动作（如何根据依赖文件来更新目标文件）。命令行必需以[Tab]键开始，以和Makefile其他行区别。就是说所有的命令行必需以[Tab] 字符开始，但并不是所有的以[Tab]键出现行都是命令行。但make程序会把出现在第一条规则之后的所有以[Tab]字符开始的行都作为命令行来处理。（记住：make程序本身并不关心命令是如何工作的，对目标文件的更新需要你在规则描述中提供正确的命令。“make”程序所做的就是当目标程序需要更新时执行规则所定义的命令）。

- 目标“clean”没有任何依赖文件，它只有一个目的，就是通过这个目标名来执行它所定义的命令。Makefile中把那些没有任何依赖只有执行动作的目标称为“伪目标”（phony targets）。需要执行“clean”目标所定义的命令，可在shell下输入：make clean。

##  makefile 如何工作

## 指定变量

当一个文件列表多次出现时，为了避免修改列表时，忘记修改全部列表发生错误，所以可以指定变量来代替该多次出现的文件列表；

为了避免这个问题，在实际工作中大家都比较认同的方法是，使用一个变量“objects”、“OBJECTS”、“objs”、“OBJS”、“obj”或者“OBJ”来作为所有的.o 文件的列表的替代。在使用到这些文件列表的地方，使用此变量来代替。在上例的 Makefile
中我们可以添加这样一行：

        objects = main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o 

“objects”作为一个变量，它代表所有的.o文件的列表。在定义了此变量后，我们就可以在需要使用这些.o文件列表的地方使用“$(objects)”来表示它，从而减少错误可能。

因此上例的规则就可以这样写：

        objects = main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
        edit : $(objects)
        cc -o edit $(objects)
        …….
        …….
        clean :
        rm edit $(objects) 

##  makefile自动推导规则

在Makefile中我们只需要给出需要重建的目标文件名（一个.o文件），make会自动为这个.o文件寻找合适的依赖文件（对应的.c文件。对应是指：文件名除后缀外，其余都相同的两个文件），而且使用正确的命令来重建这个目标文件。

对于上边的例子，此默认规则就使用命令“cc -c main.c -o main.o”来创建文件“main.o”。对一个目标文件是“N.o”，倚赖文件是“N.c”的规则，完全可以省略其规则的命令行，而由make自身决定使用默认命令。此默认规则称为make的隐含规则

因此上边的例子就可以
以更加简单的方式书写，我们同样使用变量“objects”。Makefile 内容如下：

        # sample Makefile
        objects = main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    
        edit : $(objects)
        cc -o edit $(objects)
    
        main.o : defs.h
        kbd.o : defs.h command.h
        command.o : defs.h command.h
        display.o : defs.h buffer.h
        insert.o : defs.h buffer.h
        search.o : defs.h buffer.h
        files.o : defs.h buffer.h command.h
        utils.o : defs.h
    
        .PHONY : clean
        clean :
        rm edit $(objects) 

## 另类风格的makefile

上一节中我们提到过，Makefile 中，所有的.o 目标文件都可以使用隐含规则由 make自动重建，我们可以根据这一点书写更加简洁的 Makefile。而且在这个 Makefile 中，我们是根据依赖而不是目标对规则进行分组。形成另外一种风格的 Makefile。实现如下：

        #sample Makefile
        objects = main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
        edit : $(objects)
        cc -o edit $(objects)
        $(objects) : defs.h
        kbd.o command.o files.o : command.h
        display.o insert.o search.o files.o : buffer.h 

同时把多个目标文件的依赖放在同一个规则中进行描述（一个规则中含有多个目标文件），这样导致规则定义不明了，比较混乱。建议大家不要在 Makefile 中采用这种方式了书写。否则后期维护将会是一件非常痛苦的事情。

**书写规则建议的方式是：单目标，多依赖。就是说尽量要做到一个规则中只存在一个目标文件，可有多个依赖文件。尽量避免使用多目标，单依赖的方式。这样书写的好处是后期维护会非常方便，而且这样做会使 Makefile 会更清晰、明了。**

## 清理工作目录过程

规则除了完成源代码编译之外，也可以完成其它任务。例如：前边提到的为了实现清除当前目录中编译过程中产生的临时文件（edit 和哪些.o 文件）的规则：

        clean :
        rm edit $(objects)

在实际应用时，我们把这个规则写成如下稍微复杂一些的样子。以防止出现始料未及的情况。

        .PHONY : clean
        clean :
        -rm edit $(objects)

这两个实现有两点不同： 

1. 通过“.PHONY”特殊目标将“clean”目标声明为伪目标。避免当磁盘上存在一个名为“clean”文件时，目标“clean”所在规则的命令无
法执行。

2. 在命令行之前使用“-”，意思是忽略命令
“rm”的执行错误。

# makefile 总述

## makefile 的内容

在一个完整的 Makefile 中，包含了 5 个东西：显式规则、隐含规则、变量定义、指示符和注释。

- 显式规则:它描述了在何种情况下如何更新一个或者多个被称为目标的文件（Makefile 的目标文件）。书写 Makefile 时需要明确地给出目标文件、目标的依赖文件列表以及更新目标文件所需要的命令

- 隐含规则:它是make根据一类目标文件（典型的是根据文件名的后缀）而自动推导出来的规则。make根据目标文件的名，自动产生目标的依赖文件并使用默认的命令来对目标进行更新

- 变量定义:使用一个字符或字符串代表一段文本串，当定义了一个变量以后，Makefile后续在需要使用此文本串的地方，通过引用这个变量来实现对文本串的使用。第一章的例子中，我们就定义了一个变量“objects”来表示一个.o文件列表。

- 指示符:指示符指明在 make 程序读取 makefile 文件过程中所要执
行的一个动作。其中包括：

  - 读取一个文件，读取给定文件名的文件，将其内容作为makefile文件的一部分。
  - 决定（通常是根据一个变量的得值）处理或者忽略Makefile中的某一特定部分。
  - 定义一个多行变量
  
- 注释:：Makefile 中“#”字符后的内容被作为是注释内容（和 shell 脚本一样）处理。如果此行的第一个非空字符为“#”，那么此行为注释行。注释行的结尾如果存在反斜线（\），那么下一行也被作为注释行。一般在书写 Makefile时推荐将注释作为一个独立的行，而不要和 Makefile 的有效行放在一行中书写。当在 Makefile 中需要使用字符“#”时，可以使用反斜线加“#”（\#）来实现（对特殊字符“#”的转义），其表示将“#”作为一字符而不是注释的开始标志。

## makefile 文件名

常见以“makefile","Makefile”，“GNUmakefile"

一般建议以"makefile","Makefile"命名，最好是”Makefile",首字母大写比较显眼。

当 makefile 文件的命名不是这三个任何一个时，需要通过 make 的“-f”或者“--file”选项来指定 make 读取的 makefile 文件。给 make 指定 makefile 文件的格式为：“-fNAME”或者“—file=NAME”，它指定文件“NAME”作为执行 make 时读取的 makefile文件。也可以通过多个“-f”或者“--file”选项来指定多个需要读取的 makefile 文件，

多个 makefile 文件将会被按照指定的顺序进行链接并被 make 解析执行。当通过“-f”或者“--file”指定 make 读取 makefile 的文件时，make 就不再自动查找这三个标准命名的 makefile 文件。

        注释：通过命令指定目标使用make的隐含规则：
        当前目录下不存在以“GNUmakefile”、“makefile”、“Makefile”命名的任何文件，
        1. 当前目录下存在一个源文件foo.c的，我们可以使用“make foo.o”来使用make的隐含规
        则自动生成foo.o。当执行“make foo.o”时。我们可以看到其执行的命令为：
        cc –c –o foo.o foo.c
        之后，foo.o将会被创建或者更新。
        2. 如果当前目录下没有foo.c文件时，就是make对.o文件目标的隐含规则中依赖文件不存在。
        如果使用命令“make foo.o”时，将回到到如下提示：
        make: *** No rule to make target ‘foo.o’. Stop.
        3. 如果直接使用命令“make”时，得到的提示信息如下：
        make: *** No targets specified and no makefile found. Stop.

## 包含其他makefile 文件

makefile包含其他文件时所需要使用的关键字是  include

“include”指示符告诉 make 暂停读取当前的 Makefile，而转去读取“include”
指定的一个或者多个文件，完成以后再继续当前 Makefile 的读取。Makefile 中指示符“include”书写在独立的一行，其形式如下：

        include FILENAMES... 

FILENAMES 是 shell 所支持的文件名（可以使用通配符）。

不能以tab开头，否则会被当成命令处理

指示符“include”所在的行可以一个或者多个空格（make程序在处理时将忽略这
些空格）开始，指示符“include”和文件名之间、多个文件之间使用空格或者[Tab]键隔开。行尾的空白字符在处理时被忽略。使用指示符包含进来的Makefile中，如果存在变量或者函数的引用。它们将会在包含它们的Makefile中被展开。

通常指示符“include”用在以下场合：

1. 有多个不同的程序，由不同目录下的几个独立的Makefile来描述其重建规则。它们需要使用一组通用的变量定义（可参考 6.5 如何设置变量 一节）或者模式
规则。通用的做法是将这些共同使用的变量或者模式规则定义在一个文件中（没有具体的文件命名限制），在需要使用的Makefile中使用指示符“include”来包含此文件。

2. 当根据源文件自动产生依赖文件时；我们可以将自动产生的依赖关系保存在另
外一个文件中，主Makefile使用指示符“include”包含这些文件。这样的做法
比直接在主Makefile中追加依赖文件的方法要明智的多。其它版本的make已经
使用这种方式来处理


- 当在这些目录下都没有找到“include”指定的文件时，make将会提示一个包含文件未找到的告警提示，但是不会立刻退出。而是继续处理Makefile的后续内容。当完成读取整个Makefile后，make将试图使用规则来创建通过指示符“include”指定的但未找到的文件，当不能创建它时（没有创建这个文件的规则），make将提示致命错误并退出。会输出类似如下错误提示：

        Make： *** No rule to make target ‘<filename>’. Stop 

- 通常我们在 Makefile 中可使用“-include”来代替“include”，来忽略由于包含文件不存在或者无法创建时的错误提示（“-”的意思是告诉 make，忽略此操作的错误。make 继续执行）。像下边那样：

        -include FILENAMES... 

“include "和“-include"比较：

include filenames：make 程序处理时，如果“FILENAMES”列表中的任何一个文件不能正常读取而且不存在一个创建此文件的规则时 make 程序将会提示错误并退出。

-inlude filenames:make 程序处理时，如果“FILENAMES”列表中的任何一个文件不能正常读取而且不存在一个创建此文件的规则时 make 程序将会提示错误并退出。

也可以使用”sinclude"代替“-include”

## 变量MAKEFILES

如果在当前环境定义了一个“MAKEFILES”环境变量，make执行时首先将此变量的值作为需要读入的Makefile文件，多个文件之间使用空格分开。量的值作为需要读入的Makefile文件，多个文件之间使用空格分开。

与include的区别：

1. 环境变量指定的 makefile 文件中的“目标”不会被作为 make 执行的“终极目标”。

如果在 make 的工作目录下没有一个名为“Makefile”、“makefile”或
者“GNUmakefile”的文件，make 同样会提示

        “make: *** No targets specified and no makefile found. Stop.”；而在 

make 的工作目录下存在这样一个文件（“Makefile”、“makefile”或者“GNUmakefile”），那么 make 执行时的“终极目标”就是当前目录下这个文件中所定义的“终极目标”。

2. 环境变量所定义的文件列表，在执行 make 时，如果不能找到其中某一个文件（不存在或者无法创建）。make 不会提示错误，也不退出。就是说环境变量“MAKEFILES”定义的包含文件是否存在不会导致 make 错误（这是比较隐蔽的地方）。

3. make 在执行时，首先读取的是环境变量“MAKEFILES”所指定的文件列表，之后才是工作目录下的 makefile 文件，“include”所指定的文件是在 make 发现此关键字的时、暂停正在读取的文件而转去读取“include”所指定的文件。

## 变量MAKEFILE_LIST

make 程序在读取多个 makefile 文件时，包括由环境变量“MAKEFILES”指定、命令行指、当前工作下的默认的以及使用指示符“include”指定包含的，在对这些文件进行解析执行之前 make 读取的文件名将会被自动依次追加到变量“MAKEFILE_LIST”的定义域中。

## 其他特殊变量

“.VARIABLES”，此变量不能通过任何途经给它赋值。它被展开为一个特定的值。

##  makefile 文件的重建

make 在读入所有 makefile 文件之后，首先将所读取的每个 makefile 作为一个目
标，寻找更新它们的规则。如果存在一个更新某一个 makefile 文件明确规则或者隐含规则，就去更新对应的 makefile 文件。完成对所有的 makefile 文件的更新之后，如果之前所读取任何一个 makefile 文件被更新，那么 make 就清除本次执行的状态重新读取一遍所有的 makefile 文件（此过程中，同样在读取完成以后也会去试图更新所有的已经读取的 makefile 文件，但是一般这些文件不会再次被重建，因为它们在时间戳上已经是最新的）。读取完成以后再开始解析已经读取的 makefile 文件并开始执行必要的动作。

## 重载另一个makefile

有些情况下，存在两个比较类似的 makefile 文件。其中一个（makefile-A）需要使
用另外一个（makefile-B）中所定义的变量和规则。通常我们会想到在“makefile-A”
中使用指示符“include”包含“mkaefile-B”来达到目的。但使用这种方式，如果在两
个 makefile 文件中存在相同目标，而在不同的文件中其描述规则使用不同的命令。这样，相同的目标文件就同时存在两个不同的规则命令，这是 makefile 所不允许的。遇到这种情况，使用指示符“include”显然是行不通的。GNU make 提供另外一种途径来实现此目的。具体的做法如下：

如果在当前makefile文件中不能找到重建一个目标的规则时，就使用“所有匹配模式”所在的规则来重建这个目标。

##  make 如何解析makefile文件

两个阶段：

第一阶段：读取所有的 makefile 文件（包括“MAKIFILES”变量指定的、指示符
“include”指定的、以及命令行选项“-f(--file)”指定的 makefile 文件），内建所有的变量、明确规则和隐含规则，并建立所有目标和依赖之间的依赖关系结构链表。

在第二阶段：根据第一阶段已经建立的依赖关系结构链表决定哪些目标需要更新，
并使用对应的规则来重建这些目标。

## 变量取值

        变量定义解析的规则如下：
        IMMEDIATE = DEFERRED
        IMMEDIATE ?= DEFERRED
        IMMEDIATE := IMMEDIATE
        IMMEDIATE += DEFERRED or IMMEDIATE
        define IMMEDIATE
        DEFERRED
        Endef 

当变量使用追加符（+=）时，如果此前这个变量是一个简单变量（使用 :=定义的）
则认为它是立即展开的，其它情况时都被认为是“延后”展开的变量。

##  条件语句

所有使用到条件语句在产生分支的地方，make 程序会根据预设条件将正确地分支
展开。就是说条件分支的展开是“立即”的。其中包括：“ifdef”、“ifeq”、“ifndef”和“ifneq”所确定的所有分支命令。

## 3.9.3 规则的定义

所有的规则在 make 执行时，都按照如下的模式展开：

        IMMEDIATE : IMMEDIATE ; DEFERRED
        DEFERRED

其中，规则中目标和依赖如果引用其他的变量，则被立即展开。而规则的命令行中的变量引用会被延后展开。此模板适合所有的规则，包括明确规则、模式规则、后缀规则、静态模式规则。

## 总结

make 的执行过程如下：

1. 依次读取变量“MAKEFILES”定义的 makefile 文件列表
   
2. 读取工作目录下的 makefile 文件（根据命名的查找顺序“GNUmakefile”，
“makefile”，“Makefile”，首先找到那个就读取那个）

3. 依次读取工作目录 makefile 文件中使用指示符“include”包含的文件
   
4. 查找重建所有已读取的 makefile 文件的规则（如果存在一个目标是当前读取的
某一个 makefile 文件，则执行此规则重建此 makefile 文件，完成以后从第一步
开始重新执行）

5. 初始化变量值并展开那些需要立即展开的变量和函数并根据预设条件确定执行
分支

6. 根据“终极目标”以及其他目标的依赖关系建立依赖关系链表
   
7. 执行除“终极目标”以外的所有的目标的规则（规则中如果依赖文件中任一个
文件的时间戳比目标文件新，则使用规则所定义的命令重建目标文件）

8. 执行“终极目标”所在的规则

  **执行一个规则的过程是这样的**：

对于一个存在的规则（明确规则和隐含规则）首先，make程序将比较目标文件和所有的依赖文件的时间戳。如果目标的时间戳比所有依赖文件的时间戳更新（依赖文件在上一次执行make之后没有被修改），那么什么也不做。否则（依赖文件中的某一个或者全部在上一次执行make后已经被修改过），规则所定义的重建目标的命令将会被执行。这就是make工作的基础，也是其执行规制所定义命令的依据。

# makefile 的规则

## makefile 规则

“$<”表示规则中的第一个依赖文件，

“$@”表示规则中的目标文件

## 一，Shell 是一种脚本语言

有的编程语言，如 C/C++、Pascal、Go语言、汇编等，必须在程序运行之前将所有代码都翻译成二进制形式，也就是生成可执行文件，用户拿到的是最终生成的可执行文件，看不到源码。

这个过程叫做 **编译（Compile）**，这样的编程语言叫做 **编译型语言**，完成编译过程的软件叫做编译器（Compiler）。

而有的编程语言，如 Shell、JavaScript、Python、PHP等，需要一边执行一边翻译，不会生成任何可执行文件，用户必须拿到源码才能运行程序。程序运行后会即时翻译，翻译完一部分执行一部分，不用等到所有代码都翻译完。

这个过程叫做 **解释**，这样的编程语言叫做 **解释型语言** 或者 **脚本语言（Script）**，完成解释过程的软件叫做解释器。
站在这个角度讲，Shell 也是一种编程语言，它的编译器（解释器）是 Shell 这个程序。

我们平时所说的 Shell，**有时候是指连接用户和内核的这个程序，有时候又是指 Shell 编程**。

编译型语言的优点是执行速度快、对硬件要求低、保密性好，适合开发操作系统、大型应用程序、数据库等。
脚本语言的优点是使用灵活、部署容易、跨平台性好，非常适合Web开发以及小工具的制作。

## 二，Shell 与 Linux 运维工程师（OPS）
Linux被广泛地用于服务器端，OPS 的主要工作就是维护应用的运行环境，包括安装操作系统／部署和维护代码的运行依赖／修复漏洞防止服务器被攻击／监控服务器压力，别让服务器宕机／分析日志，及时发现代码或者环境的问题。

因为 OPS 面对的可能是成千上万台的服务器，很难通过人工手段逐个维护，所以通过一份代码让指定服务器完成相同工作就非常重要。例如服务的监控、代码快速部署、服务启动停止、数据备份、日志分析等。

Shell 脚本很适合处理纯文本类型的数据，而 Linux 中几乎所有的配置文件、日志文件（如 NFS、Rsync、Httpd、Nginx、MySQL 等），以及绝大多数的启动文件都是纯文本类型的文件。

在运维过程中，若把上述文件内容比作珍珠，那么 Shell 就是将珍珠连接成手链的绳索。

**Python** 不但可以用于脚本程序开发，也可以实现 Web 程序开发（知乎、豆瓣、YouTube、Instagram 都是用 Python 开发），甚至还可以实现软件的开发、游戏开发、大数据开发、移动端开发。

*每一个运维人员在熟悉了 Shell 之后，都应该再学习 Python 语言*。

Python 语言的优势在于开发复杂的运维软件、Web 页面的管理工具和Web业务的开发（例如 CMDB 自动化运维平台、跳板机、批量管理软件 SaltStack、云计算 OpenStack 软件）等。

**Shell 脚本的优势在于处理偏操作系统底层的业务**，例如，Linux 内部的很多应用（有的是应用的一部分）都是使用 Shell 脚本开发的，因为有 1000 多个 Linux 系统命令为它作支撑，特别是 Linux 正则表达式以及三剑客 grep、awk、sed 等命令。
对于一些常见的系统脚本，使用 Shell 更符合 Linux 运维**简单、易用、高效**的三大原则。例如，让软件一键自动化安装、优化，监控报警脚本，软件启动脚本，日志分析脚本等。

## 三，几种常见的 Shell
Linux 是一个开源的操作系统，由分布在世界各地的多个组织机构或个人共同开发完成，每个组织结构或个人负责一部分功能，最后组合在一起。
不同的组织机构为了发展自己的 Linux 分支可能会开发出功能类似的软件，它们各有优缺点，用户可以自由选择。

Shell 也是，**不同的组织**机构开发了**不同的 Shell**，有的占用资源少，有的支持高级编程功能，有的兼容性好，有的重视用户体验。常见的 Shell 有 sh、bash、csh、tcsh、ash 等。 

**sh** 的全称是 Bourne shell，由 AT&T 公司的 Steve Bourne开发，为了纪念他，就用他的名字命名了。sh 是 UNIX 上的标准 shell，很多 UNIX 版本都配有 sh。sh 是第一个流行的 Shell。在现代的 Linux 上，sh 已经被 bash 代替，/bin/sh往往是指向/bin/bash的符号链接。

**csh** 的语法有点类似C语言，所以才得名为 C shell ，简称为 csh。

**tcsh** 是 csh 的增强版，加入了命令补全功能，提供了更加强大的语法支持。

**ash** 一个简单的轻量级的 Shell，占用资源少，适合运行于低内存环境，但是与下面讲到的 bash shell 完全兼容。

**bash** 是 Linux 的默认 shell，保持了对 sh shell 的兼容性，针对 sh 编写的 Shell 代码可以不加修改地在 bash 中运行,是各种 Linux 发行版默认配置的 shell。

## 四，进入Shell
在 Linux 操作系统一般有两种方式进入 Shell 环境。

一种进入 Shell 的方法是通过快捷键**退出图形界面模式**，进入控制台模式，这样一来，显示器上只有一个简单的带着白色文字的“黑屏”，就像图形界面出现之前的样子。这种模式称为 **Linux 控制台（Console）**。

**图形界面也是一个程序**，会占用CPU时间和内存空间，当 Linux 作为服务器系统时，安装调试完毕后，应该让 Linux 运行在控制台模式下，以节省服务器资源。正是由于这个原因，很多服务器甚至不安装图形界面程序，管理员只能使用命令来完成各项操作。

另外一种方法是使用 Linux 桌面环境中的**终端模拟包**（Terminal emulation package），也就是我们常说的终端（Terminal），这样在图形桌面中就可以使用 Shell。

进入 Shell 环境后，便可以看到 Shell 提示符，对于普通用户，Base shell 默认的提示符是美元符号$；对于超级用户（root 用户），Bash Shell 默认的提示符是井号#。该符号表示 Shell 等待输入命令。

不同的 Linux 发行版使用的提示符格式不同，一般是用户名 主机名 和当前目录等信息拼接而成。
```
CentOS 默认的提示符格式为：
[wang@localhost ~] $

OPENTHOS 默认提示符格式为：
bash-4.4 $
root@OPENTHOS:/ ＃
```

Shell 通过PS1和PS2两个环境变量来控制提示符格式：
* PS1 控制最外层命令行的提示符格式。
* PS2 控制第二层命令行的提示符格式。

默认提示符为 PS1，当用户输入了如单个 **“** 字符时按回车，Shell 认为语句是不完整的需要附加信息补充，这时提示符就会变成 PS2，直到单个 **”**  字符出现。

```
[wang@localhost ~ ]$: echo "Hello Shell"
Hello Shell
[wang@localhost ~ ]$: echo "
> Hello
> Shell
>"
Hello
Shell
[wang@localhost ~ ]$:
```

## 五，脚本文件的创建和执行
利用文本编辑器新建一个文件，扩展为 sh（代表Shell类型的脚本，也可以是 php等，不影响执行）。
编辑文件内容写入一些 Shell 语句如：
```
#! /bin/bash
echo "Hello Shell"
```
“#!” 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。

运行Shell脚本有两种方法。
#### 1 作为可执行程序
将上面的代码保存为test.sh，并 cd 到相应目录：

chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本

注意，一定要写成./test.sh，而不是test.sh。运行其它二进制的程序也一样，**直接写test.sh，linux系统会去PATH里寻找**有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。

通过这种方式运行bash脚本，第一行一定要写对，好让系统查找到正确的解释器。

##### 2 作为解释器参数
这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，如：

/bin/sh test.sh
/bin/php test.php

这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

## 六，Shell 变量
在 Bash shell 中，每一个变量的值都是字符串，无论给变量赋值时有没有使用引号，值都会以字符串的形式存储。这意味着，Bash shell 在默认情况下不会区分变量类型。

#### 定义和变量
Shell 支持以下三种定义变量的方式：

    variable=value
    variable='value'
    variable="value"

如果 value 不包含任何空白符（例如空格、Tab缩进等），那么可以不使用引号。
***注意，赋值号的周围不能有空格.***

以单引号' '包围 value 时，变量保存的是 value 直观表面的字符串；
以双引号" "包围 value 时，变量保存的是将 value 内的命令或变量解析替换之后的结果。
```
#! /bin/bash
log="log std..."
var1='${log} yes'
var2="${log} yes"
echo ${var1} 
echo ${var2}
```
输出结果分别为：
```
${log} yes
log std... yes
```
其中，$ 符号表示引用变量，花括号｛｝可以帮助解释器识别变量的边界，最好一直都加上｛｝来规范习惯。

类似的还有 $() 表示输出括号内的命令的执行结果。
例如:
```
#! /bin/bash
var=$(ls ~)
echo ${var}
```
变量 var 的值为命令 ls ～ 的执行结果，然后 echo 引用变量 var 的值进行输出。

#### 变量类型
运行shell时，会同时存在三种变量：
1) 局部变量
局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
2) 环境变量
所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
3) shell变量
shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行。



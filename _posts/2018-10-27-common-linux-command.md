---
title: "Common Linux Command"
layout: post
date: 2018-10-27 22:44
tag:
- linux
- command
star: true
category: blog
author: jiaqixu
description: note for linux command for self-lookup
---

## Linux Command Note

#### Linux查看系统位数
1. getconf LONG_BIT: 如果返回结果是32则说明是32位的,返回是64则说明是64位的.
2. uname -a: 输出结果中有x86_64则说明是64位的,否则是32位的.

#### find 命令
基本格式: find start_directory test options criteria_to_match action_to_perform_on_results

用法:
find . -name "*.d": 将开始在当前目录(用"."表示) 中查找任何扩展名为".d"的文件
find . -name \*.d: 因为没有使用引号,所以使用反斜杠对通配符进行转义以确保它传递到find命令并且不由shell解释.
find /home/src -name \*.d: 指定查找的起始目录的完整路径.
find usr/ home/ tmp/ -name \*.d: 指定多个起始目录.
find . -type d: 按类型搜索文件, 这个命令用来查找所有的子目录.
find . -type f: 查找所有的文件.
find . -type l: 查找所有的符号链接.

#### grep 命令
grep 全称为global search regular expression and print out the line, 它是一种强大的文本搜索工具. 它能使用正则表达式搜索文件,并把匹配的行打印出来.

用法:
grep -nirH -e "process$" --include *1.xom

-e: 指定字符串作为查找文件内容的范本样式(regex)
-n: 在显示符合范本样式的那一列之前,标示出该列的编号
-H: 在显示符合范本样式那一列之前.标示该列的文件名称
-R: 查找所有文件包含子目录
-i: 忽略大小写
-l: 列出文件诶通符合指定的范本样式的文件名称
-f: 指定范本文件, 可以包含以个或者多个范本样式,
-c: 计算符合范文样式的数量
-- include: 只在目录中所有文件名符合该格式的的文件中查找.

#### cd 命令
cd 命令用来切换工作目录至dirname. 其中dirName表示法可为绝对路径或者相对路径. 若目录名称省略,则变换至使用者的home directory (~). 另外home_directory 可以用~表示, 目前所在的目录用.标示,目前目录位置的上以层目录用..表示.

用法:
cd  进入用户主目录
cd ~ 进入用户主目录
cd - 返回进入此目录之前所在的目录
cd .. 返回上级目录
cd ../.. 返回上两级目录

#### ls 命令
ls 命令用来显示目标列表,在Linux中是使用率较高的命令. ls命令的输出信息可以进行彩色加亮显示,以区分不能类型的文件.

用法:
-a 显示所有文件以及目录. (ls默认将文件名或者目录名称为"."的视为隐藏,不会列出)
-l 所有输出信息用单列格式输出,不输出为多列(-C默认)
--color: 使用不同的颜色高亮显示不同类型的文件


#### pwd 命令
pwd命令以绝对路径的方式显示用户当前工作目录. 命令将当前目录的全路径名称(从根目录)写入标准输出. 全部目录使用/分割.第一个/表示根目录,最后一个目录是当前目录.

#### echo 命令
echo命令用于在shell中打印shell变量的值,或者直接输出指定的字符串. Linux的echo命令,在shell编程中极为常用,在终端下打印变量value的时候也是常常用到的,因此有必要了解下echo的用法. echo命令的功能是在显示器上显示一段文字,一般起到以个提示的作用.

用法:
-e 激活转义字符.
使用-e选项时,若字符串中出现以下字符,则特别加以处理,而不会将它当成一般文字输出:
\a 发出警告声;
\b 删除前一个字符;
\c 最后不加上换行符号;
\t 插入tab


#### cat 命令
cat命令连接文件并打印到标准输出设备上, cat经常用来显示文件的内容, 类似以下的type命令.
注意: 当文件较大时, 文本在屏幕上迅速闪过(滚屏), 用户往往看不清所显示的内容. 因此, 一般用more等命令分屏显示. 为了控制滚屏, 可以按Ctrl + S键, 停止滚屏; 按Ctrl + Q键可以恢复滚屏. 按Ctrl + C (中断) 键可以终止该命令的执行, 并且返回Shell提示符状态.

用法:
-n 由1开始对所有输出的行数编号;

cat file_1 在屏幕上显示文件file_1的内容
cat file_1 file_2 同时显示文件file_1 和 file_2 的内容
cat file_1 file_2 > file 将文件file_1和file_2合并后放入文件file中

#### tee 命令
tee命令用于将数据重定向到文件, 另一方面还可以提供一份重定向数据的副本作为后续命令的stdin. 简单的说就是把数据重定向到给定文件和屏幕上.
存在缓存机制,每1024个字节将输出一次. 若从管道接收输入数据,应该是缓冲区满,才将数据转存到指定的文件中.若文件内容不到1024个字节,则接收完从标准输入设备读入的数据后, 将刷新一次缓冲区, 并转存数据到指定文件.

用法:
ls | tee out.txt


#### | 命令
管道(pipe)命令操作符是*|* , 它只能处理经由前面一个指令传出的正确输出信息, 对错误的信息没有直接处理能力. 然后传递给下一个命令, 作为标准的输入.

#### IO redirection

#### which 命令
which命令用于查找并显示给定命令的绝对路径,环境变量PATH中保存了查找命令时需要遍历的目录. Which指令会在环境变量$PATH设置的目录里查找符合条件的文件. 也就是说, 使用which命令,就可以看到某个系统命令是否存在,以及执行的到底是哪一个位置的命令.

用法:                                                                                                          
which pwd  输出/bin/pwd                                                                                        
which python  输出 /usr/bin/python                                                                             
可见,which是根据使用者所配置的PATH变量内的目录去搜寻可运行文件的,所以不同的PATH配置内容所找到的命令当然不一样.

#### chmod 命令

#### type 命令

#### file 命令

#### cksum 命令

#### hostname 命令

 
 

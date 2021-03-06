---
title: "Common Linux Command"
layout: post
date: 2017-10-27 22:44
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
1. find . -name "*.d": 将开始在当前目录(用"."表示) 中查找任何扩展名为".d"的文件
2. find . -name \*.d: 因为没有使用引号,所以使用反斜杠对通配符进行转义以确保它传递到find命令并且不由shell解释.
3. find /home/src -name \*.d: 指定查找的起始目录的完整路径.
4. find usr/ home/ tmp/ -name \*.d: 指定多个起始目录.
5. find . -type d: 按类型搜索文件, 这个命令用来查找所有的子目录.
6. find . -type f: 查找所有的文件.
7. find . -type l: 查找所有的符号链接.

other usage:
```text
for i in 'find input -name '19827*' -and -type d '
do
  touch $i/.SKIP-VISA
done
```

#### grep 命令
grep 全称为global search regular expression and print out the line, 它是一种强大的文本搜索工具. 它能使用正则表达式搜索文件,并把匹配的行打印出来.

用法:<br>
grep -nirH -e "process$" --include \*1.xom

1. -e: 指定字符串作为查找文件内容的范本样式(regex)
2. -n: 在显示符合范本样式的那一列之前,标示出该列的编号
3. -H: 在显示符合范本样式那一列之前.标示该列的文件名称
4. -R: 查找所有文件包含子目录
5. -i: 忽略大小写
6. -l: 列出文件诶通符合指定的范本样式的文件名称
7. -f: 指定范本文件, 可以包含以个或者多个范本样式,
8. -c: 计算符合范文样式的数量
9. -- include: 只在目录中所有文件名符合该格式的的文件中查找.

#### cd 命令
cd 命令用来切换工作目录至dirname. 其中dirName表示法可为绝对路径或者相对路径. 若目录名称省略,则变换至使用者的home directory (~). 另外home_directory 可以用~表示, 目前所在的目录用.标示,目前目录位置的上以层目录用..表示.

用法:<br>
1. cd  进入用户主目录
2. cd ~ 进入用户主目录
3. cd - 返回进入此目录之前所在的目录
4. cd .. 返回上级目录
5. cd ../.. 返回上两级目录

#### ls 命令
ls 命令用来显示目标列表,在Linux中是使用率较高的命令. ls命令的输出信息可以进行彩色加亮显示,以区分不能类型的文件.

用法:<br>
1. -a 显示所有文件以及目录. (ls默认将文件名或者目录名称为"."的视为隐藏,不会列出)
2. -l 所有输出信息用单列格式输出,不输出为多列(-C默认)
3. --color: 使用不同的颜色高亮显示不同类型的文件

ls -l(以列表的形式显示文件)每一列的含义<br>
e.g. drwxr-xr-x 2 jxu rd  4096 Oct 11 15:26 20181011

<img src="/assets/images/blog/linux_ls.png">


#### pwd 命令
pwd命令以绝对路径的方式显示用户当前工作目录. 命令将当前目录的全路径名称(从根目录)写入标准输出. 全部目录使用/分割.第一个/表示根目录,最后一个目录是当前目录.

#### echo 命令
echo命令用于在shell中打印shell变量的值,或者直接输出指定的字符串. Linux的echo命令,在shell编程中极为常用,在终端下打印变量value的时候也是常常用到的,因此有必要了解下echo的用法. echo命令的功能是在显示器上显示一段文字,一般起到以个提示的作用.

用法:<br>
-e 激活转义字符.<br>
使用-e选项时,若字符串中出现以下字符,则特别加以处理,而不会将它当成一般文字输出:
1. \a 发出警告声;
2. \b 删除前一个字符;
3. \c 最后不加上换行符号;
4. \t 插入tab


#### cat 命令
cat命令连接文件并打印到标准输出设备上, cat经常用来显示文件的内容, 类似以下的type命令.
注意: 当文件较大时, 文本在屏幕上迅速闪过(滚屏), 用户往往看不清所显示的内容. 因此, 一般用more等命令分屏显示. 为了控制滚屏, 可以按Ctrl + S键, 停止滚屏; 按Ctrl + Q键可以恢复滚屏. 按Ctrl + C (中断) 键可以终止该命令的执行, 并且返回Shell提示符状态.

用法:<br>
-n 由1开始对所有输出的行数编号;

1. cat file_1 在屏幕上显示文件file_1的内容
2. cat file_1 file_2 同时显示文件file_1 和 file_2 的内容
3. cat file_1 file_2 > file 将文件file_1和file_2合并后放入文件file中

#### tee 命令
tee命令用于将数据重定向到文件, 另一方面还可以提供一份重定向数据的副本作为后续命令的stdin. 简单的说就是把数据重定向到给定文件和屏幕上.
存在缓存机制,每1024个字节将输出一次. 若从管道接收输入数据,应该是缓冲区满,才将数据转存到指定的文件中.若文件内容不到1024个字节,则接收完从标准输入设备读入的数据后, 将刷新一次缓冲区, 并转存数据到指定文件.

用法:<br>
ls | tee out.txt


#### | 命令
管道(pipe)命令操作符是*|* , 它只能处理经由前面一个指令传出的正确输出信息, 对错误的信息没有直接处理能力. 然后传递给下一个命令, 作为标准的输入.

#### IO redirection

#### which 命令
which命令用于查找并显示给定命令的绝对路径,环境变量PATH中保存了查找命令时需要遍历的目录. Which指令会在环境变量$PATH设置的目录里查找符合条件的文件. 也就是说, 使用which命令,就可以看到某个系统命令是否存在,以及执行的到底是哪一个位置的命令.

用法:<br>                                                                                                          
1. which pwd  输出/bin/pwd                                                                                        
2. which python  输出 /usr/bin/python
                                                                             
可见,which是根据使用者所配置的PATH变量内的目录去搜寻可运行文件的,所以不同的PATH配置内容所找到的命令当然不一样.

#### chmod 命令
chmod命令用来变更文件或目录的权限. 在UNIX系统家族里,文件或目录权限的控制分别以读取,写入,执行3种一般权限来区分,另有3种特殊权限可供运用.用户可以使用chmod指令去变更文件与目录的权限,设置方式采用文字或数字代号皆可.符号连接的权限无法变更,如果用户对符号连接修改权限,其改变会作用在被连接的原始文件.

Linux用户分为: 拥有者,群组(Group),其他(other).

用法:<br>
chmod u+x, g+w file_1: 为文件file_1设置自己可以执行,组员可以写入的权限
1. u 表示该文件的拥有者, g 表示与该文件拥有者属于同一个群体(group)者, o 表示其他以外的人, a 表示三者皆是.
2. + 表示增加权限, - 表示取消权限, = 表示唯一设定权限.
3. r 表示可读取, w 表示可写入, x 表示可执行

#### type 命令
type命令用来显示指定命令的类型, 判断给出的指定是内部指令还是外部指令.

命令类型:
<ul>
<li>alias: 别名</li>
<li>keyword: 关键字, Shell保留字</li>
<li>function: 函数, Shell函数</li>
<li>builtin: 内建命令, Shell内建命令</li>
<li>文件,磁盘文件,外部命令</li>
<li>unfound: 没有找到</li>
</ul>

用法:<br>
1. -t: 输出"file", "alias", "builtin", 分别表示给定的指令为"外部指令", "命令别名" 或者"内部指令";
2. -p: 如果给出的指令为外部指令,则显示其绝对路径.
3. -a: 在环境变量"PATH"指定的路径中,显示给定指令的信息,包括命令别名.


#### file 命令
file命令用来探测给定文件的类型. file命令对文件的检查分为文件系统,魔法幻数检查和语言检查3个过程.

用法:<br>
file test.txt 输出test.txt: ASCII text
1. -b 列出辨识结果时,不显示文件名称;
2. -c 详细显示指令执行过程


#### cksum 命令
cksum命令是检查文件的CRC是否正确，确保文件从一个系统传输到另一个系统的过程中不被损坏。这种方法要求校验和在源系统中被计算出来，在目的系统中又被计算一次，两个数字进行比较，如果校验和相等，则该文件被认为是正确传输了。

注意：CRC是指一种排错检查方法，即循环冗余校验法。

指定文件交由cksum命令进行校验后，会返回校验结果供用户核对文件是否正确无误。若不指定任何文件名称或是所给予的文件名为"-"，则cksum命令会从标准输入设备中读取数据。

用法:<br>
cksum test_file

以上命令执行后，将输出校验码等相关的信息，具体输出信息如下所示：<br>
1263453430 78 test_file

上面的输出信息中，"1263453430"表示校验码，"78"表示字节数。

注意：如果文件中有任何字符被修改，都将改变计算后CRC校验码的值。


#### hostname 命令
hostname命令用于显示和设置系统的主机名称。环境变量HOSTNAME也保存了当前的主机名。在使用hostname命令设置主机名后，系统并不会永久保存新的主机名，重新启动机器之后还是原来的主机名。如果需要永久修改主机名，需要同时修改/etc/hosts和/etc/sysconfig/network的相关内容。

用法:
1. -v：详细信息模式；
2. -a：显示主机别名；
3. -d：显示DNS域名；
4. -f：显示FQDN名称；
5. -i：显示主机的ip地址；
6. -s：显示短主机名称，在第一个点处截断；
7. -y：显示NIS域名。
 
#### shell
1. 什么是shell? 网上有一下几种说法:<br>
shell就是系统内核(kernel)的一层壳,作用是用来保护内核同时传递人与计算机交互的信息.它只是系统的一个工具,我们可以使用它来操作计算机.
 
shell是用户和Linux（或者更准确的说，是用户和Linux内核）之间的接口程序。你在提示符下输入的每个命令都由shell先解释然后传给Linux内核。

shell 是一个命令语言解释器（command-language interpreter）。拥有自己内建的 shell 命令集。此外，shell也能被系统中其他有效的Linux 实用程序和应用程序（utilities and application programs）所调用。 不论何时你键入一个命令，它都被Linux shell所解释。一些命令，比如打印当前工作目录命令（pwd），是包含在Linux bash内部的（就象DOS的内部命令）。其他命令，比如拷贝命令（cp）和移动命令（rm），是存在于文件系统中某个目录下的单独的程序。而对用户来说，你不知道（或者可能不关心）一个命令是建立在shell内部还是一个单独的程序。

shell 首先检查命令是否是内部命令，不是的话再检查是否是一个应用程序，这里的应用程序可以是Linux本身的实用程序，比如ls rm， 
然后shell试着在搜索路径($PATH)里寻找这些应用程序。搜索路径是一个能找到可执行程序的目录列表。如果你键入的命令不是一个内部命令并且在路径里没有找到这个可执行文件，将会显示一条错误信息。而如果命令被成功的找到的话，shell的内部命令或应用程序将被分解为系统调用并传给Linux内核。


#### diff 命令
diff命令在最简单的情况下，比较给定的两个文件的不同。如果使用“-”代替“文件”参数，则要比较的内容将来自标准输入。diff命令是以逐行的方式，比较文本文件的异同处。如果该命令指定进行目录的比较，则将会比较该目录中具有相同文件名的文件，而不会对其子目录文件进行任何比较操作.

用法:<br>
diff file_1 file_2

#### cut 命令
cut命令用来显示行中的指定部分，删除文件中指定字段。cut经常用来显示文件的内容.

用法:<br>
1. b 仅显示行中指定直接范围的内容
2. c 仅显示行中指定范围的字符
3. d 指定字段的分隔符,默认的字段分隔符为TAB
4. f 显示指定字段的内容

cut -f 1 test.txt 使用-f选项提取第一个字段的内容
cut -f 1,2 test.txt 提取多个字段的内容


#### ps 命令
ps命令用于报告当前系统的进程状态. 可以搭配kill指令随时中断,删除不必要的程序. ps命令是最基本同时也是非常强大的进程查看命令,使用该命令可以确定有哪些进程正在运行和运行的状态,进程是否结束,进程有没有僵死,哪些进程占用了过多的资源等等,总之大部分信息都是可以通过执行该命令得到的.

用法:<br>
1. ps aux BSD格式,参数前面不加-
2. ps -aux UNIX/LINUX格式, 参数前面通常要加-
3. ps -AF | grep python


#### top 命令 (性能分析工具)
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。

top显示系统当前的进程和其他状况,是一个动态显示过程,即可以通过用户按键来不断刷新当前状态.如果在前台执行该命令,它将独占前台,直到用户 终止该程序为止. 比较准确的说,top命令提供了实时的对系统处理器的状态监视.它将显示系统中CPU最“敏感”的任务列表.该命令可以按CPU使用,内存使用和执行时间 对任务进行排序；而且该命令的很多特性都可以通过交互式命令或者在个人定制文件中进行设定.

top使用格式: top [-][d][p][q][c][C][S][s][n]<br>
用法:<br>
1. d 指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。 
2. p 通过指定监控进程ID来仅仅监控某个进程的状态。 
3. q 该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。 
4. S 指定累计模式 
5. s 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。 
6. i 使top不显示任何闲置或者僵死进程。 
7. c 显示整个命令行而不只是显示命令名 

top命令执行过程中可以使用一些交互命令:<br>
1. Ctrl+L: 擦除并且重写屏幕。 
2. h或者?: 显示帮助画面，给出一些简短的命令总结说明。 
3. k: 终止一个进程。系统将提示用户输入需要终止的进程PID，以及需要发送给该进程什么样的信号。一般的终止进程可以使用15信号；如果不能正常结束那就使用信号9强制结束该进程。默认值是信号15。在安全模式中此命令被屏蔽。 
4. i: 忽略闲置和僵死进程。这是一个开关式命令。 
5. q: 退出程序。 
6. r: 重新安排一个进程的优先级别。系统提示用户输入需要改变的进程PID以及需要设置的进程优先级值。输入一个正值将使优先级降低，反之则可以使该进程拥有更高的优先权。默认值是10。 
7. S: 切换到累计模式。 
8. s: 改变两次刷新之间的延迟时间。系统将提示用户输入新的时间，单位为s。如果有小数，就换算成m s。输入0值则系统将不断刷新，默认值是5 s。需要注意的是如果设置太小的时间，很可能会引起不断刷新，从而根本来不及看清显示的情况，而且系统负载也会大大增加。 
9. f或者F: 从当前显示中添加或者删除项目。 
10. o或者O: 改变显示项目的顺序。 
11. l: 切换显示平均负载和启动时间信息。 
12. m: 切换显示内存信息。 
13. t: 切换显示进程和CPU状态信息。 
14. c: 切换显示命令名称和完整命令行。 
15. M: 根据驻留内存大小进行排序。 
16. P: 根据CPU使用百分比大小进行排序。 
17. T: 根据时间/累计时间进行排序。 
18. W: 将当前设置写入~/.toprc文件中。这是写top配置文件的推荐方法。

常用操作:<br>
1. top   //每隔5秒显式所有进程的资源占用情况
2. top -d 2  //每隔2秒显式所有进程的资源占用情况
3. top -c  //每隔5秒显式进程的资源占用情况，并显示进程的命令行参数(默认只有进程名)
4. top -p 12345 -p 6789//每隔5秒显示pid是12345和pid是6789的两个进程的资源占用情况
5. top -d 2 -c -p 123456 //每隔2秒显示pid是12345的进程的资源使用情况，并显式该进程启动的命令行参数



#### command line history
history命令用于显示指定数目的指令命令，读取历史命令文件中的目录到历史命令缓冲区和将历史命令缓冲区中的目录写入命令文件。

该命令单独使用时，仅显示历史命令，在命令行中，可以使用符号!执行指定序号的历史命令。例如，要执行第2个历史命令，则输入!2

历史命令是被保存在内存中的，当退出或者登录shell时，会自动保存或读取。在内存中，历史命令仅能够存储1000条历史命令，该数量是由环境变量HISTSIZE进行控制。

用法:<br>
1. history 10 使用history命令显示最近使用的10条历史命令
2. !2 执行序号为2的历史命令.



#### backtick

#### subshells

### xargs
xargs命令是给其他命令传递参数的一个过滤器, 也是组合多个命令的一个工具. 它擅长将标准输入数据转换称命令行参数, xargs能够处理管道或者stdin并将其转换成特定命令的命令参数.
xargs也可以将单行或多行文本输入转换为其他格式, 例如多行变单行, 单行变多行. xargs的默认命令是echo, 空格是默认定界符. 这意味着通过管道传递给xargs的输入将会包含换行和空白,
不过通过xargs的处理,换行和空白将被空格取代. xargs是构建单行命令的重要组件之一.

用法:<br>
1. find . -name ".SKIP-VISA" | xargs p4 add


to be continue...


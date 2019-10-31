---
title: "python learning note"
layout: post
date: 2017-09-09 22:45
tag:
- python
- basic
star: true
category: blog
author: jiaqixu
---

Reference: https://github.com/jiaqi-xu/PythonLearning

### Can I Become a Great Coder?
yes - in time. The best coder go through several phases on their programming journey.

<ol>
<li>**The "I know nothing" phase**</li>
Everything is new, nothing is easy.
<li>**The "it's starting to make sense" phase**</li>
You've written a few programs and are making fewer mistakes.
<li>**The "I'm invincible" phase**</li>
Your confidence matches your competence. No challenge seems too difficult.
<li>**The "I know nothing" phase, part II**</li>
The sudden realization that development is infinitely more complex and you begin to doubt your own ability.
<li>**The "I know a bit and that's ok" phase**</li>
You have decent coding skills but recognize your limitations and can find solutions to most problems(even if that means hiring another developer).
</ol>

### Python 2 or Python 3?
区别：
<ul>
<li>**PRINT IS A FUNCTION**</li>
The statement has been replaced with a print() function, with keyword arguments to replace most of the special syntax of the old statement (PEP 3105). Examples: <br>


{% highlight python %}
Old: print "The answer is", 2*2 New: print("The answer is", 2*2)
Old: print x, # Trailing comma suppresses newline New: print(x, end=" ") # Appends a space instead of a newline
Old: print # Prints a newline
New: print() # You must call the function!
Old: print >>sys.stderr, "fatal error" New: print("fatal error", file=sys.stderr)
Old: print (x, y) # prints repr((x, y))
New: print((x, y)) # Not the same as print(x, y)!
{% endhighlight %}

<li>**ALL IS UNICODE NOW**</li>
从此不再为讨厌的字符编码而烦恼
<li>**Some module name change**</li>
</ul>

### 还有谁不支持python3
One popular module that don't yet support Python 3 is Twisted (for networking and other applications). Most
actively maintained libraries have people working on 3.x support. For some libraries, it's more of a priority than
others: Twisted, for example, is mostly focused on production servers, where supporting older versions of
Python is important, let alone supporting a new version that includes major changes to the language. (Twisted is
a prime example of a major package where porting to 3.x is far from trivial.


### 变量
变量的作用： 存储， 调用
Variables are used to store information to be referenced and manipulated in a computer program. They also provide a way of labeling data with a descriptive name, so our programs can be understood more clearly by the reader and ourselves. It is helpful to think of variables as containers that hold information. Their sole purpose is to label and store data in memory. This data can then be used throughout your program.

内存和硬盘的区别：
内存是用于临时存储，如果断电，数据就丢失了
硬盘用于永久存储

##### 声明变量：

#### 字符编码
python解释器在加载 .py 文件中的代码时，会对内容进行编码（默认ascill）

ASCII（American Standard Code for Information Interchange，美国标准信息交换代码）是基于拉丁字母的一套电脑编码系统，主要用于显示现代英语和其他西欧语言，其最多只能用 8 位来表示（一个字节），即：2**8 = 256-1，所以，ASCII码最多只能表示 255 个符号。
<img src='/assets/images/blog/ascii.jpg'>  

关于中文

为了处理汉字，程序员设计了用于简体中文的GB2312和用于繁体中文的big5。

GB2312(1980年)一共收录了7445个字符，包括6763个汉字和682个其它符号。汉字区的内码范围高字节从B0-F7，低字节从A1-FE，占用的码位是72*94=6768。其中有5个空位是D7FA-D7FE。

GB2312 支持的汉字太少。1995年的汉字扩展规范GBK1.0收录了21886个符号，它分为汉字区和图形符号区。汉字区包括21003个字符。2000年的 GB18030是取代GBK1.0的正式国家标准。该标准收录了27484个汉字，同时还收录了藏文、蒙文、维吾尔文等主要的少数民族文字。现在的PC平台必须支持GB18030，对嵌入式产品暂不作要求。所以手机、MP3一般只支持GB2312。

从ASCII、GB2312、GBK 到GB18030，这些编码方法是向下兼容的，即同一个字符在这些方案中总是有相同的编码，后面的标准支持更多的字符。在这些编码中，英文和中文可以统一地处理。区分中文编码的方法是高字节的最高位不为0。按照程序员的称呼，GB2312、GBK到GB18030都属于双字节字符集 (DBCS)。

有的中文Windows的缺省内码还是GBK，可以通过GB18030升级包升级到GB18030。不过GB18030相对GBK增加的字符，普通人是很难用到的，通常我们还是用GBK指代中文Windows内码。


显然ASCII码无法将世界上的各种文字和符号全部表示，所以，就需要新出一种可以代表所有字符和符号的编码，即：Unicode

Unicode（统一码、万国码、单一码）是一种在计算机上使用的字符编码。Unicode 是为了解决传统的字符编码方案的局限而产生的，它为每种语言中的每个字符设定了统一并且唯一的二进制编码，规定虽有的字符和符号最少由 16 位来表示（2个字节），即：2^16 = 65536，
注：此处说的的是最少2个字节，可能更多

UTF-8，是对Unicode编码的压缩和优化，他不再使用最少使用2个字节，而是将所有的字符和符号进行分类：ascii码中的内容用1个字节保存、欧洲的字符用2个字节保存，东亚的字符用3个字节保存...

所以，python解释器在加载 .py 文件中的代码时，会对内容进行编码（默认ascill），如果是如下代码的话：

报错：ascii码无法表示中文

```python
#!/usr/bin/env python

print "你好，世界"
```

改正：应该显示的告诉python解释器，用什么编码来执行源代码，即：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

print "你好，世界"
```

### 注释
单行注视：# 被注释内容

多行注释：""" 被注释内容 """

python编码中每一行最多不能超过80个字符（开发规范pep8）

### #!/usr/bin/env python与#!/usr/bin/python的区别
脚本语言的第一行，目的就是指出，你想要你的这个文件中的代码用什么可执行程序去运行它，就这么简单

#!/usr/bin/python是告诉操作系统执行这个脚本的时候，调用/usr/bin下的python解释器；

#!/usr/bin/env python这种用法是为了防止操作系统用户没有将python装在默认的/usr/bin路径里。当系统看到这一行的时候，首先会到env设置里查找python的安装路径，再调用对应路径下的解释器程序完成操作。

#!/usr/bin/python相当于写死了python路径;
#!/usr/bin/env python会去环境设置寻找python目录,推荐这种写法

### 用户输入

```python
#!/usr/bin/env python
#_*_coding:utf-8_*_

#name = raw_input("What is your name?") #only on python 2.x
name = input("What is your name?")
print("Hello " + name )
```python

在python2.X中，如果你用input（），然后输入一个字符串，他会认为你输入的是一个变量;但是在3.x版本中不存在这个问题
```pyhton
In [12]: a ='jiaqixu'

In [13]: user_input = input('what your name?:')
what your name?:a

In [14]: user_input
Out[14]: 'jiaqixu'
```


### 常用模块初识

1.getpass模块

```python
In [20]: import getpass
    ...:
    ...: name = raw_input("please input your user
    ...: name:")
    ...: password = getpass.getpass('please type
    ...: your password:')
    ...:
    ...: print name+', ' + password
    ...:
please input your username:jq
please type your password:
j, 123456
```

2.os模块

```python
In [22]: import os

In [23]: os.system('df -h')
Filesystem                              Size   Used  Avail Capacity iused      ifree %iused  Mounted on
/dev/disk1                             465Gi   64Gi  401Gi    14% 1405751 4293561528    0%   /
devfs                                  187Ki  187Ki    0Bi   100%     646          0  100%   /dev
map -hosts                               0Bi    0Bi    0Bi   100%       0          0  100%   /net
map auto_home                            0Bi    0Bi    0Bi   100%       0          0  100%   /home
/Users/jiaqixu/Downloads/Caffeine.app  465Gi   56Gi  408Gi    13% 1316264 4293651015    0%   /private/var/folders/wg/l7bm_36j0j7d050l8hctyvc80000gp/T/AppTranslocation/64794938-D7CF-4843-8A15-E6C86397DD4B
/dev/disk2s1                            10Mi  1.5Mi  8.5Mi    15%      94 4294967185    0%   /Volumes/FileZilla
/dev/disk3s1                            15Mi  4.9Mi   10Mi    33%     262 4294967017    0%   /Volumes/Install Cisco WebEx Add-On
Out[23]: 0#返回值为0 代表linux命令执行成功
```

3.自定义tab模块
```python
import sys
import readline
import rlcompleter

if sys.platform == 'darwin' and sys.version_info[0] == 2:
    readline.parse_and_bind("bind ^I rl_complete")
else:
    readline.parse_and_bind("tab: complete")  # linux and python3 on mac
```

4. csv模块

### 什么是pyc(what is pyc)
1. python是一门解释性语言
我初学Python时，听到的关于Python的第一句话就是，Python是一门解释性语言，我就这样一直相信下去，直到发现了*.pyc文件的存在。如果是解释型语言，那么生成的*.pyc文件是什么呢？c应该是compiled的缩写才对啊！

为了防止其他学习Python的人也被这句话误解，那么我们就在文中来澄清下这个问题，并且把一些基础概念给理清。

2.解释型语言和编译型语言
计算机是不能够识别高级语言的，所以当我们运行一个高级语言程序的时候，就需要一个“翻译机”来从事把高级语言转变成计算机能读懂的机器语言的过程。这个过程分成两类，第一种是编译，第二种是解释。

编译型语言在程序执行之前，先会通过编译器对程序执行一个编译的过程，把程序转变成机器语言。运行时就不需要翻译，而直接执行就可以了。最典型的例子就是C语言。

解释型语言就没有这个编译的过程，而是在程序运行的时候，通过解释器对程序逐行作出解释，然后直接运行，最典型的例子是Ruby。

通过以上的例子，我们可以来总结一下解释型语言和编译型语言的优缺点，因为编译型语言在程序运行之前就已经对程序做出了“翻译”，所以在运行时就少掉了“翻译”的过程，所以效率比较高。但是我们也不能一概而论，一些解释型语言也可以通过解释器的优化来在对程序做出翻译时对整个程序做出优化，从而在效率上超过编译型语言。

此外，随着Java等基于虚拟机的语言的兴起，我们又不能把语言纯粹地分成解释型和编译型这两种。

用Java来举例，Java首先是通过编译器编译成字节码文件，然后在运行时通过解释器给解释成机器文件。所以我们说Java是一种先编译后解释的语言。

3. Python到底是什么
其实Python和Java/C#一样，也是一门基于虚拟机的语言，我们先来从表面上简单地了解一下Python程序的运行过程吧。

当我们在命令行中输入python hello.py时，其实是激活了Python的“解释器”，告诉“解释器”：你要开始工作了。可是在“解释”之前，其实执行的第一项工作和Java一样，是编译

熟悉Java的同学可以想一下我们在命令行中如何执行一个Java的程序：

javac hello.java

java hello


只是我们在用Eclipse之类的IDE时，将这两部给融合成了一部而已。其实Python也一样，当我们执行python hello.py时，他也一样执行了这么一个过程，所以我们应该这样来描述Python，Python是一门先编译后解释的语言。

4.简述Python的运行过程
在说这个问题之前，我们先来说两个概念，PyCodeObject和pyc文件。

我们在硬盘上看到的pyc自然不必多说，而其实PyCodeObject则是Python编译器真正编译成的结果。我们先简单知道就可以了，继续向下看。

当python程序运行时，编译的结果则是保存在位于内存中的PyCodeObject中，当Python程序运行结束时，Python解释器则将PyCodeObject写回到pyc文件中。

当python程序第二次运行时，首先程序会在硬盘中寻找pyc文件，如果找到，则直接载入，否则就重复上面的过程。

所以我们应该这样来定位PyCodeObject和pyc文件，我们说pyc文件其实是PyCodeObject的一种持久化保存方式。

注意:如果原.py文件改了，而且对应原来的.pyc文件还存在，那怎么办？
每次运行python文件的时候，系统会自动比较已存在的python字节码文件（.pyc）的修改时间与对应源文件（.py）修改时间，如果发现源文件的修改时间
相对于字节码文件的修改时间较新，说明源文件被改动过，这时候会重新编译。


### Set集合

set集合，是一个无序且不重复的可嵌套的元素集合

```python
class set(object):
    """
    set() -> new empty set object
    set(iterable) -> new set object

    Build an unordered collection of unique elements.
    """
    def add(self, *args, **kwargs): # real signature unknown
        """
        Add an element to a set，添加元素

        This has no effect if the element is already present.
        """
        pass

    def clear(self, *args, **kwargs): # real signature unknown
        """ Remove all elements from this set. 清除内容"""
        pass

    def copy(self, *args, **kwargs): # real signature unknown
        """ Return a shallow copy of a set. 浅拷贝  """
        pass

    def difference(self, *args, **kwargs): # real signature unknown
        """
        Return the difference of two or more sets as a new set. A中存在，B中不存在

        (i.e. all elements that are in this set but not the others.)
        """
        pass

    def difference_update(self, *args, **kwargs): # real signature unknown
        """ Remove all elements of another set from this set.  从当前集合中删除和B中相同的元素"""
        pass

    def discard(self, *args, **kwargs): # real signature unknown
        """
        Remove an element from a set if it is a member.

        If the element is not a member, do nothing. 移除指定元素，不存在不保错
        """
        pass

    def intersection(self, *args, **kwargs): # real signature unknown
        """
        Return the intersection of two sets as a new set. 交集

        (i.e. all elements that are in both sets.)
        """
        pass

    def intersection_update(self, *args, **kwargs): # real signature unknown
        """ Update a set with the intersection of itself and another.  取交集并更更新到A中 """
        pass

    def isdisjoint(self, *args, **kwargs): # real signature unknown
        """ Return True if two sets have a null intersection.  如果没有交集，返回True，否则返回False"""
        pass

    def issubset(self, *args, **kwargs): # real signature unknown
        """ Report whether another set contains this set.  是否是子序列"""
        pass

    def issuperset(self, *args, **kwargs): # real signature unknown
        """ Report whether this set contains another set. 是否是父序列"""
        pass

    def pop(self, *args, **kwargs): # real signature unknown
        """
        Remove and return an arbitrary set element.
        Raises KeyError if the set is empty. 移除元素
        """
        pass

    def remove(self, *args, **kwargs): # real signature unknown
        """
        Remove an element from a set; it must be a member.

        If the element is not a member, raise a KeyError. 移除指定元素，不存在保错
        """
        pass

    def symmetric_difference(self, *args, **kwargs): # real signature unknown
        """
        Return the symmetric difference of two sets as a new set.  对称差集

        (i.e. all elements that are in exactly one of the sets.)
        """
        pass

    def symmetric_difference_update(self, *args, **kwargs): # real signature unknown
        """ Update a set with the symmetric difference of itself and another. 对称差集，并更新到a中 """
        pass

    def union(self, *args, **kwargs): # real signature unknown
        """
        Return the union of sets as a new set.  并集

        (i.e. all elements that are in either set.)
        """
        pass

    def update(self, *args, **kwargs): # real signature unknown
        """ Update a set with the union of itself and others. 更新 """
        pass
```


### 函数
1. def
2. 函数名
3. 函数体
4. 返回值
==》 定义函数，函数体不执行
5. 参数

  a. 普通参数
    {% highlight python%}
    def function(a, b, c):
      print(a, b, c)
    {% endhighlight %}

  b. 默认参数，(注意，默认参数必须放在参数列表的后面)
    {% highlight python%}
    def function(c, b, a='a'):
      print(a, b, c)
    {% endhighlight %}

  c. 指定参数
    {% highlight python%}
    def function(a, b, c):
      print(a, b, c)
    function(a='a', b='b', c)
    {% endhighlight %}

  d. 动态参数
  一个星*:
  {% highlight python%}
    In [51]: def f(*args):
    ...:     print args
    ...:

    In [52]: f(1,2,3)
    (1, 2, 3)

    In [53]: li = [123,3]

    In [54]: f(li,4)
    ([123, 3], 4)

    In [55]: a = 'a12'

    In [56]: f(a)
    ('a12',)

    In [57]: f(*a)
    ('a', '1', '2')
  {% endhighlight %}


  两个星**：
  {% highlight python%}
  def f2(**args):
    print(args, type(args))

  f2(n1='alex', n2=18) # {'n1':'alex', 'n2':18} <class 'dict'>
  dict = {'k1': 'v1', 'k2': 'v2'}
  f2(k1=dict) # {'k1': {'k1': 'v1', 'k2': 'v2'}} <class'dict'>
  f2(**dict) # {'k2':'v2', 'k1':'v1'} <class 'dict'>
  {% endhighlight %}

  万能参数
  {% highlight python%}

  def f3(*args, **kwargs):
      print(args)
      print(kwargs)

  f3(11, 22, 33, 44, k1='v1', k2='v2') # (11, 22, 33, 44) {'k2': 'v2', 'k1': 'v1'}
{% endhighlight %}

### 内置函数
```python
#-*- coding: utf-8 -*-
'''
内置函数
'''

# abs绝对值
n = abs(-1)
print(n)

# 为False的一些值： 0， None, "", [], (), {}
print(bool(1))


# all函数的参数为一个可迭代的参数，所有的元素为True才为True
n = all([1, 2, 0])
print(n)

# any函数参数也为一个可迭代的参数，只要有一个为True就为True
n = any([1, 2, 0])
print(n)

# ascii函数自动执行对象的__repr__方法
class Foo():
    def __repr__(self):
        return '111'
n = ascii(Foo())
print(n)

# bin函数，将10进制数转换成2进制
print(bin(5))
# oct函数，将10进制数转换成8进制
print(oct(9))
# hex函数，将10进制数转换成16进制
print(hex(16))

# utf-8 一个汉字：三个字节
# gbk  一个汉字：两个字节
'''
字符串转换成字节类型
可以用bytes(要转换成的字符串，按照什么编码)
'''
s = '你好'
n = bytes(s, encoding="utf-8")
print(n) #'\xe4\xbd\xa0\xe5\xa5\xbd'
n= bytes(s, encoding='gbk')
print(n) #'\xc4\xe3\xba\xc3'

# 字节转换成字符串
'''
bytes(s, encoding="utf-8")是字节（类型）
'''
new_str = str(bytes(s, encoding="utf-8"), encoding='utf-8')
print(new_str)
```


### 文件操作

1. 打开文件

```python
# 打开文件
'''
打开文件的模式：
'''
f = open('db', 'r') # 只读模式【默认】
f = open('db', 'w') # 只写模式【不可读；不存在则创建；存在则清空内容；】
f = open('db', 'x') # 只写模式(python3 新加的功能)【不可读；不存在则创建并写内容，存在则报错】
f = open('db', 'a') # 追加模式【可读；不存在则创建；存在则只追加内容；】


'''
二进制方式读写
注：以b方式打开时，读取到的内容是字节类型，写入时也需要提供字节类型
'''
f = open('db', 'rb')
data = f.read()
print(data, type(data)) # b'admin|123' <class 'bytes'>

f = open('db', 'ab')
# f.write('hello') #TypeError: a bytes-like object is required, not 'str'
f.write(bytes('Hello', encoding='utf-8')) # 写二进制有助于跨平台
f.close()

'''
+表示可以同时读写某个文件
'''
f = open('db', 'r+', encoding='utf-8')
data = f.read()
print(data)
f.write('777') # 因为前面已经读了，文件指针位置达到了最后，所以当你写的时候是在末尾加上了内容
f.close()

# 为了在特定的位置开始写，可以用seek函数
f = open('db', 'r+', encoding='utf-8')
# 如果打开模式无b,则read，按照字符读取；
# 因为是r不是rb，所以会读取一个字符；反之读取一个字节
data = f.read(1)
# tell当前指针所在的位置 （字节）
print(f.tell()) # 3,因为第一个是中文，三个字节
print(data)
# 调整当前指针的位置（字节）
f.seek(1) # 无论是r或者rb，都是以字节位置算，所以中文三个字节在重写的时候就会被劈开出现乱码
# 从当前指针位置开始覆盖
f.write('888')
f.close()
'''
注意w+和a+没有r+这么常用，因为w+，往往都会先清空，a+总是会在最后添加内容无论你如何调整指针。
所以当我们想重写（覆盖）一些内容时，不是那么的好用。
'''
```

2. 文件操作

```python
'''
read() # 无参数，读全部内容；有参数，b按字节，无b，按字符读。
tell() # 获取当前指针位置（按字节算）
seek() # 指针跳转到指定位置（按字节算）
write() # 写数据，b，字节；无b, 字符
close() # 关闭文件
fileno() # 文件描述符
flush() # 将缓存中的内容强制刷入（写入）硬盘
readable() # 判断文件是否可读，比如一个文件是以w方式打开的，则是不可读的。
readline() # 仅读取一行
truncate() # 截断，清空文件中指针后面的内容
# 常用的操作
f = open('db', 'r+', encoding='utf-8')
for line in f:
    print(line)
'''
```

3. 关闭文件

```python
'''
文件操作完毕后，可以用
f.close();另外如果是以with打开的，with代码块里的文件操作执行完之后会自动关闭文件。
'''
with open('db', 'r+') as f:
    pass
```

4. 文件操作之with处理上下文

```python
with open('xb') as f:
    pass

# 读一行，写一行
with open('db1', 'r', encoding='utf-8') as f1, open('db2','w', encoding='utf-8') as f2:
    times = 0
    for line in f1:
        times += 1
        print(line)
        if times <= 10:
            f2.write(line)
        else:
            break
```


### 内置函数
1. filter与map的区别
filter: 函数返回True,则将元素添加到结果中
map: 将函数返回值添加到结果中
```python
li = [11, 22, 33, 44, 55]
# filter
result = filter(lambda a: a > 33, li)
print(list(result))
# map
result = map(lambda a:a+100, li)
print(list(result))
```


### 装饰器
```python

def outer(func):
    def inner():
        print('before')
        r = func()
        print(r)
        print('after')
        return r
    return inner


'''
@+函数名， 功能：
1. 自动执行outer函数并且将其下面的函数名f1当作参数传递
2. 将outer函数的返回值，重新赋值给f1
装饰器应用场景：在不改变原函数的前提下，如果你想在执行原函数之前或者之后做一些额外统一的操作。
之后，我们会知道，装饰器会用在定义权限的验证等方面
'''
@outer
def f1():
    print('f1')
    return 'hello'
@outer
def f2():
    print('f2')

@outer
def f100():
    print('f100')
```

### 装饰器流程剖析之参数
```python
#用了装饰器之后，你原来函数(f1)有几个参数，那么装饰器的inner就得定义几个参数
def outer(func):
    def inner(arg):
        print('log')
        r = func(arg)
        print(r)
        print('after')
        return r
    print('first')
    return inner

@outer
def f1(arg):
    print(arg)
    print('f1')
    return 'hello'

f1('miss')
```


### Python的字符串格式化有两种方式: 百分号方式、format方式
1. 百分号方式：%[(name)][flags][width].[precision]typecode

```python
# %s %d
s = 'name %s age %d' % ('jqx', 18)

# (name) 可选 用于选择指定的key
s = 'name %(name)s age %(age)d' % {'name': 'jqx', 'age': 18}
print(s)

# flags: 可供选择的值
# width：占位符所占的宽度
'''
+ 右对齐；正数前加正好，负数前加负号；
- 左对齐；正数前无符号，负数前加负号；
空格 右对齐；正数前加空格，负数前加负号；
0 右对齐；正数前无符号，负数前加负号；用0填充空白处
'''
s = 'name %(name)+10s age %(age)d' % {'name': 'jqx', 'age': 18}
print(s)

#.precision   可选，小数点后保留的位数
s = 'name %(name)+10s age %(age)d  %(p).2f' % {'name': 'jqx', 'age': 18, 'p':1.23456}
print(s)

# typecode    必选
'''
s，获取传入对象的__str__方法的返回值，并将其格式化到指定位置
c，整数：将数字转换成其unicode对应的值，10进制范围为 0 <= i <= 1114111（py27则只支持0-255）；字符：将字符添加到指定位置
o，将整数转换成 八  进制表示，并将其格式化到指定位置
x，将整数转换成十六进制表示，并将其格式化到指定位置
e，将整数、浮点数转换成科学计数法，并将其格式化到指定位置（小写e）
E，将整数、浮点数转换成科学计数法，并将其格式化到指定位置（大写E）
g，自动调整将整数、浮点数转换成 浮点型或科学计数法表示（超过6位数用科学计数法），并将其格式化到指定位置（如果是科学计数则是e；）
f， 将整数、浮点数转换成浮点数表示，并将其格式化到指定位置（默认保留小数点后6位）
%，当字符串中存在格式化标志时，需要用 %%表示一个百分号
'''
s = 'aaaaa %c bbbb %o cccc %x dddd %e' % (65, 15, 15, 1000000000)
print(s)

# 单独测试一下百分号
s1 = 'jqx %'
print(s1) # jqx %
# 当格式化时，字符串中出现占位符%s.. 则需要使用%%输出%
s2 = 'jqx %s %%' % ('xx') # jqx xx %
print(s2)
```

2. Format 方式: [[fill]align][sign][#][0][width][,][.precision][type]

```python
'''
fill  【可选】空白处填充的字符
align 【可选】对齐方式（需配合width使用）
    <，内容左对齐
    >，内容右对齐(默认)
    ＝，内容右对齐，将符号放置在填充字符的左侧，且只对数字类型有效。 即使：符号+填充物+数字
    ^，内容居中
sign  【可选】有无符号数字
    +，正号加正，负号加负；
     -，正号不变，负号加负；
    空格 ，正号空格，负号加负；
#    【可选】对于二进制、八进制、十六进制，如果加上#，会显示 0b/0o/0x，否则不显示
， 【可选】为数字添加分隔符，如：1,000,000
width 【可选】格式化位所占宽度
.precision 【可选】小数位保留精度
type  【可选】格式化类型
    传入” 字符串类型 “的参数
            s，格式化字符串类型数据
            空白，未指定类型，则默认是None，同s
    传入“ 整数类型 ”的参数
            b，将10进制整数自动转换成2进制表示然后格式化
            c，将10进制整数自动转换为其对应的unicode字符
            d，十进制整数
            o，将10进制整数自动转换成8进制表示然后格式化；
            x，将10进制整数自动转换成16进制表示然后格式化（小写x）
            X，将10进制整数自动转换成16进制表示然后格式化（大写X）
    传入“ 浮点型或小数类型 ”的参数
            e， 转换为科学计数法（小写e）表示，然后格式化；
            E， 转换为科学计数法（大写E）表示，然后格式化;
            f ， 转换为浮点型（默认小数点后保留6位）表示，然后格式化；
            F， 转换为浮点型（默认小数点后保留6位）表示，然后格式化；
            g， 自动在e和f中切换
            G， 自动在E和F中切换
            %，显示百分比（默认显示小数点后6位）
'''

s1 = 'name{0}age{1}'.format('jqx', 18)
print(s1)

s2 = '---{name:s}, ---{age:d}==={name:s}'.format(name='jqx', age=18)
print(s2)

s3 = '------{:*^20}======={:+d}-------{:#b}'.format('jqx', 123, 15)
print(s3)

s4 = '-----{:%}'.format(0.123)
print(s4)

```

### 生成器generator
什么是生成器：一个具有生成指定条件数据的能力的一个对象。

我们之前在使用filter函数的时候，用过以下一段代码:
```python
li = [11, 22, 33, 44]
result = filter(lambda x: x > 22, li)
print(result) # result是一个具有生成指定条件数据的能力的一个对象（生成器）
```
同理，看下面一段代码：
```python
'''
range(10)会在内存中生成一个list包含所有的元素，但是试想如果是range(10.....0)，这些在内存
中占用的空间就会很大；所以为了节约内存，我们可以使用xrange()这个函数，这个函数是会生成一个具有生成
数据能力的一个对象（生成器），通过遍历这个对象，我们可以得到所有的数据，并且对于已经生成的那些不会再使用的数据，
python的垃圾回收机制也可以回收，不会占用太多空间。
另外注意：python3中，没有xrange，xrange都被整合到range中了。
'''
In [10]: range(10)
Out[10]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [11]: xrange(10)
Out[11]: xrange(10)

In [12]: for item in xrange(10):
    ...:     print item
    ...:
0
1
2
3
4
5
6
7
8
9
```

生成器由函数创造，请看如下代码：
```python
'''
生成器，使用函数创造
'''
# 普通函数
def func():
    return 123
ret =func()
print(ret)

# 普通的函数中如果加入yield，则这个函数就称之为生成器
def func2():
    print('1111')
    yield 1
    print('2222')
    yield 2
    print('3333')
    yield 3
ret = func2()
print(ret) #<generator object func2 at 0x10212d1a8>

'''
进入函数找到yield，获取yield后面的数据并停止；下次获取下个yield的时候会继续从停止的地方执行.
以下代码也可以用以下代码具有相同的功能:
for i in ret:
    print(i)
'''
r1 = ret.__next__() # 进入函数找到yield，获取yield后面的数据并保存此时代码执行到的位置；相当于只执行了print('1111')和yield 1
print(r1)

r2 = ret.__next__() # 进入函数，从上次保存的位置继续执行，获取下一个yield后面的数据并停止；相当于只执行了print('2222')和yield 1
print(r2)

r3 = ret.__next__() # 同上
print(r3)
```


### 迭代器和生成器的区别

1. 迭代器

迭代器是访问集合元素的一种方式。迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退，不过这也没什么，因为人们很少在迭代途中往后退。另外，迭代器的一大优点是不要求事先准备好整个迭代过程中所有的元素。迭代器仅仅在迭代到某个元素时才计算该元素，而在这之前或之后，元素可以不存在或者被销毁。这个特点使得它特别适合用于遍历一些巨大的或是无限的集合，比如几个G的文件

特点：

访问者不需要关心迭代器内部的结构，仅需通过next()方法不断去取下一个内容
不能随机访问集合中的某个值 ，只能从头到尾依次访问
访问到一半时不能往回退
便于循环比较大的数据集合，节省内存

```python
>>> a = iter([1,2,3,4,5])
>>> a
<list_iterator object at 0x101402630>
>>> a.__next__()
1
>>> a.__next__()
2
>>> a.__next__()
3
>>> a.__next__()
4
>>> a.__next__()
5
>>> a.__next__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

2. 2、生成器

一个函数调用时返回一个迭代器，那这个函数就叫做生成器（generator）；如果函数中包含yield语法，那这个函数就会变成生成器；
```python
def func():
    yield 1
    yield 2
    yield 3
    yield 4
```

上述代码中：func是函数称为生成器，当执行此函数func()时会得到一个迭代器。
```python
>>> temp = func()
>>> temp.__next__()
1
>>> temp.__next__()
2
>>> temp.__next__()
3
>>> temp.__next__()
4
>>> temp.__next__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

### python模块
1. 为什么要有模块？
将代码归类，设计架构

2. 模块导入的依据／规则是什么？
模块必须存在于sys.path中，才能导入成功。否则找不到

3. 模块名称的重要性
不能重名：一定不要创建与内置模块名一样的模块，否则会先去找你自己定义的模块

4. 导入模块方式：
   import
   from xx import xxx
   from xx import *  星号(asterisk)表示导入模块中的所有函数
   from A import c  as a1
   from B import c  as b1

5. 下载第三方模块
   pip下载: pip install xxx
   源码安装：cd 文件解压路径
            python setup.py install


### python序列化之json
```python
'''
序列化之json:
1. json.dumps 可以将基本数据类型转化成字符串
2. json.loads 可以将具有基本数据类型的字符串形式转化成基本数据类型
3. json.dump 先进行序列化，然后写到文件里
4. json.load 从文件里读出来，然后反序列化
'''
# dumps 与 loads
import json
dict = {'k1': 'v1'}
print(dict, type(dict))
# 将python基本数据类型转化成字符串形式
result = json.dumps(dict)
print(result, type(result))


str1 = '{"k1": "v1"}'
print(str1, type(str1))
# 将python字符串转化成基本数据类型
result = json.loads(str1)
print(result, type(result))


str2 = '["v1", "v2"]'
print(str2, type(str2))
result = json.loads(str2)
print(result, type(result))

# dump 与 loads
import json
li = [11, 22, 33]
json.dump(li, open('db', 'w'))
r = json.load(open('db', 'r'))
print(type(r), li)

```
一个基于天气API获取相关JSON数据例子：
```python
import requests
import json

response = requests.get('http://wthrcdn.etouch.cn/weather_mini?city=上海')
'''
#使用apparent_encoding可以获得真实编码
print response.apparent_encoding
'''
response.encoding = 'utf-8'
print(response.text, type(response.text))

dict = json.loads(response.text)
print(dict, type(dict))
```

### python序列化之pickle
```python
# -*- encoding: utf-8 -*-
'''
序列化之pickle:
1. pickle.dumps
2. pickle.loads
3. pickle.dump
4. pickle.load
'''
import pickle

# dumps与loads
li = [11, 22, 33]
ret = pickle.dumps(li)
print(ret, type(ret))

r = pickle.loads(ret)
print(r, type(r))

# dump与load: 只支持以字节的方式去写和读
pickle.dump(li, open('db', 'wb'))

r = pickle.load(open('db', 'rb'))
print(r)
```
### json与pickle的区别
1. pickle可以对所有数据类型做操作（序列化; json只能对基本数据类型做操作（序列化），更适合跨语言
2. pickle的缺点：仅使用于python，并且有可能还得统一python版本

### time和datetime模块

time模块

```python
#_*_coding:utf-8_*_

import time

print(time.clock()) #返回处理器时间,3.3开始已废弃 , 改成了time.process_time()测量处理器运算时间,不包括sleep时间,不稳定,mac上测不出来
print(time.altzone)  #返回与utc时间的时间差,以秒计算\
print(time.asctime()) #返回时间格式"Fri Aug 19 11:14:16 2016",
print(time.localtime()) #返回本地时间 的struct time对象格式
print(time.gmtime(time.time()-800000)) #返回utc时间的struc时间对象格式

print(time.asctime(time.localtime())) #返回时间格式"Fri Aug 19 11:14:16 2016",
#print(time.ctime()) #返回Fri Aug 19 12:38:29 2016 格式, 同上



# 日期字符串 转成  时间戳
string_2_struct = time.strptime("2016/05/22", "%Y/%m/%d") #将 日期字符串 转成 struct时间对象格式
print(string_2_struct)

struct_2_stamp = time.mktime(string_2_struct) #将struct时间对象转成时间戳
print(struct_2_stamp)

#将时间戳转为字符串格式
print(time.gmtime(time.time()-86640)) #将utc时间戳转换成struct_time格式
print(time.strftime("%Y-%m-%d %H:%M:%S",time.gmtime()) ) #将utc struct_time格式转成指定的字符串格式
```

datetime模块

```python
# -*- encoding:utf-8 -*-
'''
python时间处理之datatime模块
'''
import time
import datetime

print(datetime.date.today()) # 输出格式 2018-03-11
print(datetime.date.fromtimestamp(time.time())) # 直接把时间戳转换成日期

current_time = datetime.datetime.now()
print(current_time) # 输出2018-03-11 16:36:17.427490

print(current_time.timetuple()) # 返回struct_time对象格式

# 时间的加减
print(datetime.datetime.now())
print(datetime.datetime.now() + datetime.timedelta(days=10)) # 比现在加10天
print(datetime.datetime.now() + datetime.timedelta(days=-10)) # 比现在减10天
print(datetime.datetime.now() + datetime.timedelta(hours=3, minutes=-4)) # 比现在加3小时和比现在少4分钟

# 时间替换
print(current_time.replace(2014, 9, 12))

print(datetime.datetime.strptime("21/11/06 16:30", "%d/%m/%y %H:%M"))
```

### logging模块
最简单的用法:
很多程序都有记录日志的需求，并且日志中包含的信息即有正常的程序访问日志，还可能有错误、警告等信息输出，python的logging模块提供了标准的日志接口，你可以通过它存储各种格式的日志，logging的日志可以分为 debug(), info(), warning(), error() and critical() 5个级别，下面我们看一下怎么用。

```python
import logging

logging.warning("user [jqx] attempted wrong password more than 3 times")
logging.critical("server is down")

#输出
WARNING:root:user [jqx] attempted wrong password more than 3 times
CRITICAL:root:server is down
```

看一下这几个日志级别分别代表什么意思
1. DEBUG: Detailed information, typically of interest only when diagnosing problems.
2. INFO: Confirmation that things are working as expected.
3. WARNING: An indication that something unexpected happened, or indicative of some problem in the near future (e.g. ‘disk space low’). The software is still working as expected.
4. ERROR: Due to a more serious problem, the software has not been able to perform some function.
5. CRITICAL: A serious error, indicating that the program itself may be unable to continue running.

如果想把日志写到文件中
```python
import logging

logging.basicConfig(filename='example.log',level=logging.INFO) # 比这个level级别相同或者更严重的日志才会写入文件中
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
```

如果要给日志加上对应的时间
```python
import logging
logging.basicConfig(format='%(asctime)s %(message)s', datefmt='%m/%d/%Y %I:%M:%S %p')
logging.warning('is when this event was logged.')

#输出
12/12/2010 11:46:36 AM is when this event was logged.
```
其他日志格式
<table>
<tr>
<td>%(name)s</td>
<td>Logger的名字</td>
</tr>
<tr>
<td>%(levelno)s</td>
<td>数字形式的日志级别</td>
</tr>
<tr>
<td>%(levelname)s</td>
<td>文本形式的日志级别</td>
</tr>
<tr>
<td>%(pathname)s</td>
<td>调用日志输出函数的模块的完整路径名，可能没有</td>
</tr>
<tr>
<td>%(filename)s</td>
<td>调用日志输出函数的模块的文件名</td>
</tr>
<tr>
<td>%(module)s</td>
<td>调用日志输出函数的模块名</td>
</tr>
<tr>
<td>%(funcName)s</td>
<td>调用日志输出函数的函数名</td>
</tr>
<tr>
<td>%(lineno)d</td>
<td>调用日志输出函数的语句所在的代码行</td>
</tr>
<tr>
<td>%(created)f</td>
<td>当前时间，用UNIX标准的表示时间的浮 点数表示</td>
</tr>
<tr>
<td>%(relativeCreated)d</td>
<td>输出日志信息时的，自Logger创建以 来的毫秒数</td>
</tr>
<tr>
<td>%(asctime)s</td>
<td>字符串形式的当前时间。默认格式是 “2003-07-08 16:49:45,896”。逗号后面的是毫秒</td>
</tr>
<tr>
<td>%(thread)d</td>
<td>线程ID。可能没有</td>
</tr>
<tr>
<td>%(threadName)s</td>
<td>线程名。可能没有</td>
</tr>
<tr>
<td>%(process)d</td>
<td>进程ID。可能没有</td>
</tr>
<tr>
<td>%(message)s</td>
<td>用户输出的消息</td>
</tr>
</table>

如果想同时把log打印在屏幕和文件日志里，就需要了解一点复杂的知识:


Python 使用logging模块记录日志涉及四个主要类，使用官方文档中的概括最为合适：

logger提供了应用程序可以直接使用的接口；

handler将(logger创建的)日志记录发送到合适的目的输出；

filter提供了细度设备来决定输出哪条日志记录；

formatter决定日志记录的最终输出格式。

```python
'''
logger模块进阶
'''
import logging

# create logger
logger = logging.getLogger('TEST-LOG') # 先获取logger对象
logger.setLevel(logging.DEBUG) # 设置全局日志级别(全局优先级别高)

# create console handler and set level to debug
ch = logging.StreamHandler() # print the log on monitor
ch.setLevel(logging.DEBUG)

# create file handler and set level to warning
fh = logging.FileHandler('access.log')
fh.setLevel(logging.WARNING)

# create formatter
formatter_for_console = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
formatter_for_file = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
ch.setFormatter(formatter_for_console)
fh.setFormatter(formatter_for_file)

# add ch and fh to logger
logger.addHandler(ch) # 告诉logger输出日志到某个给定的handler上
logger.addHandler(fh)

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')
```


### 反射 Reflection
反射最初是由web http(get, post, ...)请求中衍生而来的
实例: 伪造Web框架的路由系统
反射: 基于字符串的形式去对象(模块)中操作(寻找/检查/删除/设置)其成员
    getattr, delattr, setattr, hasattr
扩展: 导入模块
    import xxx
    from xxx import xxx
    obj = __import__('xxx')
    obj = __import__('xxx.ooo.xxx', fromlist=True)

```python
#import manager
#import account
#import commons
# obj = __import__('commons')
# obj.login()

def run():
    # 输入'commons/login'
    inp = input('请输入要访问的url:')
    # 利用字符串的形式去对象(模块)中操作(寻找/检查/删除/设置)成员 --->反射
    #delattr(commons, inp) 基于内存
    #setattr(commons, inp) 基于内存

    m, f = inp.split('/') # 以字符串的形式导入了模块，还以字符串的形式找到了函数
    # obj = __import__('lib.' + m, fromlist=True)
    print(obj)

    if hasattr(obj, f):
        func = getattr(obj, f)
        func()
    else:
        print('404')

if __name__ == '__main__':
    run()
```


### 模块中特殊变量

```python
import s2
from bin import admin
#print(vars(s2))
# 查看一些内置变量与函数，如abs,...
print(s2.__dict__)

# __doc__  获取文件的注释（文件顶头三引号中的注释）
print(__doc__)

# __file__
print(__file__)  # 表示py文件所在的路径（见bin/admin）

# __package__
print(__package__) # None
print(admin.__package__) # bin

# __cached__ python在导入某个模块时，在__pycache__文件夹中会自动生成其pyc文件

# 执行当前文件的时候，当前文件特殊变量__name__ == "__main__"
print(__name__)


def func():
    print('main file')


if __name__ == "main":
    func()
```

### sys模块
用于提供对python解释器相关的操作:
```python
import sys
sys.argv           #命令行参数List，第一个元素是程序本身路径
sys.exit(n)        #退出程序，正常退出时exit(0)
sys.version        #获取Python解释程序的版本信息
sys.maxint         #最大的Int值
sys.path           #返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
sys.platform       #返回操作系统平台名称
sys.stdout.write('please:')
val = sys.stdin.readline()[:-1]
```

进度条例子

```python
# 进度条
import sys
import time
def viewbar(num, total):
    rate = num/total
    rate_number = int(rate*100)
    r = '\r%s>%d%%' % (num*'=',rate_number, ) #\r表示重新回到当前行的首个位置
    #print(r) # print默认加换行符
    sys.stdout.write(r) # write不加换行符
    sys.stdout.flush() # 清空输出


if __name__ == "__main__":
    for i in range(0, 101):
        time.sleep(0.1)
        viewbar(i, 100)
```


### 加密模块 hashlib
用于加密相关的操作，3.x里代替了md5模块和sha模块，主要提供 SHA1, SHA224, SHA256, SHA384, SHA512 ，MD5 算法
```python
# -*- encoding: utf-8 -*-

import hashlib

md5_obj = hashlib.md5()
# python 3.x 需要将字符串转换成字节
md5_obj.update(bytes('123', encoding='utf-8'))
result = md5_obj.hexdigest()
# '123'加密后成了md5值:202cb962ac59075b964b07152d234b70
print(result)

'''
如果想在上面的基础上在加密的话，可以如下操作
'''
md5_obj = hashlib.md5(bytes('abc', encoding='utf-8'))
md5_obj.update(bytes('123', encoding='utf-8'))
result = md5_obj.hexdigest()
# 这样的话'123'加密后成了e99a18c428cb38d5f260853678922e03，这样就相对与上面更安全了，因为每个人创建md5时给的
# 那个值是可以不同的。
print(result)
```


### 正则表达式 re模块
常用正则表达式符号

'.'     默认匹配除\n之外的任意一个字符，若指定flag DOTALL,则匹配任意字符，包括换行
'^'     匹配字符开头，若指定flags MULTILINE,这种也可以匹配上(r"^a","\nabc\neee",flags=re.MULTILINE)
'$'     匹配字符结尾，或e.search("foo$","bfoo\nsdfsf",flags=re.MULTILINE).group()也可以
'*'     匹配*号前的字符0次或多次，re.findall("ab*","cabb3abcbbac")  结果为['abb', 'ab', 'a']
'+'     匹配前一个字符1次或多次，re.findall("ab+","ab+cd+abb+bba") 结果['ab', 'abb']
'?'     匹配前一个字符1次或0次
'{m}'   匹配前一个字符m次
'{n,m}' 匹配前一个字符n到m次，re.findall("ab{1,3}","abb abc abbcbbb") 结果'abb', 'ab', 'abb']
'|'     匹配|左或|右的字符，re.search("abc|ABC","ABCBabcCD").group() 结果'ABC'
'(...)' 分组匹配，re.search("(abc){2}a(123|456)c", "abcabca456c").group() 结果 abcabca456c


'\A'    只从字符开头匹配，re.search("\Aabc","alexabc") 是匹配不到的
'\Z'    匹配字符结尾，同$
'\d'    匹配数字0-9
'\D'    匹配非数字
'\w'    匹配[A-Za-z0-9]
'\W'    匹配非[A-Za-z0-9]
's'     匹配空白字符、\t、\n、\r , re.search("\s+","ab\tc1\n3").group() 结果 '\t'

'(?P<name>...)' 分组匹配 re.search("(?P<province>[0-9]{4})(?P<city>[0-9]{2})(?P<birthday>[0-9]{4})","371481199306143242").groupdict("city") 结果{'province': '3714', 'city': '81', 'birthday': '1993'}

re模块中的函数

1. re.match: 只从字符串的起始位置开始匹配
          如果匹配成功，会得到一个match对象
2. re.search: 只会找到第一个的匹配
3. re.findall: 把所有匹配到的字符放到以列表中的元素返回
4. re.finditer: 与findall的唯一区别就是：findall返回一个列表，finditer返回一个迭代对象
5. re.sub(pattern, repl, string, max=0): 找到并替换字符串
6. re.split
7. re.compile
8. 有一些函数有flag，这表示了匹配模式:
    re.I 使匹配对大小写敏感
    re.L 做本地化识别(local-ware)匹配
    re.M 多行匹配，影响^和$
    re.S 使.匹配包括换行在内的所有字符

反斜杠的困扰<br>
与大多数编程语言相同，正则表达式里使用"\"作为转义字符，这就可能造成反斜杠困扰。假如你需要匹配文本中的字符"\"，那么使用编程语言表示的正则表达式里将需要4个反斜杠"\\\\"：前两个和后两个分别用于在编程语言里转义成反斜杠，转换成两个反斜杠后再在正则表达式里转义成一个反斜杠。Python里的原生字符串很好地解决了这个问题，这个例子中的正则表达式可以使用r"\\"表示。同样，匹配一个数字的"\\d"可以写成r"\d"。有了原生字符串，你再也不用担心是不是漏写了反斜杠，写出来的表达式也更直观

```pyhton
# -*- encoding: utf-8 -*-
'''
正则表达式模块_2
'''

import re
a = re.sub('g.t', 'have', 'I get one, I got one', 1)
# I have one, I got one
print(a)


b = re.split('\d+', 'one1two2three3four4')
['one', 'two', 'three', 'four', '']
print(b)

text = 'JGood is a handsome boy, he is cool, clever, and so on...'
regex = re.compile('\w*oo\w*')
c = regex.findall(text)
# ['JGood', 'cool']
print(c)

# re模块需要接受2个反斜杠，才能转成1个匹配字符串的反斜杠，所以得给python传4个，python解释器
# 转成2个传给re模块
d = re.findall('\\\\com', 'abc\com')
print(d)

# 为了解决上面反斜杠的困扰，我们可以使用原生字符串
e = re.findall(r'\\com', 'abc\com')
print(e)
```

### 正则分组匹配

分组匹配的实质就是去匹配到的数据中再去提取数据。

```python
# -*- encoding: utf-8 -*-
'''
正则分组匹配
'''

import re

origin = 'has shasdasda12312s'

# 无分组
r = re.match('h\w+', origin)
print(r.group())  # 获取匹配到的所有结果 has
print(r.groups()) # 获取模型中匹配到的分组结果 ()
print(r.groupdict()) # 获取模型中匹配到的分组结果 {}

# 有分组
r = re.match('h(\w+)', origin)
print(r.group())  # 获取匹配到的所有结果  has
print(r.groups()) # 获取模型中匹配到的分组结果 ('as',)
print(r.groupdict()) # 获取模型中匹配到的分组结果 {}

# 有分组，并且对分组设置一个key，这样groupdict就有值了
r = re.match('h(?P<name>\w+)', origin)
print(r.group())  # 获取匹配到的所有结果  has
print(r.groups()) # 获取模型中匹配到的分组结果 ('as',)
print(r.groupdict()) # 获取模型中匹配到的分组结果  {'name': 'as'}

# findall函数的分组匹配
origin = 'has sdasd hal 12123'
r = re.findall('h(\w+)', origin)
print(r) # 只返回分组的内容，不会匹配所有的，所以结果是 ['as', 'al']


origin = 'hasaabc sdasd halaabc 12123'
r = re.findall('h(\w+)a(ab)c', origin)
print(r) # [('as', 'ab'), ('al', 'ab')]

# sub函数与分组匹配无关

'''
split函数无分组与分组对比
'''
# 无分组
origin = 'hello alex bcd alex lge alex acd 19'
r = re.split('alex', origin, 1)
print(r) # ['hello ', ' bcd alex lge alex acd 19']

# 有分组
r = re.split('a(le)x', origin, 1)
print(r) # ['hello ', 'le', ' bcd alex lge alex acd 19'] 括号表示要从匹配到的值中拿取分组中所有的值
```

---
title: "python object-oriented programming"
layout: post
date: 2018-01-02 22:45
tag:
- python
- Object-Oriented Programming
star: true
category: blog
author: jiaqixu
---

Reference: https://github.com/jiaqi-xu/PythonLearning

### 面向对象基础

1. python支持函数式编程 + 面向对象编程
2. 函数式编程，面向对象编程实现: 发送邮件的功能

```python
# 函数
def mail(email, message):
  print('发送邮件')
  return True

mail('jxu033@uottawa.ca', 'good job')

# 面向对象: 类，对象
class Foo:
  # 方法
  def mail(self, email, message):
    print('发送邮件')
    return True

# 调用，1.创建对象，通过类名()
obj = Foo()
# 2. 通过对象去执行方法
obj.mail('jxu033@uottawa.ca', 'good job')
```

3. 类和对象

```python
  a. 创建类
    class 类名:

      def 方法名(self, 参数):
        pass
  b. 创建对象
    对象 = 类名()
  c. 通过对象执行方法
    对象.方法名(参数)
```

4. 函数式编程实现CRUD

```python
  def fetch(host, username, password, sql):
    pass
  def create(host, username, password, sql):
    pass
  def remove(host, username, password, nid):
    pass
  def modify(host, username, password, name):
    pass
``

  面向对象实现CRUD
```python
  class SQLHelper:

      def fetch(self, sql):
        # 连接数据库
        pass
      def create(self, sql):
        pass
      def remove(self, nid):
        pass
      def modify(self, name):
        pass

  obj = SQLHelper()
  obj.host = 'cl.salt.com'
  obj.user_name = 'jqx'
  obj.pwd = '123'
  obj.fetch('select * from A')
```

由上面例子可知，当某一些函数具有相同参数时，可以使用面向对象的方式，将参数值一次性的封装到对象，以后去对象中取指即可。

5. self是什么？
    self是一个python自动会给传值的参数
    简单而言:哪个对象执行方法，self就是谁。

```python
    obj1.fetch('selce..') #self = obj1
    obj2.fetch('selce..') #self = obj2
```

6. 面向对象之构造方法
 类中有一个特殊的方法 __init__,叫其构造方法，类()自动被执行。
 当构造对象时，会默认调用构造函数__init__，所以可以在构造函数中进行对一些成员变量的初始化。

```python
class SQLHelper:
    def __init__(self, a1, a2, a3)
      print('自动执行__init')
      self.host= a1
      self.user_name = a2
      self.pwd = a3

    def fetch(self, sql):
       # 连接数据库
       pass
    def create(self, sql):
       pass
    def remove(self, nid):
       pass
    def modify(self, name):
       pass

obj_1 = SQLHelper('c1.salt.com', 'jqx', '123')
obj_1.fetch('select * from A')
#obj_2 = SQLHelper('c2.salt.com', 'jqx2', '12333')
```

7. 面向对象
  三大特性： 封装，继承，多态(多态在python这种轻类型语言中没什么意思，在java,c#这种强类型语言中很有用)

8. 对象中封装对象

```python
# 封装
class c1:

    def __init__(self, name, obj):
        self.name = name
        self.obj = obj


class c2:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def show(self):
        print(self.name)


class c3:

    def __init__(self, a1):
        self.money = 123
        self.aaa = a1


c2_obj = c2('aa', 11)
c1_obj = c1('bb', c2_obj)
print(c1_obj.obj.name)
c3_obj = c3(c1_obj)
print(c3_obj.aaa)
```

9. 面向对象之继承

```python
# 单继承
class F1: # 父类(aka. 基类)
    def show(self):
        print('show')

    def foo(self):
        print(self.name)


class F2(F1): # 子类(aka. 派生类)
    def __init__(self, name):
        self.name = name

    def bar(self):
        print('bar')

    def show(self):
        print('F2.show')

class F3(F2):
    pass


obj = F2('jqx')
obj.show()
obj.bar()
obj.foo()
```

```python
# 多继承(在没有共同顶层父类的情况下，深度优先寻找要执行的函数)
class C0:
    def f2(self):
        print('C0.f2')

class C1(C0):
    def f2(self):
        print('C1.f2')

class C2:
    def f2(self):
        print('C2.f2')

class C3(C1, C2):
    def f3(self):
        print('C3.f3')

obj = C3()
# 会执行C1的f2，因为继承的时候，先继承的C1，如果C1中没有f2，则会去C0中找f2。
obj.f2()
```

10. 面向对象之多态

```python
def func(arg):
             print(arg)

          func(1)
          func('jqx')
          func([11, 22, 33])

          C#/Java: 可以使用类的继承关系来表现多态
          void func(int arg){
                print(arg);
          }

          func(123)
          func('jqx') # 报错

          Class A:
                pass

          Class B(A):
                pass

          Class C(A):
                pass

          # arg参数: 必须是A类型或者A的子类类型
          def func(A arg):
                print(arg)

          #obj = B()
          #obj = C()
          obj = A()
          func(obj)
```

11. pickle load对象会出现的问题

如果在文件中不引入那个对象所属的类，当执行该文件时，那个类当然是不存在内存中，所以load的时候
会失败

```python
AttributeError: Can't get attribute 'Foo' on <module '__main__' from '/Users/jiaqixu/PycharmProjects/PythonLearning/面向对象基础/phase-8/pickle常见问题_2.py'>
出错的原因是，加载对象的时候，对象的类没有写在内存中；想要load成功，必须将类加载到内存：
1. 可以直接将类写在此文件里
2. 可以from pickle常见问题_1 import Foo
```

```python
# -*- encoding: utf-8 -*-
'''
pickle load对象的时候会出现的问题
'''

import pickle
from pickle常见问题_1 import Foo
# class Foo:
#     def __init__(self, name):
#         self.name = name
#
#     def show(self):
#         print(self.name)

re = pickle.load(open('db', 'rb'))
print(re)
```


12. 类成员之普通字段和静态字段
一般的情况下：自己访问自己的字段
规则:
   普通字段只能用对象去访问
   静态字段用类访问(万不得已的时候可以用对象去访问）
   类加载的时候，静态字段就已经创建
```python
'''
面向对象之类成员
'''

class Foo():
    # 字段(静态字段)
    CC = 123

    def __init__(self):
        # 字段(普通的字段)
        self.name = 'jqx'

    def show(self):
        print(self.name)


class Province:

    country = 'China'

    def __init__(self, name):
        self.name = name
        # 每个对象都有这个country字段，为了节约内存，可以将该字段作为静态字段(让所有对象共享一份)
        # self.country = 'China'

hn = Province('HeNan')
hb = Province('HeBei')
sd = Province('ShanDong')
db = Province('HeiLongjiang')
```

13. 类成员之普通方法，静态方法和类方法

所有的方法属于类
1. 普通方法: 至少有一个self, 通过对象执行
2. 静态方法: 任意参数(可有可无), 由类执行(其实也可以用对象去执行但不推荐)
3. 类方法: 至少有一个cls， 由类执行(其实也可以用对象去执行但不推荐)

```python
class Province:

    country = 'China'

    def __init__(self, name):
        self.name = name

    # 普通方法，由对象调用执行(方法属于类)
    def show(self):
        print(self.name)

    @staticmethod
    def f1(arg1, arg2):
        # 静态方法, 由类调用执行
        print('static method')
        print(arg1,arg2)

    @classmethod
    def f2(cls): # cls的全拼是class
        # cls是类名，cls()是创建对象
        # 类方法，至少要有一个参数叫做cls，类方法是静态方法的一种
        print(cls)
```

14. 类成员之属性
属性: 不伦不类的东西，具有方法的表现形式，具有字段的访问形式
     i.e. 调用property(男儿身(方法)，通过女性(字段)方式访问), 完全伪造成字段的样子(
     像字段一样可以获取，设置，删除值)
     也可以说属性提供了一种关联方式，具体做什么都是自己在方法中定义的。

```python
'''
属性
'''
class Pager:
    def __init__(self, all_count):
        self.all_count = all_count

    @property
    def all_pager(self):
        a1, a2 = divmod(self.all_count, 10)
        if a2 ==0:
            return a1
        else:
            return a1+1

    @all_pager.setter
    def all_pager(self, value):
        print(value)

    @all_pager.deleter
    def all_pager(self):
        # del self.all_pager
        print('del all_pager')

p = Pager(101)
# 获取属性（）
result = p.all_pager
print(result)
# 设置值
p.all_pager = '100'
# 删除
del p.all_pager
```

15. 以上是属性的一种表现形式，现在再介绍一种属性的表现形式

```python
'''
属性的另外一种表达方法
'''
class Pager:

    def __init__(self, all_count):
        self.all_count = all_count

    def f1(self):
        return 123

    def f2(self, value):
        print('set value %s' % value)

    def f3(self):
        print('del value')

    foo = property(fget=f1, fset=f2, fdel=f3)

p = Pager(101)
result = p.foo
print(result)
p.foo = 'jqx'
del p.foo
```

16. 类成员之成员修饰符
    私有:
        只能类本身成员内部可以访问
    公有:
        正常访问
    PS: python变态，说是分私有公有，但是其实，我们可以通过'对象._类名私有变量名'去访问私有变量（不推荐）
        不到万不得已，不要在外部强制访问私有变量

```python
# -*- encoding: utf-8 -*-
'''
类成员之成员修饰符
如果在字段前面加两个下划线__，就可以将该字段设置为私有变量
'''
class Foo:

    #定义一个静态字段
    __cc = '123'

    def __init__(self, name, age):
        # 公有变量
        self.name = name
        # 私有变量
        self.__age = age

    def f1(self):
        print(self.name)
        print(self.__age)

    def f3(self):
        print(Foo.__cc)

    @staticmethod
    def f4():
        print(Foo.__cc)

obj = Foo('jqx', 18)
print(obj.name)
# 因为age是私有的，所以只能在内部访问，在外部无法访问
#print(obj.age)
# f1是内部函数，可以通过他访问私有变量
obj.f1()

#print(Foo.__cc) # AttributeError: type object 'Foo' has no attribute '__cc'
obj.f3() # 私有的静态字段也必须通过内部访问
Foo.f4() # 也可以通过静态方法调用私有静态字段
# print(obj._Foo__cc) 不到万不得以，不得强制访问

class Bar(Foo):
    def f2(self):
        print(self.name)
        print(self.__age)

obj_f2 = Bar('jqx_f2', 19)
# 会报错，看来子类的内部函数无法访问父类的私有变量
# 其实，在python私有变量，只有他本身才可以访问，其他谁来都不行
# obj_f2.f2()  # AttributeError: 'Bar' object has no attribute '_Bar__age'
```

17. 类的特殊成员
上文介绍了Python的类成员以及成员修饰符，从而了解到类中有字段、方法和属性三大类成员，并且成员名前如果有两个下划线，则表示该成员是私有成员，私有成员只能由类内部调用。无论人或事物往往都有不按套路出牌的情况，Python的类成员也是如此，存在着一些具有特殊含义的成员，详情如下：


1. __doc__: 表示类的描述信息

```python
class Foo:
    """ 描述类信息，这是用于看片的神奇 """

    def func(self):
        pass

print Foo.__doc__
#输出：类的描述信息
```
2. __module__ 和  __class__:
    __module__ 表示当前操作的对象在那个模块
　 　__class__     表示当前操作的对象的类是什么

```python
# lib.aa.py
class C:

    def __init__(self):
        self.name = 'jqx'
```
```python
from lib.aa import C

obj = C()
print obj.__module__  # 输出 lib.aa，即：输出模块
print obj.__class__      # 输出 lib.aa.C，即：输出类
```

3. __init__: 构造方法，通过类创建对象时，自动触发执行

```python
class Foo:

    def __init__(self, name):
        self.name = name
        self.age = 18


obj = Foo('jqx') # 自动执行类中的 __init__ 方法
```

4. __del__: 析构方法，当对象在内存中被释放时，自动触发执行。

注：此方法一般无须定义，因为Python是一门高级语言，程序员在使用时无需关心内存的分配和释放，因为此工作都是交给Python解释器来执行，所以，析构函数的调用是由解释器在进行垃圾回收时自动触发执行的。

```python
class Foo:

    def __del__(self):
        pass
```

5. __call__: 对象后面加括号，触发执行。

注：构造方法的执行是由创建对象触发的，即：对象 = 类名() ；而对于 __call__ 方法的执行是由对象后加括号触发的，即：对象() 或者 类()()

```python
class Foo:

    def __init__(self):
        pass

    def __call__(self, *args, **kwargs):

        print '__call__'


obj = Foo() # 执行 __init__
obj()       # 执行 __call__
```

6.  __dict__: 类或对象中的所有成员,我们知道：类的普通字段属于对象；类中的静态字段和方法等属于类

```python
class Province:

    country = 'China'

    def __init__(self, name, count):
        self.name = name
        self.count = count

    def func(self, *args, **kwargs):
        print 'func'

# 获取类的成员，即：静态字段、方法、
print Province.__dict__
# 输出：{'country': 'China', '__module__': '__main__', 'func': <function func at 0x10be30f50>, '__init__': <function __init__ at 0x10be30ed8>, '__doc__': None}

obj1 = Province('HeBei',10000)
print obj1.__dict__
# 获取 对象obj1 的成员
# 输出：{'count': 10000, 'name': 'HeBei'}

obj2 = Province('HeNan', 3888)
print obj2.__dict__
# 获取 对象obj1 的成员
# 输出：{'count': 3888, 'name': 'HeNan'}
```

7. __str__: 如果一个类中定义了__str__方法，那么在打印 对象 时，默认输出该方法的返回值。

```python
class Foo:

    def __str__(self):
        return 'jqx'


obj = Foo()
print obj
# 输出：jqx
# 也可以用str(obj)触发__str__方法
```

8. __getitem__、__setitem__、__delitem__: 用于索引操作，如字典。以上分别表示获取、设置、删除数据

```python
class Foo(object):

    def __getitem__(self, key):
        print '__getitem__',key

    def __setitem__(self, key, value):
        print '__setitem__',key,value

    def __delitem__(self, key):
        print '__delitem__',key


obj = Foo()

result = obj['k1']      # 自动触发执行 __getitem__
obj['k2'] = 'xujiaqi'   # 自动触发执行 __setitem__
del obj['k1']           # 自动触发执行 __delitem__
```

9. __getslice__、__setslice__、__delslice__: 该三个方法用于分片操作，如：列表

```python
'''
python3以后，这三个方法就被遗弃了
'''
class Foo(object):

    def __getslice__(self, i, j):
        print '__getslice__',i,j

    def __setslice__(self, i, j, sequence):
        print '__setslice__',i,j

    def __delslice__(self, i, j):
        print '__delslice__',i,j

obj = Foo()

obj[-1:1]                   # 自动触发执行 __getslice__
obj[0:1] = [11,22,33,44]    # 自动触发执行 __setslice__
del obj[0:2]                # 自动触发执行 __delslice__

```

10. __iter__: 用于迭代器，之所以列表、字典、元组可以进行for循环，是因为类型内部定义了 __iter__

```python
class Foo(object):
    pass


obj = Foo()

for i in obj:
    print i

# 报错：TypeError: 'Foo' object is not iterable
# ---------------------------------------------
class Foo(object):

    def __iter__(self):
        pass

obj = Foo()

for i in obj:
    print i

# 报错：TypeError: iter() returned non-iterator of type 'NoneType'
# ---------------------------------------------
class Foo(object):

    def __init__(self, sq):
        self.sq = sq

    def __iter__(self):
        return iter(self.sq)

obj = Foo([11,22,33,44])

for i in obj:
    print i
```

以上步骤可以看出，for循环迭代的其实是  iter([11,22,33,44]) ，所以执行流程可以变更为：
```python
obj = iter([11,22,33,44])

for i in obj:
    print i
# -----------------------------
# for循环语法内部
obj = iter([11,22,33,44])

while True:
    val = obj.next()
    print val
```

11. __new__ 和 __metaclass__

阅读以下代码:

```python
class Foo(object):

    def __init__(self):
        pass

obj = Foo()   # obj是通过Foo类实例化的对象
```
上述代码中，obj 是通过 Foo 类实例化的对象，其实，不仅 obj 是一个对象，Foo类本身也是一个对象，因为在Python中一切事物都是对象。

如果按照一切事物都是对象的理论：obj对象是通过执行Foo类的构造方法创建，那么Foo类对象应该也是通过执行某个类的 构造方法 创建。

```python
print type(obj) # 输出：<class '__main__.Foo'>     表示，obj 对象由Foo类创建
print type(Foo) # 输出：<type 'type'>              表示，Foo类对象由 type 类创建
```

所以，obj对象是Foo类的一个实例，Foo类对象是 type 类的一个实例，即：Foo类对象 是通过type类的构造方法创建。

那么，创建类就可以有两种方式：
a). 普通方式

```python
class Foo(object):

    def func(self):
        print 'hello xujiaqi'
```

b).特殊方式（type类的构造函数）

```python
def func(self):
    print 'hello xujiaqi'

Foo = type('Foo',(object,), {'func': func})
#type第一个参数：类名
#type第二个参数：当前类的基类
#type第三个参数：类的成员
```

＝＝》 类 是由 type 类实例化产生

那么问题来了，类默认是由 type 类实例化产生，type类中如何实现的创建类？类又是如何创建对象？

答：类中有一个属性 \__metaclass__，其用来表示该类由 谁 来实例化创建，所以，我们可以为 \__metaclass__ 设置一个type类的派生类，从而查看 类 创建的过程。

<img class="image" src="/assets/images/blog/metaclass.jpg">


17. isinstance和issubclass

```python
'''
isinstance, issubclass
'''
class Bar:
    pass

class Foo(Bar):
    pass

obj = Foo()
# obj, Bar
ret = isinstance(obj, Foo)
ret2 = isinstance(obj, Bar)
print(ret)
print(ret2)

ret3 = issubclass(Foo, Bar)
print(ret3)
```

18. 使用super调用父类的方法

```python
class C1:

    def f1(self):
        print('c1.f1')


class C2(C1):

    def f1(self):
        # 执行父类的f1方法
        super(C2, self).f1()
        print('c2.f1')
        C1.f1(self) # python3.0之后不建议使用这种方式(因为f1通常得用对象去访问，但是这里传了self也可以)

obj = C2()
obj.f1()
```

19. 自定义有序字典

```python

class MyDict(dict):

    def __init__(self):
        self.li = []
        super(MyDict, self).__init__()

    def __setitem__(self, key, value):
        self.li.append(key)
        super(MyDict, self).__setitem__(key, value)

    # 重写str,按顺序输出字典
    def __str__(self):
        temp_list = []
        for key in self.li:
            value = self.get(key)
            temp_list.append("'%s':'%s'" % (key, value))

        temp_str = "{" + ",".join(temp_list) + "}"
        return temp_str


obj = MyDict()
obj['k1'] = 'v1'
obj['k2'] = 'v2'
print(obj)
```

19. 单例模式

```python
'''
Singleton设计模式，应用场景：JDBC, 数据库连接池，通过单例模式使其只创建一份连接池
'''


class Foo:

    instance = None

    def __init__(self, name):
        self.name = name

    @classmethod
    def get_instance(cls):
        # cls 类名
        if cls.instance:
            return cls.instance
        else:
            obj = cls('jqx')
            cls.instance = obj
            return obj


# 这样的话，obj和obj_2是一样的
obj = Foo.get_instance()
obj_2 = Foo.get_instance()
print(obj)
print(obj_2)
```

20. 基本异常处理

在编程过程中为了增加友好性，在程序出现bug时一般不会将错误信息显示给用户，而是现实一个提示的页面，通俗来说就是不让用户看见大黄页！！！

```python
while True:
    num1 = input('num1:')
    num2 = input('num2:')
    try:
        result = int(num1) + int(num2)
    except Exception as ex: # ex 是 Exception的一个对象，封装了错误信息
        print(ex)
```

python中的异常种类非常多，每个异常专门用于处理某一项异常！！！

常用异常：
```text
AttributeError 试图访问一个对象没有的树形，比如foo.x，但是foo没有属性x
IOError 输入/输出异常；基本上是无法打开文件
ImportError 无法引入模块或包；基本上是路径问题或名称错误
IndentationError 语法错误（的子类） ；代码没有正确对齐
IndexError 下标索引超出序列边界，比如当x只有三个元素，却试图访问x[5]
KeyError 试图访问字典里不存在的键
KeyboardInterrupt Ctrl+C被按下
NameError 使用一个还未被赋予对象的变量
SyntaxError Python代码非法，代码不能编译(个人认为这是语法错误，写错了）
TypeError 传入对象类型与要求的不符合
UnboundLocalError 试图访问一个还未被设置的局部变量，基本上是由于另有一个同名的全局变量，
导致你以为正在访问它
ValueError 传入一个调用者不期望的值，即使值的类型是正确的
```

更多异常：
```python
ArithmeticError
AssertionError
AttributeError
BaseException
BufferError
BytesWarning
DeprecationWarning
EnvironmentError
EOFError
Exception
FloatingPointError
FutureWarning
GeneratorExit
ImportError
ImportWarning
IndentationError
IndexError
IOError
KeyboardInterrupt
KeyError
LookupError
MemoryError
NameError
NotImplementedError
OSError
OverflowError
PendingDeprecationWarning
ReferenceError
RuntimeError
RuntimeWarning
StandardError
StopIteration
SyntaxError
SyntaxWarning
SystemError
SystemExit
TabError
TypeError
UnboundLocalError
UnicodeDecodeError
UnicodeEncodeError
UnicodeError
UnicodeTranslateError
UnicodeWarning
UserWarning
ValueError
Warning
ZeroDivisionError
```

异常类只能用来处理指定的异常情况，如果非指定异常则无法处理。

```python
# 未捕获到异常，程序直接报错

s1 = 'hello'
try:
    int(s1)
except IndexError,e:
    print e
```

所以，写程序时需要考虑到try代码块中可能出现的任意异常，可以这样写：

```python
s1 = 'hello'
try:
    int(s1)
except IndexError,e:
    print e
except KeyError,e:
    print e
except ValueError,e:
    print e
```

万能异常 在python的异常中，有一个万能异常：Exception，他可以捕获任意异常，即：

```python
s1 = 'hello'
try:
    int(s1)
except Exception,e:
    print e
```

接下来你可能要问了，既然有这个万能异常，其他异常是不是就可以忽略了！

答：当然不是，对于特殊处理或提醒的异常需要先定义，最后定义Exception来确保程序正常运行。

```python
s1 = 'hello'
try:
    int(s1)
except KeyError,e:
    print '键错误'
except IndexError,e:
    print '索引错误'
except Exception, e:
    print '错误'
```

异常其他结构

```python
try:
    # 主代码块
    pass
except KeyError,e:
    # 异常时，执行该块
    pass
else:
    # 主代码块执行完，执行该块; 报错则不执行
    pass
finally:
    # 无论异常与否，最终执行该块
    pass
```

主动触发异常

```python
# 主动抛出异常
try:
    raise ValueError('value is error') # self.message = 'value is error'
    print('start')
except ValueError as ex:
    print(ex) # __str__, return self.message
else:
    pass
finally:
    pass
```

21. 自定义异常和断言

```python
# 自定义异常
class JiaqiXuException(Exception):

    def __init__(self, msg):
        self.message = msg

    def __str__(self):
        return self.message

try:
    raise JiaqiXuException('my custom exception')
except JiaqiXuException as ex:
    print(ex)
```

断言应用：
在程序里，如果assert 通过则继续执行，否则程序报错。

```python
'''
断言assert应用:
有一个对象p，他有一个start方法，但是在调用这个方法之前，得先执行一个p.status = False；如果不满足这个条件，start方法会出错
解决方法:
可以在start方法里先判断assert p.status == False
'''
p = object()
p.status = True
p.status() # 应该先执行一个asser p.status == False
```

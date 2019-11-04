---
title: "Design Pattern"
layout: post
date: 2017-09-11 22:44
tag:
- Design Pattern
star: true
category: blog
author: jiaqixu
description: note for Design pattern
---

### 目录
- [设计模式介绍](#设计模式介绍)
- [设计模式分类](#设计模式分类)
    * [创建型模式](#创建型模式)
    * [结构型模式](#结构型模式)
    * [行为型模式](#行为型模式)
- [单例模式](#单例模式)
    * [单例模式的UML](#单例模式的uml)
    * [单例模式的分类](#单例模式的分类)
    * [基于python的单例模式实现](#基于python的单例模式实现)
- [简单工厂模式](#简单工厂模式)
    * [简单工厂模式的UML](#简单工厂模式的uml)
    * [基于python的简单工厂模式实现](#基于python的简单工厂模式实现)
    * [简单工厂优缺点](#简单工厂优缺点)
- [工厂方法模式](#工厂方法模式)
    * [工厂方法模式的UML](#工厂方法模式的uml)
    * [基于python的工厂方法模式实现](#基于python的工厂方法模式实现)
    * [工厂方法模式的优缺点](#工厂方法模式的优缺点)
- [抽象工厂模式](#抽象工厂模式)
    * [抽象工厂模式的UML](#抽象工厂模式的uml)
    * [基于python的抽象工厂模式实现](#基于python的抽象工厂模式实现)
    * [总结](#总结)


### 设计模式介绍
设计模式(Design Pattern)是一套被反复使用，多数人知晓的，经过分类编目的，代码设计经验的总结。
使用设计模式是为了可重用代码，让代码更容易被他人理解，保证代码可靠性。毫无疑问，设计模式于己于他人于系统都是多赢的；
设计模式使代码编织真正工程化；设计模式是软件工程的基石脉络。

#### 设计模式的起源
* 《Design Pattern: Elements of Reusable Object-Oriented Software》(即<设计模式>一书)，由Erich Gamma, Richard Helm, Ralph Johnson和John Vlissides合著。
*  这个四个人是被合称为GoF(Gang of Four)
*  设计模式也都是符合OOD基本原则(SOLID): S(单一职责原则) O(开放关闭原则) L(里氏替换原则) I(接口隔离原则) D(依赖倒置原则)

### 设计模式分类

#### 创建型模式
社会化的分工越来越细，自然在软件设计方面也是如此，因此对象的创建和对象的使用分开也就成为了必然趋势。
因为对象的创建会消耗系统的很多资源，所以单独对对象的创建进行研究，从而能够高效地创建对象就是创建型模式要探讨的问题。
这里有六个具体的创建型模式可供研究，它们分别是:
```text
1. 简单工厂模式(Simple Factory)严格讲这个不是GoF提出的23种设计模式之一
2. 工厂方法模式(Factory Method)
3. 抽象工厂模式(Abstract Factory)
4. 创建者模式(Builder)
5. 原型模式(Prototype)
6. 单例模式(Singleton)
```

#### 结构型模式
在解决了对象的创建问题之后，对象的组成以及对象之间的依赖关系就成了开发人员关注的焦点，因为如何设计对象的结构，继承和依赖关系会影响到
后续程序的维护性，代码的健壮性，耦合性等。对象结构的设计很容易体现出设计人员水平的高低，这里有7个具体的结构型模式可供研究，它们分别是:
```text
1. 外观模式(Facade)
2. 适配器模式(Adapter)
3. 代理模式(Proxy)
4. 装饰模式(Decorator)
5. 桥接模式(Bridge)
6. 组合模式(Composite)
7. 享元模式(Flyweight)
```

#### 行为型模式
在对象的结构和对象的创建问题都解决了之后，就剩下对象的行为问题了，如果对象的行为设计的好，那么对象的行为就会更为清晰，它们之间的协作效率就会提高，这里有
11个具体的行为型模式可供研究，它们分别是:
```text
1. 模版方法模式(Template Method)
2. 观察者模式(Observer)
3. 状态模式(State)
4. 策略模式(Strategy)
5. 职责链模式(Chain of Responsibility)
6. 命令模式(Command)
7. 访问者模式(Visitor)
8. 中介者模式(Mediator)
9. 备忘录模式(Memento)
10. 迭代器模式(Iterator)
11. 解释器模式(Interpreter)
```



### 单例模式
单例模式(Singleton Pattern)最初的定义: "保证一个类仅有一个实例，并提供一个访问它的全局访问点。"

关键点:
```text
1. 某个类只能有一个实例
2. 它必须自行创建这个实例
3. 它必须自行向整个系统提供这个实例
```

解决的问题: 提供全局需要使用的，唯一的数据访问。

#### 单例模式的UML
![image](/assets/images/blog/designpattern-1.png)

```text
成员: 静态的实例
方法: 私有构造函数
      静态的获取实例的方法
```

#### 单例模式的分类
按创建时机分类:
```text
饿汉式(hungry):在类加载的时候创建。
    优点: 线程安全
    缺点: 过早浪费资源
懒汉式(lazy): 在使用时做判断，如果需要再创建。
    优点: 使用的时候才创建，资源节约
    缺点: 为线程安全要付出额外代价
```

#### 基于python的单例模式实现
最开始的想法很简单，实现如下:
```python
class Singleton(object):
    __instance = None
    def __new__(cls, *args, **kwargs): # 这里不能使用__init__，因为__init__是在instance已经生成以后才去调用的
        if cls.__instance is None:
            cls.__instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
        return cls.__instance
        
s1 = Singleton()
s2 = Singleton()
print s1
print s2


# 打印结果如下:
# <__main__.Singleton object at 0x7f3580dbe110>
# <__main__.Singleton object at 0x7f3580dbe110>
# 可以看出两次创建对象，结果返回的是同一个对象实例
```

上面的实现有一个问题没有解决，就是多线程问题，当有多个线程同时去初始化对象时，就很可能同时判断
__instance__ is None, 从而进入初始化instance代码中。所以为了解决这个问题，我们必须通过
同步锁来解决。
```python
import time
import threading
class Singleton(object):
    _instance_lock = threading.Lock()

    def __init__(self):
        time.sleep(1)

    @classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            with Singleton._instance_lock:
                if not hasattr(Singleton, "_instance"):
                    Singleton._instance = Singleton(*args, **kwargs)
        return Singleton._instance
        
def task(arg):
    obj = Singleton.instance()
    print(obj)

for i in range(10):
    t = threading.Thread(target=task,args=[i,])
    t.start()
time.sleep(20)
obj = Singleton.instance()
print(obj)
```


### 简单工厂模式
简单工厂模式(Simple Factory Pattern)属于类的创新型模式，又叫静态工厂方法模式(Static Factory Method Pattern),
是通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

#### 简单工厂模式的UML
![image](/assets/images/blog/designpattern-2.png)

```text
工厂角色(SimpleFactory): 这是简单工厂模式的核心，由它负责创建所有的类的内部逻辑。
当然工厂类必须能够被外接调用，创建所需要的产品对象。

抽象(IProduct)产品角色: 简单工厂模式所创建的所有对象的父类，注意，这里的
父类可以是接口也可以是抽象类，它负责描述所有实例所共有的公共接口。

具体产品(Concrete Product)角色: 简单工厂所创建的具体实例对象，这些具体的产品往往都拥有共同的父类。  
```

#### 基于python的简单工厂模式实现
```python
from abc import ABCMeta, abstractmethod
class Payment(metaclass=ABCMeta):
    @abstractmethod
    def pay(self, money):
        pass


class AliPay(Payment):
    def __init__(self, yu_e_bao=False):
        self.yu_e_bao = yu_e_bao

    def pay(self, money):
        if self.yu_e_bao:
            print(f'use yu_e_bao pay {money}')
        else:
            print(f'use zhifubao pay {money}')


class WeChat(Payment):
    def pay(self, money):
        print(f'use Wechat pay {money}')


class Paymethod():
    def create_payment(self, method):
        if method == 'yu_e_bao':
            return AliPay(True)
        elif method == 'zhifubao':
            return AliPay(False)
        elif method == 'Wechat':
            return WeChat()
        else:
            logging.log.error('method error')
            
p = Paymethod()
f = p.create_payment('zhifubao')
f.pay(30)     #  use zhifubao pay 30
f = p.create_payment('Wechat')
f.pay(300)    #  use Wechat pay 300
```


#### 简单工厂优缺点
优点: 工厂类是整个模式的关键所在。它包含必要的判断逻辑，能够根据外界给定的信息，决定究竟应该创建哪个具体类的对象。
用户在使用时可以直接根据工厂类去创建所需的实例，而无需了解这些对象是如何创建以及如何组织的。
有利于整个软件体系结构的优化。

缺点: 如果具体产品角色很多的时候，定义简单工厂就会比较麻烦，不利于扩展。


### 工厂方法模式
* 工厂方法: 一抽象产品类派生出多个具体产品类; 一抽象工厂类派生出多个具体工厂类; 每个具体工厂类只能创建一个具体产品类的实例。
* 即定义一个创建对象的接口(即抽象工厂类), 让其子类(具体工厂类)决定实例化哪一个类(具体产品类)。"一对一"的关系。


#### 工厂方法模式的UML
![image](/assets/images/blog/designpattern-3.png)

* 工厂接口。工厂接口是工厂方法模式的核心，与调用者直接交互用来提供产品。在实际编程中，有时候也会使用一个抽象类
来作为与调用者交互的接口，其本质上是一样的。
* 工厂实现。在编程中，工厂实现决定如何实例化产品，是实现扩展的途径，需要有多少种产品，就需要有多少个具体的工厂实现。
* 产品接口。产品接口的主要目的是定义产品的规范，所有产品实现都必须遵循产品接口定义的规范。
产品接口是调用者最为关心的，产品接口定义的优劣直接决定了调用者代码的稳定性。同样，产品接口也可以用抽象类来代替，但要注意最好不要违反里氏替换原则(通俗地讲:子类可以扩展父类的功能，但不能改变父类原有的功能)。
* 产品实现。实现产品接口的具体类，决定了产品在客户端中的具体行为。

#### 基于python的工厂方法模式实现
```python
from abc import ABCMeta, abstractmethod

# ------ 抽象产品 -----
class Chips(metaclass=ABCMeta):
    @abstractmethod
    def eat(self):
        pass

# ------ 具体产品 -----    
class MCchips(Chips):
    def eat(self):
        print("Eatting McDonald's Chips")


class KFCchips(Chips):
    def eat(self):
        print("Eatting KFC's Chips")


# ------ 抽象工厂 -----
class Store(metaclass=ABCMeta):
    @abstractmethod
    def getChips(self):
        pass


# ---- 具体工厂 -----
class Mcdonald(Store):
    def getChips(self):
        print("麦当劳生产了薯条")
        return MCchips()

class KFC(Store):
    def getChips(self):
        print("肯德基生成了薯条")
        return KFCchips()


if __name__ == '__main__':
    print("this is Factory Method Main:")
    store = Mcdonald().getChips().eat()
    store = KFC().getChips().eat()
```


#### 工厂方法模式的优缺点
优点: 
1. 在工厂方法中，用户只需要知道所要产品的具体工厂，无须关心具体的创建过程，甚至不需要具体产品类的类名。
2. 在系统增加新的产品时，我们只需要添加一个具体产品类和对应的实现工厂，无需对原工厂进行任何修改，很好地符合了"开闭原则"

缺点:
每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定成都上增加了系统的复杂度，同时
也增加了系统具体类的依赖。这并不是什么好事。

与简单工厂的对比:
工厂方法模式是简单工厂模式的延伸。在工厂方法模式中，核心工厂类不再负责产品的创建，而是将具体的创建工作交给子类去完成。
也就是说这个核心共厂仅仅只是提供创建的接口，具体实现方法交给继承它的子类去完成。当我们的系统需要增加其他新的对象时，我们只需要添加
一个具体的产品和它的创建工厂即可，不需要对原工厂进行任何修改，这样很好地符合了"开闭原则"。


### 抽象工厂模式
* 抽象工厂: 多个抽象产品类，派生出多个具体产品类；一个抽象工厂类，派生出多个具体工厂类；每个具体工厂类可创建多个具体产品类的实例。
* 即提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们的具体的类。"一对多"的关系。

#### 抽象工厂模式的UML
![image](/assets/images/blog/designpattern-4.png)

抽象工厂模式是工厂方法模式的升级版本，他用来创建一组相关或者相互依赖的对象。他与工厂方法模式的区别就在于，
工厂方法模式针对的是一个产品等级结构；而抽象工厂模式则是针对的多个产品等级结构。在编程中，
通常一个产品结构，表现为一个接口或者抽象类，也就是说，工厂方法模式提供的所有产品都是衍生自同一个接口或抽象类，
而抽象工厂模式所提供的产品则是衍生自不同的接口或抽象类。

#### 基于python的抽象工厂模式实现
```python
from abc import ABCMeta, abstractmethod

# ------ 抽象产品 -----
class Chips(metaclass=ABCMeta):
    @abstractmethod
    def eat(self):
        pass

class Chicken(metaclass=ABCMeta):
    @abstractmethod
    def eat(self):
        pass

    
# ------ 具体产品 -----    
class MCchips(Chips):
    def eat(self):
        print("Eatting McDonald's Chips")


class KFCchips(Chips):
    def eat(self):
        print("Eatting KFC's Chips")


class OrleanChicken(Chicken):
    def eat(self):
        print("Eatting KFC's OrleanChicken")

class McChicken(Chicken):
    def eat(self):
        print("Eatting McDonald's Chicken")
    
# ------ 抽象工厂 -----
class Store(metaclass=ABCMeta):
    @abstractmethod
    def getChips(self):
        pass

    @abstractmethod
    def getChicken(self):
        pass


# ---- 具体工厂 -----
class Mcdonald(Store):
    def getChips(self):
        print("麦当劳生产了薯条")
        return MCchips()

    def getChicken(self):
        print("麦当劳生成了麦乐鸡")
        return McChicken()

class KFC(Store):
    def getChips(self):
        print("肯德基生成了薯条")
        return KFCchips()


    def getChicken(self):
        print("肯德基生成了奥尔良烤翅")
        return OrleanChicken()
    

if __name__ == '__main__':
    print("this is Abstract Factory  Main:")
    store = Mcdonald().getChicken().eat()
    store = KFC().getChips().eat()
```


#### 总结
无论是简单工厂模式，工厂方法模式，还是抽象工厂模式，它们都属于工厂模式，在形式和特点上也是极为相似，
它们的最终目的都是为了解耦。在使用时，我们不必去在一这个模式到底是工厂方法模式还是抽象工厂模式，因为它们
之间的演变常常让人琢磨不透。经常你会发现，明明使用的工厂方法模式，当新需要来临，稍加修改，加入一个新方法后，由于类中的产品构成了不同
等级结构中的产品族，他就编程了抽象工厂模式了；而对于抽象工厂模式，当减少一个方法使得提供的产品不再构成产品族之后，它就演变成了
工厂方法模式。

所以，在使用工厂模式时，只需要关系降低耦合度的目的是否达到了。

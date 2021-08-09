# 参考《精通Python设计模式》书籍

## 前期知识

### Python抽象类的实现方式

在工作中常常有这样的需求：公司要求开发一款app，app的支付模块第一版只做微信支付，后期升级时可能会添加支付宝、网银等支付手段。

先假设小张同志接手了这个模块，那么刚开始小张可能是怎么写的：

```python
# 微信支付类
class Wechatpay:
    def pay(self, money):
        print("微信支付%d元" % money)

```

调用时：

```python
wechat = Wechatpay()
wechat.pay(100)
```

第一版只有微信支付的时候，感觉还可以，也没有什么不对的，那么我们继续后期升级添加支付宝支付手段，可能会是如下写法：

```python
# 支付宝支付类
class Alipay:
    def pay(self, money):
        print("支付宝支付%d元" % money)

        
# 微信支付类
class Wechatpay:
    def pay(self, money):
        print("微信支付%d元" % money)
```

调用时：

```python
# 支付宝支付调用
ali = Alipay()
ali.pay(100)

# 微信支付调用
wechat = Wechatpay()
wechat.pay(100)
```

咦！对不，在编写业务代码的时候，我希望不管是调用微信支付还是支付宝支付都统一用一个方法去调用，有可能在支付前还需要做一些其它操作，比如支付账户是否非法，金额是否超额等，这给我两套代码显然不是很妥当，行！继续改造：

```python
# 封装了一个公用的支付方法
def common_pay(p, money):
    p.pay(money)
```

调用时：

```python
# 支付宝支付调用
ali = Alipay()
common_pay(ali, 100)

# 微信支付调用
wechat = Wechatpay()
common_pay(wechat, 100)
```

em，这样感觉还可以，但是这样有一个问题，如果这一个项目是由一个人去编写完成，那么他可能会很清楚的知道自己给common_pay方法传递了什么参数，事实并非如此，我们都是多人协同开发，此时小张离职了，小明同学接脚。组长要求他在支付模块上再添加多一个支付手段：网银支付，这是小张可能会写做如下：

```python
# 网银支付类
class Ebankpay:
    def pay(self, money, unit):
        print("网银支付%d%s" % (money, unit))
```

此时小明看了小张之前的代码，只看了调用，所以他也调用common_pay方法来完成业务：

```python
# 网银支付调用
ebank = Ebankpay()
common_pay(ebank, 100, '元')
```

TypeError: common_pay() takes 2 positional arguments but 3 were given

凉凉，抛了异常。

那么我们回想一下在Java场景中，如果需要定义统一的接口方法，一般都会定义一个接口类，然后其它实现类都统统实现该接口定义的接口方法就可以规避上述问题：

+ 小明不知道如何规范的定义支付类
+ 小明不知道如何规范的调用支付类来支付

那么在Python中我们应该怎么写。

可以定义一个父类，父类定义一个公共的支付方法，其它继承该父类，实现并覆盖父类定义的方法就可以了，完整代码如下：

```python
# 支付父类
class Pay:
    def pay(self, money):
        raise NotImplementedError

# 支付宝支付类
class Alipay(Pay):
    def pay(self, money):
        print("支付宝支付%d元" % money)

# 微信支付类
class Wechatpay(Pay):
    def pay(self, money):
        print("微信支付%d元" % money)

# 网银支付类
class Ebankpay(Pay):
    def pay(self, money):
        print("网银支付%d元" % money)


# 公共支付方法
def common_pay(p: Pay, money):
    p.pay(money)
```

调用时：

```python
ali = Alipay()
wechat = Wechatpay()
ebank = Ebankpay()
common_pay(ali, 100)
common_pay(wechat, 100)
common_pay(ebank, 100)
```

但是这样也有一个问题，就是如果再实现一个QQ支付类，但是这个类并没有实现pay(self, money)方法，但是只要QQ支付类的实例对象不去调用pay方法就不会有任何问题，感觉还是有一点小问题，那么Python有没有像Java这种强制类型的语言一样有强制的抽象类或接口要求强制实现一样。

答案当然是有的，在Python中一个abc包（abstract class的缩写）中就提供了相关抽象的定义，使用规范后的写法：

```python
from abc import ABCMeta, abstractmethod

# 支付父类
class Pay(metaclass=ABCMeta):
    @abstractmethod
    def pay(self, money):
        pass

# 支付宝支付类
class Alipay(Pay):
    def pay(self, money):
        print("支付宝支付%d元" % money)

# 微信支付类
class Wechatpay(Pay):
    def pay(self, money):
        print("微信支付%d元" % money)

# 网银支付类
class Ebankpay(Pay):
    def pay(self, money):
        print("网银支付%d元" % money)
```

调用时：

```python
ali = Alipay()
wechat = Wechatpay()
ebank = Ebankpay()
common_pay(ali, 100)
common_pay(wechat, 100)
common_pay(ebank, 100)
```

这样就挺完美的了，如果不实现父类的抽象方法的话，则会抛出一个异常：

```python
TypeError: Can't instantiate abstract class xxx with abstract methods pay
```

这就是Python一般在工作中常用的定义抽象类以及抽象方法的方式。

### 面向对象编程规范

我们都知道当今主流的编程方式一般分三种

+ 面向过程编程
+ 面向对象编程
+ 函数式编程

今天只要讲的是面向对象编程，面向对象一般都三个特性：封装、继承、多态，这三者是一种递增的联系。

当我们编程重复代码时就会想到把相同的代码进行封装，以便重复使用；当我们编写一个类时想复用其它类的封装好的方法，就可以继承其它类，以便实现封装的代码继承过来复用；当我们需要编写规范代码时，使用统一的类进行统一的调用，用父类的表现形式来调用子类实现的封装方法，这就叫多态。

面向对象的设计模式遵循SOLID原则

+ SRP单一职责原则
+ OCP开放封闭原则
+ LSP里氏替代原则
+ ISP接口隔离原则
+ DIP依赖倒置原则

#### SRP单一职责原则

不要存在多余的原因导致一个类的代码变更。通俗一点来说就是一个类只负责一项职责。

#### OCP开放封闭原则

一个软件实体如类、模块和函数应该对扩展开放，对修改原有代码关闭。就是说在尽量不去修改原有代码的情况下进行功能上的扩展。

#### LSP里氏替代原则

所有引用父类的地方必须能透明地使用其子类的对象。也就说一个函数传一个父类对象进去时正常，当传一个子类对象进去时也要表现正常。

#### ISP接口隔离原则

使用多个专门的接口，而不使用单一的总接口，也就是说一个类需要实现多个接口的使用，这些接口都应该由多个接口类来提供，并这个类实现这些接口类，而不是去实现一个总的接口类。因为有些类的实现可能有时候并不需要全部都实现总的接口类中的接口。

#### DIP依赖倒置原则

高层模块不应该依赖底层模块，二者都应该依赖其抽象，意思就是高层模块的代码调用不应该直接调用底层模块的接口，而是让底层模块接口实现一个已经规范好的抽象接口，两者都应该通过这个规范好的抽象接口进行调用和实现；而规范好的抽象接口不应该依赖细节；细节应该依赖抽象。就是底层实现是一种细节，实现应该依赖抽象接口规范好的接口及参数。换言之，要针对接口编程，而不是针对实现编程。



设计模式可以让代码更容易维护，做到规范化，统一化。但是并不是所有代码都适用于设计模式，设计模式是在原方案的基础上进行的优化方案，如果编写的代码在当前场景下最为合适就不需要套用设计模式

**注：**设计模式不是随意使用的，每个设计模式都是依据特定的场景，符合场景才可以使用，如果泛滥使用设计模式会适得其反！

Python的设计模式大体分三类，如下三大标题：

## 第一类：创建型模式

这类模式主要是在处理对象创建的时候运用的设计模式

### 一、工厂模式

使用工厂模式（工厂方法和抽象工厂）来创建对象，相比直接实例化对象的好处在于：在一个地方集中的创建对象，便于对象的跟踪

#### 1）简单工厂模式

不直接向高层暴露对象创建的实现细节，而是通过一个工厂类来负责创建相关类的实例。

简单工厂模式需要三个角色：

+ 工厂角色（Creator）
+ 抽象角色（Product）
+ 具体角色（Concrete Product）

```python
from abc import ABCMeta, abstractmethod

# 水果（抽象角色）
class Fruit(metaclass=ABCMeta):
    @abstractmethod
    def show_name(self):
        pass
    
# 苹果（具体角色）
class Apple(Fruit):
    __color_map = {
        "red": "红",
        "green": "青"
    }

    def __init__(self, color='red'):
        self.name = f'{Apple.__color_map.get(color, "未知")}苹果'
        
    def show_name(self):
        print(f'这是一个{self.name}')
        
# 香蕉（具体角色）
class Banana(Fruit):
    def __init__(self):
        self.name = '香蕉'

    def show_name(self):
        print(f'这是一个{self.name}')
        
# 梨（具体角色）
class Pear(Fruit):
    def __init__(self):
        self.name = '梨'

    def show_name(self):
        print(f'这是一个{self.name}')


# 简单水果工厂（工厂角色）
class SimpleFactory:
    def create_fruit(self, type):
        if type.lower() == 'red apple':
            return Apple()
        if type.lower() == 'green apple':
            return Apple(color='green')
        elif type.lower() == 'banana':
            return Banana()
        elif type.lower() == 'pear':
            return Pear()

        return TypeError("No such %s type" % type)


if __name__ == '__main__':
    sf = SimpleFactory()
    # 使用工厂角色提供的统一方式来创建
    fruit1 = sf.create_fruit('red apple')
    fruit2 = sf.create_fruit('green apple')
    fruit3 = sf.create_fruit('banana')
    fruit4 = sf.create_fruit('pear')
	
    # 使用抽象角色定义好的方法统一调用
    fruit1.show_name()
    fruit2.show_name()
    fruit3.show_name()
    fruit4.show_name()
```

对于苹果这个具体角色来说分两种实现：一种是红苹果一种是青苹果，高层调用时可以使用工厂来提供的统一创建方式来创建，并不需要太在意实现细节，换言之：只需要传入指定要创建那种苹果就可以了，具体需要传入给苹果类传入什么参数对于高层调用来说是完全屏蔽的。总结：

优点：

+ 对高层调用来说隐藏了对象创建的实现细节

缺点：

+ 违反了单一职责原则，将所有的具体角色创建的逻辑都集中到了一个工厂类中
+ 当添加一个新的水果角色时，需要返回修改工厂类的代码，违反了开放封闭原则

所以为了解决简单工厂模式的缺点并保留优点，我们提出了工厂方法模式

#### 2）工厂方法模式

工厂方法模式也是不直接向高层暴露对象创建的实现细节，而是通过每个类都有一一对应的具体工厂类来负责创建相关类的实例

工厂方法模式需要四个角色：

+ 抽象角色（Product）
+ 具体角色（Concrete Creator）
+ 抽象工厂角色（Creator）
+ 具体工厂角色（Concrete Creator）

```python
from abc import ABCMeta, abstractmethod

# 水果（抽象角色）
class Fruit(metaclass=ABCMeta):
    @abstractmethod
    def show_name(self):
        pass

# 苹果（具体角色）
class Apple(Fruit):
    __color_map = {
        "red": "红",
        "green": "青"
    }

    def __init__(self, color='red'):
        self.name = f'{Apple.__color_map.get(color, "未知")}苹果'

    def show_name(self):
        print(f'这是一个{self.name}')

# 香蕉（具体角色）
class Banana(Fruit):
    def __init__(self):
        self.name = '香蕉'

    def show_name(self):
        print(f'这是一个{self.name}')

# 梨（具体角色）
class Pear(Fruit):
    def __init__(self):
        self.name = '梨'

    def show_name(self):
        print(f'这是一个{self.name}')

# 水果工厂类（抽象工厂角色）
class FruitFactory(metaclass=ABCMeta):
    @abstractmethod
    def create_fruit(self) -> Fruit:
        pass

# 红苹果工厂类（具体工厂角色）
class RedAppleFactory(FruitFactory):
    def create_fruit(self) -> Fruit:
        return Apple(color='red')

# 青苹果工厂类（具体工厂角色）
class GreenAppleFactory(FruitFactory):
    def create_fruit(self) -> Fruit:
        return Apple(color='green')

# 香蕉工厂类（具体工厂角色）
class BananaFactory(FruitFactory):
    def create_fruit(self) -> Fruit:
        return Banana()

# 梨工厂类（具体工厂角色）
class PearFactory(FruitFactory):
    def create_fruit(self) -> Fruit:
        return Pear()

if __name__ == '__main__':
    # 每种水果都有自己单独的工厂类
    factory1 = RedAppleFactory()
    factory2 = GreenAppleFactory()
    factory3 = BananaFactory()
    factory4 = PearFactory()

    # 使用抽象工厂角色定义好的统一创建类的方式
    fruit1 = factory1.create_fruit()
    fruit2 = factory2.create_fruit()
    fruit3 = factory3.create_fruit()
    fruit4 = factory4.create_fruit()

    # 使用抽象角色定义好的统一调用方法
    fruit1.show_name()
    fruit2.show_name()
    fruit3.show_name()
    fruit4.show_name()

```

工厂方法模式解决了简单工厂模式的缺点：

+ 通过将每个类的创建都独立出来一一对应一个工厂类，而不再使用公共的工厂类来创建，解决了简单工厂模式违反单一职责原则将所有的具体角色创建的逻辑都集中到了一个工厂类中的问题。
+ 通过将每个类的创建都独立出来一一对应一个工厂类，而不再使用公共的工厂类来创建，解决了简单工厂模式违反了开放封闭原则，在添加一个新的水果角色时需要返回到公共工厂类修改代码。而是通过再创建一个新的水果工厂类来对应，符合了开放封闭的原则。

但这其实都看的起来工厂方法模式好像很麻烦，创建一个具体角色就需要创建一个具体工厂角色，这样增加了很多的代码。总结：

优点：

+ 每个具体角色都有一个对应的具体工厂角色，不需要再去修改公共工厂角色的代码
+ 隐藏了对象创建的实现细节

缺点：

+ 每增加一个具体角色就必须要增加一个具体工厂角色，简而言之就是增加代码量

#### 3）抽象工厂模式

定义一个抽象工厂角色，让具体工厂角色来创建一系列相关或相互依赖的对象，简而言之就是抽象工厂模式的工厂是创建两个或两个以上的对象，而简单工厂模式和工厂方法模式的具体工厂角色都是创建一个对象而已。

举个例子：咱们买台式电脑一般都是需要购买主机、显示器、键盘、鼠标四件套，然后就是购买台式电脑一般有几种需求：商务办公配置、游戏娱乐配置等，如果按照游戏来分的话简单点也可以分成：低端玩家配置、高端玩家配置、发烧友玩家配置等，那么我们需要实现这样一个功能，就是只需要针对低端玩家配置的需求使用一个低端工厂一次性创建对应的低端主机、低端显示器、低端键盘、低端鼠标的四个对象，其它同理。那么我们按照玩游戏的电脑配置来编写一个抽象工厂模式的例子：

抽象工厂模式需要五个角色：

+ 抽象角色（Product）
+ 具体角色（Concrete Creator）
+ 抽象工厂角色（Creator）
+ 具体工厂角色（Concrete Creator）
+ 客户端（Client）

```python
from abc import ABCMeta, abstractmethod

# 主机类（抽象角色）
class Mainframe(metaclass=ABCMeta):
    @abstractmethod
    def show_mainframe(self):
        pass

# 显示器类（抽象角色）
class Monitor(metaclass=ABCMeta):
    @abstractmethod
    def show_monitor(self):
        pass

# 键盘类（抽象角色）
class Keyboard(metaclass=ABCMeta):
    @abstractmethod
    def show_keyboard(self):
        pass

# 键盘类（抽象角色）
class Mouse(metaclass=ABCMeta):
    @abstractmethod
    def show_mouse(self):
        pass

# 低端主机（具体角色）
class I3GTX1030Mainframe(Mainframe):
    def show_mainframe(self):
        print('i3+GTX1030的低端主机')

# 高端主机（具体角色）
class I7GTX2080TiMainframe(Mainframe):
    def show_mainframe(self):
        print('i7+GTX2080Ti的高端主机')

# 发烧级主机（具体角色）
class I9GTX3090TiMainframe(Mainframe):
    def show_mainframe(self):
        print('i9+GTX3090Ti的发烧级主机')

# 低端显示器（具体角色）
class R1080Hz60Monitor(Monitor):
    def show_monitor(self):
        print('1080分辨率60刷新率的低端显示器')

# 高端显示器（具体角色）
class R1440Hz144Monitor(Monitor):
    def show_monitor(self):
        print('2K分辨率144刷新率的高端显示器')

# 发烧级显示器（具体角色）
class R2160Hz240Monitor(Monitor):
    def show_monitor(self):
        print('4K分辨率240刷新率的发烧级显示器')

# 低端机械键盘（具体角色）
class K835Keyboard(Keyboard):
    def show_keyboard(self):
        print('K835面向低端玩家的有线机械键盘')

# 高端机械键盘（具体角色）
class G213Keyboard(Keyboard):
    def show_keyboard(self):
        print('G213面向高端玩家的有线机械键盘')

# 发烧级机械键盘（具体角色）
class AW510KKeyboard(Keyboard):
    def show_keyboard(self):
        print('AW510K面向发烧级玩家的有线机械键盘')

# 低端鼠标（具体角色）
class M220Mouse(Mouse):
    def show_mouse(self):
        print('M220面向低端玩家的有线鼠标')

# 高端鼠标（具体角色）
class G903Mouse(Mouse):
    def show_mouse(self):
        print('G903面向高端玩家的有线鼠标')

# 发烧级鼠标（具体角色）
class ROMMouse(Mouse):
    def show_mouse(self):
        print('ROM面向发烧级玩家的有线鼠标')

# 电脑工厂（抽象工厂角色）
class ComputerFactory(metaclass=ABCMeta):
    @abstractmethod
    def make_mainframe(self) -> Mainframe:
        pass

    @abstractmethod
    def make_monitor(self) -> Monitor:
        pass

    @abstractmethod
    def make_keyboard(self) -> Keyboard:
        pass

    @abstractmethod
    def make_mouse(self) -> Mouse:
        pass

# 低端电脑工厂（具体工厂角色）
class LowEndComputerFactory(ComputerFactory):
    def make_mainframe(self) -> Mainframe:
        return I3GTX1030Mainframe()

    def make_monitor(self) -> Monitor:
        return R1080Hz60Monitor()

    def make_keyboard(self) -> Keyboard:
        return K835Keyboard()

    def make_mouse(self) -> Mouse:
        return M220Mouse()

# 高端电脑工厂（具体工厂角色）
class HighEndComputerFactory(ComputerFactory):
    def make_mainframe(self) -> Mainframe:
        return I7GTX2080TiMainframe()

    def make_monitor(self) -> Monitor:
        return R1440Hz144Monitor()

    def make_keyboard(self) -> Keyboard:
        return G213Keyboard()

    def make_mouse(self) -> Mouse:
        return G903Mouse()

# 高端电脑工厂（具体工厂角色）
class CrazyComputerFactory(ComputerFactory):
    def make_mainframe(self) -> Mainframe:
        return I9GTX3090TiMainframe()

    def make_monitor(self) -> Monitor:
        return R2160Hz240Monitor()

    def make_keyboard(self) -> Keyboard:
        return AW510KKeyboard()

    def make_mouse(self) -> Mouse:
        return ROMMouse()

# 台式电脑类（一系列对象的组合类，台式电脑是由主机、显示器、鼠标、键盘构成）
class Computer:
    def __init__(self, mainframe: Mainframe, monitor: Monitor, keyboard: Keyboard, mouse: Mouse):
        self.mainframe = mainframe
        self.monitor = monitor
        self.keyboard = keyboard
        self.mouse = mouse

    def show_info(self):
        print("电脑配置清单：")
        self.mainframe.show_mainframe()
        self.monitor.show_monitor()
        self.keyboard.show_keyboard()
        self.mouse.show_mouse()

# 简单的封装一下创建电脑的过程
def make_computer(factory: ComputerFactory) -> Computer:
    mainframe = factory.make_mainframe()
    monitor = factory.make_monitor()
    keyboard = factory.make_keyboard()
    mouse = factory.make_mouse()

    return Computer(mainframe, monitor, keyboard, mouse)

if __name__ == '__main__':
    # 高层只需要调用封装好的make_computer并传一个想要的指定一套设备配置的工厂对象即可
    computer = make_computer(LowEndComputerFactory())
    computer.show_info()
    computer = make_computer(HighEndComputerFactory())
    computer.show_info()
    computer = make_computer(CrazyComputerFactory())
    computer.show_info()
```

执行结果：

```python
电脑配置清单：
i3+GTX1030的低端主机
1080分辨率60刷新率的低端显示器
K835面向低端玩家的有线机械键盘
M220面向低端玩家的有线鼠标
电脑配置清单：
i7+GTX2080Ti的高端主机
2K分辨率144刷新率的高端显示器
G213面向高端玩家的有线机械键盘
G903面向高端玩家的有线鼠标
电脑配置清单：
i9+GTX3090Ti的发烧级主机
4K分辨率240刷新率的发烧级显示器
AW510K面向发烧级玩家的有线机械键盘
ROM面向发烧级玩家的有线鼠标

Process finished with exit code 0
```

抽象工厂模式只适用于拥有一系列对象的组合需要创建的时候

## 第二类：结构型模式

这类模式用于处理一个系统中不同实体（类、对象等）之间关系的设计模式



## 第三类：行为型模式

处理系统实体之间通信的设计模式


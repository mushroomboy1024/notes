# Python试题

## 1.说说Python的传参引用

Python传参是引用方式，参数可以分可变和不可变，可变比如字典、列表、集合这些可以改变内部的值；不可变比如字符串、元组、数字等这些修改值的时候会在内存中重新创建一个新的值。所以在传参时可变的参数在函数内部修改时会影响外部的实参，而不可变的参数则不会影响，因为每次修改都会重新创建一个新的值



## 2.Python的僵尸进程

在Python中的如果子进程运行结束而主进程并没有进行回收子进程，此时子进程的资源（打开文件的句柄、占用的内存）会被系统收回，但子进程的信息（进程号PID、运行时间、退出状态等）会被保留，这种情况子进程则变成了僵尸进程

僵尸进程的危害：

+ 占用系统的PID号，因为PID是有限的，这样会影响新进程的创建

**补充说明：**如果父进程在子进程还有运行结束时结束的话，子进程会被init进程接管，init会以父进程的身份处理子进程运行结束后遗留的状态信息

案例：文件名test.py

```python
import os
from multiprocessing import Process
from time import sleep


def child_def():
    print(f"子进程PID：${os.getpid()}")


if __name__ == '__main__':
    child = Process(target=child_def)  # 创建一个子进程
    child.start()  # 启动子进程
    # child.join() # 等待结束并回收
    print(f"父进程PID：${os.getpid()}")
    sleep(100)
```

执行并查看结果，Z+表示该进程的状态为僵尸状态

```shell
$ python3 test.py
子进程PID：10728
父进程PID：10727
$ ps -aux|grep python3|grep -v grep
root     10727  0.2  0.0 174272  9692 pts/1    S+   10:15   0:00 python3 test.py
root     10728  0.0  0.0      0     0 pts/1    Z+   10:15   0:00 [python3] <defunct>
```



## 3.Python上下文管理

上下文管理器，with open() as f: 这种语句中with会帮我们进行资源的收回，实际对于f对象来说内部已经实现了两个魔法方法：

```python
# 进入时执行
def __enter__():
    pass

# 退出时执行
def __exit__():
    pass
```

也就是说只要对象实现了这两个魔法方法，with就会自动调用这两个方法实现进入和退出时要执行的动作，如open()打开文件，with进入时会打开文件，退出时释放文件句柄。

**说明：**这类主要是用于资源管理、数据库连接等操作，需要对资源的释放或关闭。



## 4.说说Python的锁

Python可以分为两种锁：互斥锁和GIL全局解释器锁。

**互斥锁**是使用在多线程中，常用于避免对全局变量的一个资源竞争的问题，使用 Thread 对象的 Lock 和 Rlock 可以实现简单的线程同步，这两个对象都有 acquire 方法和 release 方法，对于那些需要每次只允许一个线程操作的数据，可以将其操作放到 acquire 和 release 方法之间

**GIL全局解释器锁**是因为Python使用的是C语言编写的，所以在C语言的层面上对Python语言代码进行解释执行时做了一个全局的锁定，即只能使用CPU的一个核心运行，称为单核语言。所以Python的多线程实际上并不是真正的多线程，而是多个线程同时去争抢一个CPU核心。此时可以使用其他语言的解释器或者利用Python的胶水语言的特性，关键的代码使用其它语言进行编写

锁会引申出一个问题：死锁，在锁嵌套调用时会造成互相持有对方的锁资源，无法释放各自持有的锁导致死锁。经典的案例：银行家算法

案例：文件名test.py

**未使用锁：**

```shell
import threading
from time import sleep
class myThread(threading.Thread):
    # 共享资源
    resources = ''
    def __init__(self, thread_id):
        threading.Thread.__init__(self)
        self.thread_id = thread_id

    def run(self):
        print(f'线程ID：{self.thread_id}，运行')

        myThread.resources = self.thread_id
        if self.thread_id == '01':
            sleep(1)
        print(f'线程ID：{self.thread_id}, 修改值：{myThread.resources}')

        print(f'线程ID：{self.thread_id}，结束')
if __name__ == '__main__':
    myThread(thread_id='01').start()
    myThread(thread_id='02').start()
```

执行结果：未使用锁，线程01修改的值被线程02给改了

```python
线程ID：01，运行
线程ID：02，运行
线程ID：02, 修改值：02
线程ID：02，结束
线程ID：01, 修改值：02
线程ID：01，结束
```

**使用锁：**

```python
import threading
from time import sleep
class myThread(threading.Thread):
    resources = '' # 共享资源
    lock = threading.Lock() # 锁
    def __init__(self, thread_id):
        threading.Thread.__init__(self)
        self.thread_id = thread_id

    def run(self):
        print(f'线程ID：{self.thread_id}，运行')
        
        myThread.lock.acquire()
        myThread.resources = self.thread_id
        if self.thread_id == '01':
            sleep(1)
        print(f'线程ID：{self.thread_id}, 修改值：{myThread.resources}')
        myThread.lock.release()
        
        print(f'线程ID：{self.thread_id}，结束')
if __name__ == '__main__':
    myThread(thread_id='01').start()
    myThread(thread_id='02').start()

```

执行结果：使用锁，线程01修改的值并未被线程02修改，因为需要等待线程01的锁范围代码执行完毕，线程02才可以被执行

```python
线程ID：01，运行
线程ID：02，运行
线程ID：01, 修改值：01
线程ID：01，结束
线程ID：02, 修改值：02
线程ID：02，结束
```



## 5.Python方法解析顺序

Python在历史的版本中出现过三个不同的MRO（方法解析顺序）算法：

+ Python2.1的经典类的深度遍历
+ Python2.2的新式类预计算
+ Python2.3的新式类的C3算法，这也是Python3唯一支持的方式

其中经典类和新式类的区别就是新式类继承了object。经典类的深度遍历从左往右的深度优先遍历，遇到有重复类则保留第一个；新式类预计算也是按深度优先遍历，但是它的每个类都继承了object，重复只保留最后一个；以上两个MRO在多继承的情况下都有可以能违反【局部优先级】和继承关系违反线性化的【单调性】，导致了具有二义性的继承关系。

而2.3版本之后的新式类的C3算法则可以很好的解决上述出现的问题，算法大致为把类 C 的线性化（MRO）记为 L[C] = [C1, C2,…,CN]。其中 C1 称为 L[C] 的头，其余元素 [C2,…,CN] 称为尾。如果一个类 C 继承自基类 B1、B2、……、BN，那么可以根据以下两步计算出 L[C]：

```python
L[object] = [object]
L[C(B1…BN)] = [C] + merge(L[B1]…L[BN], [B1]…[BN])
```

这里的关键在于 merge，其输入是一组列表，按照如下方式输出一个列表：

1. 检查第一个列表的头元素（如 L[B1] 的头），记作 H。
2. 若 H 未出现在其它列表的尾部，则将其输出，并将其从所有列表中删除，然后回到步骤1；否则，取出下一个列表的头部记作 H，继续该步骤。
3. 重复上述步骤，直至列表为空或者不能再找出可以输出的元素。如果是前一种情况，则算法结束；如果是后一种情况，说明无法构建继承关系，Python 会抛出异常。

该方法有点类似于图的拓扑排序，但它同时还考虑了基类的出现顺序。用 C3 分析以下例子

**具有二义性的继承关系：**

```python
# 具有二义性的继承关系
class A(object): pass
class B(object): pass
class C(A, B): pass
class D(B, A): pass
class E(C, D): pass
if __name__ == '__main__':
    E()
    print(E.__mro__) # 打印出继承顺序
    
# 执行结果，禁止创建具有二义性的继承关系
TypeError: Cannot create a consistent method resolution
order (MRO) for bases A, B
```

分析：

```python
L[object] = [object]
L[A] = [A, object]
L[B] = [B, object]
L[C] = [C, A, B, object]
L[D] = [D, B, A, object]

L[E] = [E] + merge(L[C], L[D], [C], [D])
     = [E] + merge([C, A, B, object], [D, B, A, object], [C], [D])
     = [E, C] + merge([A, B, object], [D, B, A, object], [D])
     = [E, C, D] + merge([A, B, object], [B, A, object])
```

到第四步 = [E, C, D] + merge([A, B, object], [B, A, object])的时候就已经无法继续下去了，因为第一个队列头部的A在第二个队列的非头部，而第二个队列头部的B在第一个队列的非头部，C3算法无法进行下去。解决办法：将D的继承顺序修改一下D(A, B)，然后再进行分析。

**不具有二义性的继承关系**：

```python
# 具有二义性的继承关系
class A(object): pass
class B(object): pass
class C(A, B): pass
class D(A, B): pass # 修改后
class E(C, D): pass
if __name__ == '__main__':
    E()
    print(E.__mro__) # 打印出继承顺序

# 执行结果
(<class '__main__.E'>, <class '__main__.C'>, <class '__main__.D'>, <class '__main__.A'>, <class '__main__.B'>, <class 'object'>)
```

分析：

```python
L[object] = [object]
L[A] = [A, object]
L[B] = [B, object]
L[C] = [C, A, B, object]
L[D] = [D, A, B, object]

L[E] = [E] + merge(L[C], L[D], [C], [D])
     = [E] + merge([C, A, B, object], [D, A, B, object], [C], [D])
     = [E, C] + merge([A, B, object], [D, A, B, object], [D])
     = [E, C, D] + merge([A, B, object], [A, B, object])
     = [E, C, D, A] + merge([B, object], [B, object])
     = [E, C, D, A, B] + merge([object], [object])
     = [E, C, D, A, B, object]
```



## 6.说说python的解释器

参考官网文档介绍[Python Implementations](https://wiki.python.org/moin/PythonImplementations?action=show&amp;redirect=implementation)，Python解释器目前市面上大致有以下几种：

+ CPython：官方版本的C语言实现，默认解释器
+ IPython：I（interraction交互），基于CPython之上的一种交互式解释器，只是在交互上有所增强，其它功能都和CPython一样
+ Jython：运行在Java平台上，直接把Python代码编译成Java的字节码，[官网介绍](https://www.jython.org/)
+ IronPython：运行在.NET和Mono平台上，直接把Python代码编译成.NET的字节码
+ PyPy：这个是使用Python实现的，支持JIT即时编译，补充：PyPI是Python包索引，是Python变成语言的软件存储库

**常用的则是：**CPython默认和Jython。

补充：

+ 在交互上CPython则使用>>>作为提示符，而IPython则用In[序号]:作为提示符
+ PyPy是另一种Python解释器，目标是执行速度，PyPy采用了即时编译，解释器对Python代码进行编译（注意：不是解释），编译后生成的代码能够得到更快的执行速度。绝大多数的Python代码使用CPython解释器和PyPy解释器执行的结果是一样的，但是也有一些代码在执行是结果不一样的地方，如果要将Python代码放到PyPy解释器上就需要了解CPython和PyPy之间的不同点。



## 7.说说Python和Java的区别

+ Python是解释型语言，Java是编译型语言
  + 相同：解释型和编译型语言都需要转换为二进制机器码才可以执行
  + 编译器：Python运行时需要编译器编译再执行，Java在运行时已经编译过了，所以可以直接执行
  + 运行速度：Java在运行时已经编译过，Python在运行时会比Java多一步编译过程，所以会慢
  + 可移植性：Python的可移植性比Java要好，因为Java是事先编译好代码，所以一旦CPU指令发生改变，Java则需要再次编译去匹配当前的CPU指令，而Python不需要，因为Python在运行时才编译
  + 升级：编译型语言的二进制文件如果需要升级，就得重新下载新的二进制文件，如QQ的升级就需要重新下载、安装、覆盖，而解释型语言的产品则不需要这样做，如Web页面的HTML、Js，只需要刷新一下即可
  + 应用领域：编译型语言的产品PC或手机安装的软件，解释型语言的产品互联网、网站等
+ 如在声明变量是需要指明类型，Python则不需要。所以Python在运行时解释器每次遇到变量都会去确认变量类型，而Java不需要，因为Java在编译成字节码的时候就已经确认了。
+ Python是一切皆对象，没有基本数据类型，只有标准数据类型，而Java有基本数据类型（Python中一个int变量都是对象，可以直接使用对象的方法，而Java一个int变量就是一个单纯的值）
+ Python的多线程是伪线程，因为GIL使得变成单核语言，而Java则是多核。解决Python并发的办法就是使用多进程代替多线程或者使用Cython库编写C语言的多线程，然后用Python调用，并将Python默认的PVM换成PyPy解释器。


















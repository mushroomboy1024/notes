# Python3的零散知识点

### 1.关键字is和==操作符的区别

```python
# 代码：
name1 = "Dolphin"
name2 = f"Dolphi{'n'}"
print(name1, name2, id(name1), id(name2)) # Dolphin Dolphin 2351140610288 2351140838128
print(name1 == name2)  #True, == 比较值
print(name1 is name2)  #False, is 比较地址

# 输出：
Dolphin Dolphin 2351140610288 2351140838128
True
False

Process finished with exit code 0
```

### 2.深拷贝和浅拷贝

```python
# 地址赋值：
arr1 = [1, [2, 3]]
arr2 = arr1  # arr1将地址赋值给了arr2
arr2[1][1] = 100  # arr2将下标1的下标1的元素重新赋值100，同时arr1也会受到影响
print(arr1, arr2)  # [1, [2, 100]] [1, [2, 100]]
print(id(arr1), id(arr2))  # 2101280461832 2101280461832
print(id(arr1[1]), id(arr2[1]))  # 2101280461320 2101280461320
# 输出：
[1, [2, 100]] [1, [2, 100]]
2101280461832 2101280461832
2101280461320 2101280461320

Process finished with exit code 0


# 浅拷贝：
arr1 = [1, [2, 3]]
arr2 = arr1[:]  # 浅拷贝，arr1地址指向内存的对象复制一份并赋值给arr2，但是下标3的元素并未进行复制操作
arr2[0] = 100
arr2[1][1] = 100  # arr2将下标3的下标1的元素重新赋值100，arr1并不会受到影响，因为arr1和arr2此时指向的内存地址完全不同
print(arr1, arr2)  # [1, [2, 100]] [100, [2, 100]]
print(id(arr1), id(arr2))  # 2374813504520 2374814071752
print(id(arr1[1]), id(arr2[1]))  # 2374813504008 2374813504008
# 输出：
[1, [2, 100]] [100, [2, 100]]
2374813504520 2374814071752
2374813504008 2374813504008

Process finished with exit code 0


# copy.copy()浅拷贝：
import copy
a = (1, (2, 3))  # tuple不可变类型
b = copy.copy(a)  # 当变量类型是不可变类型时直接进行地址赋值
print(a, b)  # (1, (2, 3)) (1, (2, 3))
print(id(a), id(b))  # 1772585128328 1772585128328
print(id(a[1]), id(b[1]))  # 1772585066888 1772585066888

arr1 = [1, [2, 3]]  # list可变类型
arr2 = copy.copy(arr1)  # 当变量类型是可变类型时进行浅拷贝操作
print(arr1, arr2)  # [1, [2, 3]] [1, [2, 3]]
print(id(arr1), id(arr2))  # 1772585838856 1772586094472
print(id(arr1[1]), id(arr2[1]))  # 1772585839368 1772585839368
# 输出：
(1, (2, 3)) (1, (2, 3))
1772585128328 1772585128328
1772585066888 1772585066888
[1, [2, 3]] [1, [2, 3]]
1772585838856 1772586094472
1772585839368 1772585839368

Process finished with exit code 0


# 深拷贝：
import copy
arr1 = [1, [2, 3]]
arr2 = copy.deepcopy(arr1)  # 深拷贝，arr1在内存中重新复制一份同时递归迭代每个元素以及子元素都将复制一份出来
arr2[0] = 100
arr2[1][1] = 100  # arr2将下标3的下标1的元素重新赋值100，arr1并不会受到影响，因为arr1和arr2此时指向的内存地址完全不同
print(arr1, arr2)  # [1, [2, 3]] [100, [2, 100]]
print(id(arr1), id(arr2))  # 2708559390280 2708559390344
print(id(arr1[1]), id(arr2[1]))  # 2708559655560 2708559390472
# 输出：
[1, [2, 3]] [100, [2, 100]]
2708559390280 2708559390344
2708559655560 2708559390472

Process finished with exit code 0
```

### 3.Unicode(兼容ASCII)码字符与int之间的转换

```python
if __name__ == '__main__':
    a_num = ord('a')  # Unicode码字符转int
    a_char = chr(97)  # int转Unicode码字符
    print(a_num)
    print(a_char)
    
# 输出：    
97
a

Process finished with exit code 0
```

### 4.将\x十六进制的字符型变量转换为int整型

```python
# 将\x十六进制的字符型转换为int整型
# 1.先循环拼接为二进制字符串
# 2.再转十进制
def bit_to_int(bits):
    ret = ''
    for bit in bits:
        tp = '{:08b}'.format(ord(bit))
        ret += tp
    return int(ret, 2)


if __name__ == '__main__':
    print(bit_to_int('\x00\x00\x00\x00\x00\x00\x00\x04'))
    print(bit_to_int('\x00\x00\x00\x00\x00\x00\xff\x04'))
    
# 输出：
4
65284

Process finished with exit code 0
```

### 5.Python的一致性

```python
# Python最好的品质之一：一致性
# Python的解释器遇到特定的语法时会去执行相对应的特殊方法，
# 所以对于用户编程来说，自定义的一些类型如果也想使用Python的特定语法上的话就实现相对应的特殊方法即可


# 自定义类型，实现水果列表类可以像Python内置类型列表一个使用
class FruitList:
    def __init__(self):
        self._fruits = ["苹果脆好甜", "香蕉甜", "雪梨可以煮汤"]

    def __getitem__(self, item):
        return self._fruits[item]

    # 特殊方法/魔法方法/双下划线方法
    def __len__(self):
        return len(self._fruits)


if __name__ == '__main__':
    fruits = FruitList()
    print(fruits[1])
    print(fruits[1:])
    print(len(fruits))

    # key 自定义排序规则
    fruits = sorted(fruits, key=lambda f: -len(f))
    for f in fruits:
        print(f)
    print('-' * 10)
    fruits = reversed(fruits)
    for f in fruits:
        print(f)
        
        
# 输出：
香蕉甜
['香蕉甜', '雪梨可以煮汤']
3
雪梨可以煮汤
苹果脆好甜
香蕉甜
----------
香蕉甜
苹果脆好甜
雪梨可以煮汤

Process finished with exit code 0
```

### 6.Python列表推导式和生成器表达式的区别

```python
# 如果想生成一个列表，可以使用列表推导式，列表推导式的作用只有一个就是生成列表
# 列表推导式和生成器表达式的区别：
# 1.列表推导式是一口气生成一个完整的列表，如果需要生成一个100万个元素的列表，那么列表推导式就会在内存中生成一个完整的100万个元素列表
# 2.生成器表达式就不同，如果需要生成一个100万个元素，那么生成器表达式会一个一个元素的生成，比如在for循环中，一次迭代生成一个元素，下一次迭代又会生成一个，而上一次迭代生成的元素会被垃圾回收掉！

import time
import sys


# 耗时统计的函数装饰器
def time_consuming(func):
    def inner():
        start = time.time()
        func()
        end = time.time()
        print(f"耗时：{end - start}秒")

    return inner


@time_consuming
def list_expression_func():
    # 列表推导式使用的是：[]方括号
    list_expression = [i for i in range(0, 10 ** 7)]
    print(f"数据类型：{type(list_expression)}")
    print(f"内存空间：{sys.getsizeof(list_expression)}字节")  # 约等于77MB


@time_consuming
def generator_expression_func():
    # 生成器表达式使用的是：()圆括号
    generator_expression = (i for i in range(0, 10 ** 7))
    print(f"数据类型：{type(generator_expression)}")
    print(f"内存空间：{sys.getsizeof(generator_expression)}字节")


list_expression_func()
print("\n", "*" * 10)
generator_expression_func()


# 输出：
数据类型：<class 'list'>
内存空间：81528056字节
耗时：0.6131536960601807秒

 **********
数据类型：<class 'generator'>
内存空间：120字节
耗时：0.0秒

Process finished with exit code 0
```

### 7.元组的使用

```python
# 元组的一些使用技巧，不单单是不可变类型这一特性，用于存储没有字段名称的记录
from collections import namedtuple

print("一、元组拆包（平行赋值），也可以成为可迭代元素拆包")
a, b, c = (1, 2, 3)
print(a, b, c)

print("二、使用*星号存储不关注的元素")
id, name, *_ = (1, "张三", "男", "北京", "码农")
print(id, name, _)
id, name, *_, job = (1, "张三", "男", "北京", "码农")
print(id, name, _, job)

print("三、嵌套拆包")
student_infos = ((1, "张三", (180, 75)), (2, "里斯", (170, 60)), (3, "娜娜", (160, 42)))
for id, name, (height, weight) in student_infos:
    print(id, name, height, weight)

print("四、具名元组，元组的元素也是可以有字段名")
nt_nana = namedtuple("nana", "id name")  # 定义一个具名元组对象，参数：对象名称，每个元素名称用空格隔开
nana = nt_nana(1, "娜娜")  # 将赋值元素给具名元组
print(nana)
print(nana.id, nana.name)  # 可以使用字段访问
print(nana[0], nana[1])  # 也可以使用位置来方法

# 输出：
一、元组拆包（平行赋值），也可以成为可迭代元素拆包
1 2 3
二、使用*星号存储不关注的元素
1 张三 ['男', '北京', '码农']
1 张三 ['男', '北京'] 码农
三、嵌套拆包
1 张三 180 75
2 里斯 170 60
3 娜娜 160 42
四、具名元组，元组的元素也是可以有字段名
nana(id=1, name='娜娜')
1 娜娜
1 娜娜

Process finished with exit code 0
```

### 8.序列切片用法

```python
# 在Python中支持切片[::]语法的序列类型有：列表（list）、元组（tuple）、字符串（str）
# 序列的元素下标从左到右是从0逐一递增的
# 序列的元素下标从右到左是从-1逐一递减的
# 例如：
#      0       1       2      3
#   ["张三", "李四", "王五", "赵六"]
#     -4      -3      -2      -1
#
#    0 1 2 3
#   "张李王赵"
#   -4-3-2-1

# 1.列表切片
names = ["张三", "李四", "王五", "赵六"]
print(names[1], names[-3])  # 李四 李四

# 2.元组切片（和列表相同用法）
names = ("张三", "李四", "王五", "赵六")
print(names[1], names[-3])  # 李四 李四

# 3.字符串切片（和列表相同用法）
names = "张李王赵"
print(names[1], names[-3])  # 李 李

# 4.如果将切片使用在赋值操作上可以达到就地修改的效果
names = ["张三", "李四", "王五", "赵六"]
names[1:3] = ["阿猫", "阿狗"]
print(names)  # ['张三', '阿猫', '阿狗', '赵六']

# 5.序列增量赋值 += *=
names = ["张三", "李四", "王五", "赵六"]
print(id(names), names)  # 1603624152904 ['张三', '李四', '王五', '赵六']
# 增量是会在原列表的基础上添加元素
names += ["阿猫", "阿狗"]
print(id(names), names)  # 1603624152904 ['张三', '李四', '王五', '赵六', '阿猫', '阿狗']
# 如果常规的+操作符则会新建一个列表
names = names + ["王富贵", "周芙蓉"]
print(id(names), names)  # 1603624151560 ['张三', '李四', '王五', '赵六', '阿猫', '阿狗', '王富贵', '周芙蓉']

# 6.特殊的例子
names = ("张三", "李四", ["北京", "上海"])
print(id(names), names)  # 1603624919112 ('张三', '李四', ['北京', '上海'])
try:
    names[2] += ["深圳", "广州"]
except Exception as ept:
    print(ept)  # 'tuple' object does not support item assignment
print(id(names), names)  # 1603624919112 ('张三', '李四', ['北京', '上海', '深圳', '广州'])
# 查看一条Python语句的CPU执行指令
import dis
dis.dis('names[2] += ["深圳", "广州"]')
# 查看执行的CPU指令
  1           0 LOAD_NAME                0 (names)	# 加载names序列
              2 LOAD_CONST               0 (2)		# 加载一个常量 2
              4 DUP_TOP_TWO							# 将names序列中的2下标的元素['北京', '上海']压到栈顶
              6 BINARY_SUBSCR
              8 LOAD_CONST               1 ('深圳')  # 加载一个常量 深圳
             10 LOAD_CONST               2 ('广州')  # 加载一个常量 广州
             12 BUILD_LIST               2			# 定义了一个列表
             14 INPLACE_ADD							# 将栈顶的对象['北京', '上海']进行添加操作：['北京', '上海'] + ['深圳', '广州']
             16 ROT_THREE	
             18 STORE_SUBSCR						# 存储names[2] = ['北京', '上海', '深圳', '广州']
             20 LOAD_CONST               3 (None)
             22 RETURN_VALUE

Process finished with exit code 0
```


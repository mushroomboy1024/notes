#### 坑1

示例代码：

```python
if __name__ == '__main__':
    class Person:
        def __init__(self, name):
            self.name = name

        def __repr__(self):
            return self.name


    list1 = [Person('张三')]
    print(list1)

    list2 = list1 * 2  # 浅拷贝！
    print(list2)

    list2[0].name = '李四'
    print(list2)
```

运行结果：

```python
[张三]
[张三, 张三]
[李四, 李四]

Process finished with exit code 0
```



#### 坑2

示例代码：

```python
if __name__ == '__main__':
    def timestamp_to_date(timestamp=time.time(), format='%Y-%m-%d %H:%M:%S'):
        """
        时间戳转为时间字符串
        :param timestamp: 时间戳，单位秒
        :param format: 格式
        :return: 时间字符串
        """
        return time.strftime(format, time.localtime(timestamp))

    print('Python虚拟机的JIT保存了函数信息'.center(30, '*'))
    for i in range(3):
        print(timestamp_to_date())
        time.sleep(1)

    print('每次调用都重新赋值'.center(30, '*'))
    for i in range(3):
        print(timestamp_to_date(time.time()))
        time.sleep(1)
```

运行结果：

```python
*****Python虚拟机的JIT保存了函数信息*****
2021-07-09 17:49:53
2021-07-09 17:49:53
2021-07-09 17:49:53
**********每次调用都重新赋值***********
2021-07-09 17:49:56
2021-07-09 17:49:57
2021-07-09 17:49:58

Process finished with exit code 0
```

|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |


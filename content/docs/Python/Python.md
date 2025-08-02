---
title: Python学习笔记
---



> 笔者为 Javaer 所以仅记录和Java语言的不同之处，方便快速学习



## 基础数据类型

- 数字

bool（布尔型）：True | False

> 笔者按： 布尔值要驼峰命令

complex （复数），如 1 + 2j、 1.1 + 2.2j。

> 笔者按： 已经忘记复数这个数学概念啦~

- 字符串

1. 索引

字符串的组成元素为字符，每个字符在字符串中有对应的索引。

> 和java不同的是，不光有正序索引，还有倒序索引

![image-20250725122701207](http://oss.chenayin.com/pic/image-20250725122701207.png)



2. 截取

字符串的截取，都是 前闭后开 或者 叫 含头不含尾

> python字符串的截取和索引强相关；
>
> 但又因为有正序和倒序，两排索引，所以截取时非常灵活，使用时也要谨慎

![image-20250725124333658](http://oss.chenayin.com/pic/image-20250725124333658.png)

> 图片来自网络，倒数第二个例子有点奇怪： 描述和结果都不对；要实现输出结果ab，可通过如下方式
>
> ```python
> print(str[:2]) # ab
> print(str[:-5]) # ab
> ```

```python
str="abcdefg"
print(str[1:]) # bcdefg
print(str[:5]) # abcde
print(str[1:5]) # bcde
print(str[-4:]) # defg
print(str[-2:-6]) # 空字符串 是 '' 不是None
print(str[-6:-2]) #bcde
print(str[::-1]) # 字符串反转  gfedcba
print(str[:2]) # ab
print(str[:-5]) # ab
```



- 列表

> 笔者按： 和Java中的数组特点一样； 最大最大的区别是 一个数组可以有不同类型的元素；
>
> 换句话说： 一个数组中可以同时存在 数字，字符串。
>
> ```python
> list1 = ['Google', 'baidu', 1997, 2000]
> ```

删除方法

```python
del list[1]
```

添加方法

```python
list.append('red')
```

---



**【思考】如何预创建一个容量为n的列表？**

📝 注意事项

- Python 列表是**动态数组**，不需要像 C++ 的 `vector` 或 Java 的 `ArrayList` 那样“预分配容量”来提高性能。
- 如果你只是想创建一个**将来要存 n 个元素的列表**，直接使用空列表即可：

python深色版本

```
lst = []
```

然后通过 `append()` 添加元素即可。

如果非要创建呢？

```python
lst = [None] * 6
print(lst)  # 输出: [None, None, None, None, None, None]
```



- 元组

和列表功能类似，区别有两个

1. 元祖不可变： 不可变指：元素数量和值在定义之后都不允许发生变化

2. 元祖用 () ; 列表用 [] .

```python

tup = (1, 2, 3, 4, 5 )
print(tup) # (1, 2, 3, 4, 5)

#tup[0] = 15
#print(tup) # TypeError: 'tuple' object does not support item assignment

#tup.append(6)
#print(tup) # AttributeError: 'tuple' object has no attribute 'append'
```



- 集合

使用方式:注意是 `{}` 

特点有三：

1. 可变
2. 无序
3. 元素唯一 （不重复）

> 和Java的Set的特点相似，在Python中也叫Set;

> 元素唯一的特性常常用来去重

```python
fruits = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
print(fruits) # 这⾥演示的是去重功能 {'pear', 'orange', 'banana', 'apple'}
print( 'orange' in fruits) # 快速判断元素是否在集合内 True
print( 'crabgrass' in fruits) # False
```



- 字典

字典就是KV；可以依据key，查询value

> 在Java中就是Map; put的方式不一样

字典的初始化、进和出 示例如下:

```python
zhangsan={
"name":"zhangsan",
"age":"17",
"height":"180",
"weight":"80kg"
}
#给字典添加元素
zhangsan["city"]="BeiJing"
#获取字典的元素
print(zhangsan.get("age"))
```



## 数据类型的判断和转换

### 判断

`type(变量)`

常规的判断就不展示了，展示一些有悖常规的。

```python
>>> type([1,2,3])
<class 'list'>
>>> type((1,2,3))
<class 'tuple'>
>>> type({1,2,3})
<class 'set'>
>>> type({'a':123})
<class 'dict'>
```

### 转换

隐式转换

```python
print('1'+1) # 报错
print(1+2.3) # 3.3 
```



显式转换 

> 常规和非常规 是笔者自己的逻辑划分； 
>
> 常规就是 不用理解直接用或者说 不学习也能猜到的正常使用方法；
>
> 非常规就是需要 认真学习一下，和平常想的可能会不一样

- 常规类

预定义函数 int()、float()、str() ；

- 非常规类

list()：将序列转换为⼀个列表。list()可以将字符串、元组、字典、集合转化为列表。

```python
>>> list('abc')
['a', 'b', 'c']
>>> list((1,2,3))
[1, 2, 3]
>>> list({1,2,3})
[1, 2, 3]
>>> list({'a':1,'b':2})
['a', 'b']
```

> 笔者按： 字符串和字典的转换真是看呆啦。 字符串直接切分字符串，字典则是转换为key的列表

set()：转换为可变集合。可以将字符串、列表、元组、字典转化为集合。

```python
>>> set('abc')
{'c', 'b', 'a'}
>>> set([1,2,3])
{1, 2, 3}
>>> set((1,2,3))
{1, 2, 3}
>>> set({'a':1,'b':2})
{'b', 'a'}
```

> 笔者按：set无序的特点还是蛮明显的

dict()：创建⼀个字典。字典必须是⼀个（key, value）元组序列。

```python
>>>dict() # 创建空字典
{}
>>> dict(a='a', b='b', t='t')#传⼊关键字
{'a': 'a', 'b': 'b', 't': 't'}
#传⼊2个元素的元组列表，元组的第⼀个元素代表key,第⼆个元素代表value
>>> dict([('one', 1), ('two', 2)])
{'two': 2, 'one': 1}
```

> 笔者按： 这里除了转换方式需要注意；
>
> 还有一个额外需要注意到的： 就是 元组的使用场景，通常都是一些强相关的元素的组合使用元组效果好。


---
title: "Python的一些用法（可能不定时更新）"
date: 2019-12-01T10:45:46+08:00
draft: false
categories: ["技术文章"]
---

## strip()、lstrip()、和rstrip()

Python strip() 方法用于移除字符串头尾指定的字符（默认为空格或换行符）或字符序列。

注意：该方法只能删除开头或是结尾的字符，不能删除中间部分的字符。

lstrip()就是从左边匹配然后删除字符，rstrip()从右边匹配然后删除字符。

表面上挺好理解的，但是用起来还是有一些陷阱。

如：
```python
if __name__ == '__main__':
    string = 'abcdefghijkl'
    print(string.lstrip('bac'))
    # 输出 defghijkl
```
可以看到，虽然左侧开头的'abc'和'bac‘顺序不同，但lstrip()方法依旧将其匹配然后删除了。

所以如果我只是要删除开头的某一部分，比如获取<a>标签内的字符：
```python
if __name__ == '__main__':
    string = '<a href="http://www.windypath.com">abcde</a>'
    print(string.lstrip('<a href="http://www.windypath.com">').rstrip('</a>'))
    # 输出 bcde
```

就会把<a>标签内容的最左边的a给匹配到了。

那么如何实现只根据字符顺序，匹配前面的字符呢？

用正则表达式：re.sub()

Python 的 re 模块提供了re.sub用于替换字符串中的匹配项。

语法：
```python
re.sub(pattern, repl, string, count=0, flags=0)
```
参数：

- pattern : 正则中的模式字符串。
- repl : 替换的字符串，也可为一个函数。
- string : 要被查找替换的原始字符串。
- count : 模式匹配后替换的最大次数，默认 0 表示替换所有的匹配。

使用该方法：
```python
import re
if __name__ == '__main__':
    string = '<a href="http://www.windypath.com">abcde</a>'
    clear_pre = re.sub(r'<a href="http://www.windypath.com">', '', string)
    clear_post = re.sub(r'</a>', '', clear_pre)
    print(clear_post)
    # 输出 abcde
```

## js的indexOf()对应python的index()

对于数组而言：

js的indexOf()返回元素在数组中的下标：

```python
var fruit = ['apple', 'banana', 'orange', 'pear']
console.log(fruit.indexOf('banana'))
// 输出 1
```

python里也有一个名字很像的index

```python
if __name__ == '__main__':
    fruit = ['apple', 'banana', 'orange', 'pear']
    print(fruit.index('banana'))
    # 输出 1
```
但对于字符串：

对于js而言，依旧可以使用indexOf()函数。

找得到的情况：
```javascript
var string = 'I love apple and banana'
console.log(string.indexOf('apple'))
// 输出 7
```
找不到的情况：
```javascript
var string = 'I love apple and banana'
console.log(string.indexOf('orange'))
// 输出 -1
```
找不到时，输出-1。

而python的index在字符串中找不到时，将抛出异常。
```python
if __name__ == '__main__':
    string = 'I love apple and banana'
    print(string.index('orange'))
    # Traceback (most recent call last):
    # File "/****.py", line 12, in <module>
    # print(string.index('orange'))
    # ValueError: substring not found
```
index()方法在找不到时，并不返回-1。

但find()函数在找不到时可以返回-1.
```python
if __name__ == '__main__':
    string = 'I love apple and banana'
    print(string.find('orange'))
    # 输出 -1
    print(string.find('apple'))
    # 输出 7
```

## python连接Oracle和MongoDB

这个。。待写，下篇文章再见。

2020.2.9已更新：[点击这里](/posts/python_connect_mongodb_and_oracle/)

### 文件读取f.truncate()

关于文件的w,a,r和它们的+，在网上资料很多，这里我只想备份一下：

f.truncate()是将文件清空的函数。

只有在w的模式下可以执行，在a下没有效果，r则抛出异常。
```python
if __name__ == '__main__':
    file_name = '测试.txt'
    with open(file_name, 'w') as f:
        f.truncate()
```
### 对dict对象的.keys()取出所有键

这是个非常实用的方法。
```python   
if __name__ == '__main__':
    obj = {
        'website': 'www.windypath.com',
        'name': '风萧古道',
        'favorite_team': 'Los Angeles Lakers'
    }
    print(obj.keys())
    # dict_keys(['website', 'name', 'favorite_team'])
```
### 使用range()和enumerate()对数组遍历

range()：传入某个整数，获取从0到该数-1的数组
```python
if __name__ == '__main__':
    sum = 5
    for i in range(sum):
        print(i)
    # 输出
    # 0
    # 1
    # 2
    # 3
    # 4
```
enumerate()：传入某个list，获取每个元素的下标和自身
```python
if __name__ == '__main__':
    fruit_list = ['apple', 'banana', 'orange', 'pear']
    for index, fruit in enumerate(fruit_list):
        print(index, fruit)
    # 输出
    # 0 apple
    # 1 banana
    # 2 orange
    # 3 pear
```
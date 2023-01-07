---
title: "CSAPP第二章-信息的表示与处理-随手记"
date: 2022-05-05T17:53:43+08:00
draft: false
categories: ["技术文章", "CSAPP"]
tags: ["CSAPP"]
---

> 仅作为学习《深入理解计算机系统》第二章时的笔记，仅记录对自己有启发的部分，不作为知识整理。（直接看电子书就可以了）。

>因为这本书知识点非常多，所以我会抽时间多次阅读，本文也会经常更新。

## 原码和反码会有两个0：正0和负0
原码：用第一个位来表示正负，后面的位来表示数的大小
反码：用一个正数取反来表示这个正数的相反数

这两种表示法都会存在两个0：+0和-0。

而使用补码就只有一个0了。

## 补码表示的新理解
关于补码，为了计算某个正数的相反数，可以用过取反+1的方式计算得到负数的补码表示。但是还有另一种方式能够更好的理解补码。
如果用5位来表示一个数：
|下标|4|3|2|1|0|代表的十进制数|
|:--|:--|:--|:--|:--|:--|:--|
|每个下标的数值|-16|8|4|2|1||
|二进制数1|0|1|1|0|1|13|
|二进制数2|1|1|1|0|1|-3|
|二进制数3|1|1|1|1|1|-1|
|二进制数3|0|1|1|1|1|15|

可以看到其实如果按照补码的逻辑，当使用5个位来存储数字时，最高位第5位作为符号位，它的数值为-16，其他第1到4位的数值为1、2、4、8，然后再对二进制数的各个位，乘以其对应的数值，再累加，就能得到十进制数的大小。
比如二进制数1，其十进制数=0\*(-16) + 1\*8 + 1\*4 + 0\*2 + 1\*1 = 13。
而负数二进制2，其十进制数=1\*(-16) + 1\*8 + 1\*4 + 0\*2 + 1\*1 = -3。

我们现在已经知道-3的二进制补码表示是：11101，很容易能通过这个数值表得到正3的二进制补码表示是00011。
使用上面的“取反后+1”的公式，也可以将-3转为正3：~11101 + 1 = 00010 + 1 = 00011

另外，还有两个比较有意思的内容：
1. 想获得某个二进制补码表示的最小的数，只需要让其符号位为1，其他位为0即可。
    比如在5位表示数的情况下，最小的数是-16，即10000。
2. 想获取-1，只需要让所有位都为1即可，那么想获得-1，可以直接用0按位取反。

## 按位左移和右移
左移无所谓，左移几位就往右边补0即可。
右移有两种，一种是算数右移，就是右移之后在左边补的位为符号位的那个数字；另一种是逻辑右移，右移之后在左边补0

## 按位移动位的数量，系统默认帮你取模
来源：[https://www.jianshu.com/p/304bfdda6b6a](https://www.jianshu.com/p/304bfdda6b6a)
控制硬件时，常涉及打开/关闭特定的位或查看他们的状态，一般都会使用到按位运算符技术。

一个面试题：
``` C
int a = 1, b = 32;

print("%d, %d", a<<b, 1<<32);
```
答案是 1，0

a << b 的结果是1，是因为运行时会将操作数b对32取模，然后在进行移位操作。


## 布尔代数
所谓布尔代数，就是按位与（&），或（|），非（~），异或（^）。
需要注意和强调的是（应该已经强调无数遍了），&和&&不一样，|和||不一样，~和负号-不一样。

比较有意思的是：
1. 异或有一个性质是：a ^ a = 0， (a ^ b) ^ a = b（因为0 ^ b = b）
2. (x | -x) >> 31 ，当x为0时依旧为0，当x不为0时，为-1

## 小端和大端
简单记的话，就记一个数 0x12345678
存储地址从小到大依次从左到右（有语病，但意思是那个意思）
大端存的是12 34 56 78
小端存的是78 56 34 12

## C语言unsigned int，会导致问题
一般来说数组的长度是大于等于0的，所以在设计过程中，为了能多获得一位存储空间，有的人会设计使用unsigned int存储。在C语言有一个size_t类型，其定义就是long unsigned int
但在遍历过程中，有可能会出现肉眼难以察觉的bug。

正常的代码：
``` C
   
    int a[5] = { 1, 2, 3, 4, 5 };

    int cnt = 5;
      printf("show values\n");
    for (int j = 0;j < cnt;j++) {
        printf("a[%d]=%d\n", j, a[j]);
    }

    int i;
    printf("start loop\n");
    for (i = cnt -2;i >= 0;i--) {
        printf("in loop,%u %u\n", i, cnt);
        a[i] += a[i+1];
    }
    printf("end loop\n");

    printf("show values\n");
    for (int j = 0;j < cnt;j++) {
        printf("a[%d]=%d\n", j, a[j]);
    }

```

此段代码给出一个数组a，该数组有5个元素。然后进行的操作是从数组的后面往前累加，最终a数组的第一个元素是之前a数组各元素的总和。

其运行结果如下：
``` shell
show values
a[0]=1
a[1]=2
a[2]=3
a[3]=4
a[4]=5
start loop
in loop,3 5
in loop,2 5
in loop,1 5
in loop,0 5
end loop
show values
a[0]=15
a[1]=14
a[2]=12
a[3]=9
a[4]=5
```

如果我们把代码“int i”改成“size_t i”会如何呢？
需要修改的错误代码：
``` C
   
    int a[5] = { 1, 2, 3, 4, 5 };

    int cnt = 5;
      printf("show values\n");
    for (int j = 0;j < cnt;j++) {
        printf("a[%d]=%d\n", j, a[j]);
    }

    size_t i;
    printf("start loop\n");
    for (i = cnt -2;i >= 0;i--) {
        printf("in loop,%u %u\n", i, cnt);
        a[i] += a[i+1];
    }
    printf("end loop\n");

    printf("show values\n");
    for (int j = 0;j < cnt;j++) {
        printf("a[%d]=%d\n", j, a[j]);
    }

```
输出结果：
``` shell
show values
a[0]=1
a[1]=2
a[2]=3
a[3]=4
a[4]=5
start loop
in loop,3 5
in loop,2 5
in loop,1 5
in loop,0 5
in loop,4294967295 5
in loop,4294967294 5
Segmentation fault (core dumped)
```
根据文章[https://blog.csdn.net/wang93IT/article/details/72782379](https://blog.csdn.net/wang93IT/article/details/72782379)所说：
> 有些时候我们在一段 C/C++ 代码的时候，由于对一个非法内存进行了操作，在程序运行的过程中，出现了“Segmentation fault (core dumped)”——段错误。

可以看到当i为0的时候，i--操作使i变成了一个特别大的数字，然后取a[i]时没有找到了非法内存，于是报错。
解决办法就是不要用size_t，也就是不要用无符号类型去作为下标索引。但如果a数组本身非常大呢？因为sizeof()函数的返回值类型就是size_t。

不要担心，在CSAPP课上，老教授给出了一种解决方案，虽然不是很符合正常逻辑：
修改后的正常代码：
``` C
   
    int a[5] = { 1, 2, 3, 4, 5 };

    int cnt = 5;
      printf("show values\n");
    for (int j = 0;j < cnt;j++) {
        printf("a[%d]=%d\n", j, a[j]);
    }

    size_t i;
    printf("start loop\n");
    for (i = cnt -2;i < cnt;i--) {
        printf("in loop,%u %u\n", i, cnt);
        a[i] += a[i+1];
    }
    printf("end loop\n");

    printf("show values\n");
    for (int j = 0;j < cnt;j++) {
        printf("a[%d]=%d\n", j, a[j]);
    }

```
仔细对比可以发现，在第二个for循环`for (i = cnt -2;i < cnt;i--)`中，其第二格判断条件从`i >= 0`改成了`i < cnt`。

运行结果：
``` shell
show values
a[0]=1
a[1]=2
a[2]=3
a[3]=4
a[4]=5
start loop
in loop,3 5
in loop,2 5
in loop,1 5
in loop,0 5
end loop
show values
a[0]=15
a[1]=14
a[2]=12
a[3]=9
a[4]=5
```
输出结果正常了。这是因为它借助了“无符号数减到（有符号数意义下的）负数时，会变成一个非常大的数”的性质。

为什么无符号的0再减一个正数就会变成很大的数呢？
以无符号0减去1为例：
假设有一个5位表示的数00000，如果这个数是二进制补码，则当00000减去1时，会得到11111。
11111在二进制补码中，指的是-1，但如果在无符号数中，它指的是UMax，也就是无符号数能表示的最大数。

## 二进制补码下，所有数字都有相反数吗？
不是的，TMin 100...000那个数字没有，也就是我们int下最小的数：-2147483648。
这个数字取反+1后会变成它自己，而不是正的2147483647。

但这也是为了在二进制补码中只有唯一的0作出的牺牲。

## 关于浮点数，非常形象的一张图
浮点数有五部分：
1. 0
2. 非规格化部分
3. 规格化部分
4. 无穷
5. 不是一个正数（NaN）

![](/images/float_number_table.png)
图片来自《深入理解计算机系统》第三版第80页。

## 浮点数的“舍入”
因为浮点数有精度限制，所以在运算中，需要进行一定的舍入，从而用有限的位来表示最接近目标实数的浮点数。
IEEE标准要求使用“向偶数舍入”，不是我们常说的四舍五入，而是四舍六入，五向偶数舍入。
四舍六入下：
1.4 -> 1
1.6 -> 2

向偶数舍入下：
1.5 -> 2
-1.5 -> -2
因为1.5两边的数字是1和2，要向偶数舍入，所以取2；同理，-1.5两边的数字是-1和-2，向偶数舍入，取-2。
关于二进制向偶数舍入，书上描述为：

![](/images/round_to_even.png)
图片来自《深入理解计算机系统》第三版第84页。

之所以这么做，是因为如果全部向上舍入/向下舍入，会出现统计偏差。而向偶数/奇数舍入，则有50%的几率向上舍入，50%向下舍入，从而避免统计偏差。

## 关于溢出
不管是二进制补码、无符号数或者浮点数，都会存在溢出的情况。而溢出这个行为本身，C语言不会给出任何警告。所以只能通过良好的编程习惯和思维去避免。

溢出部分在计算机中会舍去。这带来了一些问题。比如加法和乘法，很有可能超过了Tmax或者Umax。

## 关于除法
除法运算是十分消耗资源和时间的。计算机发展到现在，除法依旧需要消耗大量的CPU时钟。
但由于我们存储整数使用的是二进制补码/无符号数，所以如果我们的除数是2的幂，则可以通过位移来进行除法运算。

当负数需要进行除2的幂的时候，需要加上偏移量，来保证舍入正确。
> 仅做记录，还未研究。
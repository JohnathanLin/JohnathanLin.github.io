---
title: "如何用C++分割一个字符串？"
date: 2023-07-02T11:54:44+08:00
draft: false
categories: ["技术文章"]
---
# 前言
在上机面试的时候，遇到了一道题，它的输入是两行字符串，每行字符串有未知数量的数字（两行数字数量一致），用空格分隔开，输入形如：
```
12 34 567 888 99 100
358 74 58454 742 4469 88
```
并不提前提供每行的数字数量。而是让用户自己切分。

当时在上机考试时，我没有使用C++实现这一功能，而是使用Java里的split()进行处理。

后来，考试结束后，我上网查询C++切分字符串的写法，发现C++并没有原生提供类似split(某个字符)的写法。

那么有什么方法能替代呢？
# 方法1：使用string的find等函数()配合substr()进行切分
根据知乎大佬的回答，他提供的第一种解决方案是：

C++ 的 string 为什么不提供 split 函数？ - 知乎用户的回答 - 知乎
https://www.zhihu.com/question/36642771/answer/865135551

```C++
#include <iostream>
#include <cstring>
#include <vector>
void split(const std::string& s, std::vector<std::string>& tokens, const std::string& delimiters = " ") {
    std::string::size_type lastPos = s.find_first_not_of(delimiters, 0);
    std::string::size_type pos = s.find_first_of(delimiters, lastPos);
    while (std::string::npos != pos || std::string::npos != lastPos) {
        tokens.push_back(s.substr(lastPos, pos - lastPos));
        lastPos = s.find_first_not_of(delimiters, pos);
        pos = s.find_first_of(delimiters, lastPos);
    }
}
int main() {
    std::string str = "12 34 567 888 99 100";
    std::vector<std::string> res;
    split(str, res, " ");
    for (auto r: res) {
        printf("%s\n", r.c_str());
    }
    return 0;
}
```
输出结果：
```
12
34
567
888
99
100
```
这个split()函数，通过记录两个下标来确定需要裁出的字符串。
- 第一个下标lastPos，寻找从字符串开始后，第一个不是分隔符的字符下标；
- 第二个下标pos，寻找从lastPos之后，第一个是分隔符的字符下标。
如此一来，从lastPos到pos之间的字符串，就是我们需要裁出的字符串。

裁出第一个字符串后，将两个下标按之前的逻辑往后移动，直到两个下标都找不到合适的值（返回string::npos）时，结束。
## 函数介绍：
### find_first_of()
string的find_first_of()接收两个参数，第一个参数是要寻找的字符，它可能是string，char*或者char，第二个参数是开始寻找的下标（可以不传，默认传0）。
从第二个参数所指的字符串数组下标开始，往后寻找，直到找到要寻找的字符时，返回找到字符的下标。
### find_first_not_of()
string的find_first_not_of()接收的参数与find_first_of()一致，但它是寻找直到不是寻找字符时，返回不是寻找字符的下标。
### substr()
用于切分字符串，接收两个参数pos和len，从pos位置开始，切出往后len个长度的字符。不会修改原字符串。

具体的C++文档位置：
find_first_of()：https://cplusplus.com/reference/string/string/find_first_of/
find_first_not_of()：https://cplusplus.com/reference/string/string/find_first_not_of/
substr()：https://cplusplus.com/reference/string/string/substr/

# 方法2：C++11 正则表达式
``` c++
#include <iostream>
#include <vector>
#include <regex>
int main() {
    std::string str = "12 34 567 888 99 100";
  
    std::regex ws_re("\\s+");
    std::vector<std::string> res(
        std::sregex_token_iterator(
            str.begin(), str.end(), ws_re, -1
        ),
        std::sregex_token_iterator()
    );
    for (auto r: res) {
        printf("%s\n", r.c_str());
    }
    return 0;
}
```
这个例子是来自regex_token_iterator的介绍中。

https://en.cppreference.com/w/cpp/regex/regex_token_iterator

使用sregex_token_iterator()迭代器来进行切分操作（前面的s指代使用字符串类型string）。在这个例子中，9-11行中，构造了一个sregex_token_iterator迭代器，传入了4个参数，分别是：字符串的开始位置的迭代器，字符串结束的迭代器，正则表达式对象，是否使用匹配的部分（0使用，-1不使用）。

构造sregex_token_iterator()时，确定了要从字符串开始位置，查找到字符串末尾，通过正则表达式匹配，然后查找不匹配的部分。

该迭代器的末尾，是一个默认构造的sregex_token_iterator()对象。

vector的构造函数中，可以通过传入两个迭代器，获取迭代器之间的元素。

# 方法3：使用stringstream分割字符串（仅支持空格、回车、tab换行）
信息来源：https://www.cnblogs.com/narjaja/p/10044157.html
``` C++
#include <iostream>
#include <sstream>
#include <vector>
int main() {
    std::string str = "12 34 567 888 99 100";
    std::vector<std::string> res;
    std::istringstream ss(str);
    std::string word;
    while(ss>>word) {
        res.push_back(word);
    }
 
    for (auto r: res) {
        printf("%s\n", r.c_str());
    }
    return 0;
}
```
通过C++的 >> ，像用户cin一样，将字符串“输入”，从而得到切分的效果。

如果要支持自定义分隔符，则可以使用getline()进行处理
```C++
#include <iostream>
#include <sstream>
#include <vector>
 
using namespace std;
 
int main() {
    std::string data = "1_2_3_4_5_6";
    std::stringstream ss(data);
    std::string item;
    cout << data << endl;
    while (std::getline(ss, item, '_')) 
        cout << item << ' ';  
}
```

# 方法4：通过C语言的strtok()函数实现
## 第一种错误写法
还有一种做法是通过C语言的strtok()函数进行处理。
在网上能找到类似这样结构的代码：
```C++
#include <iostream>
#include <cstring>
int main() {
    char str[] = "A B C D";
    char* p = strtok(str, " ");
    std::cout << p << std::endl;
    while (p != NULL) {
        p = strtok(NULL, " ");
        std::cout << p << std::endl;
    }

    return 0;
}
```

也能正常得到结果：
```
A
B
C
D
```
但是，当我打算在第11行进行cout输出时，发现结果并不会正常输出：
```c++
#include <iostream>
#include <cstring>
int main() {
    char str[] = "A B C D";
    char* p = strtok(str, " ");
    std::cout << p << std::endl;
    while (p != NULL) {
        p = strtok(NULL, " ");
        std::cout << p << std::endl;
    }
    std::cout << "abc" << std::endl;
    return 0;
}
```
此时输出的结果还是：
```
A
B
C
D
```
并没有最后的abc。

问了ChatGPT后，得到的解释是：

这段C++代码中的问题是在循环输出切割后的子字符串时，先打印p的值再进行下一次循环。这会导致在最后一次循环结束后，p被设置为NULL，然后再尝试打印p的值，导致未定义行为。

在第8行，你打印了p的值，然后在第9行将其设置为下一个切割后的子字符串。接下来，在第10行的循环开始时，你再次打印p的值，但此时它可能已经为NULL，导致未定义行为。

为了修复这个问题，你可以将打印语句移到循环的末尾，这样先进行下一次循环再打印p的值，确保p不为NULL。

以下是修正后的代码：

``` c++
#include <iostream>
#include <cstring>

int main() {
    char str[] = "A B C D";
    char* p = strtok(str, " ");
    while (p != NULL) {
        std::cout << p << std::endl;
        p = strtok(NULL, " ");
    }
    std::cout << "abc" << std::endl;
    return 0;
}
```
这样修改后，代码将正常输出切割后的子字符串，然后输出"abc"。请注意，std::endl用于在输出后添加换行符，以保证输出结果的正确格式。

## 第二种错误写法
``` c++
#include <stdio.h>
#include<string.h>
int main()
{
  char s[] = "A B C D";
  char *p; 
  const char *d = " ";
  p = strtok(s, d);

  while(p)
  {  
    printf("%s\n", p);  
    p = strtok(NULL, " ");  

  }
  printf("abc");
  return 0;
}
```
这一次我们在循环体中，先输出上一次切分的p，再读取下一次的p。但字符串“abc”还是没有正常输出。ChatGPT解答如下：

ChatGPT

这段代码中无法正常输出"abc"的原因是因为在使用strtok函数对字符串进行分割时，它会将原字符串中的分隔符替换为'\0'（空字符）来分割子字符串。这导致原字符串s被修改，后续的printf语句在遇到'\0'时会提前结束。

在代码中，原字符串s的内容为"A B C D"，而strtok(s, d)会将它分割成"A"、"B"、"C"和"D"四个子字符串。因此，循环中的printf语句会输出这四个子字符串，分别为"A"、"B"、"C"和"D"。

然后，在最后一次调用strtok(NULL, " ")时，由于已经没有分隔符了，strtok会返回NULL，结束循环。然后，代码中的printf("abc")会尝试输出"abc"，但是由于之前的strtok调用修改了原字符串s，将其中的空格替换为了'\0'，所以在输出"abc"之前，printf函数遇到了'\0'字符，将其视为字符串的结尾，导致输出提前结束。

如果你希望能够输出"abc"，你可以在printf语句中添加换行符\n，如下所示：


``` c++
#include <stdio.h>
#include <string.h>

int main()
{
  char s[] = "A B C D";
  char *p;
  const char *d = " ";
  p = strtok(s, d);

  while (p)
  {
    printf("%s\n", p);
    p = strtok(NULL, " ");
  }

  printf("abc\n");  // 在"abc"后添加换行符\n
  return 0;
}
```
通过上述修改，代码将能够正常输出"A"、"B"、"C"、"D"和"abc"。
# strtok()函数
strtok()函数切割后，会在切断的位置上填充\0，会修改原字符串。所以一定要多加注意，慎用这个函数。
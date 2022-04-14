---
title: "java正则表达式 - 双反斜杠（\\）和Pattern的matches()与find()"
date: 2019-11-24T19:43:31+08:00
draft: false
categories: ["技术文章"]
---

> 参考文献
> [java正则表达式（find()和 matches()）](https://www.cnblogs.com/huhongy/p/7541875.html)
> [java正则表达式，求匹配：双反斜杠（\\）合法，单反斜杠不合法（\）](https://bbs.csdn.net/topics/390739619)
> [Java 正则表达式-菜鸟教程](https://www.runoob.com/java/java-regular-expressions.html)
> [正则表达式-菜鸟教程](https://www.runoob.com/regexp/regexp-syntax.html)

## Pattern类和Matcher类

在Java中，与正则表达式相关的类有两个：Pattern和Matcher

菜鸟教程已经介绍的很好了。
> java.util.regex 包主要包括以下三个类：
>
> - Pattern 类：pattern 对象是一个正则表达式的编译表示。Pattern 类没有公共构造方法。要创建一个 Pattern 对象，你必须首先调用其公共静态编译方法，它返回一个 Pattern 对象。该方法接受一个正则表达式作为它的第一个参数。
> - Matcher 类：Matcher 对象是对输入字符串进行解释和匹配操作的引擎。与Pattern 类一样，Matcher 也没有公共构造方法。你需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象。
> - PatternSyntaxException：PatternSyntaxException 是一个非强制异常类，它表示一个正则表达式模式中的语法错误。

然后菜鸟教程的第一个代码样例如下：
```java
import java.util.regex.*;
 
class RegexExample1{
   public static void main(String args[]){
      String content = "I am noob " +
        "from runoob.com.";
 
      String pattern = ".*runoob.*";
 
      boolean isMatch = Pattern.matches(pattern, content);
      System.out.println("字符串中是否包含了 'runoob' 子字符串? " + isMatch);
   }
}
```



但我尝试把他改成自己的正则表达式，如”^I am“来匹配开头的”I am"字符串时，我发现程序一直返回false。

这是为什么呢？

原因是，matches()函数是用于字符串全匹配的。若正则内的字符串与待匹配的字符串存在完全匹配，则返回false。

具体的阳602说的更好一些。
### find()和matches()



1.find()方法是部分匹配，是查找输入串中与模式匹配的子串，如果该匹配的串有组还可以使用group()函数。
matches()是全部匹配，是将整个输入串与模式匹配，如果要验证一个输入的数据是否为数字类型或其他类型，一般要用matches()。

2.
```java
Pattern pattern= Pattern.compile(".*?,(.*)");
Matcher matcher = pattern.matcher(result);
if (matcher.find()) {
    return matcher.group(1);
}
```
3.详解：
matches
public static boolean matches(String regex, CharSequence input)
编译给定正则表达式并尝试将给定输入与其匹配。
调用此便捷方法的形式
Pattern.matches(regex, input);
Pattern.compile(regex).matcher(input).matches() ;
如果要多次使用一种模式，编译一次后重用此模式比每次都调用此方法效率更高。
参数：
regex - 要编译的表达式
input - 要匹配的字符序列
抛出：
PatternSyntaxException - 如果表达式的语法无效

find
public boolean find()尝试查找与该模式匹配的输入序列的下一个子序列。
此方法从匹配器区域的开头开始，如果该方法的前一次调用成功了并且从那时开始匹配器没有被重置，则从以前匹配操作没有匹配的第一个字符开始。
如果匹配成功，则可以通过 start、end 和 group 方法获取更多信息。
matcher.start() 返回匹配到的子字符串在字符串中的索引位置.
matcher.end()返回匹配到的子字符串的最后一个字符在字符串中的索引位置.
matcher.group()返回匹配到的子字符串
返回：
当且仅当输入序列的子序列匹配此匹配器的模式时才返回 true。

4.部分JAVA正则表达式实例

①字符匹配
```java
Pattern p = Pattern.compile(expression); // 正则表达式
Matcher m = p.matcher(str); // 操作的字符串
boolean b = m.matches(); //返回是否匹配的结果
System.out.println(b);
```
```java
Pattern p = Pattern.compile(expression); // 正则表达式
Matcher m = p.matcher(str); // 操作的字符串
boolean b = m. lookingAt (); //返回是否匹配的结果
System.out.println(b);
```
```java
Pattern p = Pattern.compile(expression); // 正则表达式
Matcher m = p.matcher(str); // 操作的字符串
boolean b = m..find (); //返回是否匹配的结果
System.out.println(b);
```
②分割字符串
```java
Pattern pattern = Pattern.compile(expression); //正则表达式
String[] strs = pattern.split(str); //操作字符串 得到返回的字符串数组
```
③替换字符串
```java
Pattern p = Pattern.compile(expression); // 正则表达式
Matcher m = p.matcher(text); // 操作的字符串
String s = m.replaceAll(str); //替换后的字符串
```
④查找替换指定字符串
```java
Pattern p = Pattern.compile(expression); // 正则表达式
Matcher m = p.matcher(text); // 操作的字符串
StringBuffer sb = new StringBuffer();
int i = 0;
while (m.find()) {
    m.appendReplacement(sb, str);
    i++; //字符串出现次数
}
m.appendTail(sb);//从截取点将后面的字符串接上
String s = sb.toString();
```
⑤查找输出字符串
```java
Pattern p = Pattern.compile(expression); // 正则表达式
Matcher m = p.matcher(text); // 操作的字符串
while (m.find()) {
    matcher.start() ;
    matcher.end();
    matcher.group(1);
}
```
## 有趣的双反斜杠（\\）

在Java字符串中，存在诸如\n，\r等转义字符，而反斜杠\自己本身也是转义字符，所以在Java字符串中，要输出\，需要写两个\，即\\。

而正则表达式也需要匹配转义字符，故正则表达式要匹配一个\的时候，也需要写两个\\。

所以，在Java中使用正则表达式，要匹配一个\，需要写四个\。

## 正则表达式特殊字符（来自菜鸟教程）
|特别字符|	描述|
|:---|:---|
|$|	匹配输入字符串的结尾位置。如果设置了 RegExp 对象的 Multiline 属性，则 $ 也匹配 '\n' 或 '\r'。要匹配 $ 字符本身，请使用 \$。|
|( )|	标记一个子表达式的开始和结束位置。子表达式可以获取供以后使用。要匹配这些字符，请使用 \( 和 \)。|
|*|	匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \*。|
|+	|匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \+。|
|.	|匹配除换行符 \n 之外的任何单字符。要匹配 . ，请使用 \. 。|
|[	|标记一个中括号表达式的开始。要匹配 [，请使用 \[。|
|?	|匹配前面的子表达式零次或一次，或指明一个非贪婪限定符。要匹配 ? 字符，请使用 \?。|
|\	|将下一个字符标记为或特殊字符、或原义字符、或向后引用、或八进制转义符。例如， 'n' 匹配字符 'n'。'\n' 匹配换行符。序列 '\\' 匹配 "\"，而 '\(' 则匹配 "("。|
|^	|匹配输入字符串的开始位置，除非在方括号表达式中使用，此时它表示不接受该字符集合。要匹配 ^ 字符本身，请使用 \^。|
|{	|标记限定符表达式的开始。要匹配 {，请使用 \{。|
|\|	|指明两项之间的一个选择。要匹配 |，请使用 \|。|
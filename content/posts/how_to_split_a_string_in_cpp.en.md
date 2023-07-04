---
title: "How To Split A String In C++?"
date: 2023-07-02T11:54:44+08:00
draft: false
categories: ["Tech"]
---
# Preface
During a coding interview, I encountered a question that involved two lines of input. Each line contained an unknown number of numbers separated by spaces. The input format was as follows:
```
12 34 567 888 99 100
358 74 58454 742 4469 88
```
They did not provide the number of digits in each line in advance. Instead, they required the user to split them themselves.

During the coding exam, I didn't implement this functionality using C++, but instead used the split() method available in Java to handle it.

Later, after the exam was over, I searched online for ways to split strings in C++, and I found that C++ does not have a built-in method similar to split(character) like in Java.

So, what methods can be used as alternatives?
# Method 1: Splitting using string's find() function along with substr()
According to the response from a Zhihu user, the first solution they suggested is:

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
The output is：
```
12
34
567
888
99
100
```

The split() function works by keeping track of two indices to determine the substrings to be extracted:

The first index, lastPos, finds the index of the first character after the start of the string that is not a delimiter.
The second index, pos, finds the index of the first character after lastPos that is a delimiter.
By doing this, the substring between lastPos and pos is the portion of the string that needs to be extracted.
After extracting the first substring, both indices are moved forward according to the previous logic, until both indices cannot find suitable values (returning string::npos). This marks the end of the process.

## Function Introduction:
### find_first_of()
The find_first_of() function of string takes two parameters. The first parameter is the character to search for, which can be a string, char*, or char. The second parameter is the index to start searching from (it is optional and defaults to 0). It searches from the specified index onward until it finds the character being searched for, and then returns the index where the character is found.
### find_first_not_of()
The find_first_not_of() function of string accepts the same parameters as find_first_of(). However, it searches until it encounters a character that is not in the search characters. It returns the index of the first character that is not part of the search characters.
### substr()
The substr() function is used to extract a substring from a string. It takes two parameters, pos and len, indicating the starting position and the length of the substring to be extracted. The original string is not modified.


C++ document location：

find_first_of()：[https://cplusplus.com/reference/string/string/find_first_of/](https://cplusplus.com/reference/string/string/find_first_of/)


find_first_not_of()：[https://cplusplus.com/reference/string/string/find_first_not_of/](https://cplusplus.com/reference/string/string/find_first_not_of/)


substr()：[https://cplusplus.com/reference/string/string/substr/](https://cplusplus.com/reference/string/string/substr/)

# Method 2：C++11 regular expression
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
This example comes from the introduction to regex token iterator.

https://en.cppreference.com/w/cpp/regex/regex_token_iterator

Using the sregex_token_iterator() iterator for splitting operations (where s refers to a string type). In this example, in lines 9-11, a sregex_token_iterator iterator is constructed with four parameters: the iterator to the beginning of the string, the iterator to the end of the string, a regular expression object, and whether to use the matched parts (0 for using, -1 for not using).

By constructing the sregex_token_iterator(), it is determined to start searching from the beginning of the string and continue until the end of the string, using regular expression matching, and finding the unmatched parts.

The end of this iterator is represented by a default-constructed sregex_token_iterator() object.

In the constructor of vector, by passing in two iterators, the elements between the iterators can be obtained.

# Method 3: Splitting strings using stringstream (supports only space, newline, tab)
Source：[https://www.cnblogs.com/narjaja/p/10044157.html](https://www.cnblogs.com/narjaja/p/10044157.html)
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
By using the >> operator in C++, similar to how cin works for user input, we can "input" a string and achieve the splitting effect.


If you want to support custom delimiters, you can use getline() for handling.
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

# Method 4: Implementing with the C function strtok()
## The first incorrect approach
Another approach is to use the strtok() function from C language for handling the string splitting. 

On the internet, you may come across code structures similar to the following:
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

The output is：
```
A
B
C
D
```
However, when I attempted to perform a cout output on line 11, I found that the result was not displayed correctly:
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
The output is：
```
A
B
C
D
```

The final "abc" is missing in the output。

After consulting with ChatGPT, the explanation received is as follows:

The issue in this C++ code is that when iterating and outputting the split substrings, the value of p is printed first before proceeding to the next iteration. This causes the value of p to be set as NULL after the last iteration, and then attempting to print the value of p leads to undefined behavior.

In line 8, you print the value of p and then in line 9, it is set to the next split substring. Then, in line 10, when the loop begins again, you print the value of p again, but at this point, it may already be NULL, resulting in undefined behavior.

To fix this issue, you can move the printing statement to the end of the loop, so that the value of p is printed after proceeding to the next iteration, ensuring that p is not NULL.

Here is the corrected code:

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
After making this modification, the code will correctly output the split substrings and then output "abc". Please note that std::endl is used to add a newline character after the output to ensure the correct formatting of the output result.

## The second incorrect approach
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
This time, we output the previous split p first within the loop body and then read the next p. However, the string "abc" still doesn't output correctly. Here's ChatGPT's answer:

ChatGPT:

The reason why "abc" cannot be output correctly in this code is that when using the strtok function to split the string, it replaces the delimiter in the original string with a null character '\0' to separate the substrings. This modification alters the original string s, and subsequent printf statements will terminate prematurely when encountering '\0'.

In the code, the content of the original string s is "A B C D", and strtok(s, d) will split it into four substrings: "A", "B", "C", and "D". Therefore, the printf statement in the loop will output these four substrings: "A", "B", "C", and "D".

Then, in the last call to strtok(NULL, " "), since there are no more delimiters, strtok will return NULL and terminate the loop. Subsequently, the printf("abc") statement in the code will attempt to output "abc". However, due to the previous strtok calls modifying the original string s and replacing the spaces with '\0', when reaching the printf statement, the function encounters the '\0' character, treating it as the end of the string and causing the output to terminate prematurely.

If you want to output "abc", you can add a newline character \n to the printf statement as shown below:

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

With the above modification, the code will be able to output "A", "B", "C", "D", and "abc" correctly.

# strtok() Function
After the strtok() function performs the splitting, it fills the cut positions with '\0', thereby modifying the original string. Therefore, it is essential to exercise caution and use this function carefully.
通过上述修改，代码将能够正常输出"A"、"B"、"C"、"D"和"abc"。
strtok()函数
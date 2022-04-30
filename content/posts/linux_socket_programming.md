---
title: "Linux C++ Socket实战"
date: 2022-04-30T23:23:49+08:00
draft: false
tags: ["Linux", "Socket"]
categories: ["技术文章"]
---

> 本文主要介绍Linux C++ 基础Socket网络编程。
> 大部分知识来自于网站：[https://www.geeksforgeeks.org/socket-programming-cc/](https://www.geeksforgeeks.org/socket-programming-cc/)

## Socket编程状态图
![](/images/StatediagramforserverandclientmodelofSocketdrawio2-448x660.png)

从图中可以看到，服务端这边需要处理四步才能进入等待连接的状态，而客户端只要两步。

## Socket变成中各函数简单解析
本解析仅为自己理解所用，可能有些纰漏，有则改之。
原文中的知识总结得比我更好，尽量参考原文，我的理解仅做辅助之用。

### 服务端
先说服务端。服务端需要指定好端口并监听，所以需要bind()绑定好端口，需要listen()进入监听状态，然后通过accept()阻塞等待客户端的消息。

引用表：
- #include <sys/socket>
  - socket()
  - setsockopt()
  - bind()
  - listen()
  - accept()
- #include <netinet/in.h>
  - struct sockaddr_in
- #include <unistd.h>
  - read()
- #include <arpa/inet.h>
  - inet_pton()
#### socket()
这个函数是用来创建一个socket，3个参数中，需要特别传的就是前两个。返回一个socket编号，是个int值。

> int sockfd = socket(domain, type, protocol)

domain: IPV4 用 AF_INET， IPV6 用 AF_INET6
type: TCP 用 SOCK_STREAM, UDP 用 SOCK_UGRAM

#### setsockopt()
这个函数用来给上面那个socket()函数返回的socket设置属性，作为服务端，为了方便？ 可以设置重用地址和端口号。
> int setsockopt(int sockfd, int level, int optname,  const void *optval, socklen_t optlen);
为了重用地址和端口号，需要这么做：
- level传SOL_SOCKET，代表你这次设置的属性值是给哪个模块用的
- optname传SO_REUSEADDR|SO_REUSEPORT，代表你打算同时设置这两个属性
- optval传一个int*指针，指向某一个数字
- optlen传sizeof()上面的optval

C++ socket很多函数都需要你再传一个length长度，以确定你真正想传给这个函数的数据是多长。

#### sockaddr_in
那么地址在socket编程中是怎么表示的呢？是使用struct sockaddr_in来定义的。
用的时候需要设置3个值：sin_family, sin_addr的s_addr, sin_port
- sin_family：和前面socket()的domain一样，AF_INET
- sin_addr.s_addr：这个属性是将我们的点分十进制ip地址转化为一个数字，需要使用专门的函数来处理，比如inet_pton()
- sin_port: 指定端口号，但是不是直接传一个int数字，需要用htons转成16进制的数字



#### bind()
这个函数用来给socket绑定地址信息。

上一节已经说了地址怎么设置，这一节讲socket绑定地址。

> int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

照例将sockFd和address关联即可。
> 注意 这里的addr的类型是struct sockaddr * 而不是struct sockaddr_in *。
> struct sockaddr的结构里并没有提供存放ip地址，端口号的属性，所以需要用struct sockaddr_in来强制类型转换。
> 在文档中，作者说struct sockaddr和struct sockaddr_in长度一定是一样的，所以一定可以强制类型转换，让大家不要担心。
> [https://www.gta.ufrj.br/ensino/eel878/sockets/sockaddr_inman.html](https://www.gta.ufrj.br/ensino/eel878/sockets/sockaddr_inman.html)
> 注意，后面的addrlen是一个值，不是指针。

#### listen()
> int listen(int sockfd, int backlog);
这个函数将socket切换为被动模式，进入监听状态。第二个参数backlog指定消息等待队列的最大长度。

#### accept()
> int new_socket= accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
函数执行之后，socket就会等待客户端的连接。当连接建立之后，返回一个用于通信的新的socket，这个新的socket用于客户端与服务端之间的通信。

> 注意：accept()的第三个参数socklen_t *addrlen和bind()的第三个参数socklen_t addrlen不一样，accept需要一个指针。
> 有人说是accept()的参数是双向参数，会更新地址的长度值，但也有人说是在accept()函数里，它不知道int能不能存的下这个长度，万一长度特别大就不好存了，为了统一存储结构，仅传一个指针即可。



#### send()
send()用于通过socket来给对方发送消息。

#### read()
read()函数是放在#include <unistd.h>中的。函数通过一个char数组来存储接收到的消息

### 客户端
再说服务端。客户端需要指定好ip地址和端口号，然后发起连接。

#### connect()
> int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

客户端也要创建一个socket，指定sockaddr_in然后强转。这个socket不仅负责发起连接，也负责发送，接收数据。

### 示例代码（没有错误处理）
目前我只想学习这些socket底层api，所以不想浪费过多精力去记住api的返回值可能意味着什么错误，仅专注于能正常实现一个最简单的server和client。
#### 服务器server端
server.cpp
``` C++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <string.h>
using namespace std;
#define PORT 8000

int main() {
    int sockFd, newSockFd, valread;
    int opt = 1;
    char buffer[1024] = {0};
    char* helloFromServer = "hello from server";
    struct sockaddr_in address;

    sockFd = socket(AF_INET, SOCK_STREAM, 0);
    setsockopt(sockFd, SOL_SOCKET, SO_REUSEADDR|SO_REUSEPORT, &opt, sizeof(opt));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);
    int addrlen = sizeof(address);
    bind(sockFd, (struct sockaddr*)&address, addrlen);
    listen(sockFd, 3);
    newSockFd = accept(sockFd, (struct sockaddr*)&address, (socklen_t*)&addrlen);
    read(newSockFd, buffer, 1024);
    printf("receive: %s\n", buffer);
    send(newSockFd, helloFromServer, strlen(helloFromServer), 0);
    printf("server sent message\n");

    return 0;
}
```
#### 客户端client端
client.cpp
``` C++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#define PORT 8000
using namespace std;

int main() {
    int sockFd = 0;
    char buffer[1024] = {0};
    char* helloFromClient = "hello from client";
    struct sockaddr_in address;
    address.sin_family = AF_INET;
    inet_pton(AF_INET, "127.0.0.1", &address.sin_addr.s_addr);
    address.sin_port = htons(PORT);
    sockFd = socket(AF_INET, SOCK_STREAM, 0);
    

    connect(sockFd, (struct sockaddr*)&address, sizeof(address));
    send(sockFd, helloFromClient, strlen(helloFromClient), 0);
    printf("client sent\n");
    read(sockFd, buffer, 1024);
    printf("read message:%s\n", buffer);
    return 0;
}
```

#### 编译、运行和目标输出
在linxu命令行下，分别输入：
``` shell
g++ -o server server.cpp
g++ -o client client.cpp
```
然后开两个控制台，分别输入：
``` shell
./server
```
``` shell
./client
```

目标输出为：
server输出：
```
receive: hello from client
server sent message
```

client输出：
```
client sent message
receive: hello from server
```
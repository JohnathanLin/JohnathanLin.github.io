---
title: "Linux C++ Socket in Action"
date: 2022-04-30T23:23:49+08:00
draft: false
tags: ["Linux", "Socket"]
categories: ["Tech"]
---

> This article primarily introduces the basics of Linux C++ Socket network programming.
> Most of the knowledge is sourced from the website: [https://www.geeksforgeeks.org/socket-programming-cc/](https://www.geeksforgeeks.org/socket-programming-cc/)

## Socket Programming State Diagram
![](/images/StatediagramforserverandclientmodelofSocketdrawio2-448x660.png)

From the diagram, we can see that the server side needs to go through four steps to enter the "waiting for connection" state, while the client side only requires two.

## Brief Analysis of Socket Programming Functions
This analysis is for personal understanding and may have some omissions. If there are any, please correct them.
The original article summarizes the knowledge better than I do. Please refer to the original article as much as possible. My understanding is only for auxiliary purposes.

### Server
Let's start with the server. The server needs to specify a port and listen, so it needs to use bind() to bind the port, use listen() to enter the listening state, and then use accept() to block and wait for messages from the client.

C++ including:
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

This function is used to create a socket. Among the three parameters, the first two are particularly important. It returns a socket number, which is an integer value.

> int sockfd = socket(domain, type, protocol)

domain: Use AF_INET for IPv4, and AF_INET6 for IPv6.
type: Use SOCK_STREAM for TCP, and SOCK_DGRAM for UDP.

#### setsockopt()
This function is used to set properties for the socket returned by the socket() function above. As a server, for convenience, you can set the options for address and port reuse.

> int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

To reuse the address and port, you need to do the following:

- Set level to SOL_SOCKET, which represents which module the property value you are setting is for.
- Set optname to SO_REUSEADDR|SO_REUSEPORT, which means you intend to set both of these properties at the same time.
- optval should be a pointer to an int, pointing to a specific number.
- optlen should be sizeof() of the above optval.

Many C++ socket functions require you to pass a length to determine how long the data you really want to pass to this function is.

#### sockaddr_in

So how is an address represented in socket programming? It's defined using struct sockaddr_in.

When using it, you need to set three values: sin_family, sin_addr.s_addr, and sin_port.

- sin_family: Same as the domain parameter in the socket() function, it's AF_INET.
- sin_addr.s_addr: This property converts our dotted decimal IP address into a number, and it requires a dedicated function for processing, such as inet_pton().
- sin_port: Specifies the port number, but it's not directly passed as an integer. You need to use htons() to convert it into a hexadecimal number.


#### bind()
This function is used to bind address information to a socket.

In the previous section, we discussed how to set up the address. In this section, we'll talk about binding the socket with an address.

> int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

As usual, associate sockFd with the address.

>Note: The type of addr here is struct sockaddr *, not struct sockaddr_in *. 
> The structure of struct sockaddr does not provide properties to hold the IP address and port number, so you need to use struct sockaddr_in for type casting. 
> In the documentation, the author mentions that the lengths of struct sockaddr and struct sockaddr_in must be the same, so you can definitely perform the type casting without worry. 
> [https://www.gta.ufrj.br/ensino/eel878/sockets/sockaddr_inman.html](https://www.gta.ufrj.br/ensino/eel878/sockets/sockaddr_inman.html)
> Note that addrlen later is a value, not a pointer.

#### listen()
> int listen(int sockfd, int backlog);

This function switches the socket to passive mode, entering the listening state. The second parameter backlog specifies the maximum length of the message waiting queue.

#### accept()
> int new_socket= accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
After this function is executed, the socket will wait for a client connection. Once the connection is established, it returns a new socket for communication between the client and server.

> Note: The third parameter socklen_t *addrlen of accept() is different from the third parameter socklen_t addrlen of bind(). accept() requires a pointer.
> Some say that the parameter of accept() is a bidirectional parameter and will update the length value of the address. However, others say that in the accept() function, it doesn't know if an int can hold this length. If the length is particularly large, it might not be suitable for storage. To ensure a uniform storage structure, just pass a pointer.

#### send()
send() is used to send messages to the other party through the socket.

#### read()
The read() function is included in #include <unistd.h>. It uses a char array to store the received message.

### Client
Now let's talk about the client. The client needs to specify the IP address and port number, and then initiate the connection.

#### connect()
> int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

The client also needs to create a socket, specify sockaddr_in, and then typecast. This socket is responsible not only for initiating the connection but also for sending and receiving data.

### Sample Code (Without Error Handling)
Currently, I only want to learn these low-level socket APIs, so I don't want to spend too much effort on memorizing what the return values of the APIs might indicate in terms of errors. I just want to focus on successfully implementing a very basic server and client.

#### Server-side Code:
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

#### Client-side Code:
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
#### Compilation, Execution, and Targeted Output
In the Linux command line, input the following separately:
``` shell
g++ -o server server.cpp
g++ -o client client.cpp
```
Then open two terminals, and input the following respectively:
``` shell
./server
```
``` shell
./client
```
The expected outputs are:

For the server:
```
receive: hello from client
server sent message
```

client输出：
```
client sent message
receive: hello from server
```
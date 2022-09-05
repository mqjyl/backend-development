# Socket编程

网络编程也称为socket编程，socket通常译作“套接字”，Socket本身有“插座”的意思，在Linux环境下，用于表示进程间网络通信的特殊文件类型。**本质为内核借助缓冲区形成的伪文件。**

既然是文件，那么理所当然的，我们可以使用文件描述符引用套接字。与管道类似的，Linux系统将其封装成文件的目的是为了统一接口，使得读写套接字和读写文件的操作一致。区别是管道主要应用于本地进程间通信，而套接字多应用于网络进程间数据的传递。

在`TCP/IP`协议中，**`“IP地址+TCP或UDP端口号”`**唯一标识网络通讯中的一个进程。“IP地址+端口号”就对应一个socket。欲建立连接的**两个进程各自有一个socket来标识**，那么这两个socket组成的socket pair就唯一标识一个连接。因此可以用Socket来描述网络连接的一对一关系。

![](<../../.gitbook/assets/image (9).png>)

**在网络通信中，套接字一定是成对出现的。**一端的发送缓冲区对应对端的接收缓冲区。我们使用同一个文件描述符发送缓冲区和接收缓冲区。 TCP/IP协议最早在BSD UNIX上实现，为TCP/IP协议设计的应用层编程接口称为socket API，也就是操作系统提供给开发人员进行网络开发的API接口。这套接口通常可以通过参数的调整支持多种协议，包括`TCP`、`UDP`和`IP`等等。

## :pencil2: 1、基础知识

### :pen\_fountain: 1.1、网络字节序

我们已经知道，内存中的多字节数据相对于内存地址有大端和小端之分，磁盘文件中的多字节数据相对于文件中的偏移地址也有大端小端之分。网络数据流同样有大端小端之分，那么如何定义网络数据流的地址呢？发送主机通常将发送缓冲区中的数据按内存地址从低到高的顺序发出，接收主机把从网络上接到的字节依次保存在接收缓冲区中，也是按内存地址从低到高的顺序保存，因此，网络数据流的地址应这样规定：先发出的数据是低地址，后发出的数据是高地址。

`TCP/IP`协议规定，网络数据流应采用大端字节序，即低地址高字节。例如`UDP`段格式，地址0-1是16位的源端口号，如果这个端口号是`1000（0x3e8）`，则地址0是`0x03`，地址1是`0xe8`，也就是先发`0x03`，再发`0xe8`，这16位在发送主机的缓冲区中也应该是低地址存`0x03`，高地址存`0xe8`。但是，如果发送主机是小端字节序的，这16位被解释成`0xe803`，而不是1000。因此，发送主机把1000填到发送缓冲区之前需要做字节序的转换。同样地，接收主机如果是小端字节序的，接到16位的源端口号也要做字节序的转换。如果主机是大端字节序的，发送和接收都不需要做转换。同理，32位的IP地址也要考虑网络字节序和主机字节序的问题。&#x20;

为使网络程序具有可移植性，使同样的C代码在大端和小端计算机上编译后都能正常运行，可以调用以下库函数做**网络字节序和主机字节序的转换。**

```cpp
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

> h 表示host，n 表示network，l 表示32位长整数，s 表示16位短整数。&#x20;

如果主机是小端字节序，这些函数将参数做相应的大小端转换然后返回，如果主机是大端字节序，这些函数不做转换，将参数原封不动地返回。

### :pen\_fountain: 1.2、IP地址转换函数

早期：

```cpp
#include <arpa/inet.h>

/* Convert Internet host address from numbers-and-dots notation in CP
   into binary data and store the result in the structure INP.  */
int inet_aton (const char *__cp, struct in_addr *__inp);

/* Convert Internet host address from numbers-and-dots notation in CP
   into binary data in network byte order.  */
in_addr_t inet_addr (const char *__cp);

/* Convert Internet number in IN to ASCII representation.  The return value
   is a pointer to an internal array containing the string.  */
char *inet_ntoa (struct in_addr __in);
```

> 以上三个函数在点分十进制数串（如“127.0.0.1"）和32位网络字节序二进制值之间转换IPv4地址。是不可重入函数。

> **`inet_aton`**将`__cp`指向的字符串转成网络序的地址存在`__inp`指向的地址结构。成功返回1，否则返回0。(据书中所说，如果`__inp`指针为空，那么该函数仍然对输入字符串进行有效性检查但是不存储任何结果）
>
> **`inet_addr`**功能和`inet_aton`类似，但是`inet_addr`出错时返回`INADDR_NONE`常值（通常是32位均为1的值），这就意味着至少有一个`IPv4`的地址（通常为广播地址255.255.255.255）不能由该函数处理。建议使用`inet_aton`代替`inet_addr`。
>
> **`inet_ntoa`**将网络序二进制`IPv4`地址转换成点分十进制数串。该函数的返回值所指向的字符串驻留在静态内存中。这意味着该函数是不可重入的。同时我们也该注意到该函数以一个结构体为参数而不是常见的以一个结构体指针作为参数。

现在：

```cpp
#include <arpa/inet.h>

int inet_pton(int af, const char *src, void *dst);
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```

> 支持`IPv4`和`IPv6`，可重入函数。其中`inet_pton`和`inet_ntop`不仅可以转换`IPv4`的`in_addr`，还可以转换`IPv6`的`in6_addr`。因此函数接口是`void *addrptr`。函数的`__af`参数既可以是`AF_INET`也可以是`AF_INET6`。如果以不支持的地址族作为参数就会返回一个错误，并将`errno`置为`EAFNOSUPPORT`。
>
> **`inet_pton`**将字串转成对应的网络序二进制值，**`inet_ntop`**做相反的事情，其中`__len`参数指定目标缓冲区的大小，在`<netinet/in.h>`头文件中定义了：

```cpp
#define INET_ADDRSTRLEN 16    /* for IPv4 dotted-decimal */
#define INET6_ADDRSTRLEN 46     /* for IPv6 hex string */
```

> 如果给的`__len`太小那么会返回一个空指针，并置`errno`为`ENOSPC`。

### :pen\_fountain: 1.3、`sockaddr`数据结构

很多网络编程函数诞生早于`IPv4`协议，那时候都使用的是`sockaddr`结构体,为了向前兼容，现在`sockaddr`退化成了（`void *`）的作用，传递一个地址给函数，至于这个函数是`sockaddr_in`还是`sockaddr_in6`，由地址族确定，然后函数内部再强制类型转化为所需的地址类型。

![](<../../.gitbook/assets/image (7).png>)

```cpp
struct sockaddr {
	sa_family_t sa_family; 		/* address family, AF_xxx */
	char sa_data[14];			    /* 14 bytes of protocol address */
};
```

使用 `sudo grep -r "struct sockaddr_in {" /usr` 命令可查看到`struct sockaddr_in`结构体的定义。一般其默认的存储位置：`/usr/include/linux/in.h` 文件中。

```cpp
struct sockaddr_in {
	__kernel_sa_family_t sin_family; 			/* Address family */  	//地址结构类型
	__be16 sin_port;					 		        /* Port number */		//端口号
	struct in_addr sin_addr;					/* Internet address */	//IP地址
	/* Pad to size of `struct sockaddr'. */
	unsigned char __pad[__SOCK_SIZE__ - sizeof(short int) -
	sizeof(unsigned short int) - sizeof(struct in_addr)];
};

struct in_addr {						/* Internet address. */
	__be32 s_addr;
};
```

```cpp
struct sockaddr_in6 {
	unsigned short int sin6_family; 		/* AF_INET6 */
	__be16 sin6_port; 					/* Transport layer port # */
	__be32 sin6_flowinfo; 				/* IPv6 flow information */
	struct in6_addr sin6_addr;			/* IPv6 address */
	__u32 sin6_scope_id; 				/* scope id (new in RFC2553) */
};

struct in6_addr {
	union {
		__u8 u6_addr8[16];
		__be16 u6_addr16[8];
		__be32 u6_addr32[4];
	} in6_u;
	#define s6_addr 		in6_u.u6_addr8
	#define s6_addr16 	in6_u.u6_addr16
	#define s6_addr32	 	in6_u.u6_addr32
};
```

```cpp
#define UNIX_PATH_MAX 108
struct sockaddr_un {
	__kernel_sa_family_t sun_family; 	/* AF_UNIX */
	char sun_path[UNIX_PATH_MAX]; 	/* pathname */
};
```

`Pv4`和`IPv6`的地址格式定义在`netinet/in.h`中，`IPv4`地址用`sockaddr_in`结构体表示，包括16位端口号和32位IP地址，`IPv6`地址用`sockaddr_in6`结构体表示，包括16位端口号、128位IP地址和一些控制字段。UNIX Domain Socket的地址格式定义在`sys/un.h`中，用`sock-addr_un`结构体表示。各种socket地址结构体的开头都是相同的，前16位表示整个结构体的长度（并不是所有UNIX的实现都有长度字段，如Linux就没有），后16位表示地址类型。`IPv4`、`IPv6`和`Unix Domain Socket`的地址类型分别定义为常数`AF_INET`、`AF_INET6`、`AF_UNIX`。这样，只要取得某种`sockaddr`结构体的首地址，不需要知道具体是哪种类型的`sockaddr`结构体，就可以根据地址类型字段确定结构体中的内容。因此，socket API可以接受各种类型的`sockaddr`结构体指针做参数，例如`bind`、`accept`、`connect`等函数，这些函数的参数应该设计成`void *`类型以便接受各种类型的指针，但是sock API的实现早于ANSI C标准化，那时还没有`void *` 类型，因此这些函数的参数都用`struct sockaddr *`类型表示，在传递参数之前要强制类型转换一下，例如：

```cpp
struct sockaddr_in servaddr;
/* initialize servaddr */
bind(listen_fd, (struct sockaddr *)&servaddr, sizeof(servaddr));
```

## :pencil2: 2、网络套接字函数

![socket模型创建流程图](<../../.gitbook/assets/image (8).png>)

### :pen\_fountain: 2.1、socket函数

```cpp
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

> 参数：

> domain:
>
> * `AF_INET` 这是大多数用来产生socket的协议，使用TCP或UDP来传输，用IPv4的地址；
> * `AF_INET6` 与上面类似，不过是来用IPv6的地址；
> * `AF_UNIX` 本地协议，使用在Unix和Linux系统上，一般都是当客户端和服务器在同一台及其上的时候使用。

> type:
>
> * SOCK\_STREAM 这个协议是按照顺序的、可靠的、数据完整的基于字节流的连接。这是一个使用最多的socket类型，这个socket是使用TCP来进行传输。
> * SOCK\_DGRAM 这个协议是无连接的、固定长度的传输调用。该协议是不可靠的，使用UDP来进行它的连接。
> * SOCK\_SEQPACKET该协议是双线路的、可靠的连接，发送固定长度的数据包进行传输。必须把这个包完整的接受才能进行读取。
> * SOCK\_RAW socket类型提供单一的网络访问，这个socket类型使用ICMP公共协议。（ping、traceroute使用该协议）。
> * SOCK\_RDM 这个类型是很少使用的，在大部分的操作系统上没有实现，它是提供给数据链路层使用，不保证数据包的顺序。

> protocol:
>
> * 传0 表示使用默认协议。

> 返回值：成功：返回指向新创建的socket的文件描述符，失败：返回-1，设置`errno`。

### :pen\_fountain: 2.2、bind函数

```cpp
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

>> 参数：
>
> * `sockfd`：socket 文件描述符
> * `addr`：构造出IP地址加端口号
> * `addrlen`：sizeof(addr)长度

> 返回值：成功返回0，失败返回-1，设置`errno`。

服务器程序所监听的网络地址和端口号通常是固定不变的，客户端程序得知服务器程序的地址和端口号后就可以向服务器发起连接，因此服务器需要调用bind绑定一个固定的网络地址和端口号。 bind()的作用是将参数`sockfd`和`addr`绑定在一起，使`sockfd`这个用于网络通讯的文件描述符监听`addr`所描述的地址和端口号。前面讲过，`struct sockaddr *`是一个通用指针类型，`addr`参数实际上可以接受多种协议的`sockaddr`结构体，而它们的长度各不相同，所以需要第三个参数`addrlen`指定结构体的长度。如：

```cpp
struct sockaddr_in servaddr;
bzero(&servaddr, sizeof(servaddr));
servaddr.sin_family = AF_INET;
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
servaddr.sin_port = htons(6666);
```

首先将整个结构体清零，然后设置地址类型为`AF_INET`，**网络地址为`INADDR_ANY`，这个宏表示本地的任意IP地址**，因为服务器可能有多个网卡，每个网卡也可能绑定多个IP地址，这样设置可以在所有的IP地址上监听，直到与某个客户端建立了连接时才确定下来到底用哪个IP地址，端口号为6666。

### :pen\_fountain: 2.3、listen函数

```cpp
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>

int listen(int sockfd, int backlog);
```

> 参数：
>
> sockfd：socket文件描述符
>
> backlog：排队建立3次握手队列和刚刚建立3次握手队列的链接数和
>
> 返回值：成功返回0，失败返回-1。

查看系统默认backlog&#x20;：c`at /proc/sys/net/ipv4/tcp_max_syn_backlog`

当socket函数创建一个套接字时，它被假设为一个主动套接字，也就是说，它是一个将调用connect发起连接的客户套接字。listen函数把一个未连接的套接字转换为一个被动套接字，指示内核应该接受指向该套接字的连接请求。根据TCP状态转换图，调用listen导致套接字从CLOSED状态转换到LISTEN状态。

### :pen\_fountain: 2.4、accept函数

```cpp
#include <sys/types.h> 		/* See NOTES */
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

> 参数：

> sockdf：socket文件描述符

> addr：传出参数，返回链接客户端地址信息，含IP地址和端口号

> addrlen：传入传出参数（值-结果），传入`sizeof(addr)`大小，函数返回时返回真正接收到地址结构体的大小。

> 返回值：成功返回一个新的socket文件描述符，用于和客户端通信，失败返回-1，设置errno。

三方握手完成后，服务器调用accept()接受连接，如果服务器调用accept()时还没有客户端的连接请求，就**阻塞等待**直到有客户端连接上来。addr是一个传出参数，accept()返回时传出客户端的地址和端口号。addrlen参数是一个传入传出参数（value-result argument），传入的是调用者提供的缓冲区addr的长度以避免缓冲区溢出问题，传出的是客户端地址结构体的实际长度（有可能没有占满调用者提供的缓冲区）。如果给addr参数传NULL，表示不关心客户端的地址。

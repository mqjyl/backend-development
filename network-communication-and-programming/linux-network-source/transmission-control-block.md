# 传输控制块TCB

{% embed url="https://blog.csdn.net/xiaoyu_750516366/article/details/83242042" %}

TCP协议目的是为了保证数据能在两端准确连续的流动，可以想象两个建立起TCP通道的设备就如同接起了一根水管，数据就是水管中的水由一头流向另一头。然而TCP为了能让一个设备连接多根“水管”，让一个设备能同时与多个设备交互信息，它必须要保证不同水管之间不会产生串联或相互影响，一根水管中的水绝不能流入另一根水管，要保证这样的效果，TCP协议使用socket数据结构来实现不同设备之间的连接。

socket包含两个成分，一个是IP地址，一个是端口号。同一个设备可以对应一个IP端口，但不同的“水管”用不同的端口号区分开来，于是同一个设备发送给其他不同设备的信息就不会产生混乱。在同一时刻，设备可能会产生多种数据需要分发给不同的设备，为了确保数据能够正确分发，TCP用一种叫做TCB，也叫传输控制块的数据结构把发给不同设备的数据封装起来，我们可以把该结构看做是信封。

一个TCB数据块包含了数据发送双方对应的socket信息以及拥有装载数据的缓冲区。在两个设备要建立连接发送数据之前，双方都必须要做一些准备工作，分配内存建立起TCB数据块就是连接建立前必须要做的准备工作。我们还需要了解的一点是TCP连接的建立方式，由于TCP协议建立在服务器–客户端的模式之上，因此对于两种不同角色的设备，他们发起连接的方式不一样。

客户端发起连接的方式叫Active Open。也就是客户端需要主动向服务器发送消息，表达自己想建立数据连接的请求,通常而言客户端会向服务器发送一个SYNC数据包。服务器发起连接的方式叫Passive Open，通来说服务器不可能知道当前时刻有哪个设备想向它发起连接，因此它只能构建一个端口，然后监听该端口，等待客户端从该端口向它发起连接请求。在OPEN阶段无论是客户端还是服务器都需要准备好TCB数据结构，但由于服务器不知道要连接它的客户端信息，因此在构建TCB模块时会默认将客户端对应的socket数据初始化为0.

当双方都把自己的socket和TCB数据结构准备好后，双方就可以进入所谓的“三次握手”连接建立过程

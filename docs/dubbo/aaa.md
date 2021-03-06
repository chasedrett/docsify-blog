# TCP 篇

## TCP 报文格式

![image-20210820151318988](https://gitee.com/chasedrett/sdfgldsjgljslgjhosj/raw/upload/images/20210820151319.png)

- 源端口号和目的端口号：可将其想象为 16 个 0 和 1；
- 序号：也叫 seq，32 位，计算机随机生成的一个数字，用来对报文进行标识；
- 确认序号：也叫 ack，32 位，用来表示对接收到的报文的应答；
- 6 个标志位（各占 1 位）需要用到其中 3 个：
  - ACK：确认序号标志位，注意是标志位不是序号，只有 1 位，即只能表示 0 和 1；ACK = 1 表示确认序号有效；
  - SYN：用于发起新连接。如果 SYN = 1，表示这个连接是新发起的；
  - FIN：用于结束连接。FIN = 1，表示我要结束一个连接；



## TCP 三次握手

![image-20210820152827438](https://gitee.com/chasedrett/sdfgldsjgljslgjhosj/raw/upload/images/20210820152827.png)

- 第一次：客户端向服务端握手，建立连接，SYN = 1，随机生成一个 seq，假设为 10000；

> seq 可判断报文是否合法。比如一开始是 10000 的，后面应该是 10001、10002 。。。如果突然来个 900，肯定不合法，丢弃。

- 第二次：服务端向客户端握手，发送确认与建立连接，ACK = 1，ack = 接收到的 seq + 1；同时回传 SYN = 1，随机生成一个 seq；
- 第三次：客户端向服务端握手，确认连接，ACK = 1，ack = 收到的 seq + 1；



## TCP 四次挥手

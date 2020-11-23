---
title: 网络协议学习记录_tcp
tags: ['tcp']
---
不管作为前端还是后端，对网络协议的掌握还是很有必要的，深入理解tcp/ip协议、udp协议、http1/2、https等，可以使我们对开发、技术有更深的理解。最近阅读了图解http、http in action两本书，再次对所学知识做下总结

## OSI模型（开放式系统互联通信参考模型）
**自下而上分为七层：物理层(网线、Wi-Fi、网卡等硬件设备)—>数据链接层(以太网)->网络层(ip)->传输层(tpc)->会话层(https安全加密SSL、TLS)->表现层(文件格式：JPG、PNG、UTF-8等)->应用层(http超文本传输协议)**

身为前端，我认为目前需要深入了解的是应用层、会话层、传输层、网络层
一般来说，一次完整的请求经历的过程是：
* 客户端发起http请求，在请求头中Content-type会标识请求数据的格式；
* DNS解析域名获取服务器ip地址，再通过ARP协议查询服务器MAC（网卡所属的固定地址）地址;
* 客户端与服务器三次握手建立tcp链接;
* 客户端与服务器互相验证带有加密公钥的证书，建立https安全通信链接；
* 在https的基础上，对客户端、服务器间的数据加密后，进行基于tcp协议的数据传输。
上面只是进行一次请求的基本流程，后续文章会针对每个流程进行深入讲解。

## tpc/ip协议
### 目前常用的数据传输协议是tpc、udp；
* tcp协议提供了一种可靠的数据传输方式，它在数据传输过程中通过慢启动算法，拥塞窗口，丢包后重新发送数据包等方式保证数据的完整性、有序性，但同时这些也很大程度的影响了tcp传输数据的性能，例如在丢包的情况下，tcp协议下会发生数据的队头阻塞，其他完成的数据要等丢包的数据重新发送完成，才可以被使用；

* udp协议相对于tcp就简单了很对，udp只进行数据的传输，对与数据是否完整有序的到达并不保证，在丢包的情况下，不会重新发送丢失的数据包，因此不会造成数据堵塞，这种协议一般用于对丢包容忍度较高的场景，例如视频直播、在线游戏等；

### tcp为了保证数据完整的传输，需要进行三次握手建立可靠的链接
* 客户端向服务器发送一个链接请求，报文中带有SYN标志；
* 服务端收到请求链接报文后，如果同意链接，会向客户端发送带有SYN、ACK标记的确认报文；
* 客户端收到确认报文后，还有向服务器发送带有ACK标志的确认报文，完成tcp连接；

**为什么要发送第三次确认报文，是为了防止已失效的链接请求报文突然传送到服务器，使服务器以为客户端要再次建立连接，产生不必要的浪费**

#### 四次挥手断开tcp链接
* 客户端发送链接断开报文，报文首部带有FIN标记；
* 服务器手袋断开链接报文后，发送首部带有ACK标志的确认报文， 此时服务器就会进入CLOSE-WAIT(等待关闭)状态；
* 服务器将最后的数据发送完成后，向客户端发送带有FIN标志的断开链接报文，此时服务器链接处于半关闭，进入LAST-ACK(最后确认)状态； 
* 客户端接收到服务器断开链接报文后，发送带有ACK标志的确认报文；注意此时TCP连接还没有释放，必须经过2*MSL（最长报文段寿命）的时间后，当客户端撤销相应的TCB后，才进入CLOSED状态。
* 服务器收到确认后就会立即进入CLOSE状态。

#### 为什么客户端最后还要等待2MSL
保证客户端发送的最后一个确认报文能够到达服务器，因为最后的确认报文有可能丢失，如果服务器发送的断开链接报文没有收到客户端的确认，服务器就会重新发送一次，当客户端再次收到断开链接的报文后，会重新发送确认报文并重新计算2*MSL的时间；同时和三次握手类似，为了防止已失效的断开链接的影响；

#### 为什么握手是三次挥手是四次
因为服务端收到客户端的断开链接报文后，只代表客户端不会发送新的数据的，但仍可以接受未接受完成的数据，所以需要服务器额外发送一个FIN标志的报文表示数据发送完成，客户端才可以发送确认报文，完成断开链接；



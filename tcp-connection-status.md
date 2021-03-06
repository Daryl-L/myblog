# TCP 状态转移

在《Linux 高性能服务器编程》中，有下面这样的状态转移图。

![](https://o90cnn3g2.qnssl.com/tcp%E7%8A%B6%E6%80%81%E8%BD%AC%E7%A7%BB%E5%9B%BE.png)

## TCP 的各个状态

### 客户端

##### 建立连接（三次握手）

- `SYN_SENT` 在客户端发送第一个同步报文段（**第一次握手**）之后，就会进入这个状态。

- `ESTABLISHED` 在收到服务端发送的确认和同步报文段（**第二次握手**）后，客户端只需要发送出一个确认报文段（**第三次握手**），就算完成了三次握手了，即客户端认为连接已经建立。

##### 断开连接（四次挥手）

- `FIN_WAIT_1` TCP 断开连接时，需要进行四次挥手。当客户端主动关闭连接，发出第一个 `FIN` 报文段（**第一次挥手**）后，即进入此状态。

- `FIN_WAIT_2` 服务端在收到第一个 `FIN` 报文段后，回复给客户端确认报文段（**第二次挥手**）后的状态。其实 `FINE_WAIT` 的含义，就是等待服务端发来的 `FIN` 报文段。

- `TIME_WAIT` 在客户端收到服务端的 `FIN` 报文（**第三次挥手**）后，返回给服务端确认报文段（**第四次挥手**）后的状态。

### 服务端

##### 建立连接（三次握手）

- `CLOSED` 一个假想的起点和终点，不是一个实际的状态。

- `LISTEN` 在程序启动后，等待客户端连接的状态。

- `SYN_RCVD` 在每一个 TCP 连接建立时，都要进行三次握手，这个状态表示服务端接收到客户端发来的同步报文段（**第一次握手**），并且向客户端发送了确认同步报文段（**第二次握手**）之后的状态，在这个状态时，其实连接已经经历了两次握手。

- `ESTABLISHED` 收到客户端发来的确认报文段（**第三次握手**），即三次握手成功后，双方正式建立连接的状态。

##### 断开连接（四次挥手）

- `CLOSE_WAIT` 在收到客户端主动断开连接的 `FIN` 报文段（**第一次挥手**）后，返回给客户端确认报文段（**第二次挥手**）后的状态。

- `LAST_ACK` 服务端发送 `FIN` 报文段（**第三次挥手**）后的状态。在该状态之后，收到确认报文段（**第四次握手**），连接就关闭了。

## 时序图

对于书中的状态转移图，有一些复杂和抽象。我根据自己的分析，画出了一份时序图。

### 建立连接（三次握手）

![](https://o90cnn3g2.qnssl.com/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%283%29.png)

### 断开连接（四次挥手）

![](https://o90cnn3g2.qnssl.com/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%284%29.png)

## TIME_WAIT 状态

第一次看这个转移图的时候，我很疑惑，为什么要有一个 `TIME_WAIT` 状态？为什么不能直接到达 `CLOSED` 状态？相信很多人看到这个状态转移图，也会有这样的疑惑。其实，原因大致有下面两点。

#### 可靠地终止连接

假设没有 `TIME_WAIT` 这种状态。现实中，网络环境不是理想的。在数据包传输的过程中，难免会有一些延时、丢包的情况发生。如果，在客户端最后一个确认报文段发出去之后，由于某种原因，没有到达服务端，这样，服务端在超时后，就会向客户端重新发一个 `FIN` 报文段，请求重传。由于在客户端，连接实际上已经断开，端口已经关闭。那么在客户端收到这个报文段后，会向服务端发送一个 `RST` 报文段。而服务器收到该报文段后，会认为是错误的，它所期望收到的是确认报文段。

#### 保证让迟来的报文段有足够的时间被处理

同样在不理想的网络环境中，有些包会在网络中有所延迟。假如没有 `TIME_WAIT` 这种状态，在关闭连接后并且建立新的连接后，可能会收到该数据包。由于序号不同，客户端会要求服务端重传数据包，这样，连接就会混乱出错。而 `TIME_WAIT` 状态的时间，一般是 `2MAL` 时间，并且端口没有释放。这样，前一个连接的报文段有足够的时间被识别或者丢弃，也就不会出现这个问题。
 





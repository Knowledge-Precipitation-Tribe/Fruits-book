# 什么是拥塞控制
流量控制是避免发送方的数据填满了接收方的缓存，但是并不知道网络中现在处于什么情况，当TCP发现网络出现拥塞时，TCP就会主动降低发送的数据量。

# 如何发现网络发生拥塞
只要接收方没有在规定的时间内收到ACK的应答报文，也就发生了超时重传，就会认为网络出现了拥塞。

# 如何表示网络拥塞
拥塞窗口cwnd是发送方维护的一个状态变量，它会根据网络的拥塞情况动态变化。加入拥塞窗口后，发送窗口swnd就是接收窗口rwnd与拥塞窗口cwnd的最小值

cwnd的变化规则：
- 只要网络中没有出现拥塞，cwnd就会增大；
- 网络中出现了拥塞，cwnd就减少；

# 如何实现拥塞控制
![](https://tva1.sinaimg.cn/large/008eGmZEgy1goa3vla6aej30mf0ba0tx.jpg)

## 慢启动
![](https://tva1.sinaimg.cn/large/008eGmZEgy1goa3wmt5byj30mh0eaab6.jpg)

TCP在连接建立完成后，首先有一个慢启动的过程，就是一点一点的提高发送数据包的数量。当发送方每收到一个ACK，拥塞窗口cwnd的大小就会加1。

慢启动增长到什么时候
- 慢启动门限ssthresh（slow start threshold）状态变量，一般来说ssthresh的大小是65535字节。
- 当cwnd<ssthresh时，使用慢启动算法。
- 当cwnd>=ssthresh时，使用拥塞避免算法。

## 拥塞避免
![](https://tva1.sinaimg.cn/large/008eGmZEgy1goa3wtnwppj30ls0idt9n.jpg)

每当收到一个 ACK 时，cwnd增加1/cwnd。拥塞避免算法就是将原本慢启动算法的指数增长变成了线性增长，还是增长阶段，但是增长速度缓慢了一些。

持续增长下去就会出现丢包现象，就会触发重传机制，此时就需要拥塞发生算法。

## 拥塞发生
当发生拥塞时就需要调用两种重传机制：
- 超时重传：ssthresh和cwnd的值会发生变化，ssthresh设为cwnd/2，cwnd重置为1。
![](https://tva1.sinaimg.cn/large/008eGmZEgy1goa3yrlt3gj30mk0h3q4c.jpg)

- 快速重传：cwnd=cwnd/2 ，也就是设置为原来的一半，ssthresh=cwnd，进入快速恢复算法，快速重传和快速恢复算法一般同时使用，快速恢复算法是认为，你还能收到3个重复ACK说明网络也不那么糟糕，所以没有必要像RTO超时那么强烈。
	- 快速恢复
![](https://tva1.sinaimg.cn/large/008eGmZEgy1goa40i0hysj30mi0gvta7.jpg)
		
		- 拥塞窗口cwnd = ssthresh + 3（3的意思是确认有3个数据包被收到了）
		- 重传丢失的数据包
		- 如果再收到重复的 ACK，那么 cwnd 增加 1
		- 如果收到新数据的 ACK 后，把 cwnd 设置为第一步中的 ssthresh 的值，原因是该 ACK 确认了新 的数据，说明从 duplicated ACK 时的数据都已收到，该恢复过程已经结束，可以回到恢复之前的 状态了，也即再次进入拥塞避免状态；


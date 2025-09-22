
1. 通常情况下,一个端口在同一时间只能被一个程序占用。这是为了避免通信冲突。

2. 但是,TCP和UDP是两种不同的传输层协议。Linux确实允许一个TCP程序和一个UDP程序同时使用相同的端口号。

3. 这是因为TCP和UDP使用不同的协议栈,操作系统可以区分开来自TCP和UDP的数据包。

4. 此外,IPv4和IPv6也被视为不同的协议。所以理论上可以有一个IPv4 TCP程序、一个IPv4 UDP程序、一个IPv6 TCP程序和一个IPv6 UDP程序同时使用同一个端口号。



```cpp
void ip_protocol_deliver_rcu(struct net *net, struct sk_buff *skb, int protocol)
{
	const struct net_protocol *ipprot;
	int raw, ret;

resubmit:
	raw = raw_local_deliver(skb, protocol);
	// 查找协议
	ipprot = rcu_dereference(inet_protos[protocol]);
	if (ipprot) {
		// 调用`ipprot->handler`指向的处理函数,如`tcp_v4_rcv`或`udp_rcv`。
		ret = INDIRECT_CALL_2(ipprot->handler, tcp_v4_rcv, udp_rcv,
				      skb);
		if (ret < 0) {
			protocol = -ret;
			goto resubmit;
		}
		__IP_INC_STATS(net, IPSTATS_MIB_INDELIVERS);
	} 
}
```


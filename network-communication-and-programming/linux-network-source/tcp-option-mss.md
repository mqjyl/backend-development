# TCP选项之MSS

## ****:pencil2: **1、三次握手**

### ****:pen\_fountain: **1.1、客户端三次握手**

客户端处理`MSS`的机会为：

1. 发送SYN段时告诉服务器端本端能够接收的`MSS`；
2. 收到`SYN+ACK`后接收服务器端通告的`MSS`。

#### :hamster: **1.1.1、**SYN段MSS选项值

SYN段也是通过`tcp_transmit_skb()`发送的，在该函数中会调用`tcp_syn_build_options()`构造SYN段携带的选项，代码如下：

```cpp
static int tcp_transmit_skb(struct sock *sk, struct sk_buff *skb, int clone_it,
			    gfp_t gfp_mask)
{
...
	//如果发送的是SYN段，则调用tcp_syn_build_options()构造要携带的TCP选项，其中
	//MSS选项的值就是tcp_advertise_mss()的返回值
	if (unlikely(tcb->flags & TCPCB_FLAG_SYN)) {
		tcp_syn_build_options((__be32 *)(th + 1),
				      tcp_advertise_mss(sk),
				      (sysctl_flags & SYSCTL_FLAG_TSTAMPS),
				      (sysctl_flags & SYSCTL_FLAG_SACK),
				      (sysctl_flags & SYSCTL_FLAG_WSCALE),
				      tp->rx_opt.rcv_wscale,
				      tcb->when,
				      tp->rx_opt.ts_recent,
#ifdef CONFIG_TCP_MD5SIG
				      md5 ? &md5_hash_location :
#endif
				      NULL);
	}
...
}
```

**`tcp_advertise_mss()`**

advertise为广告的意思，该函数用来计算要告诉对端的`MSS`值，对应到TCB中，其实就是根据本端设备的`MTU`计算`tp->advmss`的值。

```cpp
static __u16 tcp_advertise_mss(struct sock *sk)
{
	struct tcp_sock *tp = tcp_sk(sk);
	//获取路由
	struct dst_entry *dst = __sk_dst_get(sk);
	//用tp->advmss初始化临时变量mss
	int mss = tp->advmss;

	//如果路由有效并且路由中的MSS比当前值小，那么用路由中的MSS更新tp->advmss
	//因为路由中的MSS是根据网络设备的MTU得来的，必须尊重，可以认为路由中的MSS
	//取值为min(65536-40, MTU-40)，其中65535为IP报文的最大长度
	if (dst && dst_metric(dst, RTAX_ADVMSS) < mss) {
		mss = dst_metric(dst, RTAX_ADVMSS);
		tp->advmss = mss;
	}
	//返回确定的mss
	return (__u16)mss;
}
```

从上面可以看出，`tcp_advertise_mss()`实际上就是取当前`tp->advmss`和路由表`MSS`二者中的最小值，而后者就是本地网卡的`MTU-40`，`tp->advmss`的初始化是在`tcp_connect_init()`中完成的，该函数被`tcp_connect()`调用，也就是说该值的初始化是在SYN段的发送过程中进行的，代码如下：

```cpp
static void tcp_connect_init(struct sock *sk)
{
...
	//即也是来源于本端MTU
	tp->advmss = dst_metric(dst, RTAX_ADVMSS);
...
}
```

**SYN段中携带的`MSS`选项值实际上就是本端网络设备的`MTU-40`。**

#### :hamster: **1.1.2、**接收SYN+ACK段

接收`SYN+ACK`段是在`tcp_rcv_synsent_state_process()`中处理的，其中会调用`tcp_paser_option()`解析SYN段中携带的选项，其中和`MSS`选项相关的代码如下：

```cpp
void tcp_parse_options(struct sk_buff *skb, struct tcp_options_received *opt_rx,
		       int estab)
{
...
	switch (opcode) {
	case TCPOPT_MSS:
		if (opsize == TCPOLEN_MSS && th->syn && !estab) {
			//取得选项中携带的MSS值记录到临时变量in_mss中
			u16 in_mss = ntohs(get_unaligned((__be16 *)ptr));
			if (in_mss) {
				//user_mss是应用程序通过TCP选项TCP_MAXSEG设定的，如果不设置，默认为0；
				//如果设定了user_mss并且设定的值小于对端通告的，那么调整in_mss为两者中的最小值。
				if (opt_rx->user_mss &&
					opt_rx->user_mss < in_mss)
					in_mss = opt_rx->user_mss;
				//将min(user_mss, in_mss)记录到mss_clamp中
				opt_rx->mss_clamp = in_mss;
			}
		}
		break;
	}
	...
}
```

总结：客户端在收到服务器端通告的`MSS`后，将其与应用程序通过`TCP_MAXSEG`设定的`MSS`值比较，将二者中的较小值保存在`tp->rx_opt.mss_clamp`中，该值会影响到客户端发送`MSS`的确定。

### :pen\_fountain: 1.2、服务端三次握手

服务器端处理`MSS`的机会为：

1. 收到SYN段后对客户端通告的`MSS`的处理；
2. 发送`SYN+ACK`时携带的`MSS`选项值的确定；

#### :hamster: 1.2.1、接收SYN段

SYN段的核心处理在`tcp_v4_conn_request()`中完成的，其中和`MSS`选项解析相关的内容如下：

```cpp
int tcp_v4_conn_request(struct sock *sk, struct sk_buff *skb)
{
...
	//将SYN段中携带的选项先解析到临时变量tmp_opt中
	struct tcp_options_received tmp_opt;
	struct request_sock *req;
...

	//先清空临时变量
	tcp_clear_options(&tmp_opt);
	//将临时变量中的mss_clamp初始化为536
	tmp_opt.mss_clamp = 536;
	//user_mss设置为用户通过套接字选项TCP_MAXSEG设定的值
	tmp_opt.user_mss  = tcp_sk(sk)->rx_opt.user_mss;
	//该函数上面已经见过，它会比较SYN段中携带的MSS和user_mss，
	//然后取二者中较小者保存在tmp_opt.mss_clamp中
	tcp_parse_options(skb, &tmp_opt, 0);
...
	//初始化连接请求块
	tcp_openreq_init(req, &tmp_opt, skb);
...
}

static inline void tcp_openreq_init(struct request_sock *req,
				    struct tcp_options_received *rx_opt,
				    struct sk_buff *skb)
{
...
	//最终确定下来的MSS记录到了连接请求块的mss字段中
	req->mss = rx_opt->mss_clamp;
...
}
```

记录到`req`中的`mss`，在三次握手完成后，创建子套接字的`TCB`时会赋值给`tp->rx_opt.mss_clamp`，代码如下：

```cpp
struct sock *tcp_create_openreq_child(struct sock *sk, struct request_sock *req, struct sk_buff *skb)
{
...
	newtp->rx_opt.mss_clamp = req->mss;
...
}
```

从代码上看，其实服务器端接收到SYN后，对`MSS`选项的处理流程与客户端收到`SYN+ACK`段后对`MSS`选项的处理流程是相同的。

#### :hamster: 1.2.2、发送SYN+ACK段

`SYN+ACK`段的发送过程主要由`tcp_v4_send_synack()`完成，其中和`MSS`选项相关的内容如下：

```cpp
static int tcp_v4_send_synack(struct sock *sk, struct request_sock *req,
			      struct dst_entry *dst)
{
...
	//创建SYN+ACK段
	skb = tcp_make_synack(sk, dst, req);
...
}

struct sk_buff *tcp_make_synack(struct sock *sk, struct dst_entry *dst,
				struct request_sock *req)
{
...
	//为SYN+ACK段构造TCP选项，第二个参数就是要发送给客户端的MSS，来自于路由
	tcp_syn_build_options((__be32 *)(th + 1), dst_metric(dst, RTAX_ADVMSS), ireq->tstamp_ok,
			      ireq->sack_ok, ireq->wscale_ok, ireq->rcv_wscale,
			      TCP_SKB_CB(skb)->when,
			      req->ts_recent,
			      (
#ifdef CONFIG_TCP_MD5SIG
			       md5 ? &md5_hash_location :
#endif
			       NULL)
			      );
...
}
```

从代码上看，其实服务器端发送`SYN+ACK`段时，对`MSS`选项的处理流程与客户端发送SYN段时对`MSS`选项的处理流程是相同的。

#### :hamster: 1.2.3、收到ACK段

在上面，我们并没有看到服务器端对`tp->advmss`的初始化，实际上，这个过程是在收到握手的第三个`ACK`报文后执行的，代码如下：

```cpp
struct sock *tcp_v4_syn_recv_sock(struct sock *sk, struct sk_buff *skb,
				  struct request_sock *req,
				  struct dst_entry *dst)
{
...
	//取值就是路由中的MSS值
	newtp->advmss = dst_metric(dst, RTAX_ADVMSS);
...
}
```

### :pen\_fountain: 1.3、三次握手总结

从前面的代码中可以看到，无论是服务器端还是客户端，它们对通告给对端的`MSS`值的选择方式是一样的，都是取自本地网卡的`MTU-40`，该值会被记录在`tp->advmss`中；收到对端通告的`MSS`后，和应用程序设定的`tp->rx_opt.user_mss`比较后，取二者中的较小者保存在`tp->rx_opt.mss_clamp`中，`mss_clamp`会影响发送过程中发送`MSS`的选择。

## :pencil2: 2、发送过程中获取MSS

&#x20;在[TCP数据发送之`tcp_sendmsg()`](tcp-send-tcp\_sendmsg.md)中有看到，`tcp_sendmsg()`的核心逻辑就是根据`MSS`将待发送数据切割成一个个的`skb`，过程中是通过调用`tcp_current_mss()`确定当前的发送`MSS`的。这篇笔记前面有提到，对端通告的`MSS`经过矫正后最终保存在了`tp->rx_opt.mss_clamp`中，按道理我们使用该值作为发送`MSS`就很好，然而并非如此简单，如下：

```cpp
int tcp_sendmsg(struct kiocb *iocb, struct socket *sock, struct msghdr *msg,
		size_t size)
{
...
	//获取当前有效的MSS，如果没有带外数据，就可以使用更大的段（TSO相关）
	mss_now = tcp_current_mss(sk, !(flags&MSG_OOB));
	size_goal = tp->xmit_size_goal;
...
}
```

### `tcp_current_mss()`

该接口用于获取当前有效的`MSS`，并且会根据`MTU`的大小设定`tp->xmit_size_goal`变量，该变量后续将用于组织`skb`，它的取值和`TSO`、`GSO`等特性相关，不是这篇笔记要讨论的话题，先忽略。

```cpp
/* Compute the current effective MSS, taking SACKs and IP options,
 * and even PMTU discovery events into account.
 *
 * LARGESEND note: !urg_mode is overkill, only frames up to snd_up
 * cannot be large. However, taking into account rare use of URG, this
 * is not a big flaw.
 */
unsigned int tcp_current_mss(struct sock *sk, int large_allowed)
{
	struct tcp_sock *tp = tcp_sk(sk);
	struct dst_entry *dst = __sk_dst_get(sk);
	//下面最终决策出的当前有效发送MSS值会记录到该变量中
	u32 mss_now;
	u16 xmit_size_goal;
	int doing_tso = 0;

	//mss_now来自于tp->mss_cache,一脸懵逼，到目前为止还没见过该字段（见下文）
	mss_now = tp->mss_cache;

	//判断是否允许TSO，忽略
	if (large_allowed && sk_can_gso(sk) && !tp->urg_mode)
		doing_tso = 1;

	if (dst) {
		//获取路由中保存的PMTU值
		u32 mtu = dst_mtu(dst);
		//icsk_pmut_cookie为上次缓存的PMTU值，其初始值为本端MTU大小，
		//如果二者不等，则说明PMTU发生了变化，需要调用tcp_sync_mss()更新MSS
		if (mtu != inet_csk(sk)->icsk_pmtu_cookie)
			mss_now = tcp_sync_mss(sk, mtu);
	}
	//如果TCP有SACK选项，则从MSS中减去相应的开销
	if (tp->rx_opt.eff_sacks)
		mss_now -= (TCPOLEN_SACK_BASE_ALIGNED + (tp->rx_opt.eff_sacks * TCPOLEN_SACK_PERBLOCK));

#ifdef CONFIG_TCP_MD5SIG
	if (tp->af_specific->md5_lookup(sk, sk))
		mss_now -= TCPOLEN_MD5SIG_ALIGNED;
#endif

	//下面的代码用来确定xmit_size_goal的值，该值和TSO相关，先忽略
	xmit_size_goal = mss_now;
	if (doing_tso) {
		xmit_size_goal = (65535 -
				  inet_csk(sk)->icsk_af_ops->net_header_len -
				  inet_csk(sk)->icsk_ext_hdr_len -
				  tp->tcp_header_len);

		xmit_size_goal = tcp_bound_to_half_wnd(tp, xmit_size_goal);
		xmit_size_goal -= (xmit_size_goal % mss_now);
	}
	tp->xmit_size_goal = xmit_size_goal;
	//返回当前有效的MSS值
	return mss_now;
}
```

### `tcp_sync_mss()`

该函数用参数`pmtu`更新`PMTU`相关字段，其中`icsk->icsk_pmtu_cookie`保存的就是之前缓存的`PMTU`值，根据该`PMTU`值计算`MSS`后，将计算结果保存到`tp->mss_cache`中，该值就是当前最新的`MSS`值。

```cpp
/* This function synchronize snd mss to current pmtu/exthdr set.

   tp->rx_opt.user_mss is mss set by user by TCP_MAXSEG. It does NOT counts
   for TCP options, but includes only bare TCP header.

   tp->rx_opt.mss_clamp is mss negotiated at connection setup.
   It is minimum of user_mss and mss received with SYN.
   It also does not include TCP options.

   inet_csk(sk)->icsk_pmtu_cookie is last pmtu, seen by this function.

   tp->mss_cache is current effective sending mss, including
   all tcp options except for SACKs. It is evaluated,
   taking into account current pmtu, but never exceeds
   tp->rx_opt.mss_clamp.

   NOTE1. rfc1122 clearly states that advertised MSS
   DOES NOT include either tcp or ip options.

   NOTE2. inet_csk(sk)->icsk_pmtu_cookie and tp->mss_cache
   are READ ONLY outside this function.		--ANK (980731)
 */
//注释很重要，仔细看
unsigned int tcp_sync_mss(struct sock *sk, u32 pmtu)
{
	struct tcp_sock *tp = tcp_sk(sk);
	struct inet_connection_sock *icsk = inet_csk(sk);
	int mss_now;

	if (icsk->icsk_mtup.search_high > pmtu)
		icsk->icsk_mtup.search_high = pmtu;
	//根据PMTU值计算MSS，这里除了标准的IP、TCP首部外，还会考虑IP选项、TCP选项
	mss_now = tcp_mtu_to_mss(sk, pmtu);
	//调整MSS为当前发送窗口的一半
	mss_now = tcp_bound_to_half_wnd(tp, mss_now);

	/* And store cached results */
	//将PMTU缓存到icsk_pmtu_cookie中
	icsk->icsk_pmtu_cookie = pmtu;
	if (icsk->icsk_mtup.enabled)
		mss_now = min(mss_now, tcp_mtu_to_mss(sk, icsk->icsk_mtup.search_low));
	//最终决策出来的MSS保存在mss_cache中
	tp->mss_cache = mss_now;

	return mss_now;
}
```

### `tcp_mtu_to_mss()`

该函数将`MTU`转变为`MSS`值，会考虑IP选项、`TCP`选项。

```cpp
/* Not accounting for SACKs here. */
int tcp_mtu_to_mss(struct sock *sk, int pmtu)
{
	struct tcp_sock *tp = tcp_sk(sk);
	struct inet_connection_sock *icsk = inet_csk(sk);
	int mss_now;

	/* Calculate base mss without TCP options:
	   It is MMS_S - sizeof(tcphdr) of rfc1122
	 */
	//从MTU中减去标准TCP首部、IP首部
	mss_now = pmtu - icsk->icsk_af_ops->net_header_len - sizeof(struct tcphdr);

	/* Clamp it (mss_clamp does not include tcp options) */
	//MSS不能超过对端通告的MSS
	if (mss_now > tp->rx_opt.mss_clamp)
		mss_now = tp->rx_opt.mss_clamp;

	/* Now subtract optional transport overhead */
	//减去扩展首部，启用IPsec时，会有扩展首部
	mss_now -= icsk->icsk_ext_hdr_len;

	/* Then reserve room for full set of TCP options and 8 bytes of data */
	//MSS最小不能小于48字节
	if (mss_now < 48)
		mss_now = 48;

	/* Now subtract TCP options size, not including SACKs */
	//减去TCP选项长度（不包括选择ACK选项），tp->tcp_header_len的值是该TCP连接中可能的最大TCP
	//首部长度，该值是在三次握手过程中根据双方对TCP选项的支持情况确定的
	mss_now -= tp->tcp_header_len - sizeof(struct tcphdr);

	return mss_now;
}
```

实际上在三次握手过程中，`TCP`会多次调用`tcp_sync_mss()`更新`MSS`值，不过其原理相同，无非是根据设备`MTU`或者当前最新的`PMTU`更新。

## :pencil2: 3、总结

![](../../.gitbook/assets/100.png)

`MSS`的更新规则：`PMTU`会更新到路由`metrics[RTAX_MTU]`中，每次发送操作时，都会重新检查是否需要更新发送`MSS`，此时会首先检查路由中的`MSS`和`icsk_pmtu_cookie`中缓存的上一次`PMTU`值，如果二者不等，说明两次发送期间`MTU`发生了变化，这时会用路由`MSS`更新`icsk_pmtu_cookie`。然后决定最终的发送`MSS`时也会参考三次握手过程中协商的值`mss_clamp`，保证最终值不会超过该值，而`mss_clamp`又受限于`TCP`选项`TCP_MAXSEG`。

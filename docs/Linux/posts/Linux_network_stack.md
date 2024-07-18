## Case study: Receiving data

### High-level data path

The high level path a packet takes from arrival to socket receive buffer is as follows:

1. Driver is loaded and initialized.
2. Packet arrives at the NIC from the network.
3. Packet is copied (via DMA) to a ring buffer in kernel memory.
4. Hardware interrupt is generated to let the system know a packet is in memory.
5. Driver calls into [NAPI](http://www.linuxfoundation.org/collaborate/workgroups/networking/napi) to start a poll loop if one was not running already.
6. `ksoftirqd` processes run on each CPU on the system. They are registered at boot time. The `ksoftirqd` processes pull packets off the ring buffer by calling the NAPI `poll` function that the device driver registered during initialization.
7. Memory regions in the ring buffer that have had network data written to them are unmapped.
8. Data that was DMA’d into memory is passed up the networking layer as an ‘skb’ for more processing.
9. Incoming network data frames are distributed among multiple CPUs if packet steering is enabled or if the NIC has multiple receive queues.
10. Network data frames are handed to the protocol layers from the queues.
11. Protocol layers process data.
12. Data is added to receive buffers attached to sockets by protocol layers.



### Detailed process

This blog post will look at the `igb` network driver. This driver is used for a relatively common server NIC, the Intel Ethernet Controller I350.

#### Initial setup

> Devices have many ways of alerting the rest of the computer system that **some work is ready for processing**. In the case of network devices, it is common for the NIC to **raise an IRQ** to signal that a packet has arrived and is ready to be processed.

![sofirq systegm diagram](https://cdn.buttercms.com/hwT5dgTatRdfG7UshrAF)

The initialization of the _softIRQ_ system:

1. softIRQ **kernel threads** are created (one per CPU): ksoftirq
2. The ksoftirqd threads begin executing their **processing loops** in the `run_ksoftirqd` function
3. `softnet_data` structures are created (one per CPU), including `poll_list`
4. `net_dev_init` then registers the `NET_RX_SOFTIRQ` softirq with the softirq system. The handler function  `net_rx_action` is the function the softirq kernel threads will execute to process packets.



#### Data arrives

> **DMA**:  a feature of computer systems that allows certain hardware subsystems to access main system [memory](https://en.wikipedia.org/wiki/Computer_memory) independently of the [central processing unit](https://en.wikipedia.org/wiki/Central_processing_unit) (CPU). [4]
>
> With DMA, the CPU first initiates the transfer, then it does other operations while the transfer is in progress, and it finally receives an [interrupt](https://en.wikipedia.org/wiki/Interrupt) from the DMA controller (DMAC) when the operation is done.
>
> Many hardware systems use DMA, including [disk drive](https://en.wikipedia.org/wiki/Disk_storage) controllers, [graphics cards](https://en.wikipedia.org/wiki/Video_card), [network cards](https://en.wikipedia.org/wiki/Network_interface_controller) and [sound cards](https://en.wikipedia.org/wiki/Sound_card).

![NIC DMA data arrival](https://cdn.buttercms.com/yharphBYTEm2Kt4G2fT9)

1. Data is received by the NIC from the network
2. The NIC uses DMA to write the network data to RAM (ring buffer)
3. The NIC raises IRQ
4. The device driver's registered IRQ handlder is executed
5. The IRQ is cleared on the NIC (in order to generate IRQs for new arriving packets)
6. NAPI softIRQ poll loop is started with a call to `napi_schedule`



The call to `napi_schedule` triggers the step 5-8 in setup:

5. `napi_schedule` adds the driver's NAPI poll structure to the `poll_list` for the current CPU

6. The **softirq pending bit** is set, so that the ksoftirq process on this CPU knows that there are packets to process

7. `run_ksoftirqd` function (which is being run in a loop by the `ksoftirq` kernel thread) executes

8. `__do_softirq` is called which [checks the pending bitfield](https://packagecloud.io/blog/monitoring-tuning-linux-networking-stack-receiving-data/#dosoftirq), sees that a softIRQ is pending, and calls the handler registered for the pending softIRQ: `net_rx_action`

**NOTICE**: the softIRQ kernel thread is executing `net_rx_action`, not the device driver IRQ handler



**SUMMARY**: in general, the first two sections have done those things:

1. create some threads to continuously execute some functions
2. prepare some data structures for storage, execution, etc.
3. register some handler functions for interrupts
4. when events happen, raise interrupts, then change some flags. The functions in the loop will check those flags and call the handler functions



#### Network data processing begins

> [NAPI](https://docs.kernel.org/networking/napi.html)
>
> NAPI is the event handling mechanism used by the Linux networking stack.
>
> In basic operation the device notifies the host about new events **via an interrupt**. The host then schedules a NAPI instance to process the events.
>
> 
>
> Driver API
>
> The two most important elements of NAPI are the struct `napi_struct` and the associated poll method
>
> - struct `napi_struct`: holds the state of the NAPI instance
> - the method: the driver-specific event handler. The method will typically free Tx packets that have been transmitted and process newly received packets.

![net_rx_action data processing diagram](https://cdn.buttercms.com/oarQVM3vQn6JLjih6tW6)

The `net_rx_action` function (called from the `ksoftirqd` kernel thread) will start to process any NAPI poll structures that have been added to the `poll_list` for the current CPU:

1. `net_rx_action` loop starts by checking the **NAPI poll list** for NAPI structures.
2. The `budget` and elapsed time [are checked](https://github.com/torvalds/linux/blob/v3.13/net/core/dev.c#L4300-L4309) to ensure that the softIRQ **will not monopolize CPU time**.
3. The registered [`poll` function is called](https://packagecloud.io/blog/monitoring-tuning-linux-networking-stack-receiving-data/#napi-poll-function-and-weight). In this case, the function `igb_poll` was registered by the `igb` driver.
4. The driver’s `poll` function [harvests packets from the ring buffer in RAM](https://packagecloud.io/blog/monitoring-tuning-linux-networking-stack-receiving-data/#napi-poll).



## Case study: Sending data

![图片](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwqUG6AE07rYfdAq7rDK2kZNC00DTfspwPMWXLjTYWB23MPL7Uz1TsDhgdsdOF920V4GqZwJyZs8gg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwpjubHIvu4Oia9L0YZzz7aRj2G1jzTCJt4RcZOvXSJ60jgpQWAHIcwjvTGL0uqGy4jT701LjK3QBgg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





## struct sk_buff

> All network-related queues and buffers in the kernel use a common data structure, [struct sk_buff](https://elixir.bootlin.com/linux/latest/source/include/linux/skbuff.h#L852). This is a large struct containing all the control information required for the packet (datagram, cell, whatever).
>
> - `struct sk_buff` is a doubly linked list
> - `struct sk_buff_head` includes a head and a tail pointer to `sk_buff` elements

Data structure hierarchy in Linux source code:

```c
// network layer representation of sockets
struct sock {
	struct sk_buff_head     sk_receive_queue;
	struct sk_buff_head     sk_write_queue;
    // ... 
};
```

`sock` has receive queue and write queue.

```c
struct sk_buff_head {
    struct sk_buff  *next;
	struct sk_buff  *prev;
	
	__u32           qlen;
	spinlock_t      lock;
}
```

`sk_buff_head` has head and tail pointer to the actual `sk_buff` elements. It defines a queue and each element's header is `sk_buff_head`. [7]

```c
struct sk_buff {
    struct sk_buff *next;
    struct sk_buff *prev;
    
    struct sock		*sk;	// record the socket assosciated with this SKB
   	
    // ...
    
	unsigned int    truesize;	// used for socket buffer accounting
    atomic_t		users;		// reference count SKB objects using the 'users' field
    // These four pointers provide the core management 
    // of the linear packet data area of an SKB.
    unsigned char		*head,
                        *data,
                        *tail,
                        *end;
}
```

### SKB data layout

![image-20240322160351581](D:\typora_img\image-20240322160351581.png)

- `skb = alloc_skb(len, GFP_KERNEL);`

![image-20240322160853512](D:\typora_img\image-20240322160853512.png)

- `skb_reserve(skb, header_len);`
    - for output packets, we reserve maximum bytes of header space
    - when setting up receive packets that an ethernet device will DMA into, we typically call `skb_reserve(skb, NET_IP_ALIGN)`
        - by default, `NET_IP_ALIGN` is `2` which means that after ethernet header, the protocol header will be aligned on at least a 4-byte boundary.

![image-20240322160914709](D:\typora_img\image-20240322160914709.png)

- push the UDP header to the front of the SKB
    - it will decrement the `skb->data` pointer
    - which means that it will take up the end space of head room

```c
struct inet_sock *inet = inet_sk(sk); // create IP socket
struct flowi *fl = &inet->cork.fl;
struct udphdr *uh;	// UDP header

skb->h.raw = skb_push(skb, sizeof(struct udphdr)); // reserve udphdr space
uh = skb->h.uh
uh->source = fl->fl_ip_sport;						// fill up udphdr attributes
uh->dest = fl->fl_ip_dport;
uh->len = htons(user_data_len);
uh->check = 0;
skb->csum = csum_partial((char *)uh,
             sizeof(struct udphdr), skb->csum);		// compute checksum
uh->check = csum_tcpudp_magic(fl->fl4_src, fl->fl4_dst,
                  user_data_len, IPPROTO_UDP, skb->csum);
if (uh->check == 0)
    uh->check = -1;
```

![image-20240322161643882](D:\typora_img\image-20240322161643882.png)

- push the IPv4 header to the front of the SKB
    - similarly, it will take up the end space of head room
    - now this packet is basically ready to be pushed out to the device

```c
struct rtable *rt = inet->cork.rt;
struct iphdr *iph;

skb->nh.raw = skb_push(skb, sizeof(struct iphdr));		// reserve iphdr space
iph = skb->nh.iph;
iph->version = 4;
iph->ihl = 5;
iph->tos = inet->tos;
iph->tot_len = htons(skb->len);
iph->frag_off = 0;
iph->id = htons(inet->id++);
iph->ttl = ip_select_ttl(inet, &rt->u.dst);
iph->protocol = sk->sk_protocol; /* IPPROTO_UDP in this case */
iph->saddr = rt->rt_src;
iph->daddr = rt->rt_dst;
ip_send_check(iph);

skb->priority = sk->sk_priority;
skb->dst = dst_clone(&rt->u.dst);
```

![image-20240322162057325](D:\typora_img\image-20240322162057325.png)



### TCP output engine in the skb point of view

![image-20240322164339237](D:\typora_img\image-20240322164339237.png)

- `TCP socket` maintains a doubly linked list of the pending queue (`sk_write_queeu`) in red rectangle
- `sk_send_head` points to the packet we have not sent out yet.
    - if receives ACK and window available, it will begin to send as many frames as we can.

![image-20240322164635239](D:\typora_img\image-20240322164635239.png)

- in TCP output engine, all paths lead to `tcp_transmit_skb()` in all cases:
    - for the first time, retransmit, a SYN packet in response to a `connect()` syscall



## iptables

![图片](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwq7DnzQ3XJ3erYYW9x8CIwhLsYXlx9wOhvvQ6lMVeE7e1LwRzWFdjwjicsaAkRLd5BqIsQLl8z1LxQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





## References

[1] [Illustrated Guide to Monitoring and Tuning the Linux Networking Stack: Receiving Data](https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/)

[2] [Monitoring and Tuning the Linux Networking Stack: Receiving Data](https://blog.packagecloud.io/illustrated-guide-monitoring-tuning-linux-networking-stack-receiving-data/)

[3] [Linux networking stack from the ground up, part 1](https://www.privateinternetaccess.com/blog/linux-networking-stack-from-the-ground-up-part-1/)

[4] [Direct memory access (DMA)](https://en.wikipedia.org/wiki/Direct_memory_access)

[5] https://wiki.linuxfoundation.org/networking/sk_buff

[6] http://vger.kernel.org/~davem/skb_data.html

[7] http://vger.kernel.org/~davem/skb.html

[8] http://vger.kernel.org/~davem/skb_list.html

[9] https://wiki.linuxfoundation.org/networking/kernel_flow

[10] https://mp.weixin.qq.com/s/wThfD9th9e_-YGHJJ3HXNQ
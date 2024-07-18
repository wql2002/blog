## Network interface

### Check whether a network interface is physical (device) or virtual (alias)?

`ifconfig`

We can simply use `ifconfig` (deprecated) to quickly the current network interfaces

``` shell
$ ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 141923345  bytes 28793753296 (28.7 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 141923345  bytes 28793753296 (28.7 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

some outputs:

- `inet`: protocol names (TCP/IP, default), others:
    - `inet6` (IPv6)



For more detailed information, like whether a network interface is physical (device) or virtual (alias), we can use:

```shell
$ ls -l /sys/class/net/
lrwxrwxrwx 1 root root 0 Mar 11 07:35 docker0 -> ../../devices/virtual/net/docker0
lrwxrwxrwx 1 root root 0 Mar 11 07:35 eno1 -> ../../devices/pci0000:00/0000:00:1c.2/0000:02:00.0/net/eno1
lrwxrwxrwx 1 root root 0 Mar 11 07:35 eno2 -> ../../devices/pci0000:00/0000:00:1c.2/0000:02:00.1/net/eno2
lrwxrwxrwx 1 root root 0 Mar 11 07:35 eno3 -> ../../devices/pci0000:00/0000:00:1c.3/0000:03:00.0/net/eno3
lrwxrwxrwx 1 root root 0 Mar 11 07:35 eno4 -> ../../devices/pci0000:00/0000:00:1c.3/0000:03:00.1/net/eno4
lrwxrwxrwx 1 root root 0 Mar 11 07:35 enp5s0 -> ../../devices/pci0000:00/0000:00:03.0/0000:05:00.0/net/enp5s0
lrwxrwxrwx 1 root root 0 Mar 11 07:35 enp5s0d1 -> ../../devices/pci0000:00/0000:00:03.0/0000:05:00.0/net/enp5s0d1
lrwxrwxrwx 1 root root 0 Mar 11 07:35 lo -> ../../devices/virtual/net/lo
lrwxrwxrwx 1 root root 0 Mar 11 07:35 virbr0 -> ../../devices/virtual/net/virbr0
lrwxrwxrwx 1 root root 0 Mar 11 07:35 virbr0-nic -> ../../devices/virtual/net/virbr0-nic
lrwxrwxrwx 1 root root 0 Mar 11 07:35 virbr2 -> ../../devices/virtual/net/virbr2
lrwxrwxrwx 1 root root 0 Mar 11 07:35 virbr2-nic -> ../../devices/virtual/net/virbr2-nic
lrwxrwxrwx 1 root root 0 Mar 11 07:35 virbr3 -> ../../devices/virtual/net/virbr3
lrwxrwxrwx 1 root root 0 Mar 11 07:35 virbr3-nic -> ../../devices/virtual/net/virbr3-nic
lrwxrwxrwx 1 root root 0 Mar 11 07:35 vnet0 -> ../../devices/virtual/net/vnet0
lrwxrwxrwx 1 root root 0 Mar 11 07:35 vnet1 -> ../../devices/virtual/net/vnet1
lrwxrwxrwx 1 root root 0 Mar 11 07:35 vnet2 -> ../../devices/virtual/net/vnet2
lrwxrwxrwx 1 root root 0 Mar 11 07:35 vnet3 -> ../../devices/virtual/net/vnet3
```

We can distinct this output by checking `../../devices/{class}/net/{device name}`:

- if `{class}` is virtual, it is a virtual device



### Check whether is TUN/TAP, Bridges, etc.

```shell
$ ip -details tuntap
virbr3-nic: tap persist
        Attached to processes:
virbr2-nic: tap persist
        Attached to processes:
```

- Physical devices have a `/sys/class/net/eth0/device` symlink
- Bridges have a `/sys/class/net/br0/bridge` directory
- TUN and TAP devices have a `/sys/class/net/tap0/tun_flags` file
- Bridges and loopback interfaces have `00:00:00:00:00:00` in `/sys/class/net/lo/address`



### Flow generating

> [Understanding host network stack overheads](https://dl.acm.org/doi/10.1145/3452296.3472888)
>
> - for generating long flows, we use a standard network benchmarking tool, **iPerf** [14], which transmits a flow from sender to receiver; 
> - for generating short flows, we use **netperf** [22] that supports pingpong style RPC workloads.


### iptables-nfqueue

https://asphaltt.github.io/post/iptables-nfqueue/

https://home.regit.org/netfilter-en/using-nfqueue-and-libnetfilter_queue/


## network namespace

> [comprehensive turoial](https://frfahim.github.io/posts/network-namespace/)



## congestion control

[one-article-fit-all](https://jasonmurray.org/posts/2021/tcpcca/)


[load and change cc](https://unix.stackexchange.com/questions/278215/add-tcp-congestion-control-variant-to-linux-ubuntu)

## proxy

[http/https proxy](https://tecadmin.net/setup-http-proxy-for-user-on-linux/#google_vignette)


---
title: OpenStack Neutron -- Linux 虚拟网络基础
date: 2019-05-14 11:59:26
updated: 2019-05-14 12:28:54
category: OpenStack
tags: [Neutron]
comments: true
header_image: /images/bing/BingWallPaper-2019-05-14.jpg
---

Neutron 在构建网络服务时，利用了许多 Linux 虚拟网络的功能（Linux 内核中的虚拟网络设备以及其他网络功能）。下面介绍一些与 Neutron 相关的 Linux 虚拟网络功能。
<!--more-->

## TAP/TUN

TAP/TUN 是 Linux 内核实现的一对虚拟网络设备。TAP 工作在二层，TUN 工作在三层。Linux 内核通过 TAP/TUN 设备向绑定该设备的用户空间程序发送数据，反之，用户空间程序也可以像操作硬件网络设备那样，通过 TAP/TUN 设备发送数据。

Linux 中设备的含义并不是指实际的物理硬件，而是一个类似于数据结构、内核模块或设备驱动。像 TAP/TUN 这样的设备，其数据结构如下：

```c
struct tun_struct {
    char name[8];				// 设备名
    unsigned long flags;			// 区分 TAP 和 TUN 设备
    struct fasync_struct *fasync;		// 文件异步通知结构
    wait_queue_head_t read_wait;		// 文件等待队列
    struct net_device dev;			// Linux 抽象网络设备结构
    struct sk_buff_head txq;			// 网络缓冲区队列
    struct net_device_status stats;		// 网卡状态信息结构
};
```

TAP 与 TUN 的定义相同，通过 `flags` 来进行区分。但是从其背后所承载的功能而言，两者有着较大的区别：TAP 位于网络 OSI 模型的第二层（数据链路层），TUN 位于第三层（网络层）。

TAP 从功能定位上来讲，位于数据链路层，数据链路层的主要协议有：

1. 点对点协议（Point-to-Point Protocol）
2. 以太网（Ethernet）
3. 高级数据链路协议（High-Level Data Link Protocol）
4. 帧中继（Frame Relay）
5. 异步传输模式（Asynchronous Transfer Mode）

但是 TAP 只与以太网（Ethernet）协议对于。所以，TAP 有时也称为“虚拟以太设备”。

想要使用 Linux 命令行（基于 CentOS7 x86_64）操作一个 TAP，首先需要 Linux tun 模块：

```bash
# 如果输入 Linux 命令 modinfo tun，有如下输出，则说明有 tun 模块
modinfo tun
filename:	/lib/modules/3.10.0-862.14.4.el7.x86_64/kernel/drivers/net/tun.ko.xz
alias:		devname:net/tun
alias:		char-major-10-200
......
```

当 Linux 版本具有 tun 模块时，还需要查看其是否已经加载：

```bash
lsmod | grep tun
tun 31665 1
```

如果已经加载，则会出现上述的”tun ***“那一行。如果没有加载，则可使用如下命令进行加载：

```bash
modprob tun
```

当确认 Linux 加载 tun 模块之后，还需要确认 Linux 是否操作 TAP/TUN 的命令行工具 tunctl：

```bash
tunctl help
```

如果 Linux 有输出，则说明已有命令行工具，否则表示当前 Linux 系统并没有安装 tunctl，可以通过如下命令进行安装：

```bash
yum install tunctl
```

> 若安装时出现 no package tunctl avaliable 时，需要手动添加安装源：
>
> 1. 添加安装源配置文件 /etc/yun.repos.d/nux-misc.repo
>
> {% codeblock lang:ini %}
> [nux-misc]
> name=Nux Misc
> baseurl=http://li.nux.ro/download/nux/misc/el7/x86_64
> enabled=0
> gpgcheck=1
> gpgkey=http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
> {% endcodeblock %}
>
> 2. 重新执行安装命令：
> {% codeblock lang:bash %}
> yum --enablerepo=nux-misc install tunctl
> {% endcodeblock %}
> 
> ref: [CentOS 7 安装tunctl](<https://blog.csdn.net/lopng/article/details/72821438>)

具备了 tun 和 tunctl 后，就可以创建一个 TAP 设备了：

```bash
tunctl -t tap_test
Set 'tap_test' persistent and owned by uid 0
```

可以通过如下命令来查看所创建的 TAP（名字为 `tap_test`）：

```bash
ip link list
```

也可以通过如下命令查看：

```bash
ifconfig -a
```

通过 tunctl 创建的 `tap_test` 还未绑定 IP 地址，可以通过如下命令进行绑定：

```bash
# 使用 ip addr 命令绑定 IP 地址
ip addr add local 192.168.100.1/24 dev tap_test
# 或者使用 ifconfig 命令绑定 IP 地址
ifconfig tap_test 192.168.100.1/24
```

使用 `ifconfig -a` 命令再次查看，可以发现 `tap_test` 已绑定了所设置的 IP 地址。

## namespace

namespace 是 Linux 虚拟网络的一个重要概念。传统 Linux 的许多资源是全局的，比如进程 ID 资源。而 namespace 的目的就是将这些资源进行隔离。Linux 可以在一个 Host 内创建许多 namespace，于是那些原本是 Linux 全局的资源，就变成了 namespace 范围内的”全局“资源，而且不同的 namespace 的资源互不可见、彼此透明（感觉类似于 C/C++ 的 namespace）。

Linux 内核对哪些资源进行了隔离可以从 include/linux/nsproxy.h 中看出：

```c
// nsproxy.h
struct nsproxy {
    atomic_t count;
    struct uts_namepsace *uts_ns;
    struct ipc_namespace *ipc_ns;
    struct mnt_namespace *mnt_ns;
    struct pid_namespace *pid_ns;
    struct user_namepsace *user_ns;
    struct net *net_ns;
};
```

以上 6 个资源就是 Linux namespace 所隔离的资源，其含义如下表：

|  资源   | 含义                                                         |
| :-----: | :----------------------------------------------------------- |
| uts_ns  | UTS 为 Unix Timesharing System 的简称，包含内存名称、版本、底层体系结构等信息 |
| ipc_ns  | 所有与进程间通信（IPC）有关的信息                            |
| mnt_ns  | 当前装载的文件系统                                           |
| pid_ns  | 有关进程 ID 的信息                                           |
| user_ns | 资源配额的信息                                               |
| net_ns  | 网络信息                                                     |

单纯从网络角度来看，一个 namespace 提供了一份独立的网络协议栈（网络设备接口、IPv4、IPv6、IP 路由、防火墙规则、sockets 等）。一个设备（Linux Device）只能位于一个 namespace 中，不同 namespace 中的设备可以利用 veth pair 进行桥接。

Linux 操作 namespace 的命令为 `ip netns`，可以通过如下命令查看其功能：

```bash
ip netns help
Usage:	ip netns list
	ip netns add NAME
	ip netns set NAME NETNSID
	ip [-all] netns delete [NAME]
	ip netns identify [PID]
	ip netns pids NAME
	ip [-all] netns exec [name] cmd ...
	ip netns monitor
	ip netns list-id
```

首先可以创建一个 namespace ：

```bash
# 首先查看当前的 namespace 列表
ip netns list
# 由于当前没有 namespace，所以命令行没有任何返回
# 创建一个 namespace，命名为 ns_test
ip netns add ns_test
#在此查看当前 namespace 列表，可以发现刚刚创建的 namespace: ns_test
ip netns list
ns_test # 这个是 ip netns list 的返回值
```

当我们创建一个 namespace 后，可以将原来创建的虚拟设备 tap_test 迁移到这个 namespace 中去，命令如下：

```bash
ip link set tap_test netns ns_test
```

这个时候，在原来的 host/vm 里再执行 `ip link list` 命令，就会发现设备 `tap_test` 消失了，因为我们已经将其迁移到 `ns_test` 中去了。

通过一下命令可以查看或者操作 namespace 里的设备：

```bash
ip [-all] netns exec [NAME] cmd ...		# cmd 为想要操作的命令
```

比如要管理 `ns_test` 里面的设备，执行命令如下：

1. 在 `ns_test` 里执行 `ip link list`

```bash
ip netns exec ns_test ip link list
```

2. 在 ` ns_test` 里执行 `ifconfig -a`

```bash
ip netns exec ns_test ifconfig -a
```

3. 绑定 IP 地址

```bash
ip netns exec ns_test ifconfig tap_test 192.168.50.1/24 up
```

4. 查看 IP 地址

```bash
ip netns exec ns_test ifconfig -a
```

## veth pair

veth pair 不是一个设备，而是一对设备，用以连接两个虚拟以太端口。操作 veth pair，需要结合 namespace，不然就没有意义。下面举一个例子，两个 namespace ns1/ns2 中各有一组 tap 组成 veth pair，两者的 IP 地址分别为 192.168.50.1 和 192.168.50.2，两个 IP 进行互 ping 测试，ping 通表示测试通过。

```bash
# 1. 创建 veth pair
ip link add tap1 type veth peer name tap2
# 2. 创建 namespace: ns1、ns2
ip netns add ns1
ip netns add ns2
# 3. 把两个 tap 分别迁移到对应的 namespace 中
ip link set tap1 netns ns1
ip link set tap2 netns ns2
# 4. 分别给两个 tap 绑定 IP 地址
ip netns exec ns1 ip addr add local 192.168.50.1/24 dev tap1
ip netns exec ns2 ip addr add local 192.168.50.2/24 dev tap2
# 5. 将两个 tap 设置为 up
ip netns exec ns1 ifconfig tap1 up
ip netns exec ns2 ifconfig tap2 up
# 6. ping
ip netns exec ns2 ping 192.168.50.1		# ping ns1 中的 tap
...		# 输出结果
ip netns exec ns1 ping 192.168.50.2		# ping ns2 中的 tap
...		# 输出结果
```

上述用例给出了 veth pair 连接两个namespace 的方法，veth pair 只有一对 tap，如果需要实现两个 以上的namespace 互通，则 veth pair 无法满足需求。

## Bridge

Bridge/Switch 可以实现两个以上 namespace 之间的互通。在 Linux 的语境里，Bridge（网桥）与 Switch（交换机）是一个概念，这里也不对两者进行区分。Linux 中实现 Bridge 功能的是 brctl 模块。在命令行中输入 `brctl` 可以查看是否正确安装模块：

```bash
brctl
Usage: brctl [Commands]
commands:
	addbr			<bridge>			add bridge
	delbr			<bridge>			delete bridge
	addif			<bridge> <device>		add interface to bridge
	delif			<bridge> <device>		delete interface from bridge
	hairpin			<bridge> <port> {on|off}	turn hairpin on/off
	setageing		<bridge> <time>			set ageing time
	setbridgeprio	        <bridge> <prio>			set bridge priority
	setfd			<bridge> <time>			set bridge forward delay
	sethello		<bridge> <time>			set hello time
	setmaxage		<bridge> <time>			set max message age
	setpathcost		<bridge> <prot> <cost>		set path cost
	setportprio		<bridge> <prot> <prio>		set port priority
	show			[ <bridge> ]			show a list of bridges
	showmacs		<bridge>			show a list of mac addrs
	showstp			<bridge>			show bridge stp info
	stp			<bridge> {on|off}		turn stp on/off
```

若未安装，可通过如下命令进行安装：

```bash
yum install bridge-utils
```

接下来也通过一个例子来说明 Bridge 的基本用法，同时也涵盖了之前所述的几个概念：tap、namespace、veth pair。样例中有 4 个 namespace，每个 namespace 都有一个 tap 与交换机上一个 tap 口组成 veth pair。这样 4 个 namespace 就通过 veth pair 及 Bridge 互联起来。

```bash
# 1. 创建 veth pair
ip link add tap1 type veth peer name tap1_peer
ip link add tap2 type veth peer name tap2_peer
ip link add tap3 type veth peer name tap3_peer
ip link add tap4 type veth peer name tap4_peer
# 2. 创建 namespace
ip netns add ns1
ip netns add ns2
ip netns add ns3
ip netns add ns4
# 3. 把 tap 迁移到相应的 namespace 中
ip link set tap1 netns ns1
ip link set tap2 netns ns2
ip link set tap3 netns ns3
ip link set tap4 netns ns4
# 4. 创建 Bridge
brctl addbr br1
# 5. 把相应的 tap 添加到 Bridge 中
brctl addif br1 tap1_peer
brctl addif br1 tap2_peer
brctl addif br1 tap3_peer
brctl addif br1 tap4_peer
# 6. 配置相应 tap 的 IP 地址
ip netns exec ns1 ip addr add local 192.168.50.1/24 dev tap1
ip netns exec ns2 ip addr add local 192.168.50.2/24 dev tap2
ip netns exec ns3 ip addr add local 192.168.50.3/24 dev tap3
ip netns exec ns4 ip addr add local 192.168.50.4/24 dev tap4
# 7. 将 Bridge 及所有 tap 状态设置为 up
ip link set br1 up
ip link set tap1_peer up
ip link set tap2_peer up
ip link set tap3_peer up
ip link set tap4_peer up
ip netns exec ns1 ip link set tap1 up
ip netns exec ns2 ip link set tap2 up
ip netns exec ns3 ip link set tap3 up
ip netns exec ns4 ip link set tap4 up
# 8. 互 ping 测试
ip netns exec ns1 ping 192.168.50.2
ip netns exec ns1 ping 192.168.50.3
ip netns exec ns1 ping 192.168.50.4
...
ip netns exec ns4 ping 192.168.50.1
ip netns exec ns4 ping 192.168.50.2
ip netns exec ns4 ping 192.168.50.3
```

若能够互相 ping 通，则表示测试通过。

## Router

Linux 中 Router 能够用于不同网段之间的互通。通过如下命令可以查看系统是否开启路由转发功能：

```bash
less /proc/sys/net/ipv4/if_forward
```

如果返回的结果是 `0`，则表示未开启，若为 `1`，则表示已开启路由转发功能。可以修改配置文件 “/etc/systcl.conf”，将 `net.ipv4.ip_forward` 的值修改为 `1` 来开启该功能。

下面举个例子，用于说明 Router 的作用。分别设计 ns1/tap1 和 ns2/tap2，且它们不在同一个网段中，中间需要经过一个路由转发才能互通。

```bash
# 1. 创建 veth pair
ip link add tap1 type veth peer name tap1_peer
ip link add tap2 type veth peer name tap2_peer
# 2. 创建 namespace
ip netns add ns1
ip netns add ns2
# 3. 将 tap 迁移到 namespace
ip link set tap1 netns ns1
ip link set tap2 netns ns2
# 4. 配置 tap IP 地址
ip addr add local 192.168.100.1/24 dev tap1_peer
ip addr add local 192.168.200.1/24 dev tap2_peer
ip netns exec ns1 ip addr add local 192.168.100.2/24 dev tap1
ip netns exec ns2 ip addr add local 192.168.200.2/24 dev tap2
# 5. 将 tap 设置为 up
ip link set tap1_peer up
ip link set tap2_peer up
ip netns exec ns1 ip link set tap1 up
ip netns exec ns2 ip link set tap2 up
# 6. 为 ns1、ns2 添加静态路由，分别到达对方的网段
ip netns exec ns1 route add -net 192.168.200.0 netmask 255.255.255.0 gw 192.168.100.1
ip netns exec ns2 route add -net 192.168.100.0 netmask 255.255.255.0 gw 192.168.200.1
# 7. 互 ping 测试
ip netns exec ns1 ping 192.168.200.2
...
ip netns exec ns2 ping 192.168.100.2
...
```

能够互相 ping 通表示测试通过。

添加了静态路由信息后，可以使用如下命令查看 namespace 的路由表：

```bash
ip netns exec ns1 route -nee
...
ip netns exec ns1 route -nee
...
```

上述命令可以分别查看 ns1 和 ns2 的路由表信息。

## tun

tun 是一个网络层（IP）的点对点设备，它启用了 IP 层隧道功能。Linux 原生支持的三层隧道，可以通过 `ip tunnle help` 查看。

Linux 一共原生支持 5 种三层隧道（tunnel），如表所示：

|  隧道  | 简述                                                         |
| :----: | :----------------------------------------------------------- |
|  ipip  | IP in IP，在 IPv4 报文的基础上再封装一个 IPv4 报文头，属于 IPv4 in IPv4 |
|  gre   | 通用路由封装（Generic Routing Encapsulation），定义了在任意一种网络层协议上封装任意一个其他网络层协议的协议，属于 IPv4/IPv6 over IPv4 |
|  sit   | 与 ipip 类似，用一个IPv4 的报文头封装 IPv6 的报文，属于 IPv6 over IPv4 |
| isatap | 站内自动隧道寻址协议，一般用于 IPv4 网络中的 IPv6/IPv4 节点间的通信 |
|  vti   | 全称是 Virtual Tunnel Interface，为 IPsec 隧道提供了一个可路由的接口类型 |

下面给出 tun 的具体用例来进行说明，以 ipip tunnel为例进行配置。在上一节 Router 的基础上，分别在 ns1 和 ns2 中添加 tun1 和 tun2，tun1 和 tun2 不互通，且与 tap1、tap2也没有关系。

```bash
# 1. 加载 ipip 模块，可通过 lsmod | grep ip 命令查看是否加载
modprobe ipip
# 2. 在 ns1 上创建 tun1 和 ipip tunnel
ip netns exec ns1 ip tunnel add tun1 mode ipip remote 192.168.200.2 local 192.168.100.2 ttl 255		# 创建 tun1，模式为 ipip，分别配置远端地址和近端（本地）地址以及 ttl
ip netns exec ns1 ip link set tun1 up		# 启动 tun1
ip netns exec ns1 ip addr add 192.168.50.10 peer 192.168.60.10 dev tun1		# 为 tun1 添加 ipip 隧道的内层 IP 地址，并设置对端的 ipip 隧道内层 IP 地址
# 3. 在 ns2 上创建 tun2 和 ipip tunnel
ip netns exec ns2 ip tunnel add tun2 mode ipip remote 192.168.100.2 local 192.168.200.2 ttl 255
ip netns exec ns2 ip link set tun2 up
ip netns exec ns2 ip addr add 192.168.60.10 peer 192.168.50.10 dev tun2
# 4. 互 ping 测试 （测试时遇到了问题，在 CentOS 7 系统下，按照上述命令无法实现两个 tun ping 通，但是前面的 Router 是可以的，原因未知。）
ip netns exec ns1 ping 192.168.60.10
...
ip netns exec ns2 ping 192.168.50.10
...
```

> 将上述命令中的 `ipip` 改为 `gre`，其余保持不变，即可创建一个 gre 隧道的 tun 设备对。

查看 ns1 的路由表，发现已添加了一个直连路由条目，从 tun1 可以直接到达 192.168.60.10，ns2 亦然。

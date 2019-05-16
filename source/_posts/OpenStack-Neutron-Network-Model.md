---
title: OpenStack Neutron -- Neutron 的网络实现模型
date: 2019-05-16 14:26:17
updated: 2019-05-16 14:26:17
category: OpenStack
tags: [Neutron]
comments: true
header_image: /images/bing/BingWallPaper-2019-05-16.jpg
---
承载 Neutron 抽象的网络资源模型的方案，称之为 Neutron 的网络实现模型。Neutron 仅为管理系统（或者说是控制系统），它本身并不能实现任何网络功能，实现网络功能的是各种网元。
<!--more-->

## 计算节点

一个基于 OpenStack 的云系统会有很多计算节点，一个计算节点就是一个 Host。一个计算节点里包含多个 VM。计算节点的网络模型主要服务于二层网络，用于构建各种类型的二层网络。而二层网络通信需要 Bridge/Switch（这里可以将 Bridge/Switch 理解为同一个概念）。只考虑网络层面，计算节点可分为 2 层：用户网络层、本地网络层。

### 用户网络层

用户网络层（User Network），指的是 OpenStack 的用户创建的网络，也即外部网络，这个外部网络是相对于 Host 内部网络而言的。用户网络层对应的 Bridge 是 br-ethx （对应Flat、VLAN 等非隧道型二层网络）或者 br-tun （对应 VXLAN、GRE 等隧道型二层网络），其实现载体一般来说是 OVS （Open vSwitch）。用户网络层实现的是用户网络和本地网络之间的相互转换。**用户网络层是对本地网络层的一个屏蔽，即不管用户网络采用什么技术（如 VXLAN、GRE 等），本地网络永远感知的仅仅是一个技术：VLAN**。

### 本地网络层

本地网络指的是 Host 内部的本地网络。本地网络只要感知一种技术：VLAN。本地网络还可以分为两层：安全层、Bridge 层。

#### qbr

qbr 的实现载体是 Linux Bridge，它仅仅负责安全相关设置，所以称之为安全层。

#### br-int

br-int 的实现载体一般是 OVS，她负责内部交换，所以称之为 Bridge 层。**Bridge 层是对 VM 层的一个屏蔽。从 VM 发出的 Untag 报文，被 Bridge 层转换为 Tag 报文转发到 br-ethx/br-tun；从 br-ethx/br-tun 转发到 br-int 的 Tag 报文，被 br-int 剥去 Tag，变成 Untag 报文，然后再转发给 VM**。

位于同一个 Host 的本地网络中的不同 VM 之间的通信，他们经过本地层网络（即经过 br-int）即可完成，无需通再往上通过用户网络层。

## 网络节点

在计算节点中，同属一个二层网络的 VM 可以自由进行二层通信。若要访问二层网络之外的网络，则需要网络节点。Neutron 除了在网络节点部署 Router （此路由器为虚拟路由器，利用了 Linux 内核功能）之外，还部署了 DHCP Server 等服务。从网络视角看，网络节点可分为 4 层：用户网络层、本地网络层、网络服务层和外部网络层。前两层与计算节点几乎相同。

### 网络服务层

网络服务层为计算节点的 VM 提供网络服务，典型的服务有 DHCP Service 和 Router Service。关于 DHCP 有如下概念：

1. Neutron 的 DHCP Service，采用的是 dnsmasq 进程（轻量级服务进程，可以提供 dns、dhcp、tftp等服务）
2. 一个网络一个 DHCP Service
3. 由于存在多个 DHCP Service （多个 dnsmasq 进程），Neutron 采用的是 namespace 方法做隔离，即一个 DHCP Service 运行在一个 namespace 中。

网络服务层中的 Router 本质上是 Linux 内核模块。Router 做路由转发外，还提供 SNAT/DNAT 功能。为了达到隔离的目的，每个 Router 运行在一个 namespace 中。准确地说，Neutron 创建了 namespace，并且在 namespace 中开启了路由转发功能。

> 1. SNAT：Source Network Address Translation，源地址路由转换；DNAT：Destination Network Address Translation，目的地址路由转换
> 2. OpenStack Juno 版本引入了 DVR 特性，DVR 部署在计算节点上。计算节点访问 Internet，不必经过网络节点，直接从计算节点的 DVR 即可访问

### 外部服务层

外部服务层包含 br-ex 和 Router。Router 是与外部网络联通的主体，而 br-ex 则是将 Router 对接到网络节点的物理网口。br-ex 相当于一个 Hub，而实际上 br-ex 是一个 Bridge，一般选用 OVS。

## 控制节点

计算机节点与网络节点承担 OpenStack 中网络构建的任务，实现网络功能的是两个节点中的各个 Bridge、DHCP Service 和 Router 等虚拟网元。控制节点并不实现具体网络功能，只是对各种虚拟网元进行管理和控制。控制节点部署着 OpenStack 的各种进程，对于 Neutron 来说，它的进程名为 neutron-server。

Neutron 中的控制功能不仅仅体现于一个控制节点，还包含计算节点和网络节点中的各种 Agent。控制节点中的 Neutron 进程只是 Neutron 控制系统的一部分。

控制节点的 Neutron 进程通过 RESTful 或者 CLI （Command Line Interface，命令行）接口接收外部请求，通过 RPC 与 Agent 进行交互。Neutron 进程与各个 Agent 进程共同完成管理控制任务。

## 总结

从部署角度来说，Neutron 分为三类节点：控制节点、网络节点和计算节点。**网络节点和计算节点为 VM 构建了具体网络，控制节点则对这些网络进行管理。**

控制节点的 Neutron 进程与网络节点、计算节点的各个 Agent 进程互相配合，对内完成对网络节点、控制节点**？**（书上是控制节点，但是感觉应该是计算节点）中虚拟网元的配置管理，对外提供 RESTful 等服务接口。Neutron 进程与 Agent 进程之间的通信协议是 RPC（Remote Procedure Call，远程过程调用）。

计算节点中的各个 Bridge 构建了 Neutron 中的 Local、Flat、VLAN、GRE、VXLAN、Geneve 6 种二层网络。br-ethx 与 br-tun 对外构建用户网络，内对则为 br-int 屏蔽用户网络的各种差异，将不同类型的用户网络转换为 VLAN 网络。br-int 在 Host 内部为各个 VM 构建了一个本地网络。qbr 为 br-int （也是为各个 VM）提供辅助的安全功能。

网络节点为 Neutron 提供了其他网络服务，比如 DHCP Service 等。网络节点中的 Router，则提供了三层服务，除了提供普通的路由转发功能外，还提供了 SNAT/DNAT 等功能。

Neutron 的三类节点互相配合，共同完成了 Neutron 对外宣称的使命：NaaS（Network as a Service，网络即服务）。
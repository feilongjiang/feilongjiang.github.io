---
title: OpenStack Neutron -- Neutron 的资源模型
date: 2019-05-22 15:06:13
updated: 2019-05-22 15:06:13
category: OpenStack
tags: [Neutron]
comments: true
header_image: /images/bing/BingWallPaper-2019-05-22.jpg
---

Neutron 把它管理的对象统统称为资源，表面上看起来这些资源的名称与传统电信领域的命名（比如 Network、Subnet）等完全一致，但是由于管理的范围（Neutron 的管理范围主要还是 DC 内）和管理对象的特点（Neutron 主要管理 Host 内部的虚拟网元）等原因，Neutron 的资源模型在传统电信的理论基础上又有其特点。
<!--more-->

## Neutron 资源的租户隔离

租户隔离指的是将不同的用户隔离开来，用户对其他用户无感知。租户隔离有三种含义：管理面的隔离、数据面的隔离和故障面的隔离。

### 管理面的隔离

管里面的隔离，指的是”管理权限“的隔离，每个租户只知道他自己的网络，对其他网络毫无感觉。

### 数据面的隔离

数据面的隔离，指的是数据转发的隔离。不同租户之间的网络，一般来说是不能互通的。从管理权限的角度，一个租户感知不到另外一个租户的网络。不仅无法感知，而且还可以”重复“。从这个角度来说，数据面的隔离是为了复用。

### 故障面的隔离

故障隔离，主要是管里面的故障、数据面的故障要做到租户隔离，而物理资源层面，无法真正做到故障隔离。Neutron 在故障面的目标不是租户隔离，而是容错——尽量保证不受故障影响。

## Network

Network 是 Neutron 的一个二层网络的资源模型，它支持的网络类型有：Local、Flat、VLAN、VXLAN、GRE、Geneve 等。其中，Local 仅仅是一个主机内的网络类型，只会用于测试，不会用于生产环境。VXLAN、GRE、Geneve 属于隧道型网络，Flat 和 VLAN 属于非隧道型网络。一般来说，这些网络内的虚拟机之间的二层通信并不需要用到路由器。

### 运营商网络和租户网络

由租户创建并管理的网络，Neutron 称之为租户网络。有时候，Neutron 还需要创建一个网络来映射外部网络，这个网络称为运营商网络（Provider Network）。

运营商网络与租户网络，从模型角度来讲，都是 Neutron 的资源模型 Network。两者的区别如下：

1. 管理的权限和角色不同。租户创建的网络，就是租户网络，而运营商管理员创建的网络，就是运营商网络
2. 创建网络时，传入的参数不同。创建运营商网络时，需要传入 provider:network_type、provider:physical_network、provider:segmentation_id 三个参数，而创建租户网络时，没有办法传入这三个参数，它们是由 OpenStack 在管理员配置的范围内自动分配的

Neutron 创建了一个网络，如果这个网络只是为了映射（匹配）另外一个网络，而且这个被映射的网络不在 Neutron 的管理范围内，这样的场景就是运营商网络的一般化使用场景。Neutron 创建的这个网络，也称为运营商网络。运营商网络可以认为是运营商的某个物理网络在 OpenStack （Neutron）上的延伸。

### 物理网络

一般来说，物理网络可以理解为运营商网络需要匹配的那个网络。物理网络的实际意义如下：

1. 非隧道型网络角度
    - 对于运营商网络，物理网络（provider:physical_network）就是这个运营商网络所要匹配的外部网络的名称，这个含义是为了方便阅读和理解
    - 无论是对于运营商网络还是租户网络，物理网络都意味着 br-ehtx 的选择（背后是主机网卡的选择），这个含义是 Neutron 所需要的，因此创建运营商网络时，需要直接传入这个参数，创建租户网络时，需要间接传入这个参数
2. 隧道型网络角度
    - 对于运营商网络，物理网络（provider:physical_network）就是这个运营商网络所要匹配的外部网络名称，之歌含义只为了方便阅读和理解，对 Neutron 无意义

## Trunk Networking

Trunk Networking 是 OpenStack Newton 版本所增加的特性，目的是为了支持 VLAN aware VM。一般情况下，VM 发送和接收的报文都是不带 VLAN Tag 的，不过有时候，VM 也期望能够发送和接收带有 VLAN Tag 的报文，这种情形即可称为 VLAN aware VM。

### Bridge 的 VLAN 接口模式

一个 Bridge 可以抽象为两大部分：交换模块（基于 VLAN ID 做报文交换）和接口。

Bridge 接口关于 VLAN ID 的处理方式有三种模式：Access、Trunk 和 Hybrid。

#### Access 接口模式

Access 模式，在报文入接口，对于 Tag 报文，直接丢弃，对于 Untag 报文，则打上 Default VID Tag 后送入交换模块（进行 VLAN 交换）。在报文出接口，先将报文去除 Tag，然后再从接口转发。

#### Trunk 接口模式

Trunk 模式，首先要配置允许进入接口的 VLAN ID 列表（范围）。在列表范围内的 VLAN ID 可以进入端口，其他的则不允许进入。

#### Hybrid 接口模式

Hybrid 模式，在 Trunk 模式的基础上又多了一部分内容。Trunk 模式，在报文出接口时，如果 VLAN ID 等于 Default ID，那么 VLAN Tag 会去除。而 Hybrid 模式，允许配置哪些 VLAN ID 的报文，在出接口时，需要去除 VLAN Tag，比如配置 VLAN ID 在 40 ~ 50 这个范围内的报文，当其出接口时，需要去除 VLAN Tag。

> 上述描述中的几个名词解释：
>
> - Tag 报文：指的是报文中有 VLAN ID，简称 Tag
> - Untag 报文：指的是报文中没有 VLAN ID，简称 Untag
> - Default ID：端口默认 VLAN ID。Bridge 端口的默认 VLAN ID 为 “1”。这个默认值可以被修改

### VLAN aware VM 与 Trunk Networking

VM 能够发送和接收带有 VLAN Tag 的报文，这种情形称作 VLAN aware VM。在没有引入 Trunk Networking 特性之前，Neutron 的模型设计中有这样的约束：一个 Port 只能属于一个 Network。假设一个 VM 只有一个 Port，如果想让 VM 具备 VLAN aware 特性，就意味着这个 Port 必须要属于多个 Network，这与 Neutron 的约束是矛盾的。这里所涉及的 Port，可以简单理解为 VM 的虚拟网口（virtual Network Interface Card，vNIC）。为了解决这个问题，Neutron 提出了 Trunk Networking 方案，主要解决实现 VLAN aware 的以下问题：

1. VM aware 的 VLAN ID，在 Host 内部不能冲突
2. VM aware 的 VLAN ID，不需要在 Host 之间的物理网络透传（不能要求物理网络透传该 VLAN ID，因为物理网络很可能不在 Neutron 的管理范围内）
3. 不要打破原来的 Network、Port 模型，否则会引发 Neutron 的源代码大量修改

#### Trunk Networking 的实现模型

为了解决 VM aware 的 VLAN ID 与 Host 内部的 VLAN ID 冲突的问题， Neutron 引入了一个 Trunk Bridge。Trunk Bridge 仍然是一个普通的 Bridge，只不过接口模式有所不同。Neutron 利用 br-trunk 做 VLAN ID 的转换，就可以解决上述的 VLAN ID 冲突的问题。

#### Trunk Networking 的资源模型

Neutron 最终采取的模型不是“一个 Port 对应多个 Network”，而是增加了一个模型，名为 trunk，这个模型里最核心的就是两个字段：Parent Port 和 Sub Port List，如下表所示：

|   名称    |  类型  |          描述          |
| :-------: | :----: | :--------------------: |
|  port_id  | stirng |       父端口 ID        |
| sub_ports | array  | Trunk 关联的子端口列表 |

这个模型在不同的视角有不同的解读，并解决了对应的问题。

从 VM 视角，它仍然相当于一个端口对接多个网络，因为它的端口可以发送带 VLAN ID 的报文，每个 VLAN ID 对应一个网络。这解决了 VM 的端口不能太多的问题。

从 Neutron 模型的角度，Trunk 模型利用一个 Parent Port （对应字段为 port_id）和多个 Sub Port（对应字段为 sub_ports）与不同的 Network 相对应。Parent Port 与 Sub Port 对应的都是 Port 模型，它们与 Network 的关系仍是一个 Port 对应一个 Network。

Trunk Networking 方案没有修改原来的实现模型和资源模型，而是在原来的实现模型中引入了 Trunk Bridge。在原来的资源模型中引入了 Trunk 资源模型，对原来的代码修改较少，就支持了 VLAN aware VM 特性。

## Subnet

Subnet（子网）在一般概念中，有两个基本含义：

1. 这个子网的网段（CIDR）和 IP 版本
2. 这个子网的路由信息（含默认路由）

Neutron 中，Subnet 模型除了标识 CIDR、IP version 这样的纯逻辑资源外，为了解决 VM 的 IP 地址分配和 DNS 设置问题，还蕴含了管理功能，这些管理功能又称为 IP 的核心服务。

### IP 核心网络服务

IP 核心网络服务（IP Core Network Services），又称 DDI 服务，包括：DNS、DHCP 和 IPAM。这三个服务是所有 IP 网络及应用系统得以顺利运行的基础。

## Port

如果说 Network 是 Neutron 模型中的“根”，那么 Port 则是 Neutron 模型中的“灵魂”，尤其是对于三层转发来说。因为无论 Neutron 的模型怎么设计，它的三层转发总归是绕不开 IP 地址，而承载 IP 地址的就是 Port。

Port 是一个逻辑模型，但是也可以理解为其代表一个虚拟网口。所以一个 VM 需要绑定 Port，一个 Router 也需要绑定 Port。作为一个虚拟网口，Port 具备两个基本属性：IP 地址和 MAC 地址。一个 Port 可以有多个 IP 地址，但在一般情况下，一个 Port 只有一个 MAC 地址。但是 Port 模型中有一个 allowed_address_pairs字段，允许多个绑定多个 MAC 地址，其典型的应用场景为 Antispoofing（一种识别和删除有错误源地址的数据包技术）。

Network、Subnet、Port 三者的关系如下：

一个 Network 可以有多个 Subnet，一个 Subnet 只能归属一个 Network。同时一个  Network 可以有多个 Port，而一个 Port 可以与其所在的 Network 中的所有 Subnet 相关联。当然，一个 Subnet 也可以有多个 Port。

## Router

如果说 Port 是 Neutron 模型的“灵魂”，那么 Router 就是 Neutron 模型的 “发动机”，它承担着路由转发的功能。Router 可以简单地抽象为三部分：端口、路由表、路由协议处理单元。Router 最关键的两个概念就是端口和路由表。

Neutron 的 Router 模型中，蕴含三种路由：直连路由、默认静态路由和静态路由。前两种路由不需要显式地增加路由表项（routes 的 [destination (string), nexthop (string)]），也不会体现在路由表（routes）中。

路由表中的路由也是静态路由，它与默认静态路由一样，都是通往外部网络（Neutron 管理范围外的网络）。静态路由中的外部网络，一般指的是私网，而默认静态路由中的外部网络，一般指的是公网。

#### Floating IP

Floating IP 首先是一个 SNAT/DNAT 转换规则：floating_ip_address（外网/公网 IP）与 fixed_ip_address （内网/私网 IP）互相转换。然后，从实现角度来讲，他才是绑定到一个 Router（router_id）的端口（port_id）上，以让报文在进出这个端口时，Router 能对其做 SNAT/DNAT 转换。

## BGP VPN

Neutron 的 BGP VPN，指的是 MP-BGP MPLS L3VPN 或者是 E-VPN （E-VPN 的控制协议也是 MP-BGP）。

Neutron 中 BGP VPN 的实现模型主要是关于 PE/CE 的实现方法，有如下几种：

1. PE/CE 都位于计算节点内，VM 扮演 CE 的角色，PE 则有 br-ehtx/br-tun（OVS）承担
2. PE/CE 都位于网络节点内， Router 承担 CE 角色，而由 L3 Agent 承担 PE 角色
3. CE 位于网络节点内，PE 位于外部路由器，Router 承担 CE 角色，而由外部路由器承担 PE 角色

> 名词解释：
>
> PE：Provider Edge，运营商边缘路由器
>
> CE：Customer Edge，用户边缘路由器
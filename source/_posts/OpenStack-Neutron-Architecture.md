---
title: OpenStack Neutron -- Neutron 架构分析
date: 2019-05-24 14:51:53
updated: 2019-05-24 14:51:53
category: OpenStack
tags: [Neutron]
comments: true
header_image: /images/bing/BingWallPaper-2019-05-24.jpg
---
Neutron 在 OpenStack 中的定位是 NaaS （Networking as a Service）。NaaS 有两层含义：

1. 对外接口：Neutron 为 Network、Subnet、Port、Router 等网络资源建立了逻辑模型，并提供了 RESTful API、CLI（命令行）和 GUI（图形化用户接口）
2. 内部实现：利用 Linux 原生以及其他虚拟网络功能，再加上一些硬件网络功能，构建出真正的网络

Neutron 管理的网元，主要以”软“网元为主（也称作虚拟网络功能）。这些”软“网元有三种来源：

1. Linux 原生（内核提供）的网络功能，如 Linux Router、Linux Bridge 等
2. 开源的网络功能，如 OVS 等
3. 厂商提供的闭源产品

在 Neutron 的抽象架构中，Neutron 接到 RESTful API 的请求后，交由模块 WSGI Application 进行初步的处理，然后这个模块通过 Python API 调用 Neutron 的 Plugin 模块。Plugin 模块做了相应处理后，通过 RPC 调用 Neutron 的 Agent 模块，Agent 再通过某种协议（比如 CLI）对 VNF（虚拟网络功能）进行配置。Neutron 内部由不同的组件组成，这些组件之间需要通信，从而引出了 Neutron 消息总线的通信机制。同时，为了提高效率，Neutron 采用了协程来做并发处理。

<!--more-->

## Neutron 中的 WSGI Application

WSGI 是 Web Server Gateway Interface 的缩写。WSGI 本身只是一套标准接口，而非具体实现。这套接口位于 Web Service 和 Web Application（或者 Framework）之间，为的是提升 Web Applications 针对各种 Web Servers 的适配能力。

能够被 Web Server 按照 WSGI 接口调用的 Web Application 称为 WSGI Application，它有三个特征：

1. 是一个可调用的对象
2. 有两个参数：environ、start_response
3. 有一个可迭代的返回值

对于 Neutron 来说，只需要按照 WSGI 规范写好 WSGI Application 即可，WSGI Application 才是真正的业务处理单元。

## Neutron 的消息通信机制

Neutron 内部组件（进程）之间的通信（比如 Neutron Server 与 Neutron Agent），采用 AMQP （Advanced Message Queuing Protocol，高级消息队列协议）机制，进行 RPC 通信。Neutron 采用了三种 AMQP 标准的具体实现：RabbitMQ、Qpid 和 ZMQ，具体应用中根据配置文件选用某种实现。

### AMQP 基本概念

AMQP 把通信的双方（发送方和接收方）分别称为 Producer 和 Consumer。Producer 把消息发送到 Consumer 需要经过 Message Broker 的处理和传递。AMQP 中，承担 Message Broker 功能的是 AMQP Server，而 AMQP 的 Producer 和 Consumer 都是 AMQP Client。

在 AMQP Server 中，有两大部件：Exchange 和 Message Queues，其含义如下：

1. Exchange 接收 Producers 发过来的消息，按照一定规则转发到相应的 Message Queues 中

1. Message Queues 再将消息转发到相应的 Consumers

### AMQP 的消息转发

#### AMQP 消息转发模型

Producer 发送的消息，经过 AMQP Server 转发以后，到达对应的 Consumer，在这个消息转发中，起关键的“路由标识”作用的是一个字符串 Routing Key。具体的说，就是 Producer 首先发消息到 Exchange，Exchange 根据一定的转发规则转发到相应的 Message Queue，然后 Queue 通知绑定到它的 Consumer。Exchange 转发消息给 Message Queue 的规则根据 Routing Key 进行匹配。其匹配模式可分为三种：

1. Direct Exchanges

    - Routing Key 是一个字符串
    - 匹配规则为全值匹配

    Message Queue 绑定的 Routing Key 与 Producer 发送的 Routine Key 完全相同时，Exchange 才会将消息转发给该 Message Queue。

2. Topic Exchanges

    - Routing Key 是一个字符串，但是由“.” 分割为多个子字符串
    - 匹配规则是模式匹配

    所谓模式匹配，是有两个通配符，一个是“*”，代表任意一个字符串；另一个是“#”，代表任意多个子字符串。Producer 发送的 Routing Key 是一个完整的字符串，没有通配符，Message Queue 绑定的 Routing Key 可以有通配符。

3. Fanout Exchanges

    - 没有 Routing Key
    - 所有绑定到 Fanout Exchange Message Queue都能收到相应的 Producer 发送的消息

#### AMQP 的几种通信模式

基于 AMQP 的消息转发模型，有以下几种通信模式：

1. 远程过程调用（RPC）
2. 发布—订阅（Publish-Subscribe）
3. 广播（Broadcast）

发布订阅只需采用 Topic Exchanges 消息转发模型即可，而广播只需采用 Fanout Exchanges 消息转发模型即可。RPC（Remote Procedure Call Protocol，远程过程调用协议），一般称为“远程过程调用”，是一种 Client/Server 通信模型。Client 和 Server 之间有一来（request）一往（response）两个消息。在 request 消息中，RPC Client 担任 Producer 的角色，RPC Server 担任 Consumer 的角色。在 response 消息中，两种角色进行了互换，RPC Server 担任 Producer 角色，RPC Client 担任 Consumer 角色。

## Neutron 的并发机制

Neutron 北向提供 RESTful 接口，需要处理并发请求，南向对接多个“网元”，Neutron 不能算作“计算密集型”应用，而可以归类为“I/O阻塞型”应用，因此 Neutron 主要采用协程，并且在必要的时候，结合多进程方案来共同构建其并发机制。

#### 协程与进程、线程的比较

协程与进程和线程的比较如下：

1. 协程既不是进程，也不是线程，协程仅仅是一个特殊的函数，协程与进程、线程的含义不在同一维度
2. 一个进程可以包含多个线程，一个线程可以包含多个协程
3. 一个线程内的多个协程虽然可以切换，但是这**多个协程是串行执行的**，只能在这一个线程内运行，没法利用 CPU 多核的能力
4. 协程与进程一样，它们的切换都存在上下文切换问题

表面上看，进程、线程和协程均存在上下文切换的问题，但是三者上下文切换又有显著的不同，如下表所示：

|                | 进程                                   | 线程                                   | 协程                                 |
| :------------- | :------------------------------------- | :------------------------------------- | :----------------------------------- |
| 切换者         | 操作系统                               | 操作系统                               | 用户（编程者/应用程序）              |
| 切换时机       | 根据操作系统自己的切换策略，用户不感知 | 根据操作系统自己的切换策略，用户不感知 | 用户自己（的程序）决定               |
| 切换内容       | 页全局目录、内核栈、硬件上下文         | 内核栈、硬件上下文                     | 硬件上下文                           |
| 切换内容的保存 | 保存于内核栈中                         | 保存于内核栈中                         | 保存于用户自己的变量（用户栈或堆）中 |
| 切换过程       | 用户态 - 内核态 - 用户态               | 用户态 - 内核态- 用户态                | 用户态（没有陷入内核态）             |
| 切换效率       | 低                                     | 中                                     | 高                                   |

## 通用库 Oslo

为了降低代码的冗余度，OpenStack 社区创建了 Oslo 项目，从 OpenStack 代码中提取公共的部分，构建出一批 lib 库以供 OpenStack 其他项目使用。Neutron 使用了 Oslo 中的 lib。
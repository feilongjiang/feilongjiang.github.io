---
title: LeetCode 751. IP to CIDR
date: 2017-12-29 15:09:13
updated: 2017-12-29 15:42:58
category: LeetCode
tags: LeetCode
comments: true
---

## Problem
Given a start IP address `ip` and a number of ips we need to cover `n`, return a representation of the range as a list (of smallest possible length) of CIDR blocks.

A CIDR block is a string consisting of an IP, followed by a slash, and then prefix length. For example: "123.45.67.89/20". That prefix length "20" represents the number of common prefix bits in the specified range.

<!--more-->

### Example 1

> **Input:** ip = "255.0.0.7", n = 10
**Output:** ["255.0.0.7/32", "255,0.0.8/29", "255.0.0.16/32"]
**Explanation:**
The initial ip address, when convered to binary, look like this (spaces added for clarity):
255.0.0.7 -> 1111111 00000000 00000000 00000111
The address "255.0.0.7/32" specifies all address with a common prefix of 32 bits to the given address,
ie. just this one address.

> The address "255.0.0.8/29" specifies all address with a common prefix of 29 bits to the given address:
255.0.0.8 -> 11111111 00000000 0000000 00001000
Address with common prefix of 29 bits are:
11111111 00000000 00000000 00001000
11111111 00000000 00000000 00001001
11111111 00000000 00000000 00001010
11111111 00000000 00000000 00001011
11111111 00000000 00000000 00001100
11111111 00000000 00000000 00001101
11111111 00000000 00000000 00001110
11111111 00000000 00000000 00001111

> The address "255.0.0.16/32" specifies all address with a common prefix of 32 bits to the given address,
ie. just 11111111 00000000 00000000 00010000

> In total, the answer specifies the range of 10 ips starting with the address 255.0.0.7 .

> There were other representations, such as:
["255.0.0.7/32", ""255.0.0.8/30", "255.0.0.12/30", "255.0.0.16/32"],
but our answer was the shortest possible.

> Also note that a representation beginning with say, "255.0.0.7/30" would be incorrect,
because it includes address like 255.0.0.4 = 11111111 00000000 00000000 00000100
that are outside the specified range.

### Note

 1. `ip` will be a valid IPv4 address.
 2. Every implied address `ip + x` (for `x < n`) will be a valid IPv4 address.
 3. `n` will be an integer in the range `[1, 1000]`.


## Answer

这个问题是在给定的起始IP地址，求最少的[CIDR][1]正好覆盖n个IP地址。所求的地址要求是连续的，并且要在起始IP的后面。

先说明一下`255.255.0.8/29`中29的含义：
一个IPv4的地址总共有32位，可以表示29表示的是IP的前29为是固定的，后三位可以改变，因此可以覆盖$2^3$(8)个IP。那么要使CIDR尽量的少，就让固定的IP的位数尽量少，同时要保证IP是连续的。

比如我们以`255.0.0.7`开始，覆盖30个地址，那么就有：
> 255.0.0.7/32
只有一个IP地址 剩余30 - 1 = 29

> 接下来的IP地址是 255.0.0.8，最后8位是: 00001000，还有3位可以更改
则有255.0.0.8/29 可以覆盖8个IP，剩余29 - 8 = 21

> 8个地址后的IP是255.0.0.16，最后8位是: 00010000，还有4位可以更改
则有255.0.0.16/28 可以覆盖16个IP，剩余21 - 16 = 5


> 接下来的IP地址是255.0.0.32，最后8位是： 00100000，还有5位可以更改，而剩余的要覆盖的IP只有5个，因此要小于5，否则覆盖的IP范围就会超过给定的个数，因此是4个，即对最后两位进行更改
则有255.0.0.32/30 可以覆盖4个IP，剩余5 - 4 = 1

> 那么最后一个要覆盖的就是255.0.0.36/32，剩余0个要覆盖
至此可以得到最少的CIDR。

### 实现代码

将IP转为数字:

```c++
vector<string> split(const string &str, char delim)
{
    stringstream ss(str);
    string word;
    vector<string> ret;
    while (getline(ss, word, delim))
    {
        ret.emplace_back(word);
    }
    return ret;
}

int ip2Num(const string &ip)
{
    vector<string> vec = split(ip, '.');
    int num = 0;
    num = static_cast<int>(stoi(vec[0));
    num = num << 8 | static_cast<int>(stoi(vec[1]));
    num = num << 8 | static_cast<int>(stoi(vec[2]));
    num = num << 8 | static_cast<int>(stoi(vec[3]));
    return num;
}
```

数字转换为IP
```c++
string num2Ip(int num)
{
    vector<string> vec;
    for (int i = 0; i < 4; ++i)
    {
        vec.emplace_back(to_string(num & 0xff));
        num >>= 8;
    }
    string ret;
    ret.append(vec[3] + '.');
    ret.append(vec[2] + '.');
    ret.append(vec[1] + '.');
    ret.append(vec[0]);
    return ret;
}
```

主函数
```c++
vector<string> ipToCIDR(string ip, int range)
{
    vector<string> ret;
    unsigned num = ip2Num(ip);
    
    while (range)
    {
        int weight = 1;
        int i = 0;
        while (i < 32)
        {
            weight <<= 1;
            if ((1 << i & num) || (weight > range))
                break;
            ++i;
        }
        weight >>= 1;
        range -= weight;
        ret.emplace_back(num2Ip(num) + "/" + to_string(32 - i));
        num += weight;
    }
    return ret;
}
```

[1]: https://baike.baidu.com/item/%E6%97%A0%E7%B1%BB%E5%9F%9F%E9%97%B4%E8%B7%AF%E7%94%B1/240168?fr=aladdin&fromid=3695195&fromtitle=CIDR
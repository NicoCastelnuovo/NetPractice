# NetPractice

NetPractice is a project developed for 42 Heilbroon School.

## Keywords
TCP/IP - network - host - netmask - bit conversion - subnetting - routing

## Subject
The project asks to solve 10 networking simulation exercises. This requires a basic knowledge about IP addresses, routing and subnetting. Here are the notes I collected to understand the topic.

## Index
- [IP Address](#ip-address)
- [Subnet Mask pt.1](#subnet-mask-pt1)
- [Bits conversion](#bits-conversion)
- [Subnet Mask pt.2](#subnet-mask-pt2)
- [Gain more networks](#gain-more-networks)
- [Subnetting reversed](#subnetting-reversed)
- [Do it faster](#do-it-faster)
- [Definitions](#definitions)
- [About the exercises](#about-the-exercises)

## IP Address
The _IP address_ identifies a device into a network. Is composed by a _network_ and a _host_ part. Each octet (so called because composed by 8 bits each) can be a number between 0 and 255, for a total of 2^32 possible IP addresses.

Each IP address belongs to one of the following classes:
```
a) 1.0.0.0   - 126.255.255.255     -> minimum mask of 255.0.0.0
b) 128.0.0.0 - 191.255.0.0         -> minimum mask of 255.255.0.0
c) 192.0.0.0 - 223.255.255.0       -> minimum mask of 255.255.255.0
d) 224.0.0.0 - 239.255.255.255
e) 240.0.0.0 - 255.255.255.255
```
`127.0.0.0 - 127.255.255.255` are left. They are called _loopback_ addresses which point to the local device.

Because of the limited number of public IP addresses, the private IP addresses were invented. They are not routable to the public internet, for which a public address is needed. We can say that all the private IP are mapped to a single public one, through the _NAT_ (Network Address Translation). The Internet Service Provider let our devices to borrow a public IP address. The router manages the translation to connect the device to the external network, using the mentioned public IP.

Ranges of private IP addresses:
```
a) 10.0.0.0    - 10.255.255.255
b) 172.16.0.0  - 172.31.255.255
c) 192.168.0.0 - 192.168.255.255
```

## Subnet Mask pt1
The _subnet mask_ locks some portions of the IP address, telling us which ranges of a given IP are available. This is easy to understand by example.

_Example 1_

255.255.255.0 mask, locks the first three octets of the IP address and allows the last octet to be everything between 0 and 255.

_Example 2_

255.0.0.0 mask, locks the first octet of the IP address and allows the other three octet to be everything between 0 and 255.

_Example 3_

255.255.255.192 mask, locks the first three octets of the IP address and allows the last octet to be just certain values/ranges. Which values/ranges? We'll see it in the next sections.

## Bits conversion
We talk about octets because each number of the mask is 1 byte, or 8 bits. That because we can build the following "power of 2 chart", that we will call from now, chart (A). Each one of those values represent 2^n, where n is their position starting from 0 on the right.
```
(A) 128 - 64 - 32 - 16 - 8 - 4 - 2 - 1
```

_From bit to dec_
```
00110101

(A) 128 - 64 - 32 - 16 - 8 - 4 - 2 - 1
     0	  0    1    1  0   1   0   1
     0  + 0  + 32 + 16 + 0 + 4 + 0 + 1 = 53

00110101 ----------------------> 53
```
_From decimal to bit_
These are two ways to convert decimal to bit.
```
131

131 / 2 = 65 ---> remainder 1
65 / 2  = 32 ---> remainder 1
32 / 2  = 16 ---> remainder 0
16 / 2  = 8  ---> remainder 0
8 / 2   = 4  ---> remainder 0
4 / 2   = 2  ---> remainder 0
2 / 2   = 1  ---> remainder 0
1 / 2   0 0  ---> remainder 1

131 ----------------------> 1000 0011
```

```
167

167 - 128 = 39                --> 1
39 - 64 = negative            --> 0
39 - 32 = 7                   --> 1
7 - 16 = negative             --> 0
7 - 8 = negative              --> 0
7 - 4 = 3                     --> 1
3 - 2 = 1                     --> 1
1 - 1 = 0                     --> 1

167 ----------------------> 1010 0111
```

## Subnet Mask pt2
The subnet mask tell us different information about the IP address, just counting its 1 and 0. The contiguous zeroes in the mask represent the _host_ part of the IP address, while the contiguous ones represent the _network_ part of it
From counting the zeroes, we can obtain for example the number of IP addresses which are available to be used in each subnet.
Remember also, that the first and last available addresses in a range, are always reserved for the _Network_ address and the _Broadcast_ address.

_Example_
```
11111111.11111111.11111111.11100000 which results in 255.255.252.224
N of zeroes = 5
2^5 = `32` IP addresses per network (from which you have to remove 2 reserved)
```

### Gain more networks
Let's suppose that you have a network, from which you want to create other subnetworks. Follow those steps:

0) Convert the mask to binary
1) Calculate how many host bits need to steal (see the chart B later)
2) Set this number of bits to 1 and convert to decimal
3) Find the increment
4) Create the networks

_Example_

0) At `192.168.1.0/24` (see CDIR in [Definitions](#definitions)) in I want to create other `30` networks. I first convert the mask to a binary.
```
255.255.255.0
11111111.11111111.11111111.00000000 is our mask (because of `/24`).
```
1) 30 can be contained first into 32 in chart b), so counting the position from there, we discover that we need to "steal" `5` set bits from the network part and turn them to 1.
```
B) 256 - 128 - 64 - [32] - 16 - 8 - 4 - 2
```
2) Setting 5 bits to 1, the new mask become
```
11111111.11111111.11111111.11111000
255.255.255.248
192.168.1.0/29
```
3) First, what is the _increment_? The increment represents the size of the created networks and the range of host addresses which belongs to. The increment can be found in using the last bit on the right which is set to 1. Using the chart (A), we notice that is the increment is 8.
```
     1     1     1      1     1    0   0   0
(A) 128 - 64  - 32  -  16 -  [8] - 4 - 2 - 1
```
4) Now, using the increment, the resulting networks are the following (remember: what about the first and last one?):
```
192.168.1.0 - 192.168.1.7
192.168.1.8 - 192.168.1.15
192.168.1.16 - 192.168.1.23
...etc...
192.168.1.248 - 192.168.1.255
```

### Gain more hosts
What if we need more IP addresses to be used? The solution is "stealing" bits from the network part of the IP and set them to 0. _Subnetting_ means this, changing the subnet mask to fit the needs.

But what if I want exactly a certain number of hosts? The process is similar to the previous one.

0) Convert the mask to binary
1) Calculate how many bits need to steal (char B)
2) _Reserve_ this number of bits and convert to decimal
3) Find the increment
4) Create the networks

_Example_

Let's suppose I have an address, and from mask `255.255.255.255` I want to obtain `45` hosts.
```
B) 256 - 128 - [64] - 32 - 16 - 8 - 4 - 2 ---> 6 bits
```
What I do with these bits, is just reserve them and don't touch, from the right to the left of the mask. The rest can be turned to 1, obtaining the following new mask, whose Networks
```
11111111.11111111.11111111.11000000
```
From this mask, we can then obtain everything we need to subnetting: we can have he mask in decimal and find the increment and the address ranges, and we already have the tools to do it. _Just remember_: Use chart (A) to convert bin to decimal and find the increment/range. Use char (B) to find the bits needed to gain networks (which will turn them to 1) or hosts (which will be reserved).

### Subnetting reversed
Given some IP addresses and their mask, is possible to calculate the networks and then get rid of eventual connections problem. The operation includes the next steps:

0) Find the increment, using the mask provided
1) Build the network ranges
2) Verify if the devices are in the same network

### Do it faster
Use this table instead of calculating everything by hand (it can go over 255.255.0.0, but for the pourposes of this guide it's enough).

| Subnet Mask       | Number of Hosts | CIDR Notation |
|-------------------|------------------|---------------|
| 255.255.255.254   | 2                | /31           |
| 255.255.255.252   | 4                | /30           |
| 255.255.255.248   | 8                | /29           |
| 255.255.255.240   | 16               | /28           |
| 255.255.255.224   | 32               | /27           |
| 255.255.255.192   | 64               | /26           |
| 255.255.255.128   | 128              | /25           |
| 255.255.255.0     | 256              | /24           |
| 255.255.254.0     | 512              | /23           |
| 255.255.252.0     | 1024             | /22           |
| 255.255.248.0     | 2048             | /21           |
| 255.255.240.0     | 4096             | /20           |
| 255.255.224.0     | 8192             | /19           |
| 255.255.192.0     | 16384            | /18           |
| 255.255.128.0     | 32768            | /17           |
| 255.255.0.0       | 65536            | /16           |

Given an IP address and its mask, use this formula to calculate the range in which the host is:

0) Use the subnet table to understand the number/range of hosts (also called increment)
1) Find quotient and remainder between the number and the increment
2) Find the _network address_ subtracting the remainder from the number
3) Find the _broadcast address_ adding the (increment - 1)

_Example_
```
IP = 151.26.32.131 --- mask = 255.255.255.192

131 / 64 = 2             (quotient)
131 - (64 * 2) = 3       (remainder)
131 - 3 = 128            (network address)
128 + (64 - 1) = 191     (broadcast address)
```

## Definitions

### CDIR (Classless Inter-Domain Routing)
It is a notation to represent IP address with its routing prefix (the mask). It can be obtained just postfixing the number of the set bits into the mask, preceeded by a slash (`/24`).

### Gateway
The _Gateway_ refers to the router, which allows out device to commnunicate with other devices not connected locally. It usually takes an address of our network and has to be to the same (sub)network.

### Routing table
If a router is directly connected with the network, whose "address is on the packet", it will route the packet directly to it, without consulting the routing tablr. If the router is not directly connected with it, the router check its table to find the way where to direct it. The routing table contains the possible _end destinations_ and the closest router from which the packet has to travel through (called _next hop_). So _remember_: the routing table is needed only for those hosts which are NOT directly connected!

_Example_
If we talk about the routing table of "some place in the internet", the next hop could be the closest router, while the destination is the _network address_ (with CDIR mask) which cover all possible destination.

### HUB vs Switch
_HUB_ used in the past, send the message to all the members of a network, despite the specified IP address. The _Switch_ addresses the message correctly, so that only the correct recipient recieves it. The switch has an internal memory which learns and remembers each device's MAC address (Media Access Control).

### TCP/IP
*TCP/IP* stays for Transmission Control Protocol/Internet Protocol. It is a network communication protocol, a set of rules and standards which describe how software and hardware should interact/communicate in a network.


## About the exercises
Give the solution has no meaning, I prefer to give you some tips:

0) _Bits conversion_. It is important to understand the basics of conversion, and have a solid technique to perform from bit to decimal and viceversa. Just learn a way to do it in which you're comfortable.
1) _Subnetting_. This is the main thing you have to understand. What is subnetting and how can be done?
2) _Public vs Private IP_. Yes, sometimes the given configuration it'll contain random IP addresses, sometimes private IP which may not be used to communicate through a gateway. So google which are those IP addresses ranges and use the propers.
3) _Routing table_. Understand exactly what goes in _destination_ field and _next hop_ field. "In the destination goes the destination" is not an answer.
4) _What am I doing?_. Have a global vision of what the exercise is asking you, the meaning of what you are doing.

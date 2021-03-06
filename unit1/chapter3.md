# IP 地址规划设计技术

## 一些概念名词

* 子网 subnet
* 掩码 mask
* NAT 网络地址转换
* 无类域间路由 CIDR

## 标准分类的 IP 地址

IPv4 的地址长度为 32 bit

* A 类地址：1.0.0.0 ~ 127.255.255.255
* B 类地址：128.0.0.0 ~ 191.255.255.255
* C 类地址：192.0.0.0 ~ 223.255.255.255
* D 类地址：224.0.0.0 ~ 239.255.255.255
* E 类地址：240.0.0.0 ~ 247.255.255.255

### A 类地址

网络号（net ID）第一位为 0，其余 7 位可以分配

A 类地址共分为大小相同的 128（2^7）块，每一块的 net ID 不同

第一块：0.0.0.0 ~ 0.255.255.255 (net ID = 0)
最后一块：127.0.0.0 ~ 127.255.255.255 (net ID = 127)

第一块和最后一块地址留作特殊用途，net ID = 10 的 10.0.0.0 ~ 10.255.255.255 用于专用地址

主机号为全 0 和全 1 的两个地址保留用于特殊目的

### B 类地址

网络号长度为 14 位，网络号总数为 16384（2^14）；主机号（host ID）长度为 16 位

主机号为全 0 和全 1 的两个地址保留用于特殊目的

### C 类地址

网络号长度为 21 位，主机号长度为 8 位

主机号为全 0 和全 1 的两个地址保留用于特殊目的

### 特殊地址形式

特殊的 IP 地址包括：直接广播地址、受限广播地址、“这个网络上的特定主机”地址与回送地址

#### 直接广播地址

A、B、C 类 IP 地址中，主机号全位 1 的地址为直接广播地址，用来使路由器将一个分组以广播方式发送给特定网络上的所有主机

#### 受限广播地址

32 位全为 1 的 IP 地址（255.255.255.255）为受限广播地址，用来将一个分组以广播方式发送给本网络中的所有主机

#### “这个网络上的特定主机”地址

网络号全为 0

#### 回送地址

127.0.0.1

用于网络软件测试和本地进程间通信

## 划分子网的三级地址结构

地址结构：net ID - subnet ID - host ID

### 子网掩码

* 用点分十进制表示：255.255.252.0
* 用“/”加上网络号+子网号的长度：网络号/22

### 无类域间路由（CIDR）技术

CIDR 地址可以采用“斜线记法”，即在 IP 地址后面加一个斜线“/”，然后加上网络前缀所占的比特数

## IP 地址规划方法

### 基本步骤

* 判断用户对网络与主机数的需求
* 计算满足用户需求的基本网络地址结构
* 计算地址掩码
* 计算网络地址
* 计算网络广播地址
* 计算网络的主机地址

### 基本方法

#### 步骤一：判断用户对网络与主机数的需求

* 网络中最多可能使用的子网数量 N<sub>net</sub>
* 网络中最大网段已有的和可能扩展的主机数量 N<sub>host</sub>

#### 步骤二：计算满足用户需求的基本网络地址结构参数

##### 选择 subnet ID 字段的长度值 X，要求 N<sub>net</sub><=2<sup>X</sup>

例如，子网数量 N<sub>net</sub> 为 10，那么选择 subnet ID 字段的长度值 X=4

##### 选择 host ID 字段的长度值 Y，要求 N<sub>host</sub><=2<sup>Y</sup>

例如，子网主机数量 N<sub>host</sub> 为 12，那么选择 host ID 字段的长度值 Y=4

注意 host ID 字段的值为全 0 表示的是该网络的 net ID，host ID 字段的值为全 1 表示的是该网络的广播地址，因此 Y=4 时，最多可用的主机号只有 14 个

##### 根据 X+Y 的值可以确定需要申请哪一类  IP 地址

X=4、Y=4，总和长度为 8，那么一个 C 类地址段就可以满足要求

#### 步骤三：计算地址掩码

根据地址掩码的定义，没有划分子网的 C 类网络的地址掩码是 255.255.255.0

划分子网之后的地址掩码是将一个标准 32 位 IP 地址中高于 host ID（Y 位以上）的高位置 1 即可，也就是需要将标准 IP 地址的第 4 个 8bit 中的前 4bit 位置 1，即该地址掩码为 255.255.255.240，如果这个 C 类地址为 192.168.1.0，那么可以表示为 192.168.1.0/28

#### 步骤四：计算网络地址

由于 host ID 长度 Y=4，那么每个子网中最多有 14 个主机，也就是说相邻子网的主机地址增加值为 16，那么本例中地址一个网络号为 192.168.1.0，那么下一个网络号是在此基础上增加 16，以此类推

注意，RFC 文档中规定不使用第一个和最后一个网络地址，即本例中不使用 192.168.1.0 和 192.168.1.240 地址号，但是如果 TCP/IP 协议设定允许的话，也可以使用

#### 步骤五：计算网络广播地址

一个子网的定向广播地址是比下一个子网地址号小 1 的地址

#### 步骤六：计算网络的主机地址

剔除网络地址与广播地址以外的网络地址都是主机可以使用的 IP 地址

## 子网地址规划方法

### 基本方法与步骤

* 确定所需要的 net ID 数
* 确定所需要的 host ID 数
* 基于以上要求，创建以下内容
    - 为整个网络设定一个子网掩码
    - 为每个物理网络设定一个不同的 subnet ID
    - 为每个子网确定主机的合法地址空间

### 规划示例

#### 用户需求

* 一个校园网获得一个 B 类 IP 地址（156.26.0.0），要进行子网划分
* 该校园将由近 210 个网络组成
* 为了便于管理，要求根据目前的情况进行子网划分

#### 确定子网号 subnet ID 的长度

* 子网数量在 254 个之内，所以取子网号的长度为 8 位，则子网掩码为 255.255.255.0
* 由于子网号 subnet ID 与主机号 host ID 不能为全 0 或全 1，因此校园网只能拥有 254 个子网，每个子网只能用 254 台主机

#### 确定子网地址

根据以上方案，校园网可用的 IP 地址为：

* 子网 1：156.26.1.1 ~ 156.26.1.254
* 子网 2：156.26.2.1 ~ 156.26.2.254
* ...
* 子网 254：156.26.254.1 ~ 156.26.254.254

## IPv6 地址规划基本方法

### 主要特征

IPv6 地址长度规定为 128 位，可以提供超过 3.4x10<sup>38</sup> 个 IP 地址

### 分类

* 单播地址
* 组播地址
* 多播地址
* 特殊地址

### 表示方法

IPv6 的 128 位地址按每 16 位划分为一个位段，每个位段被转换为一个 4 位的十六进制数，并用冒号“:”隔开，这种表示法成为冒号十六进制表示法

* 可以通过压缩某个位段中的前导 0 来简化 IPv6 地址的表示，例如“00D3”可以简写为“D3”，“02AA”可以简写为“2AA”，但“FE08”不能简写为“FE8”，同时每个位段至少应该又一个数组，“0000”可以简写为“0”
* 双冒号表示法：如果几个连续位段的值都是 0，那么这些 0 就可以简写为“::”

### IPv6 地址表示时需要注意的问题

* 在使用零压缩法时，不能把一个位段内部的有效 0 也压缩掉
* :: 双冒号在一个地址中只能出现一次
* IPv6 地址不支持子网掩码

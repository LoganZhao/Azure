# 第二次checkpoint

[TOC]

## 知识学习
### 本周完成

**完成RDS相关知识的学习**
- 关系型数据库基本概念学习。
- RDS相关实操，包括mysql,postgresql,redis的创建以及一些基本运维操作
- 操作了杜康,了解了一些基本功能以及排查案例
```bash
工单：691GL9DY
RDS 实例ID ： dds-3nsc3ee7114b6204
数据库引擎 ： MongoDB
性能问题时间段 ： 11月4号到11月5号
问题描述 ： mongodb 有一些慢日志查询，不是我们业务产生的
```
- 大杜康找到instance, direct到具体rds实例
- 找到慢日志位置，定位具体日志内容：
```javascript
11.192.145.216	local	{"op":"getmore","ns":"local.oplog.rs","query":{"getMore":{"$numberLong":"40778703473"},"collection":"oplog.rs","maxTimeMS":{"$numberLong":"5000"},"term":{"$numberLong":"1"},"lastKnownCommittedOpTime":{"ts":{"$timestamp":{"t":1604594072,"i":72}},"t":{"$numberLong":"1"}}},"originatingCommand":{"find":"oplog.rs","filter":{"ts":{"$gte":{"$timestamp":{"t":1599819071,"i":1}}}},"tailable":true,"oplogReplay":true,"awaitData":true,"maxTimeMS":{"$numberLong":"60000"},"term":{"$numberLong":"1"}},"cursorid":40778703473,"keysExamined":0,"docsExamined":7,"numYield":0,"locks":{"Global":{"acquireCount":{"r":{"$numberLong":"2"}}},"Database":{"acquireCount":{"r":{"$numberLong":"1"}},"acquireWaitCount":{"r":{"$numberLong":"1"}},"timeAcquiringMicros":{"r":{"$numberLong":"808694"}}},"oplog":{"acquireCount":{"r":{"$numberLong":"1"}}}},"nreturned":7,"responseLength":3906,"protocol":"op_command","millis":808,"planSummary":"COLLSCAN"}	808	7	0	7	2020-11-06 00:34:33	__system
11.195.33.95	local	{"op":"getmore","ns":"local.oplog.rs","query":{"getMore":{"$numberLong":"41765567940"},"collection":"oplog.rs","maxTimeMS":{"$numberLong":"5000"},"term":{"$numberLong":"1"},"lastKnownCommittedOpTime":{"ts":{"$timestamp":{"t":1604594072,"i":72}},"t":{"$numberLong":"1"}}},"originatingCommand":{"find":"oplog.rs","filter":{"ts":{"$gte":{"$timestamp":{"t":1599819071,"i":1}}}},"tailable":true,"oplogReplay":true,"awaitData":true,"maxTimeMS":{"$numberLong":"60000"},"term":{"$numberLong":"1"}},"cursorid":41765567940,"keysExamined":0,"docsExamined":7,"numYield":0,"locks":{"Global":{"acquireCount":{"r":{"$numberLong":"2"}}},"Database":{"acquireCount":{"r":{"$numberLong":"1"}},"acquireWaitCount":{"r":{"$numberLong":"1"}},"timeAcquiringMicros":{"r":{"$numberLong":"808762"}}},"oplog":{"acquireCount":{"r":{"$numberLong":"1"}}}},"nreturned":7,"responseLength":3906,"protocol":"op_command","millis":808,"planSummary":"COLLSCAN"}	808	7	0	7	2020-11-06 00:34:33	__system
```
- 有关oplog生成很多日志，经研究 oplog主要是mongodb用于做主从同步的，类似于mysql的binlog，咨询了PD，验证了想法并告知客户并不是客户端的操作。


**OSS+CDN 学习中**
- CDN 学习了原理和工具(包括神农鼎，常见的域名绑IP，httpstat等等)

- 如果客户反应CDN的加速效果没有很理想，可以建议客户使用httpstat这个工具，效果：

```bash
D:\Tools>httpstat https://cdn.emarineonline.com.cn

Connected to 117.25.156.167:443

HTTP/2.0 200 OK
Server: Tengine
Accept-Ranges: bytes
Age: 6
Ali-Swift-Global-Savetime: 1605162460
Content-Type: text/html; charset=utf-8
Date: Thu, 12 Nov 2020 06:27:40 GMT
Eagleid: 6e5084a516051624662385728e
Etag: W/"qiyk2v21e"
Last-Modified: Thu, 29 Oct 2020 10:06:31 GMT
Timing-Allow-Origin: *
Vary: Accept-Encoding
Via: cache15.l2cn1801[1856,200-0,M], cache12.l2cn1801[1858,0], cache12.l2cn1801[1858,0], kunlun9.cn210[0,200-0,H], kunlun7.cn210[3,0]
X-Cache: HIT TCP_MEM_HIT dirn:-2:-2
X-Swift-Cachetime: 3600
X-Swift-Savetime: Thu, 12 Nov 2020 06:27:40 GMT

Body discarded

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[     23ms  |          20ms  |        234ms  |             25ms  |             1ms  ]
            |                |               |                   |                  |
   namelookup:23ms           |               |                   |                  |
                       connect:44ms          |                   |                  |
                                   pretransfer:278ms             |                  |
                                                     starttransfer:304ms            |
                                                                                total:305ms
                                                                                
```

- OSS 进行中，实操了ossutil，赤骥等工具。 了解了ACL以及对于用户RAM授权之类的问题。

**Aliyun SDK的使用**
- 发现客户很多情况下使用SDK报错，在泽航同学帮忙下做了aliyun sdk的lab。

### 下周计划
- 容器以及中间件的了解
- 需要花一天时间了解咨询业务基础
- 计划下周周中（周四）开始弹性上线接工单，从ECS，网络和存储开始

### 感想
- 根据上工单的同事，还是在readiness阶段尽可能多了解一些产品以及客户可能的使用场景，避免出现完全没听过这个产品的现象。
- 目前ECS工单量很大。注意了一下 问题类型比较杂，且客户可能不够专业（不能提供足够信息）, 可能要求工程师在OS内部原理，管控以及计费方式有多方面了解。后面垂直线划分以及招聘的时候会多考虑进去。
- 容器的工单相对比较难做，升级率比较高，接下来也会多花些时间研究容器这块的readiness。

## 业务运营

**跟佳琪就技术小二做交接**
- 目前基本完成包括： 招聘，权限管理，sp负责人对接， 人员层级以及画像
- 招聘已经进行，（1个pass,1 fail, 3 pending）

**下周要对技术小二进行软技能话术培训**

**垂直线2.0方案的计划 （已初步写好，会与张琦讨论）**

### 挑战
- 依旧是技术readiness和合作伙伴事宜的精力问题。



# 第三次checkpoint

## 知识学习
**完成容器学习**
**对咨询类问题有初步了解**
**补补边角料 包括安全，日志服务，邮件推送，视频服务等等，熟悉产品以及具体流程。

**周二开始弹性上线工单，目前经手10个工单，已关闭3个工单 2个好评
## 工单处理
5P1G0T8W Ddos高防（国际）+加速路线，发现从国内到hk服务器IP latency很高。

- 首先看到解析发现域名解析已经切换到高防IP了,且銩包严重

```bash
|------------------------------------------------------------------------------------------|
|                                      WinMTR statistics                                   |
|                       Host              -   %  | Sent | Recv | Best | Avrg | Wrst | Last |
|------------------------------------------------|------|------|------|------|------|------|
|                              30.43.32.1 -    0 | 2647 | 2647 |    2 |    5 |  122 |    3 |
|                             30.42.2.205 -    0 | 2647 | 2647 |    4 |   49 |  290 |   26 |
|                               30.42.2.1 -    0 | 2647 | 2647 |    2 |    5 |   89 |    3 |
|                               30.42.1.5 -    4 | 2343 | 2266 |    5 |   15 |  101 |   30 |
|                         116.251.116.245 -    0 | 2647 | 2647 |    2 |    6 |  135 |    4 |
|                          115.238.21.126 -    1 | 2639 | 2637 |    2 |    6 |   93 |    4 |
|                          220.191.200.53 -   38 | 1062 |  665 |    0 |    5 |   32 |    5 |
|                             202.97.26.9 -   38 | 1062 |  665 |    0 |   11 |   45 |   13 |
|                           202.97.24.210 -   39 | 1051 |  651 |    0 |   57 |  169 |    6 |
|                            202.97.35.86 -   39 | 1051 |  651 |    0 |    8 |   59 |    7 |
|                           202.97.71.194 -   63 |  759 |  286 |    0 |  150 |  184 |  146 |
|                            202.97.50.30 -   58 |  804 |  342 |    0 |  160 |  194 |  157 |
|                            218.30.53.87 -   62 |  764 |  292 |    0 |  155 |  259 |  151 |
|hundredge0-1-0-0.scbr01.sjo01.pccwbtn.net -   59 |  795 |  331 |    0 |  153 |  268 |  151 |
|                             63.218.7.30 -   39 | 1051 |  651 |    0 |  140 |  252 |  138 |
|                   No response from host -  100 |  531 |    0 |    0 |    0 |    0 |    0 |
|                   No response from host -  100 |  531 |    0 |    0 |    0 |    0 |    0 |
|                   No response from host -  100 |  531 |    0 |    0 |    0 |    0 |    0 |
|                             170.33.9.91 -   43 |  979 |  561 |    0 |  187 |  356 |  139 |
|________________________________________________|______|______|______|______|______|______|
   WinMTR v0.92 GPL V2 by Appnor MSP - Fully Managed Hosting & Cloud Provider

```
- 通过云盾免登发现客户的实例发现他的加速ip已经黑洞了 T.T
- 黑洞过后，域名解析到高速IP恢复正常。
- 建议客户做些调整来




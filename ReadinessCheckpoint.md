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







[TOC]


# 第三次checkpoint

## 知识学习
**完成容器学习**
**对咨询类问题有初步了解**
**补补边角料 包括安全，日志服务，邮件推送，视频服务等等，熟悉产品以及具体流程**

**周二开始弹性上线工单，目前经手14个工单，已关闭4个工单 3个好评

![avatar](https://github.com/LoganZhao/Azure/blob/master/4EB6D8A3-5544-4de5-ADC1-0A4BD57E9C3D.png)


## 工单处理
### 5P1G0T8W 

- Ddos高防（国际）+加速路线，发现从国内到hk服务器IP latency很高。

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
- 黑洞过后，域名解析到高速IP恢复正常
- 建议客户做些调整来避免他的加速IP被频繁黑洞

1）由于beaver是在侧机房的设备，所以流量已经到达了机房被判定了ddos攻击，在域名或者IP段设置黑名单已经没有用了
![avatar](https://github.com/LoganZhao/Azure/blob/master/beaver.png.jpg)
2）经排查，可以建议客户升级成安全加速线路
![avatar](https://github.com/LoganZhao/Azure/blob/master/testresult.png)

### F41GF6P8 
日志服务，学习如何配置告警，而不通过云监控来配置。

![avatar](https://github.com/LoganZhao/Azure/blob/master/{FE1144EC-8FAB-4B24-9BA0-C08D83662ED6}.png.jpg)

## 合作伙伴事宜
- 招聘
目前已经面试8人，这边pass interview 2人，一人预计月底入职 加入ECS组，另一人在谈offer阶段

- 垂直线划分
拟定计划，已和几位垂直线负责人私下对齐过，各垂直线也基本确定。准备这两天继续推动

- 周三向技术组deliver 话术以及处理工单流程上的培训，并且将新版话术应用工单处理中


## 下周计划

- 继续弹性工单上线争取了解更多产品及其处理流程
- 需要想一下弹性计算垂直线的新人readiness plan以及目前人员的能力提升计划



# 第四次checkpoint

## 知识学习

- 本周主要对一些工单处理的陌生问题进行补齐，包括消息队列，高速通道，RDS，大数据的排查思路等等
- 熟练工具的使用：云台，嫦娥(实例健康检查，winPE挂盘操作)，齐天(查询VPN 连接信息/SLB后端健康检查日志)，赤骥（通过requestid查具体失败原因），鼎（查看客户CDN的配置）等等
- 流程了解：加白流程, aone拉群时机，工单转派原则等等
- 加入弹性计算虚拟小组，希望能够学习一些东西给国际线同学

## 工单处理
![avatar](https://github.com/LoganZhao/Azure/blob/master/cp4.png.jpg)

## 案例分享

- F41GF888 - 高速通道VBR无法加入CEN

- 1）熟悉文档检索
  ```cs
  通过Aone,云智库等方式查找类似报错
  ```
- 2）熟悉加白流程
  ```cs
  加白流程目前已经由L2承接，网络加白问题九善为对接人
  ```
- 3）明白底层原理
  ```cs
  https://yuque.antfin.com/docs/share/d162a5d6-5021-48be-bfd6-161321f9e448?#
  某些VBR是华三老旧设备，因为不支持MPBGP，在使用多专线冗余接入并且使用BGP的情况下，会导致环路。因此产品上加入CEN后会报错。
  ```
  
 - 4) 确认客户信息，给客户解决办法
  ```cs
  他们VBR既用了BGP+静态，又用了多专线冗余。所以是不支持CEN加白。
  
  解决办法1. 客户在这个VBR上只使用静态路由
         2.如果使用BGP，只有一条专线
  如果上述条件满足其一，则开放白名单
  ```

- F41GF828 - 健康检查异常

- 1）熟悉齐天上查看客户的配置
  ```cs
     · 七层 https 443 转发 后端 87口
     · 且客户的nginx87端口是监听的
     · 转化成4层TCP 就没问题了
   ```
- 2）自己手动配置健康检查
   ```cs
      · 7层 nginx 健康检查结果正常
    ```
- 3）通过与客户配置的对比得出结论
   ```cs
      · 建议客户直接curl 健康检查的网页, 发现报了405
      · 根据健康检查的配置 只有2xx和3xx才返回healthy，建议客户修改他的健康检查页面或者配置
   ```


## 合作伙伴事宜
-  招聘 
```cs
周二任洋洋已经入职，加入弹性计算组，亚青为mentor。 任锐（12/4 计划加入网络组）目前pipeline还有三人，预计本周面好。
王福坤已经申请离职，具体时间还未决定，要与SP进行商量，尽快做好人员交接以及招聘工作
```
- 垂直线划分以及系统派单规则，周二已经正式生效并且使用。通过反馈，修改相应规则。
- 接手技术组合作伙伴的大部分事宜，比如排班，周会等等

### 问题
- 经过L1以及小二反馈，一是小二升级L1 case summary写的不详细 二是已经tag L1升级的工单，L1有时会派单回给小二。
```cs
· 周会已经与小二强调升级L1时要标注足够的排查信息，如果没有做到，周会上点名
· 与L1沟通，主要是因为现在加入了升级组和垂直线组，单量太大。确定派单规则，同时修改他们的垂直线组，只留下L1，兜底和一条垂直线。根据反馈再进行调整
```

## 下周计划
- 工单L1上线
- 技术合作伙伴的各项事宜
- 弹性计算的readiness







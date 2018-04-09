---
layout: post
title:  "java technology selection for siov"
categories: computer
tags:  computer java
author: Jixianzhao
---

* content
{:toc}



## 1. 中间件技术选型 即 2017年二三季度 JAVA组的技术目标
1  **rpc**: grpc  
2  **configure**: spring-cloud-configserver  
3  **discovery**: spring-cloud-eureka  
4  **message queue**: rabbitMQ  

以上4个中间件，计划在今年应用到生产环境。作为**本年度JAVA组的技术目标**。  
之后会不断地加入新技术应用。    
本节详细说明各备选方案之间的权衡。  
  
    
### **为什么不选择vert.x**
vert.x是全新的框架，与spring类似的定位，多语言，跨平台，出生就为了高并发，基于reactive pattern，是技术追求者的首选，但**学习成本过高**。    
作为竞争对手，spring团队推出了 spring_reactive 组件。  
vert.x在3.0版本之后，全面优化了设计体系，将自身定义为工具包，可以在多种平台上使用，包括spring。真正有需要且有能力的时候，我们可以考虑接入。

<br>


## 2. 选型过程中需要权衡的指标
1. 体系设计清晰，可扩展。   如果使用的技术本身混乱不堪，会严重增加项目的维护成本。  
2. 引入过程所需知识面小。   确保项目引进可控。  
3. 有活跃的社区支持。  目前我们团队力量较薄弱，若该技术有bug需要开源社区尽快支持。  
4. 技术发展前景好。     需求和技术的发展均日新月异，考虑到全新技术的学习成本大，最好团队使用的技术本身就具有成长性，能一脉相承，一起成长。  
5. 性能足够优秀。  
    
    
## 3. 方案对比
### 3.1 rpc
grpc / thrift / ice / dubbo 

|     |grpc| thrift|ice|dubbo|
|----|------|------|------|------|
|协议层| protocol buffer |	TBinaryProtocol  <br> TCompactProtocol <br> TJSONProtocol  <br> TDebugProtocol|||
|传输层|	http2|	Tsocket <br> TFramedTransport <br> TFileTransport <br> TMemoryTransport <br> TZlibTransport |||
|服务端| asynchronize server |	TSimpleServer <br>  TThreadPoolServer <br>  TNonblockingServer|||
|权限认证|	ssl/tls <br>  OAuth 2.0 <br> API | ssl |||
|流式处理|	支持|	不支持|||
||||||
|特性|少，实现的很好|很多，不同的语言间有差异|多||
|设计和代码质量|生成代码少，干净|生成代码太多|体系较完善,，但设计略差且不易在原有设计上继续扩展完善，不易使用| 一般|
|开放程度|open mailing list <br> code base and issue tracker <br> 由 google 驱动项目演进 | apache 开源项目|开源，由zeroc公司开发| 由阿里巴巴公司团队开发维护|
|文档|非常详细|非常少，而且很多特性没有说明|较多，有系统的中文书籍|详细，全中文|
|其他| http2协议适合移动终端通信。 <br> 2015年开源，越来越多的公司投入到生产中使用。 |支持的语言更多，好的例子难找|  | 已经进入废弃阶段|
|是否有使用经验|无|有|有|有，不熟|
  
<br>  
#### 排除的备选方案及原因
**ice**: 基于纪贤钊个人的经验，ICE相较thrift和dubbo体系比较完善，但还不足够好，且技术生态封闭，官方团队没有继续优化。  
**dubbo**: 整体设计及提供的功能满足现我们阶段的需求，但自身的整体设计耦合严重，不能很好地支持微服务架构。且整个项目现在只有2个人在维护，阿里团队已经逐步废弃该项目。    
**thrift**: 经过了facebook及各大厂商的实践检验，性能优秀，但本身文档很少，新成员很难上手，且不适用于移动端通信。    

#### 确定方案为 grpc
该项目在2015年由谷歌开源，在此之前已经经过谷歌团队的实践检验，每秒的远程调用次数能达到 O(10^10^) 。同时其使用的序列化协议 protocol buffer 也是久经考验且性能卓越。http2 协议在效率、安全和节能3个方面都更适用于移动端应用。  
谷歌团队还在大力推动grpc的改进和标准化，http2协议本身是国际标准协议，有利于以后各厂商的技术兼容。  
参考第2节列出的标准，grpc符合第1，3，4，5点。不符合第2点，技术引入可能需要花较多时间，我们团队目前项目的RPC需求不急迫，在接下来半年中完成即可。  
网上有部分信息指出grpc性能严重低于其他几个框架，是因为每次调用均开启了tls（传输层加密）,如果改成批量传输，http2的IO多路复用性能非常优秀，再把加密特性关掉，能达到平均每次远程调用只需要5微秒。相关技术指标有跟谷歌官方团队确认。参考《gRPC Proto3 performance vs Apache thrift perfomance》。  

**参考资料**：    
**开源RPC（gRPC/Thrift）框架性能评测** http://www.itdadao.com/articles/c15a818908p0.html  
**gRPC vs Thrift** http://m.blog.csdn.net/article/details?id=48830511  
**Apache Thrift简介，与其它RPC对比** http://m.blog.csdn.net/article/details?id=7835957  
**怎么看待谷歌今年开源的RPC框架GRPC** https://www.zhihu.com/question/30027669  
**ice thrif dubbo grpc性能测试对比** http://m.aichengxu.com/other/6751582.htm  
**gRPC Proto3 performance vs Apache thrift perfomance** https://groups.google.com/forum/m/#!topic/grpc-io/JeGybvbz8nc  
**gRPC 101 for Spring Developers - Ray Tsang @ Spring I-O 2016**: https://www.youtube.com/watch?v=xpmFhTMqWhc    

<br>


### 3.2 configure
spring-cloud-configserver    
在 spring-cloud 的服务注册发现方案中，配置内容的存储和发布被拆分到2个应用中，其中存储由 spring-cloud-configserver 完成。  
<br>

### 3.3 discovery
spring-cloud-eukera / spring-cloud-zookeeper / spring-cloud-consul / spring-cloud-etcd / alibaba dubbo / yy 集线塔
#### 排除的备选方案及原因
**alibaba dubbo**: 不满足第2小节的第1，2，3，4点。  
**yy集线塔**: 不满足第2节的第1，2，3，4点。  

<br>
  
#### 确定方案为 spring-cloud-eureka
在spring cloud的组件中选择

| |eukera|zookeeper|consul|etcd|
|----|----|----|----|----|
|开发语言|java|java|go|go|
|理论实现|peer-to-peer <br> 确保ap|paxos  确保cp|||

考虑到配置服务对宕机时间近于零容忍，却可以允许短时间（10～30分钟）内数据不一致。
最终选择 spring cloud eureka。

**参考资料**:  
**Spring Cloud Netflix Eureka源码导读与原理分析** : http://blog.spring-cloud.io/blog/sc-eureka.html  
**Cloud Native Java - Josh Long**: https://www.youtube.com/watch?v=5q8B6lYhFvE&t=38s  


### 3.4 message queue: rabbitMQ / kafka / activeMQ    


| | rabbitMQ | kafka | activeMQ |
| -- | -- | -- | -- |
| 可用性、可靠性、稳定性 | 优 | 良 | 优 |
| 并发、吞吐量 | 较高 | 高 | 较高|
| 持久化 | 支持 | 不支持 | 支持|
| 技术成熟度 | 成熟 | 一般 | 成熟 |

#### 确定方案为 rabbitMQ
在综合性能方面，rabbitMQ比activeMQ要好，可靠性上rabbitMQ比kafka更高，因此最终选用rabbitMQ



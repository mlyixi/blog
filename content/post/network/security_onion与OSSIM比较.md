---
title: 'Security_onion与OSSIM比较'
date: "2021-01-11T14:29:48+08:00"
tags: ["SecurityOnion","OSSIM"]
categories: ["network"]
toc: true
---

https://securityonion.readthedocs.io/en/latest/about.html

# SecurityOnion与OSSIM(也叫AlienVault)对比
I use them both (AV and SO) in parallel, and while they are similar in many respects they both have different strengths.

我同时使用了它们（AV和SO），尽管它们在许多方面都相似，但是两者都有不同的优势。

AlienVault centrally captures and logs a lot of different data from many different data sources: syslog from devices, Windows Event Logs, vulnerability scan results, Snort/Surricata, etc. This itself is not too different from SO, but AV then correlates and cross correlates multiple events and sources as well as data from their Open Threat Exchange (threat intelligence data) to allow things to bubble up into alarms based on risk score calculated through the process of correlation. In effect AlienVault is extremely effective at separating the needles from the haystack of all that data from all those sources.

AV集中捕获并记录来自许多不同数据源的许多不同数据：来自设备的syslog，Windows事件日志，漏洞扫描结果，Snort / Surricata等。这本身与SO并没有太大的区别，但是随后AV关联并互关联多个事件和来源，包括来自其Open Threat Exchange 的数据（威胁情报数据），从而可以根据通过相关过程计算出的风险评分，从而形成警报。 实际上，AlienVault能非常有效地从各类数据中分离出关键信息来。

When it comes to maintaining daily visibility into my network investigating alarms, Security Onion has the tools of choice for me.

在维护网络调查警报的日常可见性时，我选择SO为工具。

I have tuned Snort/Snorby to filter out a lot of noisy and common stuff like exe downloads and timing window errors such that it will show me the kind of events that I want to see that might not make it to Alarm status in AV such as RDP connections. I use ratop/Argus to keep daily track of top talkers and where they are talking. I love http_agent and Squert for tracking down what people were doing leading up to an event/alarm. And Sguil with Network Miner and some of the other connected tools is just plain awesome.

我已经对Snort / Snorby进行了调优，以过滤掉许多嘈杂的常见内容，例如exe下载和计时窗口错误，从而可以向我展示我想看到的事件类型，这些事件可能无法使其进入AV中的“警报”状态，例如 RDP连接。 我使用ratop / Argus来保持对日常谈话者及其谈话位置的日常跟踪。 我喜欢http_agent和Squert来跟踪人们在导致事件/警报之前正在做什么。 Sguil和Network Miner以及其他一些连接工具简直很棒。

AV and SO share many of the same components, but AV also has a built in vulnerability scanning, and some pretty good asset tracking, although I hope to see improvement there.

AV和SO共享许多相同的组件，但是AV也具有内置的漏洞扫描功能，并且具有相当不错的资产跟踪功能，尽管我希望在那里有所改善。

AV signs&logs everything but does not keep packet capture like SO does.

AV对所有内容进行签名和日志记录，但不会像以往那样保持数据包捕获。

AV has great reporting if you need that.

如果需要，AV可以提供出色的报告。

Conclusion: AV really doesn't replace SO, and neither does SO replace AV. They are extremely similar at their core, but the end results are very different and they compliment each other very well

结论：AV确实不能替代SO，SO也不能替代AV。它们的核心极为相似，但最终结果却大不相同，而且彼此之间互为称赞。

While AlienVailt is pretty seamless and installs quickly, it also costs a fair amount. SecurityOnion is free, and Doug has done a really outstanding job of packaging it all up into a single product that is just about as seamless and simple as (if not more than) a very expensive commercial product. That alone makes SO stand out.

尽管AV非常无缝并且可以快速安装，但它的成本也很高。SO是免费的，Doug在将其全部打包成一个产品方面做得非常出色，这几乎与非常昂贵的商业产品一样无缝和简单（如果不超过）。仅凭这一点，SO就能脱颖而出。

# 应用收集框架（ACF）
威胁定义-> 量化风险->识别数据源->焦点缩小

1. 威胁定义的原则：是否对数据的机密性、完整性和可用性有负面的影响。根据实际的应用来定义威胁。

2. 量化风险：风险定义为“影响”和“概率”的乘积。一般度量范围各为1～5。这种量化标准需要由专业机构进行评估。

3. 识别数据源：识别有价值的数据源。

4. 焦点缩小：数据源产生的数据在存储、处理和管理等方面产生开销，需要判断是否值得纳入到应用收集框架里。

# SO架构
一个典型的分布式SO监控网络包括3种节点：master server, forward node(sensor)和storge node.

![data-flow.png](/network/data-flow.png "SO数据流图")
简化版：

![distributed.png](/network/distributed.png "简化架构图")
## master server
1. Elasticsearch
2. Logstash
3. Kibana
4. Curator
5. Elastalert
6. Redis (Only if configured to output to a storage node)
7. OSSEC
8. Sguild

## forward node(sensor)
1. Zeek
2. Snort/Suricata
3. Netsniff-NG
4. OSSEC
5. Syslog-NG

## storage node
1. Elasticsearch
2. Logstash
3. Curator
4. OSSEC

## 各种组合
### Standalone
在独立部署中，主服务器组件和传感器组件都在单个机器上运行，因此，您的硬件要求将反映出这一点。 建议将此部署类型用于评估目的，POC（概念验证）和中小型单传感器部署。 尽管可以以这种方式部署Security Onion，但建议您将后端组件和传感器组件分开。

![standalone.png](/network/standalone.png "独立部署架构")
### Master server with local log storage
在企业分布式部署中，主服务器可以存储来自其自身和转发节点的日志。 它也可以用作其他日志源的系统日志目标，以将其索引到Elasticsearch中。企业级主服务器应至少具有8个CPU内核，16-128GB RAM和足够的磁盘空间（建议为数TB），以满足保留要求。

### Master server with storage nodes
这种部署类型利用存储节点来解析和索引事件。它降低了主机的硬件要求。 企业主服务器应具有4-8个CPU内核，8-16GB RAM和100GB至1TB的磁盘空间。 许多人选择将主服务器托管在其VM场中，因为它的硬件要求比传感器低，但需要更高的可靠性和可用性。
### Heavy Node
重节点在本地运行所有传感器组件和弹性组件。这大大增加了硬件要求。在这种情况下，所有索引的元数据和PCAP都保留在本地。通过Kibana执行搜索时，主服务器将查询该节点的Elasticsearch实例。

![heavy-distributed.png](/network/heavy-distributed.png "重型分布式架构")

# 网络接口
## 管理接口
是用来访问SO系统的接口。

## 嗅探接口






---
title: 'DPI工具比较'
date: "2021-01-20T09:10:47+08:00"
tags: ["DPI"]
categories: ["network","Linux"]
toc: true
---


Independent Comparison of Popular DPI Tools for Traffic Classification
采集工具：
<https://sourceforge.net/projects/vbsi/files/Release%206/>
数据集：
<https://cba.upc.edu/monitoring/traffic-classification>
数据集好像不公开，可能要通过邮件申请。

# 摘要：
深度数据包检查（DPI）是用于流量分类的最新技术。按照传统观点，DPI是最准确的分类技术。因此，大多数流行的产品，无论是商业产品还是开源产品，都依赖某种DPI进行流量分类。但是，由于缺乏公共数据集，DPI的实际性能对于研究界仍然不清楚，因为它们无法比较结果和再现性。本文对流量分类文献中常用的6种著名DPI工具进行了全面比较,包括2种商业产品（PACE和NBAR）和4种开源工具（OpenDPI，L7过滤器，nDPI和Libprotoident）。我们研究了它们在各种场景（包括数据包和流截断）以及不同分类级别（应用程序协议，应用程序和Web服务）中的性能。我们精心构建了包含750 K流量的标记数据集，其中包含来自流行应用程序的流量。我们使用奥尔堡大学开发的自愿者系统（VBS）来保证数据集的正确标记。我们将此数据集（包括完整的数据包载荷）发布给了研究社区。我们认为该数据集可以成为比较和验证网络流量分类器的通用基准。我们的结果表明，商用工具PACE是最准确的解决方案。令人惊讶的是，我们发现某些开源工具（例如nDPI和Libprotoident）也实现了很高的准确性。

Deep Packet Inspection (DPI) is the state-of-the-art technology for traffic classification. According to the conventional wisdom, DPI is the most accurate classification technique. Consequently, most popular products, either commercial or open-source, rely on some sort of DPI for traffic classification. However, the actual performance of DPI is still unclear to the research community, since the lack of public datasets prevent the comparison and reproducibility of their results. This paper presents a comprehensive comparison of 6 well-known DPI tools, which are commonly used in the traffic classification literature. Our study includes 2 commercial products (PACE and NBAR) and 4 open-source tools (OpenDPI, L7-filter, nDPI, and Libprotoident). We studied their performance in various scenarios (including packet and flow truncation) and at different classification levels (application protocol, application and web service). We carefully built a labeled dataset with more than 750 K flows, which contains traffic from popular applications. We used the Volunteer-Based System (VBS), developed at Aalborg University, to guarantee the correct labeling of the dataset. We released this dataset, including full packet payloads, to the research community. We believe this dataset could become a common benchmark for the comparison and validation of network traffic classifiers. Our results present PACE, a commercial tool, as the most accurate solution. Surprisingly, we find that some open-source tools, such as nDPI and Libprotoident, also achieve very high accuracy.

# 引言：
网络通信成为位于不同主机上的应用程序之间交换信息的标准方法。交换的应用程序层数据被分段并封装为IP数据包，然后通过网络进行传输。深度数据包检查（DPI）工具通过搜索特定的模式（即签名）来分析数据包的内容。因此，DPI成为用于执行流量分类、网络管理、入侵检测和网络取证的许多工具的基本流量分析方法之一。

基于DPI的流量分类依赖于协议（例如HTTP或POP3）、应用程序（例如Skype或BitTorrent）和Web服务（例如Facebook或YouTube）的特征签名的数据库。必须首先提取并更新这些签名，以使其适应应用程序的不断发展（例如，新应用程序，新版本，新混淆技术）。后来，通常在在线阶段，将它们用于对网络中的流量进行分类。用于在线分类的模式匹配算法计算复杂，通常需要昂贵的硬件。尽管如此，DPI通常被认为是用于流量分类的最准确的技术，并且大多数商业解决方案都依赖于它[1、2、3、4]。

科学文献表明，现有的流量分类技术可以得出准确的结果。不幸的是，正如[5]中指出的，这些技术的比较和验证存在许多问题。验证过程的质量直接取决于用于构建地面真实数据集并对其进行预分类的方法，因此比较各种分类器的执行情况并非易事。研究人员必须在两种建立真相的方法之间进行选择：创建自己的数据集，或使用其他人已经创建的现有数据集。

第一种方法要求研究人员自己构建和标记数据集。这两项任务都是具有挑战性的。构建数据集涉及选择要包含在数据集中的应用程序。鉴于其高准确性，通常使用DPI工具进行数据标记。但是，基于DPI的工具的实际准确性仍不清楚，因为先前尝试比较不同DPI工具性能的工作表明，用于验证提案的基础是通过基于端口的分类器（其他DPI）获得的的工具或可靠性未知的方法[6、7、8、9、10]。因此，比较的结果是有争议的。此外，大多数商业工具都是黑匣子，它们根据自己的研究声称具有很高的准确性，由于它们是使用私人数据集执行的，因此无法验证。

正确验证流量分类技术的第二种方法是使用公开可用的数据集。这样，比较从不同工具获得的结果变得更加容易。示例是由互联网数据分析合作协会（CAIDA）[11]或互联网测量数据目录[12]发布的数据集。尽管数据集已经过预先分类，但此标签的实际可靠性未知，这是测试流量分类器的重要因素。大多数跟踪没有有效负载，或者只有每个数据包的第一个字节。 MAWI存储库[13]包含各种数据包跟踪，包括在跨太平洋线路（150 Mbit / s链路）上每天进行的15分钟跟踪。这些链路的带宽多年来一直在变化。跟踪包含有效载荷的前96个字节，流量通常是不对称的。另一个有用的数据源是Dartmouth存档的无线数据社区资源（CRAWDAD）[14]，该资源存储了来自许多贡献位置的无线跟踪数据。另一个有趣的项目是Waikato互联网流量存储（WITS）[15]，该项目旨在收集和记录Waikato大学WAND网络研究小组拥有的所有互联网踪迹。某些跟踪可以免费下载，其中包含来自各个地区和不同类型的流量跟踪（例如DSL住宅流量，大学校园流量等）。大多数跟踪没有有效载荷（即被清零）或被截断。所有这些数据集尽管可用于许多网络评估，但由于无法对其进行正确的标记，因此对于DPI验证的兴趣有限。

Szabo等[16]介绍了一种用于验证分类算法的方法，该方法独立于其他分类方法（具有确定性），并且可以自动对大型数据集进行测试。作者开发了基于网络驱动程序接口规范（NDIS）库的Windows XP驱动程序。在[17]中采用了另一种获取ground-truth的方法。作者创建了一个工具，该工具从网络收集数据并使用真实的应用程序名称（例如Thunderbird）和应用程序协议名称（例如SMTP）标记流。此工具类似于我们的志愿者系统（VBS），该系统在我们的工作中用于生成ground-truth，并在3.1.1节中进行了进一步说明。在[18]中显示了建立真实性的另一种方法，该方法描述了为加速手动验证过程而开发的系统。作者基于从互联网上可用的数据库（包括L7过滤器）获得的DPI签名，提出了ground-truth验证系统（GTVS）。但是，GTVS不会从操作系统收集应用程序名称，因此无法完全验证所建立的真实性。

在以前的会议论文[19]中，我们试图面对ground-truth可信性的问题。我们描述了获得用于测试各种流量分类工具的真实性的各种方法。作为评估的一部分，我们测试了几个基于DPI的流量分类器，并评估了它们是否可用于获得可靠的真实性。本文不仅是对前文的扩展，而且也是对DPI工具的更广泛、更全面的验证。同时，本文的关注点也不同，因为我们直接比较了所选DPI工具的每类准确性。本文中使用的参考数据集以及前文的参考数据集都是通过第3节中进一步说明的相同方法生成的。但是，这两个数据集存在显着差异。本文中使用的数据集比会议论文中使用的数据集大得多。此外，新数据集还包含属于各种Web服务的带标签的非HTTP流，这是一个独特功能。测试分类器的方法也不同。在我们之前的论文中，我们在单个级别上测试了分类器的准确性。在当前的论文中，我们评估了不同的级别（即应用程序、Web服务等），因此新的评估方法更加详细和完整。

本文比较并验证了用于网络流量分类的六个著名的基于DPI的工具。为了验证我们的工作，我们发布了用于执行研究的可靠的标记数据集。构建此数据集时，已经仔细解决了两个主要方面：标签的可靠性和数据的代表性。我们使用VBS工具[20]来保证贴标过程的正确性。如第3节所述，该工具能够使用创建流程的流程名称来标记流程。这使我们能够创建可靠的事实真相，用作研究团体比较其他建议的参考基准。根据最常用的Internet应用程序和Web服务的众所周知的索引来选择我们的数据集的应用程序。为了允许发布数据集并避免任何隐私问题，我们通过运行大量应用程序并精心模拟用户的常见行为来创建流量。

本文的主要贡献可归纳如下：

* 我们发布了带有完整数据包有效负载的可靠标记数据集。在第4节中进一步描述的数据集包含来自各种常用应用程序的流量。尽管是人为创建的，但我们还是仔细模拟了人类的行为，以生成尽可能真实的数据集。研究社区可以使用此数据集来测试和比较各种流量分类器。

* 我们比较了六个广泛用于网络流量分类的著名的基于DPI的工具。使用以前的数据集，我们评估了PACE，OpenDPI，nDPI，L7过滤器，Libprotoident和NBAR的精度，并比较了它们在不同分类级别和不同场景下的结果。我们知道OpenDPI和L7-filter是废弃的项目，几年前就停止了开发。但是，由于许多现有的科学论文都基于这两个分类器，因此我们决定将它们包括在我们的评估中作为参考和完整性。

* 我们对这些工具在不同分类级别的性能进行了独立而公正的分析。这些结果可以帮助研究人员更好地理解用于设置基本事实的各种工具的准确性，以及网络管理员可以更好地确定哪些基于DPI的分类器更适合其需求和方案。间接地，我们还提供了有关文献中提出的那些非DPI分类技术的可靠性的信息，这些技术使用了本文中比较的一种DPI工具来设定其基本事实。

在本文中，我们集中于一个性能参数：分类精度。我们承认其他性能参数也很重要，例如速度，可伸缩性，复杂性，健壮性或解决方案的价格。但是，鉴于研究社区主要将基于DPI的技术用于离线地面真相生成，因此我们认为这些指标的决定性较小。此外，我们认为我们的数据集也将有助于评估大多数这些参数。

本文的其余部分安排如下。第2节介绍了基于DPI的工具及其用于评估的配置。第3节介绍了用于获取第4节中描述的可靠数据集的方法。第5节介绍了不同基于DPI的工具的性能评估结果。第6节讨论了结果，将其与文献进行了比较，并评论了我们评估的局限性。第7节回顾了相关工作。最后，第8节总结并总结了论文的成果和贡献。

Network communication became the standard way of exchanging information between applications located on different hosts. The exchanged application-layer data is segmented and encapsulated into IP packets, which are transmitted through the network. Deep Packet Inspection (DPI) tools analyze the content of the packets by searching for specific patterns (i.e., signatures). Thus, DPI became one of the fundamental traffic analysis methods for many tools performing traffic classification, network management, intrusion detection, and network forensics. 

DPI-based traffic classification relies on a database of characteristic signatures of protocols (e.g., HTTP, or POP3), applications (e.g., Skype, or BitTorrent), and web services (e.g., Facebook, or YouTube). These signatures must be initially extracted and kept up to date in order to adapt them to the continuous evolution of the applications (e.g., new applications, new versions, new obfuscation techniques). They are later employed, usually in an online phase, to classify the traffic flowing in a network. The pattern matching algorithms for online classification are computationally complex and usually require expensive hardware. Nevertheless, DPI is commonly considered as the most accurate technique for traffic classification, and most commercial solutions rely on it [1, 2, 3, 4]. 

The scientific literature shows that the existing traffic classification techniques give accurate results. Unfortunately, as pointed out in [5], there are numerous problems with the comparison and validation of these techniques. The quality of the validation process directly depends on the methods used to build and pre-classify the ground-truth dataset, so it is not an easy task to compare how various classifiers perform. The researchers must choose between two approaches of ground-truth establishment: creating their own dataset, or using an already existing dataset created by someone else. 

The first approach requires the researcher to build and label the dataset by himself. Both of these tasks are challenging. Building the dataset involves the selection of the applications to be included in the dataset. Data labeling is usually carried out by DPI tools given their presumably high accuracy. However, the actual accuracy of DPI-based tools is still not clear, as the previous works that tried to compare the performance of different DPI tools showed that the ground-truth used to validate the proposals was obtained through port-based classifiers, other DPI-based tools, or methodologies of unknown reliability [6, 7, 8, 9, 10]. Thus, the results of the comparison are arguable. In addition, most commercial tools are black boxes that claim high accuracy based on their own studies, which cannot be validated, because they were performed using private datasets. 

The second approach to the proper validation of traffic classification techniques is the use of publicly available datasets. This way, it is easier to compare the results obtained from different tools. Examples are the datasets published by the Co-operative Association for Internet Data Analysis (CAIDA) [11] or the Internet Measurement Data Catalog [12]. Although the datasets are pre-classified, the actual reliability of this labeling is unknown, which is a very important factor for testing traffic classifiers. Most of the traces have no payload or just the first bytes of each packet. The MAWI repository [13] contains various packet traces, including daily 15-minutes traces made at an trans-Pacific line (150 Mbit/s link). The bandwidth of this link has changed through the years. The traces contain the first 96 bytes of the payload and the traffic is usually asymmetric. Another useful data source is the Community Resource for Archiving Wireless Data At Dartmouth (CRAWDAD) [14], which stores wireless trace data from many contributing locations. Another interesting project is The Waikato Internet Traffic Storage (WITS) [15], which aims to collect and document all the Internet traces that the WAND Network Research Group from the University of Waikato has in their possession. Some of the traces can be freely downloaded and they contain traffic traces from various areas and of different types (as DSL residential traffic, university campus traffic, etc). Most of the traces do not have payload (i.e., it is zeroed) or truncated. All these datasets although useful for many network evaluations are of limited interest for DPI validation given that no correct labeling can be performed on them. 

Szabo et al. [16] introduced a method for validation of classification algorithms, which is independent of other classification methods, deterministic, and allows to automatize testing of large data sets. The authors developed a Windows XP driver based on the Network Driver Interface Specification (NDIS) library. Another approach to obtain the ground-truth was taken in [17]. The authors created a tool, which collects the data from the network and labels the flows with the real application names (e.g., Thunderbird) and application protocol names (e.g., SMTP). This tool is similar to our Volunteer-Based System (VBS), which was used in our work for the ground-truth generation and is further described in Section 3.1.1. Yet another way of establishing the ground-truth was shown in [18], which describes a system developed to accelerate the manual verification process. The authors proposed Ground Truth Verification System (GTVS) based on the DPI signatures derived from the databases available in the Internet, including L7-filter. GTVS, however, does not collect the application names from the operating systems, so the established truth cannot be completely verified. 

In a former conference paper [19], we tried to face the problem of ground-truth reliability. We described various methods of obtaining the ground-truth for testing various traffic classification tools. As part of the evaluation, we tested several DPI-based traffic classifiers and assessed if they can be used for obtaining reliable ground-truth. This paper is not only an extension but a more broad and comprehensive validation of DPI-based tools. The focus of this paper is different, as we directly compare the selected DPI tools regarding their per-class accuracy. Reference datasets used in this paper as well as the previous one are generated by the same methodology further explained in Section 3. However, both datasets are significantly different. The dataset used in this paper is significantly larger than the one used in the conference paper. Furthermore, the new dataset also contains labeled non-HTTP flows belonging to various web services, which is a unique feature. The methodology of testing the classifiers is also different. In our previous paper, we tested the accuracy of the classifiers on a single level. In the current paper, we evaluate different levels (i.e., application, web service, etc), so the new evaluation method is more detailed and complete.

This paper compares and validates six well-known DPI-based tools used for network traffic classification. In order to allow the validation of our work, we publish the reliable labeled dataset used to perform our study. Two main aspects have been carefully addressed when building this dataset: the reliability of the labeling and the representativeness of the data. We used the VBS tool [20] to guarantee the correctness of the labeling process. This tool, described in Section 3, is able to label the flows with the name of the process that creates them. This allowed us to create a reliable ground-truth that can be used as a reference benchmark for the research community to compare other proposals. The selection of applications for our dataset was made based on well-known indexes of the most commonly used Internet applications and web services. In order to allow the publication of the dataset and avoid any privacy issues, we created the traffic by running a large set of applications and meticulously simulating common behaviors of the users. 

The main contributions of this paper can be summarized as follows:

* We published a reliable labeled dataset with full packet payloads. The dataset, further described in Section 4, contains traffic from a diverse set of commonly used applications. Although artificially created, we carefully simulated human behaviors in order to produce a dataset that is as much realistic as possible. This dataset can be used by the research community to test and compare various traffic classifiers.

* We compared six well-known DPI-based tools widely used for network traffic classification. Using the previous dataset, we evaluated the precision of PACE, OpenDPI, nDPI, L7-filter, Libprotoident, and NBAR, and compared their results on various classification levels and in different scenarios. We are aware that OpenDPI and L7-filter are abandoned projects, on which development stopped several years ago. However, we decided to include them into our evaluation as a reference and for completeness, as many existing scientific papers base their results on these two classifiers.

* We provide an independent and impartial analysis of the performance of these tools at different classification levels. These results can help researchers to better understand the accuracy of the different tools used to set their ground truth as well as network managers to better decide which DPI-based classifiers are more suitable for their needs and scenarios. Indirectly, we also provide the information about the reliability of those non-DPI classification techniques proposed in the literature that used one of the DPI tools compared in this paper to set their ground truth.

In this paper, we focus on a single performance parameter: the classification accuracy. We acknowledge that other performance parameters are also important, such as speed, scalability, complexity, robustness, or price of the solutions. However, given that the research community is mainly using DPI-based technique for offline ground-truth generation, we think that those metrics are less decisive. Furthermore, we believe that our dataset will be useful to evaluate most of these parameters as well.

The remainder of this paper is organized as follows. Section 2 describes the DPI-based tools and their configurations used for the evaluation. Section 3 presents the methodology used to obtain the reliable dataset that is described in Section 4. Section 5 presents the results of the performance evaluation of the different DPI-based tools. Section 6 discusses the results, compares them with the literature, and comments the limitations of our evaluation. Section 7 reviews the related work. Finally, Section 8 concludes and summarizes the outcomes and contributions of the paper.

# 分类工具

市场上有许多可用的基于DPI的流量分类解决方案。对于我们的实验，我们选择了PACE，OpenDPI，nDPI，Libprotoident，NBAR和L7过滤器，这将在本节中广泛介绍。选择的依据是科学文献中使用的工具的流行程度，或基于市场上可用的操作系统中嵌入的工具来对流量进行分类或确定基本事实。我们特别评估了研究人员可以使用的那些工具，包括最受欢迎的开源工具，以及那些供应商接受免费与我们共享的商业工具。表1总结了这些基于DPI的工具的特点。

PACE。它是ipoque完全用C开发的专有分类库，它支持经典DPI（模式匹配）、行为、启发式和统计分析。根据其网站，PACE能够检测加密协议以及使用混淆的协议。总体而言，支持超过1000个应用程序和200个网络协议。还可以包括用户定义的用于检测应用程序和协议的规则。据我们所知，PACE是文献中用于建立ground truth的唯一商业工具[8]。

OpenDPI。它是从PACE的早期版本派生的开源分类器，它取消了对加密协议的支持以及所有性能优化。现在，该项目被视为已关闭。在[6，7]中，作者提到OpenDPI并不是经典的DPI工具，因为它使用了除模式匹配之外的其他技术（即行为和统计分析）。因此，它可以大大减少错误分类的数量，但是某些流量仍可以保持未分类状态[6]。另一个有趣的功能是流关联，它依赖于检查已知流的有效负载以发现新流，就像检查控制FTP会话以获得新启动的数据会话的五个元组一样[10]。

nDPI。它是一个OpenDPI分支，使用新协议进行了优化和扩展[21]。由于对会话证书的分析，它重新引入了对许多加密证书的支持。总体而言，nDPI现在支持100多种协议[22]。当前的体系结构是可伸缩的，但是不能提供最佳的性能和结果：每个协议都有自己的签名扫描器，通过该扫描器可以检查数据包。不论是否找到匹配项，每个扫描器都会检查每个数据包。如果每个流有多个匹配项，则返回的值是最详细的值[10]。另外，没有TCP或IP有效载荷的重组，因此不可能检测到被分成多个TCP段/ IP数据包的签名[22]。

Libprotoident。这个C库[8]引入了轻量数据包检查（LPI），它仅检查每个方向上有效载荷的前四个字节。这样可以最大程度地减少隐私问题，同时减少存储分类所需的数据包跟踪所需的磁盘空间。 Libprotoident支持200多种不同的协议，分类基于使用有效负载模式匹配，有效负载大小，端口号和IP匹配的组合方法。

思科基于网络的应用程序识别（NBAR）。开发它是为了增加使用现有基础设施对网络流量进行分类的能力[23]。它能够对使用动态TCP和UDP端口号的应用程序进行分类。由于设备（例如路由器）可以为特定的应用程序动态分配一定数量的带宽，丢弃数据包或以选定的方式对其进行标记，因此NBAR具有服务质量（QoS）功能。作者声称NBAR支持广泛的状态协议，这些协议很难分类。

L7过滤器。它创建于2003年，是Linux Netfilter的分类器，可以识别应用程序层的流量[24]。分类基于三种技术。首先，基于标准iptables模块的简单数字识别，可以处理端口号，IP协议号，传输的字节数等。第二，基于正则表达式的有效负载模式匹配。第三，可以基于功能识别应用程序。 L7过滤器是作为一组规则和分类引擎开发的，可以彼此独立使用。 L7过滤器分类引擎的最新版本是2011年1月，而分类规则是2009年。

On the market, there are many available DPI-based traffic classification solutions. For our experiment, we selected PACE, OpenDPI, nDPI, Libprotoident, NBAR, and L7-filter, which will be broadly introduced in this section. The selection was based on the popularity of the tools used in the scientific literature or embedded in the operating systems available on the market to classify the traffic or set the ground truth. In particular, we evaluated those tools available to researchers, including most popular open source tools, as well as those commercial tools whose vendors accepted to share with us free of charge. Table 1 summarizes these DPI-based tools along their characteristics.

PACE. It is a proprietary classification library developed by ipoque entirely in C, which supports classical DPI (pattern matching), behavioral, heuristic, and statistical analysis. According to its website, PACE is able to detect encrypted protocols as well as protocols which use obfuscation. Overall, more than 1000 applications and 200 network protocols are supported. It is also possible to include user-defined rules for de- tection of applications and protocols. To the best of our knowledge, PACE is the only commercial tool used in the literature to build the ground truth [8].

OpenDPI. It was an open-source classifier derived from early versions of PACE by removing support for encrypted protocols, as well as all performance optimizations. The project is now considered as closed. In [6, 7], the authors mention that OpenDPI is not a classic DPI tool, as it uses other techniques apart from pattern matching (i.e., behavioral and statistical analysis). Thanks to that, it should severely reduce the amount of false classification, but some traffic can remain unclassified [6]. Another interesting feature is flow association, which relies on inspecting the payload of a known flow to discover a new flow, as inspecting a control FTP session to obtain the five tuple of the newly initiated data session [10].

nDPI. It is an OpenDPI fork, which is optimized and extended with new protocols [21]. It re-introduced support for many encrypted ones due to analysis of session certificates. Overall, nDPI for now supports more than 100 protocols [22]. The current architecture is scalable, but it does not provide the best performance and results: each of the protocols has its own signature scanner, through which the packets are examined. Every packet is examined by each scanner, regardless, if a match was found. If there are multiple matches per flow, the returned value is the most detailed one [10]. Additionally, there is no TCP or IP payload re-assembly, so there is no possibility to detect a signature split into multiple TCP segments / IP packets [22].

Libprotoident. This C library [8] introduces Lightweight Packet Inspection (LPI), which examines only the first four bytes of payload in each direction. That allows to minimize privacy concerns, while decreasing the disk space needed to store the packet traces necessary for the classification. Libprotoident supports over 200 different protocols and the classification is based on a combined approach using payload pattern matching, payload size, port numbers, and IP matching.

Cisco Network Based Application Recognition (NBAR). It was developed to add the ability to classify the network traffic by using the existing infrastructure [23]. It is able to classify applications, which use dynamic TCP and UDP port numbers. NBAR works with Quality of Service (QoS) features, thanks to what the devices (e.g., routers) can dynamically assign a certain amount of bandwidth to a particular application, drop packets, or mark them in a selected way. The authors claim that NBAR supports a wide range of stateful protocols, which are difficult to classify.

L7-filter. It was created in 2003 as a classifier for Linux Netfilter, which can recognize the traffic on the application layer [24]. The classification is based on three techniques. At first, simple numerical identification based on the standard iptables modules, which can handle port numbers, IP protocol numbers, number of transferred bytes, etc. At second, payload pattern matching based on regular expressions. At third, the applications can be recognized based on functions. L7-filter is developed as a set of rules and a classification engine, which can be used independently of each other. The most recent version of L7-filter classification engine is from January, 2011, and the classification rules from 2009.


# Methodology
我们的实验涉及许多步骤，这些步骤将在本节中定义和描述。 我们有两个主要目标–建立数据集和测试分类器。他们每个人都需要一种单独的方法。
3.1数据集的建立

3.1.1试验台
我们的测试台由7台机器组成，用于运行选定的应用程序并生成通信数据，还包括一台服务器。我们为数据生成计算机配备了Windows 7（3台计算机），Ubuntu（3台计算机）和Windows XP（1台计算机）。另一台Ubuntu服务器计算机配备了MySQL数据库用于数据存储。

为了收集和准确标记流量，我们采用了奥尔堡大学开发的自愿者系统（VBS）[25]。 VBS项目的目标是从Internet流量中收集流量级别的信息（例如，流量的开始时间，该流量包含的数据包数量，本地和远程IP地址，本地和远程端口，传输层协议）以及有关每个数据包的详细信息（例如，方向，大小，TCP标志以及流中前一个数据包的相对时间戳）。
对于每个流，系统都会收集与之关联的进程名称，该名称是从系统套接字获得的。另外，系统收集有关所传输的HTTP内容类型的一些信息（例如text / html，video / x-flv）。捕获的信息将传输到VBS服务器，该服务器将数据存储在MySQL数据库中。

在每台数据生成机上，我们都安装了VBS的修改版。原始系统的源代码以及修改后的版本可在[26]中的GNU通用公共许可证v3.0中公开获得。除上述数据外，VBS客户端的修改版本还捕获每个数据包的完整以太网帧，并提取HTTP URL和Referrer字段-所有信息均传输到服务器，并存储在MySQL数据库中。我们添加了一个名为pcapBuilder的模块，该模块负责将数据包从数据库转储到PCAP文件。同时，将生成INFO文件以提供有关每个流的详细信息，这使我们可以将PCAP文件中的每个数据包分配给一个单独的流。

Our experiment involved numerous steps, which will be defined and described in this section. We had two main goals –building the dataset and testing the classifiers. Each of them required an individual methodology. 

3.1. Building of the dataset 

3.1.1. The testbed 

Our testbed consisted of 7 machines, which were used for running the selected applications and generating the traffic data, and of a server. We equipped the data generating machines with Windows 7 (3 machines), Ubuntu (3 machines), and Windows XP (1 machine). The additional Ubuntu server machine was equipped with a MySQL database for data storage. 

To collect and accurately label the flows, we adapted the Volunteer-Based System (VBS) developed at Aalborg University [25]. The goal of the VBS project is to collect flow-level information from the Internet traffic (e.g., start time of the flow, number of packets contained by the flow, local and remote IP addresses, local and remote ports, transport layer protocol) together with detailed information about each packet (e.g., direction, size, TCP flags, and relative timestamp to the previous packet in the flow). For each flow, the system collects the process name associated with it, which is obtained from the system sockets. Additionally, the system collects some information about the types of the transferred HTTP contents (e.g., text/html, video/x-flv). The captured information is transmitted to the VBS server, which stores the data in a MySQL database. 

On every data generating machine, we installed a modified version of VBS. The source code of the original system as well as the modified version is publicly available in [26] under GNU General Public License v3.0. The modified version of the VBS client captures, apart from the data described above, full Ethernet frames for each packet, and extracts the HTTP URL and Referrer fields – all the information is transmitted to the server, and stored in the MySQL database. We added a module called pcapBuilder, which is responsible for dumping the packets from the database to PCAP files. At the same time, INFO files are generated to provide detailed information about each flow, which allows us to assign each packet from the PCAP file to an individual flow.

3.1.2 数据选择

建立代表典型用户行为特征的代表性数据集的过程是一项艰巨的任务，从测试和比较不同的流量分类器的角度来看至关重要。因此，为了确保所包含数据的正确多样性和数量，我们决定在多维级别上组合数据。根据w3schools的统计数据[27]，我们发现大多数PC用户使用的是Windows 7（占用户的56.7％），Windows XP（占12.4％），Windows 8（占9.9％）和Linux（占4.9％）的状态-2013年10月。苹果计算机占总流量的9.6％，移动设备占总流量的3.3％。由于缺少用于Apple计算机，Windows 8和移动设备的设备和/或软件，我们决定在研究中包括Windows 7（W7），Windows XP（XP）和Linux（LX），这些内容现已涵盖已使用操作系统的74.0％。

下面显示了为该研究选择的应用程序协议，应用程序和Web服务。为了对它们进行分组，我们采用了Palo Alto [28]的报告中使用的相同分类类别。

1. 文件共享应用程序。根据[28]，它们占总带宽的6％。在该组中，BitTorrent占53％，FTP占21％，Dropbox占5％，Xunlei占4％，eMule占3％。根据[28]中的统计数据，以及CNET [29]和OPSWAT P2P客户端受欢迎程度列表，CNET FTP客户端受欢迎程度列表[30]和直接下载受欢迎程度列表[31]中的统计信息，我们选择了以下应用程序：

* BitTorrent：uTorrent（Windows），kTorrent（Linux）。
* eDonkey：eMule（Windows），aMule（Linux）。
所研究的配置为：outgoing-non-obfuscated-incoming-all，全部混淆。
* FTP：处于主动模式（PORT）和被动模式（PASV）的FileZilla（Windows，Linux）。
* Dropbox（Windows，Linux）。
* 基于Web的直接下载：4Shared（+ Windows应用程序），MediaFire，Putlocker。
* Webdav（Windows）。

2. 影音组。根据Palo Alto的报告[28]，它们占总带宽的16％，其中YouTube占总带宽的6％，Netflix占总带宽的2％，其他HTTP视频占总带宽的2％，RTMP占2％占总数的4％。为了选择该类别的应用，我们还使用了视频网站的Ebizmba排名[32]。

* YouTube：根据全球排名[33]以来观看次数最多的视频。
* RTMP：从Justin.tv观看了大约30个随机的短实时视频流（1-10分钟）。
* Vimeo –基于Web的照片共享解决方案。
* PPStream（Windows）– P2P流视频软件。
* 其他HTTP视频。

3. Web浏览流量。根据w3schools的统计[34]，最受欢迎的Web浏览器是：Chrome（占用户的48.4％），Firefox（占30.2％）和Internet Explorer（占14.3％）。这些浏览器用于生成网络流量。根据Palo Alto的报告[28]，它们占总带宽的20％。网站的选择基于Alexa统计信息[35]，Ebizmba网站统计信息[36]，Quantcast统计信息[37]和Ebizmba搜索引擎的流行度[38]。为了使数据集尽可能具有代表性，我们在使用这些网站时模拟了不同的人类行为。例如，在Facebook上，我们登录，与朋友互动（例如，聊天，发送消息，在他们的墙上写字），上传图片，创建活动或玩游戏。对于其他流行的Web服务（例如Twitter，Google +，eBay等），也模拟了类似的行为。使用这些服务执行的操作的详细说明在我们的技术报告中列出[39]。
4. 加密的隧道流量。根据Palo Alto的报告[28]，它们占总带宽的9％，其中SSL占6％，SSH占2％。 
* SSL（Windows，Linux）：在使用各种应用程序和Web服务时收集。
* SSH（Linux）。
* TOR（Windows）。首先，我们使用TOR浏览各种网站并下载大文件。然后，我们将TOR配置为充当内部中继，因此我们参与了为其他用户创建不可见路径的工作。
* Freenet（Windows）：用于浏览Intranet并作为其他对等方的中继。
* SOCKSv5（Windows）。我们在Linux上创建了SOCKSv5服务器，并将其用于Firefox和uTorrent。

5. 存储备份流量。根据Palo Alto [28]的说法，它们占总带宽的16％，其中至少一半的带宽由MS-SMB占用，其余的则由许多不同的应用程序占用。因此，唯一经过测试的应用程序是MS-SMB（Windows，Linux）。

6. 电子邮件和通讯流量。根据Palo Alto的报告[28]，电子邮件流量占总带宽的3％。 2013年10月的电子邮件市场份额[40]显示，在使用的邮件客户端的前10名中，只有一个桌面邮件客户端Microsoft Outlook（17％）。其余部分分为基于Web的客户端（如GMail）和移动客户端（Mac，Android）。经过测试的应用程序/基于Web的邮件服务包括：Gmail，Hotmail，Windows Live Mail（Windows）和Mozilla Thunderbird（Windows）。测试了桌面电子邮件应用程序（Windows Live Mail和Mozilla Thunderbird）以使用各种协议：SMTP-PLAIN（端口587），SMTP-TLS（端口465），POP3-PLAIN（端口110），POP3TLS（端口995） ，IMAP-STARTTLS（端口143）和IMAP-TLS（端口993）。我们还测试了Windows和Android操作系统之间的Skype：视频会话，语音会话和文件传输。

7. 管理流量。 DNS，ICMP，NETBIOS，NTP，RDP。
8. 游戏。根据DFC Intelligence [41]根据美国最受欢迎的在线游戏，我们选择了：
* 英雄联盟（Windows）–所有启动器。
* 魔兽世界（Windows）–包括所有启动器。
* Pando Media Booster（Windows）–由英雄联盟添加的将游戏安装程序播种到其他用户的过程，由于下载是在P2P模式下进行的，因此可以减轻服务器的负担。它产生大量的流量帐户并填充连接。
* Steam –直接在计算机桌面上提供各种游戏。包括自动更新，游戏和价格列表，海报，以及访问大量游戏的权限。我们将Steam列入名单，因为它是众多游戏的平台。
* 美国陆军-Steam推出的热门游戏。
9.其他。此类别包括：
* 音乐应用程序：Spotify，iTunes（Windows）。
* P2P互联网电视：PPLive，Sopcast（Windows）。

3.1.2. Selection of the data

The process of building a representative dataset, which characterizes a typical user behavior, is a challenging task, crucial from the point of testing and comparing different traffic classifiers. Therefore, to ensure the proper diversity and amount of the included data, we decided to combine the data on a multidimensional level. Based on w3schools statistics [27], we found that most PC users use Windows 7 (56.7 % of users), Windows XP (12.4 %), Windows 8 (9.9 %), and Linux (4.9 %) - state for October 2013. Apple computers contribute for 9.6 % of the overall traffic, and mobile devices for 3.3 %. Because of the lack of the equipment and/or software for Apple computers, Windows 8, and mobile devices, we decided to include in our study Windows 7 (W7), Windows XP (XP), and Linux (LX), which cover now 74.0 % of the used operating systems. 

The application protocols, applications, and web services selected for this study are shown below. To group them, we adopted the same classification categories used in the reports from Palo Alto [28].

1. File-sharing applications. According to [28], they account for 6 % of the total bandwidth. Inside that group, BitTorrent accounts for 53 %, FTP for 21 %, Dropbox for 5 %, Xunlei for 4 %, and eMule for 3 %. Based on the statistics found in [28], as well as those in the CNET [29] and the OPSWAT P2P clients popularity list, the CNET FTP clients popularity list [30], and the Direct Download popularity list [31], we selected the following applications:

* BitTorrent: uTorrent (Windows), kTorrent (Linux).
* eDonkey: eMule (Windows), aMule (Linux).
The studied configurations were: outgoing-non-obfuscated-incoming-all, all-obfuscated.
* FTP: FileZilla (Windows, Linux) in active mode (PORT) and passive mode (PASV).
* Dropbox (Windows, Linux).
* Web-based direct downloads: 4Shared (+ Windows app), MediaFire, Putlocker.
* Webdav (Windows).

2. Photo-video group. According to the reports from Palo Alto [28], they account for 16 % of the total bandwidth, where YouTube accounts for 6 % of total, Netflix for 2 % of total, other HTTP video for 2 % of total, RTMP for 2 % of total, and others for 4 % of traffic in total. To select the applications in this category we also used the Ebizmba ranking of video websites [32].

* YouTube: most watched videos from all the times according to the global ranking [33].
* RTMP: around 30 random short live video streams (1–10 minutes) were watched from Justin.tv.
* Vimeo – a web-based photo sharing solution.
* PPStream (Windows) – P2P streaming video software.
* Other HTTP video.

3. Web browsing traffic. Based on w3schools statistics [34], the most popular web browsers are: Chrome (48.4 % of users), Firefox (30.2 %), and Internet Explorer (14.3 %). These browsers were used to generate the web traffic. According to the reports from Palo Alto [28], they account for 20 % of the total bandwidth. The selection of the websites was based on Alexa statistics [35], Ebizmba web statistics [36], Quantcast statistics [37], and Ebizmba search engines popularity [38]. In order to make the dataset as representative as possible, we simulated different human behaviors when using these websites. For instance, on Facebook, we log in, interact with friends (e.g., chat, send messages, write in their walls), upload pictures, create events or play games. Similar behaviors were simulated for other popular web services, such as Twitter, Google+, eBay, etc. The detailed description of actions performed with the services is listed in our technical report [39].


4. Encrypted tunnel traffic. According to the reports from Palo Alto [28], they account for 9 % of the total bandwidth, where 6 % of total is SSL and 2 % of total is SSH. • SSL (Windows, Linux): collected while using various applications and web services.
* SSH (Linux).
* TOR (Windows). First, we used TOR to browse various websites and download big files. Then, we configured TOR to act as an internal relay, so we participated in creating the invisible path for other users.
* Freenet (Windows): for browsing the intranet and also as a relay for other peers.
* SOCKSv5 (Windows). We created a SOCKSv5 server on Linux, and used it for Firefox and uTorrent.

5. Storage-backup traffic. According to Palo Alto [28], they account for 16 % of the total bandwidth, where at least half of the bandwidth is consumed by MS-SMB, and the rest by many different applications. Therefore, the only tested application was MS-SMB (Windows, Linux).

6. E-mail and communication traffic. According to the reports from Palo Alto [28], e-mail traffic accounts for 3 % of the total bandwidth. E-mail market share from October 2013 [40] shows that only one desktop mail client, Microsoft Outlook (17 %), is in the top 10 of used mail clients. The rest is split between web-based clients (as GMail) and mobile clients (Mac, Android). The tested applications / web-based mail services include: Gmail, Hotmail, Windows Live Mail (Windows), and Mozilla Thunderbird (Windows). The desktop e-mail applications (Windows Live Mail and Mozilla Thunderbird) were tested to use various protocols: SMTP-PLAIN (port 587), SMTP-TLS (port 465), POP3-PLAIN (port 110), POP3TLS (port 995), IMAP-STARTTLS (port 143), and IMAP-TLS (port 993). We also tested Skype between Windows and Android OS: video sessions, voice conversations, and file transfers.

7. Management traffic. DNS, ICMP, NETBIOS, NTP, RDP.
8. Games. Based on the most played online games in USA according to DFC Intelligence [41], we selected:
* League of Legends (Windows) – with all launchers.
* World of Warcraft (Windows) – including all launchers.
* Pando Media Booster (Windows) – a process added by League of Legends to seed the game installer to other users, which offloads the servers, because the download is performed in the P2P mode. It generates enormous accounts of traffic and fills the connection.
* Steam – delivers a range of games straight to a computer’s desktop. Includes automatic updates, lists of games and prices, posters, plus access to a large number of games. We included Steam on the list as it is a platform for numerous games.
* America’s Army – a popular game from Steam.

9. Others. This category includes:
* Music applications: Spotify, iTunes (Windows).
* P2P Internet TVs: PPLive, Sopcast (Windows).

3.2 DPI工具的测试

测试不同DPI工具的过程非常复杂，因此，我们将其分为几个部分：数据标记，分类过程和分类日志分析。对于某些DPI，某些步骤可能与其他DPI有所不同–在这些情况下，差异会明确显示出来。我们仅评估一个性能参数：分类准确性。我们承认，其他性能参数也很重要，例如速度、可伸缩性、复杂性、健壮性或解决方案的价格，但是，对它们的评估不在本文讨论范围之内。

3.2.1数据标签

需要通过将应用程序，应用程序协议，Web服务，内容类型或Internet域的标签附加到数据库上来正确标记数据库中存储的所有流。一个流可以与多个标签关联。未标记的流在提取到PCAP文件时不会被考虑。我们将流量分为不同级别。我们通过标识Web流并将其分配给选定的Web服务来开始标记过程。每个Web服务都由一组域标识。根据域在收集的HTTP流中出现的次数来选择域。仅当HTTP流包含来自匹配域的流量时，才用Web服务标签标记。如果流包含来自属于多个服务的域（或属于未分配给所选服务的域）的流量，则该流将保持未标记状态。如果HTTP流传输音频或视频，则它们也用传输内容的类型的标签（例如video / x-flv）进行标记。不属于HTTP的Web服务（例如SSL）流将使用启发式方法进行标记，如下所示。要被识别为非HTTP Web流，与该流关联的应用程序名称应为Web浏览器的名称（例如chrome），Web浏览器插件的名称（例如plugin-container，flashgc-play） ，否则名称应该丢失。然后，我们看一下HTTP流，这些流是从非HTTP流之前的2分钟到之后的2分钟。如果所有对应的（即，源自同一本地计算机并到达同一远程主机的）HTTP流都分配了一个Web服务标签，并且所有流的服务标签都相同，则将非HTTP流分类为相同的Web服务标签。之后，我们确定应用程序协议。应用协议标签仅应用于我们确定为其传输特定应用协议的流。最后，我们确定应用程序。我们分别考虑应用程序和协议是因为，例如，Web浏览器可能会使用HTTP之外的许多其他协议，或者BitTorrent客户端可以连接到网站以使用HTTP或SSL等下载文件。

3.2.2分类过程

数据包以3种不同的模式提取到PCAP文件中：正常模式，被截断的数据包（即以太网帧在第70个字节后被0覆盖）和被截断的流（每个流仅提取10个第一个数据包）。我们设计了一个名为dpi_benchmark的工具，该工具能够读取PCAP文件并将数据包一一提供给PACE，OpenDPI，L7过滤器，nDPI和Libprotoident。基于包含时间戳的INFO文件中的信息来启动和终止所有流。在流的最后一个数据包发送到分类器之后，该工具将获得与该流关联的标签。标签与流标识符一起写入日志文件，这使我们以后能够将分类结果与数据库中的原始流相关联。

我们使用所有分类器的默认配置，但L7过滤器除外，后者在两种不同的配置中进行了评估。第一个版本（L7-filter-all）激活了所有模式，但是其作者标记为不匹配的模式具有较低的优先级。对于第二个版本（L7-filter-com），我们采用了[42]中提出的方法，该方法不会激活声明为过度匹配的模式，并且会修改默认模式优先级。

NBAR进行分类需要构建完整的工作环境。我们没有任何可用于实验的Cisco设备。因此，我们使用了GNS3 –一个图形框架，该框架使用Dynamips来模拟我们的Cisco硬件。我们模拟了7200平台，因为这是GNS3支持的唯一可以运行最新版本的Cisco IOS（版本15）的平台，该版本包含Flexible NetFlow。以前版本的Cisco IOS仅包含传统的NetFlow，不支持按流进行NBAR报告。我们使用虚拟接口将虚拟路由器连接到真实计算机。路由器被配置为在创建的接口上使用带有NBAR的Flexible NetFlow。

Flexible NetFlow识别的每个流都由从NBAR获得的应用程序名称标记。在计算机上，我们使用tcpreplay将PCAP文件以不导致数据包丢失的最大速度重播到路由器。同时，我们使用了nfacctd（它是PMACCT工具的一部分[43]）来捕获路由器发送到计算机的Flexible NetFlow记录。

处理存储在分类日志中的数据并将其导入回数据库。我们使用每个流记录所包含的流标识符将日志记录与数据库中的适当流进行匹配。 NBAR依赖于Flexible NetFlow，它以单向方式处理流。这意味着我们需要基于2个单向流（入站和出站）评估双向流的类型。在大多数情况下，两个单向流的标签是相同的。在少数情况下，仅存在入站或出站流，因为没有数据包朝相反的方向流动。如果两个单向流都存在并且每个标签的标签不同，则双向流从占更多字节的单向流中获取标签。

3.3 结果分析

结果分析的方法取决于标记流的分类级别：

* 应用协议级别，例如DNS，HTTP或POP3：

为了将分类视为正确，分类器报告的标签必须是应用协议（例如DNS，HTTP），但不能处于其他级别（例如FLASH，YOUTUBE）。这对于测试工具是否可以识别特定的应用程序协议很有用。如果在不同级别给出结果，则该流被视为未分类。但是，例如，当我们寻找名为YouTube的网络服务时，在不同级别的其他测试中，同一流程将被归类为正确。带有标签的流属于不同的应用程序协议，不属于该应用程序协议的服务和应用程序被认为是错误的。
* Web服务级别，例如Yahoo或YouTube：只有给出了Web服务的名称，该分类才被认为是正确的。如果以其他级别（例如HTTP或FLASH）给出，则该流被视为未分类。
* 应用程序级别（当应用程序使用其专有的应用程序级别协议时）。例如，uTorrent和Skype应用程序可以使用多种协议，包括分别称为Skype和BitTorrent的专有协议以及其他协议，例如HTTP或SSL。例如，HTTP和SSL可用于连接到网络服务器以下载用户的数据或广告。因此，对于Skype，DPI工具标记为Skype，HTTP，SSL的流都标记为正确。
* 应用程序级别（当应用程序不使用其专有的应用程序级别协议，而直接使用HTTP，SSL等时）。它涉及例如Spotify。然后，仅认为标记为Spotify的流被正确标记，因为该应用程序不存在特定的应用程序级协议，因此我们希望可以识别应用程序名称本身。

由于有了这种多级测试方法，我们获得了关于哪个分类器能够在每个特定分类级别上提供结果的知识。这种知识将使最终用户能够根据所需的分类级别调整DPI技术的选择。

3.2. Testing of the DPI tools

The process of testing different DPI tools is complex and, therefore, we split it into several parts: labeling of the data, the classification process, and analysis of the classification logs. Some of the steps can be different for some DPIs than for the others – in these cases the differences are explicitly highlighted. We evaluate only one performance parameter: the classification accuracy. We acknowledge that other performance parameters are also important, such as speed, scalability, complexity, robustness, or price of the solutions, however, their evaluation is outside the scope of this paper.

3.2.1. Labeling of the data

All the flows stored in the database need to be properly marked by attaching to them the labels of the applications, application protocols, web services, types of the content, or Internet domains. One flow can be associated with multiple labels. Flows, which are not labeled, are not taken into consideration while extracting them to PCAP files. We classify the flows at different levels. We start the labeling process by identifying the web flows and assigning them to the selected web services. Every web service is identified by a set of domains. The domains were chosen based on the number of their occurrences in the collected HTTP flows. The HTTP flows are marked with a web service label only if they contain the traffic from the matching domains. In case the flow contains traffic from domains belonging to multiple services (or to domains, which are not assigned to the selected services), the flow is left as unlabeled. The HTTP flows are also marked with the labels of the type of the transmitted content (e.g., video/x-flv), if they transmit audio or video. Those flows that are not HTTP belonging to the web services (e.g., SSL) are labeled using an heuristic method as follows. To be recognized as a non-HTTP web flow, the application name associated with the flow should be the name of a web browser (e.g., chrome), a name of a web browser plugin (e.g., plugin-container, flashgc-play), or the name should be missing. Then, we look at the HTTP flows, which were originated from 2 minutes before to 2 minutes after the non-HTTP flow. If all the corresponding (i.e., originated from the same local machine and reaching the same remote host) HTTP flows have a web service label assigned, and the service label is the same for all of the flows, the non-HTTP flow is classified with the same web service label. Afterwards, we identify the application protocols. The application protocol label is applied only to those flows for which we are sure that transmit the specific application protocol. Finally, we identify the applications. We consider applications and protocols individually because, for example, a web-browser may use many different protocols besides HTTP, or a BitTorrent client can connect to websites to download files using HTTP or SSL, etc.

3.2.2. The classification process

The packets were extracted into PCAP files in 3 different modes: the normal one, with truncated packets (i.e., Ethernet frames were overwritten by 0s after the 70th byte), and with truncated flows (we extracted only 10 first packets for each flow). We designed a tool, called dpi_benchmark, which is able to read the PCAP files and provide the packets one-by-one to PACE, OpenDPI, L7-filter, nDPI, and Libprotoident. All the flows are started and terminated based on the information from the INFO files, which contain the timestamps. After the last packet of the flow is sent to the classifier, the tool obtains the label associated with that flow. The labels are written to the log files together with the flow identifier, which makes us later able to relate the classification results to the original flows in the database.

We used the default configurations of all classifiers except for L7-filter, which was evaluated in two different configurations. The first version (L7-filter-all) had all the patterns activated , but the patterns marked as overmatching by their authors have a low priority. For the second version (L7-filter-com) we adapted the methodology proposed in [42], which does not activate the patterns declared as overmatching and the default pattern priorities were modified.

Classification by NBAR required to build a complete working environment. We did not have any Cisco device that could be used for the experiment. Therefore, we used GNS3 – a graphical framework, which uses Dynamips to emulate our Cisco hardware. We emulated 7200 platform, since this is the only platform supported by GNS3 that can run the newest version of Cisco IOS (version 15), which contains Flexible NetFlow. Previous versions of Cisco IOS contain only traditional NetFlow, which does not support NBAR reporting on the per flow basis. We connected the virtual router to a real computer by using a virtual interface. The router was configured to use Flexible NetFlow with NBAR on the created interface.

Every flow recognized by Flexible NetFlow was tagged by the application name obtained from NBAR. On the computer,we used tcpreplay to replay the PCAP files to the router with the maximal speed that did not cause packet loss. At the same time, we used nfacctd, which is part of PMACCT tools [43], to capture the Flexible NetFlow records sent by the router to the computer.

The data stored in the classification logs was processed and imported back to the database. We matched the log records to the proper flows in the database using the flow identifier contained by each flow record. NBAR relies on Flexible NetFlow, which treats the flows in a unidirectional way. It means that we needed to assess the type of the bi-directional flow based on 2 unidirectional flows (inbound and outbound). In most of the cases the label from both unidirectional flows were the same. In a few cases there was only an inbound or an outbound flow, since there were no packets going in the opposite direction. In case, both unidirectional flows existed and the label of each of them was different, the bidirectional flow got the label from the unidirectional flow that accounted for more bytes.


3.3. Analysis of the results

The method for analysis of the results depend on the level of classification on which the flows were labeled:

* Application protocol level, such as DNS, HTTP, or POP3:

To consider the classification as correct, the label reported by the classifier must be an application protocol (e.g., DNS, HTTP), but not at a different level (e.g., FLASH, YOUTUBE). This is useful to test if the tool can recognize the specific application protocol. If the result is given at a different level, the flow is considered as unclassified. However, the same flow will be classified as correct during other tests at different levels, when we for example look for a web service called YouTube. Those flows with labels belonging to different application protocols, and services and applications that do not belong to this application protocol are considered as wrong. 
* Web service level, such as Yahoo or YouTube: the classification is considered to be correct only if the name of the web service is given. If it is given at a different level, such as HTTP or FLASH, the flow is considered as unclassified. 
* Application level (when the application uses its proprietary application-level protocols). For example, uTorrent and Skype applications can use multiple protocols, including their proprietary protocols called respectively Skype and BitTorrent, and other protocols, such as HTTP or SSL. For example, HTTP and SSL can be used to connect to the web server to download the user’s data or advertisements. Therefore, in the case of Skype, flows labeled by DPI tools as Skype, HTTP, SSL are all marked as correct.
* Application level (when the application does not use its proprietary application-level protocols, but directly uses HTTP, SSL, etc.) It concerns for example Spotify. Then, only the flows marked as Spotify are considered to be labeled correctly, as no specific application-level protocol exists for this application, so we expect the application name itself to be identified.

Thanks to this multilevel testing approach, we obtained the knowledge of which classifier is able to provide results on each particular level of classification. This knowledge would allow, for example, the end user to adjust the choice of the DPI technique according to the desired level of classification.


4.数据集

我们的基本数据集（没有截断的数据包或数据流）包含767 690个数据流，占53.31 GB的纯数据包数据。 存在759 720个流的应用程序名称（占所有流的98.96％），占数据量的51.93 GB（97.41％）。 剩余的流由于寿命短（通常低于1 s）而没有标签，这使得VBS无法可靠地建立相应的套接字。 表2中显示了应用程序协议以及流的数量和数据量，表3中显示了应用程序，表4中显示了Web服务。此处显示的数据量仅用于概述–其余部分 本文仅使用流量数作为参考值。 我们将在我们的网站上发布带有完整数据包有效负载的基本标记数据集[44]。 因此，它可以被研究团体用作验证网络流量分类器的参考基准。

4. Dataset

Our basic dataset (without truncated packets or flows) contains 767 690 flows, which account for 53.31 GB of pure packet data. The application name was present for 759 720 flows (98.96 % of all the flows), which account for 51.93 GB (97.41 %) of the data volume. The remaining flows are unlabeled due to their short lifetime (usually below 1 s), which made VBS incapable of reliably establishing the corresponding sockets. The application protocols together with the number of flows and the data volume are shown in Table 2, while the applications are shown in Table 3 and the web services are shown in Table 4. The data volume is presented here only for an overview – the rest of the paper uses only the number of flows as the reference value. We are going to publish our basic labeled dataset with full packet payloads on our website [44]. Therefore, it can be used by the research community as a reference benchmark for the validation of network traffic classifiers.

5.结果
本节概述了每个分类器对不同类型流量的分类结果。对3个数据集执行了评估：正常集，具有截短数据包的集合和具有截断流的集合。本节介绍了最有趣的结果，但稍后在第6节中进行讨论，并从中得出结论。以下小节介绍了常规数据集的结果，而包含截短的数据包或数据流的集合的结果将单独讨论。所有精度结果均以流量给出。第3.3节显示了用于计算精度的方法。有兴趣的读者可以在技术报告中找到完整的混淆矩阵[39]。

5.1。应用协议
表5中显示了对应用协议分类的评估。各列显示了每个分类器正确分类和错误分类以及属于各种应用协议的未分类流的百分比。

基于DPI的技术的重要性能参数是其结果的完整性（即，可以分类的应用程序数量）。本节评估17种不同的应用程序协议。如表5所示，没有一种技术能够对所有这些技术进行分类。在研究的不同技术中，nDPI和Libprotoident最完整，在17种中分类了15种。在远端，L7过滤器仅在17种中分类了9种。

DPI技术的另一个重要方面是其误报率（即错误分类）。通常，技术会将无法识别的流程留为未分类，以尝试减少误报的数量。即使这两种版本的L7-filter都具有产生大量错误分类的特征（例如，L7-filter-all将HTTP流量的85.79％分类为Finger）。关于特定分类，大多数传统应用程序协议（例如，DNS，HTTP，IMAP-STARTTLS，POP3-PLAIN，SMTP-PLAIN和SSH）通常都可以通过所有技术很好地检测到（例如，准确度在70.92％和100％之间） 。出乎意料的是，Libprotoident是唯一能够识别所有经过测试的加密协议的分类器。与分类器无关，未检测到的加密流量通常被标识为常规SSL。 RTMP的分类提供了一个有趣的案例。只有nDPI和Libprotoident能够对其正确分类。 PACE和OpenDPI将此流量分类为Flash。尽管通常两种流量都相关，但由于Flash只是内容容器，因此分类为Flash不能被认为是正确的。可以使用各种应用协议（例如HTTP，RTMP）甚至不同的传输协议（TCP和UDP）来传输Flash内容（音频，视频或任何其他二进制文件）。

5.2应用领域

第二级分类研究使用其专有应用程序级协议（例如BitTorrent，Skype）的应用程序。表6中显示了对各种应用程序分类的评估。各列显示了每个分类器正确和错误分类的百分比以及属于各种应用程序的未分类流的百分比。

在应用程序级别，最完整的技术是PACE，在22个评估的应用程序中对20个进行了分类，其次是nDPI（17）和Libprotoident（14）。同样，L7过滤器是最差的技术之一（8），但NBAR仅对4种应用程序进行了分类，克服了它。

关于误报率，从PACE和OpenDPI获得了错误分类流的最低百分比。相反，两种版本的L7过滤器又从分类中获得了错误分类流量的最高比例（例如，美国陆军流量的97％被归类为Skype和RTP）。

如表6所示，与应用程序协议级别相比，应用程序级别的分类提出了更多的问题。尤其令人惊讶的是，几乎没有分类器能够完全分类来自特定应用程序的所有流（即100％）。这可以从以下事实得出：通常，应用程序使用产生不同流量的不同内部操作。因此，技术需要针对每种类型的操作的特定模式。例如，Skype的准确性始终低于100％，因为这些技术都无法对Skype文件传输和视频进行分类。在不同的研究技术中，PACE是最准确的，其次是nDPI和Libprotoident。令人惊讶的是，PACE在传统应用程序FTP中带来了严重的问题，几乎没有对所有流量进行分类。从结果中提取的另一个有趣的观察结果如下所示：

* L7过滤器，在此级别上最不可靠，通常将流分类为Skype和Finger。但是，大约1/3的Skype流被其错误分类为RTP，Finger，eDonkey或NTP。

* 流量分类器的作者专注于流行的应用程序，这些应用程序可能会产生大量数据，或者对于QoS要求至关重要。非加密的BitTorrent流和Skype流是所有分类器通常都能很好地检测到的仅有的应用程序组。

* 美国陆军游戏未使用任何工具分类。 nDPI获得的一些正确分类是由于识别了与游戏集成的TeamSpeak客户端发起的一些流程。

5. Results
This section provides an overview of the classification results of the different types of traffic by each of the classifiers. The evaluation was performed on 3 datasets: a normal set, a set with truncated packets, and a set with truncated flows. This section presents a description of the most interesting results, but a discussion is later presented in Section 6 with the conclusions that can be drawn from them. The following subsections present the results for the normal dataset, while the results for the sets with truncated packets or flows are discussed separately. All the accuracy results are given in terms of flows. The methodology used to compute the accuracy is shown in Section 3.3. The interested reader may find the complete confusion matrix in the technical report [39].

5.1. Application protocols
The evaluation of the classification of application protocols is shown in Table 5. The columns present the percentage of correctly and wrongly classified as well as unclassified flows belonging to various application protocols by each classifier. 

An important performance parameter of DPI-based techniques is the completeness of their results (i.e., number of applications they can classify). This section evaluates 17 different application protocols. As shown in Table 5, none of the techniques is able to classify all of them. Among the different techniques studied, nDPI and Libprotoident are the most complete, classifying 15 out of 17. At the far end, L7-filter only classifies 9 of 17.

Another important aspect of DPI techniques is their ratio of false positives (i.e., incorrect classifications). Usually techniques leave the non-recognized flows as unclassified, trying to decrease the number of false positives. Even though, both versions of L7-filter are characterized for producing a high number of incorrect classifications (e.g., L7-filter-all classifies 85.79% of HTTP traffic as Finger). Regarding the specific classifications, most of traditional application protocols (i.e., DNS, HTTP, IMAP-STARTTLS, POP3-PLAIN, SMTP-PLAIN and SSH) are generally well detected by all the techniques (e.g., accuracy between 70.92% and 100%). Unexpectedly, Libprotoident is the only classifier able to identify all the tested encrypted protocols. Regardless of the classifier, the undetected encrypted traffic is usually identified as regular SSL. An interesting case is presented by the classification of RTMP. Only nDPI and Libprotoident are able to properly classify it. PACE and OpenDPI classify this traffic as Flash. Although both traffics are usually related, the classification as Flash cannot be considered as being correct, as Flash is only a content container. Flash content (audio, video or any other binary file) can be transported using various applications protocols (e.g., HTTP, RTMP) or even different transport protocols (both TCP and UDP).

5.2. Applications

The second level of classification studies the application that uses its proprietary application-level protocols (e.g., BitTorrent, Skype). The evaluation of the classification of various applications is shown in Table 6. The columns present the percentage of correctly and wrongly classified as well as unclassified flows belonging to various applications by each of the classifier. 

At application level the most complete technique is PACE, classifying 20 out of the 22 evaluted applications, followed by nDPI (17) and Libprotoident (14). Again, L7-filter is among the worst techniques (8), but it is overcomed by NBAR that classifies only 4 applications.

Regarding the false positive ratio, the lowest percentage of misclassified flows were obtained from PACE and OpenDPI. Contrary, the highest ratio of misclassified flows is again obtained from the classifications by both versions of L7-filter (e.g., 97% of America’s Army traffic is classified as Skype and RTP).

As shown in Table 6, the classification at application level presents more problems than at application protocol level. It is particularly striking that almost no classifier is able to completely classify all the flows (i.e., 100%) from a specific application. This can be derived from the fact that usually applications use different internal operations that produce different traffic. Therefore, techniques need a specific pattern for every type of operation. For instance, the accuracy with Skype is always lower than 100% because none of the techniques is able to classify neither Skype filetransfers nor videos. Among the different studied techniques, PACE is the most accurate followed by nDPI and Libprotoident. Surprisingly, PACE presents severe problems with a traditional application as FTP, almost non classifying all its traffic. Another interesting observations extracted from the results are shown below:

* L7-filter, the most unreliable at this level, usually misclassifies the flows as Skype and Finger. However, around 1/3 of the Skype flows are misclassified by it as RTP, Finger, eDonkey, or NTP.

* The authors of traffic classifiers focus on popular applications, which either generate heavy data volume, or are critical regarding QoS requirements. Non-encrypted BitTorrent flows and Skype flows are the only groups of applications that are generally well detected by all the classifiers.

* America’s Army game is not classified by any tool. The few correct classifications obtained by nDPI are due to the recognition of some flows originated by the TeamSpeak client integrated with the game.

5.3。网页服务

研究的最后一个级别评估许多不同的Web服务。由于清楚和理解，我们不会将结果显示为表格，而只是重要结果的摘要。

Web服务的结果遵循先前级别获得的结果。 PACE是最完整，最准确的技术。其余技术的不良结果主要是由于没有足够的特定分类（例如，将Facebook流量分类为HTTP）。

PACE认可4Shared（84.69％），Amazon（58.97％），Apple（0.84％），Blogspot（3.83％），eBay（67.97％），Facebook（80.79％），Google（10.79％），Instagram（88.89％）， Linkedin（77.42％），Mediafire（30.30％），Myspace（100％），QQ.com（32.14％），Twitter（71.18％），Windows Live（96.15％），Yahoo（54.70％）和YouTube（81.97％） ）。 PACE在识别属于这些服务的SSL流方面没有任何问题，这意味着PACE必须使用其他技术，而不仅仅是直接查看数据包以将流与特定服务相关联，这可能是通过分析服务器证书来实现的。

该商业工具明显改进了其开源版本OpenDPI，该版本仅识别直接下载网站：4Shared（83.67％）和MediaFire（30.30％）。

L7过滤器仅识别Apple（0.42％）。此外，L7过滤器（尤其是L7-filter-all）的特征是属于Web服务的错误流非常多（通常为80–99％）。绝大多数人将这些流量识别为Finger和Skype。

nDPI识别了亚马逊（83.89％），苹果（74.63％），博客（4.68％），Doubleclick（85.92％），eBay（72.24％），Facebook（80.14％），谷歌（82.39％），雅虎（83.16％）， Wikipedia（68.96％）和YouTube（82.16％）是该级别上第二好的技术。

与以前的级别不同，Libprotoident仅识别Yahoo（2.36％）Web服务。鉴于Libprotoident仅使用数据包有效负载的前4个字节对流进行分类，因此使特定分类作为Web服务变得更加困难，因此这一结果是可以理解的。

在此级别上最差的技术是NBAR，它无法识别任何Web服务。

5.3. Web services

The last level studied evaluates many different web services. Because of clarity and understanding, we do not present the results as a table but as a summary of the important outcomes. 

The results with web services follow the outcomes obtained at previous levels. PACE is the most complete and accurate technique. The bad results of the rest of techniques are mainly due to a not enough specific classification (e.g., Facebook traffic classified as HTTP).

PACE recognizes 4Shared (84.69 %), Amazon (58.97 %), Apple (0.84 %), Blogspot (3.83 %), eBay (67.97 %), Facebook (80.79 %), Google (10.79 %), Instagram (88.89 %), Linkedin (77.42 %), Mediafire (30.30 %), Myspace (100 %), QQ.com (32.14 %), Twitter (71.18 %), Windows Live (96.15 %), Yahoo (54.70 %), and YouTube (81.97 %). PACE does not have problems with recognizing SSL flows belonging to these services, which means that PACE must use other techniques than just looking directly into the packets to associate the flows with the particular services, probably by analyzing the server certificates. 

The commercial tool clearly improves upon its open-source version OpenDPI that recognizes only Direct Download websites: 4Shared (83.67 %) and MediaFire (30.30 %). 

L7-filter recognizes only Apple (0.42 %). Furthermore, L7-filter (especially L7-filter-all) is characterized by a very high number of misclassified flows belonging to web services (usually 80–99 %). The flows are recognized in a vast majority as Finger and Skype.

nDPI recognizes Amazon (83.89 %), Apple (74.63 %), Blogspot (4.68 %), Doubleclick (85.92 %), eBay (72.24 %), Facebook (80.14 %), Google (82.39 %), Yahoo (83.16 %), Wikipedia (68.96 %), and YouTube (82.16 %) being the second best technique at this level.

Unlike at previous levels, Libprotoident recognizes only the Yahoo (2.36 %) web service. This result is understandable given that Libprotoident only uses the first 4 bytes of packet payload to classify a flow, making considerably more difficult a specific classification as web service.

The worst technique at this level is NBAR that does not recognize any web services.


5.4 数据包截断的影响

每个DPI工具的一个重要特征是每个数据包识别流量所需的信息量。这极大地影响了分类速度和所需的资源。此外，出于隐私原因，发布了许多流量跟踪，其中有效负载被截断为每个数据包最多一定数量的字节。如前所述，Libprotoident是唯一的工具，该工具被公告使用特定级别的检查数据包，即前4个字节。因此，为了发现每个工具的内部属性，我们决定测试数据包截断的影响。本小节介绍了正常数据集的分类结果与前70 B的以太网帧被截断的数据集之间的差异。

数据包的截断对除Libprotoident和NBAR以外的所有工具（大多数倾向于保持其正常准确性）的大多数应用程序协议的分类有相当大的影响。这表明NBAR可以某种方式实现为Libprotoident来对应用程序协议进行分类，而其余技术则基于完整流程对它们进行分类。 L7过滤器无法在此集合上检测DNS流量，而所有其他分类器的准确性都超过99％。出乎意料的是，NBAR无法在正常集合上检测到NTP，而在带有截断数据包的集合上正确地100％地检测到它。鉴于NBAR不是开源工具，我们无法给出这个结果的确切原因。

在应用程序级别，只有Libprotoident能够保持其正常准确性，而其余技术会大大降低其准确性。

关于Web服务级别，只有nDPI能够识别此集合中的某些Web服务。例外地，检测率几乎与正常情况相同。其他分类器倾向于将此类流量视为未知。

5.5 截流的影响

另一个主要问题是要对每个流进行分类需要多少个数据包。这取决于分类工具以及我们要识别的应用程序或协议。但是，流量分类器的文档并未涵盖这些问题，尽管它们对于在发布用于测试工具的数据跟踪时节省磁盘空间非常重要。因此，我们决定研究流量截断的影响。本小节介绍了普通数据集的分类结果与前10个数据包截断流的数据集之间的区别。

流的截断对应用程序协议的分类没有任何明显的影响。该结果表明，应用协议的分类依赖于从流的第一个数据包中提取的模式或签名。

在应用程序级别获得类似的行为。但是，在这种情况下，对应用程序分类的影响是显而易见的–检测率下降。唯一的例外是Libprotoident，它提供的结果与普通数据集相同。因此，这暗示某些应用程序的分类可能依赖于基于统计的技术（例如，机器学习）。活动模式下的FTP是一个非常有趣的情况，因为Libprotoident保持其100％的准确性，而其他分类器的准确性下降到5.56％。对于普通的eDonkey流量，出现了一个奇怪的情况，因为我们在截断流的集合上使用PACE获得了最佳分类精度（45.28％），而在正常集合上的准确性仅为16.50％。

正确分类的Web服务的百分比通常与正常集合的百分比相同或几乎相同。

5.4. Impact of packet truncation

An important characteristic of each DPI tool is the amount of information needed from each packet to identify the traffic. That significantly influences the classification speed and the resources needed. Furthermore, many traffic traces are published with payload truncated up to a certain number of bytes per packet for privacy reasons. As mentioned before, Libprotoident is the only tool, which is advertised to use the particular extent of the examined packets, namely first 4 bytes. Therefore, in order to discover the internal properties of each tool, we decided to test the impact of packet truncation. This subsection presents the differences between the classification results for the normal dataset and the dataset with truncated Ethernet frames to the first 70 B.

Truncation of packets has a considerable impact on the classification of most application protocols by all tools except Libprotoident and NBAR, which tend to maintain their normal accuracy. This suggests that NBAR can be somehow implemented as Libprotoident to classify application protocols while the rest of techniques base their classification on the complete flow. L7-filter is not able to detect DNS traffic on this set, while all the other classifiers present the accuracy of over 99 %. Unexpectedly, NBAR cannot detect NTP on the normal set, while it detects it 100 % correctly on the set with truncated packets. We can not present a verificable reason of this result given that NBAR is not an open-sourcer tool.

At application level, only Libprotoident is able to keep its normal accuracy whereas the rest of techniques considerably decreases their accuracies.

Regarding the web services level, only nDPI is able to recognize some web services in this set. Exceptionally, the detection rate is almost the same as for the normal set. Other classifiers tend to leave such traffic as unknown.

5.5. Impact of flow truncation

Another major concern is how many packets are needed in order to classify each flow. That depends on the classification tool as well as on the application or protocol, which we want to identify. However, the documentation of the traffic classifiers do not cover these issues, although they are very important for conserving disk space while publishing data traces used to test the tools. Therefore, we decided to study the impact of flow truncation. This subsection presents the differences between the classification results for the normal dataset and the dataset with truncated flows to the first 10 packets.

Truncation of flows does not have any noticeable impact on the classification of application protocols. This result suggests that the classification of application protocols relies on patterns or signatures extracted from the first packets of the flows.

Similar behavior is obatained at application level. However, in this case the impact on the classification of applications is noticeable – the detection rate decreases. The only exception is Libprotoident, which provides the same results as for the normal dataset. Therefore, this insinuate that the classification of some applications probably rely on techniques based on statistics (e.g., Machine Learning). FTP in the active mode is a very interesting case, as Libprotoident maintains its 100 % accuracy, while the accuracy of the other classifiers drops to 5.56 %. An strange case is presented with Plain eDonkey traffic, as the best classification accuracy (45.28 %) we obtained by using PACE on the set with truncated flows, while the accuracy on the normal set was only 16.50 %.

The percentage of correctly classified web services is usually the same or nearly the same as for the normal set.

6.讨论

本节从性能比较期间获得的结果中提取结果。另外，我们讨论了研究的局限性。

如上一部分所示，对于大多数研究的分类组，PACE是最佳分类器。如此高的排名是由于能够在各种级别上提供结果，例如HTTP：generic：facebook。其他分类器根本不提供此功能，并且仅给出一个选择的级别，因此，例如，当它们识别所传输内容的Web服务时，它们不提供解决HTTP或SSL流量的可能性。但是，PACE在这方面也不是完全一致的。 Facebook视频（我们观察到是通过HTTP传输的）被检测为FLASH：no_subprotocols：facebook，而来自Justin.tv的使用RTMP的实时视频流被分类为FLASH：no_subprotocols：not_detected。因此，我们不知道从分类器中获得的结果，因为使用了哪个应用程序协议（HTTP，RTMP或其他），因为返回了内容容器级别。理想情况下，DPI技术应在所有可能的级别上提供结果，例如HTTP：FLASH：VIDEO：YOUTUBE，以便可以执行一致的计费。但是，PACE是并非所有研究团体都可以使用的商业工具。在可用的开源工具中，nDPI和Libprotoident显示为最可靠的解决方案。对于我们来说令人惊讶的是，通过为每个方向使用有效载荷的前四个字节，Libprotoident取得了很好的结果，而没有给出明显数量的错误分类。另一方面，L7过滤器和NBAR在对数据集中的流量进行分类时效果较差。

我们没有观察到在正常数据集上进行的分类与截断流到最大10个数据包的分类之间没有太大差异。通常，除Libprotoident以外，所有工具都将带有截断包的集合比其他集合分类的差得多，后者保持了相同的准确性。我们发现，经过改进的L7-filter-com版本比默认的L7-filter-all总体而言具有更好的结果，这是因为正确分类的数量增加了，并且错误分类的比率大大降低了（特别是关于Web服务）。

但是，先前的结论显然与我们数据集中包含的特定应用程序协议，应用程序和Web服务相关。尽管我们尽了最大的努力来模拟用户的真实行为，但是许多应用程序，行为和配置并未在其上表示出来。因此，我们接下来讨论一些限制：

*在我们的研究中，我们评估了17个著名的应用程序协议，19个应用程序（包括4个不同配置的应用程序）和34个Web服务。从不同分类器获得的结果与那些组直接相关。因此，引入不同的群体可能会产生不同的结果。但是，所有组都是单独评估的，因此，将新组添加到我们的数据集中不会改变本文中介绍的结果。

*尽管已手动创建并实际创建了用于构建数据集的流量，但这是人为的。骨干网流量会携带未在我们的数据集中完整表示的组的不同行为（例如，在端口80上运行的P2P客户端）。可能无法从当前结果直接推断出所研究工具的性能。但是，人为创建的流量使我们能够发布具有完整数据包有效负载的数据集。

*NBAR和L7过滤器的性能不佳可能会受到我们数据集特征的影响。因此，基于它们的先前作品的可靠性不会受到质疑。不同的配置[42、45、10]和不同或更旧的分类组可能会产生不同的结果。

*分类级别对结果有很大影响。例如，Libprotoident目前无法对Facebook，Google或Twitter进行分类，但是nDPI和PACE可以对Facebook，Google或Twitter进行分类。

*可用的数据量也会影响性能。本文介绍的研究是使用完整的有效载荷数据包执行的。但是，在其他工作中，通常使用少量字节的数据[46、13、47]（例如96个字节）来收集跟踪，以避免数据包丢失，磁盘空间和隐私问题。对于这种情况，似乎Libprotoident是更合适的解决方案，因为它仅使用每个数据包的前4个字节。
*流量的性质，分布和异构性也会影响性能。 PACE检测到的类别数量远大于其余分类器检测到的类别数量，这使得PACE更适合于异构场景。而且，PACE和nDPI能够在非对称方案中对流量进行分类。

6. Discussion

This section extracts the outcomes from the results obtained during the performance comparison. Also, we discuss the limitations of our study. 

As shown in the previous section, PACE is the best classifier for most of the studied classification groups. This high ranking is due to the ability of providing the results on various levels, as for example HTTP:generic:facebook. Other classifiers do not offer this ability at all and only one chosen level is given, so, for example, they do not offer the possibility to account the HTTP or SSL traffic, while they recognize the web service of the transported content. However, PACE also is not totally consistent in that matter. Facebook videos (which we observed as transported by HTTP) were detected as, for example, FLASH:no_subprotocols:facebook, while the live video streams from Justin.tv using RTMP, were classified as FLASH:no_subprotocols:not_detected. So, we do not have the knowledge from the results obtained from the classifier which application protocol was used (HTTP, RTMP, or other), because the content container level is returned instead. Ideally, the DPI techniques should provide results on all the possible levels, as HTTP:FLASH:VIDEO:YOUTUBE, so that kind of consistent accounting could be performed. However, PACE is a commercial tool not accessible for all the research community. Among the available open-source tools, nDPI and Libprotoident reveal as the most reliable solutions. Surprisingly for us, Libprotoident achieves very good results without giving a noticeable number of false classifications by using the first four bytes of payload for each direction. On the other hand, L7-filter and NBAR perform poorly in classifying the traffic from our dataset. 

We did not observe large differences between the classifications performed on the normal dataset and the set with truncated flows to maximum 10 packets. The set with truncated packets is usually much worse classified than the other sets by all tools except Libprotoident, which maintains the same accuracy. We found that our modified version of L7-filter-com provides overall better results than the default L7-filter-all by increased number of correct classifications and greatly reduced rate of misclassifications (especially, regarding the web services).

Nonetheless, the previous conclusions are obviously tied to the particular application protocols, applications, and web services included in our dataset. Although we tried our best to emulate the real behavior of the users, many applications, behaviors and configurations are not represented on it. Because of that it has some limitations that we discuss next:

• In our study, we evaluated 17 well-known application protocols, 19 applications (including 4 in various configurations), and 34 web services. The results obtained from the different classifiers are directly related to those groups. Thus, the introduction of different groups could arise different outcomes. However, all the groups are evaluated individually and, therefore, the addition of new groups to our dataset would not change the results presented in this paper.

• The traffic generated for building the dataset, although has been manually and realistically created, is artificial. The backbone traffic would carry different behaviors of the groups that are not fully represented in our dataset (e.g., P2P clients running on port 80). The performance of the tools studied might not be directly extrapolated from the current results. However, the artificially created traffic allowed us to publish the dataset with full packet payloads.

• The poor performance of NBAR and L7-filter might be affected by the characteristics of our dataset. Thus, the reliability of previous works based on them is not called into question. Different configurations [42, 45, 10] and different or older classification groups would probably produce different results.

• The classification levels have considerable impact on the results. For instance, classifying Facebook, Google or Twitter is currently not possible by Libprotoident, however it is possible by nDPI and PACE.

• The amount of data available would also have impacted on the performance. The study presented in this paper is performed with full payload packets. However, in other works the traces are usually collected with a few bytes of data [46, 13, 47] (e.g., 96 bytes) in order to avoid packet loss, disk space, and privacy issues. For this scenario, it seems that Libprotoident is a more suitable solution, giving it only uses the first 4 bytes of every packet.
• The nature, distribution, and heterogeneity of the traffic would also impact the performance. The amount of classes detected by PACE is considerably bigger than detected by the rest of the classifiers, which makes PACE more suitable for heterogeneous scenarios. Also, PACE and nDPI are able to classify traffic in asymmetric scenarios.



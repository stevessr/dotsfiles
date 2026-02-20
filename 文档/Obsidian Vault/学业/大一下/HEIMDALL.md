# HEIMDALL: 通过广泛的机器学习流水线优化存储I/O接纳
[[HEIMDALL en]]
Daniar H. Kurniawan
芝加哥大学，伊利诺伊州芝加哥市，美国
MangoBoost Inc.，华盛顿州贝尔维尤市，美国
daniar.h.k@gmail.com

Rani Ayu Putri
万隆理工学院，西爪哇省万隆市，印度尼西亚
芝加哥大学，伊利诺伊州芝加哥市，美国
raniayu@uchicago.edu

Peiran Qin
芝加哥大学，伊利诺伊州芝加哥市，美国
peiranqin@uchicago.edu

Kahfi S. Zulkifli
万隆理工学院，西爪哇省万隆市，印度尼西亚
sbhnkahfi@gmail.com

Ray A. O. Sinurat
芝加哥大学，伊利诺伊州芝加哥市，美国
rayandrew@uchicago.edu

Janki Bhimani
佛罗里达国际大学，佛罗里达州迈阿密市，美国
jbhimani@fiu.edu

Sandeep Madireddy
阿贡国家实验室，伊利诺伊州芝加哥市，美国
smadireddy@anl.gov

Achmad Imam Kistijantoro
万隆理工学院，西爪哇省万隆市，印度尼西亚
imam@itb.ac.id

Haryadi S. Gunawi
芝加哥大学，伊利诺伊州芝加哥市，美国
haryadi@cs.uchicago.edu

# 摘要

本文介绍了HEIMDALL，一种高精度、高效率、由机器学习驱动的闪存I/O接纳策略，设计为以黑盒方式运行。我们在机器学习的各个阶段进行了领域特定的创新，引入了精确的基于周期的标记、三阶段噪声过滤、深入的特征工程和精细的调优，这些共同将决策准确率从 $67\%$ 提升至 $93\%$。我们执行了各种部署优化，以实现亚微秒级的推理延迟和仅28KB的小内存开销。通过源自生产环境踪迹的500个无偏随机实验，我们展示了HEIMDALL相比现有最先进技术，平均I/O延迟降低了 $15 - 35\%$，并且比基线快高达 $2\倍$。HEIMDALL已准备好用于用户级、内核内和分布式部署。

# CCS 概念: - 软件及其工程 $\rightarrow$ 操作系统; - 计算方法论 $\rightarrow$ 机器学习方法。

关键词: 面向系统的机器学习；I/O接纳控制；文件和存储系统；操作系统；分布式系统

# ACM 引用格式:

Daniar H. Kurniawan, Rani Ayu Putri, Peiran Qin, Kahfi S. Zulkifli, Ray A. O. Sinurat, Janki Bhimani, Sandro Madireddy, Achmad Imam Kistijantoro, and Haryadi S. Gunawi. 2025. HEIMDALL: Optimizing Storage I/O Admission with Extensive Machine Learning Pipeline. In Twentieth European Conference on Computer Systems (EuroSys '25), March 30- April 3, 2025, Rotterdam, Netherlands. ACM, New York, NY, USA, 17 pages. https://doi.org/10.1145/3689031.3717496

# 1 引言

数据中心近期的几项进展直接受到闪存阵列更广泛采用的推动。SSD以其微秒级的访问速度，已成为存储硬件堆栈中的首选。然而，笼罩在SSD高性能之上的一片乌云是其不确定的内部操作，例如垃圾回收（GC）、内部缓冲区刷新和磨损均衡。这些日益增长的后台复杂性导致显著的尾延迟放大，引发不可预测的中断，从而降低用户体验。例如，GC操作可能会使延迟增加高达 $60倍$，对性能产生负面影响。为应对这一挑战，I/O接纳控制（在某些文献中称为副本选择）应运而生，以避免向正在进行密集内部操作的闪存阵列提交I/O请求。

关于I/O接纳控制技术的研究已有大量文献，这些技术依赖于启发式方法，如对冲（hedging）、副本评分和速率限制[38, 36-38, 74, 87]，以及预测和速率控制。然而，随着存储行业的发展，由于硬件能力的增强，现实世界中的工作负载呈现出更复杂的模式，使得可以完成更多的工作。结果，基于启发式的I/O接纳控制因其预定义的规则阻碍了其适应性而显得不足。为了解决这些局限性，最近的研究已转向利用机器学习（ML）来应对存储的各个领域，因为其学习能力克服了基于启发式算法的僵化。越来越多的复杂决策挑战转向由ML驱动，包括I/O接纳、缓存、配置调优、去重、故障检测、索引、预取、调度等。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/f28c965c3ebe784673a0da97d24dff1070408d3f7eaf1f70c7223c93b495dd9e.jpg)  
图1. 机器学习流水线 (§1)。

尽管基于ML的解决方案通常优于传统启发式方法，但许多研究在设计这些系统时未能探索ML流水线的全部范围。当ML设计过程的关键阶段（如图1所示）未得到充分处理时，就会出现这些差距。跳过或过度简化这些阶段可能会在准确性、性能和适应性方面阻碍模型的潜力。这在由ML驱动的I/O接纳控制系统中尤为明显。在现代设备和近期的I/O踪迹（来自MSR Cambridge、阿里巴巴和腾讯）上测试时，几种由ML驱动的I/O接纳控制的准确性显著下降——平均为 $67\%$，远低于其最初声称的水平。在I/O接纳控制的背景下，低准确性尤其成问题，因为错误的接纳和重路由会对I/O延迟产生负面影响。这些发现强调了更彻底地执行ML流水线以实现性能更好的模型的重要性。

为此，HEIMDALL引入了一个更强大的由ML驱动的I/O接纳控制系统，该系统通过遵循ML流水线的关键阶段精心开发。通过注入深厚的存储领域知识并在每个阶段严格优化模型，HEIMDALL旨在实现更高的准确性和更好的性能，以应对延迟关键环境中现代存储工作负载的挑战。HEIMDALL作为一个处理I/O请求的黑盒系统运行，做出接纳或重定向决策，同时监控请求延迟以检测SSD内部进程的影响。HEIMDALL引入了三项技术贡献。

首先，HEIMDALL引入了“基于周期的标记”概念，这源于我们对生产踪迹的观察。我们观察到SSD内部管理过程不仅影响单个I/O，还影响一个周期内的连续I/O，因此称为基于周期的标记。这种数据标记技术使我们能够训练出更准确的ML模型。该技术也可以应用于与SSD和调度相关的其他研究领域。

其次，为确保HEIMDALL适用于现实世界的生产系统，我们引入了一种学习粒度机制，该机制非常适合粗粒度决策已足够的情景。通过允许粗粒度预测，HEIMDALL用微小的准确性损失换取了推理吞吐量的显著提升。这种方法很好地推广到“面向系统的机器学习”中的一个常见挑战：现实世界部署的困难。据我们所知，我们是第一个应用此技术来优化I/O接纳控制的。我们通过成功将其集成到Linux内核和Ceph RADOS中，进一步验证了HEIMDALL的现实世界适用性。

第三，我们提出了一个通用且可扩展的ML流水线，它支持HEIMDALL接纳控制系统的开发。这个灵活的流水线强调了整合领域知识的重要性，并可适用于解决广泛的存储系统挑战。

最后，我们在来自公司（MSR、阿里巴巴和腾讯）的生产踪迹上评估了我们的解决方案，并设置了三个集成级别：用户级存储（用于快速和大规模评估）、Linux内核（用于模拟真实部署）和Ceph RADOS（用于分布式存储设置）。我们通过500个实验进行的全面评估表明，与流行的算法（如对冲和先进的接纳启发式及ML模型）相比，HEIMDALL的平均延迟降低了 $15 - 35\%$，并且比基线快高达 $2倍$。此外，我们实现了亚微秒级的推理延迟，在 $2.30\mathrm{GHz}$ 处理器上高达 $0.08\mu$s。

我们通过展示HEIMDALL的优化I/O接纳系统如何能作为ML驱动存储解决方案进一步研究和创新的基础来结束本文。尽管是为I/O接纳控制而开发的，HEIMDALL广泛的ML流水线也可以支持更广泛的存储优化探索，使学生和研究人员能够在该流水线内试验新技术或将其应用扩展到其他存储挑战。

# 2 背景与动机

接纳问题：接纳问题是许多操作的基础，例如作业提交、虚拟机放置、缓存数据、RPCs和I/O请求。接纳策略需要决定

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/f8fb3b13dc4ca348bd8493c579980abeb7bb1ce301667e6bd39afa80666e765c.jpg)  
图2. I/O接纳 (§2)。(a) I/O接纳决策 和 (b) 每个后端节点中基于神经网络的决策。

是接纳操作到底层资源，还是延迟或将其重路由到另一个资源。这样的策略对于表现出尾延迟行为的资源非常有用，即大多数时候操作很快，但有时（例如，$1 - 10\%$ 的时间）由于资源争用而变慢。

I/O接纳：I/O接纳问题涵盖了几个不同的研究目标，例如减少过多的闪存写入和最小化尾延迟。我们专注于在并行、冗余的闪存存储阵列中，在块级别进行I/O接纳，这些阵列使用数据复制，目标是减少尾延迟。例如，在数据中心，像RAID这样的冗余机制通过存储复制数据来确保容错，但它们也为性能优化带来了机会。先前的工作，如FusionRAID、Tiny-Tail Flash和ECCache，已经证明从完整的条带中重构延迟数据通常比等待缓慢的I/O响应更快。这些发现表明，可以利用冗余感知的调度来优化存储性能。I/O接纳的工作方式是选择一个I/O请求应该提交到哪个副本，以降低I/O请求的延迟。如图2a所示：(1) 前端层向存储数据的后端SSD发送一个I/O请求。每个后端节点做出接纳决策，(2) 接纳请求到底层SSD，或(3) 拒绝请求并要求前端层(4) 将其重路由到另一个拥有副本的节点。

闪存存储：接纳决策对于将I/O从正在经历由GC、内部缓冲区刷新、磨损均衡以及突发工作负载引起的严重资源争用的闪存设备上重路由非常有用。没有适当的接纳/重路由，所有这些都会导致读取尾延迟并增加平均延迟。由于设备内部的写入缓冲区，写入尾延迟非常罕见，因此我们专注于优化读取延迟。

基于ML的策略：I/O接纳可以基于ML模型，如LinnOS，它使用一个轻量级神经网络来预测黑盒SSD内部的争用。如图2b所示，LinnOS被设计为以统一的每页（4KB I/O）粒度做出接纳决策。因此，它不包括I/O大小作为输入特征。相反，LinnOS的决策基于历史延迟和I/O队列长度特征，例如最后几个I/O的延迟及其到达时的队列长度。利用这些信息，模型预测一个传入的

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/dfe5e1771c8c2d42a2d7d0fb3fd67c290617a57283373cb58cf1127e167909ce.jpg)  
图3. 精确标记 (§3.1)。在所有图中，一个点代表一个I/O。(a) 快/慢截止点，基于延迟。(b) 不精确的标记，带有大I/O标注的延迟CDF。(c) 时间线图，基于周期的标记。(d) 梯度下降。

I/O将是“快的”（因此，接纳I/O）还是“慢的”（因此，重路由I/O）。LinnOS部署在每个闪存设备的内核块层。

训练：为了做出准确的“快/慢”二元预测，像LinnOS这样的ML模型必须首先进行训练。在启用接纳决策之前，存储操作员可以记录过去15分钟I/O的特征，记录它们的静态/运行时特征和I/O延迟。然后，操作员根据某种标记算法（详见第3.1节）将每个记录的I/O标记为“快”或“慢”。在训练期间，模型学习哪些I/O模式会导致特定工作负载-设备对的慢I/O。训练得到的神经元权重随后应用于内核内模型进行部署。

准确性：ML模型可能做出两种不准确的决策：错误接纳（false admits），即I/O被预测为“快”，但实际上经历了缓慢；或错误重路由（false reroutes），即预测为“慢”，但实际上没有问题。我们最近的评估发现，LinnOS的4年老模型的准确性下降到 $67\%$，因为它未能跟上现代工作负载和更快的SSD。

# 3 HEIMDALL的流水线

我们首先描述我们受上述挑战启发的解决方案。在本节中，我们将展示如何通过利用设计过程中的每一步来显著提高I/O接纳控制的准确性。特别地，我们在数据分析阶段通过精确标记(§3.1)和噪声过滤(§3.2)进行领域特定的创新；在建模阶段通过彻底的特征工程(§3.3)、模型探索(§3.4)和超参数调优(§3.5)进行创新；以及在训练阶段(§3.6)进行创新。

# 3.1 精确标记

我们首先评估了先前工作在基于延迟的建模背景下进行的自动标记。

```
1. 函数 AccurateLabeling()
2. 输入: 数据 D {大小, 吞吐量, 延迟}
3. 输出: 数据 D {大小, 吞吐量, 延迟, 标签}
4. high_lat, low_thpt = CalcThreshold(D)
5. MAX_DROP = CalcThptDropThreshold(D)
6. thpt_median = CalcMedian(吞吐量)
7. for io in D do: // 初始化
8.   io.label = 0; io.mark_start = 0
9.   if IsBusy(io, high_lat, low_thpt, MAX_DROP) do:
10.    io.label = 1 // 尾区开始
11. for io in D do:
12.   if io.label == 1 do: // 标记尾区
13.     while io.next > 吞吐量 < thpt_median do:
14.       io.label = 1 // 1 = 拒绝; 0 = 接纳
15.       io = io.next
```
图4. 精确标记算法 (§3.1)。

我们发现，有几种方法使用基于延迟的算法来决定延迟截止点，如图3a所示，其中算法根据算法生成的拐点（截止点）将训练数据集中的I/O标记为“快”或“慢”。

虽然基于延迟截止点的算法在某些领域工作得很好——例如，在网络领域中，每个数据包/请求的大小保持稳定，或者在存储领域中，每页延迟模型只能对每4KB的I/O进行推断——但这种方法不适用于在整个、可变I/O级别（从一页(4KB)到大请求(2MB)）做出决策的ML模型。例如，在图3b中，一个大的I/O（红点）在这里被标记为“慢”，因为其在数据集中的测量延迟大于截止点。然而，这是不准确的，因为即使该I/O被重路由到另一个设备，由于其大尺寸，它仍然会是“慢”的。

为了纠正这个问题，我们发现基于周期的算法为我们的问题领域提供了最好的结果。也就是说，我们不是决定哪些特定的I/O应该被标记为慢或快，而是根据周期（时间窗口）进行标记，我们的算法猜测设备是处于快速周期（例如，没有内部争用）还是慢速周期（例如，经历GC争用等）。尽管我们无法精确预测其发生，但我们仍然可以在数据中发现一些模式，以缩小数据集中可能的尾延迟段。例如，在图3c中，慢速周期中的所有I/O（例如，延迟飙升和吞吐量下降的地方）都将被标记为“慢”。

我们的算法由3个阶段组成，如图4所示。(a) 首先，在第9行，我们对延迟和吞吐量之间的关系进行分类。我们只应该在延迟高且吞吐量同时低时才怀疑设备繁忙。我们观察到内部争用会导致吞吐量下降和延迟飙升。吞吐量对于检测此类事件的开始和结束更为敏感，因为它也考虑了I/O大小。(b) 因此，我们使用延迟和吞吐量阈值（在第4行声明）来决定何时延迟看起来高或吞吐量看起来低。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/908c1b04976ad4b1c26f35d332c67842195a60fcb0d00613ef9dbdc375bb3e55.jpg)  
图5. 预处理的重要性 (§3.1 §3.2)。(a) 截止点与基于周期的标记。(b) 噪声误预测率。

找到适用于不同设备和工作负载特征的阈值是具有挑战性的。我们使用基于梯度下降的方法来选择适当的阈值，以平衡不同SSD和工作负载特征下的灵敏度和准确性。例如，图3d显示了准确性和灵敏度的全局最优值，分别由蓝线和红线表示。最后，(c) 基于这些值，我们决定繁忙周期的开始和结束时间（第12至15行）。图5a显示了我们从基于截止点的标记迁移到更准确的基于周期的标记所获得的准确性提升，强调了标记后数据更好的可学习性。在评估(§6.4)中，我们证明了精确标记将HEIMDALL的准确性提高了5.5%，使其准确性高达93%。

# 3.2 三阶段噪声过滤

为了最小化噪声训练数据对模型的影响，我们引入了一个领域特定的三阶段噪声过滤过程。该过程针对(1)慢周期内的离群点，(2)快周期内的离群点，以及(3)短噪声。我们的离群点移除专门针对由不规则或不具代表性事件引起的噪声，这些事件对尾延迟没有有意义的贡献，因此与改善尾性能的目标不冲突。如图5b所示，这些离群点经常导致模型误预测，使其具有破坏性而非信息性。与第§6.4节图14a中的评估结果一致，我们发现这三种类型的噪声累计会使准确性下降16%。消除它们可以让模型更有效地专注于学习和处理持续的争用模式，最终改善尾性能而不是削弱它。

在第一阶段，我们移除慢周期内的离群点，如图6a所示。在这里，长的慢I/O序列可能表示设备内部繁忙。然而，有时少数I/O可能会“幸运地”击中设备内部缓存，即使设备正忙于其他只影响NAND级别读写的活动。因此，如图所示，我们移除了这些“快”的离群点，特别是那些延迟低于且吞吐量高于周期内相应中位值的I/O。

在第二阶段，如图6c和图6d所示，我们移除快周期内的离群点，与上述第一种情况相反。这些慢I/O可能是由于一些其他罕见的设备特性，如电压不匹配导致的读重试、错误检查和纠正

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/49fafe82045d36c60bcb4ae08d7b8fa5ade7e3c73fb448db251799e897837c2f.jpg)  
图6. 噪声过滤 (§3.2)。来自阿里巴巴踪迹的样本数据。(a) 慢周期内的离群点。(b) 短噪声。快周期内的离群点：(c) 延迟和 (d) 吞吐量。

(ECC)等。由于这些罕见情况本质上是瞬时错误，将它们从数据集中移除将提高模型的准确性。

数据现在变得“更干净”了，但由于我们的标记过程，我们仍然发现轻微的不规则性。我们观察到一个短促的慢周期爆发（例如，仅3个连续的I/O），这不太可能是由设备内部争用引起的，如图6b所示。因为提供这种短促的爆发可能会迷惑模型，所以在过滤的第三阶段，我们采用与第3.1节图3d中相同的梯度下降方法，以找到一个合理的阈值，该阈值将提供高准确性但低灵敏度。在大多数数据集中，我们发现应移除3个（或更少）连续“慢”I/O的快速爆发。总体而言，我们的三阶段噪声过滤使准确性平均进一步提高了 $16\%$ (§6.4)。

# 3.3 深入的特征工程

最初，典型的基于请求的踪迹包含基本特征，如请求到达时间、大小和I/O类型（读/写）。从原始特征中派生出更多特征将有助于ML模型捕捉更多特性，以增加计算开销为代价提高准确性。由于我们处理的是延迟敏感系统，我们需要通过提取足够的先进特征、选择最佳的重要特征集并对数字进行良好缩放以避免不同特征的权重不均，来平衡所获得的模型准确性和计算开销。

为了确保对特征工程（例如特征提取、特征选择等）进行公平评估，我们使用神经网络（NN）模型，因为它避免了基于树的模型中的架构约束，例如决策树中的深度限制。这种灵活性使得NN特别适合于在不受僵化结构假设限制的情况下识别特征质量的变化。虽然先前的工作已经通过包含提交每个I/O时观察到的延迟和I/O队列长度来进行基本特征提取，通过移除时间戳和磁盘ID来进行特征选择，并使用数字化来进行特征缩放，但他们最终得到了31个特征。结果，每次推理的开销为2毫秒。我们试图回答这个问题：“在不牺牲准确性的前提下，我们能否使用更简单的模型和更少的特征来减少计算开销？”

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/051e259e1416fc00657cb81217c758737aa7fb1a84403ec23e31e6c68b2d6696.jpg)  
图7. 特征工程 (§3.3)。(a) 每个特征的相关性值，(b) 每个特征带来的准确性改进，(c) 模型在各种历史深度下的准确性，以及 (d) 模型在不同归一化方法下的准确性。

首先，我们通过移除相关性得分低的特征来进行特征选择，例如I/O到达时间（时间戳），这表明它与I/O是否引起尾延迟行为几乎没有关联。图7a按特征与决策的相关性对特征进行排序。

接下来，我们对结果特征进行了特征分析。图7b显示了这些特征如何影响整体模型准确性，证实了早期的发现，即五个主要特征（队列长度、历史队列长度、历史延迟、历史吞吐量和I/O大小）对于提高准确性至关重要。

第三，我们还改变了某些输入特征的历史深度 $(N)$，以确定模型需要多少过去的信息才能获得合理的准确性。通过历史深度，我们指的是最后 $N$ 个I/O的信息（例如最近的队列长度和I/O延迟）。这些信息可以暗示，例如，如果最近我们观察到一个高延迟但队列长度短的I/O，则设备内部繁忙。图7c显示，在各种数据集中，$N = 3$ 足以提高准确性。

最后，我们探索了特征缩放技术，以减少模型对特定特征的偏见，同时保持较低的计算开销。我们尝试了各种归一化方法，如图7d的前三条所示，发现min-max在平均上提供了最好的准确性。标准化方法，如鲁棒和标准缩放器，提供了更高的准确性，但由于其为保持所有历史延迟值以进行标准差和分位数计算而产生的高内存开销，对于我们的领域是不可行的。相比之下，min-max归一化只需要最小值和最大值，使其既准确又轻量。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/47411529eafbf47a4aa3583e707e2840db8b022268d85a13722f06a57ddc8b93.jpg)  
图8. 模型探索 (§3.4)。NN模型实现了高且稳定的准确性。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/7e6458cd9e8148d8849f00199982ea181864df27773df0bc88d61b3b0b144f9e.jpg)  
图9. 超参数调优 (§3.5)。(a) 每页 vs. 每I/O，(b) 隐藏层，(c) 层数，(d) 激活函数，(e) 输出层，以及 (f) 最终NN设计。

# 3.4 模型探索

通过选定的特征集，我们进行了多次模型探索，以确定最适合我们问题领域的模型。为确保公平，每个模型都经过轻微的超参数调优，直到没有观察到显著的准确性改进。图8总结了我们的发现，其中x轴是准确性变化（在不同数据集上测量），y轴是归一化准确性；图的左上角是更合适的模型。总体而言，我们发现神经网络（"NN"）与其他模型相比，实现了良好的准确性和最高的稳定性。

# 3.5 神经网络超参数调优

我们进行超参数调优，以确定最佳的层数、神经元数和激活函数，以平衡准确性和复杂性。由于严谨的特征和模型工程，我们新的NN（如图9f所示）比LinnOS的NN模型简单得多，原因如下：

(a) 首先，如图9a所示，由于LinnOS使用基于截止点的每页标记，其模型只能对每个4KB请求进行推理。因此，一个大的I/O必须被分割成小的统一大小的块I/O，增加了每个I/O的推理次数。而我们使用基于周期的每I/O标记，因此对于任何大小的每个I/O只需要一次推理。

(b) 其次，与LinnOS只使用一个隐藏层相比，我们使用2个隐藏层。图9b显示，最具影响力的准确性增长来自于增加第二个隐藏层。

(c) 第三，为了最小化计算开销，我们为第一个和第二个隐藏层分别使用128和16个神经元（而LinnOS在一个层中就使用了256个神经元）。在图9c中，x轴和y轴分别代表第一和第二隐藏层中的层数，单元格颜色代表实现的准确性。我们选择了给出相对较高准确性（颜色较深）的最轻量模型设计。

(d) 第四，我们为隐藏层的激活函数保留了ReLU。图9d中的x轴和y轴分别代表第一和第二层激活函数的排列。我们选择ReLU是因为它能带来高准确性（颜色较深）、轻开销和简单的计算。

(e) 最后，对于输出层，我们实验了softmax、linear和sigmoid。根据图9e中显示的结果，我们选择了一个单神经元的sigmoid。这与LinnOS的双神经元输出层不同，后者在从其相邻隐藏层传播梯度时会承担计算加倍的后果。

# 3.6 训练

由于尾延迟的固有性质，慢速和快速I/O之间存在数据分布不平衡，其中快速I/O在整体延迟分布中占主导地位。为了在训练过程中解决这个问题，我们尝试了有偏训练，通过定制加权损失函数来在模型接纳慢速I/O时对其进行惩罚。然而，我们没有看到显著的改进，甚至结果更差。经分析，由于慢速和快速I/O分布的变化，不同的数据集导致了不同的最优损失值，因此不可行。

其他可能的方法是数据采样和数据选择。由于过采样和欠采样可能会带来一些风险，我们通过确保我们的数据选择过程(§3.3)包括一些具有大量写I/O的时期（以触发设备后台活动）并进一步增强数据（远程和调整大小）来解决这个问题。

# 4 部署优化

在讨论了建模和训练之后，我们现在转向部署，重点关注我们通过联合推理技术和Python到C的转换与优化来优化推理延迟的努力。

# 4.1 可忽略的推理延迟

许多存储系统，如Linux块层和Ceph，是用C语言而不是Python编写的。这使得使用像TensorFlow或PyTorch这样在用户空间运行的优化推理库变得困难。将这些库嵌入到像Linux内核这样的延迟关键系统中会引入显著的开销。因此，为了在实际部署中提高推理延迟，我们必须将我们的模型从Python转换为C。为实现这一目标，我们遵循一个三步过程：Python到C/C++的转换、gcc优化和量化，使我们能够将推理时间减少到亚微秒级别。

首先，通过仔细和手动的Python到C++转换，我们将推理时间减少到20 $\mu$s。其次，我们使用额外的gcc优化来减少执行时间，避免了以牺牲性能为代价的快速编译。

表1. 实现规模 $(55)$ 。HEIMDALL用20.9K行代码（LOC）编写，主要为Python和 $C/C++$

| 组件 | 集成 + 评估 | |
| :--- | :--- | :--- |
| 数据集准备 | 在用户级 | 3.7 K |
| 设计流水线 | 在Linux内核中 | 2.1 K |
| 优化 | 在Ceph RADOS中 | 2.3 K |
| 重训练 | 评估模块 | 5.3 K |

我们选择了-03，因为它提供了最高的优化，同时仍然遵守严格的语言标准并保留了计算精度，从而将推理速度加快到 $0.08\mu \mathrm{s}$。

最后，我们执行量化以降低我们模型的计算复杂性并最小化内存占用。我们将所有层的权重乘以1,024，并相应地量化每层的偏置以匹配尺度。我们使用1,024是因为我们可以捕获大多数权重在小数点后4位内的非零数字。最终的延迟降至每个推理 $0.05\mu \mathrm{s}$。

我们还在各种CPU上进行了测试，有趣地发现推理延迟可能会有一个数量级的变化。例如，在AMD Ryzen 9 5900HS 3.30GHz上我们得到 $0.12\mu \mathrm{s}$ 的延迟，在AMD EPYC 7352 2.30GHz上得到 $0.08\mu \mathrm{s}$ 的延迟。在Apple M1 Pro 3.20GHz上，它甚至更快，为 $0.05\mu \mathrm{s}$。我们将进一步的调查留作未来工作。

# 4.2 联合/组推理

在某些部署中，对每个I/O都做接纳决策可能过于精细，导致在密集的I/O工作负载和有限资源下CPU开销过高。最近的方法，如LAKE，提出了GPU批处理，但这只在大型批处理推理中通过利用并行性才能很好地工作。虽然提高了吞吐量，但主机到GPU的延迟开销会带来额外的延迟。

相比之下，我们受到联合/组推理的启发。我们修改模型，使其能够接收多达 $P$ 个I/O的特征。联合推理比批处理更有效，它代表所有 $P$ 个I/O进行一次推理（想象一个绿灯让 $P$ 辆车通过），而批处理仍然需要模型运行 $P$ 次并做出 $P$ 个决策。

在HEIMDALL中，可以指定模型的推理粒度，范围从每次推理1个I/O到每次推理多达 $P$ 个I/O。对于所有推理粒度，HEIMDALL使用相同的模型架构。然而，历史数据的数量（见§3.3）与粒度 $(P)$ 成正比。存储操作员可以决定设置粒度。开发联合推理模型的挑战在于特征选择阶段，因为我们不想通过聚合所有 $P$ 个I/O的输入特征来增加模型的复杂性。相反，我们认为最新的设备行为反映在最近的I/O中。因此，优先考虑最近I/O的特征，忽略其余I/O的大部分特征。例如，对于 $P = 5$，我们仍然只提供这组5个I/O之前的最后三个I/O的历史信息（§3.3），从而通过减少冗余来保持模型的轻量级。稍后在第6.5节中，我们评估了其中的权衡，例如，更高的 $P$ 会带来更高的吞吐量/性能，但只会轻微降低准确性。

# 5 实现规模

我们为构建一个应用机器学习于存储系统的实验平台所做的努力如表1所示，该表分解了我们20.9 KLOC的HEIMDALL流水线实现。左栏显示了主要组件，包括数据集准备脚本、所有设计阶段(§3)、部署优化(§4)和重训练(§7)，右栏显示了我们在用户级存储(§6.1)、Linux内核(§6.2)和Ceph RADOS(§6.3)中的三个集成级别，包括我们重新实现其他策略的评估模块。这个可扩展的实验平台可以很容易地为其他研究重用。

# 6 评估

我们全面的评估设置如下：
• 数据大小：我们使用来自阿里巴巴、微软和腾讯的2 TB原始I/O块踪迹，并为所有实验生成11 TB的中间数据。
• 目标部署：我们将HEIMDALL集成到用户级存储（用于快速、大规模评估）、Linux内核（以模拟真实部署）和Ceph（用于分布式评估）中。
• 机器和SSD：默认情况下，我们使用Chameleon的Storage-NVMe节点，该节点配备AMD EPYC 7352 2.30GHz 24核CPU和256 GB DRAM。我们使用了来自不同制造商的10种不同型号的SSD。
• 训练/测试：所有实验均采用50:50的训练-测试方法，与80:20的划分相比，这提供了更均衡的表示，确保评估集在训练过程中完全未见，保证我们的评估指标是无偏的，并准确反映模型在未见数据上的性能。

本节将回答以下关键问题：

在大规模评估中，HEIMDALL的性能与最先进的算法相比如何？(§6.1)
当部署在Linux内核中时，HEIMDALL的性能如何？(§6.2)
在多节点Ceph集群中，HEIMDALL如何扩展以改善I/O延迟？(§6.3)
ML流水线中的每一步对HEIMDALL的整体性能有何贡献？(§6.4)
如何通过联合推理优化HEIMDALL的推理吞吐量？(§6.5)

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/abbf812944061dbc555f4c60898c32ec3a78de5311551eb2f2e22e27a374dc24.jpg)  
图10. 基于启发式算法的比较 ( § 6.1 )。C3的性能优于其他最先进的算法。

• HEIMDALL的轻量级神经网络模型的CPU和内存开销以及训练时间是多少？(§6.6 和 §6.7)

# 6.1 大规模评估

为了全面且无偏地评估HEIMDALL，我们使用来自阿里巴巴、微软和腾讯的数百个随机时间窗口（“踪迹”）进行了大规模评估。为确保这数百个踪迹代表各种工作负载特征，我们根据五个标准选择了踪迹，即读/写比率、大小、IOPS、随机性和总体排名。对于每个标准，我们选择了具有p10、p25、p50、p75、p90和p100值的时窗，这些值是相对于长达数天的踪迹中的所有时窗而言的。

为了进一步增加我们数据集的多样性，我们还应用了5种不同的数据增强函数（$0.1\times$ rerate, $0.5\times$ rerate, $2\times$ rerate, $2\times$ resize, and $4\times$ resize）。这种方法模拟了比实际应用中通常遇到的更具挑战性的场景。从这个大型数据集中，我们随机挑选了500个踪迹。每个踪迹随后被限制在3分钟长，其中包含10万到1000万个I/O。一个3分钟的踪迹足够长，足以让底层设备因GC、写放大和其他争用而表现出一些尾延迟，但同时又足够短，适合大规模实验。

为确保评估的真实性，我们专注于轻-重工作负载组合，反映了负载条件变化的真实世界场景，如果I/O选择处理不当，可能导致尾延迟。如果I/O计数少于30万，我们将踪迹分类为轻。在这种组合中，避免盲目地将重踪迹的I/O重路由到轻踪迹至关重要。这样做可能会无意中使另一台设备过载，导致整体性能下降。一个高效的模型应该只在绝对必要时才拒绝和重路由I/O。

与踪迹来源一致，我们的踪迹不包括线程ID。相反，我们遵循标准做法，通过在客户端部署多个线程 $(N\geq 8)$ 来执行并发I/O操作。这些线程根据I/O的时间戳并行提交I/O流，遵循并发应用程序发出I/O操作的行为。此外，我们在消费级和企业级SSD上进行了全面测试，这些SSD在重负载下容易发生垃圾回收（GC）。

为了更快地进行实验设置（仅限本小节），我们将HEIMDALL部署在一个用户级的I/O重放器中，以简单模拟带有直接I/O的应用级存储。对于每个实验，我们模拟一个双向复制的存储环境，使用一台配备2个三星SSD 970 PRO 1TB SSD的机器。在每个实验期间，我们运行一个随机踪迹，其中的I/O将通过特定于该设备的HEIMDALL模型，该模型决定I/O接纳决策。如果一个I/O被拒绝，它将被重定向到另一台设备，默认情况下会被接纳。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/00d400067f3abe8d3109209dbd4ed6131424cf70c817e4ae2280ebc76db56b13.jpg)  
图11. 大规模评估 ( § 6.1 )。子图 (a) 和 (b) 描述了从 $p50$ 到 $p99.99$ 百分位的读取延迟和平均延迟。

为了恰当地评估HEIMDALL，我们将其与几个著名的策略进行了比较，包括简单的始终接纳无重路由（基线）、随机目标选择（即，将I/O发送到随机选择的设备），以及最先进的接纳和重路由算法，如C3、AMS、Heron、LinnOS和对冲（hedging）。在第一个实验中，我们专注于从基于启发式的类别中选择一个代表性算法，因为许多提出的算法都属于这一类。这个选择简化了后续的评估，同时仍然捕捉了该算法组的最佳性能。如图10所示，我们评估了三个关键的基于启发式的算法：AMS、C3和Heron。结果显示C3和AMS的性能非常相似，两者都比Heron提供了更低的延迟。鉴于C3在业界的广泛采用及其有竞争力的性能，我们选择C3作为基于启发式算法的代表，并在下一个实验以及稍后的内核评估中(§6.2)进一步评估它。

在此基础上，图11a突出了在不同百分位上的尾延迟，进一步说明了HEIMDALL优于最先进算法。这些延迟是500个实验的平均百分位，因此显示HEIMDALL在大规模、无偏的实验中获胜。此外，图11b显示了HEIMDALL在提供最低平均延迟方面的另一个令人印象深刻的结果。

我们还可以看到，虽然对冲在p99以上提供了更短的尾延迟，但其平均延迟远差于HEIMDALL。例如，对冲在p98（在$0.75ms - 1.5ms$之间），在2ms超时后，会发送一个备用I/O，导致过多的过载（不稳定性），从而导致比基线更高的平均延迟。因此，对于低延迟请求，对冲似乎是无效的。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/275f1eafc07d67679fc4fd89132abdd3985ed0e0942b0f119b71fcce6ecdd79e.jpg)  
图12. 内核级评估 (§6.2)。HEIMDALL 在 (a) $p50$ 到 $p99.99$ 百分位实现了最稳定的延迟，并 (b) 实现了最低的平均延迟。

# 6.2 内核级评估

虽然上一节展示了用户级部署，本节简要展示了HEIMDALL在部署于Linux内核块层内部时的结果。由于我们之前已经用同构的数据中心三星SSD 970 PRO 1TB SSD进行了大规模实验，这里我们提供了一个不同的设置，在一台配备两个消费级SSD（Intel DC-S3610和Samsung PM961）的机器上运行一部分微软踪迹。如图12a所示，HEIMDALL在内核部署和异构SSD上也同样有效，通过在不同百分位（x轴）上提供更快的延迟（y轴）来优于其他方法。图12b进一步显示了HEIMDALL在内核中的成功部署，提供了最低的平均延迟，与非基线方法相比快了 $38 - 48\%$。

# 6.3 广域评估

到目前为止，我们已经在单台机器上分析了HEIMDALL的多驱动器性能。本节通过将HEIMDALL部署在Ceph分布式存储系统中，展示了其在“广域”设置下的性能。为此，我们使用了10台Chameleon Ice Lake机器，每台机器配备两个 $2.30\mathrm{GHz}$ 40核Intel(R) Xeon(R) Platinum 8380 CPU和256 GB DRAM。在每台机器上，我们部署两个Ceph对象存储守护进程（OSD），以设置一个带主备OSD的复制存储。由于Chameleon集群中双SSD机器的可用性有限，我们在FEMU模拟的SSD（每个100 GB）之上设置了OSD。为了向这20个OSD发送请求，我们创建了20个客户端节点，并运行噪声注入器，以观察接纳策略如何对有噪声的邻居做出反应。根据前一节的结果，我们比较了三种方法：基线、随机和HEIMDALL。在基线Ceph设置中，每个请求都指向主OSD，而在随机设置中，请求是随机负载均衡的。LinnOS被排除在此次评估之外，因为它只支持固定的4KB每页预测，这使其与Ceph的可变大小I/O操作不兼容。由于Ceph工作负载涉及跨分布式存储环境的各种请求大小，像LinnOS这样的系统无法有效泛化，因为它被设计用于统一的、页面级的决策。这进一步凸显了

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/905a9f88babe0a16e385e0c866b1b39494df227d2562ccea0b92eaca9f900b73.jpg)  
图13. 广域评估 (§6.3)。Ceph集群上的CDF延迟，(a) $SF = 1$, (b) $SF = 10$; (c) 在各种 $SF$ 下，从 $p50$到$p95$百分位的延迟降低情况。

HEIMDALL的灵活性，因为它能无缝集成到Ceph和其他大规模系统中，使其更易于部署和适应现实世界的存储环境。

图13a中的延迟CDF显示，在广域评估中，HEIMDALL也提供了最好的结果。此外，我们还改变了“扩展因子”（SF）。根据谷歌开创性的“Tail at Scale”论文，一个最终用户请求可以包含多个到不同目标服务器的并行“子请求”，但只有当所有子请求都完成时，最终用户请求才被认为完成。为了显示HEIMDALL在尾部被规模放大时的好处，我们改变了 $SF$ 因子（例如，$SF = 10$ 意味着一个最终用户请求有10个并行的子请求）。图13b显示了当 $SF = 10$ 时的延迟CDF，表明尾部行为在基线中从 $p75$ 开始出现，而HEIMDALL可以有效地削减大部分尾部区域。此外，图13c显示了HEIMDALL相对于随机的尾延迟降低百分比（y轴），在不同百分位（x轴）和各种扩展因子（如图例所示）下的情况。HEIMDALL在所有场景中都获胜（延迟降低百分比为正）。

# 6.4 准确性

HEIMDALL强大性能的背后是其实现的高准确性。本节剖析了每个设计决策如何为提高HEIMDALL的整体准确性做出贡献。我们使用的准确性主要有五个指标：ROC-AUC、PR-AUC、F1-Score、FNR和FPR，基于真/假阳性和阴性的数量。它们的方程可以在这里找到。在我们的案例中，真阳性（TP）意味着模型正确地将I/O识别为“慢”，而假阳性（FP）则相反（标记为“慢”，但如果提交到设备，I/O实际上会是快的）。

在我们的领域，ROC-AUC是不平衡数据集的首选指标，因为它平衡了灵敏度和特异性。更具体地说，与快速延迟相比，尾延迟是少数。与准确率或精确率不同，ROC-AUC提供了一个全面的评估，考虑了在各种分类阈值下真阳性率和假阳性率之间的权衡。

如图14a所示，我们测量了每一步优化（y轴）所得到的ROC-AUC分数（x轴）。这些优化有助于增强数据集和模型，从而在每一步都提高了HEIMDALL的整体

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/6f5e82e61ba02af8203be83352acb4be390e9ea0a278709dad62c72953b3cdfc.jpg)  
图14. 准确性评估 (§6.4)。每一步的贡献：比较 (a) 仅ROC-AUC 和 (b) 从LinnOS到Heimdall设计步骤的所有指标。基本标记(LB)，特征缩放(FC)，精确标记(LA)，特征提取(FE)，特征选择(FS)，模型工程(M)，和噪声过滤(LN)。

准确性。下面的编号 $(n)$ 对应于图14a中的y轴值。

(0) 我们首先测量了LinnOS的准确性作为基线，在超过100个随机数据集上平均为 $67\%$。在测量LinnOS的准确性时，为确保公平比较，我们使用了LinnOS的架构，并在测试期间应用了与我们的Heimdall架构相同的设置和条件。与作者四年前在论文中报告的相比，LinnOS的准确性显著下降。鉴于现代SSD的不同行为（更快的延迟）和社区发布的更新的工作负载，该模型已经过时，这凸显了用更广泛的ML流水线重新设计模型的必要性。

(1) 在这里，我们开始使用LinnOS的原始特征集、模型架构和基本的基于截止点的标记（LB）来开发Heimdall。然而，我们移除了数字化——LinnOS中使用的特征缩放技术——因为它专门为每页（4KB）I/O接纳设计，并假设I/O大小统一。由于Heimdall的主要方法从每页决策转向处理可变大小的I/O，保留数字化会引入偏差并扭曲学习过程。移除它确保了更有意义的比较，使我们能够直接评估基于周期的标记在更广泛的I/O接纳场景中的影响。这一调整导致准确性下降了 $16.8\%$，降至 $50.2\%$。然而，这次下降是有价值的，因为它建立了一个受控的下限，为量化由基于周期的标记和后续优化引入的改进提供了一个清晰的基线。通过在这一阶段将我们的重点转移到可变大小的I/O，我们创建了一个更灵活、更现实的评估框架，更好地反映了现代存储工作负载。

(2) 然后，我们应用了min-max缩放（FC）（§3.3）来归一化输入特征，而不是使用数字化（如LinnOS中），结果准确性略有提高 $(67.5\%)$，与LinnOS相当。

(3) 之后，我们精心设计了我们更准确的基于周期的标记（LA）方法（§3.1），并用它取代了LinnOS的基于截止点的标记。结果，我们将准确性提高了 $5.5\%$，达到 $73\%$。

(4) 然后，通过额外的特征提取（FE）（§3.3），包括I/O大小和历史吞吐量，准确性又获得了 $4\%$ 的提升（现在达到 $77\%$），因为Heimdall现在可以辨别与数据传输量和速率相关的模式。

(5) 为了降低模型的推理开销，我们应用了一种特征选择（FS）方法（§3.3），该方法在不引起性能下降的情况下保持了准确性。

(6) 同样，我们采用了模型工程（M）（§3.4和§3.5）步骤，包括学习任务探索（ML）、模型探索（ME）、超参数调优（MT）和验证（MV），以在模型的复杂性和准确性之间找到平衡。我们获得了一个极简模型，能够维持迄今为止达到的相同准确性水平。

(7) 最后，我们的噪声过滤（LN）（§3.2）被证明是最重要的贡献之一，它将准确性提高了 $16\%$，从而使模型准确性达到 $93\%$。

此外，为了表明我们没有偏向于某一个准确性指标，图14b显示了在我们逐步添加贡献（x轴）时，五个准确性指标（五条线）的最终准确性（y轴）。总体而言，随着更多优化的引入，ROC-AUC、PR-AUC和F1-Score持续增加。此外，假阴性率和假阳性率（FNR和FPR）也如预期般持续下降。

# 6.5 联合/组推理

我们现在讨论Heimdall的联合推理性能（§4.2）。如图15a所示，在Heimdall的默认设置下（没有联合推理，由联合大小 $= 1$ 表示），它只能接收 $0.5 \mathrm{mLOPS}$ 的工作负载，然后其延迟就会飙升至 $2\mu \mathrm{s}$。然而，当联合大小 $= 9$ 时，即使在1个CPU核心上面临 $4 \mathrm{mLOPS}$ 的工作负载（比默认Heimdall重 $8\times$），Heimdall也能将延迟维持在 $2\mu \mathrm{s}$ 以下。请注意，这个延迟包括排队延迟，使其比我们最快的推理延迟（§4.1）要慢。联合推理会降低准确性，如图15b量化所示。例如，从默认的Heimdall过渡到 $9 \mathrm{I/O}$ 的联合推理，中位值的准确性从 $88\%$ 下降到 $81\%$。该图显示了在50个随机数据集上的准确性分布。鉴于上述结果，我们认为联合大小 $= 3$ 对于平衡吞吐量/准确性的权衡是合适的。在部署Heimdall时，存储管理员也可以动态调整联合大小。

我们通过与LAKE进行比较，进一步评估了联合推理的有效性。LAKE通过在内核空间提供更便捷的GPU访问来增强LinnOS的推理吞吐量，

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/e44cad0f655a0f88fe248b8187cd0e4f96a471b8338f0903dddd390860a75851.jpg)  
图15. 联合推理 (§6.5)。(a) 吞吐量稳定性，(b) 模型准确性，以及 (c) 与LAKE的比较。

使得批处理ML推理可以卸载到GPU。为了更好的比较，我们实现了三个版本的HEIMDALL：GPU批处理（由LAKE的机制加速）、CPU批处理和基于CPU的联合推理。图15c展示了在GeForce GTX 1660 SUPER GPU上的结果，我们将同时推理的I/O数量从1变化到128。首先，在GPU加速下，LAKE和HEIMDALL的性能相似，HEIMDALL最多有 $0.01\mathrm{ms}$ 的优势。其次，与基于GPU的方法（LAKE和HEIMDALL GPU批处理）相比，HEIMDALL基于CPU的联合推理将推理延迟降低了高达 $10\times$。第三，与HEIMDALL CPU批处理相比，当同时处理的I/O数量增加时，HEIMDALL基于CPU的联合推理具有更低的推理延迟。这是因为批处理的计算复杂度为 $N\times$，其中 $N$ 是批处理大小。然而，联合推理在多个I/O之间共享相似的输入特征，并将它们合并为单个推理，使得在批处理大小较大时计算开销可以忽略不计。因此，带有联合推理的HEIMDALL可以容忍更多I/O密集型场景，并提供更好的可访问性，而无需依赖GPU。

# 6.6. CPU和内存开销

我们现在评估HEIMDALL的内存和CPU开销。图16显示，与LinnOS相比，我们的模型实现了(a) $2.4\times$ 更少的内存开销，68KB vs. 28KB，以及(b) $2.5\times$ 更少的CPU开销。HEIMDALL总共有3472个权重和偏置，小于LinnOS的8706个。此外，我们的模型乘法运算少了 $2.4\times$，3472 vs. LinnOS的8448次。而且，HEIMDALL做出的推理决策要少得多，因为它是在每个I/O的基础上操作，而不是像LinnOS那样基于每页。为了进一步减少开销，HEIMDALL可以部署联合大小 $= 3$（由HEIMDALL-J3，浅蓝色条表示），与LinnOS相比，CPU开销减少了 $85\%$。

# 6.7 训练时间

最后，我们报告HEIMDALL的平均训练时间，这取决于I/O的数量。对于我们用于训练的每一百万个I/O，

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/1999ed8329f141752dace3f1a833808819e94247f84e41480497e9f924879913.jpg)  
图16. 开销 (§6.6)。(a) 内存 & (b) CPU。

在CPU上需要16.8秒的训练时间，在GPU上需要3.7秒。预处理包括标记、提取特征、归一化和打乱数据。我们对每个踪迹文件训练模型一次，假设每个踪迹属于不同的硬件和工作负载，因此在使用前需要一个学习阶段。下一个要回答的问题是，需要多少数据进行训练，以及在长期部署场景中我们应该如何重新训练模型。这个重要的研究问题超出了本文的范围。回答它需要对ML流水线进行进一步的探索，我们将在下一节中深入探讨。

# 7 长期部署的再训练

在现实中，ML模型是长期部署的。在此背景下，我们通过在一个8小时的真实世界踪迹上测试HEIMDALL，进行了一项初步的长期评估，该踪迹是准确性长期波动的最“具挑战性”的踪迹之一。在这里，我们使用了一个腾讯踪迹，其中写IOPS是读IOPS的 $2\times$ 多，从而触发了更多的GC活动。此外，该踪迹表现出几乎恒定的I/O到达间隔时间，导致所有设备都经历相似的工作负载和繁重的利用率。

图17a显示了HEIMDALL在使用踪迹的最初1、5和15分钟进行单次训练后的准确性。我们在一个10分钟的窗口内测量准确性（一个点是模型在过去10分钟内的平均准确性）。我们可以得出两个简单的结论。首先，更长的训练踪迹（例如，15分钟的踪迹）会带来更好的长期整体准确性，但需要更长的训练时间(§6.7)。其次，准确性随时间波动，最小-最大准确性为 $63\% - 82\%$。这也被称为模型的性能漂移，可能源于工作负载行为的变化（输入漂移）、设备/环境变化（概念漂移）等因素。

为了解决这个问题，我们建立了一个初步的再训练策略，该策略每分钟监控模型的准确性，并在准确性低于 $80\%$ 时触发再训练。为了保持再训练的轻量级，我们只使用触发前的最后一分钟数据进行再训练。图17b显示了结果，其中垂直蓝线表示触发再训练的时间。更具体地说，在这个8小时的窗口内，再训练发生了 $37\times$ 次，每次平均使用81.6万个I/O，

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/e61079940dd55b9cf4a30374b9c866a424270afd663bae8f9b746bdb9238aa59.jpg)  
图17. 长期评估 (§7)。HEIMDALL使用不同训练方法的性能：“First N min”仅使用工作负载的前N分钟训练模型一次，而“Retrain”使用§7中描述的简单再训练策略。垂直蓝线标记了触发再训练的时间。

这可以在几秒钟内完成(§6.7)。这个初步结果也指向了更多的研究问题。例如，连续的再训练实例（彼此靠近的垂直蓝线）表明某些再训练决策可能没有用。其次，我们不能指望再训练会话前的最后一分钟踪迹是可用的，因为由于显著的开销，默认情况下逐请求日志记录是关闭的。

总的来说，这些发现表明，在这个面向存储的长机器学习流水线中（如图1所示）有更多的主题需要探索，例如高效的再训练、持续学习模型和模型管理。有许多研究问题需要提出。何时再训练模型？如何在再训练期间避免灾难性遗忘？如何检测性能漂移？哪些I/O特性可以提供工作负载漂移的线索？多久以及使用多少数据来检查漂移？

# 8 讨论与未来工作

鉴于HEIMDALL在Linux内核和Ceph存储系统等真实世界部署中，实现了高准确性、降低了尾延迟，同时保持了亚微秒级的开销，我们相信它的成就将引发讨论，并为未来的研究提出重要问题：

# 8.1 HEIMDALL的更广泛影响

HEIMDALL是在严谨细致地应用各种机器学习方法，并融入深厚领域知识的成果。将这种领域专长融入到基于ML的解决方案设计中被证明是至关重要的。因此，与以往解决同一问题的方法相比，HEIMDALL取得了显著的性能提升。因此，尽管ML在存储领域的应用激增，我们相信ML的全部潜力尚未被释放。

为了评估这种更长的ML流水线在多大程度上应用于面向存储的ML研究，我们在表2中总结了近期的研究论文。表中的五个行组代表了流行存储领域的论文，例如接纳、缓存、索引、预取和杂项类别。我们得出结论，面向存储的ML文献在探索该流水线的各个阶段方面存在差距。

例如，在数据标记阶段，先前的工作只采用一种标记方法，因此为通过将领域知识整合到更“精确的标记”和可能的“噪声过滤”中来提高准确性，以避免“垃圾进，垃圾出”提供了机会。在特征工程阶段，“特征缩放”没有得到充分研究，可能导致模型对某些特征赋予不成比例的重要性。最后，应用我们的“联合/组推理”以提高效率并使其可用于生产系统部署有很大的机会。许多研究也没有考虑长期部署中“再训练/再学习”的需求，可能是在“多层/多目标”中。因此，在提升ML驱动应用的性能和可靠性方面存在巨大机遇。

如表2最后一行所强调的，HEIMDALL的流水线涵盖了更多的机器学习方法。这种更严谨的方法论归功于HEIMDALL在评估部分(§6)中展示的最佳性能。将我们在本文中设计的每种方法应用于表2中的所有论文超出了我们的范围，将作为未来工作。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-07/a381fd85-226a-44cb-96e7-e5d9f77801b7/d57a4b64ddc25e4bc01c6887c6259125901074e63db924273a37071a85e8ba92.jpg)  
图18. HEIMDALL vs. AutoML (§8)。HEIMDALL和AutoML在以下方面的比较：(a) 准确性，(b) 探索时间，(c) 模型泛化能力。

# 8.2 迈向自动化的HEIMDALL流水线

在理解了将领域知识注入以选择和应用合适技术所付出的努力之后，人们可能会想，自动化机器学习（“AutoML”）是否可以取代我们的手动方法。AutoML确实可以促进快速的设计原型制作，减少人力劳动。然而，有三个限制阻碍了AutoML的可行性：(1) 在理解和应用领域知识方面的限制，(2) 探索计算开销，以及 (3) 部署考虑。为了展示这些限制，我们使用了auto-Sklearn，一个自动进行ML算法选择及其超参数调优的先进框架。

在第一个实验中，我们随机挑选了50个数据集，并将原始数据集提供给AutoML框架，而没有进行我们所做的手动特征工程步骤(§3.3)。我们使用了图18中所示的各种算法类别下的16个AutoML分类器。这是故意的，以观察在用户付出最少可能操作的情况下，AutoML方法能提供多大帮助。

使用auto-Sklearn，AutoML自主进行超参数调优。图18a比较了AutoML模型（顶行）和HEIMDALL（最后一行）的ROC-AUC（x轴）。AutoML模型表现出次优性能，平均准确性比HEIMDALL低 $22\%$。这个结果与我们的预期一致：鉴于AutoML仅使用原始特征集，而没有领域特定的派生特征（例如，队列长度），AutoML模型在识别特征集与结果标签之间的有意义相关性方面是次优的。虽然可以配置AutoML框架来探索派生特征，但该过程将显著增加执行时间。

在第二个实验中，如图18b所示，我们强调了AutoML在探索期间会产生高昂的探索时间，导致显著的计算开销。具体来说，AutoML模型的探索时间（x轴）从1.8小时到4.8小时不等。这是由于AutoML模型的固有探索性质和无界复杂性。需要注意的是，该实验是在CPU上进行的，因为auto-Sklearn默认不支持GPU加速。HEIMDALL在CPU上的训练时间可以通过将训练代码移植到C/C++并利用CPU优化的训练库来进一步优化（尽管超出了本文的范围）。未来的工作可以探索优化AutoML模型并降低其训练复杂性，因为它们的详尽探索仍然是高效部署的挑战。

在第三个实验中（图18c），我们评估了AutoML生成的模型在不同数据集上的泛化能力。结果显示，AutoML为每个数据集创建了架构差异很大的模型，而HEIMDALL则对数据集保持不可知。我们通过计算模型的余弦相似度（对数刻度的x轴）来量化这种泛化能力的缺乏，其中HEIMDALL始终保持相似度得分为1。相比之下，AutoML模型表现出较差的跨数据集泛化能力（余弦相似度 < 0.01）。这表明AutoML生成的模型在不同数据集之间不可重用，需要为每个新工作负载重新探索和重新训练，这给生产系统部署带来了巨大成本。总而言之，尽管将HEIMDALL转变为一个类似于AutoML的全自动系统有巨大潜力，但我们将其作为未来工作。

# 9 结论与竞赛

本文介绍了HEIMDALL，一个以性能和部署可行性为重点设计的ML驱动的I/O接纳控制。HEIMDALL显著降低了尾延迟，同时保持了亚微秒的推理开销。我们广泛的评估证明了它在多种部署环境中的有效性，包括用户级存储、内核内系统和分布式存储集群。除了对I/O接纳控制的直接影响外，HEIMDALL的可扩展ML流水线为ML驱动的存储优化领域的更广泛研究提供了基础。为鼓励进一步创新，我们已将HEIMDALL用作“迷你竞赛”的试验平台，灵感来自ImageNet和Kaggle等流行竞赛。在这样一项工作中，15名学生参与探索新技术，实验了20种数据增强形式、35种分类模型和20种回归模型，以及如采样和量化等训练策略。我们公开发布HEIMDALL的代码，希望它能造福研究社区——不仅在推进I/O接纳策略方面，也在探索其他关键的存储优化和系统级ML应用方面。

# 致谢

我们感谢我们的指导人Yubin Xia和匿名审稿人给予的巨大反馈和意见。我们也想感谢Erci Xu对本文的有用讨论和评论。本材料得到了NSF（拨款号CCF-2119184、CNS-2402327、CNS-2027170和CNS-2411425）以及来自Google、Meta、NetApp和Seagate的慷慨捐赠的支持。本文中的实验是在Chameleon上进行的。本文中表达的任何观点、发现、结论或建议均为作者的观点，不一定反映NSF或其他机构的观点。

# A 工件附录

# A.1 摘要

A.1 摘要 该工件记录了Heimdall的数据科学流水线、联合推理技术，以及客户端和内核部署。

# A.2 描述与要求

实验说明文件中有4个实验，所有步骤都在文档文件夹下给出。

[如何访问] 源代码在 https://github.com/ucare-uchicago/Heimdall.git 上公开可用，DOI 10.5281/zenodo.14874299。

[硬件依赖] 客户端级和内核级集成必须在具有两个未挂载SSD的机器上运行，而其他实验可以在任何机器上运行。

[基准测试] 我们使用来自微软、阿里巴巴和腾讯的公共I/O踪迹。这些踪迹是§6.1中解释的预处理的示例结果。

# A.3 评估工作流程

A.3.1 主要声明。请注意，由于大规模实验耗时较长，我们为工件展示了一个随机的小数据集样本。

首先，通过多项领域特定的设计，包括基于周期的标记和三阶段噪声过滤，Heimdall的数据科学流水线将决策准确率从 $63\%$ 提高到 $93\%$，与LinnOS相比（实验E1）。

其次，通过联合推理，Heimdall在比默认重8倍的工作负载（4 mIOPS）下仍保持低开销，同时准确率仍高于 $80\%$（实验E2）。

第三，在C客户端级别并使用500个工作负载的情况下，Heimdall相比其他算法实现了更低的平均延迟和尾延迟（实验E3）。

第四，Heimdall在Linux内核原型中实现了更低的平均和尾延迟（实验E4）。

A.3.2 实验。每个模拟实验需要不到30分钟的人工时间，而原型实验可能需要长达3.5小时。我们建议在Chameleon测试平台上运行这些实验。

\$实验 (E1): Heimdall-Pipeline [15分钟人工时间]：评估Heimdall的决策准确性。

[准备] 请按照测试平台预订指南在CHI@TACC站点预订一个storage_hierarchy节点。然后，请遵循文档。

[执行] 首先，在没有HEIMDALL帮助的情况下重放踪迹，并分析重放结果。我们首先编译踪迹重放器并运行示例踪迹。然后，我们研究我们重放的踪迹。尾部分析器脚本作为我们分析的一部分，生成了重放踪迹的深刻特征剖析。接下来，我们进行基于周期的标记、特征提取和选择，最后进行模型训练。

\$实验 (E2): 联合推理 [5分钟人工时间]：评估使用联合推理时的准确性损失。

[准备] E2的准备阶段与E1相同，E2的环境将与E1相似。

[执行] 为了准备联合推理的训练，在这一步中，我们将多个I/O合并为一个具有扩展特征和对齐标签的I/O。然后我们训练模型，部署并在测试集上评估准确性。您可以更改-batch_size参数的值，观察推理加速和准确性的权衡。

\$实验 (E3): 客户端级 [60分钟人工时间]：评估最终的I/O延迟和尾部情况。

[准备] 请预订并设置一台在Chameleon Cloud上有两个未挂载SSD的机器。

[执行] 文档概述了运行Heimdall与LinnOS、Random、Hedging和LinnOS+Hedging进行比较的步骤。Heimdall和LinnOS的训练可以并行进行，而重放必须按顺序运行。为每个算法运行脚本后，运行脚本分析结果并获得延迟CDF图。

\$实验 (E4): Linux内核部署 [60分钟人工时间]：评估延迟改进。

[准备] 请遵循测试平台预订指南。克隆工件仓库后，按照文档设置机器。

[执行] 编译Heimdall内核并更新grub配置。接下来，重启机器以让编译后的内核启动。重启后，您可以通过`uname -r`检查内核是否成功更改，预计输出为`6.0.0-heimdall`。要训练Heimdall模块，我们需要配置设备和要测试的踪迹。请在`$HEIMDALL_KERNEL/config/config.conf`中更改两个变量：

SSD_DEVICE0: 未挂载的SSD作为主副本。
SSD_DEVICE1: 未挂载的SSD作为故障转移副本。

配置后，您现在可以训练模型、重放踪迹并绘制结果。您应该会看到基线（无I/O接纳和预测）、随机和Heimdall的结果。将生成两个图，一个展示平均读取I/O延迟，另一个绘制延迟百分位。
# References

[1] [n.d.]. Alibaba Block Traces. http://github.com/alibaba/block- traces. [2] [n.d.]. Chameleon - A configurable experimental environment for large- scale cloud research. https://www.chameleoncloud.org. [3] [n.d.]. Kaggle: Your Machine Learning and Data Science Community. https://www.kaggle.com/. [4] [n.d.]. The Evolution of Image Classification Explained. https://stanford.edu/\~shervinov/blog/evolution- image- classificationexplained. [5] 2021. Colossus under the hood: a peek into Google's scalable storage system. https://cloud.google.com/blog/products/storage- datatransfer/a- peek- behind- colossus- googles- file- system. [6] 2022. Alibaba Block Traces. https://github.com/alibaba/block- traces. [7] 2024. Reflecting on 2023- Azure Storage. https://azure.microsoft.com/en- us/blog/reflecting- on- 2023- azurestorage/. [8] Najmeddine Abdennour, Tarek Ouni, and Nader Ben Amor. 2021. The importance of signal pre- processing for machine learning: The influence of Data scaling in a driver identity classification.. In ACS 18th International Conference on Computer Systems and Applications (AICCSA). [9] Ibrahim Umit Akgun, Ali Selman Aydin, Andrew Burford, Michael McNeill, Michael Arkhangelskiy, and Erez Zadok. 2023. Improving Storage Systems Using Machine Learning. In ACM Transaction on Storage. [10] Jacob Alter, Ji Xue, Alma Dimnaku, and Evgenia Smirni. 2019. SSD failures in the field: symptoms, causes, and prediction models. In Proceedings of International Conference on High Performance Computing, Networking, Storage and Analysis (SC). [11] Ganesh Ananthanarayanan, Ali Ghodsi, Scott Shenker, and Ion Stoica. 2013. Effective Straggler Mitigation: Attack of the Clones. In Proceedings of the 10th Symposium on Networked Systems Design and Implementation (NSDI). [12] Shane Bergsma, Timothy Zeyl, Arik Senderovich, and J. Christopher Beck. 2021. Generating Complex, Realistic Cloud Workloads using Recurrent Neural Networks. In Proceedings of the 29th ACM Symposium on Operating Systems Principles (SOSP). [13] Andrew P. Bradley. 1997. The use of the area under the ROC curve in the evaluation of machine learning algorithms. In The Journal of the Pattern Recognition Society Volume 30. [14] Kishan K. C., Rui Li, and Mahdi Gilany. 2021. Joint inference for neural network depth and dropout regularization.. In Proceedings of the 35th Conference on Neural Information Processing Systems (NIPS). [15] Yu Cai, Saugata Ghose, Erich F. Haratsch, Yixin Luo, and Onur Mutlu. 2017. Errors in Flash- Memory- Based Solid- State Drives: Analysis, Mitigation, and Recovery. In Computing Research Repository. [16] Zhichao Cao, Siying Dong, Sagar Vemuri, and David H. C. Du. 2020. Characterizing, Modeling, and Benchmarking RocksDB Key- Value Workloads at Facebook. In Proceedings of the 18th USENIX Symposium on File and Storage Technologies (FAST). [17] Zhen Cao, Geoff Kuenning, and Erez Zadok. 2020. Carver: Finding Important Parameters for Storage System Tuning. In Proceedings of the 18th USENIX Symposium on File and Storage Technologies (FAST). [18] Haihua Chen, Jiangping Chen, and Junhua Ding. 2021. Data Evaluation and Enhancement for Quality Improvement of Machine Learning. In IEEE Transactions on Reliability. [19] Quan Chen, Hailong Yang, Minyi Guo, Ram Srivatsa Kannan, Jason Mars, and Lingjia Tang. 2017. Prophet: Precise QoS Prediction on Non- Preemptive Accelerators to Improve Utilization in Warehouse- Scale Computers. In Proceedings of the 22nd International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS).

[20] Andrew Chung, Subru Krishnan, Konstantinos Karanasos, Carlo Curino, and Gregory R. Ganger. 2020. Unearthing inter- job dependencies for better cluster scheduling. In Proceedings of the 14th Symposium on Operating Systems Design and Implementation (OSDI). [21] Yifan Dai, Yien Xu, Aishwarya Ganesan, Ramnatthan Alagappan, Brian Kroth, Andrea C. Arpaci- Dusseau, and Remzi H. Arpaci- Dusseau. 2020. From WiscKey to Bourbon: A Learned Index for Log- Structured Merge Trees. In Proceedings of the 14th Symposium on Operating Systems Design and Implementation (OSDI). [22] Anwesha Das, Frank Mueller, Paul Hargrove, Eric Roman, and Scott B. Baden. 2018. Doomsday: predicting which node will fail when on supercomputers. In Proceedings of International Conference on High Performance Computing, Networking, Storage and Analysis (SC). [23] Jesse Davis and Mark Goadrich. 2006. The relationship between Precision- Recall and ROC curves. In Proceedings of the 23rd International Conference on Machine Learning (ICML). [24] Jeffrey Dean and Luiz Andre Barroso. 2013. The Tail at Scale. Communications of the ACM (CACM) 56, 2 (2013). [25] Matthias Feurer, Aaron Klein, Katharina Eggensperger, Jost Tobias Springenberg, Manuel Blum, and Frank Hutter. 2015. Efficient and Robust Automated Machine Learning. In Proceedings of the 30th Conference on Neural Information Processing Systems (NIPS). [26] Henrique Fingler, Isha Tarte, Hangchen Yu, Ariel Szekely, Bodun Hu, Aditya Akella, and Christopher J. Rossbach. 2023. Towards a Machine Learning- Assisted Kernel with LAKE. In Proceedings of the 28th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS). [27] Yu Gan, Mingyu Liang, Sundar Dev, David Lo, and Christina Delimitrou. 2021. Sage: practical and scalable ML- driven performance debugging in microservices. In Proceedings of the 26th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS). [28] Yu Gan, Yanqi Zhang, Kelvin Hu, Dailun Cheng, Yuan He, Meghna Pancholi, and Christina Delimitrou. 2019. Seer: Leveraging Big Data to Navigate the Complexity of Performance Debugging in Cloud Microservices. In Proceedings of the 24th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS). [29] Klaus Greff, Antti Rasmus, Mathias Berglund, Tele Hotloo Hao, Jurgen Schmidhuber, and Harri Valpota. 2016. Tagger: Deep Unsupervised Perceptual Grouping. In Proceedings of the 30th Conference on Neural Information Processing Systems (NIPS). [30] Ajay Gulati, Arif Merchant, and Peter J. Varman. 2010. mClock: Handling Throughput Variability for Hypervisor IO Scheduling. In Proceedings of the 9th Symposium on Operating Systems Design and Implementation (OSDI). [31] Mingzhe Hao, Huaicheng Li, Michael Hao Tong, Chrisma Pakha, Riza O. Suminto, Cesar A. Stuardo, Andreja A. Chien, and Haryadi S. Gunawi. 2017. MittOS: Supporting Millisecond Tail Tolerance with Fast Rejecting SLO- Aware OS Interface. In Proceedings of the 26th ACM Symposium on Operating Systems Principles (SOSP). [32] Mingzhe Hao, Levent Toksoz, Nanqinjin Li, Edward Edberg Halim, Henry Hoffmann, and Haryadi S. Gunawi. 2020. LinnOS: Predictability on Unpredictable Flash Storage with a Light Neural Network. In Proceedings of the 14th Symposium on Operating Systems Design and Implementation (OSDI). [33] Vikas Jaiman, Sonia Ben Mokhtar, Vivien Quema, Lydia Y. Chen, and Etienne Riviere. 2018. Héron: Taming Tail Latencies in Key- Value Stores Under Heterogeneous Workloads. In The 43rd International Symposium on Reliable Distributed Systems (SRDS 2018). [34] Virajith Jalaparti, Peter Bodik, Srikanth Kandula, Ishai Menache, Mikhail Rybalkin, and Chenyu Yan. 2013. Speeding up distributed request- response workflows. In Proceedings of the ACM Special Interest Group on Data Communication (SIGCOMM).

[35] Tianyang Jiang, Guangyan Zhang, Zican Huang, Xiaosong Ma, Junyu Wei, Zhiyue Li, and Weinin Zheng. 2021. FusionRAID: Achieving Consistent Low Latency for Commodity SSD Arrays. In Proceedings of the 19th USENIX Symposium on File and Storage Technologies (FAST). [36] Wanchun Jiang, Yujia Qiu, Fa Ji, Yongjia Zhang, Xiangqian Zhou, and Jianxin Wang. 2023. AMS: Adaptive Multiget Scheduling Algorithm for Distributed Key- Value Stores. In IEEE Transactions on Cloud Computing (TCC). [37] Wanchun Jiang, HaiMing Xie, Xiangqian Zhou, Liyuan Fang, and Jianxin Wang. 2019. Hasle makes waste: The On- Off algorithm for replica selection in key- value stores. In Journal of Parallel and Distributed Computing. [38] Wanchun Jiang, HaiMing Xie, Xiangqian Zhou, Liyuan Fang, and Jianxin Wang. 2019. Understanding and improvement of the selection of replica servers in key- value stores. In Information Systems Journal (IS), Volume 83. [39] Michael I. Jordan and Robert A. Jacob. 1993. Hierarchical mixtures of experts and the EM algorithm. In IEEE International Joint Conference on Neural Network (IJCNN). [40] Sangeetha Abdu Jyothi, Carlo Curino, Ishai Menache, Shravan Matthur Narayanamurthy, Alexey Tumanov, Jonathan Yaniv, Ruslan Mavlyutov, Inigo Goiri, Subru Krishnan, Janardhan Kulkarni, and Sriram Rao. 2016. Morpheus: Towards Automated SLOs for Enterprise Clusters. In Proceedings of the 12th Symposium on Operating Systems Design and Implementation (OSDI). [41] Kapil Karkra. [n.d.]. Using Software to Reduce High Tail Latencies on SSDs. https://www.flashmemorysummit.com/English/Collaterals/ Proceedings/2018/20180808. FOFT- 201- 1_Karkar.pdf. [42] Kate Keahey, Jason Anderson, Zhuo Zhen, Pierre Riteau, Paul Ruth, Dan Stanzione, Mert Cevik, Jacob Colleran, Harvadi S. Gunawi, Cody Hammock, Joe Mambretti, Alexander Barnes, Francis Halbach, Alex Rocha, and Joe Stubbs. 2020. Lessons Learned from the Chameleon Testbed. In Proceedings of the 2020 USENIX Annual Technical Conference (ATC). [43] Redwan Ibne Seraj Khan, Ahmad Hossein Yazdani, Yuqi Fu, Arnab K. Paul, Bo Ji, Xun Jian, Yue Cheng, and Ali Raza Butt. 2023. SHADE: Enable Fundamental Cachecability for Distributed Deep Learning Training. In Proceedings of the 21st USENIX Symposium on File and Storage Technologies (FAST). [44] Mehrdad Khani, Ganesh Ananthanarayanan, Kevin Hsieh, Junchen Jiang, Ravi Netravali, Yuanshao Shu, Mohammad Alizadeh, and Victor Bahl. 2023. RECL: Responsive Resource- Efficient Continuous Learning for Video Analytics. In Proceedings of the 20th Symposium on Networked Systems Design and Implementation (NSDI). [45] Sangwook Kim, Hwanju Kim, Joonwon Lee, and Jinkyu Jeong. 2017. Enlightening the I/O Path: A Holistic Approach for Application Performance. In Proceedings of the 15th USENIX Symposium on File and Storage Technologies (FAST). [46] Abhishek Vijaya Kumar and Muthian Sivathanu. 2020. Quiver: An Informed Storage Cache for Deep Learning. In Proceedings of the 18th USENIX Symposium on File and Storage Technologies (FAST). [47] Daniar H. Kurniawan, Levent Toksoz, Mingzhe Hao, Anirudh Badam, Tim Emami, Sandeep Madireddy, Robert B. Ross, Henry Hoffmann, and Haryadi S. Gunawi. 2021. IONET: Towards an Open Machine Learning Training Ground for I/O Performance Prediction. In Technical Report University of Chicago TR- 2021- 03. [48] Matthias De Lange, Rahaf Aljudi, Marc Masana, Sarah Parisot, Xu Jia, Ales Leonardis, Gregory Slabaugh, and Tinne Tuytelaars. 2022. A continual learning survey: Defying forgetting in classification tasks. In IEEE Transactions on Pattern Analysis and Machine Intelligence. [49] Timothée Lesort, Massimo Caccia, and Irina Rish. 2022. Understanding Continual Learning Settings with Data Distribution Drift Analysis. In Computing Research Repository.

[50] Huaicheng Li, Mingzhe Hao, Michael Hao Tong, Swaminathan Sundararaman, Matias Bjorling, and Haryadi S. Gunawi. 2018. The CASE of FEMU: Cheap, Accurate, Scalable and Extensible Flash Emulator. In Proceedings of the 16th USENIX Symposium on File and Storage Technologies (FAST). [51] Huaicheng Li, Martin L. Putra, Ronald Shi, Xing Lin, Gregory R. Ganger, and Haryadi S. Gunawi. 2021. IODA: A Host/Device Co- Design for Strong Predictability Contract on Modern Flash Storage. In Proceedings of the 29th ACM Symposium on Operating Systems Principles (SOSP). [52] Nanqingin Li, Mingzhe Hao, Xing Lin, Huaicheng Li, Levent Toksoz, Tim Emami, and Haryadi S. Gunawi. 2022. Fantastic SSD Internals and How to Learn and Use Them. In Proceedings of the 15th ACM International Systems and Storage Conference (SYSTOR). [53] Ning Li, Hong Jiang, Dan Feng, and Zhan Shi. 2016. PSLO: Enforcing the Xth Percentile Latency and Throughput SLOs for Consolidated VM Storage. In Proceedings of the 2016 EuroSys Conference (EuroSys). [54] Pengfei Li, Yu Hua, Pengfei Zuo, Zhangyu Chen, and Jiajie Sheng. 2023. ROLEX: A Scalable RDMA- oriented Learned Key- Value Store for Disaggregated Memory Systems. In Proceedings of the 21st USENIX Symposium on File and Storage Technologies (FAST). [55] Lei Liu, Xinglei Dou, and Yuetao Chen. 2023. Intelligent Resource Scheduling for Co- located Latency- critical Services: A Multi- Model Collaborative Learning Approach. In Proceedings of the 21st USENIX Symposium on File and Storage Technologies (FAST). [56] Sidi Lu, Bing Luo, Tirthak Patel, Yongtao Yao, Devesh Tiwari, and Weisong Shi. 2020. Making Disk Failure Predictions SMARTer!. In Proceedings of the 18th USENIX Symposium on File and Storage Technologies (FAST). [57] Martin Maas, David G. Andersen, Michael Isard, Mohammad Mahdi Javanmard, Kathryn S. McKinley, and Colin Raffel. 2020. Learning- based Memory Allocation for  $\mathrm{C + + }$  Server Workloads. In Proceedings of the 25th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS). [58] Zheda Mai, Ruiwen Li, Jihwan Jeong, David Quispe, Hyunwoo Kim, and Scott Sanner. 2022. Online continual learning in image classification: An empirical survey. In Neurocomputing. [59] Andrew McCallum. 2009. Joint inference for natural language processing.. In Proceedings of the 13th Conference on Computational Natural Language Learning (CoNLL). [60] Rino Micheloni. 2017. Solid- State Drive (SSD): A Nonvolatile Storage System. In 2017 Proceedings of the IEEE (Proc. IEEE). [61] Dushyanth Narayanan, Austin Donnelly, and Antony Rowstron. 2008. MSR Cambridge traces (SNIA IOTTA trace set 388). In Proceedings of the 6th USENIX Symposium on File and Storage Technologies (FAST). [62] Iratxe Nino- Adan, Eva Portillo, Itziar Landa- Torres, and Diana Manjarres. 2021. Normalization influence on ANN- based models performance: A new proposal for Feature- contribution analysis. In IEEE Access. [63] Satadru Pan, Theano Stavrinos, Yunqiao Zhang, Atul Sikaria, Pavel Zakharov, Abhinav Sharma, Shiva Shankar P., Mike Shuey, Richard Wareing, Monika Gangapuram, Guanglei Cao, Christian Preseau, Pratap Singh, Kestutis Patiejunas, J. Ripton, Ethan Katz- Bassett, and Wyatt Lloyd. 2021. Facebook's Tectonic Filesystem: Efficiency from Exascale. In Proceedings of the 18th USENIX Symposium on File and Storage Technologies (FAST). [64] Jisung Park, Jeonggyun Kim, Yeseong Kim, Sungjin Lee, and Onur Mutlu. 2022. DeepSketch: A New Machine Learning- Based Reference Search Technique for Post- Deduplication Delta Compression. In Proceedings of the 20th USENIX Symposium on File and Storage Technologies (FAST). [65] Mia Primorac, Katerina J. Argyraki, and Edouard Bugnion. 2021. When to Hedge in Interactive Services. In Proceedings of the 18th Symposium on Networked Systems Design and Implementation (NSDI).

[66] K. V. Rashmi, Mosharaf Chowdhury, Jack Kosaian, Ion Stoica, and Kannan Ramchandran. 2016. EC- Cache: Load- Balanced, Low- Latency Cluster Caching with Online Erasure Coding. In Proceedings of the 12th Symposium on Operating Systems Design and Implementation (OSDI). [67] Carlos Riquelme, Joan Puigcerver, Basil Mustafa, Maxim Neumann, Rodolphe Jenatton, Andre Susano Pinto, Daniel Keysers, and Neil Houlsby. 2021. Scaling Vision with Sparse Mixture of Experts. In Proceedings of the 35th Conference on Neural Information Processing Systems (NIPS). [68] Liana V. Rodriguez, Farzana Beente Yusuf, Steven Lyons, Eysler Paz, Raju Rangaswami, Jason Liu, Ming Zhao, and Giri Narasimhan. 2021. Learning Cache Replacement with CACHEUS. In Proceedings of the 19th USENIX Symposium on File and Storage Technologies (FAST). [69] Sultan Mahmud Sajal, Luke Marshall, Beibin Li, Shandan Zhou, Abhisek Pan, Konstantina Mellou, Deepak Narayanan, Timothy Zhu, David Dion, Thomas Mowcibroda, and Ishai Menache. 2023. Kerveros: Efficient and Scalable Cloud Admission Control. In Proceedings of the 17th Symposium on Operating Systems Design and Implementation (OSDI). [70] Siddharth Sharma, Simone Sharma, and Anidhya Athaiya. 2020. Activation Functions in Neural Networks. In International Journal of Engineering Applied Sciences and Technology (IJEAST). [71] Noam Shazeer, Azalia Mirhoseini, Krzysztof Maziarz, Andy Davis, Quoc Le, Geoffrey Hinton, and Jeff Dean. 2017. Outrageously Large Neural Networks: The Sparsely- Gated Mixture- of- Experts Layer. In Computing Research Repository. [72] Zhan Shi, Akanksha Jairi, Kevin Swersky, Milad Hashemi, Parthasarathy Ranganathan, and Calvin Lin. 2021. A hierarchical neural model of data prefetching. In Proceedings of the 26th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS). [73] Jinghan Sun, Shaobo Li, Yunxin Sun, Chao Sun, Dejan Vucinic, and Jian Huang. 2023. LeaFTL: A Learning- Based Flash Translation Layer for Solid- State Drives. In Proceedings of the 28th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS). [74] Lalith Suresh, Marco Camini, Stefan Schmid, and Anja Feldmann. 2015. C3: Cutting Tail Latency in Cloud Data Stores via Adaptive Replica Selection. In Proceedings of the 12th Symposium on Networked Systems Design and Implementation (NSDI). [75] Ruben van den Goorbergh, Maarten van Smeden, Dirk Timmerman, and Ben Van Calster. 2022. The harm of class imbalance corrections for risk prediction models: illustration and simulation using logistic regression.. In Journal of the American Medical Informatics Association.

[76] Shiva Verma. 2020. How to Create a Custom Loss Function | Keras.

[77] Xingda Wei, Rong Chen, and Haibo Chen. 2020. Fast RDMA- based Ordered Key- Value Store using Remote Learned Cache. In Proceedings of the 14th Symposium on Operating Systems Design and Implementation (OSDI). [78] Sage A. Weil, Scott A. Brandt, Ethan L. Miller, Darrell D. E. Long, and Carlos Maltzahn. 2006. Ceph: A Scalable and High- Performance Distributed File System. In Proceedings of the 7th Symposium on Operating Systems Design and Implementation (OSDI). [79] Gerhard Widmer and Miroslav Kubat. 1996. Learning in the presence of concept drift and hidden contexts. In Machine Learning (ML). [80] Daniel Lin- Kit Wong, Hao Wu, Carson Molder, Sathya Gunasekar, Jimmy Lu, Snehal Khandkar, Abhinav Sharma, Daniel S. Berger, Nathan Beckmann, Gregory R. Gangger for ML admission, and cache prefetching. 2024. Baleen: ML Admission & Prefetching for Flash Caches. In Proceedings of the 22nd USENIX Symposium on File and Storage Technologies (FAST). [81] Shiqin Yan, Huaicheng Li, Mingzhe Hao, Michael Hao Tong, Swaminathan Sundararaman, Andrew A. Chien, and Haryadi S. Gunawi. 2017. Tiny- Tail Flash: Near- Perfect Elimination of Garbage Collection Tail Latencies in NAND SSDs. In Proceedings of the 15th USENIX Symposium on File and Storage Technologies (FAST). [82] Juncheng Yang, Ziming Mao, Yao Yue, and K. V. Rashmi. 2023. GL- Cache: Group- level learning for efficient and high- performance caching. In Proceedings of the 21st USENIX Symposium on File and Storage Technologies (FAST). [83] Yu Zhang, Ping Huang, Ke Zhou, Hua Wang, Jianying Hu, Yongguang Ji, and Bin Cheng. 2020. Tencent block storage traces (SNIA IOTTA trace set 27917). In Proceedings of the 2020 USENIX Annual Technical Conference (ATC). [84] Yiwen Zhang, Gautam Kumar, Nandita Dukkipati, Xian Wu, Priyaranjan Jha, Mosharaf Chowdhury, and Amin Vahdat. 2022. Aequitas: admission control for performance- critical RPCs in datacenters. In Proceedings of the ACM Special Interest Group on Data Communication (SIGCOMM). [85] Alice Zheng and Amanda Casari. 2018. Feature engineering for machine learning: principles and techniques for data scientists. [86] Xianqian Zhou, Liyuan Fang, HaiMing Xie, and Wanchun Jiang. 2019. TAP: Timeliness- aware predication- based replica selection algorithm for key- value stores. In Concurrency and Computation: Practice and Experience (CCPE'19), Volume 31. [87] Timothy Zhu, Alexey Tumanov, Michael A. Kozuch, More Harchol- Balter, and Gregory R. Gangger. 2014. PriorityMeister: Tail Latency QoS for Shared Networked Storage. In Proceedings of the 5th ACM Symposium on Cloud Computing (SoCC). [88] Lorenzo Zuolo, Cristian Zambelli, Rino Micheloni, Davide Bertozzi, and P. Olivo. 2014. Analysis of reliability/performance trade- off in Solid State Drives. In IEEE International Symposium on Reliability Physics (IRPS).
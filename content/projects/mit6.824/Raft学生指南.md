---
created: 2025-03-08T09:43:58 (UTC +08:00)
tags: 
source: https://thesquareplanet.com/blog/students-guide-to-raft/
author: Jon Gjengset <jon@thesquareplanet.com>
translator: hypervoid
---
# 学生版Raft指南（翻译）

> 过去几个月，我一直担任麻省理工学院6.824分布式系统课程的助教。这门课程传统上包含多个基于Paxos共识算法的实验，但今年我们决定转向Raft。Raft被"设计为易于理解"，我们希望这个改变能让学生的生活更轻松。

过去几个月，我一直担任麻省理工学院[6.824分布式系统](https://pdos.csail.mit.edu/6.824/)课程的助教。这门课程传统上包含多个基于Paxos共识算法的实验，但今年我们决定转向[Raft](https://raft.github.io/)。Raft被"设计为易于理解"，我们希望这个改变能让学生的生活更轻松。

本文及配套的[教师版Raft指南](https://thesquareplanet.com/blog/instructors-guide-to-raft/)记录了我们在Raft上的探索历程，希望能对Raft协议的实现者和希望更好理解Raft内部机制的学生有所帮助。如果你正在寻找Paxos与Raft的比较，或需要更教学化的Raft分析，请阅读教师版指南。本文末尾列出了6.824学生常问的问题及其答案。如果遇到本文主体内容未涵盖的问题，请查看[问答部分](https://thesquareplanet.com/blog/raft-qa/)。文章较长，但提出的所有问题都是许多6.824学生（和助教）实际遇到的真实问题，值得一读。

## [背景](https://thesquareplanet.com/blog/students-guide-to-raft/#background)

在深入探讨Raft之前，了解背景可能有所帮助。6.824过去有一组基于[Go语言](https://golang.org/)的[Paxos实验](http://nil.csail.mit.edu/6.824/2015/labs/lab-3.html)；选择Go语言既因为它对学生容易学习，也因为它非常适合编写并发分布式应用（goroutine特别方便）。通过四个实验，学生需要构建一个容错的分布式键值存储。第一个实验构建基于共识的日志库，第二个在其上添加键值存储，第三个将键空间分片到多个容错集群，由容错的分片主节点处理配置变更。我们还有第四个实验，学生需要处理机器故障和恢复（包括磁盘完好和损坏的情况）。这个实验作为学生的默认期末项目。

今年我们决定用Raft重写所有实验。前三个实验保持不变，但第四个实验被取消，因为持久化和故障恢复已内置在Raft中。本文将主要讨论第一个实验的经验，因为它与Raft直接相关，同时也会涉及在Raft之上构建应用（如第二个实验）。

对于刚接触Raft的读者，该协议的最佳描述来自其[官网](https://raft.github.io/)：

> Raft是一种旨在易于理解的共识算法。它在容错性和性能上与Paxos等效。不同之处在于它将问题分解为相对独立的子问题，并清晰解决了实际系统所需的所有主要部分。我们希望Raft能让更广泛的受众理解共识算法，并基于此开发出比现有系统更高质量的各种共识系统。

像[这个可视化演示](http://thesecretlivesofdata.com/raft/)很好地概述了协议的核心组件，论文也很好地解释了为什么需要各个部分。如果你还没读过[Raft扩展论文](https://raft.github.io/raft.pdf)，请先阅读再继续本文，因为我会假设读者对Raft有一定了解。

与所有分布式共识协议一样，关键在于细节。在没有故障的稳定状态下，Raft的行为容易理解，可以直观解释。例如，从可视化中很容易看出，假设没有故障，最终会选出领导者，最终所有发送给领导者的操作都会被追随者按正确顺序应用。但当引入延迟消息、网络分区和故障服务器时，每个条件、但是、和都变得至关重要。特别是，由于误解或疏忽，我们会反复看到一些重复出现的错误。这个问题并非Raft独有，所有提供正确性的复杂分布式系统都会遇到。

## [实现Raft](https://thesquareplanet.com/blog/students-guide-to-raft/#implementing-raft)

Raft的终极指南是其论文中的图2。这个图规定了Raft服务器之间交换的所有RPC行为，给出了服务器必须维护的各种不变量，并规定了某些动作何时发生。在本文剩余部分我们将大量讨论图2。它需要被_严格遵循_。

图2定义了每个服务器在每个状态下对每个传入RPC的处理方式，以及某些其他事件（如何时可以安全应用日志条目）的触发条件。起初你可能想把图2当作非正式指南：读一遍就开始编码，大致遵循其描述。这样做可以快速得到一个基本可用的Raft实现。但问题随之而来。

实际上，图2非常精确，每个陈述都应被视为规范中的**必须**，而非**应该**。例如，你可能合理地在收到`AppendEntries`或`RequestVote` RPC时重置选举定时器，因为两者都表明其他节点认为自己是领导者或试图成为领导者。直觉上这意味着我们不应干扰。但如果仔细阅读图2：

> 如果在选举超时期间未收到_当前领导者_的`AppendEntries` RPC或未_授予_候选人投票：转换为候选人

这个区别在实际中非常重要，因为前者在某些情况下可能导致活性显著降低。

## [细节的重要性](https://thesquareplanet.com/blog/students-guide-to-raft/#the-importance-of-details)

举个让许多6.824学生困惑的例子。Raft论文多次提到_心跳RPC_。具体来说，领导者会定期（至少每个心跳间隔）向所有节点发送`AppendEntries` RPC以防止新选举。如果领导者没有新条目要发送，`AppendEntries` RPC不包含条目，视为心跳。

许多学生认为心跳是"特殊"的：节点收到心跳时应以不同方式处理。特别是许多人在收到心跳时只是重置选举定时器然后返回成功，不执行图2规定的检查。这_非常危险_。通过接受RPC，追随者隐式告诉领导者其日志在`prevLogIndex`之前与领导者匹配。收到回复后，领导者可能错误认为某个条目已被复制到多数节点并开始提交。

另一个常见问题（通常在修复上述问题后出现）是，收到心跳时，追随者会截断`prevLogIndex`之后的日志，然后追加`AppendEntries`中的条目。这_也不正确_。再次参考图2：

> _如果_现有条目与新条目冲突（相同索引但不同任期），删除现有条目及其后续条目

这里的_如果_至关重要。如果追随者已有领导者发送的所有条目，则_不得_截断日志。领导者发送条目_之后_的内容必须保留。因为我们可能收到领导者的过时`AppendEntries` RPC，截断日志意味着"收回"可能已告知领导者的条目。

## [调试Raft](https://thesquareplanet.com/blog/students-guide-to-raft/#debugging-raft)

不可避免地，你的Raft实现初版会有缺陷。第二、第三、第四版也是如此。一般来说，每版缺陷会减少，根据经验，大多数错误源于未严格遵循图2。

调试Raft时，主要错误来源有四种：活锁、错误或不完整的RPC处理程序、未遵守规则和任期混淆。死锁也常见，但通常可通过记录所有锁操作来调试。我们逐一分析：

## [活锁](https://thesquareplanet.com/blog/students-guide-to-raft/#livelocks)

当系统活锁时，每个节点都在工作，但整体无法进展。这在Raft中很常见，尤其当未严格遵循图2时。一个常见活锁场景是：没有领导者当选，或领导者当选后其他节点立即发起选举迫使新领导者退位。

可能原因有很多，但我们看到学生常犯以下错误：

- 确保_严格_按图2重置选举定时器。具体来说，仅在以下情况重置：a) 收到_当前领导者_的`AppendEntries` RPC（若参数中的任期过时则不应重置）；b) 自己开始选举；c) 给其他候选人_授予_投票。

  最后一种情况在不稳定网络中尤为重要。如果每次收到投票请求都重置定时器，会导致过时日志的节点同样可能当选。

- 按图2规定发起选举。特别注意，如果已是候选人（正在选举中）但选举超时，应开始_新_选举。这对避免因延迟或丢失RPC导致系统停滞至关重要。
  
- 处理传入RPC前先遵循"服务器规则"第二条：
  
  > 如果RPC请求或响应包含任期`T > currentTerm`：设置`currentTerm = T`，转换为追随者（§5.1）

  例如，如果你在当前任期已投票，而传入的`RequestVote` RPC有更高任期，应_先_退位并采用新任期（重置`votedFor`），_然后_处理RPC，这将导致你授予投票！

## [错误的RPC处理程序](https://thesquareplanet.com/blog/students-guide-to-raft/#incorrect-rpc-handlers)

尽管图2明确规定了每个RPC处理程序的行为，仍有一些细节容易被忽视。以下是我们反复看到的常见错误：

- 如果某步骤要求"回复false"，意味着应_立即回复_，不执行后续步骤。
- 如果收到`prevLogIndex`超过日志长度的`AppendEntries` RPC，应视为条目存在但任期不匹配（即回复false）。
- `AppendEntries` RPC处理程序的检查2（即使领导者未发送条目）必须执行。
- `AppendEntries`最后步骤5中的`min`是_必要_的，需用最后_新_条目索引计算。不能简单地在`lastApplied`和`commitIndex`之间应用日志时停在日志末尾。因为领导者发送条目后可能有与你日志冲突的条目。由于步骤3规定仅在有冲突时截断日志，这些条目不会被删除。如果`leaderCommit`超过领导者发送的条目，可能应用错误条目。
- 必须严格按照5.4节实现"最新日志"检查，不能仅检查长度。

## [未遵守规则](https://thesquareplanet.com/blog/students-guide-to-raft/#failure-to-follow-the-rules)

虽然Raft论文明确了每个RPC处理程序的实现，但许多规则和不变量的实现细节未明确。这些列在图2右侧的"服务器规则"中。虽然有些不言自明，但有些需要精心设计以避免违规：

- 任何时候若`commitIndex > lastApplied`，应应用相应日志条目。不需立即执行（如在`AppendEntries`处理程序中），但必须确保只有一个实体执行应用。需专用"应用器"或在应用时加锁。
- 定期或在`commitIndex`更新后检查`commitIndex > lastApplied`。例如，如果在发送`AppendEntries`时检查`commitIndex`，可能要等到_下个_条目追加才能应用刚发送的条目。
- 如果领导者发送`AppendEntries`被拒绝（非日志不一致原因），应立即退位且_不_更新`nextIndex`。否则可能在立即重选时与`nextIndex`重置竞争。
- 领导者不能将`commitIndex`更新到_先前_任期（或未来任期）。如规则所述，必须检查`log[N].term == currentTerm`。因为Raft领导者无法确认非当前任期的条目已提交（未来不会改变）。论文图8说明了这点。

常见困惑点是`nextIndex`与`matchIndex`的区别。注意`matchIndex = nextIndex - 1`但不安全。`nextIndex`是领导者对与追随者共享日志前缀的_乐观猜测_，通常设置为日志末尾，仅在被拒时回退。`matchIndex`是_保守测量_，初始化为-1，仅在收到肯定回复时更新，用于安全推进`commitIndex`。

## [任期混淆](https://thesquareplanet.com/blog/students-guide-to-raft/#term-confusion)

任期混淆指服务器被旧任期RPC混淆。处理传入RPC时通常没问题，但旧RPC回复可能导致问题。最简单做法是：先记录回复中的任期（可能高于当前任期），比较当前任期与原始RPC的任期。若不同则丢弃回复。仅任期相同时处理回复。不做此处理将导致严重问题。

相关问题是在处理RPC回复时假设状态未变。例如收到RPC回复时设置`matchIndex = nextIndex - 1`或`matchIndex = len(log)`。这不安全，因为这些值可能在发送RPC后已改变。正确做法是使用原始RPC参数中的`prevLogIndex + len(entries[])`更新`matchIndex`。

## [优化说明](https://thesquareplanet.com/blog/students-guide-to-raft/#an-aside-on-optimizations)

Raft论文包含一些可选功能。在6.824中，学生需要实现其中两个：日志压缩（第7节）和加速日志回溯（第8页左上）。前者防止日志无限增长，后者帮助过时追随者快速同步。

这些不是"核心Raft"部分，论文未重点讨论。日志压缩在论文图13有说明，但需注意：

- 快照应用状态必须对应某个已知日志索引。应用层需告知Raft快照对应的索引，或Raft需延迟应用新条目直到快照完成。
- 论文未讨论服务器崩溃恢复协议。若Raft状态和快照分开提交，服务器可能在保存快照后崩溃，未保存更新后的Raft状态。这会导致读取旧日志，应用已包含在快照中的条目。需在Raft持久状态中记录日志起始索引，与快照的`lastIncludedIndex`比较以丢弃重复条目。

加速日志回溯优化未详细说明，可能方案：

- 若追随者没有`prevLogIndex`，返回`conflictIndex = len(log)`，`conflictTerm = None`
- 若有`prevLogIndex`但任期不匹配，返回`conflictTerm = log[prevLogIndex].Term`，并搜索该任期的第一个索引
- 领导者收到冲突响应后，先搜索自身日志的`conflictTerm`。若找到，设置`nextIndex`为该任期最后条目索引+1
- 若未找到，设置`nextIndex = conflictIndex`

折中方案是仅使用`conflictIndex`（忽略`conflictTerm`），简化实现但可能导致发送更多日志条目。

## [Raft之上的应用](https://thesquareplanet.com/blog/students-guide-to-raft/#applications-on-top-of-raft)

在Raft之上构建服务（如第二个实验的键值存储）时，服务与Raft日志的交互可能很棘手。本节详述开发过程中需要注意的方面。

## [应用客户端操作](https://thesquareplanet.com/blog/students-guide-to-raft/#applying-client-operations)

你可能会困惑如何基于复制日志实现应用。初始想法可能是：服务收到客户端请求后，转发给领导者，等待Raft应用，执行操作后返回客户端。这在单客户端系统中可行，但不支持并发客户端。

正确做法是将服务构建为_状态机_，客户端操作使状态机转换。需要有一个循环逐个按顺序应用客户端操作（所有服务器顺序相同），这是Raft的作用。这个循环应是唯一接触应用状态（如键值映射）的代码。客户端RPC方法应将操作提交给Raft，然后_等待_该操作被"应用循环"处理。只有当客户端命令出现时才执行并读取返回值。注意_读请求也需要这样处理_！

如何知道客户端操作何时完成？无故障时简单：等待提交的日志条目被应用。但发生故障时（如领导者变更导致日志条目被丢弃），需让客户端重试。解决方案是记录客户端操作在日志中的位置，当该索引的条目被应用时，检查是否是原操作，否则返回错误。

## [重复检测](https://thesquareplanet.com/blog/students-guide-to-raft/#duplicate-detection)

客户端重试时需检测重复。需要为每个客户端请求提供唯一标识符，确保相同操作只应用一次。简单有效的方法是为每个客户端分配唯一ID，并为每个请求标记单调递增序列号。服务器跟踪每个客户端的最新序列号，忽略已处理的请求。

## [棘手边界情况](https://thesquareplanet.com/blog/students-guide-to-raft/#hairy-corner-cases)

遵循上述方案时，可能遇到两个难以调试的边界情况：

**索引重复出现**：假设Raft库有`Start()`方法，返回命令在日志中的索引。你假设不会看到相同索引两次，或相同索引意味着前次操作失败。但即使无服务器崩溃，这两种情况都可能发生。

考虑五服务器场景：
1. S1为领导者，日志为空
2. 客户端操作C1、C2到达S1
3. `Start()`返回1和2
4. S1向S2发送含C1、C2的`AppendEntries`，其他消息丢失
5. S3成为候选人
6. S3当选领导者，接收C3
7. S3发送`AppendEntries`给S1，S1丢弃C1、C2，添加C3
8. S3故障，S1当选领导者
9. 客户端C4到达S1，`Start()`返回2（与C2相同）
10. S1消息丢失，S2当选领导者
11. C5到达S2，`Start()`返回3
12. S2提交日志`[C1 C2 C5]`，索引2的条目为C2，尽管S1的索引2对应C4

**四路死锁**：由助教Steven Allen发现，构建Raft应用时可能陷入的死锁：

Raft代码可能有获取锁`a`的`Start()`方法和应用循环。应用层可能在RPC处理中调用`Start()`（获取锁`b`），在应用通知中也需要锁`b`。当以下情况同时发生：
- `App.RPC`持有`a.mutex`并调用`Raft.Start`（等待`r.mutex`）
- `Raft.AppendEntries`持有`r.mutex`并调用`App.apply`（等待`a.mutex`）

解决方案：
1. 在`App.RPC`中先调用`a.raft.Start`再获取`a.mutex`
2. 使用专用线程调用`r.app.apply`，避免在应用时持有锁

通过仔细处理锁顺序和专用应用线程，可以避免这种死锁。
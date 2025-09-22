Alpha-Beta剪枝是一种在[游戏树](Game_tree.md)搜索中常用的优化策略，它可以大大减少需要搜索的节点数，从而提高搜索效率。

在一个完全信息的二人对抗游戏中，我们通常使用Minimax算法来决定最优的行动策略。Minimax算法的基本思想是：假设所有玩家都会采取最优的行动策略，那么在当前状态下，我应该采取什么行动才能使我在最坏情况下的收益最大化。

然而，Minimax算法需要遍历整个游戏树，当游戏树的大小非常大时，这将需要消耗大量的计算资源。这时，我们就可以使用Alpha-Beta剪枝来优化搜索过程。

Alpha-Beta剪枝的基本思想是：在搜索过程中，一旦发现某个节点的**最终得分肯定不如已知的其他节点**，那么就没有必要再搜索这个节点下面的子节点，可以**直接剪掉**这个节点。

具体来说，Alpha-Beta剪枝维护了两个值：Alpha和Beta。Alpha表示的是我们已经找到的**最好的Maximizer**节点的得分，Beta表示的是我们已经找到的**最好的Minimizer**节点的得分。在搜索过程中，如果某个Maximizer节点的得分小于Alpha，或者某个Minimizer节点的得分大于Beta，那么就可以剪掉这个节点。

通过Alpha-Beta剪枝，我们可以大大减少需要搜索的节点数，从而提高搜索效率。但是，Alpha-Beta剪枝并不会改变Minimax算法的结果，也就是说，它只是优化了搜索过程，但并不会影响搜索结果。


推导例子：[CSCI 6350 Artificial Intelligence: Minimax and Alpha-Beta Pruning Algorithms and Psuedocodes - YouTube](https://youtu.be/J1GoI5WHBto?t=1818)
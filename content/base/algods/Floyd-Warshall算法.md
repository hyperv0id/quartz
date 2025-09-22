判断有向图是不是有环
#todo 

假设有向图中节点编号为0到n-1

问题：所有节点之间的最短路径
子问题$R_{ij}^k$：从节点 $i$ 到节点 $j$ **中间节点**编号 $\leq k$ 的所有路径。

递推公式：
$$
R^k_{ij} = R^{k-1}_{ik}(R^{k-1}_{kk} )^*R^{k-1}_{kj} | R^{k-1}_{ij}
$$

也就是
1. i到j所有节点都$\leq k-1$
2. i到k所有节点都$\leq k-1$ 再加上 k到j 所有节点都$\leq k-1$
	1. 在[DFA](../compiler/nju编译原理/概念/DFA.md)转换成[正则表达式](../compiler/nju编译原理/概念/正则表达式.md)中，还要考虑一下可能有多个k节点的情况
	2. 这里使用[Kleene闭包](../compiler/nju编译原理/algos/Kleene闭包.md)
























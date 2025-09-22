---
aliases:
  - UCS
  - 统一代价搜索
tags:
  - algorithm/search
---

## 原理

Uniform-cost search算法是一种广泛使用的最优路径搜索算法,它能找到从起点到目标点的最小代价路径。这种算法适用于状态空间是加权图(weighted graph)的情况,每条边都有一个代价值。

作为对比，广度优先搜索适用于代价都是1。因此UCS更泛用。



[ ![Wayback Machine ](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/04/25/20240425140044-f3f689.png)](https://web.archive.org/web/20200218150951/https://www.aaai.org/ocs/index.php/SOCS/SOCS11/paper/viewFile/4017/4357)

### 数据结构

1. 创建一个**优先级队列**，用于存储待扩展的节点。
2. 将起点节点放入队列，代价为0。
3. 从队列中取出代价最小的节点n。
4. 如果n是目标节点，搜索结束，返回n所对应的最小代价路径。
5. 否则,将n的所有后继节点加入优先级队列。`后继节点代价 = n的代价 + 从n到该节点的边代价`。
6. 回到步骤**3**，直到找到目标节点或队列为空(无解)。

## 算法实现

> 来自：[Uniform-Cost Search (Dijkstra for large Graphs) - GeeksforGeeks](https://www.geeksforgeeks.org/uniform-cost-search-dijkstra-for-large-graphs/)

```c++
#include <bits/stdc++.h>

using namespace std;

// graph
vector<vector<int> > graph;

// map to store cost of edges
map<pair<int, int>, int> cost;

// returns the minimum cost in a vector( if 
// there are multiple goal states)
vector<int> uniform_cost_search(vector<int> goal, int start)
{
	// minimum cost upto
	// goal state from starting
	// state
	vector<int> answer;

	// create a priority queue
	priority_queue<pair<int, int> > queue;

	// set the answer vector to max value
	for (int i = 0; i < goal.size(); i++)
		answer.push_back(INT_MAX);

	// insert the starting index
	queue.push(make_pair(0, start));

	// map to store visited node
	map<int, int> visited;

	// count
	int count = 0;

	// while the queue is not empty
	while (queue.size() > 0) {

		// get the top element of the 
		// priority queue
		pair<int, int> p = queue.top();

		// pop the element
		queue.pop();

		// get the original value
		p.first *= -1;

		// check if the element is part of
		// the goal list
		if (find(goal.begin(), goal.end(), p.second) != goal.end()) {

			// get the position
			int index = find(goal.begin(), goal.end(), 
							p.second) - goal.begin();

			// if a new goal is reached
			if (answer[index] == INT_MAX)
				count++;

			// if the cost is less
			if (answer[index] > p.first)
				answer[index] = p.first;

			// pop the element
			queue.pop();

			// if all goals are reached
			if (count == goal.size())
				return answer;
		}

		// check for the non visited nodes
		// which are adjacent to present node
		if (visited[p.second] == 0)
			for (int i = 0; i < graph[p.second].size(); i++) {

				// value is multiplied by -1 so that 
				// least priority is at the top
				queue.push(make_pair((p.first + 
				cost[make_pair(p.second, graph[p.second][i])]) * -1, 
				graph[p.second][i]));
			}

		// mark as visited
		visited[p.second] = 1;
	}

	return answer;
}

// main function
int main()
{
	// create the graph
	graph.resize(7);

	// add edge
	graph[0].push_back(1);
	graph[0].push_back(3);
	graph[3].push_back(1);
	graph[3].push_back(6);
	graph[3].push_back(4);
	graph[1].push_back(6);
	graph[4].push_back(2);
	graph[4].push_back(5);
	graph[2].push_back(1);
	graph[5].push_back(2);
	graph[5].push_back(6);
	graph[6].push_back(4);

	// add the cost
	cost[make_pair(0, 1)] = 2;
	cost[make_pair(0, 3)] = 5;
	cost[make_pair(1, 6)] = 1;
	cost[make_pair(3, 1)] = 5;
	cost[make_pair(3, 6)] = 6;
	cost[make_pair(3, 4)] = 2;
	cost[make_pair(2, 1)] = 4;
	cost[make_pair(4, 2)] = 4;
	cost[make_pair(4, 5)] = 3;
	cost[make_pair(5, 2)] = 6;
	cost[make_pair(5, 6)] = 3;
	cost[make_pair(6, 4)] = 7;

	// goal state
	vector<int> goal;

	// set the goal
	// there can be multiple goal states
	goal.push_back(6);

	// get the answer
	vector<int> answer = uniform_cost_search(goal, 0);

	// print the answer
	cout << "Minimum cost from 0 to 6 is = "
		<< answer[0] << endl;

	return 0;
}
```
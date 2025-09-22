---
aliases: [康威生命游戏]
---

## 游戏规则
生命游戏中，对于任意细胞，规则如下：

- 每个细胞有两种状态 - 存活或死亡，每个细胞与以自身为中心的周围**八格**细胞产生互动（如图，黑色为存活，白色为死亡）
- 当前细胞为存活状态时，当周围的存活细胞低于2个时（不包含2个），该细胞变成死亡状态。（模拟生命数量稀少）
- 当前细胞为存活状态时，当周围有2个或3个存活细胞时，该细胞保持原样。
- 当前细胞为存活状态时，当周围有超过3个存活细胞时，该细胞变成死亡状态。（模拟生命数量过多）
- 当前细胞为死亡状态时，当周围有3个存活细胞时，该细胞变成存活状态。（模拟繁殖）

可以把最初的细胞结构定义为种子，当所有在种子中的细胞**同时**被以上规则处理后，可以得到第一代细胞图。按规则继续处理当前的细胞图，可以得到下一代的细胞图，周而复始。

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/12/03/Gospers_glider_gun-24adef.gif)![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/12/03/Conways_game_of_life_breeder_animation-f389b5.gif)

而且它是图灵完备的，因此可以模拟任何现实的问题。

## References

- [维基百科](https://zh.wikipedia.org/wiki/%E5%BA%B7%E5%A8%81%E7%94%9F%E5%91%BD%E6%B8%B8%E6%88%8F)
- 在线演示
	- [生命游戏](http://home.ustc.edu.cn/~zzzz/lifegame/lifegame.html)
	- [康威的生命游戏](https://playgameoflife.com/)
- 实现
	- 元胞自动机模拟软件：[Golly Game of Life Home Page](https://golly.sourceforge.net/)
	- go语言：go语言实现：[https://go.dev/doc/play/life.go](https://go.dev/doc/play/life.go)

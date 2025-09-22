## 二者等价吗

1. 所有[DFSM](DFA.md)都是一个[NFSM](NFA.md)
2. 所有[NFSM](NFA.md)都有一个等价的[DFSM](DFA.md)
3. 因此二者等价
## 谁更Powerful

1. 因为二者等价，你不能找出一个[DFSM](DFA.md)能辨认的语言，但是[NFSM](NFA.md)不能辨认，反过来同理。
2. 因此二者一样强大

因为二者等价，那么可以将DFA转换成等价的NFA,亦可以反过来转换。[子集构造法](子集构造法.md)就是将NFA转换成DFA的算法。
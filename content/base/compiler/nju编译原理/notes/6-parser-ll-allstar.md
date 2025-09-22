1. ANTLR4自动将类似expr的左递归规则重写成非左递归形式
2. ANTLR4提供优秀的错误报告功能和复杂的错误恢复机制
3. ANTLR4使用了一种名为`Adaptive LL(*)`的新技术
4. ANTLR4几乎能处理任何文法（二义性文法/间接左递归）

- 1995年：[ANTLR: A predicated‐LL(k) parser generator - Parr - 1995 - Software: Practice and Experience - Wiley Online Library](https://onlinelibrary.wiley.com/doi/abs/10.1002/spe.4380250705)
- 2011年：[LL(\*): the foundation of the ANTLR parser generator: ACM SIGPLAN Notices: Vol 46, No 6](https://dl.acm.org/doi/abs/10.1145/1993316.1993548)
- 2014年：[Adaptive LL(\*) parsing: the power of dynamic analysis: ACM SIGPLAN Notices: Vol 49, No 10](https://dl.acm.org/doi/abs/10.1145/2714064.2660202)

## ANTLR是如何消除左递归的

```antlr
stat: 
	expr ';' EOF;
expr:
	| expr '*' expr
	| expr '+' expr
	| INT
	| ID
	;
```

#todo 
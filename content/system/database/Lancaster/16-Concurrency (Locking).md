
## two-phase-locking

读写的时候加锁，不让别人抢
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/06/25/20230625000648-b4e7bd.png)

### 原理

增长阶段只能**加锁**不能释放，收缩阶段只能**放锁**，不能加锁。
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/06/25/20230625000751-c5329d.png)

你放一点锁，我拿一点锁，你放一点，我拿一点。

问题：级联回滚问题

解决办法：COMMIT时一下子撤销。
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/06/25/20230625001248-45835e.png)


## UNIVERSE OF SCHEDULES


![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/06/25/20230625001648-d17a9b.png)


## 死锁避免

- Timing out locks
	- 只会等一段时间，超时释放所有锁滚蛋，干别的去
	- MYSQL 是 50s
	- 

## 死锁检测

1. 直接死掉
2. 年轻的去等
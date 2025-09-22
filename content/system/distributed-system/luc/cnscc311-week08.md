# Security and Cryptography

<iframe src="https://onedrive.live.com/embed?resid=C5FEA982BBD2F6E%21263963&amp;authkey=%21ACA9aPVQ4-tbCCA&amp;em=2" width="714px" height="432px" frameborder="0"></iframe>

## 分布式系统的安全问题

### Eavesdropping(窃听)

![image-20231126153920774](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/26/image-20231126153920774-f8787f.png)

### Masquerading(冒充)

假装成别人。

![image-20231126154118416](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/26/image-20231126154118416-a669e1.png)

### Tampering(篡改)

非法更改讯息或资料。

![image-20231126154124549](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/26/image-20231126154124549-536f12.png)

### Replay attack [重放攻击](../../../wiki/重放攻击.md)

攻击者发送一个目的主机已接收过的包，来达到欺骗系统的目的，主要用于身份认证过程，破坏认证的正确性。

![image-20231126154131489](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/26/image-20231126154131489-186c29.png)

### Repudiation(拒绝)

否认某事的真实性或有效性。

![image-20231126154137646](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/26/image-20231126154137646-058e57.png)



## 分布式系统的安全机制

1. 加密
   - 将正常文本转换为难以理解的代码。
   - 实现消息的机密性。
2. 认证
   - 核实各方声称的身份。
   - 现在主流的是OAuth2
3. 数字签名
   - 防止篡改
   - **比较**签名**哈希值**来证明消息的真实性。
4. 授权
   - 定义给定验证方的访问权限。
   - eg, 只有CNSCC311的学生可以访问分布式系统模块的材料。
5. 审核
   - 用于分析安全漏洞。
   - eg, 检查系统的哪个部分安全性最弱，所有人都遵守安全规定。



## 密码学的角色

### 加密图片的例子

FROM：https://github.com/Jinnrry/Jencryption

加密前: ![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/26/20231126155019-5a70e3.png) 加密后: ![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/26/20231126155048-c781d7.png)



## 实施各种安全机制





## 进行安全管理
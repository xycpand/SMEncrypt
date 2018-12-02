# Encrypt
图解SM2算法流程 

https://blog.csdn.net/samsho2/article/details/80772228

SM2公钥加解密算法解析

https://blog.csdn.net/boliwu/article/details/81510305

Java实现国密算法SM2,SM3,SM4

https://blog.csdn.net/Soul_Programmer_Swh/article/details/80375958
 
 
一、椭圆曲线密码算法

    椭圆曲线：是一类二元多项式方程，它的解构成一个椭圆曲线。

有限域上的椭圆曲线：椭圆曲线上的解不是连续的，而是离散的，解的值满足有限域的限制。有限域有两种，Fp和F2m(不解释，想要了解的看客请阅读相关标准)。

Fp ：一个素整数的集合，最大值为P-1，集合中的值都是素数，里面元素满足以下模运算: a+b=(a+b) mod p 和 a*b=(a*b) mod p。

椭圆曲线参数：定义一条唯一的椭圆曲线。介绍其中两个参数G（基点）和n（阶）。G点(xG, yG)是椭圆曲线上的基点, 有限域椭圆曲线上所有其他的点都可以通过G点的倍乘运算计算得到，即P=[d]G， d也是属于有限域，d的最大值为素数n。

SM2：有限域Fp上的一条椭圆曲线，其椭圆曲线参数是固定值。

       加密的依据：P=[d]G，G是已知的，通过d计算P点很容易，但是通过P点倒推d是计算上难以实现的，因此，以d为私钥，P点(XP, YP)为公钥。

 

二、SM2加密算法

输入：长度为klen的比特串M, 公钥PB

输出：SM2结构密文比特串C

算法：

    1) 产生随机数k, k的值从1到n-1；

       2) 计算椭圆曲线点C1=[k]G=(x1,y1), 将C1转换成比特串；

       3) 验证公钥PB, 计算S=[h] PB，如果S是无穷远点，出错退出；

       4) 计算(x2,y2)=[k] PB

              5) 计算t=KDF(x2||y2, klen), KDF是密钥派生函数，如果t是全0比特串，返回第1）步。

       6) 计算C2=M+t

       7) 计算C3=Hash(x2||M||y2)

       8) 输出密文C=C1||C3||C2,  C1和C3的长度是固定的，C1是64字节，C3是32字节，很方便C从中提取C1，C3和C2。

注：通过密钥派生函数计算，才能进行第6)步的按位异或计算。

 

三、SM2解密算法

输入：SM2结构密文比特串C, 私钥dB

输出：明文M’

算法：

    1) 从密文比特串C=C1||C3||C2中取出C1， 将C1转换成椭圆曲线上的点；

       2) 验证C1, 计算S=[h] C1，如果S是无穷远点，出错退出；

       4) 计算(x2,y2)=[dB] C1

              5) 计算t=KDF(x2||y2, klen), KDF是密钥派生函数，如果t是全0比特串，出错退出。

       6) 从C=C1||C3||C2中取出C2，计算M’= C2+t。

       7) 计算u=Hash(x2||M’||y2)，比较u是否与C3相等，不相等则退出。

       8) 输出明文M’。

 

四、SM2加解密算法浅析

       想要成功解密出原文，必须是公钥PB和私钥dB是匹配的，即满足PB=[dB]G，为什么呢？

       (1)成功解密的前提

加密算法中计算得到的(x2,y2)=[k] PB与解密算法中计算得到的(x2,y2)=[dB] C1两个(x2,y2)要相等。

       那么 (x2,y2) =[k] PB=[dB] C1 ，而C1=[k]G，则可以推导

[dB] C1=[ dB][k]G=[k][ dB]G=[k] PB

 

       (2) C2=M+t和M’= C2+t 解密

       在加密和解密算法中，当(x2,y2)的值相等了，那么t=KDF(x2||y2, klen)必定相等。那么问题来了，为什么M’= C2+t就能还原出原文？

   M’= C2+t=M+t+t

 原文经过两次与同一比特串的异或计算，结果还是原文。所以，SM2算法想要成功解密，必须使用与加密公钥对于的私钥，这样才能通过密钥派生函数计算出的异或比特串才能和加密时计算的异或比特串完全一致。

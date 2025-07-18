## 通用基础

### 古典密码

凯撒密码

维吉尼亚密码

栅栏密码

### 数学基础

数论（模运算、欧拉函数、质因数分解）

代数（群、环、域的基本概念）

概率论（随机性分析、期望与方差）

计算复杂性理论（P类、NP类、NP完全问题）

### 后量子密码学前置知识

量子计算机计算原理

初步学习格问题（Lattice Problem）

矩阵论基础

初步学习学习具有误差（LWE）问题

## CTF中的密码学

### 现代密码学前置知识

#### 常用工具

##### 通用工具

CyberChef

Hashcat

John the Ripper

##### sage

代数运算、因数分解、椭圆曲线计算

算力一般低于在线网站 [factordb](https://factordb.com/)

##### python

最常用的工具，也最具有普适性

常用函数库：

- `math`：基本数学运算。
- `random` 和 `secrets`：随机数生成。
- `hashlib`：计算散列值。
- `pycryptodome`：实现对称加密和非对称加密算法。
- `gmpy2`：

##### yafu

基于python的工具，常用来在整数域内进行

##### ~~chatgpt~~

用于辅助作用，而非直接参与密码学核心计算

#### 编码与转换

Base64

URL编码

Hex编码

相关工具

#### 随机数与伪随机数

随机数生成原理与伪随机数预测

### 对称加密

使用相同密钥进行加密和解密

优点：高效快速，适合处理大数据

缺点：密钥管理困难

#### AES（高级加密标准）

加密模式：ECB、CBC、CFB、OFB、GCM

分组长度固定为128位，密钥长度为128、192、256位

#### DES（数据加密标准）

分组长度为64位，密钥长度为56位

安全性不足，易受暴力破解，常用于历史遗留系统

#### 3DES（三重数据加密标准）

使用三个不同的DES密钥，提高安全性；但性能较差

#### Blowfish

分组长度为64位，密钥长度1-448位，速度快且灵活

 常用于密码保护工具。

### 非对称加密

一对密钥，公钥加密，私钥解密。

优点：密钥分发简单

缺点：计算效率较低

#### 算法

##### RSA（Rivest–Shamir–Adleman）

基于的数学困难问题：基于大整数因数分解的困难性

##### ECC（椭圆曲线密码学）

基于的数学困难问题：椭圆曲线离散对数问题（ECDLP）

##### ElGamal

由DH（Diffie-Hellman密钥交换）产生的加密算法

#### 常见攻击

##### RSA-直接分解

##### RSA-低密度指数攻击

##### RSA-维纳攻击



### 散列密码

#### 前置知识

抗碰撞安全性和不可逆安全性

#### 具体算法

MD5

SHA系列

#### 碰撞与破解

生日攻击、彩虹表

工具与脚本实现

### 后量子密码学

背景：量子计算对传统加密算法的威胁

基于格的密码学（如NTRU、LWE）

哈希签名方案

代码密码学



1. - 常用解密工具（如CyberChef、Hashcat、John the Ripper等）
   - 编写自己的脚本（Python与常用密码库）
   - 自动化脚本化破解方法
2. 案例分析
   - 国内外经典CTF比赛的密码学题目解析
   - 解题流程与思维分享

## 密码学科研方向

### 入门路径

#### 基础知识

科普书籍《数字签名密史 从急需到有趣》郭福春

入门书籍《Modern Cryptography: Theory and Practice》Mao Wenbo 或其中文版

#### 安全规约

入门书籍《Introduction to Security Reduction》Fuchun Guo 或其中文版

#### 公钥密码30篇

强烈推荐《公钥密码方案构造及安全证明的知识要点和方法论》赵臻  

### 常用资源

#### 论文检索

[谷歌学术](https://scholar.google.com.hk/)

[DBLP](https://dblp.org/pid/67/5990.html)

~~[找学姐要](https://www.bilibili.com/video/BV1gV411i7vq)~~

#### 三大密与BIG4

##### 密码学三大顶会

| 简称      | 名称                                                         | 出版社   | 会议网址                             | 推荐等级 |
| --------- | ------------------------------------------------------------ | -------- | ------------------------------------ | -------- |
| Crypto    | International Cryptology Conference                          | Springer | https://iacr.org/meetings/crypto/    | A        |
| Eurocrypt | European Cryptology Conference                               | Springer | https://iacr.org/meetings/eurocrypt/ | A        |
| Asiacrypt | Annual International Conferenceon the Theory and Application of Cryptology and Information Security | Springer | https://iacr.org/meetings/asiacrypt/ | A        |

##### 信息安全四大顶会与分级

| 编号 | 会议简称        | 会议名称                                               | 出版社             | 会议网址                              | 推荐等级 |
| ---- | --------------- | ------------------------------------------------------ | ------------------ | ------------------------------------- | -------- |
| 1    | CCS             | ACM Conference on Computer and Communications Security | ACM                | http://dblp.uni-trier.de/db/conf/ccs/ | A        |
| 2    | S&P             | IEEE Symposium on Security and Phivacy                 | IEEE               | http://dblp.uni-trier.de/db/conf/sp/  | A        |
| 3    | Usenix Security | Usenix Security Symposium                              | USENIX Association | http://dblp.uni-trier.de/db/conf/uss/ | A        |
| 4    | NDSS            | ISOC Network and Disthibuted System SecuritySyumposium | ISOC               | https://www.ndss-symposium.org/       | B        |

#### 论文评级

##### CCF评级

##### 中科院评级

##### SCI评级

### 热门方向

#### 后量子密码学

#### 同态加密

#### 区块链

#### 多方安全计算（MPC）

### 不是太热门但是比较容易做的方向

#### 国密

##### SM2

##### SM3

##### SM9

#### 群签名/环签名

#### 变色龙哈希/可修改区块链

##### 密钥泄露

##### 即时陷门

#### 盲签名

#### 隐写
# PENETRATION-渗透

渗透测试（Penetration Testing），简称“渗透”，是一种模拟黑客攻击的网络安全技术，目的是在不造成实际破坏的前提下，测试系统、网络或应用程序的安全性。通过发现和利用系统中的漏洞，渗透测试帮助企业或组织识别潜在风险，从而提前修补漏洞，提升整体防护能力。渗透测试常涉及信息收集、漏洞分析、利用、权限提升与横向移动等多个阶段。

## 入门路线

### 熟练使用计算机

熟练使用计算机可以先从学习编程语言开始

编程语言可以先从 `C语言` 或者 `Python`(C语言最为基础, 更容易理解计算机的底层, Python最为简单上手) 学起, 但是并不是要求要掌握很熟练的地步(没有长期的使用是不可能的), 而是只要能看懂代码的整体结构, 会运用 AI 辅助编写代码即可, 提高则需要在平时使用中继续学习

千万不要让学习编程语言拖累后续学习的进度

### Web 基础

可以先从学习 OWASP Top 10 开始, 掌握 Web 漏洞基础的原理(知道漏洞如何利用, 如何产生, 如何修复)

> 或者也可以先去学习 CTF 的 Web 方向

| 漏洞类型 | 描述 | 典型攻击场景 |
|--------|------|-------------|
| **A01: Broken Access Control** | 权限控制不严，导致越权访问或操作。 | 修改请求参数访问他人数据。 |
| **A02: Cryptographic Failures** | 加密实现不当，敏感数据泄露。 | 明文传输密码或敏感信息。 |
| **A03: Injection** | SQL/NoSQL/命令注入，攻击者可执行恶意代码。 | 输入特殊字符绕过验证，执行 SQL 语句。 |
| **A04: Insecure Design** | 业务流程设计不严谨，易被绕过。 | 跳过支付流程直接访问成功页面。 |
| **A05: Security Misconfiguration** | 配置错误导致安全缺陷。 | 默认密码、调试接口未关闭。 |
| **A06: Vulnerable and Outdated Components** | 使用有漏洞的第三方库或组件。 | 依赖未及时更新，存在已知漏洞。 |
| **A07: Identification and Authentication Failures** | 身份认证机制不安全。 | 弱密码、验证码可被暴力破解。 |
| **A08: Software and Data Integrity Failures** | 代码或数据完整性校验不足。 | 未校验更新包来源，遭供应链攻击。 |
| **A09: Security Logging and Monitoring Failures** | 缺乏安全日志和监控，攻击难以发现。 | 无法及时检测和响应安全事件。 |
| **A10: Server-Side Request Forgery (SSRF)** | 服务器被诱导请求恶意地址。 | 利用接口让服务器访问内网资源。 |

这里提供几个经典的开源靶场(推荐自己搭建, 而不是去找网上现成的靶场, 便于学习网站运维部署), 自己根据自己情况选择练习(一定要练习实践)

| 靶场名称 | 简介 | 项目地址 |
|---------|------|---------|
| **SQLi-Labs** | 专注于 SQL 注入的靶场，涵盖报错注入、盲注（布尔、时间）、联合注入等多种场景。 | [GitHub - Audi-1/sqli-labs](https://github.com/Audi-1/sqli-labs) |
| **XSS-Labs** | XSS 漏洞学习平台，包含多种反射型、存储型、DOM 型 XSS 练习关卡。 | [GitHub - do0dl3/xss-labs](https://github.com/do0dl3/xss-labs) |
| **DVWA** | 著名的综合性 Web 安全靶场，包含 SQLi、XSS、CSRF、文件包含等多类漏洞，难度可调。 | [GitHub - digininja/DVWA](https://github.com/digininja/DVWA) |
| **upload-labs** | 专注于文件上传漏洞的靶场，帮助学习各种绕过手法（后缀、MIME、内容绕过等）。 | [GitHub - c0ny1/upload-labs](https://github.com/c0ny1/upload-labs) |
| **pikachu** | 轻量、易用的 Web 漏洞靶场，涵盖 XSS、CSRF、SQLi、文件包含、命令执行等常见漏洞。 | [GitHub - zhuifengshaonianhanlu/pikachu](https://github.com/zhuifengshaonianhanlu/pikachu) |
| **bWAPP (bee-box)** | 综合性漏洞学习平台，配套虚拟机 bee-box 使用，覆盖 OWASP Top 10 漏洞。 | [SourceForge - bWAPP](https://sourceforge.net/projects/bwapp/files/bee-box/) |
| **DoraBox** | 基础 Web 漏洞训练平台，适合入门练习常见漏洞。 | [GitHub - 0verSp4ce/DoraBox](https://github.com/0verSp4ce/DoraBox) |

## 其他路线

对于初学者来说，也可以先阅读 [靶场-tryhackme](./靶场/tryhackme/index.md) 目录下的 `路径选择` 部分，稍作学习后再进行其他目录下的学习
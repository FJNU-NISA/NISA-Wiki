# PENETRATION-渗透

渗透测试（Penetration Testing），简称“渗透”，是一种模拟黑客攻击的网络安全技术，目的是在不造成实际破坏的前提下，测试系统、网络或应用程序的安全性。通过发现和利用系统中的漏洞，渗透测试帮助企业或组织识别潜在风险，从而提前修补漏洞，提升整体防护能力。渗透测试常涉及信息收集、漏洞分析、利用、权限提升与横向移动等多个阶段。

## 其他 Wiki 推荐, 部分内容参考引用

- [https://www.vul-wiki.org/](https://www.vul-wiki.org/)

## 入门路线

### 熟练使用计算机

熟练使用计算机可以先从学习编程语言开始

编程语言可以先从 `C语言` 或者 `Python`(C语言最为基础, 更容易理解计算机的底层, Python最为简单上手) 学起, 但是并不是要求要掌握很熟练的地步(没有长期的使用是不可能的), 而是只要能看懂代码的整体结构, 会运用 AI 辅助编写代码即可, 提高则需要在平时使用中继续学习

千万不要让学习编程语言拖累后续学习的进度

### Web 基础(可以去这个[Wiki](https://www.vul-wiki.org/vulnerability/web/web)了解更多内容)

可以先从学习几种基础的漏洞开始, 掌握 Web 漏洞基础的原理(知道漏洞如何利用, 如何产生, 如何修复)

> 或者也可以先去学习 CTF 的 Web 方向

| 漏洞类型                    | 简述                   | 典型危害             |
| ----------------------- | -------------------- | ---------------- |
| SQL 注入（SQLi）            | 用户输入未过滤直接拼接进 SQL 语句  | 数据泄露、远程命令执行      |
| 跨站脚本攻击（XSS）             | 恶意脚本注入页面，在用户浏览器执行    | 会话劫持、钓鱼、仿冒       |
| 跨站请求伪造（CSRF）            | 利用用户已登录的身份发送伪造请求     | 越权操作、资金转账        |
| 服务器端请求伪造（SSRF）          | 诱导服务端向攻击者控制的目标发请求    | 内网探测、远程代码执行      |
| 命令注入（Command Injection） | 用户输入拼接进系统命令          | 远程命令执行           |
| 文件上传漏洞                  | 文件扩展名、MIME 类型、内容验证不足 | 木马上传、WebShell 入侵 |
| 目录遍历                    | 构造路径读取服务器任意文件        | 配置泄露、源代码泄露       |
| 逻辑漏洞                    | 业务流程设计不合理、缺乏校验       | 越权、刷单、数据篡改       |


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

### Linux 基础



## 其他路线

对于初学者来说，也可以先阅读 [靶场-tryhackme](./靶场/tryhackme/index.md) 目录下的 `路径选择` 部分，稍作学习后再进行其他目录下的学习
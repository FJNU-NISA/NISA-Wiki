# Git基本使用

## 安装 Git

### Windows 上安装

[CSDN上的详细教程](https://blog.csdn.net/mukes/article/details/115693833)

## 配置 Git 

### 设置邮箱和用户名

``` bash
git config --global user.name 用户名
git config --global user.email 邮箱
```

 其中 `--global` 代表全局配置, 如果不设置, 只对当前所处仓库生效

> 如果想把仓库同步到 Github 上后, 可以正确显示自己账号和头像, 需要正确设置与 Github 相同的用户名和邮箱
> 
> 如果直接使用自己的邮箱, 在同步仓库时, Github 会发出警告, 这个时候需要把邮箱改为 ******@users.noreply.github.com 的邮箱, 可以前往 [https://github.com/settings/emails](https://github.com/settings/emails), 查看自己对应的地址, 以起到保护隐私的作用
> 
> ![github_email.png](./assets/github_email.png)

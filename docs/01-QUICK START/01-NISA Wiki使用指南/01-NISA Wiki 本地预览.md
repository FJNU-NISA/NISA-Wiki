# NISA Wiki 本地预览

请在同步到云端仓库前, 先在本地进行测试

## Clone-克隆

在确保安装 git 与可以正常访问 Github 后, 在一个合适的存储路径, 运行如下命令

```bash
git clone https://github.com/FJNU-NISA/NISA-Wiki
```

如果顺利的话, 这时会下载下来一个 `NISA-Wiki` 命名的文件夹

## 安装Python及其依赖库

python 安装这里就不详细介绍了, 下面是安装依赖的命令

```bash
pip install -r requestment.txt
```

## 启动本地预览服务

```bash
mkdocs serve
```
# NISA Wiki 常见问题

## 有序列表中嵌套 Latex 语法

由于一些问题无法解决, 只能通过如下方法实现

1. 分数：
> $$
> \frac{a + b}{c}
> $$
2. 开方：
> $$
> \sqrt{a^2 + b^2}
> $$

语法原文：

```txt
1. 分数：
> $$
> \frac{a + b}{c}
> $$
2. 开方：
> $$
> \sqrt{a^2 + b^2}
> $$
```

## 高级提示框写法

语法模板

```
!!! <类型> "<小标题>"
    <内容>
```

<类型> 支持以下几种

- note（笔记）
- abstract（摘要）
- info（信息）
- tip（提示）
- success（成功）
- question（问题）
- warning（警告）
- failure（失败）
- danger（危险）
- bug（错误）
- example（范例）
- quote（引言）

其他的自己试一下就知道原文与渲染后所对应的位置了, 渲染后示例如下

!!! note "这是note提示框小标题"
    这是note提示框内容

!!! abstract "这是abstract提示框小标题"
    这是abstract提示框内容

!!! info "这是info提示框小标题"
    这是info提示框内容

!!! tip "这是tip提示框小标题"
    这是tip提示框内容

!!! success "这是success提示框小标题"
    这是success提示框内容

!!! question "这是question提示框小标题"
    这是question提示框内容

!!! warning "这是warning提示框小标题"
    这是warning提示框内容

!!! failure "这是failure提示框小标题"
    这是failure提示框内容

!!! danger "这是danger提示框小标题"
    这是danger提示框内容

!!! bug "这是bug提示框小标题"
    这是bug提示框内容

!!! example "这是example提示框小标题"
    这是example提示框内容

!!! quote "这是quote提示框小标题"
    这是quote提示框内容


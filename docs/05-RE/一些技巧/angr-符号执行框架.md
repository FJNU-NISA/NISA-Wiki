# 参考
[[angr\_ctf——从0学习angr（一）：angr简介与核心概念 - Uiharu - 博客园](https://www.cnblogs.com/level5uiharu/p/16925991.html)]

[[angr学习与angr\_CTF题解（持续更新） \| starrysky](https://starrysky1004.github.io/2024/12/17/angr-xue-xi-yu-angr-ctf-ti-jie-chi-xu-geng-xin/angr-xue-xi-yu-angr-ctf-ti-jie-chi-xu-geng-xin/#toc-heading-5)]

# 符号执行简介

随着程序的执行流程的日益复杂, 我们避免通过极大的经历去手动分析程序, 因此科学家提出了符号执行的算法

angr是一个支持多处理架构的用于二进制文件分析的工具包，它提供了动态符号执行的能力以及多种静态分析的能力。项目创建的初衷，是为了整合此前多种二进制分析方式的优点，并开发一个平台，以供二进制分析人员比较不同二进制分析方式的优劣，并根据自身需要开发新的二进制分析系统和方式。

也正是因为angr是一个二进制文件分析的工具包，因此它可以被使用者扩展，用于自动化逆向工程、漏洞挖掘等多个方面。

# 基本的引入

**我会通过列举一些代码示例(当成伪代码看吧)**

```cpp
if(input_lenth > 10){
	printf("hello angr");
}
```
这个就是非常的简单, 当我们满足了 `input_lenth > 10` 这一个不等式, 就可以得到输出

```cpp
int main(){
	int user_input;
	scanf("%d",&user_input);
	if(user_input - 10 >= 0 ){
		if(user_input <= 20){
			puts("success");
		}
		else{
			pust("no");
		}
	}
	else{
		puts("false");
	}
}
```

# angr的核心概念

## 顶层接口

Project类是angr的主类，也是angr的开始，通过初始化该类的对象，可以将你想要分析的二进制文件加载进来，就像这样：

```python
import angr
p = angr.Project('/bin/true')
```

参数为待分析的文件路径，它是唯一必须传入的参数，此外还有一个比较常用的参数load-options，它指明加载的方式，如下：

|                     |              |      |
| ------------------- | ------------ | ---- |
| 名称                  | 描述           | 传入参数 |
| auto_load_libs      | 是否自动加载程序的依赖  | 布尔   |
| skip_libs           | 希望避免加载的库     | 库名   |
| except_missing_libs | 无法解析库时是否抛出异常 | 布尔   |
| force_load_libs     | 强制加载的库       | 库名   |
| ld_path             | 共享库的优先搜索路径   | 路径名  |

使用angr时最重要的就是效率问题，少加载一些无关结果的库能够提升angr的效率，如下：

```python
import angr
p = angr.Project('/bin/true', auto_load_libs=False)
```

任何附加的参数都会被传递到angr的加载器，即CLE.loader中（CLE 即 CLE Loads Everything的缩写）

Project类中有许多方法和属性，例如加载的文件名、架构、程序入口点、大小端等等：

```python
>>> print(p.arch, hex(p.entry), p.filename, p.arch.bits, p.arch.memory_endness )
<Arch AMD64 (LE)> 0x4023c0 /bin/true 64 Iend_LE
```

### State

Project实际上只是将二进制文件加载进来了，要执行它，实际上是对SimState对象进行操作，它是程序的状态。用docker来比喻，Project相当于开发环境，State则是使用开发环境制作的镜像。

要创建状态，需要使用Project对象中的factory，它还可以用于创建模拟管理器和基本块（后面提到），如下：

`init_state = p.factory.entry_state()`

预设状态有四种方式如下：

|                    |                               |
| ------------------ | ----------------------------- |
| 预设状态方式             | 描述                            |
| entry_state        | 初始化状态为程序运行到程序入口点处的状态          |
| blank_state(addr=) | 大多数数据都没有初始化，状态中下一条指令为addr处的指令 |
| full_init_state    | 共享库和预定义内容已经加载完毕，例如刚加载完共享库     |
| call_state         | 准备调用函数的状态                     |

状态包含了程序运行时的一切信息，寄存器、内存的值、文件系统以及**符号变量**等，这些信息的使用等用到时再进一步说明。

entry_state和blank_state是常用的两种方式，后者通常用于跳过一些极大降低angr效率的指令，它们间的对比如下：

```python
>>> state = p.factory.entry_state()
>>> print(state.regs.rax, state.regs.rip)
<BV64 0x1c> <BV64 0x4023c0>
```

```python
>>> state = p.factory.blank_state(addr=0x4023c0)
>>> print(state.regs.rax, state.regs.rip)
<BV64 reg_rax_42_64{UNINITIALIZED}> <BV64 0x4023c0>
```

在blank_state方式中，我们仍将地址设定为程序的入口点，然而rax中的值由于没有初始化，它现在是一个名字，也即符号变量，这是符号执行的基础，后续在细说。

此外，可以看到寄存器中的数据类型并不是int，而是BV64，它是一个位向量（Bit Vector），有关位向量的细节之后再说。

### Simulation Manager

上述方式只是预设了程序开始分析时的状态，我们要分析程序就必须要让它到达下一个状态，这就需要模拟管理器的帮助（简称SM）.

使用以下指令能创建一个SM，它需要传入一个state或者state的列表作为参数：

```python
simgr  = p.factory.simgr(state)
```

SM中有许多列表，这些列表被称为stash，它保存了处于某种状态的state，stash有如下几种：

|               |                                                                                     |
| ------------- | ----------------------------------------------------------------------------------- |
| stash         | 描述                                                                                  |
| active        | 保存接下来可以执行并且将要执行的状态                                                                  |
| deadended     | 由于某些原因不能继续执行的状态，例如没有合法指令，或者有非法指针                                                    |
| pruned        | 与solve的策略有关，当发现一个不可解的节点后，其后面所有的节点都优化掉放在pruned里                                      |
| unconstrained | 如果创建SM时启用了save_unconstrained，则被认定为不受约束的state会放在这，不受约束的state是指由用户数据或符号控制的指令指针（例如eip） |
| unsat         | 如果创建SM时启用了save_unsat，则被认为不可满足的state会放在这里                                            |

默认情况下，`state`会被存放在`active`中。

`stash`中的`state`可以通过`move()`方法来转移，将`fulter_func`筛选出来的`state`从`from_stash`转移到`to_stash`：

```python
simgr.move(from_stash='deadended', to_stash='more_then_50', filter_func=lambda s: '100' in s.posix.dumps(1))
```

`stash`是一个列表，可以使用python支持的方式去遍历其中的元素，也可以使用常见的列表操作。但`angr`提供了一种更高级的方式，在stash名字前加上one_，可以得到stash中的第一个状态，加上mp_，可以得到一个mulpyplexed版本的stash

此外，稍微解释一下上面代码中的posix.dumps：
- `state.posix.dumps(0`):表示到达当前状态所对应的程序输入
- `state.posix.dumps(1)`:表示到达当前状态所对应的程序输出

上述代码就是将deadended中输出的字符串包含'100'的`state`转移到`more_then_50`这个stash中。

可以通过step()方法来让处于active的state执行一个基本块，这种操作不会改变state本身：

```python
>>> state = p.factory.entry_state()
>>> simgr = p.factory.simgr(state)
>>> print(state.regs.rax, state.regs.rip)
<BV64 0x1c> <BV64 0x4023c0>

>>> print(simgr.one_active)
<SimState @ 0x4023c0>

>>> simgr.step()
<SimulationManager with 1 active>
>>> print(simgr.one_active)
<SimState @ 0x529240>

>>> print(state.regs.rax, state.regs.rip)
<BV64 0x1c> <BV64 0x4023c0>
```

### explorer

用于探索模拟状态直到找到特定条件满足的状态的方法，有 **find**（目标指令的地址或地址列表） 和 **avoid**（避免的指令地址或地址列表） 两种参数，找到符合的 find 状态会保存在 **simger.found** 列表中，可遍历元素获取状态

可以使用explorer方法去执行某个状态，直到找到目标指令或者active中没有状态为止，它有如下参数：
- `find`：传入目标指令的地址或地址列表，或者一个用于判断的函数，函数以state为形参，返回布尔值
- `avoid`：传入要避免的指令的地址或地址列表，或者一个用于判断的函数，用于减少路径
它还有一些的搜索策略

# 总结

1. 创建Project, 预设state
2. 创建位向量和符号变量, 保存在内存, 寄存器, 或者在其它的地方
3. 将state添加在SM中
4. 运行, 探索满足条件的路径
5. 约束求解获得执行结果

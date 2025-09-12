[IDAPython: Welcome to IDAPython API Reference 9.1](https://python.docs.hex-rays.com/index.html)


# 基础



## idapython的结构


- **idc模块** --> 封装idc函数
- **idautils模块** --> IDA中的高级实用函数
- **idaapi模块** --> 允许访问更多的低级数据, 可以通过ida被引用
## ida的基础知识
- `段名称`
- `地址`
- `操作符`
- `EA` : 当前光标的地址 
## 基础的函数操作函数
- `idc.get_screen_ea()`
- `idc.here()`
- ` idc.get_inf_attr(INF_MIN_EA)`
- `idc.get_inf_attr(INF_MAX_EA)`
- `idc.get_segm_name()
- `idc.generate_disasm_line()`
- `idc.print_insn_name()
- `idc.print_oprand()`
- `ida_ida.inf_get_min_ea()`  
    获取最小地址
- `ida_ida.inf_get_start_ip()`
- `ida_ida.inf_get_max_ea()`  
    获取最大地址
- `idc.read_selection_start()`  
    当前选择区块的起始地址
- `idc.read_selection_end()`  
    当前选择区块的结束地址

## 段的操作
- ` idautils.Segments()`
     获取一个段的遍历类型
- `idc.get_next_seg(ea)`
     获取下一个段
- `idc.selector_by_name(segname)
     返回段选择器并且传递单个字符段名称的函数
- `idc.get_segm_by_sel(idc.selector_by_name(str,SectionName))`
     获取一个段的开始地址
## 函数操作
- `idautils.Functions() /  idautils.Funtions(start_addr,end_addr)`
- `idautils.Function()将返回一个已经知道的函数列表`
- `ida.get_func_name()返回函数名称`
- `idaapi.get_func(ea)获取函数边界`
- `start_ea和end_ea 获取起始和结束地址`

```cs
Python>for func in idautils.Functions(): print("0x%x, %s" % (func, idc.get_func_name(func)))
 
Python>
 
>>> 0x401000, sub_401000
 
>>> 0x401006, w_vfprintf
 
>>> 0x401034, _main
 
>>> …removed…
 
>>> 0x401c4d, terminate
 
>>> 0x401c53, IsProcessorFeaturePresent
```

- `idc.get_next_func(ea)和idc.get_prev_func(ea) `访问上/下函数 
- 访问上/下函数 idc.get_next_func(ea)和idc.get_prev_func(ea)

```cs
Python>func = idaapi.get_func(ea) Python>type(func) >>> <class 'ida_funcs.func_t'> Python>print("Start: 0x%x, End: 0x%x" % (func.start_ea, func.end_ea)) >>> Start: 0x45c7c3, End: 0x45c7cd
```

- `idc.get_func_attr(ea,FUNCATTR_START)和idc.get_func_attr(ea,FUNCATTR_END)`访问函数边界
- `idc.generate_disasm_line(ea,0)`打印当前汇编代码
- `idc.next_head(eax)`下一条指令，直到函数结束\
- 
```cs
for func in idautils.Functions():
 
           flags = idc.get_func_attr(func,FUNCATTR_FLAGS)
 
           if flags & FUNC_NORET:                 #无返回标志的函数
 
               print("0x%x FUNC_NORET" % func)
 
           if flags & FUNC_FAR:                   # 这个标志很少出现，除非逆向软件使用分段内存。它的内部表示为一个整数 2。
 
               print("0x%x FUNC_FAR" % func)
 
           if flags & FUNC_LIB:  #此标志用于查找库代码，在内部表示为整数4。
 
               print("0x%x FUNC_LIB" % func)
 
           if flags & FUNC_STATIC: #此标志用于标识基于静态ebp框架的库函数
 
               print("0x%x FUNC_STATIC" % func)
 
           if flags & FUNC_FRAME:  # 这个标志表明该函数使用帧指针 EBP。使用帧指针的函数通常以设置堆栈框架的标准函数序言开始。
 
               print("0x%x FUNC_FRAME" % func)
 
           if flags & FUNC_USERFAR:    #这个标志是罕见的，hexrays 描述标志为“用户指定了函数距离”。它的内部值为 32。
 
               print("0x%x FUNC_USERFAR" % func)
 
           if flags & FUNC_HIDDEN:  #函数带 FUNC_HIDDEN 标志意味着他们是隐藏的将需要扩展到视图。如果我们转到一个被标记为隐藏的函数的地址，它会自动扩展。
 
               print("0x%x FUNC_HIDDEN" % func)
 
           if flags & FUNC_THUNK: #这标志标识函数是 thunk 函数。一个简单的功能是跳到另一个函数。
 
               print("0x%x FUNC_THUNK" % func)
 
           if flags & FUNC_BOOTOMBP:          # 此标志用于跟踪帧指针。标识指针指向堆栈指针的函数
 
               print("0x%x FUNC_BOTTOMBP" % func)
```

## 获取和处理数值

###### 获取数值
- `idc.get_wide_byte(addr) 旧版：Byte(addr)`  
    以字节为单位获取地址处的值
- `idc.get_wide_word(addr) 旧版：Word(addr)`  
    以2字节(字)为单位获取地址处的值
- `idc.get_wide_dword(addr) 旧版：Dword(addr)`  
    以4字节为单位获取地址处的值
- `idc.get_qword(addr) 旧版：Qword(addr)`  
    以8字节为单位获取地址处的值
###### 处理数值
- `ida_bytes.patch_byte(addr,value) 旧版：idc.PatchByte(addr,value)`  
    修改addr地址的值为value.每次修改一个字节
- `ida_bytes.patch_word(addr,value) 旧版：idc.PatchWord(addr,value)`  
    修改addr地址的值为value.每次修改两个字节
- `ida_bytes.patch_Dword(addr,value) 旧版：idc.PatchDword(addr,value)`  
    修改addr地址的值为value.每次修改四个字节
- `ida_bytes.patch_Qword(addr,value) 旧版：idc.PatchQword(addr,value)`  
    修改addr地址的值为value.每次修改八个字节
## 提取函数参数

`idaapi.get_arg_addrs(ea)`

## 示例

```python
dism_addr = list(idautils.FuncItems(here()))
print(dism_addr)
 
for line in dism_addr:
    print("0x%x %s" % (line,idc.generate_disasm_line(line,0)))
```

```python
for func in idautils.Functions():
 
    flags = idc.get_func_attr(func, FUNCATTR_FLAGS)
 
    if flags & FUNC_LIB or flags & FUNC_THUNK:
 
       continue
 
    dism_addr = list(idautils.FuncItems(func))
 
    for line in dism_addr:  #循环每条指令访问下一条指令
 
          m = idc.print_insn_mnem(line)
 
          if m == 'call' or m == 'jmp':
 
                   op = idc.get_operand_type(line, 0)
 
                   if op == o_reg:
 
                      print "0x%x %s" % (line, idc.generate_disasm_line(line,0))
 
>>>0x43ebde call eax ; VirtualProtect
```

```python
ea = here()
print("0x%x %s" % (ea, idc.generate_disasm_line(ea,0)))
>>> 0x10004f24 call    sub_10004f32
next_instr = idc.next_head(ea)
print("0x%x %s" % (ea, idc.generate_disasm_line(next_instr,0)))
>>> 0x10004f29 mov  [esi],eax
prev_instr = idc.prev_head(ea)
print("0x%x %s" % (ea, idc.generate_disasm_line(prev_instr,0)))
>>>0x10004f1e mov [esi+98h], eax
print ("0x%x" % idc.next_addr(ea))
>>> 0x10004f25                                #注意和next_head的区别
print ("0x%x" % idc.prev_addr(ea))
>>>0x10004f23
```

```python
Python>JMPS = [idaapi.NN_jmp, idaapi.NN_jmpfi, idaapi.NN_jmpni]
 
Python>CALLS = [idaapi.NN_call, idaapi.NN_callfi, idaapi.NN_callni]
 
Python>for func in idautils.Functions():
 
          flags = idc.get_func_attr(func, FUNCATTR_FLAGS)
 
          if flags & FUNC_LIB or flags & FUNC_THUNK:
 
             continue
 
          dism_addr = list(idautils.FuncItems(func))
 
          for line in dism_addr:
 
             ins = ida_ua.insn_t()
 
             idaapi.decode_insn(ins, line)
 
             if ins.itype in CALLS or ins.itype in JMPS:
 
                if ins.Op1.type == o_reg:   #这就找到了call 和 jmp指令地址
 
                   print("0x%x %s" % (line, idc.generate_disasm_line(line, 0))
 
Python>
 
>>> 0x43ebde call eax ; VirtualProtect
```

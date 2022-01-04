+++ 
draft = true
date = 2017-09-04T18:54:41+08:00
title = "如何恢复被破坏的OEP"
description = "How to recover a damaged OEP"
slug = ""
authors = ['he110w0r1d']
tags = ['pe', 'oep']
categories = ['Reverse Analysis', 'PE']
externalLink = ""
series = []
+++

# 遇到一个坑
练习使用反汇编引擎的时候，作者提供了一个练习的样本；但是在用的时候，怎么也反汇编不出来结果。一度怀疑是代码出了问题。难道样本用了反反汇编？那也不应该不出结果啊。换了一个样本之后，出现了反汇编结果，可以确定作者提供的样本出了问题。


使用IDA打开，尝试是否能够成功反汇编，打开出现异常的入口点地址提示。


![FKXXh6.png](https://s1.ax1x.com/2018/12/03/FKXXh6.png)

图一:IDA打开截图


查看作者提供的PE文件，竟然都是同一个入口点。可能是作者为了防止误操作运行。但是还让练习反汇编和动态分析，是不是坑。管他是不是坑，先踩了再说。

![FMZ1aV.png](https://s1.ax1x.com/2018/12/03/FMZ1aV.png)

图二：无效的入口点

---
**思路：我们都知道编程从main()函数开始，实际上系统先调用start()函数，然后再调用用户的main()函数(start函数负责系统的环境初始化)。根据start函数找入口点（类比相同的编译器的入口点函数的特征，因为不同的编译器，start函数会有所不同）**

# 0x00 找start函数的FOA
通过对比相同编译器的编译的其他样本，找到start()函数的前三条指令。

```
 push    ebp         55
 mov     ebp, esp    8B EC
 push    0FFFFFFFFh  FF 68 
```

利用十六进制编辑器，搜索前三条指令558BECFF68,在文件中共发现了三处。


![FQkhWQ.png](https://s1.ax1x.com/2018/12/04/FQkhWQ.png)

图三：搜索的结果


根据节表的信息，可以看到这三处，都位于代码节.text中。无法进行排处，先用第一个偏移`11590h`进行尝试。

![FQJ6Ig.png](https://s1.ax1x.com/2018/12/04/FQJ6Ig.png)

图四：节表

# 0x01根据FOA算出RVA
## 1.先算出节内偏移

根据节表可知，.text节在文件中的偏移为400h
```math
offset = 11590h-400h = 11190h
```
## 2.再根据内存布局算出入口点的RVA

根据节表可知,.text节的RVA地址为1000h
```math
RVA = 11190h+1000h = 12190h
```
# 0x02修改入口点地址
修复入口点地址，另存为exe.

![FQtWD0.png](https://s1.ax1x.com/2018/12/04/FQtWD0.png)

图五：修改后的入口点

用IDA反汇编后，发现已正确找到入口点。

![FQNpPe.png](https://s1.ax1x.com/2018/12/04/FQNpPe.png)

图六：start()函数



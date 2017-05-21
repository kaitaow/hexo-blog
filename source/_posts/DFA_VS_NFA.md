---
title: 正则表达式匹配原理
categories: '工具与经验'
tags: [regex]
---

## DFA 和 NFA的区别

- 1、忽略优先量词是否得到支持

如果得到支持，说明是传统型NFA，否则是POSIX NFA或DFA。

- 2、DFA 不支持捕获括号和反向引用。

据此可以区分DFA和POSIX NFA。所以，awk和egrep都不支持反向引用。

GNU版本的egrep确实支持反向引用，这是因为它包含了两种不同的引擎。

- 3、预编译阶段和匹配速度的差异

NFA的编译过程通常要快一些 需要的内存也更少一些。传统型NFA和POSIX NFA之间并没有实质的区别。

DFA在预编译阶段所做的工作提供的优化效果，要好于大多数NFA引擎复杂的优化措施。

DFA的匹配速度与正则表达式无关，而NFA中匹配速度和正则表达式直接相关。

- 4、多选结构

传统型NFA从左到右顺序检查 POSIX或DFA选择匹配文本最多的那个。

用正则表达式(nfa|nfa not) 来匹配字符串nfa not，如果只有nfa匹配了，这就是传统型NFA。

用one(self)?(selfsufficient)匹配 oneselfsufficient NFA会匹配到oneself DFA会匹配到selfsufficient

- 5、测试用例

用'X(.+)+X'匹配'=XX============================'

如果执行需要花很长时间，就是NFA。

如果时间很短，就是DFA或者是支持某些高级优化的NFA。

如果显示stack overflow，或者超时退出，那么是NFA引擎。

- 6、常用工具的分类

DFA：egrep、awk

NFA：Java、python、vi、grep



## 匹配原理

匹配过程中两条普适的原则

1、优先选择最左端的匹配结果。

2、标准的匹配量词是匹配优先的。

匹配量词总是尽可能的匹配更多的字符。如果正则表达式剩下的部分不能够继续匹配，那么开始回溯，即此时开始 ‘交还’一些字符。

用'^.*[0-9]+'匹配 'Copyright 2003.’  [0-9]+只会匹配到3 而不会匹配到2003。

### DFA(确定型有穷自动机) 

文本主导

不支持忽略优先 

DFA是确定型的(Deterministic) 目标文本中的字符只会(最多)检查一遍

DFA优势匹配速度非常快

### 传统型NFA (非确定型有穷自动机)

NFA相较与DFA的特性 忽略优先 环视 条件判断 反向引用 固化分组

表达式主导

对于传统型NFA 如果能够匹配同样文本的分支不只一个，需要注意多选结构的顺序。

通过改变表达式的编写方式，用户可以对表达式进行多方面的控制。

#### **- 匹配优先与忽略优先**

? 与 ??、* 与 *?

忽略优先量词的用途 避免过度匹配 比如匹配两个引号之间的内容，匹配html tag之间的内容等。

hello "world" hello "world” hello 如果使用[“.*”]匹配双引号之间的内容 匹配到的结果是"world" hello "world”

解决过度匹配问题的方法

1、使用忽略优先连词 [“.*?”]

2、["[^”]*”] 限定双引号之间的内容必须是非双引号 但是这种方式对html tag并不适用

3、对于html tag的情形 可以使用环视 如果要匹配<B>...</B>之间的内容

<B>((?!</?B>).)*</B>  匹配<B>和</B>之间的内容 不允许出现<B>或者</B>

#### **- 固化分组和占有优先量词**

占有优先量词和固化分组很相似。

如果某个可选元素已经匹配成功 那么就不在尝试此元素的忽略状态。

使用固化分组 意味着要去正则引擎放弃某些尝试路径 可以加快报告失败的速度。

比如 [^\w+:] 无法匹配 “Subject” 但是匹配引擎在第一次匹配冒号失败之后仍然会多次重复 ”回溯-匹配-失败“的流程。

显然所有的回溯都是没有意义的，因此可以使用固化分组放弃所有的备用状态。[^(?>\w+)]

占有优先量词 ?+ 、*+、++和{m,n}+ 

#### \- 环视 向前查找或向后查找

如果正则引擎不支持固化分组 可以考虑使用肯定环视模拟固化分组。

[(?>regex)] 可以用[(?=(regex))\1] 来模拟 比如。[^(?>\w+)] 可以用[^(?=(\w+))\1:]来模拟

多选结构是有序的。据此可以对匹配过程进行更多的控制。

如果要匹配"Jan 31”之类的日期 正则表达式应该是怎样的

Jan [0123][0-9] 无法匹配"Jan 7"

改进 使用多选结构 Jan 0?[1-9]|[12][1-9]|3[01] 只会匹配到Jan 3而不是Jan 31 有序多选结构的陷阱

调整顺序   Jan [12][1-9]|3[01]| 0?[1-9]

#### 回溯

在依次处理各个子表达式或组成元素过程中，遇到量词或者多选结果需要作出选择。对于匹配优先量词，引擎会优先选择进行尝试，对于忽略优先量词，会优先选择跳过尝试。

回溯时选择的分支 与对二叉树的深度优先遍历类似 往左节点表示匹配优先 往右节点表示忽略优先 匹配失败则退回到堆栈中的上一个节点继续尝试。

### POSIX NFA

如果传统型NFA的效率问题值得关注，那么POSIX NFA的效率就更值得关注，因为它需要尝试正则表达式的所有变体，进行更多的回溯。
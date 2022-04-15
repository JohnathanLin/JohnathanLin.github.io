---
title: "[编译原理]FIRST、FOLLOW和SELECT"
date: 2018-04-30T16:23:34+08:00
draft: false
categories: ["技术文章"]
---


## FIRST集

FIRST(α)为α的开始符号集或者首符号集。

### 定义

设G=(VT，VN，S，P)是上下文无关文法 ，FIRST(α)={a|α能推导出aβ,a∈VT，α,β∈V*} 。

特别的，若α能推导出ε,则规定ε∈FIRST(α)．

VT为终结符集，VN为非终结符集，S称作识别符或开始符，P为规则(α→β)的集合。

### 根据定义求解FIRST集

（对每一文法符号X∈V 计算FIRST(X)）

若X∈VT，则FIRST(X)={X}；
若X∈VN，且有产生式X→a...，则把a加入到FIRST(X)中；
若X∈VN，X→ε也是一条产生式，则把ε也加到FIRST(X)中；
若X→Y...是一个产生式且Y∈VN，则把FIRST(Y)中的所有非ε元素都加到FIRST(X)中；
若X→Y1 Y2 ... Yk是一个产生式，Y1，Y2，...Y（i-1）都∈VN（1≤i≤K），而且，对于任何j(1≤j≤i-1)，FIRST(Yj)都含有ε （即Y1...Y(i-1)=>*ε），则把FIRST(Yj)中的所有非ε元素和FIRST(Yi)中的所有元素都加到FIRST(X)中；
特别是，若所有的FIRST(Yj，j=1,2,..,K)均含有ε，则把ε加到FIRST(X)中。

反复使用上述2~5步，直到每个符号的FIRST集合不再增大为止。

## FOLLOW集

FOLLOW(A)为非终结符A的后跟符号集合。

### 定义

设G=(VT，VN，S，P)是上下文无关文法，A∈VN，S是开始符号，
FOLLOW(A)=｛a|S能推导出μAβ,且a∈VT，a∈FIRST(β),μ∈VT* ,β∈V+｝，若S能推导出μAβ,且β能推导出ε, 则#∈FOLLOW(A)。 也可定义为：FOLLOW(A)={a|S能推导出…Aa…,a ∈VT} ，若有S能推导出…A，则规定#∈FOLLOW(A) ，这里我们用‘#’作为输入串的结束符。

### 计算FOLLOW集
任何FOLLOW(S)都包含输入终止符号#,其中S是开始符号
如果存在产生式,A->αBβ,则将FIRST(β)中除ε以外的符号都放入FOLLOW(B)中
如果存在产生式,A->αB,或A->αBβ,其中FIRST(β)中包含ε,则将FOLLOW(A)中的所有符号都放入FOLLOW(B)中.
## SELECT集

SELECT集是选择符号集。

### 定义及计算过程

给定上下文无关文法的产生式A→α, A∈VN,α∈V*, 若α不能推导出ε,则SELECT(A→α)=FIRST(α)
如果α能推导出ε则：SELECT(A→α)=（FIRST(α) –{ε}）∪FOLLOW(A)
需要注意的是，SELECT集是针对产生式而言的。

## 例题

《编译原理》第三版，p100页，第2题：

对下面的文法G：
E→TE‘
E‘→+E|ε
T→FT'
T‘→T|ε
F→PF'
F'→*F'|ε
P→(E)|a|b|^

问：

1. 计算这个文法的每个非终结符的FIRST集和FOLLOW集。
2. 证明这个文法是LL(1)的。
3. 构造它的预测分析表。
4. 构造它的递归下降分析程序。

答：

1.计算这个文法的每个非终结符的FIRST集和FOLLOW集。

FIRST(P) = {a,b,(,^}
FIRST(F) = FIRST(P) = {a,b,(,^}
FIRST(T) = FIRST(F) = {a,b,(,^}
FIRST(E) = FIRST(T) = {a,b,(,^}
FIRST(E') = {+,ε}
FIRST(T') = FIRST(T) ∪ {ε} = {a,b,(,^,ε}
FIRST(F') = {*,ε}

FOLLOW(E) = {#,)}
FOLLOW(E') = FOLLOW(E) = {#,)}
FOLLOW(T) = (FIRST(E') - {ε}) ∪ FOLLOW(E) = {+,#,)}
FOLLOW(T') = FOLLOW(T) = {+,#,)}
FOLLOW(F) = (FIRST(T') - {ε}) ∪ FOLLOW(T) = {a,b,(,#,^,+,)}
FOLLOW(F') = FOLLOW(F) = {a,b,(,#,^,+,)}
FOLLOW(P) = (FIRST(F') - {ε}) ∪ FOLLOW(F) = {*,a,b,(,#,^,+,)}


| |FIRST|FOLLOW|
|:---|:---|:---|
|E	|{a,b,(,^}	|{#,)}|
|E'	|{+,ε}	|{#,)}|
|T	|{a,b,(,^}	|{+,#,)}|
|T'	|{a,b,(,^,ε}|{+,#,)}|
|F	|{a,b,(,^}|{a,b,(,#,^,+,)}|
|F'	|{*,ε}	|{a,b,(,#,^,+,)}|
|P	|{a,b,(,^}	|{*,a,b,(,#,^,+,)}|

2.证明这个文法是LL(1)的。

考虑下列产生式：
E‘→+E|ε
T‘→T|ε
F'→*F'|ε
P→(E)|a|b|^

FIRST(+E) ∩ FIRST(ε) = {+} ∩ {ε} = ∅
FIRST(T) ∩ FIRST(ε) = {(,a,b,^} ∩ {ε} = ∅
FIRST(*F') ∩ FIRST(ε) = {*} ∩ {ε} = ∅
FIRST((E)) ∩ FIRST(a) ∩ FIRST(b) ∩ FIRST(^) = ∅

FIRST(E') ∩ FOLLOW(E') = {+,ε} ∩ {#,)} = ∅
FIRST(T') ∩ FOLLOW(T') = {(,a,b,^,ε} ∩ {+,#,)} = ∅
FIRST(F') ∩ FOLLOW(F') = {*,ε} ∩ {(,A,B,^,+,),#} = ∅

所以，该文法是LL(1)文法。
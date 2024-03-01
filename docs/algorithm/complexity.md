---
title: Algorithm Complexity
tags: [p, np, npc, ntm]
sidebar_label: Algorithm Complexity
sidebar_position: 2
---

# DTM, NTM

## 图灵机

图灵机是一种抽象的机器，也是计算机的理论模型，由英国数学家艾伦·图灵于1936年提出。它基于人们使用纸笔进行数学运算的过程，将虚拟的机器替代人们进行数学运算。

在图灵机中，有一条无限长的纸带，纸带分成了一个一个的小方格，每个方格有不同的颜色。有一个机器头在纸带上移来移去。机器头有一组内部状态，还有一些固定的程序。在每个时刻，机器头都要从当前纸带上读入一个方格信息，然后结合自己的内部状态查找程序表，根据程序输出信息到纸带方格上，并转换自己的内部状态。

虽然图灵机看起来很简单，但它可以模拟计算机的任何算法，无论这个算法有多复杂。因此，图灵机是计算机科学中非常重要的概念，也是理解计算机的工作原理和算法设计的基础之一。

## 确定性图灵机（DTM）

在确定性图灵机 ([Deterministic Turing Machine](https://en.wikipedia.org/wiki/Turing_machine)) 中，一组规则规定针对任何给定情况最多执行一个操作。

确定性图灵机具有一个转换函数，对于磁带头下的给定状态和符号，指定三件事：
1. 要写入磁带的符号（它可能与当前位于该位置的符号相同，或者根本不写入，导致没有实际变化），
2. 头部应移动的方向（左、右或两者都不移动），以及
3. 有限控制的后续状态。

## 非确定性图灵机（NTM）

在理论计算机科学中，非确定性图灵机 ([Nondeterministic Turing Machine](https://en.wikipedia.org/wiki/Nondeterministic_Turing_machine)) 是一种计算理论模型，其控制规则指定在某些给定情况下多个可能的操作。 也就是说，与确定性图灵机不同，NTM 的下一个状态并不完全由它的动作和它看到的当前符号决定。

NTM 有时用于思想实验，以检查计算机的能力和局限性。 理论计算机科学中最重要的开放问题之一是 P 与 NP 问题，该问题（以及其他等效公式）涉及用确定性计算机模拟非确定性计算有多困难的问题。

## DTM vs NTM

![](../../static/images/algorithm/Difference_between_deterministic_and_Nondeterministic.svg.png)


# P, NP, NP-complete, NP-hard Problem

## P问题

P问题是指可以在多项式时间内解决或验证的问题。具体来说，P问题指的是可以在多项式时间复杂度内解决判定性问题或可以在多项式时间内验证一个解的问题。这类问题通常可以归结为求解一些数学问题，比如排序、找最短路径等。

> In computational complexity theory, P, also known as PTIME or DTIME(nO(1)), is a fundamental complexity class. It contains all decision problems that can be solved by a deterministic Turing machine using a polynomial amount of computation time, or polynomial time.

## NP问题

与P问题相对的是NP问题，这类问题很难找到多项式时间的算法（或许根本不存在），但是可以在多项式时间内验证一个解的问题。

> NP is the set of decision problems for which the problem instances, where the answer is "yes", have proofs verifiable in polynomial time by a deterministic Turing machine, or alternatively the set of problems that can be solved in polynomial time by a nondeterministic Turing machine.
> * NP is the set of decision problems solvable in polynomial time by a nondeterministic Turing machine.
> * NP is the set of decision problems verifiable in polynomial time by a deterministic Turing machine.

很容易看出，复杂度类 P（所有在多项式时间内可确定性解决的问题）包含在 NP（可以在多项式时间内验证解决方案的问题）中，因为如果一个问题在多项式时间内可解决，则解决方案 通过简单地解决问题，也可以在多项式时间内验证。

最重要的 P 与 NP（“**P = NP？**”）问题，询问是否存在用于解决 NP 完全问题以及由此推论的所有 NP 问题的多项式时间算法。 人们普遍认为事实并非如此。

## NP-complete 和 NP-hard 问题

> A decision problem C is NP-complete, if:
> 1. C is in NP, and 
> 2. Everty problem in NP is reducible to C in polynomial time.
>
> C can be shown to be in NP by demonstrating that a candidate solution to C can be verified in polynomial time.
> Note that a problem satisfying condition 2 is said to be NP-hard, whether or not it satisfies condition 1.
> A consequence of this definition is that if we had a polynomial time algorithm (on a UTM, or any other Turing-equivalent abstract machine) for C, we could solve all problems in NP in polynomial time.

## P versus NP 问题

[P versus NP problem](https://en.wikipedia.org/wiki/P_versus_NP_problem)

![](../../static/images/algorithm/P_np_np-complete_np-hard.svg.png)

# 参考资料

* [Wikipedia - Turing Maching](https://en.wikipedia.org/wiki/Turing_machine)
* [Wikipedia - Nondeterministic Turing Machine](https://en.wikipedia.org/wiki/Nondeterministic_Turing_machine)
* [Wikipedia - P](https://en.wikipedia.org/wiki/P_(complexity))
* [Wikipedia - NP](https://en.wikipedia.org/wiki/NP_(complexity))
* [Wikipedia - NP-completeness](https://en.wikipedia.org/wiki/NP-completeness)
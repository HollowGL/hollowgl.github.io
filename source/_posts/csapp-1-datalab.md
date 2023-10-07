---
title: csapp实验一datalab
toc: false
date: 2023-09-29 22:33:03
updated: 2023-09-29 22:33:03
cover:
thumbnail:
categories: csapp
tags: C
---

{% raw %}
<style type="text/css">
    .heimu { color: #888f96; background-color: #888f96; }
    .heimu:hover { color: #fff; }
</style>
{% endraw %}


一些感想，留作纪念。~~后续可能会给出分析~~
<!-- more -->

花了数个小时完成实验一，整体难度确实很高，但做完收获很大。一些题没有思路，参考了网上解法。我主要参考了[这里的解法](https://zhuanlan.zhihu.com/p/339047608)，因为它还提供了用docker搭建环境的教程。我的解法[传送门](https://github.com/HollowGL/CSAPP-Labs/blob/main/1_datalab/bits.c)，我用蹩脚的英文在部分题里写了解题思路。

## 整体感受

题目分为int和float两部分。

int部分限制了很多操作符和关键字的使用，要求使用最基础的位运算符和一元运算法，迫使我们以二进制的视角去观察遇到的每一个整型数字，同时实现一些简单的逻辑结构。题目还有操作数个数限制，不过大多数题目这个规则都很容易遵守，但在 `logicalNeg` 这个题中我的第一个解法刚好超过了限制，还好我知道 `x ^ (x - 1)` 这个神奇的操作，并用它优雅地完成题解。`howManyBits` 这个题压轴出场，是最难的一个，我在这个题上可能花了两个小时。

float部分虽然放开了各种操作数和关键字的限制，但是它使用 unsigned int 传参，所以我们还是要将其看作是二进制序列。第一题 `floatScale2` 就让我理解到浮点数的魅力：我一开始将非规范化数按小数字段第一位是否为0分作两类分开讨论，结果后来发现他俩之间可以“**无缝衔接**”！第二题 `floatFloat2Int` 感觉我的解法很丑陋，一点也不优雅，完全是在面向test解题。{% raw %}<span class="heimu">虽然其它题也差不多是面向test求解</span>{% endraw %}

## 后续

目前还有更重要的事情要做，csapp的探索可能要搁置一阵子了。

<a class="tag is-medium" href="https://www.imaegoo.com/2020/icarus-with-bulma/" target="_blank">
<span class="icon"><i class="fa fa-bookmark"></i></span>&nbsp;&nbsp;
鼠标悬浮显字技巧来源：@iMaeGoo
</a>


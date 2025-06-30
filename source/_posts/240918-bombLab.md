---
title: CSAPP实验二Bomb
toc: true
excerpt: 传奇耐炸王
date: 2024-09-18 21:00:00
updated: 2024-09-18 21:00:00
cover:
thumbnail:
categories: csapp
tags: C
---


<!-- more -->

有点晦涩的汇编代码，看着十分抽象。大量参考了网络上的详解，仍一知半解，只有`phase5`是独立完成的。

没想到会在学会gdb调试C程序之前，先学会用gdb调试汇编。列出本次实验中常用的gdb命令如下：

```shell
objdump -d <exec-file> > <filename.s>
```
将C语言编译生成的二进制文件反汇编为汇编代码。之后可以在编辑器中查看汇编代码，不过主要还是在命令行窗口查看实时反汇编代码，这两者的格式不完全一样。

比如`phase_1`函数汇编代码如下：
```asm
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  400ee9:	e8 4a 04 00 00       	call   401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	call   40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	ret    
```
其中的跳转命令`je`目标是绝对地址，可读性很差。而在gdb页面看到的汇编代码如下：
```asm
0x400ee0 <phase_1>      sub    $0x8,%rsp 
0x400ee4 <phase_1+4>    mov    $0x402400,%esi 
0x400ee9 <phase_1+9>    call   0x401338 <strings not equal> 
0x400eee <phase_1+14>   test   %eax, %eax
0x400ef0 <phase_1+16>   je     0x400ef7 <phase_1+23> 
0x400ef2 <phase_1+18>   call   0x40143a <explode_bomb> 
0x400ef7 <phase_1+23>   add    $0x8,%rsp 
0x400efb <phase_1+27>   ret 
```



```shell
gdb <exec-file>
```
进入gdb调试页面，之后的命令均为gdb命令。

```gdb
layout asm
```
切换到汇编指令模式，可以实时查看汇编代码、当前运行位置和断点位置。

```gdb
refresh
```
页面如果字符重叠，用该命令刷新。

```gdb
disas <func_name>
```
查看指定函数的汇编代码。


```gdb
b <func_name>
b * <address>
```
给指定函数或者指定位置打断点。

```gdb
r
```
开始运行，直到第一个断点位置。

```gdb
ni
si
c
```
单步执行、步入、继续执行。


```gdb
x /s <address>
```
打印指定地址存放的字符串

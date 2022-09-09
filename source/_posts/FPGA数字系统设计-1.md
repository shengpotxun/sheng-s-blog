---
title: FPGA数字系统设计(1)
tags: FPGA
categories: 课堂笔记
cover: 'https://tva1.sinaimg.cn/large/008tudqVgy1h5wnd2yiaaj33dj18g4qy.jpg'
abbrlink: c51af522
date: 2022-09-06 09:45:15
---

# 课程资料

《基于VHDL的数字系统设计方法》

EDA：电子计算机设计辅助工具

# 课程概述

FPGA：现场的可编程的门阵列

什么是数字系统：包括计算机系统的一个完整智能系统

数字系统设计途径：

- 用固化的MCU、MPU构建(芯片本身是固定好了)
- FPGA为中心构建(可以编辑电路，更加方便快捷)
- 基于PSoc芯片构建系统
- 芯片自我设计再设计系统

EPROM：可擦除的可编程的记忆存储器，这是FPGA理念的始祖

PSoc：可编程的CPU固化内嵌

# 电子电路内容回顾

非门：一个PMOS,一个NMOS

A：NMOS，PMOS,CMOS，

时序逻辑：

组合逻辑：

竞争：

冒险：

# 第一课：数字抽象、组合逻辑、Verilog

工作电压

例子： 5v(高电压2.1～5，低电压0～0.7)

《模拟电路基础》（童诗白，华成黄）
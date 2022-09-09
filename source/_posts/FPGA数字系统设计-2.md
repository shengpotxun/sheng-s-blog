---
title: FPGA数字系统设计(2)
abbrlink: ee37a6e1
date: 2022-09-07 14:24:11
tags: FPGA
katex: true
categories: 课堂笔记
cover: 'https://tva1.sinaimg.cn/large/008tudqVgy1h5wnd2yiaaj33dj18g4qy.jpg'
---

# 模拟电路计量分析方法

- 微变等效电路
- 图形分析法
  - 输入特性曲线
  - 输出特性曲线

## 实例

恒温调节器：

通过设计数字电路实现的。通过温度传感器测到当前温度，与预设温度相比，如果大就调低如果小就调高

关键是反馈

$$放大=\begin{cases}
    基本放大\\
    正/负反馈\\
    功率放大\\
    差分放大\\
    运算放大
\end{cases}$$

运算放大电路的输入都是差分放大电路

# 描述函数的方法

- 硬件描述语言VHDL
- 真值表
- 卡罗图
- 电路原理图


---
title: FPGA数字系统设计(3)
tags: FPGA
categories: 课堂笔记
cover: 'https://tva1.sinaimg.cn/large/008tudqVgy1h5wnd2yiaaj33dj18g4qy.jpg'
abbrlink: f72c97a0
katex: true
date: 2022-09-13 10:01:08
---

# 老师的唠嗑

linux是好东西

**IDE：集成开发平台**

altera被intel收购了，说明FPGA前途光明，它的开发平台是Quartus

xilinx全球FPGA领头的，Vivado开发平台

**EDA：电子开发辅助工具**

Cadence：做模拟和数字

SynopSys：集成电路供应商，做数字芯片设计

{% note info %}

优化两大方向：

- 程序的优化，汇编指令的条数越少越好
- 硬件的优化，希望并行和同步的进行

{% endnote %}

# 冒险

关键是输入和输出之间的关系不能有逻辑错误，如果有错误的情况就称为**冒险**，这很关键

电路有时间的延迟$\Delta t$,在时间延迟中发生交叉的跳变就会出现逻辑问题，虽然只有很短的时间，但这个结果也是很严重的。

## 解决方案

# 

在电路图设计的时候要考虑门电路的实际效率，比如**与非门**大于**与门**

# 硬件描述语言

> HDL与程序有什么区别?

程序是PC上线性执行的，而HDL是并发的是对硬件关系的描述，针对物理体(仿真和触发)

## 触发与描述情况

### case

[链接指向——电平触发与边沿触发](https://blog.csdn.net/Gdadiao123/article/details/80958165)

```
always @(in) begin
    case(in)
    endcase
end

```

边沿触发，而不是电平触发，边沿触发是要求必须系统必须同时动作，电平有时间误差

边沿触发的代价：

两倍电路加非门，为了同步

![](http://www.elecfans.com/uploads/allimg/171114/2755783-1G11411404Q00.png)
> D触发器需要两个D锁存器，但是电平触发不需要

### casex

逻辑状态：0、1、z、x

```
casex(in)
endcasex
```
这是表示不确定的内容，不确定的情况，一般穷举情况

### 不用寄存器

```
wire isprime = (in[0] & in[3]) |
               (in[1] & in[4]) |

```

## 综合报告

- 要看时间
- 要看路径
- 要看占用资源
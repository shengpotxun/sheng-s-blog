---
title: Hexo主题Aurora设置数学输入
tags: mathjax
categories: Hexo主题配置
cover: 'http://tva1.sinaimg.cn/large/008tudqVgy1h54vrer3rvj32au1duu10.jpg'
mathjax: true
katex: true
abbrlink: aa70025d
date: 2022-08-13 08:55:18
---
# 起因

想在自己的博客中输入数学公式，但是发现不能通过markdown键入，查资料发现是无法渲染数学公式，这里给出解决方案。

# 解决方案

主题根目录下终端执行 `npm install hexo-filter-mathjax`

接着在根目录下 **_config.yml** 中加入一些设置语言

```
mathjax:
  tags: none               # or 'ams' or 'all'
  single_dollars: true     # enable single dollar signs as in-line math delimiters
  cjk_width: 0.9           # relative CJK char width
  normal_width: 0.6        # relative normal (monospace) width
  append_css: true         # add CSS to every page
  every_page: false        # if true, every page will be rendered by mathjax regardless the `mathjax` setting in Front-matter of each article
```

分别是：

- 标签(不用管，默认)
- 是否开启$符号作为行内公式
- 相对CJK字符宽度
- 相对正常字符宽度
- 是否将CSS添加到每个界面
- 是否为每一页都加入mathjax渲染数学公式(默认关闭，毕竟有很多没写数学公式的博客)

至此结束，下面是示例

# 示例

$$
\frac{1} {2}
$$

$$
\int \_ {1} ^ {\infty}f(x)
$$

$$
f(x)=
    \begin{cases}
        y=x+3,& \text{if x>0} \\\\
        y=-x-1,& \text{if x <= {0}}
    \end{cases}
$$

今天吃了$x=e^{\frac{3^y_1-2} {y_{1}+a}}$

## 还是有问题的

有一部分的数学公式无法输入包括，二重闭合曲面积分

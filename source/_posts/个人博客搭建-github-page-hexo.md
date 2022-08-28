---
title: 个人博客搭建(github page+hexo)
tags: 前端
categories: 前端
cover: 'http://tva1.sinaimg.cn/large/008tudqVgy1h5273spschj32pg1wwqv7.jpg'
feature: true
abbrlink: 951898c4
date: 2022-08-10 22:15:49
---
# 引入

为什么想要搭建个人博客呢？主要原因是不想使用云端笔记了，从我开始写笔记以来已经使用过了word+百度网盘、onenote、享做笔记、simplenote、joplin等云端笔记了，大抵是都不太满意，后来接触到了github，这个时候就想通过github同步笔记，但是转念一想我都用github了那干脆开一个个人博客吧，而最终的结果就是现在看见的**Sheng's Blog**

# 基本环境

由于需要我搭建这个博客的时候正在学习使用linux系统，于是搭建的过程都是在linux上完成的。当然，用Windows的同志们不是说不能看，流程和指令在Windows终端上也是类似的。你需要：

- manjaro-gnome 21.3.6
- 熟悉 git
- 一些linux基本终端操作

有人会说，会用linux的不都是大神吗？朋友，总有刚开始的菜鸟的，比如说我(摊手)

# 基础理论

全都是个人理解

- **网站**：网站由域名、服务器和网页程序三部分组成。
- **域名**：域名是相当于地址的存在，我们在浏览器中输入的网页地址就是域名。
- **服务器**：服务器用来存放网站程序，数据库等，相当于承载房子的土地结构。
- **网页程序**：相当于网页的主体部分，承载网页的功能实现，相当于我们在我们凭租的土地上建房子，让这个房子发挥作用。个人建设网页可以使用成熟的开源网页程序，类似于wordpress

但是我们今天用的是**github page+hexo**，这种网页是没有前后端交互的，换言之就是静态的网页，你只能看，网站本身不会和你有交流，是简单而适合于建设个人博客的一种方法。如果说动态网页是真人主播可以互动的话，静态就是不能动的纸片人。所以我们不需要一个额外的服务器，只要把静态的网页内容(纸片人)上传到github page上(海报墙)就行了。hexo其实只是用来管理静态网页的。

# 细节步骤

## 域名购买(选做)

在买房之前，我们一般会到售楼处去选址，同样的，我们先来购买域名。我的网址是在[Godday](https://www.godaddy.com/zh-sg)上买的，上面提供的一般是国外的域名，国外域名加上github page可以让建好的网站不用去报备，当然你也可以选择其他的域名购买网站进行域名购买。

![Screenshot from 2022-08-10 23-05-52.png](http://tva1.sinaimg.cn/mw690/008tudqVgy1h522ouv0wpj31b60eiagu.jpg)

我们在搜索栏中搜索想要的域名，网站就会给出选项和套餐，自己选购就行了，顺便一提Godday支持微信登陆和支付宝支付。

![Screenshot from 2022-08-10 23-14-47.png](http://tva1.sinaimg.cn/mw690/008tudqVgy1h522tirg1bj30oa0hwt9y.jpg)

> 购买完成之后的个人页面

## 申请github page

感谢开源社区github，我们可以使用github page来轻松搭建自己的个人网站，这里注册账号就略过了，主要讲一下是怎么申请到github page的

首先我们去github的仓库新建一个仓库(右上角绿色按钮 `new`)，将仓库的名称设定成 `username.github.io`(这里username必须和自己的github用户名**保持一致**我github用户名叫做shengpotxun，这里就应该写 `shengpotxun.github.io`)这会自动让github确认你是在申请github page，其他默认直接确定创建(最下方绿色按钮)。

完了之后进入设置
![Screenshot from 2022-08-10 23-23-39.png](http://tva1.sinaimg.cn/mw690/008tudqVgy1h5232sh9xfj30h603gdg2.jpg)

> 一排小标志最右边Settings

检查左侧侧边栏有没有一个Pages，有就说明成功了
![Screenshot from 2022-08-10 23-25-57.png](http://tva1.sinaimg.cn/mw690/008tudqVgy1h523532xjoj30c606cjrq.jpg)

## 安装hexo

这个简单，manjaro自带node.js，要是没有就自己搜索一下node.js是怎么安装的，aur提供社区安装包，自己装一下就行，关键在于**npm**，这个用来管理环境和插件的

终端输入 `yay -S npm`可以直接下载。

`npm install -g hexo-cli #通过npm下载hexo`

我查资料的时候，据说国内**npm**可能网不是很好，有能力的可以科学一下，我本身是科学的没遇到这种情况，如果遇到下载不下来，没有能力的国内应该有个镜像叫做**cnpm**，如果你是用**cnpm**，那下面所有的**npm**指令你都改成**cnpm**应该就行了

`hexo init hexoblog #初始化并放在当前文件夹的hexoblog文件夹里面`后面的文件夹名称想写什么就写什么，如果有任何做错了无法挽回的地方，删掉整个hexoblog文件夹从这一步开始做就行了。

这个时候我们已经得到了一个基本完全的静态网页了，换句话说就是房子已经准备好了，纸片人已经画完了，我们给它放到它应该在的地方，也就是github page。进入hexoblog文件夹，我们会发现有一个叫做 `_config.yml`的文件，这就是配置文件，用文本编辑器打开配置文件。翻到最下面会有两行

```
deploy:
  type: 
  repository: 
```

这个部分就是用来填写上传信息的，我们按照自己的信息改掉，改成下面的格式

```
deploy:
  type: git
  repository: https://github.com/username/username.github.io.git
  branch: main
```

username换成你们自己的github用户名，我的就是 `……shengpotxun/shengpotxun.github……`

填完了之后我们就可以进行同步了，安装同步插件，在hexoblog文件夹中打开终端输入`npm install hexo-deployer-git --save`

接下来是三个比较常用的指令

```
hexo g #编译载入
hexo serve #本地加载预览，在浏览器地址中输入http://localhost:4000/进行查看，在终端中按下ctrl+c中止进程
hexo d #编译并向github page同步
```

{% note info %} 

#### github要求你输入密码

我们在同步的时候会被要求输入github账号和密码，这里说一下，由于github安全措施改了，要求你输入密码的地方全部应该输入令牌(token)。展示一下怎么获得令牌。

- 主页右上角个人头像鼠标放上去之后会有一个子菜单，选择子菜单中的**settings**
- 进入设置页面之后选择左侧最下方**Developer settings**
- 选择左侧最下方**Personal access tokens**
- 右上方白色按钮**Generate new token**获取新令牌
- 出来的表单是问你这个令牌权限有多大，不清楚你就全选好了，令牌是有时限，你可以修改为不限时令牌，下方绿色按钮确认
- 令牌只会出现一次就是你刚刚申请的这一次，及时保存令牌否则就再也无法查看令牌了。

{% endnote %}

`hexo d`指令中让我们输入密码，我们应该输入自己的令牌。

## 完成(没有买域名)

打开**username.github.io**(username对应自己的用户名)网址，就可以看见自己的博客了，如果没有出现，就等一会。

## 解析域名

我们要把域名和实际网址对应起来，打开你购买域名的网站。选择管理DNS服务
![Screenshot from 2022-08-11 00-18-31.png](http://tva1.sinaimg.cn/mw690/008tudqVgy1h524nsjy1kj30lg0g7myz.jpg)
向其中添加两条内容，分别是
![Screenshot from 2022-08-11 00-20-12.png](http://tva1.sinaimg.cn/mw690/008tudqVgy1h524shyo2mj31nx05sgmf.jpg)
![Screenshot from 2022-08-11 00-23-52.png](http://tva1.sinaimg.cn/mw690/008tudqVgy1h524tqddqrj31nx05swfe.jpg)

{% note info %}

1小时在一些域名网站上面显示的是3600,问题在于如何获得IP地址

终端输入 `ping username.github.io`(同样的**username**改掉)返回：

```
64 bytes from cdn-185-199-108-153.github.com (185.199.108.153): icmp_seq=1 ttl=46 time=64.8 ms
64 bytes from cdn-185-199-108-153.github.com (185.199.108.153): icmp_seq=2 ttl=46 time=66.3 ms
```

获得IP地址

{% endnote %}

接下来我们在本地的hexo里确认这个域名

进入public子文件夹， `touch CNAME` 创建文件，用文本编辑器修改文件**CNAME**，在文件中加入自己的域名

```
# in CNAME(文件里面没有这一行，一共一行)
shengxun.xyz
```


> 这个文档要经常检查有没有被删，因为每当我们习惯性地用 `hexo clean`清空原来的静态网页的时候会把这个文件删掉。


`hexo d` 进行同步

## 完成(购买了域名)

在浏览器中输入自己的域名可以查看，DNS服务修改后可能要过几个小时才能生效

# 安装博客主题

打开[hexo官方网站](https://hexo.io/themes/)查看主题，点击图标可以预览，点击标题就可以进入github项目介绍，跟着主题的介绍就可以完善自己的博客了。以后有时间我会写出自己博客的主题是怎么搭建的。

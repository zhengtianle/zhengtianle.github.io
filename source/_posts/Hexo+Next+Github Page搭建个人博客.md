---
title: Hexo+Next+Github Page搭建个人博客
date: 2018-11-20 11:12:20
tags: 
- hexo
- next
categories: 
- hexo
- next
---

今天是个值得纪念的日子，由于本人重装系统(逃离ubuntu入了manjaro的坑)，在没有备份hexo+next配置文件的情况下，时隔两个月零十二天再次恢复了本博客的正常更新运行。写下本文记录恢复博客的操作并借此文章再次告诫自己备份的重要性！

<!-- more -->

{% note danger no-icon %}
**本机环境**
- Manjaro 18.0.0 (Arch系Linux)
- git version 2.19.1
- nodejs version 10.13.0
- npm 6.4.1

**注：本文内容理论上适合各种环境下搭建Hexo博客**
{% endnote %}

<a id="head"></a>
### 安装Hexo
Hexo官网文档很清晰：[Hexo官网](https://hexo.io/zh-cn/)
文档中包括了`git`、`nodejs`、`hexo`的安装和部分配置

### 安装Next主题
本人沿用了之前用过的主题Next，此主题简约舒适，更多主题请右拐 [Hexo主题](https://hexo.io/themes/)
同样，Next官方文档也相当详细：[Next官网](https://theme-next.iissnan.com/)

### 配置优化主题
本文重点不在这里，放几个本人参考的链接，如有疑问可以评论区留言
[Next官网-主题配置](https://theme-next.iissnan.com/theme-settings.html)
[Hexo搭建的GitHub博客之优化大全](https://zhuanlan.zhihu.com/p/33616481)
[打造个性超赞博客Hexo+NexT+GitHubPages的超深度优化](https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html)
[添加本文左下角看门猫：hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d)

### 本地博客上传至github page

#### 创建github page
同样直接上官方文档：[github page官网](https://pages.github.com/)
**注意**：只要创建好`yourname.github.io`仓库即可,例如本博客创建仓库名称为`zhengtianle.github.io`

#### 本地同步至github page

1. 在本地站点目录终端执行`npm install hexo-deployer-git --save`
2. 打开`站点配置文件`将deploy修改为如下内容：
```yml
deploy:
  type: git
  repository: https://github.com/yourname/yourname.github.io.git
  branch: master
```
3. 在本地站点目录终端执行`hexo d -g`
命令解析请右拐 [Hexo指令](https://hexo.io/zh-cn/docs/commands.html)
4. 愉快的在浏览器地址栏中输入`yourname.github.io`就可以看到同步至github page的博客了

#### 备份Hexo配置和源文件
{% note warning no-icon %}
这时候可能就有人疑问了，我们不是已经把本地博客同步至github了么 🤨
好吧，那你打开你创建的`yourname.github.io`个人仓库，`master`分支下文件是不是和你`本地站点`文件夹完全不一样 😎
因为github里面上传的是hexo生成的`静态文件`,也就是`本地站点`里面的`public`文件夹
这样就会有一个问题，如果我们换了另一台电脑或者重装了系统，还怎么继续更新维护博客啊
这时候我们就要再对`本地站点`的`source`和`配置文件`进行备份
**本文的做法是在`yourname.github.io`仓库里另起一个分支保存hexo生成的其他文件**
{% endnote %}

*在本地站点目录终端执行如下操作：*
1. 初始化repository：`git init`
**提示**：`Hexo`自带了`.gitignore`文件，需要忽略的文件都已经默认配置好了
2. 添加工作区所有文件到暂存区：`git add .`
3. 将暂存区的文件提交到本地仓库：`git commit -m "提交注释"`
4. 把本地仓库映射到github：`git remote add origin https://github.com/yourname/yourname.github.io.git`
5. 创建新分支`hexo`：`git branch hexo`
6. 把本地仓库全部push到hexo分支上：`git push -u origin hexo`

现在再查看github里的`github.yourname.io`仓库，会有`master`和`hexo`分支分别保存`静态文件`和`hexo其他文件`

### 新环境继续维护博客
1. clone：`git clone https://github.com/yourname/yourname.github.io.git`
2. 切换分支：`git checkout hexo`(默认分支是master)
3. 其他步骤同 [安装Hexo](#head)
**注意：不用执行hexo init进行初始化！！！**


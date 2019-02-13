---
title: Hexo+Next+Github Pages搭建个人博客
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

### 安装Hexo<span id="jump"></span>
Hexo官网文档很清晰：[Hexo官网](https://hexo.io/zh-cn/)
文档中包括了`git`、`nodejs`、`hexo`的安装和部分配置

### 安装Next主题
本人沿用了之前用过的主题Next，此主题简约舒适，更多主题请右拐 [Hexo主题](https://hexo.io/themes/)
同样，Next官方文档也相当详细：[Next官网](https://theme-next.iissnan.com/)

### 配置优化主题
本文重点不在这里，放几个本人参考的链接，如有疑问可以评论区留言
[Next官网-主题配置](https://theme-next.iissnan.com/theme-settings.html)
[Hexo搭建的Github博客之优化大全](https://zhuanlan.zhihu.com/p/33616481)
[打造个性超赞博客Hexo+NexT+GitHubPages的超深度优化](https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html)
[添加本博客左下角看门猫：hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d/)

### 本地博客上传至 GitHub Pages

#### 创建 GitHub Pages
同样直接上官方文档：[GitHub Pages 官网](https://pages.github.com/)
**注意：**只要创建好`yourname.github.io`仓库即可，例如本博客创建的仓库名称为`zhengtianle.github.io`

#### 本地同步至 GitHub Pages
1. 在本地站点目录终端执行`npm install hexo-deployer-git --save`
2. 打开`站点配置文件`将`deploy`部分修改为如下内容：
```yml
deploy:
  type: git
  repository: https://github.com/zhengtianle/zhengtianle.github.io.git
  branch: master

```
3. 在本地站点目录终端执行`hexo d -g` (命令解析请右拐[Hexo指令](https://hexo.io/zh-cn/docs/commands))
4. 愉快的在浏览器地址栏中输入`yourname.github.io`就可以看到同步至Github Pages的博客了

#### 备份Hexo配置和源文件
{% note warning no-icon %}
看到这部分可能就会有人疑问了：我不是已经把本地博客上传到github上了么？🤨
好吧，那你打开创建的`yourname.github.io`个人仓库，`master`分支下的文件是不是和你的`本地站点`文件夹完全不一样？🤣
因为github里面上传的是hexo生成的`静态文件`，也就是`本地站点`里面的`public`文件夹
如果仅仅做到使用`hexo d -g`将本地博客同步至Github这一步，那么显然就会有一个问题：如果我们换了另一台电脑或者重装了系统(本人血与泪的教训！😭)，怎么继续更新维护博客？
**本文介绍的解决办法：在`yourname.github.io`仓库里另起一个分支保存hexo生成的其他文件**
{% endnote %}

{% note danger no-icon %}
**执行下述操作时会有一个不容易被发现的坑(又是血与泪的教训！😭)**：
如果你下载next主题是使用`git clone https://github.com/iissnan/hexo-theme-next themes/next`这种直接从仓库克隆到本地的方法，那么克隆下来的文件夹内会有`.git`文件，此文件绑定的是next官网github的地址，不额外处理会使得next文件夹下的所有文件不会上传至你的github仓库!解决办法有两个：
1. 简单粗暴直接删除克隆自next仓库的`.git`文件：
`cd 你的next文件夹目录下`
`rm -rf .git`
`cd ..`(返回到上一级themes目录下)
`git rm -r –cached /next`
2. [issues-远程仓库无法备份theme/next主题的问题](https://github.com/iissnan/hexo-theme-next/issues/932)
{% endnote %}

在`本地站点目录`终端执行如下操作：
1. 初始化repository：`git init`
**提示**：`Hexo`自己已经默认生成了`.gitignore`文件，需要忽略的文件已经默认配置好了，从这里也可以看出，Hexo作者应该也是想让使用者将其他文件上传至云端备份
2. 添加工作区所有文件到暂存区：`git add .`
3. 暂存区的文件提交到本地仓库：`git commit -m "提交注释"`
4. 本地仓库映射到github：
`git remote add origin https//github.com/yourname/yourname.github.io.git`
5. 创建并切换到新分支hexo：`git checkout -b hexo`
6. push本地仓库到hexo分支上：`git push -u origin hexo`

现在再次查看github里的`yourname.github.io`仓库，会有`master`和`hexo`两个分支分别保存`静态文件`和`hexo生成的其他文件`(hexo分支下的文件和本地站点目录下的文件是不是基本一致？🤓问题解决了！)

### 新环境如何继续维护博客
1. clone：`git clone https//github.com/yourname/yourname.github.io.git`
2. 切换分支：`git checkout hexo`(默认分支是master)
3. 其他步骤同[安装Hexo](#jump)
**注意：不用执行 hexo init 进行初始化！！！**

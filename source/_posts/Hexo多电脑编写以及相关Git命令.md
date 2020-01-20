---
title: Hexo多电脑编写以及相关Git命令
date: 2020-01-20 15:40:55
tags: ["Hexo", "Git"]
categories: ["个人"]
---

### 场景
现实情况下我们会存在在公司一台电脑, 在家一台电脑的情况. 使用Hexo编写文章, 并将编译过后的页面放在Github库中可以直接访问很方便, 但是由于Hexo的配置等源文件都在电脑本地存放, 所以当想在另一台电脑进行编写文章就无法实现, 在网上查询了一些资料, 挑选了将Hexo源文件也提交到Github库的方法, 并借此学习了一波Git的操作.

### 环境准备
Git: [https://git-scm.com/download/win](https://git-scm.com/download/win)

Node: [http://nodejs.cn/download/](http://nodejs.cn/download/)

### Github仓库准备

- 创建分支

正常情况下, **Github** 库有默认的分支 **master** 这个分支是用来存放 **hexo** 编写文章编译成的页面的. 我们要将 **hexo** 的源文件上传到同一个库中, 所以要先创建一个分支, 此分支用 **source** 命名, 如图, 我之前已经创建过 **scource** 分支

![](1.png)

- 默认分支

**master** 分支是仓库默认分支, 我们要操作的是源文件, 所以需要将默认分支修改为 **source**

![](2.png)

### 准备带有Hexo环境的电脑

- 将source分支克隆到本地

首先复制 **Github** 库项目路径

![](3.png)

然后在本地创建一个合适文件夹作为存放源文件的路径, 在此文件夹下右键点击 **Git base here** 打开 **Git** 命令行

![](4.png)

复制好需要克隆的项目地址, 用如下命令克隆到本地

	git clone https://github.com/xxx // 将项目路径改为你的项目路径

- 复制源文件到分支文件夹下

将 **source** 分支的本地文件夹下的所有文件删除只保留 **.Git** 然后将你平时用以写文章的源文件除了 **.Git** 都复制到 **source** 分支下

![](5.png)

<font style="color: red;">注: 如果你的 themes 主题使用的 Github 上的, 文件夹内的 .Git需要删除, git提交不能覆盖提交</font>

- 分支提交

复制好之后, 将分支进行提交

	git add . // 将整个文件夹下代码加入缓冲区
	git commit -m "注释" // 代码添加注释
	git push // 项目提交

查看 **Github** 仓库代码是否已提交完成

![](6.png)

到此, 此电脑操作完毕

### 新电脑拉取分支

- 将source分支克隆到本地

此处操作和上面一样

- 安装依赖 运行测试

将项目克隆下来之后需要安装依赖, 并运行代码进行测试, 看是否存在问题

	npm install // 安装依赖包

安装依赖包可能会存在报错, 我这可能是因为之前编译的npm版本不一样, 所以使用上面命令执行结束后让我使用命令 `npm audit fix`, 执行完就可以了

	// 运行hexo代码
	hexo clean
	hexo g
	hexo s // 本地运行

没有报错并且浏览器访问正常, 测试完成.

### 修改提交代码

修改提交代码与第一次提交项目大致命令相同, 但是有一点需要注意, 如果你只提交部分代码 `git add .` 需要改为 `git add *`, 不然无法提交

### git常用命令

	git init // 初始化本地项目
	git clone xxx // xxx为项目地址  此操作将库中项目克隆到本地
	git add . 
	git commit -m "注释"
	git push // 提交代码到仓库
	git pull // 更新代码到本地

	git branch -a  // 显示仓库所有分支
	git checkout 分支名 // 在本地切换当前分支

	git status // 查看当前项目代码状态
	git add * // 添加当前路径代码




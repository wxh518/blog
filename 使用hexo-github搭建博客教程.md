---
title: 使用hexo+github搭建博客教程
date: 2023-12-23 22:39:42
tags: 教程
---

### 前置下载

  Node环境下载、npm命令全局安装hexo脚手架

  ```c++
  npm install -g hexo-cli
  ```

### 本地博客搭建
- 创建自己的博客

  ```c++
  hexo init blog
  ```

  脚手架会帮你创建名为blog的文件夹，里面有一系列的文件，先本地跑起来再说。blog文件夹下目录长这个样子😊
  {% asset_img hexo_default_generate.png Hexo创建博客产物 %}

- 本地run起来
  ```c++
  hexo server 本地查看
  在浏览器中输入http://localhost:4000/，回车，博客网站就跑起来啦👏
  ```
  {% asset_img hexo_localhost.png 这是本地Hexo搭建好的样子 %}

### 将博客部署到Github上

  1. 首先需要注册一个GitHub账户
  2. 新建一个仓库，名字为自己账户名字+.github.io，比如zhangsan.github.io
  3. 在仓库的Settings标签页下面，选择网站使用哪个分支的产物，最好改为master主分支
  4. 修改目录下的_config.yml中的deploy字段，映射到上面GitHub中的设置。
    {% asset_img hexo_deploy.png 这是配置Github仓库的图片 %}
- 最后三条命令一套带走，每次发布最新文章都是这一套操作哦

  ```c++
  先安装一个依赖用于hexo deploy命令：npm install hexo-deployer-git --save
  hexo clean 清理上一次编译结果
  hexo generate 编译生成
  hexo deploy 推到远程
  ```

### 更换主题

  hexo有很多主题可供更换，我以keep主题为例来记录下操作流程，其他主题应该类似
- 安装主题
  ```c++
  npm install hexo-theme-keep
  ```
- 修改根目录下的_config.yml，将theme配置字段修改为keep
  ```c++
  theme: keep
  ```
- 设置好主题之后，对该主题生效的配置就都在对应主题目录下的_config.yml中生效了，比如修改主页标题、作者等信息。

### 图片问题
  目前发现两种可以使用图片的方法，并且在浏览器中可以正常展示
- keep主题自带方法
  ```c++
  // 注意需要将图片放到主题下面的images文件夹下才可使用这种方法
  ![](/images/back.jpeg)
  ```
- 安装hexo-assert-image依赖方法
  ```c++
  // 下载依赖
  npm install hexo-assert-image --save
  // 使用格式
  {% asset_img back.jpeg 这是一张图片 %}
  ```
  这个方法需要改下hexo的配置，需要去外层_config.yml中将post_asset_folder字段改为true，这时当我们用hexo new 'xxx'文章时，hexo会同时生成xxx文件夹和xxx.md文件，这时在xxx.md中使用上述格式使用的文件必须放在xxx文件夹下才可正常使用。

### 文章保存问题（建议）
  由于我们写的每一个md文件是一篇文章，并且这些md文件保存在source/_posts文件夹下，所以这个文件夹就显得特别重要，所以建议还是建一个git仓库，只存储md文件，这样就不用担心作业丢的问题啦🤣

  {% asset_img back.jpeg 这是一张图片 %}

---
title: 搭建Hexo博客时常见问题与踩坑
date: 2025-08-02 14:31:46
tags: [Hexo,博客搭建,踩坑记录]
---
这里主要是介绍Hexo基本部署方式，然后是我在搭建Hexo个人博客时遇见的问题和踩过的坑，希望对你有帮助。
<!-- more -->
## 背景知识：

### Hexo常见的两种部署方式：

- 使用Github Actions自动化部署
- 使用 hexo d 命令（hexo-deployer-git）

【public 目录是 **Hexo 生成的静态文件输出目录**，里面存放了所有网站最终要发布到网上的 HTML、CSS、JS、图片等静态资源。】

#### 使用Github Actions自动化部署（优点：仓库结构清晰、自动化强）

**推送前的步骤：**

1. 在Hexo根目录中执行Hexo clean（清除原本的Hexo构建时所生成的public静态文件目录）。
2. 接着执行Hexo generate（可简化为Hexo g）；目的：Hexo 会把源码、主题、文章等内容编译成静态网页，全部输出到 public 目录。
3. 非必需：执行Hexo s来在本地尝试运行，若成功显示则说明构建成功。

**部署时的步骤（不用管，自动部署的）**：Actions 会在云端运行 Hexo，生成 public 目录，然后把这个 public 目录中的所有内容推送到 GitHub Pages 用的分支（一般是 gh-pages 或 main），这样网页内容才会被 GitHub Pages 正确托管。

每次只需将源码即与仓库同名的文件推送到仓库即可自动部署【使用Git推送整个目录】此时并不会将你的public目录推送，因为是由github actions来帮你部署public目录。因此使用这种方式更加安全和规范

注意：必须保证主题和依赖完整提交，尤其不要用子模块，GitHub Actions 环境需要能拉取完整代码。

#### 使用 hexo d 命令（hexo-deployer-git）（优点：操作简单，适合小白）

**推送前的步骤**与1.1.1中相同，接着只需输入：Hexo d 即可。

public/ 目录的构建产物会被推送到远程分支，仓库中会有大量静态文件，仓库会增大。

### 一些常见问题

有了1.1的背景知识，因此一些问题就会衍生出来，这些都是我实际操作时所遇见的：

#### hexo d和git推送不能同时

由1.1的背景知识，两种部署方式是完全不同的，所以不要混用。（当时混用之后我排了两天的错）

#### Hexo主题相关问题

##### 部分主题文件不被git管理，因此需要在推送时手动勾选。（我用的是Tortoise Git）

##### 将文件名改为和根目录config.yml中theme后面相同的名字，否则无法找到对应主题。

在 Hexo 的主配置文件 _config.yml 中，有这么一行：theme: yilia  这句话告诉 Hexo到 themes/yilia/ 这个路径中去找主题文件。因此必须和文件夹名保持一致

##### 在选完主题后最好是下载zip压缩包再解压，否则会有git子模块管理问题。

#### 图片显示问题

首先我用的是Typora文本编辑器，在一开始我将图片加进去之后，在本地运行都无法显示。但后来调试，我发现是以下几个问题：

1. Typora自带的会将图片生成放在.assets同目录文件夹中，而hexo是会去同名文件夹寻找，因此还需要将图片放在同名文件夹中。（不用自己创建，使用hexo new post创建新的post后就会自动生成该文件夹）
2. post中的文件路径不要写绝对路径，可写相对路径，类似于(./图片名.jpg)

【原理：Hexo 会把 source/ 下的文件复制到生成后的 public/ 目录。

当你写 ./haidong.jpg 时，如果页面生成后 HTML 文件和 haidong.jpg 恰好在同一目录，浏览器就能找到。

所以在 hexo s 启动的本地预览环境中，有时看起来是“能显示”的。】



**优化：推荐用 /images/图片名.jpg**

**为什么：**Hexo 在生成静态网站时，会把 source/ 下面的所有文件和文件夹（除了 _posts/、_data/ 这些特殊目录）原样复制到 public/ 目录。

因此在markdown中写：[海东] (/images/haidong.jpg)：无论本地运行还是部署到服务器，都会从网站根目录 /images/ 下找到 haidong.jpg，路径稳定，不会因为文章所在目录层级不同而失效。

### 额外

在与远程仓库进行git连接时，可能会出现main/master等命名混乱，这是由于旧版本一般使用的master，但现在较新版本改为了main，因此若遇见远程与本地仓库名字不一致问题，直接在本地bash中修改：

```
$ git branch -m master main  //将本地master分支改为main

$ git fetch origin从远程仓库 origin //拉取（抓取）最新的所有分支和更新，但不合并到当前本地分支。

$ git branch --set-upstream-to=origin/main main  //让本地 main 跟踪远程 main，这样以后就可以直接用 git pull、git push，不需要加分支名

branch 'main' set up to track 'origin/main'. //上面一句输入后的成功提示

$ git push origin main //推送至远端main
```


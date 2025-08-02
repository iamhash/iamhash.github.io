---
title: 搭建Hexo博客时常见问题与踩坑
date: 2025-08-02 14:31:46
tags: [Hexo,博客搭建,常见错误]

---

## 背景知识：

### Hexo常见的两种部署方式：

- 使用Github Actions自动化部署
- 使用 hexo d 命令（hexo-deployer-git）

【public 目录是 **Hexo 生成的静态文件输出目录**，里面存放了所有网站最终要发布到网上的 HTML、CSS、JS、图片等静态资源。】

<!-- more -->

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



### 额外

在与远程仓库进行git连接时，可能会出现main/master等命名混乱，这是由于旧版本一般使用的master，但现在较新版本改为了main，因此若遇见远程与本地仓库名字不一致问题，直接在本地bash中修改：

```
$ git branch -m master main  //将本地master分支改为main

$ git fetch origin从远程仓库 origin //拉取（抓取）最新的所有分支和更新，但不合并到当前本地分支。

$ git branch --set-upstream-to=origin/main main  //让本地 main 跟踪远程 main，这样以后就可以直接用 git pull、git push，不需要加分支名

branch 'main' set up to track 'origin/main'. //上面一句输入后的成功提示

$ git push origin main //推送至远端main
```

